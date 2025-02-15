以下のImportSectionsWithUnitsFromGithubを修正してください。
エラーを解消したいです。
JSONをよく確認してください。
units の内部には、level_id や subject_id はありません。親に存在しています。
また、このエラーとは関係ないですが、sections のデータを作成する際に、JSON のlevel_id と subject_id を直接インサートしていますが、
これらは　levels 及び subjects の json_id なので、これを元に levels 及び　subjects のプライマリーキーである ID を探して、それをDBにインサートする必要があります。

ImportSectionsWithUnitsFromGithubは先程あなたが作成したコードですが、私が少し修正を加えていますので再度確認してください。

ーーエラー
Error upserting unit json_id=unit_s1_l3_sec14_001: SQLSTATE[23000]: Integrity constraint violation: 1048 Column 'subject_id' cannot be null (Connection: mariadb, SQL: insert into `units` (`id`, `json_id`, `order`, `status`, `section_id`, `subject_id`, `level_id`, `name`, `description`, `version`, `updated_at`, `created_at`) values (d0632c6b-f45e-46d4-9a43-799f49804b66, unit_s1_l3_sec14_001, 1, 300, 5bffdb8e-6666-4afe-ad9d-c5a207b84f74, ?, ?, ?, ?, 1.0.0, 2025-02-15 11:21:59, 2025-02-15 11:21:59))
Error upserting unit json_id=unit_s1_l3_sec14_002: SQLSTATE[23000]: Integrity constraint violation: 1048 Column 'subject_id' cannot be null (Connection: mariadb, SQL: insert into `units` (`id`, `json_id`, `order`, `status`, `section_id`, `subject_id`, `level_id`, `name`, `description`, `version`, `updated_at`, `created_at`) values (513cf127-f3c1-40c6-b7b1-086fdf8ab64b, unit_s1_l3_sec14_002, 2, 300, 5bffdb8e-6666-4afe-ad9d-c5a207b84f74, ?, ?, ?, ?, 1.0.0, 2025-02-15 11:21:59, 2025-02-15 11:21:59))
Error upserting unit json_id=unit_s1_l3_sec14_003: SQLSTATE[23000]: Integrity constraint violation: 1048 Column 'subject_id' cannot be null (Connection: mariadb, SQL: insert into `units` (`id`, `json_id`, `order`, `status`, `section_id`, `subject_id`, `level_id`, `name`, `description`, `version`, `updated_at`, `created_at`) values (cd573a2d-28ae-4031-885f-acf86f8ae3a5, unit_s1_l3_sec14_003, 3, 300, 5bffdb8e-6666-4afe-ad9d-c5a207b84f74, ?, ?, ?, ?, 1.0.0, 2025-02-15 11:21:59, 2025-02-15 11:21:59))
Error upserting unit json_id=unit_s1_l3_sec14_004: SQLSTATE[23000]: Integrity constraint violation: 1048 Column 'subject_id' cannot be null (Connection: mariadb, SQL: insert into `units` (`id`, `json_id`, `order`, `status`, `section_id`, `subject_id`, `level_id`, `name`, `description`, `version`, `updated_at`, `created_at`) values (c2861a6a-07bb-4ea0-ba50-4b7b53e36b9c, unit_s1_l3_sec14_004, 4, 300, 5bffdb8e-6666-4afe-ad9d-c5a207b84f74, ?, ?, ?, ?, 1.0.0, 2025-02-15 11:21:59, 2025-02-15 11:21:59))
Upserted section json_id=sec_s1_l3_0014

＝＝＝

# 条件
1.
パスは、「contents/sections/{version}/{subject}」です。
例：contents/sections/v1.0.0/math/sections_s1_l1_001.json
例：contents/sections/v1.0.0/math/sections_s1_l2_001.json
例：contents/sections/v1.0.0/math/sections_s1_l3_001.json

versionは、v1.0.0 などのバージョンの文字列が入っています
s1 は、subject ID が 1を表しています
l1 は、Level ID が 1を表しています

コマンドのオプションで subject を指定している時は subject 内のファイルのみ処理の対象とし、
subject の指定が無い場合は、subjects/以下すべてを対象としたい

例 subject が math の場合、
contents/sections/v1.0.0/math/sections_s1_l1_001.json
contents/sections/v1.0.0/math/sections_s1_l2_001.json
これらのファイルが対象

例 subject が指定されていない場合、
contents/sections/v1.0.0/math/sections_s1_l1_001.json
contents/sections/v1.0.0/math/sections_s1_l2_001.json
contents/sections/v1.0.0/logic/sections_s1_l1_001.json
contents/sections/v1.0.0/logic/sections_s1_l2_001.json
contents/sections/v1.0.0/sience/sections_s1_l1_001.json
contents/sections/v1.0.0/sience/sections_s1_l2_001.json
これら全てのファイルが対象。（subject はここで示したもの以外にも無数に存在します。

つまり、レスポンスが type = dir の時は再帰的に処理をして json ファイルを見つける必要があります。


