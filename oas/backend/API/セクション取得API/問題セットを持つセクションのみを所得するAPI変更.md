以下のsections APIを修正してください
以下の sections API（/api/sections)で、デフォルト仕様は、sections に紐づいている、units が question_sets を持っている sections のみを取得するように修正してください。

そして、question_setsの有無に関係なく全てのsections を取得する為のリクエストパラメータを追加してください

# 実装方針
生成また修正変更するコードはそのコードだけは必ず省略せずに必ず全てアウトプットすること。
・questions, user_questions は、それぞれ xx_translations テーブルを持っています。Accept-Languageに設定されている言語に翻訳した結果をレスポンスすることを忘れないでください

namespace は User

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

レスポンスに　user_question_id　や、user_questions　の status は含めたい。

# 説明
question_sets：問題（questions）を束ねるグループ。Unit（単元）に紐づく
questions: 問題
question_set_questions：question_sets と questions を紐づけるPivotテーブル。questions は複数のquestion_sets に紐づく事があり多対多の関係なのでこのような設計になっている。
user_question_sets: ユーザーが学習した questions_sets。学習開始時に question_sets_id と紐づいて status （UserQuestionSetsStatus::NOT_START) が未開始の状態で生成され、進捗ステータスはスコアなどが管理される
user_questions: ユーザーが学習した questions。学習開始時に、学習を開始した question_sets に紐づく questions が全て、 user_question_sets_id と question_id（questions）を紐づけて status （UserQuestionStatus::NOT_START) が未開始の状態で全てのquestionsの数の分、生成され、進捗ステータスはスコアなどが管理される
sections: セクション
units: ユニット（単元）。セクションに紐づく


ーーテーブル
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
        Schema::create('sections', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('subject_id');
            $table->uuid('level_id');
            $table->uuid('grade_id');
            $table->string('json_id')->nullable()->unique();
            $table->string('name')->nullable();
            $table->text('description')->nullable();
            $table->longText('requirement')->nullable();
            $table->longText('required_competency')->nullable();
            $table->string('version')->default('0.0.1');
            $table->integer('status')->default(1);
            $table->integer('order');
            $table->string('learning_category')->nullable()
                ->comment('分類 e.g. "A", "B"');
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('subject_id')->references('id')->on('subjects')->onDelete('cascade');
            $table->foreign('level_id')->references('id')->on('levels')->onDelete('cascade');
            $table->foreign('grade_id')->references('id')->on('grades')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('sections');
    }
};

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
        Schema::create('units', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('subject_id');
            $table->uuid('level_id');
            $table->uuid('grade_id');
            $table->uuid('section_id');
            $table->string('json_id')->nullable()->unique();
            $table->longText('requirement')->nullable();
            $table->longText('required_competency')->nullable();
            $table->longText('background')->nullable();
            $table->string('version')->default('0.0.1');
            $table->integer('status')->default(1);
            $table->integer('order');
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('subject_id')->references('id')->on('subjects')->onDelete('cascade');
            $table->foreign('level_id')->references('id')->on('levels')->onDelete('cascade');
            $table->foreign('grade_id')->references('id')->on('grades')->onDelete('cascade');
            $table->foreign('section_id')->references('id')->on('sections')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('units');
    }
};

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
        Schema::create('question_sets', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('unit_id')->nullable();
            $table->string('json_id')->nullable()->unique();
            $table->string('version')->default('0.0.1');
            $table->integer('status')->default(1);
            $table->integer('order');
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('unit_id')->references('id')->on('units')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        DB::statement('SET FOREIGN_KEY_CHECKS=0;');
        Schema::dropIfExists('question_sets');
        DB::statement('SET FOREIGN_KEY_CHECKS=1;');
    }
};



<?php

namespace App\Models\Question;

