以下の　QuestionSetJsonService.php は、
問題セットJSONをバリデーションするクラスです。

QuestionSetJsonService、以下の条件に合致しているか確認して、
不足があれば修正してください


※注意点

・エラーメッセージは日本語で分かりやすく出力して

・バリデーションルールに違反したら処理を止めるのではなく、その問題はスキップという事にしてほしい

・後で分かりやすいようにソースコードのコメントには必ず実装の仕様をまとめて残しておくこと

・アプリケーション全体で使うので言語定数は定数として扱うこと


ーー言語定数
ja, en

ーーJSON仕様
json_id：必須、文字列
order：必須、数値
unit_id：必須、文字列、units テーブルのjson_idに一致する値があること
unit：必須、オブジェクト（言語定数全て含んでいるか）。
title：必須、オブジェクト（言語定数全て含んでいるか）。
description：必須、オブジェクト（言語定数全て含んでいるか）。
background：必須、オブジェクト（言語定数全て含んでいるか）。
memo：必須、文字列
version： 必須、文字列、文字列が x.x.x のようなバージョニング形式になっているか
status：必須、QuestionSetStatus に値が存在しているか
questions：必須、配列。questions テーブルの json_id に値が存在しているか。

llm_generation_status: 必須、QuestionSetLLMGenerationStatus に値が存在しているか

generate_question_prompt：llm_generation_statusが、ENABLED の時に必須、オブジェクト（言語定数全て含んでいるか）。
generate_question_prompt_file_name：llm_generation_statusが、ENABLED の時に必須、数字。/resources/prompt/generate/question/{generate_question_prompt_file_name}.txt が存在しているか確認。無ければエラー。

--- 問題セットJSONの全体構造
```json
{
  "json_id": "qset_s1_g3_sec100_u300_v100_800",
  "order": 800,
  "unit_id": "unit_s1_g3_sec100_300",
  "unit": {
    "ja": "3位数や4位数の加法及び減法",
    "en": "Addition and subtraction of three- and four-digit numbers"
  },
  "title": {
    "ja": "",
    "en": ""
  },
  "description": {
    "ja": "",
    "en": ""
  },
  "background": {
    "ja": "",
    "en": ""
  },
  "generate_question_prompt": {
    "ja": "",
    "en": ""
  },
  "generate_question_prompt_file_name": 1,
  "llm_generation_status": "ENABLED",

  "memo": "4桁-3桁の引き算",
  "version": "1.0.0",
  "status": "PUBLISHED",
  "questions": [
    "ques_s1_g3_sec100_u300_diff100_qt51_v100_1500",
    "ques_s1_g3_sec100_u300_diff100_qt51_v100_1600"
  ]
}


```



# 実装方針
生成また修正変更するコードはそのコードだけは必ず省略せずに必ず全てアウトプットすること。

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

--- DB構造
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

```
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
        Schema::create('question_set_translations', function (Blueprint $table) {
            $table->id();
            $table->uuid('question_set_id');
            $table->string('locale', 10);
            $table->string('title')->nullable();
            $table->text('description')->nullable();
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('question_set_id')->references('id')->on('question_sets')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('question_set_translations');
    }
};

```

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
        Schema::create('units', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('subject_id');
            $table->uuid('level_id');
            $table->uuid('grade_id');
            $table->uuid('section_id');
            $table->string('json_id')->nullable()->unique();
            $table->longText('requirement')->nullable();
            $table->longText('required_competency')->nullable();
            $table->longText('background')->nullable();
            $table->string('version')->default('0.0.1');
            $table->integer('status')->default(1);
            $table->integer('order');
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('subject_id')->references('id')->on('subjects')->onDelete('cascade');
            $table->foreign('level_id')->references('id')->on('levels')->onDelete('cascade');
            $table->foreign('grade_id')->references('id')->on('grades')->onDelete('cascade');
            $table->foreign('section_id')->references('id')->on('sections')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('units');
    }
};


```

--- QuestionSetStatus(問題セット用のステータス)
```php
<?php

namespace App\Enums;

enum QuestionSetStatus: int
{
    case DRAFT = 1;         // 下書き
    case PUBLISHED = 300;   // 公開中
    case HIDDEN = 600;      // 非公開
    case TEST_PUBLISHED = 900; // テスト公開

    /**
     * ラベルを取得
     */
    public function label(): string
    {
        return match($this) {
            self::DRAFT          => '下書き',
            self::PUBLISHED      => '公開中',
            self::HIDDEN         => '非公開',
            self::TEST_PUBLISHED => 'テスト公開',
        };
    }

