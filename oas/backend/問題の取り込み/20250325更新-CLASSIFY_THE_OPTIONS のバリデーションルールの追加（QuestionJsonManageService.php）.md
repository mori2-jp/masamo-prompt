以下、QuestionJsonManageService　の　validateQuestionJson　に
CLASSIFY_THE_OPTIONS　の場合のバリデーションルールを追加してほしい
修正箇所を知らせてください。
 
# 追加するルール
metadata.question_type が、CLASSIFY_THE_OPTIONS　の時のルールを追加
- metadata.input_format.input_components: metadata.question_type が、CLASSIFY_THE_OPTIONS　の時は必須。（回答の選択肢。入力キーパッドのボタン）CLASSIFY_THE_OPTIONS　の時は、question を カンマ区切りで分割した値が全て含まれていること。例：question が、14, 15, 18, 20, 21, 25, 27, 35, 45, 50 だった場合は、14〜50までのそれぞれの分割した input_components が 同じ数（ここでは10個）存在していること。

metadata.question_type が、CLASSIFY_THE_OPTIONS　の時の、question のルールを追加
必須、値は選択肢を示すので値がカンマ区切りになっていること

metadata.question_type が、CLASSIFY_THE_OPTIONS　の時の、evaluation_spec.response_format.fields.user_answer のルールを追加
必須、FieldType::SEQUENCE　と値が一致すること

metadata.question_type が、CLASSIFY_THE_OPTIONS　の時の、metadata.input_format.fields.user_answer のルールを追加
必須、FieldType::SEQUENCE　と値が一致すること

metadata.question_type が、CLASSIFY_THE_OPTIONS　の時の、metadata.input_format.fields.attribute のルールを追加
QuestionMetadataInputFormatFieldAttribute::ARRAY と値が一致（array になっていること。

metadata.question_type が、CLASSIFY_THE_OPTIONS　の時の、evaluation_spec.evaluation_method のルールを追加
CODEが選択されていて　evaluation_spec.evaluation_method　が必要となれる場合は、　CLASSIFY_THE_OPTIONS　の時は、CHECK_BY_UNORDERED_ARRAY_ELEMENTS_MATCH　に限定

※注意点
・現在は question_type FILL_IN_THE_BLANK の仕様だけですが、将来的に複数の種類が追加されて行く予定です。１００個以上になるかも
バリデーションのルールは、question_typeによっては重複するものも出てくるので、
[FILL_IN_THE_BLANK, SCENARIO]など、バリデーションルールに対応するquestion Type を簡単追加したり取り消したり出来るような実装がよい

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
evaluation_spec.response_format.fields.user_answer：必須、FieldType の定数に値があるか。ユーザーの回答。CLASSIFY_THE_OPTIONS　の時は、sequence　と値が一致すること。
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
metadata.input_format.fields.attribute: 必須、（QuestionMetadataInputFormatFieldAttribute の定数に値が存在しているか）。ブランクのフォームの属性。例えば number であれば、<input type="number">になる
metadata.input_format.fields.user_answer: 必須、次の条件を満たした文字列型かどうかをチェックする。FieldType の定数に値があるか。ユーザーが入力する正答の型。CLASSIFY_THE_OPTIONS　の時は、FieldType::SEQUENCE　と値が一致すること。
metadata.input_format.fields.collect_answer: 絶対に存在してはいけない。ユーザーに回答が見えてしまうため。

metadata.input_format.question_components: 必須（問題を更生する要素）
metadata.input_format.question_components.type：必須、VALID_COMPONENT_TYPES 定数と値が合っているか
metadata.input_format.question_components.content：question_components.type　が、COMPONENT_TYPES_REQUIRE_CONTENT に含まれる場合に必須、オブジェクト、言語定数と一致する値が全て含まれているか
metadata.input_format.question_components.order: 必須、数値、重複する値が存在しないこと。問題を構築するときの表示順番