use App\Enums\QuestionSetStatus;
use App\Models\BaseModel;
use App\Traits\HasOrder;
use App\Traits\UsesUuid;
use Illuminate\Database\Eloquent\Builder;
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
 * @property-read \Illuminate\Database\Eloquent\Collection<int, \App\Models\Question\QuestionSetTranslation> $translations
 * @property-read int|null $translations_count
 * @method static Builder<static>|QuestionSet newModelQuery()
 * @method static Builder<static>|QuestionSet newQuery()
 * @method static Builder<static>|QuestionSet onlyTrashed()
 * @method static Builder<static>|QuestionSet published()
 * @method static Builder<static>|QuestionSet query()
 * @method static Builder<static>|QuestionSet whereCreatedAt($value)
 * @method static Builder<static>|QuestionSet whereDeletedAt($value)
 * @method static Builder<static>|QuestionSet whereId($value)
 * @method static Builder<static>|QuestionSet whereJsonId($value)
 * @method static Builder<static>|QuestionSet whereOrder($value)
 * @method static Builder<static>|QuestionSet whereStatus($value)
 * @method static Builder<static>|QuestionSet whereUnitId($value)
 * @method static Builder<static>|QuestionSet whereUpdatedAt($value)
 * @method static Builder<static>|QuestionSet whereVersion($value)
 * @method static Builder<static>|QuestionSet withTrashed()
 * @method static Builder<static>|QuestionSet withoutTrashed()
 * @property-read \Illuminate\Database\Eloquent\Collection<int, \App\Models\Question\QuestionSetQuestion> $questionSetQuestions
 * @property-read int|null $question_set_questions_count
 * @property-read \Illuminate\Database\Eloquent\Collection<int, \App\Models\Question\Question> $questions
 * @property-read int|null $questions_count
 * @mixin \Eloquent
 */
class QuestionSet extends BaseModel
{
    use HasOrder, UsesUuid, SoftDeletes;

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
    * Relation
    *=====================================================*/
    /**
     * 翻訳テーブルとのリレーション
     */
    public function translations()
    {
        return $this->hasMany(QuestionSetTranslation::class, 'question_set_id');
    }

    public function questionSetQuestions()
    {
        // PIVOT: question_set_questions
        return $this->hasMany(QuestionSetQuestion::class, 'question_set_id');
    }

    public function questions()
    {
        return $this->belongsToMany(Question::class, 'question_set_questions', 'question_set_id', 'question_id')
            ->withPivot('order')
            ->withTimestamps();
    }

    /*=====================================================
    * Scope
    *=====================================================*/

