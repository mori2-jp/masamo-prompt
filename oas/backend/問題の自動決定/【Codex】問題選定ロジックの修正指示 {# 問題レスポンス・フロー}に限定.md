以下の仕様書をもう少し具体的に追加したい。

### 2‑1. レスポンス優先順位
に


ーーー
# 問題レスポンス仕様書

最終更新: 2025‑04‑26  
担当: backend team

---

## 1. 用語と構造

| 用語                | 説明                                                                       |
| ----------------- | ------------------------------------------------------------------------ |
| **grade**         | 学年。ユーザーが属する学習ステージを示す。                                                    |
| **section**       | 問題の大カテゴリ。`grade_id` で **grade** と 1 \:N。例: *足し算*。                        |
| **unit**          | **section** をさらに細分化したカテゴリ。`section_id` で **section** と 1 \:N。例: *２桁＋１桁*。 |
| **question\_set** | 問題セット。`unit_id` で **unit** と 1 \:N。                                      |

```
Grade ── 1:N ── Section ── 1:N ── Unit ── 1:N ── QuestionSet
```


---

## 2. 問題レスポンス・フロー
### 2‑1. レスポンス優先順位
1. **途中再開**
   ユーザーの現在のGrade（`users.grade_id`）に紐づく **全てのSections → それらに紐づくUnits → それらに紐づくQuestionSet** を対象として、
   `UserQuestionSet.status` が `NOT_START` または `PROGRESS` のレコードが存在するかを確認し、以下の基準で1件返す。

    * **優先度1: `PROGRESS` のレコードがあればそちらを優先**

        * 複数の `PROGRESS` がある場合は、`UserQuestionSet.started_at` が **最も古い** レコードを返す。
    * **優先度2: `PROGRESS` が1件も無く、`NOT_START` のレコードがある場合**

        * （必要に応じて）最も早く作成され}たもの、または最も `order` が小さいものを返す、などルールを定める。


2. **新規選定**

    1. で該当レコードが無い場合（つまり、現在のGrade内のQuestionSetに `NOT_START` / `PROGRESS` のものが一切無い場合）には、
       下記「2‑2. 次の問題決定ロジック」に従って **次に提示するQuestionSet** を決定する。



### 2‑2. 次の問題決定ロジック
本ロジックでは、*現在学習中* の `QuestionSet` を解答した際の「正答率」に応じて、次に提示する `QuestionSet` を決定します。
ここでいう「正答率」とは、ユーザーが直近の学習(あるいは累積学習)で記録した該当 `QuestionSet` における正解率を示します。
さらに「order」については、以下のように各テーブル内での並びを示すためのカラムであり、**同じ親キー（同じ `unit_id` や同じ `section_id`、同じ `grade_id`）の中でのみ有効なソート順**であることに注意してください。

#### A. *現在学習中* の `QuestionSet` の正答率が **TH_PASS** 以上を **SUCCESS_STREAK** 回達成した場合

1. **【Unit 内での次の `QuestionSet` 探索】**
   同じ `unit_id` を持つ `QuestionSet` のうち、`order` が *現在の `QuestionSet`* よりも次(大きい)の並びのものを検索し、該当する `QuestionSet` があればそれを次に提示します。
   なお、学習リストから個別に呼び出すケースもあるため、自動選定時には「同じ Section 内であるかどうか」を条件に含める必要はありません。あくまで同じ `Unit` 内で `order` が“現在より次”かどうかのみを見ます。

2. **【Section 内での未挑戦問題(`QuestionSet`)の探索】**
   手順 1 で該当する `QuestionSet` が見つからない場合、次は「同じ `grade` に属する、*現在の `QuestionSet` に紐づく `Units` が属する `Sections`*」の領域内において、**未挑戦**の `QuestionSet` を探します。

    * **未挑戦とは**: ユーザーの `user_question_sets` テーブルにまだ `question_set_id` が登録されたことのない `QuestionSet` のことを指します。
    * このとき、*現在の `Sections` と異なる別の `Sections`* の問題が提示されると学習者が違和感を覚える可能性を考慮し、まずは *現在の `Sections`*（複数の場合もあるが基本的には同じセクション階層）に紐づく *全ての `Units`* をチェックし、その中に未挑戦の `QuestionSet` があればその中で任意の一つを提示します。

