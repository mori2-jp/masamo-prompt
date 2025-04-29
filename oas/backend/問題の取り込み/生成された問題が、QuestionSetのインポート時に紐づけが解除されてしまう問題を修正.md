生成された問題が、QuestionSetのインポート時に紐づけが解除されてしまう問題を修正したい

以下の、ImportQuestionSetsFromGithub は、問題セットのJSON をパースして question_sets にデータをインサートします。
この時に、question_set_questions に question_sets と　questions を紐づけます。

紐づけの関係は、問題セットのJSON　の questions で定義されていますが、この定義は予め自前で用意した questions のみにとどまります。

このアプリケーションでは、自前で用意した questions と、LLMに生成してらもった　questions （AutoGenerateQuestionsCommand）の２種類が存在していますが、
現状の、question_sets のインポートコマンドは、予め用意した questions のみにしか対応していないので、
新たに question_sets を追加してインポートコマンドを叩くと LLM によって生成された questions との紐づけが解除されてしまいます。

この問題を修正してもらうのが今回のオーダーです。

具体的には、 LLMによって生成された questions は question_generation_logs で管理しているので、
それを元に、問題セットJSONで定義されたquestions の紐づけが終わった後にLLMによって生成された questions の紐づけを行ってほしい。


# 前提条件

# 条件
１）Github or Google Drive から、問題セットのJSONのインポートをして questions の紐づけを行う
STG環境
（既存の実装をそのまま維持）

２）question_generation_logs から生成された questions を取得して紐づけを行う。
 question_set_id を元に、question_sets を探索。無ければスキップ。
 question_id を元に、questions を探索。無ければスキップ
どちらもあれば、question_set_questions に紐づけ。すでに紐づけがあり重複ならスキップ。
order は、question_set_id がグループのキーとしたいので、question_set_id が重複するデータの中で一番大きいorderの値に+1した値とする。


3)LLMで生成された問題（generated_by_llm = true）の紐づけが、再インポート時に解除されないように変更する
具体的には、「自前で用意された問題」だけをいったん差し替える（消去→再登録）処理にし、LLM 生成問題（generated_by_llm=true）は削除対象から除外する

# 実装方針
生成また修正変更するコードはそのコードだけは必ず省略せずに必ず全てアウトプットすること。

修正する場合は修正したコードは全て出力し、修正箇所以外のコードやコメントは現状のままにしてください。

仕様はソースコードにまとめてコメントとして残すこと
コメントを残す時に以下のようにコメントへ処理順を示すようなナンバリングは不用です。
// 7) memo => 空白
// 8) generate_question_prompt => そのまま
// 9) generate_question_prompt_file_name => そのまま

ーーー 問題セットのJSON
```json
{
  "json_id": "qset_s1_g3_sec100_u300_v100_100",
  "order": 100,
  "unit_id": "unit_s1_g3_sec100_300",
  "unit": {
    "ja": "3位数や4位数の加法及び減法",
    "en": "Addition and Subtraction of Three- and Four-Digit Numbers"
  },
  "title": {
    "ja": "3桁+3桁の足し算",
    "en": "3-Digit Plus 3-Digit Addition"
  },
  "description": {
    "ja": "このドリルでは、3桁+3桁の整数を正しく筆算できるようになることを目指します。繰り上がりに注意しながら位をそろえて計算するコツを身につけ、素早く正確な計算力を養いましょう。",
    "en": "In this drill, we focus on accurately performing addition of two three-digit numbers. Learn how to handle carrying properly while aligning each place value, and build speed and accuracy in your calculations."
  },
  "background": {
    "ja": "学習指導要領No.37（3位数や4位数の加法及び減法）を踏まえ、3桁の整数同士を正確に加算する力を身につけることを目的としています。問題では、くり上がりが一度だけ起きるパターンから二度起きるパターンまで、段階的に難易度を上げています。例えば、繰り上がりが2回必要になる問題や、十の位と一の位でそれぞれ繰り上がりが発生するケースなどを用意しました。これにより、単に答えを求めるだけでなく、筆算の手順や位の考え方、そして繰り上がり処理を体系的に学習できます。",
    "en": "Based on Guideline No. 37 (Addition and subtraction of three- and four-digit numbers), the goal is to develop precise addition skills for two three-digit numbers. This set includes problems ranging from those requiring a single carry to those with multiple carries, gradually increasing in difficulty. For instance, there are cases where carrying occurs twice or in multiple digit places, helping you not just find answers but also systematically learn written methods, place value concepts, and carrying procedures."
  },
  "generate_question_prompt": {
    "ja": "3桁+3桁の整数同士の足し算を、繰り上がりの有無ともにバリエーション豊富に1問作成してください。位を正しくそろえることや、くり上がりの処理に焦点を当てた問題にしてください。問題文は小学生が取り組みやすい形式で、式の形や穴埋めの形など、やさしい日本語でまとめてください。",
    "en": "Please create a single addition problem involving two three-digit numbers, including variations of carrying or no carrying. Emphasize aligning digits correctly and handling carries. Present the problem in a clear format suitable for elementary school learners, using either a direct equation or a fill-in-the-blank style."
  },
  "generate_question_prompt_file_name": "fill_in_the_blank",
  "llm_generation_status": "ENABLED",
  "memo": "3桁の足し算",
  "version": "1.0.0",
  "status": "PUBLISHED",
  "questions": [
    "ques_s1_g3_sec100_u300_diff100_qt51_v100_100",
    "ques_s1_g3_sec100_u300_diff100_qt51_v100_200"
  ]
}

```

