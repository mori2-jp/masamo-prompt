単元のJSONはGithubのプライベートリポジトリで管理しています。
github リポジトリからGithubAPIを利用してファイルを取得してDBにインサートするLaravel11のCommands\Import\Github\ImportQuestionSetsFromGithubというコマンドを実装し、
実行方法となるコマンドも出力してください


# 条件
1.
パスは、「contents/question_sets/{subject}/{level}/{unit}」です。
例：contents/question_sets/math/l2/u1/ques-s1-l2-001.json
例：contents/question_sets/math/l2/u2/ques-s1-l2-001.json

s1 は、subject ID が 1を表しています
l1 は、Level ID が 1を表しています
u1 は、Unit ID が1を表しています。

コマンドのオプションで subject を指定している時は subject 内のファイルのみ処理の対象とし、
subject の指定が無い場合は、questions/以下すべてを対象としたい

例 subject が math の場合、
例：contents/question_sets/math/l2/u1/ques-s1-l2-001.json
例：contents/question_sets/math/l2/u1/ques-s1-l2-001.json
これらのファイルが対象

例 subject が指定されていない場合、
contents/question_sets/math/l2/u1/ques-s1-l2-001.json
contents/question_sets/math/l2/u1/ques-s1-l2-001.json
contents/question_sets/math/l2/u1/ques-s1-l2-001.json
contents/question_sets/programming/l2/u1/ques-s1-l2-001.json
contents/question_sets/programming/l2/u1/ques-s1-l2-001.json
contents/question_sets/programming/l2/u1/ques-s1-l2-001.json
これら全てのファイルが対象。（subject と question_type はここで示したもの以外にも無数に存在します。

つまり、レスポンスが type = dir の時は再帰的に処理をして json ファイルを見つける必要があります。

2.
リポジトリは、
git@github.com:NousContentsManagement/masamo-content.git
です。

3.
order: 1、order: 2は、そのJSON内での順番を表しています。
order の順番どおり（小さい順）で処理を進めてください。
テーブル内部では、他の order も登録されており値の重複は避けたいです。
データを新規で作成する場合は、テーブル内で重複しないように一番大きい値を探してその次の順番をインサートしてください。
既にDBにインサートされているデータの順番が、JSONで定義された order の順番と一致しない場合は並び順を修正してください

例：
json
[id: 1, order: 1],[id: 2, order: 3],[id: 3, order: 2],[id: 4, order: 4],
DB
[id: 1, order: 59],[id: 2, order: 73],[id: 3, order: 75],[id: 4, order: 80],
↓修正後
[id: 1, order: 59],[id: 2, order: 75],[id: 3, order: 73],[id: 4, order: 80],

4.
JSON ID が同一のものが既にテーブルに存在する場合はデータを上書きしてください。

JSONの requirement と required_competency は、配列になっていますがDBにはそれぞれ改行して一つのカラムに入れてください。
なた、units, unit_translations の両方にカラムは存在しますが、units には、日本語をインサートして、unit_translations には言語ごとにインサートしてください

5.
github token は config/services.php github.api_token

6.
jsonの、difficulty_id　や　level_id , unit_id は、levels や difficulties , units の ID ではなく json_id のことを指しています。
これは、primary key である id は生成時に自動生成されるため予め定義されたJSONではIDを知り得ないため参照用のIDとしてjson_idを容易しています。
つまり、jsonのlevel_id から levels の json_id と対応するデータを探す必要があります（dificulties, units も同様に）


7.
status は、QuestionSetStatusを参照して正しく保存してください

8.
questions には、questions テーブルの,json_id が配列で入っています。
questions を探索して、該当したquestions の id を使って pivot を作成してください

9.
question_set_questions にはModelがありません。
Questions または QuestionSet にリレーションを定義するか何かする必要があります

ーーDB
```json
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
$table->text('description')->nullable();
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

```

