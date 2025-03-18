ありがとう。
以下の、LlmManageServiceで、evaluateUserAnswerWithLLMの、buildEvaluationPromptのように、generateQuestionでも、Promptを作りたい。

これから、generateQuestionの、Prompt ビルドするメソッドを何回かに分けて完成させますが、
まずは、１）QuestionSetJson 生成の仕様　に沿って、question_sets の json を作成して、
fill_in_the_blank.txt
の、{$question_sets_json}　を置き換えると、
２）
fill_in_the_blank.txt の、
generate_question_prompt:
question_sets の　generate_question_prompt をそのままインサート
実装をしてもらえますか？


１）
以下の　fill_in_the_blank.txt はプロンプトのテンプレートファイルです。
fill_in_the_blank.txt の、
{$question_sets_json}
には、question_sets の json が置き換えられる想定です。
プロンプトテンプレートファイルはいくも種類があり、
question_sets の generate_question_prompt_file_name を使って動的に指定します。
/generate/question/{generate_question_prompt_file_name}.txt
今回の例では、fill_in_the_blank.txt　になっているということです。

２）
{$generate_question_prompt}に
question_sets の　generate_question_prompt をそのままインサート

# 実装方針
生成また修正変更するコードはそのコードだけは必ず省略せずに必ず全てアウトプットすること。

修正する場合は修正したコードは全て出力し、修正箇所以外のコードやコメントは現状のままにしてください。

仕様はソースコードにまとめてコメントとして残すこと
コメントを残す時に以下のようにコメントへ処理順を示すようなナンバリングは不用です。
// 7) memo => 空白
// 8) generate_question_prompt => そのまま
// 9) generate_question_prompt_file_name => そのまま


---　QuestionSetJson 生成の仕様


# 前提条件
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

ーーー
<?php

namespace App\Services\Utils\Llm;

use App\Models\Question\Question;
use App\Models\Question\QuestionSet;
use App\Models\User\UserQuestion;
use Illuminate\Support\Facades\Http;

/**
 * Class LlmManageService
 *
 * 【概要】
 *   このサービスクラスは LLM (Large Language Model) とのやりとりに関連する機能を提供します。
 *   具体的には、以下の2つの機能を想定しています:
 *     1) ユーザー回答を LLM で正誤評価する (旧 AnswerService の callLLMEvaluationApi部分)
 *     2) LLMに問題を生成させる (作問)
 *
 * 【機能一覧】
 *   - evaluateUserAnswerWithLLM():
 *       ユーザー回答を LLM に投げて評価させ、JSONレスポンスを返す。
 *   - buildEvaluationPrompt():
 *       LLM評価時に使うプロンプトを組み立てる。
 *   - generateQuestion():
 *       指定された QuestionSet の情報をもとに LLM へ作問依頼を行うサンプルメソッド。
 *
 */
