以下のコードはImportQuestionsFromGithubは、Githubから、questionsのJSONを取得してDBにインサートするコマンドです。
JSONには、skills という配列がありますが、コマンドはまだこちらのインポートには対応出来ていません。
skills 内の id は sklls テーブルのプライマリーキーであるIDを示しています。

ーー
<?php

namespace App\Console\Commands\Import\Github;

use App\Models\Question\Question;
use App\Models\Question\QuestionTranslation;
use App\Models\Level\Level;
use App\Models\Difficulty\Difficulty;
use App\Enums\QuestionType;
use App\Enums\QuestionFormat;
use App\Enums\QuestionStatus;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Str;

/**
 * Import question JSON from a private GitHub repository
 * and upsert into questions / question_translations.
 */
class ImportQuestionsFromGithub extends Command
{
    /**
     * artisan で呼び出すコマンド名
     *
     * 例: php artisan import:questions-from-github --subject=math
     */
    protected $signature = 'import:questions-from-github
                            {--subject= : If specified, only import files under contents/questions/{subject} }';

    protected $description = 'Import question data from a GitHub repository JSON files and sync DB';

    /**
     * メインのハンドラ
     */
    public function handle()
    {
        // 1. GitHub API token を config から取得
        $token = config('services.github.api_token');
        if (!$token) {
            $this->error('GitHub API token not found in config/services.php [github.api_token]');
            return 1; // エラー終了
        }

        // 2. リポジトリとパスの指定
        $repoOwnerAndName = 'NousContentsManagement/masamo-content';

        // ベースパスは contents/questions
        $basePath = 'contents/questions';

        // subject が指定されていれば path を変更
        $subjectOption = $this->option('subject');
        if ($subjectOption) {
            $basePath .= '/' . $subjectOption;
            $this->info("Subject specified: {$subjectOption}, path= {$basePath}");
        } else {
            $this->info("No subject specified. Will import from all subdirectories under contents/questions");
        }

        // 3. ディレクトリを再帰的に辿って .json ファイルを収集
        $allJsonFiles = $this->fetchAllJsonFilesRecursively($repoOwnerAndName, $basePath, $token);

        if (empty($allJsonFiles)) {
            $this->warn("No JSON files found under path: {$basePath}");
            return 0;
        }

        $this->info("Found " . count($allJsonFiles) . " JSON file(s) under {$basePath}");

        // 4. 各 JSON ファイルを処理して DB に upsert
        foreach ($allJsonFiles as $file) {
            $this->processJsonFile($file, $token);
        }

        $this->info("Import process completed.");
        return 0;
    }

    /**
     * 再帰的に GitHub 上のディレクトリを走査し、.json ファイル一覧を返す
     */
    private function fetchAllJsonFilesRecursively(string $repo, string $path, string $token): array
    {
        $url = "https://api.github.com/repos/{$repo}/contents/{$path}";
        $response = Http::withToken($token)->get($url);

        if ($response->failed()) {
            $this->warn("Failed to fetch from GitHub: {$url}, status={$response->status()}");
            return [];
        }

        $items = $response->json();
        if (!is_array($items)) {
            return [];
        }

        $result = [];

        foreach ($items as $item) {
            $type = $item['type'] ?? null;  // 'file' or 'dir'
            $itemPath = $item['path'] ?? '';
            $name     = $item['name'] ?? '';
            // ディレクトリなら再帰的に取得
            if ($type === 'dir') {
                $descendantFiles = $this->fetchAllJsonFilesRecursively($repo, $itemPath, $token);
                $result = array_merge($result, $descendantFiles);
            } elseif ($type === 'file') {
                // .json ファイルかチェック
                if (str_ends_with($name, '.json')) {
                    // 対象に追加
                    $result[] = $item;
                }
            }
        }

        return $result;
    }

    /**
     * JSONファイル一件をダウンロードしてパースし、DBに反映
     */
    private function processJsonFile(array $file, string $token)
    {
        if (!isset($file['download_url'])) {
            $this->warn("No download_url for file: " . ($file['path'] ?? 'unknown'));
            return;
        }

        $this->info("Fetching JSON: " . $file['path']);

        // JSON ファイル内容を取得
        $jsonContent = Http::withToken($token)
            ->get($file['download_url'])
            ->body();

        $decoded = json_decode($jsonContent, true);
        if (!is_array($decoded)) {
            $this->warn("File {$file['name']} is not valid JSON. Skipped.");
            return;
        }

        // 今回は 1ファイル=1問題 という想定
        // もし複数問題があるならループで回す

        // JSON内に "id" が必須
        if (!isset($decoded['id'])) {
            $this->warn("JSON missing 'id' field. Skipped. File={$file['name']}");
            return;
        }

        $this->upsertQuestion($decoded);
    }

