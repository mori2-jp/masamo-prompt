以下の /sections /sections/{id} の２つのAPIに以下の変更、仕様追加を加えてほしいです。

# 変更点
・with_question_sets パラメータを追加
・with_question_sets が設定されている場合は、units の中に questions_sets を含めてレスポンスする

・section, units、questions_sets は order の昇順でレスポンスする
・section, units、questions_sets は status が Published のものだけレスポンスする

・section, units、questions_setsは、それぞれ xx_translations テーブルを持っています。Accept-Languageに設定されている言語に翻訳した結果をレスポンスすることを忘れないでください
・UnitDtoに、QuestionSetDtoをネストする必要がありますが、SectionDtoと同じようにCastWithを使ってください

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


# 説明
question_sets：問題（questions）を束ねるグループ。Unit（単元）に紐づく
questions: 問題
question_set_questions：question_sets と questions を紐づけるPivotテーブル。questions は複数のquestion_sets に紐づく事があり多対多の関係なのでこのような設計になっている。
user_question_sets: ユーザーが学習した questions_sets。学習開始時に question_sets_id と紐づいて status （UserQuestionSetsStatus::NOT_START) が未開始の状態で生成され、進捗ステータスはスコアなどが管理される
user_questions: ユーザーが学習した questions。学習開始時に、学習を開始した question_sets に紐づく questions が全て、 user_question_sets_id と question_id（questions）を紐づけて status （UserQuestionStatus::NOT_START) が未開始の状態で全てのquestionsの数の分、生成され、進捗ステータスはスコアなどが管理される
sections: セクション
units: ユニット（単元）。セクションに紐づく






---現在の実装
     Route::middleware(['auth:sanctum'])->group(function () {
        Route::prefix('sections')->group(function () {
            Route::get('/', [\App\Http\Controllers\API\V1\Section\SectionController::class, 'index'])->name('v1.sections.index');
            Route::get('/{id}', [\App\Http\Controllers\API\V1\Section\SectionController::class, 'show'])->name('v1.sections.show');
        });
});

<?php
// app/Http/Controllers/API/V1/Section/SectionController.php
namespace App\Http\Controllers\API\V1\Section;

use App\Http\Controllers\Controller;
use App\Http\Requests\API\V1\Section\SectionDetailRequest;
use App\Http\Requests\API\V1\Section\SectionListRequest;
use App\Http\Resources\V1\Section\SectionResource;
use App\UseCases\V1\Section\GetSectionListUseCase;
use App\UseCases\V1\Section\GetSectionDetailUseCase;

class SectionController extends Controller
{
    public function __construct(
        private GetSectionListUseCase $getSectionListUseCase,
        private GetSectionDetailUseCase $getSectionDetailUseCase
    ) {
    }

    /**
     * GET /api/v1/sections
     *
     * @param SectionListRequest $request
     * @return \Illuminate\Http\Resources\Json\AnonymousResourceCollection
     *
     * パラメータ:
     *   - with_units=1 でunitsも取得
     *   - level_id, subject_id で絞り込み
     */
    public function index(SectionListRequest $request)
    {
        $withUnits = $request->getWithUnits();
        $levelId   = $request->getLevelId();
        $subjectId = $request->getSubjectId();

        // UseCaseが SectionDto のコレクションを返却
        $sectionDtoCollection = $this->getSectionListUseCase->handle(
            withUnits:  $withUnits,
            levelId:    $levelId,
            subjectId:  $subjectId,
        );

        // ResourceにDTOのコレクションをそのまま渡す
        return SectionResource::collection($sectionDtoCollection);
    }

    /**
     * GET /api/v1/sections/{id}
     *
     * @param string               $id
     * @param SectionDetailRequest $request
     * @return SectionResource
     *
     * パラメータ:
     *   - with_units=1 でunitsも取得
     */
    public function show(string $id, SectionDetailRequest $request)
    {
        $withUnits = $request->getWithUnits();

        // UseCaseが単一の SectionDto を返却
        $sectionDto = $this->getSectionDetailUseCase->handle($id, $withUnits);

        // ResourceにDTOを渡す
        return new SectionResource($sectionDto);
    }
}


