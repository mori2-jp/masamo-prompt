
UserQuestionSet の ステータスをUserQuestionStatus::SKIPにするAPIを実装したい。
以下の 仕様を｛参考の仕様書｝を参考にして作成してください。
# 条件
１
API のエンドポイントは、
/api/v1/user-questions/{user_question_id}/skip
です。

２
パラメータで受け取った user_question_id の UserQuestion が存在するかチェックする

3
UserQuestion の Status が、UserQuestionStatus::CORRECT または INCORRECT の場合は、問題は回答済み　とエラーにする

4
UserQuestion の Status が、UserQuestionStatus::NOT_START または PROGRESS または SKIP の場合は、UserQuestion の Status を UserQuestionStatus::SKIP にする

5
次の問題を取得する。UserQuestion が属する、UserQuestionSet に属する UserQuestion　で、status が NOT_START または PROGRESS のデータで Order  が一番 asc のものを取得する

6
次の問題が存在しなくて、UserQuestion が属する、UserQuestionSet の status が UserQuestionSetStatus::PROGRESS の場合は、その UserQuestionSet の finished_at に今の時間を保存し、Status を UserQuestionSetStatus::COMPLETE にする 

7
次の問題の uuid を "next_user_question_id": "next-uuid" としてレスポンスする

（5,6,7 の処理について、AnswerService で getNextLearningQuestion を使用している箇所と同様）



# 説明
question_sets：問題（questions）を束ねるグループ
questions: 問題
sections: 単元をまとめるセクション
unit: 問題セットをまとめる単元
question_set_questions：question_sets と questions を紐づけるPivotテーブル。questions は複数のquestion_sets に紐づく事があり多対多の関係なのでこのような設計になっている。
user_question_sets: ユーザーが学習した questions_sets。学習開始時に question_sets_id と紐づいて status （UserQuestionSetsStatus::NOT_START) が未開始の状態で生成され、進捗ステータスはスコアなどが管理される
user_questions: ユーザーが学習した questions。学習開始時に、学習を開始した question_sets に紐づく questions が全て、 user_question_sets_id と question_id（questions）を紐づけて status （UserQuestionStatus::NOT_START) が未開始の状態で全てのquestionsの数の分、生成され、進捗ステータスはスコアなどが管理される


以下は Group に関する仕様です。
以下の 仕様を｛参考の仕様書｝を参考にして作成してください。
非エンジニア対象なので、説明は、SQLクエリなどではなく文字列で説明してください（カラム名などを使うのはOK


