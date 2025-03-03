
APIの問題のレスポンスの metadata を多言語に対応したい。



"question_type": "FILL_IN_THE_BLANK", の場合は以下に固定。
"type"が"text"の場合は、content は必ず多言語化されたオブジェクトになる。


ーー言語定数
ja, en

--input_format type 定数
fixed: 固定。ユーザーは項目の増減はコントロール出来ず、input_format.fields の内容に固定
custom: input_format.fields の内容はあくまでもデフォルト表記で、固定されず、ユーザー自由に回答の数を増減出来る。

-- input_format.fields.type定数
number: 数値型

-- input_format.question_components.type定数
text: テキスト
image: 画像
movie: 動画
blank: 空欄。入力項目

ーーMetadata オブジェクト仕様
question_type：必須、App\Enums\QuestionType.php に値が存在しているか。JSONには、文字列でも数値でもどちらでも入力可にする

question ：必須、オブジェクト（言語定数全て含んでいるか）。問題文

input_format: 必須、オブジェクト（言語定数全て含んでいるか）
input_format.type: 必須、input_format type 定数と値が一致しているか。
input_format.fields: 必須、配列。ユーザが回答する入力フォームの仕様を定義
input_format.fields.field_id: 必須、f_x のフォーマットになっているか。同じ fields 内に重複した値が存在しないか。 question_components内の type: "blank"の数と総数が合っているか。
input_format.fields.type: 必須、（input_format.fields.type定数に値が存在しているか）。ブランクのフォームの属性。例えば number であれば、<input type="number">になる
input_format.fields.collect_answer: 必須、問題の正解（ユーザーには隠す）

input_format.question_components: 必須（問題を更生する要素）
input_format.question_components.type：必須、input_format.question_components.type定数と値が合っているか
input_format.question_components.content：必須、オブジェクト、言語定数と一致する値が全て含まれているか
input_format.question_components.order: 必須、数値、重複する値が存在しないこと。問題を構築するときの表示順番

