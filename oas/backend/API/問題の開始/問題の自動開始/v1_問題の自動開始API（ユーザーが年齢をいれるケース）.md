以下の{# 学習指導要領}は、文部科学省、学習指導要領に従って学習水準をリスト化したものです。


ユーザーが自分で学習する問題を選ぶのではなく、
ロジックでユーザーが学習する問題を自動で決定されるロジックを考えています。（アダブティプ教育）
将来的にIRTを実装する予定ですが、スケジュールと工数の兼ね合いで取り急ぎ簡易的なロジックを実装したいです。
実装担当者がIRTへの知見がないので、将来的なIRTへの拡張などは考慮しないでよいです。
手伝ってください。

以下は、ユーザーと問題のDB設計とモデルです。関係がわかります。
例えば、{# アイディア例}　のようなものを自分で考えてみました。
具体的な実装コードとか詳細な実装の指針は不要なので{# アイディア例}をベースに仕様のアイディアをもらえませんか？

# 条件
・問題の自動生成は気にしないでよい
・問題は、１＋１＝□　□を埋めなさい。のような問題で、選択式ではない
・基本的にはユーザーは１日１QuestionSetやる予定。（もちろん例外あり）
・

# 不満な点
・ユーザーは小１〜小６が対象だが、ちょうど良い箇所から始めるようにしたい、または最初５回程度の学習でちょうど良いレベル箇所に調整されたいが、現在は一番最初から（小１）から開始して順次レベルアップするしかない。初期値を真ん中（例えば小３）にすると小１が学習開始した時に難しすぎてサービス離脱するので必ず小１から開始しなければいけない。ラダーアップ
・グレードをスキップしすぎた場合の調整もしたいのでグレードダウンも考慮したい
・最初の実力を測るテストは実施したくない（作成するリソースがないという事情）
・忘却曲線に対応したい

# アイディア例
問題レスポンスロジック
１）UserQuestion のNOT_START または PROGRESS を返す

→フロントエンドでは「途中のものがあります？再開しますか？」のようなアナウンスを表示する
２）存在しない場合は次の問題の決定ロジックを呼び出して次の問題を決定する

問題の構造
grades は学年
sections は問題の大カテゴリ（ grade_id でリレーション）
question_sets は問題の小カテゴリ（section_id でリレーション）

例）セクション：足し算、問題セット：２桁＋１桁、１桁＋１桁など

次の問題の決定ロジック
１）現在学習中の、QuestionSet で正答率が80%以上３回したら同セクション内の次の question_sets に移動する（order カラムを昇順で並べて次にある問題）

２）同セクション内の QuestionSet の全てが正答率８０％以上を３回達成したら、同セクションと同じ grade 内で 挑戦したことの無い問題があればそれに移動する
３）２）が無い場合は、同セクション内の QuestionSet の全てが正答率８０％以上を３回達成したら次のセクションに移動する

その他

・この仕様で新しい問題が追加された時のケースに対応している。

→新作問題の懸念

１）現在小３の問題をやっている時に、新しく追加された問題が小１の場合だと簡単すぎてUXが下がる懸念、

２）作問の完成度が中途半端なので出来るだけ新作を学習してもらうようにしたい

・全てのパラメータは変更可能なパラメータとしたいので定数として管理する



問題の見積もり
現在の　question_sets の数 ：109
年間想定学習日数：260 (365 * (5/7) 週５日想定)
1 question_sets 辺りの学習回数: 3

109 * 3 = 327 なのでだいたい１年分になる




# 既に実装済みの現在の仕様の説明
・問題は questions から直接取得するのではなく question_sets 経由で question_sets 単位で出題する
・ユーザーは一度出題された問題をもう一度解くことは基本的に（例外あり）ありません。questions が同じユーザで既に学習済みの問題が出題されることのなにように無いように、question_sets 単位で常に一定の新しい問題をストックされるよう閾値を下回ったら自動生成されています。
・まだ一度も学習してもらったことがないので正答率などのデータは持っていません。
・UserQuestion に PROGRESS があればまずはそれをレスポンスする。
・問題が自動生成されるなどの事情から難易度を手動でラベル付けするなどは行いたくない。IRT導入で将来的に自動で難易度付けされる想定だが実装工数があがるので今回はスコープから外したい。

# UserQuestionSet のステータスの説明
・ NOT_START: 開始していない
・PROGRESS：学習途中
・SKIP：学習途中でユーザーが中断した
・COMPLETE：学習完了

# 説明
question_sets：問題（questions）を束ねるグループ（※マイグレーションには書いてないが、grade_id を持っている）
questions: 問題
question_set_questions：question_sets と questions を紐づけるPivotテーブル。questions は複数のquestion_sets に紐づく事があり多対多の関係なのでこのような設計になっている。
user_question_sets: ユーザーが学習した questions_sets。学習開始時に question_sets_id と紐づいて status （UserQuestionSetsStatus::NOT_START) が未開始の状態で生成され、進捗ステータスはスコアなどが管理される。１回の学習の塊なので学習の度に新規作成される
user_questions: ユーザーが学習した questions。学習開始時に、学習を開始した question_sets に紐づく questions が全て、 user_question_sets_id と question_id（questions）を紐づけて status （UserQuestionStatus::NOT_START) が未開始の状態で全てのquestionsの数の分、生成され、進捗ステータスはスコアなどが管理される
skills： 論理的思考能力、判断力、表現力、知識・技能、創造性　など、問題の特徴づけ（実際は問題が自動生成されるなどの運用上の理由から適切なskillの関連付けが不可能なので使えない）


# 学習要件
No	学年	カテゴリ	サブカテゴリー	セクション	セクションID	要件	Unit ID	必要水準	補足・背景
1	小1	A 数と計算	数の概念	数の数え方	sec_s1_g1_100	数を順番に数えられる	u100	「1つずつ順番に数える」「どちらが大きいか比較する」などの基礎的操作が可能	数直線や具体物を用いて理解させる
2	小1	A 数と計算	数の概念	数の数え方	sec_s1_g1_100	2位数，簡単な3位数の読み書き・比較		"・最大3桁の整数について，正しく読み・書き・大小比較ができる。
"	"・10進法の基礎（位取り）に親しむための導入期。
"
3	小1	A 数と計算	計算の意味・方法	足し算・引き算	sec_s1_g1_200	加法及び減法の意味		・たし算・ひき算を「ものを増やす／減らす」という生活場面に結びつけて説明できる。	・具体物（積み木やおはじき等）を操作しながら，計算の意義を直感的に身に付ける。
4	小1	A 数と計算	計算の意味・方法	足し算・引き算	sec_s1_g1_200	(1位数や簡単な2位数の加法及び減法)		・繰り上がり・繰り下がりのない範囲であれば，正しく加減計算できる。	・計算練習を通してスピードと正確性を養う。文章題への橋渡しを意識する。
5	小1	A 数と計算	式・関係	足し算・引き算	sec_s1_g1_200	加法及び減法の場面の表現		"・「3 + 2 = 5」を「3に2を足すと5になる」と読める。
"	・数量と言葉，式を結び付ける力は文章題の理解に直結する。
6	小1	A 数と計算	式・関係	足し算・引き算	sec_s1_g1_200	加法及び減法の場面の式読み		・具体的な状況（りんごが3個あって… など）を式に変換する初歩的スキルがある。	・数量と言葉，式を結び付ける力は文章題の理解に直結する。
7	小1	A 数と計算	日常生活への活用	数の活用	sec_s1_g1_300	数の活用	u100	・身の回りの物や人数などを数えて，必要な数を報告できる。	・生活の中で数え方や比べ方を自然に使っていることに気付かせる。
8	小1	A 数と計算	日常生活への活用	数の活用	sec_s1_g1_300	逆算		□のある式
9	小1	A 数と計算	日常生活への活用	数の活用	sec_s1_g1_300	加法、減法の活用	u200	・「りんごが5個あって2個食べたら残りは何個か」など，日常的な場面でたし算・ひき算を使える。	・買い物ごっこや簡単なゲームなどを通じて計算を実践し，算数への親しみを深める。
10	小1	A 数と計算	日常生活への活用	数の活用	sec_s1_g1_300	計算のくふう			
11	小1	A 数と計算	計算の意味・方法	規則性	sec_s1_g1_800	魔法陣			
12	小1	A 数と計算	計算の意味・方法	規則性	sec_s1_g1_800				
13	小2	A 数と計算	計算の意味・方法	足し算・引き算	sec_s1_g2_100	２位数の加法とその逆の減法	u100	1位数で習得した加減法を基に、2位数の繰り上がり・繰り下がりを正しく理解し筆算できる	・十進位取り記数法を具体物や図で再確認し、筆算の形式が成立する理由を体感的に理解させる。
14	小2	A 数と計算	計算の意味・方法	足し算・引き算	sec_s1_g2_100	"簡単な場合の３位数などの加法，減法
大きな数を習った後じゃないと解けないのでに移動。SectionJSONを修正"	u200		
15	小2	A 数と計算	計算の意味・方法	足し算・引き算	sec_s1_g2_100	加法や減法に関して成り立つ性質	u300	・加法および減法について、交換法則(a + b = b + a)や結合法則((a + b) + c = a + (b + c))が成り立つことを具体的に理解できる。	・具体例や操作活動で、順序・まとまりを変えても結果が変わらないことを確認し、高学年の式操作へつなぐ素地を作る。
16	小2	A 数と計算	計算の意味・方法	足し算・引き算	sec_s1_g2_100	加法と減法との相互関係	u400	・加法と減法が三つの数量のどれを求めるかで相互に対応していることを理解し、式立てに活用できる。	・文章題などでA+B=CやC-B=Aなどの関係を押さえ、加法・減法の使い分けの基礎を育成する。
17	小2	A 数と計算	計算の意味・方法	足し算・引き算	sec_s1_g2_100	図を使って考える			
18	小2	A 数と計算	数の概念	大きな数	sec_s1_g2_200	4位数、1万の比べ方や数え方	u100	・最大5桁（1万まで）の整数を正しく読み書きでき，大小比較できる。	・位取りの概念をさらに拡張し，万の位を扱うことに慣れる。
19	小2	A 数と計算	数の概念	大きな数	sec_s1_g2_200	数の相対的大きさ	u200	・300と2000のような大きさの違いを位の考え方で説明できる。	・数の大小を主観ではなく客観的（位取り）に判断する基礎力を養う。
20	小2	A 数と計算	数の概念	大きな数	sec_s1_g2_200	簡単な場合の３位数などの加法，減法		・百単位などに注目して、3位数の加法・減法（繰り上がりや繰り下がりが簡単な場合）を2位数の計算を拡張して処理できる。	・2位数計算の延長として3位数に取り組み、位ごとのまとまりを捉える力を養う。
21	小2	A 数と計算	数の概念	簡単な分数	sec_s1_g2_700	簡単な分数	u100	・1/2，1/3などの基本的な分数を「何をいくつに分けたか」というイメージと結び付けられる。	・折り紙や円形のピザなどを用いて分割の感覚をつかむ導入期。
22	小2	A 数と計算	数の概念	簡単な分数	sec_s1_g2_700	倍数と分数の関係	u100	分数(1/2, 1/3 など)を繰り返し足したり，かけ算と関連付けたりして，倍数的な見方（たとえば1/2を2倍すると1になる等）を簡単な事例で理解できる	「1/2を2回足すと1になる」「1/3を3倍すると1になる」など，分数と倍数の関係を体感的に捉えさせる。2年生段階ではイメージ中心の扱いに留め，難しい式変形は避けるとよい
23	小2	A 数と計算	計算の意味・方法	掛け算	sec_s1_g2_600	乗法の意味	u100	・かけ算は同じ数の繰り返し加法であるという本質を理解できる。	・具体例（2+2+2=2×3）で体感させることが大切。
24	小2	A 数と計算	計算の意味・方法	掛け算	sec_s1_g2_600	2位数や簡単な3位数の乗法	u200	・2桁×1桁や3桁×1桁（繰り上がり含む）を正確に筆算できる。	・九九と連動して素早く処理できるよう演習量を確保。
25	小2	A 数と計算	計算の意味・方法	掛け算	sec_s1_l2_600	乗法九九、簡単な2位数の乗法	u300	・1～9の九九を暗唱し，2桁×1桁程度の計算は瞬時にこなせる。	・九九の習熟は継続的練習が必須。家庭学習との連携も重要。
26	小2	A 数と計算	計算の意味・方法	計算の決まり	sec_s1_g2_800	加法の交換法則を式で示す        	u100	a + b = b + a を具体例（3+5=5+3等）で理解し，同じ結果になることを確認できる	実際に小物を数える・ブロックを移し替えるなど操作的活動と組み合わせると，法則への納得感が高まる。後の乗法の交換法則にも発展しやすい
27	小2	A 数と計算	計算の意味・方法	計算の決まり	sec_s1_g2_800	加法の結合法則を式で示す        	u200	(a + b) + c = a + (b + c) を具体例（(3+5)+2と3+(5+2)等）で確認し，同じ結果になることを理解	「どの順序で足しても結果が同じ」という構造を，ブロックやイラストを移動させる活動で可視化し，高学年以降の式操作の基礎づくりに役立てる
28	小2	A 数と計算	式・関係	掛け算	sec_s1_l2_600	乗法の場面の表現・式読み	u400	・「りんごが3個ずつ2袋ある」を3×2の式にでき，答えを導ける。	・文章題でかけ算をどう見抜くかがポイント。
29	小2	A 数と計算	式・関係	計算の決まり	sec_s1_g2_800	加法と減法の相互関係	u300	・a + b - b = a のような逆演算を認識し，簡単に検算できる。	・逆算や問題解決のアプローチに応用できる。
30	小2	A 数と計算	式・関係	計算の決まり	sec_s1_g2_800	( )や□を用いた式	u400	・(3 + 2) × 4 など( )付きの式を正しく読み，下線(□)を未知の数として扱う問題に慣れる。	・中学での文字式への入り口として，未知数を記号化する感覚を育む。
31	小2	A 数と計算	日常生活への活用	大きな数	sec_s1_g2_200	大きな数の活用	u300	・1万程度までの数を，生活や社会の中で扱える（人口，お金など）。	・「学年全体の人数」「地域の人口」など具体的な例で親しみを高める。
32	小2	A 数と計算	日常生活への活用	掛け算	sec_s1_l2_600	乗法の活用	u500	・同じ数を繰り返し足すよりかけ算の方が便利だと実感でき、2桁×1桁程度の乗法を生活の簡単な計算に使える。	・計算力を生活で役立てる体験をし，学ぶ意義を感じさせる。
33	小2	A 数と計算	計算の意味・方法	規則性	sec_s1_g2_1200	 繰り返しパターンを見つけ，次にくる要素を推測する	u100	○→△→□→○→△→□…のような簡単な周期がある並びを理解し，周回の周期がわかれば次の要素を正しく求められる	身近な繰り返し（曜日のサイクルやリズムパターンなど）を例に挙げると理解しやすい。図形や色カードなど具体物を用いて，周回の概念や指折りによる確認を体験的に学ばせる
34	小2	A 数と計算	計算の意味・方法	規則性	sec_s1_g2_1200	一定の差で増減する並びをとらえ，指定された番目や合計を求める	u200	足し算・引き算の繰り返しとして扱い，感覚的に「差を加えていく」ことを経験させる。文章題や図を用いて，「1段目から何段目までの合計」を求める練習などにも応用しやすい	階段を上がる段数の増加や，何個ずつ増やす/減らすなど日常の場面に結びつけ，「同じ差で進む」イメージを育む。のちのかけ算や一次関数的な考え方の土台になるので，簡単な数例で繰り返し慣れさせる
35	小2	A 数と計算	計算の意味・方法	和差算・分配算	sec_s1_g2_1300		u100		
36	小2	A 数と計算	計算の意味・方法	植木算	sec_s1_g2_1400		u100		
37	小3	A 数と計算	数の概念	大きな数の概念と活用	sec_s1_g3_100	万の単位、1億などの比べ方や表し方	u100	・1万，10万，100万，1億などの上位の位を正しく読み書きできる。	・さらに大きな数への理解を深め，数のスケールを体感できるようにする。
38	小3	A 数と計算	数の概念	大きな数の概念と活用	sec_s1_g3_100	大きな数の相対的大きさ	u200	 1万と1億の差を人口やお金の桁などの具体例で説明できる	・単なる記号操作ではなく，日常との結び付きで「いくつ分違うか」を考えさせる。
39	小3	A 数と計算	数の概念	分数と少数	sec_s1_g3_700	小数	u100	・少数の意味を理解できる	・少数、整数の分類や比較などの問題を通して意味や概念を理解する
40	小3	A 数と計算	数の概念	分数と少数	sec_s1_g3_700	簡単な分数	u200	・分数の意味を理解できる	・整数、分数の分類や比較などの問題を通して意味や概念を理解する
41	小3	A 数と計算	数の概念	分数と少数	sec_s1_g3_700	小数や簡単な分数の大きさの比較	u300	・少数と少数、分数と分数の比較。0.5や0.3など小数第1位を分数(1/2, 3/10など)と対応づけられる。	・小数と分数が互いに表し合えることを具体的に示す（長さや重さなど）
"	u300	"・隣り合う数の差が一定であることを把握し，それを繰り返し足す・引く考え方ができる。
"	買い物でのお金の残高や，カウントダウンなど，減少していく場面との関連付けで学習すると理解が深まり，実生活とのつながりを感じやすい。
56	小3	A 数と計算	計算の意味・方法	規則性	sec_s1_g3_1200	・変化のきまりを理解し，式や表に整理できる。	u400	・はじめの数と差から，いくつ目の数を求める簡単な式（○＋(n−1)×差 等）を導入部として扱う。	中学以降の一次関数に向けた前段階として，「1増えるごとに一定を足す」という考え方をしっかり身に付ける。
57	小3	A 数と計算	計算の意味・方法	そろばん	sec_s1_g3_1000	そろばんによる計算	u100	・そろばんの基本操作を身につけ，簡単な加減乗除ができる。	・アナログの数操作を通じて数概念を可視化し，計算の理解を深める。
58	小3	A 数と計算	式・関係	割り算	sec_s1_g3_300	わり算の場面の式表現・式読み	u600	・「12個のあめを3人に等しく分ける」→12÷3といった式を立てられる。	・わり算を使う文脈を増やして，式の読み書きをスムーズに行う。
59	小3	A 数と計算	式・関係	式の表現	sec_s1_g3_1100	図及び式による表現・関連付け	u100	・テープ図などを描いて数量を整理し，式に落とし込むスキルを身に付ける。	・段階的に思考を可視化し，文章題への対応力を養う。
60	小3	A 数と計算	式・関係	式の表現	sec_s1_g3_1100	□を用いた式	u200	・□+4=10のように，□を未知数として扱い答えを導ける。	・中学での文字式に進む前段階として重要な経験。
61	小3	A 数と計算	式・関係	式の表現	sec_s1_g3_1100	文章題	u300		
62	小3	A 数と計算	日常生活への活用	大きな数の概念と活用	sec_s1_g3_100	大きな数の活用	u500	・買い物や距離の測定等で，大きな数を用いた計算を体験できる。	・実際の金額やメジャーを用いるなど，リアルな課題設定で応用力を促す。
63	小3	A 数と計算	日常生活への活用	分数と少数	sec_s1_g3_700	小数の活用	u600	・買い物や距離の測定等で，小数を用いた計算を体験できる。	・実際の金額やメジャーを用いるなど，リアルな課題設定で応用力を促す。
64	小3	A 数と計算	日常生活への活用	分数と少数	sec_s1_g3_700	分数の活用	u700	・買い物や距離の測定等で，分数を用いた計算を体験できる。	・実際の金額やメジャーを用いるなど，リアルな課題設定で応用力を促す。
65	小3	A 数と計算	日常生活への活用	割り算	sec_s1_g3_300	わり算の活用	u700	・人数で分ける，単価を求めるなど，わり算が不可欠な場面を処理できる。	・クラブ費やお菓子の分け方等，日常事例を取り入れ理解を深める。
66	小3	A 数と計算	計算の意味・方法	割り算	sec_s1_g3_300	2桁のわり算	u500	あまりあり、あまりなしの2桁以上の除法(2桁÷1桁、2桁÷2桁など)を正しく計算できる。繰り上がりや0の処理を含めた筆算・暗算に対応し、答えを正確に求められる	筆算や暗算を組み合わせ、大きな数の除法へ移行する基礎を育成する
67	小3	A 数と計算	プログラミング	プログラミングの基礎	sec_s1_g3_1600	順次処理の体験	u100	・簡単な指示（例:キャラクターを前進→ジャンプ→回転）の手順を正しい順番で並べ想定どおりに動かせる。	・日常の手順やビジュアル型プログラミング環境を活用し順次処理の概念を楽しく学ばせる。
68	小3	A 数と計算	プログラミング	プログラミングの基礎	sec_s1_g3_1600	くり返し処理の体験	u200	・同じ動作を複数回くり返す命令を使いプログラムの効率化を図れる。	・簡単なゲーム要素や図形描画などを例に繰り返し命令の便利さと修正のしやすさを体感させる。
69	小3	A 数と計算		学びのワーク	sec_s1_g3_1700	重なりに注目して			
70	小3	A 数と計算		学びのワーク	sec_s1_g3_1700	倍の計算			
71	小3	A 数と計算		学びのワーク	sec_s1_g3_1700	間の数			
72	小3	A 数と計算		難問	sec_s1_g3_1800				
73	小4	A 数と計算	数の概念	大きな数の概念と活用	sec_s1_g4_100	億、兆の単位などの比べ方や表し方（統合的）	u100	・1億や1兆などを正しく読み書きし，桁数を把握できる。	・ニュースなどでよく見る大きな数を実感としてつかむ。
74	小4	A 数と計算	数の概念	簡単な割合・概数	sec_s1_g4_600	目的に合った数の処理	u100	・四捨五入や概数を用いておおよその値を出すなど，計算の手間を調整できる。	・場面に応じて厳密な数値と概算を使い分ける力を育成する。
75	小4	A 数と計算	数の概念	大きな数の概念と活用	sec_s1_g4_100	10倍にした数、1/10 にした数	u200	・10倍や1/10倍の変化を位取りで正しく理解し，数の増減を説明できる。	・桁の移動を再確認し，小数を含む整数のスケール感や計算の仕組みを深める。
76	小4	A 数と計算	数の概念	大きな数の概念と活用	sec_s1_g4_100	掛け算	u300	・筆算で大きめの乗法を処理し，誤りなく計算できる。	・筆算で大きめの乗法を処理し，誤りなく計算できる。
77	小4	A 数と計算	数の概念	少数	sec_s1_g4_500	小数の相対的大きさ	u100	・0.03と0.3のどちらが大きいかを位取りの考え方で正しく説明できる。	・整数から小数へ，位を横へ広げる段階の学習。
78	小4	A 数と計算	数の概念	分数	sec_s1_g4_900	分数（真分数、仮分数、帯分数）とその大きさの相違	u100	・3/2=1 1/2(帯分数)など，分数表示の変換と大小関係を正しく理解する。	・分数を統合的に捉えられるよう，図や具体物と結びつけて学ぶ。
79	小4	A 数と計算	計算の意味・方法	少数	sec_s1_g4_500	小数を用いた n 倍の意味	u200	・1.2倍や2.5倍など，小数倍を具体的な例と式で示せる。	・拡大・縮小や単位量あたりの考え方とも関連づける。
80	小4	A 数と計算	計算の意味・方法	割り算	sec_s1_g4_300	1位数などによる除法（筆算）	u200	・2桁÷1桁や3桁÷1桁などのわり算を筆算で処理できる。	・桁の扱いが複雑化するため，しっかりと手順を教える
103	小4	A 数と計算	計算の意味・方法	規則性	sec_s1_g4_1000	大小関係や加減乗除の規則を応用して、未知の項や幅を推測できる。	u200	・図や表を活用して、途中の段階や次の段階にある要素を論理的に導ける。	・規則の多様な捉え方を経験し、問題解決において見通しを立てる力を養う。
104	小4	A 数と計算	計算の意味・方法	規則性	sec_s1_g4_1000	数や図形の変化を整理し、複数の法則を組み合わせたパターンにも対応する。	u100	・複合的な規則を含む問題を試行錯誤し、仮説を立てて検証できる。	・多段階のパターン認識を通じて、中学数学の方程式や関数の理解を後押しする。
105	小4	A 数と計算	数の概念	倍の見方	sec_s1_g4_1700	倍の概念と活用	u100	・同じ量をいくつ分に当たるかを掛け算で表し図や式で説明できる。	・「倍」の意味を定着させ割合学習への橋渡しとする。
106	小4	A 数と計算	数の概念	がい数	sec_s1_g4_600	がい数の加減算	u300	・がい数にした後でも筆算の手順を用いて概算値を計算できる。	・実生活で迅速な判断が求められる場面に概算を生かす。
107	小4	A 数と計算	思考・判断	図を使って考える	sec_s1_g4_1800	図を用いた差異の発見	u100	・テープ図やベン図を使い数量の差や共通部分を視覚的に整理できる。	・複数情報を図解し論理的に整理する力を伸ばす。
108	小4	A 数と計算	思考・判断	共通部分に注目	sec_s1_g4_1900	共通部分の図示	u100	・ベン図で集合の共通部分を示し数量の重なりを説明できる。	・集合的思考を養い重複計算や条件整理の基礎を体感する。
109	小4	A 数と計算	思考・判断		sec_s1_g4_2000	考える力の伸ばそう	u100		
110	小5	A 数と計算	数の概念	整数・分数の概念	sec_s1_g5_800	観点を決めることによる整数の類別や数の構成	u100	・整数を奇数・偶数，約数・倍数などの観点で分類できる。	・因数分解や最小公倍数の考えにもつながる基礎力。
111	小5	A 数と計算	数の概念	整数と小数のしくみをまとめよう	sec_s1_g5_100	整数・小数を位ごとに分解／再構成し0.001単位までの個数で表現できる	u100	・「1×a＋0.1×b＋…」の形で数を構成・分解し，0.001 を基準に大きさを表せる。	位取りと単位量に基づく数構成力を確立し，小数計算・単位換算の基礎を固める。
112	小5	A 数と計算	数の概念	整数と小数のしくみをまとめよう	sec_s1_g5_100	10ⁿ倍と10⁻ⁿ倍（1/10, 1/100, 1/1000）の桁移動規則を説明し計算に適用できる。	u200	・小数点の右左移動で×10,×100,…と÷10,÷100,…を暗算・筆算に正しく活用できる。	乗除演算・概算の効率化に不可欠な位取り移動の因果関係を数直線や表で確認させる。
113	小5	A 数と計算	数の概念	整数と小数のしくみをまとめよう	sec_s1_g5_100	位と桁数を基に小数第3位までの大小比較や条件付き数づくりを行い論理的に説明できる。	u300	・<,> や数直線で大小を判断し，指定カードで最大・最小・近似値を構成できる。	位の価値と大小関係を統合的に活用し，データ読解や発展課題（カード並べ）に対応する力を養う。
114	小5	A 数と計算	数の概念	整数・分数の概念	sec_s1_g5_1700	数の相対的大きさ	u100	・10万と1000万のように，さらに大きい数の比較を位の仕組みで的確に行える。	・数の大小比較の考え方を拡張し，社会での大きい数（予算など）にも目を向ける。
115	小5	A 数と計算	数の概念	整数・分数の概念	sec_s1_g5_1000	分数の相約及び大小関係	u100	・分数を約分し，既約分数にする手順を理解し大小を判断できる。	・分数計算の効率化に直結するため，しっかりした習得が必要。
116	小5	A 数と計算	数の概念	整数・分数の概念	sec_s1_g5_900	分数と整数、小数の関係	u100	・2/5=0.4など，分数⇔小数の変換を自由に行える。	・具体例(1/2=0.5など)から一般化し，小数は分数の一形態と捉えさせる。
117	小5	A 数と計算	数の概念	整数・分数の概念	sec_s1_g5_900	除法の結果の分数による表現	u200	・3÷2=3/2のように，わり算の商を分数で表せる。	・中学以降の「分数の乗除」や文字式への接続を見据えて理解させる。
118	小5	A 数と計算	計算の意味・方法	計算の意味と方法	sec_s1_g5_400	乗法の意味の拡張（小数）	u100	・小数×小数，小数÷小数も，整数の繰り返し加法や等分除の延長線で捉えられる。	・桁と小数点の扱いを復習しつつ，より複雑な事例に挑戦する。
119	小5	A 数と計算	計算の意味・方法	計算の意味と方法	sec_s1_g5_400	数の乗法（小数×小数）	u200	・(2.5)×(1.2)や(3.6)÷(1.2)などで筆算を正しく行い，答えを導ける。	・4年生までの延長として桁数の増加にも対応し，習熟度を高める
130	小5	A 数と計算	プログラミング	手続き処理	sec_s1_g5_1800	指定された条件から実行結果を理解する	u100	・基本的な指示の流れをトレースし，期待される出力を把握できる。	・簡単なアルゴリズムを追う中で論理力を育て，手順を正確に実行する意識を高める。
131	小5	A 数と計算	プログラミング	手続き処理	sec_s1_g5_1800	実行結果から条件を理解する	u200	・実行後の変化を見て，どんな条件や判断があったかを推測できる。	・結果から逆に処理を推論し，条件分岐や繰り返しの仕組みに慣れる。
132	小5	A 数と計算	計算の意味・方法	規則性	sec_s1_g5_1900	くり返すパターンの発見	u100	・○△□など繰り返し図形や数列を見分け、次や途中の要素を推定できる。	・パターン認識を図や表と紐付ける基礎として扱い、アルゴリズム的思考へ繋げる。
133	小5	A 数と計算	計算の意味・方法	規則性	sec_s1_g5_1900	周期を用いた要素の位置づけ	u200	・一定の周期で繰り返される並びに対し、「n番目には何が来るか」を表や簡単な式で求められる。	・中学以降の一次関数へ接続する前段階として、対応関係を捉える力を涵養する。
134	小5	A 数と計算	計算の意味・方法	規則性	sec_s1_g5_1900	等差数列の仕組み	u300	・隣り合う項の差が一定になる数列を捉え、表や簡単な式(n番目=...)で求められる。	・買い物の残金推移や日々の増減など、身近な場面で差が一定になる例を検証することで興味を深める。
135	小5	A 数と計算	計算の意味・方法	小数の倍	sec_s1_g5_100	基準量に対する小数倍の計算	u400	・基準量に小数を掛けて倍の量を正確に計算できる（例：3.2の2.5倍）。	・割合を小数で扱う前提として増減や縮尺など実生活の場面で頻出する。
136	小6	A 数と計算	計算の意味・方法	分数の計算（乗法・除法）	sec_s1_g6_300	分数の掛け算	u100	・分数×整数、分数÷整数、分数×分数など多様な乗除混合計算を約分を含め正確に行える。	・図や操作活動を通じて，分数の乗法が「基準量の○/○倍」を表すことを理解し，中学の比例・関数へスムーズに橋渡しする。
137	小6	A 数と計算	計算の意味・方法	分数の計算（乗法・除法）	sec_s1_g6_300	分数÷分数	u200	・分数÷分数を逆数を用いて乗法に変換し，筆算で正確に計算できる。	・逆数の概念を活用する考え方は，中学以降の有理数計算や式の変形の基礎となるため，理由付けとセットで定着させる。
138	小6	A 数と計算	計算の意味・方法	分数の計算（乗法・除法）	sec_s1_g6_300	乗法及び除法の適用範囲の拡張（分数）	u300	・(a/b)×(c/d)，(a/b)÷(c/d)など分数同士の乗除を式で表し，計算できる。	・中学の有理数計算に向け，整数・小数・分数すべての乗除を総合的に扱う。
139	小6	A 数と計算	計算の意味・方法	分数の計算（乗法・除法）	sec_s1_g6_300	分数の乗法及び除法（多面的）	u400	・分数×分数，分数÷分数を面積モデルや数直線など複数の観点で捉えられる。	・計算手順だけでなく，なぜそうなるのかを図解で理解させる。
140	小6	A 数と計算	計算の意味・方法	分数の計算（乗法・除法）	sec_s1_g6_300	分数・小数の混合計算（統合的）	u500	・分数と小数が混在する計算（例：1.2 + 3/5や0.75×2/3）を正しく処理できる。	・中学以降のあらゆる数を扱う計算へスムーズにつなげる最終仕上げ。
141	小6	A 数と計算	計算の意味・方法	分数の計算（乗法・除法）	sec_s1_g6_300	分数の倍	u600	・「○の□倍」を分数で表し，数量を求められる（例：2/3倍は×2/3であることを使用）。	・割合の感覚を深化させ，中学の比例・関数へ接続するために「倍」を比やスケールの視点で扱う。
142	小6	A 数と計算	計算の意味・方法	分数の計算（乗法・除法）	sec_s1_g6_300	分数の文章題	u700	・日常場面を表す文章題を分数の乗除を用いて式化し，解答まで一貫して示せる。	・数量の単位や比較対象を明確化し，誤った分数操作を防ぐ読解・モデリング力を養う。
143	小6	A 数と計算	計算の意味・方法	文字を用いた式	sec_s1_g6_200	文字を用いて数量や計算の関係を一次式で表せる。	u100	・場面に応じて a や x などの文字を使い「4×x」「y＝7＋x」のような式を正しく立てられる。	・中学で学ぶ文字式・方程式への橋渡しとして数の一般化を体験させる。
144	小6	A 数と計算	計算の意味・方法	文字を用いた式	sec_s1_g6_200	文字式に数を代入して値を求めたり対応関係を読み取れる。	u200	・x＝15 など具体値を代入して計算し逆に式から x と y の対応関係を読み取れる。	・比例・関数の式操作に自然につながるよう活用事例に慣れさせる。
145	小6	A 数と計算	式・関係	式とプログラミング	sec_s1_g6_1200	文字 a、n などを用いた式表現・式の読み書き（簡潔・一般的）	u100	・“n + 5” “2a - 1”のような文字式を読み書きし，問題を整理できる。	・中学での方程式や不等式導入が円滑になるよう，文字化に慣れさせる。
146	小6	A 数と計算	プログラミング	式とプログラミング	sec_s1_g6_1200	与えられたロジックに渡した値の結果を予測する	u200	・入力値の変化に伴う出力を表や簡単な式で整理し、結果を正確に特定できる。	・中学での方程式や不等式導入が円滑になるよう，文字化に慣れさせる。
147	小6	A 数と計算	プログラミング	式とプログラミング	sec_s1_g6_1200	複数の処理を経た結果を予想区する	u300	・条件分岐や繰り返し処理を把握し、いくつかの入力を通して最終的な出力を予測できる。	・中学での方程式や不等式導入が円滑になるよう，文字化に慣れさせる。
148	小6	A 数と計算	計算の意味・方法	規則性	sec_s1_g6_1300	繰り返しパターンを見いだし、次の形や数を予測する	u100	・図や表を用いて、一定間隔や繰り返しパターンを抽出し式に表せる。	・数列や図形パターンの本質を見極め、中学の関数的思考へつなぐ。


```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('grades', function (Blueprint $table) {
            $table->id();
            $table->uuid('uuid')->unique();
            $table->string('json_id')->nullable()->unique()->comment('リポジトリ上で管理するための一意識別子');
            $table->text('description')->nullable();
            $table->text('memo')->nullable();
            $table->string('version')->default('0.0.1');
            $table->integer('order');
            $table->timestamps();
            $table->softDeletes();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('grades');
    }
};
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('sections', function (Blueprint $table) {
            $table->id();
            $table->uuid('uuid')->unique();
            $table->unsignedBigInteger('subject_id');
            $table->unsignedBigInteger('level_id');
            $table->unsignedBigInteger('grade_id');
            $table->string('json_id')->nullable()->unique();
            $table->string('name')->nullable();
            $table->text('description')->nullable();
            $table->longText('requirement')->nullable();
            $table->longText('required_competency')->nullable();
            $table->string('version')->default('0.0.1');
            $table->integer('status')->default(1);
            $table->integer('order');
            $table->string('learning_category')->nullable()
                ->comment('分類 e.g. "A", "B"');
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('subject_id')->references('id')->on('subjects')->onDelete('cascade');
            $table->foreign('level_id')->references('id')->on('levels')->onDelete('cascade');
            $table->foreign('grade_id')->references('id')->on('grades')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('sections');
    }
};
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('question_sets', function (Blueprint $table) {
            $table->id();
            $table->uuid('uuid')->unique();
            $table->unsignedBigInteger('unit_id')->nullable();
            $table->string('json_id')->nullable()->unique();
            $table->string('version')->default('0.0.1');
            $table->integer('status')->default(1);
            $table->integer('order');
            $table->longText('generate_question_prompt')->nullable();
            $table->string('generate_question_prompt_file_name')->nullable();
            $table->integer('llm_generation_status')->nullable()->comment('LLMの問題生成を許可するかどうか。LLMでの問題生成が可能な問題かどうかなどを管理');
            $table->integer('consecutive_generation_failures')->default(0)->comment('LLMの問題生成の連続失敗回数');
            $table->boolean('generation_blocked')->default(false)->comment('LLMの問題生成の停止フラグ。失敗が続いた場合に停止');
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('unit_id')->references('id')->on('units')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        DB::statement('SET FOREIGN_KEY_CHECKS=0;');
        Schema::dropIfExists('question_sets');
        DB::statement('SET FOREIGN_KEY_CHECKS=1;');
    }
};
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('questions', function (Blueprint $table) {
            $table->id();
            $table->uuid('uuid')->unique();
            $table->unsignedBigInteger('level_id');
            $table->unsignedBigInteger('grade_id');
            $table->unsignedBigInteger('difficulty_id');
            $table->string('json_id')->nullable()->unique();
            $table->json('metadata')->nullable();
            $table->string('version')->default('0.0.1');
            $table->integer('status')->default(1);
            $table->integer('evaluation_method')->default(1);
            $table->integer('checker_method')->nullable();
            $table->string('llm_evaluation_prompt_file_name')->nullable();
            $table->json('evaluation_response_format')->nullable();
            $table->integer('question_type')->default(1);
            $table->json('learning_requirement_json')->nullable();
            $table->string('learning_subject')->nullable()
                ->comment('科目 (学習要件)');
            $table->integer('learning_no')->nullable()
                ->comment('学習要件の番号');
            $table->text('learning_requirement')->nullable()
                ->comment('学習要件の内容');
            $table->text('learning_required_competency')->nullable()
                ->comment('必要水準');
            $table->text('learning_background')->nullable()
                ->comment('背景・補足');
            $table->string('learning_category')->nullable()
                ->comment('分類');
            $table->string('learning_grade_level')->nullable()
                ->comment('学年');
            $table->string('learning_url')->nullable()
                ->comment('URLリンク');
            $table->integer('order');
            $table->boolean('generated_by_llm')->default(false);
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('level_id')->references('id')->on('levels')->onDelete('cascade');
            $table->foreign('grade_id')->references('id')->on('grades')->onDelete('cascade');
            $table->foreign('difficulty_id')->references('id')->on('difficulties')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        DB::statement('SET FOREIGN_KEY_CHECKS=0;');
        Schema::dropIfExists('questions');
        DB::statement('SET FOREIGN_KEY_CHECKS=1;');
    }
};
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('question_set_questions', function (Blueprint $table) {
            $table->id();
            $table->uuid('uuid')->unique();
            $table->unsignedBigInteger('question_set_id');
            $table->unsignedBigInteger('question_id');
            $table->integer('order');
            $table->timestamps();

            $table->foreign('question_set_id')->references('id')->on('question_sets')->onDelete('cascade');
            $table->foreign('question_id')->references('id')->on('questions')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('question_set_questions');
    }
};

<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('user_question_sets', function (Blueprint $table) {
            $table->id();
            $table->uuid('uuid')->unique();
            $table->unsignedBigInteger('user_id');
            $table->unsignedBigInteger('question_set_id');
            $table->integer('status')->default(1);
            $table->decimal('score', 8, 2)->nullable();
            $table->timestamp('started_at')->nullable();
            $table->timestamp('finished_at')->nullable();
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('user_id')->references('id')->on('users')->onDelete('cascade');
            $table->foreign('question_set_id')->references('id')->on('question_sets')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('user_question_sets');
    }
};

<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('user_questions', function (Blueprint $table) {
            $table->id();
            $table->uuid('uuid')->unique();
            $table->unsignedBigInteger('user_question_set_id');
            $table->unsignedBigInteger('question_id');

            $table->json('metadata')->nullable();
            $table->string('version')->default('0.0.1');
            $table->integer('evaluation_method')->default(1);
            $table->integer('checker_method')->nullable();
            $table->string('llm_evaluation_prompt_file_name')->nullable();
            $table->json('evaluation_response_format')->nullable();
            $table->integer('question_type')->default(1);
            $table->json('learning_requirement_json')->nullable();
            $table->string('learning_subject')->nullable()
                ->comment('科目 (学習要件)');
            $table->integer('learning_no')->nullable()
                ->comment('学習要件の番号');
            $table->text('learning_requirement')->nullable()
                ->comment('学習要件の内容');
            $table->text('learning_required_competency')->nullable()
                ->comment('必要水準');
            $table->text('learning_background')->nullable()
                ->comment('背景・補足');
            $table->string('learning_category')->nullable()
                ->comment('分類');
            $table->string('learning_grade_level')->nullable()
                ->comment('学年');
            $table->string('learning_url')->nullable()
                ->comment('URLリンク');
            $table->boolean('generated_by_llm')->default(false);

            $table->integer('status')->default(1);
            $table->json('answer_data')->nullable();
            $table->longText('prompt_for_evaluation')->nullable();
            $table->timestamp('answered_at')->nullable();
            $table->integer('order');
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('user_question_set_id')->references('id')->on('user_question_sets')->onDelete('cascade');
            $table->foreign('question_id')->references('id')->on('questions')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('user_questions');
    }
};
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->uuid('uuid')->unique();
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password')->nullable();
            $table->rememberToken();
            $table->timestamps();
            $table->softDeletes();
        });

        Schema::create('password_reset_tokens', function (Blueprint $table) {
            $table->string('email')->primary();
            $table->string('token');
            $table->timestamp('created_at')->nullable();
        });

        Schema::create('sessions', function (Blueprint $table) {
            $table->string('id')->primary();
            $table->foreignId('user_id')->nullable()->index();
            $table->string('ip_address', 45)->nullable();
            $table->text('user_agent')->nullable();
            $table->longText('payload');
            $table->integer('last_activity')->index();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('users');
        Schema::dropIfExists('password_reset_tokens');
        Schema::dropIfExists('sessions');
    }
};

<?php

namespace App\Models\Grade;

use App\Models\BaseModel;
use App\Traits\HasOrder;
use App\Traits\UsesUuid;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

/**
 *
 *
 * @property-read mixed $created_at
 * @property-read mixed $deleted_at
 * @property-read mixed $updated_at
 * @property-read \App\Models\Grade\TFactory|null $use_factory
 * @property-read \Illuminate\Database\Eloquent\Collection<int, \App\Models\Grade\GradeTranslation> $translations
 * @property-read int|null $translations_count
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Grade newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Grade newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Grade onlyTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Grade query()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Grade withTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Grade withoutTrashed()
 * @property string $id
 * @property string|null $json_id リポジトリ上で管理するための一意識別子
 * @property string|null $description
 * @property string|null $memo
 * @property string $version
 * @property int $order
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Grade whereCreatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Grade whereDeletedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Grade whereDescription($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Grade whereId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Grade whereJsonId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Grade whereMemo($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Grade whereOrder($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Grade whereUpdatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Grade whereVersion($value)
 * @mixin \Eloquent
 */
class Grade extends BaseModel
{
    use HasOrder, UsesUuid, SoftDeletes, HasFactory;


    public $sortable = [
        'order_column_name' => 'order',
        'sort_when_creating' => true,
    ];

    protected $with = [
        'translations',
    ];

    /*=====================================================
    * リレーション
    *=====================================================*/

    /**
     * 翻訳テーブルとのリレーション
     */
    public function translations()
    {
        return $this->hasMany(GradeTranslation::class, 'grade_id');
    }
}
<?php

namespace App\Models\Section;

use App\Enums\SectionStatus;
use App\Models\BaseModel;
use App\Models\Level\Level;
use App\Models\Subject\Subject;
use App\Models\Unit\Unit;
use App\Traits\HasOrder;
use App\Traits\UsesUuid;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Database\Eloquent\Builder;

/**
 *
 *
 * @property string $id
 * @property string $subject_id
 * @property string $level_id
 * @property string|null $json_id
 * @property string|null $name
 * @property string|null $description
 * @property string|null $requirement
 * @property string|null $required_competency
 * @property string $version
 * @property int $status
 * @property int $order
 * @property string|null $learning_subject 科目 (学習要件) e.g. "Arithmetic"
 * @property int|null $learning_no 学習要件の番号 e.g. 10
 * @property string|null $learning_requirement 学習要件の内容 "Numbers and Calculation..."
 * @property string|null $learning_required_competency 必要水準 "Understand multiplication..."
 * @property string|null $learning_category 分類 e.g. "A", "B"
 * @property string|null $learning_grade_level 学年 e.g. "Grade 2"
 * @property string|null $learning_url URLリンク e.g. "https://docs.google.com/..."
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property \Illuminate\Support\Carbon|null $deleted_at
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section onlyTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section query()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereCreatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereDeletedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereDescription($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereJsonId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereLearningCategory($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereLearningGradeLevel($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereLearningNo($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereLearningRequiredCompetency($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereLearningRequirement($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereLearningSubject($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereLearningUrl($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereLevelId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereName($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereOrder($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereRequiredCompetency($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereRequirement($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereStatus($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereSubjectId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereUpdatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section whereVersion($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section withTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Section withoutTrashed()
 * @property-read bool $is_published
 * @property-read \App\Models\Section\TFactory|null $use_factory
 * @property-read \Illuminate\Database\Eloquent\Collection<int, \App\Models\Section\SectionTranslation> $translations
 * @property-read int|null $translations_count
 * @property-read \Illuminate\Database\Eloquent\Collection<int, Unit> $units
 * @property-read int|null $units_count
 * @method static \Database\Factories\Section\SectionFactory factory($count = null, $state = [])
 * @method static Builder<static>|Section ofStatus(\App\Enums\SectionStatus $status)
 * @method static Builder<static>|Section published()
 * @property string $grade_id
 * @method static Builder<static>|Section whereGradeId($value)
 * @mixin \Eloquent
 */
class Section extends BaseModel
{
    use HasOrder, SoftDeletes, UsesUuid, HasFactory;

    public $sortable = [
        'order_column_name' => 'order',
        'sort_when_creating' => true,
    ];

    /**
     * このModelの関連テーブルを常にロードするやつ
     * (翻訳を常時読み込みなど)
     */
    protected $with = [
        'translations',
    ];

    protected $fillable = [
        'uuid',
        'subject_id',
        'level_id',
        'json_id',
        'name',
        'description',
        'requirement',
        'required_competency',
        'version',
        'status',
        'order',
        'learning_category'
    ];

    /*=====================================================
     * リレーション
     *=====================================================*/

    /**
     * 翻訳テーブルとのリレーション
     */
    public function translations()
    {
        return $this->hasMany(SectionTranslation::class, 'section_id');
    }

    /**
     * Unitモデルとのリレーション例
     */
    public function units()
    {
        return $this->hasMany(Unit::class);
    }

    public function subject()
    {
        return $this->belongsTo(Subject::class);
    }

    public function level()
    {
        return $this->belongsTo(Level::class);
    }

    /*=====================================================
     * スコープ
     *=====================================================*/

    /**
     * 公開(PUBLISHED)になっているデータだけを抽出するクエリスコープ
     */
    public function scopePublished(Builder $query): Builder
    {
        return $query->where('status', SectionStatus::PUBLISHED->value);
    }

    /**
     * 任意のSectionStatusを指定して抽出するクエリスコープ
     * Section::ofStatus(SectionStatus::DRAFT)->get();
     */
    public function scopeOfStatus(Builder $query, SectionStatus $status): Builder
    {
        return $query->where('status', $status->value);
    }

    /*=====================================================
     * アクセサ/ミューテータ
     *=====================================================*/

    /**
     * 「このSectionは公開状態か？」をboolで返す
     */
    public function getIsPublishedAttribute(): bool
    {
        return $this->status === SectionStatus::PUBLISHED;
    }

    /*=====================================================
     * カスタムメソッド/Custom Method
     *=====================================================*/

    /**
     * 現在のlocaleに応じた翻訳を返す
     * なければfallback_localeの翻訳を返す
     */
    public function getTranslation(?string $locale = null): ?SectionTranslation
    {
        $locale = $locale ?: app()->getLocale();
        return $this->translations->firstWhere('locale', $locale)
            ?: $this->translations->firstWhere('locale', config('app.fallback_locale'));
    }
}
<?php

namespace App\Models\Question;

use App\Enums\QuestionSetStatus;
use App\Models\BaseModel;
use App\Models\Unit\Unit;
use App\Models\User\UserQuestionSetTranslation;
use App\Traits\HasOrder;
use App\Traits\UsesUuid;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\SoftDeletes;

/**
 *
 *
 * @property string $id
 * @property string|null $unit_id
 * @property string|null $json_id
 * @property string $version
 * @property int $status
 * @property int $order
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property \Illuminate\Support\Carbon|null $deleted_at
 * @property-read \Illuminate\Database\Eloquent\Collection<int, \App\Models\Question\QuestionSetTranslation> $translations
 * @property-read int|null $translations_count
 * @method static Builder<static>|QuestionSet newModelQuery()
 * @method static Builder<static>|QuestionSet newQuery()
 * @method static Builder<static>|QuestionSet onlyTrashed()
 * @method static Builder<static>|QuestionSet published()
 * @method static Builder<static>|QuestionSet query()
 * @method static Builder<static>|QuestionSet whereCreatedAt($value)
 * @method static Builder<static>|QuestionSet whereDeletedAt($value)
 * @method static Builder<static>|QuestionSet whereId($value)
 * @method static Builder<static>|QuestionSet whereJsonId($value)
 * @method static Builder<static>|QuestionSet whereOrder($value)
 * @method static Builder<static>|QuestionSet whereStatus($value)
 * @method static Builder<static>|QuestionSet whereUnitId($value)
 * @method static Builder<static>|QuestionSet whereUpdatedAt($value)
 * @method static Builder<static>|QuestionSet whereVersion($value)
 * @method static Builder<static>|QuestionSet withTrashed()
 * @method static Builder<static>|QuestionSet withoutTrashed()
 * @property-read \Illuminate\Database\Eloquent\Collection<int, \App\Models\Question\QuestionSetQuestion> $questionSetQuestions
 * @property-read int|null $question_set_questions_count
 * @property-read \Illuminate\Database\Eloquent\Collection<int, \App\Models\Question\Question> $questions
 * @property-read int|null $questions_count
 * @property string|null $generate_question_prompt
 * @property string|null $generate_question_prompt_file_name
 * @property int|null $llm_generation_status LLMの問題生成を許可するかどうか。LLMでの問題生成が可能な問題かどうかなどを管理
 * @property int $consecutive_generation_failures LLMの問題生成の連続失敗回数
 * @property int $generation_blocked LLMの問題生成の停止フラグ。失敗が続いた場合に停止
 * @property-read Unit|null $unit
 * @method static Builder<static>|QuestionSet whereConsecutiveGenerationFailures($value)
 * @method static Builder<static>|QuestionSet whereGenerateQuestionPrompt($value)
 * @method static Builder<static>|QuestionSet whereGenerateQuestionPromptFileName($value)
 * @method static Builder<static>|QuestionSet whereGenerationBlocked($value)
 * @method static Builder<static>|QuestionSet whereLlmGenerationStatus($value)
 * @mixin \Eloquent
 */
class QuestionSet extends BaseModel
{
    use HasOrder, UsesUuid, SoftDeletes;

    public $sortable = [
        'order_column_name' => 'order',
        'sort_when_creating' => true,
    ];
    protected $with = [
        'translations',
    ];

    /*=====================================================
    * Relation
    *=====================================================*/
    /**
     * 翻訳テーブルとのリレーション
     */
    public function translations()
    {
        return $this->hasMany(QuestionSetTranslation::class, 'question_set_id');
    }

    public function questionSetQuestions()
    {
        // PIVOT: question_set_questions
        return $this->hasMany(QuestionSetQuestion::class, 'question_set_id');
    }

    public function questions()
    {
        return $this->belongsToMany(Question::class, 'question_set_questions', 'question_set_id', 'question_id')
            ->withPivot('order')
            ->withTimestamps();
    }

    public function unit()
    {
        return $this->belongsTo(Unit::class);
    }

    /*=====================================================
 * カスタムメソッド/Custom Method
*=====================================================*/

    /**
     * 現在のlocaleに応じた翻訳を返す
     * なければfallback_localeの翻訳を返す
     */
    public function getTranslation(?string $locale = null): ?QuestionSetTranslation
    {
        $locale = $locale ?: app()->getLocale();
        return $this->translations->firstWhere('locale', $locale)
            ?: $this->translations->firstWhere('locale', config('app.fallback_locale'));
    }

    /*=====================================================
    * Scope
    *=====================================================*/

    /**
     * 公開(PUBLISHED)だけを絞り込むスコープ
     * local/staging 環境なら TEST_PUBLISHED もOK
     */
    public function scopePublished(Builder $query): Builder
    {
        return $query->where(function ($subQuery) {

            $subQuery->where('status', QuestionSetStatus::PUBLISHED->value);
            if (app()->environment(['local','staging'])) {
                $subQuery->orWhere('status', QuestionSetStatus::TEST_PUBLISHED->value);
            }
        });
    }
}
<?php

namespace App\Models\Question;

use App\Models\BaseModel;
use App\Models\Difficulty\Difficulty;
use App\Models\Grade\Grade;
use App\Models\Level\Level;
use App\Models\Skill\Skill;
use App\Traits\HasOrder;
use App\Traits\UsesUuid;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

/**
 *
 *
 * @property string $id
 * @property string $level_id
 * @property string $difficulty_id
 * @property string|null $json_id
 * @property string|null $metadata
 * @property string $version
 * @property int $status
 * @property int $question_type
 * @property string|null $learning_subject 科目 (学習要件) e.g. "Arithmetic"
 * @property int|null $learning_no 学習要件の番号 e.g. 10
 * @property string|null $learning_requirement 学習要件の内容 "Numbers and Calculation..."
 * @property string|null $learning_required_competency 必要水準 "Understand multiplication..."
 * @property string|null $learning_category 分類 e.g. "A", "B"
 * @property string|null $learning_grade_level 学年 e.g. "Grade 2"
 * @property string|null $learning_url URLリンク e.g. "https://docs.google.com/..."
 * @property int $order
 * @property int $generated_by_llm
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property \Illuminate\Support\Carbon|null $deleted_at
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question onlyTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question query()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereCreatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereDeletedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereDifficultyId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereGeneratedByLlm($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereJsonId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLearningCategory($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLearningGradeLevel($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLearningNo($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLearningRequiredCompetency($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLearningRequirement($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLearningSubject($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLearningUrl($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLevelId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereMetadata($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereOrder($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereQuestionType($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereStatus($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereUpdatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereVersion($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question withTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question withoutTrashed()
 * @property int $evaluation_method
 * @property int|null $checker_method
 * @property-read \Illuminate\Database\Eloquent\Collection<int, \App\Models\Question\QuestionTranslation> $translations
 * @property-read int|null $translations_count
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereCheckerMethod($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereEvaluationMethod($value)
 * @property string|null $llm_evaluation_prompt
 * @property string|null $evaluation_response_format
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLlmEvaluationPrompt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLlmEvaluationResponseFormat($value)
 * @property int|null $llm_evaluation_prompt_file_name
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLlmEvaluationPromptNumber($value)
 * @property string|null $learning_requirement_json
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLearningRequirementJson($value)
 * @property string|null $learning_background 背景・補足
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLearningBackground($value)
 * @property string $grade_id
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereGradeId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereEvaluationResponseFormat($value)
 * @property-read \Illuminate\Database\Eloquent\Collection<int, Skill> $skills
 * @property-read int|null $skills_count
 * @method static \Illuminate\Database\Eloquent\Builder<static>|Question whereLlmEvaluationPromptFileName($value)
 * @property-read Difficulty $difficulty
 * @property-read Grade $grade
 * @property-read Level $level
 * @mixin \Eloquent
 */
class Question extends BaseModel
{
    use HasOrder, UsesUuid, SoftDeletes;

    public $sortable = [
        'order_column_name' => 'order',
        'sort_when_creating' => true,
    ];
    protected $with = [
        'translations',
    ];

    /*=====================================================
    * リレーション
    *=====================================================*/

    /**
     * 翻訳テーブルとのリレーション
     */
    public function translations()
    {
        return $this->hasMany(QuestionTranslation::class, 'question_id');
    }
    public function skills()
    {
        return $this->belongsToMany(Skill::class, 'question_skill', 'question_id', 'skill_id')
            ->withPivot('order')
            ->withTimestamps();
    }

    public function level()
    {
        return $this->belongsTo(Level::class);
    }

    public function grade()
    {
        return $this->belongsTo(Grade::class);
    }
    public function difficulty()
    {
        return $this->belongsTo(Difficulty::class);
    }
}
<?php

namespace App\Models\Question;

use App\Models\BaseModel;
use App\Traits\HasOrder;
use App\Traits\UsesUuid;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

/**
 *
 *
 * @property string $id
 * @property string|null $unit_id
 * @property string|null $json_id
 * @property string $version
 * @property int $status
 * @property int $order
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property \Illuminate\Support\Carbon|null $deleted_at
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet onlyTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet query()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet whereCreatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet whereDeletedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet whereId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet whereJsonId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet whereOrder($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet whereStatus($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet whereUnitId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet whereUpdatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet whereVersion($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet withTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSet withoutTrashed()
 * @property string $question_set_id
 * @property string $question_id
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSetQuestion whereQuestionId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|QuestionSetQuestion whereQuestionSetId($value)
 * @property-read \App\Models\Question\Question|null $question
 * @mixin \Eloquent
 */
class QuestionSetQuestion extends BaseModel
{
    use UsesUuid, HasOrder;

    public $sortable = [
        'order_column_name' => 'order',
        'sort_when_creating' => true,
    ];

    /*=====================================================
    * Relation
    *=====================================================*/
    public function question()
    {
        return $this->belongsTo(Question::class, 'question_id');
    }
}
<?php

namespace App\Models\User;

use App\Enums\UserQuestionStatus;
use App\Models\BaseModel;
use App\Models\Question\QuestionSet;
use App\Traits\UsesUuid;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

/**
 *
 *
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet onlyTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet query()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet withTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet withoutTrashed()
 * @property string $id
 * @property string $user_id
 * @property string $question_set_id
 * @property string|null $score
 * @property string $started_at
 * @property string|null $finished_at
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property \Illuminate\Support\Carbon|null $deleted_at
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereCreatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereDeletedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereFinishedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereQuestionSetId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereScore($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereStartedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereUpdatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereUserId($value)
 * @property int $status
 * @property-read QuestionSet $questionSet
 * @property-read \App\Models\User\User $user
 * @property-read \Illuminate\Database\Eloquent\Collection<int, \App\Models\User\UserQuestion> $userQuestions
 * @property-read int|null $user_questions_count
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSet whereStatus($value)
 * @property-read \Illuminate\Database\Eloquent\Collection<int, \App\Models\User\UserQuestionSetTranslation> $translations
 * @property-read int|null $translations_count
 * @mixin \Eloquent
 */
class UserQuestionSet extends BaseModel
{
    use UsesUuid, SoftDeletes;

    protected $with = [
        'translations',
    ];

    protected array $iso8601Timestamps = [
        'started_at',
        'finished_at',
    ];

    /*=====================================================
    * リレーション
    *=====================================================*/
    public function questionSet()
    {
        return $this->belongsTo(QuestionSet::class);
    }

    public function userQuestions()
    {
        return $this->hasMany(UserQuestion::class);
    }

    public function user()
    {
        return $this->belongsTo(User::class);
    }

    /**
     * 翻訳テーブルとのリレーション
     */
    public function translations()
    {
        return $this->hasMany(UserQuestionSetTranslation::class, 'user_question_set_id');
    }


    /*=====================================================
     * カスタムメソッド/Custom Method
    *=====================================================*/

    /**
     * 現在のlocaleに応じた翻訳を返す
     * なければfallback_localeの翻訳を返す
     */
    public function getTranslation(?string $locale = null): ?UserQuestionSetTranslation
    {
        $locale = $locale ?: app()->getLocale();
        return $this->translations->firstWhere('locale', $locale)
            ?: $this->translations->firstWhere('locale', config('app.fallback_locale'));
    }

    /**
     * 次の未開始問題 (status=NOT_START) を order 昇順 で1件取得して返す
     * なければ null
     * @return UserQuestion|null
     */
    public function getNextNotStartedQuestion(): ?UserQuestion
    {
        return $this->userQuestions()
            ->where('status', UserQuestionStatus::NOT_START->value)
            ->orderBy('order', 'asc')
            ->first();
    }

}
<?php

namespace App\Models\User;

use App\Models\BaseModel;
use App\Models\Question\Question;
use App\Traits\UsesUuid;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

/**
 *
 *
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion onlyTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion query()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion withTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion withoutTrashed()
 * @property string $id
 * @property string $user_question_set_id
 * @property string $question_id
 * @property int $status
 * @property string|null $answer_data
 * @property string|null $answered_at
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property \Illuminate\Support\Carbon|null $deleted_at
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereAnswerData($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereAnsweredAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereCreatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereDeletedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereQuestionId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereStatus($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereUpdatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereUserQuestionSetId($value)
 * @property-read Question|null $question
 * @property-read \App\Models\User\UserQuestionSet|null $userQuestionSet
 * @property string|null $metadata
 * @property string $version
 * @property int $evaluation_method
 * @property int|null $checker_method
 * @property int|null $llm_evaluation_prompt_file_name
 * @property string|null $evaluation_response_format
 * @property int $question_type
 * @property string|null $learning_requirement_json
 * @property string|null $learning_subject 科目 (学習要件)
 * @property int|null $learning_no 学習要件の番号
 * @property string|null $learning_requirement 学習要件の内容
 * @property string|null $learning_required_competency 必要水準
 * @property string|null $learning_background 背景・補足
 * @property string|null $learning_category 分類
 * @property string|null $learning_grade_level 学年
 * @property string|null $learning_url URLリンク
 * @property int $generated_by_llm
 * @property int $order
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereCheckerMethod($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereEvaluationMethod($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereGeneratedByLlm($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereLearningBackground($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereLearningCategory($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereLearningGradeLevel($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereLearningNo($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereLearningRequiredCompetency($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereLearningRequirement($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereLearningRequirementJson($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereLearningSubject($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereLearningUrl($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereLlmEvaluationPromptNumber($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereLlmEvaluationResponseFormat($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereMetadata($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereOrder($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereQuestionType($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereVersion($value)
 * @property string|null $prompt_for_evaluation
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion wherePromptForEvaluation($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereEvaluationResponseFormat($value)
 * @property-read \Illuminate\Database\Eloquent\Collection<int, \App\Models\User\UserQuestionTranslation> $translations
 * @property-read int|null $translations_count
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestion whereLlmEvaluationPromptFileName($value)
 * @mixin \Eloquent
 */
class UserQuestion extends BaseModel
{
    use UsesUuid, SoftDeletes;

    protected $with = [
        'translations',
    ];

    protected array $iso8601Timestamps = [
        'answered_at',
    ];


    /*=====================================================
     * リレーション
     *=====================================================*/
    public function userQuestionSet()
    {
        return $this->belongsTo(UserQuestionSet::class, 'user_question_set_id');
    }

    public function question()
    {
        return $this->belongsTo(Question::class, 'question_id');
    }

    /**
     * 翻訳テーブルとのリレーション
     */
    public function translations()
    {
        return $this->hasMany(UserQuestionTranslation::class, 'user_question_id');
    }


    /*=====================================================
    * カスタムメソッド/Custom Method
    *=====================================================*/

    /**
     * 現在のlocaleに応じた翻訳を返す
     * なければfallback_localeの翻訳を返す
     */
    public function getTranslation(?string $locale = null): ?UserQuestionTranslation
    {
        $locale = $locale ?: app()->getLocale();
        return $this->translations->firstWhere('locale', $locale)
            ?: $this->translations->firstWhere('locale', config('app.fallback_locale'));
    }
}
<?php

namespace App\Models\User;

// use Illuminate\Contracts\Auth\MustVerifyEmail;
use App\Traits\Iso8601TimestampAccessors;
use App\Traits\UsesUuid;
use Carbon\Carbon;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens;

/**
 *
 *
 * @property string $id
 * @property string $email
 * @property \Illuminate\Support\Carbon|null $email_verified_at
 * @property string|null $password
 * @property string|null $remember_token
 * @property string|null $google_id
 * @property string|null $google_auth_response_json
 * @property string|null $google_auth_picture
 * @property string|null $google_auth_avatar_url
 * @property string|null $google_auth_full_name
 * @property string|null $family_name
 * @property string|null $given_name
 * @property string|null $family_name_kana
 * @property string|null $given_name_kana
 * @property int|null $gender
 * @property string|null $mst_prefecture_id
 * @property string|null $user_name
 * @property string|null $date_of_birth
 * @property string|null $phone_number
 * @property string|null $invitation_code
 * @property int $status
 * @property int $goal_type
 * @property string|null $last_sign_in_at
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property \Illuminate\Support\Carbon|null $deleted_at
 * @property-read \App\Models\User\TFactory|null $use_factory
 * @property-read \Illuminate\Notifications\DatabaseNotificationCollection<int, \Illuminate\Notifications\DatabaseNotification> $notifications
 * @property-read int|null $notifications_count
 * @property-read \Illuminate\Database\Eloquent\Collection<int, \Laravel\Sanctum\PersonalAccessToken> $tokens
 * @property-read int|null $tokens_count
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User onlyTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User query()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereCreatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereDateOfBirth($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereDeletedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereEmail($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereEmailVerifiedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereFamilyName($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereFamilyNameKana($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereGender($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereGivenName($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereGivenNameKana($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereGoogleAuthAvatarUrl($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereGoogleAuthFullName($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereGoogleAuthPicture($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereGoogleAuthResponseJson($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereGoogleId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereInvitationCode($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereLastSignInAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereMstPrefectureId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User wherePassword($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User wherePhoneNumber($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereRememberToken($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereStatus($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereUpdatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereUserName($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User withTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User withoutTrashed()
 * @property string|null $token_password
 * @property string|null $expires_at_token_password
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereExpiresAtTokenPassword($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User whereTokenPassword($value)
 * @mixin \Eloquent
 */
class User extends Authenticatable
{
    /** @use HasFactory<\Database\Factories\UserFactory> */
    use HasFactory, UsesUuid,  Notifiable, HasApiTokens, SoftDeletes, Iso8601TimestampAccessors;

    protected array $iso8601Timestamps = [
        'last_sign_in_at',
        'email_verified_at',
    ];

    /**
     * The attributes that are mass assignable.
     *
     * @var list<string>
     */
    protected $fillable = [
        'uuid',
        'name',
        'email',
        'password',
        'user_name',
        'google_auth_response_json',
        'google_auth_picture',
        'google_auth_avatar_url',
        'google_auth_full_name',
        'email_verified_at',
        'last_sign_in_at',
        'goal_type',
        'line_id',
        'line_auth_picture',
        'line_auth_avatar_url',
        'line_auth_full_name',
    ];

    /**
     * The attributes that should be hidden for serialization.
     *
     * @var list<string>
     */
    protected $hidden = [
        'password',
        'remember_token',
    ];

    /**
     * Get the attributes that should be cast.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'password' => 'hashed',
        ];
    }
}


<?php

namespace App\Enums;

enum UserQuestionSetStatus: int
{
    case NOT_START = 1;
    case COMPLETE = 100;
    case PROGRESS = 300;
    case SKIP = 400;

    public function description(): string
    {
        return match ($this) {
            self::NOT_START => 'Not Started',
            self::COMPLETE => 'Complete',
            self::PROGRESS => 'Progress',
            self::SKIP => 'skip',
        };
    }

    public static function getKeyForValue(int $value): ?string
    {
        foreach (UserQuestionSetStatus::cases() as $case) {
            if ($case->value === $value) {
                return $case->name;
            }
        }

        return null;
    }

    /**
     * 文字列から QuestionStatus を取得
     * @param string $statusString 例: "DRAFT", "PUBLISHED" など
     * @return self
     */
    public static function fromString(string $statusString): self
    {
        return match (strtoupper($statusString)) {
            'NOT_START'          => self::NOT_START,
            'COMPLETE'      => self::COMPLETE,
            'PROGRESS'         => self::PROGRESS,
            'SKIP'         => self::SKIP,
            default => throw new \InvalidArgumentException("Unknown status string: {$statusString}")
        };
    }
}

```