# 関連ファイル
```php
<?php

namespace App\Enums;

enum UserQuestionStatus: int
{
    case NOT_START = 1;
    case CORRECT = 50;
    case INCORRECT = 100;
    case SKIP = 150;
    case PROGRESS = 200;

    public function description(): string
    {
        return match ($this) {
            self::NOT_START => 'Not Started',
            self::CORRECT => 'Correct',
            self::INCORRECT => 'Incorrect',
            self::SKIP => 'Skip',
            self::PROGRESS => 'Progress',
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
            'PROGRESS' => self::PROGRESS,
            default => throw new \InvalidArgumentException("Unknown status string: {$statusString}")
        };
    }
}


<?php

namespace App\Services\V1\Answer;

use App\Dtos\V1\Answer\AnswerCheckDto;
use App\Enums\EvaluationCheckerMethod;
use App\Enums\EvaluationMethod;
use App\Enums\UserQuestionSetStatus;
use App\Enums\UserQuestionStatus;
use App\Models\Question\Question;
use App\Models\User\UserQuestion;
use App\Models\User\UserQuestionSet;
use App\Services\Utils\Evaluation\EvaluationCheckService;
use App\Services\Utils\Llm\LlmManageService;
use App\Services\Utils\Question\QuestionJsonManageService;
use Carbon\Carbon;
use Illuminate\Support\Facades\DB;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;
use Illuminate\Validation\ValidationException;

/**
 * Class AnswerService
 *
 * 【概要】
 *   ユーザーからの回答を受け取り、正誤判定を行うビジネスロジックを提供。
 *   LLM 評価部分は LlmService に切り出した。
 */
class AnswerService
{
    private QuestionJsonManageService $questionJsonManageService;
    private EvaluationCheckService $evaluationCheckService;
    private LlmManageService $llmManageService;

    public function __construct(
        QuestionJsonManageService $questionJsonManageService,
        EvaluationCheckService $evaluationCheckService,
        LlmManageService $llmManageService
    ) {
        $this->questionJsonManageService = $questionJsonManageService;
        $this->evaluationCheckService = $evaluationCheckService;
        $this->llmManageService = $llmManageService;
    }

    /**
     * @param string $userId
     * @param string $userQuestionId
     * @param string $answerDataJson
     * @return AnswerCheckDto
     *
     * 【処理概要】
     *   1) user_question_id と userId から user_questions, user_question_sets, question を特定
     *   2) (LLM or CODE) による回答判定
     *   3) user_questions に正誤ステータスや回答データを保存
     *   4) 次の問題があれば取得 / なければ user_question_sets を COMPLETE に
     *   5) AnswerCheckDto を返す
     */
    public function evaluateAnswer(?int $userId = null, string $userQuestionId, string $answerDataJson): AnswerCheckDto
    {
        // user_questions を取得し、ユーザー本人か判定
        $userQuestion = UserQuestion::where('uuid', $userQuestionId)->first();
        if (!$userQuestion) {
            throw new NotFoundHttpException(
                __("errors.api.answer.user_question_not_found", ['id' => $userQuestionId])
            );
        }
        $userQuestionSet = UserQuestionSet::find($userQuestion->user_question_set_id);
        if (!$userQuestionSet) {
            throw new NotFoundHttpException(
                __("errors.api.answer.user_question_set_not_found", ['id' => $userQuestion->user_question_set_id])
            );
        }

        // question
        $question = Question::find($userQuestion->question_id);
        if (!$question) {
            throw new NotFoundHttpException(
                __("errors.api.answer.question_not_found", ['id' => $userQuestion->question_id])
            );
        }

        // user_question->metadata を array化
        $metadataArr = json_decode($userQuestion->metadata, true);

        if (!is_array($metadataArr)) {
            throw ValidationException::withMessages(["metadata" => "user_questionのmetadataが不正です。"]);
        }
        $decodedLocalized = $this->questionJsonManageService->localizeMetadata($metadataArr);

        // ユーザー回答
        $answerArr = json_decode($answerDataJson, true);
        if (!is_array($answerArr)) {
            throw ValidationException::withMessages([
                'answer_data' => "回答JSONの形式が不正です。"
            ]);
        }

        // ユーザー回答バリデーション
        $this->questionJsonManageService->validateUserAnswer($answerArr, $decodedLocalized);

        $answeredAt = Carbon::now();

        DB::beginTransaction();
        try {
            // LLMかCODEか
            if ($question->evaluation_method == EvaluationMethod::LLM->value) {
                // ★ LlmService を使って評価
                $llmResult = $this->llmManageService->evaluateUserAnswerWithLLM($answerArr, $question, $userQuestion);

                // LLM応答をローカライズ
                $convertedLocalized = $this->questionJsonManageService->localizeLlmResult($llmResult);

                // LLMの応答に is_correct が含まれているかを判定
                $isAnyCorrect = data_get($llmResult, 'is_correct');
                if (empty($isAnyCorrect)) {
                    // fields のいずれかが true なら correct
                    $fields = data_get($llmResult, 'fields', []);
                    $isAnyCorrect = false;
                    foreach ($fields as $field) {
                        $flag = data_get($field, 'is_correct');
                        if ($flag === true || $flag === "true") {
                            $isAnyCorrect = true;
                            break;
                        }
                    }
                }

                $status = $isAnyCorrect
                    ? UserQuestionStatus::CORRECT->value
                    : UserQuestionStatus::INCORRECT->value;

                $saveAnswerData = $convertedLocalized;

            } else {
                // CODE
                $checkerMethodRaw = $userQuestion->checker_method;

                if (!$checkerMethodRaw) {
                    throw ValidationException::withMessages([
                        'checker_method' => "evaluation_spec.checker_method が指定されていません。"
                    ]);
                }
                $checkerEnum = EvaluationCheckerMethod::from($checkerMethodRaw);
                $methodName = $checkerEnum->methodName();

                $evalResp = json_decode($userQuestion->evaluation_response_format ?? '{}', true);
                if (!is_array($evalResp)) {
                    throw ValidationException::withMessages([
                        'evaluation_response_format' => "evaluation_response_format が不正です。"
                    ]);
                }

                // EvaluationCheckService で正誤判定
                if (!method_exists($this->evaluationCheckService, $methodName)) {
                    throw new \RuntimeException("Checker method '{$methodName}' not found in EvaluationCheckService");
                }
                $resultFormat = call_user_func_array(
                    [$this->evaluationCheckService, $methodName],
                    [$answerArr, $evalResp]
                );

                $isCorrectOverall = (data_get($resultFormat, 'is_correct') === true);
                $status = $isCorrectOverall
                    ? UserQuestionStatus::CORRECT->value
                    : UserQuestionStatus::INCORRECT->value;

                $saveAnswerData = $resultFormat;
            }

            // user_questions の status を更新
            if (! in_array($userQuestion->status, [UserQuestionStatus::CORRECT, UserQuestionStatus::INCORRECT])) {
                $userQuestion->status      = $status;
                $userQuestion->answered_at = $answeredAt;
                $userQuestion->answer_data = $saveAnswerData;
            }
            // answer_user_idに保存
            $userQuestion->answer_user_id = $userId;
            $userQuestion->save();

            // 次に学習する問題を取得
            $nextUserQuestion = $userQuestionSet->getNextLearningQuestion();
            if (!$nextUserQuestion
                && $userQuestionSet->status == UserQuestionSetStatus::PROGRESS->value) {
                $userQuestionSet->status = UserQuestionSetStatus::COMPLETE->value;
                $userQuestionSet->finished_at = Carbon::now();
                $userQuestionSet->save();
            }

            DB::commit();

            return $this->buildAnswerCheckDto(
                $userQuestion,
                $question,
                $nextUserQuestion,
                $saveAnswerData
            );

        } catch (\Throwable $e) {
            DB::rollBack();
            throw $e;
        }
    }

    /**
     * 回答後のレスポンス用DTO構築
     */
    private function buildAnswerCheckDto(
        UserQuestion $userQuestion,
        Question $question,
        ?UserQuestion $nextUserQuestion,
        array $answerData
    ): AnswerCheckDto {
        $nextUserQuestionId = $nextUserQuestion?->uuid;
        return new AnswerCheckDto(
            content: $answerData,
            next_user_question_id: $nextUserQuestionId
        );
    }
}
<?php

namespace App\Models\User;

use App\Enums\UserQuestionStatus;
use App\Models\BaseModel;
use App\Models\Question\QuestionSet;
use App\Traits\UsesUuid;
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
 * @property string $id
 * @property string $user_id
 * @property string $question_set_id
 * @property string|null $score
 * @property string|null $question_link_token
 * @property string|null $expires_at_question_link_token
 * @property string $started_at
 * @property string|null $finished_at
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property \Illuminate\Support\Carbon|null $deleted_at
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereCreatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereDeletedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereFinishedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereQuestionSetId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereScore($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereStartedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereUpdatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereUserId($value)
 * @property int $status
 * @property-read QuestionSet $questionSet
 * @property-read \App\Models\User\User $user
 * @property-read \Illuminate\Database\Eloquent\Collection<int, \App\Models\User\UserQuestion> $userQuestions
 * @property-read int|null $user_questions_count
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereStatus($value)
 * @property-read \Illuminate\Database\Eloquent\Collection<int, \App\Models\User\UserQuestionSetTranslation> $translations
 * @property-read int|null $translations_count
 * @property string $uuid
 * @property int $selection_type 1: manual, 2: auto
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereExpiresAtQuestionLinkToken($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereQuestionLinkToken($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereSelectionType($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereUuid($value)
 * @mixin \Eloquent
 */
class UserQuestionSet extends BaseModel
{
    use UsesUuid, SoftDeletes;

    protected $with = [
        'translations',
    ];

    protected array $iso8601Timestamps = [
        'started_at',
        'finished_at',
    ];

    /*=====================================================
    * リレーション
    *=====================================================*/
    public function questionSet()
    {
        return $this->belongsTo(QuestionSet::class);
    }

    public function userQuestions()
    {
        return $this->hasMany(UserQuestion::class);
    }

    public function user()
    {
        return $this->belongsTo(User::class);
    }

    /**
     * 翻訳テーブルとのリレーション
     */
    public function translations()
    {
        return $this->hasMany(UserQuestionSetTranslation::class, 'user_question_set_id');
    }


    /*=====================================================
     * カスタムメソッド/Custom Method
    *=====================================================*/

    /**
     * 現在のlocaleに応じた翻訳を返す
     * なければfallback_localeの翻訳を返す
     */
    public function getTranslation(?string $locale = null): ?UserQuestionSetTranslation
    {
        $locale = $locale ?: app()->getLocale();
        return $this->translations->firstWhere('locale', $locale)
            ?: $this->translations->firstWhere('locale', config('app.fallback_locale'));
    }

    /**
     * 次に学習する問題 (status=NOT_START, PROGRESS) を order 昇順で1件取得して返す
     * なければ null
     * @return UserQuestion|null
     */
    public function getNextLearningQuestion(): ?UserQuestion
    {
        return $this->userQuestions()
            // 未開始だけでなく進行中も対象とすることで、PROGRESS が放置されるバグを防ぐ
            ->whereIn('status', [
                UserQuestionStatus::NOT_START->value,
                UserQuestionStatus::PROGRESS->value,
            ])
            ->orderBy('order', 'asc')
            ->first();
    }

}

```