--- metadataのJSON
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
				"type": "number",
				"collect_answer": "13"
			},
			{
				"field_id": "f_2",
				"type": "number",
				"collect_answer": "73"
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

＝＝＝
<?php

namespace App\Services\V1\StudySession;

use App\Dtos\V1\User\UserQuestionDto;
use App\Dtos\V1\User\UserQuestionSetDto;
use App\Dtos\V1\Question\QuestionSetDto;
use App\Dtos\V1\Question\QuestionDto;
use App\Enums\UserQuestionSetStatus;
use App\Enums\UserQuestionStatus;
use App\Models\Question\Question;
use App\Models\Question\QuestionSet;
use App\Models\Question\QuestionSetQuestion;
use App\Models\User\UserQuestion;
use App\Models\User\UserQuestionSet;
use App\Models\User\UserQuestionSetTranslation;
use Carbon\Carbon;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Str;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

class StudySessionService
{
    /**
     * question_set_id を直接受け取り、user_question_sets (NOT_START or PROGRESS) があればそれを返却、
     * なければ作成し、該当 question_set に紐づく question を user_questions として生成する
     */
    public function getOrCreateUserQuestionSetByQuestionSetId(string $userId, string $questionSetId): UserQuestionSetDto
    {
        // question_set_id をキーに question_sets を取得
        $questionSet = QuestionSet::with([
            'translations',
            'questionSetQuestions' => function($q) {
                $q->orderBy('order','asc');
            },
            'questionSetQuestions.question.translations'
        ])
            ->where('id', $questionSetId)
            ->first();

        if (!$questionSet) {
            // TODO エラーメッセージ多言語化
            throw new NotFoundHttpException("QuestionSet not found");
        }

        // 既存の user_question_set (NOT_START or PROGRESS) があるか確認
        $existing = UserQuestionSet::where('user_id', $userId)
            ->where('question_set_id', $questionSet->id)
            ->whereIn('status', [
                UserQuestionSetStatus::NOT_START->value,
                UserQuestionSetStatus::PROGRESS->value
            ])
            ->first();

        if ($existing) {
            // DTOに変換して返す (question_set を翻訳付きでネスト)
            return $this->toUserQuestionSetDto($existing, $questionSet);
        }

        // なければ新規作成 → user_questions 生成
        DB::beginTransaction();
        try {
            // -------------------------
            // user_question_sets の作成
            // -------------------------
            $userQuestionSet = new UserQuestionSet();
            $userQuestionSet->id = (string) Str::uuid();
            $userQuestionSet->user_id = $userId;
            $userQuestionSet->question_set_id = $questionSet->id;
            $userQuestionSet->status = UserQuestionSetStatus::NOT_START->value;
            $userQuestionSet->score = null;
            $userQuestionSet->started_at = null;
            $userQuestionSet->finished_at = null;
            $userQuestionSet->save();

            // user_question_set_translations へコピー
            //     questionSet->translations の全locale/title/descriptionを複製
            foreach ($questionSet->translations as $qsTrans) {
                UserQuestionSetTranslation::create([
                    'user_question_set_id' => $userQuestionSet->id,
                    'locale'      => $qsTrans->locale,
                    'title'       => $qsTrans->title,
                    'description' => $qsTrans->description,
                ]);
            }


            // user_questions を作成
            //     question_set_questions より:
            //       -> question_id, order
            //       -> question の各種フィールドを user_questions にコピー
            // question_set_questions から question_id を取得し、user_questions 作成
            foreach ($questionSet->questionSetQuestions as $qSq) {
                $question = $qSq->question;
                if (!$question) {
                    continue;
                }

                // user_questions レコード作成
                $userQuestion = new UserQuestion();
                $userQuestion->id = (string) Str::uuid();
                $userQuestion->user_question_set_id = $userQuestionSet->id;
                $userQuestion->question_id = $question->id;
                $userQuestion->status = UserQuestionStatus::NOT_START->value;
                $userQuestion->answer_data = null;
                $userQuestion->answered_at = null;

                // question のフィールドをコピー
                $userQuestion->metadata = $question->metadata;
                $userQuestion->version = $question->version;
                $userQuestion->evaluation_method = $question->evaluation_method;
                $userQuestion->checker_method = $question->checker_method;
                $userQuestion->llm_evaluation_prompt_number = $question->llm_evaluation_prompt_number;
                $userQuestion->llm_evaluation_response_format = $question->llm_evaluation_response_format;
                $userQuestion->question_type = $question->question_type;
                $userQuestion->learning_requirement_json = $question->learning_requirement_json;
                $userQuestion->learning_subject = $question->learning_subject;
                $userQuestion->learning_no = $question->learning_no;
                $userQuestion->learning_requirement = $question->learning_requirement;
                $userQuestion->learning_required_competency = $question->learning_required_competency;
                $userQuestion->learning_background = $question->learning_background;
                $userQuestion->learning_category = $question->learning_category;
                $userQuestion->learning_grade_level = $question->learning_grade_level;
                $userQuestion->learning_url = $question->learning_url;
                $userQuestion->generated_by_llm = $question->generated_by_llm;

                // pivotテーブルのorderを userQuestionにコピー
                $userQuestion->order = $qSq->order;

                $userQuestion->save();
            }
            DB::commit();

            return $this->toUserQuestionSetDto($userQuestionSet, $questionSet);
        } catch (\Throwable $e) {
            DB::rollBack();
            throw $e;
        }
    }

    /**
     * user_question_sets を PROGRESS にし、最初の user_question (order 昇順) をDTOで返す
     * 返却の際に question の翻訳をネストする
     */
    public function beginUserQuestionSet(string $userId, string $userQuestionSetId): UserQuestionDto
    {
        $userQuestionSet = UserQuestionSet::where('id', $userQuestionSetId)
            ->where('user_id', $userId)
            ->first();

        // TODO エラーメッセージの多言語化
        if (!$userQuestionSet) {
            throw new NotFoundHttpException("Question data not found ({$userQuestionSetId}).");
        }

        // 1) まず status=PROGRESS の中で orderが最小のものを検索
        $progressQuestion = UserQuestion::where('user_question_set_id', $userQuestionSet->id)
            ->where('status', UserQuestionStatus::PROGRESS->value)
            ->orderBy('order', 'asc')
            ->first();

        // 2) なければ status=NOT_START の中で orderが最小のものを検索
        $firstUserQuestion = $progressQuestion;
        if (!$progressQuestion) {
            $firstUserQuestion = UserQuestion::where('user_question_set_id', $userQuestionSet->id)
                ->where('status', UserQuestionStatus::NOT_START->value)
                ->orderBy('order', 'asc')
                ->first();
        }

        // TODO エラーメッセージの多言語化
        if (!$firstUserQuestion) {
            throw new NotFoundHttpException("No question found ({$userQuestionSetId}).");
        }

        // 問題がまだ開始ではない場合に回答中ステータスに更新する
        if ($firstUserQuestion->status = UserQuestionStatus::NOT_START->value) {
            $firstUserQuestion->status = UserQuestionStatus::PROGRESS->value;
            $firstUserQuestion->save();
        }


        // ステータスを PROGRESS に変更
        if ($userQuestionSet->status !== UserQuestionSetStatus::PROGRESS->value) {
            $userQuestionSet->status = UserQuestionSetStatus::PROGRESS->value;
            $userQuestionSet->started_at = Carbon::now();
            $userQuestionSet->save();
        }

        // 取得した user_questions レコードを元に DTO 生成
        // user_questions テーブル上の question_id を使って question を取得 → 翻訳を適用
        $questionId = $firstUserQuestion->question_id;
        $questionModel = Question::find($questionId);

        $questionDto = null;
        if ($questionModel) {
            $translation = $questionModel->translations->firstWhere('locale', app()->getLocale())
                ?: $questionModel->translations->firstWhere('locale', config('app.fallback_locale'));

            $questionDto = new QuestionDto([
                'id'          => (string) $questionModel->id,
                'question_text'       => $translation?->question_text ?? '',
                'explanation' => $translation?->explanation ?? '',
                'metadata' => $questionModel->metadata ?? '',
                'version' => $questionModel->version ?? '',
                'question_type' => $questionModel->question_type ?? '',
            ]);
        }

        // UserQuestionDto を返す
        return new UserQuestionDto([
            'id'          => (string) $firstUserQuestion->id,
            'question_id' => (string) $firstUserQuestion->question_id,
            'status'      => (int) $firstUserQuestion->status,
            'answer_data' => $firstUserQuestion->answer_data,
            'answered_at' => $firstUserQuestion->answered_at,
            'question'    => $questionDto,
        ]);
    }

    /**
     * UserQuestionSet + QuestionSet を翻訳込みで DTO へ変換
     */
    private function toUserQuestionSetDto(UserQuestionSet $userQuestionSet, QuestionSet $questionSet): UserQuestionSetDto
    {
        // question_set の翻訳を取得
        $qsTranslation = $questionSet->translations->firstWhere('locale', app()->getLocale())
            ?: $questionSet->translations->firstWhere('locale', config('app.fallback_locale'));

        $questionSetDto = new QuestionSetDto([
            'id'          => (string) $questionSet->id,
            'unit_id'     => (string) $questionSet->unit_id,
            'title'       => $qsTranslation?->title ?? '',
            'description' => $qsTranslation?->description ?? '',
            'status'      => (int) $questionSet->status,
            'order'       => (int) $questionSet->order,
        ]);

        return new UserQuestionSetDto([
            'id'            => (string) $userQuestionSet->id,
            'user_id'       => (string) $userQuestionSet->user_id,
            'question_set_id' => (string) $userQuestionSet->question_set_id,
            'status'        => (int) $userQuestionSet->status,
            'score'         => $userQuestionSet->score ? (float) $userQuestionSet->score : null,
            'started_at'    => $userQuestionSet->started_at,
            'finished_at'   => $userQuestionSet->finished_at,
            'question_set'  => $questionSetDto,
        ]);
    }
}
