単元のJSONはGithubのプライベートリポジトリで管理しています。
github リポジトリからGithubAPIを利用してファイルを取得してDBにインサートするLaravel11のImportUnitsFromGithubというコマンドを実装し、
実行方法となるコマンドも出力してください

# 条件
1.
パスは、「contents/units/{version}/{subject}」です。
例：contents/units/v1.0.0/math/units-s1-l3-001.json
例：contents/units/v1.0.0/math/units-s1-l2-001.json

versionは、v1.0.0 などのバージョンの文字列が入っています
s1 は、subject ID が 1を表しています
l1 は、Level ID が 1を表しています

コマンドのオプションで subject を指定している時は subject 内のファイルのみ処理の対象とし、
subject の指定が無い場合は、units/以下すべてを対象としたい

例 subject が math の場合、
contents/units/v1.0.0/math/units-s1-l3-001.json
contents/units/v1.0.0/math/units-s1-l3-002.json
これらのファイルが対象

例 subject が指定されていない場合、
contents/units/v1.0.0/math/units-s1-l3-001.json
contents/units/v1.0.0/logic/units-s2-l3-001.json
contents/units/v1.0.0/logic/units-s2-l3-002.json
contents/units/v1.0.0/sience/units-s3-l3-001.json
contents/units/v1.0.0/english/units-s4-l3-001.json
これら全てのファイルが対象。（subject はここで示したもの以外にも無数に存在します。

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

jsonの、difficulty_id　や　level_id , unit_id , subject_id は、levels や difficulties , units, subjects の ID ではなく json_id のことを指しています。
これは、primary key である id は生成時に自動生成されるため予め定義されたJSONではIDを知り得ないため参照用のIDとしてjson_idを容易しています。
つまり、jsonのlevel_id から levels の json_id と対応するデータを探す必要があります（dificulties, units , subjectsも同様に）

ーーDB
```json
        Schema::create('units', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('subject_id');
            $table->uuid('level_id');
            $table->string('json_id')->nullable()->unique();
            $table->string('name')->nullable();
            $table->text('description')->nullable();
            $table->string('version')->default('0.0.1');
            $table->integer('order');
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('subject_id')->references('id')->on('subjects')->onDelete('cascade');
            $table->foreign('level_id')->references('id')->on('levels')->onDelete('cascade');
        });
    }

Schema::create('unit_translations', function (Blueprint $table) {
$table->id();
$table->uuid('unit_id');
$table->string('locale', 10);
$table->string('name')->nullable();
$table->text('description')->nullable();
$table->timestamps();
$table->softDeletes();

$table->foreign('unit_id')->references('id')->on('units')->onDelete('cascade');
});
```

--JSON
```json
{
  "status": "PUBLISHED",
  "units": [
    {
      "order": 1,
      "json_id": "unit-s1-l3-001",
      "subject_id": "sub_001",
      "level_id": "lev_003",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "掛け算の基礎",
        "en": "Basic Multiplication"
      },
      "description": {
        "ja": "この単元では掛け算の基本を学びます。",
        "en": "This unit covers the basics of multiplication."
      },
      "version": "0.0.1"
    },
    {
      "order": 2,
      "json_id": "unit-s1-l3-002",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "割り算の基礎",
        "en": "Basic Division"
      },
      "description": {
        "ja": "この単元では割り算の基本を学びます。",
        "en": "This unit covers the basics of division."
      },
      "version": "0.0.1"
    }
  ]
}

```

```json
<?php

namespace App\Models\Unit;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class UnitTranslation extends Model
{
    use SoftDeletes;
}

```

```json
<?php

namespace App\Models\Unit;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class UnitTranslation extends Model
{
    use SoftDeletes;
}

```

```json
<?php

namespace App\Models\Skill;

use App\Traits\HasOrder;
use App\Traits\UsesUuid;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Skill extends Model
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
```

```json
<?php

namespace App\Models\Skill;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class SkillTranslation extends Model
{
    use SoftDeletes;
}

<?php

namespace App\Enums;

enum UnitStatus: int
{
case DRAFT = 1;         // 下書き
case PUBLISHED = 300;   // 公開中
case HIDDEN = 600;      // 非公開
case TEST_PUBLISHED = 900; // テスト公開

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
default => throw new \InvalidArgumentException("Unknown status string: {$statusString}")
};
}
}

```

