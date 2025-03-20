user_question_sets の生成時に、未回答の問題や、問題の数を１０問までなど数を制限するように修正したい。

# 前提条件
question_sets：問題（questions）を束ねるグループ
questions: 問題
question_set_questions：question_sets と questions を紐づけるPivotテーブル。questions は複数のquestion_sets に紐づく事があり多対多の関係なのでこのような設計になっている。
user_question_sets: ユーザーが学習した questions_sets。学習開始時に question_sets_id と紐づいて status （UserQuestionSetsStatus::NOT_START) が未開始の状態で生成され、進捗ステータスはスコアなどが管理される
user_questions: ユーザーが学習した questions。学習開始時に、学習を開始した question_sets に紐づく questions が全て、 user_question_sets_id と question_id（questions）を紐づけて status （UserQuestionStatus::NOT_START) が未開始の状態で全てのquestionsの数の分、生成され、進捗ステータスはスコアなどが管理される


未回答の問題の定義：
user_questions に id がコピーされていない（存在しない）questions
または、user_questions に id がコピーされている（存在する）が、status NOT_START のもの

コピーする（回答に挑戦する）数を定義する。例えば、10問。

# 条件
１）まだ、回答したことの無い問題、かつ status PUBLISHED の、問題を order 昇順で先頭から
何問にするかは後で、ユーザーによって変更するなど設定にバラエイティを持たせるために定数に切り出して管理するようにして

２）もし未回答の問題の総数が、コピーする（回答に挑戦する）数の定義（例えば10問）に達しない場合は、
足しなかった分は、すでに user_questions にコピー済み（status NOT_STARTも含む）、から status PUBLISHED の問題をランダムでコピーするようにしてください。

３）それでも件数に満たない場合は、しょうがないので満たない件数で処理を完了する

# 実装方針
生成また修正変更するコードはそのコードだけは必ず省略せずに必ず全てアウトプットすること。

修正する場合は修正したコードは全て出力し、修正箇所以外のコードやコメントは現状のままにしてください。

仕様はソースコードにまとめてコメントとして残すこと
コメントを残す時に以下のようにコメントへ処理順を示すようなナンバリングは不用です。
// 7) memo => 空白
// 8) generate_question_prompt => そのまま
// 9) generate_question_prompt_file_name => そのまま

