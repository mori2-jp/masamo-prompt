学習者に1日何問の問題を解くのか目標を設定させたい

参考のテーブル を参考に、学習者が毎日の学習目標を設定するAPIを実装してください

参考のテーブルでは、1日何分となっていましたが、1日何問を目標としたい

問題は Problem ではなく Question としてね

まず、マスタデータとして参考テーブルのように一日の学習目標の選択肢を管理するテーブルをSeederも合わせて作ってください。


# 将来的にやること
目標未設定の場合は user 取得APIでレスポンスする

===
# 実装方針
生成また修正変更するコードはそのコードだけは必ず省略せずに必ず全てアウトプットすること。

修正する場合は修正したコードは全て出力し、修正箇所以外のコードやコメントは現状のままにしてください。

仕様はソースコードにまとめてコメントとして残すこと
コメントを残す時に以下のようにコメントへ処理順を示すようなナンバリングは不用です。
// 7) memo => 空白
// 8) generate_question_prompt => そのまま
// 9) generate_question_prompt_file_name => そのまま

ちゃんと解説してね

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


---------
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

ーー 参考のテーブル
```php
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
        Schema::create('targets', function (Blueprint $table) {
            $table->id();
            $table->json('name');
            $table->json('label');
            $table->string('icon');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('targets');
    }
};


<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;

class TargetsTableSeeder extends Seeder
{
    /**
     * Run the database seeds.
     */
    public function run(): void
    {
        DB::statement('SET FOREIGN_KEY_CHECKS=0;');
        DB::table('targets')->truncate();
        DB::table('targets')->insert(array(
            0 => array(
                'id' => 0,
                'name' => json_encode(array(
                    "en" => '5 min/day',
                    "ja" => '5分/日'
                ), JSON_FORCE_OBJECT),
                'label'  => json_encode(array(
                    "en" => 'Casual',
                    "ja" => 'カジュアル'
                ), JSON_FORCE_OBJECT),
                'icon' => 'channels.tv',
                'created_at' => '2024-02-02 02:10:59.000',
                'updated_at' => '2024-02-02 02:10:59.000'
            ),
            1 => array(
                'id' => 0,
                'name' => json_encode(array(
                    "en" => '10 min/day',
                    "ja" => '10分/日'
                ), JSON_FORCE_OBJECT),
                'label'  => json_encode(array(
                    "en" => 'Normal',
                    "ja" => '普通'
                ), JSON_FORCE_OBJECT),
                'icon' => 'channels.youtube',
                'created_at' => '2024-02-02 02:10:59.000',
                'updated_at' => '2024-02-02 02:10:59.000'
            ),
            2 => array(
                'id' => 0,
                'name' => json_encode(array(
                    "en" => '15 min/day',
                    "ja" => '15分/日'
                ), JSON_FORCE_OBJECT),
                'label'  => json_encode(array(
                    "en" => 'Serious',
                    "ja" => '真剣'
                ), JSON_FORCE_OBJECT),
                'icon' => 'channels.twitter',
                'created_at' => '2024-02-02 02:10:59.000',
                'updated_at' => '2024-02-02 02:10:59.000'
            ),
            3 => array(
                'id' => 0,
                'name' => json_encode(array(
                    "en" => '20 min/day',
                    "ja" => '20分/日'
                ), JSON_FORCE_OBJECT),
                'label'  => json_encode(array(
                    "en" => 'Superhuman',
                    "ja" => '超人'
                ), JSON_FORCE_OBJECT),
                'icon' => 'channels.facebook',
                'created_at' => '2024-02-02 02:10:59.000',
                'updated_at' => '2024-02-02 02:10:59.000'
            ),
            4 => array(
                'id' => 0,
                'name' => json_encode(array(
                    "en" => '30+ min/day',
                    "ja" => '30分以上/日'
                ), JSON_FORCE_OBJECT),
                'label'  => json_encode(array(
                    "en" => 'Ironman',
                    "ja" => '鉄人'
                ), JSON_FORCE_OBJECT),
                'icon' => 'channels.facebook',
                'created_at' => '2024-02-02 02:10:59.000',
                'updated_at' => '2024-02-02 02:10:59.000'
            ),
        ));
    }
}

```
