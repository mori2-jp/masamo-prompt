以下、EvaluationCheckService　の　checkByExactMatch　は、問題JSONの、checker_methodがCHECK_BY_EXACT_MATCHの時に
evaluation_spec.response_format.fieldsのcollect_answer が question に
以下のルールに適合しているかをバリデーションするメソッドです。
ImportQuestionsFromGithub　は、問題JSONをDBに取り込みするコマンドです。

QuestionJsonManageService　の　validateQuestionJson　を使って、ImportQuestionsFromGithubで問題JSONをDBに取り込みする際に、
問題JSONがルールに適合しているかバリデーションする処理を追加してください。
 
UseCase などの実装は不用で、ImportQuestionsFromGithubに直接処理を追加してください


--- 問題JSONの全体構造
```json
{
  "order": 900,
  "id": "ques_s1_g3_sec100_u300_diff100_qt51_v100_900",
  "level_id": "lev_003",
  "grade_id": "gra_003",
  "difficulty_id": "diff_100",
  "version": "1.0.0",
  "status": "PUBLISHED",
  "generated_by_llm": false,
  "created_at": "2025-01-01 10:00:00",
  "updated_at": "2025-01-01 10:00:00",
  "skills": [
    {
      "skill_id": "sk_004",
      "name": "知識・技能"
    }
  ],
  "learning_requirements": [
    {
      "learning_subject": "算数",
      "learning_no": 37,
      "learning_requirement": "計算の意味・方法 大きな数の概念と活用 3位数や4位数の加法及び減法",
      "learning_required_competency": "・3～4桁どうしの足し算・引き算を繰り上がり・繰り下がり含め正確に計算できる。",
      "learning_background": "・筆算の手順をしっかり確立させる。",
      "learning_category": "A",
      "learning_grade_level": "小3"
    }
  ],
  "evaluation_spec": {
    "evaluation_method": "CODE",
    "checker_method": "CHECK_BY_EXACT_MATCH",
    "response_format": {
      "is_correct": "boolean",
      "score": "number",
      "question_text": {
        "ja": "つぎの ▢ にあてはまる数を答えなさい。",
        "en": "Please answer the numbers that fit in the blanks."
      },
      "explanation": {
        "ja": "4桁どうしの足し算を、位ごとに分けて計算する問題です。くり上がりがあっても、各位をしっかりそろえて考えましょう。マイナスの数が出ないよう、丁寧に位を確かめながら計算してください。",
        "en": "This problem involves adding two four-digit numbers, split by each place (thousands, hundreds, tens, and ones). Even if carrying occurs, be sure to align each place carefully. Make sure no negative values appear; check your place values carefully."
      },
      "question": {
        "ja": "6427 + 2314 = (6000 + 2000) + (400 + 300) + (20 + 10) + (7 + 4) = ▢ + ▢ + ▢ + ▢ = ▢",
        "en": "6427 + 2314 = (6000 + 2000) + (400 + 300) + (20 + 10) + (7 + 4) = ▢ + ▢ + ▢ + ▢ = ▢"
      },
      "fields": [
        {
          "field_id": "f_1",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": 8000,
          "field_explanation": {
            "ja": "千のくらいは 6000 と 2000 を足して 8000 になります。",
            "en": "For the thousands place, adding 6000 and 2000 gives 8000."
          }
        },
        {
          "field_id": "f_2",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": 700,
          "field_explanation": {
            "ja": "百のくらいは 400 と 300 を足して 700 になります。",
            "en": "For the hundreds place, adding 400 and 300 gives 700."
          }
        },
        {
          "field_id": "f_3",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": 30,
          "field_explanation": {
            "ja": "十のくらいは 20 と 10 を足して 30 になります。",
            "en": "For the tens place, adding 20 and 10 results in 30."
          }
        },
        {
          "field_id": "f_4",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": 11,
          "field_explanation": {
            "ja": "一のくらいは 7 と 4 を足して 11 になります。",
            "en": "For the ones place, adding 7 and 4 gives 11."
          }
        },
        {
          "field_id": "f_5",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": 8741,
          "field_explanation": {
            "ja": "8000 + 700 + 30 + 11 を合計すると 8741 です。",
            "en": "Adding 8000, 700, 30, and 11 results in 8741."
          }
        }
      ]
    }
  },
  "metadata": {
    "question_type": "FILL_IN_THE_BLANK",
    "question_format": "NUMERIC_ANSWER",
    "question_text": {
      "ja": "つぎの ▢ にあてはまる数を答えなさい。",
      "en": "Please answer the numbers that fit in the blanks."
    },
    "explanation": {
      "ja": "4桁 + 4桁のたし算を各桁に分けて考える筆算の問題です。くり上がりがあるときも、位をきちんと整理して計算してみましょう。",
      "en": "This is a column addition exercise for two four-digit numbers. Split each place and keep track of any carry carefully."
    },
    "background": {
      "ja": "4桁同士の足し算で繰り上がりを正確に計算できるようになることを目指しています。位を整理して筆算の手順を理解するための問題です。",
      "en": "The goal is to practice carrying over accurately when adding two four-digit numbers. By splitting each place value, learners can reinforce the correct column addition process."
    },
    "question": {
      "ja": "6427 + 2314 = (6000 + 2000) + (400 + 300) + (20 + 10) + (7 + 4) = ▢ + ▢ + ▢ + ▢ = ▢",
      "en": "6427 + 2314 = (6000 + 2000) + (400 + 300) + (20 + 10) + (7 + 4) = ▢ + ▢ + ▢ + ▢ = ▢"
    },
    "input_format": {
      "type": "fixed",
      "fields": [
        {
          "field_id": "f_1",
          "attribute": "number",
          "collect_answer": "8000",
          "user_answer": "number"
        },
        {
          "field_id": "f_2",
          "attribute": "number",
          "collect_answer": "700",
          "user_answer": "number"
        },
        {
          "field_id": "f_3",
          "attribute": "number",
          "collect_answer": "30",
          "user_answer": "number"
        },
        {
          "field_id": "f_4",
          "attribute": "number",
          "collect_answer": "11",
          "user_answer": "number"
        },
        {
          "field_id": "f_5",
          "attribute": "number",
          "collect_answer": "8741",
          "user_answer": "number"
        }
      ],
      "question_components": [
        {
          "type": "text",
          "content": {
            "ja": "6427 + 2314 = ",
            "en": "6427 + 2314 = "
          },
          "order": 50
        },
        {
          "type": "newline",
          "order": 100
        },
        {
          "type": "text",
          "content": {
            "ja": "(6000 + 2000) + (400 + 300) + (20 + 10) + (7 + 4) = ",
            "en": "(6000 + 2000) + (400 + 300) + (20 + 10) + (7 + 4) = "
          },
          "order": 150
        },
        {
          "type": "newline",
          "order": 200
        },
        {
          "type": "input_field",
          "field_id": "f_1",
          "order": 250
        },
        {
          "type": "text",
          "content": {
            "ja": " + ",
            "en": " + "
          },
          "order": 300
        },
        {
          "type": "input_field",
          "field_id": "f_2",
          "order": 350
        },
        {
          "type": "text",
          "content": {
            "ja": " + ",
            "en": " + "
          },
          "order": 400
        },
        {
          "type": "input_field",
          "field_id": "f_3",
          "order": 450
        },
        {
          "type": "text",
          "content": {
            "ja": " + ",
            "en": " + "
          },
          "order": 500
        },
        {
          "type": "input_field",
          "field_id": "f_4",
          "order": 550
        },
        {
          "type": "text",
          "content": {
            "ja": " = ",
            "en": " = "
          },
          "order": 600
        },
        {
          "type": "input_field",
          "field_id": "f_5",
          "order": 650
        }
      ]
    }
  }
}

```



