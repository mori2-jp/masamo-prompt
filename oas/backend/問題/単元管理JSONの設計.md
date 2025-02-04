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
  Schema::create('units', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('subject_id');
            $table->string('name')->nullable();
            $table->text('description')->nullable();
            $table->string('version')->default('0.0.1');
            $table->integer('order');
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('subject_id')->references('id')->on('subjects')->onDelete('cascade');
        });
        
        
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
NousContentsManagement/masamo-content
