以下のQuestionJsonの構造のevaluation_specと、metadataの説明を作成してもらえませんか？
Fill_in_the_blank 特有の構造です。

evaluation_spec 以下は、ユーザーの回答の正誤評価に使うデータです。

このQuestionJsonは、Vue.jsで作成されたフロントエンドのアプリケーションで、
問題画面のUIを構築するときのルールを定義しています。
metadata が、回答の入力ボックスやデータの種類などを定義しています。

特に、metadata の
evaluation_spec.response_format.fields.は
field_id　によって、
metadata.input_format.fields　と
関係していることをうまく説明したい


わからないことがあれば質問してください。

ーー
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
metadata.input_format.type: 必須、input_format type 定数と値が一致しているか。
metadata.input_format.fields: 必須、配列。ユーザが回答する入力フォームの仕様を定義
metadata.input_format.fields.field_id: 必須、f_x のフォーマットになっているか。同じ fields 内に重複した値が存在しないか。 question_components内の type: "blank"の数と総数が合っているか。
metadata.input_format.fields.attribute: 必須、（input_format.fields.type、evaluation_spec.response_format.fields.user_answer,evaluation_spec.response_format.fields.collect_answer の定数に値が存在しているか）。ブランクのフォームの属性。例えば number であれば、<input type="number">になる
metadata.input_format.fields.user_answer: 必須、次の条件を満たした文字列型かどうかをチェックする。input_format.fields.type、evaluation_spec.response_format.fields.user_answer,evaluation_spec.response_format.fields.collect_answer の定数に値があるか。ユーザーが入力する正答の型。
metadata.input_format.fields.collect_answer: 絶対に存在してはいけない。ユーザーに回答が見えてしまうため。

metadata.input_format.question_components: 必須（問題を更生する要素）
metadata.input_format.question_components.attribute：必須、input_format.question_components.type定数と値が合っているか
metadata.input_format.question_components.content：必須、オブジェクト、言語定数と一致する値が全て含まれているか
metadata.input_format.question_components.order: 必須、数値、重複する値が存在しないこと。問題を構築するときの表示順番


ーー参考のQuestion JSON
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
    "question": {
      "ja": "315 + 276 = (300 + 200) + (10 + 70) + (5 + 6) = ▢ + ▢ + ▢ = ▢",
      "en": "315 + 276 = (300 + 200) + (10 + 70) + (5 + 6) = ▢ + ▢ + ▢ = ▢"
    },
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
          "user_answer": "number"
        },
        {
          "field_id": "f_2",
          "attribute": "number",
          "user_answer": "number"
        },
        {
          "field_id": "f_3",
          "attribute": "number",
          "user_answer": "number"
        },
        {
          "field_id": "f_4",
          "attribute": "number",
          "user_answer": "number"
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