-- # 参考の仕様書
# 問題レスポンス仕様書

最終更新: 2025‑04‑26  
担当: backend team

---

## 1. 用語と構造

| 用語                | 説明                                                                       |
| ----------------- | ------------------------------------------------------------------------ |
| **grade**         | 学年。ユーザーが属する学習ステージを示す。                                                    |
| **section**       | 問題の大カテゴリ。`grade_id` で **grade** と 1 :N。例: *足し算*。                        |
| **unit**          | **section** をさらに細分化したカテゴリ。`section_id` で **section** と 1 :N。例: *２桁＋１桁*。 |
| **question_set** | 問題セット。`unit_id` で **unit** と 1 :N。                                      |

```
Grade ── 1:N ── Section ── 1:N ── Unit ── 1:N ── QuestionSet
```


---

## 2. 問題レスポンス・フロー
下記に **「自動出題フローでは `user_question_sets.selection_type = AUTO` のみを対象とする」** という条件を反映したリライト版を示します。

---

## 2. 問題レスポンス・フロー

> **注意**: ここで説明するロジックは「自動選定モード」での出題フローを想定しています。
> そのため、本フロー内で検索する `user_question_sets` レコードは、**必ず `selection_type = UserQuestionSetSelectionType::AUTO`** であることが前提です。
> （ユーザーが手動で選択した問題 = `selection_type = MANUAL` は対象外とし、本ロジックでは扱いません。）