    /**
     * 文字列から QuestionSetStatus を取得
     * @param string $statusString 例: "DRAFT", "PUBLISHED" など
     * @return self
     */
    public static function fromString(string $statusString): self
    {
        return match (strtoupper($statusString)) {
            'DRAFT'          => self::DRAFT,
            'PUBLISHED'      => self::PUBLISHED,
            'HIDDEN'         => self::HIDDEN,
            'TEST_PUBLISHED' => self::TEST_PUBLISHED,
            default => throw new \InvalidArgumentException("Unknown status string: {$statusString}")
        };
    }
}

```
--- QuestionSetLLMGenerationStatus Enum
```php
<?php

namespace App\Enums;

/**
 * QuestionSetにおける LLM生成ステータスを表すEnum
 */
enum QuestionSetLLMGenerationStatus: int
{
    /**
     * LLMによる問題生成を行わない
     */
    case DISABLED = 0;

    /**
     * LLMによる問題生成を行う
     */
    case ENABLED  = 1;

    public function label(): string
    {
        return match($this) {
            self::DISABLED => 'LLM生成なし',
            self::ENABLED  => 'LLM生成あり',
        };
    }

    /**
     * 文字列から QuestionSetLLMGenerationStatus を取得
     *
     * @throws \InvalidArgumentException
     */
    public static function fromString(string $raw): self
    {
        return match (strtoupper($raw)) {
            'DISABLED' => self::DISABLED,
            'ENABLED'  => self::ENABLED,
            default => throw new \InvalidArgumentException("Unknown QuestionSetLLMGenerationStatus: {$raw}")
        };
    }
}

```

--- EvaluationCheckerMethod Enum
```php
<?php

namespace App\Enums;

// 問題の正誤を評価するメソッドをマップしたEnum
enum EvaluationCheckerMethod: int
{
    case CALCULATE_METHOD = 1;

    public function label(): string
    {
        return match($this) {
            self::CALCULATE_METHOD          => 'CALCULATE_METHOD',
        };
    }

    /**
     * 文字列から EvaluationCheckerMethod を取得
     * @param string $statusString 例: "CALCULATE_METHOD" など
     * @return self
     */
    public static function fromString(string $statusString): self
    {
        return match (strtoupper($statusString)) {
            'CALCULATE_METHOD'          => self::CALCULATE_METHOD,
            default => throw new \InvalidArgumentException("Unknown status string: {$statusString}")
        };
    }
}

```


--- QuestionType Enum
```php
<?php

namespace App\Enums;

// 問題のカテゴリ（出題形式とは別）
enum QuestionType: int
{
    case CALCULATION             = 1;
    case FILL_IN_THE_BLANK       = 51;
    case SCENARIO                = 101;
    case MULTIPLE_CHOICE         = 151;

//    case SIMPLE_ARITHMETIC        = 10;
//    case COMBINATION             = 101;
//    case HELL                    = 151;
//    case STORY                   = 201;
//    case FREE_TEXT               = 251;
//    case DRILL                   = 301;
//    case SIMULATION              = 401;
//    case DEBUG                   = 451;
//    case PROOF                   = 501;
//    case REASONING               = 551;
//    case MULTI_STEP_COMPARATIVE  = 601;
//    case FUSION                  = 651;
//    case INFERENCE               = 701;
//    case DATA_INTERPRETATION     = 751;
//    case LOGIC                   = 801;