    /**
     * 公開(PUBLISHED)だけを絞り込むスコープ
     */
    public function scopePublished(Builder $query): Builder
    {
        return $query->where('status', QuestionSetStatus::PUBLISHED->value);
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
 * @property string $grade_id
 * @method static Builder<static>|Section whereGradeId($value)
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
 * @property-read \Illuminate\Database\Eloquent\Collection<int, QuestionSet> $questionSets
 * @property-read int|null $question_sets_count
 * @property string $grade_id
 * @method static Builder<static>|Unit whereGradeId($value)
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
     * パラメータ:
     *   - with_units=1 でunitsも取得
     *   - with_question_sets=1 でunit内のquestion_setsも取得
     *   - level_id, subject_id で絞り込み
     */
    public function index(SectionListRequest $request)
    {
        $withUnits         = $request->getWithUnits();
        $withQuestionSets  = $request->getWithQuestionSets();
        $levelId           = $request->getLevelId();
        $subjectId         = $request->getSubjectId();

        // UseCaseが SectionDto のコレクションを返却
        $sectionDtoCollection = $this->getSectionListUseCase->handle(
            withUnits:         $withUnits,
            levelId:           $levelId,
            subjectId:         $subjectId,
            withQuestionSets:  $withQuestionSets
        );

        // ResourceにDTOのコレクションをそのまま渡す
        return SectionResource::collection($sectionDtoCollection);
    }

    /**
     * GET /api/v1/sections/{id}
     *
     * パラメータ:
     *   - with_units=1 でunitsも取得
     *   - with_question_sets=1 でunit内のquestion_setsも取得
     */
    public function show(string $id, SectionDetailRequest $request)
    {
        $withUnits        = $request->getWithUnits();
        $withQuestionSets = $request->getWithQuestionSets();

        // UseCaseが単一の SectionDto を返却
        $sectionDto = $this->getSectionDetailUseCase->handle(
            id:               $id,
            withUnits:        $withUnits,
            withQuestionSets: $withQuestionSets
        );

        // ResourceにDTOを渡す
        return new SectionResource($sectionDto);
    }
}
<?php

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
     * @param bool        $withUnits        unitsを含めるかどうか
     * @param string|null $levelId          絞り込み条件
     * @param string|null $subjectId        絞り込み条件
     * @param bool        $withQuestionSets question_setsを含めるかどうか
     * @return Collection  (of SectionDto)
     */
    public function handle(
        bool $withUnits,
        ?string $levelId,
        ?string $subjectId,
        bool $withQuestionSets
    ): Collection {
        return $this->sectionService->getAll(
            withUnits:        $withUnits,
            levelId:          $levelId,
            subjectId:        $subjectId,
            withQuestionSets: $withQuestionSets
        );
    }
}
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
            $query->with([
                'units' => fn($q) =>
                $q->published()->orderBy('order')
                    ->when($withQuestionSets, function ($subQ) {
                        $subQ->with([
                            'questionSets' => fn($qq) => $qq->published()->orderBy('order'),
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
                            'questionSets' => fn($qq) => $qq->published()->orderBy('order'),
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
     */
    private function toSectionDto(Section $section, bool $withUnits, bool $withQuestionSets): SectionDto
    {
        // Sectionの翻訳
        $translation = $section->getTranslation();

        // units のDTO配列
        $unitsArray = [];
        if ($withUnits && $section->relationLoaded('units')) {
            $unitsArray = $section->units->map(function (Unit $unit) use ($withQuestionSets) {
                $unitTranslation = $unit->getTranslation();

                // question_sets のDTO配列
                $questionSetsArray = [];
                if ($withQuestionSets && $unit->relationLoaded('questionSets')) {
                    $questionSetsArray = $unit->questionSets
                        ->map(function (QuestionSet $qs) {
                            // question_set の翻訳
                            $qsTranslation = $qs->translations->firstWhere('locale', app()->getLocale())
                                ?: $qs->translations->firstWhere('locale', config('app.fallback_locale'));

                            // ◆ もし $qs->id が null の場合 → return null でスキップ
                            //    「例外を投げたい」場合は throw new \Exception("Invalid question_set with null id");
                            if (empty($qs->id)) {
                                return null; // この要素は map結果で null となる
                            }

                            return new QuestionSetDto([
                                'id'          => (string) $qs->id,      // nullの場合は map前に return null
                                'unit_id'     => (string) $qs->unit_id, // ここは null を stringキャストすると "" になる
                                'title'       => $qsTranslation?->title ?? '',
                                'description' => $qsTranslation?->description ?? '',
                                'status'      => (int) $qs->status,
                                'order'       => (int) $qs->order,
                            ]);
                        })
                        ->filter()   // mapの結果が null の要素を除外
                        ->toArray(); // 配列に
                }

                return [
                    'id'            => (string) $unit->id,
                    'order'         => (int) $unit->order,
                    'status'        => (int) $unit->status,
                    'version'       => (string) $unit->version,
                    'requirement'   => $unitTranslation?->requirement ?? $unit->requirement,
                    'question_sets' => $questionSetsArray,
                ];
            })->toArray();
        }

        return new SectionDto([
            'id'                => (string) $section->id,
            'subject_id'        => (string) $section->subject_id,
            'level_id'          => (string) $section->level_id,
            'json_id'           => $section->json_id,
            'name'              => $translation?->name ?? $section->name,
            'description'       => $translation?->description ?? $section->description,
            'requirement'       => $translation?->requirement ?? $section->requirement,
            'version'           => (string) $section->version,
            'status'            => (int) $section->status,
            'order'             => (int) $section->order,
            'learning_category' => (string) $section->learning_category,

            'units'             => $unitsArray,
        ]);
    }
}
<?php

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
            'with_units'          => ['sometimes', 'boolean'],
            'with_question_sets'  => ['sometimes', 'boolean'],
            'level_id'            => ['sometimes', 'string'],
            'subject_id'          => ['sometimes', 'string'],
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
     * question_setsを含めるかどうか
     */
    public function getWithQuestionSets(): bool
    {
        return $this->boolean('with_question_sets', false);
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