--JSON
/contents/questions/math/FILL_IN_THE_BLANK/ques-s1-l2-001.json
```json
{
  "json_id": "qset_s1_l2_u1_001",
  "unit_id": "unit_s1_l1_001",
  "unit": {
    "ja": "掛け算の基礎",
    "en": "Basic Multiplication"
  },
  "title": {
    "ja": "掛け算問題セット１",
    "en": "Multiplication Problem Set 1"
  },
  "description": {
    "ja": "この問題セットでは掛け算の基礎を～",
    "en": "This set covers the fundamentals of multiplication..."
  },
  "order": 1,
  "version": "0.0.1",
  "status": "PUBLISHED",
  "questions": [
    "question_s1_l2_001",
    "question_s1_l2_002"
  ]
}

```


```
<?php

namespace App\Models\Question;

use App\Traits\HasOrder;
use App\Traits\UsesUuid;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class QuestionSet extends Model
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


--
<?php

namespace App\Models\Question;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class QuestionSetTranslation extends Model
{
    use SoftDeletes;
}

--
<?php

namespace App\Models\Level;

use App\Traits\HasOrder;
use App\Traits\UsesUuid;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Level extends Model
{
    use UsesUuid, HasOrder, SoftDeletes;

    // 主キーがUUIDであることを指定
    protected $keyType = 'string';
    public $incrementing = false;

    public $sortable = [
        'order_column_name' => 'order',
        'sort_when_creating' => true,
    ];
}

--
<?php

namespace App\Models\Difficulty;

use App\Traits\HasOrder;
use App\Traits\UsesUuid;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Difficulty extends Model
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


```json
<?php

namespace App\Enums;

enum QuestionFormat: int
{
    case MULTIPLE_CHOICE = 1;   // 選択式
    case NUMERIC_ANSWER   = 300;  // 数値回答式
    case TEXT_ANSWER      = 600;  // テキスト回答式

    public function label(): string
    {
        return match($this) {
            self::MULTIPLE_CHOICE => '選択式',
            self::NUMERIC_ANSWER  => '数値回答式',
            self::TEXT_ANSWER     => 'テキスト回答式',
        };
    }

    /**
     * 文字列を受け取り、該当する QuestionFormat を返す静的メソッド
     * 存在しない場合は例外を投げるサンプルです
     *
     * @param string $formatString "MULTIPLE_CHOICE", "NUMERIC_ANSWER", "TEXT_ANSWER"
     * @return QuestionFormat
     */
    public static function fromString(string $formatString): QuestionFormat
    {
        return match ($formatString) {
            'MULTIPLE_CHOICE' => self::MULTIPLE_CHOICE,
            'NUMERIC_ANSWER'  => self::NUMERIC_ANSWER,
            'TEXT_ANSWER'     => self::TEXT_ANSWER,
            default => throw new \InvalidArgumentException("Unknown QuestionFormat string: {$formatString}")
        };
    }
}



<?php

namespace App\Enums;

enum QuestionFormat: int
{
    case MULTIPLE_CHOICE = 1;   // 選択式
    case NUMERIC_ANSWER   = 300;  // 数値回答式
    case TEXT_ANSWER      = 600;  // テキスト回答式

    public function label(): string
    {
        return match($this) {
            self::MULTIPLE_CHOICE => '選択式',
            self::NUMERIC_ANSWER  => '数値回答式',
            self::TEXT_ANSWER     => 'テキスト回答式',
        };
    }

    /**
     * 文字列を受け取り、該当する QuestionFormat を返す静的メソッド
     * 存在しない場合は例外を投げるサンプルです
     *
     * @param string $formatString "MULTIPLE_CHOICE", "NUMERIC_ANSWER", "TEXT_ANSWER"
     * @return QuestionFormat
     */
    public static function fromString(string $formatString): QuestionFormat
    {
        return match ($formatString) {
            'MULTIPLE_CHOICE' => self::MULTIPLE_CHOICE,
            'NUMERIC_ANSWER'  => self::NUMERIC_ANSWER,
            'TEXT_ANSWER'     => self::TEXT_ANSWER,
            default => throw new \InvalidArgumentException("Unknown QuestionFormat string: {$formatString}")
        };
    }
}

<?php

namespace App\Enums;

