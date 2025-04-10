以下、ImportQuestionsFromGithub.php　に新しい処理を追加してください。
修正箇所を知らせてください。

必ず、既存のバリデーションルールは全てそのままにしてください。


# 追加する処理
処理後に questions のデータで、
status が PUBLISHED　以外のデータをコンソールに出力してください
コンソールには、json_id と、status を文字列で出力してください

コード内のコメントは消さずに全て残してください。

--- 問題JSONの全体構造
```json
{
  "order": 900,
  "id": "ques_s1_g3_sec100_u100_diff100_qt51_v100_900",
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
      "learning_no": 33,
      "learning_requirement": "A 数と計算 大きな数の概念と活用 万の単位、1億などの比べ方や表し方",
      "learning_required_competency": "万の単位や億の単位を使って大きな数を正しく読み書きし、合計や比較などの計算に活用できる",
      "learning_background": "日常生活で大きな数を扱う場面（人口やお金など）に対応しやすくするための基礎力として重要",
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
        "ja": "つぎの ▢ にあてはまる数を答えなさい。",
        "en": "Please answer the numbers that fit in the blanks."
      },
      "explanation": {
        "ja": "1億を3つ、千万を2つ、百万を5つ、十万を4つ、一万を6つ合わせた数を合計します。各単位の大きさに気をつけて足してみましょう。",
        "en": "We combine 3 sets of 100 million, 2 sets of 10 million, 5 sets of 1 million, 4 sets of 100 thousand, and 6 sets of 10 thousand. Be mindful of each place value and sum them correctly."
      },
      "question": {
        "ja": "1億を3つ、千万を2つ、百万を5つ、十万を4つ、一万を6つ合わせた数はいくつですか？",
        "en": "How many in total if you combine 3 sets of 100 million, 2 sets of 10 million, 5 sets of 1 million, 4 sets of 100 thousand, and 6 sets of 10 thousand?"
      },
      "fields": [
        {
          "field_id": "f_1",
          "user_answer": "number",
          "is_correct": "boolean",
          "collect_answer": {
            "ja": 325460000,
            "en": 325460000
          },
          "field_explanation": {
            "ja": "1億×3 + 1000万×2 + 100万×5 + 10万×4 + 1万×6 をすべて合計すると325460000になります。",
            "en": "Adding 3×100,000,000 + 2×10,000,000 + 5×1,000,000 + 4×100,000 + 6×10,000 equals 325,460,000."
          }
        }
      ]
    }
  },
  "metadata": {
    "question_type": "FILL_IN_THE_BLANK",
    "question_text": {
      "ja": "つぎの ▢ にあてはまる数を答えなさい。",
      "en": "Please answer the numbers that fit in the blanks."
    },
    "explanation": {
      "ja": "億や万などの大きな数の単位を正しくとらえて、それぞれの単位がいくつあるのかを考えてみましょう。大きな位から小さな位まで順序よく整理すると、正しく合計しやすくなります。",
      "en": "Focus on correctly understanding each large number unit, such as hundreds of millions or tens of thousands. Consider how many sets of each unit there are, and carefully add them from the largest place value to the smallest."
    },
    "question": {
      "ja": "1億を3つ、千万を2つ、百万を5つ、十万を4つ、一万を6つ合わせた数はいくつですか？",
      "en": "How many in total if you combine 3 sets of 100 million, 2 sets of 10 million, 5 sets of 1 million, 4 sets of 100 thousand, and 6 sets of 10 thousand?"
    },
    "background": {
      "ja": "この問題では、億や万の単位を使った大きな数を正しく足し算して、数のしくみを理解する力を身につけることをねらいとしています。",
      "en": "This problem aims to help you understand how to add large numbers using units like hundred million and ten thousand, reinforcing your grasp of place value."
    },
    "input_format": {
      "type": "fixed",
      "fields": [
        {
          "field_id": "f_1",
          "attribute": "number",
          "user_answer": "number"
        }
      ],
      "question_components": [
        {
          "type": "text",
          "content": {
            "ja": "1億を3つ、千万を2つ、百万を5つ、十万を4つ、一万を6つ合わせた数はいくつですか？",
            "en": "How many in total if you combine 3 sets of 100 million, 2 sets of 10 million, 5 sets of 1 million, 4 sets of 100 thousand, and 6 sets of 10 thousand?"
          },
          "order": 50
        },
        {
          "type": "newline",
          "order": 100
        },
        {
          "type": "input_field",
          "field_id": "f_1",
          "order": 150,
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

```php
<?php

namespace App\Enums;

enum QuestionStatus: int
{
    case DRAFT = 1;         // 下書き
    case PUBLISHED = 300;   // 公開中
    case HIDDEN = 600;      // 非公開
    case TEST_PUBLISHED = 900; // テスト公開
    case JSON_NOT_MATCHED = 99999; // 該当するJSONファイルが存在しなかった

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
     * 文字列から QuestionStatus を取得
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

```

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
     * ファイル名とidが不一致だったケースを保持し、最後にまとめて出力するためのリスト
     */
    private array $mismatchedFilenameAndIds = [];

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

        // ファイル名とidが不一致だったケースをまとめて出力
        if (!empty($this->mismatchedFilenameAndIds)) {
            $this->info("==== Filename/ID mismatch (skipped) ====");
            foreach ($this->mismatchedFilenameAndIds as $item) {
                $this->line("filename={$item['filename']}, json_id={$item['json_id']}");
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
                // 取得したファイルメタ情報からファイル名を取得
                $fileMetadata = $driveService->files->get($fileId, ['fields' => 'name']);
                $filename = $fileMetadata->getName();

                // alt=media で中身(JSON)を取得 (認証済ストリーム)
                $response = $driveService->files->get($fileId, ['alt' => 'media']);
                $jsonContent = $response->getBody()->getContents();

                $decoded = json_decode($jsonContent, true);
                if (!is_array($decoded)) {
                    $this->warn("File ID={$fileId} is not valid JSON. Skipped.");
                    $this->skipCount++;
                    continue;
                }

                // ファイル名とJSON内のidが不一致かをチェック
                $filenameNoExt = preg_replace('/\.json$/i', '', $filename);
                if (isset($decoded['id']) && $decoded['id'] !== $filenameNoExt) {
                    $this->warn("Filename and ID mismatch for file '{$filename}' => JSON ID='{$decoded['id']}'. Skipped.");
                    $this->mismatchedFilenameAndIds[] = [
                        'filename' => $filename,
                        'json_id'  => $decoded['id'],
                    ];
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

        // ファイル名とJSON内のidが不一致かをチェック
        $filenameNoExt = preg_replace('/\.json$/i', '', $file['name']);
        if (isset($decoded['id']) && $decoded['id'] !== $filenameNoExt) {
            $this->warn("Filename and ID mismatch for file '{$file['name']}' => JSON ID='{$decoded['id']}'. Skipped.");
            $this->mismatchedFilenameAndIds[] = [
                'filename' => $file['name'],
                'json_id'  => $decoded['id'],
            ];
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
