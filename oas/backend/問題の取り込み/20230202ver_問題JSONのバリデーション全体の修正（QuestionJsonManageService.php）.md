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

-- input_format.question_components.type 定数
text: テキスト
image: 画像
movie: 動画
blank: 空欄。入力項目

-- input_format.input_components.type 定数
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
evaluation_spec.response_format.fields.collect_answer：必須、オブジェクト（言語定数全て含んでいるか）。metadata.question_type が、FILL_IN_THE_BLANK の時は,オブジェクトの中の値が、evaluation_spec.response_format.fields.user_answerで指定されている型と一致しているか。例えば、"number" の場合は、32 などの数値となっているか。問題の正解（ユーザーには隠す）metadata.question_type が、FILL_IN_OPERATOR の時は,FillInOperator に存在する値になっているか。（ > < など）
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
metadata.input_format.question_components.type：必須、input_format.question_components.type定数と値が合っているか
metadata.input_format.question_components.content：必須、オブジェクト、言語定数と一致する値が全て含まれているか
metadata.input_format.question_components.order: 必須、数値、重複する値が存在しないこと。問題を構築するときの表示順番

metadata.input_format.input_components: metadata.question_type が、FILL_IN_OPERATOR の時は必須。（回答の選択肢。入力キーパッドのボタン）
metadata.input_format.input_components.type：必須、input_format.input_components.type定数と値が合っているか
metadata.input_format.input_components.content：必須、オブジェクト、言語定数と一致するキーが全て含まれているか。FillInOperator に存在する値になっているか。（ > < など）
metadata.input_format.input_components.order: 必須、数値、重複する値が存在しないこと。入力キーパッドを構築するときの表示順番


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

ちゃんと解説してね

