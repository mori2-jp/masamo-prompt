※一度、現在の回答APIの実装のプロンプトを読み込ませる
「これからの回答APIの更新を依頼します。まず回答APIの実装時に使ったプロンプトを読み込んでください」

Userが問題の回答を正誤するバックエンドの設計

/answers というAPIを更新したい。

・Enumで評価手法を分岐
・Enumでどのメソッドを使って評価するか分岐

# 説明
question_sets：問題（questions）を束ねるグループ
questions: 問題
question_set_questions：question_sets と questions を紐づけるPivotテーブル。questions は複数のquestion_sets に紐づく事があり多対多の関係なのでこのような設計になっている。
user_question_sets: ユーザーが学習した questions_sets。学習開始時に question_sets_id と紐づいて status （UserQuestionSetsStatus::NOT_START) が未開始の状態で生成され、進捗ステータスはスコアなどが管理される
user_questions: ユーザーが学習した questions。学習開始時に、学習を開始した question_sets に紐づく questions が全て、 user_question_sets_id と question_id（questions）を紐づけて status （UserQuestionStatus::NOT_START) が未開始の状態で全てのquestionsの数の分、生成され、進捗ステータスはスコアなどが管理される。スコアを保持


# フロー
--開始ここから
１．
ユーザーの回答（JSON）を受け取ってフォーマットが正しいかバリデーションする
2.
evaluationMethodがLLMなら、APIを使ってLLMに正誤判定を依頼。evaluationMethodがCodeなら指定されたメソッドで正誤判定を行う
３．
user_question_sets に次の問題（紐づいている user_questions の中に status がNOT_STARTのものがあれば　order 昇順で一番先頭のやつ）があればそれをレスポンスする。無ければ user_question_sets の status をCOMPLETEにする



Dto に fromModelを実装するようなことはせずに、Dtoの生成は必ずService内部で実行するようにしてください


ーーユーザーからリクエストされる回答のJSON形式


ーー現在の実装
Route::middleware(['auth:sanctum'])->group(function () {
Route::prefix('question-sets')->group(function () {
Route::get('/', [QuestionSetController::class, 'index']);
});
        Route::prefix('answer')->group(function () {
            Route::post('/', [AnswerController::class, 'store'])->name('v1.answer.store');
        });
    });
<?php

namespace App\Http\Controllers\API\V1\Answer;

use App\Http\Controllers\Controller;
use App\Http\Requests\API\V1\Answer\StoreAnswerRequest;
use App\Http\Resources\V1\Answer\AnswerCheckResource;
use App\UseCases\V1\Answer\AnswerCheckUseCase;
use Illuminate\Http\Request;

class AnswerController extends Controller
{
    protected AnswerCheckUseCase $answerCheckUseCase;

    public function __construct(
        AnswerCheckUseCase $answerCheckUseCase
    ) {
        $this->answerCheckUseCase = $answerCheckUseCase;
    }

    /**
     * /v1/answer (POST)
     *
     * ユーザーからのJSONリクエストを受け取り、
     * OpenAI API に問い合わせて結果を返す
     */
    public function store(Request $request)
    {
        $answerData = json_decode($request->get('answer_data'), true);
//        $answerData = $validatedData['answer_data'];

        // ユースケースを実行 → Service経由でOpenAIに問い合わせ
        $answerCheckDto = $this->answerCheckUseCase->handle($answerData);

        // Resourceを利用してJSONレスポンスとして返す
        return new AnswerCheckResource($answerCheckDto);
    }
}
<?php

namespace App\UseCases\V1\Answer;

use App\Services\V1\Answer\AnswerCheckService;
use App\Dtos\V1\Answer\AnswerCheckDto;

class AnswerCheckUseCase
{
    public function __construct(
        protected AnswerCheckService $answerCheckService
    ) {
    }

    /**
     * リクエストされた JSON データを使い、
     * OpenAI への問い合わせ結果を取得して DTO にまとめる
     */
    public function handle(array $data): AnswerCheckDto
    {
        return $this->answerCheckService->checkAnswer($data);
    }
}

<?php

namespace App\Services\V1\Answer;

use App\Dtos\V1\Answer\AnswerCheckDto;
use Illuminate\Support\Facades\Http;