// 問題のカテゴリ（出題形式とは別）
enum QuestionType: int
{
    case CALCULATION             = 1;
    case FILL_IN_THE_BLANK       = 51;
    case SCENARIO                = 101;
    case MULTIPLE_CHOICE         = 151;

//    case SIMPLE_ARITHMETIC        = 10;
//    case COMBINATION             = 101;
//    case HELL                    = 151;
//    case STORY                   = 201;
//    case FREE_TEXT               = 251;
//    case DRILL                   = 301;
//    case SIMULATION              = 401;
//    case DEBUG                   = 451;
//    case PROOF                   = 501;
//    case REASONING               = 551;
//    case MULTI_STEP_COMPARATIVE  = 601;
//    case FUSION                  = 651;
//    case INFERENCE               = 701;
//    case DATA_INTERPRETATION     = 751;
//    case LOGIC                   = 801;

    public function label(): string
    {
        return match($this) {
            self::CALCULATION            => 'Calculation Problem',              // 計算問題: 数値計算や演算ルールの処理を中心
            self::FILL_IN_THE_BLANK      => 'Fill-in-the-Blank Problem',        // 穴埋め問題: 問題文や式に空所があり、そこを埋める形式
            self::SCENARIO               => 'Scenario Problem',                 // シナリオ問題: 状況や経過を踏まえて解決する（回答が一意じゃない）
            self::MULTIPLE_CHOICE        => 'Multiple Choice Problem',          // 選択問題: 複数の選択肢から答えを選ぶ

//            self::SIMPLE_ARITHMETIC      => 'Simple Arithmetic Problem',        // 単純計算問題: 簡単な計算式や数値操作を問う
//            self::COMBINATION            => 'Combination Problem',              // 組み合わせ問題: 対応するペアやグループをマッチさせる
//            self::HELL                   => 'Hell Problem',                     // 地獄問題: 選択肢が多く1つだけ正解がある特許取得済の問題
//            self::STORY                  => 'Story Problem',                    // ストーリー問題: 物語やシナリオで流れを理解しながら解答
//            self::FREE_TEXT              => 'Free Text Problem',                // フリーテキスト問題: 自由記述形式での回答
//            self::DRILL                  => 'Drill Problem',                    // 発生練習問題: 何かを生成・作成したりする練習を含む
//            self::SIMULATION             => 'Simulation Problem (Real-life Modeling)', // シミュレーション問題: 実生活モデルを扱う
//            self::DEBUG                  => 'Debug Problem',                    // デバッグ問題: ミスを発見し修正する
//            self::PROOF                  => 'Proof Problem',                    // 証明問題: 定理や結論がなぜ成り立つかを示す
//            self::REASONING              => 'Reasoning Problem',                // 理由づけ問題: 結果の根拠や理由を説明
//            self::MULTI_STEP_COMPARATIVE => 'Multi-Step Comparative Problem',   // 複数手順を比較検討する問題
//            self::FUSION                 => 'Fusion Problem',                   // 融合問題: 複数領域を組み合わせて解決
//            self::INFERENCE              => 'Inference Problem',                // 推理問題: 与えられた情報から論理的に結論を導く
//            self::DATA_INTERPRETATION    => 'Data Interpretation Problem',      // 統計資料の読み取り問題: グラフや表を使う
//            self::LOGIC                  => 'Logic Problem',                    // ロジック問題: プログラム的思考やアルゴリズム判断
        };
    }

    /**
     * 文字列を受け取り、該当する QuestionType を返す静的メソッド
     * 存在しない場合は例外を投げるサンプルです
     *
     * @param string $typeString "FILL_IN_THE_BLANK" など
     * @return QuestionType
     */
    public static function fromString(string $typeString): QuestionType
    {
        return match($typeString) {
            'CALCULATION'       => self::CALCULATION,
            'FILL_IN_THE_BLANK' => self::FILL_IN_THE_BLANK,
            'SCENARIO'          => self::SCENARIO,
            'MULTIPLE_CHOICE'   => self::MULTIPLE_CHOICE,
            default => throw new \InvalidArgumentException("Unknown QuestionType string: {$typeString}")
        };
    }
}

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

```

``` 参考コマンド
<?php

namespace App\Console\Commands\Import\Github;

use App\Models\Level\Level;
use App\Models\Subject\Subject;
use App\Models\Unit\Unit;
use App\Models\Unit\UnitTranslation;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Str;

