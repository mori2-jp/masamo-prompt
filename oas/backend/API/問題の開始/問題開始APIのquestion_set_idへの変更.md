以下の /study-sessions /study-sessions/start の２つのAPIに以下の変更、仕様追加を加えてほしいです。

・Enumで評価手法を分岐
・Enumでどのメソッドを使って評価するか分岐
・user_question_sets に次の問題（紐づいている user_questions の中に status がNOT_STARTのものがあれば　order 昇順で一番先頭のやつ）があればそれをレスポンスする。無ければ user_question_sets の status をCOMPLETEにする

# 変更点
・
・現在は unit_id を受け付けていますが、これを question_set_id に変更してください
unit は複数の　questions_sets を持っているので今の設計だとよくないので変更します。

・questions, user_questions は、それぞれ xx_translations テーブルを持っています。Accept-Languageに設定されている言語に翻訳した結果をレスポンスすることを忘れないでください
・Dtoをネストする必要がありますが、SectionDtoと同じようにCastWithを使ってください


# 実装方針
生成また修正変更するコードはそのコードだけは必ず省略せずに必ず全てアウトプットすること。

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

Dto に fromModelを実装するようなことはせずに、Dtoの生成は必ずService内部で実行するようにしてください

# 説明
question_sets：問題（questions）を束ねるグループ。Unit（単元）に紐づく
questions: 問題
question_set_questions：question_sets と questions を紐づけるPivotテーブル。questions は複数のquestion_sets に紐づく事があり多対多の関係なのでこのような設計になっている。
user_question_sets: ユーザーが学習した questions_sets。学習開始時に question_sets_id と紐づいて status （UserQuestionSetsStatus::NOT_START) が未開始の状態で生成され、進捗ステータスはスコアなどが管理される
user_questions: ユーザーが学習した questions。学習開始時に、学習を開始した question_sets に紐づく questions が全て、 user_question_sets_id と question_id（questions）を紐づけて status （UserQuestionStatus::NOT_START) が未開始の状態で全てのquestionsの数の分、生成され、進捗ステータスはスコアなどが管理される。スコアを保持
sections: セクション
units: ユニット（単元）。セクションに紐づく

ーーDtoはService内部で生成例
<?php

namespace App\Services\V1\Section;

use App\Dtos\V1\Question\QuestionSetDto;
use App\Dtos\V1\Section\SectionDto;
use App\Dtos\V1\Unit\UnitDto;
use App\Enums\QuestionSetStatus;
use App\Enums\UnitStatus;
use App\Models\Question\QuestionSet;
use App\Models\Section\Section;
use App\Models\Unit\Unit;
use Illuminate\Support\Collection;

class SectionService
{
    /**
     * publishedのみ取得し、翻訳済みの値を取り扱いながらSectionDtoのコレクションを返す。
     *
     * $withUnits = true の場合は units もロード＆翻訳。
     * $withQuestionSets = true の場合は unit内の question_sets もロード＆翻訳して QuestionSetDto 化する。
     *
     * @param bool        $withUnits
     * @param string|null $levelId
     * @param string|null $subjectId
     * @param bool        $withQuestionSets
     * @return Collection<SectionDto>
     */
    public function getAll(
        bool $withUnits,
        ?string $levelId,
        ?string $subjectId,
        bool $withQuestionSets
    ): Collection {
        // sectionのpublishedのみを取得
        $query = Section::published()->orderBy('order');

        if ($withUnits) {
            // unitのpublishedのみを取得＆order順
            $query->with([
                'units' => fn($q) =>
                $q->published()->orderBy('order')
                    // with_question_sets がtrueなら questionSetsもあらかじめロード
                    ->when($withQuestionSets, function ($subQ) {
                        $subQ->with([
                            'questionSets' => fn($qq) =>
                            $qq->published()->orderBy('order')
                        ]);
                    }),
            ]);
        }

        if ($levelId) {
            $query->where('level_id', $levelId);
        }
        if ($subjectId) {
            $query->where('subject_id', $subjectId);
        }

        $sections = $query->get();

        // Model → SectionDto (翻訳適用) へ変換
        return $sections->map(
            fn (Section $section) => $this->toSectionDto($section, $withUnits, $withQuestionSets)
        );
    }

