以下、ImportQuestionSetsFromGithub.php　に新しい処理を追加してください。
修正箇所を知らせてください。

必ず、既存のバリデーションルールは全てそのままにしてください。


# 追加する処理
json に grade_id を追加しました。
grade_id が存在しない場合はバリデーションエラー。
指定された grade_id が grades テーブルの json_id に存在しない場合はエラー
question_sets に 指定された grade_id と grades の json_id が一致する grades の id をインサートする。

コード内のコメントは消さずに全て残してください。

ーーJSON仕様
grade_id：必須、文字列、Grade テーブルのjson_idに一致する値があること
json_id：必須、文字列
order：必須、数値
unit_id：必須、文字列、Units テーブルのjson_idに一致する値があること
unit：必須、オブジェクト（言語定数全て含んでいるか）。
title：必須、オブジェクト（言語定数全て含んでいるか）。
description：必須、オブジェクト（言語定数全て含んでいるか）。
background：必須、オブジェクト（言語定数全て含んでいるか）。
memo：必須、文字列
version： 必須、文字列、文字列が x.x.x のようなバージョニング形式になっているか
status：必須、QuestionSetStatus に値が存在しているか
questions：必須、配列。questions テーブルの json_id に値が存在しているか。

llm_generation_status: 必須、QuestionSetLLMGenerationStatus に値が存在しているか

generate_question_prompt：llm_generation_statusが、ENABLED の時に必須、オブジェクト（言語定数全て含んでいるか）。
generate_question_prompt_file_name：llm_generation_statusが、ENABLED の時に必須、数字。/resources/prompt/generate/question/{generate_question_prompt_file_name}.txt が存在しているか確認

--- 問題セットJSONの全体構造
```json
{
  "json_id": "qset_s1_g3_sec100_u100_v100_10000",
  "order": 100,
  "unit_id": "unit_s1_g3_sec100_100",
  "grade_id": "gra_003",
  "unit": {
    "ja": "万の単位、1億などの比べ方や表し方",
    "en": "How to compare and represent units such as ten-thousands and one hundred million"
  },
  "title": {
    "ja": "大きな数の大小比較に挑戦しよう",
    "en": "Let's Challenge Comparing Large Numbers"
  },
  "description": {
    "ja": "このドリルでは、万や億などを含む大きな数の読み書きや、比較を中心に学びます。＜、＞、＝の記号を正しく使って、大きい数をしっかり比べられるようになりましょう。",
    "en": "In this drill, we'll focus on reading, writing, and comparing large numbers, including ten-thousands and hundred millions. Practice using the symbols <, >, and = correctly to accurately compare big numbers."
  },
  "background": {
    "ja": "学習指導要領No.33に対応した問題で、1万（10,000）や1億（100,000,000）など桁数の多い整数の大小比較を扱います。実際には1万以上の位取り概念を確立するために、たとえば「3090万」と「3100万」を比べるなどの問題を用意。位ごとに比較する、あるいは単位を統一して比較するなど、学習者が具体的な大きい数を扱う感覚を身につけられるように作問しました。ここでは、マイナスの値は扱わず、繰り返し練習することで万・億単位のイメージを定着させる意図があります。",
    "en": "This set of exercises aligns with Curriculum Guideline No. 33 and covers comparing large integers, such as 10,000 (man) and 100,000,000 (oku). In practice, learners might compare figures like 3,090,000 vs. 3,100,000, focusing on place value or unifying units for comparison. The goal is to strengthen an intuitive grasp of large-scale numbers without dealing with negative values. Repeated practice will help solidify the concept of ten-thousand (man) and hundred-million (oku)."
  },
  "generate_question_prompt": {
    "ja": "小学3年生レベルで扱う大きな数（万・億など）を比較する問題を1問作成してください。比較記号（>、<、=）を使って答えられる形式にし、実際の数の規模感を意識した具体的な例文でお願いします。",
    "en": "Please create a single problem comparing large numbers (in the range of ten-thousands to one hundred million) at an elementary third-grade level. The answer should involve using comparison symbols (>, <, =) in a realistic scenario to reflect actual scale."
  },
  "generate_question_prompt_file_name": "fill_in_operator",
  "llm_generation_status": "ENABLED",
  "memo": "数字の大小比較：9000000 ＞ 8999999",
  "version": "1.0.0",
  "status": "PUBLISHED",
  "questions": [
    "ques_s1_g3_sec100_u100_diff100_qt201_v100_100",
    "ques_s1_g3_sec100_u100_diff100_qt201_v100_200",
    "ques_s1_g3_sec100_u100_diff100_qt201_v100_300"
  ]
}

```



# 実装方針
変更場所が分かりやすいように、変更を加える箇所の現在の行数を必ず出力してください

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


# 説明
question_sets：問題（questions）を束ねるグループ
questions: 問題
question_set_questions：question_sets と questions を紐づけるPivotテーブル。questions は複数のquestion_sets に紐づく事があり多対多の関係なのでこのような設計になっている。
user_question_sets: ユーザーが学習した questions_sets。学習開始時に question_sets_id と紐づいて status （UserQuestionSetsStatus::NOT_START) が未開始の状態で生成され、進捗ステータスはスコアなどが管理される
user_questions: ユーザーが学習した questions。学習開始時に、学習を開始した question_sets に紐づく questions が全て、 user_question_sets_id と question_id（questions）を紐づけて status （UserQuestionStatus::NOT_START) が未開始の状態で全てのquestionsの数の分、生成され、進捗ステータスはスコアなどが管理される

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