ーーーー
```php
<?php

namespace App\Enums;

enum UserQuestionStatus: int
{
    case NOT_START = 1;
    case CORRECT = 50;
    case INCORRECT = 100;
    case SKIP = 150;
    case PROGRESS = 200;

    public function description(): string
    {
        return match ($this) {
            self::NOT_START => 'Not Started',
            self::CORRECT => 'Correct',
            self::INCORRECT => 'Incorrect',
            self::SKIP => 'Skip',
            self::PROGRESS => 'Progress',
        };
    }

    public static function getKeyForValue(int $value): ?string
    {
        foreach (UserQuestionStatus::cases() as $case) {
            if ($case->value === $value) {
                return $case->name;
            }
        }

        return null;
    }

    /**
     * 文字列から QuestionStatus を取得
     * @param string $statusString 例: "DRAFT", "PUBLISHED" など
     * @return self
     */
    public static function fromString(string $statusString): self
    {
        return match (strtoupper($statusString)) {
            'NOT_START'          => self::NOT_START,
            'CORRECT'      => self::CORRECT,
            'INCORRECT'         => self::INCORRECT,
            'SKIP' => self::SKIP,
            'PROGRESS' => self::PROGRESS,
            default => throw new \InvalidArgumentException("Unknown status string: {$statusString}")
        };
    }
}

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

namespace App\Services\V1\StudySession;

use App\Dtos\V1\User\UserQuestionDto;
use App\Dtos\V1\User\UserQuestionSetDto;
use App\Dtos\V1\Question\QuestionSetDto;
use App\Dtos\V1\Question\QuestionDto;
use App\Enums\UserQuestionSetStatus;
use App\Enums\UserQuestionStatus;
use App\Models\Question\Question;
use App\Models\Question\QuestionSet;
use App\Models\User\UserQuestion;
use App\Models\User\UserQuestionSet;
use App\Models\User\UserQuestionSetTranslation;
use App\Models\User\UserQuestionTranslation;
use App\Services\Utils\Question\QuestionJsonManageService;
use Carbon\Carbon;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Str;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

/**
 * Class StudySessionService
 *
 *

【概要】
 *   ユーザーの学習セッション(問題セットの進行状況)を管理するサービスクラス。
 *   - user_question_sets, user_questions の作成・更新ロジックをまとめる。
 *   - beginUserQuestionSet などでUserQuestionを開始状態に変更し、現在の問題を返す。
 */
class StudySessionService
{
    /**
     * @var QuestionJsonManageService $questionJsonService
     */
    private QuestionJsonManageService $questionJsonService;

    public function __construct(QuestionJsonManageService $questionJsonService)
    {
        $this->questionJsonService = $questionJsonService;
    }

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
            foreach ($questionSet->translations as $qsTrans) {
                UserQuestionSetTranslation::create([
                    'user_question_set_id' => $userQuestionSet->id,
                    'locale'      => $qsTrans->locale,
                    'title'       => $qsTrans->title,
                    'description' => $qsTrans->description,
                ]);
            }

            // user_questions を作成
            foreach ($questionSet->questionSetQuestions as $qSq) {
                $question = $qSq->question;
                if (!$question) {
                    continue;
                }

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
                $userQuestion->llm_evaluation_prompt_file_name = $question->llm_evaluation_prompt_file_name;
                $userQuestion->evaluation_response_format = $question->evaluation_response_format;
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

                // user_question_translations へコピー
                foreach ($question->translations as $qTrans) {
                    UserQuestionTranslation::create([
                        'user_question_id' => $userQuestion->id,
                        'locale'      => $qTrans->locale,
                        'question_text'   => $qTrans->question_text,
                        'explanation'     => $qTrans->explanation,
                        'background'      => $qTrans->background,
                    ]);
                }
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

        if (!$userQuestionSet) {
            throw new NotFoundHttpException("Session data not found.");
        }

        // 1) status=PROGRESS の中で orderが最小のもの
        $progressQuestion = UserQuestion::where('user_question_set_id', $userQuestionSet->id)
            ->where('status', UserQuestionStatus::PROGRESS->value)
            ->orderBy('order', 'asc')
            ->first();

        // なければ status=NOT_START の中で orderが最小のもの
        $firstUserQuestion = $progressQuestion;
        if (!$progressQuestion) {
            $firstUserQuestion = UserQuestion::where('user_question_set_id', $userQuestionSet->id)
                ->where('status', UserQuestionStatus::NOT_START->value)
                ->orderBy('order', 'asc')
                ->first();
        }

        if (!$firstUserQuestion) {
            throw new NotFoundHttpException("No question found ({$userQuestionSetId}).");
        }

        if ($firstUserQuestion->status == UserQuestionStatus::NOT_START->value) {
            $firstUserQuestion->status = UserQuestionStatus::PROGRESS->value;
            $firstUserQuestion->save();
        }

        // user_question_sets を PROGRESS に
        if ($userQuestionSet->status !== UserQuestionSetStatus::PROGRESS->value) {
            $userQuestionSet->status = UserQuestionSetStatus::PROGRESS->value;
            $userQuestionSet->started_at = Carbon::now();
            $userQuestionSet->save();
        }

        // question の翻訳を取得
        $questionId = $firstUserQuestion->question_id;
        $questionModel = Question::find($questionId);

        $questionDto = null;
        if ($questionModel) {
            $translation = $questionModel->translations->firstWhere('locale', app()->getLocale())
                ?: $questionModel->translations->firstWhere('locale', config('app.fallback_locale'));

            // metadata をロケールに応じて差し替え
            $metadataArray = [];
            if ($questionModel->metadata) {
                $decoded = json_decode($questionModel->metadata, true) ?: [];
                $decodedLocalized = $this->questionJsonService->localizeMetadata($decoded);
                $metadataArray = $decodedLocalized;
            }
            $localizedMetadataJson = json_encode($metadataArray, JSON_UNESCAPED_UNICODE);

            $questionDto = new QuestionDto([
                'id'             => (string) $questionModel->id,
                'question_text'  => $translation?->question_text ?? '',
                'explanation'    => $translation?->explanation ?? '',
                'metadata'       => $localizedMetadataJson,
                'version'        => $questionModel->version ?? '',
                'question_type'  => $questionModel->question_type ?? '',
            ]);
        }

        $totalQuestionsCount = $userQuestionSet->userQuestions()->count();
        $remainingCount = $userQuestionSet->userQuestions()
            ->where('status', UserQuestionStatus::NOT_START->value)
            ->count();

        return new UserQuestionDto([
            'id'          => (string) $firstUserQuestion->id,
            'question_id' => (string) $firstUserQuestion->question_id,
            'status'      => (int) $firstUserQuestion->status,
            'answer_data' => $firstUserQuestion->answer_data,
            'answered_at' => $firstUserQuestion->answered_at,

            'total_questions_count' => $totalQuestionsCount,
            'remaining_questions_count' => $remainingCount,

            'question'    => $questionDto,
        ]);
    }

    /**
     * UserQuestionSet + QuestionSet を翻訳込みで DTO へ変換
     */
    private function toUserQuestionSetDto(UserQuestionSet $userQuestionSet, QuestionSet $questionSet): UserQuestionSetDto
    {
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

```
