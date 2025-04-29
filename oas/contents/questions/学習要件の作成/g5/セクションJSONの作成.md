以下の{# 学習指導要領}は、{#文部科学省、学習指導要領}に従って学習水準をリスト化したものです。
{参考JSON} は {参考の学習要件} をJSON構造にしたものです。
{参考JSON}と同じように　｛学習指導要綱｝ をJSON化してください

# 対象
小学5年生 算数

# 条件
１）json_id は、セクションIDと同じにしてください

２）以下の要素は以下と同様にしてください
"subject_id": "sub_001",
"level_id": "lev_005",
"grade_id": "gra_005",
"learning_category": "A",
"status": "PUBLISHED",
"version": "1.0.0",
"subject": {
"ja": "算数",
"en": "Mathematics"
},

３）order は、100 飛ばしで。

４）nameは　セクション　と同じ

５）units 以下
unit_id は、unit_s1_g5_sec100_UnitID としてください。例えば　sec_s1_g5_100　の Unit ID u100 なら　unit_s1_g5_sec100_100 となる。

６）requirement　は　要件、required_competency　は　必要水準、background　は　背景

７）省略せずに漏れなく全部出力してください

８）セクションIDの  g4_ 以降の値は order を表しています。　例えば、 sec_s1_g5_100 sec_s1_g5_300 sec_s1_g5_200 だとしたら order は、sec_s1_g5_100　sec_s1_g5_200 sec_s1_g5_300 が正しいです。

９）{参考JSON}と同じように　｛"ja":"","en":""｝のようにオブジェクトで日本語と英語に多言語化する必要があります。en 空欄で大丈夫です。

-- 学習指導要綱
No	学年	カテゴリ	サブカテゴリー	セクション	セクションID	要件	Unit ID	必要水準	補足・背景
100	小5	A 数と計算	数の概念	整数・分数の概念	sec_s1_g5_800	観点を決めることによる整数の類別や数の構成	u100	・整数を奇数・偶数，約数・倍数などの観点で分類できる。	・因数分解や最小公倍数の考えにもつながる基礎力。
101	小5	A 数と計算	数の概念	整数と小数のしくみをまとめよう	sec_s1_g5_100	整数・小数を位ごとに分解／再構成し0.001単位までの個数で表現できる。	u100	・「1×a+0.1×b+…」の形で数を構成・分解し，0.001 を基準に大きさを表せる。	位取りと単位量に基づく数構成力を確立し，小数計算・単位換算の基礎を固める。
102	小5	A 数と計算	数の概念	整数と小数のしくみをまとめよう	sec_s1_g5_100	10ⁿ倍と10⁻ⁿ倍（1/10, 1/100, 1/1000）の桁移動規則を説明し計算に適用できる。	u200	・小数点の右左移動で×10,×100,…と÷10,÷100,…を暗算・筆算に正しく活用できる。	乗除演算・概算の効率化に不可欠な位取り移動の因果関係を数直線や表で確認させる。
103	小5	A 数と計算	数の概念	整数と小数のしくみをまとめよう	sec_s1_g5_100	位と桁数を基に小数第3位までの大小比較や条件付き数づくりを行い論理的に説明できる。	u300	・<,> や数直線で大小を判断し，指定カードで最大・最小・近似値を構成できる。	位の価値と大小関係を統合的に活用し，データ読解や発展課題（カード並べ）に対応する力を養う。
104	小5	A 数と計算	数の概念	整数・分数の概念	sec_s1_g5_1700	数の相対的大きさ	u100	・10万と1000万のように，さらに大きい数の比較を位の仕組みで的確に行える。	・数の大小比較の考え方を拡張し，社会での大きい数（予算など）にも目を向ける。
105	小5	A 数と計算	数の概念	整数・分数の概念	sec_s1_g5_1000	分数の相約及び大小関係	u100	・分数を約分し，既約分数にする手順を理解し大小を判断できる。	・分数計算の効率化に直結するため，しっかりした習得が必要。
106	小5	A 数と計算	数の概念	整数・分数の概念	sec_s1_g5_900	分数と整数、小数の関係	u100	・2/5=0.4など，分数⇔小数の変換を自由に行える。	・具体例(1/2=0.5など)から一般化し，小数は分数の一形態と捉えさせる。
107	小5	A 数と計算	数の概念	整数・分数の概念	sec_s1_g5_900	除法の結果の分数による表現	u200	・3÷2=3/2のように，わり算の商を分数で表せる。	・中学以降の「分数の乗除」や文字式への接続を見据えて理解させる。
108	小5	A 数と計算	計算の意味・方法	計算の意味と方法	sec_s1_g5_400	乗法の意味の拡張（小数）	u100	・小数×小数，小数÷小数も，整数の繰り返し加法や等分除の延長線で捉えられる。	・桁と小数点の扱いを復習しつつ，より複雑な事例に挑戦する。
109	小5	A 数と計算	計算の意味・方法	計算の意味と方法	sec_s1_g5_400	数の乗法（小数×小数）	u200	・(2.5)×(1.2)や(3.6)÷(1.2)などで筆算を正しく行い，答えを導ける。	・4年生までの延長として桁数の増加にも対応し，習熟度を高める。
小5	A 数と計算	計算の意味・方法	計算の意味と方法	sec_s1_g5_500	除法の意味の拡張（小数）	u100	・小数×小数，小数÷小数も，整数の繰り返し加法や等分除の延長線で捉えられる。	・桁と小数点の扱いを復習しつつ，より複雑な事例に挑戦する。
小5	A 数と計算	計算の意味・方法	計算の意味と方法	sec_s1_g5_500	数の除法（小数÷小数）	u200	・(2.5)×(1.2)や(3.6)÷(1.2)などで筆算を正しく行い，答えを導ける。	・4年生までの延長として桁数の増加にも対応し，習熟度を高める。
110	小5	A 数と計算	計算の意味・方法	計算の意味と方法	sec_s1_g5_1000	異分母分数の加法及び減法	u200	・分母が異なる場合でも，共通分母をとって正確に演算できる。	・分数計算の幅が大きく広がり，中学の計算につながる重要なステップ。
111	小5	A 数と計算	計算の意味・方法	計算の意味と方法	sec_s1_g5_2000	交換法則、結合法則、分配法則	u100	・(a + b)(c + d)=ac+ad+bc+bdなどを分配法則で整理できる。	・本格的な式展開の基本となるため，例題を通じて概念をしっかり浸透。
112	小5	A 数と計算	計算の意味・方法	計算の意味と方法	sec_s1_g5_500	乗法・除法の結果の演算法	u300	・(a / b)÷(c / d)のような分数の乗除に備え，小数演算の手順を堅固にする。	・この段階で小数計算を自由に扱える状態が望ましい。
113	小5	A 数と計算	計算の意味・方法	計算の意味と方法	sec_s1_g5_2000	計算の工夫や確かめ	u200	・大きい数字でも，まとめ算や逆算などを活用して自力検証できる。	・自己点検の習慣化を図り，複数手段を使う計算力を定着させる。
114	小5	A 数と計算	式・関係	式・関係と日常活用	sec_s1_g5_900	数量の関係を表す式（簡潔・一般的）	u300	・分数や小数を含む文章題を文字や記号で整理し，式にまとめられる。	・複雑な問題でも式化する思考力を育て，中学の方程式学習への架け橋に。
115	小5	A 数と計算	日常生活への活用	式・関係と日常活用	sec_s1_g5_800	整数の類別などの活用	u200	・約数や倍数を使い，人数分けや割り切れる数の活用を日常場面でイメージできる。	・組体操や行列づくり，費用の割り勘などの具体的事例を提示。
116	小5	A 数と計算	日常生活への活用	式・関係と日常活用	sec_s1_g5_900	小数や分数の計算の活用	u400	・値引き計算やレシピの分量調整などで小数・分数計算を実際に応用できる。	・実生活の体感を通じ，計算への抵抗感をなくし活用意識を高める。
117	小5	A 数と計算	統計的な基礎	平均	sec_s1_g5_1100	平均の意味と求め方	u100	・複数の数の合計を個数で割り代表値として平均を求められる。	・身長やテスト点など実データで平均を計算し全体的な傾向を捉える基礎にする。
118	小5	A 数と計算	プログラミング	手続き処理	sec_s1_g5_1800	指定された条件から実行結果を理解する	u100	・基本的な指示の流れをトレースし，期待される出力を把握できる。	・簡単なアルゴリズムを追う中で論理力を育て，手順を正確に実行する意識を高める。
119	小5	A 数と計算	プログラミング	手続き処理	sec_s1_g5_1800	実行結果から条件を理解する	u200	・実行後の変化を見て，どんな条件や判断があったかを推測できる。	・結果から逆に処理を推論し，条件分岐や繰り返しの仕組みに慣れる。
120	小5	A 数と計算	計算の意味・方法	規則性	sec_s1_g5_1900	くり返すパターンの発見	u100	・○△□など繰り返し図形や数列を見分け、次や途中の要素を推定できる。	・パターン認識を図や表と紐付ける基礎として扱い、アルゴリズム的思考へ繋げる。
121	小5	A 数と計算	計算の意味・方法	規則性	sec_s1_g5_1900	周期を用いた要素の位置づけ	u200	・一定の周期で繰り返される並びに対し、「n番目には何が来るか」を表や簡単な式で求められる。	・中学以降の一次関数へ接続する前段階として、対応関係を捉える力を涵養する。
122	小5	A 数と計算	計算の意味・方法	規則性	sec_s1_g5_1900	等差数列の仕組み	u300	・隣り合う項の差が一定になる数列を捉え、表や簡単な式(n番目=...)で求められる。	・買い物の残金推移や日々の増減など、身近な場面で差が一定になる例を検証することで興味を深める。
123	小5	A 数と計算	計算の意味・方法	小数の倍	sec_s1_g5_100	基準量に対する小数倍の計算	u400	・基準量に小数を掛けて倍の量を正確に計算できる（例：3.2の2.5倍）。	・割合を小数で扱う前提として増減や縮尺など実生活の場面で頻出する。
155	小5	B 図形	図形の概念について理解し，その性質について考察すること	平面・立体図形の基礎概念	sec_s1_g5_1300	生活の中で平面の広さの違いに興味をもち，比較する	u100	机や床，ノートの表紙など，身近な物の広さを実際に紙やマットで覆って比べられる	「どちらのほうが広い？」「何枚必要？」といった具体的な問いから，面積を数値化・可視化する楽しさに気づき，学習意欲を高める
187	小5	B 図形	図形の概念について理解し，その性質について考察すること	平面・立体図形の基礎概念	sec_s1_g5_1500	多角形，正多角形	u100	・いろいろな多角形を見分け，正多角形において辺や角がすべて等しいことを把握できる。	・五角形，六角形など多角形を具体例で示し，辺と角の性質を探求させる。
188	小5	B 図形	図形の概念について理解し，その性質について考察すること	平面・立体図形の基礎概念	sec_s1_g5_700	三角形の三つの角，四角形の四つの角の大きさの和	u100	・三角形の内角和が180°，四角形の内角和が360°であることを理解し活用できる。	・折り紙や切り貼りで角を重ねる活動を通じて，なぜその和になるかを体感させる。二等辺三角形や正方形からなどの法則を理解し角度を答えられる
189	小5	B 図形	図形の概念について理解し，その性質について考察すること	平面・立体図形の基礎概念	sec_s1_g5_1500	直径と円周との関係	u200	・円周は直径の約3.14倍（円周率）であることを測定と比較で理解する。	・円周率の概念への入り口として，円周をひもで測り，直径で割る活動が有効。
190	小5	B 図形	図形の概念について理解し，その性質について考察すること	円周率	sec_s1_g5_1500	円周率の意味と計算への活用	u300	・円周率3.14を用いて円周や直径のどちらかを計算で求められる。	・日常の円形物の測定結果と計算値を比較し近似値の扱いに慣れる。
191	小5	B 図形	図形の概念について理解し，その性質について考察すること	平面・立体図形の基礎概念	sec_s1_g5_1600	角柱，円柱	u100	"・角柱や円柱の面・辺・頂点などの特徴を把握できる。
・底面が多角形か円かで分類できる。"	・実物（ペン立てや菓子の筒など）を観察し，柱の構造を理解させる。
192	小5	B 図形	図形の構成の仕方について考察すること	図形の構成	sec_s1_g5_1500	正多角形	u400	・正多角形をコンパスや定規を使って作図したり，紙工作で形作ったりできる。	・等分を繰り返す作図の方法などを学び，規則的に辺と角を作る技能を養う。
193	小5	B 図形	図形の構成の仕方について考察すること	図形の構成	sec_s1_g5_600	合同な図形	u100	・重ね合わせなどによって，2つの図形が合同であるかを確かめられる。	・対応する辺・角が等しいことを視覚的に捉え，合同の意味を深く理解する。
194	小5	B 図形	図形の構成の仕方について考察すること	図形の構成	sec_s1_g5_1600	柱体の見取り図，展開図	u200	・角柱や円柱の展開図を描き，組み立てられる。	・立体図形の表面積を考える前段階として展開図づくりを活用する。
195	小5	B 図形	図形の計量の仕方について考察すること	図形の計量	sec_s1_g5_1300	三角形，平行四辺形，ひし形，台形の求積	u200	・それぞれの面積公式を導き，実際に計算できる。	・三角形＝平行四辺形の半分等，図形変形を通じて公式を納得させる。
196	小5	B 図形	図形の計量の仕方について考察すること	図形の計量	sec_s1_g5_200	立方体，直方体の求積	u100	・立方体，直方体の体積を求められる（底面積×高さ など）。	・ブロックを詰める活動で体積を実感し，式との対応を明確化する。
197	小5	B 図形	図形の性質を日常生活に生かすこと	図形の活用	sec_s1_g5_1500	正多角形	u500	・建物やデザインでの正多角形の利用例を理解し，その規則性に着目できる。	・タイル敷きや装飾模様などの身近な例で多角形の美しさや機能を感じさせる。
198	小5	B 図形	図形の性質を日常生活に生かすこと	図形の活用	sec_s1_g5_1600	角柱，円柱	u300	・柱形状の安定性や収納のしやすさなどを生活に活かせる。	・円柱の転がりやすさ，角柱の積み重ねやすさ等，実生活の利便性を考えさせる。
199	小5	B 図形	図形の構成の仕方について考察すること	図形のしきつめ	sec_s1_g5_600	正多角形による平面のしきつめ	u200	・正三角形・正方形・正六角形で平面が隙間なく敷き詰められる理由を説明できる。	・壁紙やタイル模様を観察し対称性と合同の視点から美しい配置を考察する。
小5	B 図形	図形の計量	立体図形の体積	sec_s1_g5_200	立方体 1 cm³・1 m³ モデルを使い体積の単位(cm³ m³)を理解し比較できる。	u200	・1 cm角ブロックを詰める操作で体積を捉え1000 cm³ = 1 L など単位換算を説明できる。	・面積から体積への拡張とL(かさ)との接続で量感を養う。
小5	B 図形	図形の計量	立体図形の体積	sec_s1_g5_200	直方体・立方体の体積を「縦×横×高さ」の公式で計算できる。	u300	・辺長が整数/小数(0.1 単位)でも筆算・暗算で体積を求められる。	・乗法を3次元へ拡張し公式の意味と辺の対応を定着させる。
小5	B 図形	図形の計量	立体図形の体積	sec_s1_g5_200	複合直方体や板厚のある箱の容積を求め m³↔L↔cm³ の変換を活用できる。	u400	・内寸算出後に複数体積を加減し450 cm³, 5 L などを正確に導出できる。	・水槽や収納箱など実課題でモデリングと単位換算の応用力を高める。
235	小5	C 変化と関係	伴って変わる二つの数量の変化や対応の特徴を考察すること	比例の基礎	sec_s1_g5_300	簡単な場合についての比例の関係	u100	・一方が2倍，3倍…となれば他方も同じ倍になる，という比例の特徴を把握できる。	・面積や料金計算など，日常例を使い「一定の割合で増える/減る」仕組みを体感。
236	小5	C 変化と関係	ある二つの数量の関係と別の二つの数量の関係を比べること	単位量当たりの大きさ	sec_s1_g5_1200	単位量当たりの大きさ	u100	・1個当たりの値段や1cm²当たりの重さなどを求め，他の対象と比較できる。	・割り算の概念と関連付け，「何あたり」「1つあたり」の見方を身に付ける。
237	小5	C 変化と関係	ある二つの数量の関係と別の二つの数量の関係を比べること	割合・百分率	sec_s1_g5_1200	割合，百分率	u200	・全体に対する部分の比率を小数やパーセントで表現し，比較できる。	・ポスター作りや統計データの分析で，割合表現が有効であることを実感させる。
238	小5	C 変化と関係	二つの数量の関係の考察を日常生活に生かすこと	比例の基礎	sec_s1_g5_300	簡単な場合についての比例の関係	u200	・商品の個数に応じた料金や，距離に応じた時間を比例関係として捉えられる。	・乗法・除法の延長としての「比例」を，買い物や移動の場面で実感。
239	小5	C 変化と関係	二つの数量の関係の考察を日常生活に生かすこと	単位量当たりの大きさ	sec_s1_g5_1200	単位量当たりの大きさ	u300	・「1平方メートル当たりの重さ」「1リットル当たりの値段」などを使いこなし比較検討できる。	・生活のあらゆるところで「1あたり」を意識し，合理的に判断する力を育む。
240	小5	C 変化と関係	二つの数量の関係の考察を日常生活に生かすこと	割合・百分率	sec_s1_g5_1200	割合，百分率	u400	・セール価格や成績評価などで百分率を見かけたときに，もとの数量や比較を理解できる。	・身近な統計資料（割合グラフなど）を読解し，情報活用力を高める。
272	小5	D データの活用	目的に応じてデータを収集，分類整理し，結果を適切に表現すること	統計的な問題解決とグラフ	sec_s1_g5_2100	統計的な問題解決の方法	u100	・「問題設定→データ収集→分析→結論」の流れを簡単な事例で体験する。	・探究的学習と結び付け，課題を持ってデータを扱う姿勢を育む。
273	小5	D データの活用	目的に応じてデータを収集，分類整理し，結果を適切に表現すること	統計的な問題解決とグラフ	sec_s1_g5_1400	円グラフや帯グラフ	u100	・全体に対する部分の割合を可視化し，複数の項目比較をしやすくできる。	・割合の学習（パーセント）と関連し，グラフ作成を精度高く行う。
274	小5	D データの活用	目的に応じてデータを収集，分類整理し，結果を適切に表現すること	統計的な問題解決とグラフ	sec_s1_g5_1100	測定値の平均	u200	・いくつかの測定値から平均を求め，代表値として使う意味を理解できる。	・身長や走るタイムなど，個人差のあるデータで平均の概念を実感させる。
275	小5	D データの活用	統計データの特徴を読み取り判断すること	統計データの特徴	sec_s1_g5_2100	結論についての多面的な考察	u200	・得られた平均値やグラフの結果を見て，「例外はないか」「別の解釈はできないか」を考える。	・安易に結論を出さず，データに潜む多様性に目を向けさせる。
276	小5	D データの活用	統計データの特徴を読み取り判断すること	統計データの特徴	sec_s1_g5_1400	円グラフや帯グラフ	u1200	・円グラフや帯グラフを読み取り，どの項目がどれくらいの割合を占めるか説明できる。	・特に割合の大きい項目を見つけるなど，全体との対比で特徴を述べさせる。
277	小5	D データの活用	統計データの特徴を読み取り判断すること	統計データの特徴	sec_s1_g5_1100	測定値の平均	u300	・平均値が「全体を代表する値」であることを踏まえ，ばらつきとの違いを認識できる。	・「全員の身長を足して人数で割る」など計算手順を再確認し，活用場面を広げる。






-- 参考の学習要件
No	学年	カテゴリ	サブカテゴリー	セクション	セクションID	要件	Unit ID	必要水準	補足・背景
33	小3	A 数と計算	数の概念	大きな数の概念と活用	sec_s1_g3_100	万の単位、1億などの比べ方や表し方	u100	・1万，10万，100万，1億などの上位の位を正しく読み書きできる。	・さらに大きな数への理解を深め，数のスケールを体感できるようにする。
34	小3	A 数と計算	数の概念	大きな数の概念と活用	sec_s1_g3_100	大きな数の相対的大きさ	u200	 1万と1億の差を人口やお金の桁などの具体例で説明できる	・単なる記号操作ではなく，日常との結び付きで「いくつ分違うか」を考えさせる。
35	小3	A 数と計算	数の概念	分数と少数	sec_s1_g3_700	小数	u100	・少数の意味を理解できる	・少数、整数の分類や比較などの問題を通して意味や概念を理解する
36	小3	A 数と計算	数の概念	分数と少数	sec_s1_g3_700	簡単な分数	u200	・分数の意味を理解できる	・整数、分数の分類や比較などの問題を通して意味や概念を理解する
37	小3	A 数と計算	数の概念	分数と少数	sec_s1_g3_700	小数や簡単な分数の大きさの比較	u300	・少数と少数、分数と分数の比較。0.5や0.3など小数第1位を分数(1/2, 3/10など)と対応づけられる。	・小数と分数が互いに表し合えることを具体的に示す（長さや重さなど）。
38	小3	A 数と計算	計算の意味・方法	割り算	sec_s1_g3_300	わり算の意味	u400	・わり算には「等分除」と「包含除」があることを理解し，簡単な問題を式にできる。	・同じ数ずつ分けるか，何回分になるかを区別し，ミスを防ぐ。
39	小3	A 数と計算	計算の意味・方法	割り算	sec_s1_g3_300	あまりのあるわり算	u300	・あまりのある除法を理解し、商とあまりを正しく表せる。	・余りのある除法(13÷4=3あまり1等)を正しく行い、余りが除数未満であることを認識できる。
40	小3	A 数と計算	計算の意味・方法	大きな数の概念と活用	sec_s1_g3_100	3位数や4位数の加法及び減法	u300	・3～4桁どうしの足し算・引き算を繰り上がり・繰り下がり含め正確に計算できる。	・筆算の手順をしっかり確立させる。
41	小3	A 数と計算	計算の意味・方法	大きな数の概念と活用	sec_s1_g3_100	2位数や3位数の乗法	u400	・2桁×2桁や3桁×1桁など，大きめのかけ算を正確に行える。	・計算量が増えるため，桁のそろえ方や筆算手順を重点的に練習する。
42	小3	A 数と計算	計算の意味・方法	割り算	sec_s1_g3_300	1位数などのわり算	u100	・1桁わり算(27÷3等)を九九を用いてスムーズに解ける。	・2桁以上の除法に進む基礎固め。逆算としてかけ算との結び付きも確認。
43	小3	A 数と計算	計算の意味・方法	割り算	sec_s1_g3_300	わり算とかけ算や引き算との関係	u200	・わり算が乗法/減法と逆演算の関係にあることを理解し，検算に利用できる。	・問題解決で「かけ算で確かめる」などの活用を経験させる。
44	小3	A 数と計算	計算の意味・方法	分数と少数	sec_s1_g3_700	小数（10の位）の加法及び減法	u400	・0.5 + 0.3など，小数第1位までの加減計算を筆算または暗算で行える。	・小数点の位置合わせの重要性を強調し，整数との違いを意識させる。
45	小3	A 数と計算	計算の意味・方法	分数と少数	sec_s1_g3_700	簡単な分数の加法及び減法	u500	・分母が同じな場合（1/2+1/2等）の計算を誤りなく行える。	・分割のイメージを図で示し，抽象化に慣れさせる初歩段階。
46	小3	A 数と計算	計算の意味・方法	計算の決まりや工夫	sec_s1_g3_900	交換法則、結合法則、分配法則	u100	・a×(b + c)=ab+acなど，乗法・加法におけるこれらの法則を式で表せる。	・高学年での式操作に備え，体系的に理解させる。
47	小3	A 数と計算	計算の意味・方法	計算の決まりや工夫	sec_s1_g3_900	加法、減法及び乗法の結果の見積もり	u200	・計算前におおよその値を把握し，答えの妥当性を判断できる。	・日常での買い物や分量計算など，誤差の確認に役立つ習慣化が狙い。
48	小3	A 数と計算	計算の意味・方法	計算の決まりや工夫	sec_s1_g3_900	計算の工夫や確かめ	u300	・逆算や暗算，そろばんなどを状況に応じて使い分け，結果を検証できる。	・複数の手段で自己点検することで，計算力と自信を高める。
49	小3	A 数と計算	計算の意味・方法	規則性	sec_s1_g3_1200	"・図形や記号，数字などが繰り返すパターンを見いだし，次にくる要素や途中の要素を推測できる。
"	u100	"・簡単なくり返しパターン（例：○→△→□→○→△→□…）を見つけ，周期を言葉や式で表すことができる。
"	階段を上がる段数や毎日同じ枚数のシールを貼るなど，実生活で一定ずつ増えていく事象を取り上げるとイメージしやすい。
50	小3	A 数と計算	計算の意味・方法	規則性	sec_s1_g3_1200	・周期を把握して，「何番目には何がくるか」を判断できる。	u200	・位置と要素の対応（n番目→要素）を整理する基礎力を養う。	文字式を導入しなくても，図や表を使って視覚化しながら「n番目＝…」と示すプロセスを経験させると定着しやすい。
51	小3	A 数と計算	計算の意味・方法	規則性	sec_s1_g3_1200	"・一定の差で増減する数列（等差数列）を見いだし，次にくる数や特定の番目の値を求める。
"	u300	"・隣り合う数の差が一定であることを把握し，それを繰り返し足す・引く考え方ができる。
"	買い物でのお金の残高や，カウントダウンなど，減少していく場面との関連付けで学習すると理解が深まり，実生活とのつながりを感じやすい。
52	小3	A 数と計算	計算の意味・方法	規則性	sec_s1_g3_1200	・変化のきまりを理解し，式や表に整理できる。	u400	・はじめの数と差から，いくつ目の数を求める簡単な式（○+(n−1)×差 等）を導入部として扱う。	中学以降の一次関数に向けた前段階として，「1増えるごとに一定を足す」という考え方をしっかり身に付ける。
53	小3	A 数と計算	計算の意味・方法	そろばん	sec_s1_g3_1000	そろばんによる計算	u100	・そろばんの基本操作を身につけ，簡単な加減乗除ができる。	・アナログの数操作を通じて数概念を可視化し，計算の理解を深める。
54	小3	A 数と計算	式・関係	割り算	sec_s1_g3_300	わり算の場面の式表現・式読み	u600	・「12個のあめを3人に等しく分ける」→12÷3といった式を立てられる。	・わり算を使う文脈を増やして，式の読み書きをスムーズに行う。
55	小3	A 数と計算	式・関係	式の表現	sec_s1_g3_1100	図及び式による表現・関連付け	u100	・テープ図などを描いて数量を整理し，式に落とし込むスキルを身に付ける。	・段階的に思考を可視化し，文章題への対応力を養う。
56	小3	A 数と計算	式・関係	式の表現	sec_s1_g3_1100	□を用いた式	u200	・□+4=10のように，□を未知数として扱い答えを導ける。	・中学での文字式に進む前段階として重要な経験。
57	小3	A 数と計算	日常生活への活用	大きな数の概念と活用	sec_s1_g3_100	大きな数の活用	u500	・買い物や距離の測定等で，大きな数を用いた計算を体験できる。	・実際の金額やメジャーを用いるなど，リアルな課題設定で応用力を促す。
58	小3	A 数と計算	日常生活への活用	分数と少数	sec_s1_g3_700	小数の活用	u600	・買い物や距離の測定等で，小数を用いた計算を体験できる。	・実際の金額やメジャーを用いるなど，リアルな課題設定で応用力を促す。
59	小3	A 数と計算	日常生活への活用	分数と少数	sec_s1_g3_700	分数の活用	u700	・買い物や距離の測定等で，分数を用いた計算を体験できる。	・実際の金額やメジャーを用いるなど，リアルな課題設定で応用力を促す。
60	小3	A 数と計算	日常生活への活用	割り算	sec_s1_g3_300	わり算の活用	u700	・人数で分ける，単価を求めるなど，わり算が不可欠な場面を処理できる。	・クラブ費やお菓子の分け方等，日常事例を取り入れ理解を深める。
61	小3	A 数と計算	計算の意味・方法	割り算	sec_s1_g3_300	2桁のわり算	u500	あまりあり、あまりなしの2桁以上の除法(2桁÷1桁、2桁÷2桁など)を正しく計算できる。繰り上がりや0の処理を含めた筆算・暗算に対応し、答えを正確に求められる	筆算や暗算を組み合わせ、大きな数の除法へ移行する基礎を育成する
133	小3	B 図形	図形の概念について理解し，その性質について考察すること	面積の基礎		形が違う図形でもマス目を使って比較できる		長方形以外の形（L字形や多角形など）でもマスの数を工夫して数え，広さを比べられる	どんな形でも「1cm角をいくつ集めたか」で面積を考えられることを体感し，後の面積公式への導入をスムーズにする
136	小3	B 図形	図形の概念について理解し，その性質について考察すること	三角形の形		二等辺三角形，正三角形の概念		・2つの辺が等しい三角形，3つの辺が等しい三角形の特徴をとらえられる。	・作図や折り曲げなどを通じて，辺や角の性質に注目する。
137	小3	B 図形	図形の概念について理解し，その性質について考察すること	円と球		円の性質理解		・円の中心，半径，直径の関係を理解できる。	・コンパスや丸い物の型取りなどで，中心から等距離にある点の集合として円を体感。
138	小3	B 図形	図形の構成の仕方について考察すること	三角形の形		二等辺三角形，正三角形の作図・構成		・二等辺三角形や正三角形を紙で作り，辺の長さを測って確かめることができる。	・作業を通じて辺の等しさを体感し，作図技術につなげる。
139	小3	B 図形	図形の構成の仕方について考察すること	円と球		円の作図・構成		・コンパスを使って円を描き，複数の円を組み合わせるなどの活動ができる。	・コンパスの正しい扱い方を指導し，円の構成要素（半径・直径）を意識させる。
140	小3	B 図形	図形の性質を日常生活に生かすこと	三角形の形		二等辺三角形，正三角形の発見		・身の回りの中で，等しい辺や三角形の性質を利用している事例に気づく。	・建築物や標識などに，二等辺三角形・正三角形がよく使われる背景を考察する。
141	小3	B 図形	図形の性質を日常生活に生かすこと	円と球		円，球		・円や球形の特性（転がりやすさ，中心からの距離が一定 など）を日常で活かす場面を探せる。	・ボールや円形テーブルなど，球や円を見つけて性質を言葉にする。
142	小3	B 図形	"図形の構成の仕方について考察すること / 図形の性質	"	平面図		2本の直線のすい直や平行を理解し，図に正しく表現できる		直角（90°）の概念を把握し，定規や三角定規などを用いてすい直・平行な線を描く基本操作ができる	生活の中の平行（線路やノートの罫線など）や直角（机の角，交差点など）を例に挙げると理解が深まる
143	小3	B 図形	"図形の構成の仕方について考察すること / 図形の性質	"	平面図		三角形・四角形を切り貼りしたり，角度を量ったりして，内角の和の理解を深める		三角形・四角形を切り貼りしたり，角度を量ったりして，内角の和の理解を深める	紙を折りたたんだり，三角形や四角形を並べたりして実感的に学ぶと，数学的な根拠を理解しやすい
144	小3	B 図形	"図形の構成の仕方について考察すること / 図形の性質	"	平面図		辺や角を実際に測定して，辺の長さの等しさ・平行関係・直角の有無などを確かめることができる		辺や角を実際に測定して，辺の長さの等しさ・平行関係・直角の有無などを確かめることができる	身近な建物や標識，デザインに使われている三角形・四角形を探す活動を通して，図形の性質への興味を高める
145	小3	B 図形	"図形の構成の仕方について考察すること / 図形の性質	"	平面図		コンパスで円を描き，円周や半径の概念を確認できる。必要に応じて3.14を用いた近似値の扱いに慣れる		コンパスで円を描き，円周や半径の概念を確認できる。必要に応じて3.14を用いた近似値の扱いに慣れる	コンパスの正しい使い方を身につけながら，日常での円（皿，車輪，コインなど）との結びつきを意識する
146	小3	B 図形	"図形の構成の仕方について考察すること / 図形の性質	"	立体図		辺・面・頂点などの基本用語を押さえ，立体を構成する要素（稜線や角の数など）を数えられる		直方体や立方体などの基本的な立体において，辺の数・面の数・頂点の数を正しく把握できる	お菓子の箱やティッシュ箱など，身近な物を使って「どこが面で，どこが辺か」を調べる体験を通じ，立体の捉え方に慣れる
147	小3	B 図形	"図形の構成の仕方について考察すること / 図形の性質	"	立体図		立体の面同士・辺同士が平行またはすい直かを識別できる		直方体・立方体においては，ある1つの面や辺に対し，どれが平行/すい直かを実際に示したり説明したりできる	図で見るだけでなく，実際の箱を回転させたりしながら確かめると理解しやすく，空間把握能力が養われる
148	小3	B 図形	"図形の構成の仕方について考察すること / 図形の性質	"	立体図		立体の体積や容積の基本的な単位（cm³やm³など）とその概念をとらえ，大小を比較する		「1cm³は1辺が1cmの立方体の大きさ」という基礎概念を理解し，いくつ分かを数えたり，簡単な積み上げ方で体積を求めたりできる	計量カップや箱などを使い，水や物を詰める実験的な活動を交えて学ぶと，体積のイメージが具体化する
149	小3	B 図形	"図形の構成の仕方について考察すること / 図形の性質	"	立体図		直方体と立方体の違いや共通点を言葉で整理し，自分なりに図を用いて説明できる		 面の形や辺の長さなどを比較し，「どの面も正方形」「2つの面は長方形」といった特徴を正しく言語化できる	箱やブロックなどを組み合わせる工作を通じて直方体と立方体の性質を再確認し，空間認識力を高める
150	小3	B 図形	"図形の構成の仕方について考察すること / 図形の性質	"	立方体の展開図		立方体の展開図とは何かを知り，6つの面をつなげた「開いた形」を理解する		立方体を展開した形（ネット図）を見て，どの面がどこと隣り合うかを判断し，正しいパターンか見分けられる	いらない箱を切り開く体験や，紙で立方体の模型を組み立てる活動を通して，平面と立体の対応関係を体感的に学ぶ
151	小3	B 図形	"図形の構成の仕方について考察すること / 図形の性質	"	立方体の展開図		 展開図を組み立てたときに，どの面が上・下・左右になるかイメージしながら図を描ける		立方体の6面を見取図で示し，折り畳んだ場合の位置関係（例えば「正面の面と上面・右面はどこ？」など）を想定できる	多様なパターン（「T字型」「十字型」「L字型」など）を提示し，どれが実際に立方体になるか試行錯誤する活動を取り入れて空間把握力を育む
152	小3	B 図形	図形の構成の仕方について考察すること / 図形の性質	立方体の展開図		立方体の各面に数字や記号を書き込み，展開図と見取り図を対応させて考察できる		ある面に書いてある数字や矢印などから，展開図でどこの面とつながるか，自力で推理し組み立てを想像できる	展開図上のマークや位置関係を利用しながら，脳内で立体を再構築する訓練をすることで，中学以降の立体図形問題（投影図など）への導入となる
153	小3	B 図形	図形の構成の仕方について考察すること / 図形の性質	立方体の展開図		立方体の展開図から，辺の長さやすい直関係を説明し，立体の性質を言葉でまとめる		展開図において，どの辺が一致するかや面同士の平行・すい直を把握し，折りたたんだ立方体の空間構成を言語化できる	面と面のつながり方を俯瞰することで，立方体のみならず他の多面体に対しても展開図を想像できるベースとなる
199	小3	C 測定	量の概念を理解し，その大きさの比べ方を見いだすこと	ものを計ろう		重さの比較		・天秤やはかりで重さを比較し，軽い/重いを定量的に把握できる。	・比重を実感するために各種のものを量り，順序を付ける活動を行う。
200	小3	C 測定	目的に応じた単位で量の大きさを的確に表現・比較すること	ものを計ろう		長さ，重さの単位(km 及び g，kg)		・km，g，kg の単位を理解し，適宜使い分けられる。	・歩幅を使って距離を推定したり，重さをはかって表記するなど体験的活動を重視。
201	小3	C 測定	目的に応じた単位で量の大きさを的確に表現・比較すること	ものを計ろう		測定の意味の理解		・より大きな長さ(1km)や重さ(1kg)を日常の実感と結びつけられる。	・「学校から家までの距離は何kmくらい？」など生活と関連づける。
202	小3	C 測定	目的に応じた単位で量の大きさを的確に表現・比較すること	ものを計ろう		適切な単位や計器の選択とその表現		・km単位なら徒歩や車の移動距離，kg単位なら人や物の質量測定などを使い分けられる。	・はかりや計測器を正しく扱い，記録して比較する力を育てる。
203	小3	C 測定	目的に応じた単位で量の大きさを的確に表現・比較すること	時刻と時間		時間の単位(秒)		・1分=60秒の概念を実感し，短時間計測(秒単位)ができる。	・徒競走やストップウォッチ測定などで秒の感覚を身に付ける。
204	小3	C 測定	目的に応じた単位で量の大きさを的確に表現・比較すること	時刻と時間		時刻と時間		・○時○分○秒という読み方と，経過時間(例えば何分何秒経過)を区別できる。	・実験等で時間を測り，グラフや表にまとめる習慣を付ける。
205	小3	C 測定	単位の関係を統合的に考察すること	ものを計ろう		長さ，重さ，かさの単位間の関係の統合的考察		・m/cm/mm，kg/g，L/mLなど，それぞれの換算関係を改めて整理できる。	・整理表や一覧表を作り，どれくらいの倍数になるのかを把握する。
206	小3	C 測定	量とその測定の方法を日常生活に生かすこと	ものを計ろう		目的に応じた適切な量の単位や測定の方法を用い数表現		・道のりをkmで表す，野菜の重さをgで表すなど，生活上の測定に応用できる。	・計画を立てる上で「何をどう測るか」を自分で選択し，結果をまとめる力を育む。
207	小3	C 測定	量とその測定の方法を日常生活に生かすこと	時刻と時間		日常生活での時刻と時間		・「今は○時○分で，行事まであと何分あるか」を判断し行動できる。	・学校生活の場面を活かし，時間管理を自然に身につけさせる。
235	小3	D データの活用	目的に応じてデータを収集，分類整理し，結果を適切に表現すること	記録を整理して表や棒グラフで整理しよう		日時の観点や場所の観点などからデータを分類整理		・例えば「何月に採れた」「どの場所で採れた」など複数の観点を使って整理できる。	・分類基準を増やすとデータの特徴がより明確化することを体感させる。
236	小3	D データの活用	目的に応じてデータを収集，分類整理し，結果を適切に表現すること	記録を整理して表や棒グラフで整理しよう		表		・縦軸・横軸に異なる観点を取り，2次元表でデータをまとめられる。	・「好きなスポーツ」「性別」など2つの軸で整理し，比較をしやすくする。
237	小3	D データの活用	目的に応じてデータを収集，分類整理し，結果を適切に表現すること	記録を整理して表や棒グラフで整理しよう		棒グラフ		・軸を決め，棒の高さで数量を表す簡単な棒グラフを描ける。	・縦方向・横方向どちらの形式でも良いが，軸の目盛りに注意して作成する。
238	小3	D データの活用	目的に応じてデータを収集，分類整理し，結果を適切に表現すること	記録を整理して表や棒グラフで整理しよう		見いだしたことを表現する		・「○○のときがいちばん多い」「△△は少ない」など，グラフから読み取った特徴を文章にできる。	・データの読み取りだけでなく，自分の言葉でまとめるアウトプットを大切にする。
239	小3	D データの活用	統計データの特徴を読み取り判断すること	記録を整理して表や棒グラフで整理しよう		身の回りの事象についての考察		・グラフや表の結果から，簡単な原因や傾向を推測・検討できる。	・「夏にアイスがよく売れるのは暑いからだろう」といった素朴な考察でOK。
240	小3	D データの活用	統計データの特徴を読み取り判断すること	記録を整理して表や棒グラフで整理しよう		表，棒グラフ		・自他が作成した棒グラフや表を見て，何が言えるかを互いに伝え合える。	・多様な見方・考え方を認め合う活動を設定し，コミュニケーション力を育む。








-- 参考JSON
```json
{
  "sections": [
    {
      "order": 100,
      "json_id": "sec_s1_g3_100",
      "subject_id": "sub_001",
      "level_id": "lev_003",
      "grade_id": "gra_003",
      "learning_category": "A",
      "status": "PUBLISHED",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "大きな数の概念と活用",
        "en": "Concepts and Applications of Large Numbers"
      },
      "description": {
        "ja": "",
        "en": ""
      },
      "units": [
        {
          "order": 100,
          "unit_id": "unit_s1_g3_sec100_100",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "万の単位、1億などの比べ方や表し方",
            "en": "How to compare and represent units such as ten-thousands and one hundred million"
          },
          "required_competency": {
            "ja": "1万，10万，100万，1億などの上位の位を正しく読み書きできる",
            "en": "Be able to correctly read and write higher place values such as 10,000; 100,000; 1,000,000; and 100,000,000"
          },
          "background": {
            "ja": "さらに大きな数への理解を深め，数のスケールを体感できるようにする",
            "en": "Deepen understanding of even larger numbers and allow students to experience their scale"
          }
        },
        {
          "order": 200,
          "unit_id": "unit_s1_g3_sec100_200",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "大きな数の相対的大きさ",
            "en": "Relative magnitude of large numbers"
          },
          "required_competency": {
            "ja": "1万と1億の差を人口やお金の桁などの具体例で説明できる",
            "en": "Be able to explain the difference between 10,000 and 100,000,000 using concrete examples such as population or monetary figures"
          },
          "background": {
            "ja": "単なる記号操作ではなく，日常との結び付きで「いくつ分違うか」を考えさせる",
            "en": "Go beyond mere symbolic manipulation by linking to everyday life and considering how many times larger one figure is than another"
          }
        },
        {
          "order": 300,
          "unit_id": "unit_s1_g3_sec100_300",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "3位数や4位数の加法及び減法",
            "en": "Addition and subtraction of three- and four-digit numbers"
          },
          "required_competency": {
            "ja": "3～4桁どうしの足し算・引き算を繰り上がり・繰り下がり含め正確に計算できる",
            "en": "Be able to accurately perform addition and subtraction of three- to four-digit numbers, including carrying and borrowing"
          },
          "background": {
            "ja": "筆算の手順をしっかり確立させる",
            "en": "Firmly establish the steps of paper-and-pencil calculation"
          }
        },
        {
          "order": 400,
          "unit_id": "unit_s1_g3_sec100_400",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "2位数や3位数の乗法",
            "en": "Multiplication of two- and three-digit numbers"
          },
          "required_competency": {
            "ja": "2桁×2桁や3桁×1桁など，大きめのかけ算を正確に行える",
            "en": "Be able to accurately perform larger multiplications such as two-digit by two-digit or three-digit by one-digit"
          },
          "background": {
            "ja": "計算量が増えるため，桁のそろえ方や筆算手順を重点的に練習する",
            "en": "Because the amount of calculation increases, focus on digit alignment and practicing the paper-and-pencil multiplication procedure"
          }
        },
        {
          "order": 500,
          "unit_id": "unit_s1_g3_sec100_500",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "大きな数の活用",
            "en": "Utilizing large numbers"
          },
          "required_competency": {
            "ja": "買い物や距離の測定等で，大きな数を用いた計算を体験できる",
            "en": "Experience calculations using large numbers in contexts such as shopping or measuring distances"
          },
          "background": {
            "ja": "実際の金額やメジャーを用いるなど，リアルな課題設定で応用力を促す",
            "en": "Promote application skills with realistic tasks, such as using actual prices or measuring tapes"
          }
        }
      ]
    },
    {
      "order": 200,
      "json_id": "sec_s1_g3_200",
      "subject_id": "sub_001",
      "level_id": "lev_003",
      "grade_id": "gra_003",
      "learning_category": "C",
      "status": "PUBLISHED",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "時刻と時間",
        "en": "Clock Times and Durations"
      },
      "units": [
        {
          "order": 100,
          "unit_id": "unit_s1_g3_sec200_100",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "時間の単位(秒)",
            "en": "Units of Time (Seconds)"
          },
          "required_competency": {
            "ja": "1分=60秒の概念を実感し，短時間計測(秒単位)ができる",
            "en": "Understand the concept of 1 minute = 60 seconds and be able to measure short durations in seconds"
          },
          "background": {
            "ja": "徒競走やストップウォッチ測定などで秒の感覚を身に付ける",
            "en": "Develop a sense of seconds through footraces or stopwatch measurements"
          }
        },
        {
          "order": 200,
          "unit_id": "unit_s1_g3_sec200_200",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "時刻と時間",
            "en": "Clock Times and Elapsed Time"
          },
          "required_competency": {
            "ja": "○時○分○秒という読み方と，経過時間(例えば何分何秒経過)を区別できる",
            "en": "Be able to read clock times (e.g., X hours, X minutes, X seconds) and distinguish elapsed time (e.g., how many minutes and seconds have passed)"
          },
          "background": {
            "ja": "実験等で時間を測り，グラフや表にまとめる習慣を付ける",
            "en": "Develop the habit of measuring time during experiments and summarizing it in graphs or tables"
          }
        },
        {
          "order": 300,
          "unit_id": "unit_s1_g3_sec200_300",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "日常生活での時刻と時間",
            "en": "Daily Clock Times and Durations"
          },
          "required_competency": {
            "ja": "今は○時○分で，行事まであと何分あるか」を判断し行動できる",
            "en": "Be able to identify the current time (X hours, X minutes) and figure out how many minutes remain until an event, then act accordingly"
          },
          "background": {
            "ja": "学校生活の場面を活かし，時間管理を自然に身につけさせる",
            "en": "Make use of everyday school situations to help students naturally acquire time management skills"
          }
        }
      ]
    },
    {
      "order": 300,
      "json_id": "sec_s1_g3_300",
      "subject_id": "sub_001",
      "level_id": "lev_003",
      "grade_id": "gra_003",
      "learning_category": "A",
      "status": "PUBLISHED",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "割り算",
        "en": "Division"
      },
      "units": [
        {
          "order": 100,
          "unit_id": "unit_s1_g3_sec300_100",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "1位数などのわり算",
            "en": "Division with Single-Digit Divisors"
          },
          "required_competency": {
            "ja": "1桁わり算(27÷3等)を九九を用いてスムーズに解ける",
            "en": "Be able to smoothly solve single-digit division (e.g., 27 ÷ 3) by using multiplication tables"
          },
          "background": {
            "ja": "2桁以上の除法に進む基礎固め。逆算としてかけ算との結び付きも確認",
            "en": "Lay the foundation for moving on to division of two-digit (and larger) numbers, confirming its relationship with multiplication as inverse operations"
          }
        },
        {
          "order": 200,
          "unit_id": "unit_s1_g3_sec300_200",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "わり算とかけ算や引き算との関係",
            "en": "Relationship of Division to Multiplication and Subtraction"
          },
          "required_competency": {
            "ja": "わり算がかけ算や引き算と逆演算の関係にあることを理解し，検算に利用できる",
            "en": "Understand that division is an inverse operation relative to multiplication (and is related to subtraction), and be able to use this concept for verification"
          },
          "background": {
            "ja": "問題解決で「かけ算で確かめる」などの活用を経験させる",
            "en": "Have students experience solving problems by verifying with multiplication, etc."
          }
        },
        {
          "order": 300,
          "unit_id": "unit_s1_g3_sec300_300",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "あまりのあるわり算",
            "en": "Division with Remainders"
          },
          "required_competency": {
            "ja": "あまりのあるわり算を理解し、商とあまりを正しく表せる",
            "en": "Understand division with remainders and accurately express the quotient and remainder"
          },
          "background": {
            "ja": "余りのある除法(13÷4=3あまり1等)を正しく行い、余りが除数未満であることを認識できる",
            "en": "Be able to perform division with remainders correctly (e.g., 13 ÷ 4 = 3 remainder 1) and recognize that the remainder is less than the divisor"
          }
        },
        {
          "order": 400,
          "unit_id": "unit_s1_g3_sec300_400",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "わり算の意味",
            "en": "Meaning of Division"
          },
          "required_competency": {
            "ja": "わり算には「等分除」と「包含除」があることを理解し，簡単な問題を式にできる",
            "en": "Understand that there are two types of division—partitive (等分除) and quotative (包含除)—and be able to represent simple problems using equations"
          },
          "background": {
            "ja": "同じ数ずつ分けるか，何回分になるかを区別し，ミスを防ぐ",
            "en": "Distinguish whether you are dividing into equal shares or determining how many times one quantity fits into another, in order to avoid mistakes"
          }
        },
        {
          "order": 500,
          "unit_id": "unit_s1_g3_sec300_500",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "2桁のわり算",
            "en": "Two-Digit Division"
          },
          "required_competency": {
            "ja": "あまりあり、あまりなしの2桁以上の除法(2桁÷1桁、2桁÷2桁など)を正しく計算できる。繰り上がりや0の処理を含めた筆算・暗算に対応し、答えを正確に求められる",
            "en": "Be able to correctly perform division of two-digit or more (e.g., two-digit ÷ one-digit, two-digit ÷ two-digit) with or without remainders. This includes handling carrying and zeros in both written (pencil-and-paper) and mental calculations to arrive at accurate answers."
          },
          "background": {
            "ja": "筆算や暗算を組み合わせ、大きな数の除法へ移行する基礎を育成する",
            "en": "Lay the foundation for transitioning to larger-number division by combining written and mental calculation methods."
          }
        },
        {
          "order": 600,
          "unit_id": "unit_s1_g3_sec300_600",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "わり算の場面の式表現・式読み",
            "en": "Equation Representation and Reading in Division Situations"
          },
          "required_competency": {
            "ja": "「12個のあめを3人に等しく分ける」→12÷3といった式を立てられる",
            "en": "Be able to formulate an equation such as 12 ÷ 3 from a situation like “Divide 12 candies equally among three people.”"
          },
          "background": {
            "ja": "わり算を使う文脈を増やして，式の読み書きをスムーズに行う",
            "en": "Increase contexts in which division is used, so that students can smoothly read and write division equations"
          }
        },
        {
          "order": 700,
          "unit_id": "unit_s1_g3_sec300_700",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "わり算の活用",
            "en": "Applications of Division"
          },
          "required_competency": {
            "ja": "人数で分ける，単価を求めるなど，わり算が不可欠な場面を処理できる",
            "en": "Be able to handle situations where division is essential, such as splitting items by the number of people or finding the unit price."
          },
          "background": {
            "ja": "クラブ費やお菓子の分け方等，日常事例を取り入れ理解を深める",
            "en": "Deepen understanding by including everyday examples, such as dividing club fees or snacks."
          }
        }
      ]
    },
    {
      "order": 400,
      "json_id": "sec_s1_g3_400",
      "subject_id": "sub_001",
      "level_id": "lev_003",
      "grade_id": "gra_003",
      "learning_category": "C",
      "status": "PUBLISHED",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "ものを計ろう",
        "en": "Measuring Objects"
      },
      "units": [
        {
          "order": 100,
          "unit_id": "unit_s1_g3_sec400_100",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "重さの比較",
            "en": "Comparison of Weights"
          },
          "required_competency": {
            "ja": "天秤やはかりで重さを比較し，軽い/重いを定量的に把握できる",
            "en": "Be able to compare weights using a balance scale or other measuring devices, and quantitatively identify what is lighter or heavier"
          },
          "background": {
            "ja": "比重を実感するために各種のものを量り，順序を付ける活動を行う",
            "en": "Have students measure various items to experience relative density and arrange them in order"
          }
        },
        {
          "order": 200,
          "unit_id": "unit_s1_g3_sec400_200",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "長さ，重さの単位(km 及び g，kg)",
            "en": "Units of Length and Weight (km, g, kg)"
          },
          "required_competency": {
            "ja": "km，g，kg の単位を理解し，適宜使い分けられる",
            "en": "Understand the units km, g, and kg, and use them appropriately"
          },
          "background": {
            "ja": "歩幅を使って距離を推定したり，重さをはかって表記するなど体験的活動を重視",
            "en": "Emphasize hands-on activities such as estimating distances by stride or measuring and labeling weight"
          }
        },
        {
          "order": 300,
          "unit_id": "unit_s1_g3_sec400_300",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "測定の意味の理解",
            "en": "Understanding the Meaning of Measurement"
          },
          "required_competency": {
            "ja": "より大きな長さ(1km)や重さ(1kg)を日常の実感と結びつけられる",
            "en": "Be able to connect larger units such as 1 km or 1 kg with everyday experiences"
          },
          "background": {
            "ja": "「学校から家までの距離は何kmくらい？」など生活と関連づける",
            "en": "Relate to daily life by asking questions like “How many kilometers is it from school to home?”"
          }
        },
        {
          "order": 400,
          "unit_id": "unit_s1_g3_sec400_400",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "適切な単位や計器の選択とその表現",
            "en": "Choosing Appropriate Units and Instruments, and Their Representation"
          },
          "required_competency": {
            "ja": "km単位なら徒歩や車の移動距離，kg単位なら人や物の質量測定などを使い分けられる",
            "en": "Be able to appropriately use kilometers for walking or driving distances, and kilograms for measuring the mass of people or objects"
          },
          "background": {
            "ja": "はかりや計測器を正しく扱い，記録して比較する力を育てる",
            "en": "Develop the ability to correctly use scales or measuring instruments, record the results, and make comparisons"
          }
        },
        {
          "order": 500,
          "unit_id": "unit_s1_g3_sec400_500",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "長さ，重さ，かさの単位間の関係の統合的考察",
            "en": "Comprehensive Consideration of Relationships Among Units of Length, Weight, and Volume"
          },
          "required_competency": {
            "ja": "m/cm/mm，kg/g，L/mLなど，それぞれの換算関係を改めて整理できる",
            "en": "Be able to systematically organize and understand the conversion relationships between units such as m/cm/mm, kg/g, and L/mL"
          },
          "background": {
            "ja": "整理表や一覧表を作り，どれくらいの倍数になるのかを把握する",
            "en": "Encourage creating tables or lists to understand how many times one unit is of another"
          }
        },
        {
          "order": 600,
          "unit_id": "unit_s1_g3_sec400_600",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "目的に応じた適切な量の単位や測定の方法を用い数表現",
            "en": "Use appropriate units of measurement and methods according to the purpose, and represent the results numerically"
          },
          "required_competency": {
            "ja": "道のりをkmで表す，野菜の重さをgで表すなど，生活上の測定に応用できる",
            "en": "Be able to apply measurement in daily life, such as expressing distances in kilometers or the weight of vegetables in grams"
          },
          "background": {
            "ja": "計画を立てる上で「何をどう測るか」を自分で選択し，結果をまとめる力を育む",
            "en": "Develop the skill to decide what and how to measure when making a plan, and to summarize the results"
          }
        }
      ]
    },
    {
      "order": 500,
      "json_id": "sec_s1_g3_500",
      "subject_id": "sub_001",
      "level_id": "lev_003",
      "grade_id": "gra_003",
      "learning_category": "D",
      "status": "PUBLISHED",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "記録を整理して表や棒グラフで整理しよう",
        "en": "Organize Records Using Tables and Bar Graphs"
      },
      "units": [
        {
          "order": 100,
          "unit_id": "unit_s1_g3_sec500_100",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "日時の観点や場所の観点などからデータを分類整理",
            "en": "Classify and organize data based on factors such as date/time or location"
          },
          "required_competency": {
            "ja": "例えば「何月に採れた」「どの場所で採れた」など複数の観点を使って整理できる",
            "en": "For instance, able to organize data by multiple criteria, such as 'Which month it was harvested?' or 'Where it was harvested?'"
          },
          "background": {
            "ja": "分類基準を増やすとデータの特徴がより明確化することを体感させる",
            "en": "Help students experience how increasing classification criteria clarifies the characteristics of the data"
          }
        },
        {
          "order": 200,
          "unit_id": "unit_s1_g3_sec500_200",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "表",
            "en": "Tables"
          },
          "required_competency": {
            "ja": "縦軸・横軸に異なる観点を取り，2次元表でデータをまとめられる",
            "en": "Can summarize data in a two-dimensional table by using different perspectives for the vertical and horizontal axes"
          },
          "background": {
            "ja": "「好きなスポーツ」「性別」など2つの軸で整理し，比較をしやすくする",
            "en": "Facilitate comparisons by organizing data around two axes, such as 'favorite sport' and 'gender'"
          }
        },
        {
          "order": 300,
          "unit_id": "unit_s1_g3_sec500_300",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "棒グラフ",
            "en": "Bar Graph"
          },
          "required_competency": {
            "ja": "軸を決め，棒の高さで数量を表す簡単な棒グラフを描ける",
            "en": "Can draw a simple bar graph by establishing an axis and representing quantities with bar heights"
          },
          "background": {
            "ja": "縦方向・横方向どちらの形式でも良いが，軸の目盛りに注意して作成する",
            "en": "Either vertical or horizontal format is fine; pay attention to the scale on the axis when creating the graph"
          }
        },
        {
          "order": 400,
          "unit_id": "unit_s1_g3_sec500_400",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "見いだしたことを表現する",
            "en": "Expressing Findings"
          },
          "required_competency": {
            "ja": "「○○のときがいちばん多い」「△△は少ない」など，グラフから読み取った特徴を文章にできる",
            "en": "Able to articulate features observed in a graph (e.g., '○○ is the most common,' '△△ is fewer')"
          },
          "background": {
            "ja": "データの読み取りだけでなく，自分の言葉でまとめるアウトプットを大切にする",
            "en": "Emphasize not only interpreting data but also expressing the findings in one's own words"
          }
        },
        {
          "order": 500,
          "unit_id": "unit_s1_g3_sec500_500",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "身の回りの事象についての考察",
            "en": "Considering Everyday Phenomena"
          },
          "required_competency": {
            "ja": "グラフや表の結果から，簡単な原因や傾向を推測・検討できる",
            "en": "Able to infer and discuss simple causes or trends based on the results shown in graphs or tables"
          },
          "background": {
            "ja": "「夏にアイスがよく売れるのは暑いからだろう」といった素朴な考察でOK",
            "en": "Simple reasoning such as 'Ice cream probably sells well in summer because it's hot' is acceptable"
          }
        },
        {
          "order": 600,
          "unit_id": "unit_s1_g3_sec500_600",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "表，棒グラフ",
            "en": "Tables, Bar Graphs"
          },
          "required_competency": {
            "ja": "自他が作成した棒グラフや表を見て，何が言えるかを互いに伝え合える",
            "en": "Able to examine bar graphs and tables created by oneself or others and communicate what can be inferred"
          },
          "background": {
            "ja": "多様な見方・考え方を認め合う活動を設定し，コミュニケーション力を育む",
            "en": "Encourage activities that acknowledge diverse perspectives and foster communication skills"
          }
        }
      ]
    },
    {
      "order": 600,
      "json_id": "sec_s1_g3_600",
      "subject_id": "sub_001",
      "level_id": "lev_003",
      "grade_id": "gra_003",
      "learning_category": "D",
      "status": "PUBLISHED",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "円と球",
        "en": "Circles and Spheres"
      },
      "units": [
        {
          "order": 100,
          "unit_id": "unit_s1_g3_sec600_100",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "円の性質理解",
            "en": "Understanding the Properties of Circles"
          },
          "required_competency": {
            "ja": "円の中心，半径，直径の関係を理解できる",
            "en": "Able to understand the relationship among the center, radius, and diameter of a circle"
          },
          "background": {
            "ja": "コンパスや丸い物の型取りなどで，中心から等距離にある点の集合として円を体感",
            "en": "Use a compass or trace round objects to experience a circle as the set of points equidistant from its center"
          }
        },
        {
          "order": 200,
          "unit_id": "unit_s1_g3_sec600_200",
          "status": "HIDDEN",
          "version": "1.0.0",
          "requirement": {
            "ja": "円の作図・構成",
            "en": "Constructing and Composing Circles"
          },
          "required_competency": {
            "ja": "コンパスを使って円を描き，複数の円を組み合わせるなどの活動ができる",
            "en": "Able to draw circles with a compass and engage in activities combining multiple circles"
          },
          "background": {
            "ja": "コンパスの正しい扱い方を指導し，円の構成要素（半径・直径）を意識させる",
            "en": "Teach proper compass usage and emphasize the elements of a circle (radius, diameter)"
          }
        },
        {
          "order": 300,
          "unit_id": "unit_s1_g3_sec600_300",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "円，球",
            "en": "Circles and Spheres"
          },
          "required_competency": {
            "ja": "円や球形の特性（転がりやすさ，中心からの距離が一定 など）を日常で活かす場面を探せる",
            "en": "Able to identify daily-life situations where circle or sphere properties (such as ease of rolling or constant distance from the center) can be utilized"
          },
          "background": {
            "ja": "ボールや円形テーブルなど，球や円を見つけて性質を言葉にする",
            "en": "Observe spherical or circular objects (like balls or round tables) and describe their properties in words"
          }
        }
      ]
    },
    {
      "order": 700,
      "json_id": "sec_s1_g3_700",
      "subject_id": "sub_001",
      "level_id": "lev_003",
      "grade_id": "gra_003",
      "learning_category": "A",
      "status": "PUBLISHED",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "分数と小数",
        "en": "Fractions and Decimals"
      },
      "units": [
        {
          "order": 100,
          "unit_id": "unit_s1_g3_sec700_100",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "小数",
            "en": "Decimals"
          },
          "required_competency": {
            "ja": "・少数の意味を理解できる",
            "en": "Be able to understand the meaning of decimals"
          },
          "background": {
            "ja": "・少数、整数の分類や比較などの問題を通して意味や概念を理解する",
            "en": "Understand the meaning and concepts through problems involving the classification and comparison of decimals and integers"
          }
        },
        {
          "order": 200,
          "unit_id": "unit_s1_g3_sec700_200",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "簡単な分数",
            "en": "Simple fractions"
          },
          "required_competency": {
            "ja": "・分数の意味を理解できる",
            "en": "Be able to understand the meaning of fractions"
          },
          "background": {
            "ja": "・整数、分数の分類や比較などの問題を通して意味や概念を理解する",
            "en": "Understand the meaning and concepts through problems involving the classification and comparison of integers and fractions"
          }
        },
        {
          "order": 300,
          "unit_id": "unit_s1_g3_sec700_300",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "小数（10の位）や簡単な分数の大きさの比較・計算",
            "en": "Comparing and calculating the size of decimals (tenths) and simple fractions"
          },
          "required_competency": {
            "ja": "0.5や0.3など小数第1位を分数(1/2, 3/10など)と対応づけられる。",
            "en": "Can relate decimals in the first decimal place (e.g., 0.5, 0.3) to fractions (1/2, 3/10, etc.)"
          },
          "background": {
            "ja": "小数と分数が互いに表し合えることを具体的に示す（長さや重さなど）",
            "en": "Demonstrate concretely how decimals and fractions can represent each other (using length, weight, etc.)"
          }
        },
        {
          "order": 400,
          "unit_id": "unit_s1_g3_sec700_400",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "小数（10の位）の加法及び減法",
            "en": "Addition and Subtraction of Decimals (Tenths)"
          },
          "required_competency": {
            "ja": "0.5 + 0.3など，小数第1位までの加減計算を筆算または暗算で行える",
            "en": "Can perform addition and subtraction of decimals up to the first decimal place (e.g., 0.5 + 0.3) by written or mental calculation"
          },
          "background": {
            "ja": "小数点の位置合わせの重要性を強調し，整数との違いを意識させる",
            "en": "Emphasize aligning the decimal points and make them aware of how it differs from integer calculations"
          }
        },
        {
          "order": 500,
          "unit_id": "unit_s1_g3_sec700_500",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "簡単な分数の加法及び減法",
            "en": "Addition and Subtraction of Simple Fractions"
          },
          "required_competency": {
            "ja": "分母が同じな場合（1/2+1/2等）の計算を誤りなく行える",
            "en": "Able to perform calculations correctly when denominators are the same (e.g., 1/2 + 1/2)"
          },
          "background": {
            "ja": "分割のイメージを図で示し，抽象化に慣れさせる初歩段階",
            "en": "Use diagrams to illustrate the concept of splitting, helping them get used to the abstraction in its early stage"
          }
        },
        {
          "order": 600,
          "unit_id": "unit_s1_g3_sec700_600",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "小数の活用",
            "en": "Application of Decimals"
          },
          "required_competency": {
            "ja": "買い物や距離の測定等で，小数を用いた計算を体験できる",
            "en": "Able to use decimal calculations in practical contexts such as shopping or measuring distances"
          },
          "background": {
            "ja": "実際の金額やメジャーを用いるなど，リアルな課題設定で応用力を促す",
            "en": "Encourage practical application by using real situations such as actual prices or measuring tapes"
          }
        },
        {
          "order": 700,
          "unit_id": "unit_s1_g3_sec700_700",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "分数の活用",
            "en": "Application of Fractions"
          },
          "required_competency": {
            "ja": "買い物や距離の測定等で，分数を用いた計算を体験できる",
            "en": "Able to use fraction calculations in everyday contexts such as shopping or measuring distances"
          },
          "background": {
            "ja": "実際の金額やメジャーを用いるなど，リアルな課題設定で応用力を促す",
            "en": "Encourage practical application by using real situations such as actual prices or measuring tapes"
          }
        }
      ]
    },
    {
      "order": 800,
      "json_id": "sec_s1_g3_800",
      "subject_id": "sub_001",
      "level_id": "lev_003",
      "grade_id": "gra_003",
      "learning_category": "B",
      "status": "PUBLISHED",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "三角形の形",
        "en": "Triangle Shapes"
      },
      "units": [
        {
          "order": 100,
          "unit_id": "unit_s1_g3_sec800_100",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "二等辺三角形，正三角形の概念",
            "en": "Concept of isosceles and equilateral triangles"
          },
          "required_competency": {
            "ja": "2つの辺が等しい三角形，3つの辺が等しい三角形の特徴をとらえられる",
            "en": "Be able to grasp the characteristics of triangles with two equal sides or three equal sides"
          },
          "background": {
            "ja": "作図や折り曲げなどを通じて，辺や角の性質に注目する",
            "en": "Focus on the properties of sides and angles through activities such as drawing and folding"
          }
        },
        {
          "order": 200,
          "unit_id": "unit_s1_g3_sec800_200",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "二等辺三角形，正三角形の作図・構成",
            "en": "Constructing and drawing isosceles and equilateral triangles"
          },
          "required_competency": {
            "ja": "二等辺三角形や正三角形を紙で作り，辺の長さを測って確かめることができる",
            "en": "Be able to create isosceles or equilateral triangles on paper and measure their sides to confirm equality"
          },
          "background": {
            "ja": "作業を通じて辺の等しさを体感し，作図技術につなげる",
            "en": "Experience equal side lengths through hands-on activities and connect this to drafting skills"
          }
        },
        {
          "order": 300,
          "unit_id": "unit_s1_g3_sec800_300",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "二等辺三角形，正三角形の発見",
            "en": "Discovering isosceles and equilateral triangles"
          },
          "required_competency": {
            "ja": "身の回りの中で，等しい辺や三角形の性質を利用している事例に気づく",
            "en": "Notice examples in the surroundings where equal sides or triangular properties are utilized"
          },
          "background": {
            "ja": "建築物や標識などに，二等辺三角形・正三角形がよく使われる背景を考察する",
            "en": "Consider why isosceles and equilateral triangles are often used in architecture and signage"
          }
        }
      ]
    },
    {
      "order": 900,
      "json_id": "sec_s1_g3_900",
      "subject_id": "sub_001",
      "level_id": "lev_003",
      "grade_id": "gra_003",
      "learning_category": "A",
      "status": "PUBLISHED",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "計算の決まりや工夫",
        "en": "Rules and Techniques of Calculation"
      },
      "units": [
        {
          "order": 100,
          "unit_id": "unit_s1_g3_sec900_100",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "交換法則、結合法則、分配法則",
            "en": "Commutative, Associative, and Distributive Properties"
          },
          "required_competency": {
            "ja": "a×(b + c)=ab+acなど，乗法・加法におけるこれらの法則を式で表せる",
            "en": "Be able to express these properties in equations for multiplication and addition, such as a × (b + c) = ab + ac"
          },
          "background": {
            "ja": "高学年での式操作に備え，体系的に理解させる",
            "en": "Prepare for algebraic manipulation in higher grades by teaching them systematically"
          }
        },
        {
          "order": 200,
          "unit_id": "unit_s1_g3_sec900_200",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "加法、減法及び乗法の結果の見積もり",
            "en": "Estimating the results of addition, subtraction, and multiplication"
          },
          "required_competency": {
            "ja": "計算前におおよその値を把握し，答えの妥当性を判断できる",
            "en": "Before doing the calculation, grasp an approximate value and judge the reasonableness of the answer"
          },
          "background": {
            "ja": "日常での買い物や分量計算など，誤差の確認に役立つ習慣化が狙い",
            "en": "The aim is to form habits that help confirm errors in daily tasks such as shopping or measuring quantities"
          }
        },
        {
          "order": 300,
          "unit_id": "unit_s1_g3_sec900_300",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "計算の工夫や確かめ",
            "en": "Calculation techniques and verification"
          },
          "required_competency": {
            "ja": "逆算や暗算，そろばんなどを状況に応じて使い分け，結果を検証できる",
            "en": "Be able to verify results by using inverse operations, mental arithmetic, or an abacus as appropriate"
          },
          "background": {
            "ja": "複数の手段で自己点検することで，計算力と自信を高める",
            "en": "Enhance computational skills and confidence by self-checking with multiple methods"
          }
        }
      ]
    },
    {
      "order": 1000,
      "json_id": "sec_s1_g3_1000",
      "subject_id": "sub_001",
      "level_id": "lev_003",
      "grade_id": "gra_003",
      "learning_category": "A",
      "status": "HIDDEN",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "そろばん",
        "en": "Abacus"
      },
      "units": [
        {
          "order": 100,
          "unit_id": "unit_s1_g3_sec1000_100",
          "status": "HIDDEN",
          "version": "1.0.0",
          "requirement": {
            "ja": "そろばんによる計算",
            "en": "Calculation using the abacus"
          },
          "required_competency": {
            "ja": "そろばんの基本操作を身につけ，簡単な加減乗除ができる",
            "en": "Master the basic operations of the abacus and perform simple addition, subtraction, multiplication, and division"
          },
          "background": {
            "ja": "アナログの数操作を通じて数概念を可視化し，計算の理解を深める",
            "en": "Visualize numerical concepts through hands-on (analog) manipulation, thus deepening understanding of calculations"
          }
        }
      ]
    },
    {
      "order": 1100,
      "json_id": "sec_s1_g3_1100",
      "subject_id": "sub_001",
      "level_id": "lev_003",
      "grade_id": "gra_003",
      "learning_category": "A",
      "status": "PUBLISHED",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "式の表現",
        "en": "Expression of Equations"
      },
      "units": [
        {
          "order": 100,
          "unit_id": "unit_s1_g3_sec1100_100",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "図及び式による表現・関連付け",
            "en": "Representation and connection using diagrams and equations"
          },
          "required_competency": {
            "ja": "テープ図などを描いて数量を整理し，式に落とし込むスキルを身に付ける",
            "en": "Develop the skill to organize quantities by drawing tape diagrams, etc., and then convert them into equations"
          },
          "background": {
            "ja": "段階的に思考を可視化し，文章題への対応力を養う",
            "en": "Gradually visualize the thought process, thereby enhancing the ability to handle word problems"
          }
        },
        {
          "order": 200,
          "unit_id": "unit_s1_g3_sec1100_200",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "□を用いた式",
            "en": "Equations using □ as a placeholder"
          },
          "required_competency": {
            "ja": "□+4=10のように，□を未知数として扱い答えを導ける",
            "en": "Be able to treat □ as an unknown (e.g., in □ + 4 = 10) and solve for it"
          },
          "background": {
            "ja": "中学での文字式に進む前段階として重要な経験",
            "en": "An important experience prior to moving on to algebraic expressions in junior high school"
          }
        }
      ]
    },
    {
      "order": 1200,
      "json_id": "sec_s1_g3_1200",
      "subject_id": "sub_001",
      "level_id": "lev_003",
      "grade_id": "gra_003",
      "learning_category": "A",
      "status": "PUBLISHED",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "規則性",
        "en": "Patterns and Regularities"
      },
      "units": [
        {
          "order": 100,
          "unit_id": "unit_s1_g3_sec1200_100",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "図形や記号，数字などが繰り返すパターンを見いだし，次にくる要素や途中の要素を推測できる",
            "en": "Identify repeating patterns in shapes, symbols, and numbers, and predict upcoming or intermediate elements"
          },
          "required_competency": {
            "ja": "簡単なくり返しパターン（例：○→△→□→○→△→□…）を見つけ，周期を言葉や式で表すことができる",
            "en": "Be able to find simple repeating patterns (e.g., ○→△→□→○→△→□…) and express their cycle in words or equations"
          },
          "background": {
            "ja": "階段を上がる段数や毎日同じ枚数のシールを貼るなど，実生活で一定ずつ増えていく事象を取り上げるとイメージしやすい",
            "en": "Using everyday examples of incremental increases, such as the number of steps climbed or the number of stickers added daily, helps foster a clear image"
          }
        },
        {
          "order": 200,
          "unit_id": "unit_s1_g3_sec1200_200",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "周期を把握して，「何番目には何がくるか」を判断できる",
            "en": "Grasp the cycle and determine “which element appears in each position”"
          },
          "required_competency": {
            "ja": "位置と要素の対応（n番目→要素）を整理する基礎力を養う",
            "en": "Develop a foundational skill in correlating positions and elements (n-th term → element)"
          },
          "background": {
            "ja": "文字式を導入しなくても，図や表を使って視覚化しながら「n番目＝…」と示すプロセスを経験させると定着しやすい",
            "en": "Even without introducing algebraic expressions, visually demonstrate the process using diagrams or tables (n-th term = ...), which aids retention"
          }
        },
        {
          "order": 300,
          "unit_id": "unit_s1_g3_sec1200_300",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "一定の差で増減する数列（等差数列）を見いだし，次にくる数や特定の番目の値を求める",
            "en": "Identify sequences with a constant difference (arithmetic progressions) and find the next term or the value of a specific term"
          },
          "required_competency": {
            "ja": "隣り合う数の差が一定であることを把握し，それを繰り返し足す・引く考え方ができる",
            "en": "Recognize that consecutive terms differ by a fixed amount, and understand how to repeatedly add or subtract that amount"
          },
          "background": {
            "ja": "買い物でのお金の残高や，カウントダウンなど，減少していく場面との関連付けで学習すると理解が深まり，実生活とのつながりを感じやすい",
            "en": "Linking to real-life contexts such as remaining money when shopping or countdowns helps deepen understanding and highlight practical relevance"
          }
        },
        {
          "order": 400,
          "unit_id": "unit_s1_g3_sec1200_400",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "変化のきまりを理解し，式や表に整理できる",
            "en": "Understand the rule of change and organize it in equations or tables"
          },
          "required_competency": {
            "ja": "はじめの数と差から，いくつ目の数を求める簡単な式（○+(n−1)×差 等）を導入部として扱う",
            "en": "Introduce simple formulas, such as the first term plus (n−1) times the difference, to find the n-th term"
          },
          "background": {
            "ja": "中学以降の一次関数に向けた前段階として，「1増えるごとに一定を足す」という考え方をしっかり身に付ける",
            "en": "This serves as a preparatory stage for linear functions in middle school; firmly establish the idea of adding a constant for each increment by 1"
          }
        }
      ]
    },
    {
      "order": 1300,
      "json_id": "sec_s1_g3_1300",
      "subject_id": "sub_001",
      "level_id": "lev_003",
      "grade_id": "gra_003",
      "learning_category": "B",
      "status": "PUBLISHED",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "平面図",
        "en": "Plane Figures"
      },
      "units": [
        {
          "order": 100,
          "unit_id": "unit_s1_g3_sec1300_100",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "2本の直線のすい直や平行を理解し，図に正しく表現できる",
            "en": "Understand perpendicular and parallel lines between two lines and represent them correctly in diagrams"
          },
          "required_competency": {
            "ja": "直角（90°）の概念を把握し，定規や三角定規などを用いてすい直・平行な線を描く基本操作ができる",
            "en": "Grasp the concept of a right angle (90°) and be able to carry out basic tasks of drawing perpendicular and parallel lines using rulers or set squares"
          },
          "background": {
            "ja": "生活の中の平行（線路やノートの罫線など）や直角（机の角，交差点など）を例に挙げると理解が深まる",
            "en": "Use everyday examples of parallel lines (railroad tracks, notebook lines) or right angles (desk corners, intersections) to deepen understanding"
          }
        },
        {
          "order": 200,
          "unit_id": "unit_s1_g3_sec1300_200",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "三角形・四角形を切り貼りしたり，角度を量ったりして，内角の和の理解を深める",
            "en": "Cut and paste triangles or quadrilaterals and measure angles to deepen understanding of the sum of interior angles"
          },
          "required_competency": {
            "ja": "三角形・四角形を切り貼りしたり，角度を量ったりして，内角の和の理解を深める",
            "en": "By cutting and pasting triangles or quadrilaterals and measuring angles, gain a deeper grasp of the sum of interior angles"
          },
          "background": {
            "ja": "紙を折りたたんだり，三角形や四角形を並べたりして実感的に学ぶと，数学的な根拠を理解しやすい",
            "en": "Folding paper or arranging triangles and quadrilaterals in a hands-on manner makes it easier to comprehend the mathematical reasoning"
          }
        },
        {
          "order": 300,
          "unit_id": "unit_s1_g3_sec1300_300",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "辺や角を実際に測定して，辺の長さの等しさ・平行関係・直角の有無などを確かめることができる",
            "en": "Measure sides and angles to verify equality of side lengths, parallel relationships, and the presence or absence of right angles"
          },
          "required_competency": {
            "ja": "辺や角を実際に測定して，辺の長さの等しさ・平行関係・直角の有無などを確かめることができる",
            "en": "Be able to measure sides and angles in order to check for equal side lengths, parallel lines, or right angles"
          },
          "background": {
            "ja": "身近な建物や標識，デザインに使われている三角形・四角形を探す活動を通して，図形の性質への興味を高める",
            "en": "Foster interest in the properties of shapes by looking for triangles and quadrilaterals in everyday buildings, signs, and designs"
          }
        },
        {
          "order": 400,
          "unit_id": "unit_s1_g3_sec1300_400",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "コンパスで円を描き，円周や半径の概念を確認できる。必要に応じて3.14を用いた近似値の扱いに慣れる",
            "en": "Draw circles with a compass and confirm the concepts of circumference and radius; practice using the approximate value 3.14 as needed"
          },
          "required_competency": {
            "ja": "コンパスで円を描き，円周や半径の概念を確認できる。必要に応じて3.14を用いた近似値の扱いに慣れる",
            "en": "Be able to draw circles with a compass, confirm the concepts of circumference and radius, and become accustomed to approximate values with 3.14"
          },
          "background": {
            "ja": "コンパスの正しい使い方を身につけながら，日常での円（皿，車輪，コインなど）との結びつきを意識する",
            "en": "Learn to use a compass correctly while recognizing everyday connections to circles (such as plates, wheels, coins, etc.)"
          }
        }
      ]
    },
    {
      "order": 1400,
      "json_id": "sec_s1_g3_1400",
      "subject_id": "sub_001",
      "level_id": "lev_003",
      "grade_id": "gra_003",
      "learning_category": "B",
      "status": "PUBLISHED",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "立体図",
        "en": "Solid Figures"
      },
      "units": [
        {
          "order": 100,
          "unit_id": "unit_s1_g3_sec1400_100",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "辺・面・頂点などの基本用語を押さえ，立体を構成する要素（稜線や角の数など）を数えられる",
            "en": "Understand basic terms such as edges, faces, and vertices, and be able to count the components (e.g. number of edges or corners) that form a solid"
          },
          "required_competency": {
            "ja": "直方体や立方体などの基本的な立体において，辺の数・面の数・頂点の数を正しく把握できる",
            "en": "Correctly identify the number of edges, faces, and vertices in basic solids such as rectangular prisms or cubes"
          },
          "background": {
            "ja": "お菓子の箱やティッシュ箱など，身近な物を使って「どこが面で，どこが辺か」を調べる体験を通じ，立体の捉え方に慣れる",
            "en": "Become familiar with visualizing solids by using everyday items like candy boxes or tissue boxes to determine which parts are faces and which are edges"
          }
        },
        {
          "order": 200,
          "unit_id": "unit_s1_g3_sec1400_200",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "立体の面同士・辺同士が平行またはすい直かを識別できる",
            "en": "Identify whether the faces or edges of a solid are parallel or perpendicular"
          },
          "required_competency": {
            "ja": "直方体・立方体においては，ある1つの面や辺に対し，どれが平行/すい直かを実際に示したり説明したりできる",
            "en": "In rectangular prisms or cubes, be able to practically show or explain which faces/edges are parallel or perpendicular relative to a specific face or edge"
          },
          "background": {
            "ja": "図で見るだけでなく，実際の箱を回転させたりしながら確かめると理解しやすく，空間把握能力が養われる",
            "en": "Not only look at diagrams, but also rotate and examine actual boxes to enhance comprehension and develop spatial awareness"
          }
        },
        {
          "order": 300,
          "unit_id": "unit_s1_g3_sec1400_300",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "立体の体積や容積の基本的な単位（cm³やm³など）とその概念をとらえ，大小を比較する",
            "en": "Understand the basic units (cm³, m³, etc.) and concepts of volume/capacity in solids, and compare their sizes"
          },
          "required_competency": {
            "ja": "「1cm³は1辺が1cmの立方体の大きさ」という基礎概念を理解し，いくつ分かを数えたり，簡単な積み上げ方で体積を求めたりできる",
            "en": "Grasp the fundamental concept that \"1cm³ is the size of a cube with 1cm edges,\" and be able to count how many of such cubes fit or use simple stacking to find volume"
          },
          "background": {
            "ja": "計量カップや箱などを使い，水や物を詰める実験的な活動を交えて学ぶと，体積のイメージが具体化する",
            "en": "Use measuring cups or boxes, and incorporate hands-on activities of filling them with water or objects to concretize the idea of volume"
          }
        },
        {
          "order": 400,
          "unit_id": "unit_s1_g3_sec1400_400",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "直方体と立方体の違いや共通点を言葉で整理し，自分なりに図を用いて説明できる",
            "en": "Explain the differences and similarities between rectangular prisms and cubes in words, and illustrate them with diagrams"
          },
          "required_competency": {
            "ja": "面の形や辺の長さなどを比較し，「どの面も正方形」「2つの面は長方形」といった特徴を正しく言語化できる",
            "en": "Compare features such as face shapes or side lengths, and accurately articulate characteristics like \"all faces are squares\" or \"two faces are rectangles\""
          },
          "background": {
            "ja": "箱やブロックなどを組み合わせる工作を通じて直方体と立方体の性質を再確認し，空間認識力を高める",
            "en": "Reinforce understanding of rectangular prisms and cubes by assembling boxes or blocks, thereby improving spatial recognition"
          }
        }
      ]
    },
    {
      "order": 1500,
      "json_id": "sec_s1_g3_1500",
      "subject_id": "sub_001",
      "level_id": "lev_003",
      "grade_id": "gra_003",
      "learning_category": "B",
      "status": "PUBLISHED",
      "version": "1.0.0",
      "subject": {
        "ja": "算数",
        "en": "Mathematics"
      },
      "name": {
        "ja": "立方体の展開図",
        "en": "Nets of a Cube"
      },
      "units": [
        {
          "order": 100,
          "unit_id": "unit_s1_g3_sec1500_100",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "立方体の展開図とは何かを知り，6つの面をつなげた「開いた形」を理解する",
            "en": "Know what a cube net is and understand the ‘unfolded shape’ formed by connecting six faces"
          },
          "required_competency": {
            "ja": "立方体を展開した形（ネット図）を見て，どの面がどこと隣り合うかを判断し，正しいパターンか見分けられる",
            "en": "Examine a net of a cube and determine which faces are adjacent to each other, identifying whether it is a correct pattern"
          },
          "background": {
            "ja": "いらない箱を切り開く体験や，紙で立方体の模型を組み立てる活動を通して，平面と立体の対応関係を体感的に学ぶ",
            "en": "Through experiences such as cutting open a spare box or assembling a paper model of a cube, gain a hands-on understanding of the relationship between planar and solid forms"
          }
        },
        {
          "order": 200,
          "unit_id": "unit_s1_g3_sec1500_200",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "展開図を組み立てたときに，どの面が上・下・左右になるかイメージしながら図を描ける",
            "en": "Be able to draw a diagram while envisioning which faces become the top, bottom, and sides once the net is folded"
          },
          "required_competency": {
            "ja": "立方体の6面を見取図で示し，折り畳んだ場合の位置関係（例えば「正面の面と上面・右面はどこ？」など）を想定できる",
            "en": "Show the six faces of a cube in a rough sketch, and predict their positional relationships once folded (e.g., \"Where are the top and right faces relative to the front?\")"
          },
          "background": {
            "ja": "多様なパターン（「T字型」「十字型」「L字型」など）を提示し，どれが実際に立方体になるか試行錯誤する活動を取り入れて空間把握力を育む",
            "en": "Present various patterns (T-shaped, cross-shaped, L-shaped, etc.) and incorporate trial-and-error activities to determine which ones can form a cube, thereby fostering spatial reasoning skills"
          }
        },
        {
          "order": 300,
          "unit_id": "unit_s1_g3_sec1500_300",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "立方体の各面に数字や記号を書き込み，展開図と見取り図を対応させて考察できる",
            "en": "Write numbers or symbols on each face of a cube and correlate the net with a three-dimensional diagram"
          },
          "required_competency": {
            "ja": "ある面に書いてある数字や矢印などから，展開図でどこの面とつながるか，自力で推理し組み立てを想像できる",
            "en": "From the numbers or arrows on one face, deduce which face it connects to on the net, and imagine how the cube folds up"
          },
          "background": {
            "ja": "展開図上のマークや位置関係を利用しながら，脳内で立体を再構築する訓練をすることで，中学以降の立体図形問題（投影図など）への導入となる",
            "en": "By practicing mental reconstruction of a solid using marks and positional relationships on the net, build a foundation for tackling solid geometry problems (like projection diagrams) in later grades"
          }
        },
        {
          "order": 400,
          "unit_id": "unit_s1_g3_sec1500_400",
          "status": "PUBLISHED",
          "version": "1.0.0",
          "requirement": {
            "ja": "立方体の展開図から，辺の長さやすい直関係を説明し，立体の性質を言葉でまとめる",
            "en": "From the net of a cube, explain relationships such as edge lengths or perpendicularity, and summarize the solid’s properties verbally"
          },
          "required_competency": {
            "ja": "展開図において，どの辺が一致するかや面同士の平行・すい直を把握し，折りたたんだ立方体の空間構成を言語化できる",
            "en": "In the net, identify which edges coincide and which faces are parallel or perpendicular to each other, and articulate the spatial structure of the folded cube"
          },
          "background": {
            "ja": "面と面のつながり方を俯瞰することで，立方体のみならず他の多面体に対しても展開図を想像できるベースとなる",
            "en": "By overviewing how faces connect, establish a basis for visualizing not only cubes but also other polyhedra through their nets"
          }
        }
      ]
    }
  ]
}

```