```php
<?php

namespace App\Enums;

enum QuestionSetStatus: int
{
    case DRAFT = 1;         // 下書き
    case PUBLISHED = 300;   // 公開中
    case HIDDEN = 600;      // 非公開
    case TEST_PUBLISHED = 900; // テスト公開
    case JSON_NOT_MATCHED = 99999; // 該当するJSONファイルが存在しなかった

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
            self::JSON_NOT_MATCHED => '該当JSON無し',
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
            'JSON_NOT_MATCHED' => self::JSON_NOT_MATCHED,
            default => throw new \InvalidArgumentException("Unknown status string: {$statusString}")
        };
    }
}

<?php

namespace App\Models\Question;

use App\Enums\QuestionSetStatus;
use App\Models\BaseModel;
use App\Models\Grade\Grade;
use App\Models\Unit\Unit;
use App\Models\User\UserQuestionSetTranslation;
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
 * @property string|null $generate_question_prompt
 * @property string|null $generate_question_prompt_file_name
 * @property int|null $llm_generation_status LLMの問題生成を許可するかどうか。LLMでの問題生成が可能な問題かどうかなどを管理
 * @property int $consecutive_generation_failures LLMの問題生成の連続失敗回数
 * @property int $generation_blocked LLMの問題生成の停止フラグ。失敗が続いた場合に停止
 * @property-read Unit|null $unit
 * @method static Builder<static>|QuestionSet whereConsecutiveGenerationFailures($value)
 * @method static Builder<static>|QuestionSet whereGenerateQuestionPrompt($value)
 * @method static Builder<static>|QuestionSet whereGenerateQuestionPromptFileName($value)
 * @method static Builder<static>|QuestionSet whereGenerationBlocked($value)
 * @method static Builder<static>|QuestionSet whereLlmGenerationStatus($value)
 * @mixin \Eloquent
 */
class QuestionSet extends BaseModel
{
    use HasOrder, UsesUuid, SoftDeletes;

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
    public function grade()
    {
        return $this->belongsTo(Grade::class);
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

    public function unit()
    {
        return $this->belongsTo(Unit::class);
    }

    /*=====================================================
 * カスタムメソッド/Custom Method
*=====================================================*/

    /**
     * 現在のlocaleに応じた翻訳を返す
     * なければfallback_localeの翻訳を返す
     */
    public function getTranslation(?string $locale = null): ?QuestionSetTranslation
    {
        $locale = $locale ?: app()->getLocale();
        return $this->translations->firstWhere('locale', $locale)
            ?: $this->translations->firstWhere('locale', config('app.fallback_locale'));
    }

    /*=====================================================
    * Scope
    *=====================================================*/

    /**
     * 公開(PUBLISHED)だけを絞り込むスコープ
     * local/staging 環境なら TEST_PUBLISHED もOK
     */
    public function scopePublished(Builder $query): Builder
    {
        return $query->where(function ($subQuery) {

            $subQuery->where('status', QuestionSetStatus::PUBLISHED->value);
            if (app()->environment(['local','staging'])) {
                $subQuery->orWhere('status', QuestionSetStatus::TEST_PUBLISHED->value);
            }
        });
    }
}
<?php

namespace App\Models\Question;

use App\Models\BaseModel;
use App\Models\Difficulty\Difficulty;
use App\Models\Grade\Grade;
use App\Models\Level\Level;
use App\Models\Skill\Skill;
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
 * @property-read \Illuminate\Database\Eloquent\Collection<int, Skill> $skills
 * @property-read int|null $skills_count
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLlmEvaluationPromptFileName($value)
 * @property-read Difficulty $difficulty
 * @property-read Grade $grade
 * @property-read Level $level
 * @mixin \Eloquent
 */
class Question extends BaseModel
{
    use HasOrder, UsesUuid, SoftDeletes;

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
    public function skills()
    {
        return $this->belongsToMany(Skill::class, 'question_skill', 'question_id', 'skill_id')
            ->withPivot('order')
            ->withTimestamps();
    }

    public function level()
    {
        return $this->belongsTo(Level::class);
    }

    public function grade()
    {
        return $this->belongsTo(Grade::class);
    }
    public function difficulty()
    {
        return $this->belongsTo(Difficulty::class);
    }
}

```

```php
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
        Schema::create('grades', function (Blueprint $table) {
            $table->id();
            $table->uuid('uuid')->unique();
            $table->string('json_id')->nullable()->unique()->comment('リポジトリ上で管理するための一意識別子');
            $table->text('description')->nullable();
            $table->text('memo')->nullable();
            $table->string('version')->default('0.0.1');
            $table->integer('order');
            $table->timestamps();
            $table->softDeletes();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('grades');
    }
};

```

＝＝＝問題取り込みコード