2.
リポジトリは、
git@github.com:NousContentsManagement/masamo-content.git
です。

3.
order: 1、order: 2は、そのJSON内(sections または　units ）での順番を表しています。
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
なた、sections, section_translations, または、units, unit_translations の両方にカラムは存在しますが、sections または、units には、日本語をインサートして、sction_translations または、unit_translations には言語ごとにインサートしてください

5.
github token は config/services.php github.api_token

jsonの、difficulty_id　や　level_id , unit_id , subject_id は、levels や difficulties , units, subjects の ID ではなく json_id のことを指しています。
これは、primary key である id は生成時に自動生成されるため予め定義されたJSONではIDを知り得ないため参照用のIDとしてjson_idを容易しています。
つまり、jsonのlevel_id から levels の json_id と対応するデータを探す必要があります（dificulties, units , subjectsも同様に）

6.
Sections は Enum\SectionStatus
Units は Enum\UnitStatus
を使うこと

ーーDB
```json
        Schema::create('units', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('subject_id');
            $table->uuid('level_id');
            $table->string('json_id')->nullable()->unique();
            $table->string('name')->nullable();
            $table->text('description')->nullable();
$table->longText('background')->nullable();
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
$table->longText('background')->nullable();
$table->timestamps();
$table->softDeletes();

$table->foreign('unit_id')->references('id')->on('units')->onDelete('cascade');
});

Schema::create('sections', function (Blueprint $table) {
$table->uuid('id')->primary();
$table->uuid('subject_id');
$table->uuid('level_id');
$table->string('json_id')->nullable()->unique();
$table->string('name')->nullable();
$table->text('description')->nullable();
$table->longText('requirement')->nullable();
$table->longText('required_competency')->nullable();
$table->string('version')->default('0.0.1');
$table->integer('status')->default(1);
$table->integer('order');
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
$table->timestamps();
$table->softDeletes();

$table->foreign('subject_id')->references('id')->on('subjects')->onDelete('cascade');
$table->foreign('level_id')->references('id')->on('levels')->onDelete('cascade');
});

Schema::create('section_translations', function (Blueprint $table) {
$table->id();
$table->uuid('section_id');
$table->string('locale', 10);
$table->string('name')->nullable();
$table->text('description')->nullable();
$table->longText('requirement')->nullable();
$table->longText('required_competency')->nullable();
$table->timestamps();
$table->softDeletes();

$table->foreign('section_id')->references('id')->on('sections')->onDelete('cascade');
});

```

--JSON
```json
{
  "sections": [
    {
      "order": 1,
      "json_id": "sec_s1_l1_001",
      "subject_id": "sub_001",
      "level_id": "lev_001",
      "learning_category": "A",
      "status": "PUBLISHED",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "数の数え方",
        "en": "Counting Numbers"
      },
      "description": {
        "ja": "",
        "en": ""
      },
      "units": [
        {
          "order": 1,
          "unit_id": "unit_s1_l1_sec1_001",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "数を順番に数えられる",
              "en": "Can count numbers in sequence"
            }
          ],
          "required_competency": [
            {
              "ja": "「1つずつ順番に数える」「どちらが大きいか比較する」などの基礎的操作が可能",
              "en": "Capable of basic operations such as counting items one by one in order and comparing which is larger"
            }
          ],
          "background": [
            {
              "ja": "数直線や具体物を用いて理解させる",
              "en": "Use number lines and concrete objects to aid understanding"
            }
          ]
        },
        {
          "order": 2,
          "unit_id": "unit_s1_l1_sec1_002",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "4位数、1万の比べ方や数え方",
              "en": "How to compare and count 4-digit numbers and up to ten thousand"
            }
          ],
          "required_competency": [
            {
              "ja": "最大5桁（1万まで）の整数を正しく読み書きでき，大小比較できる",
              "en": "Able to correctly read, write, and compare integers up to five digits (up to 10,000)"
            }
          ],
          "background": [
            {
              "ja": "位取りの概念をさらに拡張し，万の位を扱うことに慣れる",
              "en": "Further expand the concept of place value and become familiar with handling the ten-thousands place"
            }
          ]
        }
      ]
    },
    {
      "order": 2,
      "json_id": "sec_s1_l1_002",
      "subject_id": "sub_001",
      "level_id": "lev_001",
      "learning_category": "A",
      "status": "PUBLISHED",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "足し算・引き算",
        "en": "Addition and Subtraction"
      },
      "description": {
        "ja": "",
        "en": ""
      },
      "units": [
        {
          "order": 1,
          "unit_id": "unit_s1_l1_sec2_001",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "加法及び減法の意味",
              "en": "Meaning of Addition and Subtraction"
            }
          ],
          "required_competency": [
            {
              "ja": "たし算・ひき算を「ものを増やす／減らす」という生活場面に結びつけて説明できる",
              "en": "Able to explain addition and subtraction by relating them to everyday situations of 'increasing or decreasing things'"
            }
          ],
          "background": [
            {
              "ja": "具体物（積み木やおはじき等）を操作しながら，計算の意義を直感的に身に付ける",
              "en": "Acquire an intuitive understanding of the significance of calculation by manipulating concrete objects such as building blocks or counters"
            }
          ]
        },
        {
          "order": 2,
          "unit_id": "unit_s1_l1_sec2_002",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "1位数や簡単な2位数の加法及び減法",
              "en": "Addition and subtraction of single-digit and simple two-digit numbers"
            }
          ],
          "required_competency": [
            {
              "ja": "繰り上がり・繰り下がりのない範囲であれば，正しく加減計算できる",
              "en": "Can perform addition and subtraction accurately when there is no carrying or borrowing involved"
            }
          ],
          "background": [
            {
              "ja": "計算練習を通してスピードと正確性を養う。文章題への橋渡しを意識する。",
              "en": "Develop speed and accuracy through practice while keeping in mind the transition to word problems"
            }
          ]
        },
        {
          "order": 3,
          "unit_id": "unit_s1_l1_sec2_003",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "加法及び減法の場面の表現",
              "en": "Representation of situations involving addition and subtraction"
            }
          ],
          "required_competency": [
            {
              "ja": "「3 + 2 = 5」を「3に2を足すと5になる」と読める",
              "en": "Able to read “3 + 2 = 5” as “Adding 2 to 3 makes 5”"
            }
          ],
          "background": [
            {
              "ja": "数量と言葉，式を結び付ける力は文章題の理解に直結する",
              "en": "The ability to connect quantities, words, and expressions directly leads to understanding word problems"
            }
          ]
        },
        {
          "order": 4,
          "unit_id": "unit_s1_l1_sec2_004",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "加法及び減法の場面の式読み",
              "en": "Interpreting expressions for addition and subtraction situations"
            }
          ],
          "required_competency": [
            {
              "ja": "具体的な状況（りんごが3個あって… など）を式に変換する初歩的スキルがある",
              "en": "Has basic skills to convert concrete scenarios (e.g., having three apples, etc.) into expressions"
            }
          ],
          "background": [
            {
              "ja": "数量と言葉，式を結び付ける力は文章題の理解に直結する",
              "en": "The ability to connect quantities, words, and expressions directly leads to understanding word problems"
            }
          ]
        }
      ]
    },
    {
      "order": 3,
      "json_id": "sec_s1_l1_003",
      "subject_id": "sub_001",
      "level_id": "lev_001",
      "learning_category": "A",
      "status": "PUBLISHED",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "数の活用",
        "en": "Using Numbers"
      },
      "description": {
        "ja": "",
        "en": ""
      },
      "units": [
        {
          "order": 1,
          "unit_id": "unit_s1_l1_sec3_001",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "数の活用",
              "en": "Utilization of Numbers"
            }
          ],
          "required_competency": [
            {
              "ja": "身の回りの物や人数などを数えて，必要な数を報告できる",
              "en": "Able to count objects and people around and report the required numbers"
            }
          ],
          "background": [
            {
              "ja": "生活の中で数え方や比べ方を自然に使っていることに気付かせる",
              "en": "Help students realize they naturally use counting and comparison in everyday life"
            }
          ]
        },
        {
          "order": 2,
          "unit_id": "unit_s1_l1_sec3_002",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "加法、減法の活用",
              "en": "Application of Addition and Subtraction"
            }
          ],
          "required_competency": [
            {
              "ja": "「りんごが5個あって2個食べたら残りは何個か」など，日常的な場面でたし算・ひき算を使える",
              "en": "Able to use addition or subtraction in everyday scenarios (e.g., “If you have 5 apples and eat 2, how many are left?”)"
            }
          ],
          "background": [
            {
              "ja": "買い物ごっこや簡単なゲームなどを通じて計算を実践し，算数への親しみを深める",
              "en": "Deepen familiarity with mathematics by practicing calculations through pretend shopping and simple games"
            }
          ]
        }
      ]
    },
    {
      "order": 4,
      "json_id": "sec_s1_l1_004",
      "subject_id": "sub_001",
      "level_id": "lev_001",
      "learning_category": "B",
      "status": "PUBLISHED",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "ものの形",
        "en": "Shapes of Objects"
      },
      "description": {
        "ja": "",
        "en": ""
      },
      "units": [
        {
          "order": 1,
          "unit_id": "unit_s1_l1_sec4_001",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "形の特徴",
              "en": "Features of Shapes"
            }
          ],
          "required_competency": [
            {
              "ja": "身身近な物の形を見て，丸み・角・辺などの特徴をとらえることができる",
              "en": "Able to observe everyday objects and identify features such as roundness, corners, and edges"
            }
          ],
          "background": [
            {
              "ja": "具体物やイラストを活用し，形に着目する感覚を育む",
              "en": "Foster a sense of focusing on shapes by using concrete objects and illustrations"
            }
          ]
        },
        {
          "order": 2,
          "unit_id": "unit_s1_l1_sec4_002",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "形作り・分解",
              "en": "Creating and Decomposing Shapes"
            }
          ],
          "required_competency": [
            {
              "ja": "紙を折る，切る，積み木を組み合わせる等によって形を作ったり分解したりできる",
              "en": "Able to create or decompose shapes by folding or cutting paper, or by combining building blocks"
            }
          ],
          "background": [
            {
              "ja": "操作活動を通じて「形が変わる」ことを体験し，図形への興味を高める",
              "en": "Experience how shapes change through hands-on activities and increase interest in geometry"
            }
          ]
        },
        {
          "order": 3,
          "unit_id": "unit_s1_l1_sec4_003",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "形",
              "en": "Shapes"
            }
          ],
          "required_competency": [
            {
              "ja": "身の回りの物の形を認識し，簡単な分類（丸いもの，四角いもの など）ができる",
              "en": "Able to recognize the shapes of everyday objects and categorize them simply (e.g., round items, square items)"
            }
          ],
          "background": [
            {
              "ja": "生活空間で「形」に目を向ける姿勢を育てる",
              "en": "Develop awareness of shapes in the living environment"
            }
          ]
        },
        {
          "order": 4,
          "unit_id": "unit_s1_l1_sec4_004",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "ものの位置",
              "en": "Positions of Objects"
            }
          ],
          "required_competency": [
            {
              "ja": "上下・左右など，物の位置関係を表す言葉を使ってコミュニケーションできる",
              "en": "Able to use words describing positional relationships (up/down, left/right, etc.) to communicate"
            }
          ],
          "background": [
            {
              "ja": "教室や家の中での「場所の説明」を通じて空間把握を養う",
              "en": "Foster spatial awareness by describing locations in the classroom or at home"
            }
          ]
        }
      ]
    },
    {
      "order": 5,
      "json_id": "sec_s1_l1_005",
      "subject_id": "sub_001",
      "level_id": "lev_001",
      "learning_category": "C",
      "status": "PUBLISHED",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "もののはかり方",
        "en": "How to Measure Objects"
      },
      "description": {
        "ja": "",
        "en": ""
      },
      "units": [
        {
          "order": 1,
          "unit_id": "unit_s1_l1_sec5_001",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "長さの比較",
              "en": "Comparison of Length"
            }
          ],
          "required_competency": [
            {
              "ja": "長い/短いを具体物の直接比較で判断できる",
              "en": "Able to judge longer or shorter by directly comparing concrete objects"
            }
          ],
          "background": [
            {
              "ja": "ひもや物差しを使う前段階として，視覚・触覚での比較を体感",
              "en": "Experience visual and tactile comparisons as a preliminary step before using strings or rulers"
            }
          ]
        },
        {
          "order": 2,
          "unit_id": "unit_s1_l1_sec5_002",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "広さの比較",
              "en": "Comparison of Area"
            }
          ],
          "required_competency": [
            {
              "ja": "広い/狭いを，並べたり重ねたりして大きさを感じ取れる",
              "en": "Able to perceive wide or narrow by aligning or overlapping surfaces"
            }
          ],
          "background": [
            {
              "ja": "机の上や紙の面積を比べるなど，操作活動を通じて理解を深める",
              "en": "Deepen understanding through hands-on activities such as comparing the area on a desk or a piece of paper"
            }
          ]
        },
        {
          "order": 3,
          "unit_id": "unit_s1_l1_sec5_003",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "かさの比較",
              "en": "Comparison of Volume"
            }
          ],
          "required_competency": [
            {
              "ja": "多い/少ないを，容器に入れて比べたり移し替えたりする活動で判断できる",
              "en": "Able to judge which is more or less by comparing or transferring contents between containers"
            }
          ],
          "background": [
            {
              "ja": "水や砂を使った実験的活動を行い，「かさ」の概念をつかむ",
              "en": "Engage in experimental activities with water or sand to grasp the concept of volume"
            }
          ]
        },
        {
          "order": 4,
          "unit_id": "unit_s1_l1_sec5_004",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "量の比べ方",
              "en": "Ways to Compare Quantities"
            }
          ],
          "required_competency": [
            {
              "ja": "いろいろな物の長さ・広さ・かさを日常的に見比べ，言葉で説明できる",
              "en": "Able to compare the length, area, and volume of various objects in daily life and describe them verbally"
            }
          ],
          "background": [
            {
              "ja": "教室探検や家庭内での観察を通じて，測定の必要性を感じさせる",
              "en": "Develop an awareness of the necessity of measurement through classroom exploration and household observations"
            }
          ]
        }
      ]
    },
    {
      "order": 6,
      "json_id": "sec_s1_l1_006",
      "subject_id": "sub_001",
      "level_id": "lev_001",
      "learning_category": "C",
      "status": "PUBLISHED",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "時刻と時間",
        "en": "Time and Clock Reading"
      },
      "description": {
        "ja": "",
        "en": ""
      },
      "units": [
        {
          "order": 1,
          "unit_id": "unit_s1_l1_sec6_001",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "日常生活の中での時刻の読み",
              "en": "Reading Time in Daily Life"
            }
          ],
          "required_competency": [
            {
              "ja": "アナログ時計やデジタル時計を見て，大まかな時刻を読むことができる",
              "en": "Able to look at analog or digital clocks and read approximate times"
            }
          ],
          "background": [
            {
              "ja": "生活の場面と結びつけ，何時に登校/給食/下校かなど時間感覚を育てる",
              "en": "Develop a sense of time by connecting it to daily routines, such as what time to go to school, have lunch, or go home"
            }
          ]
        }
      ]
    },
    {
      "order": 7,
      "json_id": "sec_s1_l1_007",
      "subject_id": "sub_001",
      "level_id": "lev_001",
      "learning_category": "D",
      "status": "PUBLISHED",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "データの活用",
        "en": "Utilization of Data"
      },
      "description": {
        "ja": "",
        "en": ""
      },
      "units": [
        {
          "order": 1,
          "unit_id": "unit_s1_l1_sec7_001",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "データの個数への着目",
              "en": "Focusing on the Number of Data Items"
            }
          ],
          "required_competency": [
            {
              "ja": "「いくつあるか」数を数えてまとめ，絵や記号などで表せる",
              "en": "Able to count how many there are, summarize the data, and represent it using pictures or symbols"
            }
          ],
          "background": [
            {
              "ja": "身近なデータ（好きな色アンケート等）を取り，○や絵シンボルで一覧化する活動を行う",
              "en": "Collect familiar data (e.g., a favorite color survey) and list them using circles or picture symbols"
            }
          ]
        },
        {
          "order": 2,
          "unit_id": "unit_s1_l1_sec7_002",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "絵や図",
              "en": "Pictures and Diagrams"
            }
          ],
          "required_competency": [
            {
              "ja": "棒線や簡単な絵を用いて，一覧表やグラフのまねごとを作れる",
              "en": "Able to create a simple table or an imitation of a graph using lines or basic drawings"
            }
          ],
          "background": [
            {
              "ja": "いきなりグラフではなく，絵やシールを貼って可視化する初歩的段階を重視",
              "en": "Instead of starting directly with graphs, emphasize a basic stage of visualization using pictures or stickers"
            }
          ]
        },
        {
          "order": 3,
          "unit_id": "unit_s1_l1_sec7_003",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "身の回りの事象の特徴についての把握",
              "en": "Understanding Characteristics of Everyday Phenomena"
            }
          ],
          "required_competency": [
            {
              "ja": "クラスで一番多い好きな果物は何か，など簡単な事柄を把握できる",
              "en": "Able to identify simple facts, such as which fruit is the most popular in class"
            }
          ],
          "background": [
            {
              "ja": "「多い/少ない」に着目して気づきを共有する",
              "en": "Focus on 'more or less' to share observations"
            }
          ]
        },
        {
          "order": 4,
          "unit_id": "unit_s1_l1_sec7_004",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "絵や図",
              "en": "Pictures and Diagrams"
            }
          ],
          "required_competency": [
            {
              "ja": "自分や友達が作った絵図を見て，すぐに特徴を読み取ることができる",
              "en": "Able to quickly grasp the main features by looking at diagrams drawn by themselves or classmates"
            }
          ],
          "background": [
            {
              "ja": "絵図の読み取りに慣れさせ，シンプルなデータ可視化への抵抗感をなくす",
              "en": "Familiarize students with interpreting pictorial diagrams to reduce any resistance to simple data visualization"
            }
          ]
        }
      ]
    },
    {
      "order": 8,
      "json_id": "sec_s1_l1_008",
      "subject_id": "sub_001",
      "level_id": "lev_001",
      "learning_category": "A",
      "status": "PUBLISHED",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "規則性",
        "en": "Patterns"
      },
      "description": {
        "ja": "",
        "en": ""
      },
      "units": [
        {
          "order": 1,
          "unit_id": "unit_s1_l1_sec8_001",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "",
              "en": ""
            }
          ],
          "required_competency": [
            {
              "ja": "",
              "en": ""
            }
          ],
          "background": [
            {
              "ja": "",
              "en": ""
            }
          ]
        },
        {
          "order": 2,
          "unit_id": "unit_s1_l1_sec8_002",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": [
            {
              "ja": "",
              "en": ""
            }
          ],
          "required_competency": [
            {
              "ja": "",
              "en": ""
            }
          ],
          "background": [
            {
              "ja": "",
              "en": ""
            }
          ]
        }
      ]
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
<?php

namespace App\Models\Section;

use App\Traits\HasOrder;
use App\Traits\UsesUuid;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Section extends Model
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
<?php

namespace App\Models\Section;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class SectionTranslation extends Model
{
use SoftDeletes;
}
<?php

namespace App\Enums;

enum SectionStatus: int
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
 * 文字列から SectionStatus を取得
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

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Str;

// Models
use App\Models\Section\Section;
use App\Models\Section\SectionTranslation;
use App\Models\Unit\Unit;
use App\Models\Unit\UnitTranslation;
use App\Models\Subject\Subject;
use App\Models\Level\Level;

// Enums
use App\Enums\SectionStatus;
use App\Enums\UnitStatus;

/**
 * Import sections & their units JSON from a private GitHub repository
 * and upsert into sections / section_translations / units / unit_translations.
 */
class ImportSectionsWithUnitsFromGithub extends Command
{
    protected $signature = 'import:sections-with-units-from-github
                            {--subject= : If specified, only import files under contents/sections/*/{subject} }';

    protected $description = 'Import sections data (and child units) from GitHub JSON and sync to DB';

    public function handle()
    {
        // 1. GitHubトークンを config から取得
        $token = config('services.github.api_token');
        if (!$token) {
            $this->error('GitHub API token not found in config/services.php [github.api_token]');
            return 1;
        }

        // 2. リポジトリ情報
        $repoOwnerAndName = 'NousContentsManagement/masamo-content';

        // ベースディレクトリ: contents/sections
        $basePath = 'contents/sections';

        // subject オプションの有無でパスを変える
        $subjectOption = $this->option('subject');
        if ($subjectOption) {
            // 例: contents/sections/*/math
            $basePath .= '/*/' . $subjectOption;
            $this->info("Subject specified: {$subjectOption}, path= {$basePath}");
        } else {
            $this->info("No subject specified. Will import from all subdirectories under contents/sections");
        }

        // 3. GitHub API で再帰的にファイル一覧(.json)を取得
        $allJsonFiles = $this->fetchAllJsonFilesRecursively($repoOwnerAndName, $basePath, $token);
        if (empty($allJsonFiles)) {
            $this->warn("No JSON files found under path: {$basePath}");
            return 0;
        }

        $this->info("Found " . count($allJsonFiles) . " JSON file(s) under path: {$basePath}");

        // 4. 各 JSON ファイルを処理
        foreach ($allJsonFiles as $file) {
            $this->processJsonFile($file, $token);
        }

        $this->info("Import process completed successfully.");
        return 0;
    }

    /**
     * 再帰的にディレクトリをたどり .jsonファイルを集める
     */
    private function fetchAllJsonFilesRecursively(string $repo, string $path, string $token): array
    {
        if (str_contains($path, '*')) {
            return $this->fetchWildcardPathFiles($repo, $path, $token);
        }

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
            $name = $item['name'] ?? '';
            $itemPath = $item['path'] ?? '';

            if ($type === 'dir') {
                // ディレクトリ → 再帰
                $descendants = $this->fetchAllJsonFilesRecursively($repo, $itemPath, $token);
                $result = array_merge($result, $descendants);
            } elseif ($type === 'file') {
                if (str_ends_with($name, '.json')) {
                    $result[] = $item;
                }
            }
        }

        return $result;
    }

    /**
     * pathにワイルドカード('*')が含まれる場合の処理
     */
    private function fetchWildcardPathFiles(string $repo, string $path, string $token): array
    {
        $parts = explode('/', $path);
        $wildcardIndex = array_search('*', $parts);

        if ($wildcardIndex === false) {
            return $this->fetchAllJsonFilesRecursively($repo, $path, $token);
        }

        $prefix = implode('/', array_slice($parts, 0, $wildcardIndex));
        $suffix = implode('/', array_slice($parts, $wildcardIndex + 1));

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
                $dirPath = $item['path'] . '/' . $suffix;
                $descendants = $this->fetchAllJsonFilesRecursively($repo, $dirPath, $token);
                $result = array_merge($result, $descendants);
            }
        }
        return $result;
    }

    /**
     * 単一JSONファイルをダウンロードし "sections" 配列をDBへupsert
     * さらに各sectionの"units"配列もupsert
     */
    private function processJsonFile(array $file, string $token)
    {
        if (!isset($file['download_url'])) {
            $this->warn("No download_url for file: " . ($file['path'] ?? 'unknown'));
            return;
        }

        $this->info("Fetching JSON from: " . $file['path']);
        $jsonContent = Http::withToken($token)
            ->get($file['download_url'])
            ->body();

        $decoded = json_decode($jsonContent, true);
        if (!is_array($decoded) || !isset($decoded['sections'])) {
            $this->warn("File {$file['name']} does not contain 'sections' array. Skipped.");
            return;
        }

        // sections配列
        $sectionsArray = $decoded['sections'];
        // order順にソート
        usort($sectionsArray, fn($a, $b) => ($a['order'] ?? 9999) <=> ($b['order'] ?? 9999));

        foreach ($sectionsArray as $sectionItem) {
            $this->upsertSectionWithUnits($sectionItem);
        }
    }

    /**
     * sections テーブル + section_translations テーブルに upsert
     * さらにその配下のunits も upsert
     */
    private function upsertSectionWithUnits(array $sectionItem)
    {
        DB::beginTransaction();
        try {
            $sectionJsonId = $sectionItem['json_id'] ?? null;
            if (!$sectionJsonId) {
                $this->warn("Section JSON missing 'json_id'. Skipped.");
                DB::rollBack();
                return;
            }

            // 既存セクション検索
            $sectionRecord = Section::where('json_id', $sectionJsonId)->first();

            // subject_id, level_id 解決
            $subjectUuid = $this->findSubjectUuid($sectionItem['subject_id'] ?? null);
            $levelUuid   = $this->findLevelUuid($sectionItem['level_id'] ?? null);

            // order 衝突回避
            $jsonOrder = $sectionItem['order'] ?? 9999;

            if (!$sectionRecord) {
                // 新規
                $sectionRecord = new Section();
                $sectionRecord->id      = (string) Str::uuid();
                $sectionRecord->json_id = $sectionJsonId;

                // order の最大値を探し +1 など
                $maxOrder = Section::max('order');
                if (is_null($maxOrder)) {
                    $maxOrder = 0;
                }
                $finalOrder = max($maxOrder + 1, $jsonOrder);
                while (Section::where('order', $finalOrder)->exists()) {
                    $finalOrder++;
                }
                $sectionRecord->order = $finalOrder;
            } else {
                // 既存 → order更新
                $this->reorderSection($sectionRecord->id, $jsonOrder);
            }

            // status は Enum\SectionStatus
            $statusString = $sectionItem['status'] ?? 'DRAFT';
            $statusEnum   = SectionStatus::fromString($statusString);

            $sectionRecord->status      = $statusEnum->value;
            $sectionRecord->subject_id  = $subjectUuid;
            $sectionRecord->level_id    = $levelUuid;
            $sectionRecord->learning_category = $sectionItem['learning_category'] ?? null;
            $sectionRecord->version     = $sectionItem['version'] ?? '0.0.1';

            // JSONの name/description は日本語を sectionsテーブルに
            $jaName = $sectionItem['name']['ja'] ?? null;
            $jaDesc = $sectionItem['description']['ja'] ?? null;

            $sectionRecord->name        = $jaName;
            $sectionRecord->description = $jaDesc;

            $sectionRecord->save();

            // requirement と required_competency は配列かもしれない→改行で結合
            $requirementLines         = [];
            $requiredCompetencyLines  = [];
            if (isset($sectionItem['requirement']) && is_array($sectionItem['requirement'])) {
                foreach ($sectionItem['requirement'] as $reqItem) {
                    $jaReq = $reqItem['ja'] ?? '';
                    if ($jaReq) {
                        $requirementLines[] = $jaReq;
                    }
                }
            }
            if (isset($sectionItem['required_competency']) && is_array($sectionItem['required_competency'])) {
                foreach ($sectionItem['required_competency'] as $reqComp) {
                    $jaReqC = $reqComp['ja'] ?? '';
                    if ($jaReqC) {
                        $requiredCompetencyLines[] = $jaReqC;
                    }
                }
            }

            // sectionsテーブル上の requirement / required_competency カラム
            $sectionRecord->requirement         = implode("\n", $requirementLines);
            $sectionRecord->required_competency = implode("\n", $requiredCompetencyLines);
            $sectionRecord->save();

            // section_translations に多言語を upsert
            $this->upsertSectionTranslations($sectionRecord, $sectionItem);

            // セクション配下のunits
            if (isset($sectionItem['units']) && is_array($sectionItem['units'])) {
                // JSON内のunits を order順にソート
                $unitsArr = $sectionItem['units'];
                usort($unitsArr, fn($a, $b) => ($a['order'] ?? 9999) <=> ($b['order'] ?? 9999));

                // ここでセクションのDB上のUUIDをユニットに渡す
                foreach ($unitsArr as $unitItem) {
                    $this->upsertUnit($unitItem, $sectionRecord->id);
                }
            }

            DB::commit();
            $this->info("Upserted section json_id={$sectionJsonId}");
        } catch (\Throwable $e) {
            DB::rollBack();
            $this->error("Error upserting section json_id={$sectionJsonId}: " . $e->getMessage());
        }
    }

    /**
     * 既存セクションの並び順を更新
     */
    private function reorderSection(string $sectionId, int $targetOrder)
    {
        $section = Section::find($sectionId);
        if (!$section) {
            return;
        }

        // 衝突回避
        while (Section::where('order', $targetOrder)->where('id','!=',$sectionId)->exists()) {
            $targetOrder++;
        }
        $section->order = $targetOrder;
        $section->save();
    }

    /**
     * section_translations に多言語を upsert
     */
    private function upsertSectionTranslations(Section $sectionRecord, array $sectionItem)
    {
        foreach (['ja','en'] as $locale) {
            // name, description
            $localeName = $sectionItem['name'][$locale] ?? null;
            $localeDesc = $sectionItem['description'][$locale] ?? null;

            // requirement, required_competency
            $reqLines = [];
            $reqCompLines = [];
            if (isset($sectionItem['requirement']) && is_array($sectionItem['requirement'])) {
                foreach ($sectionItem['requirement'] as $reqItem) {
                    $txt = $reqItem[$locale] ?? '';
                    if ($txt) {
                        $reqLines[] = $txt;
                    }
                }
            }
            if (isset($sectionItem['required_competency']) && is_array($sectionItem['required_competency'])) {
                foreach ($sectionItem['required_competency'] as $rcItem) {
                    $txt = $rcItem[$locale] ?? '';
                    if ($txt) {
                        $reqCompLines[] = $txt;
                    }
                }
            }

            $reqJoined    = implode("\n", $reqLines);
            $reqCompJoined= implode("\n", $reqCompLines);

            if ($localeName || $localeDesc || $reqJoined || $reqCompJoined) {
                SectionTranslation::updateOrCreate(
                    [
                        'section_id' => $sectionRecord->id,
                        'locale'     => $locale,
                    ],
                    [
                        'name'               => $localeName,
                        'description'        => $localeDesc,
                        'requirement'        => $reqJoined,
                        'required_competency'=> $reqCompJoined,
                    ]
                );
            }
        }
    }

    /**
     * JSONデータ1件(ユニット)を DB に upsert.
     * $sectionId: 親セクションのDB上のUUID
     */
    private function upsertUnit(array $unitItem, string $sectionId)
    {
        DB::beginTransaction();
        try {
            // JSONによって 'json_id' と 'unit_id' のどちらを使うか揺れがあるなら両方をチェック
            $jsonId = $unitItem['json_id'] ?? $unitItem['unit_id'] ?? null;
            if (!$jsonId) {
                $this->warn("unit JSON missing 'json_id' or 'unit_id'. Skipped.");
                DB::rollBack();
                return;
            }

            // 既存レコード検索
            $unitRecord = Unit::where('json_id', $jsonId)->first();

            // subject_id, level_id 解決
            $subjectUuid = $this->findSubjectUuid($unitItem['subject_id'] ?? null);
            $levelUuid   = $this->findLevelUuid($unitItem['level_id']   ?? null);

            // order 衝突回避
            $jsonOrder = $unitItem['order'] ?? 9999;
            if (!$unitRecord) {
                $unitRecord = new Unit();
                $unitRecord->id      = (string) Str::uuid();
                $unitRecord->json_id = $jsonId;

                $maxOrder = Unit::max('order');
                if (is_null($maxOrder)) {
                    $maxOrder = 0;
                }
                $finalOrder = max($maxOrder+1, $jsonOrder);
                while (Unit::where('order', $finalOrder)->exists()) {
                    $finalOrder++;
                }
                $unitRecord->order = $finalOrder;
            } else {
                // 既存 → 順序更新
                $this->reorderUnit($unitRecord->id, $jsonOrder);
            }

            // status は Enum\UnitStatus
            $statusString = $unitItem['status'] ?? 'DRAFT';
            $statusEnum   = UnitStatus::fromString($statusString);
            $unitRecord->status     = $statusEnum->value;

            // **section_id をセット**
            $unitRecord->section_id = $sectionId;

            $unitRecord->subject_id = $subjectUuid;
            $unitRecord->level_id   = $levelUuid;

            // unitsテーブルには日本語版を
            $jaName = $unitItem['name']['ja'] ?? null;
            $jaDesc = $unitItem['description']['ja'] ?? null;
            $unitRecord->name        = $jaName;
            $unitRecord->description = $jaDesc;
            $unitRecord->version     = $unitItem['version'] ?? '0.0.1';

            $unitRecord->save();

            // unit_translations
            $this->upsertUnitTranslations($unitRecord, $unitItem);

            DB::commit();
            $this->info("Upserted unit json_id={$jsonId}");
        } catch (\Throwable $e) {
            DB::rollBack();
            $this->error("Error upserting unit json_id={$jsonId}: " . $e->getMessage());
        }
    }

    private function reorderUnit(string $unitId, int $targetOrder)
    {
        $unit = Unit::find($unitId);
        if (!$unit) return;

        while (Unit::where('order', $targetOrder)->where('id','!=',$unitId)->exists()) {
            $targetOrder++;
        }
        $unit->order = $targetOrder;
        $unit->save();
    }

    private function upsertUnitTranslations(Unit $unitRecord, array $unitItem)
    {
        foreach (['ja','en'] as $locale) {
            $localeName = $unitItem['name'][$locale] ?? null;
            $localeDesc = $unitItem['description'][$locale] ?? null;

            if ($localeName !== null || $localeDesc !== null) {
                UnitTranslation::updateOrCreate(
                    [
                        'unit_id' => $unitRecord->id,
                        'locale'  => $locale,
                    ],
                    [
                        'name'        => $localeName,
                        'description' => $localeDesc,
                    ]
                );
            }
        }
    }

    /**
     * Subject ID解決
     */
    private function findSubjectUuid(?string $jsonId): ?string
    {
        if (!$jsonId) return null;
        return Subject::where('json_id',$jsonId)->value('id');
    }

    /**
     * Level ID解決
     */
    private function findLevelUuid(?string $jsonId): ?string
    {
        if (!$jsonId) return null;
        return Level::where('json_id',$jsonId)->value('id');
    }
}
```