### 2‑1. レスポンス優先順位

1. **途中再開**
   ユーザーの現在のGrade（`users.grade_id`）に紐づく **全てのSections → 全てのUnits → 全てのQuestionSet** を対象として、
   **かつ `user_question_sets.selection_type = AUTO`** であり、`UserQuestionSet.status` が `NOT_START` または `PROGRESS` のレコードが存在するかを確認し、該当すれば以下の基準で1件返します。

    * **優先度1: `PROGRESS` のレコードがあればそちらを優先**
      複数の `PROGRESS` がある場合は、`UserQuestionSet.started_at` が **最も古い** レコードを返す。

    * **優先度2: `PROGRESS` が1件も無く、`NOT_START` のレコードがある場合**
      最も早く作成されたもの、あるいは `order` が最小のものなど、事前定義したルールで1件を返す。

2. **新規選定**
   上記 1. に該当レコードが無い場合（つまり、自動選定モードで見る限り現在のGrade内に `NOT_START` / `PROGRESS` が存在しない場合）は、
   次に下記 **「2‑2. 次の問題決定ロジック」** に従って **「次に提示するQuestionSet」** を決定します。

---

### 2‑2. 次の問題決定ロジック

本ロジックでは、*現在学習中* の `QuestionSet`（selection_type = AUTO）における正答率に応じて、
**次に提示する `QuestionSet`（selection_type = AUTO）** を決定します。
ここでいう「正答率」とは、ユーザーが直近または累積で記録した該当 `QuestionSet` の正解率です。

