Laravel11 , OpenAPI の APIに以下のプロンプトを投げてJSONレスポンスを受け取るAPIを実装したいです。

# 条件
1.
OpenAPI の APIキーは configのservices.openai.api_key です。

2.
{# RequestJSON} は、ユーザーがAPI経由でリクエストするリクエストボディです。
プロンプトの{# RequestJSON}の箇所をユーザーから渡されたリクエストボディに差し替えてリクエストするようにしてください。

ーープロンプト
{#条件}に従って以下の{#RequestJSON}のuser_answer が question に正答しているか確認してください。

# 条件
・回答は、{#回答フォーマット}と同一にすること。
・回答のアウトプットは、JSONのみとすること。
・この問題の対象者は{学習要件}を参考にしてください。解説は対象者に分かりやすい説明であること

# 説明
user_answer: ユーザー（学習者）の回答
collect_answer: 正しい回答。LLMが入力する
is_collect: ユーザーの回答が正答かどうが。正答なら true, 誤答なら false。LLMが入力する
field_explanation：フィールドごとに、なぜその回答なのか解説する。LLMが入力する
explanation: この問題の解説。LLMが入力する

# RequestJSON
```json
{
  "question_text": {
    "ja": "▢にあてはまる数を答えなさい。",
    "en": "Please answer the numbers that fit in the blanks."
  },
  "explanation": {
    "ja": "",
    "en": ""
  },
  "question": "8 × 4 = ▢ × 8 = ▢",
  "problem_id": "prob_s1_l2_001",
  "fields": [
    {
      "field_id": "f_1",
      "user_answer": "2",
      "is_collect": "",
      "collect_answer": "",
      "field_explanation": {
        "ja": "",
        "en": ""
      }
    },
    {
      "field_id": "f_2",
      "user_answer": "32",
      "is_collect": "",
      "collect_answer": "",
      "field_explanation": {
        "ja": "",
        "en": ""
      }
    }
  ]
}

```

# 回答フォーマット
```json
{
  "question_text": {
    "ja": "▢にあてはまる数を答えなさい。",
    "en": "Please answer the numbers that fit in the blanks."
  },
  "explanation": {
    "ja": "",
    "en": ""
  },
  "question": "8 × 4 = ▢ × 8 = ▢",
  "fields": [
    {
      "field_id": "f_1",
      "user_answer": "2",
      "is_collect": "",
      "collect_answer": "",
      "field_explanation": {
        "ja": "",
        "en": ""
      }
    },
    {
      "field_id": "f_2",
      "user_answer": "32",
      "is_collect": "",
      "collect_answer": "",
      "field_explanation": {
        "ja": "",
        "en": ""
      }
    }
  ],
  "learning_requirements": [
    {
      "learning_subject": "算数",
      "learning_no": 10,
      "learning_requirement": "数と計算 計算の意味・方法 乗法の意味",
      "learning_required_competency": "かけ算は同じ数の繰り返し加法であるという本質を理解できる",
      "learning_category": "A",
      "learning_grade_level": "小2",
      "learning_url": "https://docs.google.com/spreadsheets/d/1W5vaFHcyU_BrMwb1JLZ-DpyFmXXZPaYMTHIITcwLqqY/edit?gid=0#gid=0&range=15:15"
    }
  ]
}

```

