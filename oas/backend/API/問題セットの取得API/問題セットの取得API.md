


単元から、演習（question_sets）の一覧を取得するバックエンドAPIの設計
unit_id から、演習（question_sets）の一覧をレスポンスするAPIを以下にならって実装してほしい

# 実装方針
生成また修正変更するコードはそのコードだけは必ず省略せずに必ず全てアウトプットすること。
エンドポイントは、/question-sets として実装してください

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
        Schema::create('question_set_translations', function (Blueprint $table) {
            $table->id();
            $table->uuid('question_set_id');
            $table->string('locale', 10);
            $table->string('title')->nullable();
            $table->text('description')->nullable();
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('question_set_id')->references('id')->on('question_sets')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('question_set_translations');
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
        Schema::create('question_set_questions', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('question_set_id');
            $table->uuid('question_id');
            $table->integer('order');
            $table->timestamps();

            $table->foreign('question_set_id')->references('id')->on('question_sets')->onDelete('cascade');
            $table->foreign('question_id')->references('id')->on('questions')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('question_set_questions');
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
            $table->integer('question_type')->default(1);
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
        Schema::create('question_translations', function (Blueprint $table) {
            $table->id();
            $table->uuid('question_id');
            $table->string('locale', 10);
            $table->longText('question_text')->nullable();
            $table->longText('explanation')->nullable();
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('question_id')->references('id')->on('questions')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('question_translations');
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
        Schema::create('user_question_sets', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('user_id');
            $table->uuid('question_set_id');
            $table->integer('status')->default(1);
            $table->decimal('score', 8, 2)->nullable();
            $table->timestamp('started_at');
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

＝＝＝
<?php

namespace App\Enums;

enum UserQuestionStatus: int
{
    case NOT_START = 1;
    case CORRECT = 50;
    case INCORRECT = 100;
    case SKIP = 150;

    public function description(): string
    {
        return match ($this) {
            self::NOT_START => 'Not Started',
            self::CORRECT => 'Correct',
            self::INCORRECT => 'Incorrect',
            self::SKIP => 'Skip',
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
            default => throw new \InvalidArgumentException("Unknown status string: {$statusString}")
        };
    }
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
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet onlyTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet query()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet withTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet withoutTrashed()
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

namespace App\Models\Question;

use App\Models\BaseModel;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

/**
 * 
 *
 * @property int $id
 * @property string $question_set_id
 * @property string $locale
 * @property string|null $title
 * @property string|null $description
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property \Illuminate\Support\Carbon|null $deleted_at
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSetTranslation newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSetTranslation newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSetTranslation onlyTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSetTranslation query()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSetTranslation whereCreatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSetTranslation whereDeletedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSetTranslation whereDescription($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSetTranslation whereId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSetTranslation whereLocale($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSetTranslation whereQuestionSetId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSetTranslation whereTitle($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSetTranslation whereUpdatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSetTranslation withTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSetTranslation withoutTrashed()
 * @mixin \Eloquent
 */
class QuestionSetTranslation extends BaseModel
{
    use SoftDeletes;

    protected $fillable = [
        'question_set_id',
        'locale',
        'title',
        'description',
    ];
}

