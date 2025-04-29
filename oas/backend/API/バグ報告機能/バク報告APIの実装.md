バグ報告APIを実装したい

# 仕様
報告API（store）のみで良い。
報告が合った場合に、メールと Slack で通知が来る

報告はDBに保存される

この変更のブランチ名とコミットメッセージ一行も教えて

レスポンスは 成功か失敗かのみで大丈夫。

\Report\BugReport としてね。

テストコードは生成不用

# 必要な情報
user_question_set_id（リレーション）BugReportType::QUESTION の時は必須。
question_set_id（リレーション）user_question_set　から取得可能なので　request には不用。BugReportType::QUESTION の時は　user_question_set_id から取得してDBに保存
question_sets json_id user_question_set　から取得可能なので　request には不用。BugReportType::QUESTION の時は　user_question_set_id から取得してDBに保存
user_question_id（リレーション）user_question_set　から取得可能なので　request には不用。BugReportType::QUESTION の時は　user_question_set_id から取得してDBに保存
question_id（リレーション）user_question_set　から取得可能なので　request には不用。BugReportType::QUESTION の時は　user_question_set_id から取得してDBに保存
question json_id　user_question_set　から取得可能なので　request には不用。BugReportType::QUESTION の時は　user_question_set_id から取得してDBに保存
user_id（リレーション）ログインユーザー　から取得可能なので　request には不用。BugReportType::QUESTION の時は　user_question_set_id から取得してDBに保存
bug_report_type (BugReportType に値があるか）必須
詳細情報 content なくても良い　文字列
日付情報
問題情報（question_data を JSON で。"question_data": {
"question_type": 201,
"question_text": "\u6b21\u306e \u25a2 \u306b\u300c\uff1e\u300d\u300c\uff1c\u300d\u300c\uff1d\u300d\u3092\u5165\u308c\u3066\u5927\u5c0f\u3092\u8868\u3057\u306a\u3055\u3044\u3002",
"explanation": "\u5927\u304d\u3044\u6570\u540c\u58eb\u3092\u6bd4\u3079\u308b\u3068\u304d\u306b\u306f\u3001\u4f4d\u306e\u6570\u3084\u6841\u6570\u306b\u6ce8\u610f\u3057\u307e\u3057\u3087\u3046\u3002\u6841\u6570\u304c\u591a\u3044\u65b9\u304c\u5927\u304d\u3044\u6570\u306b\u306a\u308a\u307e\u3059\u3002",
"question": "1234567 \u25a2 12345607",
"input_format": {
"type": "fixed",
"fields": [
{
"field_id": "f_1",
"attribute": "operator",
"user_answer": "string"
}
],
"input_components": [
{
"type": "text",
"content": {
"ja": "<",
"en": "<"
},
"order": 50
},
{
"type": "text",
"content": {
"ja": ">",
"en": ">"
},
"order": 100
},
{
"type": "text",
"content": {
"ja": "=",
"en": "="
},
"order": 150
}
],
"question_components": [
{
"type": "text",
"content": "1234567 ",
"order": 50
},
{
"type": "input_field",
"field_id": "f_1",
"order": 100
},
{
"type": "text",
"content": " 12345607",
"order": 150
}
]
}
},
"version": "1.0.0",
"question_type": "201"
}）bug の原因を把握したいので、フロントエンドからリクエストをもらう。BugReportType::QUESTION の時は必須。



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


=== 参考コード メール送信
public function request(array $data): PasswordResetDto
{
return DB::transaction(function () use ($data) {
$user = User::where('email', $data['email'])->first();
$dto = new PasswordResetDto();
if($user) {
$user->token_password = CommonLib::createGeneralToken(24);
$user->expires_at_token_password = date("Y-m-d H:i:s", strtotime("+1 hour"));
$user->save();

                \Mail::to($user->email)->cc(config()->get('mail.from.address'))->send(new ChangePassword(['token' => $user->token_password]));
            }
            DB::commit();
            $dto->result = true;

            return $dto;
        });
    }
<?php

namespace App\Mail\API\V1\Auth;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;

class ChangePassword extends Mailable
{
    use Queueable, SerializesModels;

