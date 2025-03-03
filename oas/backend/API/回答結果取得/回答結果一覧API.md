学習が完了したUserQuestionSetを全て取得するバックエンドAPI設計

学習したUserQuestionSetの一覧を取得するAPIを以下にならって実装してほしい


# 実装方針
生成また修正変更するコードはそのコードだけは必ず省略せずに必ず全てアウトプットすること。
・questions, user_questions は、それぞれ xx_translations テーブルを持っています。Accept-Languageに設定されている言語に翻訳した結果をレスポンスすることを忘れないでください

namespace は User

user_question_sets の全部と、それに紐づく user_questions  をレスポンスすｒ．

user_question_sets の status が UserQuestionSetStatus::COMPLETE でなければ、
その user_questions_sets に紐づく user_questions の中で status が UserQuestionStatus::PROGRESS のもの、
もしくはstatus が UserQuestionStatus::PROGRESS のものがなければ、status が UserQuestionStatus::NOT_STRATED の中で order が一番若いものを
nextUserQuestionとしてレスポンスすること。

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

レスポンスに　user_question_id　や、user_questions　の status は含めたいし
next_user_question　は　id だけでよい。無ければNULL
user_questions.questions は不用

# 説明
question_sets：問題（questions）を束ねるグループ
questions: 問題
question_set_questions：question_sets と questions を紐づけるPivotテーブル。questions は複数のquestion_sets に紐づく事があり多対多の関係なのでこのような設計になっている。
user_question_sets: ユーザーが学習した questions_sets。学習開始時に question_sets_id と紐づいて status （UserQuestionSetsStatus::NOT_START) が未開始の状態で生成され、進捗ステータスはスコアなどが管理される
user_questions: ユーザーが学習した questions。学習開始時に、学習を開始した question_sets に紐づく questions が全て、 user_question_sets_id と question_id（questions）を紐づけて status （UserQuestionStatus::NOT_START) が未開始の状態で全てのquestionsの数の分、生成され、進捗ステータスはスコアなどが管理される