class LlmManageService
{
    /**
     * --------------------------------------------------------------------------------
     * LLMによるユーザー回答の正誤評価
     * --------------------------------------------------------------------------------
     * 【仕様】
     *   - 引数の $answerArr を JSON化して、LLM prompt を構築する。
     *   - LLM API (chat/completions等) を呼び出し、レスポンスを JSON decode して返す。
     *   - 実際の DB更新 (answered_at や answer_data への保存など) は呼び出し元で行う。
     *
     * @param array $answerArr      ユーザー回答 (例: ["fields"=>[...]])
     * @param Question $question    関連する問題(Question)オブジェクト (evaluation_method, llm_evaluation_prompt_file_name等)
     * @param UserQuestion $userQuestion user_questionsレコード (prompt_for_evaluation にプロンプト保存など)
     * @return array LLMの評価結果をJSON配列として返す
     * @throws \RuntimeException LLM呼び出し失敗などの場合
     */
    public function evaluateUserAnswerWithLLM(array $answerArr, Question $question, UserQuestion $userQuestion): array
    {
        // 1) evaluation_response_format / learning_requirement などを取得
        $responseFormat     = $question->evaluation_response_format;
        $learningRequirement = $question->learning_requirement_json;  // 例: JSONで学習要件
        $metadata           = $userQuestion->metadata;                // user_question上のmetadata

        // LLMに投げるユーザー回答JSON文字列
        $userAnswerJson = json_encode($answerArr, JSON_UNESCAPED_UNICODE | JSON_PRETTY_PRINT);

        // question に設定されている prompt番号
        $promptId = $question->llm_evaluation_prompt_file_name;

        // 2) 実際のプロンプトを組み立てる
        $prompt = $this->buildEvaluationPrompt(
            $promptId,
            $userAnswerJson,
            $responseFormat,
            $learningRequirement,
            $metadata
        );

        // user_questions.prompt_for_evaluation に格納するなら
        $userQuestion->prompt_for_evaluation = $prompt;
        $userQuestion->save();

        // 3) LLM呼び出し (例: OpenAI API)
        // モデル名や API エンドポイントは適宜調整してください
        $model = 'gpt-4o-latest';  // 例: chatgpt-4o-latest
        $response = Http::withToken(config('services.openai.api_key'))
            ->post('https://api.openai.com/v1/chat/completions', [
                'model' => $model,
                'messages' => [
                    [
                        'role'    => 'system',
                        'content' => 'You are a helpful assistant. Return your answer in valid JSON format only, with no extra text.'
                    ],
                    [
                        'role'    => 'assistant',
                        'content' => $responseFormat
                    ],
                    [
                        'role'    => 'user',
                        'content' => $prompt
                    ]
                ],
                'response_format' => [
                    'type' => 'json_object'
                ],
                'temperature' => 0.0,
            ]);

        // 4) レスポンス整形
        $responseBody = $response->json();
        // ChatGPTの場合、'choices.0.message.content' に回答が入る想定
        $rawContent = data_get($responseBody, 'choices.0.message.content');

        // JSONとしてdecode
        $parsed = json_decode($rawContent, true);

        return $parsed ?? [];
    }

    /**
     * --------------------------------------------------------------------------------
     * LLM用プロンプト(評価)を組み立てる
     * --------------------------------------------------------------------------------
     * 【仕様】
     *   - LLM評価用のテンプレートファイル (resources/prompts/evaluation/{promptId}.txt) を読み込み
     *   - プレースホルダを置換して文字列を返す
     *
     * @param int|null    $promptId
     * @param string      $userAnswerJson
     * @param string|null $responseFormat
     * @param string|null $learningRequirement
     * @param string|null $metadata
     * @return string
     */
    public function buildEvaluationPrompt(
        ?int $promptId,
        string $userAnswerJson,
        ?string $responseFormat,
        ?string $learningRequirement,
        ?string $metadata
    ): string {
        if (!$promptId) {
            // プロンプト番号が無い場合の fallback
            throw new \RuntimeException("No prompt ID specified for LLM evaluation.");
        }

        $promptPath = resource_path("prompts/evaluation/{$promptId}.txt");
        if (! file_exists($promptPath)) {
            throw new \RuntimeException("Prompt file not found: {$promptPath}");
        }

        // テンプレート読み込み
        $template = file_get_contents($promptPath);

        // 文字列置換
        // テンプレート内で想定する変数名 '{$userAnswerJson}', '{$responseFormat}' 等を一括置換
        $replaced = str_replace(
            [
                '{$userAnswerJson}',
                '{$responseFormat}',
                '{$learningRequirement}',
                '{$metadata}'
            ],
            [
                $userAnswerJson,
                $responseFormat ?? '',
                $learningRequirement ?? '',
                $metadata ?? ''
            ],
            $template
        );

        return $replaced;
    }