    public $content;

    /**
     * Create a new message instance.
     *
     * @return void
     */
    public function __construct($content)
    {
        $this->content = (object)$content;
    }

    /**
     * Build the message.
     *
     * @return \App\Mail\Api\V1\User\Auth\ChangePassword
     */
    public function build()
    {
        // 件名
        $subject = nl2br(__('mail.changePass.subject'));
        // テンプレート
        $view = 'emails.api.v1.auth.change_password';

        /* ---------------------------------------------------------
         以下共通
        --------------------------------------------------------- */
        $subject = sprintf('【%s】', config()->get('app.name')) . $subject;
        if (config('app.env') != 'production') {
            $subject = sprintf('[%s]', config('app.env')) . $subject;
        }

        return $this
            ->from(config()->get('mail.from.address'), config()->get('mail.from.name'))
            ->subject($subject)
            ->view($view);
    }
}

ーー参考コードSlack通知
<?php

namespace App\Console\Commands\Question;

use App\Enums\QuestionSetLLMGenerationStatus;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Carbon;
use App\Models\Question\QuestionSet;
use App\Models\Question\QuestionGenerationLog;
use App\Services\Utils\Llm\LlmManageService;
use App\Services\Utils\Question\QuestionJsonManageService;
use Illuminate\Support\Str;

/**
 * Class AutoGenerateQuestionsCommand。
 *
 * 【仕様】
 *   - スケジュール: 5分に1回など (Kernel.phpで ->everyFiveMinutes() 等を登録)
 *   - 排他制御: Cache::add() によるシンプルロックを使用 (LOCK_TTL=300秒)
 *     => 同時実行を避ける
 *   - QuestionSet の "generation_blocked" フラグが false かつ "consecutive_generation_failures" < 5 のみ対象
 *   - 未回答問題数(「status=NOT_START or SKIP」のみカウント）< 10 の場合に新規問題を生成
 *   - LlmManageService::generateQuestion($questionSet) で問題JSONを作成し、
 *     QuestionJsonManageService::upsertQuestionByJson($problemJson) でDB登録
 *   - 生成ログは question_generation_logs に記録し、失敗時は consecutive_generation_failures++、5回以上なら generation_blocked=true
 *
 */
class AutoGenerateQuestionsCommand extends Command
{
    protected $signature   = 'question:auto-generate';
    protected $description = 'Check all question_sets and automatically generate new questions if needed.';

    private const MIN_QUESTIONS = 10;
    private const LOCK_KEY = 'auto-generate-questions-lock';
    private const LOCK_TTL = 300; // 5分

    private string $slackWebhookUrl;

    public function __construct()
    {
        parent::__construct();
        $this->slackWebhookUrl = config('services.slack.monitoring_webhook_url') ?? '';
    }

    public function handle()
    {
        // ローカル環境以外では、排他制御
        if (app()->environment() != 'local') {
            if (!Cache::add(self::LOCK_KEY, true, self::LOCK_TTL)) {
                $this->warn("Another 'AutoGenerateQuestionsCommand' is already running. Skipping...");
                return 0;
            }
        }

        try {
            Log::channel('generate_question')->info("[AutoGenerateQuestions] Start. Env=" . app()->environment());
            $this->info("Starting AutoGenerateQuestionsCommand...");

            // generation_blocked = false, consecutive_generation_failures < 5 の QuestionSet を取得
            $query = QuestionSet::where('generation_blocked', false)
                ->where('consecutive_generation_failures', '<', 5);

            $questionSets = $query->get();

            if ($questionSets->isEmpty()) {
                $this->info("No QuestionSets found that are eligible for generation.");
                Log::channel('generate_question')->info("[AutoGenerateQuestions] No QuestionSets found. End.");
                return 0;
            }

            $this->info("Found {$questionSets->count()} QuestionSets to check for question generation.");
            Log::channel('generate_question')->info("[AutoGenerateQuestions] Found {$questionSets->count()} sets.");

            foreach ($questionSets as $qSet) {
                $this->processOneQuestionSet($qSet);
            }

            $this->info("AutoGenerateQuestionsCommand completed.");
            Log::channel('generate_question')->info("[AutoGenerateQuestions] Completed.");

        } finally {
            Cache::forget(self::LOCK_KEY);
        }

        return 0;
    }