変更のブランチ名とコミットメッセージを一行で

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
    case FILL_IN_OPERATOR         = 201;

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
            self::FILL_IN_OPERATOR        => 'FILL_IN_OPERATOR Problem',          // 演算子の選択問題

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
            'FILL_IN_OPERATOR'   => self::FILL_IN_OPERATOR,
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
--- FillInOperator Enum
```php
<?php

namespace App\Enums;

/**
 * 大小比較や等号など、FILL_IN_OPERATOR問題で
 * ユーザーが入力すべき演算子を列挙する Enum
 */
enum FillInOperator: string
{
    case Greater = '>';
    case Less = '<';
    case Equal = '=';

    /**
     * 演算子に対応するラベルを取得します。
     * ここでは演算子そのものをラベルとして返していますが、
     * 必要に応じて日本語/英語混在に変えることもできます。
     */
    public function label(): string
    {
        return match($this) {
            self::Greater => '>',
            self::Less    => '<',
            self::Equal   => '=',
        };
    }

    /**
     * 演算子の配列を取得します。
     * [文字列 => stringラベル] の配列として返す例です。
     * キーをEnumの value にし、バリューを label() にするケースが多いです。
     */
    public static function labels(): array
    {
        return [
            self::Greater->value => self::Greater->label(),
            self::Less->value    => self::Less->label(),
            self::Equal->value   => self::Equal->label(),
        ];
    }
}

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
use App\Enums\QuestionStatus;
use App\Enums\QuestionType;
use App\Models\Question\Question;
use App\Models\Question\QuestionTranslation;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Str;

/**
 * Class QuestionJsonManageService
 *
 * 【概要】
 *   このサービスクラスは、問題データ（QuestionJSON）のバリデーションおよび
 *   DBへの登録（upsert）処理を行う共通ロジックを提供します。
 *   もともと ImportQuestionsFromGithub で行っていた JSON のバリデーション/DB登録を移管し、
 *   他の機能（LLM 生成問題の取り込みなど）でも共通利用できるようにしました。
 *
 * 【機能一覧】
 *   1) validateQuestionJson() ：QuestionJSON 全体をバリデーションする
 *   2) upsertQuestionByJson() ：バリデーション後に Question テーブルへ upsert する
 *   3) validateUserAnswer()   ：ユーザー回答データをバリデーションする（サンプル）
 *   4) localizeMetadata()     ：metadata をアプリケーションロケールに合わせて整形（サンプル）
 *   5) localizeLlmResult()    ：LLM応答をアプリケーションロケールに合わせて整形（サンプル）
 *
 * 【備考】
 *   - validateQuestionJson() では、トップレベルルールの他に、アフターコールバックで
 *     level_id / grade_id / difficulty_id / skills / evaluation_spec / metadata など
 *     詳細チェックを行っています。
 *   - upsertQuestionByJson() 内でバリデーション通過後、Question/QuestionTranslation/Skills の
 *     更新を行うことで、GitHubインポートでも LLM生成問題取り込みでも同一メソッドを利用できます。
 */
class QuestionJsonManageService
{
    /**
     * アプリがサポートする言語一覧
     */
    public const LANGUAGES = ['ja', 'en'];

    /**
     * fields[].user_answer でサポートしている属性
     */
    public const FIELD_TYPE = ['number', 'string'];

    /**
     * question_components.type でサポートしている属性
     */
    public const VALID_COMPONENT_TYPES = ['text','image','movie','input_field','newline','options'];

    /**
     * question_components のうち content(多言語オブジェクト) が必須な type
     */
    private const COMPONENT_TYPES_REQUIRE_CONTENT = ['text','image','movie','options'];

    /**
     * [CHANGED] FILL_IN_OPERATOR のときに必須となる input_components の type/attribute で使える定数
     *  (例: "text","image","movie","blank")
     */
    private const VALID_INPUT_COMPONENTS_TYPES = ['text','image','movie','blank'];

    /**
     * --------------------------------------------------------------------------------
     * QuestionJSON 全体をバリデーション
     * --------------------------------------------------------------------------------
     * 【目的】
     *   1) トップレベルの必須項目（id, order, metadata など）をチェック
     *   2) after コールバックで DB存在チェック (level/grade/difficulty/skills)、
     *      evaluation_spec や metadata構造を追加検証
     *   3) バリデーションに通過しなければ ValidationException を投げる
     *
     * 【戻り値】
     *   - 検証が通ったあとの配列を返す ( Laravel の validate() と同じ )
     *   - 失敗時は例外を投げる
     */
    public function validateQuestionJson(array $json): array
    {
        // トップレベルのルール
        $validator = \Validator::make($json, $this->getTopLevelRules(), $this->messages());

        // after コールバックでさらに詳細検証
        $validator->after(function ($v) use ($json) {
            $this->validateLevelGradeDifficulty($v, $json);
            $this->validateSkills($v, $json);
            $this->validateEvaluationSpec($v, $json);
            $this->validateMetadata($v, $json);
        });

        // 通過した場合は結果を返す
        return $validator->validate();
    }

    /**
     * --------------------------------------------------------------------------------
     * QuestionJSON をデータベースへ upsert
     * --------------------------------------------------------------------------------
     * 【目的】
     *   - まず validateQuestionJson() を実施し、問題がなければ Question / QuestionTranslation /
     *     question_skill ピボット等を更新 (upsert) する。
     *   - GitHubインポート処理や LLM 生成問題の保存などに共通利用できる。
     *
     * 【パラメータ】
     *   @param array $questionJson
     *     - 取り込みたい問題 JSON
     *
     * 【戻り値】
     *   @return bool
     *     - true なら登録成功、 false ならバリデーションエラー等でスキップ扱い
     */
    public function upsertQuestionByJson(array $questionJson): bool
    {
        // バリデーション
        try {
            $this->validateQuestionJson($questionJson);
        } catch (\Illuminate\Validation\ValidationException $ve) {
            // バリデーション失敗 → false
            return false;
        }

        // JSON 内の必須 key チェック
        $jsonId = $questionJson['id'];
        $metadata = $questionJson['metadata'] ?? [];
        if (empty($metadata)) {
            // metadata が空なら取り込み不可
            return false;
        }

        // Question レコードの新規 or 既存を確認
        /** @var Question|null $question */
        $question = Question::where('json_id', $jsonId)->first();

        // JSON上の order, version, status を取得
        $jsonOrder    = $questionJson['order'] ?? 9999;
        $version      = $questionJson['version'] ?? '0.0.1';
        $rawStatus    = $questionJson['status'] ?? null;
        $statusValue  = $rawStatus
            ? $this->parseQuestionStatus($rawStatus)
            : QuestionStatus::DRAFT->value;

        // level/grade/difficulty を DB から取得
        $levelUuid      = $this->findLevelUuid($questionJson['level_id'] ?? null);
        $gradeUuid      = $this->findGradeUuid($questionJson['grade_id'] ?? null);
        $difficultyUuid = $this->findDifficultyUuid($questionJson['difficulty_id'] ?? null);

        // 新規作成 or 既存更新
        if (!$question) {
            // 新規作成
            $question = new Question();
            $question->id      = (string) Str::uuid();
            $question->json_id = $jsonId;

            // order の初期値を設定 (DB内 max+1 と比較)
            $maxOrder = Question::max('order');
            if ($maxOrder === null) {
                $maxOrder = 0;
            }
            $finalOrder = max($maxOrder + 1, $jsonOrder);

            // order が既に使われている場合はインクリメントしていく
            while (Question::where('order', $finalOrder)->exists()) {
                $finalOrder++;
            }
            $question->order = $finalOrder;

        } else {
            // 既存の場合 → order を再調整
            $this->reorderQuestion($question->id, $jsonOrder);
        }

        // Question メイン項目をセット
        $question->level_id      = $levelUuid;
        $question->grade_id      = $gradeUuid;
        $question->difficulty_id = $difficultyUuid;
        $question->version       = $version;
        $question->status        = $statusValue;

        // metadata の反映
        $question->metadata = json_encode($metadata, JSON_UNESCAPED_UNICODE);
        if (!empty($metadata)) {
            $rawQuestionType = $metadata['question_type'] ?? null;
            $questionTypeValue = $this->parseQuestionType($rawQuestionType);
            $question->question_type = $questionTypeValue;
        }

        // evaluation_spec をセット
        if (isset($questionJson['evaluation_spec'])) {
            $eval = $questionJson['evaluation_spec'];
            // evaluation_method (CODE / LLM)
            $question->evaluation_method = isset($eval['evaluation_method'])
                ? EvaluationMethod::fromString($eval['evaluation_method'])
                : null;

            // checker_method
            $question->checker_method = isset($eval['checker_method'])
                ? EvaluationCheckerMethod::fromString($eval['checker_method'])
                : null;

            // LLM関連
            $question->llm_evaluation_prompt_file_name = $eval['llm_prompt_file_name'] ?? null;

            // response_format
            $question->evaluation_response_format = isset($eval['response_format'])
                ? json_encode($eval['response_format'], JSON_UNESCAPED_UNICODE)
                : null;
        }

        // 8) generated_by_llm
        if (array_key_exists('generated_by_llm', $questionJson)) {
            $question->generated_by_llm = (bool)$questionJson['generated_by_llm'];
        }

        // 9) learning_requirements のセット
        $this->applyLearningRequirements($question, $questionJson);

        // 10) セーブ (Question)
        $question->save();

        // 11) question_translations の upsert
        $this->upsertQuestionTranslations($question, $questionJson);

        // 12) skills → question_skill ピボットの更新
        $this->applySkillsPivot($question, $questionJson);

        return true;
    }

    /**
     * --------------------------------------------------------------------------------
     * 既存 question の order を再設定（衝突回避しながら更新）
     * --------------------------------------------------------------------------------
     */
    private function reorderQuestion(string $questionId, int $targetOrder)
    {
        $question = Question::find($questionId);
        if (!$question) {
            return;
        }

        // targetOrder が既に使われていれば +1 する
        while (Question::where('order', $targetOrder)
            ->where('id','!=',$questionId)
            ->exists()) {
            $targetOrder++;
        }
        $question->order = $targetOrder;
        $question->save();
    }

    /**
     * --------------------------------------------------------------------------------
     * skills 配列を DBの question_skill ピボットに反映する
     * --------------------------------------------------------------------------------
     * 【ロジック】
     *  - JSON から skill_id を取り出し、skillsテーブルを検索
     *  - 該当しないスキルはスキップ
     *  - 既存との比較で追加/削除/並び順更新を行う
     */
    private function applySkillsPivot(Question $question, array $questionJson)
    {
        $skillsFromJson = $questionJson['skills'] ?? [];
        $newSkillDbIds = [];
        $skillIndexMap = [];

        foreach ($skillsFromJson as $index => $skillData) {
            $skillJsonId = $skillData['skill_id'] ?? null;
            if (!$skillJsonId) {
                continue;
            }
            $skillDbId = DB::table('skills')
                ->where('json_id', $skillJsonId)
                ->value('id');
            if (!$skillDbId) {
                // DBに該当 skill が無ければスキップ
                continue;
            }

            // インデックス順を order に反映
            $newSkillDbIds[] = $skillDbId;
            $skillIndexMap[$skillDbId] = $index + 1;
        }

        // 既存 pivot
        $existingSkillDbIds = DB::table('question_skill')
            ->where('question_id', $question->id)
            ->pluck('skill_id')
            ->toArray();

        // JSON 側に無い skill を削除
        $skillIdsToRemove = array_diff($existingSkillDbIds, $newSkillDbIds);
        if (!empty($skillIdsToRemove)) {
            DB::table('question_skill')
                ->where('question_id', $question->id)
                ->whereIn('skill_id', $skillIdsToRemove)
                ->delete();
        }

        // JSON 側にある skill を upsert
        foreach ($newSkillDbIds as $skillDbId) {
            $existingPivot = DB::table('question_skill')
                ->where('question_id', $question->id)
                ->where('skill_id', $skillDbId)
                ->first();

            if ($existingPivot) {
                // 既存 → 順番更新
                DB::table('question_skill')
                    ->where('id', $existingPivot->id)
                    ->update([
                        'order'      => $skillIndexMap[$skillDbId],
                        'updated_at' => now(),
                    ]);
            } else {
                // 新規
                DB::table('question_skill')->insert([
                    'id'          => (string) Str::uuid(),
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
     * --------------------------------------------------------------------------------
     * question_translations へ多言語の質問文や解説文を upsert
     * --------------------------------------------------------------------------------
     */
    private function upsertQuestionTranslations(Question $question, array $questionJson)
    {
        $locales = ['ja','en'];
        $meta = $questionJson['metadata'] ?? [];

        foreach ($locales as $locale) {
            // metadata.question_text / metadata.explanation / metadata.background
            $qText = $meta['question_text'][$locale] ?? null;
            $qExp  = $meta['explanation'][$locale]   ?? null;
            $qBack = $meta['background'][$locale]    ?? null;

            // いずれかが存在する場合のみ upsert
            if ($qText !== null || $qExp !== null) {
                QuestionTranslation::updateOrCreate(
                    [
                        'question_id' => $question->id,
                        'locale'      => $locale,
                    ],
                    [
                        'question_text' => $qText,
                        'explanation'   => $qExp,
                        'background'    => $qBack,
                    ]
                );
            }
        }
    }

    /**
     * --------------------------------------------------------------------------------
     * learning_requirements (複数) を Question テーブルへ統合
     * --------------------------------------------------------------------------------
     * 【仕様】
     *  - JSON 内の learning_requirements を学習要件の配列として受け取り、
     *    Questionテーブルの learning_subject, learning_no, ... 等へ改行区切りで集約します。
     */
    private function applyLearningRequirements(Question $question, array $questionJson)
    {
        $items = $questionJson['learning_requirements'] ?? [];
        if (!is_array($items) || empty($items)) {
            return;
        }

        // とりあえず JSON 全体を保持
        $question->learning_requirement_json = json_encode($items, JSON_UNESCAPED_UNICODE);

        $subjects             = [];
        $nos                  = [];
        $requirements         = [];
        $requiredCompetencies = [];
        $backgrounds          = [];
        $categories           = [];
        $gradeLevels          = [];
        $urls                 = [];

        foreach ($items as $lr) {
            if (isset($lr['learning_subject'])) {
                $subjects[] = $lr['learning_subject'];
            }
            if (isset($lr['learning_no'])) {
                $nos[] = (string) $lr['learning_no'];
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

        // 改行区切りでまとめて保存
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
     * --------------------------------------------------------------------------------
     * 下記以降はバリデーション関連メソッド
     * --------------------------------------------------------------------------------
     */

    /**
     * トップレベルのバリデーションルール
     */
    private function getTopLevelRules(): array
    {
        return [
            // 基本情報
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

            // skills / learning_requirements
            'skills'        => ['required','array'],
            'learning_requirements' => ['required','array'],
            'learning_requirements.*.learning_subject'            => ['required','string'],
            'learning_requirements.*.learning_no'                 => ['required','integer'],
            'learning_requirements.*.learning_requirement'        => ['required','string'],
            'learning_requirements.*.learning_required_competency'=> ['required','string'],
            'learning_requirements.*.learning_background'         => ['required','string'],
            'learning_requirements.*.learning_category'           => ['required','string'],
            'learning_requirements.*.learning_grade_level'        => ['required','string'],
            'learning_requirements.*.learning_url'                => ['sometimes','url'],

            // evaluation_spec
            'evaluation_spec'                   => ['required','array'],
            'evaluation_spec.evaluation_method' => ['required','string'],

            // metadata
            'metadata'                          => ['required','array'],
            'metadata.question_type'            => ['required'],
            'metadata.question_text'            => ['required','array'],
            'metadata.question_text.ja'         => ['required','string'],
            'metadata.question_text.en'         => ['required','string'],
            'metadata.explanation'              => ['required','array'],
            'metadata.explanation.ja'           => ['required','string'],
            'metadata.explanation.en'           => ['required','string'],
            'metadata.background'               => ['required','array'],
            'metadata.background.ja'            => ['required','string'],
            'metadata.background.en'            => ['required','string'],
            'metadata.question'                 => ['required','array'],
            'metadata.question.ja'              => ['required','string'],
            'metadata.question.en'              => ['required','string'],
            'metadata.input_format'             => ['required','array'],
            'metadata.input_format.fields'      => ['required','array'],
            'metadata.input_format.question_components' => ['required','array'],
        ];
    }

    /**
     * バリデーションエラーメッセージ
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
     * level_id, grade_id, difficulty_id が DB 上に存在するか確認
     */
    private function validateLevelGradeDifficulty($validator, array $json)
    {
        $levelId = $json['level_id'] ?? null;
        $gradeId = $json['grade_id'] ?? null;
        $diffId  = $json['difficulty_id'] ?? null;

        if ($levelId) {
            $exists = DB::table('levels')->where('json_id', $levelId)->exists();
            if (!$exists) {
                $validator->errors()->add('level_id',
                    "指定された level_id='{$levelId}' はDBに存在しません。"
                );
            }
        }
        if ($gradeId) {
            $exists = DB::table('grades')->where('json_id', $gradeId)->exists();
            if (!$exists) {
                $validator->errors()->add('grade_id',
                    "指定された grade_id='{$gradeId}' はDBに存在しません。"
                );
            }
        }
        if ($diffId) {
            $exists = DB::table('difficulties')->where('json_id', $diffId)->exists();
            if (!$exists) {
                $validator->errors()->add('difficulty_id',
                    "指定された difficulty_id='{$diffId}' はDBに存在しません。"
                );
            }
        }
    }

    /**
     * skills 配列 の内容を DBの skills テーブルと照合
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
                // skill_id or name がないならスキップ
                continue;
            }
            $row = DB::table('skills')->where('json_id', $sid)->first();
            if (!$row) {
                $validator->errors()->add("skills.{$idx}.skill_id",
                    "skill_id='{$sid}' はDBに存在しません。"
                );
                continue;
            }
            // name が DB上の display_name と一致しているか
            if ($row->display_name !== $sname) {
                $validator->errors()->add("skills.{$idx}.name",
                    "skill_id='{$sid}' の display_name と name='{$sname}' が一致しません。"
                );
            }
        }
    }

    /**
     * evaluation_method(CODE/LLM) や checker_method, response_format などを検証
     */
    private function validateEvaluationSpec($validator, array $json)
    {
        $eval = $json['evaluation_spec'] ?? [];
        $methodRaw = $eval['evaluation_method'] ?? null;
        if (!$methodRaw) {
            return;
        }

        // CODE / LLM
        try {
            $method = EvaluationMethod::fromString($methodRaw);
        } catch (\InvalidArgumentException) {
            $validator->errors()->add('evaluation_spec.evaluation_method',
                "evaluation_method='{$methodRaw}' は無効です (CODE/LLM)。"
            );
            return;
        }

        if ($method === EvaluationMethod::CODE) {
            // checker_method 必須
            if (empty($eval['checker_method']) || !is_string($eval['checker_method'])) {
                $validator->errors()->add('evaluation_spec.checker_method',
                    "evaluation_method=CODE のため checker_method が必須です。"
                );
            } else {
                try {
                    EvaluationCheckerMethod::fromString($eval['checker_method']);
                } catch (\InvalidArgumentException) {
                    $validator->errors()->add('evaluation_spec.checker_method',
                        "checker_method='{$eval['checker_method']}' は未定義です。"
                    );
                }
            }
            // response_format 必須
            if (!isset($eval['response_format']) || !is_array($eval['response_format'])) {
                $validator->errors()->add('evaluation_spec.response_format',
                    "evaluation_method=CODE のため response_format が必須です。"
                );
            }
        }
        elseif ($method === EvaluationMethod::LLM) {
            // llm_prompt_file_name 必須
            if (!isset($eval['llm_prompt_file_name'])) {
                $validator->errors()->add('evaluation_spec.llm_prompt_file_name',
                    "evaluation_method=LLM のため llm_prompt_file_name(文字列) が必須です。"
                );
            } else {
                $path = resource_path("prompts/evaluation/{$eval['llm_prompt_file_name']}.txt");
                if (!file_exists($path)) {
                    $validator->errors()->add('evaluation_spec.llm_prompt_file_name',
                        "LLM promptファイルが見つかりません。(path={$path})"
                    );
                }
            }
            if (!isset($eval['response_format']) || !is_array($eval['response_format'])) {
                $validator->errors()->add('evaluation_spec.response_format',
                    "evaluation_method=LLM のため response_format が必須です。"
                );
            }
        }
    }

    /**
     * metadata.question_type / input_format などを検証
     */
    private function validateMetadata($validator, array $json)
    {
        $meta = $json['metadata'] ?? [];
        $qtRaw = $meta['question_type'] ?? null;

        // question_type が有効な Enumか
        $questionTypeValue = null;
        if ($qtRaw) {
            try {
                $questionTypeValue = \App\Enums\QuestionType::fromString((string)$qtRaw)->value;
            } catch (\InvalidArgumentException) {
                $validator->errors()->add('metadata.question_type',
                    "question_type='{$qtRaw}' は未定義です。"
                );
            }
        }

        // fields
        $fieldsArr = $meta['input_format']['fields'] ?? [];
        if (is_array($fieldsArr)) {
            $fieldIds = [];
            foreach ($fieldsArr as $idx => $f) {
                // field_id='f_数字'形式
                if (empty($f['field_id']) || !preg_match('/^f_\d+$/', $f['field_id'])) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.field_id",
                        "field_id='f_数字'形式が必須です。"
                    );
                }
                // 重複チェック
                if (in_array($f['field_id'] ?? '', $fieldIds, true)) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.field_id",
                        "field_id='{$f['field_id']}' が重複しています。"
                    );
                }
                $fieldIds[] = $f['field_id'] ?? '';

                // attribute
                $attrRaw = $f['attribute'] ?? null;
                if (!$attrRaw) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.attribute",
                        "attribute が指定されていません。"
                    );
                } else {
                    // enumチェック
                    try {
                        QuestionMetadataInputFormatFieldAttribute::fromString($attrRaw);
                    } catch (\InvalidArgumentException) {
                        $validator->errors()->add("metadata.input_format.fields.{$idx}.attribute",
                            "attribute='{$attrRaw}' はサポート対象外です。"
                        );
                    }
                }

                // user_answer => 'number' のみ
                if (!isset($f['user_answer']) || !is_string($f['user_answer'])) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.user_answer",
                        "user_answer は必須の文字列です。"
                    );
                } else {
                    if (!in_array($f['user_answer'], self::FIELD_TYPE, true)) {
                        $validator->errors()->add("metadata.input_format.fields.{$idx}.user_answer",
                            "user_answer='{$f['user_answer']}' は 'number' のみ有効です。"
                        );
                    }
                }

                // collect_answer は基本的に metadata では非公開(禁止)
                if (array_key_exists('collect_answer', $f)) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.collect_answer",
                        "collect_answer は指定できません。"
                    );
                }
            }
        }

        // question_components
        $comps = $meta['input_format']['question_components'] ?? [];
        if (is_array($comps)) {
            $inputFieldCount = 0;
            $orders = [];
            foreach ($comps as $cidx => $comp) {
                $ctype = $comp['type'] ?? '';
                if (!in_array($ctype, self::VALID_COMPONENT_TYPES, true)) {
                    $validator->errors()->add("metadata.input_format.question_components.{$cidx}.type",
                        "不正なコンポーネントtype='{$ctype}'です。"
                    );
                }
                // input_field => field_id必須 & カウント
                if ($ctype === 'input_field') {
                    $inputFieldCount++;
                    if (!isset($comp['field_id'])) {
                        $validator->errors()->add("metadata.input_format.question_components.{$cidx}.field_id",
                            "type=input_field の場合 field_id が必須です。"
                        );
                    }
                }
                // content が必須な type
                elseif (in_array($ctype, self::COMPONENT_TYPES_REQUIRE_CONTENT, true)) {
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
                // order => 数値 & 重複チェック
                if (!isset($comp['order']) || !is_numeric($comp['order'])) {
                    $validator->errors()->add("metadata.input_format.question_components.{$cidx}.order",
                        "order(数値) は必須です。"
                    );
                } else {
                    if (in_array($comp['order'], $orders, true)) {
                        $validator->errors()->add("metadata.input_format.question_components.{$cidx}.order",
                            "order='{$comp['order']}' が重複しています。"
                        );
                    }
                    $orders[] = $comp['order'];
                }
            }
            // input_field数 と fields数 が一致するか
            $fieldsCount = count($fieldsArr);
            if ($inputFieldCount !== $fieldsCount) {
                $validator->errors()->add("metadata.input_format.question_components",
                    "input_field要素数({$inputFieldCount}) と fields数({$fieldsCount}) が一致しません。"
                );
            }
        }

        // [CHANGED] 下記: question_type=FILL_IN_OPERATOR の場合、 input_components が必須
        if ($questionTypeValue === QuestionType::FILL_IN_OPERATOR->value) {
            $icKey = 'metadata.input_format.input_components';
            if (!isset($meta['input_format']['input_components'])) {
                $validator->errors()->add($icKey,
                    "FILL_IN_OPERATOR のため input_components が必須です。"
                );
                return; // これ以降の検証は難しいので return
            }

            $inputComps = $meta['input_format']['input_components'];
            if (!is_array($inputComps)) {
                $validator->errors()->add($icKey,
                    "FILL_IN_OPERATOR のため input_components は配列である必要があります。"
                );
                return;
            }

            $icOrders = [];
            foreach ($inputComps as $idx => $ic) {
                // type 必須 & 値チェック
                $typeVal = $ic['type'] ?? null;
                if (!$typeVal) {
                    $validator->errors()->add("{$icKey}.{$idx}.type", 
                        "type が必須です。"
                    );
                } elseif (!in_array($typeVal, self::VALID_INPUT_COMPONENTS_TYPES, true)) {
                    $validator->errors()->add("{$icKey}.{$idx}.type",
                        "不正な type='{$typeVal}' です。"
                    );
                }

                // attribute も必須 & 値チェック
                $attrVal = $ic['attribute'] ?? null;
                if (!$attrVal) {
                    $validator->errors()->add("{$icKey}.{$idx}.attribute",
                        "attribute が必須です。"
                    );
                } elseif (!in_array($attrVal, self::VALID_INPUT_COMPONENTS_TYPES, true)) {
                    $validator->errors()->add("{$icKey}.{$idx}.attribute",
                        "不正な attribute='{$attrVal}' です。"
                    );
                }

                // content => 多言語で FillInOperator の値になっているか
                if (empty($ic['content']) || !is_array($ic['content'])) {
                    $validator->errors()->add("{$icKey}.{$idx}.content",
                        "content(オブジェクト) が必須です。"
                    );
                } else {
                    foreach (self::LANGUAGES as $lang) {
                        if (!isset($ic['content'][$lang])) {
                            $validator->errors()->add("{$icKey}.{$idx}.content.{$lang}",
                                "content.{$lang} がありません。"
                            );
                        } else {
                            $val = $ic['content'][$lang];
                            // FillInOperator enum に含まれるかチェック
                            // 例: ">", "<", "="
                            try {
                                \App\Enums\FillInOperator::from($val);
                            } catch (\ValueError $ve) {
                                $validator->errors()->add("{$icKey}.{$idx}.content.{$lang}",
                                    "不正な演算子 '{$val}' です。"
                                );
                            }
                        }
                    }
                }

                // order => 数値 & 重複
                if (!isset($ic['order']) || !is_numeric($ic['order'])) {
                    $validator->errors()->add("{$icKey}.{$idx}.order",
                        "order(数値) は必須です。"
                    );
                } else {
                    if (in_array($ic['order'], $icOrders, true)) {
                        $validator->errors()->add("{$icKey}.{$idx}.order",
                            "order='{$ic['order']}' が重複しています。"
                        );
                    }
                    $icOrders[] = $ic['order'];
                }
            }
        }
        // [CHANGED] ここまで
    }

    /**
     * --------------------------------------------------------------------------------
     * 文字列→QuestionType enum の int値に変換（未定義なら CALCULATION）
     * --------------------------------------------------------------------------------
     */
    private function parseQuestionType(?string $typeString): int
    {
        if (!$typeString) {
            // デフォルト
            return QuestionType::CALCULATION->value;
        }
        try {
            return QuestionType::fromString($typeString)->value;
        } catch (\InvalidArgumentException) {
            return QuestionType::CALCULATION->value;
        }
    }

    /**
     * --------------------------------------------------------------------------------
     * 文字列→QuestionStatus enum の int値に変換（未定義なら DRAFT）
     * --------------------------------------------------------------------------------
     */
    private function parseQuestionStatus(string $statusString): int
    {
        try {
            return QuestionStatus::fromString($statusString)->value;
        } catch (\InvalidArgumentException) {
            return QuestionStatus::DRAFT->value;
        }
    }

    /**
     * --------------------------------------------------------------------------------
     * level_id → levelsテーブル(json_id)検索 → UUID
     * --------------------------------------------------------------------------------
     */
    private function findLevelUuid(?string $levelJsonId): ?string
    {
        if (!$levelJsonId) {
            return null;
        }
        return DB::table('levels')
            ->where('json_id', $levelJsonId)
            ->value('id') ?: null;
    }

    /**
     * --------------------------------------------------------------------------------
     * grade_id → gradesテーブル(json_id)検索 → UUID
     * --------------------------------------------------------------------------------
     */
    private function findGradeUuid(?string $gradeJsonId): ?string
    {
        if (!$gradeJsonId) {
            return null;
        }
        return DB::table('grades')
            ->where('json_id', $gradeJsonId)
            ->value('id') ?: null;
    }

    /**
     * --------------------------------------------------------------------------------
     * difficulty_id → difficultiesテーブル(json_id)検索 → UUID
     * --------------------------------------------------------------------------------
     */
    private function findDifficultyUuid(?string $difficultyJsonId): ?string
    {
        if (!$difficultyJsonId) {
            return null;
        }
        return DB::table('difficulties')
            ->where('json_id', $difficultyJsonId)
            ->value('id') ?: null;
    }

    /**
     * --------------------------------------------------------------------------------
     * ③ ユーザー回答のバリデーション例
     * --------------------------------------------------------------------------------
     * 【目的】
     *   - question_type='FILL_IN_THE_BLANK' を例にした回答検証
     *   - metadata.input_format との整合性 (fields数/attribute/field_id) をチェック
     *   - ここでは一部簡易実装のみ示している。 (実運用では他 question_type にも対応)
     */
    public function validateUserAnswer(array $answerData, array $metadata): void
    {
        $qt = $metadata['question_type'] ?? null;
        if ($qt !== 'FILL_IN_THE_BLANK') {
            // 他タイプは別途実装を考える
            return;
        }

        // input_format がない
        $inFmt = $metadata['input_format'] ?? [];
        if (!is_array($inFmt)) {
            throw new \Illuminate\Validation\ValidationException(
                \Validator::make([],[]),
                "metadata.input_formatが存在しません。"
            );
        }

        // fields
        $metaFields = $inFmt['fields'] ?? [];
        if (!is_array($metaFields)) {
            throw new \Illuminate\Validation\ValidationException(
                \Validator::make([],[]),
                "metadata.input_format.fields が配列ではありません。"
            );
        }

        // type=fixed/custom
        $type = $inFmt['type'] ?? '';
        if (!in_array($type, ['fixed','custom'], true)) {
            throw new \Illuminate\Validation\ValidationException(
                \Validator::make([],[]),
                "input_format.type='{$type}' は未対応です。"
            );
        }

        // 回答 JSON
        $answerFields = $answerData['fields'] ?? null;
        if (!is_array($answerFields)) {
            throw new \Illuminate\Validation\ValidationException(
                \Validator::make([],[]),
                "回答JSONの 'fields' が配列ではありません。"
            );
        }

        // fixed => fields数が一致するか
        if ($type === 'fixed') {
            $countMeta = count($metaFields);
            $countAns  = count($answerFields);
            if ($countMeta !== $countAns) {
                throw new \Illuminate\Validation\ValidationException(
                    \Validator::make([],[]),
                    "fields数が一致しません。(metadata={$countMeta}, answer={$countAns})"
                );
            }
        }

        // field_id -> 定義マップ
        $metaFieldMap = [];
        foreach ($metaFields as $mf) {
            $fid = $mf['field_id'] ?? null;
            if ($fid) {
                $metaFieldMap[$fid] = $mf;
            }
        }

        // 回答の1件ずつチェック
        foreach ($answerFields as $idx => $af) {
            $fid = $af['field_id'] ?? '';
            if (!$fid) {
                throw new \Illuminate\Validation\ValidationException(
                    \Validator::make([],[]),
                    "fields.{$idx}.field_id は必須です。"
                );
            }
            $attribute = $af['attribute'] ?? null;
            $userAns   = $af['user_answer'] ?? null;

            if (!$attribute) {
                throw new \Illuminate\Validation\ValidationException(
                    \Validator::make([],[]),
                    "fields.{$idx}.attribute が存在しません。"
                );
            }

            // fixed => metadataにない field_id はエラー
            if ($type === 'fixed' && !isset($metaFieldMap[$fid])) {
                throw new \Illuminate\Validation\ValidationException(
                    \Validator::make([],[]),
                    "metadataに存在しない field_id='{$fid}' です。"
                );
            }

            // attribute='number' => user_answer は数値
            if ($attribute === 'number') {
                if (!is_numeric($userAns)) {
                    throw new \Illuminate\Validation\ValidationException(
                        \Validator::make([],[]),
                        "attribute='number' の場合 user_answer には数値が必要です。"
                    );
                }
            } else {
                throw new \Illuminate\Validation\ValidationException(
                    \Validator::make([],[]),
                    "attribute='{$attribute}' は無効です。('number'のみ対応)"
                );
            }
        }
    }

    /**
     * --------------------------------------------------------------------------------
     * ④ metadata を現在のアプリケーションロケールに合わせて整形（サンプル）
     * --------------------------------------------------------------------------------
     */
    public function localizeMetadata(array $metadata): array
    {
        $locale = \App::getLocale();

        // question_text
        if (isset($metadata['question_text']) && is_array($metadata['question_text'])) {
            $metadata['question_text'] = $metadata['question_text'][$locale] ?? '';
        }

        // explanation
        if (isset($metadata['explanation']) && is_array($metadata['explanation'])) {
            $metadata['explanation']   = $metadata['explanation'][$locale] ?? '';
        }

        // background は表示しない例
        if (isset($metadata['background']) && is_array($metadata['background'])) {
            unset($metadata['background']);
        }

        // question
        if (isset($metadata['question']) && is_array($metadata['question'])) {
            $metadata['question'] = $metadata['question'][$locale] ?? '';
        }

        // question_components -> textコンポーネントの content は多言語を取得
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
     * --------------------------------------------------------------------------------
     * ⑤ LLM応答 (evaluation) のローカライズ例
     * --------------------------------------------------------------------------------
     */
    public function localizeLlmResult(array $llmResult): array
    {
        $locale = \App::getLocale();

        $mapped = [];
        // is_correct -> "true"/"false"
        $rawIsCorrect = data_get($llmResult, 'is_correct', false);
        $mapped['is_correct'] = ($rawIsCorrect === true || $rawIsCorrect === "true") ? "true" : "false";

        // score
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
                $mf['is_correct'] = ($rawCorrect === true || $rawCorrect === "true") ? "true" : "false";
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
