以下、QuestionJsonManageService　の　validateQuestionJson　が以下の条件を満たすように修正してください。
修正箇所を知らせてください。
 

※注意点
・現在は question_type FILL_IN_THE_BLANK の仕様だけですが、将来的に複数の種類が追加されて行く予定です。１００個以上になるかも
バリデーションのルールは、question_typeによっては重複するものも出てくるので、
[FILL_IN_THE_BLANK, SCENARIO]など、バリデーションルールに対応するquestion Type を簡単追加したり取り消したり出来るような実装がよい

・エラーメッセージは日本語で分かりやすく出力して

・バリデーションルールに違反したら処理を止めるのではなく、その問題はスキップという事にしてほしい

・後で分かりやすいようにソースコードのコメントには必ず実装の仕様をまとめて残しておくこと

・アプリケーション全体で使うので言語定数は定数として扱うこと

ーー言語定数
ja, en

--input_format type 定数
fixed: 固定。ユーザーは項目の増減はコントロール出来ず、input_format.fields の内容に固定
custom: input_format.fields の内容はあくまでもデフォルト表記で、固定されず、ユーザー自由に回答の数を増減出来る。

-- input_format.fields.type、evaluation_spec.response_format.fields.user_answer,metadata.input_format.fields.collect_answer の定数
number: 数値型

-- input_format.question_components.type定数
text: テキスト
image: 画像
movie: 動画
blank: 空欄。入力項目

ーーJSON仕様
order：必須、数値
id：必須、文字列
level_id：必須、文字列、Levels テーブルのjson_idに一致する値があること
grade_id：必須、文字列、Grade テーブルのjson_idに一致する値があること
difficulty_id：必須、文字列、Difficulties テーブルのjson_idに一致する値があること
version： 必須、文字列、文字列が x.x.x のようなバージョニング形式になっているか
status：必須、QuestionStatus に値が存在しているか
generated_by_llm：必須、Boolean
created_at：必須、Y-m-d H:i:s 形式になっていること
updated_at：必須、Y-m-d H:i:s 形式になっていること

question_text: 必須、オブジェクト（言語定数全て含んでいるか）
explanation: 必須、オブジェクト（言語定数全て含んでいるか）
background: 必須、オブジェクト（言語定数全て含んでいるか）

skills: 必須、配列
skills.skill_id: 必須、Skills テーブルのjson_idに一致する値があること
skills.name: 必須、文字列、skill_id と一致するSkills テーブルのjson_idを持つデータの　display_name と値が一致すること

learning_requirements:必須、オブジェクト
learning_requirements.learning_subject: 必須、文字列
learning_requirements.learning_no: 必須、数値型
learning_requirements.learning_requirement: 必須、文字列
learning_requirements.learning_required_competency: 必須、文字列
learning_requirements.learning_background: 必須、文字列
learning_requirements.learning_category: 必須、文字列
learning_requirements.learning_grade_level: 必須、文字列
learning_requirements.learning_url: 必須ではない、文字列、URL

