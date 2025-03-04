以下の条件を満たすように AnswerService　と QuestionJsonManageService を修正してください 

# 指示
1.
AnswerService　の parseAndValidateAnswerData　は、ユーザーから送られてくる問題への回答JSONの形式が正しいかチェックするメソッドです。
Question に関するJSONの取り扱いは、QuestionJsonManageService に一任しているので移動してください。
parseAndValidateAnswerData は消してよいという意味です。

2.
ユーザーからの回答は、metadata.input_format.fields と同様の形式で送られてくる必要があります。
現在のルールは定性的に定義されていますが、metadata.input_format.fields　に整合しているか動的にチェックするように実装してください。
metadata.input_format は、question_type ごとに定義が変わる可能性があるという意図です。
question_type: FILL_IN_THE_BLANK については、以下の metadata.input_format の形式のみです。

3.
metadata.input_format は、user_questions の metadata から取得してください。
QuestionJSONの metadata が全てJSONとして保存されています。
以下、保存されているデータの例）
```json
{
    "question_type": "FILL_IN_THE_BLANK",
    "question": {
      "ja": "46 + 27 = 40 + 20 + 6 + 7 = 60 + ▢ = ▢",
      "en": "46 + 27 = 40 + 20 + 6 + 7 = 60 + ▢ = ▢"
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
            "ja": "46 + 27 = 40 + 20 + 6 + 7 = 60 + ",
            "en": "46 + 27 = 40 + 20 + 6 + 7 = 60 + "
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
```
4.
metadata.input_format の仕様は以下を参考にしてください

5.
input_format.fields が "fixed" の時は、fields の数は input_format.fields で定義されている数に固定されます。
input_format.fields が "custom" の時は、fields の数は input_format.fields で定義されている数に固定されず、任意の数が送信されます。

6.
user_answer は、attribute と同じ形式になっているか確認する。
例えば、attribute が number の場合は、user_answer は数値でなければいけない。input_format.fields.type、evaluation_spec.response_format.fields.user_answer,evaluation_spec.response_format.fields.user_answer の定義と一致する

7.
AnswerService  の、evaluateAnswer の answerDataJson には、{ユーザーが回答する時に送るJSON} が渡されます。※ この時点では、answer_data というキーはありません。

※注意点
・現在は question_type FILL_IN_THE_BLANK の仕様だけですが、将来的に複数の種類が追加されて行く予定です。１００個以上になるかも

・エラーメッセージは多言語化が必要なので、その旨を日本語でTODOコメントとして残しておいてください

・後で分かりやすいようにソースコードのコメントには必ず実装の仕様をまとめて残しておくこと

・QuestionJson の仕様は、QuestionJsonManageService を参考にしてください

ーー言語定数
ja, en

--input_format type 定数
fixed: 固定。ユーザーは項目の増減はコントロール出来ず、input_format.fields の内容に固定
custom: input_format.fields の内容はあくまでもデフォルト表記で、固定されず、ユーザー自由に回答の数を増減出来る。

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

question_text: 必須、オブジェクト（言語定数全て含んでいるか）
explanation: 必須、オブジェクト（言語定数全て含んでいるか）
background: 必須、オブジェクト（言語定数全て含んでいるか）

skills: 必須、配列
skills.skill_id: 必須、Skills テーブルのjson_idに一致する値があること
skills.name: 必須、文字列、skill_id と一致するSkills テーブルのjson_idを持つデータの　display_name と値が一致すること


metadata.input_format: 必須、オブジェクト（言語定数全て含んでいるか）
metadata.input_format.type: 必須、input_format type 定数と値が一致しているか。
metadata.input_format.fields: 必須、配列。ユーザが回答する入力フォームの仕様を定義
metadata.input_format.fields.field_id: 必須、f_x のフォーマットになっているか。同じ fields 内に重複した値が存在しないか。 question_components内の type: "blank"の数と総数が合っているか。
metadata.input_format.fields.attribute: 必須、（input_format.fields.type、evaluation_spec.response_format.fields.user_answer,evaluation_spec.response_format.fields.user_answer の定数に値が存在しているか）。ブランクのフォームの属性。例えば number であれば、<input type="number">になる
metadata.input_format.fields.user_answer: 必須、次の条件を満たした文字列型かどうかをチェックする。input_format.fields.type、evaluation_spec.response_format.fields.user_answer,evaluation_spec.response_format.fields.collect_answer の定数に値があるか。ユーザーが入力する正答の型。

