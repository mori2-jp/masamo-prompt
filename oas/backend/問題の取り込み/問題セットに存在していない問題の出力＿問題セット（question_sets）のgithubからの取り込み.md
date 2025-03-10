以下の、ImportQuestionSetsFromGithub は、問題セットの取り込みと問題セットと問題の紐づけ（question_set_questions）を行うスクリプトです。

スクリプトの実行完了時に、１つの問題セットにも紐づけが行われていない問題の json_id の一覧をコンソールに出力するコードを追加してください。
また、この修正のブランチ名も作成して。


# 実装方針
生成また修正変更するコードはそのコードだけは必ず省略せずに必ず全てアウトプットすること。

仕様はソースコードにまとめてコメントとして残すこと

ーーー 問題セットJSON
```json

{
  "json_id": "qset_s1_g2_sec800_u100_v100_100",
  "unit_id": "unit_s1_g2_sec800_100",
  "unit": {
    "ja": "加法の交換法則を式で示す",
    "en": "Express the commutative property of addition in equations"
  },
  "title": {
    "ja": "",
    "en": ""
  },
  "description": {
    "ja": "",
    "en": ""
  },
  "order": 100,
  "version": "1.0.0",
  "status": "PUBLISHED",
  "questions": [
    "ques_s1_g2_sec800_u100_diff100_qt51_v100_100",
    "ques_s1_g2_sec800_u100_diff100_qt51_v100_200"
  ]
}

```


ーーー　DB構造
```php
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
        Schema::create('question_sets', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('unit_id')->nullable();
            $table->string('json_id')->nullable()->unique();
            $table->string('version')->default('0.0.1');
            $table->integer('status')->default(1);
            $table->integer('order');
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('unit_id')->references('id')->on('units')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        DB::statement('SET FOREIGN_KEY_CHECKS=0;');
        Schema::dropIfExists('question_sets');
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
        Schema::create('questions', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('level_id');
            $table->uuid('grade_id');
            $table->uuid('difficulty_id');
            $table->string('json_id')->nullable()->unique();
            $table->json('metadata')->nullable();
            $table->string('version')->default('0.0.1');
            $table->integer('status')->default(1);
            $table->integer('evaluation_method')->default(1);
            $table->integer('checker_method')->nullable();
            $table->integer('llm_evaluation_prompt_number')->nullable();
            $table->json('evaluation_response_format')->nullable();
            $table->integer('question_type')->default(1);
            $table->json('learning_requirement_json')->nullable();
            $table->string('learning_subject')->nullable()
                ->comment('科目 (学習要件)');
            $table->integer('learning_no')->nullable()
                ->comment('学習要件の番号');
            $table->text('learning_requirement')->nullable()
                ->comment('学習要件の内容');
            $table->text('learning_required_competency')->nullable()
                ->comment('必要水準');
            $table->text('learning_background')->nullable()
                ->comment('背景・補足');
            $table->string('learning_category')->nullable()
                ->comment('分類');
            $table->string('learning_grade_level')->nullable()
                ->comment('学年');
            $table->string('learning_url')->nullable()
                ->comment('URLリンク');
            $table->integer('order');
            $table->boolean('generated_by_llm')->default(false);
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('level_id')->references('id')->on('levels')->onDelete('cascade');
            $table->foreign('grade_id')->references('id')->on('grades')->onDelete('cascade');
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
        Schema::create('question_set_questions', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('question_set_id');
            $table->uuid('question_id');
            $table->integer('order');
            $table->timestamps();

            $table->foreign('question_set_id')->references('id')->on('question_sets')->onDelete('cascade');
            $table->foreign('question_id')->references('id')->on('questions')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('question_set_questions');
    }
};

```