    public function label(): string
    {
        return match($this) {
            self::CALCULATION            => 'Calculation Problem',              // 計算問題: 数値計算や演算ルールの処理を中心
            self::FILL_IN_THE_BLANK      => 'Fill-in-the-Blank Problem',        // 穴埋め問題: 問題文や式に空所があり、そこを埋める形式
            self::SCENARIO               => 'Scenario Problem',                 // シナリオ問題: 状況や経過を踏まえて解決する（回答が一意じゃない）
            self::MULTIPLE_CHOICE        => 'Multiple Choice Problem',          // 選択問題: 複数の選択肢から答えを選ぶ

//            self::SIMPLE_ARITHMETIC      => 'Simple Arithmetic Problem',        // 単純計算問題: 簡単な計算式や数値操作を問う
//            self::COMBINATION            => 'Combination Problem',              // 組み合わせ問題: 対応するペアやグループをマッチさせる
//            self::HELL                   => 'Hell Problem',                     // 地獄問題: 選択肢が多く1つだけ正解がある特許取得済の問題
//            self::STORY                  => 'Story Problem',                    // ストーリー問題: 物語やシナリオで流れを理解しながら解答
//            self::FREE_TEXT              => 'Free Text Problem',                // フリーテキスト問題: 自由記述形式での回答
//            self::DRILL                  => 'Drill Problem',                    // 発生練習問題: 何かを生成・作成したりする練習を含む
//            self::SIMULATION             => 'Simulation Problem (Real-life Modeling)', // シミュレーション問題: 実生活モデルを扱う
//            self::DEBUG                  => 'Debug Problem',                    // デバッグ問題: ミスを発見し修正する
//            self::PROOF                  => 'Proof Problem',                    // 証明問題: 定理や結論がなぜ成り立つかを示す
//            self::REASONING              => 'Reasoning Problem',                // 理由づけ問題: 結果の根拠や理由を説明
//            self::MULTI_STEP_COMPARATIVE => 'Multi-Step Comparative Problem',   // 複数手順を比較検討する問題
//            self::FUSION                 => 'Fusion Problem',                   // 融合問題: 複数領域を組み合わせて解決
//            self::INFERENCE              => 'Inference Problem',                // 推理問題: 与えられた情報から論理的に結論を導く
//            self::DATA_INTERPRETATION    => 'Data Interpretation Problem',      // 統計資料の読み取り問題: グラフや表を使う
//            self::LOGIC                  => 'Logic Problem',                    // ロジック問題: プログラム的思考やアルゴリズム判断
        };
    }

    /**
     * 文字列を受け取り、該当する QuestionType を返す静的メソッド
     * 存在しない場合は例外を投げるサンプルです
     *
     * @param string $typeString "FILL_IN_THE_BLANK" など
     * @return QuestionType
     */
    public static function fromString(string $typeString): QuestionType
    {
        return match($typeString) {
            'CALCULATION'       => self::CALCULATION,
            'FILL_IN_THE_BLANK' => self::FILL_IN_THE_BLANK,
            'SCENARIO'          => self::SCENARIO,
            'MULTIPLE_CHOICE'   => self::MULTIPLE_CHOICE,
            default => throw new \InvalidArgumentException("Unknown QuestionType string: {$typeString}")
        };
    }
}
```
--- QuestionStatus enum
```php
<?php

namespace App\Enums;

enum QuestionStatus: int
{
    case DRAFT = 1;         // 下書き
    case PUBLISHED = 300;   // 公開中
    case HIDDEN = 600;      // 非公開
    case TEST_PUBLISHED = 900; // テスト公開

    public function label(): string
    {
        return match($this) {
            self::DRAFT          => '下書き',
            self::PUBLISHED      => '公開中',
            self::HIDDEN         => '非公開',
            self::TEST_PUBLISHED => 'テスト公開',
        };
    }