ーーーー
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
        Schema::create('question_generation_logs', function (Blueprint $table) {
            $table->id();
            $table->uuid('question_set_id');
            $table->uuid('question_id')->nullable()->comment('成功時のみ、生成された question を紐づけ');
            $table->timestamp('attempted_at')->nullable()->comment('LLM生成を試みた日時');
            $table->longText('prompt')->nullable()->comment('LLMに投げたプロンプト');

            /**
             * question_set_content: 生成時点のQuestionSetのスナップショットをJSON形式で保存
             * 例)
             * {
             *   "version": "1.2.3",
             *   "title": {
             *     "ja": "日本語タイトル",
             *     "en": "English Title"
             *     // 他の言語分も必要に応じて
             *   },
             *   "description": {
             *     "ja": "日本語の説明文",
             *     "en": "English description"
             *   },
             *   "background": {
             *     "ja": "背景情報(日本語)",
             *     "en": "Background info in English"
             *   }
             * }
             */
            $table->json('question_set_content')->comment('生成時点のQuestionSet情報 (version,多言語のtitle/description/background等)をJSONで');

            $table->boolean('is_success')->default(false)->comment('生成に成功したかどうか');

            $table->longText('error_message')->nullable()->comment('失敗時のエラー内容や例外メッセージ等');
            $table->longText('llm_response')->nullable()->comment('LLM応答全文 (JSON等)');

            $table->boolean('google_drive_uploaded')
                ->default(false)
                ->comment('Google Driveへアップロード済みかどうか');

            $table->timestamps();
            $table->softDeletes();

            // 外部キー設定
            $table->foreign('question_set_id')
                ->references('id')
                ->on('question_sets')
                ->onDelete('cascade');

            $table->foreign('question_id')
                ->references('id')
                ->on('questions')
                ->onDelete('set null');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('question_generation_logs');
    }
};


<?php

namespace App\Models\Question;

use App\Models\BaseModel;
use Illuminate\Database\Eloquent\SoftDeletes;

/**
 *
 *
 * @property int $id
 * @property string $question_set_id
 * @property string|null $question_id 成功時のみ、生成された question を紐づけ
 * @property string|null $attempted_at LLM生成を試みた日時
 * @property string|null $prompt LLMに投げたプロンプト
 * @property string $question_set_content 生成時点のQuestionSet情報 (version,多言語のtitle/description/background等)をJSONで
 * @property int $is_success 生成に成功したかどうか
 * @property string|null $error_message 失敗時のエラー内容や例外メッセージ等
 * @property string|null $llm_response LLM応答全文 (JSON等)
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property \Illuminate\Support\Carbon|null $deleted_at
 * @property-read \App\Models\Question\Question|null $question
 * @property-read \App\Models\Question\QuestionSet $questionSet
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionGenerationLog newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionGenerationLog newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionGenerationLog onlyTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionGenerationLog query()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionGenerationLog whereAttemptedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionGenerationLog whereCreatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionGenerationLog whereDeletedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionGenerationLog whereErrorMessage($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionGenerationLog whereId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionGenerationLog whereIsSuccess($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionGenerationLog whereLlmResponse($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionGenerationLog wherePrompt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionGenerationLog whereQuestionId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionGenerationLog whereQuestionSetContent($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionGenerationLog whereQuestionSetId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionGenerationLog whereUpdatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionGenerationLog withTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionGenerationLog withoutTrashed()
 * @mixin \Eloquent
 */
class QuestionGenerationLog extends BaseModel
{
    use SoftDeletes;


