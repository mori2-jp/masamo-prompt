セクションを一覧及び単体で取得するAPIを実装してください

# 実装方針
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


ーーー
<?php

use App\Http\Controllers\API\V1\Answer\AnswerController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\API\V1\Auth\AuthController;

Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');

Route::prefix('v1')->group(function () {
    Route::post('/answer', [AnswerController::class, 'store'])->name('v1.answer.store');

    Route::prefix('lessons')->group(function () {

    });

    Route::prefix('questions')->group(function () {
        Route::get('/sample1', function () {
            $json = <<<'JSON'
{
  "order": 1,
  "id": "prob_s1_l2_001",
  "level_id": "level-001",
  "difficulty_id": "diff-001",
  "version": "1.0.0",
  "problem_type": "FILL_IN_THE_BLANK",
  "question_format": "NUMERIC_ANSWER",
  "question_text": {
    "ja": "▢にあてはまる数を答えなさい。",
    "en": "Please answer the numbers that fit in the blanks."
  },
  "explanation": {
    "ja": "",
    "en": ""
  },
  "metadata": {
    "question": "8 × 4 = ▢ × 8 = ▢",
    "inputFormat": {
      "type": "fixed",
      "fields": [
        {
          "field_id": "f_1",
          "attribute": "number",
          "collect_answer": "4"
        },
        {
          "field_id": "f_2",
          "attribute": "number",
          "collect_answer": "4"
        }
      ],
      "question_components": [
        {
          "type": "text",
          "content": "8 × 4 = "
        },
        {
          "type": "blank",
          "field_id": "f_1"
        },
        {
          "type": "text",
          "content": " × 8 = "
        },
        {
          "type": "blank",
          "field_id": "f_2"
        }
      ]
    },
    "": [],
    "evaluationMethod": "LLM"
  },
  "learning_requirements": [
    {
      "learning_subject": "算数",
      "learning_no": 10,
      "learning_requirement": "数と計算 計算の意味・方法 乗法の意味",
      "learning_required_competency": "かけ算は同じ数の繰り返し加法であるという本質を理解できる",
      "learning_category": "A",
      "learning_grade_level": "小2",
      "learning_url": "https://docs.google.com/spreadsheets/d/1W5vaFHcyU_BrMwb1JLZ-DpyFmXXZPaYMTHIITcwLqqY/edit?gid=0#gid=0&range=15:15"
    }
  ],
  "created_at": "2025-01-01 00:00:00",
  "updated_at": "2025-01-01 00:00:00"
}
JSON;

            return response($json, 200)
                ->header('Content-Type', 'application/json');
        });

        Route::get('/sample2', function () {
            $json = <<<'JSON'
{
  "order": 1,
  "id": "prob_s1_l2_002",
  "level_id": "level-001",
  "difficulty_id": "diff-001",
  "version": "1.0.0",
  "problem_type": "SCENARIO",
  "question_format": "NUMERIC_ANSWER",
  "question_text": {
    "ja": "カレールー3つとりんご2つは必須で購入する必要があります。具材ごとの価格は次のとおり：りんご130円、カレールー100円、じゃがいも150円、にんじん200円、たまねぎ150円。予算は1,000円です。合計金額が予算内に収まるように買い物をし、(合計金額, お釣り)を導き出してください。",
    "en": "You must buy at least 3 curry roux and 2 apples. Prices per item: apple 130 yen, curry roux 100 yen, potato 150 yen, carrot 200 yen, onion 150 yen, with a total budget of 1,000 yen. Make sure the total cost does not exceed the budget and answer (total cost, change)."
  },
  "explanation": {
    "ja": "",
    "en": ""
  },
  "metadata": {
    "options": [
      {
        "option_id": 1,
        "name_ja": "りんご",
        "name_en": "apple",
        "price": 130,
        "required": 2
      },
      {
        "option_id": 2,
        "name_ja": "カレールー",
        "name_en": "curry roux",
        "price": 100,
        "required": 3
      },
      {
        "option_id": 3,
        "name_ja": "じゃがいも",
        "name_en": "potato",
        "price": 150,
        "required": 0
      },
      {
        "option_id": 4,
        "name_ja": "にんじん",
        "name_en": "carrot",
        "price": 200,
        "required": 0
      },
      {
        "option_id": 5,
        "name_ja": "たまねぎ",
        "name_en": "onion",
        "price": 150,
        "required": 0
      }
    ],
    "conditions": [
      {
        "total_amount": 1000
      }
    ],
    "inputFormat": {
      "type": "custom",
      "fields": [
        {
          "field_id": "f_1",
          "attribute": "array",
          "name": "selected options",
          "keys": [
            "option_id",
            "quantity"
          ]
        },
        {
          "field_id": "f_2",
          "attribute": "number",
          "name": "price"
        },
        {
          "field_id": "f_3",
          "attribute": "number",
          "name": "change"
        }
      ]
    },
    "evaluationMethod": "CODE",
    "evaluationSpec": {
      "checkerMethod": "caluclateMethod"
    }
  },
  "learning_requirements": [
    {
      "learning_subject": "算数",
      "learning_no": 5,
      "learning_requirement": "A 数と計算 日常生活への活用 数の活用",
      "learning_required_competency": "身の回りの物や人数などを数えて，必要な数を報告できる",
      "learning_category": "A",
      "learning_grade_level": "小1",
      "learning_url": "https://docs.google.com/spreadsheets/xxxx#range=20:20"
    }
  ],
  "created_at": "2025-01-01 00:00:00",
  "updated_at": "2025-01-01 00:00:00"
}
JSON;

            return response($json, 200)
                ->header('Content-Type', 'application/json');
        });

        Route::get('/sample3', function () {
            $json = <<<'JSON'
{
  "order": 1,
  "id": "ques_s1_l2_qt51_002",
  "level_id": "lev_002",
  "difficulty_id": "diff_001",
  "version": "1.0.0",
  "problem_type": "FILL_IN_THE_BLANK",
  "question_format": "NUMERIC_ANSWER",
  "question_text": {
    "ja": "▢にあてはまる数を答えなさい。",
    "en": "Please answer the numbers that fit in the blanks."
  },
  "explanation": {
    "ja": "",
    "en": ""
  },
  "metadata": {
    "question": "8 × 4 = 8 × 3 + ▢ = ▢",
    "inputFormat": {
      "type": "fixed",
      "fields": [
        {
          "field_id": "f_1",
          "attribute": "numeric",
          "collect_answer": "8"
        },
        {
          "field_id": "f_2",
          "attribute": "numeric",
          "collect_answer": "32"
        }
      ]
    },
    "evaluationMethod": "LLM"
  },
  "learning_requirements": [
    {
      "learning_subject": "算数",
      "learning_no": 10,
      "learning_requirement": "数と計算 計算の意味・方法 乗法の意味",
      "learning_required_competency": "かけ算は同じ数の繰り返し加法であるという本質を理解できる",
      "learning_category": "A",
      "learning_grade_level": "小2",
      "learning_url": "https://docs.google.com/spreadsheets/d/1W5vaFHcyU_BrMwb1JLZ-DpyFmXXZPaYMTHIITcwLqqY/edit?gid=0#gid=0&range=15:15"
    }
  ],
  "status": "PUBLISHED",
  "generated_by_llm": false,
  "created_at": "2025-01-01 00:00:00",
  "updated_at": "2025-01-01 00:00:00"
}
JSON;

            return response($json, 200)
                ->header('Content-Type', 'application/json');
        });
    });



    Route::prefix('auth')->group(function () {
        // テスト用、Emailログインは採用してない。
        Route::post('/loginByEmail', [AuthController::class, 'loginByEmail']);

        Route::post('/loginWithGoogle', [AuthController::class, 'loginWithGoogle']);

//        Route::post('/loginWithApple', [AuthController::class, 'loginWithApple']);
    });

    /**
     * 認証済み
     */
    Route::middleware(['auth:sanctum'])->group(function () {
        Route::prefix('auth')->group(function () {
            Route::post('/logout', [AuthController::class, 'logout']);
            Route::get('/me', [AuthController::class, 'me']);
//            Route::put('/me', [AuthController::class, 'update']);
        });
    });
});

ーーー
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

ーーー
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