metadata.input_format.question_components: 必須（問題を更生する要素）
metadata.input_format.question_components.attribute：必須、input_format.question_components.type定数と値が合っているか
metadata.input_format.question_components.content：必須、オブジェクト、言語定数と一致する値が全て含まれているか
metadata.input_format.question_components.order: 必須、数値、重複する値が存在しないこと。問題を構築するときの表示順番

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

--- 問題JSON
```json
{
  "order": 100,
  "id": "ques_s1_g2_sec100_u100_diff100_qt51_v100_100",
  "level_id": "lev_002",
  "grade_id": "gra_002",
  "difficulty_id": "diff_100",
  "version": "1.0.0",
  "status": "PUBLISHED",
  "generated_by_llm": true,
  "created_at": "2025-01-01 00:00:00",
  "updated_at": "2025-01-01 00:00:00",
  "question_text": {
    "ja": "▢にあてはまる数を答えなさい。",
    "en": "Please answer the numbers that fit in the blanks."
  },
  "explanation": {
    "ja": "2桁のたし算では、一の位を足して10をこえるときにくり上がりがあります。たとえば 46 + 27 は、一の位が 6 + 7 = 13 となり、そこから 40 + 20 = 60 と合わせることで 73 に計算できます。筆算では、それぞれの位をきちんとそろえて足し算し、くり上がりに注意しましょう。",
    "en": "When adding two-digit numbers, if the ones digit sum is 10 or more, you need to carry over. For example, 46 + 27 has 6 + 7 = 13 in the ones place, then add 40 + 20 = 60 for a total of 73. In column addition, line up each digit carefully and handle the carry properly."
  },
  "background": {
    "ja": "この問題では、2桁のたし算でくり上がりがある場合の計算方法を理解し、筆算の基礎となる位の考え方を身につけてほしいという狙いがあります。",
    "en": "The purpose of this question is to help learners understand how to handle carrying in two-digit addition, reinforcing the concept of place value as the foundation of columnar addition."
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
      "learning_no": 11,
      "learning_requirement": "A 数と計算 計算の意味・方法 ２位数の加法とその逆の減法",
      "learning_required_competency": "1位数で習得した加減法を基に、2位数の繰り上がり・繰り下がりを正しく理解し筆算できる",
      "learning_background": "十進位取り記数法を具体物や図で再確認し、筆算の形式が成立する理由を体感的に理解させる",
      "learning_category": "A",
      "learning_grade_level": "小2",
      "learning_url": "https://docs.google.com/spreadsheets/d/1W5vaFHcyU_BrMwb1JLZ-DpyFmXXZPaYMTHIITcwLqqY/edit?gid=0#gid=0&range=11:11"
    }
  ],
  "evaluation_spec": {
    "evaluation_method": "CODE",
    "checker_method": "CALCULATE_METHOD",
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
          "collect_answer": 13,
          "field_explanation": {
            "ja": "text",
            "en": "text"
          }
        },
        {
          "field_id": "f_2",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": 73,
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
      "ja": "46 + 27 = 40 + 20 + 6 + 7 = 60 + ▢ = ▢",
      "en": "46 + 27 = 40 + 20 + 6 + 7 = 60 + ▢ = ▢"
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
            "ja": "46 + 27 = 40 + 20 + 6 + 7 = 60 + ",
            "en": "46 + 27 = 40 + 20 + 6 + 7 = 60 + "
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

--- AnswerService
```php
<?php

namespace App\Services\V1\Answer;

use App\Dtos\V1\Answer\AnswerCheckDto;
use App\Enums\EvaluationMethod;
use App\Enums\UserQuestionSetStatus;
use App\Enums\UserQuestionStatus;
use App\Models\Question\Question;
use App\Models\User\UserQuestion;
use App\Models\User\UserQuestionSet;
use Carbon\Carbon;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Http;
use Illuminate\Validation\ValidationException;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