<?php
// app/Http/Requests/API/V1/Section/SectionListRequest.php
namespace App\Http\Requests\API\V1\Section;

use Illuminate\Foundation\Http\FormRequest;

class SectionListRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'with_units' => ['sometimes', 'boolean'],
            'level_id'   => ['sometimes', 'string'],
            'subject_id' => ['sometimes', 'string'],
        ];
    }

    /**
     * unitsを含めるかどうか
     */
    public function getWithUnits(): bool
    {
        return $this->boolean('with_units', false);
    }

    /**
     * level_id 取得
     */
    public function getLevelId(): ?string
    {
        return $this->input('level_id');
    }

    /**
     * subject_id 取得
     */
    public function getSubjectId(): ?string
    {
        return $this->input('subject_id');
    }
}


<?php
// app/Http/Requests/API/V1/Section/SectionDetailRequest.php

namespace App\Http\Requests\API\V1\Section;

use Illuminate\Foundation\Http\FormRequest;

class SectionDetailRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'with_units' => ['sometimes', 'boolean'],
        ];
    }

    /**
     * unitsを含めるかどうか
     */
    public function getWithUnits(): bool
    {
        return $this->boolean('with_units', false);
    }
}
<?php
// app/UseCases/V1/Section/GetSectionListUseCase.php
namespace App\UseCases\V1\Section;

use App\Services\V1\Section\SectionService;
use Illuminate\Support\Collection;

class GetSectionListUseCase
{
    public function __construct(
        private SectionService $sectionService
    ) {
    }

    /**
     * セクション一覧を SectionDto のコレクションで返却する。
     *
     * @param bool        $withUnits   unitsを含めるかどうか
     * @param string|null $levelId     絞り込み条件
     * @param string|null $subjectId   絞り込み条件
     * @return Collection  (of SectionDto)
     */
    public function handle(bool $withUnits, ?string $levelId, ?string $subjectId): Collection
    {
        return $this->sectionService->getAll($withUnits, $levelId, $subjectId);
    }
}
<?php
// app/UseCases/V1/Section/GetSectionDetailUseCase.php
namespace App\UseCases\V1\Section;

use App\Dtos\V1\Section\SectionDto;
use App\Services\V1\Section\SectionService;
use Illuminate\Database\Eloquent\ModelNotFoundException;

class GetSectionDetailUseCase
{
    public function __construct(
        private SectionService $sectionService
    ) {
    }

    /**
     * 指定IDのSectionDtoを返却する。
     * 見つからなければModelNotFoundExceptionを投げる。
     *
     * @param string $id
     * @param bool   $withUnits
     * @return SectionDto
     */
    public function handle(string $id, bool $withUnits): SectionDto
    {
        $sectionDto = $this->sectionService->findById($id, $withUnits);
        if (!$sectionDto) {
            throw new ModelNotFoundException('Section not found');
        }
        return $sectionDto;
    }
}
<?php
// app/Services/V1/Section/SectionService.php

namespace App\Services\V1\Section;

use App\Dtos\V1\Section\SectionDto;
use App\Models\Section\Section;
use App\Models\Unit\Unit;
use Illuminate\Support\Collection;

class SectionService
{
    /**
     * publishedのみ取得し、翻訳済みの値を取り扱いながらSectionDtoのコレクションを返す。
     * $withUnits = true の場合は units もロード＆翻訳して UnitDto 化する。
     *
     * @param bool        $withUnits
     * @param string|null $levelId
     * @param string|null $subjectId
     * @return Collection<SectionDto>
     */
    public function getAll(
        bool $withUnits,
        ?string $levelId,
        ?string $subjectId
    ): Collection {
        $query = Section::published()->orderBy('order');

        if ($withUnits) {
            $query->with(['units' => fn($q) => $q->orderBy('order')]);
        }

        if ($levelId) {
            $query->where('level_id', $levelId);
        }
        if ($subjectId) {
            $query->where('subject_id', $subjectId);
        }

        // Section のコレクションを取得
        $sections = $query->get();

        // Model → SectionDto (翻訳適用) へ変換
        return $sections->map(
            fn (Section $section) => $this->toSectionDto($section, $withUnits)
        );
    }