なお、以下で登場する「order」カラムは **同じ親キー（`unit_id` / `section_id` / `grade_id`）内でのみ有効なソート順** です。

#### A. *現在学習中* の `QuestionSet` の正答率が **TH_PASS** 以上を **SUCCESS_STREAK** 回達成した場合

1. **【Unit 内での次の `QuestionSet` 探索】**
   同じ `unit_id` を持つ `QuestionSet` のうち、`order` が *現在の `QuestionSet`* よりも次(大きい)の並びのものを検索します。
   さらに、該当の `QuestionSet` に対して既に `user_question_sets` レコードが存在する場合は、
   **`selection_type = AUTO` であるか** を確認し、対象があればそれを次に提示します。
   （まだ `user_question_sets` にレコードが無い場合は、ここで新規に `selection_type = AUTO` のレコードを作成し出題します。）

2. **【Section 内での未挑戦問題(`QuestionSet`)の探索】**
   手順 1 で該当する `QuestionSet` が無い場合、次は「同じ `grade` に属する、*現在の `QuestionSet` に紐づく `Units` が属する `Sections`*」領域における **未挑戦** の `QuestionSet` を探します。

    * **未挑戦とは**: `user_question_sets`（selection_type = AUTO）にまだレコードが無い `QuestionSet` のことを指します。
    * まずは現在の `Sections` 内に紐づく全 `Units` をチェックし、未挑戦の `QuestionSet` があれば、その中からいずれかを提示します。
    * 新作問題への対応の意図。Grade 全体だとスコープが広すぎるので、同じSection内で新作問題があれば出題するようなロジックとしている。

3. **【Section 内での次の `Units` 探索】**
   手順 2 でも該当が無い場合、同じ `Sections` 内で `order` が “現在の `unit_id` よりも次(大きい)” の `Units` を探し、
   見つかったらその `Units` に紐づく `QuestionSet`（`order` が最小）を提示します。
   （提示時は新規に `user_question_sets.selection_type = AUTO` レコードを作成する想定）

4. **【Grade 内での次の `Sections` 探索】**
   手順 3 でも見つからない場合、同じ `grade` 内で *現在の `sections` より“次(大きい)の `order`”を持つ `sections`* を探します。
   見つかったら、その中で最も `order` が小さい `Sections` → そこに紐づく最も `order` の小さい `Units` → さらに最も `order` の小さい `QuestionSet` を提示します。
   こちらも新規に `user_question_sets`（selection_type = AUTO）を登録して出題。

