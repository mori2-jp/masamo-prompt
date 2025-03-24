あなたはバックエンドエンジニアです
小学生向けの算数や国語の学習サービスを作っています。

・回答は修正または作成したクラスは省略せずに必ず全てのコードを書き出してください。

# 使用技術
- Laravel11
- PHP8.4
- mariadb
- laravel sail
- Laravel Sanctum

# composer.json
    "require": {
        "php": "^8.4",
        "google/apiclient": "^2.18",
        "laravel/framework": "^11.31",
        "laravel/sanctum": "^4.0",
        "laravel/slack-notification-channel": "^3.4",
        "laravel/socialite": "^5.17",
        "laravel/tinker": "^2.9",
        "league/csv": "^9.21",
        "spatie/eloquent-sortable": "^4.4"
    },
    "require-dev": {
        "barryvdh/laravel-ide-helper": "^3.5",
        "fakerphp/faker": "^1.23",
        "laravel/pail": "^1.1",
        "laravel/pint": "^1.13",
        "laravel/sail": "^1.40",
        "mockery/mockery": "^1.6",
        "nunomaduro/collision": "^8.1",
        "phpunit/phpunit": "^11.0.1"
    },

# 画面認証構成（モノレポ）API
- ユーザー画面用API(app ユーザーが利用する画面) api.masamo.yashio-corp.com or api.masamo.local

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


# 実装方針
1.
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

2.
バージョニングしたいので、namespaceには全て V1 というPrefix をつけること

3.
DBを多言語化させる為に、xx_translations というテーブルがあります
例）sections, section_translations 

4.
必ずテストコードとセットで実装してください

5.
生成また修正変更するコードはそのコードだけは必ず省略せずに必ず全てアウトプットすること。

6.
修正する場合は修正したコードは全て出力し、修正箇所以外のコードやコメントは現状のままにしてください。

7.
仕様はソースコードにまとめてコメントとして残すこと
コメントを残す時に以下のようにコメントへ処理順を示すようなナンバリングは不用です。
// 7) memo => 空白
// 8) generate_question_prompt => そのまま
// 9) generate_question_prompt_file_name => そのまま

8.
ちゃんと解説してね

9.
変更のブランチ名とコミットメッセージを一行で


実装の参考は以下です。
ーーDto、親クラス（作成済み
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


-- Resource
<?php

namespace App\Http\Resources\Invitation;

use Illuminate\Http\Resources\Json\JsonResource;

class InvitationCompanyResource extends JsonResource
{
    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'name' => $this->name,
            'categories' => $this->categories,
            'clientsWithCategories' => $this->clientsWithCategories,
            'organization' => $this->organization,
            'partnerLabel' => $this->partnerLabel,
        ];
    }
}


ーー Controller
<?php

namespace App\Http\Controllers;

use App\Http\Requests\Invitation\CreateTokenRequest;
use App\Http\Requests\Invitation\RegisterRequest;
use App\Http\Requests\Invitation\VerifyTokenRequest;
use App\Http\Resources\Invitation\InvitationCompanyResource;
use App\Http\Resources\Invitation\TokenResource;
use App\Http\Resources\Invitation\VerifyTokenResource;
use App\Services\Invitation\InvitationService;
use App\UseCases\Invitation\CreateTokenUseCase;
use App\UseCases\Invitation\GetInvitationCompanyUseCase;
use App\UseCases\Invitation\RegisterUseCase;
use Auth;

class InvitationController extends Controller
{
    protected $createTokenUseCase;
    protected $getInvitationCompanyUseCase;
    protected $registerUseCase;

    public function __construct(
        CreateTokenUseCase $createTokenUseCase,
        GetInvitationCompanyUseCase $getInvitationCompanyUseCase,
        RegisterUseCase $registerUseCase,
        protected InvitationService $invitationService
    ) {
        $this->createTokenUseCase = $createTokenUseCase;
        $this->getInvitationCompanyUseCase = $getInvitationCompanyUseCase;
        $this->registerUseCase = $registerUseCase;
    }

    public function createToken(CreateTokenRequest $request)
    {
        $companyId = optional(Auth::user())->company_id;
        return new TokenResource($this->createTokenUseCase->handle(type: $request->type, companyId: $companyId));
    }

    public function verifyToken(VerifyTokenRequest $request)
    {
        $invitationDto = $this->invitationService->verifyToken(token: $request->token);
        return new VerifyTokenResource($invitationDto);
    }