    /**
     * 指定IDのSectionをpublishedのみから取得し、翻訳済みの SectionDto を返す。
     * 見つからなければ null を返す。
     *
     * @param string $id
     * @param bool   $withUnits
     * @return SectionDto|null
     */
    public function findById(string $id, bool $withUnits): ?SectionDto
    {
        $query = Section::published();

        if ($withUnits) {
            $query->with(['units' => fn($q) => $q->orderBy('order')]);
        }

        $section = $query->find($id);
        if (!$section) {
            return null;
        }

        return $this->toSectionDto($section, $withUnits);
    }

    /**
     * Section モデルを翻訳込みで SectionDto に変換するための共通処理
     *
     * ここで各プロパティを「必ず型に合うように」補正している。
     */
    private function toSectionDto(Section $section, bool $withUnits): SectionDto
    {
        $translation = $section->getTranslation();

        $unitsArray = [];
        if ($withUnits && $section->relationLoaded('units')) {
            $unitsArray = $section->units->map(function (Unit $unit) {
                $translation = $unit->getTranslation();
                return [
                    'id'          => $unit->id,
                    'order'       => $unit->order,
                    'status'      => $unit->status,
                    'version'     => $unit->version,
                    'requirement' => $translation?->requirement ?? $unit->requirement,
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
            'learning_category' => $section->learning_category,
            'units'             => $unitsArray,
        ]);
    }
}
<?php
// app/Dtos/V1/Section/SectionDto.php
namespace App\Dtos\V1\Section;

use App\Dtos\Dto;
use App\Dtos\V1\Unit\UnitCollectionCaster;
use Illuminate\Support\Collection;
use Spatie\DataTransferObject\Attributes\CastWith;

class SectionDto extends Dto
{
    public string $id;
    public string $subject_id;
    public string $level_id;
    public string $json_id;
    public string $name;
    public ?string $description;
    public string $requirement;
    public string $version;
    public int $status;
    public int $order;
    public string $learning_category;

    #[CastWith(UnitCollectionCaster::class)]
    public ?Collection $units;
}
<?php
// app/Http/Resources/V1/Section/SectionResource.php
namespace App\Http\Resources\V1\Section;

use App\Dtos\V1\Section\SectionDto;
use App\Http\Resources\V1\Unit\UnitResource;
use Illuminate\Http\Resources\Json\JsonResource;

class SectionResource extends JsonResource
{
    /**
     * @param  \Illuminate\Http\Request  $request
     * @return array
     *
     * このResourceは SectionDto を $this->resource として受け取る想定。
     */
    public function toArray($request): array
    {
        /** @var SectionDto $dto */
        $dto = $this->resource;

        return [
            'id'                => $dto->id,
            'subject_id'        => $dto->subject_id,
            'level_id'          => $dto->level_id,
            'json_id'           => $dto->json_id,
            'name'              => $dto->name,
            'description'       => $dto->description,
            'requirement'       => $dto->requirement,
            'version'           => $dto->version,
            'status'            => $dto->status,
            'order'             => $dto->order,
            'learning_category' => $dto->learning_category,

            'units' => UnitResource::collection($dto->units),
        ];
    }
}
<?php
// app/Http/Resources/V1/Unit/UnitResource.php
namespace App\Http\Resources\V1\Unit;

use App\Dtos\V1\Unit\UnitDto;
use Illuminate\Http\Resources\Json\JsonResource;

class UnitResource extends JsonResource
{
    /**
     * @param  \Illuminate\Http\Request  $request
     * @return array
     *
     * このResourceは UnitDto を $this->resource として受け取る想定。
     */
    public function toArray($request): array
    {
        /** @var UnitDto $dto */
        $dto = $this->resource;

        return [
            'id'         => $dto->id,
            'order'      => $dto->order,
            'status'     => $dto->status,
            'version'    => $dto->version,
            'requirement'=> $dto->requirement,
        ];
    }
}
<?php

namespace App\Http\Resources\V1\Question;

use Illuminate\Http\Resources\Json\JsonResource;

class QuestionSetResource extends JsonResource
{
    /**
     * DTO (QuestionSetDto) を配列に変換する
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request): array
    {
        return [
            'id'          => $this->id,
            'unit_id'     => $this->unit_id,
            'title'       => $this->title,
            'description' => $this->description,
            'status'      => $this->status,
            'order'       => $this->order,
        ];
    }
}

<?php

namespace App\Dtos\V1\Question;

use App\Dtos\Dto;
use App\Models\Question\QuestionSet;
use App\Models\Question\QuestionSetTranslation;
use Carbon\Carbon;

/**
 * @property string $id
 * @property string|null $unit_id
 * @property string|null $title
 * @property string|null $description
 * @property int $status
 * @property int $order
 */
class QuestionSetDto extends Dto
{
    public string $id;
    public ?string $unit_id;
    public ?string $title;
    public ?string $description;
    public int $status;
    public int $order;

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

enum UnitStatus: int
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

namespace App\Enums;

enum QuestionSetStatus: int
{
    case DRAFT = 1;         // 下書き
    case PUBLISHED = 300;   // 公開中
    case HIDDEN = 600;      // 非公開
    case TEST_PUBLISHED = 900; // テスト公開

    /**
     * ラベルを取得
     */
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
     * 文字列から QuestionSetStatus を取得
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

namespace App\Models\Question;

use App\Models\BaseModel;
use App\Traits\HasOrder;
use App\Traits\UsesUuid;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

/**
 *
 *
 * @property string $id
 * @property string|null $unit_id
 * @property string|null $json_id
 * @property string $version
 * @property int $status
 * @property int $order
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property \Illuminate\Support\Carbon|null $deleted_at
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet onlyTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet query()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet whereCreatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet whereDeletedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet whereId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet whereJsonId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet whereOrder($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet whereStatus($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet whereUnitId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet whereUpdatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet whereVersion($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet withTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet withoutTrashed()
 * @mixin \Eloquent
 */
class QuestionSet extends BaseModel
{
    use HasOrder, UsesUuid, SoftDeletes;

    // 主キーがUUIDであることを指定
    protected $keyType = 'string';
    public $incrementing = false;

    public $sortable = [
        'order_column_name' => 'order',
        'sort_when_creating' => true,
    ];
    protected $with = [
        'translations',
    ];

    /*=====================================================
    * リレーション
    *=====================================================*/

    /**
     * 翻訳テーブルとのリレーション
     */
    public function translations()
    {
        return $this->hasMany(QuestionSetTranslation::class, 'question_set_id');
    }

}
<?php

namespace App\Models\Section;

use App\Enums\SectionStatus;
use App\Models\BaseModel;
use App\Models\Unit\Unit;
use App\Traits\HasOrder;
use App\Traits\UsesUuid;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Database\Eloquent\Builder;

/**
 * 
 *
 * @property string $id
 * @property string $subject_id
 * @property string $level_id
 * @property string|null $json_id
 * @property string|null $name
 * @property string|null $description
 * @property string|null $requirement
 * @property string|null $required_competency
 * @property string $version
 * @property int $status
 * @property int $order
 * @property string|null $learning_subject 科目 (学習要件) e.g. "Arithmetic"
 * @property int|null $learning_no 学習要件の番号 e.g. 10
 * @property string|null $learning_requirement 学習要件の内容 "Numbers and Calculation..."
 * @property string|null $learning_required_competency 必要水準 "Understand multiplication..."
 * @property string|null $learning_category 分類 e.g. "A", "B"
 * @property string|null $learning_grade_level 学年 e.g. "Grade 2"
 * @property string|null $learning_url URLリンク e.g. "https://docs.google.com/..."
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property \Illuminate\Support\Carbon|null $deleted_at
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section onlyTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section query()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereCreatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereDeletedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereDescription($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereJsonId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereLearningCategory($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereLearningGradeLevel($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereLearningNo($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereLearningRequiredCompetency($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereLearningRequirement($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereLearningSubject($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereLearningUrl($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereLevelId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereName($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereOrder($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereRequiredCompetency($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereRequirement($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereStatus($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereSubjectId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereUpdatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereVersion($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section withTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section withoutTrashed()
 * @property-read bool $is_published
 * @property-read \App\Models\Section\TFactory|null $use_factory
 * @property-read \Illuminate\Database\Eloquent\Collection<int, \App\Models\Section\SectionTranslation> $translations
 * @property-read int|null $translations_count
 * @property-read \Illuminate\Database\Eloquent\Collection<int, Unit> $units
 * @property-read int|null $units_count
 * @method static \Database\Factories\Section\SectionFactory factory($count = null, $state = [])
 * @method static Builder<static>|Section ofStatus(\App\Enums\SectionStatus $status)
 * @method static Builder<static>|Section published()
 * @mixin \Eloquent
 */
class Section extends BaseModel
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
        'json_id',
        'name',
        'description',
        'requirement',
        'required_competency',
        'version',
        'status',
        'order',
        'learning_category'
    ];

    /*=====================================================
     * リレーション
     *=====================================================*/

    /**
     * 翻訳テーブルとのリレーション
     */
    public function translations()
    {
        return $this->hasMany(SectionTranslation::class, 'section_id');
    }

    /**
     * Unitモデルとのリレーション例
     */
    public function units()
    {
        return $this->hasMany(Unit::class);
    }

    /*=====================================================
     * スコープ
     *=====================================================*/

    /**
     * 公開(PUBLISHED)になっているデータだけを抽出するクエリスコープ
     */
    public function scopePublished(Builder $query): Builder
    {
        return $query->where('status', SectionStatus::PUBLISHED->value);
    }

    /**
     * 任意のSectionStatusを指定して抽出するクエリスコープ
     * Section::ofStatus(SectionStatus::DRAFT)->get();
     */
    public function scopeOfStatus(Builder $query, SectionStatus $status): Builder
    {
        return $query->where('status', $status->value);
    }

    /*=====================================================
     * アクセサ/ミューテータ
     *=====================================================*/

    /**
     * 「このSectionは公開状態か？」をboolで返す
     */
    public function getIsPublishedAttribute(): bool
    {
        return $this->status === SectionStatus::PUBLISHED;
    }

    /*=====================================================
     * カスタムメソッド/Custom Method
     *=====================================================*/

    /**
     * 現在のlocaleに応じた翻訳を返す
     * なければfallback_localeの翻訳を返す
     */
    public function getTranslation(?string $locale = null): ?SectionTranslation
    {
        $locale = $locale ?: app()->getLocale();
        return $this->translations->firstWhere('locale', $locale)
            ?: $this->translations->firstWhere('locale', config('app.fallback_locale'));
    }
}
<?php
// app/Dtos/V1/Unit/UnitDto.php
namespace App\Dtos\V1\Unit;

use App\Dtos\Dto;

class UnitDto extends Dto
{
    public string $id;
    public int $order;
    public int $status;
    public string $version;
    public string $requirement;
}
<?php
// app/Dtos/V1/Unit/UnitCollectionCaster.php
namespace App\Dtos\V1\Unit;

use Illuminate\Support\Collection;
use Spatie\DataTransferObject\Caster;

class UnitCollectionCaster implements Caster
{
    /**
     * @param  mixed  $value
     * @return \Illuminate\Support\Collection
     */
    public function cast(mixed $value): Collection
    {
        // array 形式で渡ってくる想定なので、Collection化 → UnitDtoへmap
        return collect($value)->mapInto(UnitDto::class);
    }
}
<?php
// app/Dtos/V1/Section/SectionDto.php
namespace App\Dtos\V1\Section;

use App\Dtos\Dto;
use App\Dtos\V1\Unit\UnitCollectionCaster;
use Illuminate\Support\Collection;
use Spatie\DataTransferObject\Attributes\CastWith;

class SectionDto extends Dto
{
    public string $id;
    public string $subject_id;
    public string $level_id;
    public string $json_id;
    public string $name;
    public ?string $description;
    public string $requirement;
    public string $version;
    public int $status;
    public int $order;
    public string $learning_category;

    #[CastWith(UnitCollectionCaster::class)]
    public ?Collection $units;
}
