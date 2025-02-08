単元のJSONはGithubのプライベートリポジトリで管理しています。
github リポジトリからGithubAPIを利用してファイルを取得してDBにインサートするLaravel11のコマンドを実装してください。

# 条件
1.
パスは、「contents/units/{subject}」です。
例：contents/units/math/units-s1-l3-001.json
例：contents/units/math/units-s1-l2-001.json

s1 は、subject ID が 1を表しています
l1 は、Level ID が 1を表しています

コマンドのオプションで subject を指定している時は subject 内のファイルのみ処理の対象とし、
subject の指定が無い場合は、units/以下すべてを対象としたい

例 subject が math の場合、
contents/units/math/units-s1-l3-001.json
contents/units/math/units-s1-l3-002.json
これらのファイルが対象

例 subject が指定されていない場合、
contents/units/math/units-s1-l3-001.json
contents/units/logic/units-s2-l3-001.json
contents/units/logic/units-s2-l3-002.json
contents/units/sience/units-s3-l3-001.json
contents/units/english/units-s4-l3-001.json
これら全てのファイルが対象。（subject はここで示したもの以外にも無数に存在します。

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