    /**
     * 指定IDのSectionを published のみから取得し、翻訳済みの SectionDto を返す。
     * 見つからなければ null を返す。
     *
     * @param string $id
     * @param bool   $withUnits
     * @param bool   $withQuestionSets
     * @return SectionDto|null
     */
    public function findById(string $id, bool $withUnits, bool $withQuestionSets): ?SectionDto
    {
        $query = Section::published();

        if ($withUnits) {
            $query->with([
                'units' => fn($q) =>
                $q->published()->orderBy('order')
                    ->when($withQuestionSets, function ($subQ) {
                        $subQ->with([
                            'questionSets' => fn($qq) =>
                            $qq->published()->orderBy('order')
                        ]);
                    }),
            ]);
        }

        $section = $query->find($id);
        if (!$section) {
            return null;
        }

        return $this->toSectionDto($section, $withUnits, $withQuestionSets);
    }

    /**
     * Section モデルを翻訳込みで SectionDto に変換するための共通処理
     *
     * @param  Section $section
     * @param  bool    $withUnits
     * @param  bool    $withQuestionSets
     * @return SectionDto
     */
    private function toSectionDto(Section $section, bool $withUnits, bool $withQuestionSets): SectionDto
    {
        // Sectionの翻訳を取得
        $translation = $section->getTranslation();

        // unitsのDTO配列
        $unitsArray = [];
        if ($withUnits && $section->relationLoaded('units')) {
            $unitsArray = $section->units->map(function (Unit $unit) use ($withQuestionSets) {
                $unitTranslation = $unit->getTranslation();

                // question_setsのDTO配列
                $questionSetsArray = [];
                if ($withQuestionSets && $unit->relationLoaded('questionSets')) {
                    $questionSetsArray = $unit->questionSets->map(function (QuestionSet $qs) {
                        $qsTranslation = $qs->translations->firstWhere('locale', app()->getLocale())
                            ?: $qs->translations->firstWhere('locale', config('app.fallback_locale'));

                        return new QuestionSetDto([
                            'id'          => (string) $qs->id,
                            'unit_id'     => (string) $qs->unit_id,
                            'title'       => $qsTranslation?->title ?? '',
                            'description' => $qsTranslation?->description ?? '',
                            'status'      => (int) $qs->status,
                            'order'       => (int) $qs->order,
                        ]);
                    })->toArray();
                }

                return [
                    'id'            => (string) $unit->id,
                    'order'         => (int) $unit->order,
                    'status'        => (int) $unit->status,
                    'version'       => (string) $unit->version,
                    'requirement'   => $unitTranslation?->requirement ?? $unit->requirement,
                    // question_setsをDTO配列化
                    'question_sets' => $questionSetsArray,
                ];
            })->toArray();
        }

        return new SectionDto([
            'id'                => (string) ($section->id ?? ''),
            'subject_id'        => (string) ($section->subject_id ?? ''),
            'level_id'          => (string) ($section->level_id ?? ''),
            'json_id'           => $section->json_id,
            'name'              => $translation?->name ?? $section->name,
            'description'       => $translation?->description ?? $section->description,
            'requirement'       => $translation?->requirement ?? $section->requirement,
            'version'           => (string) ($section->version ?? ''),
            'status'            => (int) ($section->status ?? 0),
            'order'             => (int) ($section->order ?? 0),
            'learning_category' => (string) ($section->learning_category ?? ''),

            'units'             => $unitsArray,
        ]);
    }
}


