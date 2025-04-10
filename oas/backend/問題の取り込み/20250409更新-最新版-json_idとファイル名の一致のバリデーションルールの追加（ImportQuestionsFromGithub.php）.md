以下、ImportQuestionsFromGithub.php　の処理に新しいバリデーションのルールを追加してください。
修正箇所を知らせてください。

必ず、既存のバリデーションルールは全てそのままにしてください。


# 追加するルール
インポートしたファイルの id と ファイル名が一致しているかチェックしてほしい。
例えば、
qset_s1_g3_sec700_u300_v100_700.json
というファイル名であれば、
JSON内の、id は、ファイル名から、.json を除いた qset_s1_g3_sec700_u300_v100_700　でなければいけません。
ファイル名とidが不一致の場合はバリデーションエラーとしてDBに取り込みません。
処理の最後に、ファイル名とidが不一致だったファイル名とidをコンソールに全て出力してください。

コード内のコメントは消さずに全て残してください。


ーーJSON仕様
order：必須、数値
id：必須、文字列
level_id：必須、文字列、Levels テーブルのjson_idに一致する値があること
grade_id：必須、文字列、Grade テーブルのjson_idに一致する値があること
difficulty_id：必須、文字列、Difficulties テーブルのjson_idに一致する値があること
version： 必須、文字列、文字列が x.x.x のようなバージョニング形式になっているか
status：必須、QuestionStatus に値が存在しているか
generated_by_llm：必須、Boolean
created_at：必須、Y-m-d H:i:s 形式になっていること
updated_at：必須、Y-m-d H:i:s 形式になっていること

skills: 必須、配列
skills.skill_id: 必須、Skills テーブルのjson_idに一致する値があること
skills.name: 必須、文字列、skill_id と一致するSkills テーブルのjson_idを持つデータの　display_name と値が一致すること

learning_requirements:必須、オブジェクト
learning_requirements.learning_subject: 必須、文字列
learning_requirements.learning_no: 必須、数値型
learning_requirements.learning_requirement: 必須、文字列
learning_requirements.learning_required_competency: 必須、文字列
learning_requirements.learning_background: 必須、文字列
learning_requirements.learning_category: 必須、文字列
learning_requirements.learning_grade_level: 必須、文字列
learning_requirements.learning_url: 必須ではない、文字列、URL

