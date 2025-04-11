以下の条件を満たす、Service クラスを実装してください。

式問題（question_type: FILL_IN_FORMULA）の正誤判定を実装を追加してください。

# 指示
1.
QuestionJson の　evaluation_spec.evaluation_method が CODE の場合は、予め用意してあるメソッドで正誤判定をしたいです。
正誤メソッドの名前は、App\Enums\EvaluationCheckerMethod で管理し、evaluation_spec.checker_method で指定します。
evaluation_spec.checker_method は、user_questions テーブルの、checker_method に保存されているのでそちらを使用してください。

2.
正誤メソッド は、今後も無数に増え続けることと、正誤判定（AnswerService） 以外でも使用しますので、
正誤判定用メソッドだけを集めた Service クラスを実装してください。
ネームスペースは、namespace App\Services\Utils　以下に作ってください

3.
1で実装した判定メソッドを、AnswerService で、evaluation_spec.evaluation_method が CODE の時に、
evaluation_spec.checker_method で指定されているメソッドで判定するように実装してください

4.
今回は、まず第一弾として、穴埋め問題（question_type: FILL_IN_THE_BLANK）の正誤判定を実装してください。
レスポンスは、user_questions.evaluation_response_format と同じ形式でレスポンスしてください。

判定方式は、
user の回答データは、metadata.input_format と同じ形式で｛#ユーザーが回答する時に送るJSON}がリクエストされます。
｛#問題JSONの全体構造} を見てもらうと、metadata.input_format.user_answer に、定義された型（例えば "number" とあれば 数値型）で値が入力されます。


ユーザーが回答する時に送るJSONの fields　の field_id と一致する、
evaluation_spec.response_format.fields（必ず user_questions から取得すること） の field_id を探し、その値の
evaluation_spec.response_format.fields.collect_answer　と、
ユーザーが回答する時に送るJSONの fields　の　user_answer　が一致すれば正答です。

5.
レスポンスは、evaluation_spec.response_format と同じ形式でレスポンスします。

is_correct: fields の値が全て正答だった場合は、true, そうでなければ false
score: 100 点を fields の数で割った値が１問辺りの点数で、正答数で計算して入力
question_text,explanation,question は、App::getLocale で現在設定されている言語設定を、user_questions から取得して入れる。
fields.user_answer は、field_id が一致するユーザーの回答
fields.is_correct は、field_id が一致するユーザーの回答が、collect_answerと一致すれば true
fields.user_answer は、field_id が一致するユーザーの回答
fields.field_explanation は、App::getLocale で現在設定されている言語設定を、evaluation_spec.response_format.fields.field_explanation　で定義されている文字列を入れる

6.
"check_method": "CALCULATE_METHOD",と指定されていますが、CALCULATE_METHOD は今回の実装の文脈には合わないので、
新しい　Enum の値を追加してください。
FILL_IN_THE_BLANKでも文字列ケースの評価とか色々想定されるので、値が完全に一致しているかを評価することが分かるメソッド名にしてください