    /**
     * --------------------------------------------------------------------------------
     * (3) LLMに作問させる
     * --------------------------------------------------------------------------------
     * 【仕様】
     *   - QuestionSet 情報をもとにプロンプトを作り、LLMに投げて問題JSONを受け取る。
     *   - 受け取った JSON は呼び出し元でバリデーション＋DB登録するのが推奨。
     *
     * @param QuestionSet $questionSet
     * @return array 生成された問題JSON (連想配列)
     * @throws \RuntimeException
     */
    public function generateQuestion(QuestionSet $questionSet): array
    {
        // 1) questionSet が持つプロンプトデータを読み取る
        $promptFileName = $questionSet->generate_question_prompt_file_name;
        $promptText   = $questionSet->generate_question_prompt; // 個別要望など

        // 例: どのように promptNumber, promptText を組み合わせるかは設計次第
        // ここではシンプルに "promptNumber.txt" を読み込んだあと、
        // 後半に $promptText を追加するイメージなど

        if (!$promptFileName) {
            throw new \RuntimeException("No generate_question_prompt_file_name found in QuestionSet {$questionSet->id}");
        }

        $promptPath = resource_path("prompts/generate/question/{$promptFileName}.txt");
        if (! file_exists($promptPath)) {
            throw new \RuntimeException("Prompt file not found: {$promptPath}");
        }

        $baseTemplate = file_get_contents($promptPath);
        // 追加の文言を追記
        $combinedPrompt = $baseTemplate . "\n" . $promptText;

        // LLM呼び出し (OpenAI例)
        $model = 'o1';
//        $response = Http::withToken(config('services.openai.api_key'))
//            ->post('https://api.openai.com/v1/chat/completions', [
//                'model' => $model,
//                'messages' => [
//                    [
//                        'role'    => 'system',
//                        'content' => 'You are a helpful assistant. Return your answer in valid JSON format only, with no extra text.'
//                    ],
//                    [
//                        'role'    => 'user',
//                        'content' => $combinedPrompt
//                    ]
//                ],
////                'temperature' => 0.0,
//            ]);

        $responseFormat = '{
  "order": 100,
  "id": "ques_s1_g3_sec100_u300_diff100_qt51_v100_100",
  "level_id": "lev_003",
  "grade_id": "gra_003",
  "difficulty_id": "diff_100",
  "version": "1.0.0",
  "status": "PUBLISHED",
  "generated_by_llm": false,
  "created_at": "2025-01-01 00:00:00",
  "updated_at": "2025-01-01 00:00:00",
  "skills": [
    {
      "skill_id": "sk_004",
      "name": "知識・技能"
    }
  ],
  "learning_requirements": [
    {
      "learning_subject": "算数",
      "learning_no": 37,
      "learning_requirement": "計算の意味・方法 大きな数の概念と活用 3位数や4位数の加法及び減法",
      "learning_required_competency": "3～4桁どうしの足し算・引き算を繰り上がり・繰り下がり含め正確に計算できる",
      "learning_background": "筆算の手順をしっかり確立させる",
      "learning_category": "A",
      "learning_grade_level": "小3"
    }
  ],
  "evaluation_spec": {
    "evaluation_method": "CODE",
    "checker_method": "CHECK_BY_EXACT_MATCH",
    "response_format": {
      "is_correct": "boolean",
      "score": "number",
      "question_text": {
        "ja": "▢にあてはまる数を答えなさい。",
        "en": "Please answer the numbers that fit in the blanks."
      },
      "explanation": {
        "ja": "これは、3桁どうしの足し算を位ごとにわけて考える練習です。百の位、十の位、一の位をそれぞれ計算し、最後に合わせると簡単に正しい合計が求められます。",
        "en": "This exercise practices adding two three-digit numbers by separating the hundreds, tens, and ones places. Calculate each place value separately, then combine them to get the correct total easily."
      },
      "question": {
        "ja": "315 + 276 = (300 + 200) + (10 + 70) + (5 + 6) = ▢ + ▢ + ▢ = ▢",
        "en": "315 + 276 = (300 + 200) + (10 + 70) + (5 + 6) = ▢ + ▢ + ▢ = ▢"
      },
      "fields": [
        {
          "field_id": "f_1",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": 500,
            "en": 500
          },
          "field_explanation": {
            "ja": "300 と 200 を足すと 500 になるからです。",
            "en": "Because adding 300 and 200 results in 500."
          }
        },
        {
          "field_id": "f_2",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": 80,
            "en": 80
          },
          "field_explanation": {
            "ja": "10 と 70 を足すと 80 になるからです。",
            "en": "Because adding 10 and 70 gives 80."
          }
        },
        {
          "field_id": "f_3",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": 11,
            "en": 11
          },
          "field_explanation": {
            "ja": "5 と 6 を足すと 11 になるからです。",
            "en": "Because adding 5 and 6 results in 11."
          }
        },
        {
          "field_id": "f_4",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": 591,
            "en": 591
          },
          "field_explanation": {
            "ja": "500 + 80 + 11 をすべて足すと 591 になるからです。",
            "en": "Because adding 500, 80, and 11 totals 591."
          }
        }
      ]
    }
  },
  "metadata": {
    "question_type": "FILL_IN_THE_BLANK",
    "question": {
      "ja": "315 + 276 = (300 + 200) + (10 + 70) + (5 + 6) = ▢ + ▢ + ▢ = ▢",
      "en": "315 + 276 = (300 + 200) + (10 + 70) + (5 + 6) = ▢ + ▢ + ▢ = ▢"
    },
    "question_text": {
      "ja": "▢にあてはまる数を答えなさい。",
      "en": "Please answer the numbers that fit in the blanks."
    },
    "explanation": {
      "ja": "3桁の数の足し算では、位を分けて考えることで正確に計算ができるようになります。百の位でまとまりを作り、十の位と一の位は繰り上がりに注意しながら合計しましょう。",
      "en": "When adding three-digit numbers, separating each digit place helps ensure accuracy. Group the hundreds place together and be mindful of any carrying over in the tens or ones places."
    },
    "background": {
      "ja": "この問題は、3桁の足し算に慣れることと、位ごとの計算手順を身につけるためのものです。すべての位を正しく合計すると、簡単に正解にたどりつけます。",
      "en": "This problem is designed to help you become comfortable with three-digit addition and master the step-by-step process of adding each place value correctly."
    },
    "input_format": {
      "type": "fixed",
      "fields": [
        {
          "field_id": "f_1",
          "attribute": "number",
          "user_answer": "number"
        },
        {
          "field_id": "f_2",
          "attribute": "number",
          "user_answer": "number"
        },
        {
          "field_id": "f_3",
          "attribute": "number",
          "user_answer": "number"
        },
        {
          "field_id": "f_4",
          "attribute": "number",
          "user_answer": "number"
        }
      ],
      "question_components": [
        {
          "type": "text",
          "content": {
            "ja": "315 + 276 = ",
            "en": "315 + 276 = "
          },
          "order": 10,
          "attribute": "text"
        },
        {
          "type": "newline",
          "order": 15
        },
        {
          "type": "text",
          "content": {
            "ja": "(300 + 200) + (10 + 70) + (5 + 6) = ",
            "en": "(300 + 200) + (10 + 70) + (5 + 6) = "
          },
          "order": 20,
          "attribute": "text"
        },
        {
          "type": "newline",
          "order": 25
        },
        {
          "type": "input_field",
          "field_id": "f_1",
          "order": 30,
          "attribute": "blank",
          "content": {
            "ja": "",
            "en": ""
          }
        },
        {
          "type": "text",
          "content": {
            "ja": " + ",
            "en": " + "
          },
          "order": 40,
          "attribute": "text"
        },
        {
          "type": "input_field",
          "field_id": "f_2",
          "order": 50,
          "attribute": "blank",
          "content": {
            "ja": "",
            "en": ""
          }
        },
        {
          "type": "text",
          "content": {
            "ja": " + ",
            "en": " + "
          },
          "order": 60,
          "attribute": "text"
        },
        {
          "type": "input_field",
          "field_id": "f_3",
          "order": 70,
          "attribute": "blank",
          "content": {
            "ja": "",
            "en": ""
          }
        },
        {
          "type": "text",
          "content": {
            "ja": " = ",
            "en": " = "
          },
          "order": 80,
          "attribute": "text"
        },
        {
          "type": "input_field",
          "field_id": "f_4",
          "order": 90,
          "attribute": "blank",
          "content": {
            "ja": "",
            "en": ""
          }
        }
      ]
    }
  }
}';
        $response = Http::retry(3, 5000)  // (回数, ミリ秒) => 最大3回、5秒間隔で再試行
        ->withOptions([
            'timeout' => 180,        // 全体のタイムアウト(秒)を3分
            'connect_timeout' => 15, // 接続確立までのタイムアウト(秒)
        ])
            ->withToken(config('services.openai.api_key'))
            ->post('https://api.openai.com/v1/chat/completions', [
                'model' => $model,
                'messages' => [
                    [
                        'role'    => 'system',
                        'content' => 'You are a helpful assistant. Return your answer in valid JSON format only, with no extra text.'
                    ],
                    [
                        'role'    => 'assistant',
                        'content' => $responseFormat
                    ],
                    [
                        'role'    => 'user',
                        'content' => $combinedPrompt
                    ]
                ],
                'response_format' => [
                    'type' => 'json_object'
                ],
//                'temperature' => 0.0,
            ]);
        dump($response);
        if ($response->failed()) {
            throw new \RuntimeException(
                "LLM question generation request failed. status={$response->status()} body=" . $response->body()
            );
        }

        $responseBody = $response->json();
        dump($responseBody);
        $rawContent = data_get($responseBody, 'choices.0.message.content');
        if (!$rawContent) {
            throw new \RuntimeException(
                "LLM question generation returned empty content. Full response=" . json_encode($responseBody)
            );
        }

        // JSON としてパース
        $parsed = json_decode($rawContent, true);
        if (empty($parsed) || !is_array($parsed)) {
            throw new \RuntimeException(
                "LLM question generation returned invalid JSON. rawContent={$rawContent}"
            );
        }

        // 3) 生成結果を返す (呼び出し元で validate + DB登録を推奨)
        return $parsed;
    }
}