---現在の実装
Route::middleware(['auth:sanctum'])->group(function () {
Route::prefix('question-sets')->group(function () {
Route::get('/', [QuestionSetController::class, 'index']);
});

        Route::prefix('study-sessions')->group(function () {
            Route::post('/', [\App\Http\Controllers\API\V1\StudySession\StudySessionController::class, 'store'])->name('v1.study-session.store');
            Route::post('/start', [\App\Http\Controllers\API\V1\StudySession\StudySessionController::class, 'begin'])->name('v1.study-session.begin');
        });
    });

<?php

namespace App\Dtos\V1\User;

use App\Dtos\Dto;

class UserQuestionSetDto extends Dto
{
    public string $id;
    public string $user_id;
    public string $question_set_id;
    public int $status;
    public ?float $score;
    public ?string $started_at;
    public ?string $finished_at;
}
<?php

namespace App\Enums;

enum UserQuestionSetStatus: int
{
    case NOT_START = 1;
    case COMPLETE = 100;
    case PROGRESS = 300;

    public function description(): string
    {
        return match ($this) {
            self::NOT_START => 'Not Started',
            self::COMPLETE => 'Complete',
            self::PROGRESS => 'Progress',
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
            default => throw new \InvalidArgumentException("Unknown status string: {$statusString}")
        };
    }
}
<?php

namespace App\Http\Controllers\API\V1\StudySession;

use App\Http\Controllers\Controller;
use App\Http\Requests\API\V1\StudySession\BeginStudySessionRequest;
use App\Http\Requests\API\V1\StudySession\StartStudySessionRequest;
use App\Http\Resources\V1\StudySession\StartStudySessionResource;
use App\Http\Resources\V1\StudySession\BeginStudySessionResource;
use App\UseCases\V1\StudySession\StartStudySessionUseCase;
use App\UseCases\V1\StudySession\BeginStudySessionUseCase;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class StudySessionController extends Controller
{
    public function __construct(
        protected StartStudySessionUseCase $startStudySessionUseCase,
        protected BeginStudySessionUseCase $beginStudySessionUseCase
    ) {
    }

    /**
     * 学習開始API
     * ユーザーが unit_id を指定し、UserQuestionSet を取得または新規作成する
     */
    public function store(StartStudySessionRequest $request)
    {
        $userId = Auth::id(); // ログインユーザーIDの取得
        $unitId = $request->input('unit_id');

        // 該当するUserQuestionSetを取得 or 作成
        $userQuestionSetDto = $this->startStudySessionUseCase->handle($userId, $unitId);

        // Resourceでレスポンス
        return new StartStudySessionResource($userQuestionSetDto);
    }

    /**
     * user_question_set を進行状態にし、最初の問題を返すAPI
     */
    public function begin(BeginStudySessionRequest $request)
    {
        $userId = Auth::id();
        $userQuestionSetId = $request->input('user_question_set_id');

        // ユースケースを実行して、最初に取り組むべき UserQuestion を取得
        $userQuestionDto = $this->beginStudySessionUseCase->handle($userId, $userQuestionSetId);

        // Resourceでレスポンス
        return new BeginStudySessionResource($userQuestionDto);
    }
}
<?php

namespace App\Http\Requests\API\V1\StudySession;

use Illuminate\Foundation\Http\FormRequest;

class BeginStudySessionRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'user_question_set_id' => ['required', 'uuid', 'exists:user_question_sets,id'],
        ];
    }
}
<?php

namespace App\Http\Requests\API\V1\StudySession;

use Illuminate\Foundation\Http\FormRequest;

class StartStudySessionRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'unit_id' => ['required', 'uuid', 'exists:units,id'],
        ];
    }
}
<?php

namespace App\Http\Resources\V1\StudySession;

use Illuminate\Http\Resources\Json\JsonResource;

class BeginStudySessionResource extends JsonResource
{
    public function toArray($request)
    {
        return [
            'user_question_id' => $this->id,
            'question_id' => $this->question_id,
            'status' => $this->status,
            'answer_data' => $this->answer_data,
            'answered_at' => $this->answered_at,
        ];
    }
}
<?php