    /*=====================================================
    * Relation
    *=====================================================*/
    public function questionSet()
    {
        return $this->belongsTo(QuestionSet::class, 'question_set_id');
    }

    public function question()
    {
        return $this->belongsTo(Question::class, 'question_id');
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

namespace App\Console\Commands\Question;

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
 * 【変更点】
 *   - checkIfAnyUserHasFewerThanMin(string $questionSetId) のロジックを差し替え
 *     => 「特定の question_set_id に紐づく questions の総数」と
 *        「すでに回答済み(status != NOT_START,SKIP)の questions の数」を用いて
 *        ストック数(=総数 - 回答済み) が閾値未満かを判定する。
 *     => 質問ストックが閾値を下回ったら問題生成を行うように変更。
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
            Log::channel('generate_question')->info("Skipping QSet={$qSet->id} because blocked=true");
            return;
        }

        if ($qSet->consecutive_generation_failures >= 5) {
            $this->warn("QuestionSet [{$qSet->id}] already has 5 or more consecutive failures. Blocking generation.");
            $qSet->generation_blocked = true;
            $qSet->save();
            Log::channel('generate_question')->info("Set QSet={$qSet->id} => generation_blocked=true (>=5 fails).");
            return;
        }

        // ★ 修正後のロジックを用いた「ストック不足チェック」
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
     *
     *   1) question_set_questionsテーブル から question_set_id が $questionSetId のレコードを総数カウント => $questionCount
     *   2) すでに回答された question を数える
     *      user_questions.status が NOT_START(1), SKIP(150) 以外 =>「回答済み」とみなす
     *   3) $questionCount - 回答済み数 = 残りストック
     *   4) この残りストック < getMinQuestions() なら true, そうでなければ false
     *
     * @param string $questionSetId
     * @return bool true=ストック不足(生成が必要)
     */
    private function checkIfAnyUserHasFewerThanMin(string $questionSetId): bool
    {
        // 全体問題数
        $questionCount = DB::table('question_set_questions')
            ->where('question_set_id', $questionSetId)
            ->count();

        // 総数が閾値より少なければ足りない
        if ($questionCount < $this->getMinQuestions()) {
            return true;
        }

        // 回答済み数(=status が NOT_START(1), SKIP(150) 以外) をカウント
        // distinct() で同じ user_questions.question_id を重複カウントしない
        $answeredCount = DB::table('user_questions')
            ->join('question_set_questions','question_set_questions.question_id','=','user_questions.question_id')
            ->where('question_set_questions.question_set_id', $questionSetId)
            ->whereNotIn('user_questions.status',[1,150])
            ->distinct()
            ->count('user_questions.question_id');

        // 未回答ストック = 総数 - 回答済み数
        $stock = $questionCount - $answeredCount;

        // ストックが閾値未満か
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


<?php

namespace App\Console\Commands\Import\Github;

use App\Enums\QuestionSetStatus;
use App\Models\Question\Question;
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
 * 6) question_set_questions ピボットの再同期・タイトル/説明の多言語 upsert など既存処理は変更しない。
 * 7) Drive API を使用するには GCP プロジェクトで Drive API を有効化し、
 *    サービスアカウントが該当ファイルへアクセス可能(共有設定)になっている必要がある。
 */

class ImportQuestionSetsFromGithub extends Command
{
    /**
     * artisan で呼び出すコマンド名
     *
     * 例: php artisan import:question-sets-from-github --subject=math
     */
    protected $signature = 'import:question-sets-from-github
                            {--subject= : If specified, only import files under contents/question_sets/{subject} }';

    protected $description = 'Import question set data from GitHub or Google Drive JSON files and sync DB';

    /**
     * Google Drive 上の "ファイルID" 一覧 (サービスアカウント認証で取得する)
     * https://drive.google.com/drive/folders/1TH_bacpFPJe4EwobfSSapWJG6wVvWU-c
     *
     * 例: https://drive.google.com/file/d/1VyLJr31zhHT3Gt7-pkg12cQitTJKHzF2/view?usp=drive_link
     *     ↑URL中の "1VyLJr31zhHT3Gt7-pkg12cQitTJKHzF2" がファイルID
     */
    protected array $jsonFileOnGoogleDriveIds = [
        '19o2-da7x8NjUSAqDt7DOsiw82DdzLnq_',
        '1ZYsFbzslqrMuq7OIElJNMp2SkSdu0JKF',
    ];

    /**
     * 「取り込もうとしたJSONの総数」
     */
    private int $totalCount = 0;

    /**
     * 「取り込みに失敗またはスキップしたJSONの総数」
     */
    private int $skipCount = 0;