class AnswerService
{
    // TODO 回答メソッド指定への対応。例えば計算とか
    /**
     * (1) user_question_id & answer_data を受け取り
     * (2) question.evaluation_method で LLM or CODE 判定
     * (3) user_questions に正誤・回答日時を保存
     * (4) 次の問題を探して返却 / なければ user_question_sets を COMPLETE
     * (5) 返却用に AnswerCheckDto を組み立てる
     */
    public function evaluateAnswer(string $userId, string $userQuestionId, string $answerDataJson): AnswerCheckDto
    {
        // user_questions と user_question_sets を確認
        // user_questions を取得し、ユーザーが所有するレコードかチェック
        $userQuestion = UserQuestion::where('id', $userQuestionId)->first();
        if (!$userQuestion) {
            // 多言語化対応した例
            throw new NotFoundHttpException(
                __("errors.api.answer.user_question_not_found", ['id' => $userQuestionId])
            );
        }

        // user_question_sets
        $userQuestionSet = UserQuestionSet::where('id', $userQuestion->user_question_set_id)->first();
        if (!$userQuestionSet) {
            throw new NotFoundHttpException(
                __("errors.api.answer.user_question_set_not_found", ['id' => $userQuestion->user_question_set_id])
            );
        }

        // user_id が一致するか
        if ($userQuestionSet->user_id !== $userId) {
            throw new NotFoundHttpException(
                __("errors.api.answer.user_question_not_found", ['id' => $userQuestionId])
            );
        }

        // answer_dataJsonをパース & 構造バリデーション
        $answerData = $this->parseAndValidateAnswerData($answerDataJson);

        // question を取得し、evaluation_method 分岐
        $question = Question::find($userQuestion->question_id);
        if (!$question) {
            throw new NotFoundHttpException(
                __("errors.api.answer.question_not_found", ['id' => $userQuestion->question_id])
            );
        }


        $answeredAt = Carbon::now();

        // DB更新処理
        DB::beginTransaction();
        try {
            if ($question->evaluation_method == EvaluationMethod::LLM->value) {
                // LLM 評価
                $llmResult = $this->callLLMEvaluationApi($answerData, $question, $userQuestion);

                // LLMが返す JSONの中で "fields" の is_correct が true なら正解とする簡易例
                $fields = data_get($llmResult, 'fields', []);
                $isAnyCorrect = false;
                foreach ($fields as $field) {
                    // "true" 文字列か boolean か不定なので緩く判定
                    if (data_get($field, 'is_correct') === true || data_get($field, 'is_collect') === "true") {
                        $isAnyCorrect = true;
                        break;
                    }
                }
                $status = $isAnyCorrect
                    ? UserQuestionStatus::CORRECT->value
                    : UserQuestionStatus::INCORRECT->value;

                // answer_data に LLM の出力を保存したいなら、$llmResult を保存する
                $saveAnswerData = $llmResult;
            } else {
                // CODE 評価
                $isCorrect = $this->evaluateByCode($question, $answerData);
                $status = $isCorrect
                    ? UserQuestionStatus::CORRECT->value
                    : UserQuestionStatus::INCORRECT->value;

                // answer_data はユーザー入力データをそのまま保存
                $saveAnswerData = $answerData;
            }

            // 回答時間などは更新しない。ステータスが場合は無視。
            if ($userQuestion->status != UserQuestionSetStatus::COMPLETE->value) {
                // user_questions に保存
                $userQuestion->status      = $status;
                $userQuestion->answered_at = $answeredAt;
                $userQuestion->answer_data = $saveAnswerData; // JSON
            }
            $userQuestion->save();

            // 次の問題を探す
            $nextUserQuestion = $this->findNextQuestion($userQuestionSet->id);
            // 回答時間などは更新しない。ステータスが進行中じゃない場合は無視。
            if (!$nextUserQuestion && $userQuestionSet->status == UserQuestionSetStatus::PROGRESS->value) {
                // もう問題がなければ user_question_sets を COMPLETE に
                $userQuestionSet->status = UserQuestionSetStatus::COMPLETE->value;
                $userQuestionSet->finished_at = Carbon::now();
                $userQuestionSet->save();
            }

            DB::commit();

            return $this->buildAnswerCheckDto(
                userQuestion: $userQuestion,
                question:     $question,
                nextUserQuestion: $nextUserQuestion,
                content: $saveAnswerData
            );
        } catch (\Throwable $e) {
            DB::rollBack();
            throw $e;
        }
    }

