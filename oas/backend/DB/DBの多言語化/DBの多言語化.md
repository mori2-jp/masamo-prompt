

DBを多言語化させる為に、翻訳用テーブルを用意する方法 を採用し、xx_translations というテーブルがあります
例）sections, section_translations 


APIのリクエストから、ユーザーが選択している言語をレスポンスするように実装してください


--Model
<?php

namespace App\Models\Section;

use App\Traits\HasOrder;
use App\Traits\UsesUuid;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

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
 * @mixin \Eloquent
 */
class Section extends Model
{
    use HasOrder, UsesUuid, SoftDeletes;

    // 主キーがUUIDであることを指定
    protected $keyType = 'string';
    public $incrementing = false;

    public $sortable = [
        'order_column_name' => 'order',
        'sort_when_creating' => true,
    ];
}

===
<?php

namespace App\Models\Section;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

/**
 * 
 *
 * @property int $id
 * @property string $section_id
 * @property string $locale
 * @property string|null $name
 * @property string|null $description
 * @property string|null $requirement
 * @property string|null $required_competency
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property \Illuminate\Support\Carbon|null $deleted_at
 * @method static \Illuminate\Database\Eloquent\Builder<static>|SectionTranslation newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|SectionTranslation newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|SectionTranslation onlyTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|SectionTranslation query()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|SectionTranslation whereCreatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|SectionTranslation whereDeletedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|SectionTranslation whereDescription($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|SectionTranslation whereId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|SectionTranslation whereLocale($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|SectionTranslation whereName($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|SectionTranslation whereRequiredCompetency($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|SectionTranslation whereRequirement($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|SectionTranslation whereSectionId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|SectionTranslation whereUpdatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|SectionTranslation withTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|SectionTranslation withoutTrashed()
 * @mixin \Eloquent
 */
class SectionTranslation extends Model
{
    use SoftDeletes;

    protected $fillable = [
        'section_id',
        'locale',
        'name',
        'description',
    ];
}

===
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

==
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
        Schema::create('section_translations', function (Blueprint $table) {
            $table->id();
            $table->uuid('section_id');
            $table->string('locale', 10);
            $table->string('name')->nullable();
            $table->text('description')->nullable();
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('section_id')->references('id')->on('sections')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('section_translations');
    }
};

---
<?php
/**
 * Controller (V1/SectionController)
 * sections一覧を取得するエンドポイント
 */
namespace App\Http\Controllers\API\V1\Section;

use App\Http\Controllers\Controller;
use App\Http\Resources\V1\Section\SectionResource; // ← Resource
use App\UseCases\V1\Section\GetSectionListUseCase; // ← UseCase
use Illuminate\Http\Request;

class SectionController extends Controller
{
    public function __construct(
        private GetSectionListUseCase $getSectionListUseCase
    ) {
    }

    /**
     * GET /api/v1/sections
     */
    public function index(Request $request)
    {
        // ユースケースを呼び出してセクション一覧を取得
        $sections = $this->getSectionListUseCase->handle();

        // Resourceを利用して返却
        return SectionResource::collection($sections);
    }
}


---

<?php
// UseCase: app/UseCases/V1/Section/GetSectionDetailUseCase.php
namespace App\UseCases\V1\Section;

use App\Services\V1\Section\SectionService; // ※ Service
use App\Models\Section\Section;
use Illuminate\Database\Eloquent\ModelNotFoundException;

class GetSectionDetailUseCase
{
    public function __construct(
        private SectionService $sectionService
    ) {
    }

    public function handle(string $id): Section
    {
        // セクション単体を取得。見つからなければ404
        $section = $this->sectionService->findById($id);
        if (!$section) {
            throw new ModelNotFoundException("Section not found");
        }
        return $section;
    }
}

--
<?php
// Service: app/Services/V1/Section/SectionService.php
namespace App\Services\V1\Section;

use App\Models\Section\Section;
use Illuminate\Support\Collection;

class SectionService
{
    /**
     * 全てのセクションを取得
     */
    public function getAll(): Collection
    {
        // 必要に応じてwith('translations')など
        return Section::with('translations')->get();
    }