7.
例えば、
$checkerEnum = EvaluationCheckerMethod::from("CALCULATE_METHOD");
switch ($checkerEnum) {
のような形式で、switch や if などで、条件分岐をすると、評価メソッドは今後数百以上数が増える可能性があり冗長になってしまうので、
なにか enum の値から呼び出しメソッドをコール出来るような工夫をしてほしい

8.
method が存在するかはちゃんとチェックするべき。エラーになる。

9.
Enum のキーはメソッド名と同じにしてください。
例えば、以下のように、キーは、CHECK_BY_EXACT_MATCH として
メソッド名は、checkByExactMatch　として、
大文字_大文字　という形式からキャメルケースに変換するような処理にするとどうだろうか。
文字列変換は、共通処理なので、CommonLib  に、「大文字_大文字　という形式からキャメルケースに変換する」処理ということが分かりやすいメソッド名で実装して共通化してほしい



※注意点
・現在は question_type FILL_IN_THE_BLANK の仕様だけですが、将来的に複数の種類が追加されて行く予定です。１００個以上になるかも

・エラーメッセージは多言語化が必要なので、その旨を日本語でTODOコメントとして残しておいてください

・後で分かりやすいようにソースコードのコメントには必ず実装の仕様をまとめて残しておくこと

・コードは Testable に記述すること。密結合しないように実装してください

・QuestionJson の仕様は、QuestionJsonManageService を参考にしてください


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

evaluation_spec.response_format.fields：evaluation_methodが”LLM”の時は必須、配列。LLMにユーザーの回答の形を知らせる為にフォーマットを定義。
evaluation_spec.response_format.fields.field_id：必須、配列。ユーザーの回答の型。
evaluation_spec.response_format.fields.user_answer：必須、input_format.fields.type、evaluation_spec.response_format.fields.user_answer,evaluation_spec.input_format.fields.collect_answer の定数に値があるか。ユーザーの回答。
evaluation_spec.response_format.fields.is_correct：必須、テキスト型("boolean"のみ）。ユーザーの回答が正しいか
evaluation_spec.response_format.fields.collect_answer：必須、evaluation_spec.response_format.fields.user_answerで指定されている型と一致しているか。例えば、"number" の場合は、32 などの数値となっているか。問題の正解（ユーザーには隠す）
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

metadata.input_format.question_components: 必須（問題を更生する要素）
metadata.input_format.question_components.attribute：必須、input_format.question_components.type定数と値が合っているか
metadata.input_format.question_components.content：必須、オブジェクト、言語定数と一致する値が全て含まれているか
metadata.input_format.question_components.order: 必須、数値、重複する値が存在しないこと。問題を構築するときの表示順番

--- ユーザーが回答する時に送るJSON
```json
{
  "fields": [
    {
      "field_id": "f_1",
      "attribute": "number",
      "user_answer": 4
    },
    {
      "field_id": "f_2",
      "attribute": "number",
      "user_answer": 32
    }
  ]
}

```

--- 問題JSONの全体構造
```json
{
  "order": 100,
  "id": "ques_s1_g2_sec800_u100_diff100_qt51_v100_100",
  "level_id": "lev_002",
  "grade_id": "gra_002",
  "difficulty_id": "diff_100",
  "version": "1.0.0",
  "status": "PUBLISHED",
  "generated_by_llm": true,
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
      "learning_no": 22,
      "learning_requirement": "計算の意味・方法 計算の決まり 加法の交換法則を式で示す",
      "learning_required_competency": "a + b = b + a を具体例（3+5=5+3等）で理解し，同じ結果になることを確認できる",
      "learning_background": "実際に小物を数える・ブロックを移し替えるなど操作的活動と組み合わせると，法則への納得感が高まる。後の乗法の交換法則にも発展しやすい",
      "learning_category": "A",
      "learning_grade_level": "小2",
      "learning_url": ""
    }
  ],
  "evaluation_spec": {
    "evaluation_method": "CODE",
    "check_method": "CALCULATE_METHOD",
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
            "ja": "3 + 5 の順序を入れ替えると 5 + 3 なので、ブランクには 3 が入ります。",
            "en": "When swapping the order of 3 + 5, we get 5 + 3, so 3 fits in the blank."
          }
        },
        {
          "field_id": "f_2",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": 8,
          "field_explanation": {
            "ja": "3 + 5 と 5 + 3 は同じ答えで 8 です。",
            "en": "Both 3 + 5 and 5 + 3 result in 8."
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
    "question_text": {
      "ja": "▢にあてはまる数を答えなさい。",
      "en": "Please answer the numbers that fit in the blanks."
    },
    "explanation": {
      "ja": "加法の交換法則という考え方を使うと、3 + 5 の答えと、5 + 3 の答えは同じになります。今回は、どちらの順番で足しても同じ合計になることを確かめましょう。",
      "en": "Using the commutative property of addition, 3 + 5 and 5 + 3 give the same result. In this problem, you will check that the total is the same no matter the order."
    },
    "background": {
      "ja": "この問題では、加法（足し算）の交換法則「a + b = b + a」を学習します。小学生2年生の段階で、足し算は順番を変えても同じ答えになることを具体的に体験してもらうための問題です。",
      "en": "This problem focuses on the commutative property of addition, showing that a + b = b + a. It is designed for second-graders to experience that switching the order in addition yields the same result."
    },
    "input_format": {
      "type": "fixed",
      "fields": [
        {
          "field_id": "f_1",
          "attribute": "number",
          "user_answer": "number"
        },
        {
          "field_id": "f_2",
          "attribute": "number",
          "user_answer": "number"
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

