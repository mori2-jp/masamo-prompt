以下の処理を実装してください

# 指示
1.
QuestionJsonManageServiceに、metadata の中身を言語設定に合わせた値のみに置き換えるメソッドを実装してください。

言語設定は、リクエスト時に　SetLocaleMiddleware　によって、App::setLocale に設定されています。

QuestionJsonManageService　の　validateQuestionJson　に、metadata のバリデーションルールが記載されているので仕様をそこを確認してください。

question と　question_components.content　がオブジェクトとして、言語設定をキーとして値が定義されているので、ここから言語設定に従って値を取り出して、
```json
    "question":"8 × 4 = ▢ × 8 = ▢",
"question_components": [
{
"type": "text",
"content": "8 × 4 = ",
"order": 1
},
]
```
のような形式に変換してください。
question_components.content　は、question_components.type　が　"text" の場合に存在しています。

2.
StudySessionService　の beginUserQuestionSet で、QuestionDto に metadata を渡しています。
この箇所で,1で実装したメソッドを利用して、多言語化された metadata を渡すように修正してください。

# 実装方針
コードは Testable に実装してください。例えばDIする箇所はDIするなど。
生成また修正変更するコードはそのコードだけは必ず省略せずに必ず全てアウトプットすること。
仕様はソースコードにまとめてコメントとして残すこと

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

--- metadata の中身
```json
{
    "question_type": "FILL_IN_THE_BLANK",
    "question": {
      "ja": "8 × 4 = ▢ × 8 = ▢",
      "en": "8 × 4 = ▢ × 8 = ▢"
    },
    "input_format": {
      "type": "fixed",
      "fields": [
        {
          "field_id": "f_1",
          "attribute": "number",
          "collect_answer": "number"
        },
        {
          "field_id": "f_2",
          "attribute": "number",
          "collect_answer": "number"
        }
      ],
      "question_components": [
        {
          "type": "text",
          "content": {
            "ja": "8 × 4 = ",
            "en": "8 × 4 = "
          },
          "order": 1
        },
        {
          "type": "blank",
          "field_id": "f_1",
          "order": 2
        },
        {
          "type": "text",
          "content": {
            "ja": " × 8 = ",
            "en": " × 8 = "
          },
          "order": 3
        },
        {
          "type": "blank",
          "field_id": "f_2",
          "order": 4
        }
      ]
    }
  }
```

--- SetLocaleMiddleware
```php
<?php

// app/Http/Middleware/SetLocaleMiddleware.php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\App;

class SetLocaleMiddleware
{
    /**
     * APIリクエストから Locale を取得し、アプリケーションに設定するミドルウェア
     */
    public function handle(Request $request, Closure $next)
    {
        // ※ "Accept-Language" ヘッダーから取得する例
        //    指定がなければ config('app.locale') を使用
        $locale = $request->header('Accept-Language', config('app.locale'));

        // 例: 簡易的に英語(en)/日本語(ja)以外は fallback_locale にするなど
        //     (必要な場合はロジックを拡張してください)
        $supported = ['en', 'ja'];
        if (! in_array($locale, $supported)) {
            $locale = config('app.fallback_locale');
        }

        App::setLocale($locale);

        return $next($request);
    }
}

```

--- QuestionDto
```php
<?php

namespace App\Dtos\V1\Question;

use App\Dtos\Dto;

class QuestionDto extends Dto
{
    public string $id;
    public ?string $question_text;
    public ?string $explanation;
    public ?string $metadata;
    public ?string $version;
    public ?int $status;
    public ?string $question_type;
}

```