metadata.input_format.input_components: metadata.question_type が、FILL_IN_OPERATOR または、CLASSIFY_THE_OPTIONS　の時は必須。（回答の選択肢。入力キーパッドのボタン）CLASSIFY_THE_OPTIONS　の時は、question を カンマ区切りで分割した値が全て含まれていること。例：question が、14, 15, 18, 20, 21, 25, 27, 35, 45, 50 だった場合は、14〜50までのそれぞれの分割した input_components が 同じ数（ここでは10個）存在していること。
metadata.input_format.input_components.type：必須、VALID_INPUT_COMPONENTS_TYPES 定数と値が合っているか
metadata.input_format.input_components.content：必須、オブジェクト、言語定数と一致するキーが全て含まれているか。FillInOperator に存在する値になっているか。（ > < など）
metadata.input_format.input_components.order: 必須、数値、重複する値が存在しないこと。入力キーパッドを構築するときの表示順番


--- 問題JSONの全体構造
```json
{
  "order": 100,
  "id": "ques_s1_g3_sec300_u100_diff100_qt251_v100_100",
  "level_id": "lev_003",
  "grade_id": "gra_003",
  "difficulty_id": "diff_100",
  "version": "1.0.0",
  "status": "PUBLISHED",
  "generated_by_llm": false,
  "created_at": "2025-03-25 10:00:00",
  "updated_at": "2025-03-25 10:00:00",
  "skills": [
    {
      "skill_id": "sk_004",
      "name": "知識・技能"
    }
  ],
  "learning_requirements": [
    {
      "learning_subject": "算数",
      "learning_no": 40,
      "learning_requirement": "計算の意味・方法 割り算 1位数などの除法",
      "learning_required_competency": "1桁わり算(27÷3等)を九九を用いてスムーズに解ける。",
      "learning_background": "2桁以上の除法に進む基礎固め。逆算としてかけ算との結び付きも確認。",
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
        "ja": "次の数を、2, 3, 5, 7, 9で割り切れる数ごとに分けましょう。",
        "en": "Classify the following numbers based on which are divisible by 2, 3, 5, 7, or 9."
      },
      "explanation": {
        "ja": "この問題では、提示された2桁の数が2,3,5,7,9で割り切れるかを調べて分類します。割り算の九九を使い、あまりが出ないかどうかを確認して回答しましょう。",
        "en": "In this problem, you classify each two-digit number based on whether it is divisible by 2, 3, 5, 7, or 9. Use the multiplication table for quick checking and ensure there's no remainder."
      },
      "question": {
        "ja": "14, 15, 18, 20, 21, 25, 27, 35, 45, 50",
        "en": "14, 15, 18, 20, 21, 25, 27, 35, 45, 50"
      },
      "fields": [
        {
          "field_id": "f_1",
          "user_answer": "sequence",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": "[14, 18, 20, 50]",
            "en": "[14, 18, 20, 50]"
          },
          "field_explanation": {
            "ja": "14, 18, 20, 50は2で割り切れます(偶数)。",
            "en": "14, 18, 20, and 50 are multiples of 2 (even numbers)."
          }
        },
        {
          "field_id": "f_2",
          "user_answer": "sequence",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": "[15, 18, 21, 27, 45]",
            "en": "[15, 18, 21, 27, 45]"
          },
          "field_explanation": {
            "ja": "15, 18, 21, 27, 45は3で割り切れます。",
            "en": "15, 18, 21, 27, and 45 are multiples of 3."
          }
        },
        {
          "field_id": "f_3",
          "user_answer": "sequence",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": "[15, 20, 25, 35, 45, 50]",
            "en": "[15, 20, 25, 35, 45, 50]"
          },
          "field_explanation": {
            "ja": "15, 20, 25, 35, 45, 50は5で割り切れます(下1桁が0または5)。",
            "en": "15, 20, 25, 35, 45, and 50 are multiples of 5 (ending in 0 or 5)."
          }
        },
        {
          "field_id": "f_4",
          "user_answer": "sequence",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": "[14, 21, 35]",
            "en": "[14, 21, 35]"
          },
          "field_explanation": {
            "ja": "14, 21, 35は7で割り切れます。",
            "en": "14, 21, and 35 are multiples of 7."
          }
        },
        {
          "field_id": "f_5",
          "user_answer": "sequence",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": "[18, 27, 45]",
            "en": "[18, 27, 45]"
          },
          "field_explanation": {
            "ja": "18, 27, 45は9で割り切れます。",
            "en": "18, 27, and 45 are multiples of 9."
          }
        }
      ]
    }
  },
  "metadata": {
    "question_type": "CLASSIFY_THE_OPTIONS",
    "question": {
      "ja": "14, 15, 18, 20, 21, 25, 27, 35, 45, 50",
      "en": "14, 15, 18, 20, 21, 25, 27, 35, 45, 50"
    },
    "question_text": {
      "ja": "つぎの数字を2, 3, 5, 7, 9で割り切れるように分けましょう",
      "en": "Classify these two-digit numbers according to divisors 2, 3, 5, 7, and 9"
    },
    "explanation": {
      "ja": "この問題は、2桁の整数を特定の1桁の数(2,3,5,7,9)で割り切れるかどうかを考え、分類する力を養うことが目的です。わり算のきまりを活用して数を整理してみましょう。",
      "en": "The goal is to practice classifying two-digit numbers by whether they can be divided exactly by 2, 3, 5, 7, or 9. Use division rules and your multiplication facts to sort the numbers."
    },
    "background": {
      "ja": "この問題では割り算の基本的な考え方を確認するとともに、2桁の整数がどのように分類できるかを理解することを狙いとしています。",
      "en": "This problem reinforces fundamental division concepts and helps learners understand how to categorize two-digit integers based on divisibility rules."
    },
    "input_format": {
      "fields": [
        {
          "field_id": "f_1",
          "attribute": "array",
          "user_answer": "sequence"
        },
        {
          "field_id": "f_2",
          "attribute": "array",
          "user_answer": "sequence"
        },
        {
          "field_id": "f_3",
          "attribute": "array",
          "user_answer": "sequence"
        },
        {
          "field_id": "f_4",
          "attribute": "array",
          "user_answer": "sequence"
        },
        {
          "field_id": "f_5",
          "attribute": "array",
          "user_answer": "sequence"
        }
      ],
      "input_components": [
        {
          "type": "number",
          "content": {
            "ja": "14",
            "en": "14"
          },
          "order": 50
        },
        {
          "type": "number",
          "content": {
            "ja": "15",
            "en": "15"
          },
          "order": 100
        },
        {
          "type": "number",
          "content": {
            "ja": "18",
            "en": "18"
          },
          "order": 150
        },
        {
          "type": "number",
          "content": {
            "ja": "20",
            "en": "20"
          },
          "order": 200
        },
        {
          "type": "number",
          "content": {
            "ja": "21",
            "en": "21"
          },
          "order": 250
        },
        {
          "type": "number",
          "content": {
            "ja": "25",
            "en": "25"
          },
          "order": 300
        },
        {
          "type": "number",
          "content": {
            "ja": "27",
            "en": "27"
          },
          "order": 350
        },
        {
          "type": "number",
          "content": {
            "ja": "35",
            "en": "35"
          },
          "order": 400
        },
        {
          "type": "number",
          "content": {
            "ja": "45",
            "en": "45"
          },
          "order": 450
        },
        {
          "type": "number",
          "content": {
            "ja": "50",
            "en": "50"
          },
          "order": 500
        }
      ],
      "question_components": [
        {
          "type": "text",
          "content": {
            "ja": "2で割り切れる数",
            "en": "Numbers divisible by 2"
          },
          "order": 50,
          "attribute": "text"
        },
        {
          "type": "input_field",
          "field_id": "f_1",
          "order": 100
        },
        {
          "type": "newline",
          "order": 150
        },
        {
          "type": "text",
          "content": {
            "ja": "3で割り切れる数",
            "en": "Numbers divisible by 3"
          },
          "order": 250,
          "attribute": "text"
        },
        {
          "type": "input_field",
          "field_id": "f_2",
          "order": 300
        },
        {
          "type": "newline",
          "order": 350
        },
        {
          "type": "text",
          "content": {
            "ja": "5で割り切れる数",
            "en": "Numbers divisible by 5"
          },
          "order": 450,
          "attribute": "text"
        },
        {
          "type": "input_field",
          "field_id": "f_3",
          "order": 500
        },
        {
          "type": "newline",
          "order": 550
        },
        {
          "type": "text",
          "content": {
            "ja": "7で割り切れる数",
            "en": "Numbers divisible by 7"
          },
          "order": 650,
          "attribute": "text"
        },
        {
          "type": "input_field",
          "field_id": "f_4",
          "order": 700
        },
        {
          "type": "newline",
          "order": 750
        },
        {
          "type": "text",
          "content": {
            "ja": "9で割り切れる数",
            "en": "Numbers divisible by 9"
          },
          "order": 850,
          "attribute": "text"
        },
        {
          "type": "input_field",
          "field_id": "f_5",
          "order": 900
        },
        {
          "type": "newline",
          "order": 950
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
    case CALCULATION             = 1;
    case FILL_IN_THE_BLANK       = 51;
    case SCENARIO                = 101;
    case MULTIPLE_CHOICE         = 151;
    case FILL_IN_OPERATOR         = 201;
    case CLASSIFY_THE_OPTIONS = 251; // 選択肢を、あらかじめ提示された分類基準（例：割り切れる数）に従って振り分ける形式。



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
            self::CALCULATION            => 'CALCULATION',              // 計算問題: 数値計算や演算ルールの処理を中心
            self::FILL_IN_THE_BLANK      => 'FILL_IN_THE_BLANK',        // 穴埋め問題: 問題文や式に空所があり、そこを埋める形式
            self::SCENARIO               => 'SCENARIO',                 // シナリオ問題: 状況や経過を踏まえて解決する（回答が一意じゃない）
            self::MULTIPLE_CHOICE        => 'MULTIPLE_CHOICE',          // 選択問題: 複数の選択肢から答えを選ぶ
            self::FILL_IN_OPERATOR        => 'FILL_IN_OPERATOR',          // 演算子の選択問題
            self::CLASSIFY_THE_OPTIONS        => 'CLASSIFY_THE_OPTIONS',          // 演算子の選択問題

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
            'CLASSIFY_THE_OPTIONS'   => self::CLASSIFY_THE_OPTIONS,
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

```php
＝＝＝問題取り込みコード
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

        return 0;
    }

    /**
     * 【B案】Google Drive のファイルを
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

            // metadata が無いならスキップ
            $metadata = $questionJson['metadata'] ?? [];
            if (empty($metadata)) {
                $this->warn("metadata が存在しないためスキップ: json_id={$jsonId}");
                DB::rollBack();
                return false;
            }

            // JSONバリデーション
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

            // 既存レコードを検索 (json_id = $jsonId)
            $question = Question::where('json_id', $jsonId)->first();

            // order, version
            $jsonOrder = $questionJson['order'] ?? 9999;
            $version   = $questionJson['version'] ?? '0.0.1';

            // status
            $rawStatus = $questionJson['status'] ?? null;
            $statusValue = $rawStatus
                ? $this->parseQuestionStatus($rawStatus)
                : QuestionStatus::DRAFT->value;

            // level_id / difficulty_id / grade_id
            $levelUuid      = $this->findLevelUuid($questionJson['level_id'] ?? null);
            $gradeUuid      = $this->findGradeUuid($questionJson['grade_id'] ?? null);
            $difficultyUuid = $this->findDifficultyUuid($questionJson['difficulty_id'] ?? null);

            // 新規or既存
            if (!$question) {
                $question = new Question();
                $question->id      = (string) Str::uuid();
                $question->json_id = $jsonId;
            }

            // 項目反映
            $question->order = $questionJson['order'];
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

--- QuestionJsonManageService
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
            $question->id      = (string) Str::uuid();
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
            // すでにvalidation済みだが、本体に保存
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
                            // "number" の場合は数値
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
                                $fio = FillInOperator::from($val); // '>', '<', '=' 等
                            } catch (\ValueError $ve) {
                                $validator->errors()->add(
                                    "evaluation_spec.response_format.fields.{$i}.collect_answer.{$lang}",
                                    "FILL_IN_OPERATOR の場合 collect_answer.{$lang} は演算子（>,<,=）のいずれかが必要です。"
                                );
                            }
                        }
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

                // user_answer => 'number' のみ
                if (!isset($f['user_answer']) || !is_string($f['user_answer'])) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.user_answer",
                        "user_answer は必須の文字列です。"
                    );
                } else {
                    try {
                        $ft = FieldType::from($f['user_answer']);
                    } catch (\ValueError $ve) {
                        // Enumに無い文字列が渡された場合
                        $validator->errors()->add("metadata.input_format.fields.{$idx}.user_answer",
                            "user_answer='{$f['user_answer']}' は 'number','string','operator' のいずれかのみ有効です。"
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
                                $fio = FillInOperator::from($val);
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
     * ③ ユーザー回答のバリデーション例
     * --------------------------------------------------------------------------------
     * 【目的】
     *   - question_type='FILL_IN_THE_BLANK' を例にした回答検証
     *   - metadata.input_format との整合性 (fields数/attribute/field_id) をチェック
     *   - ここでは一部簡易実装のみ示している。 (実運用では他 question_type にも対応)
     */

    public function validateUserAnswer(array $answerData, array $metadata): void
    {
        // metadata は localizeMetadata() で、ローカライズされている前提
        // question_type は数値になっている
        $qt = $metadata['question_type'] ?? null;

        $inFmt = $metadata['input_format'] ?? [];
        if (!is_array($inFmt)) {
            // metadata.input_formatが存在しません。
            throw ValidationException::withMessages([
                'metadata.input_format' => [__('errors.api.answer.mismatch_answer_format')]
            ]);
        }

        $metaFields = $inFmt['fields'] ?? [];
        if (!is_array($metaFields)) {
            // metadata.input_format.fields が配列ではありません。
            throw ValidationException::withMessages([
                'metadata.input_format.fields' => [__('errors.api.answer.mismatch_answer_format')]
            ]);
        }

        $answerFields = $answerData['fields'] ?? null;
        if (!is_array($answerFields)) {
            throw ValidationException::withMessages([
                // 回答JSONの 'fields' が配列ではありません。
                'answerData.fields' => [__('errors.api.answer.mismatch_answer_format')]
            ]);
        }

        if ($qt == QuestionType::FILL_IN_THE_BLANK->value
        || $qt == QuestionType::FILL_IN_OPERATOR->value) {
            if (count($metaFields) !== count($answerFields)) {
                throw ValidationException::withMessages([
                    // fields数が一致しません。(metadata=".count($metaFields).", answer=".count($answerFields).")
                    'fields.count' => [__('errors.api.answer.mismatch_answer_format')]
                ]);
            }
        }

        $metaFieldMap = [];
        foreach ($metaFields as $mf) {
            if (!empty($mf['field_id'])) {
                $metaFieldMap[$mf['field_id']] = $mf;
            }
        }

        foreach ($answerFields as $idx => $af) {
            $fid = $af['field_id'] ?? '';
            if (!$fid) {
                throw ValidationException::withMessages([
                    // fields.{$idx}.field_id は必須です。
                    "fields.{$idx}.field_id" => [__('errors.api.answer.mismatch_answer_format')]
                ]);
            }

            $attr = $af['attribute'] ?? null;
            $uAns = $af['user_answer'] ?? null;

            if (!$attr) {
                throw ValidationException::withMessages([
                    // fields.{$idx}.attribute が存在しません。
                    "fields.{$idx}.attribute" => [__('errors.api.answer.mismatch_answer_format')]
                ]);
            }

            if ($qt == QuestionType::FILL_IN_THE_BLANK->value && !isset($metaFieldMap[$fid])) {
                throw ValidationException::withMessages([
                    // metadataに存在しない field_id='{$fid}' です。
                    "fields.{$idx}.field_id" => [__('errors.api.answer.mismatch_answer_format')]
                ]);
            }



            if ($attr === FieldType::NUMBER->value) {
                if (!is_numeric($uAns)) {
                    throw ValidationException::withMessages([
                        // attribute='number' の場合 user_answer に数値が必要です。
                        "fields.{$idx}.user_answer" => [__('errors.api.answer.mismatch_answer_format_to_number')]
                    ]);
                }
            } else if ($attr === QuestionMetadataInputFormatFieldAttribute::OPERATOR->value) {
                try {
                    $fio = FillInOperator::from($uAns);
                } catch (\ValueError) {
                    throw ValidationException::withMessages([
                        "fields.{$idx}.user_answer" => [__('errors.api.answer.mismatch_answer_format_operator')],
                    ]);
                }

            } else {
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