```php
<?php

namespace App\Console\Commands\Import\Github;

use App\Enums\QuestionSetStatus;
use App\Enums\QuestionStatus;
use App\Enums\QuestionSetLLMGenerationStatus;
use App\Models\Question\Question;
use App\Models\Question\QuestionGenerationLog;
use App\Models\Question\QuestionSet;
use App\Models\Question\QuestionSetTranslation;
use App\Models\Unit\Unit;
use App\Services\Utils\Question\QuestionSetJsonManageService;
use Google_Client;
use Google_Service_Drive;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Str;

/**
 * 【仕様まとめ】
 * 1) staging / local 環境の場合は、サービスアカウント認証 (service-account-credentials.json) で
 *    Google Drive 上の JSON をダウンロードし、question_sets テーブルへ upsert する。
 * 2) Google Drive のファイルは "fileId" ベースで取得する。URL ではなく ID を指定し、$jsonFileOnGoogleDriveIds に列挙する。
 * 3) もともとあった importFromGitHub() はそのまま残し、環境で切り替えて処理。
 * 4) 「取り込もうとしたJSONの総数」「失敗/スキップ数」「成功数」をカウントして最後に表示する。
 * 5) JSON バリデーションNGの場合はスキップする。バリデーションは QuestionSetJsonManageService::validateQuestionSetJson() 。
 * 6) question_set_questions ピボットの再同期・タイトル/説明の多言語 upsert など既存処理は基本維持。
 *   - 既存の syncQuestionSetQuestions() にて、「自前で用意された問題」だけを一旦差し替える処理
 *     => (1) generated_by_llm=false の問題だけ消去→再登録
 *     => (2) LLM 生成問題（generated_by_llm=true）は削除対象から除外（従来の紐づけを維持）
 *   - インポート後に、question_generation_logs で is_success=1 かつ generated_by_llm=true の問題を再アタッチする処理
 *     => すでに pivot があればスキップ、新規なら pivotMaxOrder+1 で order を設定して追加
 *
 * - 環境が local の時は、
 *   ・status = TEST_PUBLISHED
 *   ・かつ validation_check = false
 *   の場合にバリデーションをスキップして保存する。
 *   ※ validation_check が存在しない場合や true の場合は通常どおりバリデーションを実施。
 *
 *   バリデーションをスキップして保存した question_set (id, json_id) を処理の最後に一覧でコンソールに出力する。
 *   さらに、最後の統計出力で JSON_NOT_MATCHED なセットの数も併せて表示する。
 */
class ImportQuestionSetsFromGithub extends Command
{
    /**
     * artisan で呼び出すコマンド名
     *
     * 例: php artisan import:question-sets-from-github --path=math/g3/sec300/u300_あまりのあるわり算/v1.0.0
     */
    protected $signature = 'import:question-sets-from-github
                            {--path= : If specified, only import path }';

    protected $description = 'Import question set data from GitHub or Google Drive JSON files and sync DB';

    /**
     * Google Drive 上の "ファイルID" 一覧 (サービスアカウント認証で取得する)
     */
    protected array $jsonFileOnGoogleDriveIds = [
        '19o2-da7x8NjUSAqDt7DOsiw82DdzLnq_',
        '1ZYsFbzslqrMuq7OIElJNMp2SkSdu0JKF',
    ];

    private int $totalCount = 0;   // 取り込もうとしたJSONの総数
    private int $skipCount  = 0;   // 失敗またはスキップしたJSONの総数
    private int $successCount=0;   // 取り込みに成功したJSONの総数

    /**
     * 取り込みに成功した question_set の json_id を格納
     */
    private array $importedJsonIds = [];

    /**
     * バリデーションスキップ機能用
     * 環境が local & status=TEST_PUBLISHED & validation_check=false ならバリデーションをスキップする
     */
    private int $validationSkipCount = 0;
    private array $validationSkippedSets = [];

    /**
     * JSON_NOT_MATCHED に更新された question_set の総数
     */
    private int $notMatchedCount = 0;

    /**
     * ファイル名と json_id が不一致だったものを収集する配列
     */
    private array $filenameJsonIdMismatches = [];

    public function handle()
    {
        $env = app()->environment();
        $useGoogleDrive = false;

        if ($env === 'staging') {
            // staging環境は必ずGoogle Drive
            $useGoogleDrive = true;
        } elseif ($env === 'local') {
            // local環境は config() で切り替え
            $useGoogleDrive = config('app.use_google_drive_in_local_for_learning_data_import', true);
        }

        if ($useGoogleDrive) {
            $this->info("【INFO】Running in '{$env}' environment => Google Drive (Service Account認証) からJSONをダウンロードして処理します。");
            $this->importFromGoogleDriveWithServiceAccount();
        } else {
            $this->info("【INFO】Running in '{$env}' environment => GitHub からJSONをダウンロードして処理します。");
            $this->importFromGitHub();
        }

        // 未紐づけの問題を表示
        $this->showUnlinkedQuestions();

        /**
         * JSON ファイルに該当しなかった question_sets を status=JSON_NOT_MATCHED に更新し、
         * その id と json_id をコンソールに出力
         */
        $this->updateNotMatchedSets();

        // 統計出力
        $this->info("Import process completed.");
        $this->line("=======================================");
        $this->info("取り込もうとした Question Set JSON の総数: {$this->totalCount}");
        // バリデーションをスキップ数を追加
        $this->info("バリデーションをスキップした Question Set JSON の総数: {$this->validationSkipCount}");
        $this->info("取り込みに失敗またはスキップした Question Set JSON の総数: {$this->skipCount}");
        $this->info("取り込みに成功した Question Set JSON の総数: {$this->successCount}");
        // JSON_NOT_MATCHED なセット数
        $this->info("JSON_NOT_MATCHED な Question Set の総数: {$this->notMatchedCount}");
        $this->line("=======================================");

        // バリデーションをスキップしたセットを一覧表示
        if (!empty($this->validationSkippedSets)) {
            $this->info("==== Validation Skipped Question Sets (id, json_id) ====");
            foreach ($this->validationSkippedSets as $vs) {
                $this->line("id={$vs['id']}, json_id={$vs['json_id']}");
            }
        }

        /**
         * ファイル名と json_id の不一致リストを出力
         */
        if (!empty($this->filenameJsonIdMismatches)) {
            $this->info("==== Filename and json_id Mismatch List ====");
            foreach ($this->filenameJsonIdMismatches as $m) {
                $this->line("filename={$m['filename']}, json_id={$m['json_id']}");
            }
        }

        /**
         * ============================
         * 1) llm_generation_status = DISABLED のデータを出力
         * 2) status != PUBLISHED のデータを出力
         * ============================
         */
        $this->info("=== 1) question_sets with llm_generation_status=DISABLED ===");
        $disabledSets = QuestionSet::where('llm_generation_status', QuestionSetLLMGenerationStatus::DISABLED->value)->get();
        if ($disabledSets->isEmpty()) {
            $this->line("No question_sets found with llm_generation_status=DISABLED.");
        } else {
            foreach ($disabledSets as $ds) {
                // llm_generation_status名 (DISABLED/ENABLED)を文字列で出力
                $llmGenStr = QuestionSetLLMGenerationStatus::from($ds->llm_generation_status)->name;
                $this->line(" json_id={$ds->json_id}, llm_generation_status={$llmGenStr}");
            }
        }

        $this->info("=== 2) question_sets with status != PUBLISHED ===");
        $nonPublishedSets = QuestionSet::where('status', '!=', QuestionSetStatus::PUBLISHED->value)->get();
        if ($nonPublishedSets->isEmpty()) {
            $this->line("No question_sets found with status != PUBLISHED.");
        } else {
            foreach ($nonPublishedSets as $nps) {
                // status名 (DRAFT/PUBLISHED/HIDDEN等) を文字列で出力
                $statusStr = QuestionSetStatus::from($nps->status)->name;
                $this->line(" json_id={$nps->json_id}, status={$statusStr}");
            }
        }


        return 0;
    }

    /**
     * Google Drive からJSONを認証付きダウンロード
     */
    private function importFromGoogleDriveWithServiceAccount(): void
    {
        $this->info('Setting up Google Client for Drive API (QuestionSets) ...');

        $client = new Google_Client();
        $client->setAuthConfig(storage_path('app/private/service-account-credentials.json'));
        $client->addScope(Google_Service_Drive::DRIVE_READONLY);

        $driveService = new Google_Service_Drive($client);

        foreach ($this->jsonFileOnGoogleDriveIds as $fileId) {
            $this->info("Fetching JSON from Google Drive fileId: {$fileId}");
            $this->totalCount++;

            try {
                // ファイル名取得（metadata）
                $fileMetadata = $driveService->files->get($fileId, ['fields' => 'name']);
                $fileName = $fileMetadata->getName() ?? $fileId;

                $response = $driveService->files->get($fileId, ['alt' => 'media']);
                $jsonContent = $response->getBody()->getContents();

                $decoded = json_decode($jsonContent, true);
                if (!is_array($decoded)) {
                    $this->warn("File ID={$fileId} is not valid JSON. Skipped.");
                    $this->skipCount++;
                    continue;
                }

                // ファイル名を引数で渡す
                if ($this->upsertQuestionSet($decoded, $fileName)) {
                    $this->successCount++;
                } else {
                    $this->skipCount++;
                }
            } catch (\Exception $e) {
                $this->warn("Failed to fetch from Google Drive: fileId={$fileId}, error=" . $e->getMessage());
                $this->skipCount++;
            }
        }
    }

    /**
     * GitHub リポジトリからJSONを取得
     */
    private function importFromGitHub(): void
    {
        $token = config('services.github.api_token');
        if (!$token) {
            $this->error('GitHub API token not found in config/services.php [github.api_token]');
            return;
        }

        $repoOwnerAndName = 'NousContentsManagement/masamo-content';
        $basePath = 'contents/question_sets';

        $pathOption = $this->option('path');
        if ($pathOption) {
            $basePath .= '/' . $pathOption;
            $this->info("Path specified: {$pathOption}");
        } else {
            $this->info("No subject specified. Will import from all subdirectories under contents/question_sets");
        }

        $allJsonFiles = $this->fetchAllJsonFilesRecursively($repoOwnerAndName, $basePath, $token);
        if (empty($allJsonFiles)) {
            $this->warn("No JSON files found under path: {$basePath}");
            return;
        }

        $this->info("Found " . count($allJsonFiles) . " JSON file(s) under {$basePath}");

        foreach ($allJsonFiles as $file) {
            $this->processJsonFile($file, $token);
        }
    }

    /**
     * 再帰的にフォルダを辿り .jsonファイルを集める
     */
    private function fetchAllJsonFilesRecursively(string $repo, string $path, string $token): array
    {
        $url = "https://api.github.com/repos/{$repo}/contents/{$path}";
        $response = Http::withToken($token)->get($url);

        if ($response->failed()) {
            $this->warn("Failed to fetch from GitHub: {$url}, status={$response->status()}");
            return [];
        }

        $items = $response->json();
        if (!is_array($items)) {
            return [];
        }

        $result = [];

        foreach ($items as $item) {
            $type = $item['type'] ?? null;
            $itemPath = $item['path'] ?? '';
            $name = $item['name'] ?? '';

            if ($type === 'dir') {
                $descendants = $this->fetchAllJsonFilesRecursively($repo, $itemPath, $token);
                $result = array_merge($result, $descendants);
            } elseif ($type === 'file') {
                if (str_ends_with($name, '.json')) {
                    $result[] = $item;
                }
            }
        }

        return $result;
    }

    /**
     * 単一のJSONファイルをダウンロードしパース => DBに反映
     */
    private function processJsonFile(array $file, string $token): void
    {
        $this->totalCount++;

        if (!isset($file['download_url'])) {
            $this->warn("No download_url for file: " . ($file['path'] ?? 'unknown'));
            $this->skipCount++;
            return;
        }

        $this->info("Fetching JSON: " . $file['path']);
        $jsonContent = Http::withToken($token)->get($file['download_url'])->body();

        $decoded = json_decode($jsonContent, true);
        if (!is_array($decoded)) {
            $this->warn("File {$file['name']} is not valid JSON. Skipped.");
            $this->skipCount++;
            return;
        }

        // ファイル名を引数で渡す
        if ($this->upsertQuestionSet($decoded, $file['name'])) {
            $this->successCount++;
        } else {
            $this->skipCount++;
        }
    }

    /**
     * question_set JSONをDBへupsert
     *
     * $fileName を第二引数に追加し、"ファイル名から .json を除いた文字列" と "json_id" の一致チェックを行う
     */
    private function upsertQuestionSet(array $setJson, ?string $fileName = null): bool
    {
        DB::beginTransaction();
        try {
            $jsonId = $setJson['json_id'] ?? null;
            if (!$jsonId) {
                $this->warn("question_set JSON missing 'json_id'. Skipped.");
                DB::rollBack();
                return false;
            }

            // ファイル名と json_id が一致するかをチェック
            //         ファイル名が null でなければチェック
            if (!is_null($fileName)) {
                // 拡張子 .json を除去
                $fileNameBase = preg_replace('/\.json$/', '', $fileName);
                if ($fileNameBase !== $jsonId) {
                    $this->warn("Filename does not match json_id => skip question_set(json_id={$jsonId}, filename={$fileName}).");
                    // 不一致リストに追加
                    $this->filenameJsonIdMismatches[] = [
                        'filename' => $fileName,
                        'json_id'  => $jsonId,
                    ];
                    DB::rollBack();
                    return false;
                }
            }

            // バリデーションスキップ判定
            $env = app()->environment();
            $rawStatus = $setJson['status'] ?? 'DRAFT';
            $validationCheck = $setJson['validation_check'] ?? null;
            // ※ validation_check が存在しない(=null) or true の場合はスキップ不可

            $skipValidation = false;
            if (
                $env === 'local'
                && $rawStatus === 'TEST_PUBLISHED'
                && $validationCheck === false
            ) {
                $skipValidation = true;
                $this->info(
                    "Skipping validation for question_set(json_id={$jsonId})"
                    . " because environment=local, status=TEST_PUBLISHED, and validation_check=false"
                );
                $this->validationSkipCount++;
            }

            // バリデーション (skipValidation が false の場合のみ実施)
            if (!$skipValidation) {
                $validator = app(QuestionSetJsonManageService::class);
                $errors = $validator->validateQuestionSetJson($setJson);
                if (!empty($errors)) {
                    $this->warn("バリデーションエラー => skip question_set(json_id={$jsonId}).");
                    foreach ($errors as $field => $msgs) {
                        foreach ($msgs as $msg) {
                            $this->warn("  - {$field}: {$msg}");
                        }
                    }
                    DB::rollBack();
                    return false;
                }
            }

            // DB検索 or 新規
            $qset = QuestionSet::where('json_id', $jsonId)->first();
            if (!$qset) {
                $qset = new QuestionSet();
                $qset->uuid      = (string) Str::uuid();
                $qset->json_id = $jsonId;
            }

            $qset->order   = $setJson['order'] ?? 9999;
            $qset->unit_id = $this->findUnitUuid($setJson['unit_id'] ?? null);
            if (!$qset->unit_id) {
                $this->warn("Unit not found => skip question_set(json_id={$jsonId}).");
                DB::rollBack();
                return false;
            }

            $qset->version = $setJson['version'] ?? '0.0.1';
            $statusVal = $this->parseQuestionSetStatus($rawStatus);
            $qset->status = $statusVal;

            $qset->generate_question_prompt_file_name = $setJson['generate_question_prompt_file_name'] ?? null;
            if (isset($setJson['llm_generation_status'])) {
                $qset->llm_generation_status = $this->parseLLMGenerationStatus($setJson['llm_generation_status']);
            }
            if (isset($setJson['generate_question_prompt']) && is_array($setJson['generate_question_prompt'])) {
                $qset->generate_question_prompt = json_encode($setJson['generate_question_prompt'], JSON_UNESCAPED_UNICODE);
            }

            $qset->save();

            // バリデーションをスキップした場合は保存後に記録
            if ($skipValidation) {
                $this->validationSkippedSets[] = [
                    'id'      => $qset->id,
                    'json_id' => $qset->json_id,
                ];
            }

            // 多言語のアップサート
            $this->upsertSetTranslations($qset, $setJson);

            // pivot再同期
            if (isset($setJson['questions']) && is_array($setJson['questions'])) {
                $this->syncQuestionSetQuestions($qset, $setJson['questions']);
            }

            DB::commit();

            // ここで取り込みに成功した json_id を配列に加える
            $this->importedJsonIds[] = $jsonId;

            $this->info("Upserted question_set json_id={$jsonId}");
            return true;

        } catch (\Throwable $e) {
            DB::rollBack();
            $this->error("Error upserting question_set json_id={$setJson['json_id']}: " . $e->getMessage());
            return false;
        }
    }

    /**
     * question_set_translations の upsert
     */
    private function upsertSetTranslations(QuestionSet $qset, array $setJson): void
    {
        $locales = ['ja','en'];
        foreach ($locales as $locale) {
            $title       = $setJson['title'][$locale]       ?? null;
            $description = $setJson['description'][$locale] ?? null;
            $background  = $setJson['background'][$locale]  ?? null;

            if ($title !== null || $description !== null || $background !== null) {
                QuestionSetTranslation::updateOrCreate(
                    [
                        'question_set_id' => $qset->id,
                        'locale'          => $locale,
                    ],
                    [
                        'title'       => $title,
                        'description' => $description,
                        'background'  => $background,
                    ]
                );
            }
        }
    }

    /**
     * ◆自前の問題だけを差し替える(=generated_by_llm=false のみ削除→再登録)処理に変更。
     * ◆LLM生成問題(generated_by_llm=true)は残す
     * ◆再インポート後に question_generation_logs を参照し、LLM生成された問題を再ピボットする
     */
    private function syncQuestionSetQuestions(QuestionSet $qset, array $questionJsonIds): void
    {
        // 1) まずは "自前で用意された問題" のみ pivot を削除
        //    (==> generated_by_llm=false の question だけ)
        //    LLM問題(generated_by_llm=true)は削除しない
        $questionIdsToRemove = DB::table('question_set_questions')
            ->join('questions','questions.id','=','question_set_questions.question_id')
            ->where('question_set_questions.question_set_id', $qset->id)
            ->where('questions.generated_by_llm', false)
            ->pluck('question_set_questions.id')
            ->toArray();

        if (!empty($questionIdsToRemove)) {
            DB::table('question_set_questions')
                ->whereIn('id', $questionIdsToRemove)
                ->delete();
        }

        // 2) JSONに書かれた "自前問題" を再登録
        foreach ($questionJsonIds as $idx => $qJsonId) {
            $question = Question::where('json_id', $qJsonId)->first();
            if (!$question) {
                $this->warn("Question not found for json_id={$qJsonId}");
                continue;
            }
            // generated_by_llm=true の場合はスキップ
            if ($question->generated_by_llm) {
                $this->warn("Skip LLM question: {$qJsonId} (already pivoted or will pivot next).");
                continue;
            }

            // pivotMaxOrder
            $pivotMaxOrder = DB::table('question_set_questions')
                ->where('question_set_id', $qset->id)
                ->max('order');
            $newOrder = ($pivotMaxOrder ?? 0) + 1;

            // 既に pivot があるかどうか
            $existsPivot = DB::table('question_set_questions')
                ->where('question_set_id', $qset->id)
                ->where('question_id',$question->id)
                ->exists();

            if (!$existsPivot) {
                DB::table('question_set_questions')->insert([
                    'uuid'               => (string) Str::uuid(),
                    'question_set_id'  => $qset->id,
                    'question_id'      => $question->id,
                    'order'            => $newOrder,
                    'created_at'       => now(),
                    'updated_at'       => now(),
                ]);
            }
        }

        // 3) LLM生成の問題を再アタッチ
        $llmLogs = QuestionGenerationLog::where('question_set_id', $qset->id)
            ->where('is_success', true)
            ->whereNotNull('question_id')
            ->get();

        foreach ($llmLogs as $log) {
            $pivotMaxOrder = DB::table('question_set_questions')
                ->where('question_set_id', $qset->id)
                ->max('order');
            $newOrder = ($pivotMaxOrder ?? 0) + 1;

            $qId = $log->question_id;
            $existsQ = Question::where('id',$qId)
                ->where('status', QuestionStatus::PUBLISHED->value)
                ->where('generated_by_llm',true)
                ->exists();
            if (!$existsQ) {
                $this->warn("LLM question not found or not generated_by_llm for question_id={$qId}. Skipped.");
                continue;
            }

            $existsPivot = DB::table('question_set_questions')
                ->where('question_set_id', $qset->id)
                ->where('question_id',$qId)
                ->exists();

            if (!$existsPivot) {
                DB::table('question_set_questions')->insert([
                    'uuid'               => (string) Str::uuid(),
                    'question_set_id'  => $qset->id,
                    'question_id'      => $qId,
                    'order'            => $newOrder,
                    'created_at'       => now(),
                    'updated_at'       => now(),
                ]);
                $this->info("Re-attached LLM question_id={$qId} to QSet={$qset->id} with order={$newOrder}.");
            }
        }
    }

    private function findUnitUuid(?string $unitJsonId): ?string
    {
        if (!$unitJsonId) return null;
        return Unit::where('json_id', $unitJsonId)->value('id') ?: null;
    }

    private function parseQuestionSetStatus(string $statusString): int
    {
        try {
            return QuestionSetStatus::fromString($statusString)->value;
        } catch (\InvalidArgumentException) {
            return QuestionSetStatus::DRAFT->value;
        }
    }

    private function parseLLMGenerationStatus(string $rawStatus): int
    {
        try {
            return QuestionSetLLMGenerationStatus::fromString($rawStatus)->value;
        } catch (\InvalidArgumentException) {
            return QuestionSetLLMGenerationStatus::DISABLED->value;
        }
    }

    /**
     * 未紐づけ(どの question_set にも属していない) question を表示
     */
    private function showUnlinkedQuestions(): void
    {
        $unlinked = DB::table('questions')
            ->select('json_id')
            ->whereNotIn('id', function($sub){
                $sub->select('question_id')->from('question_set_questions');
            })
            ->get();

        if ($unlinked->isEmpty()) {
            $this->info("All questions are linked to at least one question_set.");
        } else {
            $this->warn("以下の問題は、いずれの question_set にも紐づいていません:");
            foreach ($unlinked as $q) {
                $this->line("  - " . $q->json_id);
            }
        }
    }

    /**
     * どのJSONファイルにも該当しなかった question_sets のステータスを JSON_NOT_MATCHED に更新し、
     * 最後にその id と json_id を一覧表示する
     */
    private function updateNotMatchedSets(): void
    {
        $notMatched = QuestionSet::whereNotIn('json_id', $this->importedJsonIds)->get();
        if ($notMatched->isEmpty()) {
            $this->info("No question sets found that do not match any imported JSON.");
            return;
        }

        foreach ($notMatched as $qSet) {
            if ($qSet->status !== QuestionSetStatus::JSON_NOT_MATCHED->value) {
                $qSet->status = QuestionSetStatus::JSON_NOT_MATCHED->value;
                $qSet->save();
            }
        }

        // JSON_NOT_MATCHED なセットの数を記録
        $this->notMatchedCount = $notMatched->count();

        $this->info("=== The following question_sets were set to JSON_NOT_MATCHED ===");
        foreach ($notMatched as $qSet) {
            $this->line("- ID: {$qSet->id}, json_id: {$qSet->json_id}");
        }
    }
}



```
QuestionSet のバリデーション
```php
<?php

namespace App\Services\Utils\Question;

use App\Enums\QuestionSetStatus;
use App\Enums\QuestionSetLLMGenerationStatus;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Validator;

/**
 * Class QuestionSetJsonManageService
 *
 * 【概要】
 *   このサービスクラスは、問題セットJSON（QuestionSetJSON）のバリデーションを担当する。
 *   後で分かりやすいように、日本語のエラーメッセージをまとめて返す実装を行い、
 *   バリデーションエラーがあった場合でも「例外を投げず」、呼び出し側でスキップ処理を行えるようにしている。
 *
 * 【バリデーションの大枠】
 *   - JSON仕様の各項目についてバリデーションを行う
 *   - データベース存在チェックや Enum 変換チェックは、Validator の after() コールバックで実施
 *   - バリデーションNGの場合はエラー配列を返し、呼び出し側で「スキップ」などの対処を可能にする
 *
 * 【JSON仕様】
 *   {
 *     "json_id": "qset_...",
 *     "order": 800,
 *     "unit_id": "unit_s1_g3_sec100_300",
 *     "unit": {
 *       "ja": "...",
 *       "en": "..."
 *     },
 *     "title": {
 *       "ja": "...",
 *       "en": "..."
 *     },
 *     "description": {
 *       "ja": "...",
 *       "en": "..."
 *     },
 *     "background": {
 *       "ja": "...",
 *       "en": "..."
 *     },
 *     "memo": "4桁-3桁の引き算",
 *     "version": "1.0.0",
 *     "status": "PUBLISHED",
 *     "questions": [
 *       "ques_s1_g3_sec100_u300_diff100_qt51_v100_1500",
 *       "ques_s1_g3_sec100_u300_diff100_qt51_v100_1600"
 *     ],
 *     "llm_generation_status": "ENABLED",
 *     "generate_question_prompt": {
 *       "ja": "",
 *       "en": ""
 *     },
 *     "generate_question_prompt_file_name": 1
 *   }
 *
 * 【主なバリデーション項目】
 *   - json_id                 : 必須、文字列
 *   - order                   : 必須、数値
 *   - unit_id                 : 必須、文字列 → units テーブルの json_id が存在するか
 *   - unit                    : 必須、オブジェクト({ja, en}両方必須)
 *   - title                   : 必須、オブジェクト({ja, en}両方必須)
 *   - description             : 必須、オブジェクト({ja, en}両方必須)
 *   - background             : 必須、オブジェクト({ja, en}両方必須)
 *   - memo                    : 必須、文字列
 *   - version                 : 必須、x.x.x形式
 *   - status                  : 必須、QuestionSetStatus に存在する文字列
 *   - questions               : 必須、配列。中身は questionsテーブルの json_id が存在するか
 *   - llm_generation_status   : 必須、QuestionSetLLMGenerationStatus enum に存在する文字列
 *
 *   - generate_question_prompt:
 *       llm_generation_status=ENABLED の時 → 必須オブジェクト({ja, en}両方必須)
 *       DISABLED の時 → 入力あってもなくてもOK（ビジネス上は不要だが、仕様で必須ではない）
 *
 *   - generate_question_prompt_file_name:
 *       llm_generation_status=ENABLED の時 → 必須、数値
 *         かつ /resources/prompts/generate/question/{generate_question_prompt_file_name}.txt が存在していること
 *       DISABLED の時 → 入力あってもなくてもOK
 *
 * 【使用例】
 *   $service = new QuestionSetJsonManageService();
 *   $errors = $service->validateQuestionSetJson($jsonData);
 *   if (!empty($errors)) {
 *       // バリデーションエラー → スキップ or ログ出力
 *   } else {
 *       // バリデーション成功 → DB保存処理などへ
 *   }
 *
 * 【注意】
 *   - バリデーションエラーがあっても例外は投げず errors配列を返す
 *   - スキップかどうかは呼び出し元判断
 *   - 言語定数 (ja, en) はクラス定数 LANGUAGES で定義
 */
class QuestionSetJsonManageService
{
    /**
     * 言語定数 (アプリケーション全体で共通利用)
     */
    public const LANGUAGES = ['ja', 'en'];

    /**
     * 問題セットJSONをバリデーションする
     *
     * @param array $jsonData
     * @return array
     *   バリデーションエラーが無ければ「空配列」、
     *   エラーがある場合「['フィールド名'=>['エラー文', ...], ...]」の形で返す
     */
    public function validateQuestionSetJson(array $jsonData): array
    {
        // ベースのルールチェック
        $validator = Validator::make($jsonData, $this->rules(), $this->messages());

        // afterコールバック: DB存在チェック・Enum変換チェック・LLM依存チェック
        $validator->after(function ($v) use ($jsonData) {
            // unitsテーブルに unit_id があるか
            $this->checkUnitIdExists($v, $jsonData);

            // status
            $this->checkQuestionSetStatus($v, $jsonData);

            // questionsテーブルに questions[] の各json_idが存在するか
            $this->checkQuestionsExist($v, $jsonData);

            // llm_generation_status → QuestionSetLLMGenerationStatus
            //   + それがENABLEDなら generate_question_prompt, generate_question_prompt_file_name を追加チェック
            $this->checkLlmGeneration($v, $jsonData);
        });

        // バリデーションを実行
        $validator->fails();

        // 結果を配列で返す
        return $validator->errors()->toArray();
    }

    /**
     * バリデーションルール定義
     *   - llm_generation_status も必須(string)
     *   - generate_question_prompt は「ベースの必須」には含めず、LLMで必須かどうかを after で制御
     *   - generate_question_prompt_file_name も after で必須制御
     *
     * @return array
     */
    private function rules(): array
    {
        return [
            'json_id'    => ['required','string'],
            'order'      => ['required','integer'],
            'unit_id'    => ['required','string'],
            'memo'       => ['required','string'],
            'version'    => ['required','regex:/^\d+\.\d+\.\d+$/'],
            'status'     => ['required','string'],
            'llm_generation_status' => ['required','string'],

            // unit/title/description/background => 多言語オブジェクト
            'unit'        => ['required','array'],
            'unit.ja'     => ['required','string'],
            'unit.en'     => ['required','string'],

            'title'       => ['required','array'],
            'title.ja'    => ['required','string'],
            'title.en'    => ['required','string'],

            'description'       => ['required','array'],
            'description.ja'    => ['required','string'],
            'description.en'    => ['required','string'],

            'background'       => ['required','array'],
            'background.ja'    => ['required','string'],
            'background.en'    => ['required','string'],

            // generate_question_prompt: llm_generation_status=ENABLED の時だけ必須
            // → ここでは "array" などルール記載はできるが "required" は after() で条件付け
            'generate_question_prompt' => ['sometimes','array'],
            'generate_question_prompt.ja' => ['sometimes','string'],
            'generate_question_prompt.en' => ['sometimes','string'],

            // generate_question_prompt_file_name: llm_generation_status=ENABLED の時に必須
            // → after() で条件付け
            'generate_question_prompt_file_name' => ['sometimes','string'],

            // questions => 必須(配列)
            'questions' => ['required','array'],
        ];
    }

    /**
     * 日本語エラーメッセージ
     *
     * @return array
     */
    private function messages(): array
    {
        return [
            'required' => ':attribute は必須です。',
            'integer'  => ':attribute には数値を指定してください。',
            'string'   => ':attribute は文字列を指定してください。',
            'regex'    => ':attribute の形式が不正です。(例: 1.0.0)',
            'array'    => ':attribute は配列である必要があります。',
            'numeric'  => ':attribute には数値を指定してください。',
        ];
    }

    /**
     * unit_id が units テーブルに存在するかをチェック
     */
    private function checkUnitIdExists($validator, array $jsonData): void
    {
        $unitId = $jsonData['unit_id'] ?? null;
        if ($unitId) {
            $exists = DB::table('units')
                ->where('json_id', $unitId)
                ->exists();
            if (!$exists) {
                $validator->errors()->add('unit_id', "指定された unit_id='{$unitId}' は unitsテーブルに存在しません。");
            }
        }
    }

    /**
     * status が QuestionSetStatus で定義されているかチェック
     */
    private function checkQuestionSetStatus($validator, array $jsonData): void
    {
        $rawStatus = $jsonData['status'] ?? '';
        if (!$rawStatus) {
            return; // requiredで弾かれているはず
        }
        try {
            \App\Enums\QuestionSetStatus::fromString($rawStatus);
        } catch (\InvalidArgumentException) {
            $validator->errors()->add('status', "不明なステータスです: {$rawStatus}");
        }
    }

    /**
     * questions[] の各要素が、questionsテーブルの json_id に存在するかチェック
     */
    private function checkQuestionsExist($validator, array $jsonData): void
    {
        $list = $jsonData['questions'] ?? [];
        if (!is_array($list)) {
            return;
        }

        foreach ($list as $idx => $qJsonId) {
            if (!is_string($qJsonId) || !$qJsonId) {
                $validator->errors()->add("questions.{$idx}", "questionsの要素は文字列のjson_idが必要です。");
                continue;
            }

            $exists = DB::table('questions')
                ->where('json_id', $qJsonId)
                ->exists();
            if (!$exists) {
                $validator->errors()->add("questions.{$idx}", "問題 json_id='{$qJsonId}' はDBに存在しません。");
            }
        }
    }

    /**
     * llm_generation_status → QuestionSetLLMGenerationStatus で判定し、
     * ENABLED なら generate_question_prompt, generate_question_prompt_file_name が必須
     */
    private function checkLlmGeneration($validator, array $jsonData): void
    {
        // llm_generation_status Enumチェック
        $rawLlmStatus = $jsonData['llm_generation_status'] ?? null;
        if (!$rawLlmStatus) {
            return; // requiredで弾かれているはず
        }

        try {
            $enumVal = \App\Enums\QuestionSetLLMGenerationStatus::fromString($rawLlmStatus);
        } catch (\InvalidArgumentException) {
            $validator->errors()->add('llm_generation_status', "不明な llm_generation_status: {$rawLlmStatus}");
            return;
        }

        // ENABLED の場合 → generate_question_prompt, generate_question_prompt_file_name が必須
        if ($enumVal === \App\Enums\QuestionSetLLMGenerationStatus::ENABLED) {
            // generate_question_prompt => オブジェクト必須
            if (empty($jsonData['generate_question_prompt']) || !is_array($jsonData['generate_question_prompt'])) {
                $validator->errors()->add('generate_question_prompt', "llm_generation_status=ENABLED のため generate_question_prompt(オブジェクト) が必須です。");
            } else {
                // 多言語キー必須
                foreach (self::LANGUAGES as $lang) {
                    if (empty($jsonData['generate_question_prompt'][$lang]) || !is_string($jsonData['generate_question_prompt'][$lang])) {
                        $validator->errors()->add("generate_question_prompt.{$lang}", "llm_generation_status=ENABLED のため generate_question_prompt.{$lang} は必須の文字列です。");
                    }
                }
            }

            // generate_question_prompt_file_name => 必須で numeric
            if (!isset($jsonData['generate_question_prompt_file_name'])) {
                $validator->errors()->add('generate_question_prompt_file_name', "llm_generation_status=ENABLED のため generate_question_prompt_file_name(数値) が必須です。");
            } else {
                $num = $jsonData['generate_question_prompt_file_name'];
                // テキストファイルの存在チェック
                $path = resource_path("prompts/generate/question/{$num}.txt");
                if (!file_exists($path)) {
                    $validator->errors()->add('generate_question_prompt_file_name', "generate_question_prompt_file_name={$num} のファイルが存在しません。(path={$path})");
                }
                // 数値ではなくてよい
//                if (!is_numeric($num)) {
//                    $validator->errors()->add('generate_question_prompt_file_name', "generate_question_prompt_file_name は数値を指定してください。");
//                } else {
//
//                }
            }
        }
    }
}

```