    /**
     * (1) JSONパース
     * (2) 必要なキー (fields, question など) が存在するか
     */
    private function parseAndValidateAnswerData(string $answerDataJson): array
    {
        // JSON パース
        $parsed = json_decode($answerDataJson, true);

        if (! is_array($parsed)) {
            // JSONデコード失敗, or オブジェクトでない
            throw ValidationException::withMessages([
                'answer_data' => __("errors.api.answer.invalid_json_format"),
            ]);
        }

        // 必須キーがあるか
        if (empty($parsed['question'])) {
            throw ValidationException::withMessages([
                'answer_data.question' => __("errors.api.answer.missing_question"),
            ]);
        }
        if (empty($parsed['fields']) || !is_array($parsed['fields'])) {
            throw ValidationException::withMessages([
                'answer_data.fields' => __("errors.api.answer.missing_fields"),
            ]);
        }

        // 例: fields.*.field_id & fields.*.user_answer の存在チェック
        foreach ($parsed['fields'] as $idx => $field) {
            if (empty($field['field_id'])) {
                throw ValidationException::withMessages([
                    "answer_data.fields.{$idx}.field_id" => __("errors.api.answer.field_id_required"),
                ]);
            }
            if (! array_key_exists('user_answer', $field)) {
                throw ValidationException::withMessages([
                    "answer_data.fields.{$idx}.user_answer" => __("errors.api.answer.user_answer_required"),
                ]);
            }
        }

        // OKなら返す
        return $parsed;
    }

    /**
     * LLM評価用のAPI呼び出し
     */
    private function callLLMEvaluationApi(array $answerData, Question $question, UserQuestion $userQuestion): array
    {
        $responseFormat = $question->evaluation_response_format;
        $learningRequirement = $question->learning_requirement_json;
        $userAnswerJson = json_encode($answerData, JSON_UNESCAPED_UNICODE | JSON_PRETTY_PRINT);
        $promptId = $question->llm_evaluation_prompt_number;
        $metadata = $userQuestion->metadata;


        $prompt = $this->buildPrompt($promptId, $userAnswerJson, $responseFormat, $learningRequirement, $metadata);

        $userQuestion->prompt_for_evaluation = $prompt;
        $userQuestion->save();

//        $model = 'gpt-4o-mini';
        $model = 'chatgpt-4o-latest';
//        $model = 'o3-mini';
        $response = Http::withToken(config('services.openai.api_key'))
            ->post('https://api.openai.com/v1/chat/completions', [
                'model' => $model,
                'messages' => [
                    [
                        'role' => 'system',
                        'content' => 'You are a helpful assistant. Return your answer in valid JSON format only, with no extra text.'
                    ],
                    [
                        'role' => 'assistant',
                        'content' => $responseFormat
                    ],
                    [
                        'role' => 'user',
                        'content' => $prompt
                    ]
                ],
                'response_format' => [
                  'type' => 'json_object'
                ],
                'temperature' => 0.0,
            ]);

        $responseBody = $response->json();

        $rawContent = data_get($responseBody, 'choices.0.message.content');

        $parsed = json_decode($rawContent, true);

        return $parsed ?? [];
    }

    /**
     * CODE評価 (evaluation_method=2) 用のロジック例
     */
    private function evaluateByCode(Question $question, array $answerData): bool
    {
        // 例: fields[0].user_answer が "32" なら正解
        $firstAnswer = data_get($answerData, 'fields.0.user_answer');
        return ($firstAnswer === "32");
    }

    /**
     * 次の問題( status=NOT_START )を order 昇順 で1件取得
     * なければ null
     */
    private function findNextQuestion(string $userQuestionSetId): ?UserQuestion
    {
        // user_questions と question_set_questions をJOINして
        // question_set_questions.order が最小の NOT_START を取得
        $row = DB::table('question_set_questions as qsq')
            ->join('user_questions as uq', 'uq.question_id', '=', 'qsq.question_id')
            ->where('uq.user_question_set_id', $userQuestionSetId)
            ->where('uq.status', UserQuestionStatus::NOT_START->value)
            ->orderBy('qsq.order', 'asc')
            ->select('uq.*')
            ->first();

        return $row ? (new UserQuestion)->newFromBuilder($row) : null;
    }

