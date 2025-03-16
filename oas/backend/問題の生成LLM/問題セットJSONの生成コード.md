

# 前提条件
プロンプトテンプレートファイルはいくも種類があり、
question_sets の generate_question_prompt_file_name を使って動的に指定します。
/generate/question/{generate_question_prompt_file_name}.txt
例えば、fill_in_the_blank.txt　になっているということです。

question_sets は unit_id によって units とリレーションしている。
question_sets は、question_set_translations というテーブルを持っており多言語化されている
units は、unit_translations というテーブルを持っており多言語化されている
question_sets は、question_set_questions を介して questions とリレーションしている

# JSON 仕様
json_id:
question_sets の json_id

order:
question_sets の order

unit_id:
question_sets の　リレーション先の units の json_id

unit:
オブジェクトとして、question_sets の　リレーション先の units の リレーション先の　unit_translations　全ての localeの requirement の
{ locale: requirement }

title:
オブジェクトとして、question_sets のリレーション先の　question_set_translations　全ての locale の title の
{ locale: requirement }

description:
オブジェクトとして、question_sets のリレーション先の　question_set_translations　全ての locale の description の
{ locale: requirement }

background:
オブジェクトとして、question_sets のリレーション先の　question_set_translations　全ての locale の background の
{ locale: requirement }

generate_question_prompt: 
question_sets の　generate_question_prompt をそのままインサート

generate_question_prompt_file_name:
question_sets の　generate_question_prompt_file_name をそのままインサート

llm_generation_status:
question_sets の　llm_generation_status をDBには数値で入っているので、
QuestionSetLLMGenerationStatus の対応する文字列に変更してインサート

memo:
値は空白

version：
question_sets の　version をそのままインサート

status:
question_sets の　status をDBには数値で入っているので、
QuestionSetStatus の対応する文字列に変更してインサート

questions:
question_sets にリレーションしている questions の json_id を配列で全て

# 問題セットJSON例
{
"json_id": "qset_s1_g3_sec100_u300_v100_100",
"order": 100,
"unit_id": "unit_s1_g3_sec100_300",
"unit": {
"ja": "3位数や4位数の加法及び減法",
"en": "Addition and Subtraction of Three- and Four-Digit Numbers"
},
"title": {
"ja": "3桁・4桁の足し算・引き算をマスターしよう",
"en": "Mastering Addition and Subtraction of Three- and Four-Digit Numbers"
},
"description": {
"ja": "このドリルでは、3桁や4桁の整数同士の加減計算に慣れることを目指します。繰り上がり・繰り下がりを含む筆算の正しい手順を身につけ、正確に計算できるようになりましょう。",
"en": "In this drill, you will practice adding and subtracting three- or four-digit numbers. Focus on learning the correct written methods for carrying and borrowing, and aim to calculate with accuracy."
},
"background": {
"ja": "このドリルは、繰り上がり・繰り下がりの処理を伴う3桁＋3桁の加法問題を中心に構成しています。位ごとに正しく計算できるようになることを重視し、例えば「315+276」や「459+276」のように複数の繰り上がりが絡む問題を含んでいます。4桁の加減算にも応用可能な力を養うため、位取りを意識させる練習を重ねることを狙いとしています。類似の例として、単純な2桁や4桁の問題も関連が深いですが、今回は3桁どうしの組み合わせを中心に問題を配置しています。（ユーザーには非表示）",
"en": "This drill mainly features three-digit addition problems requiring you to handle carrying and borrowing properly. For instance, you will see exercises like '315+276' or '459+276,' both of which involve multiple carry steps. By concentrating on place-value alignment, you'll develop skills that also apply to four-digit addition and subtraction. Although two-digit or four-digit problems are closely related, our primary focus here is on three-digit computations. (Not displayed to the user)"
},
"generate_question_prompt": {
"ja": "3桁の整数同士の足し算（繰り上がりあり・なし）を中心に、くり返し練習できる問題を作成してください。位を正しくそろえて計算する重要性を意識させるため、繰り上がりが一度起きる問題や、二度起きる問題など、難易度にバリエーションを持たせてください。",
"en": "Please create practice problems focusing on three-digit addition (both with and without carrying). Provide a range of difficulties, including problems with single and multiple carries, to reinforce the importance of aligning digits correctly."
},
"generate_question_prompt_file_name": 1,
"llm_generation_status": "ENABLED",
"memo": "3桁の足し算ドリル",
"version": "1.0.0",
"status": "PUBLISHED",
"questions": [
"ques_s1_g3_sec100_u300_diff100_qt51_v100_100",
"ques_s1_g3_sec100_u300_diff100_qt51_v100_200"
]
}