    /**
     * 文字列から QuestionStatus を取得
     * @param string $statusString 例: "DRAFT", "PUBLISHED" など
     * @return self
     */
    public static function fromString(string $statusString): self
    {
        return match (strtoupper($statusString)) {
            'DRAFT'          => self::DRAFT,
            'PUBLISHED'      => self::PUBLISHED,
            'HIDDEN'         => self::HIDDEN,
            'TEST_PUBLISHED' => self::TEST_PUBLISHED,
            default => throw new \InvalidArgumentException("Unknown status string: {$statusString}")
        };
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
        Schema::create('skills', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->string('json_id')->nullable()->unique();
            $table->string('version')->default('0.0.1');
            $table->string('display_name')->nullable();
            $table->integer('order');
            $table->timestamps();
            $table->softDeletes();
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

```
--- EvaluationMethod Enum
```php
<?php

namespace App\Enums;

// 回答の正誤を評価する方法をマップしたEnum
enum EvaluationMethod: int
{
    case CODE = 1;
    case LLM = 2;

    /**
     * 文字列から EvaluationMethod を取得
     * @param string $statusString 例: "LLM", "CODE" など
     * @return self
     */
    public static function fromString(string $statusString): self
    {
        return match (strtoupper($statusString)) {
            'LLM'          => self::LLM,
            'CODE'          => self::CODE,
            default => throw new \InvalidArgumentException("Unknown status string: {$statusString}")
        };
    }


}

```

＝＝＝問題取り込みコード
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
            $question->grade_id = $gradeUuid;
            $question->difficulty_id   = $difficultyUuid;
            $question->version         = $version;
            $question->status          = $statusValue;


            // question->metadata = JSONで受け取った metadata
            $question->metadata = isset($questionJson['metadata'])
                ? json_encode($questionJson['metadata'], JSON_UNESCAPED_UNICODE)
                : null;

            if (!empty($question->metadata)) {
                // question_type,  を enum に変換
                $rawQuestionType = $questionJson['metadata']['question_type'] ?? null;

                $questionTypeValue   = $this->parseQuestionType($rawQuestionType);

                $question->question_type   = $questionTypeValue;

            }

            // 評価
            if (isset($questionJson['evaluation_spec'])) {
                $question->evaluation_method = isset($questionJson['evaluation_spec']['evaluation_method'])
                    ? EvaluationMethod::fromString($questionJson['evaluation_spec']['evaluation_method'])
                    : null;

                $question->checker_method = isset($questionJson['evaluation_spec']['checkerMethod'])
                    ? EvaluationCheckerMethod::fromString($questionJson['evaluation_spec']['checkerMethod'])
                    : null;

                $question->llm_evaluation_prompt_number = $questionJson['evaluation_spec']['llm_prompt_number'] ?? null;

                $question->llm_evaluation_response_format = isset($questionJson['evaluation_spec']['response_format'])
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
        } catch (\Throwable $e) {
            DB::rollBack();
            $this->error("Error upserting question json_id={$jsonId}: " . $e->getMessage());
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
        $question->learning_background = implode("\n", $backgrounds);
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

--- QuestionSetJsonService
```php
<?php

namespace App\Services\Utils\Question;

use App\Enums\QuestionSetStatus;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Validator;

/**
 * Class QuestionSetJsonService
 *
 * 【概要】
 *   このサービスクラスは、問題セットJSON（QuestionSetJSON）のバリデーションを担当する。
 *   後で分かりやすいように、日本語のエラーメッセージをまとめて返す実装を行い、
 *   バリデーションエラーがあった場合でも「例外を投げず」、呼び出し側でスキップ処理を行えるようにしている。
 *
 * 【JSON仕様】
 *   {
 *     "json_id": "qset_...",
 *     "order": 800,
 *     "unit_id": "unit_s1_g3_sec100_300",
 *     "unit": {
 *       "ja": "...",
 *       "en": "..."
 *     },
 *     "title": {
 *       "ja": "...",
 *       "en": "..."
 *     },
 *     "description": {
 *       "ja": "...",
 *       "en": "..."
 *     },
 *     "background": {
 *       "ja": "...",
 *       "en": "..."
 *     },
 *     "memo": "4桁-3桁の引き算",
 *     "version": "1.0.0",
 *     "status": "PUBLISHED",
 *     "questions": [
 *       "ques_s1_g3_sec100_u300_diff100_qt51_v100_1500",
 *       "ques_s1_g3_sec100_u300_diff100_qt51_v100_1600"
 *     ],
 *     "generate_question_prompt": {
 *       "ja": "",
 *       "en": ""
 *     }
 *   }
 *
 * 【バリデーション項目】
 *   - json_id:     必須、文字列
 *   - order:       必須、数値
 *   - unit_id:     必須、文字列 → units テーブルの json_id が存在するか確認
 *   - unit:        必須、{ja:"...",en:"..."} のオブジェクト
 *   - title:       必須、{ja:"...",en:"..."} のオブジェクト
 *   - description: 必須、{ja:"...",en:"..."} のオブジェクト
 *   - background: 必須、{ja:"...",en:"..."} のオブジェクト
 *   - memo:        必須、文字列
 *   - version:     必須、x.x.x形式（正規表現）
 *   - status:      必須、QuestionSetStatus enum に存在する文字列
 *   - questions:   必須、配列。中身は questionsテーブルの json_id が存在するかチェック
 *   - generate_question_prompt: 必須、{ja:"...",en:"..."} のオブジェクト
 *
 * 【使用例】
 *   $service = new QuestionSetJsonService();
 *   $errors = $service->validateQuestionSetJson($jsonData);
 *   if (!empty($errors)) {
 *       // バリデーションエラー → 処理をスキップしたりログに出力
 *   } else {
 *       // バリデーション成功 → DBへの保存処理へ
 *   }
 *
 * 【注意】
 *   バリデーションエラーが発生しても例外は投げず、エラー配列を返す。
 *   スキップするかどうかは呼び出し側の実装に任せる。
 */
class QuestionSetJsonService
{
    /**
     * 言語定数 (アプリケーション全体で共通利用)
     */
    public const LANGUAGES = ['ja', 'en'];

    /**
     * 問題セットJSONをバリデーションする
     *
     * @param array $jsonData
     * @return array
     *   バリデーションエラーが無ければ空配列を返し、エラーがあれば
     *   ['フィールド名' => ['エラーメッセージ', ...], ...]
     *   の形式の配列を返す
     */
    public function validateQuestionSetJson(array $jsonData): array
    {
        // 1) まず Validator で基本ルールをチェック
        $validator = Validator::make($jsonData, $this->rules(), $this->messages());

        // 2) afterコールバック: DBチェックや追加検証
        $validator->after(function ($v) use ($jsonData) {
            $this->checkUnitIdExists($v, $jsonData);
            $this->checkQuestionSetStatus($v, $jsonData);
            $this->checkQuestionsExist($v, $jsonData);
        });

        // 3) バリデーション実行
        $validator->fails(); // Laravel上ではこのタイミングで実際にチェックを走らせる

        // 4) エラーを配列化して返す
        return $validator->errors()->toArray();
    }

    /**
     * バリデーションルール定義
     *
     * @return array
     */
    private function rules(): array
    {
        return [
            // 基本必須項目
            'json_id'    => ['required','string'],
            'order'      => ['required','integer'],
            'unit_id'    => ['required','string'],
            'memo'       => ['required','string'],
            'version'    => ['required','regex:/^\d+\.\d+\.\d+$/'],
            'status'     => ['required','string'],
            // unit/title/description/generate_question_prompt は多言語オブジェクト
            'unit'       => ['required','array'],
            'unit.ja'    => ['required','string'],
            'unit.en'    => ['required','string'],
            'title'      => ['required','array'],
            'title.ja'   => ['required','string'],
            'title.en'   => ['required','string'],
            'description'      => ['required','array'],
            'description.ja'   => ['required','string'],
            'description.en'   => ['required','string'],
            'background'      => ['required','array'],
            'background.ja'   => ['required','string'],
            'background.en'   => ['required','string'],

            'generate_question_prompt'      => ['required','array'],
            'generate_question_prompt.ja'   => ['required','string'],
            'generate_question_prompt.en'   => ['required','string'],

            // questions => 必須配列
            'questions'      => ['required','array'],
        ];
    }

    /**
     * エラーメッセージ定義（日本語）
     *
     * @return array
     */
    private function messages(): array
    {
        return [
            'required'    => ':attribute は必須です。',
            'integer'     => ':attribute には数値を指定してください。',
            'string'      => ':attribute は文字列を指定してください。',
            'regex'       => ':attribute の形式が不正です。(例: 1.0.0)',
            'array'       => ':attribute は配列である必要があります。',
        ];
    }

    /**
     * unit_id が units テーブルに存在するかをチェック
     */
    private function checkUnitIdExists($validator, array $jsonData): void
    {
        $unitId = $jsonData['unit_id'] ?? null;
        if ($unitId) {
            // units テーブルに json_id=$unitId の行があるか
            $exists = DB::table('units')
                ->where('json_id', $unitId)
                ->exists();
            if (!$exists) {
                $validator->errors()->add('unit_id', "指定された unit_id='{$unitId}' は unitsテーブルに存在しません。");
            }
        }
    }

    /**
     * status が QuestionSetStatus で定義されているかチェック
     */
    private function checkQuestionSetStatus($validator, array $jsonData): void
    {
        $rawStatus = $jsonData['status'] ?? '';
        if (!$rawStatus) {
            return; // 既に required チェック済み
        }

        // QuestionSetStatus::fromString() で変換可能かどうか
        try {
            \App\Enums\QuestionSetStatus::fromString($rawStatus);
        } catch (\InvalidArgumentException) {
            $validator->errors()->add('status', "不明なステータスです: {$rawStatus}");
        }
    }

    /**
     * questions[] の各要素が、questionsテーブルの json_id に存在するかチェック
     */
    private function checkQuestionsExist($validator, array $jsonData): void
    {
        $list = $jsonData['questions'] ?? [];
        if (!is_array($list)) {
            return;
        }

        foreach ($list as $idx => $qJsonId) {
            if (!is_string($qJsonId) || !$qJsonId) {
                $validator->errors()->add("questions.{$idx}", "questionsの要素は文字列のjson_idが必要です。");
                continue;
            }

            $exists = DB::table('questions')
                ->where('json_id', $qJsonId)
                ->exists();
            if (!$exists) {
                $validator->errors()->add("questions.{$idx}", "問題 json_id='{$qJsonId}' はDBに存在しません。");
            }
        }
    }
}


```