```
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
 *
 * 対象パス: contents/units/{version}/{subject}/...
 * 例: contents/units/v1.0.0/math/units-s1-l3-001.json
 */
class ImportUnitsFromGithub extends Command
{
    /**
     * artisan で呼び出すコマンド名。
     * 例:
     *   php artisan import:units-from-github --subject=math
     */
    protected $signature = 'import:units-from-github
                            {--subject= : If specified, only import files under contents/units/*/{subject} }';

    protected $description = 'Import units data from a GitHub repository JSON files and sync to DB';

    /**
     * コマンドハンドラ
     */
    public function handle()
    {
        // 1. GitHub API token を config から取得
        $token = config('services.github.api_token');
        if (!$token) {
            $this->error('GitHub API token not found in config/services.php [github.api_token]');
            return 1; // エラー終了
        }

        // 2. リポジトリ/パス指定
        $repoOwnerAndName = 'NousContentsManagement/masamo-content';

        // ベースパスは contents/units
        $basePath = 'contents/units';

        // subject が指定されていればパス末尾に追加
        // ただし、version (v1.0.0等) のディレクトリも含まれるため、再帰的探索が必要
        $subjectOption = $this->option('subject');
        if ($subjectOption) {
            // 例: contents/units/*/math
            // versionディレクトリが複数ある場合を想定して、全バージョン下の {subject} を探す
            $basePath .= '/*/' . $subjectOption;
            $this->info("Subject specified: {$subjectOption}, path= {$basePath}");
        } else {
            // subject 指定なし → contents/units/配下すべて
            $this->info("No subject specified. Will import from all subdirectories under contents/units");
        }

        // 3. ディレクトリを再帰的にたどり .json ファイル一覧を取得
        $allJsonFiles = $this->fetchAllJsonFilesRecursively($repoOwnerAndName, $basePath, $token);

        if (empty($allJsonFiles)) {
            $this->warn("No JSON files found under path: {$basePath}");
            return 0;
        }

        $this->info("Found " . count($allJsonFiles) . " JSON file(s) under {$basePath}");

        // 4. 各 JSON ファイルを処理して DB に upsert
        foreach ($allJsonFiles as $file) {
            $this->processJsonFile($file, $token);
        }

        $this->info("Import process completed.");
        return 0;
    }

    /**
     * 再帰的にディレクトリを走査し、.jsonファイルを集める
     */
    private function fetchAllJsonFilesRecursively(string $repo, string $path, string $token): array
    {
        // GitHub API のエンドポイント
        // '*' を含むパスは GitHub の Contents API では直接取得できないため、
        // 下層をさらに手動で再帰検索する必要がある。
        // 例: contents/units/*/math というパスはワイルドカードを使えないため、
        //     まず contents/units の下にあるディレクトリ一覧を取得 → v1.0.0 とか順番にたどる
        // ここでは雑に対応するため、 path にワイルドカードが含まれるかどうかで分岐する。
        if (str_contains($path, '*')) {
            // 例えば contents/units/*/math
            //  → contents/units の下の各ディレクトリを再帰的に処理
            //  → その下のサブディレクトリが math かどうかチェック

            return $this->fetchWildcardPathFiles($repo, $path, $token);
        }

        // '*' を含まない場合はダイレクトにリクエスト
        $url = "https://api.github.com/repos/{$repo}/contents/{$path}";
        $response = Http::withToken($token)->get($url);

        // 失敗なら空配列
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
            $type = $item['type'] ?? null; // 'file' or 'dir'
            $itemPath = $item['path'] ?? '';
            $name     = $item['name'] ?? '';

            if ($type === 'dir') {
                // ディレクトリ → 再帰
                $descendants = $this->fetchAllJsonFilesRecursively($repo, $itemPath, $token);
                $result = array_merge($result, $descendants);
            } elseif ($type === 'file') {
                // .json なら対象
                if (str_ends_with($name, '.json')) {
                    $result[] = $item;
                }
            }
        }

        return $result;
    }

    /**
     * pathにワイルドカード('*')が含まれる場合の特殊処理。
     * 例: path= contents/units/{version}/math
    */
    private function fetchWildcardPathFiles(string $repo, string $path, string $token): array
    {
        // 簡易的に実装: pathを '/' で分割し、'*' の部分で分岐
        // 例: ['contents','units','*','math']
        //     1) 'contents/units'以下のディレクトリ一覧を取得
        //     2) matchするパターンを再帰
        $parts = explode('/', $path);
        $wildcardIndex = array_search('*', $parts);

        if ($wildcardIndex === false) {
            // 万一 * がない場合は通常処理へ
            return $this->fetchAllJsonFilesRecursively($repo, $path, $token);
        }

        // 例:
        // prefix = "contents/units"
        // suffix = "math"
        $prefix = implode('/', array_slice($parts, 0, $wildcardIndex));
        $suffix = implode('/', array_slice($parts, $wildcardIndex + 1));

        // prefix 下のファイル/ディレクトリ一覧を取得
        $url = "https://api.github.com/repos/{$repo}/contents/{$prefix}";
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
            if (($item['type'] ?? null) === 'dir') {
                // 例: "contents/units/v1.0.0"
                // その下に suffix (math) ディレクトリがあるか再帰で探す
                $dirPath = $item['path'] . '/' . $suffix;  // e.g. contents/units/v1.0.0/math
                $descendants = $this->fetchAllJsonFilesRecursively($repo, $dirPath, $token);
                $result = array_merge($result, $descendants);
            }
        }
        return $result;
    }

    /**
     * 単一のJSONファイルをダウンロードし、"units"配列を読み込み DB に反映
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

        // JSON の order 順でソートしてから処理
        usort($unitsArray, fn($a, $b) => ($a['order'] ?? 9999) <=> ($b['order'] ?? 9999));

        foreach ($unitsArray as $unitItem) {
            $this->upsertUnit($unitItem);
        }
    }

    /**
     * JSONデータ1件(ユニット)を DB へ upsert
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

            // 既存レコードを検索
            $unitRecord = Unit::where('json_id', $jsonId)->first();

            // subject_id, level_id 解決
            $subjectUuid = $this->findSubjectUuid($unitItem['subject_id'] ?? null);
            $levelUuid   = $this->findLevelUuid($unitItem['level_id'] ?? null);

            // order の衝突回避
            $jsonOrder = $unitItem['order'] ?? 9999;

            if (!$unitRecord) {
                // 新規
                $unitRecord = new Unit();
                $unitRecord->id      = (string) Str::uuid();
                $unitRecord->json_id = $jsonId;

                // 既存の最大 order
                $maxOrder = Unit::max('order');
                if (is_null($maxOrder)) {
                    $maxOrder = 0;
                }

                $finalOrder = max($maxOrder + 1, $jsonOrder);

                // 衝突回避
                while (Unit::where('order', $finalOrder)->exists()) {
                    $finalOrder++;
                }
                $unitRecord->order = $finalOrder;
            } else {
                // 既存 → order 更新
                $this->reorderUnit($unitRecord->id, $jsonOrder);
            }

            $unitRecord->subject_id   = $subjectUuid;
            $unitRecord->level_id     = $levelUuid;
            // 日本語版を unitsテーブルの name/description に
            $jaName = $unitItem['name']['ja'] ?? null;
            $jaDesc = $unitItem['description']['ja'] ?? null;

            $unitRecord->name        = $jaName;
            $unitRecord->description = $jaDesc;
            $unitRecord->version     = $unitItem['version'] ?? '0.0.1';

            $unitRecord->save();

            // unit_translations (多言語)
            $this->upsertUnitTranslations($unitRecord, $unitItem);

            DB::commit();
            $this->info("Upserted unit json_id={$jsonId}");
        } catch (\Throwable $e) {
            DB::rollBack();
            $this->error("Error upserting unit json_id={$jsonId}: " . $e->getMessage());
        }
    }

    /**
     * 既存ユニットの並び順(order)を JSONのorderに合わせ、衝突を回避
     */
    private function reorderUnit(string $unitId, int $targetOrder)
    {
        $unit = Unit::find($unitId);
        if (!$unit) {
            return;
        }

        while (Unit::where('order', $targetOrder)->where('id','!=',$unitId)->exists()) {
            $targetOrder++;
        }
        $unit->order = $targetOrder;
        $unit->save();
    }

    /**
     * unit_translations に ja/en の name/description を upsert
     */
    private function upsertUnitTranslations(Unit $unitRecord, array $unitItem)
    {
        foreach (['ja','en'] as $locale) {
            $localeName        = $unitItem['name'][$locale]        ?? null;
            $localeDescription = $unitItem['description'][$locale] ?? null;

            if ($localeName !== null || $localeDescription !== null) {
                UnitTranslation::updateOrCreate(
                    [
                        'unit_id' => $unitRecord->id,
                        'locale'  => $locale,
                    ],
                    [
                        'name'        => $localeName,
                        'description' => $localeDescription,
                    ]
                );
            }
        }
    }

    /**
     * Subject ID解決
     */
    private function findSubjectUuid(?string $subjectJsonId): ?string
    {
        if (!$subjectJsonId) {
            return null;
        }
        return Subject::where('json_id', $subjectJsonId)->value('id') ?: null;
    }

    /**
     * Level ID解決
     */
    private function findLevelUuid(?string $levelJsonId): ?string
    {
        if (!$levelJsonId) {
            return null;
        }
        return Level::where('json_id', $levelJsonId)->value('id') ?: null;
    }
}


```