5. **【次の Grade 探索】**
   手順 4 で候補が無い場合、現在の `grade` を完了とみなし、
   次の `grade`（`grades` テーブルで現在の `grade` の `order` より大きいもののうち最小）を選び、その中の最初の `Sections` → 最初の `Units` → 最初の `QuestionSet` に進みます。
   ここでも、`selection_type = AUTO` のレコードを作成して出題します。

6. **【Grade 完了後のランダム選定】**
   さらに手順 5 でも候補が無い場合（当該ユーザーが対象範囲をすべて学習済み等）、
   grades の json_id が `gra_005` と `gra_006`　の `grade_id` に紐づく**全ての `Sections` → 全ての `Units` → 全ての `QuestionSet`** を対象にランダム選定します。
   こちらも提示時に `selection_type = AUTO` レコードを新規作成して返します。

#### B. *現在学習中* の `QuestionSet` の正答率が **FAIL_RATE** 未満の場合

1. **【Unit 内での QS ロールバック】**
   同じ `unit_id` の中で、*現在の `QuestionSet`* より 1 つ前(`order` が小さい方)の並びを検索し、あればその `QuestionSet` をロールバック先とします。
   既にある `user_question_sets.selection_type = AUTO` レコードを再利用し、ステータスを `NOT_START` に戻すか、
   なければ同じ `QuestionSet` で新しくレコードを作成します。

2. **【Section 内での Units ロールバック】**
   手順 1 で対象が無い場合、同じ `sections` 内で *現在の `unit_id` より 1 つ前(小さい)の `Units`* を探します。
   見つかったらその `Units` 内で `order` が降順(大→小)で最初にヒットする `QuestionSet` をロールバック先とします。
   該当 `QuestionSet` の `user_question_sets.selection_type = AUTO` レコードが無ければ新規作成。

3. **【Grade 内での Sections ロールバック】**
   手順 2 でも見つからない場合、さらに同じ `grade` 内で *現在の `sections.order` より 1 つ前(小さい)の `sections`* を探し、
   その中の最も `order` が高い `Units` → 最も `order` が高い `QuestionSet` をロールバック先とします。
   ここでも自動出題用に `selection_type = AUTO` レコードを処理します。

4. **【Grade ロールバック】**
   手順 3 でも無い場合は、現在の `grade` より 1 つ前(小さい)の `grade` を探し、
   そこで最も `order` が高い `Sections` → 最も `order` が高い `Units` → 最も `order` が高い `QuestionSet` をロールバック先とします。
   該当の `QuestionSet` に `selection_type = AUTO` が無い場合は新規作成。

5. **【最上位でも該当無しの場合】**
   4 でも見つからない場合（実運用上想定外）、
   指定した複数の `grade_id` の中で最も `order` が高い `Sections` → 最も `order` が高い `Units` → 最も `order` が高い `QuestionSet` をロールバック先として利用し、
   `selection_type = AUTO` レコードを再利用または新規作成します。

---

> **補足**
> 上記で使用する「次の `order`」「1 つ前の `order`」は、**同じ親キー（`unit_id` / `section_id` / `grade_id`）** 内でのみ通用する比較です。
> 例えば `QuestionSet.order` は同じ `unit_id` の中でのソート順であり、他の `unit_id` の `QuestionSet` とは直接比較しません。

---

このように、本フローでは **`UserQuestionSet.selection_type = AUTO`** のレコードの存在を条件に問題を検索・選定し、新規に出題するときも **selection_type = AUTO** のレコードを追加して提示する形で進行させます。

以上が、**「自動選定（AUTO）モード」** の問題レスポンス仕様です。

### 2‑3. 前提条件：order の定義（補足）

本仕様で登場する `order` カラムは、**同じ親キーのレコード群の中**でのみ有効な並び順です。異なる親キーに紐づくレコードとの間で `order` が重複しても問題ありません。以下に例を示します。