    public function invitationCompany(VerifyTokenRequest $request)
    {
        return new InvitationCompanyResource($this->getInvitationCompanyUseCase->handle(token: $request->token));
    }

    public function register(RegisterRequest $request)
    {
        $userId = optional(Auth::user())->id;
        $result = $this->registerUseCase->handle(userId: $userId, data: collect($request->all()));
        return [
            'isSuccess' => $result,
        ];
    }
}



-- UseCase
<?php

namespace App\UseCases\Invitation;

use App\Dtos\InvitationCompanyDto;
use App\Services\Invitation\InvitationService;

class GetInvitationCompanyUseCase
{
    protected $invitationService;

    public function __construct(InvitationService $invitationService)
    {
        $this->invitationService = $invitationService;
    }

    public function handle(string $token): InvitationCompanyDto
    {
        return  $this->invitationService->getCompanyByInvitationToken(token: $token);
    }
}


-- Service
<?php

namespace App\Services\Invitation;

use App\Dtos\InvitationCompanyDto;
use App\Dtos\InvitationDto;
use App\Models\Attendance\AttendanceWorkRuleAssignment;
use App\Models\Company\Company;
use App\Models\Company\CompanyLabel;
use App\Models\Company\CompanyPartner;
use App\Models\Company\CompanyUser;
use App\Models\Company\CompanyUserCategory;
use App\Models\Company\CompanyUserFunctionRole;
use App\Models\Company\CompanyUserOrganizationItem;
use App\Models\Invitation;
use App\Models\Master\FunctionRoleMaster;
use App\Models\Master\RoleMaster;
use App\Models\Partner\PartnerCategory;
use Illuminate\Support\Collection;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Str;
use Symfony\Component\HttpKernel\Exception\BadRequestHttpException;
use Symfony\Component\HttpKernel\Exception\GoneHttpException;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

class InvitationService
{
    public const INVITATION_TYPE_TO_ROLE_ID = [
        Invitation::CLIENT_TYPE => RoleMaster::ADMIN,
        Invitation::INTERNAL_USER_TYPE => RoleMaster::INTERNAL_USER,
        Invitation::EXTERNAL_USER_TYPE => RoleMaster::EXTERNAL_USER,
        Invitation::PARTNER_TYPE => RoleMaster::ADMIN,
    ];

    public function __construct()
    {
    }

    public function createToken(string $type, ?int $companyId = null): InvitationDto
    {
        $invitation = Invitation::create([
            'type' => $type,
            'company_id' => $companyId,
            'token' => Str::random(Invitation::TOKEN_LENGTH)
        ]);
        return InvitationDto::fromModel($invitation);
    }

    public function verifyToken(string $token): InvitationDto
    {
        $invitation = Invitation::with('lineWorks')->getValidToken($token)->first();
        if (!$invitation) {
            throw new BadRequestHttpException('無効なトークンです。');
        }

        return new InvitationDto([
            'id' => $invitation->id,
            'type' => $invitation->type,
            'token' => $invitation->token,
            'is_valid' => $invitation->is_valid,
            'lineWorks' => $invitation->is_partner ? null : $invitation->lineWorks
        ]);
    }

    public function getCompanyByInvitationToken(string $token): InvitationCompanyDto
    {
        $invitation = Invitation::getValidToken($token)->first();
        if (!$invitation) {
            throw new GoneHttpException('有効なトークンが存在しません');
        }

        $company = optional($invitation)->company;
        if (!$company) {
            throw new GoneHttpException('無効なトークンです');
        }

        $categories = collect([]);
        $clientsWithCategories = collect([]);
        $organization = null;
        $partnerLabel = null;
        if ($company->is_client) {
            $categories = $company->categories;
            $company->load(['organization.organizationLevels.organizationItems', 'labels']);
            $organization = $company->organization?->toArray();
            $partnerLabel = $company->labels->firstWhere('key', CompanyLabel::KEY_PARTNER)?->value;
        }
        if ($company->is_partner) {
            $company->load('clientCompanies.categories');
            $clientsWithCategories = $company->clientCompanies;
        }

        return new InvitationCompanyDto([
            'name' => optional($company)->name,
            'categories' => $categories,
            'clientsWithCategories' => $clientsWithCategories,
            'organization' => $organization,
            'partnerLabel' => $partnerLabel,
        ]);
    }