3. **【Section 内での次の `Units` 探索】**
   手順 2 でも該当がない場合、*現在の `Units` が所属する同じ `Sections`* のなかで、`order` が“現在の `Sections` の次”に当たる `Units` を探します。

    * 具体的には、「同じ `sections` 内で `order` が *現在の `unit_id`* よりも次(大きい)」のうち **最小**の `Unit` を特定し、その `Unit` に紐づく `QuestionSet` の中で `order` が最小のものを次に提示します。

4. **【Grade 内での次の `Sections` 探索】**
   手順 3 でも見つからない場合、さらに上位の階層として「同じ `grade` 内」で *現在の `sections` より次(大きい)の `order` を持つ `sections`* を探します。

    * 見つかった中で最小の `order` を持つ `Sections` に紐づく `Units`（その中で最小の `order`）の `QuestionSet`（さらにその中で最小の `order`）を提示します。

5. **【次の Grade 探索】**
   手順 4 で候補が無い場合は、*現在の `QuestionSet` が属する `grade` を完了* とみなし、次の `grade`（`grades` テーブルで現在の `grade` の `order` より大きいもののうち最小）を選び、その中の最初(最小の `order`)の `Sections` → その中の最初(最小の `order`)の `Units` → その中の最初(最小の `order`)の `QuestionSet` を次に提示します。この時に、`user_grade_transitions` を更新して、どの grade_id から、どの grade_id に進捗したのかを記録する。

6. **【Grade 完了後のランダム選定】**
   手順 5 でも該当が無い場合、指定された複数の `grade_id`（例: `json_id` が "gra\_005" や "gra\_006"）に紐づく**全ての `Sections` → 全ての `Units` → 全ての `QuestionSet`** を網羅的に見て、そこからランダムに 1 つ選定します。

    * これは、事実上すべての学習対象を完了したか、あるいは一通り学習を終えた後でも継続的に出題できるようにするための最終フェーズです。

#### B. *現在学習中* の `QuestionSet` の正答率が **FAIL\_RATE** 未満の場合

1. **【Unit 内での QS ロールバック】**
   まずは同じ `unit_id` の中で、*現在の `QuestionSet`* よりも 1 つ前(`order` が小さい方)の並びを探し、該当の `QuestionSet` があればロールバック先とします。

2. **【Section 内での Units ロールバック】**
   手順 1 で対象が無かった場合は、*現在の `QuestionSet`* が属する `unit_id` が紐づく `sections` 内で、*現在の `unit_id` より `order` が 1 つ前(小さい)の `Units`* を探します。見つかった場合、その `Units` 内で `order` が降順(大きい→小さい)で最初にヒットする `QuestionSet` をロールバック先とします。

3. **【Grade 内での Sections ロールバック】**
   手順 2 でも見つからない場合、次に *現在の `QuestionSet`* が紐づく `unit_id` → それが紐づく `sections` → さらにその `sections` が紐づく `grade` 内で、*現在の `sections` より 1 つ前(小さい)の `order` を持つ `sections`* を探し、その中でさらに *`Units` の `order` が降順で一番最初* かつ *`QuestionSet` の `order` が降順で一番最初* のものをロールバック先とします。

4. **【Grade ロールバック】**
   手順 3 でも該当が無い場合は、*現在の `grade` より 1 つ前(小さい)の `grade`* を探し、その中の *最も `order` が高い(降順で最初)* `Sections` → さらに最も `order` が高い `Units` → さらに最も `order` が高い `QuestionSet` に戻ります。この時に、`user_grade_transitions` を更新して、どの grade_id から、どの grade_id にロールバックしたのかを記録する。

5. **【最上位でも該当無しの場合】**
   4 の手順でも該当が無い場合（実運用上はありえない想定）には、指定した複数の `grades_id` の中で最も `order` が高い(降順で最初) `Sections` → そこで最も `order` が高い `Units` → さらに最も `order` が高い `QuestionSet` にロールバックする形とします。

---

> **補足**: 上記の「次の `order`」や「1 つ前の `order`」、「降順で一番最初」などは、同じ親キー（`unit_id` / `section_id` / `grade_id`）を持つレコード同士の並びで評価します。すなわち、`QuestionSet` の `order` はあくまで同じ `unit_id` の中でのみソート順が有効であり、別の `unit_id` に対して同じ `order` 数値が存在する場合でも衝突は考慮しません。同様に `Unit` の `order` は同じ `section_id` の中での順番を示し、`Section` の `order` は同じ `grade_id` の中での順番を示します。