ーーーーーーー fill_in_the_blank.txt
あなたは小学生向けの問題を作成するシステムです。
【共通ルール】及び【個別要望】に従って【問題JSONの例】を参考に、【問題セット】に関連する問題を作成して【JSONルール】を参考に【問題JSONの例】と同様のフォーマットでJSON出力してください。

【作問ルール】（評価メソッドと、出題種類によって切り替え）

【共通ルール】
- 回答は、プレーンなJSONで出力すること。
-【問題JSONの例】は、この【問題セット】に紐づく問題です。
- 学習対象：【問題JSONの例】learning_subject の 【問題JSONの例】learning_grade_level の生徒。解説は学習対象に分かりやすい説明であること。
- 生成する問題は、【問題JSONの例】と同じにならないようにしてください

【個別要望】
{$generate_question_prompt}

【問題セット】
{$question_sets_json}


【JSON項目の説明】（共通系なので別のプロンプトファイルで用意してコードでインサートする）
order：【問題JSONの例】の order に +1した値
id：【問題JSONの例】の id の一番最後の _ 以降に +1　して、一番最後の _ 以前の値を連結。例えば、qset_s1_g3_sec100_u300_v100_100　であれば一番最後の_の100に+1して、「qset_s1_g3_sec100_u300_v100_101」 とする