--- StudySessionService.php
```php
<?php

namespace App\Services\V1\StudySession;

use App\Dtos\V1\User\UserQuestionDto;
use App\Dtos\V1\User\UserQuestionSetDto;
use App\Dtos\V1\Question\QuestionSetDto;
use App\Dtos\V1\Question\QuestionDto;
use App\Enums\UserQuestionSetStatus;
use App\Enums\UserQuestionStatus;
use App\Models\Question\Question;
use App\Models\Question\QuestionSet;
use App\Models\User\UserQuestion;
use App\Models\User\UserQuestionSet;
use App\Models\User\UserQuestionSetTranslation;
use Carbon\Carbon;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Str;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

class StudySessionService
{
    /**
     * question_set_id を直接受け取り、user_question_sets (NOT_START or PROGRESS) があればそれを返却、
     * なければ作成し、該当 question_set に紐づく question を user_questions として生成する
     */
    public function getOrCreateUserQuestionSetByQuestionSetId(string $userId, string $questionSetId): UserQuestionSetDto
    {
        // question_set_id をキーに question_sets を取得
        $questionSet = QuestionSet::with([
            'translations',
            'questionSetQuestions' => function($q) {
                $q->orderBy('order','asc');
            },
            'questionSetQuestions.question.translations'
        ])
            ->where('id', $questionSetId)
            ->first();

        if (!$questionSet) {
            // TODO エラーメッセージ多言語化
            throw new NotFoundHttpException("QuestionSet not found");
        }

        // 既存の user_question_set (NOT_START or PROGRESS) があるか確認
        $existing = UserQuestionSet::where('user_id', $userId)
            ->where('question_set_id', $questionSet->id)
            ->whereIn('status', [
                UserQuestionSetStatus::NOT_START->value,
                UserQuestionSetStatus::PROGRESS->value
            ])
            ->first();

        if ($existing) {
            // DTOに変換して返す (question_set を翻訳付きでネスト)
            return $this->toUserQuestionSetDto($existing, $questionSet);
        }

        // なければ新規作成 → user_questions 生成
        DB::beginTransaction();
        try {
            // -------------------------
            // user_question_sets の作成
            // -------------------------
            $userQuestionSet = new UserQuestionSet();
            $userQuestionSet->id = (string) Str::uuid();
            $userQuestionSet->user_id = $userId;
            $userQuestionSet->question_set_id = $questionSet->id;
            $userQuestionSet->status = UserQuestionSetStatus::NOT_START->value;
            $userQuestionSet->score = null;
            $userQuestionSet->started_at = null;
            $userQuestionSet->finished_at = null;
            $userQuestionSet->save();

            // user_question_set_translations へコピー
            //     questionSet->translations の全locale/title/descriptionを複製
            foreach ($questionSet->translations as $qsTrans) {
                UserQuestionSetTranslation::create([
                    'user_question_set_id' => $userQuestionSet->id,
                    'locale'      => $qsTrans->locale,
                    'title'       => $qsTrans->title,
                    'description' => $qsTrans->description,
                ]);
            }


            // user_questions を作成
            //     question_set_questions より:
            //       -> question_id, order
            //       -> question の各種フィールドを user_questions にコピー
            // question_set_questions から question_id を取得し、user_questions 作成
            foreach ($questionSet->questionSetQuestions as $qSq) {
                $question = $qSq->question;
                if (!$question) {
                    continue;
                }

                // user_questions レコード作成
                $userQuestion = new UserQuestion();
                $userQuestion->id = (string) Str::uuid();
                $userQuestion->user_question_set_id = $userQuestionSet->id;
                $userQuestion->question_id = $question->id;
                $userQuestion->status = UserQuestionStatus::NOT_START->value;
                $userQuestion->answer_data = null;
                $userQuestion->answered_at = null;

                // question のフィールドをコピー
                $userQuestion->metadata = $question->metadata;
                $userQuestion->version = $question->version;
                $userQuestion->evaluation_method = $question->evaluation_method;
                $userQuestion->checker_method = $question->checker_method;
                $userQuestion->llm_evaluation_prompt_number = $question->llm_evaluation_prompt_number;
                $userQuestion->llm_evaluation_response_format = $question->llm_evaluation_response_format;
                $userQuestion->question_type = $question->question_type;
                $userQuestion->learning_requirement_json = $question->learning_requirement_json;
                $userQuestion->learning_subject = $question->learning_subject;
                $userQuestion->learning_no = $question->learning_no;
                $userQuestion->learning_requirement = $question->learning_requirement;
                $userQuestion->learning_required_competency = $question->learning_required_competency;
                $userQuestion->learning_background = $question->learning_background;
                $userQuestion->learning_category = $question->learning_category;
                $userQuestion->learning_grade_level = $question->learning_grade_level;
                $userQuestion->learning_url = $question->learning_url;
                $userQuestion->generated_by_llm = $question->generated_by_llm;

                // pivotテーブルのorderを userQuestionにコピー
                $userQuestion->order = $qSq->order;

                $userQuestion->save();
            }
            DB::commit();

            return $this->toUserQuestionSetDto($userQuestionSet, $questionSet);
        } catch (\Throwable $e) {
            DB::rollBack();
            throw $e;
        }
    }

    /**
     * user_question_sets を PROGRESS にし、最初の user_question (order 昇順) をDTOで返す
     * 返却の際に question の翻訳をネストする
     */
    public function beginUserQuestionSet(string $userId, string $userQuestionSetId): UserQuestionDto
    {
        $userQuestionSet = UserQuestionSet::where('id', $userQuestionSetId)
            ->where('user_id', $userId)
            ->first();

        // TODO エラーメッセージの多言語化
        if (!$userQuestionSet) {
            throw new NotFoundHttpException("Question data not found ({$userQuestionSetId}).");
        }

        // まず status=PROGRESS の中で orderが最小のものを検索
        $progressQuestion = UserQuestion::where('user_question_set_id', $userQuestionSet->id)
            ->where('status', UserQuestionStatus::PROGRESS->value)
            ->orderBy('order', 'asc')
            ->first();

        // なければ status=NOT_START の中で orderが最小のものを検索
        $firstUserQuestion = $progressQuestion;
        if (!$progressQuestion) {
            $firstUserQuestion = UserQuestion::where('user_question_set_id', $userQuestionSet->id)
                ->where('status', UserQuestionStatus::NOT_START->value)
                ->orderBy('order', 'asc')
                ->first();
        }

        // TODO エラーメッセージの多言語化
        if (!$firstUserQuestion) {
            throw new NotFoundHttpException("No question found ({$userQuestionSetId}).");
        }

        // 問題がまだ開始ではない場合に回答中ステータスに更新する
        if ($firstUserQuestion->status = UserQuestionStatus::NOT_START->value) {
            $firstUserQuestion->status = UserQuestionStatus::PROGRESS->value;
            $firstUserQuestion->save();
        }


        // ステータスを PROGRESS に変更
        if ($userQuestionSet->status !== UserQuestionSetStatus::PROGRESS->value) {
            $userQuestionSet->status = UserQuestionSetStatus::PROGRESS->value;
            $userQuestionSet->started_at = Carbon::now();
            $userQuestionSet->save();
        }

        // 取得した user_questions レコードを元に DTO 生成
        // user_questions テーブル上の question_id を使って question を取得 → 翻訳を適用
        $questionId = $firstUserQuestion->question_id;
        $questionModel = Question::find($questionId);

        $questionDto = null;
        if ($questionModel) {
            $translation = $questionModel->translations->firstWhere('locale', app()->getLocale())
                ?: $questionModel->translations->firstWhere('locale', config('app.fallback_locale'));

            $questionDto = new QuestionDto([
                'id'          => (string) $questionModel->id,
                'question_text'       => $translation?->question_text ?? '',
                'explanation' => $translation?->explanation ?? '',
                'metadata' => $questionModel->metadata ?? '',
                'version' => $questionModel->version ?? '',
                'question_type' => $questionModel->question_type ?? '',
            ]);
        }

        // UserQuestionDto を返す
        return new UserQuestionDto([
            'id'          => (string) $firstUserQuestion->id,
            'question_id' => (string) $firstUserQuestion->question_id,
            'status'      => (int) $firstUserQuestion->status,
            'answer_data' => $firstUserQuestion->answer_data,
            'answered_at' => $firstUserQuestion->answered_at,
            'question'    => $questionDto,
        ]);
    }

    /**
     * UserQuestionSet + QuestionSet を翻訳込みで DTO へ変換
     */
    private function toUserQuestionSetDto(UserQuestionSet $userQuestionSet, QuestionSet $questionSet): UserQuestionSetDto
    {
        // question_set の翻訳を取得
        $qsTranslation = $questionSet->translations->firstWhere('locale', app()->getLocale())
            ?: $questionSet->translations->firstWhere('locale', config('app.fallback_locale'));

        $questionSetDto = new QuestionSetDto([
            'id'          => (string) $questionSet->id,
            'unit_id'     => (string) $questionSet->unit_id,
            'title'       => $qsTranslation?->title ?? '',
            'description' => $qsTranslation?->description ?? '',
            'status'      => (int) $questionSet->status,
            'order'       => (int) $questionSet->order,
        ]);

        return new UserQuestionSetDto([
            'id'            => (string) $userQuestionSet->id,
            'user_id'       => (string) $userQuestionSet->user_id,
            'question_set_id' => (string) $userQuestionSet->question_set_id,
            'status'        => (int) $userQuestionSet->status,
            'score'         => $userQuestionSet->score ? (float) $userQuestionSet->score : null,
            'started_at'    => $userQuestionSet->started_at,
            'finished_at'   => $userQuestionSet->finished_at,
            'question_set'  => $questionSetDto,
        ]);
    }
}

```