class AnswerCheckService
{
    /**
     * OpenAI API へリクエストを送り、レスポンスを取得する
     */
    public function checkAnswer(array $data): AnswerCheckDto
    {
        // ユーザーのリクエストデータ全体をJSON文字列に変換してプロンプトへ埋め込む
        $requestJson = json_encode($data, JSON_UNESCAPED_UNICODE | JSON_PRETTY_PRINT);

        $prompt = <<<EOT
{#条件}に従って以下の{#RequestJSON}のuser_answer が question に正答しているか確認してください。

# 条件
・回答は、{#回答フォーマット}と同一としプレーンなJSONで返却すること。
レスポンスはPHPで json_decode() するので、正しく動作するようにJSONのみをアウトプットしてほしいということです。
「```json」等不用です。バグとなるので絶対に付与しないでください。
・回答のアウトプットは、JSONのみとすること。
・この問題の対象者は{学習要件}を参考にしてください。解説は対象者に分かりやすい説明であること
例えば、learning_grade_levelが小２であれば小学生２年生にも分かりやすい解説や言葉遣いを意識してください

# 説明
fields は、ブランクの箇所を一番最初から順番に表しています
user_answer: ユーザー（学習者）の回答
collect_answer: 正しい回答。LLMが入力する
is_collect: ユーザーの回答が正答かどうが。正答なら true, 誤答なら false。LLMが入力する
field_explanation：フィールドごとに、なぜその回答なのか解説する。LLMが入力する
explanation: この問題の解説。LLMが入力する

# RequestJSON
{$requestJson}


# サンプル回答フォーマット
{
  "question_text": {
    "ja": "▢にあてはまる数を答えなさい。",
    "en": "Please answer the numbers that fit in the blanks."
  },
  "explanation": {
    "ja": "",
    "en": ""
  },
  "question": "8 × 4 = ▢ × 8 = ▢",
  "fields": [
    {
      "field_id": "f_1",
      "user_answer": "2",
      "is_collect": "",
      "collect_answer": "",
      "field_explanation": {
        "ja": "",
        "en": ""
      }
    },
    {
      "field_id": "f_2",
      "user_answer": "32",
      "is_collect": "",
      "collect_answer": "",
      "field_explanation": {
        "ja": "",
        "en": ""
      }
    }
  ],
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
  ]
}
EOT;
        $apiToken = config('services.openai.api_key');
        // OpenAI API呼び出し (Chat Completion の例)
        $response = Http::withToken(config('services.openai.api_key'))
            ->post('https://api.openai.com/v1/chat/completions', [
                'model' => 'gpt-4o',   // 必要に応じてモデルを変更
                'messages' => [
                    [
                        'role' => 'user',
                        'content' => $prompt
                    ]
                ],
                'temperature' => 0.0,
            ]);

        // OpenAIからのレスポンスを配列として取得
        $responseBody = $response->json();

        // $responseBody['choices'][0]['message']['content'] にチャットモデルの回答が入っている想定

        // 1) レスポンス本文のJSON部分を取り出し、JSONとしてパース
        $rawContent = data_get($responseBody, 'choices.0.message.content');

        // LLMが返してくる文字列をJSONとしてパース (配列化)
        $parsedContent = json_decode($rawContent, true);

        // 2) 必要に応じてユーザーの problem_id を合体
        //    → もしLLMの応答内に problem_id がなければ、ユーザー送信データのものを上書きする例
        if (isset($data['problem_id'])) {
            $parsedContent['problem_id'] = $data['problem_id'];
        }

        // 3) これで最終的にレスポンスしたい配列を組み立てたので、
        //    AnswerCheckDtoに格納して返す
        return new AnswerCheckDto(content: $parsedContent);
    }
}
<?php

namespace App\Dtos\V1\Answer;

use App\Dtos\Dto;

/**
 * OpenAIからのレスポンスを保持するDTO
 */
class AnswerCheckDto extends Dto
{
    /**
     * @var array OpenAIからのレスポンス JSONをそのまま格納する例
     */
    public array $content;
}

<?php

namespace App\Http\Resources\V1\Answer;

use Illuminate\Http\Resources\Json\JsonResource;

class AnswerCheckResource extends JsonResource
{
    /**
     * Transform the resource into an array.
     * $this->resource は AnswerCheckDto のインスタンス
     */
    public function toArray($request)
    {
        // DTOのcontent配列を取得
        $content = $this->resource->content;

        // 下記でユーザーが欲しいと指定したキーだけ返す
        return [
            'question_text' => data_get($content, 'question_text', []),
            'explanation'   => data_get($content, 'explanation', []),
            'question'      => data_get($content, 'question', ''),
            'question_id'    => data_get($content, 'question_id', ''),
            'fields'        => data_get($content, 'fields', []),
        ];
    }
}

<?php

namespace App\Enums;

// 回答の正誤を評価する方法をマップしたEnum
enum EvaluationMethod: int
{
    case CODE = 1;
    case LLM = 2;
}
<?php

namespace App\Enums;

// 問題の正誤を評価するメソッドをマップしたEnum
enum EvaluationCheckerMethod: int
{
    case CALCULATE_METHOD = 1;

    public function label(): string
    {
        return match($this) {
            self::CALCULATE_METHOD          => 'calculateMethod',
        };
    }

    /**
     * 文字列から QuestionStatus を取得
     * @param string $statusString 例: "DRAFT", "PUBLISHED" など
     * @return self
     */
    public static function fromString(string $statusString): self
    {
        return match (strtoupper($statusString)) {
            'calculateMethod'          => self::CALCULATE_METHOD,
            default => throw new \InvalidArgumentException("Unknown status string: {$statusString}")
        };
    }
}

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

