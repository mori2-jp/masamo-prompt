GET /auth/me （ログインユーザー取得）
PUT /auth/me （ログインユーザー更新）
の２つのAPIを実装してください

Service や UseCase の実装は参考に section を提示しました

＃条件
1
id, user_name, updated_at, email（test****@gmail.com　のようにマスクする）をレスポンスする

2
user_name と email を更新できる

3
name space は auth

4
controler は AuthController の既存を使う

5
resource は MeResource を使う。ただしdto で値の受け渡しをする

6
dto は service で生成する（fromModel などを使わずに手動で生成）

7
メールアドレスのマスクは、commonLib に実装




# 実装方針
生成また修正変更するコードはそのコードだけは必ず省略せずに必ず全てアウトプットすること。

修正する場合は修正したコードは全て出力し、修正箇所以外のコードやコメントは現状のままにしてください。

仕様はソースコードにまとめてコメントとして残すこと
コメントを残す時に以下のようにコメントへ処理順を示すようなナンバリングは不用です。
// 7) memo => 空白
// 8) generate_question_prompt => そのまま
// 9) generate_question_prompt_file_name => そのまま

ちゃんと解説してね

変更のブランチ名とコミットメッセージを一行で

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


```php
<?php

namespace App\Dtos\V1\Auth;

use App\Dtos\Dto;

class MeDto extends Dto
{
    public string $id;
    public ?string $user_name;
    public ?string $email;
    public ?string $updated_at;
}

<?php

namespace App\Http\Resources\V1\Auth;

use Illuminate\Http\Resources\Json\JsonResource;

class MeResource extends JsonResource
{
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'user_name' => $this->user_name ?? null,
            'updated_at' => $this->updated_at ?? null,
        ];
    }
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

namespace App\Http\Controllers\API\V1\Auth;

use App\Http\Controllers\Controller;
use App\Http\Requests\API\V1\Auth\LoginRequest;
use App\Http\Resources\V1\Auth\AuthTokenResource;
use App\Http\Resources\V1\Auth\MeResource;
use App\UseCases\V1\Auth\LoginWithGoogleUseCase;
use App\UseCases\V1\Auth\UpdateMeUseCase;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Validation\ValidationException;

class AuthController extends Controller
{
    protected UpdateMeUseCase $updateMeUseCase;
    protected LoginWithGoogleUseCase $loginWithGoogleUseCase;


    public function __construct(
        UpdateMeUseCase        $updateMeUseCase,
        LoginWithGoogleUseCase $loginWithGoogleUseCase,
    )
    {
        $this->updateMeUseCase = $updateMeUseCase;
        $this->loginWithGoogleUseCase = $loginWithGoogleUseCase;
    }

    // TEST用
    public function loginByEmail(Request $request)
    {
        $request->validate([
            'email' => 'required|email',
            'password' => 'required',
        ]);

        if (!Auth::attempt($request->only('email', 'password'))) {
            throw ValidationException::withMessages([
                'email' => ['The provided credentials are incorrect.'],
            ]);
        }

        $user = Auth::user();

        // トークンを生成
        $token = $user->createToken('auth_token')->plainTextToken;

        return new AuthTokenResource((object) [
            'access_token' => $token,
            'expires_in' => config('sanctum.expiration') * 60,
            'user' => $user,
        ]);
    }

    public function loginWithGoogle(LoginRequest $request)
    {
        $data = $request->validated();

        $authDto = $this->loginWithGoogleUseCase->handle($data['access_token']);

        return new AuthTokenResource($authDto);
    }

    // ログアウト処理
    public function logout(Request $request)
    {
        // トークンを削除
        $request->user()->currentAccessToken()->delete();

        return response()->json([
            'message' => 'Successfully logged out',
        ]);
    }

    // 認証ユーザー情報取得
    public function me(Request $request)
    {
        $user = Auth::user();

        return new MeResource($user);
    }
}
<?php

namespace App\Models\User;

// use Illuminate\Contracts\Auth\MustVerifyEmail;
use App\Traits\Iso8601TimestampAccessors;
use App\Traits\UsesUuid;
use Carbon\Carbon;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens;

/**
 *
 *
 * @property string $id
 * @property string $email
 * @property \Illuminate\Support\Carbon|null $email_verified_at
 * @property string|null $password
 * @property string|null $remember_token
 * @property string|null $google_id
 * @property string|null $google_auth_response_json
 * @property string|null $google_auth_picture
 * @property string|null $google_auth_avatar_url
 * @property string|null $google_auth_full_name
 * @property string|null $family_name
 * @property string|null $given_name
 * @property string|null $family_name_kana
 * @property string|null $given_name_kana
 * @property int|null $gender
 * @property string|null $mst_prefecture_id
 * @property string|null $user_name
 * @property string|null $date_of_birth
 * @property string|null $phone_number
 * @property string|null $invitation_code
 * @property int $status
 * @property string|null $last_sign_in_at
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property \Illuminate\Support\Carbon|null $deleted_at
 * @property-read \App\Models\User\TFactory|null $use_factory
 * @property-read \Illuminate\Notifications\DatabaseNotificationCollection<int, \Illuminate\Notifications\DatabaseNotification> $notifications
 * @property-read int|null $notifications_count
 * @property-read \Illuminate\Database\Eloquent\Collection<int, \Laravel\Sanctum\PersonalAccessToken> $tokens
 * @property-read int|null $tokens_count
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User onlyTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User query()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereCreatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereDateOfBirth($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereDeletedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereEmail($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereEmailVerifiedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereFamilyName($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereFamilyNameKana($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereGender($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereGivenName($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereGivenNameKana($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereGoogleAuthAvatarUrl($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereGoogleAuthFullName($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereGoogleAuthPicture($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereGoogleAuthResponseJson($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereGoogleId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereInvitationCode($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereLastSignInAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereMstPrefectureId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User wherePassword($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User wherePhoneNumber($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereRememberToken($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereStatus($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereUpdatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereUserName($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User withTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User withoutTrashed()
 * @property string|null $token_password
 * @property string|null $expires_at_token_password
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereExpiresAtTokenPassword($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereTokenPassword($value)
 * @mixin \Eloquent
 */
class User extends Authenticatable
{
    /** @use HasFactory<\Database\Factories\UserFactory> */
    use HasFactory, Notifiable, UsesUuid, HasApiTokens, SoftDeletes, Iso8601TimestampAccessors;

    // 主キーがUUIDであることを指定
    protected $keyType = 'string';
    public $incrementing = false;

    protected array $iso8601Timestamps = [
        'last_sign_in_at',
        'email_verified_at',
    ];

    /**
     * The attributes that are mass assignable.
     *
     * @var list<string>
     */
    protected $fillable = [
        'name',
        'email',
        'password',
        'user_name',
        'google_auth_response_json',
        'google_auth_picture',
        'google_auth_avatar_url',
        'google_auth_full_name',
        'email_verified_at',
        'last_sign_in_at'
    ];

    /**
     * The attributes that should be hidden for serialization.
     *
     * @var list<string>
     */
    protected $hidden = [
        'password',
        'remember_token',
    ];

    /**
     * Get the attributes that should be cast.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'password' => 'hashed',
        ];
    }
}
<?php

namespace App\Services\V1\Section;

use App\Dtos\V1\Question\QuestionSetDto;
use App\Dtos\V1\Section\SectionDto;
use App\Dtos\V1\Unit\UnitDto;
use App\Enums\QuestionSetStatus;
use App\Enums\QuestionStatus;
use App\Enums\UnitStatus;
use App\Models\Question\QuestionSet;
use App\Models\Section\Section;
use App\Models\Unit\Unit;
use Illuminate\Support\Collection;

class SectionService
{
    /**
     * 【仕様】
     * 下記パラメータの組み合わせで、セクションを返却するロジック：
     * 1) 何も付与なし
     *    → include_all_sections=false & with_units=false & with_question_sets=false
     *    → "question_sets を持つ unit が1つ以上ある section" のみ返す
     *       かつ units / question_sets はレスポンスに含まない
     *
     * 2) with_units のみ
     *    → 上記(1)に加えて、unitsをレスポンスに含む（ただし question_setsは含まない）
     *
     * 3) with_question_sets のみ
     *    → (1)と同様に "question_setsを持つ unitを持つsection" のみ
     *       だが、units自体を返さないので question_sets も表には出ない(=unitsが返らないため)
     *
     * 4) with_units & with_question_sets
     *    → "question_setsを持つ unitを持つsection" だけ返す + unitsおよびquestion_setsをレスポンスに含む
     *
     * 5) include_all_sections のみ
     *    → 全てのsectionを取得し、units/question_setsはレスポンスに含めない
     *
     * 6) include_all_sections & with_units
     *    → 全てのsectionを取得し、unitsをレスポンスに含むが、question_setsは含まない
     * 7) include_all_sections & with_units & with_question_sets
     *    → 全てのsectionを取得し、units もquestion_setsもレスポンスに含める
     *
     * デフォルト:
     *   include_all_sections=false
     *   → queryに whereHas('units.questionSets')
     *   → さらに units配列から question_sets無いユニットを除外
     *
     * include_all_sections=true
     *   → 上記whereHasを外し、units配列から除外もしない
     *
     * @param bool        $withUnits          => unitsをレスポンスに含めるか
     * @param string|null $levelId
     * @param string|null $subjectId
     * @param bool        $withQuestionSets   => question_setsをレスポンスに含めるか
     * @param bool        $includeAllSections => 全セクション返すか (falseなら "question_setsあるunitを持つセクションのみ")
     */
    public function getAll(
        bool $withUnits,
        ?string $levelId,
        ?string $subjectId,
        bool $withQuestionSets,
        bool $includeAllSections
    ): Collection {
        // sectionのpublishedのみを取得
        $query = Section::published()->orderBy('order');

        if ($withUnits) {
            $query->with([
                'units' => function ($q) use ($withQuestionSets) {
                    $q->published()->orderBy('order')
                        // question_sets が必要なら
                        ->when($withQuestionSets, function ($subQ) {
                            $subQ->with([
                                'questionSets' => function ($qq) {
                                    // PUBLISHED の question_set のみ取得し、
                                    // さらに question_set.questions (status=PUBLISHED) があるものだけ返す
                                    $qq->published()
                                        ->whereHas('questions', function ($q3) {
                                            $q3->where('status', QuestionStatus::PUBLISHED->value);
                                        })
                                        ->orderBy('order');
                                }
                            ]);
                        });
                }
            ]);
        }
        if ($levelId) {
            $query->where('level_id', $levelId);
        }
        if ($subjectId) {
            $query->where('subject_id', $subjectId);
        }

        // デフォルト (includeAllSections=false) なら
        //    "units内に question_sets がある section" のみ取得
        if (!$includeAllSections) {
            $query->whereHas('units.questionSets');
        }


        $sections = $query->get();

        // Model → SectionDto (翻訳適用) へ変換
        return $sections->map(
            fn (Section $section) => $this->toSectionDto($section, $withUnits, $withQuestionSets, $includeAllSections)
        );
    }

    /**
     * 指定IDのSectionを published のみから取得し、翻訳済みの SectionDto を返す。
     * 見つからなければ null を返す。
     */
    public function findById(string $id, bool $withUnits, bool $withQuestionSets, $includeAllSections = false): ?SectionDto
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

        return $this->toSectionDto($section, $withUnits, $withQuestionSets, $includeAllSections);
    }

    /**
     * Section モデルを翻訳込みで SectionDto に変換するための共通処理
     */
    private function toSectionDto(Section $section, bool $withUnits, bool $withQuestionSets, bool $includeAllSections): SectionDto
    {
        // Section翻訳
        $translation = $section->getTranslation();

        $unitsArray = [];
        if ($withUnits && $section->relationLoaded('units')) {
            // mapでユニットを配列化
            $unitsArray = $section->units->map(function (Unit $unit) use ($withQuestionSets, $includeAllSections) {
                $unitTrans = $unit->getTranslation();

                $qsArray = [];
                if ($withQuestionSets && $unit->relationLoaded('questionSets')) {
                    $qsArray = $unit->questionSets->map(function (QuestionSet $qs) {
                        $qsTrans = $qs->translations->firstWhere('locale', app()->getLocale())
                            ?: $qs->translations->firstWhere('locale', config('app.fallback_locale'));

                        return [
                            'id'          => (string) $qs->id,
                            'unit_id'     => (string) $qs->unit_id,
                            'title'       => $qsTrans?->title ?? '',
                            'description' => $qsTrans?->description ?? '',
                            'status'      => (int) $qs->status,
                            'order'       => (int) $qs->order,
                        ];
                    })->toArray();
                }

                return [
                    'id'          => (string) $unit->id,
                    'order'       => (int) $unit->order,
                    'status'      => (int) $unit->status,
                    'version'     => (string) $unit->version,
                    'requirement' => $unitTrans?->requirement ?? $unit->requirement,
                    // question_sets
                    'question_sets' => $qsArray,
                ];
            })
                ->when(
                    !$includeAllSections,
                    fn($collection) =>
                    $collection->filter(function ($unit) {
                        return !empty($unit['question_sets']);
                    })
                )
                ->values()
                ->toArray();
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
     * @param bool $includeAllSections question_setsの有無を問わず全セクション取得するか
     * @return Collection  (of SectionDto)
     */
    public function handle(
        bool $withUnits,
        ?string $levelId,
        ?string $subjectId,
        bool $withQuestionSets,
        bool $includeAllSections
    ): Collection {
        return $this->sectionService->getAll(
            withUnits:        $withUnits,
            levelId:          $levelId,
            subjectId:        $subjectId,
            withQuestionSets: $withQuestionSets,
            includeAllSections: $includeAllSections
        );
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
        Schema::create('users', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password')->nullable();
            $table->rememberToken();
            $table->timestamps();
            $table->softDeletes();
        });

        Schema::create('password_reset_tokens', function (Blueprint $table) {
            $table->string('email')->primary();
            $table->string('token');
            $table->timestamp('created_at')->nullable();
        });

        Schema::create('sessions', function (Blueprint $table) {
            $table->string('id')->primary();
            $table->foreignId('user_id')->nullable()->index();
            $table->string('ip_address', 45)->nullable();
            $table->text('user_agent')->nullable();
            $table->longText('payload');
            $table->integer('last_activity')->index();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('users');
        Schema::dropIfExists('password_reset_tokens');
        Schema::dropIfExists('sessions');
    }
};
<?php

namespace App\Helpers;

use App\Enums\PlatformType;
use App\Enums\UserAgentKeyword;
use Illuminate\Support\Carbon;
use Illuminate\Support\Facades\Request;

class CommonLib
{


    /**
     * キャメルケース -> スネークケース
     *
     * @param string $str
     * @return string
     */
    public static function convSnakeCase($str)
    {
        return ltrim(strtolower(preg_replace('/[A-Z]/', '_\0', $str)), '_');
    }

    /**
     * スネークケース -> キャメルケース
     *
     * @param string $str
     * @return string
     */
    public static function convCamelCase($str)
    {
        return lcfirst(strtr(ucwords(strtr($str, ['_' => ' '])), [' ' => '']));
    }

    /**
     * 桁数指定のランダムパスワード生成
     *
     * @param int $n
     * @return string
     */
    public static function createRandomPassword(int $n)
    {
        return strtr(substr(base64_encode(openssl_random_pseudo_bytes($n)), 0, $n), '/+', '_-');
    }

    /**
     * 文字列から郵便番号部分を取り除く
     *
     * @param string $string
     * @return string
     */
    public static function removalZipStr(string $string)
    {
        $match_count = preg_match("/^(〒[-\d]+)?(\s+)?(.+)$/", $string, $matches);
        if ($match_count > 0) {
            $string = $matches[3];
        }
        return $string;
    }

    /**
     * 非同期バッチ(laravelのartisanだと同期してしまう)
     *
     * @param string $nameCommand
     * @param array $params
     * @return void
     */
    public static function exeBatch(string $nameCommand, array $params = [])
    {
        $param = '';
        if ($params) {
            $param = join(" ", $params);
        }
        exec(sprintf("/usr/bin/php %s/artisan %s %s > /dev/null &", base_path(), $nameCommand, $param));
    }

    /**
     * 画像パス
     *
     * @param string $type
     * @param int $id
     * @param string $path
     * @return string
     */
    public static function getFullPathImg(string $type, int $id, string $path = null)
    {
        $httpProtocol = config()->get('app.env') != 'production' ? 'http' : 'https';

        return sprintf('%s://%s/%s/%s/%s', $httpProtocol, config('app.app_static_domain'), $type, $id, urlencode(urlencode($path)));
    }

    /**
     * ユーザーエージェントを解析して、使用中のプラットフォームの種類を返します。
     *
     * @return string 使用中のプラットフォームの種類
     */
    public static function getPlatformFromUserAgent(): string
    {
        $user_agent = request()->header('User-Agent');

        // デバイスの種類に基づく判定
        // Native アプリ Android
        if (self::containsAny($user_agent, [UserAgentKeyword::NativeAndroid])) {
            return PlatformType::NativeAndroid->value;
        }
        // Native アプリ iOS
        if (self::containsAny($user_agent, [UserAgentKeyword::NativeIOS])) {
            return PlatformType::NativeIOS->value;
        }
        // ブラウザ iOS
        if (self::containsAny($user_agent, [UserAgentKeyword::iPhone, UserAgentKeyword::iPod, UserAgentKeyword::iOS])) {
            return PlatformType::iOS->value;
        }
        // ブラウザ iPad OS
        if (self::containsAny($user_agent, [UserAgentKeyword::iPad])) {
            return PlatformType::iPadOS->value;
        }
        // ブラウザ Android
        if (self::containsAny($user_agent, [UserAgentKeyword::Android])) {
            return PlatformType::Android->value;
        }
        // ブラウザ Windows Phone
        if (self::containsAny($user_agent, [UserAgentKeyword::WindowsPhone])) {
            return PlatformType::WindowsPhone->value;
        }

        // それ以外の場合は一般的なブラウザとして扱う
        return PlatformType::Browser->value;
    }

    /**
     * 指定されたユーザーエージェント文字列が、いずれかのキーワードを含むかどうかをチェックします。大文字小文字は区別しません。
     *
     * @param string $userAgent ユーザーエージェント文字列
     * @param array $keywords チェックするキーワードの配列
     * @return bool
     */
    public static function containsAny(string $userAgent, array $keywords): bool
    {
        foreach ($keywords as $keyword) {
            if (stripos($userAgent, $keyword->value) !== false) {
                return true;
            }
        }

        return false;
    }

    /**
     * ランダムなパスワードを生成する
     *
     * @param int $length
     * @return bool|string
     */
    public static function randomPass($length = 8)
    {
        return substr(str_shuffle('1234567890abcdefghijklmnopqrstuvwxyz'), 0, $length);
    }

    /**
     * ID を元に戻す
     */
    public static function restoreId($id): int
    {
        return (int)str_replace(config()->get('api.response_salt'), '', base64_decode($id));
    }

    /**
     * @param $request Request
     * @return array
     * セキュリティ関連情報マスク用
     */
    public static function protectedParams($request)
    {
        $protected = [];
        foreach ($request->all() as $key => $value) {
            $protected[$key] = in_array($key, config('const.parameter_black_list'), true) ? "████" : $value;
        }
        return $protected;
    }

    /**
     * "CHECK_BY_EXACT_MATCH" → "checkByExactMatch"
     * @param string $uppercaseSnake e.g. "CHECK_BY_EXACT_MATCH"
     * @return string
     */
    public static function snakeUpperToCamel(string $uppercaseSnake): string
    {
        // 大文字_大文字 を小文字にして _ で explode
        $parts = explode('_', strtolower($uppercaseSnake));
        // 先頭はそのまま、2つ目以降を先頭大文字に
        $parts = array_map(fn($part, $idx) => $idx === 0
            ? $part
            : ucfirst($part),
            $parts,
            array_keys($parts)
        );
        return implode('', $parts);
    }

    /**
     * 汎用トークン生成
     *
     * @param int $length トークン長
     * @return string トークン
     */
    public static function createGeneralToken(int $length)
    {
        return bin2hex(openssl_random_pseudo_bytes($length));
    }
}

```