/**
 * Import units JSON from a private GitHub repository
 * and upsert into units / unit_translations.
 */
class ImportUnitsFromGithub extends Command
{
    protected $signature = 'import:units-from-github
                            {--subject= : If specified, only import files under contents/units/{subject} }';

    protected $description = 'Import units data from a GitHub repository JSON files and sync DB';

    public function handle()
    {
        // 1. 認証トークンをconfigから取得
        $token = config('services.github.api_token');
        if (!$token) {
            $this->error('GitHub API token not found in config/services.php [github.api_token]');
            return 1;
        }

        // 2. リポジトリとパスの指定
        $repoOwnerAndName = 'NousContentsManagement/masamo-content';
        $basePath = 'contents/units';

        $subjectOption = $this->option('subject');
        if ($subjectOption) {
            // e.g. contents/units/math
            $basePath .= '/' . $subjectOption;
            $this->info("Subject specified: {$subjectOption}, path= {$basePath}");
        } else {
            $this->info("No subject specified. Will import from all subdirectories under contents/units");
        }

        // 3. 再帰的にGitHub APIでファイル一覧(JSONファイル)を収集
        $allJsonFiles = $this->fetchAllJsonFilesRecursively($repoOwnerAndName, $basePath, $token);

        if (empty($allJsonFiles)) {
            $this->warn("No JSON files found under path: {$basePath}");
            return 0;
        }

        $this->info("Found " . count($allJsonFiles) . " JSON file(s) in total under path: {$basePath}");

        // 4. 各JSONファイルを処理
        foreach ($allJsonFiles as $file) {
            $this->processJsonFile($file, $token);
        }

        $this->info("Import process completed.");
        return 0;
    }

    /**
     * 再帰的にディレクトリをたどり、.jsonファイルを収集する
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
            // ディレクトリなら再帰的に取得
            if ($type === 'dir') {
                // 再帰的に下層ディレクトリを探索
                $descendantFiles = $this->fetchAllJsonFilesRecursively($repo, $itemPath, $token);
                $result = array_merge($result, $descendantFiles);
            } elseif ($type === 'file') {
                // .json ファイルかチェック
                // 例: 末尾が .json なら対象に加える
                if (str_ends_with($name, '.json')) {
                    // 取得候補に追加
                    $result[] = $item;
                }
            }
        }

        return $result;
    }

    /**
     * 単一のJSONファイルをダウンロードし、"units"配列をDBに反映
     */
    private function processJsonFile(array $file, string $token)
    {
        if (!isset($file['download_url'])) {
            $this->warn("No download_url for file: " . ($file['path'] ?? 'unknown'));
            return;
        }
        $this->info("Fetching JSON from: " . $file['path']);

        // JSONファイル内容を取得
        $jsonContent = Http::withToken($token)
            ->get($file['download_url'])
            ->body();

        $decoded = json_decode($jsonContent, true);
        if (!is_array($decoded) || !isset($decoded['units'])) {
            $this->warn("File {$file['name']} does not contain 'units' array. Skipped.");
            return;
        }

        // "units" 配列を処理
        $unitsArray = $decoded['units'];
        // JSONのorder順にソートしてから処理する
        usort($unitsArray, fn($a, $b) => ($a['order'] ?? 9999) <=> ($b['order'] ?? 9999));

        foreach ($unitsArray as $unitItem) {
            $this->upsertUnit($unitItem);
        }
    }