ーーー 取り込みスクリプト
```php
<?php

namespace App\Console\Commands\Import\Github;

use App\Models\Question\Question;
use App\Models\Question\QuestionSet;
use App\Models\Question\QuestionSetTranslation;
use App\Enums\QuestionSetStatus;
use App\Models\Unit\Unit;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Str;

/**
 * Import question_set JSON from a private GitHub repository
 * and upsert into question_sets / question_set_translations / question_set_questions pivot table.
 */
class ImportQuestionSetsFromGithub extends Command
{
    /**
     * artisan で呼び出すコマンド名
     *
     * 例: php artisan import:question-sets-from-github --subject=math
     */
    protected $signature = 'import:question-sets-from-github
                            {--subject= : If specified, only import files under contents/question_sets/{subject} }';

    protected $description = 'Import question set data from a GitHub repository JSON files and sync DB';

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

        // ベースパスは contents/question_sets
        $basePath = 'contents/question_sets';

        // subject が指定されていれば path を変更
        $subjectOption = $this->option('subject');
        if ($subjectOption) {
            $basePath .= '/' . $subjectOption;
            $this->info("Subject specified: {$subjectOption}, path= {$basePath}");
        } else {
            $this->info("No subject specified. Will import from all subdirectories under contents/question_sets");
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
            $type = $item['type'] ?? null; // 'file' or 'dir'
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

        // 今回は 1ファイル=1セット という想定
        if (!isset($decoded['json_id'])) {
            $this->warn("question_set JSON missing 'json_id'. Skipped. File={$file['name']}");
            return;
        }

        $this->upsertQuestionSet($decoded);
    }

    /**
     * question_set JSON を DB へ upsert
     */
    private function upsertQuestionSet(array $setJson)
    {
        DB::beginTransaction();
        try {
            $jsonId = $setJson['json_id'] ?? null;
            // 既存レコードを検索
            /** @var QuestionSet|null $qset */
            $qset = QuestionSet::where('json_id', $jsonId)->first();

            // order
            $jsonOrder = $setJson['order'] ?? 9999;

            // version
            $version   = $setJson['version'] ?? '0.0.1';

            // status
            $rawStatus = $setJson['status'] ?? 'DRAFT';
            $statusVal = $this->parseQuestionSetStatus($rawStatus);

            // unit_id が空、または一致する Unit が無い場合は skip
            if (empty($setJson['unit_id'])) {
                $this->warn("No unit_id provided - skipping question_set");
                DB::rollBack();
                return;
            }
            // unit_id (JSON内の) → units テーブルの json_id に対応する ID を取得
            // e.g. "unit_s1_l1_001" → DBのunits.json_id = "unit_s1_l1_001"
            $unitUuid = $this->findUnitUuid($setJson['unit_id']);
            if (!$unitUuid) {
                $this->warn("Unit not found for json_id={$setJson['unit_id']} - skipping question_set");
                DB::rollBack();
                return;
            }


            // 新規 or 既存
            if (!$qset) {
                // 新規
                $qset = new QuestionSet();
                $qset->id      = (string) Str::uuid();
                $qset->json_id = $jsonId;

                // order の重複回避: DB内最大の order +1 と json側 order を比較
                $maxOrder = QuestionSet::max('order');
                if ($maxOrder === null) {
                    $maxOrder = 0;
                }
                $finalOrder = max($maxOrder + 1, $jsonOrder);
                // 衝突回避
                while (QuestionSet::where('order', $finalOrder)->exists()) {
                    $finalOrder++;
                }
                $qset->order = $finalOrder;
            } else {
                // 既存 → order更新 (衝突回避)
                $this->reorderQuestionSet($qset->id, $jsonOrder);
            }

            $qset->unit_id = $unitUuid;
            $qset->version = $version;
            $qset->status  = $statusVal;
            $qset->save();

            // 多言語 title / description → question_set_translations
            $this->upsertSetTranslations($qset, $setJson);

            // questions[] → pivot へ保存
            // "questions" が存在する場合、questionの json_id 配列として想定
            if (isset($setJson['questions']) && is_array($setJson['questions'])) {
                $this->syncQuestionSetQuestions($qset, $setJson['questions']);
            }

            DB::commit();
            $this->info("Upserted question_set json_id={$jsonId}");
        } catch (\Throwable $e) {
            DB::rollBack();
            $this->error("Error upserting question_set json_id={$jsonId}: " . $e->getMessage());
        }
    }

    /**
     * 既存 question_set の並び順(order)を JSONの order に合わせる
     * 衝突があればインクリメントして隙間を作る
     */
    private function reorderQuestionSet(string $setId, int $targetOrder)
    {
        $qset = QuestionSet::find($setId);
        if (!$qset) {
            return;
        }

        while (QuestionSet::where('order', $targetOrder)
            ->where('id', '!=', $setId)
            ->exists()) {
            $targetOrder++;
        }
        $qset->order = $targetOrder;
        $qset->save();
    }

    /**
     * question_set_translations に title / description を upsert
     */
    private function upsertSetTranslations(QuestionSet $qset, array $setJson)
    {
        // title, description は { "ja": "...", "en": "..."} など
        $locales = ['ja','en'];

        foreach ($locales as $locale) {
            $title       = $setJson['title'][$locale]       ?? null;
            $description = $setJson['description'][$locale] ?? null;

            if ($title !== null || $description !== null) {
                QuestionSetTranslation::updateOrCreate(
                    [
                        'question_set_id' => $qset->id,
                        'locale'          => $locale,
                    ],
                    [
                        'title'       => $title,
                        'description' => $description,
                    ]
                );
            }
        }
    }

    /**
     * question_set_questions テーブルへの関連付け
     * JSONに含まれる questions (配列)を取得し、既存/新規を調整
     */
    private function syncQuestionSetQuestions(QuestionSet $qset, array $questionJsonIds)
    {
        // まず question_set_questions から既存レコードを削除 or 置き換えたい場合 → 一旦消すか、個別に更新。
        // 例として 全削除→再インサートで実装してみる
        DB::table('question_set_questions')
            ->where('question_set_id', $qset->id)
            ->delete();

        // JSON での順番順にマッピング
        // ex) questionJsonIds = ["question_s1_l2_001", "question_s1_l2_002"]
        foreach ($questionJsonIds as $idx => $qJsonId) {
            $question = Question::where('json_id', $qJsonId)->first();
            if (!$question) {
                // 見つからなければスキップ or エラー
                $this->warn("Question not found for json_id={$qJsonId}");
                continue;
            }

            // pivot の順番 (JSONリストでのindex+1 とか)
            // ※「さらに衝突回避が必要」という要件があれば加味
            $pivotOrder = $idx + 1;

            // pivot にINSERT
            DB::table('question_set_questions')->insert([
                'id' => (string) Str::uuid(),
                'question_set_id' => $qset->id,
                'question_id' => $question->id,
                'order' => $pivotOrder,
                'created_at' => now(),
                'updated_at' => now(),
            ]);
        }
    }

    /**
     * JSON の unit_id (例: "unit_s1_l1_001") から units テーブル (json_idカラム) を検索 → uuid を返す
     */
    private function findUnitUuid(?string $unitJsonId): ?string
    {
        if (!$unitJsonId) {
            return null;
        }
        return Unit::where('json_id', $unitJsonId)->value('id') ?: null;
    }

    /**
     * status 文字列を QuestionSetStatus enum(int) に変換
     */
    private function parseQuestionSetStatus(string $statusString): int
    {
        try {
            return QuestionSetStatus::fromString($statusString)->value;
        } catch (\InvalidArgumentException) {
            // 不明なら DRAFT=1
            return QuestionSetStatus::DRAFT->value;
        }
    }
}

```