--- QuestionJsonManageService（問題JSONを操作するクラス）
```php
<?php

namespace App\Services\Utils\Question;

use App\Enums\EvaluationMethod;
use App\Enums\QuestionStatus;
use App\Enums\QuestionType;
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\ValidationException;

/**
 * Class QuestionJsonManageService
 *
 * 【概要】
 * - GitHubリポジトリ等からインポートされる問題JSONをバリデーション＆加工するサービスクラス
 * - question_type が増えた場合にルールを簡単追加・削除できるような実装方針
 * - バリデーション違反が見つかったら ValidationException をthrowし、呼び出し元がスキップ(処理停止しない)できる
 *
 * 【今回の修正内容・実装仕様】
 * 1. evaluation_spec.response_format.fields.collect_answer は、evaluation_spec.response_format.fields.user_answer の型(例:"number")に従い、
 *    実際に数値が入っているかをチェックする（user_answer="number" なら collect_answer が実際に数値型かどうか）。
 * 2. metadata.input_format.fields.collect_answer は文字列で "number" などの「型表記」が入っているかチェックする。
 * 3. evaluation_spec.response_format の is_correct="boolean" / score="number" などの要件はそのまま踏襲
 * 4. question_type が FILL_IN_THE_BLANK, SCENARIO etc. 今後100種以上増える想定 → メソッド分割 or テーブル形式にするなどで拡張容易
 * 5. エラーメッセージは日本語でわかりやすく出力
 * 6. ルールに違反しても問題全体をスキップし続行できるように ValidationException で処理
 * 7. ソースコードコメントで仕様をまとめて明示
 */
class QuestionJsonManageService
{
    /**
     * 言語定数 (アプリケーション全体で共通利用)
     */
    public const LANGUAGES = ['ja', 'en'];

    /**
     * input_format.type 定数
     */
    public const INPUT_FORMAT_TYPES = [
        'fixed',
        'custom',
    ];

    /**
     * input_format.fields.type / evaluation_spec.response_format.fields.user_answer / metadata.input_format.fields.collect_answer
     * の定数 (数値型など)
     */
    public const FIELD_TYPE = [
        'number'
    ];

    /**
     * input_format.question_components.type 定数
     */
    public const COMPONENT_TYPES = [
        'text',
        'image',
        'movie',
        'blank',
    ];

    /**
     * 問題JSONをバリデーションするメインメソッド
     *
     * @param array $json
     * @return array  バリデーション通過後のデータ (今回はそのまま返却)
     * @throws ValidationException バリデーション違反があればthrow、呼び出し元でスキップ可能
     */
    public function validateQuestionJson(array $json): array
    {
        // 1) 最上位の共通ルール
        $validator = Validator::make($json, $this->getTopLevelRules(), $this->messages());

        // 2) after()で追加チェック (DB存在確認, LLM関連, question_type別 など)
        $validator->after(function ($v) use ($json) {
            $this->validateLevelGradeDifficulty($v, $json);
            $this->validateSkills($v, $json);
            $this->validateEvaluationSpec($v, $json);
            $this->validateMetadata($v, $json);
        });

        // 3) バリデーション実行 (失敗すると ValidationException)
        $validatedData = $validator->validate();

        // 4) 必要あれば変換や加工 (ここではそのまま返す)
        return $validatedData;
    }

    /**
     * トップレベル共通ルール
     * - order, id, level_id, ... など
     * - 後段のafter()でさらにDB存在確認やLLM用のチェックを行う
     */
    private function getTopLevelRules(): array
    {
        return [
            'order'         => ['required','integer'],
            'id'            => ['required','string'],
            'level_id'      => ['required','string'],
            'grade_id'      => ['required','string'],
            'difficulty_id' => ['required','string'],
            'version'       => ['required','regex:/^\d+\.\d+\.\d+$/'],   // 例: "1.0.0"
            'status'        => ['required','string'],  // 後でQuestionStatus::fromString()でenum化
            'generated_by_llm' => ['required','boolean'],
            'created_at'    => ['required','date_format:Y-m-d H:i:s'],
            'updated_at'    => ['required','date_format:Y-m-d H:i:s'],

            // question_text, explanation, background => (ja,en)必須
            'question_text'           => ['required','array'],
            'question_text.ja'        => ['required','string'],
            'question_text.en'        => ['required','string'],
            'explanation'             => ['required','array'],
            'explanation.ja'          => ['required','string'],
            'explanation.en'          => ['required','string'],
            'background'              => ['required','array'],
            'background.ja'           => ['required','string'],
            'background.en'           => ['required','string'],

            // skills: 必須 配列
            'skills' => ['required','array'],

            // learning_requirements: 必須(オブジェクトだが複数要素想定)
            'learning_requirements'                               => ['required','array'],
            'learning_requirements.*.learning_subject'            => ['required','string'],
            'learning_requirements.*.learning_no'                 => ['required','integer'],
            'learning_requirements.*.learning_requirement'        => ['required','string'],
            'learning_requirements.*.learning_required_competency'=> ['required','string'],
            'learning_requirements.*.learning_background'         => ['required','string'],
            'learning_requirements.*.learning_category'           => ['required','string'],
            'learning_requirements.*.learning_grade_level'        => ['required','string'],
            'learning_requirements.*.learning_url'                => ['sometimes','url'], //必須ではないがあればURL

            // evaluation_spec: 必須
            'evaluation_spec' => ['required','array'],
            'evaluation_spec.evaluation_method' => ['required','string'],

            // metadata => question_type / input_format etc
            'metadata'                              => ['required','array'],
            'metadata.question_type'                => ['required'], // 後でenum化(文字列or数値OK)
            'metadata.question'                     => ['required','array'],
            'metadata.question.ja'                  => ['required','string'],
            'metadata.question.en'                  => ['required','string'],
            'metadata.input_format'                 => ['required','array'],
            'metadata.input_format.type'            => ['required','string','in:fixed,custom'],
            'metadata.input_format.fields'          => ['required','array'],
            'metadata.input_format.question_components' => ['required','array'],
        ];
    }

    /**
     * 日本語エラーメッセージ
     */
    private function messages(): array
    {
        return [
            'required' => ':attribute は必須項目です。',
            'integer'  => ':attribute は数値で指定してください。',
            'string'   => ':attribute は文字列である必要があります。',
            'boolean'  => ':attribute はtrue/falseで指定してください。',
            'date_format' => ':attribute は :format の形式が必要です。',
            'regex' => ':attribute の形式が不正です。(例: 1.0.0)',
            'array' => ':attribute は配列である必要があります。',
            'in'    => ':attribute の値が不正です( :values )。',
            'url'   => ':attribute はURL形式で入力してください。',
        ];
    }

    /**
     * level_id, grade_id, difficulty_id がDBにあるか
     * 存在しなければエラーを付加
     */
    private function validateLevelGradeDifficulty($validator, array $json)
    {
        $levelId = $json['level_id'] ?? null;
        $gradeId = $json['grade_id'] ?? null;
        $diffId  = $json['difficulty_id'] ?? null;

        // levels
        if ($levelId) {
            $exists = \DB::table('levels')->where('json_id', $levelId)->exists();
            if (!$exists) {
                $validator->errors()->add('level_id', "指定された level_id='{$levelId}' はDBに存在しません。");
            }
        }
        // grades
        if ($gradeId) {
            $exists = \DB::table('grades')->where('json_id', $gradeId)->exists();
            if (!$exists) {
                $validator->errors()->add('grade_id', "指定された grade_id='{$gradeId}' はDBに存在しません。");
            }
        }
        // difficulties
        if ($diffId) {
            $exists = \DB::table('difficulties')->where('json_id', $diffId)->exists();
            if (!$exists) {
                $validator->errors()->add('difficulty_id', "指定された difficulty_id='{$diffId}' はDBに存在しません。");
            }
        }
    }

    /**
     * skills: skill_idがDBにあるか & nameがDBのdisplay_nameと一致するか
     */
    private function validateSkills($validator, array $json)
    {
        $skills = $json['skills'] ?? [];
        if (!is_array($skills)) {
            return;
        }
        foreach ($skills as $idx => $sk) {
            $sid   = $sk['skill_id'] ?? null;
            $sname = $sk['name']     ?? null;
            if (!$sid || !$sname) {
                continue;
            }
            $row = \DB::table('skills')->where('json_id', $sid)->first();
            if (!$row) {
                $validator->errors()->add("skills.{$idx}.skill_id", "skill_id='{$sid}' はDBに存在しません。");
                continue;
            }
            if ($row->display_name !== $sname) {
                $validator->errors()->add("skills.{$idx}.name", "skill_id='{$sid}' の display_name と name='{$sname}' が一致しません。");
            }
        }
    }

    /**
     * evaluation_spec: evaluation_method=LLMの場合に追加必須項目 (llm_prompt_number, response_format) など
     */
    private function validateEvaluationSpec($validator, array $json)
    {
        $eval = $json['evaluation_spec'] ?? [];
        $methodRaw = $eval['evaluation_method'] ?? null;
        if (!$methodRaw) {
            return;
        }

        // enumに存在するか
        try {
            $enm = EvaluationMethod::fromString($methodRaw);
        } catch (\InvalidArgumentException) {
            $validator->errors()->add('evaluation_spec.evaluation_method', "evaluation_method='{$methodRaw}' は不正です (CODE/LLMのみ)。");
            return;
        }

        // method=LLM
        if ($enm === EvaluationMethod::LLM) {
            // llm_prompt_number
            if (!isset($eval['llm_prompt_number']) || !is_numeric($eval['llm_prompt_number'])) {
                $validator->errors()->add('evaluation_spec.llm_prompt_number', "evaluation_method=LLM のため数値 llm_prompt_number が必須です。");
            } else {
                $path = resource_path("prompts/evaluation/{$eval['llm_prompt_number']}.txt");
                if (!file_exists($path)) {
                    $validator->errors()->add('evaluation_spec.llm_prompt_number', "LLM promptファイルが見つかりません。(path={$path})");
                }
            }

            // response_format
            if (!isset($eval['response_format']) || !is_array($eval['response_format'])) {
                $validator->errors()->add('evaluation_spec.response_format', "evaluation_method=LLM のため response_format オブジェクトが必須です。");
            } else {
                $this->validateResponseFormat($validator, $json, $eval['response_format']);
            }
        }
    }

    /**
     * evaluation_spec.response_format の詳細チェック
     * - is_correct => "boolean"(文字列)
     * - score => "number"(文字列)
     * - question_text/explanation/question => langごとに 'text'
     * - fields => user_answer="number" / is_correct="boolean" / collect_answer => (user_answer="number"なら実際に数値か)
     */
    private function validateResponseFormat($validator, array $json, array $resp)
    {
        // is_correct => "boolean"
        if (!isset($resp['is_correct']) || $resp['is_correct'] !== 'boolean') {
            $validator->errors()->add('evaluation_spec.response_format.is_correct', "is_correct は文字列 'boolean' のみ有効です。");
        }
        // score => "number"
        if (!isset($resp['score']) || $resp['score'] !== 'number') {
            $validator->errors()->add('evaluation_spec.response_format.score', "score は文字列 'number' のみ有効です。");
        }

        // question_text => langごと 'text'
        if (empty($resp['question_text']) || !is_array($resp['question_text'])) {
            $validator->errors()->add('evaluation_spec.response_format.question_text', "question_text(オブジェクト) が必須です。");
        } else {
            foreach (self::LANGUAGES as $lang) {
                if (!isset($resp['question_text'][$lang])) {
                    $validator->errors()->add("evaluation_spec.response_format.question_text.{$lang}", "question_text.{$lang} がありません。");
                    continue;
                }
                if ($resp['question_text'][$lang] !== 'text') {
                    $validator->errors()->add("evaluation_spec.response_format.question_text.{$lang}", "question_text.{$lang} は 'text' のみ有効です。");
                }
            }
        }

        // explanation => langごと 'text'
        if (empty($resp['explanation']) || !is_array($resp['explanation'])) {
            $validator->errors()->add('evaluation_spec.response_format.explanation', "explanation(オブジェクト) が必須です。");
        } else {
            foreach (self::LANGUAGES as $lang) {
                if (!isset($resp['explanation'][$lang])) {
                    $validator->errors()->add("evaluation_spec.response_format.explanation.{$lang}", "explanation.{$lang} がありません。");
                    continue;
                }
                if ($resp['explanation'][$lang] !== 'text') {
                    $validator->errors()->add("evaluation_spec.response_format.explanation.{$lang}", "explanation.{$lang} は 'text' のみ有効です。");
                }
            }
        }

        // question => langごと 'text'
        if (empty($resp['question']) || !is_array($resp['question'])) {
            $validator->errors()->add('evaluation_spec.response_format.question', "question(オブジェクト) は必須です。");
        } else {
            foreach (self::LANGUAGES as $lang) {
                if (!isset($resp['question'][$lang])) {
                    $validator->errors()->add("evaluation_spec.response_format.question.{$lang}", "question.{$lang} がありません。");
                    continue;
                }
                if ($resp['question'][$lang] !== 'text') {
                    $validator->errors()->add("evaluation_spec.response_format.question.{$lang}", "question.{$lang} は 'text' のみ有効です。");
                }
            }
        }

        // fields => 配列必須
        if (empty($resp['fields']) || !is_array($resp['fields'])) {
            $validator->errors()->add('evaluation_spec.response_format.fields', "fields(配列) は必須です。");
        } else {
            foreach ($resp['fields'] as $idx => $f) {
                $this->validateResponseField($validator, $f, $idx);
            }
        }
    }

    /**
     * response_format.fields.* のチェック
     * - user_answer => "number"
     * - is_correct => "boolean"
     * - collect_answer => user_answer="number"なら実際に数値か?
     * - field_explanation => langごと必須
     */
    private function validateResponseField($validator, array $field, int $idx)
    {
        // field_id
        if (empty($field['field_id']) || !is_string($field['field_id'])) {
            $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.field_id", "field_id(文字列) は必須です。");
        }

        // user_answer => "number" のみ
        if (!isset($field['user_answer']) || $field['user_answer'] !== 'number') {
            $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.user_answer", "user_answer は 'number' のみ有効です。");
        }

        // is_correct => "boolean" (文字列)
        if (!isset($field['is_correct']) || $field['is_correct'] !== 'boolean') {
            $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.is_correct", "is_correct は 'boolean' のみ有効です。");
        }

        // collect_answer => user_answer="number" であれば実際に数値を期待
        if (isset($field['user_answer']) && $field['user_answer'] === 'number') {
            // 数値か？
            if (!isset($field['collect_answer']) || !is_numeric($field['collect_answer'])) {
                $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.collect_answer", "collect_answer は数値である必要があります (user_answer='number')。");
            }
        }

        // field_explanation => ja,en 文字列
        if (empty($field['field_explanation']) || !is_array($field['field_explanation'])) {
            $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.field_explanation", "field_explanation(オブジェクト) は必須です。");
        } else {
            foreach (self::LANGUAGES as $lang) {
                if (!isset($field['field_explanation'][$lang]) || !is_string($field['field_explanation'][$lang])) {
                    $validator->errors()->add("evaluation_spec.response_format.fields.{$idx}.field_explanation.{$lang}", "field_explanation.{$lang} は必須の文字列です。");
                }
            }
        }
    }

    /**
     * metadata (question_type / input_format.*) の検証
     * - question_type => Enum
     * - input_format.fields => attribute="number", collect_answer => "number"(文字列)
     * - question_components => type in [text,image,movie,blank], blank数=fields数
     */
    private function validateMetadata($validator, array $json)
    {
        $meta = $json['metadata'] ?? [];
        $qtRaw = $meta['question_type'] ?? null;

        // question_type => enum
        if ($qtRaw) {
            $str = (string)$qtRaw;
            try {
                QuestionType::fromString($str);
            } catch (\InvalidArgumentException) {
                $validator->errors()->add('metadata.question_type', "question_type='{$str}' は未定義です。");
            }
        }

        // input_format.fields => attribute="number", collect_answer => "number" (文字列)
        $fieldsArr = $meta['input_format']['fields'] ?? [];
        if (is_array($fieldsArr)) {
            $fieldIds = [];
            foreach ($fieldsArr as $idx => $f) {
                // field_id => f_数字
                if (empty($f['field_id']) || !preg_match('/^f_\d+$/', $f['field_id'])) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.field_id", "field_id='f_数字'形式で必須です。");
                }
                // 重複check
                $fid = $f['field_id'] ?? '';
                if (in_array($fid, $fieldIds, true)) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.field_id", "field_id='{$fid}' が重複しています。");
                }
                $fieldIds[] = $fid;

                // attribute => 'number'
                if (!isset($f['attribute']) || !in_array($f['attribute'], self::FIELD_TYPE, true)) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.attribute", "attribute='{$f['attribute']}' は 'number' のみ有効です。");
                }

                // collect_answer => "number" (文字列) など
                if (!isset($f['collect_answer']) || !is_string($f['collect_answer'])) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.collect_answer", "collect_answer は文字列で必須です。(例:'number')");
                } else {
                    // 現仕様では "number" のみ
                    if ($f['collect_answer'] !== 'number') {
                        $validator->errors()->add("metadata.input_format.fields.{$idx}.collect_answer", "collect_answer='{$f['collect_answer']}' は 'number' のみ有効。");
                    }
                }
            }
        }

        // question_components => type=text|image|movie|blank, blank数がfields数に一致
        $comps = $meta['input_format']['question_components'] ?? [];
        if (is_array($comps)) {
            $orders = [];
            $blankCount = 0;
            foreach ($comps as $cidx => $comp) {
                $type = $comp['type'] ?? '';
                if (!in_array($type, self::COMPONENT_TYPES, true)) {
                    $validator->errors()->add("metadata.input_format.question_components.{$cidx}.type", "不正なコンポーネントtype='{$type}'です。");
                }
                if ($type === 'blank') {
                    $blankCount++;
                    // field_id必須
                    if (!isset($comp['field_id'])) {
                        $validator->errors()->add("metadata.input_format.question_components.{$cidx}.field_id", "type=blank の場合 field_id が必須です。");
                    }
                } else {
                    // text|image|movie => content.{ja,en} が必須
                    if (empty($comp['content']) || !is_array($comp['content'])) {
                        $validator->errors()->add("metadata.input_format.question_components.{$cidx}.content", "type='{$type}' の場合 content(オブジェクト) が必須です。");
                    } else {
                        foreach (self::LANGUAGES as $lang) {
                            if (!isset($comp['content'][$lang])) {
                                $validator->errors()->add("metadata.input_format.question_components.{$cidx}.content.{$lang}", "content.{$lang} がありません。");
                            }
                        }
                    }
                }
                // order => 数値 + 重複
                if (!isset($comp['order']) || !is_numeric($comp['order'])) {
                    $validator->errors()->add("metadata.input_format.question_components.{$cidx}.order", "order(数値) は必須です。");
                } else {
                    if (in_array($comp['order'], $orders, true)) {
                        $validator->errors()->add("metadata.input_format.question_components.{$cidx}.order", "order='{$comp['order']}' が重複しています。");
                    }
                    $orders[] = $comp['order'];
                }
            }
            // blank数 と fields数 が一致？
            $fieldsCount = count($fieldsArr);
            if ($blankCount !== $fieldsCount) {
                $validator->errors()->add(
                    "metadata.input_format.question_components",
                    "blank要素数({$blankCount}) と fields数({$fieldsCount}) が一致しません。"
                );
            }
        }
    }
}


```