evaluation_spec：必須、オブジェクト
evaluation_spec.evaluation_method: 必須、文字列、EvaluationMethod　に値が存在していること
evaluation_spec.checker_method: evaluation_methodが”CODE”の時は必須、文字列型、EvaluationCheckerMethod　に値が存在していること
evaluation_spec.llm_prompt_number: evaluation_methodが”LLM”の時は必須、数値。LLMに投げるプロンプトを管理するファイルと対応している。resources/prompts/evaluation/{x}.txt の {x}の箇所と対応しているので、このファイルが存在しているかバリデーションチェックする。
evaluation_spec.response_format：必須、オブジェクト。LLMに正誤判定を依頼する時に指定するレスポンスの形や、CheckerMethod での正誤判定時のレスポンスに利用する
evaluation_spec.response_format.is_correct：必須、テキスト型("boolean"のみ）。回答全体が正解かどうか表す
evaluation_spec.response_format.score：必須、テキスト型("number"のみ）。スコアを表す
evaluation_spec.response_format.question_text：必須、オブジェクト（言語定数全て含んでいるか）, evaluation_methodが”LLM”の時は、言語ごとにそれぞれ オブジェクトの中の値は "{$言語設定例えば"ja"など}": "text"となっていること。evaluation_methodが”CODE”の時は、言語ごとにそれぞれ question_text と全く同じ値が含まれていること
evaluation_spec.response_format.explanation：必須、オブジェクト（言語定数全て含んでいるか）, evaluation_methodが”LLM”の時は、言語ごとにそれぞれ オブジェクトの中の値は "{$言語設定例えば"ja"など}": "text"となっていること。evaluation_methodが”CODE”の時は、言語ごとにそれぞれ オブジェクトの中の値は "{$言語設定例えば"ja"など}": に問題文となる文字列が含まれていること
evaluation_spec.response_format.question：必須、オブジェクト（言語定数全て含んでいるか）、言語ごとにそれぞれ オブジェクトの中の値は "{$言語設定例えば"ja"など}": "text"となっていること。evaluation_methodが”CODE”の時は、言語ごとにそれぞれ metadata.question と全く同じ値が含まれていること

evaluation_spec.response_format.fields：evaluation_methodが”LLM”の時は必須、配列。CODEのときは正答判定、LLMの時はLLMにユーザーの回答の形を知らせる為にフォーマットを定義。
evaluation_spec.response_format.fields.field_id：必須、配列。ユーザーの回答の型。
evaluation_spec.response_format.fields.user_answer：必須、FieldType の定数に値があるか。ユーザーの回答。FILL_IN_MULTIPART　の時は、sequence　と値が一致すること。
evaluation_spec.response_format.fields.is_correct：必須、テキスト型("boolean"のみ）。ユーザーの回答が正しいか
evaluation_spec.response_format.fields.collect_answer：必須、オブジェクト（言語定数全て含んでいるか）。metadata.question_type が、FILL_IN_THE_BLANK の時は,オブジェクトの中の値が、evaluation_spec.response_format.fields.user_answerで指定されている型と一致しているか。例えば、"number" の場合は、32 などの数値となっているか。問題の正解（ユーザーには隠す）metadata.question_type が、FILL_IN_OPERATOR の時は,FillInOperator に存在する値になっているか。（ > < など）
evaluation_spec.response_format.fields.field_explanation: 必須、オブジェクト、言語ごとにそれぞれ オブジェクトの中の値は "{$言語設定例えば"ja"など}": "文字列"が含まれていること。空文字禁止

metadata.question_type：必須、App\Enums\QuestionType.php に値が存在しているか。JSONには、文字列でも数値でもどちらでも入力可にする

metadata.question_text: 必須、オブジェクト（言語定数全て含んでいるか）
metadata.explanation: 必須、オブジェクト（言語定数全て含んでいるか）
metadata.background: 必須、オブジェクト（言語定数全て含んでいるか）
metadata.question ：必須、オブジェクト（言語定数全て含んでいるか）。問題文

metadata.input_format: 必須、オブジェクト（言語定数全て含んでいるか）
metadata.input_format.fields: FILL_IN_MULTIPART　の時は不要。ユーザーが自由に入力するため、こちらでフィールドを定義して限定する必要がない。

metadata.input_format.question_components: 必須（問題を更生する要素）
metadata.input_format.question_components.type：必須、VALID_COMPONENT_TYPES 定数と値が合っているか
metadata.input_format.question_components.content：question_components.type　が、COMPONENT_TYPES_REQUIRE_CONTENT に含まれる場合に必須、オブジェクト、言語定数と一致する値が全て含まれているか
metadata.input_format.question_components.order: 必須、数値、重複する値が存在しないこと。問題を構築するときの表示順番

metadata.input_format.input_components: metadata.question_type が、FILL_IN_OPERATOR または、FILL_IN_MULTIPART　の時は必須。（回答の選択肢。入力キーパッドのボタン）FILL_IN_MULTIPART　の時は、question を カンマ区切りで分割した値が全て含まれていること。例：question が、14, 15, 18, 20, 21, 25, 27, 35, 45, 50 だった場合は、14〜50までのそれぞれの分割した input_components が 同じ数（ここでは10個）存在していること。
metadata.input_format.input_components.type：必須、VALID_INPUT_COMPONENTS_TYPES 定数と値が合っているか
metadata.input_format.input_components.content：必須、オブジェクト、言語定数と一致するキーが全て含まれているか。FillInOperator に存在する値になっているか。（ > < など）
metadata.input_format.input_components.order: 必須、数値、重複する値が存在しないこと。入力キーパッドを構築するときの表示順番


--- 問題JSONの全体構造
```json
{
  "order": 100,
  "id": "ques_s1_g3_sec300_u300_diff100_qt351_v100_100",
  "level_id": "lev_003",
  "grade_id": "gra_003",
  "difficulty_id": "diff_100",
  "version": "1.0.0",
  "status": "TEST_PUBLISHED",
  "validation_check": true,
  "generated_by_llm": false,
  "created_at": "2025-03-27 13:00:00",
  "updated_at": "2025-03-27 13:00:00",
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
      "learning_requirement": "計算の意味・方法 割り算 あまりのある除法",
      "learning_required_competency": "あまりのある除法を理解し、商とあまりを正しく表せる。",
      "learning_background": "余りのある除法(13÷4=3あまり1等)を正しく行い、余りが除数未満であることを認識できる。",
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
        "ja": "次の ▢ にあてはまる数を答えなさい。",
        "en": "Please answer the numbers that fit in the blanks."
      },
      "explanation": {
        "ja": "25を4で割ると、4×6=24で1つ余るので「6 あまり 1」となります。",
        "en": "When dividing 25 by 4, 4×6=24, leaving 1 as the remainder, so the answer is “6 remainder 1.”"
      },
      "question": {
        "ja": "25 ÷ 4 = ▢",
        "en": "25 ÷ 4 = ▢"
      },
      "fields": [
        {
          "field_id": "f_1",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": 6,
            "en": 6
          },
          "field_explanation": {
            "ja": "25を4で割ったとき、ちょうど4が6回分（24）になるので商は6です。",
            "en": "When dividing 25 by 4, 4 fits exactly 6 times (24), so the quotient is 6."
          }
        },
        {
          "field_id": "f_2",
          "user_answer": "text",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": "あまり",
            "en": "remainder"
          },
          "field_explanation": {
            "ja": "余りがあるときにつける言葉です。",
            "en": "This word indicates there is a remainder in the division."
          }
        },
        {
          "field_id": "f_3",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": 1,
            "en": 1
          },
          "field_explanation": {
            "ja": "25から4×6=24を引いた残りが1なので、余りは1です。",
            "en": "Subtracting 24 (4×6) from 25 leaves 1, so the remainder is 1."
          }
        }
      ]
    }
  },
  "metadata": {
    "question_type": "FILL_IN_MULTIPART",
    "question_text": {
      "ja": "次の ▢ にあてはまる数を答えなさい。",
      "en": "Please answer the numbers that fit in the blanks."
    },
    "explanation": {
      "ja": "25を4で割ると、4×6=24で1つ余るので「6 あまり 1」となります。",
      "en": "When dividing 25 by 4, 4×6=24, leaving 1 as the remainder, so the answer is “6 remainder 1.”"
    },
    "question": {
      "ja": "25 ÷ 4 = ▢",
      "en": "25 ÷ 4 = ▢"
    },
    "background": {
      "ja": "この問題では、あまりのある除法を正しく理解し、商とあまりをきちんと求める力を身につけることを目的としています。小学校3年生でも、余りが出る割り算の計算に慣れ、答えを『○ あまり ○』の形できちんと表せるようにしましょう。",
      "en": "This problem aims to help learners understand division with remainders and accurately determine both the quotient and the remainder. Even for third-grade students, practicing divisions that result in remainders helps them express the answer correctly in the form 'quotient remainder remainder_value.'"
    },
    "input_format": {
      "input_components": [
        {
          "type": "signed_number_pad",
          "order": 50
        },
        {
          "type": "text",
          "content": {
            "ja": "あまり",
            "en": "remainder"
          },
          "order": 100
        }
      ],
      "question_components": [
        {
          "type": "text",
          "content": {
            "ja": "25 ÷ 4 = ▢",
            "en": "25 ÷ 4 = ▢"
          },
          "order": 50
        }
      ]
    }
  }
}

```



# 実装方針
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

＝＝＝問題取り込みコード
```php
<?php

namespace App\Console\Commands\Import\Github;

use App\Enums\EvaluationCheckerMethod;
use App\Enums\EvaluationMethod;
use App\Enums\QuestionStatus;
use App\Enums\QuestionType;
use App\Models\Difficulty\Difficulty;
use App\Models\Grade\Grade;
use App\Models\Level\Level;
use App\Models\Question\Question;
use App\Models\Question\QuestionTranslation;
use App\Services\Utils\Question\QuestionJsonManageService;
use Google_Client;
use Google_Service_Drive;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Str;

/**
 * 【仕様まとめ】
 * 1) staging や local 環境で Google Drive 上の JSON を取り込みたい場合、
 *    サービスアカウント (service-account-credentials.json) で認証し、Drive API を介してダウンロードする。
 * 2) Google Drive でファイルを「service account」に対して共有(閲覧可)し、かつ GCP で Drive API を有効化しておく必要がある。
 * 3) $jsonFileOnGoogleDriveIds にファイルIDを記述しておくと、全て順番にダウンロードし upsertQuestion() へ渡す。
 * 4) GitHub からの取り込み (importFromGitHub) は既存の通り残し、環境が 'staging' なら Drive API、その他はローカル設定で切り替え。
 * 5) 既存の upsertQuestion(), processJsonFile(), fetchAllJsonFilesRecursively() 等のロジックはそのまま利用。
 *
 * - すでに登録済みの questions のうち、generated_by_llm = false で、今回ダウンロードした JSON のリストに存在しない json_id のレコードは、status を QuestionStatus::JSON_NOT_MATCHED に更新。
 * - 更新後に、status = QuestionStatus::JSON_NOT_MATCHED のレコードの id, json_id をすべてコンソールに出力する。
 *
 * - 環境が local の時、status が TEST_PUBLISHED の場合はバリデーションをスキップして保存し、スキップした場合は後で id, json_id をコンソールに出力する。
 */
class ImportQuestionsFromGithub extends Command
{
    /**
     * artisan で呼び出すコマンド名
     *
     * 例: php artisan import:questions-from-github --subject=math
     */
    protected $signature = 'import:questions-from-github
                            {--subject= : If specified, only import files under contents/questions/{subject} }';

    protected $description = 'Import question data from GitHub or Google Drive JSON files and sync DB';

    /**
     * Google Drive 上の "ファイルID" 一覧 (サービスアカウント認証で取得する)
     *
     * 例:
     *   https://drive.google.com/file/d/1jPnYxj8VxeUT5JmnAayPUpl-2BnEJ-ax/view?usp=sharing
     *   ↑のURLの「1jPnYxj8VxeUT5JmnAayPUpl-2BnEJ-ax」がファイルID。
     */
    protected array $jsonFileOnGoogleDriveIds = [
        '1o6db1AIwflqfraGwC6IU2re18Zxtv3Kc',
        '1D0AG1gR9Lt7GO5O8hDqlTgRoVxDGCoMH',
        '1OOxfWOTVixHRqTWpQWXVOM1BcdWMBnxi'
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
     * 今回の取り込み処理で「存在が確認できた json_id」の一覧
     * (すでにDB登録済みか否か、import成功したか否かに関係なく、ダウンロードしたJSONに含まれていたjson_idを格納する)
     */
    private array $recognizedJsonIds = [];

    /**
     * バリデーションをスキップした (環境=local, status=TEST_PUBLISHED) の問題のリストを保管
     * 後で id, json_id をまとめて出力するため
     */
    private array $skippedTestPublishedValidation = [];

    /**
     * メインのハンドラ
     */
    public function handle()
    {
        // 環境判定
        $env = app()->environment(); // 'local', 'staging', etc
        $useGoogleDrive = false;

        if ($env === 'staging') {
            // staging は必ず Google Drive
            $useGoogleDrive = true;
        } elseif ($env === 'local') {
            // local では config('app.use_google_drive_in_local_for_learning_data_import', true) の値を参照
            $useGoogleDrive = config('app.use_google_drive_in_local_for_learning_data_import', true);
        }

        if ($useGoogleDrive) {
            $this->info("【INFO】Running in '{$env}' environment => Google Drive (Service Account認証) からJSONをダウンロードして処理します。");
            $this->importFromGoogleDriveWithServiceAccount();
        } else {
            $this->info("【INFO】Running in '{$env}' environment => GitHub からJSONをダウンロードして処理します。");
            $this->importFromGitHub();
        }

        // 統計出力
        $this->info("Import process completed.");
        $this->line("=======================================");
        $this->info("取り込もうとした Question JSON の総数: {$this->totalCount}");
        $this->info("取り込みに失敗またはスキップした Question JSON の総数: {$this->skipCount}");
        $this->info("取り込みに成功した Question JSON の総数: {$this->successCount}");
        $this->line("=======================================");

        /**
         *   - すでにDBに登録されている question のうち generated_by_llm = false で
         *     今回取り込んだ JSON 一覧($this->recognizedJsonIds) に json_id が含まれないものは、
         *     status を QuestionStatus::JSON_NOT_MATCHED に更新する。
         *   - status=JSON_NOT_MATCHED のレコードの id, json_id を最後に出力。
         */
        $notMatchedQuery = Question::where('generated_by_llm', false)
            ->whereNotIn('json_id', $this->recognizedJsonIds)
            ->where('status', '!=', QuestionStatus::JSON_NOT_MATCHED->value);

        $notMatchedCount = $notMatchedQuery->count();
        if ($notMatchedCount > 0) {
            $this->info("Marking {$notMatchedCount} existing questions as JSON_NOT_MATCHED because no matching JSON was found in this import.");
            $notMatchedQuery->update(['status' => QuestionStatus::JSON_NOT_MATCHED->value]);
        }

        $allNotMatched = Question::where('status', QuestionStatus::JSON_NOT_MATCHED->value)->get(['id','json_id']);
        if ($allNotMatched->isNotEmpty()) {
            $this->info("==== JSON_NOT_MATCHED questions (id, json_id) ====");
            foreach ($allNotMatched as $q) {
                $this->line("id={$q->id}, json_id={$q->json_id}");
            }
        }

        // バリデーションをスキップした (TEST_PUBLISHED) レコードの一覧をコンソール出力
        if (!empty($this->skippedTestPublishedValidation)) {
            $this->info("==== Validation Skipped for TEST_PUBLISHED (Local env) questions ====");
            foreach ($this->skippedTestPublishedValidation as $q) {
                $this->line("id={$q['id']}, json_id={$q['json_id']}");
            }
        }

        return 0;
    }

    /**
     * Google Drive のファイルを
     * Service Account (service-account-credentials.json) で認証して Drive API 経由でダウンロードする。
     *
     * jsonFileOnGoogleDriveIds に記載された複数ファイルを順に読み取り、DBに反映する。
     */
    private function importFromGoogleDriveWithServiceAccount(): void
    {
        $this->info('Setting up Google Client for Drive API (Questions) ...');

        // 1) Googleクライアントの認証設定
        //    例: storage_path('app/private/service-account-credentials.json')
        $client = new Google_Client();
        $client->setAuthConfig(storage_path('app/private/service-account-credentials.json'));
        $client->addScope(Google_Service_Drive::DRIVE_READONLY);

        // 2) Drive サービスインスタンス
        $driveService = new Google_Service_Drive($client);

        // 3) ファイルID をループし、中身(JSON)をダウンロード => upsert
        foreach ($this->jsonFileOnGoogleDriveIds as $fileId) {
            $this->info("Fetching JSON from Google Drive fileId: {$fileId}");

            // 総数カウント
            $this->totalCount++;

            try {
                // alt=media で取得 (認証済ストリーム)
                $response = $driveService->files->get($fileId, ['alt' => 'media']);
                $jsonContent = $response->getBody()->getContents();

                $decoded = json_decode($jsonContent, true);
                if (!is_array($decoded)) {
                    $this->warn("File ID={$fileId} is not valid JSON. Skipped.");
                    $this->skipCount++;
                    continue;
                }

                // DB upsert
                if ($this->upsertQuestion($decoded)) {
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
     * GitHubリポジトリ上の JSON を取り込み（オリジナルロジックを温存）
     */
    private function importFromGitHub(): void
    {
        $token = config('services.github.api_token');
        if (!$token) {
            $this->error('GitHub API token not found in config/services.php [github.api_token]');
            return;
        }

        $repoOwnerAndName = 'NousContentsManagement/masamo-content';

        // ベースパス: contents/questions
        $basePath = 'contents/questions';

        // subject オプション
        $subjectOption = $this->option('subject');
        if ($subjectOption) {
            $basePath .= '/' . $subjectOption;
            $this->info("Subject specified: {$subjectOption}, path= {$basePath}");
        } else {
            $this->info("No subject specified. Will import from all subdirectories under contents/questions");
        }

        $allJsonFiles = $this->fetchAllJsonFilesRecursively($repoOwnerAndName, $basePath, $token);
        if (empty($allJsonFiles)) {
            $this->warn("No JSON files found under path: {$basePath}");
            return;
        }

        $this->info("Found " . count($allJsonFiles) . " JSON file(s) under {$basePath}");

        // 各ファイルを処理
        foreach ($allJsonFiles as $file) {
            $this->processJsonFile($file, $token);
        }
    }

    /**
     * 再帰的に GitHub 上のディレクトリを走査し、.json ファイル一覧を返す
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
            $type = $item['type'] ?? null;  // 'file' or 'dir'
            $itemPath = $item['path'] ?? '';
            $name     = $item['name'] ?? '';

            if ($type === 'dir') {
                $descendantFiles = $this->fetchAllJsonFilesRecursively($repo, $itemPath, $token);
                $result = array_merge($result, $descendantFiles);
            } elseif ($type === 'file') {
                if (str_ends_with($name, '.json')) {
                    $result[] = $item;
                }
            }
        }

        return $result;
    }

    /**
     * JSONファイル一件をダウンロードしてパースし、DBに反映 (GitHub用)
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

        if ($this->upsertQuestion($decoded)) {
            $this->successCount++;
        } else {
            $this->skipCount++;
        }
    }

    /**
     * 単一の問題(JSON)をDBへ upsert
     *
     * @return bool 成功ならtrue、スキップ/失敗ならfalse
     */
    private function upsertQuestion(array $questionJson): bool
    {
        DB::beginTransaction();
        try {
            $jsonId = $questionJson['id'] ?? null;
            if (!$jsonId) {
                $this->warn("Question JSON missing 'id'. Skipped.");
                DB::rollBack();
                return false;
            }

            // この json_id を「今回認識した」一覧に加える
            $this->recognizedJsonIds[] = $jsonId;

            // metadata が無いならスキップ
            $metadata = $questionJson['metadata'] ?? [];
            if (empty($metadata)) {
                $this->warn("metadata が存在しないためスキップ: json_id={$jsonId}");
                DB::rollBack();
                return false;
            }

            // status 文字列を enum int 値へ
            $rawStatus = $questionJson['status'] ?? null;
            $statusValue = $rawStatus
                ? $this->parseQuestionStatus($rawStatus)
                : QuestionStatus::DRAFT->value;

            // 環境が local かつ status が TEST_PUBLISHED かつ validation_check が false の場合はバリデーションをスキップ
            $env = app()->environment();
            $shouldValidate = true;
            if (
                $env === 'local'
                && $statusValue === QuestionStatus::TEST_PUBLISHED->value
                && array_key_exists('validation_check', $questionJson)
                && $questionJson['validation_check'] === false
            ) {
                $this->info("Skipping validation for question json_id={$jsonId} because environment=local, status=TEST_PUBLISHED, and validation_check=false");
                $shouldValidate = false;
            }

            // JSONバリデーション
            if ($shouldValidate) {
                try {
                    $jsonManager = app(QuestionJsonManageService::class);
                    $jsonManager->validateQuestionJson($questionJson);
                } catch (\Illuminate\Validation\ValidationException $ve) {
                    $this->warn("バリデーションエラーのため問題(json_id={$jsonId})をスキップ:");
                    foreach ($ve->errors() as $field => $msgs) {
                        foreach ($msgs as $msg) {
                            $this->warn("  - {$field}: {$msg}");
                        }
                    }
                    DB::rollBack();
                    return false;
                }
            }

            // 既存レコードを検索 (json_id = $jsonId)
            $question = Question::where('json_id', $jsonId)->first();

            // order, version
            $jsonOrder = $questionJson['order'] ?? 9999;
            $version   = $questionJson['version'] ?? '0.0.1';

            // level_id / difficulty_id / grade_id
            $levelUuid      = $this->findLevelUuid($questionJson['level_id'] ?? null);
            $gradeUuid      = $this->findGradeUuid($questionJson['grade_id'] ?? null);
            $difficultyUuid = $this->findDifficultyUuid($questionJson['difficulty_id'] ?? null);

            // 新規or既存
            if (!$question) {
                $question = new Question();
                $question->uuid   = (string) Str::uuid();
                $question->json_id = $jsonId;
            }

            // 項目反映
            $question->order         = $jsonOrder;
            $question->level_id      = $levelUuid;
            $question->grade_id      = $gradeUuid;
            $question->difficulty_id = $difficultyUuid;
            $question->version       = $version;
            $question->status        = $statusValue;

            // question->metadata
            $question->metadata = json_encode($metadata, JSON_UNESCAPED_UNICODE);

            // question_type
            $rawQuestionType = $metadata['question_type'] ?? null;
            $question->question_type = $this->parseQuestionType($rawQuestionType);

            // evaluation_spec
            if (isset($questionJson['evaluation_spec'])) {
                $em = $questionJson['evaluation_spec']['evaluation_method'] ?? null;
                $cm = $questionJson['evaluation_spec']['checker_method'] ?? null;
                $question->evaluation_method = $em
                    ? EvaluationMethod::fromString($em)
                    : null;
                $question->checker_method = $cm
                    ? EvaluationCheckerMethod::fromString($cm)
                    : null;

                $question->llm_evaluation_prompt_file_name = $questionJson['evaluation_spec']['llm_prompt_file_name'] ?? null;

                $rf = $questionJson['evaluation_spec']['response_format'] ?? null;
                $question->evaluation_response_format = $rf
                    ? json_encode($rf, JSON_UNESCAPED_UNICODE)
                    : null;
            }

            // generated_by_llm
            if (array_key_exists('generated_by_llm', $questionJson)) {
                $question->generated_by_llm = (bool)$questionJson['generated_by_llm'];
            }

            // 学習要件
            $this->applyLearningRequirements($question, $questionJson);

            $question->save();

            // バリデーションをスキップした場合に記録しておく
            if (!$shouldValidate) {
                $this->skippedTestPublishedValidation[] = [
                    'id'       => $question->id,
                    'json_id'  => $question->json_id,
                ];
            }

            // question_text / explanation / background
            $this->upsertQuestionTranslations($question, $questionJson);

            // skills → pivot
            $this->applySkillsPivot($question, $questionJson);

            DB::commit();
            $this->info("Upserted question json_id={$jsonId}");
            return true;

        } catch (\Throwable $e) {
            DB::rollBack();
            $this->error("Error upserting question json_id={$jsonId}: " . $e->getMessage());
            return false;
        }
    }

    /**
     * JSON の "skills" 配列 → question_skill ピボットを同期
     */
    private function applySkillsPivot(Question $question, array $questionJson)
    {
        $skillsFromJson = $questionJson['skills'] ?? [];

        $newSkillDbIds = [];
        $skillIndexMap = [];

        foreach ($skillsFromJson as $index => $skillData) {
            $skillJsonId = $skillData['skill_id'] ?? null;
            if (!$skillJsonId) {
                continue;
            }

            // skills テーブルの json_id => id
            $skillDbId = DB::table('skills')
                ->where('json_id', $skillJsonId)
                ->value('id');

            if (!$skillDbId) {
                $this->warn("Skill with json_id='{$skillJsonId}' not found. Skipped linking.");
                continue;
            }

            $newSkillDbIds[] = $skillDbId;
            $skillIndexMap[$skillDbId] = $index + 1;
        }

        // 既存 pivot
        $existingSkillDbIds = DB::table('question_skill')
            ->where('question_id', $question->id)
            ->pluck('skill_id')
            ->toArray();

        // 削除すべきスキル
        $skillIdsToRemove = array_diff($existingSkillDbIds, $newSkillDbIds);
        if (!empty($skillIdsToRemove)) {
            DB::table('question_skill')
                ->where('question_id', $question->id)
                ->whereIn('skill_id', $skillIdsToRemove)
                ->delete();
        }

        // 追加または更新
        foreach ($newSkillDbIds as $skillDbId) {
            $existingPivot = DB::table('question_skill')
                ->where('question_id', $question->id)
                ->where('skill_id', $skillDbId)
                ->first();

            if ($existingPivot) {
                DB::table('question_skill')
                    ->where('id', $existingPivot->id)
                    ->update([
                        'order'      => $skillIndexMap[$skillDbId],
                        'updated_at' => now(),
                    ]);
            } else {
                DB::table('question_skill')->insert([
                    'uuid'        => (string) Str::uuid(),
                    'question_id' => $question->id,
                    'skill_id'    => $skillDbId,
                    'order'       => $skillIndexMap[$skillDbId],
                    'created_at'  => now(),
                    'updated_at'  => now(),
                ]);
            }
        }
    }

    /**
     * question_text, explanation, background を question_translations へ upsert
     */
    private function upsertQuestionTranslations(Question $question, array $questionJson)
    {
        $locales = ['ja','en'];

        foreach ($locales as $locale) {
            $qText = $questionJson['metadata']['question_text'][$locale] ?? null;
            $qExp  = $questionJson['metadata']['explanation'][$locale]   ?? null;
            $qBack = $questionJson['metadata']['background'][$locale]    ?? null;

            if ($qText !== null || $qExp !== null || $qBack !== null) {
                QuestionTranslation::updateOrCreate(
                    [
                        'question_id' => $question->id,
                        'locale'      => $locale,
                    ],
                    [
                        'question_text' => $qText,
                        'explanation'   => $qExp,
                        'background'    => $qBack,
                    ]
                );
            }
        }
    }

    /**
     * JSON の learning_requirements を反映
     */
    private function applyLearningRequirements(Question $question, array $questionJson)
    {
        $items = $questionJson['learning_requirements'] ?? [];
        if (!is_array($items) || empty($items)) {
            return;
        }
        $question->learning_requirement_json = json_encode($items, JSON_UNESCAPED_UNICODE);

        $subjects             = [];
        $nos                  = [];
        $requirements         = [];
        $requiredCompetencies = [];
        $backgrounds          = [];
        $categories           = [];
        $gradeLevels          = [];
        $urls                 = [];

        foreach ($items as $lr) {
            if (isset($lr['learning_subject'])) {
                $subjects[] = $lr['learning_subject'];
            }
            if (isset($lr['learning_no'])) {
                $nos[] = (string)$lr['learning_no'];
            }
            if (isset($lr['learning_requirement'])) {
                $requirements[] = $lr['learning_requirement'];
            }
            if (isset($lr['learning_required_competency'])) {
                $requiredCompetencies[] = $lr['learning_required_competency'];
            }
            if (isset($lr['learning_background'])) {
                $backgrounds[] = $lr['learning_background'];
            }
            if (isset($lr['learning_category'])) {
                $categories[] = $lr['learning_category'];
            }
            if (isset($lr['learning_grade_level'])) {
                $gradeLevels[] = $lr['learning_grade_level'];
            }
            if (isset($lr['learning_url'])) {
                $urls[] = $lr['learning_url'];
            }
        }

        $question->learning_subject             = implode("\n", $subjects);
        $question->learning_no                  = !empty($nos) ? (int)$nos[0] : null;
        $question->learning_requirement         = implode("\n", $requirements);
        $question->learning_required_competency = implode("\n", $requiredCompetencies);
        $question->learning_background          = implode("\n", $backgrounds);
        $question->learning_category            = implode("\n", $categories);
        $question->learning_grade_level         = implode("\n", $gradeLevels);
        $question->learning_url                 = implode("\n", $urls);
    }

    /**
     * JSON の level_id => levelsテーブル json_idカラム => uuid
     */
    private function findLevelUuid(?string $levelJsonId): ?string
    {
        if (!$levelJsonId) {
            return null;
        }
        return Level::where('json_id', $levelJsonId)->value('id') ?: null;
    }

    /**
     * JSON の grade_id => gradesテーブル json_idカラム => uuid
     */
    private function findGradeUuid(?string $gradeJsonId): ?string
    {
        if (!$gradeJsonId) {
            return null;
        }
        return Grade::where('json_id', $gradeJsonId)->value('id') ?: null;
    }

    /**
     * JSON の difficulty_id => difficultiesテーブル json_idカラム => uuid
     */
    private function findDifficultyUuid(?string $difficultyJsonId): ?string
    {
        if (!$difficultyJsonId) {
            return null;
        }
        return Difficulty::where('json_id', $difficultyJsonId)->value('id') ?: null;
    }

    /**
     * question_type 文字列 -> Enum (int値) に変換
     */
    private function parseQuestionType(?string $typeString): int
    {
        if (!$typeString) {
            // デフォルト
            return QuestionType::CALCULATION->value;
        }

        try {
            return QuestionType::fromString($typeString)->value;
        } catch (\InvalidArgumentException) {
            return QuestionType::CALCULATION->value;
        }
    }

    /**
     * status 文字列 -> QuestionStatus enum (int) に変換
     */
    private function parseQuestionStatus(string $statusString): int
    {
        try {
            return QuestionStatus::fromString($statusString)->value;
        } catch (\InvalidArgumentException) {
            return QuestionStatus::DRAFT->value;
        }
    }
}

```
