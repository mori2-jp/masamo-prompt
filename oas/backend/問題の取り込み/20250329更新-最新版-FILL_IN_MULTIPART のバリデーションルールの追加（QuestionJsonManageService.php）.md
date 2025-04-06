以下、QuestionJsonManageService　の　validateQuestionJson　に
FILL_IN_MULTIPART　の場合のバリデーションルールを追加してほしい
修正箇所を知らせてください。

必ず、既存の、FILL_IN_OPERATOR、FILL_IN_THE_BLANK、CLASSIFY_THE_OPTIONS　のバリデーションルールはそのままにしてください。


# 追加するルール
metadata.question_type が、FILL_IN_MULTIPART　の時のルールを追加
- metadata.input_format.input_components: metadata.question_type が、FILL_IN_MULTIPART　の時は必須。（回答の選択肢。入力キーパッドのボタン）FILL_IN_MULTIPART　の時は、signed_number_pad と、その他に入力させたい値（この例では、「あまり」）。

metadata.question_type が、FILL_IN_MULTIPART　の時の、evaluation_spec.response_format.fields.user_answer のルールを追加
必須、FieldType::SEQUENCE　と値が一致すること

metadata.question_type が、FILL_IN_MULTIPART　の時の、metadata.input_format.fields のルールを追加
不要。ユーザーが自由に入力するため、こちらでフィールドを定義して限定する必要がない。


metadata.question_type が、FILL_IN_MULTIPART　の時の、evaluation_spec.evaluation_method のルールを追加
CODEが選択されていて　evaluation_spec.evaluation_method　が必要となれる場合は、　FILL_IN_MULTIPART　の時は、CHECK_BY_EXACT_MATCH　に限定

※注意点
・現在は question_type FILL_IN_THE_BLANK や、CLASSIFY_THE_OPTIONS　などなどの仕様だけですが、将来的に複数の種類が追加されて行く予定です。１００個以上になるかも
バリデーションのルールは、question_typeによっては重複するものも出てくるので、
[FILL_IN_THE_BLANK, SCENARIO, CLASSIFY_THE_OPTIONS]など、バリデーションルールに対応するquestion Type を簡単追加したり取り消したり出来るような実装がよい

・エラーメッセージは日本語で分かりやすく出力して

・バリデーションルールに違反したら処理を止めるのではなく、その問題はスキップという事にしてほしい

・後で分かりやすいようにソースコードのコメントには必ず実装の仕様をまとめて残しておくこと

