ありがとう。
問題のデータはJSONとしてリポジトリで管理しようと思っています。

問題のセットについてもJSONで管理して、LaravelのコマンドでDBに取り込みをしたいです。
フォーマットを考えてください。

## 条件
- 多言語に対応すること
- 単元ごとに問題セットの管理をします
- 問題セットの全体は一つのjsonで管理します
- DBのIDは保存時に生成されるのであらかじめJSONで定義することは出来ません。jsonの管理用のIDを別途追加してください（json_id）
- problem_set_id は、pset_s1(subjectIdの省略）_l1（levelIdの省略)_u1(unitIdの省略)_001 のようなナンバリングにしてください
```　units
        Schema::create('problem_sets', function (Blueprint $table) {
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
        
                Schema::create('problem_set_translations', function (Blueprint $table) {
            $table->id();
            $table->uuid('problem_set_id');
            $table->string('locale', 10);
            $table->string('title')->nullable();
            $table->text('description')->nullable();
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('problem_set_id')->references('id')->on('problem_sets')->onDelete('cascade');
        });
```



ーーーー
ありがとう。ProblemSetの仕様を以下を参考に仕様書にしていただけますか？

ーーー
# JSON全体構造の解説 (Units 用, レベル対応・命名規則付き)

## 単元の管理方針

- **単元データ**は 1つの JSON ファイルにまとめて定義します。
- 各単元ごとに **`json_id`** を振り、単元が属する **科目（`subject_id`）** と **レベル（`level_id`）** を紐づけます。
- **多言語化** される文字列（科目名、レベル名、単元名、単元説明など）は、言語コードをキーにしたオブジェクトで管理します。
- **命名規則（例）:** `json_id` に **`units_s1_l1_001`** のような形式を使い、
    - `s1` → subjects の略称やID
    - `l1` → levels の略称やID
    - `_001` → 通し番号  
      といったパターンで統一すると、単元を管理しやすくなります。

---

## JSON例

```json5
{
  "units": [
    {
      "json_id": "units_s1_l1_001",
      "subject_id": "sub-001",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "level_id": "lev_001",
      "level": {
        "ja": "算数小３",
        "en": "Math Grade 3"
      },
      "name": {
        "ja": "掛け算の基礎",
        "en": "Basic Multiplication"
      },
      "description": {
        "ja": "この単元では掛け算の基礎を学びます。",
        "en": "This unit covers the basics of multiplication."
      },
      "order": 1,
      "version": "0.0.1"
    },
    {
      "json_id": "units_s1_lX_002",
      "subject_id": "sub-001",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "level_id": null,
      "level": {
        "ja": "",
        "en": ""
      },
      "name": {
        "ja": "割り算の基礎",
        "en": "Basic Division"
      },
      "description": {
        "ja": "この単元では割り算の基礎を学びます。",
        "en": "This unit covers the basics of division."
      },
      "order": 2,
      "version": "0.0.1"
    }
  ]
}
```

### 命名例：`units_s1_l1_001`
- `units_` : 「単元」を表すプリフィクス
- `s1` : 科目`s1` (例：算数)
- `l1` : レベル`s1` (例：lev_001)
- `_001` : 通し番号など

---

## 各フィールドの解説

### 1. `json_id` (string)

- **説明:**  
  リポジトリ上で管理する単元データの一意識別子。  
  DB取り込み時、`units` テーブルの `json_id` カラムと照合して更新・新規作成を判定。

- **命名規則例:**  
  `units_s1_l1_001` など
    - `s1` = subject の簡易ID
    - `l1` = level の簡易ID
    - `_001` = 通し番号

- **例:** `"units_s1_l1_001"`, `"units_s2_l2_005"`

---

### 2. `subject_id` (string)

- **説明:**  
  この単元が属する科目の JSON 管理用ID。  
  インポート時、 `subjects` テーブルで `json_id = subject_id` を検索して実際の UUID を取得し、`units.subject_id` にセットする。

- **用途:**  
  科目別に単元を分類する際の連携キー。

- **例:** `"sub-001"`

---

### 3. `subject` (object)

- **説明:**  
  JSON 閲覧時、人間がどの科目か分かりやすくするための多言語フィールド。DBには書き込まない。

- **例:**
  ```json5
  "subject": {
    "ja": "算数",
    "en": "Mathematics"
  }
  ```

---

### 4. `level_id` (string)

- **説明:**  
  この単元が属するレベル（levels テーブル）の JSON 管理用ID。  
  インポート時、 `levels` テーブルで `json_id = level_id` を検索し、その UUID を `units.level_id` にセット。

- **用途:**  
  レベル別学習を実現する際に「どのレベルの単元か」を特定する。

- **例:** `"lev_001"`, `"lev_002"`, or `null`

---

### 5. `level` (object)

- **説明:**  
  上記 `level_id` だけでは分かりづらいので、レベル名を多言語化して補足。DB挿入しない参照用。

- **例:**
  ```json5
  "level": {
    "ja": "算数小３",
    "en": "Math Grade 3"
  }
  ```

---

### 6. `name` (object)

- **説明:**  
  単元の名称を多言語対応で保持するオブジェクト。

- **用途:**  
  ユーザー・管理画面で単元名を表示。

- **例:**
  ```json5
  "name": {
    "ja": "掛け算の基礎",
    "en": "Basic Multiplication"
  }
  ```

---

### 7. `description` (object)

- **説明:**  
  単元の説明や補足を多言語化オブジェクトで保持。

- **用途:**  
  学習内容の概要をユーザーに提示する。

- **例:**
  ```json5
  "description": {
    "ja": "掛け算の基礎を学びます。",
    "en": "This unit covers the basics of multiplication."
  }
  ```

---

### 8. `order` (number)

- **説明:**  
  同一科目内での単元の表示・学習順を表す数値。

- **例:** `1`, `2`, `10` など

---

### 9. `version` (string)

- **説明:**  
  単元データのバージョン管理用。 `"0.0.1"` 等の初期値が多い。

- **用途:**  
  単元データを更新する際のリビジョン管理。

- **例:** `"0.0.1"`

---

## サンプル JSON

```json5
{
  "units": [
    {
      "json_id": "units_s1_l1_001",
      "subject_id": "sub-001",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "level_id": "lev_001",
      "level": {
        "ja": "算数小３",
        "en": "Math Grade 3"
      },
      "name": {
        "ja": "掛け算の基礎",
        "en": "Basic Multiplication"
      },
      "description": {
        "ja": "この単元では掛け算の基礎を学びます。",
        "en": "This unit covers the basics of multiplication."
      },
      "order": 1,
      "version": "0.0.1"
    },
    {
      "json_id": "units_s1_lX_002",
      "subject_id": "sub-001",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "level_id": null,
      "level": {
        "ja": "",
        "en": ""
      },
      "name": {
        "ja": "割り算の基礎",
        "en": "Basic Division"
      },
      "description": {
        "ja": "この単元では割り算の基礎を学びます。",
        "en": "This unit covers the basics of division."
      },
      "order": 2,
      "version": "0.0.1"
    }
  ]
}
```