    /**
     * 単一の問題(JSON)をDBへ upsert
     */
    private function upsertQuestion(array $questionJson)
    {
        DB::beginTransaction();
        try {
            $jsonId = $questionJson['id'] ?? null;
            if (!$jsonId) {
                $this->warn("Question JSON missing 'id'. Skipped.");
                DB::rollBack();
                return;
            }

            // 既存レコードを検索 (json_id = $jsonId)
            /** @var Question|null $question */
            $question = Question::where('json_id', $jsonId)->first();

            // JSON に含まれる order、level_id, difficulty_id, version, status
            $jsonOrder   = $questionJson['order'] ?? 9999;
            $version     = $questionJson['version'] ?? '0.0.1';

            // 追加: status を QuestionStatus に変換 (なければデフォルト DRAFT=1)
            $rawStatus   = $questionJson['status'] ?? null;
            $statusValue = $rawStatus
                ? $this->parseQuestionStatus($rawStatus)
                : QuestionStatus::DRAFT->value;

            // level_id / difficulty_id は JSON 内の "lev_XXX", "diff_XXX" など
            // levels / difficulties テーブルの json_id と照合して実際のUUIDを取得
            $levelUuid      = $this->findLevelUuid($questionJson['level_id'] ?? null);
            $difficultyUuid = $this->findDifficultyUuid($questionJson['difficulty_id'] ?? null);

            // question_type, question_format を enum に変換
            // "problem_type" を優先し、なければ "question_type"
            $rawQuestionType = $questionJson['problem_type'] ?? $questionJson['question_type'] ?? null;
            $rawFormat       = $questionJson['question_format'] ?? null;

            $questionTypeValue   = $this->parseQuestionType($rawQuestionType);
            $questionFormatValue = $this->parseQuestionFormat($rawFormat);

            // 新規 or 既存
            if (!$question) {
                // 新規作成
                $question = new Question();
                $question->id      = (string) Str::uuid();
                $question->json_id = $jsonId;

                // order の重複回避: DB内で最大の order + 1 と、json側 order を比較
                $maxOrder = Question::max('order');
                if ($maxOrder === null) {
                    $maxOrder = 0;
                }
                $finalOrder = max($maxOrder + 1, $jsonOrder);

                // 衝突を回避
                while (Question::where('order', $finalOrder)->exists()) {
                    $finalOrder++;
                }
                $question->order = $finalOrder;
            } else {
                // 既存レコードの場合 → order を更新 (衝突を回避)
                $this->reorderQuestion($question->id, $jsonOrder);
            }

            // 項目をセット
            $question->level_id        = $levelUuid;
            $question->difficulty_id   = $difficultyUuid;
            $question->version         = $version;
            $question->question_type   = $questionTypeValue;   // enum値(int)
            $question->question_format = $questionFormatValue; // enum値(int)
            $question->status          = $statusValue;

            // question->metadata = JSONで受け取った metadata
            $question->metadata = isset($questionJson['metadata'])
                ? json_encode($questionJson['metadata'], JSON_UNESCAPED_UNICODE)
                : null;

            // generated_by_llm は JSON内にある場合その値を優先
            // or  fallback としてevaluationMethod=LLMなら true
            if (array_key_exists('generated_by_llm', $questionJson)) {
                $question->generated_by_llm = (bool)$questionJson['generated_by_llm'];
            } else {
                $evaluationMethod = $questionJson['metadata']['evaluationMethod'] ?? null;
                $question->generated_by_llm = ($evaluationMethod === 'LLM');
            }

            // learning_requirements (複数) → 改行でまとめる
            $this->applyLearningRequirements($question, $questionJson);

            // セーブ
            $question->save();

            // question_text / explanation を question_translations に upsert
            $this->upsertQuestionTranslations($question, $questionJson);

            DB::commit();
            $this->info("Upserted question json_id={$jsonId}");
        } catch (\Throwable $e) {
            DB::rollBack();
            $this->error("Error upserting question json_id={$jsonId}: " . $e->getMessage());
        }
    }

    /**
     * DB内の既存 question の並び順(order)を JSONの order に合わせる
     * 衝突があればインクリメントして隙間を作る
     */
    private function reorderQuestion(string $questionId, int $targetOrder)
    {
        $question = Question::find($questionId);
        if (!$question) {
            return;
        }

        while (Question::where('order', $targetOrder)->where('id', '!=', $questionId)->exists()) {
            $targetOrder++;
        }
        $question->order = $targetOrder;
        $question->save();
    }