    /**
     * idを指定してセクションを取得
     */
    public function findById(string $id): ?Section
    {
        // 必要に応じてwith('translations')など
        return Section::with('translations')->find($id);
    }
}

ーー
<?php

// app/Http/Middleware/SetLocaleMiddleware.php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\App;

class SetLocaleMiddleware
{
    /**
     * APIリクエストから Locale を取得し、アプリケーションに設定するミドルウェア
     */
    public function handle(Request $request, Closure $next)
    {
        // ※ "Accept-Language" ヘッダーから取得する例
        //    指定がなければ config('app.locale') を使用
        $locale = $request->header('Accept-Language', config('app.locale'));

        // 例: 簡易的に英語(en)/日本語(ja)以外は fallback_locale にするなど
        //     (必要な場合はロジックを拡張してください)
        $supported = ['en', 'ja'];
        if (! in_array($locale, $supported)) {
            $locale = config('app.fallback_locale');
        }

        App::setLocale($locale);

        return $next($request);
    }
}

ーー
<?php
// bootstrap/app.php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;
use Illuminate\Validation\ValidationException;
use Symfony\Component\HttpKernel\Exception\AccessDeniedHttpException;
use Symfony\Component\HttpKernel\Exception\BadRequestHttpException;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        using: function () {
            Route::middleware('api')
                ->domain(config('domain.api'))
                ->prefix('api')
                ->group(base_path('routes/api.php'));

            Route::middleware('web')
                ->group(base_path('routes/web.php'));
        },
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->append(\App\Http\Middleware\SetLocaleMiddleware::class);
    })
    ->withExceptions(function (Exceptions $exceptions) {

        // AuthenticationExceptionのハンドリング
        $exceptions->renderable(function (\Illuminate\Auth\AuthenticationException $e, $request) {
            $detail = '';
            if (!\App::environment('production')) {
                $detail = $e->getMessage();
            }
            if ($request->is('api/*')) {
                return response()->json([
                    'title' => 'Unauthenticated',
                    'detail' => $detail,
                    'errors' => '',
                    'status' => 401,
                ], 401);
            }
        });

        // ValidationExceptionのハンドリング
        $exceptions->renderable(function (ValidationException $e, $request) {
            $detail = '';
            if (!\App::environment('production')) {
                $detail = $e->getMessage();
            }
            if ($request->is('api/*')) {
                return response()->json([
                    'title' => 'Validation Error',
                    'detail' => $detail,
                    'errors' => $e->errors(),
                    'status' => 422,
                ], 422);
            }
        });

        // NotFoundHttpExceptionのハンドリング
        $exceptions->renderable(function (NotFoundHttpException $e, $request) {
            $detail = '';
            if (!\App::environment('production')) {
                $detail = $e->getMessage();
            }
            if ($request->is('api/*')) {
                return response()->json([
                    'title' => 'Record not found',
                    'detail' => $detail,
                    'status' => 404,
                ], 404);
            }
        });

        // BadRequestHttpExceptionのハンドリング
        $exceptions->renderable(function (BadRequestHttpException $e, $request) {
            $detail = '';
            if (!\App::environment('production')) {
                $detail = $e->getMessage();
            }
            if ($request->is('api/*')) {
                return response()->json([
                    'title' => 'Bad Request',
                    'detail' => $detail,
                    'status' => 400,
                ], 400);
            }
        });

        // AccessDeniedHttpExceptionのハンドリング
        $exceptions->renderable(function (AccessDeniedHttpException $e, $request) {
            $detail = '';
            if (!\App::environment('production')) {
                $detail = $e->getMessage();
            }
            if ($request->is('api/*')) {
                return response()->json([
                    'title' => 'Forbidden',
                    'detail' => $detail,
                    'status' => 403,
                ], 403);
            }
        });

        // その他の例外のハンドリング
        $exceptions->renderable(function (Throwable $e, $request) {
            $detail = '';
            if (!\App::environment('production')) {
                $detail = $e->getMessage();
            }
            if ($request->is('api/*')) {
                return response()->json([
                    'title' => 'Internal Server Error',
                    'detail' => $detail,
                    'status' => 500,
                ], 500);
            }
        });
    })->create();