    /**
     * 回答後のレスポンス用 DTO を組み立てる
     */
    private function buildAnswerCheckDto(
        UserQuestion $userQuestion,
        Question $question,
        ?UserQuestion $nextUserQuestion,
        array $content,
    ): AnswerCheckDto {
        // ここでは既存の AnswerCheckDto::$content
        // 「現在の問題」と「次の問題」の情報を簡易的にまとめて返す例
        $nextUserQuestionId = null;

        if (!empty($nextUserQuestion)) {
            $nextUserQuestionId = $nextUserQuestion->id;
        }

        return new AnswerCheckDto(
            content: $content,
            next_user_question_id: $nextUserQuestionId
        );
    }

    public function buildPrompt(int $promptId, string $userAnswerJson, string $responseFormat, string $learningRequirement, string $metadata): string
    {
        $promptPath = resource_path("prompts/evaluation/{$promptId}.txt");
        if (! file_exists($promptPath)) {
            throw new \RuntimeException("Prompt file not found: {$promptPath}");
        }

        $template = file_get_contents($promptPath);

        $replaced = str_replace(
            [
                '{$userAnswerJson}',
                '{$responseFormat}',
                '{$learningRequirement}',
                '{$metadata}'
            ],
            [
                $userAnswerJson,
                $responseFormat,
                $learningRequirement,
                $metadata
            ],
            $template
        );

        return $replaced;
    }
}

```

--- QuestionJsonManageService
```php
<?php

namespace App\Services\Utils\Question;

use App\Enums\EvaluationCheckerMethod;
use App\Enums\EvaluationMethod;
use App\Enums\QuestionType;
use Illuminate\Support\Facades\App;
use Illuminate\Support\Facades\Validator;

/**
 * Class QuestionJsonManageService
 *
 * 【概要】
 * - GitHubリポジトリ等からインポートされる問題JSONをバリデーション＆加工するサービスクラス
 * - question_type が増えた場合にもルールを簡単に追加・削除できるように設計
 * - バリデーション違反で処理停止せず、その問題だけスキップ（ValidationException）できる
 *
 * 【今回の修正内容・実装仕様】
 * 1. トップレベルの question_text, explanation, background を削除
 *    → 代わりに metadata.question_text, metadata.explanation, metadata.background で必須チェック
 * 2. evaluation_method="CODE" 時、response_format.* が metadata の該当項目と一致することをチェック
 * 3. evaluation_method="LLM" 時、"text" 固定であることをチェック & fields が必須
 * 4. fields[*].field_explanation => 空文字禁止
 * 5. question_type が FILL_IN_THE_BLANK, SCENARIO など将来的に追加されても拡張しやすい
 * 6. エラーメッセージは日本語でわかりやすく
 * 7. バリデーション違反時に ValidationException をthrow → 呼び出し元はスキップ可能
 */
class QuestionJsonManageService
{
    /**
     * 言語定数 (アプリケーション全体で共通利用)
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
     * input_format.fields.type / evaluation_spec.response_format.fields.user_answer
     * / metadata.input_format.fields.user_answer の定数 ("number"のみ)
     */
    public const FIELD_TYPE = [
        'number',
    ];

    /**
     * question_components.type の定数
     */
    public const COMPONENT_TYPES = [
        'text',
        'image',
        'movie',
        'blank',
    ];

    /**
     * 問題JSONのバリデーション
     * バリデーションNGなら ValidationException を投げる（呼び出し元でスキップ可能）
     */
    public function validateQuestionJson(array $json): array
    {
        // 変更：トップレベルの question_text/explanation/background は削除

        // 1) トップレベル共通ルール
        $validator = Validator::make($json, $this->getTopLevelRules(), $this->messages());

        // 2) after()で追加の詳細チェック（DB存在, CODE/LLM分岐, metadata）
        $validator->after(function ($v) use ($json) {
            $this->validateLevelGradeDifficulty($v, $json);
            $this->validateSkills($v, $json);
            $this->validateEvaluationSpec($v, $json);
            $this->validateMetadata($v, $json);
        });

        // 3) 実行 (NGなら ValidationException)
        return $validator->validate();
    }