・アプリケーション全体で使うので言語定数は定数として扱うこと



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
evaluation_spec.response_format.fields.user_answer：必須、FieldType の定数に値があるか。ユーザーの回答。FILL_IN_MULTIPART　の時は、sequence　と値が一致すること。
evaluation_spec.response_format.fields.is_correct：必須、テキスト型("boolean"のみ）。ユーザーの回答が正しいか
evaluation_spec.response_format.fields.collect_answer：必須、オブジェクト（言語定数全て含んでいるか）。metadata.question_type が、FILL_IN_THE_BLANK の時は,オブジェクトの中の値が、evaluation_spec.response_format.fields.user_answerで指定されている型と一致しているか。例えば、"number" の場合は、32 などの数値となっているか。問題の正解（ユーザーには隠す）metadata.question_type が、FILL_IN_OPERATOR の時は,FillInOperator に存在する値になっているか。（ > < など）
evaluation_spec.response_format.fields.field_explanation: 必須、オブジェクト、言語ごとにそれぞれ オブジェクトの中の値は "{$言語設定例えば"ja"など}": "文字列"が含まれていること。空文字禁止

metadata.question_type：必須、App\Enums\QuestionType.php に値が存在しているか。JSONには、文字列でも数値でもどちらでも入力可にする

metadata.question_text: 必須、オブジェクト（言語定数全て含んでいるか）
metadata.explanation: 必須、オブジェクト（言語定数全て含んでいるか）
metadata.background: 必須、オブジェクト（言語定数全て含んでいるか）
metadata.question ：必須、オブジェクト（言語定数全て含んでいるか）。問題文

metadata.input_format: 必須、オブジェクト（言語定数全て含んでいるか）
metadata.input_format.fields: FILL_IN_MULTIPART　の時は不要。ユーザーが自由に入力するため、こちらでフィールドを定義して限定する必要がない。

metadata.input_format.question_components: 必須（問題を更生する要素）
metadata.input_format.question_components.type：必須、VALID_COMPONENT_TYPES 定数と値が合っているか
metadata.input_format.question_components.content：question_components.type　が、COMPONENT_TYPES_REQUIRE_CONTENT に含まれる場合に必須、オブジェクト、言語定数と一致する値が全て含まれているか
metadata.input_format.question_components.order: 必須、数値、重複する値が存在しないこと。問題を構築するときの表示順番

metadata.input_format.input_components: metadata.question_type が、FILL_IN_OPERATOR または、FILL_IN_MULTIPART　の時は必須。（回答の選択肢。入力キーパッドのボタン）FILL_IN_MULTIPART　の時は、question を カンマ区切りで分割した値が全て含まれていること。例：question が、14, 15, 18, 20, 21, 25, 27, 35, 45, 50 だった場合は、14〜50までのそれぞれの分割した input_components が 同じ数（ここでは10個）存在していること。
metadata.input_format.input_components.type：必須、VALID_INPUT_COMPONENTS_TYPES 定数と値が合っているか
metadata.input_format.input_components.content：必須、オブジェクト、言語定数と一致するキーが全て含まれているか。FillInOperator に存在する値になっているか。（ > < など）
metadata.input_format.input_components.order: 必須、数値、重複する値が存在しないこと。入力キーパッドを構築するときの表示順番


--- 問題JSONの全体構造
```json
{
  "order": 100,
  "id": "ques_s1_g3_sec300_u300_diff100_qt351_v100_100",
  "level_id": "lev_003",
  "grade_id": "gra_003",
  "difficulty_id": "diff_100",
  "version": "1.0.0",
  "status": "TEST_PUBLISHED",
  "validation_check": true,
  "generated_by_llm": false,
  "created_at": "2025-03-27 13:00:00",
  "updated_at": "2025-03-27 13:00:00",
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
      "learning_requirement": "計算の意味・方法 割り算 あまりのある除法",
      "learning_required_competency": "あまりのある除法を理解し、商とあまりを正しく表せる。",
      "learning_background": "余りのある除法(13÷4=3あまり1等)を正しく行い、余りが除数未満であることを認識できる。",
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
        "ja": "次の ▢ にあてはまる数を答えなさい。",
        "en": "Please answer the numbers that fit in the blanks."
      },
      "explanation": {
        "ja": "25を4で割ると、4×6=24で1つ余るので「6 あまり 1」となります。",
        "en": "When dividing 25 by 4, 4×6=24, leaving 1 as the remainder, so the answer is “6 remainder 1.”"
      },
      "question": {
        "ja": "25 ÷ 4 = ▢",
        "en": "25 ÷ 4 = ▢"
      },
      "fields": [
        {
          "field_id": "f_1",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": 6,
            "en": 6
          },
          "field_explanation": {
            "ja": "25を4で割ったとき、ちょうど4が6回分（24）になるので商は6です。",
            "en": "When dividing 25 by 4, 4 fits exactly 6 times (24), so the quotient is 6."
          }
        },
        {
          "field_id": "f_2",
          "user_answer": "text",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": "あまり",
            "en": "remainder"
          },
          "field_explanation": {
            "ja": "余りがあるときにつける言葉です。",
            "en": "This word indicates there is a remainder in the division."
          }
        },
        {
          "field_id": "f_3",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": 1,
            "en": 1
          },
          "field_explanation": {
            "ja": "25から4×6=24を引いた残りが1なので、余りは1です。",
            "en": "Subtracting 24 (4×6) from 25 leaves 1, so the remainder is 1."
          }
        }
      ]
    }
  },
  "metadata": {
    "question_type": "FILL_IN_MULTIPART",
    "question_text": {
      "ja": "次の ▢ にあてはまる数を答えなさい。",
      "en": "Please answer the numbers that fit in the blanks."
    },
    "explanation": {
      "ja": "25を4で割ると、4×6=24で1つ余るので「6 あまり 1」となります。",
      "en": "When dividing 25 by 4, 4×6=24, leaving 1 as the remainder, so the answer is “6 remainder 1.”"
    },
    "question": {
      "ja": "25 ÷ 4 = ▢",
      "en": "25 ÷ 4 = ▢"
    },
    "background": {
      "ja": "この問題では、あまりのある除法を正しく理解し、商とあまりをきちんと求める力を身につけることを目的としています。小学校3年生でも、余りが出る割り算の計算に慣れ、答えを『○ あまり ○』の形できちんと表せるようにしましょう。",
      "en": "This problem aims to help learners understand division with remainders and accurately determine both the quotient and the remainder. Even for third-grade students, practicing divisions that result in remainders helps them express the answer correctly in the form 'quotient remainder remainder_value.'"
    },
    "input_format": {
      "input_components": [
        {
          "type": "signed_number_pad",
          "order": 50
        },
        {
          "type": "text",
          "content": {
            "ja": "あまり",
            "en": "remainder"
          },
          "order": 100
        }
      ],
      "question_components": [
        {
          "type": "text",
          "content": {
            "ja": "25 ÷ 4 = ▢",
            "en": "25 ÷ 4 = ▢"
          },
          "order": 50
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
use App\Helpers\CommonLib;

enum EvaluationCheckerMethod: int
{
    case CALCULATE_METHOD = 10;
    case CHECK_BY_EXACT_MATCH = 50;
    case CHECK_BY_UNORDERED_ARRAY_ELEMENTS_MATCH = 100; // 順番を問わず配列の中身が合っているか

    case CHECK_BY_ARRAY_ELEMENTS_MATCH_IN_ORDER = 150; // 配列の中身が合っているか（順番も問う）

    public function label(): string
    {
        return match($this) {
            self::CALCULATE_METHOD          => 'CALCULATE_METHOD',
            self::CHECK_BY_EXACT_MATCH          => 'CHECK_BY_EXACT_MATCH',
            self::CHECK_BY_UNORDERED_ARRAY_ELEMENTS_MATCH  => 'CHECK_BY_UNORDERED_ARRAY_ELEMENTS_MATCH',
            self::CHECK_BY_ARRAY_ELEMENTS_MATCH_IN_ORDER  => 'CHECK_BY_ARRAY_ELEMENTS_MATCH_IN_ORDER',
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
            'CHECK_BY_EXACT_MATCH'          => self::CHECK_BY_EXACT_MATCH,
            'CHECK_BY_UNORDERED_ARRAY_ELEMENTS_MATCH'  => self::CHECK_BY_UNORDERED_ARRAY_ELEMENTS_MATCH,
            'CHECK_BY_ARRAY_ELEMENTS_MATCH_IN_ORDER'  => self::CHECK_BY_ARRAY_ELEMENTS_MATCH_IN_ORDER,
            default => throw new \InvalidArgumentException("Unknown status string: {$statusString}")
        };
    }


    /**
     * このEnumのキーをキャメルケースに変換して、Serviceクラスのメソッド名として使う
     * 例: "CHECK_BY_EXACT_MATCH" → "checkByExactMatch"
     */
    public function methodName(): string
    {
        return CommonLib::snakeUpperToCamel($this->label());
    }
}


```

```php
<?php

namespace App\Enums;

// 問題のカテゴリ（出題形式とは別）
enum FieldType: string
{
    case NUMBER             = 'number';
    case STRING       = 'string';
    case OPERATOR                = 'operator';

    case SEQUENCE                = 'sequence';

    public function label(): string
    {
        return match($this) {
            self::NUMBER            => 'NUMBER',
            self::STRING            => 'STRING',
            self::OPERATOR          => 'OPERATOR',
            self::SEQUENCE          => 'SEQUENCE',
        };
    }

    /**
     * 文字列を受け取り、該当する FieldType を返す静的メソッド
     * 存在しない場合は例外を投げるサンプルです
     *
     * @param string $typeString "FILL_IN_THE_BLANK" など
     * @return FieldType
     */
    public static function fromString(string $typeString): FieldType
    {
        return match($typeString) {
            'NUMBER'       => self::NUMBER,
            'STRING' => self::STRING,
            'OPERATOR'          => self::OPERATOR,
            'SEQUENCE'          => self::SEQUENCE,
            default => throw new \InvalidArgumentException("Unknown QuestionType string: {$typeString}")
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
    case CALCULATION             = 1; // 未使用
    case FILL_IN_THE_BLANK       = 51;
    case SCENARIO                = 101; // 未使用
    case MULTIPLE_CHOICE         = 151; // 未使用
    case FILL_IN_OPERATOR         = 201;
    case CLASSIFY_THE_OPTIONS = 251;
    case ORDER_THE_OPTIONS = 301;
    case FILL_IN_MULTIPART = 351;

    public function label(): string
    {
        return match($this) {
            self::CALCULATION            => 'CALCULATION',              // 計算問題: 数値計算や演算ルールの処理を中心
            self::FILL_IN_THE_BLANK      => 'FILL_IN_THE_BLANK',        // 穴埋め問題: 問題文や式に空所があり、そこを埋める形式
            self::SCENARIO               => 'SCENARIO',                 // シナリオ問題: 状況や経過を踏まえて解決する（回答が一意じゃない）
            self::MULTIPLE_CHOICE        => 'MULTIPLE_CHOICE',          // 選択問題: 複数の選択肢から答えを選ぶ
            self::FILL_IN_OPERATOR        => 'FILL_IN_OPERATOR',          // 演算子の選択問題
            self::CLASSIFY_THE_OPTIONS        => 'CLASSIFY_THE_OPTIONS',        // 分類問題：選択肢を、あらかじめ提示された分類基準（例：割り切れる数）に従って振り分ける形式。
            self::ORDER_THE_OPTIONS     => 'ORDER_THE_OPTIONS', // 順序決定問題: 選択肢を正しい順番に並べる
            self::FILL_IN_MULTIPART => 'FILL_IN_MULTIPART', // 複数パート入力問題: 穴埋め問題のように、回答数分の BLANK は用意されておらず、回答者が必要数分入力する
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
            'CLASSIFY_THE_OPTIONS'   => self::CLASSIFY_THE_OPTIONS,
            'ORDER_THE_OPTIONS'   => self::ORDER_THE_OPTIONS,
            'FILL_IN_MULTIPART'   => self::FILL_IN_MULTIPART,
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

```php
<?php

namespace App\Enums;

/**
 * input_format.fields の attribute で指定できる値をEnum化
 * 例: "number" / "text" / "textarea"
 */
enum QuestionMetadataInputFormatFieldAttribute: string
{
    case NUMBER   = 'number';
    case TEXT     = 'text';
    case TEXTAREA = 'textarea';

    case OPERATOR = 'operator';
    case ARRAY = 'array';

    /**
     * 日本語ラベルや説明を返す例
     */
    public function label(): string
    {
        return match ($this) {
            self::NUMBER   => 'number',
            self::TEXT     => 'text',
            self::TEXTAREA => 'textarea',
            self::OPERATOR => 'operator',
            self::ARRAY => 'array',
        };
    }

    /**
     * 文字列 → Enum の変換
     */
    public static function fromString(string $value): self
    {
        return match ($value) {
            'number'   => self::NUMBER,
            'text'     => self::TEXT,
            'textarea' => self::TEXTAREA,
            'operator' => self::OPERATOR,
            'array' => self::ARRAY,
            default    => throw new \InvalidArgumentException("Unknown InputFieldAttribute: {$value}")
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
use App\Enums\QuestionStatus;
use App\Enums\QuestionType;
use App\Models\Difficulty\Difficulty;
use App\Models\Grade\Grade;
use App\Models\Level\Level;
use App\Models\Question\Question;
use App\Models\Question\QuestionTranslation;
use App\Services\Utils\Question\QuestionJsonManageService;
use Google_Client;
use Google_Service_Drive;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Str;

/**
 * 【仕様まとめ】
 * 1) staging や local 環境で Google Drive 上の JSON を取り込みたい場合、
 *    サービスアカウント (service-account-credentials.json) で認証し、Drive API を介してダウンロードする。
 * 2) Google Drive でファイルを「service account」に対して共有(閲覧可)し、かつ GCP で Drive API を有効化しておく必要がある。
 * 3) $jsonFileOnGoogleDriveIds にファイルIDを記述しておくと、全て順番にダウンロードし upsertQuestion() へ渡す。
 * 4) GitHub からの取り込み (importFromGitHub) は既存の通り残し、環境が 'staging' なら Drive API、その他はローカル設定で切り替え。
 * 5) 既存の upsertQuestion(), processJsonFile(), fetchAllJsonFilesRecursively() 等のロジックはそのまま利用。
 *
 * - すでに登録済みの questions のうち、generated_by_llm = false で、今回ダウンロードした JSON のリストに存在しない json_id のレコードは、status を QuestionStatus::JSON_NOT_MATCHED に更新。
 * - 更新後に、status = QuestionStatus::JSON_NOT_MATCHED のレコードの id, json_id をすべてコンソールに出力する。
 *
 * - 環境が local の時、status が TEST_PUBLISHED の場合はバリデーションをスキップして保存し、スキップした場合は後で id, json_id をコンソールに出力する。
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

    protected $description = 'Import question data from GitHub or Google Drive JSON files and sync DB';

    /**
     * Google Drive 上の "ファイルID" 一覧 (サービスアカウント認証で取得する)
     *
     * 例:
     *   https://drive.google.com/file/d/1jPnYxj8VxeUT5JmnAayPUpl-2BnEJ-ax/view?usp=sharing
     *   ↑のURLの「1jPnYxj8VxeUT5JmnAayPUpl-2BnEJ-ax」がファイルID。
     */
    protected array $jsonFileOnGoogleDriveIds = [
        '1o6db1AIwflqfraGwC6IU2re18Zxtv3Kc',
        '1D0AG1gR9Lt7GO5O8hDqlTgRoVxDGCoMH',
        '1OOxfWOTVixHRqTWpQWXVOM1BcdWMBnxi'
    ];

    /**
     * 「取り込もうとしたJSONの総数」
     */
    private int $totalCount = 0;

    /**
     * 「取り込みに失敗またはスキップしたJSONの総数」
     */
    private int $skipCount = 0;

    /**
     * 「取り込みに成功したJSONの総数」
     */
    private int $successCount = 0;

    /**
     * 今回の取り込み処理で「存在が確認できた json_id」の一覧
     * (すでにDB登録済みか否か、import成功したか否かに関係なく、ダウンロードしたJSONに含まれていたjson_idを格納する)
     */
    private array $recognizedJsonIds = [];

    /**
     * バリデーションをスキップした (環境=local, status=TEST_PUBLISHED) の問題のリストを保管
     * 後で id, json_id をまとめて出力するため
     */
    private array $skippedTestPublishedValidation = [];

    /**
     * メインのハンドラ
     */
    public function handle()
    {
        // 環境判定
        $env = app()->environment(); // 'local', 'staging', etc
        $useGoogleDrive = false;

        if ($env === 'staging') {
            // staging は必ず Google Drive
            $useGoogleDrive = true;
        } elseif ($env === 'local') {
            // local では config('app.use_google_drive_in_local_for_learning_data_import', true) の値を参照
            $useGoogleDrive = config('app.use_google_drive_in_local_for_learning_data_import', true);
        }

        if ($useGoogleDrive) {
            $this->info("【INFO】Running in '{$env}' environment => Google Drive (Service Account認証) からJSONをダウンロードして処理します。");
            $this->importFromGoogleDriveWithServiceAccount();
        } else {
            $this->info("【INFO】Running in '{$env}' environment => GitHub からJSONをダウンロードして処理します。");
            $this->importFromGitHub();
        }

        // 統計出力
        $this->info("Import process completed.");
        $this->line("=======================================");
        $this->info("取り込もうとした Question JSON の総数: {$this->totalCount}");
        $this->info("取り込みに失敗またはスキップした Question JSON の総数: {$this->skipCount}");
        $this->info("取り込みに成功した Question JSON の総数: {$this->successCount}");
        $this->line("=======================================");

        /**
         *   - すでにDBに登録されている question のうち generated_by_llm = false で
         *     今回取り込んだ JSON 一覧($this->recognizedJsonIds) に json_id が含まれないものは、
         *     status を QuestionStatus::JSON_NOT_MATCHED に更新する。
         *   - status=JSON_NOT_MATCHED のレコードの id, json_id を最後に出力。
         */
        $notMatchedQuery = Question::where('generated_by_llm', false)
            ->whereNotIn('json_id', $this->recognizedJsonIds)
            ->where('status', '!=', QuestionStatus::JSON_NOT_MATCHED->value);

        $notMatchedCount = $notMatchedQuery->count();
        if ($notMatchedCount > 0) {
            $this->info("Marking {$notMatchedCount} existing questions as JSON_NOT_MATCHED because no matching JSON was found in this import.");
            $notMatchedQuery->update(['status' => QuestionStatus::JSON_NOT_MATCHED->value]);
        }

        $allNotMatched = Question::where('status', QuestionStatus::JSON_NOT_MATCHED->value)->get(['id','json_id']);
        if ($allNotMatched->isNotEmpty()) {
            $this->info("==== JSON_NOT_MATCHED questions (id, json_id) ====");
            foreach ($allNotMatched as $q) {
                $this->line("id={$q->id}, json_id={$q->json_id}");
            }
        }

        // バリデーションをスキップした (TEST_PUBLISHED) レコードの一覧をコンソール出力
        if (!empty($this->skippedTestPublishedValidation)) {
            $this->info("==== Validation Skipped for TEST_PUBLISHED (Local env) questions ====");
            foreach ($this->skippedTestPublishedValidation as $q) {
                $this->line("id={$q['id']}, json_id={$q['json_id']}");
            }
        }

        return 0;
    }

    /**
     * Google Drive のファイルを
     * Service Account (service-account-credentials.json) で認証して Drive API 経由でダウンロードする。
     *
     * jsonFileOnGoogleDriveIds に記載された複数ファイルを順に読み取り、DBに反映する。
     */
    private function importFromGoogleDriveWithServiceAccount(): void
    {
        $this->info('Setting up Google Client for Drive API (Questions) ...');

        // 1) Googleクライアントの認証設定
        //    例: storage_path('app/private/service-account-credentials.json')
        $client = new Google_Client();
        $client->setAuthConfig(storage_path('app/private/service-account-credentials.json'));
        $client->addScope(Google_Service_Drive::DRIVE_READONLY);

        // 2) Drive サービスインスタンス
        $driveService = new Google_Service_Drive($client);

        // 3) ファイルID をループし、中身(JSON)をダウンロード => upsert
        foreach ($this->jsonFileOnGoogleDriveIds as $fileId) {
            $this->info("Fetching JSON from Google Drive fileId: {$fileId}");

            // 総数カウント
            $this->totalCount++;

            try {
                // alt=media で取得 (認証済ストリーム)
                $response = $driveService->files->get($fileId, ['alt' => 'media']);
                $jsonContent = $response->getBody()->getContents();

                $decoded = json_decode($jsonContent, true);
                if (!is_array($decoded)) {
                    $this->warn("File ID={$fileId} is not valid JSON. Skipped.");
                    $this->skipCount++;
                    continue;
                }

                // DB upsert
                if ($this->upsertQuestion($decoded)) {
                    $this->successCount++;
                } else {
                    $this->skipCount++;
                }
            } catch (\Exception $e) {
                $this->warn("Failed to fetch from Google Drive: fileId={$fileId}, error=" . $e->getMessage());
                $this->skipCount++;
            }
        }
    }

    /**
     * GitHubリポジトリ上の JSON を取り込み（オリジナルロジックを温存）
     */
    private function importFromGitHub(): void
    {
        $token = config('services.github.api_token');
        if (!$token) {
            $this->error('GitHub API token not found in config/services.php [github.api_token]');
            return;
        }

        $repoOwnerAndName = 'NousContentsManagement/masamo-content';

        // ベースパス: contents/questions
        $basePath = 'contents/questions';

        // subject オプション
        $subjectOption = $this->option('subject');
        if ($subjectOption) {
            $basePath .= '/' . $subjectOption;
            $this->info("Subject specified: {$subjectOption}, path= {$basePath}");
        } else {
            $this->info("No subject specified. Will import from all subdirectories under contents/questions");
        }

        $allJsonFiles = $this->fetchAllJsonFilesRecursively($repoOwnerAndName, $basePath, $token);
        if (empty($allJsonFiles)) {
            $this->warn("No JSON files found under path: {$basePath}");
            return;
        }

        $this->info("Found " . count($allJsonFiles) . " JSON file(s) under {$basePath}");

        // 各ファイルを処理
        foreach ($allJsonFiles as $file) {
            $this->processJsonFile($file, $token);
        }
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

            if ($type === 'dir') {
                $descendantFiles = $this->fetchAllJsonFilesRecursively($repo, $itemPath, $token);
                $result = array_merge($result, $descendantFiles);
            } elseif ($type === 'file') {
                if (str_ends_with($name, '.json')) {
                    $result[] = $item;
                }
            }
        }

        return $result;
    }

    /**
     * JSONファイル一件をダウンロードしてパースし、DBに反映 (GitHub用)
     */
    private function processJsonFile(array $file, string $token): void
    {
        $this->totalCount++;

        if (!isset($file['download_url'])) {
            $this->warn("No download_url for file: " . ($file['path'] ?? 'unknown'));
            $this->skipCount++;
            return;
        }

        $this->info("Fetching JSON: " . $file['path']);

        $jsonContent = Http::withToken($token)
            ->get($file['download_url'])
            ->body();

        $decoded = json_decode($jsonContent, true);
        if (!is_array($decoded)) {
            $this->warn("File {$file['name']} is not valid JSON. Skipped.");
            $this->skipCount++;
            return;
        }

        if ($this->upsertQuestion($decoded)) {
            $this->successCount++;
        } else {
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

            // この json_id を「今回認識した」一覧に加える
            $this->recognizedJsonIds[] = $jsonId;

            // metadata が無いならスキップ
            $metadata = $questionJson['metadata'] ?? [];
            if (empty($metadata)) {
                $this->warn("metadata が存在しないためスキップ: json_id={$jsonId}");
                DB::rollBack();
                return false;
            }

            // status 文字列を enum int 値へ
            $rawStatus = $questionJson['status'] ?? null;
            $statusValue = $rawStatus
                ? $this->parseQuestionStatus($rawStatus)
                : QuestionStatus::DRAFT->value;

            // 環境が local かつ status が TEST_PUBLISHED かつ validation_check が false の場合はバリデーションをスキップ
            $env = app()->environment();
            $shouldValidate = true;
            if (
                $env === 'local'
                && $statusValue === QuestionStatus::TEST_PUBLISHED->value
                && array_key_exists('validation_check', $questionJson)
                && $questionJson['validation_check'] === false
            ) {
                $this->info("Skipping validation for question json_id={$jsonId} because environment=local, status=TEST_PUBLISHED, and validation_check=false");
                $shouldValidate = false;
            }

            // JSONバリデーション
            if ($shouldValidate) {
                try {
                    $jsonManager = app(QuestionJsonManageService::class);
                    $jsonManager->validateQuestionJson($questionJson);
                } catch (\Illuminate\Validation\ValidationException $ve) {
                    $this->warn("バリデーションエラーのため問題(json_id={$jsonId})をスキップ:");
                    foreach ($ve->errors() as $field => $msgs) {
                        foreach ($msgs as $msg) {
                            $this->warn("  - {$field}: {$msg}");
                        }
                    }
                    DB::rollBack();
                    return false;
                }
            }

            // 既存レコードを検索 (json_id = $jsonId)
            $question = Question::where('json_id', $jsonId)->first();

            // order, version
            $jsonOrder = $questionJson['order'] ?? 9999;
            $version   = $questionJson['version'] ?? '0.0.1';

            // level_id / difficulty_id / grade_id
            $levelUuid      = $this->findLevelUuid($questionJson['level_id'] ?? null);
            $gradeUuid      = $this->findGradeUuid($questionJson['grade_id'] ?? null);
            $difficultyUuid = $this->findDifficultyUuid($questionJson['difficulty_id'] ?? null);

            // 新規or既存
            if (!$question) {
                $question = new Question();
                $question->uuid   = (string) Str::uuid();
                $question->json_id = $jsonId;
            }

            // 項目反映
            $question->order         = $jsonOrder;
            $question->level_id      = $levelUuid;
            $question->grade_id      = $gradeUuid;
            $question->difficulty_id = $difficultyUuid;
            $question->version       = $version;
            $question->status        = $statusValue;

            // question->metadata
            $question->metadata = json_encode($metadata, JSON_UNESCAPED_UNICODE);

            // question_type
            $rawQuestionType = $metadata['question_type'] ?? null;
            $question->question_type = $this->parseQuestionType($rawQuestionType);

            // evaluation_spec
            if (isset($questionJson['evaluation_spec'])) {
                $em = $questionJson['evaluation_spec']['evaluation_method'] ?? null;
                $cm = $questionJson['evaluation_spec']['checker_method'] ?? null;
                $question->evaluation_method = $em
                    ? EvaluationMethod::fromString($em)
                    : null;
                $question->checker_method = $cm
                    ? EvaluationCheckerMethod::fromString($cm)
                    : null;

                $question->llm_evaluation_prompt_file_name = $questionJson['evaluation_spec']['llm_prompt_file_name'] ?? null;

                $rf = $questionJson['evaluation_spec']['response_format'] ?? null;
                $question->evaluation_response_format = $rf
                    ? json_encode($rf, JSON_UNESCAPED_UNICODE)
                    : null;
            }

            // generated_by_llm
            if (array_key_exists('generated_by_llm', $questionJson)) {
                $question->generated_by_llm = (bool)$questionJson['generated_by_llm'];
            }

            // 学習要件
            $this->applyLearningRequirements($question, $questionJson);

            $question->save();

            // バリデーションをスキップした場合に記録しておく
            if (!$shouldValidate) {
                $this->skippedTestPublishedValidation[] = [
                    'id'       => $question->id,
                    'json_id'  => $question->json_id,
                ];
            }

            // question_text / explanation / background
            $this->upsertQuestionTranslations($question, $questionJson);

            // skills → pivot
            $this->applySkillsPivot($question, $questionJson);

            DB::commit();
            $this->info("Upserted question json_id={$jsonId}");
            return true;

        } catch (\Throwable $e) {
            DB::rollBack();
            $this->error("Error upserting question json_id={$jsonId}: " . $e->getMessage());
            return false;
        }
    }

    /**
     * JSON の "skills" 配列 → question_skill ピボットを同期
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

            // skills テーブルの json_id => id
            $skillDbId = DB::table('skills')
                ->where('json_id', $skillJsonId)
                ->value('id');

            if (!$skillDbId) {
                $this->warn("Skill with json_id='{$skillJsonId}' not found. Skipped linking.");
                continue;
            }

            $newSkillDbIds[] = $skillDbId;
            $skillIndexMap[$skillDbId] = $index + 1;
        }

        // 既存 pivot
        $existingSkillDbIds = DB::table('question_skill')
            ->where('question_id', $question->id)
            ->pluck('skill_id')
            ->toArray();

        // 削除すべきスキル
        $skillIdsToRemove = array_diff($existingSkillDbIds, $newSkillDbIds);
        if (!empty($skillIdsToRemove)) {
            DB::table('question_skill')
                ->where('question_id', $question->id)
                ->whereIn('skill_id', $skillIdsToRemove)
                ->delete();
        }

        // 追加または更新
        foreach ($newSkillDbIds as $skillDbId) {
            $existingPivot = DB::table('question_skill')
                ->where('question_id', $question->id)
                ->where('skill_id', $skillDbId)
                ->first();

            if ($existingPivot) {
                DB::table('question_skill')
                    ->where('id', $existingPivot->id)
                    ->update([
                        'order'      => $skillIndexMap[$skillDbId],
                        'updated_at' => now(),
                    ]);
            } else {
                DB::table('question_skill')->insert([
                    'uuid'        => (string) Str::uuid(),
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
     * question_text, explanation, background を question_translations へ upsert
     */
    private function upsertQuestionTranslations(Question $question, array $questionJson)
    {
        $locales = ['ja','en'];

        foreach ($locales as $locale) {
            $qText = $questionJson['metadata']['question_text'][$locale] ?? null;
            $qExp  = $questionJson['metadata']['explanation'][$locale]   ?? null;
            $qBack = $questionJson['metadata']['background'][$locale]    ?? null;

            if ($qText !== null || $qExp !== null || $qBack !== null) {
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
     * JSON の learning_requirements を反映
     */
    private function applyLearningRequirements(Question $question, array $questionJson)
    {
        $items = $questionJson['learning_requirements'] ?? [];
        if (!is_array($items) || empty($items)) {
            return;
        }
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
     * JSON の level_id => levelsテーブル json_idカラム => uuid
     */
    private function findLevelUuid(?string $levelJsonId): ?string
    {
        if (!$levelJsonId) {
            return null;
        }
        return Level::where('json_id', $levelJsonId)->value('id') ?: null;
    }

    /**
     * JSON の grade_id => gradesテーブル json_idカラム => uuid
     */
    private function findGradeUuid(?string $gradeJsonId): ?string
    {
        if (!$gradeJsonId) {
            return null;
        }
        return Grade::where('json_id', $gradeJsonId)->value('id') ?: null;
    }

    /**
     * JSON の difficulty_id => difficultiesテーブル json_idカラム => uuid
     */
    private function findDifficultyUuid(?string $difficultyJsonId): ?string
    {
        if (!$difficultyJsonId) {
            return null;
        }
        return Difficulty::where('json_id', $difficultyJsonId)->value('id') ?: null;
    }

    /**
     * question_type 文字列 -> Enum (int値) に変換
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
     * status 文字列 -> QuestionStatus enum (int) に変換
     */
    private function parseQuestionStatus(string $statusString): int
    {
        try {
            return QuestionStatus::fromString($statusString)->value;
        } catch (\InvalidArgumentException) {
            return QuestionStatus::DRAFT->value;
        }
    }
}

```
```php
<?php

namespace App\Services\Utils\Question;

use App\Enums\EvaluationCheckerMethod;
use App\Enums\EvaluationMethod;
use App\Enums\FieldType;
use App\Enums\QuestionMetadataInputFormatFieldAttribute;
use App\Enums\FillInOperator;
use App\Enums\QuestionStatus;
use App\Enums\QuestionType;
use App\Models\Question\Question;
use App\Models\Question\QuestionTranslation;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Str;
use Illuminate\Validation\ValidationException;

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
     * question_components.type でサポートしている属性
     */
    public const VALID_COMPONENT_TYPES = ['text','image','movie','input_field','newline','options'];

    /**
     * question_components のうち content(多言語オブジェクト) が必須な type
     */
    private const COMPONENT_TYPES_REQUIRE_CONTENT = ['text','image','movie','options'];

    /**
     * FILL_IN_OPERATOR のときに必須となる input_components の type/attribute で使える定数
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
            $this->validateMetadata($v, $json);  // ← ここで question_type, input_format, etc. の追加検証
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
            // バリデーション失敗 → false（呼び出し元でスキップなどの対応）
            return false;
        }

        // JSON 内の必須 key チェック
        $jsonId = $questionJson['id'] ?? null;
        if (!$jsonId) {
            return false;
        }

        $metadata = $questionJson['metadata'] ?? [];
        if (empty($metadata)) {
            // metadata が空なら取り込み不可
            return false;
        }

        // Question の既存 or 新規を検索
        /** @var Question|null $question */
        $question = Question::where('json_id', $jsonId)->first();

        $jsonOrder    = $questionJson['order'] ?? 9999;
        $version      = $questionJson['version'] ?? '0.0.1';
        $rawStatus    = $questionJson['status'] ?? null;
        $statusValue  = $rawStatus
            ? $this->parseQuestionStatus($rawStatus)
            : QuestionStatus::DRAFT->value;

        // level/grade/difficulty
        $levelUuid      = $this->findLevelUuid($questionJson['level_id'] ?? null);
        $gradeUuid      = $this->findGradeUuid($questionJson['grade_id'] ?? null);
        $difficultyUuid = $this->findDifficultyUuid($questionJson['difficulty_id'] ?? null);

        // 新規 or 既存
        if (!$question) {
            // 新規
            $question = new Question();
            $question->uuid      = (string) Str::uuid();
            $question->json_id = $jsonId;

            // order 確定
            $maxOrder = Question::max('order');
            if ($maxOrder === null) {
                $maxOrder = 0;
            }
            $finalOrder = max($maxOrder + 1, $jsonOrder);

            while (Question::where('order', $finalOrder)->exists()) {
                $finalOrder++;
            }
            $question->order = $finalOrder;
        } else {
            // 既存 → order再調整
            $this->reorderQuestion($question->id, $jsonOrder);
        }

        // 基本項目セット
        $question->level_id      = $levelUuid;
        $question->grade_id      = $gradeUuid;
        $question->difficulty_id = $difficultyUuid;
        $question->version       = $version;
        $question->status        = $statusValue;

        // metadata
        $question->metadata = json_encode($metadata, JSON_UNESCAPED_UNICODE);
        $rawQuestionType    = $metadata['question_type'] ?? null;
        $questionTypeValue  = $this->parseQuestionType($rawQuestionType);
        $question->question_type = $questionTypeValue;

        // evaluation_spec
        if (isset($questionJson['evaluation_spec'])) {
            $eval = $questionJson['evaluation_spec'];
            // eval method
            $question->evaluation_method = isset($eval['evaluation_method'])
                ? EvaluationMethod::fromString($eval['evaluation_method'])
                : null;

            // checker_method
            $question->checker_method = isset($eval['checker_method'])
                ? EvaluationCheckerMethod::fromString($eval['checker_method'])
                : null;

            // llm_prompt_file_name
            $question->llm_evaluation_prompt_file_name = $eval['llm_prompt_file_name'] ?? null;

            // response_format
            $question->evaluation_response_format = isset($eval['response_format'])
                ? json_encode($eval['response_format'], JSON_UNESCAPED_UNICODE)
                : null;
        }

        // generated_by_llm
        if (array_key_exists('generated_by_llm', $questionJson)) {
            $question->generated_by_llm = (bool)$questionJson['generated_by_llm'];
        }

        // learning_requirements
        $this->applyLearningRequirements($question, $questionJson);

        // 保存
        $question->save();

        // question_translations
        $this->upsertQuestionTranslations($question, $questionJson);

        // skills
        $this->applySkillsPivot($question, $questionJson);

        return true;
    }

    /**
     * --------------------------------------------------------------------------------
     * reorderQuestion
     * --------------------------------------------------------------------------------
     * 既存 question の order を再設定（衝突回避しながら更新）
     */
    private function reorderQuestion(string $questionId, int $targetOrder)
    {
        $question = Question::find($questionId);
        if (!$question) {
            return;
        }
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
     * applySkillsPivot
     * --------------------------------------------------------------------------------
     * skills 配列を DBの question_skill ピボットに反映
     */
    private function applySkillsPivot(Question $question, array $questionJson)
    {
        $skillsFromJson = $questionJson['skills'] ?? [];
        $newSkillDbIds  = [];
        $skillIndexMap  = [];

        foreach ($skillsFromJson as $index => $sk) {
            $skillJsonId = $sk['skill_id'] ?? null;
            if (!$skillJsonId) {
                continue;
            }
            $skillDbId = DB::table('skills')
                ->where('json_id', $skillJsonId)
                ->value('id');
            if (!$skillDbId) {
                continue;
            }
            $newSkillDbIds[] = $skillDbId;
            $skillIndexMap[$skillDbId] = $index + 1;
        }

        $existingSkillDbIds = DB::table('question_skill')
            ->where('question_id', $question->id)
            ->pluck('skill_id')
            ->toArray();

        $skillIdsToRemove = array_diff($existingSkillDbIds, $newSkillDbIds);
        if (!empty($skillIdsToRemove)) {
            DB::table('question_skill')
                ->where('question_id', $question->id)
                ->whereIn('skill_id', $skillIdsToRemove)
                ->delete();
        }

        foreach ($newSkillDbIds as $skillDbId) {
            $pivotRow = DB::table('question_skill')
                ->where('question_id', $question->id)
                ->where('skill_id', $skillDbId)
                ->first();
            if ($pivotRow) {
                DB::table('question_skill')
                    ->where('id', $pivotRow->id)
                    ->update([
                        'order'      => $skillIndexMap[$skillDbId],
                        'updated_at' => now(),
                    ]);
            } else {
                DB::table('question_skill')->insert([
                    'uuid'          => (string) Str::uuid(),
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
     * upsertQuestionTranslations
     * --------------------------------------------------------------------------------
     * question_translations に多言語のquestion_text等を格納
     */
    private function upsertQuestionTranslations(Question $question, array $questionJson)
    {
        $locales = self::LANGUAGES;
        $meta = $questionJson['metadata'] ?? [];

        foreach ($locales as $locale) {
            $qText = $meta['question_text'][$locale] ?? null;
            $qExp  = $meta['explanation'][$locale]   ?? null;
            $qBack = $meta['background'][$locale]    ?? null;

            // いずれか存在すれば upsert
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
     * applyLearningRequirements
     * --------------------------------------------------------------------------------
     * learning_requirements (配列) を Question に格納 (改行区切り)
     */
    private function applyLearningRequirements(Question $question, array $questionJson)
    {
        $items = $questionJson['learning_requirements'] ?? [];
        if (!is_array($items) || empty($items)) {
            return;
        }
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

        // 改行結合
        $question->learning_subject             = implode("\n", $subjects);
        $question->learning_no                  = !empty($nos) ? (int)$nos[0] : null;
        $question->learning_requirement         = implode("\n", $requirements);
        $question->learning_required_competency = implode("\n", $requiredCompetencies);
        $question->learning_background          = implode("\n", $backgrounds);
        $question->learning_category            = implode("\n", $categories);
        $question->learning_grade_level         = implode("\n", $gradeLevels);
        $question->learning_url                 = implode("\n", $urls);
    }

    //==================================================
    // バリデーション関連
    //==================================================

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

            // skills
            'skills'        => ['required','array'],

            // learning_requirements
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

            // question_text / explanation / background / question => 多言語
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

            // input_format
            'metadata.input_format'             => ['required','array'],
            'metadata.input_format.fields'      => ['required','array'],
            'metadata.input_format.question_components' => ['required','array'],
        ];
    }

    /**
     * バリデーションエラーメッセージ（日本語）
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
     * skills 配列の内容を DBの skills テーブルと照合
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
                continue;
            }
            $row = DB::table('skills')->where('json_id', $sid)->first();
            if (!$row) {
                $validator->errors()->add("skills.{$idx}.skill_id",
                    "skill_id='{$sid}' はDBに存在しません。"
                );
                continue;
            }
            if ($row->display_name !== $sname) {
                $validator->errors()->add("skills.{$idx}.name",
                    "skill_id='{$sid}' の display_name と name='{$sname}' が一致しません。"
                );
            }
        }
    }

    /**
     * evaluation_spec の詳細チェック
     * - evaluation_method (CODE/LLM)
     * - checker_method (CODE時必須)
     * - llm_prompt_file_name (LLM時必須)
     * - response_format, fields 等
     *
     * - metadata.question_type が CLASSIFY_THE_OPTIONS の場合、
     *   evaluation_method=CODE のときは checker_method を CHECK_BY_UNORDERED_ARRAY_ELEMENTS_MATCH に限定する
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

        // ここで question_type を取得し、後でチェックに利用
        $qtRaw = $json['metadata']['question_type'] ?? null;
        $questionTypeValue = null;
        try {
            $questionTypeValue = QuestionType::fromString((string)$qtRaw)->value;
        } catch (\InvalidArgumentException) {
            // 既に metadata.question_type でエラーになるはずなので ここでは何もしない
        }

        // CODE
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
                return;
            }
            // CODE時でもfieldsがあれば検証するが必須ではない想定
            if (isset($eval['response_format']['fields']) && is_array($eval['response_format']['fields'])) {
                $this->validateResponseFormatFields($validator, $eval['response_format']['fields'], $json);
            }

            // CLASSIFY_THE_OPTIONS の場合、checker_method を限定
            if ($questionTypeValue === QuestionType::CLASSIFY_THE_OPTIONS->value) {
                $actualChecker = $eval['checker_method'] ?? null;
                if ($actualChecker !== EvaluationCheckerMethod::CHECK_BY_UNORDERED_ARRAY_ELEMENTS_MATCH->label()) {
                    $validator->errors()->add(
                        'evaluation_spec.checker_method',
                        "CLASSIFY_THE_OPTIONS の場合、evaluation_method=CODE では checker_method='".EvaluationCheckerMethod::CHECK_BY_UNORDERED_ARRAY_ELEMENTS_MATCH->label()."' が必須です。"
                    );
                }
            }

            // [ADD for ORDER_THE_OPTIONS start]
            // ORDER_THE_OPTIONS の場合、checker_method を CHECK_BY_ARRAY_ELEMENTS_MATCH_IN_ORDER に限定
            if ($questionTypeValue === QuestionType::ORDER_THE_OPTIONS->value) {
                $actualChecker = $eval['checker_method'] ?? null;
                if ($actualChecker !== EvaluationCheckerMethod::CHECK_BY_ARRAY_ELEMENTS_MATCH_IN_ORDER->label()) {
                    $validator->errors()->add(
                        'evaluation_spec.checker_method',
                        "ORDER_THE_OPTIONS の場合、evaluation_method=CODE では checker_method='".EvaluationCheckerMethod::CHECK_BY_ARRAY_ELEMENTS_MATCH_IN_ORDER->label()."' が必須です。"
                    );
                }
            }
            // [ADD for ORDER_THE_OPTIONS end]
        }
        // LLM
        else {
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
            // response_format 必須
            if (!isset($eval['response_format']) || !is_array($eval['response_format'])) {
                $validator->errors()->add('evaluation_spec.response_format',
                    "evaluation_method=LLM のため response_format が必須です。"
                );
                return;
            }
            // LLM時 => fields 配列必須
            if (!isset($eval['response_format']['fields']) || !is_array($eval['response_format']['fields'])) {
                $validator->errors()->add('evaluation_spec.response_format.fields',
                    "evaluation_method=LLM のため fields(配列) が必須です。"
                );
                return;
            }
            $this->validateResponseFormatFields($validator, $eval['response_format']['fields'], $json);
        }
    }

    /**
     * 新規追加メソッド
     * evaluation_spec.response_format.fields.* の collect_answer を
     * question_type に合わせてバリデーションする
     */
    private function validateResponseFormatFields($validator, array $fieldsArr, array $json)
    {
        // question_type を参照する
        $meta       = $json['metadata'] ?? [];
        $rawQt      = $meta['question_type'] ?? '';
        $questionTp = null;
        try {
            $questionTp = QuestionType::fromString((string)$rawQt)->value;
        } catch (\InvalidArgumentException) {
            // 不明なら何もしない
        }

        foreach ($fieldsArr as $i => $field) {
            // field_id
            if (empty($field['field_id']) || !is_string($field['field_id'])) {
                $validator->errors()->add("evaluation_spec.response_format.fields.{$i}.field_id",
                    "field_id は必須の文字列です。"
                );
            }
            // user_answer
            if (empty($field['user_answer']) || !is_string($field['user_answer'])) {
                $validator->errors()->add("evaluation_spec.response_format.fields.{$i}.user_answer",
                    "user_answer は必須の文字列です。"
                );
            } else {
                // CLASSIFY_THE_OPTIONS の場合は 'sequence' を必須
                if ($questionTp === QuestionType::CLASSIFY_THE_OPTIONS->value) {
                    if ($field['user_answer'] !== FieldType::SEQUENCE->value) {
                        $validator->errors()->add(
                            "evaluation_spec.response_format.fields.{$i}.user_answer",
                            "CLASSIFY_THE_OPTIONS の場合 user_answer は 'sequence' である必要があります。"
                        );
                    }
                }
                // [ADD for ORDER_THE_OPTIONS start]
                // ORDER_THE_OPTIONS の場合は 'sequence' を必須
                if ($questionTp === QuestionType::ORDER_THE_OPTIONS->value) {
                    if ($field['user_answer'] !== FieldType::SEQUENCE->value) {
                        $validator->errors()->add(
                            "evaluation_spec.response_format.fields.{$i}.user_answer",
                            "ORDER_THE_OPTIONS の場合 user_answer は 'sequence' である必要があります。"
                        );
                    }
                }
                // [ADD for ORDER_THE_OPTIONS end]
            }

            // is_correct
            if (!isset($field['is_correct']) || $field['is_correct'] !== 'boolean') {
                $validator->errors()->add("evaluation_spec.response_format.fields.{$i}.is_correct",
                    "is_correct は 'boolean' の文字列である必要があります。"
                );
            }
            // collect_answer => 多言語オブジェクト
            if (!isset($field['collect_answer']) || !is_array($field['collect_answer'])) {
                $validator->errors()->add("evaluation_spec.response_format.fields.{$i}.collect_answer",
                    "collect_answer は多言語オブジェクトが必須です。"
                );
            } else {
                // LANGUAGES に対応しているか
                foreach (self::LANGUAGES as $lang) {
                    if (!array_key_exists($lang, $field['collect_answer'])) {
                        $validator->errors()->add(
                            "evaluation_spec.response_format.fields.{$i}.collect_answer",
                            "collect_answer に言語キー'{$lang}'がありません。"
                        );
                    } else {
                        // question_type によって値の型チェック
                        $val = $field['collect_answer'][$lang];
                        if ($questionTp === QuestionType::FILL_IN_THE_BLANK->value) {
                            // "number" の場合は数値（例示）
                            if (!is_numeric($val)) {
                                $validator->errors()->add(
                                    "evaluation_spec.response_format.fields.{$i}.collect_answer.{$lang}",
                                    "FILL_IN_THE_BLANK の場合 collect_answer.{$lang} は数値である必要があります。"
                                );
                            }
                        }
                        elseif ($questionTp === QuestionType::FILL_IN_OPERATOR->value) {
                            // FillInOperator enumに存在するか
                            try {
                                FillInOperator::from($val); // '>', '<', '=' 等
                            } catch (\ValueError $ve) {
                                $validator->errors()->add(
                                    "evaluation_spec.response_format.fields.{$i}.collect_answer.{$lang}",
                                    "FILL_IN_OPERATOR の場合 collect_answer.{$lang} は演算子（>,<,=）のいずれかが必要です。"
                                );
                            }
                        }
                        // ORDER_THE_OPTIONS や CLASSIFY_THE_OPTIONS の場合は、ここでは特に型指定なし
                        //（実際の正解値は配列などの形式を想定しうるが、ここでは多言語オブジェクトで文字列保存）
                    }
                }
            }

            // field_explanation => 多言語オブジェクト
            if (!isset($field['field_explanation']) || !is_array($field['field_explanation'])) {
                $validator->errors()->add("evaluation_spec.response_format.fields.{$i}.field_explanation",
                    "field_explanation は多言語オブジェクトが必須です。"
                );
            } else {
                foreach (self::LANGUAGES as $lang) {
                    $txt = $field['field_explanation'][$lang] ?? null;
                    if (!is_string($txt) || trim($txt) === '') {
                        $validator->errors()->add(
                            "evaluation_spec.response_format.fields.{$i}.field_explanation.{$lang}",
                            "field_explanation.{$lang} は空文字不可の文字列が必須です。"
                        );
                    }
                }
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
        try {
            $questionTypeValue = QuestionType::fromString((string)$qtRaw)->value;
        } catch (\InvalidArgumentException) {
            $validator->errors()->add('metadata.question_type',
                "question_type='{$qtRaw}' は未定義です。"
            );
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

                // user_answer => enumチェック
                if (!isset($f['user_answer']) || !is_string($f['user_answer'])) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.user_answer",
                        "user_answer は必須の文字列です。"
                    );
                } else {
                    try {
                        FieldType::from($f['user_answer']);
                    } catch (\ValueError $ve) {
                        // Enumに無い文字列が渡された場合
                        $validator->errors()->add("metadata.input_format.fields.{$idx}.user_answer",
                            "user_answer='{$f['user_answer']}' は 'number','string','operator','sequence' のいずれかのみ有効です。"
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

        // --------------------------------------------------------------------------------
        // ここから FILL_IN_OPERATOR / CLASSIFY_THE_OPTIONS 向けバリデーション
        // --------------------------------------------------------------------------------
        // FILL_IN_OPERATOR なら input_components 必須
        if ($questionTypeValue === QuestionType::FILL_IN_OPERATOR->value) {
            $icKey = 'metadata.input_format.input_components';
            if (!isset($meta['input_format']['input_components'])) {
                $validator->errors()->add($icKey,
                    "FILL_IN_OPERATOR のため input_components が必須です。"
                );
                return;
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
                            // FillInOperator の値(> < =)になっているか
                            $val = $ic['content'][$lang];
                            try {
                                FillInOperator::from($val);
                            } catch (\ValueError) {
                                $validator->errors()->add("{$icKey}.{$idx}.content.{$lang}",
                                    "不正な演算子 '{$val}' です。(>,<,= のいずれか)"
                                );
                            }
                        }
                    }
                }
                // order
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

        // --------------------------------------------------------------------------------
        // CLASSIFY_THE_OPTIONS 向けのバリデーション
        // --------------------------------------------------------------------------------
        if ($questionTypeValue === QuestionType::CLASSIFY_THE_OPTIONS->value) {
            // 1) question がカンマ区切りか（言語ごとにチェック）
            foreach (self::LANGUAGES as $lang) {
                $qStr = $meta['question'][$lang] ?? '';
                if (!str_contains($qStr, ',')) {
                    $validator->errors()->add("metadata.question.{$lang}",
                        "CLASSIFY_THE_OPTIONS の場合、question.{$lang} はカンマ区切りの文字列が必須です。"
                    );
                }
            }

            // 2) input_format.input_components が必須
            $icKey = 'metadata.input_format.input_components';
            if (!isset($meta['input_format']['input_components']) || !is_array($meta['input_format']['input_components'])) {
                $validator->errors()->add($icKey,
                    "CLASSIFY_THE_OPTIONS のため input_components (配列) が必須です。"
                );
            } else {
                foreach (self::LANGUAGES as $lang) {
                    $qStr = $meta['question'][$lang] ?? '';
                    $arrQ = array_map('trim', explode(',', $qStr));
                    $arrQ = array_filter($arrQ, fn($v) => $v !== '');
                    $countQ = count($arrQ);

                    $icComps = $meta['input_format']['input_components'];
                    $icContents = [];
                    foreach ($icComps as $ic) {
                        $icContent = $ic['content'][$lang] ?? null;
                        if ($icContent !== null) {
                            $icContents[] = trim($icContent);
                        }
                    }
                    if ($countQ !== count($icContents)) {
                        $validator->errors()->add(
                            $icKey,
                            "CLASSIFY_THE_OPTIONS の場合、question.{$lang} と同じ数の input_components が必要です。"
                        );
                    } else {
                        foreach ($arrQ as $token) {
                            if (!in_array($token, $icContents, true)) {
                                $validator->errors()->add(
                                    $icKey,
                                    "CLASSIFY_THE_OPTIONS の場合、question.{$lang} の要素 '{$token}' が input_components にありません。"
                                );
                            }
                        }
                    }
                }
            }

            // 3) metadata.input_format.fields[*].user_answer と attribute
            if (is_array($fieldsArr)) {
                foreach ($fieldsArr as $idx => $f) {
                    if (($f['user_answer'] ?? '') !== FieldType::SEQUENCE->value) {
                        $validator->errors()->add("metadata.input_format.fields.{$idx}.user_answer",
                            "CLASSIFY_THE_OPTIONS の場合、user_answer は 'sequence' である必要があります。"
                        );
                    }
                    if (($f['attribute'] ?? '') !== QuestionMetadataInputFormatFieldAttribute::ARRAY->value) {
                        $validator->errors()->add("metadata.input_format.fields.{$idx}.attribute",
                            "CLASSIFY_THE_OPTIONS の場合、attribute は 'array' である必要があります。"
                        );
                    }
                }
            }
        }

        // [ADD for ORDER_THE_OPTIONS start]
        // --------------------------------------------------------------------------------
        // ORDER_THE_OPTIONS 向けのバリデーション
        // --------------------------------------------------------------------------------
        if ($questionTypeValue === QuestionType::ORDER_THE_OPTIONS->value) {
            // 1) question がカンマ区切りか（言語ごとにチェック）
            foreach (self::LANGUAGES as $lang) {
                $qStr = $meta['question'][$lang] ?? '';
                if (!str_contains($qStr, ',')) {
                    $validator->errors()->add("metadata.question.{$lang}",
                        "ORDER_THE_OPTIONS の場合、question.{$lang} はカンマ区切りの文字列が必須です。"
                    );
                }
            }

            // 2) input_format.input_components が必須
            $icKey = 'metadata.input_format.input_components';
            if (!isset($meta['input_format']['input_components']) || !is_array($meta['input_format']['input_components'])) {
                $validator->errors()->add($icKey,
                    "ORDER_THE_OPTIONS のため input_components (配列) が必須です。"
                );
            } else {
                // question内のカンマ区切り要素がすべて input_components に含まれているか、数が一致するか
                foreach (self::LANGUAGES as $lang) {
                    $qStr = $meta['question'][$lang] ?? '';
                    $arrQ = array_map('trim', explode(',', $qStr));
                    $arrQ = array_filter($arrQ, fn($v) => $v !== '');
                    $countQ = count($arrQ);

                    $icComps = $meta['input_format']['input_components'];
                    $icContents = [];
                    foreach ($icComps as $ic) {
                        $icContent = $ic['content'][$lang] ?? null;
                        if ($icContent !== null) {
                            $icContents[] = trim($icContent);
                        }
                    }
                    if ($countQ !== count($icContents)) {
                        $validator->errors()->add(
                            $icKey,
                            "ORDER_THE_OPTIONS の場合、question.{$lang} と同じ数の input_components が必要です。"
                        );
                    } else {
                        foreach ($arrQ as $token) {
                            if (!in_array($token, $icContents, true)) {
                                $validator->errors()->add(
                                    $icKey,
                                    "ORDER_THE_OPTIONS の場合、question.{$lang} の要素 '{$token}' が input_components にありません。"
                                );
                            }
                        }
                    }
                }
            }

            // 3) metadata.input_format.fields[*].user_answer と attribute
            if (is_array($fieldsArr)) {
                foreach ($fieldsArr as $idx => $f) {
                    if (($f['user_answer'] ?? '') !== FieldType::SEQUENCE->value) {
                        $validator->errors()->add("metadata.input_format.fields.{$idx}.user_answer",
                            "ORDER_THE_OPTIONS の場合、user_answer は 'sequence' である必要があります。"
                        );
                    }
                    if (($f['attribute'] ?? '') !== QuestionMetadataInputFormatFieldAttribute::ARRAY->value) {
                        $validator->errors()->add("metadata.input_format.fields.{$idx}.attribute",
                            "ORDER_THE_OPTIONS の場合、attribute は 'array' である必要があります。"
                        );
                    }
                }
            }
        }
        // [ADD for ORDER_THE_OPTIONS end]
    }

    /**
     * 文字列→QuestionType enum の int値に変換（未定義なら CALCULATION）
     */
    private function parseQuestionType(?string $typeString): int
    {
        if (!$typeString) {
            return QuestionType::CALCULATION->value;
        }
        try {
            return QuestionType::fromString($typeString)->value;
        } catch (\InvalidArgumentException) {
            return QuestionType::CALCULATION->value;
        }
    }

    /**
     * 文字列→QuestionStatus enum の int値に変換（未定義なら DRAFT）
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
     * level_id → levelsテーブル(json_id)検索 → UUID
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
     * grade_id → gradesテーブル(json_id)検索 → UUID
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
     * difficulty_id → difficultiesテーブル(json_id)検索 → UUID
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
     * ユーザー回答のバリデーション
     * --------------------------------------------------------------------------------
     *   - attribute='text' の場合に文字列チェックを追加
     *   - さらに、user_answer が metadata.input_format.input_components 内で
     *     type='text' の content に含まれているか確認（多言語含むすべての文字列を対象）
     */
    public function validateUserAnswer(array $answerData, array $metadata): void
    {
        // metadata は localizeMetadata() で、ローカライズされている前提
        // question_type は数値になっている
        $qt = $metadata['question_type'] ?? null;

        $inFmt = $metadata['input_format'] ?? [];
        if (!is_array($inFmt)) {
            throw ValidationException::withMessages([
                'metadata.input_format' => [__('errors.api.answer.mismatch_answer_format')]
            ]);
        }

        $metaFields = $inFmt['fields'] ?? [];
        if (!is_array($metaFields)) {
            throw ValidationException::withMessages([
                'metadata.input_format.fields' => [__('errors.api.answer.mismatch_answer_format')]
            ]);
        }

        $answerFields = $answerData['fields'] ?? null;
        if (!is_array($answerFields)) {
            throw ValidationException::withMessages([
                'answerData.fields' => [__('errors.api.answer.mismatch_answer_format')]
            ]);
        }

        // FILL_IN_THE_BLANK / FILL_IN_OPERATOR のみ
        // MetaData の数と回答数があっている。
        if ($qt == QuestionType::FILL_IN_THE_BLANK->value
            || $qt == QuestionType::FILL_IN_OPERATOR->value) {
            if (count($metaFields) !== count($answerFields)) {
                throw ValidationException::withMessages([
                    'fields.count' => [__('errors.api.answer.mismatch_answer_format')]
                ]);
            }
        }

        // CLASSIFY_THE_OPTIONS 対応
        if ($qt == QuestionType::CLASSIFY_THE_OPTIONS->value) {
            if (count($metaFields) !== count($answerFields)) {
                throw ValidationException::withMessages([
                    'fields.count' => ["CLASSIFY_THE_OPTIONS の場合、ユーザーの fields数が問題定義と一致しません。"]
                ]);
            }
        }

        // ORDER_THE_OPTIONS 対応
        if ($qt == QuestionType::ORDER_THE_OPTIONS->value) {
            if (count($metaFields) !== count($answerFields)) {
                throw ValidationException::withMessages([
                    'fields.count' => ["ORDER_THE_OPTIONS の場合、ユーザーの fields数が問題定義と一致しません。"]
                ]);
            }
        }

        $metaFieldMap = [];
        foreach ($metaFields as $mf) {
            if (!empty($mf['field_id'])) {
                $metaFieldMap[$mf['field_id']] = $mf;
            }
        }

        // text属性を判定するため、input_componentsの type='text' の文字列リストをあらかじめ集約
        $validTextAnswers = [];
        if (!empty($inFmt['input_components']) && is_array($inFmt['input_components'])) {
            foreach ($inFmt['input_components'] as $ic) {
                if (($ic['type'] ?? '') === 'text') {
                    // contentが多言語であればすべて取り出す
                    if (!empty($ic['content']) && is_array($ic['content'])) {
                        foreach ($ic['content'] as $langText) {
                            // 重複防止のため trim後にまとめる
                            $validTextAnswers[] = trim($langText);
                        }
                    }
                }
            }
        }

        foreach ($answerFields as $idx => $af) {
            $fid = $af['field_id'] ?? '';
            if (!$fid) {
                throw ValidationException::withMessages([
                    "fields.{$idx}.field_id" => [__('errors.api.answer.mismatch_answer_format')]
                ]);
            }

            $attr = $af['attribute'] ?? null;
            $uAns = $af['user_answer'] ?? null;

            if (!$attr) {
                throw ValidationException::withMessages([
                    "fields.{$idx}.attribute" => [__('errors.api.answer.mismatch_answer_format')]
                ]);
            }

            // FILL_IN_THE_BLANK の場合にmetadataに存在しない field_id はNG
            if ($qt == QuestionType::FILL_IN_THE_BLANK->value && !isset($metaFieldMap[$fid])) {
                throw ValidationException::withMessages([
                    "fields.{$idx}.field_id" => [__('errors.api.answer.mismatch_answer_format')]
                ]);
            }

            // ----------------------------------------------------
            // 既存チェック: number / operator / array
            // ----------------------------------------------------
            if ($attr === FieldType::NUMBER->value) {
                if (!is_numeric($uAns)) {
                    throw ValidationException::withMessages([
                        "fields.{$idx}.user_answer" => [__('errors.api.answer.mismatch_answer_format_to_number')]
                    ]);
                }
            }
            else if ($attr === QuestionMetadataInputFormatFieldAttribute::OPERATOR->value) {
                try {
                    FillInOperator::from($uAns);
                } catch (\ValueError) {
                    throw ValidationException::withMessages([
                        "fields.{$idx}.user_answer" => [__('errors.api.answer.mismatch_answer_format_operator')],
                    ]);
                }
            }
            else if ($qt == QuestionType::CLASSIFY_THE_OPTIONS->value
                && $attr === QuestionMetadataInputFormatFieldAttribute::ARRAY->value) {
                // user_answer が JSON配列文字列かどうかを簡易チェック
                $decoded = @json_decode($uAns, true);
                if (!is_array($decoded)) {
                    throw ValidationException::withMessages([
                        "fields.{$idx}.user_answer" => [__('errors.api.answer.mismatch_answer_format_classify_the_options')],
                    ]);
                }
            }
            else if ($qt == QuestionType::ORDER_THE_OPTIONS->value
                && $attr === QuestionMetadataInputFormatFieldAttribute::ARRAY->value) {
                // user_answer が JSON配列文字列かどうかを簡易チェック
                $decoded = @json_decode($uAns, true);
                if (!is_array($decoded)) {
                    throw ValidationException::withMessages([
                        "fields.{$idx}.user_answer" => [__('errors.api.answer.mismatch_answer_format_order_the_options')],
                    ]);
                }
            }
            // ----------------------------------------------------
            // 追加した text 属性のチェック
            // ----------------------------------------------------
            else if ($attr === 'text') {
                // 1) ユーザー回答が文字列かどうか
                if (!is_string($uAns)) {
                    throw ValidationException::withMessages([
                        "fields.{$idx}.user_answer" => ["attribute='text' の場合、文字列を入力してください。"]
                    ]);
                }
                // 2) その文字列が、input_components で定義された文字列のいずれかに含まれているか
                //    （「あまり」などが input_components の content に定義されている必要がある）
                if (!in_array(trim($uAns), $validTextAnswers, true)) {
                    throw ValidationException::withMessages([
                        "fields.{$idx}.user_answer" => [
                            "attribute='text' の場合、定義されていない文字列です。valid=[".implode(', ', $validTextAnswers)."]"
                        ]
                    ]);
                }
            }
            else {
                // その他は対応外としてエラー
                throw ValidationException::withMessages([
                    "fields.{$idx}.attribute" => [__('errors.api.answer.mismatch_answer_format')]
                ]);
            }
        }
    }

    /**
     * --------------------------------------------------------------------------------
     * localizeMetadata
     * --------------------------------------------------------------------------------
     * metadata を現在のアプリケーションロケールに合わせて整形（サンプル）
     */
    public function localizeMetadata(array $metadata): array
    {
        $locale = \App::getLocale();
        if (isset($metadata['question_text']) && is_array($metadata['question_text'])) {
            $metadata['question_text'] = $metadata['question_text'][$locale] ?? '';
        }
        if (isset($metadata['explanation']) && is_array($metadata['explanation'])) {
            $metadata['explanation']   = $metadata['explanation'][$locale] ?? '';
        }
        // background を表示しない
        if (isset($metadata['background']) && is_array($metadata['background'])) {
            unset($metadata['background']);
        }
        // metadata の　question_type は表示しない
        if (isset($metadata['question_type'])) {
            try {
                $metadata['question_type'] = QuestionType::fromString($metadata['question_type'])->value;
            } catch (\Exception) {
                $metadata['question_type'] = null;
            }
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
     * --------------------------------------------------------------------------------
     * localizeLlmResult
     * --------------------------------------------------------------------------------
     * LLM応答を現在のロケールに合わせて整形
     */
    public function localizeLlmResult(array $llmResult): array
    {
        $locale = \App::getLocale();

        $mapped = [];
        $rawIsCorrect = data_get($llmResult, 'is_correct', false);
        $mapped['is_correct'] = ($rawIsCorrect === true || $rawIsCorrect === "true") ? "true" : "false";

        $mapped['score'] = data_get($llmResult, 'score', 0);

        $qtArr = data_get($llmResult, 'question_text', []);
        $mapped['question_text'] = is_array($qtArr)
            ? data_get($qtArr, $locale, '')
            : '';

        $expArr = data_get($llmResult, 'explanation', []);
        $mapped['explanation'] = is_array($expArr)
            ? data_get($expArr, $locale, '')
            : '';

        $qArr = data_get($llmResult, 'question', []);
        $mapped['question'] = is_array($qArr)
            ? data_get($qArr, $locale, '')
            : '';

        $mapped['fields'] = [];
        $fieldsArr = data_get($llmResult, 'fields', []);
        if (is_array($fieldsArr)) {
            foreach ($fieldsArr as $f) {
                $mf = [];
                $mf['field_id']     = (string) data_get($f, 'field_id', '');
                $mf['user_answer']  = (string) data_get($f, 'user_answer', '');
                $rCorrect           = data_get($f, 'is_correct', false);
                $mf['is_correct']   = ($rCorrect === true || $rCorrect === "true") ? "true" : "false";
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