    /**
     * JSON の question_text, explanation を locale ごとに question_translations へ upsert
     */
    private function upsertQuestionTranslations(Question $question, array $questionJson)
    {
        // JSON内の question_text, explanation は { "ja": "...", "en": "..."} 等
        // 必要な言語を列挙
        $locales = ['ja','en'];

        foreach ($locales as $locale) {
            $qText = $questionJson['question_text'][$locale] ?? null;
            $qExp  = $questionJson['explanation'][$locale]   ?? null;

            if ($qText !== null || $qExp !== null) {
                QuestionTranslation::updateOrCreate(
                    [
                        'question_id' => $question->id,
                        'locale'      => $locale,
                    ],
                    [
                        'question_text' => $qText,
                        'explanation'   => $qExp,
                    ]
                );
            }
        }
    }

    /**
     * JSONの "learning_requirements" をテーブルに反映
     *
     * 配列の場合、複数要素を改行区切りで連結
     */
    private function applyLearningRequirements(Question $question, array $questionJson)
    {
        $items = $questionJson['learning_requirements'] ?? [];
        if (!is_array($items) || empty($items)) {
            return;
        }

        $subjects               = [];
        $nos                    = [];
        $requirements           = [];
        $requiredCompetencies   = [];
        $categories             = [];
        $gradeLevels            = [];
        $urls                   = [];

        foreach ($items as $lr) {
            if (isset($lr['learning_subject'])) {
                $subjects[] = $lr['learning_subject'];
            }
            if (isset($lr['learning_no'])) {
                $nos[] = (string)$lr['learning_no'];
            }
            if (isset($lr['learning_requirement'])) {
                $requirements[] = $lr['learning_requirement'];
            }
            if (isset($lr['learning_required_competency'])) {
                $requiredCompetencies[] = $lr['learning_required_competency'];
            }
            if (isset($lr['learning_category'])) {
                $categories[] = $lr['learning_category'];
            }
            if (isset($lr['learning_grade_level'])) {
                $gradeLevels[] = $lr['learning_grade_level'];
            }
            if (isset($lr['learning_url'])) {
                $urls[] = $lr['learning_url'];
            }
        }

        // 改行連結
        $question->learning_subject             = implode("\n", $subjects);
        $question->learning_no                  = !empty($nos) ? (int)$nos[0] : null;
        $question->learning_requirement         = implode("\n", $requirements);
        $question->learning_required_competency = implode("\n", $requiredCompetencies);
        $question->learning_category            = implode("\n", $categories);
        $question->learning_grade_level         = implode("\n", $gradeLevels);
        $question->learning_url                 = implode("\n", $urls);
    }

    /**
     * JSON の level_id (例: "lev_002") から levels テーブル (json_idカラム) を検索 → uuid を返す
     */
    private function findLevelUuid(?string $levelJsonId): ?string
    {
        if (!$levelJsonId) {
            return null;
        }
        return Level::where('json_id', $levelJsonId)->value('id') ?: null;
    }

    /**
     * JSON の difficulty_id (例: "diff-001") から difficulties テーブル (json_idカラム) を検索 → uuid を返す
     */
    private function findDifficultyUuid(?string $difficultyJsonId): ?string
    {
        if (!$difficultyJsonId) {
            return null;
        }
        return Difficulty::where('json_id', $difficultyJsonId)->value('id') ?: null;
    }

    /**
     * question_type 文字列 を Enum (int値) に変換
     * e.g. "FILL_IN_THE_BLANK", "SCENARIO"
     */
    private function parseQuestionType(?string $typeString): int
    {
        if (!$typeString) {
            // デフォルト: CALCULATION
            return QuestionType::CALCULATION->value;
        }

        try {
            return QuestionType::fromString($typeString)->value;
        } catch (\InvalidArgumentException) {
            // 例外なら デフォルトにフォールバック
            return QuestionType::CALCULATION->value;
        }
    }

    /**
     * question_format 文字列 を Enum (int値) に変換
     * e.g. "NUMBER", "NUMERIC_ANSWER"
     */
    private function parseQuestionFormat(?string $formatString): int
    {
        if (!$formatString) {
            // デフォルト: MULTIPLE_CHOICE
            return QuestionFormat::MULTIPLE_CHOICE->value;
        }

        // "NUMBER" も "NUMERIC_ANSWER" とみなす例など
        $normalized = match ($formatString) {
            'NUMBER' => 'NUMERIC_ANSWER',
            default => $formatString
        };

        try {
            return QuestionFormat::fromString($normalized)->value;
        } catch (\InvalidArgumentException) {
            // 例外ならデフォルトにフォールバック
            return QuestionFormat::MULTIPLE_CHOICE->value;
        }
    }

    /**
     * status 文字列を QuestionStatus enum (int) に変換
     */
    private function parseQuestionStatus(string $statusString): int
    {
        try {
            return QuestionStatus::fromString($statusString)->value;
        } catch (\InvalidArgumentException) {
            // 不明ならとりあえず DRAFT
            return QuestionStatus::DRAFT->value;
        }
    }
}

