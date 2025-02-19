Userが学習を開始する際にバックエンドの設計

# 説明
question_sets：問題（questions）を束ねるグループ
questions: 問題
question_set_questions：question_sets と questions を紐づけるPivotテーブル。questions は複数のquestion_sets に紐づく事があり多対多の関係なのでこのような設計になっている。
user_question_sets: ユーザーが学習した questions_sets。学習開始時に question_sets_id と紐づいて status （UserQuestionSetsStatus::NOT_START) が未開始の状態で生成され、進捗ステータスはスコアなどが管理される
user_questions: ユーザーが学習した questions。学習開始時に、学習を開始した question_sets に紐づく questions が全て、 user_question_sets_id と question_id（questions）を紐づけて status （UserQuestionStatus::NOT_START) が未開始の状態で全てのquestionsの数の分、生成され、進捗ステータスはスコアなどが管理される


# フロー
--開始ここから
１．
ユーザーの回答（JSON）を受け取ってフォーマットが正しいかバリデーションする
2.
evaluationMethodがLLMなら

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
            $table->integer('question_format')->default(1)->comment('出題形式: 1=選択式,2=数値回答,3=テキスト回答,4=画像選択など');
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