ーーテーブル


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
        Schema::create('user_questions', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('user_question_set_id');
            $table->uuid('question_id');
            $table->integer('status')->default(1);
            $table->json('answer_data')->nullable();
            $table->timestamp('answered_at')->nullable();
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('user_question_set_id')->references('id')->on('user_question_sets')->onDelete('cascade');
            $table->foreign('question_id')->references('id')->on('questions')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('user_questions');
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
        Schema::create('questions', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('level_id');
            $table->uuid('difficulty_id');
            $table->string('json_id')->nullable()->unique();
            $table->json('metadata')->nullable();
            $table->string('version')->default('0.0.1');
            $table->integer('status')->default(1);
            $table->integer('evaluation_method')->default(1);
            $table->integer('checker_method')->nullable();
            $table->integer('llm_evaluation_prompt_number')->nullable();
            $table->json('llm_evaluation_response_format')->nullable();
            $table->integer('question_type')->default(1);
            $table->json('learning_requirement_json')->nullable();
            $table->string('learning_subject')->nullable()
                ->comment('科目 (学習要件) e.g. "Arithmetic"');
            $table->integer('learning_no')->nullable()
                ->comment('学習要件の番号 e.g. 10');
            $table->text('learning_requirement')->nullable()
                ->comment('学習要件の内容 "Numbers and Calculation..."');
            $table->text('learning_required_competency')->nullable()
                ->comment('必要水準 "Understand multiplication..."');
            $table->string('learning_category')->nullable()
                ->comment('分類 e.g. "A", "B"');
            $table->string('learning_grade_level')->nullable()
                ->comment('学年 e.g. "Grade 2"');
            $table->string('learning_url')->nullable()
                ->comment('URLリンク e.g. "https://docs.google.com/..."');
            $table->integer('order');
            $table->boolean('generated_by_llm')->default(false);
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('level_id')->references('id')->on('levels')->onDelete('cascade');
            $table->foreign('difficulty_id')->references('id')->on('difficulties')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        DB::statement('SET FOREIGN_KEY_CHECKS=0;');
        Schema::dropIfExists('questions');
        DB::statement('SET FOREIGN_KEY_CHECKS=1;');
    }
};










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
 * @property string $level_id
 * @property string $difficulty_id
 * @property string|null $json_id
 * @property string|null $metadata
 * @property string $version
 * @property int $status
 * @property int $question_type
 * @property string|null $learning_subject 科目 (学習要件) e.g. "Arithmetic"
 * @property int|null $learning_no 学習要件の番号 e.g. 10
 * @property string|null $learning_requirement 学習要件の内容 "Numbers and Calculation..."
 * @property string|null $learning_required_competency 必要水準 "Understand multiplication..."
 * @property string|null $learning_category 分類 e.g. "A", "B"
 * @property string|null $learning_grade_level 学年 e.g. "Grade 2"
 * @property string|null $learning_url URLリンク e.g. "https://docs.google.com/..."
 * @property int $order
 * @property int $generated_by_llm
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property \Illuminate\Support\Carbon|null $deleted_at
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question onlyTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question query()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereCreatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereDeletedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereDifficultyId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereGeneratedByLlm($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereJsonId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLearningCategory($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLearningGradeLevel($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLearningNo($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLearningRequiredCompetency($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLearningRequirement($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLearningSubject($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLearningUrl($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLevelId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereMetadata($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereOrder($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereQuestionFormat($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereQuestionType($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereStatus($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereUpdatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereVersion($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question withTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question withoutTrashed()
 * @property int $evaluation_method
 * @property int|null $checker_method
 * @property-read \Illuminate\Database\Eloquent\Collection<int, \App\Models\Question\QuestionTranslation> $translations
 * @property-read int|null $translations_count
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereCheckerMethod($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereEvaluationMethod($value)
 * @property string|null $llm_evaluation_prompt
 * @property string|null $llm_evaluation_response_format
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLlmEvaluationPrompt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLlmEvaluationResponseFormat($value)
 * @property int|null $llm_evaluation_prompt_number
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLlmEvaluationPromptNumber($value)
 * @property string|null $learning_requirement_json
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLearningRequirementJson($value)
 * @mixin \Eloquent
 */
class Question extends BaseModel
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
        return $this->hasMany(QuestionTranslation::class, 'question_id');
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
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion onlyTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion query()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion withTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion withoutTrashed()
 * @property string $id
 * @property string $user_question_set_id
 * @property string $question_id
 * @property int $status
 * @property string|null $answer_data
 * @property string|null $answered_at
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property \Illuminate\Support\Carbon|null $deleted_at
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereAnswerData($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereAnsweredAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereCreatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereDeletedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereQuestionId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereStatus($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereUpdatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereUserQuestionSetId($value)
 * @mixin \Eloquent
 */
class UserQuestion extends BaseModel
{
    use UsesUuid, SoftDeletes;

    // 主キーがUUIDであることを指定
    protected $keyType = 'string';
    public $incrementing = false;

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
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion onlyTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion query()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion withTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion withoutTrashed()
 * @property string $id
 * @property string $user_question_set_id
 * @property string $question_id
 * @property int $status
 * @property string|null $answer_data
 * @property string|null $answered_at
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property \Illuminate\Support\Carbon|null $deleted_at
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereAnswerData($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereAnsweredAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereCreatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereDeletedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereQuestionId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereStatus($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereUpdatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereUserQuestionSetId($value)
 * @mixin \Eloquent
 */
class UserQuestion extends BaseModel
{
    use UsesUuid, SoftDeletes;

    // 主キーがUUIDであることを指定
    protected $keyType = 'string';
    public $incrementing = false;

}
<?php

namespace App\Dtos;

use App\Traits\InMemorySearch;
use Carbon\Carbon;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\Collection;
use Spatie\DataTransferObject\DataTransferObject;

abstract class Dto extends DataTransferObject
{
    use InMemorySearch;

    public function matchesQuery(string|null $query): bool
    {
        if (is_null($query) || $query === "") {
            return true;
        }
        return self::objectMatchesQuery(Dto::toArrayDeep($this), $query);
    }

    public static function fromModel(?Model $model, ?array $exceptKeys = []): ?self
    {
        if (is_null($model)) {
            return null;
        }
        $object = get_called_class();
        $array = $model->toArray();
        foreach ($exceptKeys as $key) {
            if (key_exists($key, $array)) {
                unset($array[$key]);
            }
        }
        // CarbonのtoStringがUTCで吐き出されてしまうので、ここで強制変更
        if ($model->getDates()) {
            foreach ($model->getDates() as $key) {
                if (isset($array[$key]) && !is_null($array[$key])) {
                    $array[$key] = Carbon::parse($model->{$key})->format('Y-m-d H:i:s');
                }
            }
        }
        return new $object($array);
    }

    public static function toArrayDeep(DataTransferObject $dto): array
    {
        $data = $dto->toArray();

        foreach ($data as $key => $value) {
            if ($value instanceof Collection) {
                $data[$key] = self::toArrayDeepFromCollection($value);
            }
            if ($value instanceof Dto) {
                $data[$key] = self::toArrayDeep($value);
            }
        }

        return $data;
    }

    public static function toArrayDeepFromCollection(Collection $collection): array
    {
        return $collection->map(function ($e) {
            if ($e instanceof Collection) {
                return self::toArrayDeepFromCollection($e);
            }
            if ($e instanceof DataTransferObject) {
                return self::toArrayDeep($e);
            }
            return $e;
        })->toArray();
    }
}
<?php

namespace App\Dtos\V1\User;

use App\Dtos\Dto;
use App\Dtos\V1\Question\QuestionDto;

class UserQuestionDto extends Dto
{
    public string $id;
    public string $question_id;
    public int $status;
    public ?string $answer_data;
    public ?string $answered_at;

    public ?QuestionDto $question;
}

<?php

namespace App\Dtos\V1\User;

use App\Dtos\Dto;
use App\Dtos\V1\Question\QuestionSetDto;
use App\Dtos\V1\Question\QuestionSetDtoCaster;
use Spatie\DataTransferObject\Attributes\CastWith;
use App\Dtos\V1\Question\QuestionSetCollectionCaster;

class UserQuestionSetDto extends Dto
{
    public string $id;
    public string $user_id;
    public string $question_set_id;
    public int $status;
    public ?float $score;
    public ?string $started_at;
    public ?string $finished_at;

    public ?QuestionSetDto $question_set;
}
<?php

namespace App\Dtos\V1\User;

use App\Dtos\Dto;
use Illuminate\Support\Collection;

/**
 * user_question_sets と それに紐づく user_questions (Dto)、
 * nextUserQuestion
 */
class UserQuestionSetDetailDto extends Dto
{
    public string $id;
    public string $user_id;
    public int $status;
    public ?float $score;
    public ?string $started_at;
    public ?string $finished_at;

    /** @var Collection<UserQuestionDto> */
    public Collection $user_questions;

    /** @var UserQuestionDto|null */
    public ?UserQuestionDto $next_user_question;
}
<?php

namespace App\Http\Controllers\API\V1\User;

use App\Http\Controllers\Controller;
use App\Http\Resources\V1\User\UserQuestionSetDetailResource;
use App\UseCases\V1\User\GetUserQuestionSetDetailUseCase;
use Illuminate\Http\Request;

class UserQuestionSetController extends Controller
{
    public function __construct(
        protected GetUserQuestionSetDetailUseCase $getUserQuestionSetDetailUseCase
    ) {
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

        return new UserQuestionSetDetailResource($dto);
    }
}
<?php

namespace App\Http\Resources\V1\User;

use App\Dtos\V1\User\UserQuestionDto;
use App\Dtos\V1\User\UserQuestionSetDetailDto;
use Illuminate\Http\Resources\Json\JsonResource;

class UserQuestionSetDetailResource extends JsonResource
{
    /**
     * @param \Illuminate\Http\Request $request
     * @return array
     *
     * $this->resource is UserQuestionSetDetailDto
     */
    public function toArray($request)
    {
        /** @var UserQuestionSetDetailDto $dto */
        $dto = $this->resource;

        return [
            'id'          => $dto->id,
            'user_id'     => $dto->user_id,
            'status'      => $dto->status,
            'score'       => $dto->score,
            'started_at'  => $dto->started_at,
            'finished_at' => $dto->finished_at,

            'user_questions' => $dto->user_questions->map(function(UserQuestionDto $uqDto) {
                return [
                    'id'          => $uqDto->id,
                    'question_id' => $uqDto->question_id,
                    'status'      => $uqDto->status,
                    'answer_data' => json_decode($uqDto->answer_data),
                    'answered_at' => $uqDto->answered_at,
                ];
            })->toArray(),

            'next_user_question' => $dto->next_user_question ? $dto->next_user_question->id  : null,
        ];
    }
}
<?php

namespace App\Services\V1\User;

use App\Dtos\V1\User\UserQuestionDto;
use App\Dtos\V1\User\UserQuestionSetDetailDto;
use App\Dtos\V1\Question\QuestionDto;
use App\Enums\UserQuestionSetStatus;
use App\Enums\UserQuestionStatus;
use App\Models\Question\Question;
use App\Models\User\UserQuestion;
use App\Models\User\UserQuestionSet;
use Illuminate\Support\Collection;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

class UserQuestionSetService
{
    /**
     * user_question_set_id から学習情報を取得し、
     * 完了していなければ nextUserQuestion を算出してDTOへ詰める
     */
    public function getUserQuestionSetDetail(
        string $userQuestionSetId,
        ?string $locale,
        ?string $userId = null
    ): UserQuestionSetDetailDto {
        // 1) Eagerロード (userQuestions->question->translations)
        $userQuestionSet = UserQuestionSet::with([
            'userQuestions.question.translations'
        ])
            ->where('id', $userQuestionSetId)
            ->first();

        if (!$userQuestionSet) {
            throw new NotFoundHttpException("UserQuestionSet not found for id: {$userQuestionSetId}");
        }

        // 2) 所有権チェック (必要なら)
        if ($userId && $userQuestionSet->user_id !== $userId) {
            throw new NotFoundHttpException("UserQuestionSet does not belong to this user.");
        }

        // 3) nextUserQuestion
        $nextUserQuestion = null;
        if ($userQuestionSet->status !== UserQuestionSetStatus::COMPLETE->value) {
            $nextUserQuestion = $this->findNextUserQuestion($userQuestionSet);
        }

        // 4) DTO化
        return $this->toUserQuestionSetDetailDto(
            $userQuestionSet,
            $nextUserQuestion,
            $locale
        );
    }

    /**
     * nextUserQuestion を抽出するロジック:
     * 1. status=PROGRESS があればそれを返す(複数なら最初)
     * 2. なければ status=NOT_START の中で question->order が最小のもの
     */
    private function findNextUserQuestion(UserQuestionSet $userQuestionSet): ?UserQuestion
    {
        $userQuestions = $userQuestionSet->userQuestions;

        // 1) status=PROGRESS
        $progressQ = $userQuestions
            ->filter(fn($uq) => $uq->status === UserQuestionStatus::PROGRESS->value)
            ->first();
        if ($progressQ) {
            return $progressQ;
        }

        // 2) status=NOT_START の中で order が一番小さいもの
        $notStartQs = $userQuestions
            ->filter(fn($uq) => $uq->status === UserQuestionStatus::NOT_START->value);

        // question.order で sort
        $sorted = $notStartQs->sortBy(function ($uq) {
            // question->order
            return $uq->question->order ?? 999999;
        })->values();

        return $sorted->first() ?: null;
    }

    /**
     * UserQuestionSet + userQuestions + nextUserQuestion をまとめて
     * UserQuestionSetDetailDto に変換
     */
    private function toUserQuestionSetDetailDto(
        UserQuestionSet $userQuestionSet,
        ?UserQuestion $nextUserQuestion,
        ?string $locale
    ): UserQuestionSetDetailDto {
        // user_questions のDTO化
        $userQuestionsDto = $userQuestionSet->userQuestions->map(
            fn($uq) => $this->toUserQuestionDto($uq, $locale)
        );

        // next
        $nextDto = $nextUserQuestion
            ? $this->toUserQuestionDto($nextUserQuestion, $locale)
            : null;

        return new UserQuestionSetDetailDto([
            'id'          => (string) $userQuestionSet->id,
            'user_id'     => (string) $userQuestionSet->user_id,
            'status'      => (int) $userQuestionSet->status,
            'score'       => $userQuestionSet->score,
            'started_at'  => $userQuestionSet->started_at,
            'finished_at' => $userQuestionSet->finished_at,

            'user_questions'    => $userQuestionsDto,
            'next_user_question'=> $nextDto,
        ]);
    }

    /**
     * user_questions + question の翻訳を適用し、UserQuestionDtoを作る
     */
    private function toUserQuestionDto(UserQuestion $userQuestion, ?string $locale): UserQuestionDto
    {
        $questionModel = $userQuestion->question;
        $translation = null;
        if ($questionModel && $questionModel->relationLoaded('translations')) {
            $translation = $questionModel->translations->firstWhere('locale', $locale)
                ?: $questionModel->translations->firstWhere('locale', config('app.fallback_locale'));
        }

        $questionText = $translation?->question_text ?? '';
        $explanation  = $translation?->explanation ?? '';

        return new UserQuestionDto([
            'id'          => (string) $userQuestion->id,
            'question_id' => (string) $userQuestion->question_id,
            'status'      => (int) $userQuestion->status,
            'answer_data' => $userQuestion->answer_data,
            'answered_at' => $userQuestion->answered_at,

            'question' => new QuestionDto([
                'id'             => (string) ($questionModel->id ?? ''),
                'question_text'  => $questionText,
                'explanation'    => $explanation,
                'metadata'       => $questionModel->metadata ?? null,
                'version'        => $questionModel->version ?? '',
                'question_type'  => $questionModel->question_type ?? 1,
                'status'         => $questionModel->status ?? 1,
            ]),
        ]);
    }
}
<?php

namespace App\UseCases\V1\User;

use App\Dtos\V1\User\UserQuestionSetDetailDto;
use App\Services\V1\User\UserQuestionSetService;

class GetUserQuestionSetDetailUseCase
{
    public function __construct(
        protected UserQuestionSetService $userQuestionSetService
    ) {
    }

    /**
     * user_question_set_id から情報を取得し、DTOを生成
     *
     * @param string $userQuestionSetId
     * @param string|null $locale
     * @param string|null $userId (オーナーチェックする場合)
     * @return UserQuestionSetDetailDto
     */
    public function handle(
        string $userQuestionSetId,
        ?string $locale,
        ?string $userId = null
    ): UserQuestionSetDetailDto {
        return $this->userQuestionSetService->getUserQuestionSetDetail(
            $userQuestionSetId,
            $locale,
            $userId
        );
    }
}
