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

・もし
question_text
explanation
background
のバリデーションが実装済みであれば削除して、
metadata.question_text: 必須、オブジェクト（言語定数全て含んでいるか）
metadata.explanation: 必須、オブジェクト（言語定数全て含んでいるか）
metadata.background: 必須、オブジェクト（言語定数全て含んでいるか）
へ移動してください。

ーー言語定数
ja, en


-- input_format.fields.type、evaluation_spec.response_format.fields.user_answer,metadata.input_format.fields.user_answer の定数
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
evaluation_spec.checker_method: evaluation_methodが”CODE”の時は必須、文字列型、EvaluationCheckerMethod　に値が存在していること
evaluation_spec.llm_prompt_number: evaluation_methodが”LLM”の時は必須、数値。LLMに投げるプロンプトを管理するファイルと対応している。resources/prompts/evaluation/{x}.txt の {x}の箇所と対応しているので、このファイルが存在しているかバリデーションチェックする。
evaluation_spec.response_format：必須、オブジェクト。LLMに正誤判定を依頼する時に指定するレスポンスの形や、CheckerMethod での正誤判定時のレスポンスに利用する
evaluation_spec.response_format.is_correct：必須、テキスト型("boolean"のみ）。回答全体が正解かどうか表す
evaluation_spec.response_format.score：必須、テキスト型("number"のみ）。スコアを表す
evaluation_spec.response_format.question_text：必須、オブジェクト（言語定数全て含んでいるか）, evaluation_methodが”LLM”の時は、言語ごとにそれぞれ オブジェクトの中の値は "{$言語設定例えば"ja"など}": "text"となっていること。evaluation_methodが”CODE”の時は、言語ごとにそれぞれ question_text と全く同じ値が含まれていること
evaluation_spec.response_format.explanation：必須、オブジェクト（言語定数全て含んでいるか）, evaluation_methodが”LLM”の時は、言語ごとにそれぞれ オブジェクトの中の値は "{$言語設定例えば"ja"など}": "text"となっていること。evaluation_methodが”CODE”の時は、言語ごとにそれぞれ オブジェクトの中の値は "{$言語設定例えば"ja"など}": に問題文となる文字列が含まれていること
evaluation_spec.response_format.question：必須、オブジェクト（言語定数全て含んでいるか）、言語ごとにそれぞれ オブジェクトの中の値は "{$言語設定例えば"ja"など}": "text"となっていること。evaluation_methodが”CODE”の時は、言語ごとにそれぞれ metadata.question と全く同じ値が含まれていること

evaluation_spec.response_format.fields：evaluation_methodが”LLM”の時は必須、配列。CODEのときは正答判定、LLMの時はLLMにユーザーの回答の形を知らせる為にフォーマットを定義。
evaluation_spec.response_format.fields.field_id：必須、配列。ユーザーの回答の型。
evaluation_spec.response_format.fields.user_answer：必須、input_format.fields.type、evaluation_spec.response_format.fields.user_answer,evaluation_spec.input_format.fields.collect_answer の定数に値があるか。ユーザーの回答。
evaluation_spec.response_format.fields.is_correct：必須、テキスト型("boolean"のみ）。ユーザーの回答が正しいか
evaluation_spec.response_format.fields.collect_answer：必須、オブジェクト（言語定数全て含んでいるか）。オブジェクトの中の値が、evaluation_spec.response_format.fields.user_answerで指定されている型と一致しているか。例えば、"number" の場合は、32 などの数値となっているか。問題の正解（ユーザーには隠す）
evaluation_spec.response_format.fields.field_explanation: 必須、オブジェクト、言語ごとにそれぞれ オブジェクトの中の値は "{$言語設定例えば"ja"など}": "文字列"が含まれていること。空文字禁止

metadata.question_type：必須、App\Enums\QuestionType.php に値が存在しているか。JSONには、文字列でも数値でもどちらでも入力可にする

metadata.question_text: 必須、オブジェクト（言語定数全て含んでいるか）
metadata.explanation: 必須、オブジェクト（言語定数全て含んでいるか）
metadata.background: 必須、オブジェクト（言語定数全て含んでいるか）
metadata.question ：必須、オブジェクト（言語定数全て含んでいるか）。問題文

metadata.input_format: 必須、オブジェクト（言語定数全て含んでいるか）
metadata.input_format.fields: 必須、配列。ユーザが回答する入力フォームの仕様を定義
metadata.input_format.fields.field_id: 必須、f_x のフォーマットになっているか。同じ fields 内に重複した値が存在しないか。 question_components内の type: "blank"の数と総数が合っているか。
metadata.input_format.fields.attribute: 必須、（input_format.fields.type、evaluation_spec.response_format.fields.user_answer,evaluation_spec.response_format.fields.collect_answer の定数に値が存在しているか）。ブランクのフォームの属性。例えば number であれば、<input type="number">になる
metadata.input_format.fields.user_answer: 必須、次の条件を満たした文字列型かどうかをチェックする。input_format.fields.type、evaluation_spec.response_format.fields.user_answer,evaluation_spec.response_format.fields.collect_answer の定数に値があるか。ユーザーが入力する正答の型。
metadata.input_format.fields.collect_answer: 絶対に存在してはいけない。ユーザーに回答が見えてしまうため。

metadata.input_format.question_components: 必須（問題を更生する要素）
metadata.input_format.question_components.attribute：必須、input_format.question_components.type定数と値が合っているか
metadata.input_format.question_components.content：必須、オブジェクト、言語定数と一致する値が全て含まれているか
metadata.input_format.question_components.order: 必須、数値、重複する値が存在しないこと。問題を構築するときの表示順番

metadata.input_format.input_components: 必須（回答の選択肢）
metadata.input_format.question_components.attribute：必須、input_format.question_components.type定数と値が合っているか
metadata.input_format.question_components.content：必須、オブジェクト、言語定数と一致する値が全て含まれているか
metadata.input_format.question_components.order: 必須、数値、重複する値が存在しないこと。問題を構築するときの表示順番


--- 問題JSONの全体構造
```json
{
  "order": 100,
  "id": "ques_s1_g3_sec100_u300_diff100_qt51_v100_100",
  "level_id": "lev_003",
  "grade_id": "gra_003",
  "difficulty_id": "diff_100",
  "version": "1.0.0",
  "status": "PUBLISHED",
  "generated_by_llm": false,
  "created_at": "2025-01-01 00:00:00",
  "updated_at": "2025-01-01 00:00:00",
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
      "learning_required_competency": "3～4桁どうしの足し算・引き算を繰り上がり・繰り下がり含め正確に計算できる",
      "learning_background": "筆算の手順をしっかり確立させる",
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
        "ja": "▢にあてはまる数を答えなさい。",
        "en": "Please answer the numbers that fit in the blanks."
      },
      "explanation": {
        "ja": "これは、3桁どうしの足し算を位ごとにわけて考える練習です。百の位、十の位、一の位をそれぞれ計算し、最後に合わせると簡単に正しい合計が求められます。",
        "en": "This exercise practices adding two three-digit numbers by separating the hundreds, tens, and ones places. Calculate each place value separately, then combine them to get the correct total easily."
      },
      "question": {
        "ja": "315 + 276 = (300 + 200) + (10 + 70) + (5 + 6) = ▢ + ▢ + ▢ = ▢",
        "en": "315 + 276 = (300 + 200) + (10 + 70) + (5 + 6) = ▢ + ▢ + ▢ = ▢"
      },
      "fields": [
        {
          "field_id": "f_1",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": 500,
            "en": 500
          },
          "field_explanation": {
            "ja": "300 と 200 を足すと 500 になるからです。",
            "en": "Because adding 300 and 200 results in 500."
          }
        },
        {
          "field_id": "f_2",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": 80,
            "en": 80
          },
          "field_explanation": {
            "ja": "10 と 70 を足すと 80 になるからです。",
            "en": "Because adding 10 and 70 gives 80."
          }
        },
        {
          "field_id": "f_3",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": 11,
            "en": 11
          },
          "field_explanation": {
            "ja": "5 と 6 を足すと 11 になるからです。",
            "en": "Because adding 5 and 6 results in 11."
          }
        },
        {
          "field_id": "f_4",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": 591,
            "en": 591
          },
          "field_explanation": {
            "ja": "500 + 80 + 11 をすべて足すと 591 になるからです。",
            "en": "Because adding 500, 80, and 11 totals 591."
          }
        }
      ]
    }
  },
  "metadata": {
    "question_type": "FILL_IN_THE_BLANK",
    "question_text": {
      "ja": "▢にあてはまる数を答えなさい。",
      "en": "Please answer the numbers that fit in the blanks."
    },
    "explanation": {
      "ja": "3桁の数の足し算では、位を分けて考えることで正確に計算ができるようになります。百の位でまとまりを作り、十の位と一の位は繰り上がりに注意しながら合計しましょう。",
      "en": "When adding three-digit numbers, separating each digit place helps ensure accuracy. Group the hundreds place together and be mindful of any carrying over in the tens or ones places."
    },
    "background": {
      "ja": "この問題は、3桁の足し算に慣れることと、位ごとの計算手順を身につけるためのものです。すべての位を正しく合計すると、簡単に正解にたどりつけます。",
      "en": "This problem is designed to help you become comfortable with three-digit addition and master the step-by-step process of adding each place value correctly."
    },
    "input_format": {
      "type": "fixed",
      "fields": [
        {
          "field_id": "f_1",
          "attribute": "number",
          "user_answer": "number",
          "collect_answer": {
            "ja": 500,
            "en": 500
          }
        },
        {
          "field_id": "f_2",
          "attribute": "number",
          "user_answer": "number",
          "collect_answer": {
            "ja": 80,
            "en": 80
          }
        },
        {
          "field_id": "f_3",
          "attribute": "number",
          "user_answer": "number",
          "collect_answer": {
            "ja": 11,
            "en": 11
          }
        },
        {
          "field_id": "f_4",
          "attribute": "number",
          "user_answer": "number",
          "collect_answer": {
            "ja": 591,
            "en": 591
          }
        }
      ],
      "question_components": [
        {
          "type": "text",
          "content": {
            "ja": "315 + 276 = ",
            "en": "315 + 276 = "
          },
          "order": 10,
          "attribute": "text"
        },
        {
          "type": "newline",
          "order": 15
        },
        {
          "type": "text",
          "content": {
            "ja": "(300 + 200) + (10 + 70) + (5 + 6) = ",
            "en": "(300 + 200) + (10 + 70) + (5 + 6) = "
          },
          "order": 20,
          "attribute": "text"
        },
        {
          "type": "newline",
          "order": 25
        },
        {
          "type": "input_field",
          "field_id": "f_1",
          "order": 30,
          "attribute": "blank",
          "content": {
            "ja": "",
            "en": ""
          }
        },
        {
          "type": "text",
          "content": {
            "ja": " + ",
            "en": " + "
          },
          "order": 40,
          "attribute": "text"
        },
        {
          "type": "input_field",
          "field_id": "f_2",
          "order": 50,
          "attribute": "blank",
          "content": {
            "ja": "",
            "en": ""
          }
        },
        {
          "type": "text",
          "content": {
            "ja": " + ",
            "en": " + "
          },
          "order": 60,
          "attribute": "text"
        },
        {
          "type": "input_field",
          "field_id": "f_3",
          "order": 70,
          "attribute": "blank",
          "content": {
            "ja": "",
            "en": ""
          }
        },
        {
          "type": "text",
          "content": {
            "ja": " = ",
            "en": " = "
          },
          "order": 80,
          "attribute": "text"
        },
        {
          "type": "input_field",
          "field_id": "f_4",
          "order": 90,
          "attribute": "blank",
          "content": {
            "ja": "",
            "en": ""
          }
        }
      ]
    }
  }
}

```



# 実装方針
生成また修正変更するコードはそのコードだけは必ず省略せずに必ず全てアウトプットすること。

修正する場合は修正したコードは全て出力し、修正箇所以外のコードやコメントは現状のままにしてください。

仕様はソースコードにまとめてコメントとして残すこと
コメントを残す時に以下のようにコメントへ処理順を示すようなナンバリングは不用です。
// 7) memo => 空白
// 8) generate_question_prompt => そのまま
// 9) generate_question_prompt_file_name => そのまま

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

--- QuestionJsonManageService
```php
<?php

namespace App\Services\Utils\Question;

use App\Enums\EvaluationCheckerMethod;
use App\Enums\EvaluationMethod;
use App\Enums\QuestionMetadataInputFormatFieldAttribute;
use App\Enums\QuestionType;
use Illuminate\Support\Facades\App;
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\ValidationException;

/**
 * Class QuestionJsonManageService
 *
 * 【概要】
 * このサービスクラスは、問題データ（QuestionJSON）のバリデーションおよび
 * メタデータのローカライズ処理などを担当する。
 * QuestionJSON は GitHub等からインポートされることを想定し、
 * question_typeごとのルール増加にも対応可能な設計としている。
 */
class QuestionJsonManageService
{
    /**
     * 言語定数 (アプリケーション全体で共通利用)
     */
    public const LANGUAGES = ['ja', 'en'];

    /**
     * input_format.type の定数（解答形式）
     */
    public const INPUT_FORMAT_TYPES = [
        'fixed',   // 解答欄数を固定
        'custom',  // 解答欄数を自由
    ];

    /**
     * フィールド系の定数（"number" のみをサポート）
     * （input_format.fields.type, evaluation_spec.response_format.fields.user_answer など）
     */
    public const FIELD_TYPE = [
        'number',
    ];

    /**
     * question_components.type のうち、有効とみなす一覧
     */
    public const VALID_COMPONENT_TYPES = [
        'text',
        'image',
        'movie',
        'input_field',
        'newline',
        'options'
    ];

    /**
     * question_components.type のうち、content が必須となるタイプの一覧
     * 例: text, image, movie は content オブジェクトが必須
     * newline, input_field は content 不要
     */
    private const COMPONENT_TYPES_REQUIRE_CONTENT = [
        'text',
        'image',
        'movie',
        'options'
    ];

    //============================================================
    // メソッド仕様
    //============================================================

    /**
     * validateQuestionJson
     * -------------------------------------------------
     * 【目的】
     *   インポートされる問題JSON(QuestionJSON)全体に対するバリデーションを行う。
     *   トップレベルの共通項目に対するルール検証、および
     *   afterコールバックでDB存在チェック、evaluation_spec、metadataなどをチェックし、
     *   不備があれば ValidationException を throw する。
     *
     * 【パラメータ】
     *   @param array $json
     *     - QuestionJSON全体を連想配列化したデータ
     *
     * 【戻り値】
     *   @return array
     *     - バリデーションが通過した場合、整形済み配列を返す（Laravelのvalidate()仕様）
     *     - エラー時は ValidationException を投げる
     *
     * 【利用場面】
     *   - GitHub等からの一括インポート時や、問題作成画面からの登録時に呼び出す想定
     */
    public function validateQuestionJson(array $json): array
    {
        // トップレベルバリデーション
        $validator = Validator::make($json, $this->getTopLevelRules(), $this->messages());

        // after コールバックでさらに詳細チェック
        $validator->after(function ($v) use ($json) {
            $this->validateLevelGradeDifficulty($v, $json);
            $this->validateSkills($v, $json);
            $this->validateEvaluationSpec($v, $json);
            $this->validateMetadata($v, $json);
        });

        return $validator->validate();
    }

    /**
     * validateUserAnswer
     * -------------------------------------------------
     * 【目的】
     *   ユーザー回答用JSONが、metadata.input_format と整合しているか動的にバリデーションする。
     *   question_type="FILL_IN_THE_BLANK" 向けの実装を行っている。
     *
     * 【パラメータ】
     *   @param array $answerData
     *     - ユーザー回答JSON (例: {"fields":[{"field_id":"f_1","attribute":"number","user_answer":4},...]})
     *   @param array $metadata
     *     - user_questions.metadata に格納されている配列
     *
     * 【処理概要】
     *   - question_type が FILL_IN_THE_BLANK であればチェックを実施
     *   - input_format.type (fixed/custom) ごとにフィールド数や型をチェック
     *   - attribute='number' の場合、実際の回答値が数値か等を確認
     *   - 不整合があれば ValidationException をthrow
     *
     * 【戻り値】
     *   @return void
     *
     * 【利用場面】
     *   - AnswerService などで実際にユーザーの入力を受け取った際に検証する
     *   - 将来的に question_type が増えたら分岐を追加
     */
    public function validateUserAnswer(array $answerData, array $metadata): void
    {
        // question_type='FILL_IN_THE_BLANK' 以外はスキップ
        $qt = $metadata['question_type'] ?? null;
        if ($qt !== 'FILL_IN_THE_BLANK') {
            // 未実装
            return;
        }

        // input_format 存在チェック
        $inFmt = $metadata['input_format'] ?? [];
        if (!is_array($inFmt)) {
            throw ValidationException::withMessages([
                'metadata.input_format' => "metadata.input_formatが存在しません。"
            ]);
        }

        // metadata側のfields
        $metaFields = $inFmt['fields'] ?? [];
        if (!is_array($metaFields)) {
            throw ValidationException::withMessages([
                'metadata.input_format.fields' => "metadata.input_format.fields が配列ではありません。"
            ]);
        }

        // fixed/custom
        $type = $inFmt['type'] ?? '';
        if (!in_array($type, ['fixed','custom'], true)) {
            throw ValidationException::withMessages([
                'metadata.input_format.type' => "input_format.type='{$type}' は未対応です。"
            ]);
        }

        // ユーザー回答 fields
        $answerFields = $answerData['fields'] ?? null;
        if (!is_array($answerFields)) {
            throw ValidationException::withMessages([
                'fields' => "回答JSONの 'fields' が配列ではありません。"
            ]);
        }

        // fixed の場合は fields件数の一致チェック
        if ($type === 'fixed') {
            $countMeta = count($metaFields);
            $countAns  = count($answerFields);
            if ($countMeta !== $countAns) {
                throw ValidationException::withMessages([
                    'fields' => "input_format.type=fixed ですが、fields数が一致しません。(metadata={$countMeta}, answer={$countAns})"
                ]);
            }
        }

        // メタデータ上の field_id -> 定義マップ
        $metaFieldMap = [];
        foreach ($metaFields as $mf) {
            $fid = $mf['field_id'] ?? null;
            if ($fid) {
                $metaFieldMap[$fid] = $mf;
            }
        }

        // ユーザー回答fields をチェック
        foreach ($answerFields as $idx => $af) {
            $fid = $af['field_id'] ?? '';
            if (!$fid) {
                throw ValidationException::withMessages([
                    "fields.{$idx}.field_id" => "field_id は必須です。"
                ]);
            }
            $attribute = $af['attribute'] ?? null;
            $userAns   = $af['user_answer'] ?? null;

            if (!$attribute) {
                throw ValidationException::withMessages([
                    "fields.{$idx}.attribute" => "attribute が存在しません。"
                ]);
            }

            // fixed の場合、metadataに存在しない field_idはエラー
            if ($type === 'fixed' && !isset($metaFieldMap[$fid])) {
                throw ValidationException::withMessages([
                    "fields.{$idx}.field_id" => "metadataに存在しない field_id='{$fid}' です。"
                ]);
            }

            // 'number'属性 → 数値回答か？
            if ($attribute === 'number') {
                if (!is_numeric($userAns)) {
                    throw ValidationException::withMessages([
                        "fields.{$idx}.user_answer" => "attribute='number' の場合 user_answer には数値が必要です。"
                    ]);
                }
            } else {
                // 将来的に他の属性があれば追加
                throw ValidationException::withMessages([
                    "fields.{$idx}.attribute" => "attribute='{$attribute}' は無効です。('number'のみ対応)"
                ]);
            }
        }
    }


    /**
     * getTopLevelRules
     * -------------------------------------------------
     * 【目的】
     *   QuestionJSON の最上位キー(order,idなど)に対する必須・形式ルールを定義する。
     *
     * 【戻り値】
     *   @return array Laravelバリデーション用ルール配列
     *
     * 【補足】
     *   - skills, learning_requirements, evaluation_spec, metadata なども
     *     "必須の配列" というレベルでここに定義されている。
     */
    private function getTopLevelRules(): array
    {
        return [
            // 基本必須
            'order'         => ['required','integer'],
            'id'            => ['required','string'],
            'level_id'      => ['required','string'],
            'grade_id'      => ['required','string'],
            'difficulty_id' => ['required','string'],
            'version'       => ['required','regex:/^\d+\.\d+\.\d+$/'],
            'status'        => ['required','string'],
            'generated_by_llm' => ['required','boolean'],
            'created_at'    => ['required','date_format:Y-m-d H:i:s'],
            'updated_at'    => ['required','date_format:Y-m-d H:i:s'],

            // skills => 必須
            'skills' => ['required','array'],

            // learning_requirements => 必須
            'learning_requirements'                               => ['required','array'],
            'learning_requirements.*.learning_subject'            => ['required','string'],
            'learning_requirements.*.learning_no'                 => ['required','integer'],
            'learning_requirements.*.learning_requirement'        => ['required','string'],
            'learning_requirements.*.learning_required_competency'=> ['required','string'],
            'learning_requirements.*.learning_background'         => ['required','string'],
            'learning_requirements.*.learning_category'           => ['required','string'],
            'learning_requirements.*.learning_grade_level'        => ['required','string'],
            'learning_requirements.*.learning_url'                => ['sometimes','url'],

            // evaluation_spec => 必須
            'evaluation_spec'                   => ['required','array'],
            'evaluation_spec.evaluation_method' => ['required','string'],

            // metadata => question_type / question_text / explanation / background / input_format
            'metadata'                              => ['required','array'],
            'metadata.question_type'                => ['required'],

            // question_text, explanation, background の必須チェック(多言語)
            'metadata.question_text'                => ['required','array'],
            'metadata.question_text.ja'             => ['required','string'],
            'metadata.question_text.en'             => ['required','string'],
            'metadata.explanation'                  => ['required','array'],
            'metadata.explanation.ja'               => ['required','string'],
            'metadata.explanation.en'               => ['required','string'],
            'metadata.background'                   => ['required','array'],
            'metadata.background.ja'                => ['required','string'],
            'metadata.background.en'                => ['required','string'],

            'metadata.question'                     => ['required','array'],
            'metadata.question.ja'                  => ['required','string'],
            'metadata.question.en'                  => ['required','string'],

            'metadata.input_format'                 => ['required','array'],
            'metadata.input_format.type'            => ['required','string','in:fixed,custom'],
            'metadata.input_format.fields'          => ['required','array'],
            'metadata.input_format.question_components' => ['required','array'],
        ];
    }

    /**
     * messages
     * -------------------------------------------------
     * 【目的】
     *   バリデーションエラーメッセージを日本語で定義する。
     *
     * 【戻り値】
     *   @return array
     *     - Laravel のバリデーションメッセージ配列
     *
     * 【補足】
     *   - ':attribute' 部分などは実際のフィールドキーに置換される
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
            'in'          => ':attribute に不正な値が指定されました( :values )。',
            'url'         => ':attribute は有効なURL形式で指定してください。',
        ];
    }

    /**
     * validateLevelGradeDifficulty
     * -------------------------------------------------
     * 【目的】
     *   level_id, grade_id, difficulty_id がDBに存在するかチェックする。
     *   存在しない場合は validationエラーを発生させる。
     *
     * 【パラメータ】
     *   @param $validator Laravelのバリデータインスタンス
     *   @param array $json
     *
     * 【戻り値】
     *   @return void
     *
     * 【補足】
     *   - levels, grades, difficulties テーブルに対し json_id で存在確認
     */
    private function validateLevelGradeDifficulty($validator, array $json)
    {
        $levelId = $json['level_id'] ?? null;
        $gradeId = $json['grade_id'] ?? null;
        $diffId  = $json['difficulty_id'] ?? null;

        if ($levelId) {
            $exists = \DB::table('levels')->where('json_id', $levelId)->exists();
            if (!$exists) {
                $validator->errors()->add('level_id', "指定された level_id='{$levelId}' はDBに存在しません。");
            }
        }
        if ($gradeId) {
            $exists = \DB::table('grades')->where('json_id', $gradeId)->exists();
            if (!$exists) {
                $validator->errors()->add('grade_id', "指定された grade_id='{$gradeId}' はDBに存在しません。");
            }
        }
        if ($diffId) {
            $exists = \DB::table('difficulties')->where('json_id', $diffId)->exists();
            if (!$exists) {
                $validator->errors()->add('difficulty_id', "指定された difficulty_id='{$diffId}' はDBに存在しません。");
            }
        }
    }

    /**
     * validateSkills
     * -------------------------------------------------
     * 【目的】
     *   skills 配列の skill_id と name がDBのskillsテーブル上の json_id, display_name と合致するかチェック
     *
     * 【パラメータ】
     *   @param $validator Laravelのバリデータインスタンス
     *   @param array $json
     *
     * 【戻り値】
     *   @return void
     *
     * 【補足】
     *   - skill_id がDBにない場合や name が一致しない場合はバリデーションエラーを追加
     */
    private function validateSkills($validator, array $json)
    {
        $skills = $json['skills'] ?? [];
        if (!is_array($skills)) {
            return;
        }

        foreach ($skills as $idx => $sk) {
            $sid   = $sk['skill_id'] ?? null;
            $sname = $sk['name']     ?? null;
            if (!$sid || !$sname) {
                // スキップするかどうかは仕様による
                continue;
            }
            $row = \DB::table('skills')->where('json_id', $sid)->first();
            if (!$row) {
                $validator->errors()->add("skills.{$idx}.skill_id",
                    "skill_id='{$sid}' はDBに存在しません。");
                continue;
            }
            if ($row->display_name !== $sname) {
                $validator->errors()->add("skills.{$idx}.name",
                    "skill_id='{$sid}' の display_name と name='{$sname}' が一致しません。");
            }
        }
    }

    /**
     * validateEvaluationSpec
     * -------------------------------------------------
     * 【目的】
     *   evaluation_spec(evaluation_method, checker_method, llm_prompt_numberなど) のバリデーション
     *   CODE/LLM ごとに必要な項目と形式があるかを確認
     *
     * 【パラメータ】
     *   @param $validator Laravelのバリデータインスタンス
     *   @param array $json
     *
     * 【戻り値】
     *   @return void
     */
    private function validateEvaluationSpec($validator, array $json)
    {
        $eval = $json['evaluation_spec'] ?? [];
        $methodRaw = $eval['evaluation_method'] ?? null;
        if (!$methodRaw) {
            return;
        }

        // CODE or LLM
        try {
            $method = EvaluationMethod::fromString($methodRaw);
        } catch (\InvalidArgumentException) {
            $validator->errors()->add('evaluation_spec.evaluation_method',
                "evaluation_method='{$methodRaw}' は無効です (CODE/LLM)。");
            return;
        }

        // CODE の場合
        if ($method === EvaluationMethod::CODE) {
            // checker_method 必須
            if (empty($eval['checker_method']) || !is_string($eval['checker_method'])) {
                $validator->errors()->add('evaluation_spec.checker_method',
                    "evaluation_method=CODE のため checker_method(文字列) が必須です。");
            } else {
                // enumチェック
                try {
                    EvaluationCheckerMethod::fromString($eval['checker_method']);
                } catch (\InvalidArgumentException) {
                    $validator->errors()->add('evaluation_spec.checker_method',
                        "checker_method='{$eval['checker_method']}' は未定義です。");
                }
            }

            // response_format 必須
            if (!isset($eval['response_format']) || !is_array($eval['response_format'])) {
                $validator->errors()->add('evaluation_spec.response_format',
                    "evaluation_method=CODE のため response_format(オブジェクト) が必須です。");
            } else {
                $this->validateResponseFormatForCode($validator, $json, $eval['response_format']);
            }
        }
        // LLM の場合
        elseif ($method === EvaluationMethod::LLM) {
            // llm_prompt_number 必須
            if (!isset($eval['llm_prompt_number']) || !is_numeric($eval['llm_prompt_number'])) {
                $validator->errors()->add('evaluation_spec.llm_prompt_number',
                    "evaluation_method=LLM のため llm_prompt_number(数値) が必須です。");
            } else {
                $path = resource_path("prompts/evaluation/{$eval['llm_prompt_number']}.txt");
                if (!file_exists($path)) {
                    $validator->errors()->add('evaluation_spec.llm_prompt_number',
                        "LLM promptファイルが見つかりません。(path={$path})");
                }
            }

            if (!isset($eval['response_format']) || !is_array($eval['response_format'])) {
                $validator->errors()->add('evaluation_spec.response_format',
                    "evaluation_method=LLM のため response_format(オブジェクト) が必須です。");
            } else {
                $this->validateResponseFormatForLlm($validator, $eval['response_format']);
            }
        }
    }


    /**
     * validateResponseFormatForCode
     * -------------------------------------------------
     * 【目的】
     *   evaluation_method=CODE の場合の response_format 構造を検証
     *   - is_correct, score が "boolean","number" か
     *   - question_text, question などが metadata と一致しているか
     *   - fields 配列があれば 1件ごとに検証
     *
     * 【パラメータ】
     *   @param $validator
     *   @param array $json 全体
     *   @param array $resp response_format のオブジェクト
     *
     * 【戻り値】
     *   @return void
     */
    private function validateResponseFormatForCode($validator, array $json, array $resp)
    {
        if (($resp['is_correct'] ?? '') !== 'boolean') {
            $validator->errors()->add('evaluation_spec.response_format.is_correct',
                "CODE: is_correct は 'boolean' のみ有効です。");
        }

        if (($resp['score'] ?? '') !== 'number') {
            $validator->errors()->add('evaluation_spec.response_format.score',
                "CODE: score は 'number' のみ有効です。");
        }

        // question_text => metadata一致チェック
        if (empty($resp['question_text']) || !is_array($resp['question_text'])) {
            $validator->errors()->add('evaluation_spec.response_format.question_text',
                "CODE: question_text(オブジェクト) が必須です。");
        } else {
            $this->validateResponseFormatQuestionTextForCode($validator, $json, $resp['question_text']);
        }

        // explanation => 今回は必須にしておくが、一致チェックはしない
        if (empty($resp['explanation']) || !is_array($resp['explanation'])) {
            $validator->errors()->add('evaluation_spec.response_format.explanation',
                "CODE: explanation(オブジェクト) が必須です。");
        }

        // question => metadata.question と一致
        if (empty($resp['question']) || !is_array($resp['question'])) {
            $validator->errors()->add('evaluation_spec.response_format.question',
                "CODE: question(オブジェクト) が必須です。");
        } else {
            $this->validateResponseFormatQuestionForCode($validator, $json, $resp['question']);
        }

        // fields => あれば検証
        if (!empty($resp['fields']) && is_array($resp['fields'])) {
            foreach ($resp['fields'] as $idx => $f) {
                $this->validateResponseFieldForCode($validator, $f, $idx);
            }
        }
    }

    /**
     * validateResponseFormatQuestionTextForCode
     * -------------------------------------------------
     * 【目的】
     *   CODE 用の question_text が metadata.question_text と一致しているか検証
     *
     * 【パラメータ】
     *   @param $validator
     *   @param array $json
     *   @param array $obj question_text オブジェクト
     *
     * 【戻り値】
     *   @return void
     */
    private function validateResponseFormatQuestionTextForCode($validator, array $json, array $obj)
    {
        foreach (self::LANGUAGES as $lang) {
            $expected = $json['metadata']['question_text'][$lang] ?? '';
            $actual   = $obj[$lang] ?? '';
            if ($actual !== $expected) {
                $validator->errors()->add(
                    "evaluation_spec.response_format.question_text.{$lang}",
                    "CODE: question_text.{$lang} は metadata.question_text.{$lang} と一致する必要があります( expected='{$expected}' )。"
                );
            }
        }
    }

    /**
     * validateResponseFormatQuestionForCode
     * -------------------------------------------------
     * 【目的】
     *   CODE 用の question が metadata.question と一致しているか検証
     *
     * 【パラメータ】
     *   @param $validator
     *   @param array $json
     *   @param array $obj question オブジェクト
     *
     * 【戻り値】
     *   @return void
     */
    private function validateResponseFormatQuestionForCode($validator, array $json, array $obj)
    {
        $metaQ = $json['metadata']['question'] ?? [];
        foreach (self::LANGUAGES as $lang) {
            $expected = $metaQ[$lang] ?? '';
            $actual   = $obj[$lang] ?? '';
            if ($actual !== $expected) {
                $validator->errors()->add("evaluation_spec.response_format.question.{$lang}",
                    "CODE: question.{$lang} は metadata.question.{$lang} と一致する必要があります( expected='{$expected}' )。");
            }
        }
    }

    /**
     * validateResponseFieldForCode
     * -------------------------------------------------
     * 【目的】
     *   CODE 用 fields の 1要素をバリデーションする
     *   - user_answer, collect_answer, field_explanation等
     *
     * 【パラメータ】
     *   @param $validator
     *   @param array $field
     *   @param int $idx
     *
     * 【戻り値】
     *   @return void
     */
    private function validateResponseFieldForCode($validator, array $field, int $idx)
    {
        if (empty($field['field_id']) || !is_string($field['field_id'])) {
            $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.field_id",
                "CODE: field_id は必須の文字列です。");
        }

        if (!empty($field['user_answer'])) {
            if (!in_array($field['user_answer'], self::FIELD_TYPE, true)) {
                $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.user_answer",
                    "CODE: user_answer='{$field['user_answer']}' は無効です。(例:'number')");
            }
        }

        if (!empty($field['is_correct']) && $field['is_correct'] !== 'boolean') {
            $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.is_correct",
                "CODE: is_correct は 'boolean' のみ有効です。");
        }

        if (($field['user_answer'] ?? '') === 'number') {
            if (!isset($field['collect_answer']) || !is_numeric($field['collect_answer'])) {
                $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.collect_answer",
                    "CODE: collect_answer は数値が必要です (user_answer='number')。");
            }
        }

        if (!empty($field['field_explanation']) && is_array($field['field_explanation'])) {
            foreach (self::LANGUAGES as $lang) {
                $txt = $field['field_explanation'][$lang] ?? null;
                if (!is_string($txt) || trim($txt) === '') {
                    $validator->errors()->add(
                        "evaluation_spec.response_format.fields.{$idx}.field_explanation.{$lang}",
                        "CODE: field_explanation.{$lang} は必須の文字列で、空文字は許可されません。"
                    );
                }
            }
        }
    }

    /**
     * validateResponseFormatForLlm
     * -------------------------------------------------
     * 【目的】
     *   evaluation_method=LLM の場合の response_format 構造を検証
     *   - is_correct, score が "boolean","number" か
     *   - question_text, explanation, question が langごとに "text" かどうか
     *   - fields は必須で、user_answer='number' なら collect_answerは数値
     *
     * 【パラメータ】
     *   @param $validator
     *   @param array $resp
     *
     * 【戻り値】
     *   @return void
     */
    private function validateResponseFormatForLlm($validator, array $resp)
    {
        if (($resp['is_correct'] ?? '') !== 'boolean') {
            $validator->errors()->add('evaluation_spec.response_format.is_correct',
                "LLM: is_correct は 'boolean' のみ有効です。");
        }

        if (($resp['score'] ?? '') !== 'number') {
            $validator->errors()->add('evaluation_spec.response_format.score',
                "LLM: score は 'number' のみ有効です。");
        }

        // question_text => langごと 'text'
        if (empty($resp['question_text']) || !is_array($resp['question_text'])) {
            $validator->errors()->add('evaluation_spec.response_format.question_text',
                "LLM: question_text(オブジェクト) は必須です。");
        } else {
            foreach (self::LANGUAGES as $lang) {
                if (!isset($resp['question_text'][$lang])) {
                    $validator->errors()->add("evaluation_spec.response_format.question_text.{$lang}",
                        "LLM: question_text.{$lang} がありません。");
                } else {
                    if ($resp['question_text'][$lang] !== 'text') {
                        $validator->errors()->add("evaluation_spec.response_format.question_text.{$lang}",
                            "LLM: question_text.{$lang} は 'text' のみ有効です。"
                        );
                    }
                }
            }
        }

        // explanation => langごと 'text'
        if (empty($resp['explanation']) || !is_array($resp['explanation'])) {
            $validator->errors()->add('evaluation_spec.response_format.explanation',
                "LLM: explanation(オブジェクト) は必須です。");
        } else {
            foreach (self::LANGUAGES as $lang) {
                if (!isset($resp['explanation'][$lang])) {
                    $validator->errors()->add("evaluation_spec.response_format.explanation.{$lang}",
                        "LLM: explanation.{$lang} がありません。");
                } else {
                    if ($resp['explanation'][$lang] !== 'text') {
                        $validator->errors()->add("evaluation_spec.response_format.explanation.{$lang}",
                            "LLM: explanation.{$lang} は 'text' のみ有効です。"
                        );
                    }
                }
            }
        }

        // question => langごと 'text'
        if (empty($resp['question']) || !is_array($resp['question'])) {
            $validator->errors()->add('evaluation_spec.response_format.question',
                "LLM: question(オブジェクト) は必須です。");
        } else {
            foreach (self::LANGUAGES as $lang) {
                if (!isset($resp['question'][$lang])) {
                    $validator->errors()->add("evaluation_spec.response_format.question.{$lang}",
                        "LLM: question.{$lang} がありません。");
                } else {
                    if ($resp['question'][$lang] !== 'text') {
                        $validator->errors()->add("evaluation_spec.response_format.question.{$lang}",
                            "LLM: question.{$lang} は 'text' のみ有効です。"
                        );
                    }
                }
            }
        }

        // fields => 必須配列
        if (empty($resp['fields']) || !is_array($resp['fields'])) {
            $validator->errors()->add('evaluation_spec.response_format.fields',
                "LLM: fields(配列) は必須です。");
        } else {
            foreach ($resp['fields'] as $idx => $field) {
                $this->validateResponseFieldForLlm($validator, $field, $idx);
            }
        }
    }

    /**
     * validateResponseFieldForLlm
     * -------------------------------------------------
     * 【目的】
     *   LLM 用 fields の1要素をバリデーション
     *   - field_id, user_answer="number", is_correct="boolean", collect_answer=数値, field_explanation(多言語)
     *
     * 【パラメータ】
     *   @param $validator
     *   @param array $field
     *   @param int $idx
     *
     * 【戻り値】
     *   @return void
     */
    private function validateResponseFieldForLlm($validator, array $field, int $idx)
    {
        if (empty($field['field_id']) || !is_string($field['field_id'])) {
            $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.field_id",
                "LLM: field_id は必須の文字列です。");
        }

        if (($field['user_answer'] ?? '') !== 'number') {
            $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.user_answer",
                "LLM: user_answer は 'number' のみ有効です。");
        }

        if (($field['is_correct'] ?? '') !== 'boolean') {
            $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.is_correct",
                "LLM: is_correct は 'boolean' のみ有効です。");
        }

        if (($field['user_answer'] ?? '') === 'number') {
            if (!isset($field['collect_answer']) || !is_numeric($field['collect_answer'])) {
                $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.collect_answer",
                    "LLM: collect_answer は数値を指定してください (user_answer='number')。"
                );
            }
        }

        if (empty($field['field_explanation']) || !is_array($field['field_explanation'])) {
            $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.field_explanation",
                "LLM: field_explanation(オブジェクト) は必須です。");
        } else {
            foreach (self::LANGUAGES as $lang) {
                if (!isset($field['field_explanation'][$lang])
                    || !is_string($field['field_explanation'][$lang])
                    || trim($field['field_explanation'][$lang]) === ''
                ) {
                    $validator->errors()->add(
                        "evaluation_spec.response_format.fields.{$idx}.field_explanation.{$lang}",
                        "LLM: field_explanation.{$lang} は非空文字列が必須です。"
                    );
                }
            }
        }
    }

    /**
     * validateMetadata
     * -------------------------------------------------
     * 【目的】
     *   metadata 以下の question_type, question_text, explanation, background, input_format の整合性を検証
     *   - question_type が QuestionType enum か
     *   - input_format.fields => field_id重複なし, attribute='number' など
     *   - question_components => type='input_field'なら field_id 必須、type='newline'なら content不要 など
     *
     * 【パラメータ】
     *   @param $validator
     *   @param array $json 全体
     *
     * 【戻り値】
     *   @return void
     */
    private function validateMetadata($validator, array $json)
    {
        $meta = $json['metadata'] ?? [];
        $qtRaw = $meta['question_type'] ?? null;

        // question_type => Enumチェック
        if ($qtRaw) {
            try {
                QuestionType::fromString((string)$qtRaw);
            } catch (\InvalidArgumentException) {
                $validator->errors()->add('metadata.question_type',
                    "question_type='{$qtRaw}' は未定義です。");
            }
        }

        // input_format.type が self::INPUT_FORMAT_TYPES に含まれているか
        $inputFormatType = $meta['input_format']['type'] ?? null;
        if ($inputFormatType && !in_array($inputFormatType, self::INPUT_FORMAT_TYPES, true)) {
            $validator->errors()->add(
                'metadata.input_format.type',
                "input_format.type='{$inputFormatType}' は有効な値ではありません。("
                . implode(',', self::INPUT_FORMAT_TYPES) . " のいずれかを指定してください)"
            );
        }

        // fields の定義チェック
        $fieldsArr = $meta['input_format']['fields'] ?? [];
        if (is_array($fieldsArr)) {
            $fieldIds = [];
            foreach ($fieldsArr as $idx => $f) {
                // field_id => f_\d+ 形式
                if (empty($f['field_id']) || !preg_match('/^f_\d+$/', $f['field_id'])) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.field_id",
                        "field_id='f_数字'形式が必須です。");
                }

                // 重複チェック
                if (in_array($f['field_id'] ?? '', $fieldIds, true)) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.field_id",
                        "field_id='{$f['field_id']}' が重複しています。");
                }
                $fieldIds[] = $f['field_id'] ?? '';

                // attribute => 'number'
                $attrRaw = $f['attribute'] ?? null;
                if (!$attrRaw) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.attribute",
                        "attribute が指定されていません。");
                    continue;
                }
                // Enum化 (InputFieldAttribute::fromString)
                try {
                    QuestionMetadataInputFormatFieldAttribute::fromString($attrRaw);
                } catch (\InvalidArgumentException $e) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.attribute",
                        "attribute='{$attrRaw}' はサポート対象外です。(例:'number','text','textarea')");
                }

                // user_answer => "number"
                if (!isset($f['user_answer']) || !is_string($f['user_answer'])) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.user_answer",
                        "user_answer は必須の文字列です(例:'number')。");
                } else {
                    if (!in_array($f['user_answer'], self::FIELD_TYPE, true)) {
                        $validator->errors()->add("metadata.input_format.fields.{$idx}.user_answer",
                            "user_answer='{$f['user_answer']}' は 'number' のみ有効です。");
                    }
                }
            }
        }

        // question_components の検証 (input_field数==fields数など)
        $comps = $meta['input_format']['question_components'] ?? [];
        if (is_array($comps)) {
            $inputFieldCount = 0;
            $orders = [];
            foreach ($comps as $cidx => $comp) {
                $ctype = $comp['type'] ?? '';
                if (!in_array($ctype, self::VALID_COMPONENT_TYPES, true)) {
                    $validator->errors()->add("metadata.input_format.question_components.{$cidx}.type",
                        "不正なコンポーネントtype='{$ctype}'です。");
                }

                // input_field => field_id 必須
                if ($ctype === 'input_field') {
                    $inputFieldCount++;
                    if (!isset($comp['field_id'])) {
                        $validator->errors()->add("metadata.input_format.question_components.{$cidx}.field_id",
                            "type=input_field の場合 field_id が必須です。");
                    }
                }
                // newline => content 不要
                elseif (in_array($ctype, self::COMPONENT_TYPES_REQUIRE_CONTENT, true)) {
                    // text, image, movie は content 必須
                    if (empty($comp['content']) || !is_array($comp['content'])) {
                        $validator->errors()->add(
                            "metadata.input_format.question_components.{$cidx}.content",
                            "type='{$ctype}' の場合 content(オブジェクト) が必須です。"
                        );
                    } else {
                        // 多言語キーがあるか
                        foreach (self::LANGUAGES as $lang) {
                            if (!isset($comp['content'][$lang])) {
                                $validator->errors()->add(
                                    "metadata.input_format.question_components.{$cidx}.content.{$lang}",
                                    "content.{$lang} がありません。"
                                );
                            }
                        }
                    }
                }
                // order => 数値+重複禁止
                if (!isset($comp['order']) || !is_numeric($comp['order'])) {
                    $validator->errors()->add("metadata.input_format.question_components.{$cidx}.order",
                        "order(数値) は必須です。");
                } else {
                    if (in_array($comp['order'], $orders, true)) {
                        $validator->errors()->add("metadata.input_format.question_components.{$cidx}.order",
                            "order='{$comp['order']}' が重複しています。");
                    }
                    $orders[] = $comp['order'];
                }
            }

            // input_field数 と fields数 が一致するか
            $fieldsCount = count($fieldsArr);
            if ($inputFieldCount !== $fieldsCount) {
                $validator->errors()->add("metadata.input_format.question_components",
                    "input_field要素数({$inputFieldCount}) と fields数({$fieldsCount}) が一致しません。");
            }
        }
    }

    /**
     * localizeMetadata
     * -------------------------------------------------
     * 【目的】
     *   QuestionJSON の metadata を、現在のアプリケーションロケールに合わせて
     *   特定キーを文字列化する（多言語対応のオブジェクト→単一言語の文字列に変換）。
     *   background はレスポンス不要として削除する例も記載。
     *
     * 【パラメータ】
     *   @param array $metadata
     *
     * 【戻り値】
     *   @return array ローカライズ後の配列
     *
     * 【補足】
     *   - "text" type コンポーネントの場合は content[locale] のみを取り出す
     *   - background は削除
     */
    public function localizeMetadata(array $metadata): array
    {
        $locale = App::getLocale();

        if (isset($metadata['question_text']) && is_array($metadata['question_text'])) {
            $metadata['question_text'] = $metadata['question_text'][$locale] ?? '';
        }
        if (isset($metadata['explanation']) && is_array($metadata['explanation'])) {
            $metadata['explanation']   = $metadata['explanation'][$locale] ?? '';
        }

        if (isset($metadata['background']) && is_array($metadata['background'])) {
            unset($metadata['background']);
        }

        if (isset($metadata['question']) && is_array($metadata['question'])) {
            $metadata['question'] = $metadata['question'][$locale] ?? '';
        }

        if (isset($metadata['input_format']['question_components'])
            && is_array($metadata['input_format']['question_components'])) {
            foreach ($metadata['input_format']['question_components'] as $i => $comp) {
                if (($comp['type'] ?? '') === 'text'
                    && isset($comp['content'])
                    && is_array($comp['content'])) {
                    $metadata['input_format']['question_components'][$i]['content']
                        = $comp['content'][$locale] ?? '';
                }
            }
        }

        return $metadata;
    }

    /**
     * localizeLlmResult
     * -------------------------------------------------
     * 【目的】
     *   LLMの評価結果JSON( is_correct, score, question_text[ja/en], fields[*] など ) を
     *   現行ロケールに準じて単一文字列に変換する。
     *
     * 【パラメータ】
     *   @param array $llmResult
     *     - LLM API が返す JSON形式 (例: {is_correct, score, question_text:{ja:"...",en:"..."}, ...})
     *
     * 【戻り値】
     *   @return array
     *     - ローカライズ後の配列 {"is_correct"=>"false","question_text"=>"...", ...}
     *
     * 【利用場面】
     *   - AnswerServiceなどで LLM結果をユーザー向けに返す際に、現在言語の文言に整形
     */
    public function localizeLlmResult(array $llmResult): array
    {
        $locale = App::getLocale();

        $mapped = [];
        // is_correct -> "true"/"false"
        $rawIsCorrect = data_get($llmResult, 'is_correct', false);
        $mapped['is_correct'] = ($rawIsCorrect === true || $rawIsCorrect === "true") ? "true" : "false";

        // score -> そのまま(数値)
        $mapped['score'] = data_get($llmResult, 'score', 0);

        // question_text
        $qtArr = data_get($llmResult, 'question_text', []);
        $mapped['question_text'] = is_array($qtArr)
            ? data_get($qtArr, $locale, '')
            : '';

        // explanation
        $expArr = data_get($llmResult, 'explanation', []);
        $mapped['explanation'] = is_array($expArr)
            ? data_get($expArr, $locale, '')
            : '';

        // question
        $qArr = data_get($llmResult, 'question', []);
        $mapped['question'] = is_array($qArr)
            ? data_get($qArr, $locale, '')
            : '';

        // fields
        $mapped['fields'] = [];
        $fieldsArr = data_get($llmResult, 'fields', []);
        if (is_array($fieldsArr)) {
            foreach ($fieldsArr as $f) {
                $mf = [];
                $mf['field_id'] = (string) data_get($f, 'field_id', '');
                $mf['user_answer'] = (string) data_get($f, 'user_answer', '');
                $rawCorrect = data_get($f, 'is_correct', false);
                $mf['is_correct'] = ($rawCorrect === true || $rawCorrect === "true") ? "true":"false";
                $mf['collect_answer'] = data_get($f, 'collect_answer', '');

                $fieldExpArr = data_get($f, 'field_explanation', []);
                $mf['field_explanation'] = is_array($fieldExpArr)
                    ? data_get($fieldExpArr, $locale, '')
                    : '';

                $mapped['fields'][] = $mf;
            }
        }

        return $mapped;
    }
}


```