    /**
     * JSONデータ1件(ユニット)をDBへupsert
     */
    private function upsertUnit(array $unitItem)
    {
        DB::beginTransaction();
        try {
            $jsonId = $unitItem['json_id'] ?? null;
            if (!$jsonId) {
                $this->warn("unit JSON missing 'json_id'. Skipped.");
                DB::rollBack();
                return;
            }

            $unitRecord = Unit::where('json_id', $jsonId)->first();

            // subject_id, level_id を解決
            $subjectUuid = $this->findSubjectUuid($unitItem['subject_id'] ?? null);
            $levelUuid   = $this->findLevelUuid($unitItem['level_id']   ?? null);

            // order の重複回避
            $jsonOrder = $unitItem['order'] ?? 9999;
            if (!$unitRecord) {
                // 新規
                $unitRecord = new Unit();
                $unitRecord->id = (string) Str::uuid();
                $unitRecord->json_id = $jsonId;

                $maxOrder = Unit::max('order');
                if ($maxOrder === null) {
                    $maxOrder = 0;
                }
                // 例えば maxOrder+1 or JSON指定 order が大きいか比較
                // ここでは例として、衝突時に +1 して空きを作る
                $finalOrder = max($maxOrder+1, $jsonOrder);
                while (Unit::where('order', $finalOrder)->exists()) {
                    $finalOrder++;
                }
                $unitRecord->order = $finalOrder;
            } else {
                // 既存レコードのorder更新
                $this->reorderUnit($unitRecord->id, $jsonOrder);
            }

            $unitRecord->subject_id   = $subjectUuid;
            $unitRecord->level_id     = $levelUuid;

            // unitsテーブルには日本語名を入れる（要件より）
            $jaName = $unitItem['name']['ja'] ?? null;
            $jaDesc = $unitItem['description']['ja'] ?? null;
            $unitRecord->name        = $jaName;
            $unitRecord->description = $jaDesc;
            $unitRecord->version     = $unitItem['version'] ?? '0.0.1';

            $unitRecord->save();

            // unit_translations 多言語
            foreach (['ja','en'] as $locale) {
                if (isset($unitItem['name'][$locale]) || isset($unitItem['description'][$locale])) {
                    UnitTranslation::updateOrCreate(
                        [
                            'unit_id' => $unitRecord->id,
                            'locale'  => $locale,
                        ],
                        [
                            'name'        => $unitItem['name'][$locale] ?? null,
                            'description' => $unitItem['description'][$locale] ?? null,
                        ]
                    );
                }
            }

            DB::commit();
            $this->info("Upserted unit json_id={$jsonId}");
        } catch (\Throwable $e) {
            DB::rollBack();
            $this->error("Error upserting unit json_id={$jsonId}: " . $e->getMessage());
        }
    }

    /**
     * DB内の既存unitの並び順(order)を更新する際に、JSONで指定された順序に合わせつつ衝突を回避
     */
    private function reorderUnit(string $unitId, int $targetOrder)
    {
        $unit = Unit::find($unitId);
        if (!$unit) return;

        // 順序衝突を回避 (簡易例)
        while (Unit::where('order', $targetOrder)->where('id','!=',$unitId)->exists()) {
            $targetOrder++;
        }
        $unit->order = $targetOrder;
        $unit->save();
    }

    /**
     * Subject ID解決用
     */
    private function findSubjectUuid(?string $subJsonId)
    {
        if (!$subJsonId) return null;
         return Subject::where('json_id',$subJsonId)->value('id') ?: null;
    }

    /**
     * Level ID解決用
     */
    private function findLevelUuid(?string $levJsonId)
    {
        if (!$levJsonId) return null;
        return Level::where('json_id',$levJsonId)->value('id') ?: null;
    }
}
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
            $table->uuid('difficulty_id');
            $table->string('json_id')->nullable()->unique();
            $table->json('metadata')->nullable();
            $table->string('version')->default('0.0.1');
            $table->integer('status')->default(1);
            $table->integer('question_type')->default(1);
            $table->string('learning_subject')->nullable()
                ->comment('科目 (学習要件) e.g. "Arithmetic"');
            $table->integer('learning_no')->nullable()
                ->comment('学習要件の番号 e.g. 10');
            $table->text('learning_requirement')->nullable()
                ->comment('学習要件の内容 "Numbers and Calculation..."');
            $table->text('learning_required_competency')->nullable()
                ->comment('必要水準 "Understand multiplication..."');
            $table->string('learning_category')->nullable()
                ->comment('分類 e.g. "A", "B"');
            $table->string('learning_grade_level')->nullable()
                ->comment('学年 e.g. "Grade 2"');
            $table->string('learning_url')->nullable()
                ->comment('URLリンク e.g. "https://docs.google.com/..."');
            $table->integer('order');
            $table->boolean('generated_by_llm')->default(false);
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('level_id')->references('id')->on('levels')->onDelete('cascade');
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
