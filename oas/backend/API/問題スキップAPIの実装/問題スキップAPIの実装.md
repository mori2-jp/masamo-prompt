
UserQuestionSet の ステータスをSKIPにするAPIを実装したい。
API のエンドポイントは、
/api/v1/user-question-sets/{user_question_set_id}/skip

回答済みの場合は既に回答済みとレスポンスする。
レスポンスする文字列は多言語にする。

{# 現在の実装} を参考に実装してください

今回は、テストコードは生成不用です

# 実装方針


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
unit: 単元
question_set_questions：question_sets と questions を紐づけるPivotテーブル。questions は複数のquestion_sets に紐づく事があり多対多の関係なのでこのような設計になっている。
user_question_sets: ユーザーが学習した questions_sets。学習開始時に question_sets_id と紐づいて status （UserQuestionSetsStatus::NOT_START) が未開始の状態で生成され、進捗ステータスはスコアなどが管理される
user_questions: ユーザーが学習した questions。学習開始時に、学習を開始した question_sets に紐づく questions が全て、 user_question_sets_id と question_id（questions）を紐づけて status （UserQuestionStatus::NOT_START) が未開始の状態で全てのquestionsの数の分、生成され、進捗ステータスはスコアなどが管理される

ヘッダーに、設定されたAccept-Languageによってい、question_set_translataions から値を取得してレスポンスします。