<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('questions', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('level_id');
            $table->uuid('difficulty_id');
            $table->string('json_id')->nullable()->unique();
            $table->json('metadata')->nullable();
            $table->string('version')->default('0.0.1');
            $table->integer('status')->default(1);
            $table->integer('question_type')->default(1);
            $table->integer('question_format')->default(1)->comment('出題形式: 1=選択式,2=数値回答,3=テキスト回答,4=画像選択など');
            $table->string('learning_subject')->nullable()
                ->comment('科目 (学習要件) e.g. "Arithmetic"');
            $table->integer('learning_no')->nullable()
                ->comment('学習要件の番号 e.g. 10');
            $table->text('learning_requirement')->nullable()
                ->comment('学習要件の内容 "Numbers and Calculation..."');
            $table->text('learning_required_competency')->nullable()
                ->comment('必要水準 "Understand multiplication..."');
            $table->string('learning_category')->nullable()
                ->comment('分類 e.g. "A", "B"');
            $table->string('learning_grade_level')->nullable()
                ->comment('学年 e.g. "Grade 2"');
            $table->string('learning_url')->nullable()
                ->comment('URLリンク e.g. "https://docs.google.com/..."');
            $table->integer('order');
            $table->boolean('generated_by_llm')->default(false);
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('level_id')->references('id')->on('levels')->onDelete('cascade');
            $table->foreign('difficulty_id')->references('id')->on('difficulties')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        DB::statement('SET FOREIGN_KEY_CHECKS=0;');
        Schema::dropIfExists('questions');
        DB::statement('SET FOREIGN_KEY_CHECKS=1;');
    }
};

<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('question_translations', function (Blueprint $table) {
            $table->id();
            $table->uuid('question_id');
            $table->string('locale', 10);
            $table->longText('question_text')->nullable();
            $table->longText('explanation')->nullable();
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('question_id')->references('id')->on('questions')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('question_translations');
    }
};
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('question_skill', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('question_id');
            $table->uuid('skill_id');
            $table->integer('order');
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('question_id')->references('id')->on('questions')->onDelete('cascade');
            $table->foreign('skill_id')->references('id')->on('skills')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('question_skill');
    }
};

<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('skills', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('skill_id');
            $table->string('locale', 10);
            $table->string('version')->default('0.0.1');
            $table->string('display_name')->nullable();
            $table->integer('status')->default(1);
            $table->integer('order');
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('skill_id')->references('id')->on('skills')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('skills');
    }
};

<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('skill_translations', function (Blueprint $table) {
            $table->id();
            $table->uuid('skill_id');
            $table->string('locale', 10);
            $table->string('name')->nullable();
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('skill_id')->references('id')->on('skills')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('skill_translations');
    }
};

ーー questionsのjson
{
  "order": 1,
  "id": "ques_s1_l2_qt51_001",
  "level_id": "lev_002",
  "difficulty_id": "diff_001",
  "version": "1.0.0",
  "question_type": "FILL_IN_THE_BLANK",
  "question_format": "NUMBER",
  "question_text": {
    "ja": "▢にあてはまる数を答えなさい。",
    "en": "Please answer the numbers that fit in the blanks."
  },
  "explanation": {
    "ja": "",
    "en": ""
  },
  "skills": [
    {
      "id": "e7174276-c017-4b9f-aae6-eae4251a19fa",
      "name": "知識・技能"
    }
  ],
  "metadata": {
    "question": "8 × 4 = ▢ × 8 = ▢",
    "inputFormat": {
      "type": "fixed",
      "fields": [
        {
          "field_id": "f_1",
          "attribute": "number",
          "collect_answer": "4"
        },
        {
          "field_id": "f_2",
          "attribute": "number",
          "collect_answer": "4"
        }
      ],
      "question_components": [
        {
          "type": "text",
          "content": "8 × 4 = "
        },
        {
          "type": "blank",
          "field_id": "f_1"
        },
        {
          "type": "text",
          "content": " × 8 = "
        },
        {
          "type": "blank",
          "field_id": "f_2"
        }
      ]
    },
    "": [],
    "evaluationMethod": "LLM"
  },
  "learning_requirements": [
    {
      "learning_subject": "算数",
      "learning_no": 10,
      "learning_requirement": "数と計算 計算の意味・方法 乗法の意味",
      "learning_required_competency": "かけ算は同じ数の繰り返し加法であるという本質を理解できる",
      "learning_category": "A",
      "learning_grade_level": "小2",
      "learning_url": "https://docs.google.com/spreadsheets/d/1W5vaFHcyU_BrMwb1JLZ-DpyFmXXZPaYMTHIITcwLqqY/edit?gid=0#gid=0&range=15:15"
    }
  ],
  "status": "PUBLISHED",
  "generated_by_llm": false,
  "created_at": "2025-01-01 00:00:00",
  "updated_at": "2025-01-01 00:00:00"
}