    /**
     * トップレベルの共通ルール
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

            // 変更： question_text/explanation/background をここで削除

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
            // 変更： question_text, explanation, background の必須チェックをmetadataへ移動
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
            'in'          => ':attribute に不正な値が指定されました( :values )。',
            'url'         => ':attribute は有効なURL形式で指定してください。',
        ];
    }

    /**
     * level_id,grade_id,difficulty_id が DBにあるかチェック
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
     * skills => skill_idがDBにあるか、nameがdisplay_nameと一致するか
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
     * evaluation_spec => evaluation_method=CODE or LLM
     */
    private function validateEvaluationSpec($validator, array $json)
    {
        $eval = $json['evaluation_spec'] ?? [];
        $methodRaw = $eval['evaluation_method'] ?? null;
        if (!$methodRaw) {
            return;
        }

        // enumに存在するか
        try {
            $method = EvaluationMethod::fromString($methodRaw);
        } catch (\InvalidArgumentException) {
            $validator->errors()->add('evaluation_spec.evaluation_method',
                "evaluation_method='{$methodRaw}' は無効です (CODE/LLM)。");
            return;
        }

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

            // response_format => 必須
            if (!isset($eval['response_format']) || !is_array($eval['response_format'])) {
                $validator->errors()->add('evaluation_spec.response_format',
                    "evaluation_method=CODE のため response_format(オブジェクト) が必須です。");
            } else {
                $this->validateResponseFormatForCode($validator, $json, $eval['response_format']);
            }

        } elseif ($method === EvaluationMethod::LLM) {
            // llm_prompt_number 必須 + ファイル
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

    // ======================================================================
    // CODE 用 response_format チェック
    // ======================================================================

    /**
     * CODE用 response_format を検証
     * - is_correct => "boolean"
     * - score => "number"
     * - question_text/explanation => 「metadata.question_text」「metadata.explanation」と一致
     * - question => metadata.question と一致
     * - fields => 必須ではないが、あればバリデーション
     */
    private function validateResponseFormatForCode($validator, array $json, array $resp)
    {
        // is_correct => "boolean"
        if (($resp['is_correct'] ?? '') !== 'boolean') {
            $validator->errors()->add('evaluation_spec.response_format.is_correct',
                "CODE: is_correct は 'boolean' のみ有効です。");
        }

        // score => "number"
        if (($resp['score'] ?? '') !== 'number') {
            $validator->errors()->add('evaluation_spec.response_format.score',
                "CODE: score は 'number' のみ有効です。");
        }

        // 変更： question_text ⇒ metadata.question_text と一致
        if (empty($resp['question_text']) || !is_array($resp['question_text'])) {
            $validator->errors()->add('evaluation_spec.response_format.question_text',
                "CODE: question_text(オブジェクト) が必須です。");
        } else {
            $this->validateResponseFormatQuestionTextForCode($validator, $json, $resp['question_text']);
        }

        // 変更： explanation ⇒ metadata.explanation と一致
        if (empty($resp['explanation']) || !is_array($resp['explanation'])) {
            $validator->errors()->add('evaluation_spec.response_format.explanation',
                "CODE: explanation(オブジェクト) が必須です。");
        } else {
            $this->validateResponseFormatExplanationForCode($validator, $json, $resp['explanation']);
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
     * question_text => metadata.question_text と一致
     */
    private function validateResponseFormatQuestionTextForCode($validator, array $json, array $obj)
    {
        // 変更：トップレベルから metadataへ移動
        foreach (self::LANGUAGES as $lang) {
            $expected = $json['metadata']['question_text'][$lang] ?? '';
            $actual   = $obj[$lang] ?? '';
            if ($actual !== $expected) {
                $validator->errors()->add("evaluation_spec.response_format.question_text.{$lang}",
                    "CODE: question_text.{$lang} は metadata.question_text.{$lang} と一致する必要があります( expected='{$expected}' )。");
            }
        }
    }

    /**
     * explanation => metadata.explanation と一致
     */
    private function validateResponseFormatExplanationForCode($validator, array $json, array $obj)
    {
        // 変更：トップレベル から metadata.explanation へ
        foreach (self::LANGUAGES as $lang) {
            $expected = $json['metadata']['explanation'][$lang] ?? '';
            $actual   = $obj[$lang] ?? '';
            if ($actual !== $expected) {
                $validator->errors()->add("evaluation_spec.response_format.explanation.{$lang}",
                    "CODE: explanation.{$lang} は metadata.explanation.{$lang} と一致する必要があります( expected='{$expected}' )。");
            }
        }
    }

    /**
     * question => metadata.question と一致
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
     * fields => CODE用 (存在すればチェック)
     * - is_correct => "boolean" / user_answer => "number"? / collect_answer => 数値?
     * - field_explanation => 全言語で空文字禁止
     */
    private function validateResponseFieldForCode($validator, array $field, int $idx)
    {
        // field_id
        if (empty($field['field_id']) || !is_string($field['field_id'])) {
            $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.field_id",
                "CODE: field_id は必須の文字列です。");
        }

        // user_answer => "number"のみ
        if (!empty($field['user_answer'])) {
            if (!in_array($field['user_answer'], self::FIELD_TYPE, true)) {
                $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.user_answer",
                    "CODE: user_answer='{$field['user_answer']}' は無効です。(例:'number')");
            }
        }

        // is_correct => "boolean"
        if (!empty($field['is_correct']) && $field['is_correct'] !== 'boolean') {
            $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.is_correct",
                "CODE: is_correct は 'boolean' のみ有効です。");
        }

        // collect_answer => 数値か
        if (($field['user_answer'] ?? '') === 'number') {
            if (!isset($field['collect_answer']) || !is_numeric($field['collect_answer'])) {
                $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.collect_answer",
                    "CODE: collect_answer は数値が必要です (user_answer='number')。");
            }
        }