level_id：【問題JSONの例】と同じ
grade_id：【問題JSONの例】と同じ
difficulty_id：【問題JSONの例】と同じ
version： 【問題JSONの例】と同じ
status：【問題JSONの例】と同じ
generated_by_llm：必須、boolean, true を指定
created_at：必須、生成時の時刻（UTC）を、Y-m-d H:i:s 形式で
updated_at：必須、生成時の時刻（UTC）を、Y-m-d H:i:s 形式で

skills:【問題JSONの例】と同じ
skills.skill_id:【問題JSONの例】と同じ
skills.name: 【問題JSONの例】と同じ

learning_requirements: 【問題JSONの例】と同じ
learning_requirements.learning_subject:  【問題JSONの例】と同じ
learning_requirements.learning_no: 【問題JSONの例】と同じ
learning_requirements.learning_requirement:  【問題JSONの例】と同じ
learning_requirements.learning_required_competency: 【問題JSONの例】と同じ
learning_requirements.learning_background: 【問題JSONの例】と同じ
learning_requirements.learning_category: 【問題JSONの例】と同じ
learning_requirements.learning_grade_level: 【問題JSONの例】と同じ
learning_requirements.learning_url: 【問題JSONの例】と同じ

evaluation_spec：必須、オブジェクト
evaluation_spec.evaluation_method: 【問題JSONの例】と同じ
evaluation_spec.checker_method: 【問題JSONの例】と同じ
evaluation_spec.llm_prompt_number: evaluation_methodが”LLM”の時は必須、数値。LLMに投げるプロンプトを管理するファイルと対応している。resources/prompts/evaluation/{x}.txt の {x}の箇所と対応しているので、このファイルが存在しているかバリデーションチェックする。
evaluation_spec.response_format：必須、オブジェクト。LLMに正誤判定を依頼する時に指定するレスポンスの形や、CheckerMethod での正誤判定時のレスポンスに利用する
evaluation_spec.response_format.is_correct：必須、テキスト型("boolean"のみ）。回答全体が正解かどうか表す
evaluation_spec.response_format.score：必須、テキスト型("number"のみ）。スコアを表す
evaluation_spec.response_format.question_text：必須、オブジェクト（言語定数全て含んでいるか）, evaluation_methodが”LLM”の時は、言語ごとにそれぞれ オブジェクトの中の値は "{$言語設定例えば"ja"など}": "text"となっていること。evaluation_methodが”CODE”の時は、言語ごとにそれぞれ question_text と全く同じ値が含まれていること
evaluation_spec.response_format.explanation：必須、オブジェクト（言語定数全て含んでいるか）, evaluation_methodが”LLM”の時は、言語ごとにそれぞれ オブジェクトの中の値は "{$言語設定例えば"ja"など}": "text"となっていること。evaluation_methodが”CODE”の時は、言語ごとにそれぞれ オブジェクトの中の値は "{$言語設定例えば"ja"など}": に問題文となる文字列が含まれていること
evaluation_spec.response_format.question：必須、オブジェクト（言語定数全て含んでいるか）、言語ごとにそれぞれ オブジェクトの中の値は "{$言語設定例えば"ja"など}": "text"となっていること。evaluation_methodが”CODE”の時は、言語ごとにそれぞれ metadata.question と全く同じ値が含まれていること

evaluation_spec.response_format.fields：evaluation_methodが”LLM”の時は必須、配列。CODEのときは正答判定、LLMの時はLLMにユーザーの回答の形を知らせる為にフォーマットを定義。
evaluation_spec.response_format.fields.field_id：必須、配列。ユーザーの回答の型。
evaluation_spec.response_format.fields.user_answer：必須、input_format.fields.type、evaluation_spec.response_format.fields.user_answer,evaluation_spec.input_format.fields.collect_answer の定数に値があるか。ユーザーの回答。
evaluation_spec.response_format.fields.is_correct：必須、テキスト型("boolean"のみ）。ユーザーの回答が正しいか
evaluation_spec.response_format.fields.collect_answer：必須、オブジェクト（言語定数全て含んでいるか）。オブジェクトの中の値が、evaluation_spec.response_format.fields.user_answerで指定されている型と一致しているか。例えば、"number" の場合は、32 などの数値となっているか。問題の正解（ユーザーには隠す）
evaluation_spec.response_format.fields.field_explanation: 必須、オブジェクト、言語ごとにそれぞれ オブジェクトの中の値は "{$言語設定例えば"ja"など}": "文字列"が含まれていること。空文字禁止

metadata.question_type：必須、App\Enums\QuestionType.php に値が存在しているか。JSONには、文字列でも数値でもどちらでも入力可にする

metadata.question_text: 必須、オブジェクト（言語定数全て含んでいるか）
metadata.explanation: 必須、オブジェクト（言語定数全て含んでいるか）
metadata.background: 必須、オブジェクト（言語定数全て含んでいるか）
metadata.question ：必須、オブジェクト（言語定数全て含んでいるか）。問題文

metadata.input_format: 必須、オブジェクト（言語定数全て含んでいるか）
metadata.input_format.fields: 必須、配列。ユーザが回答する入力フォームの仕様を定義
metadata.input_format.fields.field_id: 必須、f_x のフォーマットになっているか。同じ fields 内に重複した値が存在しないか。 question_components内の type: "blank"の数と総数が合っているか。
metadata.input_format.fields.attribute: 必須、（input_format.fields.type、evaluation_spec.response_format.fields.user_answer,evaluation_spec.response_format.fields.collect_answer の定数に値が存在しているか）。ブランクのフォームの属性。例えば number であれば、<input type="number">になる
metadata.input_format.fields.user_answer: 必須、次の条件を満たした文字列型かどうかをチェックする。input_format.fields.type、evaluation_spec.response_format.fields.user_answer,evaluation_spec.response_format.fields.collect_answer の定数に値があるか。ユーザーが入力する正答の型。
metadata.input_format.fields.collect_answer: 絶対に存在してはいけない。ユーザーに回答が見えてしまうため。

metadata.input_format.question_components: 必須（問題を更生する要素）
metadata.input_format.question_components.attribute：必須、input_format.question_components.type定数と値が合っているか
metadata.input_format.question_components.content：必須、オブジェクト、言語定数と一致する値が全て含まれているか
metadata.input_format.question_components.order: 必須、数値、重複する値が存在しないこと。問題を構築するときの表示順番



【問題JSONの例】（QuestionSetにあったJSONを動的にインサートする）
{
  "order": 100,
  "id": "ques_s1_g3_sec100_u300_diff100_qt51_v100_100",
  "level_id": "lev_003",
  "grade_id": "gra_003",
  "difficulty_id": "diff_100",
  "version": "1.0.0",
  "status": "PUBLISHED",
  "generated_by_llm": false,
  "created_at": "2025-01-01 00:00:00",
  "updated_at": "2025-01-01 00:00:00",
  "skills": [
    {
      "skill_id": "sk_004",
      "name": "知識・技能"
    }
  ],
  "learning_requirements": [
    {
      "learning_subject": "算数",
      "learning_no": 37,
      "learning_requirement": "計算の意味・方法 大きな数の概念と活用 3位数や4位数の加法及び減法",
      "learning_required_competency": "3～4桁どうしの足し算・引き算を繰り上がり・繰り下がり含め正確に計算できる",
      "learning_background": "筆算の手順をしっかり確立させる",
      "learning_category": "A",
      "learning_grade_level": "小3"
    }
  ],
  "evaluation_spec": {
    "evaluation_method": "CODE",
    "checker_method": "CHECK_BY_EXACT_MATCH",
    "response_format": {
      "is_correct": "boolean",
      "score": "number",
      "question_text": {
        "ja": "▢にあてはまる数を答えなさい。",
        "en": "Please answer the numbers that fit in the blanks."
      },
      "explanation": {
        "ja": "これは、3桁どうしの足し算を位ごとにわけて考える練習です。百の位、十の位、一の位をそれぞれ計算し、最後に合わせると簡単に正しい合計が求められます。",
        "en": "This exercise practices adding two three-digit numbers by separating the hundreds, tens, and ones places. Calculate each place value separately, then combine them to get the correct total easily."
      },
      "question": {
        "ja": "315 + 276 = (300 + 200) + (10 + 70) + (5 + 6) = ▢ + ▢ + ▢ = ▢",
        "en": "315 + 276 = (300 + 200) + (10 + 70) + (5 + 6) = ▢ + ▢ + ▢ = ▢"
      },
      "fields": [
        {
          "field_id": "f_1",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": 500,
            "en": 500
          },
          "field_explanation": {
            "ja": "300 と 200 を足すと 500 になるからです。",
            "en": "Because adding 300 and 200 results in 500."
          }
        },
        {
          "field_id": "f_2",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": 80,
            "en": 80
          },
          "field_explanation": {
            "ja": "10 と 70 を足すと 80 になるからです。",
            "en": "Because adding 10 and 70 gives 80."
          }
        },
        {
          "field_id": "f_3",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": 11,
            "en": 11
          },
          "field_explanation": {
            "ja": "5 と 6 を足すと 11 になるからです。",
            "en": "Because adding 5 and 6 results in 11."
          }
        },
        {
          "field_id": "f_4",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": 591,
            "en": 591
          },
          "field_explanation": {
            "ja": "500 + 80 + 11 をすべて足すと 591 になるからです。",
            "en": "Because adding 500, 80, and 11 totals 591."
          }
        }
      ]
    }
  },
  "metadata": {
    "question_type": "FILL_IN_THE_BLANK",
    "question": {
      "ja": "315 + 276 = (300 + 200) + (10 + 70) + (5 + 6) = ▢ + ▢ + ▢ = ▢",
      "en": "315 + 276 = (300 + 200) + (10 + 70) + (5 + 6) = ▢ + ▢ + ▢ = ▢"
    },
    "question_text": {
      "ja": "▢にあてはまる数を答えなさい。",
      "en": "Please answer the numbers that fit in the blanks."
    },
    "explanation": {
      "ja": "3桁の数の足し算では、位を分けて考えることで正確に計算ができるようになります。百の位でまとまりを作り、十の位と一の位は繰り上がりに注意しながら合計しましょう。",
      "en": "When adding three-digit numbers, separating each digit place helps ensure accuracy. Group the hundreds place together and be mindful of any carrying over in the tens or ones places."
    },
    "background": {
      "ja": "この問題は、3桁の足し算に慣れることと、位ごとの計算手順を身につけるためのものです。すべての位を正しく合計すると、簡単に正解にたどりつけます。",
      "en": "This problem is designed to help you become comfortable with three-digit addition and master the step-by-step process of adding each place value correctly."
    },
    "input_format": {
      "type": "fixed",
      "fields": [
        {
          "field_id": "f_1",
          "attribute": "number",
          "user_answer": "number"
        },
        {
          "field_id": "f_2",
          "attribute": "number",
          "user_answer": "number"
        },
        {
          "field_id": "f_3",
          "attribute": "number",
          "user_answer": "number"
        },
        {
          "field_id": "f_4",
          "attribute": "number",
          "user_answer": "number"
        }
      ],
      "question_components": [
        {
          "type": "text",
          "content": {
            "ja": "315 + 276 = ",
            "en": "315 + 276 = "
          },
          "order": 10,
          "attribute": "text"
        },
        {
          "type": "newline",
          "order": 15
        },
        {
          "type": "text",
          "content": {
            "ja": "(300 + 200) + (10 + 70) + (5 + 6) = ",
            "en": "(300 + 200) + (10 + 70) + (5 + 6) = "
          },
          "order": 20,
          "attribute": "text"
        },
        {
          "type": "newline",
          "order": 25
        },
        {
          "type": "input_field",
          "field_id": "f_1",
          "order": 30,
          "attribute": "blank",
          "content": {
            "ja": "",
            "en": ""
          }
        },
        {
          "type": "text",
          "content": {
            "ja": " + ",
            "en": " + "
          },
          "order": 40,
          "attribute": "text"
        },
        {
          "type": "input_field",
          "field_id": "f_2",
          "order": 50,
          "attribute": "blank",
          "content": {
            "ja": "",
            "en": ""
          }
        },
        {
          "type": "text",
          "content": {
            "ja": " + ",
            "en": " + "
          },
          "order": 60,
          "attribute": "text"
        },
        {
          "type": "input_field",
          "field_id": "f_3",
          "order": 70,
          "attribute": "blank",
          "content": {
            "ja": "",
            "en": ""
          }
        },
        {
          "type": "text",
          "content": {
            "ja": " = ",
            "en": " = "
          },
          "order": 80,
          "attribute": "text"
        },
        {
          "type": "input_field",
          "field_id": "f_4",
          "order": 90,
          "attribute": "blank",
          "content": {
            "ja": "",
            "en": ""
          }
        }
      ]
    }
  }
}






