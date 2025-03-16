## evaluation_spec（正誤判定用データ）

問題JSON の `evaluation_spec` は、ユーザーの回答を正誤評価する際に使われるデータです。主な項目は以下の通りです。

1. **evaluation_method (必須)**
    - `"CODE"` or `"LLM"` のいずれか
    - `"CODE"` の場合はアプリケーション内のロジック（CheckerMethod）で正誤判定
    - `"LLM"` の場合は大規模言語モデル(LLM)に正誤判定を依頼する

2. **checker_method**
    - `evaluation_method="CODE"` の場合は必須
    - CheckerMethod(enum) に定義されている文字列のみ許容
    - 例: `"CHECK_BY_EXACT_MATCH"`

3. **llm_prompt_number**
    - `evaluation_method="LLM"` の場合のみ必須 (数値)
    - LLMに投げる評価用のプロンプトテンプレートを示す番号
    - `resources/prompts/evaluation/{llm_prompt_number}.txt` が存在するかを確認

4. **response_format (必須、オブジェクト)**
    - ユーザーの回答全体（is_correct, score など）や、各入力欄(fields) の評価フォーマットを定義する
    - **is_correct**: `"boolean"` という文字列固定。回答全体が正解/不正解か
    - **score**: `"number"` という文字列固定。回答全体のスコア
    - **question_text**: 言語ごとのオブジェクト。
        - `evaluation_method="LLM"` の場合 → `{ja: "text", en: "text"}` のように `"text"` と固定
        - `evaluation_method="CODE"` の場合 → `metadata.question_text` と同じ内容(実際の問題文)が入る
    - **explanation**: 言語ごとのオブジェクト。
        - `evaluation_method="LLM"` の場合 → `{ja: "text", en: "text"}` のように `"text"` と固定
        - `evaluation_method="CODE"` の場合 → 解説文の実際の文字列を記載
    - **question**: 言語ごとのオブジェクト。
        - どちらの evaluation_method でも `{ja: "text", en: "text"}` となっている（CODE の場合は `metadata.question` と内容が一致する）
    - **fields**: ユーザーの回答入力欄に対応する配列。
        - `evaluation_method="LLM"` の場合は必須
        - `"CODE"` の場合はあってもよいが必須ではない（ただし、CheckerMethod 次第で利用）
        - 各要素には `field_id`, `user_answer`, `is_correct`, `collect_answer`, `field_explanation` などを持ち、**`field_id` が `metadata.input_format.fields.field_id` と一致** している必要がある

### fields 内の各項目

- **field_id**: どの入力欄かを特定するID (`"f_1"`, `"f_2"`, …)。`metadata.input_format.fields` の同じ `field_id` と対応
- **user_answer**: ユーザー入力の型 (`"number"` など)。`metadata.input_format.fields.user_answer` と同じ文字列
- **is_correct**: `"(boolean)"` という文字列固定
- **collect_answer**: 正答の実値（ユーザーには見せない）
    - 言語定数（`ja`, `en`）ごとにオブジェクトで値を保持
    - `"user_answer"="number"` の場合はここに数値が入る
- **field_explanation**: 各言語での解説文。空文字禁止。言語定数(`ja`, `en`)ごとに文字列を設定

---

## metadata（フロントエンドUI定義）

問題画面を構築する際に必要なデータをまとめたオブジェクトです。ここでは **fill_in_the_blank** タイプに特化した構造を説明します。

1. **question_type**
    - App\Enums\QuestionType に存在する値
    - `"FILL_IN_THE_BLANK"` など

2. **question_text**, **explanation**, **background**, **question**
    - それぞれ言語別オブジェクト: `{ja: "...", en: "..."}`
    - 問題文や説明文などを定義

3. **input_format (必須オブジェクト)**
    - **type**: 解答形式 (`"fixed"`, `"custom"` など)
    - **fields**: ユーザーの回答入力フォーム仕様を配列で定義
        - 各要素に `field_id`, `attribute`, `user_answer` など
        - **`field_id`**: `"f_1"`, `"f_2"`, … 重複不可
            - `"blank"` タイプの入力欄と対応（`question_components` の `type="input_field"` など）
        - **`attribute`**: UI要素の属性（ `"number"` など）
        - **`user_answer`**: 実際にユーザーが入力する解答データ型（ `"number"` など）
        - **collect_answer**: 絶対に存在してはいけない（問題の正解が利用者に露呈しないようにするため）
    - **question_components**: 文章や改行、入力欄など、問題文をパーツ化した配列
        - 各要素に `type`, `content`, `order`, `attribute` など
        - `type="input_field"` の要素数と、`fields` の要素数が一致する（blanksの数＝入力欄の数）
        - `order` で表示順を指定し、重複不可
        - `content` が言語別オブジェクトになっている場合は `{ja: "...", en: "..."}` のように必須言語を全て含む

---

### `evaluation_spec.response_format.fields` と `metadata.input_format.fields` の対応

- **field_id** が同じ文字列でなければいけない
- 例: `evaluation_spec.response_format.fields[*].field_id="f_1"` と `metadata.input_format.fields[*].field_id="f_1"`
- **user_answer** も両方で整合を取ること（`"number"` ならどちらも `"number"`）
- `collect_answer` はユーザーには見せないが、判定のために裏で保持される。
- フロントエンド側では、`metadata.input_format.fields` に定義されたUI属性（`attribute="number"` など）を用いて `<input>` や `<select>` を動的生成し、ユーザーが入力した値を `user_answer` に紐づける。
- サーバ側やLLM評価時には、`evaluation_spec.response_format.fields` で同じ `field_id` を参照し、正解(collect_answer)や正誤状態(is_correct)を扱う。