* **QuestionSet の `order`**

    * 同じ `unit_id` を持つ `QuestionSet` 同士でのみ順番を比較する。
    * 例:

        * A: question_sets.id=1, unit_id=1, order=100
        * B: question_sets.id=2, unit_id=1, order=200
        * C: question_sets.id=3, unit_id=2, order=100
        * …  (他ユニットにも `order=100` や `200` が存在しても衝突しない)

* **Unit の `order`**

    * 同じ `section_id` 内での順番を示す。
    * 例:

        * A: units.id=1, section_id=1, order=100
        * B: units.id=2, section_id=1, order=200
        * C: units.id=3, section_id=2, order=100
        * …  (section_id が異なる場合でも同じ数値を使う可能性あり)

* **Section の `order`**

    * 同じ `grade_id` 内での順番を示す。
    * 例:

        * A: sections.id=1, grade_id=1, order=100
        * B: sections.id=2, grade_id=1, order=200
        * C: sections.id=3, grade_id=2, order=100
        * …  (別の grade_id でも同じ数値を持つ可能性あり)

このように `order` は同一階層の中でのソート指標であり、階層を跨ぐと数値の比較は行わず「別のグループ」として扱います。
例えば「同じ `unit_id` で `order` が 100 → 200」と並んでいるとき、別の `unit_id` に `order` = 100 が存在していても、それらを直接比較することはありません。

---

## 3. パラメータ一覧（環境変数 / 定数で可変）
| 定数                   | 既定値 | 説明                                                          |
|----------------------|--------|-------------------------------------------------------------|
| `TH_PASS`            | **80** | 合格判定となる正答率 (%)                                              |
| `SUCCESS_STREAK`     | **3** | 連続合格回数。満たすと次の QS へ                                          |
| `FAIL_RATE`          | **50** | QS 正答率がこの値を下回ると 1 つ前へ戻る                                     |
| `DAILY_QS_TARGET`    | **1** | 1 日あたり新規学習する QuestionSet の目標値（既に users.daily_goal_type で定義） |
| `ENABLE_NEW_QS_PUSH` | `true` | 新規追加 QS を優先提示するか                                            |
| `GRADE_AUTO_DOWN`    | `true` | ロールバック動作を有効化するか                                             |

> **運用 tip**: `.env` で `TH_PASS` などを A/B テストしやすくしておく。

---

## 4. 新規 QuestionSet 追加時の扱い
- **grade / section が既習より低位の QS**: 既存ユーザーには *OPTIONAL* 提示。
- **同 grade 内への挿入**: `order` 順で自動的に学習対象へ。完了ユーザーについては `ENABLE_NEW_QS_PUSH` で制御。

---

## 5. Grade 遷移の記録
`user_grade_transitions` テーブル（新規）
| column | type | note |
|--------|------|------|
| `id` | PK | |
| `user_id` | FK → users | |
| `from_grade_id` | FK → grades | |
| `to_grade_id` | FK → grades | |
| `reason` | string | `PASS`, `FAIL_BACK`, etc. |
| `created_at` | datetime | |

> **分析用途**: 離脱率や適切レベル到達速度を可視化する。

---

## 6. 学習ボリューム見積もり
```
QuestionSet 総数 : 109
想定学習日数      : 260 / 年
1 QS あたり学習   : 3 回
⇒ 年間消化量       : 109 × 3 ≒ 327 回 (≒1 年強)
```

---

## 7. 今後の拡張メモ
- **忘却曲線 (SRS)**: 本仕様ではスコープ外。実装時は `next_review_at` と SM‑2 を導入。
- **IRT**: `NEXT_QS_DECISION()` 内部を置き換えるだけで導入可能。
- **Adaptive Difficulty**: `Question.difficulty_id` を用いて段階的調整が可能。

---

### END