    /**
     * 指定の QuestionSet を対象に、未回答問題数をチェックして足りなければ作問
     */
    private function processOneQuestionSet(QuestionSet $qSet): void
    {
        Log::channel('generate_question')->info("Check QSet={$qSet->id} blocked={$qSet->generation_blocked}");

        if ($qSet->generation_blocked) {
            $this->warn("QuestionSet [{$qSet->id}] is blocked. Skipping generation.");
            Log::channel('generate_question')->info("Skipping QSetJsonId={$qSet->json_id} because blocked=true");
            return;
        }

        if ($qSet->consecutive_generation_failures >= 5) {
            $this->warn("QuestionSet [{$qSet->id}] already has 5 or more consecutive failures. Blocking generation.");
            $qSet->generation_blocked = true;
            $qSet->save();
            Log::channel('generate_question')->info("Set QSetJsonId={$qSet->json_id} => generation_blocked=true (>=5 fails).");
            return;
        }

        // ★ LLM generation statusがDISABLEDのときはスキップ+Slack通知
        if ($qSet->llm_generation_status == QuestionSetLLMGenerationStatus::DISABLED->value) {
            $this->warn("QuestionSet [{$qSet->id}] => LLM generation is disabled. Skipping generation.");
            Log::channel('generate_question')->info("Skipping QSetJsonId={$qSet->json_id} because llm_generation_status=DISABLED.");
            // Slackにも"SKIP"として通知 (errorMessage相当で「LLM disabled」)
//            $this->notifySlack([], false, "LLM generation is disabled for QSet={$qSet->id}, skipping.");
            return;
        }

        $hasAnyShortage = $this->checkIfAnyUserHasFewerThanMin($qSet->id);
        if (!$hasAnyShortage) {
            $this->info("QuestionSet [{$qSet->id}] has enough stock. Skipping generation.");
            Log::channel('generate_question')->info("No shortage QSet={$qSet->id}, skip generation");
            return;
        }

        try {
            Log::channel('generate_question')->info("Start generation for QSet={$qSet->id}");

            /** @var LlmManageService $llm */
            $llm = app(LlmManageService::class);

            // LLM作問
            $generationResult = $llm->generateQuestion($qSet);
            $problemJson      = $generationResult['problemJson'] ?? [];
            $llmPrompt        = $generationResult['prompt']       ?? '(missing prompt)';
            $llmRawResponse   = $generationResult['rawLlmResponse'] ?? '';

            // generated_by_llm = true
            $problemJson['generated_by_llm'] = true;

            /** @var QuestionJsonManageService $manager */
            $manager = app(QuestionJsonManageService::class);

            DB::beginTransaction();
            $isSuccess = $manager->upsertQuestionByJson($problemJson);

            // 生成ログ
            $log = new QuestionGenerationLog();
            $log->question_set_id      = $qSet->id;
            $log->attempted_at         = Carbon::now();
            $log->prompt               = $llmPrompt;
            $log->question_set_content = json_encode($qSet->toArray(), JSON_UNESCAPED_UNICODE);
            $log->llm_response         = $llmRawResponse;
            $log->is_success           = $isSuccess;

            if ($isSuccess) {
                // question_id を特定 (json_id => questions.id)
                $questionId = DB::table('questions')->where('json_id', $problemJson['id'])->value('id');

                if (!$questionId) {
                    $log->error_message = 'No question record found after upsert.';
                    $this->incrementFailures($qSet);
                    $log->save();
                    DB::rollBack();

                    $this->error("Failed to pivot question for QuestionSet [{$qSet->id}]: question not found.");
                    Log::channel('generate_question')->error("Pivot failed QSet={$qSet->id}, question not found.");
                    $this->notifySlack($problemJson, false, 'Question upsert was ok but question record not found.');
                    return;
                }

                // question_set_questions PIVOT登録
                $existingPivot = DB::table('question_set_questions')
                    ->where('question_set_id', $qSet->id)
                    ->where('question_id', $questionId)
                    ->first();
                if (!$existingPivot) {
                    $pivotMaxOrder = DB::table('question_set_questions')
                        ->where('question_set_id', $qSet->id)
                        ->max('order');
                    $newPivotOrder = ($pivotMaxOrder ?? 0) + 1;

                    DB::table('question_set_questions')->insert([
                        'id'               => (string) Str::uuid(),
                        'question_set_id'  => $qSet->id,
                        'question_id'      => $questionId,
                        'order'            => $newPivotOrder,
                        'created_at'       => Carbon::now(),
                        'updated_at'       => Carbon::now(),
                    ]);
                }

                $log->question_id = $questionId;
                // 連続失敗回数リセット
                $qSet->consecutive_generation_failures = 0;
                $qSet->save();
            } else {
                $log->error_message = 'Question upsert failed (invalid data or unknown error)';
                $this->incrementFailures($qSet);
            }

            $log->save();
            DB::commit();

            $this->info("Question generation for QuestionSet [{$qSet->id}] => success={$isSuccess}");
            Log::channel('generate_question')->info("Generated QSet={$qSet->id}, success={$isSuccess}");

            $this->notifySlack($problemJson, $isSuccess);

        } catch (\Throwable $e) {
            DB::rollBack();
            Log::channel('generate_question')->error("Exception QSet={$qSet->id} => {$e->getMessage()}");

            $log = new QuestionGenerationLog();
            $log->question_set_id      = $qSet->id;
            $log->attempted_at         = Carbon::now();
            $log->prompt               = $qSet->generate_question_prompt;
            $log->question_set_content = json_encode($qSet->toArray(), JSON_UNESCAPED_UNICODE);
            $log->is_success           = false;
            $log->error_message        = $e->getMessage();
            $log->save();

            $this->incrementFailures($qSet);
            $this->error("Failed to generate question for QuestionSet [{$qSet->id}]: " . $e->getMessage());
            $this->notifySlack([], false, $e->getMessage());
        }
    }

