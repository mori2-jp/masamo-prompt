ありがとう。
一旦は問題のmetadata はこれで良いとして、これらの問題のデータはJSONとしてリポジトリで管理しようと思っています。

単元についてもJSONで管理して、LaravelのコマンドでDBに取り込みをしたいです。
フォーマットを考えてください。

## 条件
- 多言語に対応すること
- 科目ごとに単元の管理をします
- 単元の全体は一つのjsonで管理します
- DBのIDは保存時に生成されるのであらかじめJSONで定義することは出来ません。jsonの管理用のIDを別途追加してください（json_id）
```　units
        Schema::create('problems', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('level_id');
            $table->uuid('difficulty_id');
            $table->string('json_id')->nullable()->unique();
            $table->json('metadata')->nullable();
            $table->string('version')->default('0.0.1');
            $table->integer('status')->default(1);
            $table->integer('question_type')->default(1);
            $table->integer('order');
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('level_id')->references('id')->on('levels')->onDelete('cascade');
            $table->foreign('difficulty_id')->references('id')->on('difficulties')->onDelete('cascade');
        });
        
                Schema::create('problem_translations', function (Blueprint $table) {
            $table->id();
            $table->uuid('problem_id');
            $table->string('locale', 10);
            $table->longText('question_text')->nullable();
            $table->longText('explanation')->nullable();
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('problem_id')->references('id')->on('problems')->onDelete('cascade');
        });
```