    public function register(int $userId, Collection $data): bool
    {
        $invitation = Invitation::getValidToken($data->get('token'))->first();
        if (!$invitation) {
            throw new NotFoundHttpException('有効なトークンが存在しません');
        }

        $companyUser = CompanyUser::find($userId);
        if ($invitation->is_partner && $invitation->company->id === optional($companyUser)->company_id) {
            throw new BadRequestHttpException('自社を協力会社として登録することはできません。');
        }

        DB::beginTransaction();
        try {
            $companyData = [
                'name' => $data->get('company_name'),
                'zip_code' => $data->get('zip_code'),
                'prefecture_id' => $data->get('prefecture_id'),
                'city' => $data->get('city'),
                'address' => $data->get('address'),
            ];
            if ($invitation->is_partner) {
                if (!$companyUser->company_id) {
                    // 企業レコードが存在しない場合は新規で企業情報を登録する
                    $companyData['is_partner'] = true;
                    $company = Company::create($companyData);
                    $this->updateUserWithCategories($company, $companyUser, $invitation, $data);
                } else {
                    // 企業レコードが存在する場合は協力会社フラグを立てる
                    $myCompany = $companyUser->company;
                    $companyUser->company->is_partner = true;
                    $companyUser->company->save();
                }

                // 招待した企業と登録した企業を紐づける
                $companyPartner = CompanyPartner::create([
                    'company_id' => $invitation->company->id,
                    'partner_company_id' => $companyUser->company_id,
                ]);

                $partnerCategories = $data->get('partner_category_ids');
                foreach ($partnerCategories as $categoryId) {
                    PartnerCategory::create([
                        'company_partner_id' => $companyPartner->id,
                        'company_category_id' => $categoryId,
                    ]);
                }
            } else {
                if ($invitation->is_client) {
                    $invitation->company->fill($companyData);
                    $invitation->company->save();
                }
                $this->updateUserWithCategories($invitation->company, $companyUser, $invitation, $data);

                // 勤怠機能が開放されている企業のユーザーの場合は勤怠送信者の権限をデフォルトで付与する＆就業ルールを紐付ける
                if ($invitation->company->is_client && $invitation->company->is_available_attendance) {
                    CompanyUserFunctionRole::create([
                        'company_user_id' => $companyUser->id,
                        'function_role_id' => FunctionRoleMaster::ATTENDANCE_SENDER,
                    ]);
                    $workRule = $invitation->company->attendanceWorkRules->first();
                    $workRule?->attendanceWorkRuleAssignments()->create([
                        'company_user_id' => $companyUser->id,
                        'attendance_work_rule_id' => $workRule->id,
                        'start_date' => today(),
                        'end_date' => AttendanceWorkRuleAssignment::MAX_END_DATE,
                    ]);
                }
            }

            $organizationItemIds = collect($data->get('organization_item_ids'));
            if ($organizationItemIds->isNotEmpty()) {
                $params = $organizationItemIds->map(
                    fn (int $itemId) =>
                    [
                        'company_user_id' => $userId,
                        'organization_item_id' => $itemId
                    ]
                )->toArray();
                CompanyUserOrganizationItem::upsert($params, ['id']);
            }

            DB::commit();
            return true;
        } catch (\Throwable $e) {
            DB::rollback();
            throw $e;
        }
    }

    private function updateUserWithCategories(Company $company, CompanyUser $companyUser, Invitation $invitation, Collection $data): void
    {
        $companyUser->fill([
            'company_id' => $company->id,
            'first_name' => $data->get('first_name'),
            'last_name' => $data->get('last_name'),
            'first_name_kana' => $data->get('first_name_kana'),
            'last_name_kana' => $data->get('last_name_kana'),
            'tel' => $data->get('tel'),
            'role_id' => self::INVITATION_TYPE_TO_ROLE_ID[$invitation->type],
            'registration_completed_at' => now(),
        ]);
        $companyUser->save();

        $categories = collect($data->get('user_category_ids', []));
        $userCategoryParams = $categories->map(fn ($categoryId) => [ 'company_user_id' => $companyUser->id, 'company_category_id' => $categoryId ])->toArray();
        CompanyUserCategory::upsert($userCategoryParams, ['company_user_id', 'company_category_id']);
    }
}

-- Dto
<?php

namespace App\Dtos;

use App\Models\Company\CompanyLineWorks;

class InvitationDto extends Dto
{
    public int     $id;
    public string  $type;
    public string  $token;
    public ?bool   $is_valid;
    public ?CompanyLineWorks $lineWorks;
}


# オーダー
回答は日本語でお願い。

理解したらオーダーを開始するので読み込んだら教えて下さい。