    /**
     * 環境ごとの最低問題数
     */
    private function getMinQuestions(): int
    {
        $env = app()->environment();
        if ($env === 'production') {
            return 5;
        }
        return 3;
    }

    /**
     * 指定の question_set_id に紐づく質問の「ストック数」が getMinQuestions() を下回るかどうかを判定する。
     */
    private function checkIfAnyUserHasFewerThanMin(string $questionSetId): bool
    {
        $questionCount = DB::table('question_set_questions')
            ->where('question_set_id', $questionSetId)
            ->count();

        if ($questionCount < $this->getMinQuestions()) {
            return true;
        }

        $answeredCount = DB::table('user_questions')
            ->join('question_set_questions','question_set_questions.question_id','=','user_questions.question_id')
            ->where('question_set_questions.question_set_id', $questionSetId)
            ->whereNotIn('user_questions.status',[1,150])
            ->distinct()
            ->count('user_questions.question_id');

        $stock = $questionCount - $answeredCount;

        return $stock < $this->getMinQuestions();
    }

    private function incrementFailures(QuestionSet $qs): void
    {
        $qs->consecutive_generation_failures++;
        if ($qs->consecutive_generation_failures >= 5) {
            $qs->generation_blocked = true;
        }
        $qs->save();
    }

    private function notifySlack(array $problemJson, bool $isSuccess, ?string $errorMessage = null): void
    {
        if (!$this->slackWebhookUrl) {
            Log::channel('generate_question')->warning("No Slack webhook URL configured for question generation logs.");
            return;
        }

        $env = app()->environment();
        $statusStr = $isSuccess ? 'SUCCESS' : 'FAILURE';

        $text = "Env: `{$env}`\n";
        $text .= "Generation: **{$statusStr}**\n";
        if ($isSuccess) {
            // JSON は Google Drive にアップロードされるので不用
//            $jsonEncoded = json_encode($problemJson, JSON_UNESCAPED_UNICODE|JSON_PRETTY_PRINT);
//            $text .= "Generated Problem JSON:\n```\n{$jsonEncoded}\n```\n";
        } else {
            $text .= "Error: `{$errorMessage}`\n";
        }

        $payload = [
            'text' => $text,
        ];

        try {
            $response = Http::post($this->slackWebhookUrl, $payload);
            if ($response->failed()) {
                Log::channel('generate_question')->error("Slack notify failed: status={$response->status()} body=".$response->body());
            } else {
                Log::channel('generate_question')->info("Slack notify success.");
            }
        } catch (\Throwable $th) {
            Log::channel('generate_question')->error("Slack notify exception: ".$th->getMessage());
        }
    }
}

