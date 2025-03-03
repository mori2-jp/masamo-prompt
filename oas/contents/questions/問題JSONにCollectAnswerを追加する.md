evaluation_spec.response_format.fields.collect_answerの条件は以下です。
evaluation_spec.response_format.fields.collect_answer：evaluation_methodが”LLM”の時は必須、evaluation_spec.response_format.fields.user_answerで指定されている型と一致しているか。例えば、"number" の場合は、32 などの数値となっているか。問題の正解（ユーザーには隠す）

以下２つのJSONの、evaluation_spec.response_format.fields.collect_answerが"number"になっているので、question の正しい正答を埋めてください。

ーーー対象JSON
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
          "collect_answer": "number",
          "field_explanation": {
            "ja": "text",
            "en": "text"
          }
        },
        {
          "field_id": "f_2",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": "number",
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
          "collect_answer": "3"
        },
        {
          "field_id": "f_2",
          "attribute": "number",
          "collect_answer": "8"
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
2つ目
```json
{
  "order": 200,
  "id": "ques_s1_g2_sec100_u300_diff100_qt51_v100_200",
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
    "ja": "たし算には、(a + b) + c と a + (b + c) のようにまとまりを変えても結果が同じになる『けつごうほうそく』という大事な性質があります。たとえば (4 + 2) + 3 と 4 + (2 + 3) は、どちらも同じ答えになることを確かめましょう。",
    "en": "Addition has an important property called the associative property, which means (a + b) + c and a + (b + c) produce the same result. For example, (4 + 2) + 3 and 4 + (2 + 3) both yield the same answer."
  },
  "background": {
    "ja": "この問題は、加法の結合法則(a + b) + c = a + (b + c) を実際に体験し、たし算でのまとまり方を変えても答えが同じになることを理解してもらうために作られています。",
    "en": "This problem is designed to let learners experience the associative property of addition—(a + b) + c = a + (b + c)—and understand that changing how numbers are grouped does not change the result."
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
          "collect_answer": "number",
          "field_explanation": {
            "ja": "text",
            "en": "text"
          }
        },
        {
          "field_id": "f_2",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": "number",
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
      "ja": "(4 + 2) + 3 = 4 + (2 + ▢) = ▢",
      "en": "(4 + 2) + 3 = 4 + (2 + ▢) = ▢"
    },
    "input_format": {
      "type": "fixed",
      "fields": [
        {
          "field_id": "f_1",
          "attribute": "number",
          "collect_answer": "3"
        },
        {
          "field_id": "f_2",
          "attribute": "number",
          "collect_answer": "9"
        }
      ],
      "question_components": [
        {
          "type": "text",
          "content": {
            "ja": "(4 + 2) + 3 = 4 + (2 + ",
            "en": "(4 + 2) + 3 = 4 + (2 + "
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
            "ja": ") = ",
            "en": ") = "
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