namespace App\Http\Resources\V1\StudySession;

use Illuminate\Http\Resources\Json\JsonResource;

class StartStudySessionResource extends JsonResource
{
    public function toArray($request)
    {
        return [
            'user_question_set_id' => $this->id,
            'status' => $this->status,
            'started_at' => $this->started_at,
            'finished_at' => $this->finished_at,
        ];
    }
}
<?php

namespace App\Models\User;

use App\Models\BaseModel;
use App\Traits\UsesUuid;
use Illuminate\Database\Eloquent\Model;
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
 * @mixin \Eloquent
 */
class UserQuestionSet extends BaseModel
{
    use UsesUuid, SoftDeletes;

    // 主キーがUUIDであることを指定
    protected $keyType = 'string';
    public $incrementing = false;

}
<?php

namespace App\Services\V1\StudySession;

use App\Models\User\UserQuestionSet;
use App\Models\User\UserQuestion;
use App\Models\Question\QuestionSet;
use App\Models\Question\QuestionSetQuestion;
use App\Enums\UserQuestionSetStatus;
use App\Enums\UserQuestionStatus;
use App\Dtos\V1\User\UserQuestionSetDto;
use App\Dtos\V1\User\UserQuestionDto;
use Illuminate\Support\Str;
use Illuminate\Support\Facades\DB;
use Carbon\Carbon;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

class StudySessionService
{
    /**
     * 指定された userId & unitId をもとに、該当する question_set を検索し、
     * user_question_sets (NOT_START または PROGRESS) があればそれを返却、
     * なければ作成し、紐づく全 question を user_questions として生成する
     */
    public function getOrCreateUserQuestionSet(string $userId, string $unitId): UserQuestionSetDto
    {
        // unit_id をキーに question_sets を取得
        $questionSet = QuestionSet::where('unit_id', $unitId)->first();

        // TODO エラーメッセージ管理
        if (!$questionSet) {
            throw new NotFoundHttpException("QuestionSet not found for unit_id = {$unitId}");
        }

        // ユーザーがまだ学習中または未開始の user_question_set を持っているか確認
        $existing = UserQuestionSet::where('user_id', $userId)
            ->where('question_set_id', $questionSet->id)
            ->whereIn('status', [UserQuestionSetStatus::NOT_START->value, UserQuestionSetStatus::PROGRESS->value])
            ->first();

        if ($existing) {
            return UserQuestionSetDto::fromModel($existing);
        }

        // 無ければ新規作成し、該当の question_set に紐づく question をすべて作成
        DB::beginTransaction();
        try {
            // user_question_sets のレコードを作成
            $userQuestionSet = new UserQuestionSet();
            $userQuestionSet->id = (string) Str::uuid();
            $userQuestionSet->user_id = $userId;
            $userQuestionSet->question_set_id = $questionSet->id;
            $userQuestionSet->status = UserQuestionSetStatus::NOT_START->value;
            $userQuestionSet->score = null;
            $userQuestionSet->started_at = null;
            $userQuestionSet->finished_at = null;
            $userQuestionSet->save();

            // question_set_questions テーブルから、紐づく問題を取得し、user_questions をまとめて生成
            $questionSetQuestions = QuestionSetQuestion::where('question_set_id', $questionSet->id)
                ->orderBy('order', 'asc')
                ->get();

            foreach ($questionSetQuestions as $qsq) {
                $userQuestion = new UserQuestion();
                $userQuestion->id = (string) Str::uuid();
                $userQuestion->user_question_set_id = $userQuestionSet->id;
                $userQuestion->question_id = $qsq->question_id;
                $userQuestion->status = UserQuestionStatus::NOT_START->value;
                $userQuestion->answer_data = null;
                $userQuestion->answered_at = null;
                $userQuestion->save();
            }

            DB::commit();

            return UserQuestionSetDto::fromModel($userQuestionSet);
        } catch (\Throwable $e) {
            DB::rollBack();
            throw $e;
        }
    }