# 実装方針
生成また修正変更するコードはそのコードだけは必ず省略せずに必ず全てアウトプットすること。

コードは密結合は避けて、Testableなコードにすること

仕様はソースコードにまとめてコメントとして残すこと

ビジネスロジックは Model へ、
ビジネスロジックへのアクセスは service を介して行うこと。

アーキテクチャ構成は
- **アーキテクチャ構成(クリーンアーキテクチャ)**：
    - - **Model**
        - ビジネスロジックを記述
    - **Controller**
        - リクエストを受け取り、適切なUseCaseを呼び出します。
    - **UseCase**
        - 特定のユースケース（機能）に対するビジネスロジックを実装し、必要なServiceを利用して処理を行います。
    - **Service**
        - データアクセスや共通のビジネスロジックを実装します。データの取得や保存を担当し、Modelと連携します。
    - **DTO（Data Transfer Object）**
        - レイヤー間でデータを転送するためのオブジェクトで、データの構造を明示的に定義します。
    - **Resource**
        - APIレスポンスはResourceで定義する
          のように実装してください。


# 説明
question_sets：問題（questions）を束ねるグループ
questions: 問題
question_set_questions：question_sets と questions を紐づけるPivotテーブル。questions は複数のquestion_sets に紐づく事があり多対多の関係なのでこのような設計になっている。
user_question_sets: ユーザーが学習した questions_sets。学習開始時に question_sets_id と紐づいて status （UserQuestionSetsStatus::NOT_START) が未開始の状態で生成され、進捗ステータスはスコアなどが管理される
user_questions: ユーザーが学習した questions。学習開始時に、学習を開始した question_sets に紐づく questions が全て、 user_question_sets_id と question_id（questions）を紐づけて status （UserQuestionStatus::NOT_START) が未開始の状態で全てのquestionsの数の分、生成され、進捗ステータスはスコアなどが管理される

