問題のJSONはGithubのプライベートリポジトリで管理しています。
github リポジトリからgithub api を利用してファイルを取得してDBにインサートするLaravel11のコマンドを実装してください。

# 条件
1.
パスは、「contents/questions/{subject}/{question_type}」です。
例：contents/questions/math/FILL_IN_THE_BLANK/ques-s1-l2-001.json
例：contents/questions/math/FILL_IN_THE_BLANK/ques-s1-l2-001.json

s1 は、subject ID が 1を表しています
l1 は、Level ID が 1を表しています

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
なた、questions, question_translations の両方にカラムは存在しますが、units には、日本語をインサートして、unit_translations には言語ごとにインサートしてください

ーーDB
```json
        Schema::create('questions', function (Blueprint $table) {
$table->uuid('id')->primary();
$table->uuid('level_id');
$table->uuid('difficulty_id');
$table->string('json_id')->nullable()->unique();
$table->json('metadata')->nullable();
$table->string('version')->default('0.0.1');
$table->integer('status')->default(1);
$table->integer('question_type')->default(1);
$table->integer('question_format')->default(1)->comment('出題形式: 1=選択式,2=数値回答,3=テキスト回答,4=画像選択など');
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

Schema::create('question_translations', function (Blueprint $table) {
$table->id();
$table->uuid('question_id');
$table->string('locale', 10);
$table->longText('question_text')->nullable();
$table->longText('explanation')->nullable();
$table->timestamps();
$table->softDeletes();

$table->foreign('question_id')->references('id')->on('questions')->onDelete('cascade');
});
```

--JSON
```json
{
  "order": 1,
  "id": "question_s1_l2_001",
  "level_id": "level-001",
  "difficulty_id": "diff-001",
  "version": "1.0.0",
  "question_type": "FILL_IN_THE_BLANK",
  "question_format": "NUMBER",
  "question_text": {
    "ja": "▢にあてはまる数を答えなさい。",
    "en": "Please answer the numbers that fit in the blanks."
  },
  "explanation": {
    "ja": "",
    "en": ""
  },
  "metadata": {
    "question": "8 × 4 = ▢ × 8 = ▢",
    "inputFormat": {
      "type": "fixed",
      "fields": [
        {
          "field_id": "f_1",
          "attribute": "number",
          "collect_answer": "4"
        },
        {
          "field_id": "f_2",
          "attribute": "number",
          "collect_answer": "4"
        }
      ],
      "question_components": [
        {
          "type": "text",
          "content": "8 × 4 = "
        },
        {
          "type": "blank",
          "field_id": "f_1"
        },
        {
          "type": "text",
          "content": " × 8 = "
        },
        {
          "type": "blank",
          "field_id": "f_2"
        }
      ]
    },
    "": [],
    "evaluationMethod": "LLM"
  },
  "learning_requirements": [
    {
      "learning_subject": "算数",
      "learning_no": 10,
      "learning_requirement": "数と計算 計算の意味・方法 乗法の意味",
      "learning_required_competency": "かけ算は同じ数の繰り返し加法であるという本質を理解できる",
      "learning_category": "A",
      "learning_grade_level": "小2",
      "learning_url": "https://docs.google.com/spreadsheets/d/1W5vaFHcyU_BrMwb1JLZ-DpyFmXXZPaYMTHIITcwLqqY/edit?gid=0#gid=0&range=15:15"
    }
  ],
  "created_at": "2025-01-01 00:00:00",
  "updated_at": "2025-01-01 00:00:00"
}

```