evaluation_spec：必須、オブジェクト
evaluation_spec.evaluation_method: 必須、文字列、EvaluationMethod　に値が存在していること
evaluation_spec.llm_prompt_number: evaluation_methodが”LLM”の時は必須、数値。LLMに投げるプロンプトを管理するファイルと対応している。resources/prompts/evaluation/{x}.txt の {x}の箇所と対応しているので、このファイルが存在しているかバリデーションチェックする。
evaluation_spec.response_format：evaluation_methodが”LLM”の時は必須、オブジェクト。LLMに正誤判定を依頼する時に指定するレスポンスの形
evaluation_spec.response_format.is_correct：evaluation_methodが”LLM”の時は必須、テキスト型("boolean"のみ）。回答全体が正解かどうか表す
evaluation_spec.response_format.score：evaluation_methodが”LLM”の時は必須、テキスト型("number"のみ）。スコアを表す
evaluation_spec.response_format.question_text：evaluation_methodが”LLM”の時は必須、オブジェクト（言語定数全て含んでいるか）, 言語ごとにそれぞれ オブジェクトの中の値は "{$言語設定例えば"ja"など}": "text"となっていること
evaluation_spec.response_format.explanation：evaluation_methodが”LLM”の時は必須、オブジェクト（言語定数全て含んでいるか）, 言語ごとにそれぞれ オブジェクトの中の値は "{$言語設定例えば"ja"など}": "text"となっていること
evaluation_spec.response_format.question：evaluation_methodが”LLM”の時は必須、オブジェクト（言語定数全て含んでいるか）、言語ごとにそれぞれ オブジェクトの中の値は "{$言語設定例えば"ja"など}": "text"となっていること

evaluation_spec.response_format.fields：evaluation_methodが”LLM”の時は必須、配列。LLMにユーザーの回答の形を知らせる為にフォーマットを定義。
evaluation_spec.response_format.fields.field_id：evaluation_methodが”LLM”の時は必須、配列。ユーザーの回答の型。
evaluation_spec.response_format.fields.user_answer：evaluation_methodが”LLM”の時は必須、input_format.fields.type、evaluation_spec.response_format.fields.user_answer,evaluation_spec.input_format.fields.collect_answer の定数に値があるか。ユーザーの回答。
evaluation_spec.response_format.fields.is_correct：evaluation_methodが”LLM”の時は必須、テキスト型("boolean"のみ）。ユーザーの回答が正しいか
evaluation_spec.response_format.fields.collect_answer：evaluation_methodが”LLM”の時は必須、evaluation_spec.response_format.fields.user_answerで指定されている型と一致しているか。例えば、"number" の場合は、32 などの数値となっているか。問題の正解（ユーザーには隠す）
evaluation_spec.response_format.fields.field_explanation: evaluation_methodが”LLM”の時は必須、オブジェクト、（言語定数全て含んでいるか）

metadata.question_type：必須、App\Enums\QuestionType.php に値が存在しているか。JSONには、文字列でも数値でもどちらでも入力可にする

metadata.question ：必須、オブジェクト（言語定数全て含んでいるか）。問題文

metadata.input_format: 必須、オブジェクト（言語定数全て含んでいるか）
metadata.input_format.type: 必須、input_format type 定数と値が一致しているか。
metadata.input_format.fields: 必須、配列。ユーザが回答する入力フォームの仕様を定義
metadata.input_format.fields.field_id: 必須、f_x のフォーマットになっているか。同じ fields 内に重複した値が存在しないか。 question_components内の type: "blank"の数と総数が合っているか。
metadata.input_format.fields.attribute: 必須、（input_format.fields.type、evaluation_spec.response_format.fields.user_answer,evaluation_spec.response_format.fields.collect_answer の定数に値が存在しているか）。ブランクのフォームの属性。例えば number であれば、<input type="number">になる
metadata.input_format.fields.collect_answer: 必須、evaluation_spec.response_format.fields.collect_answer とは違い、次の条件を満たした文字列型かどうかをチェックする。input_format.fields.type、evaluation_spec.response_format.fields.user_answer,evaluation_spec.response_format.fields.collect_answer の定数に値があるか。正答。レスポンスされる問題の正解のデータの型

metadata.input_format.question_components: 必須（問題を更生する要素）
metadata.input_format.question_components.attribute：必須、input_format.question_components.type定数と値が合っているか
metadata.input_format.question_components.content：必須、オブジェクト、言語定数と一致する値が全て含まれているか
metadata.input_format.question_components.order: 必須、数値、重複する値が存在しないこと。問題を構築するときの表示順番


--- 問題JSONの全体構造
```json
{
  "order": 100,
  "id": "ques_s1_g2_sec100_u300_diff100_qt51_v100_100",
  "level_id": "lev_002",
  "grade_id": "gra_002",
  "difficulty_id": "diff_100",
  "version": "1.0.0",
  "status": "DRAFT",
  "generated_by_llm": true,
  "created_at": "2025-01-01 00:00:00",
  "updated_at": "2025-01-01 00:00:00",
  "question_text": {
    "ja": "▢にあてはまる数を答えなさい。",
    "en": "Please answer the numbers that fit in the blanks."
  },
  "explanation": {
    "ja": "たし算には「順番を変えても答えが同じになる」という法則があります。例えば 3 + 5 と 5 + 3 の結果は、どちらも 8 になります。この問題で、数字の入れ替えを実際にやってみましょう。",
    "en": "Addition has the property that changing the order of the numbers does not affect the result. For example, both 3 + 5 and 5 + 3 give 8. In this problem, you will experience this by rearranging the numbers."
  },
  "background": {
    "ja": "この問題では、加法の交換法則（a + b = b + a）の具体的な事例を通じて、数字の並べかえを行っても同じ答えが得られることを体感してもらうのがねらいです。",
    "en": "This exercise aims to help learners understand the commutative property of addition (a + b = b + a) through a concrete example, showing that swapping the order of the numbers yields the same result."
  },
  "skills": [
    {
      "skill_id": "sk_004",
      "name": "知識・技能"
    }
  ],
  "learning_requirements": [
    {
      "learning_subject": "算数",
      "learning_no": 13,
      "learning_requirement": "A 数と計算 計算の意味・方法 加法や減法に関して成り立つ性質",
      "learning_required_competency": "加法および減法について、交換法則(a + b = b + a)や結合法則((a + b) + c = a + (b + c))が成り立つことを具体的に理解できる",
      "learning_background": "具体例や操作活動で、順序・まとまりを変えても結果が変わらないことを確認し、高学年の式操作へつなぐ素地を作る",
      "learning_category": "A",
      "learning_grade_level": "小2",
      "learning_url": "https://docs.google.com/spreadsheets/d/1W5vaFHcyU_BrMwb1JLZ-DpyFmXXZPaYMTHIITcwLqqY/edit?gid=0#gid=0&range=13:13"
    }
  ],
  "evaluation_spec": {
    "evaluation_method": "LLM",
    "llm_prompt_number": 1,
    "response_format": {
      "is_correct": "boolean",
      "score": "number",
      "question_text": {
        "ja": "text",
        "en": "text"
      },
      "explanation": {
        "ja": "text",
        "en": "text"
      },
      "question": {
        "ja": "text",
        "en": "text"
      },
      "fields": [
        {
          "field_id": "f_1",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": 3,
          "field_explanation": {
            "ja": "text",
            "en": "text"
          }
        },
        {
          "field_id": "f_2",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": 8,
          "field_explanation": {
            "ja": "text",
            "en": "text"
          }
        }
      ]
    }
  },
  "metadata": {
    "question_type": "FILL_IN_THE_BLANK",
    "question": {
      "ja": "3 + 5 = 5 + ▢ = ▢",
      "en": "3 + 5 = 5 + ▢ = ▢"
    },
    "input_format": {
      "type": "fixed",
      "fields": [
        {
          "field_id": "f_1",
          "attribute": "number",
          "collect_answer": "number"
        },
        {
          "field_id": "f_2",
          "attribute": "number",
          "collect_answer": "number"
        }
      ],
      "question_components": [
        {
          "type": "text",
          "content": {
            "ja": "3 + 5 = 5 + ",
            "en": "3 + 5 = 5 + "
          },
          "order": 1
        },
        {
          "type": "blank",
          "field_id": "f_1",
          "order": 2
        },
        {
          "type": "text",
          "content": {
            "ja": " = ",
            "en": " = "
          },
          "order": 3
        },
        {
          "type": "blank",
          "field_id": "f_2",
          "order": 4
        }
      ]

    }
  }
}

```



# 実装方針
生成また修正変更するコードはそのコードだけは必ず省略せずに必ず全てアウトプットすること。
エンドポイントは、/study-sessions として実装してください

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
        Schema::create('levels', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('subject_id');
            $table->string('json_id')->nullable()->unique()->comment('リポジトリ上で管理するための一意識別子');
            $table->text('description')->nullable();
            $table->text('memo')->nullable();
            $table->string('version')->default('0.0.1');
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
        Schema::dropIfExists('levels');
    }
};

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

--- QuestionJsonManageService
```php
<?php

namespace App\Services\Utils\Question;

use App\Enums\EvaluationMethod;
use App\Enums\QuestionStatus;
use App\Enums\QuestionType;
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\ValidationException;

/**
 * Class QuestionJsonManageService
 *
 * 【概要】
 * - GitHubリポジトリ等からインポートされる問題JSONをバリデーション＆加工するサービスクラス
 * - question_type が増えた際にルールを簡単に追加・削除できるよう、メソッドで分割しやすい作りを目指す
 * - バリデーション違反の場合は ValidationException をthrowし、呼び出し元がスキップできる
 *
 * 【今回の修正点】
 * 1. evaluation_spec.response_format.is_correct = "boolean" (文字列) のみ  
 * 2. evaluation_spec.response_format.score       = "number"  (文字列) のみ  
 * 3. fields[*].is_correct も "boolean" (文字列) のみ
 * 4. fields[*].user_answer / collect_answer => "number" (文字列) のみ
 * 5. learning_url は必須ではないが、存在する場合はURL形式
 * 6. コードコメントで実装仕様を明示
 *
 * 【使用方針】
 * - `ImportQuestionsFromGithub`等が、このサービスの validateQuestionJson() を呼び出す
 * - バリデーションに失敗 → ValidationException → 呼び出し側が処理をスキップし継続
 */
class QuestionJsonManageService
{
    /**
     * 言語定数(アプリケーション全体で利用)
     */
    public const LANGUAGES = ['ja', 'en'];

    /**
     * input_format.type の定数
     */
    public const INPUT_FORMAT_TYPES = [
        'fixed',
        'custom',
    ];

    /**
     * input_format.fields.type / evaluation_spec.response_format.fields.user_answer / collect_answer の定数
     * → number(数値型)想定
     */
    public const FIELD_TYPE = [
        'number'
    ];

    /**
     * input_format.question_components.type の定数
     */
    public const COMPONENT_TYPES = [
        'text',
        'image',
        'movie',
        'blank',
    ];

    /**
     * 問題JSONをバリデーション
     *
     * @param array $json
     * @return array  バリデーション通過後のデータ（今回はそのまま返却）
     * @throws ValidationException バリデーションエラー時に発生
     *
     * 【仕様まとめ】
     * - トップレベル共通ルールを定義（order,id,level_id...）
     * - after()でDB存在チェックなど追加
     * - evaluation_method=LLM の時だけ追加必須ルール
     * - question_type=FILL_IN_THE_BLANKなどが増えても対応可能な構造
     */
    public function validateQuestionJson(array $json): array
    {
        // 1) トップレベル共通ルール + 日本語メッセージ
        $validator = Validator::make($json, $this->getTopLevelRules(), $this->messages());

        // 2) after() で補足検証 (DB参照, LLM関連ルールなど)
        $validator->after(function($v) use ($json) {
            $this->validateLevelGradeDifficulty($v, $json);
            $this->validateSkills($v, $json);
            $this->validateEvaluationSpec($v, $json);
            $this->validateMetadata($v, $json);
        });

        // 3) 実行 (NGの場合は ValidationException)
        $validated = $validator->validate();

        // 4) 必要があれば加工。今回はそのまま返す
        return $validated;
    }

    /**
     * トップレベル共通のバリデーションルール
     * (細かいロジックは after() コールバックで追加)
     */
    private function getTopLevelRules(): array
    {
        return [
            // 1) 基本フィールド
            'order'         => ['required','integer'],
            'id'            => ['required','string'],
            'level_id'      => ['required','string'],
            'grade_id'      => ['required','string'],
            'difficulty_id' => ['required','string'],
            'version'       => ['required','regex:/^\d+\.\d+\.\d+$/'], // x.x.x の形式
            'status'        => ['required','string'], // 後でQuestionStatus::fromString()する
            'generated_by_llm' => ['required','boolean'],
            'created_at'    => ['required','date_format:Y-m-d H:i:s'],
            'updated_at'    => ['required','date_format:Y-m-d H:i:s'],

            // 2) テキスト系 (ja,en)
            'question_text'       => ['required','array'],
            'question_text.ja'    => ['required','string'],
            'question_text.en'    => ['required','string'],
            'explanation'         => ['required','array'],
            'explanation.ja'      => ['required','string'],
            'explanation.en'      => ['required','string'],
            'background'          => ['required','array'],
            'background.ja'       => ['required','string'],
            'background.en'       => ['required','string'],

            // 3) skills: 必須・配列
            'skills'              => ['required','array'],

            // 4) learning_requirements: 必須(複数要素)
            'learning_requirements'                               => ['required','array'],
            'learning_requirements.*.learning_subject'            => ['required','string'],
            'learning_requirements.*.learning_no'                 => ['required','integer'],
            'learning_requirements.*.learning_requirement'        => ['required','string'],
            'learning_requirements.*.learning_required_competency'=> ['required','string'],
            'learning_requirements.*.learning_background'         => ['required','string'],
            'learning_requirements.*.learning_category'           => ['required','string'],
            'learning_requirements.*.learning_grade_level'        => ['required','string'],
            // 必須ではないがあればURL
            'learning_requirements.*.learning_url'                => ['sometimes','url'],

            // 5) evaluation_spec
            'evaluation_spec'                         => ['required','array'],
            'evaluation_spec.evaluation_method'       => ['required','string'],

            // 6) metadata
            'metadata'                                => ['required','array'],
            'metadata.question_type'                  => ['required'], // 文字列or数値OK
            'metadata.question'                       => ['required','array'],
            'metadata.question.ja'                    => ['required','string'],
            'metadata.question.en'                    => ['required','string'],
            'metadata.input_format'                   => ['required','array'],
            'metadata.input_format.type'              => ['required','string','in:fixed,custom'],
            'metadata.input_format.fields'            => ['required','array'],
            'metadata.input_format.question_components' => ['required','array'],
        ];
    }

    /**
     * 日本語エラーメッセージ
     */
    private function messages(): array
    {
        return [
            'required'    => ':attribute は必須項目です。',
            'integer'     => ':attribute は数値を指定してください。',
            'string'      => ':attribute は文字列である必要があります。',
            'boolean'     => ':attribute は true/false を指定してください。',
            'date_format' => ':attribute は :format 形式で指定してください。',
            'regex'       => ':attribute の形式が不正です。(例: 1.0.0)',
            'array'       => ':attribute は配列である必要があります。',
            'in'          => ':attribute に不正な値が指定されました。( :values )',
            'url'         => ':attribute は有効なURL形式で入力してください。',
        ];
    }

    /**
     * level_id / grade_id / difficulty_id がDBに存在するか
     * 見つからない場合はエラーを追加
     */
    private function validateLevelGradeDifficulty($validator, array $json)
    {
        $levelId = $json['level_id'] ?? null;
        $gradeId = $json['grade_id'] ?? null;
        $diffId  = $json['difficulty_id'] ?? null;

        // levelsテーブル確認
        if ($levelId) {
            $exists = \DB::table('levels')->where('json_id', $levelId)->exists();
            if (!$exists) {
                $validator->errors()->add('level_id', "指定された level_id='{$levelId}' はDBに存在しません。");
            }
        }
        // gradeテーブル確認
        if ($gradeId) {
            $exists = \DB::table('grades')->where('json_id', $gradeId)->exists();
            if (!$exists) {
                $validator->errors()->add('grade_id', "指定された grade_id='{$gradeId}' はDBに存在しません。");
            }
        }
        // difficultiesテーブル確認
        if ($diffId) {
            $exists = \DB::table('difficulties')->where('json_id', $diffId)->exists();
            if (!$exists) {
                $validator->errors()->add('difficulty_id', "指定された difficulty_id='{$diffId}' はDBに存在しません。");
            }
        }
    }

    /**
     * skills[] の検証
     * - skill_id が DBにあるか
     * - name が DBの display_name と一致するか
     */
    private function validateSkills($validator, array $json)
    {
        $skills = $json['skills'] ?? [];
        if (!is_array($skills)) {
            return;
        }

        foreach ($skills as $idx => $sk) {
            $skillId   = $sk['skill_id'] ?? null;
            $skillName = $sk['name']     ?? null;
            if (!$skillId || !$skillName) {
                continue; // 必須だが無ければ次へ
            }

            // DB参照
            $row = \DB::table('skills')->where('json_id', $skillId)->first();
            if (!$row) {
                $validator->errors()->add("skills.{$idx}.skill_id", "skill_id='{$skillId}' はDBに存在しません。");
                continue;
            }
            // name と display_name が一致
            if ($row->display_name !== $skillName) {
                $validator->errors()->add("skills.{$idx}.name", "skill_id='{$skillId}' の display_name と name='{$skillName}' が一致しません。");
            }
        }
    }

    /**
     * evaluation_spec の詳細検証
     * - evaluation_method=LLM の場合のみ追加必須フィールド(llm_prompt_number, response_format)
     */
    private function validateEvaluationSpec($validator, array $json)
    {
        $eval = $json['evaluation_spec'] ?? [];
        $methodRaw = $eval['evaluation_method'] ?? null;

        if (!$methodRaw) {
            return; // すでにトップレベルrequiredでチェックはあるが
        }

        // enumに存在するか
        try {
            $enm = EvaluationMethod::fromString($methodRaw);
        } catch (\InvalidArgumentException) {
            $validator->errors()->add('evaluation_spec.evaluation_method', "evaluation_method='{$methodRaw}' は不正です。(CODE/LLM)");
            return;
        }

        // method=LLM
        if ($enm === EvaluationMethod::LLM) {
            // llm_prompt_number
            if (!isset($eval['llm_prompt_number']) || !is_numeric($eval['llm_prompt_number'])) {
                $validator->errors()->add('evaluation_spec.llm_prompt_number', "evaluation_method=LLM のため llm_prompt_number(数値) が必須です。");
            } else {
                $path = resource_path("prompts/evaluation/{$eval['llm_prompt_number']}.txt");
                if (!file_exists($path)) {
                    $validator->errors()->add('evaluation_spec.llm_prompt_number', "LLM promptファイルが見つかりません。(path={$path})");
                }
            }

            // response_format
            if (!isset($eval['response_format']) || !is_array($eval['response_format'])) {
                $validator->errors()->add('evaluation_spec.response_format', "evaluation_method=LLM のため response_format(オブジェクト) が必須です。");
            } else {
                $this->validateResponseFormat($validator, $json, $eval['response_format']);
            }
        }
    }

    /**
     * evaluation_spec.response_format の詳細チェック
     * - is_correct => "boolean" (文字列)
     * - score     => "number"  (文字列)
     * - question_text/explanation/question => langごとに "text" を期待
     * - fields => user_answer=number, is_correct=boolean, collect_answer=number, field_explanation(langごと)
     */
    private function validateResponseFormat($validator, array $json, array $resp)
    {
        // is_correct => "boolean" 文字列のみ
        if (!isset($resp['is_correct']) || $resp['is_correct'] !== 'boolean') {
            $validator->errors()->add('evaluation_spec.response_format.is_correct', "is_correct は 'boolean' の文字列のみ許可されます。");
        }

        // score => "number" 文字列
        if (!isset($resp['score']) || $resp['score'] !== 'number') {
            $validator->errors()->add('evaluation_spec.response_format.score', "score は 'number' の文字列のみ許可されます。");
        }

        // question_text => langごとに 'text'
        if (empty($resp['question_text']) || !is_array($resp['question_text'])) {
            $validator->errors()->add('evaluation_spec.response_format.question_text', "question_text(オブジェクト) は必須です。");
        } else {
            foreach (self::LANGUAGES as $lang) {
                if (!isset($resp['question_text'][$lang])) {
                    $validator->errors()->add("evaluation_spec.response_format.question_text.{$lang}", "question_text.{$lang} がありません。");
                } else {
                    if ($resp['question_text'][$lang] !== 'text') {
                        $validator->errors()->add("evaluation_spec.response_format.question_text.{$lang}", "question_text.{$lang} は 'text' である必要があります。");
                    }
                }
            }
        }

        // explanation => langごと 'text'
        if (empty($resp['explanation']) || !is_array($resp['explanation'])) {
            $validator->errors()->add('evaluation_spec.response_format.explanation', "explanation(オブジェクト) は必須です。");
        } else {
            foreach (self::LANGUAGES as $lang) {
                if (!isset($resp['explanation'][$lang])) {
                    $validator->errors()->add("evaluation_spec.response_format.explanation.{$lang}", "explanation.{$lang} がありません。");
                } else {
                    if ($resp['explanation'][$lang] !== 'text') {
                        $validator->errors()->add("evaluation_spec.response_format.explanation.{$lang}", "explanation.{$lang} は 'text' である必要があります。");
                    }
                }
            }
        }

        // question => langごと 'text'
        if (empty($resp['question']) || !is_array($resp['question'])) {
            $validator->errors()->add('evaluation_spec.response_format.question', "question(オブジェクト) は必須です。");
        } else {
            foreach (self::LANGUAGES as $lang) {
                if (!isset($resp['question'][$lang])) {
                    $validator->errors()->add("evaluation_spec.response_format.question.{$lang}", "question.{$lang} がありません。");
                } else {
                    if ($resp['question'][$lang] !== 'text') {
                        $validator->errors()->add("evaluation_spec.response_format.question.{$lang}", "question.{$lang} は 'text' である必要があります。");
                    }
                }
            }
        }

        // fields: 必須配列
        if (empty($resp['fields']) || !is_array($resp['fields'])) {
            $validator->errors()->add('evaluation_spec.response_format.fields', "fields(配列) が必須です。");
        } else {
            foreach ($resp['fields'] as $idx => $field) {
                $this->validateResponseField($validator, $field, $idx);
            }
        }
    }

    /**
     * response_format.fields.* のチェック
     * - field_id => 文字列必須
     * - user_answer => "number"
     * - is_correct  => "boolean"
     * - collect_answer => "number"
     * - field_explanation => ja/en 文字列必須
     */
    private function validateResponseField($validator, array $field, int $idx)
    {
        // field_id
        if (empty($field['field_id']) || !is_string($field['field_id'])) {
            $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.field_id", "field_id は必須の文字列です。");
        }

        // user_answer => "number"
        if (!isset($field['user_answer']) || $field['user_answer'] !== 'number') {
            $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.user_answer", "user_answer は 'number' のみ有効です。");
        }

        // is_correct => "boolean"
        if (!isset($field['is_correct']) || $field['is_correct'] !== 'boolean') {
            $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.is_correct", "is_correct は 'boolean' のみ有効です。");
        }

        // collect_answer => "number"
        if (!isset($field['collect_answer']) || $field['collect_answer'] !== 'number') {
            $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.collect_answer", "collect_answer は 'number' のみ有効です。");
        }

        // field_explanation => ja/en
        if (empty($field['field_explanation']) || !is_array($field['field_explanation'])) {
            $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.field_explanation", "field_explanation(オブジェクト) は必須です。");
        } else {
            foreach (self::LANGUAGES as $lang) {
                if (!isset($field['field_explanation'][$lang]) || !is_string($field['field_explanation'][$lang])) {
                    $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.field_explanation.{$lang}", "field_explanation.{$lang} は必須の文字列です。");
                }
            }
        }
    }

    /**
     * metadata.question_type/ input_format.* の検証
     * - question_type => QuestionTypeに存在するか
     * - input_format.fields => attribute=number
     * - question_components => type in (text,image,movie,blank), blank数=fields数
     */
    private function validateMetadata($validator, array $json)
    {
        $meta     = $json['metadata'] ?? [];
        $typeRaw  = $meta['question_type'] ?? null;

        // question_type
        if ($typeRaw) {
            $str = (string)$typeRaw; // 文字列に
            try {
                QuestionType::fromString($str);
            } catch (\InvalidArgumentException) {
                $validator->errors()->add('metadata.question_type', "question_type='{$str}' は未定義です。");
            }
        }

        // input_format.fields
        $fieldsArr = $meta['input_format']['fields'] ?? [];
        if (is_array($fieldsArr)) {
            $fieldIds = [];
            foreach ($fieldsArr as $idx => $f) {
                // f_x
                if (empty($f['field_id']) || !preg_match('/^f_\d+$/', $f['field_id'])) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.field_id", "field_idは f_数字形式が必須です。");
                }

                // 重複check
                $fid = $f['field_id'] ?? '';
                if (in_array($fid, $fieldIds, true)) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.field_id", "field_id='{$fid}' が重複しています。");
                }
                $fieldIds[] = $fid;

                // attribute => number
                if (!isset($f['attribute']) || !in_array($f['attribute'], self::FIELD_TYPE, true)) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.attribute", "attribute='{$f['attribute']}' は 'number' のみ有効です。");
                }

                // collect_answer
                if (!isset($f['collect_answer'])) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.collect_answer", "collect_answer は必須です。");
                }
            }
        }

        // question_components
        $comps = $meta['input_format']['question_components'] ?? [];
        if (is_array($comps)) {
            $blankCount = 0;
            $orders     = [];

            foreach ($comps as $cidx => $c) {
                $cType = $c['type'] ?? null;
                if (!in_array($cType, self::COMPONENT_TYPES, true)) {
                    $validator->errors()->add("metadata.input_format.question_components.{$cidx}.type", "不正なコンポーネントtype='{$cType}'です。");
                }
                if ($cType === 'blank') {
                    $blankCount++;
                    if (!isset($c['field_id'])) {
                        $validator->errors()->add("metadata.input_format.question_components.{$cidx}.field_id", "blank の場合 field_id は必須です。");
                    }
                } else {
                    // text/image/movie => content.{ja,en} が必須
                    if (empty($c['content']) || !is_array($c['content'])) {
                        $validator->errors()->add("metadata.input_format.question_components.{$cidx}.content", "type='{$cType}' の場合 content(オブジェクト) が必須です。");
                    } else {
                        foreach (self::LANGUAGES as $lang) {
                            if (!isset($c['content'][$lang])) {
                                $validator->errors()->add("metadata.input_format.question_components.{$cidx}.content.{$lang}", "content.{$lang} がありません。");
                            }
                        }
                    }
                }
                // order => 数値 + 重複
                if (!isset($c['order']) || !is_numeric($c['order'])) {
                    $validator->errors()->add("metadata.input_format.question_components.{$cidx}.order", "order(数値) は必須です。");
                } else {
                    if (in_array($c['order'], $orders, true)) {
                        $validator->errors()->add("metadata.input_format.question_components.{$cidx}.order", "order='{$c['order']}' が重複しています。");
                    }
                    $orders[] = $c['order'];
                }
            }
            // blank数と fields数 が一致
            $fieldsCount = count($fieldsArr);
            if ($blankCount !== $fieldsCount) {
                $validator->errors()->add("metadata.input_format.question_components", "blank要素数({$blankCount}) と fields数({$fieldsCount}) が一致しません。");
            }
        }
    }
}


```
