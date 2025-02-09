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

```