### 2‑3. 前提条件：order の定義（補足）

本仕様で登場する `order` カラムは、**同じ親キーのレコード群の中**でのみ有効な並び順です。異なる親キーに紐づくレコードとの間で `order` が重複しても問題ありません。以下に例を示します。

* **QuestionSet の `order`**

    * 同じ `unit_id` を持つ `QuestionSet` 同士でのみ順番を比較する。
    * 例:

        * A: question\_sets.id=1, unit\_id=1, order=100
        * B: question\_sets.id=2, unit\_id=1, order=200
        * C: question\_sets.id=3, unit\_id=2, order=100
        * …  (他ユニットにも `order=100` や `200` が存在しても衝突しない)

* **Unit の `order`**

    * 同じ `section_id` 内での順番を示す。
    * 例:

        * A: units.id=1, section\_id=1, order=100
        * B: units.id=2, section\_id=1, order=200
        * C: units.id=3, section\_id=2, order=100
        * …  (section\_id が異なる場合でも同じ数値を使う可能性あり)

* **Section の `order`**

    * 同じ `grade_id` 内での順番を示す。
    * 例:

        * A: sections.id=1, grade\_id=1, order=100
        * B: sections.id=2, grade\_id=1, order=200
        * C: sections.id=3, grade\_id=2, order=100
        * …  (別の grade\_id でも同じ数値を持つ可能性あり)

このように `order` は同一階層の中でのソート指標であり、階層を跨ぐと数値の比較は行わず「別のグループ」として扱います。
例えば「同じ `unit_id` で `order` が 100 → 200」と並んでいるとき、別の `unit_id` に `order` = 100 が存在していても、それらを直接比較することはありません。

---

## 3. パラメータ一覧（環境変数 / 定数で可変）
| 定数                   | 既定値 | 説明                                                          |
|----------------------|--------|-------------------------------------------------------------|
| `TH_PASS`            | **80** | 合格判定となる正答率 (%)                                              |
| `SUCCESS_STREAK`     | **3** | 連続合格回数。満たすと次の QS へ                                          |
| `FAIL_RATE`          | **50** | QS 正答率がこの値を下回ると 1 つ前へ戻る                                     |
| `DAILY_QS_TARGET`    | **1** | 1 日あたり新規学習する QuestionSet の目標値（既に users.daily_goal_type で定義） |
| `ENABLE_NEW_QS_PUSH` | `true` | 新規追加 QS を優先提示するか                                            |
| `GRADE_AUTO_DOWN`    | `true` | ロールバック動作を有効化するか                                             |

> **運用 tip**: `.env` で `TH_PASS` などを A/B テストしやすくしておく。

---

## 4. 新規 QuestionSet 追加時の扱い
- **grade / section が既習より低位の QS**: 既存ユーザーには *OPTIONAL* 提示。
- **同 grade 内への挿入**: `order` 順で自動的に学習対象へ。完了ユーザーについては `ENABLE_NEW_QS_PUSH` で制御。

---

## 5. Grade 遷移の記録
`user_grade_transitions` テーブル（新規）
| column | type | note |
|--------|------|------|
| `id` | PK | |
| `user_id` | FK → users | |
| `from_grade_id` | FK → grades | |
| `to_grade_id` | FK → grades | |
| `reason` | string | `PASS`, `FAIL_BACK`, etc. |
| `created_at` | datetime | |

> **分析用途**: 離脱率や適切レベル到達速度を可視化する。 

---

## 6. 学習ボリューム見積もり
```
QuestionSet 総数 : 109
想定学習日数      : 260 / 年
1 QS あたり学習   : 3 回
⇒ 年間消化量       : 109 × 3 ≒ 327 回 (≒1 年強)
```

---

## 7. 今後の拡張メモ
- **忘却曲線 (SRS)**: 本仕様ではスコープ外。実装時は `next_review_at` と SM‑2 を導入。
- **IRT**: `NEXT_QS_DECISION()` 内部を置き換えるだけで導入可能。
- **Adaptive Difficulty**: `Question.difficulty_id` を用いて段階的調整が可能。

---

### END