# ディレクトリ構成
masamo-server
├── app
│   ├── Console
│   │   └── Commands
│   ├── Dtos
│   ├── Enums
│   ├── Http
│   │   ├── Controllers
│   │   ├── Middleware
│   │   ├── Requests
│   │   └── Resources
│   ├── Models
│   ├── Providers
│   ├── Services
│   ├── Traits
│   └── UseCases
├── bootstrap
├── config
├── database
│   ├── factories
│   ├── migrations
│   ├── seeders
│   ├── .gitignore
│   └── database.sqlite
├── docs
├── lang
│   ├── en
│   └── ja
├── public
├── resources
│   ├── css
│   ├── js
│   ├── prompts
│   └── views
├── routes
├── storage
├── tests
└── vendor

ーーー 問題JSONの取り込み
```php
<?php

namespace App\Console\Commands\Import\Github;

use App\Enums\EvaluationCheckerMethod;
use App\Enums\EvaluationMethod;
use App\Models\Grade\Grade;
use App\Models\Question\Question;
use App\Models\Question\QuestionTranslation;
use App\Models\Level\Level;
use App\Models\Difficulty\Difficulty;
use App\Enums\QuestionType;
use App\Enums\QuestionStatus;
use App\Services\Utils\Question\QuestionJsonManageService;
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
     * 「取り込もうとしたJSONの総数」
     * @var int
     */
    private int $totalCount = 0;

    /**
     * 「取り込みに失敗またはスキップしたJSONの総数」
     * @var int
     */
    private int $skipCount = 0;

    /**
     * 「取り込みに成功したJSONの総数」
     * @var int
     */
    private int $successCount = 0;

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

        // インポート処理がすべて完了したあと、集計結果を出力
        $this->info("Import process completed.");
        $this->line("=======================================");
        $this->info("取り込もうとしたJSONの総数: {$this->totalCount}");
        $this->info("取り込みに失敗またはスキップしたJSONの総数: {$this->skipCount}");
        $this->info("取り込みに成功したJSONの総数: {$this->successCount}");
        $this->line("=======================================");

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
    private function processJsonFile(array $file, string $token): void
    {
        // 「取り込もうとしたJSONの総数」をまずインクリメント
        $this->totalCount++;

        if (!isset($file['download_url'])) {
            $this->warn("No download_url for file: " . ($file['path'] ?? 'unknown'));
            // ダウンロードURLが無い=スキップとして扱う
            $this->skipCount++;
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
            $this->skipCount++;
            return;
        }

        // 今回は 1ファイル=1問題 という想定
        // もし複数問題があるならループで回す

        // JSON内に "id" が必須
        if (!isset($decoded['id'])) {
            $this->warn("JSON missing 'id' field. Skipped. File={$file['name']}");
            $this->skipCount++;
            return;
        }

        // upsertQuestion() が失敗/スキップとなる場合もあるので、戻り値で成功/失敗を判定
        $result = $this->upsertQuestion($decoded);
        if ($result) {
            // 成功
            $this->successCount++;
        } else {
            // スキップ or 失敗
            $this->skipCount++;
        }
    }

    /**
     * 単一の問題(JSON)をDBへ upsert
     *
     * @return bool 成功ならtrue、スキップ/失敗ならfalse
     */
    private function upsertQuestion(array $questionJson): bool
    {
        DB::beginTransaction();
        try {
            $jsonId = $questionJson['id'] ?? null;
            if (!$jsonId) {
                $this->warn("Question JSON missing 'id'. Skipped.");
                DB::rollBack();
                return false;
            }

            // metadata のバリデーション
            $metadata = $questionJson['metadata'] ?? [];
            // Metadataが無いならスキップ
            if (empty($metadata)) {
                $this->warn("metadata が存在しないためスキップします。json_id={$jsonId}");
                DB::rollBack();
                return false;
            }

            // JSONバリデーション
            try {
                /** @var QuestionJsonManageService $jsonManager */
                $jsonManager = app(QuestionJsonManageService::class);
                $jsonManager->validateQuestionJson($questionJson);
            } catch (\Illuminate\Validation\ValidationException $ve) {
                // バリデーションエラーがあれば、その問題をスキップ
                $this->warn("バリデーションエラーのため問題(json_id={$jsonId})をスキップします。");
                foreach ($ve->errors() as $field => $msgs) {
                    foreach ($msgs as $msg) {
                        $this->warn("  - {$field}: {$msg}");
                    }
                }
                DB::rollBack();
                return false;
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

            // level_id / difficulty_id / grade_id は JSON 内の "lev_XXX", "diff_XXX" 等
            $levelUuid      = $this->findLevelUuid($questionJson['level_id'] ?? null);
            $gradeUuid      = $this->findGradeUuid($questionJson['grade_id'] ?? null);
            $difficultyUuid = $this->findDifficultyUuid($questionJson['difficulty_id'] ?? null);

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
            $question->grade_id        = $gradeUuid;
            $question->difficulty_id   = $difficultyUuid;
            $question->version         = $version;
            $question->status          = $statusValue;

            // question->metadata = JSONで受け取った metadata
            $question->metadata = isset($questionJson['metadata'])
                ? json_encode($questionJson['metadata'], JSON_UNESCAPED_UNICODE)
                : null;

            if (!empty($question->metadata)) {
                // question_type を enum に変換
                $rawQuestionType = $questionJson['metadata']['question_type'] ?? null;
                $questionTypeValue = $this->parseQuestionType($rawQuestionType);
                $question->question_type = $questionTypeValue;
            }

            // 評価
            if (isset($questionJson['evaluation_spec'])) {
                $question->evaluation_method = isset($questionJson['evaluation_spec']['evaluation_method'])
                    ? EvaluationMethod::fromString($questionJson['evaluation_spec']['evaluation_method'])
                    : null;

                $question->checker_method = isset($questionJson['evaluation_spec']['checker_method'])
                    ? EvaluationCheckerMethod::fromString($questionJson['evaluation_spec']['checker_method'])
                    : null;

                $question->llm_evaluation_prompt_number = $questionJson['evaluation_spec']['llm_prompt_number'] ?? null;

                $question->evaluation_response_format = isset($questionJson['evaluation_spec']['response_format'])
                    ? json_encode($questionJson['evaluation_spec']['response_format'], JSON_UNESCAPED_UNICODE)
                    : null;
            }

            // generated_by_llm は JSON内にある場合その値を優先
            if (array_key_exists('generated_by_llm', $questionJson)) {
                $question->generated_by_llm = (bool)$questionJson['generated_by_llm'];
            }

            // learning_requirements (複数) → 改行でまとめる
            $this->applyLearningRequirements($question, $questionJson);

            // セーブ
            $question->save();

            // question_text / explanation を question_translations に upsert
            $this->upsertQuestionTranslations($question, $questionJson);

            // skills → question_skill ピボットを JSON に合わせて同期
            $this->applySkillsPivot($question, $questionJson);

            DB::commit();
            $this->info("Upserted question json_id={$jsonId}");
            // 成功
            return true;

        } catch (\Throwable $e) {
            DB::rollBack();
            $this->error("Error upserting question json_id={$jsonId}: " . $e->getMessage());
            return false;
        }
    }

    /**
     * JSON の "skills" 配列を読み込み、`skills` テーブルの json_id と対応するレコードを探して
     * `question_skill` (Pivot) を JSON 側と完全同期する:
     * - JSON にない既存スキル紐付けは削除
     * - JSON に新規追加されたスキルは挿入
     * - 既存スキルは順番 (order) を更新
     */
    private function applySkillsPivot(Question $question, array $questionJson)
    {
        // JSON中の skills を取得
        $skillsFromJson = $questionJson['skills'] ?? [];

        // 1. JSON に記載されている skill の skill_db_id を取得
        //    また、インデックス順を Pivot の order に反映できるようマップを用意
        $newSkillDbIds = [];
        $skillIndexMap = []; // skillUuid => order（例：1,2,3...）

        foreach ($skillsFromJson as $index => $skillData) {
            $skillJsonId = $skillData['skill_id'] ?? null;
            if (!$skillJsonId) {
                continue; // JSON に skill_id が無ければスキップ
            }

            // skills テーブルの json_id から id (UUID) を取得
            $skillDbId = DB::table('skills')
                ->where('json_id', $skillJsonId)
                ->value('id');

            if (!$skillDbId) {
                // 見つからない場合は警告のみ
                $this->warn("Skill with json_id='{$skillJsonId}' not found. Skipped linking.");
                continue;
            }

            $newSkillDbIds[] = $skillDbId;
            // 順番 (order) は単純にループの順序で設定する例
            $skillIndexMap[$skillDbId] = $index + 1;
        }

        // 2. 現在 DB にある該当 question_id の紐付けを取得
        $existingSkillDbIds = DB::table('question_skill')
            ->where('question_id', $question->id)
            ->pluck('skill_id')
            ->toArray();

        // 3. JSON から消えているスキル を pivot から削除
        //    ( 既存 - 新規 ) = 削除対象
        $skillIdsToRemove = array_diff($existingSkillDbIds, $newSkillDbIds);
        if (!empty($skillIdsToRemove)) {
            DB::table('question_skill')
                ->where('question_id', $question->id)
                ->whereIn('skill_id', $skillIdsToRemove)
                ->delete();
        }

        // 4. JSON にあるスキルを順に「既存ならUPDATE」「なければINSERT」
        foreach ($newSkillDbIds as $skillDbId) {
            // 既にあるか確認
            $existingPivot = DB::table('question_skill')
                ->where('question_id', $question->id)
                ->where('skill_id', $skillDbId)
                ->first();

            if ($existingPivot) {
                // 既存レコードを更新
                DB::table('question_skill')
                    ->where('id', $existingPivot->id)
                    ->update([
                        'order'      => $skillIndexMap[$skillDbId],
                        'updated_at' => now(),
                    ]);
            } else {
                // 新規レコードを挿入
                DB::table('question_skill')->insert([
                    'id'          => (string) Str::uuid(),  // 手動でUUIDを付与
                    'question_id' => $question->id,
                    'skill_id'    => $skillDbId,
                    'order'       => $skillIndexMap[$skillDbId],
                    'created_at'  => now(),
                    'updated_at'  => now(),
                ]);
            }
        }
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
            $qText = $questionJson['metadata']['question_text'][$locale] ?? null;
            $qExp  = $questionJson['metadata']['explanation'][$locale]   ?? null;
            $qBack  = $questionJson['metadata']['background'][$locale]   ?? null;

            if ($qText !== null || $qExp !== null) {
                QuestionTranslation::updateOrCreate(
                    [
                        'question_id' => $question->id,
                        'locale'      => $locale,
                    ],
                    [
                        'question_text' => $qText,
                        'explanation'   => $qExp,
                        'background' => $qBack,
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
        $question->learning_requirement_json = json_encode($items, JSON_UNESCAPED_UNICODE);

        $subjects               = [];
        $nos                    = [];
        $requirements           = [];
        $requiredCompetencies   = [];
        $backgrounds   = [];
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
            if (isset($lr['learning_background'])) {
                $backgrounds[] = $lr['learning_background'];
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
        $question->learning_background          = implode("\n", $backgrounds);
        $question->learning_category            = implode("\n", $categories);
        $question->learning_grade_level         = implode("\n", $gradeLevels);
        $question->learning_url                 = implode("\n", $urls);
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

        while (Question::where('order', $targetOrder)
            ->where('id', '!=', $questionId)
            ->exists()) {
            $targetOrder++;
        }
        $question->order = $targetOrder;
        $question->save();
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
     * JSON の grade_id (例: "gra_002") から grades テーブル (json_idカラム) を検索 → uuid を返す
     */
    private function findGradeUuid(?string $gradeJsonId): ?string
    {
        if (!$gradeJsonId) {
            return null;
        }
        return Grade::where('json_id', $gradeJsonId)->value('id') ?: null;
    }

    /**
     * JSON の difficulty_id (例: "diff_001") から difficulties テーブル (json_idカラム) を検索 → uuid を返す
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

```