＝＝
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
            $table->uuid('grade_id');
            $table->uuid('difficulty_id');
            $table->string('json_id')->nullable()->unique();
            $table->json('metadata')->nullable();
            $table->string('version')->default('0.0.1');
            $table->integer('status')->default(1);
            $table->integer('evaluation_method')->default(1);
            $table->integer('checker_method')->nullable();
            $table->integer('llm_evaluation_prompt_file_name')->nullable();
            $table->json('evaluation_response_format')->nullable();
            $table->integer('question_type')->default(1);
            $table->json('learning_requirement_json')->nullable();
            $table->string('learning_subject')->nullable()
                ->comment('科目 (学習要件)');
            $table->integer('learning_no')->nullable()
                ->comment('学習要件の番号');
            $table->text('learning_requirement')->nullable()
                ->comment('学習要件の内容');
            $table->text('learning_required_competency')->nullable()
                ->comment('必要水準');
            $table->text('learning_background')->nullable()
                ->comment('背景・補足');
            $table->string('learning_category')->nullable()
                ->comment('分類');
            $table->string('learning_grade_level')->nullable()
                ->comment('学年');
            $table->string('learning_url')->nullable()
                ->comment('URLリンク');
            $table->integer('order');
            $table->boolean('generated_by_llm')->default(false);
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('level_id')->references('id')->on('levels')->onDelete('cascade');
            $table->foreign('grade_id')->references('id')->on('grades')->onDelete('cascade');
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
        Schema::create('user_question_sets', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('user_id');
            $table->uuid('question_set_id');
            $table->integer('status')->default(1);
            $table->decimal('score', 8, 2)->nullable();
            $table->timestamp('started_at')->nullable();
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

            $table->json('metadata')->nullable();
            $table->string('version')->default('0.0.1');
            $table->integer('evaluation_method')->default(1);
            $table->integer('checker_method')->nullable();
            $table->integer('llm_evaluation_prompt_file_name')->nullable();
            $table->json('evaluation_response_format')->nullable();
            $table->integer('question_type')->default(1);
            $table->json('learning_requirement_json')->nullable();
            $table->string('learning_subject')->nullable()
                ->comment('科目 (学習要件)');
            $table->integer('learning_no')->nullable()
                ->comment('学習要件の番号');
            $table->text('learning_requirement')->nullable()
                ->comment('学習要件の内容');
            $table->text('learning_required_competency')->nullable()
                ->comment('必要水準');
            $table->text('learning_background')->nullable()
                ->comment('背景・補足');
            $table->string('learning_category')->nullable()
                ->comment('分類');
            $table->string('learning_grade_level')->nullable()
                ->comment('学年');
            $table->string('learning_url')->nullable()
                ->comment('URLリンク');
            $table->boolean('generated_by_llm')->default(false);

            $table->integer('status')->default(1);
            $table->json('answer_data')->nullable();
            $table->longText('prompt_for_evaluation')->nullable();
            $table->timestamp('answered_at')->nullable();
            $table->integer('order');
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

<?php

namespace App\Enums;

enum BugReportType: int
{
    case QUESTION = 10;

    public function label(): string
    {
        return match($this) {
            self::QUESTION          => 'QUESTION'
        };
    }

    /**
     * 文字列から BugReportType を取得
     * @param string $typeString 例: "DRAFT", "PUBLISHED" など
     * @return self
     */
    public static function fromString(string $typeString): self
    {
        return match (strtoupper($typeString)) {
            'QUESTION'          => self::QUESTION,
            default => throw new \InvalidArgumentException("Unknown type string: {$typeString}")
        };
    }
}