ーー 現在の実装
```php
<?php

namespace App\Enums;

enum UserQuestionSetStatus: int
{
    case NOT_START = 1;
    case COMPLETE = 100;
    case PROGRESS = 300;
    case SKIP = 500;

    public function description(): string
    {
        return match ($this) {
            self::NOT_START => 'Not Started',
            self::COMPLETE => 'Complete',
            self::PROGRESS => 'Progress',
            self::SKIP => 'Skip',
        };
    }

    public static function getKeyForValue(int $value): ?string
    {
        foreach (UserQuestionSetStatus::cases() as $case) {
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
            'COMPLETE'      => self::COMPLETE,
            'PROGRESS'         => self::PROGRESS,
            'SKIP'         => self::SKIP,
            default => throw new \InvalidArgumentException("Unknown status string: {$statusString}")
        };
    }
}
<?php

namespace App\Models\User;

use App\Enums\UserQuestionStatus;
use App\Models\BaseModel;
use App\Models\Question\QuestionSet;
use App\Traits\UsesUuid;
use Illuminate\Database\Eloquent\SoftDeletes;

/**
 *
 *
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet onlyTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet query()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet withTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet withoutTrashed()
 * @property string $id
 * @property string $user_id
 * @property string $question_set_id
 * @property string|null $score
 * @property string|null $question_link_token
 * @property string|null $expires_at_question_link_token
 * @property string $started_at
 * @property string|null $finished_at
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property \Illuminate\Support\Carbon|null $deleted_at
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereCreatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereDeletedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereFinishedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereQuestionSetId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereScore($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereStartedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereUpdatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereUserId($value)
 * @property int $status
 * @property-read QuestionSet $questionSet
 * @property-read \App\Models\User\User $user
 * @property-read \Illuminate\Database\Eloquent\Collection<int, \App\Models\User\UserQuestion> $userQuestions
 * @property-read int|null $user_questions_count
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereStatus($value)
 * @property-read \Illuminate\Database\Eloquent\Collection<int, \App\Models\User\UserQuestionSetTranslation> $translations
 * @property-read int|null $translations_count
 * @mixin \Eloquent
 */
class UserQuestionSet extends BaseModel
{
    use UsesUuid, SoftDeletes;

    protected $with = [
        'translations',
    ];

    protected array $iso8601Timestamps = [
        'started_at',
        'finished_at',
    ];

    /*=====================================================
    * リレーション
    *=====================================================*/
    public function questionSet()
    {
        return $this->belongsTo(QuestionSet::class);
    }

    public function userQuestions()
    {
        return $this->hasMany(UserQuestion::class);
    }

    public function user()
    {
        return $this->belongsTo(User::class);
    }

    /**
     * 翻訳テーブルとのリレーション
     */
    public function translations()
    {
        return $this->hasMany(UserQuestionSetTranslation::class, 'user_question_set_id');
    }


    /*=====================================================
     * カスタムメソッド/Custom Method
    *=====================================================*/

    /**
     * 現在のlocaleに応じた翻訳を返す
     * なければfallback_localeの翻訳を返す
     */
    public function getTranslation(?string $locale = null): ?UserQuestionSetTranslation
    {
        $locale = $locale ?: app()->getLocale();
        return $this->translations->firstWhere('locale', $locale)
            ?: $this->translations->firstWhere('locale', config('app.fallback_locale'));
    }

    /**
     * 次の未開始問題 (status=NOT_START) を order 昇順 で1件取得して返す
     * なければ null
     * @return UserQuestion|null
     */
    public function getNextNotStartedQuestion(): ?UserQuestion
    {
        return $this->userQuestions()
            ->where('status', UserQuestionStatus::NOT_START->value)
            ->orderBy('order', 'asc')
            ->first();
    }

}
<?php

namespace App\Services\V1\UserQuestionSet;

use App\Enums\QuestionStatus;
use App\Enums\SectionProgressMode;
use App\Enums\UserQuestionSetStatus;
use App\Enums\UserQuestionStatus;
use App\Helpers\CommonLib;
use App\Models\Question\Question;
use App\Models\Question\QuestionSet;
use App\Models\Section\Section;
use App\Models\Unit\Unit;
use App\Models\User\User;
use App\Models\User\UserQuestion;
use App\Models\User\UserQuestionSet;
use App\Models\User\UserQuestionSetTranslation;
use App\Models\User\UserQuestionTranslation;
use App\Services\Utils\Question\QuestionJsonManageService;
use App\Services\V1\Auth\MeService;
use Illuminate\Support\Collection;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Str;

/**
 * Class StudySessionService
 *
 * 【概要】
 *   ユーザーの学習セッション(問題セットの進行状況)を管理するサービスクラス。
 *   - user_question_sets, user_questions の作成・更新ロジックをまとめる。
 *   - user_question_sets 生成時に「未回答の問題を最大10問」だけ user_questions にコピーする
 *   - 「未回答の問題」= まだ一度も user_questions に存在しない or status=NOT_START のもの。
 *   - もし未回答の問題が10問に満たない場合は「既に user_questions に存在していて、status=NOT_START で、かつ PUBLISHED の問題」をランダムに補充
 *   - MAX_QUESTIONS_TO_COPY は定数として管理。
 */
class UserQuestionSetService
{
    private MeService $meService;

    public function __construct(
        MeService $meService,
    )
    {
        $this->meService = $meService;
    }

    // TODO ユーザが回答に挑戦する最大数を定義（将来はConfigやDBなど外部管理可能に）
    // 連続QuestionSet成功数
    private const SUCCESS_STREAK = 3;
    // MASTER正答率(%)
    private const TH_PASS = 80;
    // DOWN GRADE正答率(%)
    private const GRADE_DOWN_THRESHOLD = 50;
    private const MAX_QUESTIONS_TO_COPY = 10;

    /**
     * 以下の順で、判定を行う。
     * 1. statusがPROGRESSのuser_question_set がある。
     * user_question_setがなければ作成し、該当 question_set に紐づく question を user_questions として生成する
     */
    public function getCurrentUserQuestionSet(string $userId): UserQuestionSet
    {
        // statusがPROGRESS, NOT_STARTの UserQuestionSet を取得
        $userQuestionSet = UserQuestionSet::where('user_id', $userId)
            ->whereIn('status', [
                UserQuestionSetStatus::PROGRESS->value,
                UserQuestionSetStatus::NOT_START->value,
            ])
            ->first();

        if (!$userQuestionSet) {
            $user = User::find($userId);
            // Sampleの解答率
            $sampleUserQuestionSets = $this->getUserQuestionSetsWithSection($userId);
            $sampleCorrectRate = $this->getAverageCorrectRateFromUserQuestionSets($sampleUserQuestionSets);
            // 次の問題のグレード
            $nextSet = $this->decideSectionProgression($sampleCorrectRate, $sampleUserQuestionSets->count());

            $lastQuestionSet = $sampleUserQuestionSets->first()?->questionSet;
            $lastSampleOrder = $lastQuestionSet ? $lastQuestionSet->order : null;
            $unitId = $lastQuestionSet ? $lastQuestionSet->unit_id : null;
            $sectionId = $lastQuestionSet ? $lastQuestionSet->unit->section_id : null;

            // 次のQuestionSet を取得
            if (!$sampleUserQuestionSets->count() || $nextSet['mode'] === SectionProgressMode::MASTERED) {
                $nextQuestionSet = $this->getNextQuestionSet($user->grade_id, $sectionId, $unitId, $lastSampleOrder);
            } else {
                $nextQuestionSet = $lastQuestionSet;
            }
            DB::beginTransaction();
            try {
                $userQuestionSet = new UserQuestionSet();
                $userQuestionSet->uuid = (string) Str::uuid();
                $userQuestionSet->user_id = $userId;
                $userQuestionSet->question_set_id = $nextQuestionSet->id;
                $userQuestionSet->status = UserQuestionSetStatus::NOT_START->value;
                $userQuestionSet->score = null;
                $userQuestionSet->started_at = null;
                $userQuestionSet->finished_at = null;
                $userQuestionSet->save();

                // user_question_set_translations へコピー
                foreach ($nextQuestionSet->translations as $qsTrans) {
                    UserQuestionSetTranslation::create([
                        'user_question_set_id' => $userQuestionSet->id,
                        'locale'      => $qsTrans->locale,
                        'title'       => $qsTrans->title,
                        'description' => $qsTrans->description,
                    ]);
                }

                $this->createLimitedUserQuestions($userQuestionSet, $nextQuestionSet);

                DB::commit();
                return $userQuestionSet;
            } catch (\Throwable $e) {
                DB::rollBack();
                throw $e;
            }
        }
        return  $userQuestionSet;
    }

    /**
     * UserQuestionSet 正答率判定
     * @param Collection $userQuestions
     * @return int
     */
    private function calcCorrectAnswerRate(Collection $userQuestions): int
    {
        $total = $userQuestions->count();

        if ($total === 0) {
            return 0; // 問題がないなら0%
        }

        $correctCount = $userQuestions->filter(function ($question) {
            return $question->status === UserQuestionStatus::CORRECT->value;
        })->count();

        $rate = ($correctCount / $total) * 100;

        return round($rate);
    }

    /**
     * UserQuestionSetsから平均正答率を算出
     *
     * @param \Illuminate\Support\Collection $userQuestionSets
     * @return float|null
     */
    private function getAverageCorrectRateFromUserQuestionSets(Collection $userQuestionSets)
    {
        if ($userQuestionSets->isEmpty()) {
            return null;
        }

        $rates = $userQuestionSets->map(function ($userQuestionSet) {
            return $this->calcCorrectAnswerRate($userQuestionSet->userQuestions);
        });

        return $rates->avg();
    }

    /**
     * UserQuestionSetを複数取得
     * @param string $userId
     * @param int $count
     * @return \Illuminate\Database\Eloquent\Collection|Collection
     */
    private function getUserQuestionSetsWithSection(string $userId)
    {
        // 1. 一番最初のUserQuestionSetを取得
        $firstUserQuestionSet = UserQuestionSet::where('user_id', $userId)
            ->orderBy('id', 'desc')
            ->first();

        if (!$firstUserQuestionSet) {
            return collect(); // もし最初のデータがなければ空コレクションを返す
        }

        $questionSetId = $firstUserQuestionSet->question_set_id;

        // 2. 同じquestion_set_idを持つUserQuestionSetを$count分取得
        return UserQuestionSet::with('userQuestions', 'questionSet.unit.section')
            ->where('user_id', $userId)
            ->where('question_set_id', $questionSetId)
            ->orderBy('id', 'desc')
            ->get();
    }


    // user_question_sets を作成する際に、最大10問のルールに従って user_questions を生成
    private function createLimitedUserQuestions(UserQuestionSet $userQuestionSet, QuestionSet $questionSet): void
    {
        // 「未回答の問題」:
        //    - question.generated_by_llm などは関係なく
        //    - question.status = PUBLISHED のみ
        //    - まだ user_questions に存在しない or すでに user_questions にあるが status=NOT_START
        //      (「一度回答した」= user_questions.status != NOT_START)
        //    - question_set_questions に紐づいていること
        $allPivot = $questionSet->questionSetQuestions()
            ->orderBy('order', 'asc')
            ->orderBy('id', 'asc')
            ->with('question')
            ->get();

        // まだ回答していない or NOT_START の question_id 一覧を order順で取得
        $unansweredList = [];
        foreach ($allPivot as $p) {
            $q = $p->question;
            if (!$q) continue;

            // question が PUBLISHED 以外は対象外
            if (in_array(config('app.env'), ['local', 'staging'])) {
                // local / staging の場合
                if ($q->status !== QuestionStatus::PUBLISHED->value && $q->status !== QuestionStatus::TEST_PUBLISHED->value) {
                    continue;
                }
            } else {
                // それ以外の環境の場合
                if ($q->status !== QuestionStatus::PUBLISHED->value) {
                    continue;
                }
            }
            // user_questionsで回答したことがあるかどうか
            $uq = UserQuestion::where('question_id', $q->id)
                ->where('user_question_set_id', $userQuestionSet->id)
                ->first();

            // まだ user_questions に無い、あるいは status=NOT_START
            if (!$uq) {
                // 「初めてコピーする問題」
                $unansweredList[] = $q;
            } else {
                // すでにあるが NOT_START なら対象
                if ($uq->status === UserQuestionStatus::NOT_START->value) {
                    $unansweredList[] = $q;
                }
            }
        }

        //  $unansweredList を最大 self::MAX_QUESTIONS_TO_COPY 件だけ取り出し
        $needCount = self::MAX_QUESTIONS_TO_COPY;
        $chosenUnanswered = array_slice($unansweredList, 0, $needCount);
        $countUnanswered  = count($chosenUnanswered);

        //  もし足りなければ (10問に満たなければ)、「すでに user_questions にコピー済みで status=NOT_START のもの」かつ PUBLISHED をランダムで補充
        $deficit = $needCount - $countUnanswered;
        $additionalChosen = [];
        if ($deficit > 0) {
            // すでに user_questions にコピー済み (この user_question_set で)
            // status=NOT_START && question.status=PUBLISHED の question をランダム抽出
            $alreadyNotStart = UserQuestion::where('user_question_set_id', $userQuestionSet->id)
                ->where('status', UserQuestionStatus::NOT_START->value)
                ->pluck('question_id')
                ->toArray();

            // その question_id の question.status=PUBLISHED であれば候補
            if (in_array(config('app.env'), ['local', 'staging'])) {
                // local / staging の場合
                $publishedQs = Question::whereIn('id', $alreadyNotStart)
                    ->whereIn('status', [
                        QuestionStatus::PUBLISHED->value,
                        QuestionStatus::TEST_PUBLISHED->value
                    ])
                    ->pluck('id')
                    ->toArray();
            } else {
                // それ以外の環境の場合
                $publishedQs = Question::whereIn('id', $alreadyNotStart)
                    ->where('status', QuestionStatus::PUBLISHED->value)
                    ->pluck('id')
                    ->toArray();
            }

            // ランダム抽出 (array_rand)
            if (!empty($publishedQs)) {
                shuffle($publishedQs); // 乱数シャッフル
                $publishedQsSliced = array_slice($publishedQs, 0, $deficit);

                // $publishedQsSliced (question_id群) → questionモデル化
                $additionalChosen = Question::whereIn('id', $publishedQsSliced)->get();
            }
        }

        //  実際に user_questions レコードを作成
        //    chosenUnanswered + additionalChosen
        $finalList = collect($chosenUnanswered)->merge($additionalChosen);

        // user_questions を新規追加(すでに user_questions があればスキップ)
        foreach ($finalList as $q) {
            $exists = UserQuestion::where('user_question_set_id', $userQuestionSet->id)
                ->where('question_id', $q->id)
                ->exists();
            if ($exists) {
                // すでにあればスキップ
                continue;
            }

            // pivotテーブルのorderを取得 (question_set_questions)
            $pivotOrder = DB::table('question_set_questions')
                ->where('question_set_id', $questionSet->id)
                ->where('question_id', $q->id)
                ->value('order') ?? 9999;

            // new user_question
            $userQuestion = new UserQuestion();
            $userQuestion->uuid = (string) Str::uuid();
            $userQuestion->user_question_set_id = $userQuestionSet->id;
            $userQuestion->question_id = $q->id;
            $userQuestion->status = UserQuestionStatus::NOT_START->value;
            $userQuestion->answer_data = null;
            $userQuestion->answered_at = null;

            // question のフィールドをコピー
            $userQuestion->metadata = $q->metadata;
            $userQuestion->version = $q->version;
            $userQuestion->evaluation_method = $q->evaluation_method;
            $userQuestion->checker_method = $q->checker_method;
            $userQuestion->llm_evaluation_prompt_file_name = $q->llm_evaluation_prompt_file_name;
            $userQuestion->evaluation_response_format = $q->evaluation_response_format;
            $userQuestion->question_type = $q->question_type;
            $userQuestion->learning_requirement_json = $q->learning_requirement_json;
            $userQuestion->learning_subject = $q->learning_subject;
            $userQuestion->learning_no = $q->learning_no;
            $userQuestion->learning_requirement = $q->learning_requirement;
            $userQuestion->learning_required_competency = $q->learning_required_competency;
            $userQuestion->learning_background = $q->learning_background;
            $userQuestion->learning_category = $q->learning_category;
            $userQuestion->learning_grade_level = $q->learning_grade_level;
            $userQuestion->learning_url = $q->learning_url;
            $userQuestion->generated_by_llm = $q->generated_by_llm;

            // order
            $userQuestion->order = $pivotOrder;
            $userQuestion->save();

            // user_question_translations => question->translations から複製
            foreach ($q->translations as $qTrans) {
                UserQuestionTranslation::create([
                    'user_question_id' => $userQuestion->id,
                    'locale'      => $qTrans->locale,
                    'question_text'   => $qTrans->question_text,
                    'explanation'     => $qTrans->explanation,
                    'background'      => $qTrans->background,
                ]);
            }
        }
    }

    /**
     * Sectionと正答率から進行パターンを判断
     *
     * @param Section $section
     * @param float $correctRate
     * @param int $questionSetCount
     * @return array
     */
    private function decideSectionProgression(?float $correctRate, int $questionSetCount): array
    {
        // サンプル数が不足している場合は通常モード
        if ($questionSetCount < self::SUCCESS_STREAK) {
            return [
                'mode' => SectionProgressMode::NORMAL,
                'action' => 'learn_in_order',
            ];
        }
        if ($correctRate >= self::TH_PASS) {
            return [
                'mode' => SectionProgressMode::MASTERED,
                'action' => 'complete_section_and_move_next',
            ];
        }

        if ($correctRate < self::GRADE_DOWN_THRESHOLD) {
            return [
                'mode' => SectionProgressMode::TOO_DIFFICULT,
                'action' => 'downgrade_grade',
            ];
        }

        // 通常学習モード
        return [
            'mode' => SectionProgressMode::NORMAL,
            'action' => 'learn_in_order',
        ];
    }

    /**
     * 次に進むQuestionSetを取得する
     *
     * @param int $userGradeId
     * @param int|null $currentSectionId
     * @param int|null $currentUnitId
     * @param int|null $currentOrder
     * @return QuestionSet|null
     */
    private function getNextQuestionSet(int $userGradeId, ?int $currentSectionId = null, ?int $currentUnitId = null, ?int $currentOrder = null): ?QuestionSet
    {
        $questionSet = null;

        if (!is_null($currentSectionId)) {
            // 1. 現在のSection内で次のQuestionSetを探す
            $query = QuestionSet::with('translations', 'unit')
                ->where('grade_id', $userGradeId)
                ->whereHas('unit', function ($q) use ($currentSectionId) {
                    $q->where('section_id', $currentSectionId);
                });

            if (!is_null($currentOrder)) {
                $questionSet = $query->where('order', '>', $currentOrder)
                    ->orderBy('order', 'asc')
                    ->orderBy('id', 'asc')
                    ->first();
            } else {
                $questionSet = $query->orderBy('order', 'asc')
                    ->orderBy('order', 'asc')
                    ->orderBy('id', 'asc')
                    ->first();
            }

            if ($questionSet) {
                return $questionSet;
            }
        }

        // 2. Section内で見つからなかったら、Unitを探す
        $currentUnit = $currentUnitId ? Unit::find($currentUnitId) : null;

        if (!$currentUnit) {
            // もし現在のUnitがないなら、grade_id内で一番小さいorderのUnitを探す
            $currentUnit = Unit::where('grade_id', $userGradeId)
                ->orderBy('order', 'asc')
                ->orderBy('id', 'asc')
                ->first();
            if ($currentUnit) {
                return QuestionSet::with('translations', 'unit')
                    ->where('unit_id', $currentUnit->id)
                    ->orderBy('order', 'asc')
                    ->first();
            }
        }

        if (!$currentUnit) {
            return null; // Unitすら無ければ終了
        }

        // 3. 次のUnit以降からQuestionSetを探す
        return $this->findFirstQuestionSetFromNextUnits($userGradeId, $currentUnit->order);
    }

    /**
     * 次のUnit以降で条件を満たすQuestionSetを探す（再帰あり）
     *
     * @param int $gradeId
     * @param int $currentUnitOrder
     * @return QuestionSet|null
     */
    private function findFirstQuestionSetFromNextUnits(int $gradeId, ?int $currentUnitOrder = 0): ?QuestionSet
    {
        $nextUnit = Unit::where('grade_id', $gradeId)
            ->where('order', '>', $currentUnitOrder)
            ->orderBy('order', 'asc')
            ->orderBy('id', 'asc')
            ->first();
        if (is_null($nextUnit)) {
            $nextGrade = $gradeId + 1;
            // ユーザーのGradeを更新する
            $userId = Auth::id();
            $this->meService->updateMe($userId, ['grade_id' => $nextGrade]);
            // 次のユニットが存在しない場合はgradeをひとつあげる
            return $this->findFirstQuestionSetFromNextUnits($nextGrade);
        }
        // 次のUnitに紐づくQuestionSetの中で、条件を満たすものを探す
        $questionSet = QuestionSet::with(['translations', 'unit', 'questions'])
            ->where('unit_id', $nextUnit->id)
            ->orderBy('order', 'asc')
            ->orderBy('id', 'asc')
            ->first();
        if ($questionSet && $this->hasValidQuestions($questionSet)) {
            return $questionSet;
        }

        // 条件に合うQuestionSetがなければ、さらに次のUnitへ再帰
        return $this->findFirstQuestionSetFromNextUnits($gradeId, $nextUnit->order);
    }

    /**
     * QuestionSetに有効なquestionsが存在するかを判定
     *
     * @param QuestionSet $questionSet
     * @return bool
     */
    private function hasValidQuestions(QuestionSet $questionSet): bool
    {
        $env = config('app.env');
        foreach ($questionSet->questions as $question) {
            if (in_array($env, ['local', 'staging'])) {
                // local / staging では PUBLISHED or TEST_PUBLISHED があればOK
                if (in_array($question->status, [
                    QuestionStatus::PUBLISHED->value,
                    QuestionStatus::TEST_PUBLISHED->value,
                ])) {
                    return true;
                }
            } else {
                // 本番環境は PUBLISHED のみ
                if ($question->status === QuestionStatus::PUBLISHED->value) {
                    return true;
                }
            }
        }

        return false;
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
        Schema::create('user_question_sets', function (Blueprint $table) {
            $table->id();
            $table->uuid('uuid')->unique();
            $table->unsignedBigInteger('user_id');
            $table->unsignedBigInteger('question_set_id');
            $table->integer('status')->default(1);
            $table->decimal('score', 8, 2)->nullable();
            $table->timestamp('started_at')->nullable();
            $table->timestamp('finished_at')->nullable();
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('user_id')->references('id')->on('users')->onDelete('cascade');
            $table->foreign('question_set_id')->references('id')->on('question_sets')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('user_question_sets');
    }
};

<?php
// routes/api.php

use App\Http\Controllers\API\V1\Answer\AnswerController;
use App\Http\Controllers\API\V1\Auth\AuthController;
use App\Http\Controllers\API\V1\DailyGoal\DailyGoalController;
use App\Http\Controllers\API\V1\Question\QuestionSetController;
use App\Http\Controllers\API\V1\Report\BugReportController;
use App\Http\Controllers\API\V1\User\UserQuestionController;
use App\Http\Controllers\API\V1\User\UserQuestionSetController;
use App\Http\Controllers\API\V1\User\UserCoachController;
use App\Http\Middleware\OptionalAuthenticate;
use Illuminate\Support\Facades\Route;

Route::prefix('v1')->group(function () {
    Route::prefix('auth')->group(function () {
        /**
         * @group Auth
         *
         * ログイン（メールアドレス）
         *
         * テスト用のエンドポイントです。
         *
         * @endpoint /api/v1/auth/loginByEmail
         */
        // テスト用、Emailログインは採用してない。
        Route::post('/loginByEmail', [AuthController::class, 'loginByEmail'])->name('login');

        /**
         * @group Auth
         *
         * Googleアカウントによるログイン
         *
         * @endpoint /api/v1/auth/loginWithGoogle
         */
        Route::post('/loginWithGoogle', [AuthController::class, 'loginWithGoogle']);

        /**
         * @group Auth
         *
         * Lineアカウントによるログイン
         *
         * @endpoint /api/v1/auth/loginWithLine
         */
        Route::post('/loginWithLine', [AuthController::class, 'loginWithLine']);

        /**
         * @group Auth
         *
         * Line Liff アプリによるLogin
         *
         * @endpoint /api/v1/auth/loginWithLineLiff
         */
        Route::post('/loginWithLineLiff', [AuthController::class, 'loginWithLineLiff']);

        /**
         * @group Auth
         *
         * ユーザー登録API
         *
         * ユーザー登録を行います。
         *
         * @endpoint /api/v1/auth/register
         */
        Route::post('/register', [\App\Http\Controllers\API\V1\Auth\RegisterController::class, 'register']);

        /**
         * @group Auth
         *
         * パスワードリセットリクエストAPI
         *
         * パスワードリセットリクエストを行います。
         *
         * @endpoint /api/v1/password/reset/request
         */
        Route::post('/password/reset/request', [\App\Http\Controllers\API\V1\Auth\PasswordResetController::class, 'request']);

        /**
         * @group Auth
         *
         * パスワードリセットAPI
         *
         * パスワードリセットを行います。
         *
         * @endpoint /api/v1/password/reset
         */
        Route::post('/password/reset', [\App\Http\Controllers\API\V1\Auth\PasswordResetController::class, 'reset']);
        //        Route::post('/loginWithApple', [AuthController::class, 'loginWithApple']);
    });

    /**
     * ログイン済み、非ログイン使用可能Route(認証済みの場合は認証ユーザーを取得する)
     */
    Route::middleware(OptionalAuthenticate::class)->group(function () {
        Route::prefix('study-sessions')->group(function () {
            /**
             * @group Study Sessions
             *
             * 学習セッションを開始記録します。
             *
             * @endpoint /api/v1/study-sessions
             */
            Route::post('/', [\App\Http\Controllers\API\V1\StudySession\StudySessionController::class, 'store'])->name('v1.study-session.store');
            /**
             * @group Study Sessions
             *
             * 学習セッションを開始し、取り組みを記録します。
             *
             * @endpoint /api/v1/study-sessions/start
             */
            Route::post('/start', [\App\Http\Controllers\API\V1\StudySession\StudySessionController::class, 'begin'])->name('v1.study-session.begin');
            /**
             * @group Study Sessions
             *
             * 途中の学習セッションを取得、ない場合は次の学習セッションを開始します。
             *
             * @endpoint /api/v1/study-sessions/continue
             */
            Route::get('/continue', [\App\Http\Controllers\API\V1\StudySession\StudySessionController::class, 'continue'])->name('v1.study-session.continue');
        });

        Route::prefix('answer')->group(function () {
            /**
             * @group Answer
             *
             * 回答を登録します。
             *
             * @endpoint /api/v1/answer
             */
            Route::post('/', [AnswerController::class, 'store'])->name('v1.answer.store');
        });

        Route::prefix('user-question-sets')->group(function () {
            /**
             * @group User Question Sets
             *
             * 指定したユーザー質問セットの詳細を取得します。
             *
             * @endpoint /api/v1/user-question-sets/{user_question_set_id}
             * @urlParam user_question_set_id required ユーザー質問セットID Example: 1
             */
            Route::get('{user_question_set_id}', [UserQuestionSetController::class, 'show'])
                ->name('v1.user-question-sets.show');
        });
    });

    Route::prefix('user-questions')->group(function () {
        /**
         * @group User Questions
         *
         * 指定したユーザー質問の詳細を取得します。
         *
         * @endpoint /api/v1/user-questions/{user_question_id}
         * @urlParam user_question_id required ユーザー質問ID Example: 1
         */
        Route::get('{user_question_id}', [UserQuestionController::class, 'show'])
            ->name('v1.user-questions.show');
    });

    /**
     * 認証済み
     */
    Route::middleware(['auth:sanctum'])->group(function () {
        Route::prefix('question-sets')->group(function () {
            /**
             * @group Question Sets
             *
             * 質問セットの一覧を取得します。
             *
             * @endpoint /api/v1/question-sets
             */
            Route::get('/', [QuestionSetController::class, 'index']);
        });

        Route::prefix('bug-reports')->group(function () {
            /**
             * @group Report
             *
             * 問題のバグを報告するします
             *
             * @endpoint /api/v1/bug-reports
             */
            Route::post('', [BugReportController::class, 'store']);
        });

        Route::prefix('daily-goal')->group(function () {
            /**
             * @group Goal
             *
             * 目標を保存します
             *
             * @endpoint /api/v1/daily-goal
             */
            Route::post('', [DailyGoalController::class, 'store']);
        });



        Route::prefix('sections')->group(function () {
            /**
             * @group Sections
             *
             * セクション一覧を取得します。
             *
             * @endpoint /api/v1/sections
             */
            Route::get('/', [\App\Http\Controllers\API\V1\Section\SectionController::class, 'index'])->name('v1.sections.index');
            /**
             * @group Sections
             *
             * 指定したセクションを取得します。
             *
             * @endpoint /api/v1/sections/{id}
             * @urlParam id required セクションID Example: 1
             */
            Route::get('/{id}', [\App\Http\Controllers\API\V1\Section\SectionController::class, 'show'])->name('v1.sections.show');
        });

        Route::prefix('auth')->group(function () {
            /**
             * @group Auth
             *
             * ログアウトします。
             *
             * @endpoint /api/v1/auth/logout
             */
            Route::post('/logout', [AuthController::class, 'logout']);
            /**
             * @group Auth
             *
             * ログイン中のユーザー情報を取得します。
             *
             * @endpoint /api/v1/auth/me
             */
            Route::get('/me', [AuthController::class, 'me']);

            /**
             * @group Auth
             *
             * ログイン中のユーザー情報を更新します。
             *
             * @endpoint /api/v1/auth/me
             */
            Route::put('/me', [AuthController::class, 'updateMe']);
        });

        Route::prefix('user-coach')->group(function () {
            /**
             * @group User Coach
             *
             * ユーザー紐づけURL発行
             *
             * @endpoint /api/v1/user-links/links
             * @urlParam user_id required
             */
            Route::get('/links', [UserCoachController::class, 'links'])
                ->name('v1.user-coach.links');
            /**
             * @group User Coach
             *
             * ユーザー紐づけ
             *
             * @endpoint /api/v1/user-links/sync
             * @urlParam user_id required
             */
            Route::post('/sync', [UserCoachController::class, 'sync'])
                ->name('v1.user-coach.sync');

            /**
             * @group User Coach
             *
             * ユーザー紐づけ解除
             *
             * @endpoint /api/v1/user-links
             * @urlParam user_id required
             */
            Route::put('/unlink/{user_id}', [UserCoachController::class, 'unlink'])
                ->name('v1.user-coach.unlink');
        });

        Route::prefix('user-question-sets')->group(function () {

            Route::put('{user_question_set_id}/skip',
                [UserQuestionSetController::class, 'skip']
            )->name('v1.user-question-sets.skip');

            /**
             * @group User Question Sets Notify
             *
             * 指定したユーザー質問セットの詳細を取得します。
             *
             * @endpoint /api/v1/user-question-sets/{user_question_set_id}
             * @urlParam user_question_set_id required ユーザー質問セットID Example: 1
             */
            Route::post('{user_question_set_id}/notify', [UserQuestionSetController::class, 'notify'])
                ->name('v1.user-question-sets.notify');
            /**
             * @group User Question Sets
             *
             * ユーザー質問セットの一覧を取得します。
             *
             * @endpoint /api/v1/user-question-sets
             */
            Route::get('/', [UserQuestionSetController::class, 'index'])
                ->name('v1.user-question-sets.index');
        });
    });
});
<?php

namespace App\Http\Resources\V1\User;

use App\Dtos\V1\User\UserQuestionDto;
use App\Dtos\V1\User\UserQuestionSetDto;
use App\Http\Resources\V1\Question\QuestionResource;
use App\Http\Resources\V1\Question\QuestionSetResource;
use Illuminate\Http\Resources\Json\JsonResource;

class UserQuestionSetResource extends JsonResource
{
    /**
     * @param \Illuminate\Http\Request $request
     * @return array
     *
     * $this->resource is UserQuestionSetDto
     */
    public function toArray($request)
    {
        /** @var UserQuestionSetDto $dto */
        $dto = $this->resource;

        return [
            'id'          => $dto->id,
            'user_id'     => $dto->user_id,
            'status'      => $dto->status,
            'score'       => $dto->score,
            'started_at'  => $dto->started_at,
            'finished_at' => $dto->finished_at,

            'question_set' => $dto->question_set
                ? new QuestionSetResource($dto->question_set)
                : null,

            'user_questions' => $dto->user_questions->map(function(UserQuestionDto $uqDto) {
                return [
                    'id'          => $uqDto->id,
                    'answer_data' => json_decode($uqDto->answer_data),
                    'answered_at' => $uqDto->answered_at,
                    'question' => $uqDto->question
                        ? new QuestionResource($uqDto->question)
                        : null,
                ];
            })->toArray(),

            'next_user_question_id' => $dto->next_user_question_id ?? null,
        ];
    }
}
<?php

namespace App\Http\Controllers\API\V1\User;

use App\Http\Controllers\Controller;
use App\Http\Resources\V1\User\UserQuestionSetResource;
use App\UseCases\V1\User\GetAllUserQuestionSetsUseCase;
use App\UseCases\V1\User\GetUserQuestionSetDetailUseCase;
use App\UseCases\V1\User\NotifyUserQuestionSetUseCase;
use App\UseCases\V1\User\SkipUserQuestionSetUseCase;
use Illuminate\Http\Request;

class UserQuestionSetController extends Controller
{
    public function __construct(
        protected GetUserQuestionSetDetailUseCase $getUserQuestionSetDetailUseCase,
        protected GetAllUserQuestionSetsUseCase $getAllUserQuestionSetsUseCase,
        protected SkipUserQuestionSetUseCase      $skipUserQuestionSetUseCase,
        protected NotifyUserQuestionSetUseCase $notifyUserQuestionSetUseCase
    ) {
    }

    public function index(Request $request)
    {
        $locale = app()->getLocale(); // Accept-Language
        $userId = \Auth::id(); // ログインユーザーID

        // 一覧を取得 (複数のUserQuestionSetDtoを返す想定)
        $dtoCollection = $this->getAllUserQuestionSetsUseCase
            ->handle($userId, $locale);

        // Resourceのcollectionで返す
        return UserQuestionSetResource::collection($dtoCollection);
    }

    /**
     * GET /api/v1/user-question-sets/{user_question_set_id}
     *
     * 学習完了または完了していないUserQuestionSetを取得し、
     * nextUserQuestionなどを含めて返す
     */
    public function show(string $user_question_set_id, Request $request)
    {
        $locale = app()->getLocale(); // Accept-Language から取得済み想定
        // もしユーザーIDチェックが必要ならAuth::id()など
        $userId = optional($request->user())->id;

        $dto = $this->getUserQuestionSetDetailUseCase
            ->handle($user_question_set_id, $locale, $userId);

        return new UserQuestionSetResource($dto);
    }

    /**
     * PUT /api/v1/user-question-sets/{id}/skip
     *
     * 指定 UserQuestionSet の status を SKIP に更新して返す
     */
    public function skip(
        string $user_question_set_id,
        Request $request
    ) {
        $locale = app()->getLocale();
        $userId = \Auth::id();

        $dto = $this->skipUserQuestionSetUseCase
            ->handle($user_question_set_id, $locale, $userId);

        return new UserQuestionSetResource($dto);
    }

    /**
     * @param string $user_question_set_id
     */
    public function notify(string $user_question_set_id)
    {
        $locale = app()->getLocale();
        $userId = \Auth::id();

        $dto = $this->notifyUserQuestionSetUseCase
            ->handle($user_question_set_id, $locale, $userId);

        return new UserQuestionSetResource($dto);
    }
}
<?php
// /lang/en/validation.php
return [

    /*
    |--------------------------------------------------------------------------
    | Validation Language Lines
    |--------------------------------------------------------------------------
    |
    | The following language lines contain the default error messages used by
    | the validator class. Some of these rules have multiple versions such
    | as the size rules. Feel free to tweak each of these messages here.
    |
    */

    'accepted' => 'The :attribute field must be accepted.',
    'accepted_if' => 'The :attribute field must be accepted when :other is :value.',
    'active_url' => 'The :attribute field must be a valid URL.',
    'after' => 'The :attribute field must be a date after :date.',
    'after_or_equal' => 'The :attribute field must be a date after or equal to :date.',
    'alpha' => 'The :attribute field must only contain letters.',
    'alpha_dash' => 'The :attribute field must only contain letters, numbers, dashes, and underscores.',
    'alpha_num' => 'The :attribute field must only contain letters and numbers.',
    'array' => 'The :attribute field must be an array.',
    'ascii' => 'The :attribute field must only contain single-byte alphanumeric characters and symbols.',
    'before' => 'The :attribute field must be a date before :date.',
    'before_or_equal' => 'The :attribute field must be a date before or equal to :date.',
    'between' => [
        'array' => 'The :attribute field must have between :min and :max items.',
        'file' => 'The :attribute field must be between :min and :max kilobytes.',
        'numeric' => 'The :attribute field must be between :min and :max.',
        'string' => 'The :attribute field must be between :min and :max characters.',
    ],
    'boolean' => 'The :attribute field must be true or false.',
    'can' => 'The :attribute field contains an unauthorized value.',
    'confirmed' => 'The :attribute field confirmation does not match.',
    'contains' => 'The :attribute field is missing a required value.',
    'current_password' => 'The password is incorrect.',
    'date' => 'The :attribute field must be a valid date.',
    'date_equals' => 'The :attribute field must be a date equal to :date.',
    'date_format' => 'The :attribute field must match the format :format.',
    'decimal' => 'The :attribute field must have :decimal decimal places.',
    'declined' => 'The :attribute field must be declined.',
    'declined_if' => 'The :attribute field must be declined when :other is :value.',
    'different' => 'The :attribute field and :other must be different.',
    'digits' => 'The :attribute field must be :digits digits.',
    'digits_between' => 'The :attribute field must be between :min and :max digits.',
    'dimensions' => 'The :attribute field has invalid image dimensions.',
    'distinct' => 'The :attribute field has a duplicate value.',
    'doesnt_end_with' => 'The :attribute field must not end with one of the following: :values.',
    'doesnt_start_with' => 'The :attribute field must not start with one of the following: :values.',
    'email' => 'The :attribute field must be a valid email address.',
    'ends_with' => 'The :attribute field must end with one of the following: :values.',
    'enum' => 'The selected :attribute is invalid.',
    'exists' => 'The selected :attribute is invalid.',
    'extensions' => 'The :attribute field must have one of the following extensions: :values.',
    'file' => 'The :attribute field must be a file.',
    'filled' => 'The :attribute field must have a value.',
    'gt' => [
        'array' => 'The :attribute field must have more than :value items.',
        'file' => 'The :attribute field must be greater than :value kilobytes.',
        'numeric' => 'The :attribute field must be greater than :value.',
        'string' => 'The :attribute field must be greater than :value characters.',
    ],
    'gte' => [
        'array' => 'The :attribute field must have :value items or more.',
        'file' => 'The :attribute field must be greater than or equal to :value kilobytes.',
        'numeric' => 'The :attribute field must be greater than or equal to :value.',
        'string' => 'The :attribute field must be greater than or equal to :value characters.',
    ],
    'hex_color' => 'The :attribute field must be a valid hexadecimal color.',
    'image' => 'The :attribute field must be an image.',
    'in' => 'The selected :attribute is invalid.',
    'in_array' => 'The :attribute field must exist in :other.',
    'integer' => 'The :attribute field must be an integer.',
    'ip' => 'The :attribute field must be a valid IP address.',
    'ipv4' => 'The :attribute field must be a valid IPv4 address.',
    'ipv6' => 'The :attribute field must be a valid IPv6 address.',
    'json' => 'The :attribute field must be a valid JSON string.',
    'list' => 'The :attribute field must be a list.',
    'lowercase' => 'The :attribute field must be lowercase.',
    'lt' => [
        'array' => 'The :attribute field must have less than :value items.',
        'file' => 'The :attribute field must be less than :value kilobytes.',
        'numeric' => 'The :attribute field must be less than :value.',
        'string' => 'The :attribute field must be less than :value characters.',
    ],
    'lte' => [
        'array' => 'The :attribute field must not have more than :value items.',
        'file' => 'The :attribute field must be less than or equal to :value kilobytes.',
        'numeric' => 'The :attribute field must be less than or equal to :value.',
        'string' => 'The :attribute field must be less than or equal to :value characters.',
    ],
    'mac_address' => 'The :attribute field must be a valid MAC address.',
    'max' => [
        'array' => 'The :attribute field must not have more than :max items.',
        'file' => 'The :attribute field must not be greater than :max kilobytes.',
        'numeric' => 'The :attribute field must not be greater than :max.',
        'string' => 'The :attribute field must not be greater than :max characters.',
    ],
    'max_digits' => 'The :attribute field must not have more than :max digits.',
    'mimes' => 'The :attribute field must be a file of type: :values.',
    'mimetypes' => 'The :attribute field must be a file of type: :values.',
    'min' => [
        'array' => 'The :attribute field must have at least :min items.',
        'file' => 'The :attribute field must be at least :min kilobytes.',
        'numeric' => 'The :attribute field must be at least :min.',
        'string' => 'The :attribute field must be at least :min characters.',
    ],
    'min_digits' => 'The :attribute field must have at least :min digits.',
    'missing' => 'The :attribute field must be missing.',
    'missing_if' => 'The :attribute field must be missing when :other is :value.',
    'missing_unless' => 'The :attribute field must be missing unless :other is :value.',
    'missing_with' => 'The :attribute field must be missing when :values is present.',
    'missing_with_all' => 'The :attribute field must be missing when :values are present.',
    'multiple_of' => 'The :attribute field must be a multiple of :value.',
    'not_in' => 'The selected :attribute is invalid.',
    'not_regex' => 'The :attribute field format is invalid.',
    'numeric' => 'The :attribute field must be a number.',
    'password' => [
        'letters' => 'The :attribute field must contain at least one letter.',
        'mixed' => 'The :attribute field must contain at least one uppercase and one lowercase letter.',
        'numbers' => 'The :attribute field must contain at least one number.',
        'symbols' => 'The :attribute field must contain at least one symbol.',
        'uncompromised' => 'The given :attribute has appeared in a data leak. Please choose a different :attribute.',
    ],
    'present' => 'The :attribute field must be present.',
    'present_if' => 'The :attribute field must be present when :other is :value.',
    'present_unless' => 'The :attribute field must be present unless :other is :value.',
    'present_with' => 'The :attribute field must be present when :values is present.',
    'present_with_all' => 'The :attribute field must be present when :values are present.',
    'prohibited' => 'The :attribute field is prohibited.',
    'prohibited_if' => 'The :attribute field is prohibited when :other is :value.',
    'prohibited_unless' => 'The :attribute field is prohibited unless :other is in :values.',
    'prohibits' => 'The :attribute field prohibits :other from being present.',
    'regex' => 'The :attribute field format is invalid.',
    'required' => 'The :attribute field is required.',
    'required_array_keys' => 'The :attribute field must contain entries for: :values.',
    'required_if' => 'The :attribute field is required when :other is :value.',
    'required_if_accepted' => 'The :attribute field is required when :other is accepted.',
    'required_if_declined' => 'The :attribute field is required when :other is declined.',
    'required_unless' => 'The :attribute field is required unless :other is in :values.',
    'required_with' => 'The :attribute field is required when :values is present.',
    'required_with_all' => 'The :attribute field is required when :values are present.',
    'required_without' => 'The :attribute field is required when :values is not present.',
    'required_without_all' => 'The :attribute field is required when none of :values are present.',
    'same' => 'The :attribute field must match :other.',
    'size' => [
        'array' => 'The :attribute field must contain :size items.',
        'file' => 'The :attribute field must be :size kilobytes.',
        'numeric' => 'The :attribute field must be :size.',
        'string' => 'The :attribute field must be :size characters.',
    ],
    'starts_with' => 'The :attribute field must start with one of the following: :values.',
    'string' => 'The :attribute field must be a string.',
    'timezone' => 'The :attribute field must be a valid timezone.',
    'unique' => 'The :attribute has already been taken.',
    'uploaded' => 'The :attribute failed to upload.',
    'uppercase' => 'The :attribute field must be uppercase.',
    'url' => 'The :attribute field must be a valid URL.',
    'ulid' => 'The :attribute field must be a valid ULID.',
    'uuid' => 'The :attribute field must be a valid UUID.',

    /*
    |--------------------------------------------------------------------------
    | Custom Validation Language Lines
    |--------------------------------------------------------------------------
    |
    | Here you may specify custom validation messages for attributes using the
    | convention "attribute.rule" to name the lines. This makes it quick to
    | specify a specific custom language line for a given attribute rule.
    |
    */

    'custom' => [
        'unit_id' => [
            'required' => 'The unit_id field is required.',
            'uuid'     => 'The unit_id must be a valid UUID.',
            'exists'   => 'The specified unit_id does not exist.',
        ],
        'password' => [
            'regex' => 'The password must contain at least one uppercase letter, one lowercase letter, and one digit.',
        ],
    ],

    /*
    |--------------------------------------------------------------------------
    | Custom Validation Attributes
    |--------------------------------------------------------------------------
    |
    | The following language lines are used to swap our attribute placeholder
    | with something more reader friendly such as "E-Mail Address" instead
    | of "email". This simply helps us make our message more expressive.
    |
    */

    'attributes' => [
        'email' => 'Email',
        'password' => 'Password',
        'user_name' => 'User name',
    ],

];
<?php
// /lang/ja/validation.php
return [

    /*
    |--------------------------------------------------------------------------
    | バリデーション言語行
    |--------------------------------------------------------------------------
    |
    | 以下の言語行はバリデタークラスで使用されるデフォルトのエラーメッセージです。
    | 一部のルールはサイズルールのように複数バージョンがあります。
    | ここでメッセージを自由に調整してください。
    |
    */

    'accepted' => ':attribute は承認されなければなりません。',
    'accepted_if' => ':other が :value のとき、:attribute は承認されなければなりません。',
    'active_url' => ':attribute は有効なURLでなければなりません。',
    'after' => ':attribute は :date 以降の日付でなければなりません。',
    'after_or_equal' => ':attribute は :date 以降または同日の日付でなければなりません。',
    'alpha' => ':attribute は文字のみを含む必要があります。',
    'alpha_dash' => ':attribute は文字、数字、ハイフン、アンダースコアのみを含む必要があります。',
    'alpha_num' => ':attribute は文字と数字のみを含む必要があります。',
    'array' => ':attribute は配列でなければなりません。',
    'ascii' => ':attribute はシングルバイトの英数字および記号のみを含む必要があります。',
    'before' => ':attribute は :date 以前の日付でなければなりません。',
    'before_or_equal' => ':attribute は :date 以前または同日の日付でなければなりません。',
    'between' => [
        'array' => ':attribute の項目数は :min から :max の間でなければなりません。',
        'file' => ':attribute は :min から :max KBの間でなければなりません。',
        'numeric' => ':attribute は :min から :max の間でなければなりません。',
        'string' => ':attribute は :min 文字から :max 文字の間でなければなりません。',
    ],
    'boolean' => ':attribute は true もしくは false でなければなりません。',
    'can' => ':attribute は許可されていない値を含んでいます。',
    'confirmed' => ':attribute と確認用の値が一致しません。',
    'contains' => ':attribute には必須の値が含まれていません。',
    'current_password' => 'パスワードが正しくありません。',
    'date' => ':attribute は有効な日付ではありません。',
    'date_equals' => ':attribute は :date と同じ日付でなければなりません。',
    'date_format' => ':attribute は :format 形式と一致しなければなりません。',
    'decimal' => ':attribute は :decimal 桁の小数でなければなりません。',
    'declined' => ':attribute は拒否されなければなりません。',
    'declined_if' => ':other が :value のとき、:attribute は拒否されなければなりません。',
    'different' => ':attribute と :other は異なるものでなければなりません。',
    'digits' => ':attribute は :digits 桁でなければなりません。',
    'digits_between' => ':attribute は :min 桁から :max 桁の間でなければなりません。',
    'dimensions' => ':attribute は無効な画像サイズです。',
    'distinct' => ':attribute には重複した値が含まれています。',
    'doesnt_end_with' => ':attribute は :values のいずれかで終わっていてはなりません。',
    'doesnt_start_with' => ':attribute は :values のいずれかで始まっていてはなりません。',
    'email' => ':attribute は有効なメールアドレスでなければなりません。',
    'ends_with' => ':attribute は :values のいずれかで終わらなければなりません。',
    'enum' => '選択された :attribute は無効です。',
    'exists' => '選択された :attribute は無効です。',
    'extensions' => ':attribute は :values のいずれかの拡張子でなければなりません。',
    'file' => ':attribute はファイルでなければなりません。',
    'filled' => ':attribute は値が必要です。',
    'gt' => [
        'array' => ':attribute の項目数は :value 個より多くなければなりません。',
        'file' => ':attribute は :value KBより大きくなければなりません。',
        'numeric' => ':attribute は :value より大きくなければなりません。',
        'string' => ':attribute は :value 文字より多くなければなりません。',
    ],
    'gte' => [
        'array' => ':attribute の項目数は :value 個以上でなければなりません。',
        'file' => ':attribute は :value KB以上でなければなりません。',
        'numeric' => ':attribute は :value 以上でなければなりません。',
        'string' => ':attribute は :value 文字以上でなければなりません。',
    ],
    'hex_color' => ':attribute は有効な16進数カラーでなければなりません。',
    'image' => ':attribute は画像でなければなりません。',
    'in' => '選択された :attribute は無効です。',
    'in_array' => ':attribute は :other に存在しなければなりません。',
    'integer' => ':attribute は整数でなければなりません。',
    'ip' => ':attribute は有効なIPアドレスでなければなりません。',
    'ipv4' => ':attribute は有効なIPv4アドレスでなければなりません。',
    'ipv6' => ':attribute は有効なIPv6アドレスでなければなりません。',
    'json' => ':attribute は有効なJSON文字列でなければなりません。',
    'list' => ':attribute はリストでなければなりません。',
    'lowercase' => ':attribute は小文字でなければなりません。',
    'lt' => [
        'array' => ':attribute の項目数は :value 個より少なくなければなりません。',
        'file' => ':attribute は :value KBより小さくなければなりません。',
        'numeric' => ':attribute は :value より小さくなければなりません。',
        'string' => ':attribute は :value 文字より少なくなければなりません。',
    ],
    'lte' => [
        'array' => ':attribute の項目数は :value 個以下でなければなりません。',
        'file' => ':attribute は :value KB以下でなければなりません。',
        'numeric' => ':attribute は :value 以下でなければなりません。',
        'string' => ':attribute は :value 文字以下でなければなりません。',
    ],
    'mac_address' => ':attribute は有効なMACアドレスでなければなりません。',
    'max' => [
        'array' => ':attribute は :max 個を超えてはなりません。',
        'file' => ':attribute は :max KBを超えてはなりません。',
        'numeric' => ':attribute は :max を超えてはなりません。',
        'string' => ':attribute は :max 文字を超えてはなりません。',
    ],
    'max_digits' => ':attribute は :max 桁を超えてはなりません。',
    'mimes' => ':attribute は :values タイプのファイルでなければなりません。',
    'mimetypes' => ':attribute は :values タイプのファイルでなければなりません。',
    'min' => [
        'array' => ':attribute は少なくとも :min 個の項目が必要です。',
        'file' => ':attribute は少なくとも :min KBでなければなりません。',
        'numeric' => ':attribute は少なくとも :min でなければなりません。',
        'string' => ':attribute は少なくとも :min 文字でなければなりません。',
    ],
    'min_digits' => ':attribute は少なくとも :min 桁でなければなりません。',
    'missing' => ':attribute は存在してはいけません。',
    'missing_if' => ':other が :value の場合、:attribute は存在してはいけません。',
    'missing_unless' => ':other が :value でない限り、:attribute は存在してはいけません。',
    'missing_with' => ':values が存在する場合、:attribute は存在してはいけません。',
    'missing_with_all' => ':values がすべて存在する場合、:attribute は存在してはいけません。',
    'multiple_of' => ':attribute は :value の倍数でなければなりません。',
    'not_in' => '選択された :attribute は無効です。',
    'not_regex' => ':attribute の形式が無効です。',
    'numeric' => ':attribute は数値でなければなりません。',
    'password' => [
        'letters' => ':attribute には少なくとも1文字のアルファベットが含まれていなければなりません。',
        'mixed' => ':attribute には大文字と小文字が少なくとも1文字ずつ含まれていなければなりません。',
        'numbers' => ':attribute には少なくとも1つの数字が含まれていなければなりません。',
        'symbols' => ':attribute には少なくとも1つの記号が含まれていなければなりません。',
        'uncompromised' => '提供された :attribute は情報漏洩で確認されています。別の :attribute を選択してください。',
    ],
    'present' => ':attribute が存在していなければなりません。',
    'present_if' => ':other が :value の場合、:attribute が存在していなければなりません。',
    'present_unless' => ':other が :value でない限り、:attribute が存在していなければなりません。',
    'present_with' => ':values が存在する場合、:attribute も存在していなければなりません。',
    'present_with_all' => ':values がすべて存在する場合、:attribute も存在していなければなりません。',
    'prohibited' => ':attribute は入力禁止です。',
    'prohibited_if' => ':other が :value の場合、:attribute は入力禁止です。',
    'prohibited_unless' => ':other が :values のいずれかでない限り、:attribute は入力禁止です。',
    'prohibits' => ':attribute は :other の存在を禁止しています。',
    'regex' => ':attribute の形式が無効です。',
    'required' => ':attribute は必須です。',
    'required_array_keys' => ':attribute は :values に対するエントリを含んでいなければなりません。',
    'required_if' => ':other が :value の場合、:attribute は必須です。',
    'required_if_accepted' => ':other が承認されている場合、:attribute は必須です。',
    'required_if_declined' => ':other が拒否されている場合、:attribute は必須です。',
    'required_unless' => ':other が :values のいずれかでない限り、:attribute は必須です。',
    'required_with' => ':values が存在する場合、:attribute は必須です。',
    'required_with_all' => ':values がすべて存在する場合、:attribute は必須です。',
    'required_without' => ':values が存在しない場合、:attribute は必須です。',
    'required_without_all' => ':values がすべて存在しない場合、:attribute は必須です。',
    'same' => ':attribute と :other は一致していなければなりません。',
    'size' => [
        'array' => ':attribute は :size 個の項目を含んでいなければなりません。',
        'file' => ':attribute は :size KBでなければなりません。',
        'numeric' => ':attribute は :size でなければなりません。',
        'string' => ':attribute は :size 文字でなければなりません。',
    ],
    'starts_with' => ':attribute は :values のいずれかで始まらなければなりません。',
    'string' => ':attribute は文字列でなければなりません。',
    'timezone' => ':attribute は有効なタイムゾーンでなければなりません。',
    'unique' => ':attribute は既に使用されています。',
    'uploaded' => ':attribute のアップロードに失敗しました。',
    'uppercase' => ':attribute は大文字でなければなりません。',
    'url' => ':attribute は有効なURLでなければなりません。',
    'ulid' => ':attribute は有効なULIDでなければなりません。',
    'uuid' => ':attribute は有効なUUIDでなければなりません。',

    /*
    |--------------------------------------------------------------------------
    | カスタムバリデーション言語行
    |--------------------------------------------------------------------------
    |
    | ここでは "attribute.rule" の規約を使用して属性に対するカスタムバリデーションメッセージを
    | 指定することができます。これにより特定の属性ルールに対して
    | 素早くカスタム言語行を指定することができます。
    |
    */

    'custom' => [
        'unit_id' => [
            'required' => 'unit_id は必須です。',
            'uuid'     => 'unit_id は有効なUUIDでなければなりません。',
            'exists'   => '指定された unit_id は存在しません。',
        ],
        'password' => [
            'regex' => 'パスワードには大文字・小文字・数字をそれぞれ1文字以上含めてください。',
        ],
    ],

    /*
    |--------------------------------------------------------------------------
    | カスタムバリデーション属性
    |--------------------------------------------------------------------------
    |
    | 以下の言語行は、たとえば "email" ではなく "E-Mail アドレス" と表示したい場合などに
    | 属性プレースホルダーをよりわかりやすい表現に交換します。
    | これによりメッセージをよりわかりやすくできます。
    |
    */

    'attributes' => [
        'email' => 'メールアドレス',
        'password' => 'パスワード',
        'user_name' => 'ユーザ名',
    ],

];

```