    /**
     * user_question_set の status を PROGRESS にし、
     * 該当 user_question_set の中の最初の問題 (order 昇順) を返す
     */
    public function beginUserQuestionSet(string $userId, string $userQuestionSetId): UserQuestionDto
    {
        // user_question_sets を user_id と user_question_set_id で特定
        $userQuestionSet = UserQuestionSet::where('id', $userQuestionSetId)
            ->where('user_id', $userId)
            ->first();

        if (!$userQuestionSet) {
            throw new NotFoundHttpException("UserQuestionSet not found (id: {$userQuestionSetId}).");
        }

        // もし status が PROGRESS でなければ、PROGRESSに変更
        if ($userQuestionSet->status !== UserQuestionSetStatus::PROGRESS->value) {
            $userQuestionSet->status = UserQuestionSetStatus::PROGRESS->value;
            $userQuestionSet->started_at = Carbon::now();
            $userQuestionSet->save();
        }

        // user_questions の中から最初の問題を取得する
        // user_questions テーブル自体には order が無いので、
        // question_set_questions (order ASC) と join して最初の user_question を特定
        $questionSetId = $userQuestionSet->question_set_id;

        // JOINして最初の問題(user_question)を抽出
        $firstUserQuestion = DB::table('question_set_questions as qsq')
            ->join('user_questions as uq', 'uq.question_id', '=', 'qsq.question_id')
            ->where('qsq.question_set_id', $questionSetId)
            ->where('uq.user_question_set_id', $userQuestionSet->id)
            ->orderBy('qsq.order', 'asc')
            ->select('uq.*')
            ->first();

        if (!$firstUserQuestion) {
            throw new NotFoundHttpException("No question found in user_question_set (id: {$userQuestionSetId}).");
        }

        // 取得したレコードを配列化 → DTOに変換して返す
        $dtoArray = (array)$firstUserQuestion;

        return new UserQuestionDto($dtoArray);
    }
}
<?php

namespace App\UseCases\V1\StudySession;

use App\Dtos\V1\User\UserQuestionDto;
use App\Services\V1\StudySession\StudySessionService;

class BeginStudySessionUseCase
{
    public function __construct(
        protected StudySessionService $studySessionService
    ) {
    }

    /**
     * 指定された userQuestionSetId のステータスを PROGRESS に変更し、
     * 最初の user_questions を返す
     */
    public function handle(string $userId, string $userQuestionSetId): UserQuestionDto
    {
        return $this->studySessionService->beginUserQuestionSet($userId, $userQuestionSetId);
    }
}
<?php

namespace App\UseCases\V1\StudySession;

use App\Dtos\V1\User\UserQuestionSetDto;
use App\Services\V1\StudySession\StudySessionService;

class StartStudySessionUseCase
{
    public function __construct(
        protected StudySessionService $studySessionService
    ) {
    }

    /**
     * 指定された userId・unitId をもとに、UserQuestionSetの取得または作成を行う
     */
    public function handle(string $userId, string $unitId): UserQuestionSetDto
    {
        return $this->studySessionService->getOrCreateUserQuestionSet($userId, $unitId);
    }
}
<?php

namespace App\Models\Unit;

use App\Enums\SectionStatus;
use App\Enums\UnitStatus;
use App\Models\Question\QuestionSet;
use App\Models\Section\Section;
use App\Traits\HasOrder;
use App\Traits\UsesUuid;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