--- 回答のチェック
```php
<?php

namespace App\Services\Utils\Evaluation;

use Illuminate\Support\Facades\App;
use Illuminate\Validation\ValidationException;

/**
 * Class EvaluationCheckService
 *
 * 【概要】
 * - evaluation_spec.checker_method で指定されたメソッド名を呼び出し、CODE の正誤判定を行うためのサービス
 * - question_type=FILL_IN_THE_BLANK をはじめ、将来的に様々なチェックロジックを増やす想定
 *
 * 【今回の修正】
 *  - checkByExactMatch() を追加: ユーザー回答と collect_answer が完全一致していれば正解とみなし、
 *    evaluation_spec.response_format と同じ構造でレスポンスを作る
 */
class EvaluationCheckService
{
    /**
     * 穴埋め問題(FILL_IN_THE_BLANK) の「完全一致」チェック
     *
     * @param array $userAnswerData   ユーザー回答( { fields: [...] } 形式 )
     * @param array $evaluationResponseFormat   userQuestion.evaluation_response_format (JSONをdecodeした配列)
     * @return array レスポンス( evaluation_spec.response_format と同じ構造 )
     *
     * 【仕様】
     * - evaluation_response_format.fields[*].collect_answer と userAnswerData.fields[*].user_answer を照合
     * - 一致したら is_correct=true、そうでなければfalse
     * - 全フィールドが正解なら response_format.is_correct=true,そうでなければfalse
     * - score = 100 * (正解数 / fields総数)
     * - question_text, explanation, question は、App::getLocale() をキーとして evaluation_response_format の該当言語を取得
     * - field_explanation も同様に言語ごと
     */
    public function checkByExactMatch(array $userAnswerData, array $evaluationResponseFormat): array
    {
        // TODO: 多言語化（現在はサンプルで固定メッセージを入れています）

        // 1) fieldsリスト
        $respFields = $evaluationResponseFormat['fields'] ?? [];
        if (!is_array($respFields)) {
            // evaluation_response_format が不正
            throw ValidationException::withMessages([
                'response_format.fields' => "evaluation_response_format の fields が配列ではありません。"
            ]);
        }

        // ユーザー回答 fields
        $answerFields = $userAnswerData['fields'] ?? [];
        if (!is_array($answerFields)) {
            throw ValidationException::withMessages([
                'answer_data.fields' => "ユーザー回答の fields が配列ではありません。"
            ]);
        }

        // field_id => userAnswer
        $userFieldMap = [];
        foreach ($answerFields as $af) {
            $fid = $af['field_id'] ?? null;
            if ($fid) {
                $userFieldMap[$fid] = $af['user_answer'] ?? null;
            }
        }

        $correctCount = 0;
        $totalCount = count($respFields);

        // 2) fields をループし、is_correctを判定
        $newFields = [];
        foreach ($respFields as $f) {
            $fid = $f['field_id'] ?? null;
            if (!$fid) {
                // 不正
                continue;
            }
            // collect_answer
            $collect = $f['collect_answer'] ?? null;
            // user_answer
            $userAns = $userFieldMap[$fid] ?? null;

            // 同じなら正解
            $isCorrect = ($userAns == $collect);  // 厳密型ではなく "==" でよいか要件次第
            // field_explanation
            $fieldExp = $f['field_explanation'] ?? [];
            if (!is_array($fieldExp)) {
                $fieldExp = [];
            }

            // 現在ロケール
            $locale = App::getLocale();
            // フィールドごとの explanation も "ja"=>"...","en"=>"..." が入ってる前提
            // もしUI表示で全言語返すなら「field_explanationそのまま返却」でもOK
            // ここでは fieldsごとに "field_explanation.{locale}" のみ返す例
            $explanationForLocale = $fieldExp[$locale] ?? '';

            if ($isCorrect) {
                $correctCount++;
            }

            // 新しい fields配列を作成
            $newFields[] = [
                'field_id'          => $fid,
                'user_answer'       => $userAns,
                'is_correct'        => $isCorrect ? true : false, // "boolean"文字列ではだめ
                'collect_answer'    => $collect,  // collect_answerはそのまま(ユーザーには隠す場合は外す)
                'field_explanation' => $explanationForLocale
            ];
        }

        // 全体 is_correct
        $allCorrect = ($correctCount === $totalCount);
        $scoreEach = 100 / max($totalCount, 1);
        $scoreVal = $scoreEach * $correctCount;

        // question_text / explanation / question は LLM形式なら "text"固定だが
        // CODE形式なら userQuestion.metadata.question_text などを流用、と要件次第
        // 今回は "localeごとにevaluation_response_format" と書いているので
        // -> "ja":"text","en":"text" があったはず
        // -> そのままロケールに合わせて文字列を返す(あるいは "text"固定）
        // 例：evaluationResponseFormat["question_text"]["ja"] = "text"
        // => "text" → ここでは UI表示向けに question_text=... という文字列にするかどうかは要件次第

        $locale = App::getLocale();
        $qText = $evaluationResponseFormat['question_text'][$locale] ?? '';
        $expText = $evaluationResponseFormat['explanation'][$locale] ?? '';
        $qBody = $evaluationResponseFormat['question'][$locale] ?? '';

        // 最終的に evaluation_spec.response_format と同じ構造で返す
        //   is_correct, score, question_text, explanation, question, fields[*]
        //   ただし fields[*].is_correct は "boolean"(文字列) になっているので `'true'/'false'`
        return [
            'is_correct'   => $allCorrect ? true : false,
            'score'        => (string) $scoreVal,  // "number"文字列
            'question_text'=> $qText,
            'explanation'  => $expText,
            'question'     => $qBody,
            'fields'       => $newFields,
        ];
    }
}


```