        // field_explanation => LANGUAGES すべてで空文字禁止
        if (!empty($field['field_explanation']) && is_array($field['field_explanation'])) {
            foreach (self::LANGUAGES as $lang) {
                if (!isset($field['field_explanation'][$lang])
                    || !is_string($field['field_explanation'][$lang])
                    || trim($field['field_explanation'][$lang]) === ''
                ) {
                    $validator->errors()->add(
                        "evaluation_spec.response_format.fields.{$idx}.field_explanation.{$lang}",
                        "CODE: field_explanation.{$lang} は必須の文字列で、空文字は許可されません。"
                    );
                }
            }
        }
    }

    // ======================================================================
    // LLM用 response_format チェック
    // ======================================================================

    /**
     * LLM用 response_format を検証
     * - is_correct => "boolean", score => "number"
     * - question_text/explanation/question => langごとに 'text'
     * - fields => user_answer='number', collect_answer=数値, field_explanation=空文字禁止
     */
    private function validateResponseFormatForLlm($validator, array $resp)
    {
        // is_correct => "boolean"
        if (($resp['is_correct'] ?? '') !== 'boolean') {
            $validator->errors()->add('evaluation_spec.response_format.is_correct',
                "LLM: is_correct は 'boolean' のみ有効です。");
        }

        // score => "number"
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
     * LLM用 fields.* 検証
     * - field_id(文字列), user_answer="number", is_correct="boolean", collect_answer=数値, field_explanation=空文字禁止
     */
    private function validateResponseFieldForLlm($validator, array $field, int $idx)
    {
        // field_id
        if (empty($field['field_id']) || !is_string($field['field_id'])) {
            $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.field_id",
                "LLM: field_id は必須の文字列です。");
        }

        // user_answer => "number" のみ
        if (($field['user_answer'] ?? '') !== 'number') {
            $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.user_answer",
                "LLM: user_answer は 'number' のみ有効です。");
        }

        // is_correct => "boolean"
        if (($field['is_correct'] ?? '') !== 'boolean') {
            $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.is_correct",
                "LLM: is_correct は 'boolean' のみ有効です。");
        }

        // collect_answer => 数値が必要 (user_answer='number')
        if (($field['user_answer'] ?? '') === 'number') {
            if (!isset($field['collect_answer']) || !is_numeric($field['collect_answer'])) {
                $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.collect_answer",
                    "LLM: collect_answer は数値を指定してください (user_answer='number')。"
                );
            }
        }

        // field_explanation => 全言語で空文字禁止
        if (empty($field['field_explanation']) || !is_array($field['field_explanation'])) {
            $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.field_explanation",
                "LLM: field_explanation(オブジェクト) は必須です。");
        } else {
            foreach (self::LANGUAGES as $lang) {
                if (!isset($field['field_explanation'][$lang])
                    || !is_string($field['field_explanation'][$lang])
                    || trim($field['field_explanation'][$lang]) === ''
                ) {
                    $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.field_explanation.{$lang}",
                        "LLM: field_explanation.{$lang} は非空文字列が必須です。");
                }
            }
        }
    }

    // ======================================================================
    // metadata 検証
    // ======================================================================

    /**
     * metadata (question_type / question_text / explanation / background / input_format.*) の検証
     *
     * - question_type => Enum
     * - question_text, explanation, background => 各言語必須
     * - input_format.fields => attribute="number", user_answer="number"
     * - question_components => blank数=fields数
     */
    private function validateMetadata($validator, array $json)
    {
        $meta = $json['metadata'] ?? [];
        $qtRaw = $meta['question_type'] ?? null;

        // question_type => App\Enums\QuestionType に存在
        if ($qtRaw) {
            $str = (string)$qtRaw;
            try {
                QuestionType::fromString($str);
            } catch (\InvalidArgumentException) {
                $validator->errors()->add('metadata.question_type',
                    "question_type='{$str}' は未定義です。");
            }
        }

        // question_text, explanation, background は getTopLevelRulesで一部必須化済み => ここで特に追加ロジックは不要

        // input_format.fields => attribute='number', user_answer='number'
        $fieldsArr = $meta['input_format']['fields'] ?? [];
        if (is_array($fieldsArr)) {
            $fieldIds = [];
            foreach ($fieldsArr as $idx => $f) {
                // field_id => f_x
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
                if (!isset($f['attribute']) || !in_array($f['attribute'], self::FIELD_TYPE, true)) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.attribute",
                        "attribute='{$f['attribute']}' は 'number' のみ有効です。");
                }

                // user_answer => "number" の文字列が必須
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

        // question_components => blank数=fields数
        $comps = $meta['input_format']['question_components'] ?? [];
        if (is_array($comps)) {
            $blankCount = 0;
            $orders = [];
            foreach ($comps as $cidx => $comp) {
                $ctype = $comp['type'] ?? '';
                if (!in_array($ctype, self::COMPONENT_TYPES, true)) {
                    $validator->errors()->add("metadata.input_format.question_components.{$cidx}.type",
                        "不正なコンポーネントtype='{$ctype}'です。");
                }
                if ($ctype === 'blank') {
                    $blankCount++;
                    if (!isset($comp['field_id'])) {
                        $validator->errors()->add("metadata.input_format.question_components.{$cidx}.field_id",
                            "type=blank の場合 field_id が必須です。");
                    }
                } else {
                    // text,image,movie => content.{ja,en} 必須
                    if (empty($comp['content']) || !is_array($comp['content'])) {
                        $validator->errors()->add("metadata.input_format.question_components.{$cidx}.content",
                            "type='{$ctype}' の場合 content(オブジェクト) が必須です。");
                    } else {
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

                // order => 数値 + 重複
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

            // blank数 と fields数
            $fieldsCount = count($fieldsArr);
            if ($blankCount !== $fieldsCount) {
                $validator->errors()->add("metadata.input_format.question_components",
                    "blank要素数({$blankCount}) と fields数({$fieldsCount}) が一致しません。");
            }
        }
    }

    // --------------------------------------------------------------------
    //  多言語メタデータをローカライズするサンプルメソッド（必要なら使用）
    // --------------------------------------------------------------------
    public function localizeMetadata(array $metadata): array
    {
        $locale = App::getLocale();

        // question_text => {ja:"...", en:"..."} => 1文言へ
        if (isset($metadata['question_text']) && is_array($metadata['question_text'])) {
            $metadata['question_text'] = $metadata['question_text'][$locale] ?? '';
        }
        if (isset($metadata['explanation']) && is_array($metadata['explanation'])) {
            $metadata['explanation']   = $metadata['explanation'][$locale] ?? '';
        }
        // TODO 暫定処理。background 学習の背景でありユーザーに見える必要はないのでレスポンスしない。ここで消す処理にするよりもmetadata用のResourceを用意してそちらでレスポンスを定義するべき
        if (isset($metadata['background']) && is_array($metadata['background'])) {
            unset($metadata['background']);
        }

//        if (isset($metadata['background']) && is_array($metadata['background'])) {
//            $metadata['background']    = $metadata['background'][$locale] ?? '';
//        }

        // question => type="text" => content を langごとの文字列に
        if (isset($metadata['question']) && is_array($metadata['question'])) {
            $metadata['question'] = $metadata['question'][$locale] ?? '';
        }

        // question_components => type="text" => content を langごとの文字列に
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
}


```