-- DB 構造
```php
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

namespace App\Enums;

/**
 * QuestionSetにおける LLM生成ステータスを表すEnum
 */
enum QuestionSetLLMGenerationStatus: int
{
    /**
     * LLMによる問題生成を行わない
     */
    case DISABLED = 0;

    /**
     * LLMによる問題生成を行う
     */
    case ENABLED  = 1;

    public function label(): string
    {
        return match($this) {
            self::DISABLED => 'LLM生成なし',
            self::ENABLED  => 'LLM生成あり',
        };
    }

    /**
     * 文字列から QuestionSetLLMGenerationStatus を取得
     *
     * @throws \InvalidArgumentException
     */
    public static function fromString(string $raw): self
    {
        return match (strtoupper($raw)) {
            'DISABLED' => self::DISABLED,
            'ENABLED'  => self::ENABLED,
            default => throw new \InvalidArgumentException("Unknown QuestionSetLLMGenerationStatus: {$raw}")
        };
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
        Schema::create('question_sets', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('unit_id')->nullable();
            $table->string('json_id')->nullable()->unique();
            $table->string('version')->default('0.0.1');
            $table->integer('status')->default(1);
            $table->integer('order');
            $table->longText('generate_question_prompt')->nullable();
            $table->string('generate_question_prompt_file_name')->nullable();
            $table->integer('llm_generation_status')->nullable()->comment('LLMの問題生成を許可するかどうか。LLMでの問題生成が可能な問題かどうかなどを管理');
            $table->integer('consecutive_generation_failures')->default(0)->comment('LLMの問題生成の連続失敗回数');
            $table->boolean('generation_blocked')->default(false)->comment('LLMの問題生成の停止フラグ。失敗が続いた場合に停止');
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
            $table->longText('description')->nullable();
            $table->longText('background')->nullable();
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
        Schema::create('unit_translations', function (Blueprint $table) {
            $table->id();
            $table->uuid('unit_id');
            $table->string('locale', 10);
            $table->longText('requirement')->nullable();
            $table->longText('required_competency')->nullable();
            $table->longText('background')->nullable();
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
        Schema::dropIfExists('unit_translations');
    }
};

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
 * @property string|null $evaluation_response_format
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLlmEvaluationPrompt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLlmEvaluationResponseFormat($value)
 * @property int|null $llm_evaluation_prompt_file_name
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLlmEvaluationPromptNumber($value)
 * @property string|null $learning_requirement_json
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLearningRequirementJson($value)
 * @property string|null $learning_background 背景・補足
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLearningBackground($value)
 * @property string $grade_id
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereGradeId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereEvaluationResponseFormat($value)
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
        'background'
    ];
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

namespace App\Models\Unit;

use App\Models\BaseModel;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

/**
 * 
 *
 * @property int $id
 * @property string $unit_id
 * @property string $locale
 * @property string|null $name
 * @property string|null $description
 * @property string|null $requirement
 * @property string|null $required_competency
 * @property string|null $background
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property \Illuminate\Support\Carbon|null $deleted_at
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UnitTranslation newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UnitTranslation newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UnitTranslation onlyTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UnitTranslation query()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UnitTranslation whereBackground($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UnitTranslation whereCreatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UnitTranslation whereDeletedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UnitTranslation whereDescription($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UnitTranslation whereId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UnitTranslation whereLocale($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UnitTranslation whereName($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UnitTranslation whereRequiredCompetency($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UnitTranslation whereRequirement($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UnitTranslation whereUnitId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UnitTranslation whereUpdatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UnitTranslation withTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UnitTranslation withoutTrashed()
 * @property-read \App\Models\Unit\TFactory|null $use_factory
 * @property-read \App\Models\Unit\Unit $unit
 * @method static \Database\Factories\Unit\UnitTranslationFactory factory($count = null, $state = [])
 * @mixin \Eloquent
 */
class UnitTranslation extends BaseModel
{
    use SoftDeletes, HasFactory;

    protected $fillable = [
        'unit_id',
        'locale',
        'requirement',
        'required_competency',
        'background'
    ];

    public function unit()
    {
        return $this->belongsTo(Unit::class, 'unit_id');
    }
}

```