/**
 *
 *
 * @property string $id
 * @property string $subject_id
 * @property string $level_id
 * @property string $section_id
 * @property string|null $json_id
 * @property string|null $description
 * @property string|null $requirement
 * @property string|null $required_competency
 * @property string|null $background
 * @property string $version
 * @property int $status
 * @property int $order
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property \Illuminate\Support\Carbon|null $deleted_at
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Unit newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Unit newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Unit onlyTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Unit query()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Unit whereBackground($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Unit whereCreatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Unit whereDeletedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Unit whereDescription($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Unit whereId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Unit whereJsonId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Unit whereLevelId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Unit whereOrder($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Unit whereRequiredCompetency($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Unit whereRequirement($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Unit whereSectionId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Unit whereStatus($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Unit whereSubjectId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Unit whereUpdatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Unit whereVersion($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Unit withTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Unit withoutTrashed()
 * @property-read bool $is_published
 * @property-read \App\Models\Unit\TFactory|null $use_factory
 * @property-read Section $section
 * @property-read \Illuminate\Database\Eloquent\Collection<int, \App\Models\Unit\UnitTranslation> $translations
 * @property-read int|null $translations_count
 * @method static \Database\Factories\Unit\UnitFactory factory($count = null, $state = [])
 * @method static Builder<static>|Unit ofStatus(\App\Enums\UnitStatus $status)
 * @method static Builder<static>|Unit published()
 * @mixin \Eloquent
 */
class Unit extends Model
{
    use HasOrder, UsesUuid, SoftDeletes, HasFactory;

    // 主キーがUUIDであることを指定
    protected $keyType = 'string';
    public $incrementing = false;

    public $sortable = [
        'order_column_name' => 'order',
        'sort_when_creating' => true,
    ];

    /**
     * このModelの関連テーブルを常にロードするやつ
     * (翻訳を常時読み込みなど)
     */
    protected $with = [
        'translations',
    ];

    protected $fillable = [
        'subject_id',
        'level_id',
        'section_id',
        'json_id',
        'requirement',
        'required_competency',
        'background',
        'version',
        'status',
        'order'
    ];

    /*=====================================================
     * リレーション
     *=====================================================*/

    /**
     * 翻訳テーブルとのリレーション
     */
    public function translations()
    {
        return $this->hasMany(UnitTranslation::class, 'unit_id');
    }

    /**
     * Sectionモデルとのリレーション
     */
    public function section()
    {
        return $this->belongsTo(Section::class);
    }

    /**
     * QuestionSetとのリレーション
     */
    public function questionSets()
    {
        return $this->hasMany(QuestionSet::class);
    }

    /*=====================================================
     * スコープ
     *=====================================================*/

    /**
     * 公開(PUBLISHED)になっているデータだけを抽出するクエリスコープ
     */
    public function scopePublished(Builder $query): Builder
    {
        return $query->where('status', UnitStatus::PUBLISHED->value);
    }

    /**
     * 任意のUnitStatusを指定して抽出するクエリスコープ
     * Unit::ofStatus(UnitStatus::DRAFT)->get();
     */
    public function scopeOfStatus(Builder $query, UnitStatus $status): Builder
    {
        return $query->where('status', $status->value);
    }

    /*=====================================================
     * アクセサ/ミューテータ
     *=====================================================*/

    /**
     * 「このUnitは公開状態か？」をboolで返す
     */
    public function getIsPublishedAttribute(): bool
    {
        return $this->status === UnitStatus::PUBLISHED;
    }

    /*=====================================================
     * カスタムメソッド/Custom Method
     *=====================================================*/

    /**
     * 現在のlocaleに応じた翻訳を返す
     * なければfallback_localeの翻訳を返す
     */
    public function getTranslation(?string $locale = null): ?UnitTranslation
    {
        $locale = $locale ?: app()->getLocale();
        return $this->translations->firstWhere('locale', $locale)
            ?: $this->translations->firstWhere('locale', config('app.fallback_locale'));
    }
}

<?php

namespace App\Enums;

// 問題の正誤を評価するメソッドをマップしたEnum
enum EvaluationCheckerMethod: int
{
    case CALCULATE_METHOD = 1;

    public function label(): string
    {
        return match($this) {
            self::CALCULATE_METHOD          => 'calculateMethod',
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
            'calculateMethod'          => self::CALCULATE_METHOD,
            default => throw new \InvalidArgumentException("Unknown status string: {$statusString}")
        };
    }
}