    /**
     * 「取り込みに成功したJSONの総数」
     */
    private int $successCount = 0;

    /**
     * メインのハンドラ
     */
    public function handle()
    {
        $env = app()->environment();
        $useGoogleDrive = false;

        if ($env === 'staging') {
            // staging環境は必ずGoogle Drive
            $useGoogleDrive = true;
        } elseif ($env === 'local') {
            // local環境は config() で切り替え (例: app.use_google_drive_in_local_for_learning_data_import)
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

        // 統計出力
        $this->info("Import process completed.");
        $this->line("=======================================");
        $this->info("取り込もうとした Question Set JSON の総数: {$this->totalCount}");
        $this->info("取り込みに失敗またはスキップした Question Set JSON の総数: {$this->skipCount}");
        $this->info("取り込みに成功した Question Set JSON の総数: {$this->successCount}");
        $this->line("=======================================");

        return 0;
    }

    /**
     * 【B案】Google Drive のファイルを
     * Service Account (service-account-credentials.json) で認証して Drive API 経由でダウンロードする。
     */
    private function importFromGoogleDriveWithServiceAccount(): void
    {
        $this->info('Setting up Google Client for Drive API (QuestionSets) ...');

        // 1) Googleクライアントの認証
        $client = new Google_Client();
        // この認証ファイルパスは適宜修正 (ImportSkillsDataFromSpreadSheet と同様に)
        $client->setAuthConfig(storage_path('app/private/service-account-credentials.json'));
        $client->addScope(Google_Service_Drive::DRIVE_READONLY);

        // 2) Drive サービスインスタンス
        $driveService = new Google_Service_Drive($client);

        // 3) fileId 配列をループし、JSON を読み込む
        foreach ($this->jsonFileOnGoogleDriveIds as $fileId) {
            $this->info("Fetching JSON from Google Drive fileId: {$fileId}");

            // 読み取り対象カウント
            $this->totalCount++;

            try {
                // Drive API経由で alt=media を指定してバイナリストリーム取得
                $response = $driveService->files->get($fileId, ['alt' => 'media']);
                $jsonContent = $response->getBody()->getContents();

                $decoded = json_decode($jsonContent, true);
                if (!is_array($decoded)) {
                    $this->warn("File ID={$fileId} is not valid JSON. Skipped.");
                    $this->skipCount++;
                    continue;
                }

                if ($this->upsertQuestionSet($decoded)) {
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
     * productionやlocal(Drive未使用)等で使用: GitHubリポジトリから取り込む
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

        $subjectOption = $this->option('subject');
        if ($subjectOption) {
            $basePath .= '/' . $subjectOption;
            $this->info("Subject specified: {$subjectOption}, path= {$basePath}");
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
     * GitHub上のフォルダを再帰的にたどり、.jsonファイルを収集
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
            $name     = $item['name'] ?? '';

            if ($type === 'dir') {
                $descendantFiles = $this->fetchAllJsonFilesRecursively($repo, $itemPath, $token);
                $result = array_merge($result, $descendantFiles);
            } elseif ($type === 'file') {
                // .jsonファイルかどうか
                if (str_ends_with($name, '.json')) {
                    $result[] = $item;
                }
            }
        }

        return $result;
    }

    /**
     * 単一ファイルをダウンロード＆解析 (GitHub用)
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

        $jsonContent = Http::withToken($token)
            ->get($file['download_url'])
            ->body();

        $decoded = json_decode($jsonContent, true);
        if (!is_array($decoded)) {
            $this->warn("File {$file['name']} is not valid JSON. Skipped.");
            $this->skipCount++;
            return;
        }

        if ($this->upsertQuestionSet($decoded)) {
            $this->successCount++;
        } else {
            $this->skipCount++;
        }
    }

    /**
     * question_set JSON を DBにupsert
     */
    private function upsertQuestionSet(array $setJson): bool
    {
        DB::beginTransaction();
        try {
            $jsonId = $setJson['json_id'] ?? null;
            if (!$jsonId) {
                $this->warn("question_set JSON missing 'json_id'. Skipped.");
                DB::rollBack();
                return false;
            }

            // バリデーション
            $validator = app(QuestionSetJsonManageService::class);
            $errors = $validator->validateQuestionSetJson($setJson);
            if (!empty($errors)) {
                $this->warn("バリデーションエラーでスキップ: question_set(json_id={$jsonId}).");
                foreach ($errors as $field => $msgs) {
                    foreach ($msgs as $msg) {
                        $this->warn("  - {$field}: {$msg}");
                    }
                }
                DB::rollBack();
                return false;
            }

            // DB検索
            $qset = QuestionSet::where('json_id', $jsonId)->first();

            $jsonOrder = $setJson['order'] ?? 9999;
            $version   = $setJson['version'] ?? '0.0.1';
            $rawStatus = $setJson['status'] ?? 'DRAFT';
            $statusVal = $this->parseQuestionSetStatus($rawStatus);

            // unit_id
            if (empty($setJson['unit_id'])) {
                $this->warn("No unit_id provided => skip question_set(json_id={$jsonId})");
                DB::rollBack();
                return false;
            }
            $unitUuid = $this->findUnitUuid($setJson['unit_id']);
            if (!$unitUuid) {
                $this->warn("Unit not found for json_id={$setJson['unit_id']} => skip question_set(json_id={$jsonId})");
                DB::rollBack();
                return false;
            }

            // 新規 or 既存
            if (!$qset) {
                $qset = new QuestionSet();
                $qset->id      = (string) Str::uuid();
                $qset->json_id = $jsonId;
            }

            $qset->order = $setJson['order'];
            $qset->unit_id = $unitUuid;
            $qset->version = $version;
            $qset->status  = $statusVal;

            $qset->generate_question_prompt_file_name = $setJson['generate_question_prompt_file_name'] ?? null;
            if (isset($setJson['llm_generation_status'])) {
                $qset->llm_generation_status = $this->parseLLMGenerationStatus($setJson['llm_generation_status']);
            }

            if (isset($setJson['generate_question_prompt']) && is_array($setJson['generate_question_prompt'])) {
                $qset->generate_question_prompt = json_encode($setJson['generate_question_prompt'], JSON_UNESCAPED_UNICODE);
            }

            $qset->save();

            // question_set_translations
            $this->upsertSetTranslations($qset, $setJson);

            // questions 配列
            if (isset($setJson['questions']) && is_array($setJson['questions'])) {
                $this->syncQuestionSetQuestions($qset, $setJson['questions']);
            }

            DB::commit();
            $this->info("Upserted question_set json_id={$jsonId}");
            return true;

        } catch (\Throwable $e) {
            DB::rollBack();
            $this->error("Error upserting question_set json_id={$jsonId}: " . $e->getMessage());
            return false;
        }
    }


    /**
     * LLM Generation Status の文字列 → Enum変換
     */
    private function parseLLMGenerationStatus(string $rawStatus): int
    {
        try {
            return \App\Enums\QuestionSetLLMGenerationStatus::fromString($rawStatus)->value;
        } catch (\InvalidArgumentException) {
            return \App\Enums\QuestionSetLLMGenerationStatus::DISABLED->value;
        }
    }

    private function reorderQuestionSet(string $setId, int $targetOrder): void
    {
        $qset = QuestionSet::find($setId);
        if (!$qset) return;

        while (QuestionSet::where('order', $targetOrder)
            ->where('id','!=',$setId)
            ->exists()) {
            $targetOrder++;
        }
        $qset->order = $targetOrder;
        $qset->save();
    }

    private function upsertSetTranslations(QuestionSet $qset, array $setJson): void
    {
        $locales = ['ja','en'];
        foreach ($locales as $locale) {
            $title       = $setJson['title'][$locale]       ?? null;
            $description = $setJson['description'][$locale] ?? null;
            $background = $setJson['background'][$locale] ?? null;

            if ($title !== null || $description !== null || $background !== null) {
                QuestionSetTranslation::updateOrCreate(
                    [
                        'question_set_id' => $qset->id,
                        'locale'          => $locale,
                    ],
                    [
                        'title'       => $title,
                        'description' => $description,
                        'background' => $background,
                    ]
                );
            }
        }
    }

    private function syncQuestionSetQuestions(QuestionSet $qset, array $questionJsonIds): void
    {
        // いったん既存pivotを削除
        DB::table('question_set_questions')
            ->where('question_set_id', $qset->id)
            ->delete();

        // JSONに書かれているquestionsを順番に紐付け
        foreach ($questionJsonIds as $idx => $qJsonId) {
            $question = Question::where('json_id', $qJsonId)->first();
            if (!$question) {
                $this->warn("Question not found for json_id={$qJsonId}");
                continue;
            }

            DB::table('question_set_questions')->insert([
                'id'               => (string) Str::uuid(),
                'question_set_id'  => $qset->id,
                'question_id'      => $question->id,
                'order'            => $idx + 1,
                'created_at'       => now(),
                'updated_at'       => now(),
            ]);
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
}


```
