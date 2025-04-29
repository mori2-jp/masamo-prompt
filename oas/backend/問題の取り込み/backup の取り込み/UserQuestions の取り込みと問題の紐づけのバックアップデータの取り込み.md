元々はこのシステムの主キー(id)は uuid でしたが、事情により 主キー(id) auto increments に変更し、
元の主キー(uuid)は、uuid を移行しています。

今回取り込む CSV は以降前なので主キーが(uuid)になっています。

{参考 QuestionJson の取り込みコマンド} と {参考 questions CSV の取り込み}を参考にして、
{条件] に従い、{user_question_sets のパス｝　と {user_question_set_translations のパス} と {user_questions のパス} {user_question_translations のパス} の CSV から
user_question_sets 、user_question_set_translations, user_questions, user_question_translations　データをインサートするコマンドを作成してください

# 条件
user_id は、4 に固定してください。

questions は、 {questions のパス} の json_id が questions テーブルの json_id と対応している。
question_sets は、 {question_sets のパス} の json_id が question_sets テーブルの json_id と対応している。

user_question_sets の、question_set_id は、 question_sets の id を入れたいので、{user_question_sets のパス｝　の question_set_id  と対応する、{question_sets のパス｝ の CSV　の id　を探索して 該当した {question_sets のパス｝ のデータの json_id を元に question_sets テーブルを where('json_id') して question_set の ID を取得してください
user_questions の、question_id は、 questions の id を入れたいので、{user_questions のパス｝　の question_id  と対応する、{questions のパス｝ の CSV　の id　を探索して 該当した {questions のパス｝ のデータの json_id を元に questions テーブルを where('json_id') して questions の ID を取得してください

namespace は、namespace App\Console\Commands\Import\Backup;

それ以外のカラムはCSVと同様の名前のカラムに全てインサートして。

CSVには、JSONデータが含まれているので注意しないとJSON内のカンマなどが反応してエラーの原因になります

値が空文字だとエラーになるので、必ず全てに空文字チェックを入れて空文字なら　null を入れてください。

questions のパス
backup/questions.csv

question_sets のパス
backup/question_sets.csv

question_set_questions のパス
backup/question_set_questions.csv

user_question_sets のパス
backup/user_question_sets.csv

user_question_set_translations のパス
backup/user_question_set_translations.csv

user_questions のパス
backup/user_questions.csv

user_question_translations のパス
backup/user_question_translations.csv



↓{questions のパス｝にあるCSVデータの中身
id,level_id,grade_id,difficulty_id,json_id,metadata,version,status,evaluation_method,checker_method,llm_evaluation_prompt_file_name,evaluation_response_format,question_type,learning_requirement_json,learning_subject,learning_no,learning_requirement,learning_required_competency,learning_background,learning_category,learning_grade_level,learning_url,order,generated_by_llm,created_at,updated_at,deleted_at
7f7c14bb-6598-4ba2-bbbc-00bb42618cb6,857730f9-6862-4713-b139-d2d77fd0e64d,144e93ba-2af4-4fa8-8148-85467b5e1521,ec663c08-593f-4384-ae6f-a48ee4013e21,ques_s1_g3_sec100_u300_diff100_qt51_v100_1300,"{""question_type"":""FILL_IN_THE_BLANK"",""question_text"":{""ja"":""つぎの ▢ にあてはまる数を答えなさい。"",""en"":""Please answer the numbers that fit in the blanks.""},""explanation"":{""ja"":""4桁 + 3桁のたし算を、各くらいに分けて計算する問題です。くり上がりがあれば注意して整理しながら、筆算の手順を確認してみましょう。"",""en"":""This is an addition exercise of a four-digit number plus a three-digit number, broken down by place values. Watch out for any carries and carefully verify the column addition steps.""},""background"":{""ja"":""4桁 + 3桁の加法でも、位を意識した筆算を使いこなせるようになるための問題です。各位を分解して足し合わせることで、繰り上がりを見逃さずに正確な合計を求める練習になります。"",""en"":""The purpose of this problem is to ensure that learners can handle four-digit plus three-digit addition correctly, using place-value column addition. By breaking down each place, you can practice handling carries accurately.""},""question"":{""ja"":""3241 + 567 = (3000 + 0) + (200 + 500) + (40 + 60) + (1 + 7) = ▢ + ▢ + ▢ + ▢ = ▢"",""en"":""3241 + 567 = (3000 + 0) + (200 + 500) + (40 + 60) + (1 + 7) = ▢ + ▢ + ▢ + ▢ = ▢""},""input_format"":{""fields"":[{""field_id"":""f_1"",""attribute"":""number"",""user_answer"":""number""},{""field_id"":""f_2"",""attribute"":""number"",""user_answer"":""number""},{""field_id"":""f_3"",""attribute"":""number"",""user_answer"":""number""},{""field_id"":""f_4"",""attribute"":""number"",""user_answer"":""number""},{""field_id"":""f_5"",""attribute"":""number"",""user_answer"":""number""}],""question_components"":[{""type"":""text"",""content"":{""ja"":""3241 + 567 = "",""en"":""3241 + 567 = ""},""order"":50},{""type"":""newline"",""order"":100},{""type"":""text"",""content"":{""ja"":""(3000 + 0) + (200 + 500) + (40 + 60) + (1 + 7) = "",""en"":""(3000 + 0) + (200 + 500) + (40 + 60) + (1 + 7) = ""},""order"":150},{""type"":""newline"",""order"":200},{""type"":""input_field"",""field_id"":""f_1"",""order"":250},{""type"":""text"",""content"":{""ja"":"" + "",""en"":"" + ""},""order"":300},{""type"":""input_field"",""field_id"":""f_2"",""order"":350},{""type"":""text"",""content"":{""ja"":"" + "",""en"":"" + ""},""order"":400},{""type"":""input_field"",""field_id"":""f_3"",""order"":450},{""type"":""text"",""content"":{""ja"":"" + "",""en"":"" + ""},""order"":500},{""type"":""input_field"",""field_id"":""f_4"",""order"":550},{""type"":""text"",""content"":{""ja"":"" = "",""en"":"" = ""},""order"":600},{""type"":""input_field"",""field_id"":""f_5"",""order"":650}]}}",1.0.0,300,1,50,,"{""is_correct"":""boolean"",""score"":""number"",""question_text"":{""ja"":""つぎの ▢ にあてはまる数を答えなさい。"",""en"":""Please answer the numbers that fit in the blanks.""},""explanation"":{""ja"":""4桁の数と3桁の数のたし算です。千、百、十、一のくらいに分けて足すことで、くり上がりを整理しやすくなっています。"",""en"":""This is an addition problem involving a four-digit and a three-digit number. By splitting them into thousands, hundreds, tens, and ones places, it becomes easier to handle carrying.""},""question"":{""ja"":""3241 + 567 = (3000 + 0) + (200 + 500) + (40 + 60) + (1 + 7) = ▢ + ▢ + ▢ + ▢ = ▢"",""en"":""3241 + 567 = (3000 + 0) + (200 + 500) + (40 + 60) + (1 + 7) = ▢ + ▢ + ▢ + ▢ = ▢""},""fields"":[{""field_id"":""f_1"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":3000,""en"":3000},""field_explanation"":{""ja"":""千のくらいどうしを足すと3000です（3241の3000と567の0）。"",""en"":""Adding the thousands place gives 3000 (3000 from 3241 plus 0 from 567).""}},{""field_id"":""f_2"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":700,""en"":700},""field_explanation"":{""ja"":""百のくらいどうしを足すと700です（200+500）。"",""en"":""Adding the hundreds place results in 700 (200 + 500).""}},{""field_id"":""f_3"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":100,""en"":100},""field_explanation"":{""ja"":""十のくらいどうしを足すと100です（40+60）。"",""en"":""Adding the tens place yields 100 (40 + 60).""}},{""field_id"":""f_4"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":8,""en"":8},""field_explanation"":{""ja"":""一のくらいどうしを足すと8です（1+7）。"",""en"":""Adding the ones place gives 8 (1 + 7).""}},{""field_id"":""f_5"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":3808,""en"":3808},""field_explanation"":{""ja"":""3000 + 700 + 100 + 8 を合わせると3808となります。"",""en"":""Summing 3000 + 700 + 100 + 8 results in 3808.""}}]}",51,"[{""learning_subject"":""算数"",""learning_no"":37,""learning_requirement"":""計算の意味・方法 大きな数の概念と活用 3位数や4位数の加法及び減法"",""learning_required_competency"":""・3～4桁どうしの足し算・引き算を繰り上がり・繰り下がり含め正確に計算できる。"",""learning_background"":""・筆算の手順をしっかり確立させる。"",""learning_category"":""A"",""learning_grade_level"":""小3""}]",算数,37,計算の意味・方法 大きな数の概念と活用 3位数や4位数の加法及び減法,・3～4桁どうしの足し算・引き算を繰り上がり・繰り下がり含め正確に計算できる。,・筆算の手順をしっかり確立させる。,A,小3,,1300,0,2025-03-17 10:18:09,2025-03-26 02:46:49,
348a1d5d-c06c-48ea-b9ad-01150244ab4a,857730f9-6862-4713-b139-d2d77fd0e64d,144e93ba-2af4-4fa8-8148-85467b5e1521,ec663c08-593f-4384-ae6f-a48ee4013e21,ques_s1_g3_sec100_u400_diff100_qt51_v100_610,"{""question_type"":""FILL_IN_THE_BLANK"",""question"":{""ja"":""84 × 47 = (80×40) + (80×7) + (4×40) + (4×7) = ▢ + ▢ + ▢ + ▢ = ▢"",""en"":""84 × 47 = (80×40) + (80×7) + (4×40) + (4×7) = ▢ + ▢ + ▢ + ▢ = ▢""},""question_text"":{""ja"":""次の▢にあてはまる数を答えなさい。"",""en"":""Please fill in the blanks with the correct numbers.""},""explanation"":{""ja"":""位ごとに分けて部分積を求めることで、2桁同士の乗法を解きましょう。"",""en"":""By splitting the numbers into tens and ones, you can practice solving two-digit multiplication through partial products.""},""background"":{""ja"":""この問題では、より大きな数の乗法を理解するために、部分積の考え方を学びます。"",""en"":""This problem helps you understand multiplication of larger numbers by learning partial product strategies.""},""input_format"":{""type"":""fixed"",""fields"":[{""field_id"":""f_1"",""attribute"":""number"",""user_answer"":""number""},{""field_id"":""f_2"",""attribute"":""number"",""user_answer"":""number""},{""field_id"":""f_3"",""attribute"":""number"",""user_answer"":""number""},{""field_id"":""f_4"",""attribute"":""number"",""user_answer"":""number""},{""field_id"":""f_5"",""attribute"":""number"",""user_answer"":""number""}],""question_components"":[{""type"":""text"",""content"":{""ja"":""84 × 47 = (80×40) + (80×7) + (4×40) + (4×7) = "",""en"":""84 × 47 = (80×40) + (80×7) + (4×40) + (4×7) = ""},""order"":50,""attribute"":""text""},{""type"":""input_field"",""field_id"":""f_1"",""order"":100,""attribute"":""blank"",""content"":{""ja"":"""",""en"":""""}},{""type"":""text"",""content"":{""ja"":"" + "",""en"":"" + ""},""order"":150,""attribute"":""text""},{""type"":""input_field"",""field_id"":""f_2"",""order"":200,""attribute"":""blank"",""content"":{""ja"":"""",""en"":""""}},{""type"":""text"",""content"":{""ja"":"" + "",""en"":"" + ""},""order"":250,""attribute"":""text""},{""type"":""input_field"",""field_id"":""f_3"",""order"":300,""attribute"":""blank"",""content"":{""ja"":"""",""en"":""""}},{""type"":""text"",""content"":{""ja"":"" + "",""en"":"" + ""},""order"":350,""attribute"":""text""},{""type"":""input_field"",""field_id"":""f_4"",""order"":400,""attribute"":""blank"",""content"":{""ja"":"""",""en"":""""}},{""type"":""text"",""content"":{""ja"":"" = "",""en"":"" = ""},""order"":450,""attribute"":""text""},{""type"":""input_field"",""field_id"":""f_5"",""order"":500,""attribute"":""blank"",""content"":{""ja"":"""",""en"":""""}}]}}",1.0.0,300,1,50,,"{""is_correct"":""boolean"",""score"":""number"",""question_text"":{""ja"":""次の▢にあてはまる数を答えなさい。"",""en"":""Please fill in the blanks with the correct numbers.""},""explanation"":{""ja"":""2桁×2桁のかけ算を位に分解して、部分ごとの積を合計する練習です。80×40、80×7、4×40、4×7を順に計算し、すべてを足し合わせます。"",""en"":""In this practice problem, break down the two-digit multiplication into partial products and add them together. Calculate 80×40, 80×7, 4×40, and 4×7 in sequence and sum them all up.""},""question"":{""ja"":""84 × 47 = (80×40) + (80×7) + (4×40) + (4×7) = ▢ + ▢ + ▢ + ▢ = ▢"",""en"":""84 × 47 = (80×40) + (80×7) + (4×40) + (4×7) = ▢ + ▢ + ▢ + ▢ = ▢""},""fields"":[{""field_id"":""f_1"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":3200,""en"":3200},""field_explanation"":{""ja"":""80 × 40 は 3200 となります。"",""en"":""80 × 40 equals 3200.""}},{""field_id"":""f_2"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":560,""en"":560},""field_explanation"":{""ja"":""80 × 7 は 560 となります。"",""en"":""80 × 7 equals 560.""}},{""field_id"":""f_3"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":160,""en"":160},""field_explanation"":{""ja"":""4 × 40 は 160 となります。"",""en"":""4 × 40 equals 160.""}},{""field_id"":""f_4"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":28,""en"":28},""field_explanation"":{""ja"":""4 × 7 は 28 となります。"",""en"":""4 × 7 equals 28.""}},{""field_id"":""f_5"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":3948,""en"":3948},""field_explanation"":{""ja"":""3200 + 560 + 160 + 28 を足し合わせると 3948 となります。"",""en"":""Adding 3200, 560, 160, and 28 results in 3948.""}}]}",51,"[{""learning_subject"":""算数"",""learning_no"":38,""learning_requirement"":""計算の意味・方法 大きな数の概念と活用 2位数や3位数の乗法"",""learning_required_competency"":""・2桁×2桁や3桁×1桁など，大きめのかけ算を正確に行える。"",""learning_background"":""・計算量が増えるため，桁のそろえ方や筆算手順を重点的に練習する。"",""learning_category"":""A"",""learning_grade_level"":""小3"",""learning_url"":""""}]",算数,38,計算の意味・方法 大きな数の概念と活用 2位数や3位数の乗法,・2桁×2桁や3桁×1桁など，大きめのかけ算を正確に行える。,・計算量が増えるため，桁のそろえ方や筆算手順を重点的に練習する。,A,小3,,1776,1,2025-03-23 05:45:39,2025-03-23 05:45:39,
e4446631-5ae5-44fc-ae09-01282858654f,857730f9-6862-4713-b139-d2d77fd0e64d,144e93ba-2af4-4fa8-8148-85467b5e1521,ec663c08-593f-4384-ae6f-a48ee4013e21,ques_s1_g3_sec300_u100_diff100_qt251_v100_300,"{""question_type"":""CLASSIFY_THE_OPTIONS"",""question"":{""ja"":""28, 36, 39, 44, 54, 56, 62, 64, 81, 90"",""en"":""28, 36, 39, 44, 54, 56, 62, 64, 81, 90""},""question_text"":{""ja"":""つぎの2桁の整数を2, 3, 6, 8, 9で割り切れるように分けましょう"",""en"":""Classify these two-digit numbers according to which are divisible by 2, 3, 6, 8, and 9""},""explanation"":{""ja"":""この問題では、わり算の九九を活用し、あまりが出ない条件をもとに2桁の数を分類する力を養います。同じ数でも複数のグループに入る場合があるので注意して仕分けてみましょう。"",""en"":""In this task, you will use multiplication\/division facts to identify which numbers can be divided by 2, 3, 6, 8, or 9 without a remainder. Note that some numbers may belong to multiple categories.""},""background"":{""ja"":""1桁の除数による割り算を理解し、2桁の整数がどの数で割り切れるのかを把握する練習です。かけ算の知識を逆に使うことで素早く分類できるようになります。"",""en"":""This exercise helps learners practice single-digit divisors with two-digit numbers. By reversing multiplication facts, learners can quickly figure out which numbers fall into each divisibility category.""},""input_format"":{""input_components"":[{""type"":""number"",""content"":{""ja"":""28"",""en"":""28""},""order"":50},{""type"":""number"",""content"":{""ja"":""36"",""en"":""36""},""order"":100},{""type"":""number"",""content"":{""ja"":""39"",""en"":""39""},""order"":150},{""type"":""number"",""content"":{""ja"":""44"",""en"":""44""},""order"":200},{""type"":""number"",""content"":{""ja"":""54"",""en"":""54""},""order"":250},{""type"":""number"",""content"":{""ja"":""56"",""en"":""56""},""order"":300},{""type"":""number"",""content"":{""ja"":""62"",""en"":""62""},""order"":350},{""type"":""number"",""content"":{""ja"":""64"",""en"":""64""},""order"":400},{""type"":""number"",""content"":{""ja"":""81"",""en"":""81""},""order"":450},{""type"":""number"",""content"":{""ja"":""90"",""en"":""90""},""order"":500}],""fields"":[{""field_id"":""f_1"",""attribute"":""array"",""user_answer"":""sequence""},{""field_id"":""f_2"",""attribute"":""array"",""user_answer"":""sequence""},{""field_id"":""f_3"",""attribute"":""array"",""user_answer"":""sequence""},{""field_id"":""f_4"",""attribute"":""array"",""user_answer"":""sequence""},{""field_id"":""f_5"",""attribute"":""array"",""user_answer"":""sequence""}],""question_components"":[{""type"":""text"",""content"":{""ja"":""2で割り切れる数"",""en"":""Numbers divisible by 2""},""order"":50,""attribute"":""text""},{""type"":""input_field"",""field_id"":""f_1"",""order"":100},{""type"":""newline"",""order"":150},{""type"":""text"",""content"":{""ja"":""3で割り切れる数"",""en"":""Numbers divisible by 3""},""order"":250,""attribute"":""text""},{""type"":""input_field"",""field_id"":""f_2"",""order"":300},{""type"":""newline"",""order"":350},{""type"":""text"",""content"":{""ja"":""6で割り切れる数"",""en"":""Numbers divisible by 6""},""order"":450,""attribute"":""text""},{""type"":""input_field"",""field_id"":""f_3"",""order"":500},{""type"":""newline"",""order"":550},{""type"":""text"",""content"":{""ja"":""8で割り切れる数"",""en"":""Numbers divisible by 8""},""order"":650,""attribute"":""text""},{""type"":""input_field"",""field_id"":""f_4"",""order"":700},{""type"":""newline"",""order"":750},{""type"":""text"",""content"":{""ja"":""9で割り切れる数"",""en"":""Numbers divisible by 9""},""order"":850,""attribute"":""text""},{""type"":""input_field"",""field_id"":""f_5"",""order"":900},{""type"":""newline"",""order"":950}]}}",1.0.0,300,1,100,,"{""is_correct"":""boolean"",""score"":""number"",""question_text"":{""ja"":""次の2桁の整数を、2, 3, 6, 8, 9で割り切れる数ごとに分けましょう。"",""en"":""Classify the following two-digit numbers based on which are divisible by 2, 3, 6, 8, or 9.""},""explanation"":{""ja"":""この問題では、与えられた2桁の整数(28, 36, 39, 44, 54, 56, 62, 64, 81, 90)が、それぞれ2, 3, 6, 8, 9のいずれで割り切れるかを判別し、あまりが出ない分類に整理します。割り算の九九を使って効率よく確認してみましょう。"",""en"":""In this problem, you have ten two-digit numbers (28, 36, 39, 44, 54, 56, 62, 64, 81, 90). Determine which are divisible by 2, 3, 6, 8, or 9 with no remainder. Use your multiplication\/division facts to check each one.""},""question"":{""ja"":""28, 36, 39, 44, 54, 56, 62, 64, 81, 90"",""en"":""28, 36, 39, 44, 54, 56, 62, 64, 81, 90""},""fields"":[{""field_id"":""f_1"",""user_answer"":""sequence"",""is_correct"":""boolean"",""collect_answer"":{""ja"":""[28, 36, 44, 54, 56, 62, 64, 90]"",""en"":""[28, 36, 44, 54, 56, 62, 64, 90]""},""field_explanation"":{""ja"":""2で割り切れる、つまり偶数である数は28, 36, 44, 54, 56, 62, 64, 90です。"",""en"":""All even numbers in the list are divisible by 2: 28, 36, 44, 54, 56, 62, 64, and 90.""}},{""field_id"":""f_2"",""user_answer"":""sequence"",""is_correct"":""boolean"",""collect_answer"":{""ja"":""[36, 39, 54, 81, 90]"",""en"":""[36, 39, 54, 81, 90]""},""field_explanation"":{""ja"":""3で割り切れる数は、3の段を使ってあまりが出ないか調べると36, 39, 54, 81, 90です。"",""en"":""These are multiples of 3 with no remainder: 36, 39, 54, 81, and 90.""}},{""field_id"":""f_3"",""user_answer"":""sequence"",""is_correct"":""boolean"",""collect_answer"":{""ja"":""[36, 54, 90]"",""en"":""[36, 54, 90]""},""field_explanation"":{""ja"":""6で割り切れる数は、2の倍数かつ3の倍数である必要があります。36, 54, 90はいずれも6×(整数)で表せます。"",""en"":""Numbers divisible by 6 must be multiples of both 2 and 3: 36, 54, and 90.""}},{""field_id"":""f_4"",""user_answer"":""sequence"",""is_correct"":""boolean"",""collect_answer"":{""ja"":""[56, 64]"",""en"":""[56, 64]""},""field_explanation"":{""ja"":""8の倍数かどうかは、8×7=56, 8×8=64などの段で確認できます。"",""en"":""These numbers (56 and 64) are multiples of 8: 56 = 8×7 and 64 = 8×8.""}},{""field_id"":""f_5"",""user_answer"":""sequence"",""is_correct"":""boolean"",""collect_answer"":{""ja"":""[36, 54, 81, 90]"",""en"":""[36, 54, 81, 90]""},""field_explanation"":{""ja"":""9で割り切れる数は、各桁の合計が9の倍数になります。36, 54, 81, 90はいずれも9で割り切れます。"",""en"":""Numbers divisible by 9 have digit sums that are multiples of 9: 36, 54, 81, and 90 are all divisible by 9.""}}]}",251,"[{""learning_subject"":""算数"",""learning_no"":40,""learning_requirement"":""計算の意味・方法 割り算 1位数などの除法"",""learning_required_competency"":""1桁わり算(27÷3等)を九九を用いてスムーズに解ける。"",""learning_background"":""2桁以上の除法に進む基礎固め。逆算としてかけ算との結び付きも確認。"",""learning_category"":""A"",""learning_grade_level"":""小3""}]",算数,40,計算の意味・方法 割り算 1位数などの除法,1桁わり算(27÷3等)を九九を用いてスムーズに解ける。,2桁以上の除法に進む基礎固め。逆算としてかけ算との結び付きも確認。,A,小3,,300,0,2025-03-26 02:47:03,2025-03-26 02:47:03,
a5019024-0370-4d0f-80bd-015c68d7e3f3,857730f9-6862-4713-b139-d2d77fd0e64d,144e93ba-2af4-4fa8-8148-85467b5e1521,ec663c08-593f-4384-ae6f-a48ee4013e21,ques_s1_g3_sec100_u300_diff100_qt51_v100_213,"{""question_type"":""FILL_IN_THE_BLANK"",""question_text"":{""ja"":""▢にあてはまる数を答えなさい。"",""en"":""Please answer the numbers that fit in the blanks.""},""explanation"":{""ja"":""3桁同士の足し算で、繰り上がりが2回ある問題に取り組みましょう。"",""en"":""Let's practice adding two three-digit numbers with two carries.""},""background"":{""ja"":""この問題では、合計が4桁になる3桁の足し算を練習します。"",""en"":""In this problem, you will practice adding two three-digit numbers that result in a four-digit sum.""},""question"":{""ja"":""823 + 479 = (800 + 400) + (20 + 70) + (3 + 9) = ▢ + ▢ + ▢ = ▢"",""en"":""823 + 479 = (800 + 400) + (20 + 70) + (3 + 9) = ▢ + ▢ + ▢ = ▢""},""input_format"":{""type"":""fixed"",""fields"":[{""field_id"":""f_1"",""attribute"":""number"",""user_answer"":""number""},{""field_id"":""f_2"",""attribute"":""number"",""user_answer"":""number""},{""field_id"":""f_3"",""attribute"":""number"",""user_answer"":""number""},{""field_id"":""f_4"",""attribute"":""number"",""user_answer"":""number""}],""question_components"":[{""type"":""text"",""content"":{""ja"":""823 + 479 = "",""en"":""823 + 479 = ""},""order"":10},{""type"":""newline"",""order"":15},{""type"":""text"",""content"":{""ja"":""(800 + 400) + (20 + 70) + (3 + 9) = "",""en"":""(800 + 400) + (20 + 70) + (3 + 9) = ""},""order"":20},{""type"":""newline"",""order"":25},{""type"":""input_field"",""field_id"":""f_1"",""order"":30},{""type"":""text"",""content"":{""ja"":"" + "",""en"":"" + ""},""order"":40},{""type"":""input_field"",""field_id"":""f_2"",""order"":45},{""type"":""text"",""content"":{""ja"":"" + "",""en"":"" + ""},""order"":50},{""type"":""input_field"",""field_id"":""f_3"",""order"":60},{""type"":""text"",""content"":{""ja"":"" = "",""en"":"" = ""},""order"":70},{""type"":""input_field"",""field_id"":""f_4"",""order"":80}]}}",1.0.0,300,1,50,,"{""is_correct"":""boolean"",""score"":""number"",""question_text"":{""ja"":""▢にあてはまる数を答えなさい。"",""en"":""Please answer the numbers that fit in the blanks.""},""explanation"":{""ja"":""3桁同士の足し算で、繰り上がりが2回ある問題に取り組みましょう。"",""en"":""Let's practice adding two three-digit numbers with two carries.""},""question"":{""ja"":""823 + 479 = (800 + 400) + (20 + 70) + (3 + 9) = ▢ + ▢ + ▢ = ▢"",""en"":""823 + 479 = (800 + 400) + (20 + 70) + (3 + 9) = ▢ + ▢ + ▢ = ▢""},""fields"":[{""field_id"":""f_1"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":1200,""en"":1200},""field_explanation"":{""ja"":""800に400を足すと1200になります。"",""en"":""Adding 800 and 400 results in 1200.""}},{""field_id"":""f_2"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":90,""en"":90},""field_explanation"":{""ja"":""20に70を足すと90になります。"",""en"":""Adding 20 and 70 results in 90.""}},{""field_id"":""f_3"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":12,""en"":12},""field_explanation"":{""ja"":""3に9を足すと12になります。"",""en"":""Adding 3 and 9 gives 12.""}},{""field_id"":""f_4"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":1302,""en"":1302},""field_explanation"":{""ja"":""1200と90と12を足すと1302になります。"",""en"":""Summing 1200, 90, and 12 gives 1302.""}}]}",51,"[{""learning_subject"":""算数"",""learning_no"":37,""learning_requirement"":""計算の意味・方法 大きな数の概念と活用 3位数や4位数の加法及び減法"",""learning_required_competency"":""・3～4桁どうしの足し算・引き算を繰り上がり・繰り下がり含め正確に計算できる。"",""learning_background"":""・筆算の手順をしっかり確立させる。"",""learning_category"":""A"",""learning_grade_level"":""小3"",""learning_url"":""""}]",算数,37,計算の意味・方法 大きな数の概念と活用 3位数や4位数の加法及び減法,・3～4桁どうしの足し算・引き算を繰り上がり・繰り下がり含め正確に計算できる。,・筆算の手順をしっかり確立させる。,A,小3,,1699,1,2025-03-18 12:01:37,2025-03-18 12:01:37,
0e572334-4121-4fbb-a56e-015d8de8a43e,1e9506e6-fefd-4241-9193-e94681eb3d46,601a2782-79e1-437c-bc8e-0ac1db49ac43,ec663c08-593f-4384-ae6f-a48ee4013e21,ques_s1_g2_sec800_u100_diff100_qt51_v100_203,"{""question_type"":""FILL_IN_THE_BLANK"",""question"":{""ja"":""45 + 36 = 36 + ▢ = ▢"",""en"":""45 + 36 = 36 + ▢ = ▢""},""question_text"":{""ja"":""つぎの ▢ にあてはまる数を答えなさい。"",""en"":""Please answer the numbers that fit in the blanks.""},""explanation"":{""ja"":""2けたの足し算で順番を変えても結果が同じになる点を確認してみましょう。45 + 36 も 36 + 45 も答えは変わらず、加法の交換法則が成り立ちます。"",""en"":""Let's verify how reversing the order of two-digit addition doesn't change the result. Both 45 + 36 and 36 + 45 produce the same sum, confirming the commutative property.""},""background"":{""ja"":""この問題では、足し算の順番を変えても合計が同じになることを実際の数値で体感し、加法の交換法則を理解させる狙いがあります。繰り上がりのある場合でも同じ答えになることを確認しましょう。"",""en"":""This problem allows learners to experience that switching the order of addition yields the same sum, illustrating the commutative property. Even when carrying is involved, the sum remains unchanged.""},""input_format"":{""type"":""fixed"",""fields"":[{""field_id"":""f_1"",""attribute"":""number"",""user_answer"":""number""},{""field_id"":""f_2"",""attribute"":""number"",""user_answer"":""number""}],""question_components"":[{""type"":""text"",""content"":{""ja"":""45 + 36 = 36 + "",""en"":""45 + 36 = 36 + ""},""order"":1},{""type"":""input_field"",""field_id"":""f_1"",""order"":2},{""type"":""text"",""content"":{""ja"":"" = "",""en"":"" = ""},""order"":3},{""type"":""input_field"",""field_id"":""f_2"",""order"":4}]}}",1.0.0,300,1,50,,"{""is_correct"":""boolean"",""score"":""number"",""question"":{""ja"":""45 + 36 = 36 + ▢ = ▢"",""en"":""45 + 36 = 36 + ▢ = ▢""},""question_text"":{""ja"":""つぎの ▢ にあてはまる数を答えなさい。"",""en"":""Please answer the numbers that fit in the blanks.""},""explanation"":{""ja"":""45 と 36 の和は 81 です。足し算の順番を変えて 36 + 45 にしても、同じ 81 になることを確かめる問題です。これにより、加法の交換法則が成り立つことを確認します。"",""en"":""The sum of 45 and 36 is 81. Reversing the order to 36 + 45 yields the same result, confirming the commutative property of addition.""},""fields"":[{""field_id"":""f_1"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":45,""en"":45},""field_explanation"":{""ja"":""36 + 45 の形にするには、もとの 45 を入れて正しい式にしてください。"",""en"":""To form 36 + 45, insert the original 45.""}},{""field_id"":""f_2"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":81,""en"":81},""field_explanation"":{""ja"":""45 + 36 も 36 + 45 も合計は 81 です。"",""en"":""Both 45 + 36 and 36 + 45 total 81.""}}]}",51,"[{""learning_subject"":""算数"",""learning_no"":22,""learning_requirement"":""計算の意味・方法 計算の決まり 加法の交換法則を式で示す"",""learning_required_competency"":""a + b = b + a を具体例（3+5=5+3等）で理解し，同じ結果になることを確認できる"",""learning_background"":""実際に小物を数える・ブロックを移し替えるなど操作的活動と組み合わせると，法則への納得感が高まる。後の乗法の交換法則にも発展しやすい"",""learning_category"":""A"",""learning_grade_level"":""小2"",""learning_url"":""""}]",算数,22,計算の意味・方法 計算の決まり 加法の交換法則を式で示す,a + b = b + a を具体例（3+5=5+3等）で理解し，同じ結果になることを確認できる,実際に小物を数える・ブロックを移し替えるなど操作的活動と組み合わせると，法則への納得感が高まる。後の乗法の交換法則にも発展しやすい,A,小2,,213,1,2025-03-18 05:03:14,2025-03-18 08:42:34,
c088c85e-ef8c-430c-8ac0-02e1865b38f9,1e9506e6-fefd-4241-9193-e94681eb3d46,601a2782-79e1-437c-bc8e-0ac1db49ac43,ec663c08-593f-4384-ae6f-a48ee4013e21,ques_s1_g2_sec100_u300_diff100_qt51_v100_200,"{""question_type"":""FILL_IN_THE_BLANK"",""question"":{""ja"":""(4 + 2) + 3 = 4 + (2 + ▢) = ▢"",""en"":""(4 + 2) + 3 = 4 + (2 + ▢) = ▢""},""question_text"":{""ja"":""つぎの ▢ にあてはまる数を答えなさい。"",""en"":""Please answer the numbers that fit in the blanks.""},""explanation"":{""ja"":""たし算には、(a + b) + c と a + (b + c) のようにまとまりを変えても結果が同じになる『けつごうほうそく』という大事な性質があります。たとえば (4 + 2) + 3 と 4 + (2 + 3) は、どちらも同じ答えになることを確かめましょう。"",""en"":""Addition has an important property called the associative property, which means (a + b) + c and a + (b + c) produce the same result. For example, (4 + 2) + 3 and 4 + (2 + 3) both yield the same answer.""},""background"":{""ja"":""この問題は、加法の結合法則(a + b) + c = a + (b + c) を実際に体験し、たし算でのまとまり方を変えても答えが同じになることを理解してもらうために作られています。"",""en"":""This problem is designed to let learners experience the associative property of addition—(a + b) + c = a + (b + c)—and understand that changing how numbers are grouped does not change the result.""},""input_format"":{""fields"":[{""field_id"":""f_1"",""attribute"":""number"",""user_answer"":""number""},{""field_id"":""f_2"",""attribute"":""number"",""user_answer"":""number""}],""question_components"":[{""type"":""text"",""content"":{""ja"":""(4 + 2) + 3 = "",""en"":""(4 + 2) + 3 = ""},""order"":1},{""type"":""newline"",""order"":5},{""type"":""text"",""content"":{""ja"":""4 + (2 + "",""en"":""4 + (2 + ""},""order"":10},{""type"":""input_field"",""field_id"":""f_1"",""order"":20},{""type"":""text"",""content"":{""ja"":"") = "",""en"":"") = ""},""order"":30},{""type"":""input_field"",""field_id"":""f_2"",""order"":40}]}}",1.0.0,1,1,50,,"{""is_correct"":""boolean"",""score"":""number"",""question"":{""ja"":""(4 + 2) + 3 = 4 + (2 + ▢) = ▢"",""en"":""(4 + 2) + 3 = 4 + (2 + ▢) = ▢""},""question_text"":{""ja"":""つぎの ▢ にあてはまる数を答えなさい。"",""en"":""Please answer the numbers that fit in the blanks.""},""explanation"":{""ja"":""この式は加法の結合法則を示しています。(4 + 2) + 3 も 4 + (2 + 3) も計算結果が同じ9になります。まとめ方を変えても結果は変わらないので、空欄には『3』が入り、最終的に『9』になることを確認しましょう。"",""en"":""This equation illustrates the associative property of addition. Both (4 + 2) + 3 and 4 + (2 + 3) equal 9. The arrangement of the sums does not affect the final result, so the blank is '3' and the total becomes '9'.""},""fields"":[{""field_id"":""f_1"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":3,""en"":3},""field_explanation"":{""ja"":""text"",""en"":""text""}},{""field_id"":""f_2"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":9,""en"":9},""field_explanation"":{""ja"":""text"",""en"":""text""}}]}",51,"[{""learning_subject"":""算数"",""learning_no"":13,""learning_requirement"":""A 数と計算 計算の意味・方法 加法や減法に関して成り立つ性質"",""learning_required_competency"":""加法および減法について、交換法則(a + b = b + a)や結合法則((a + b) + c = a + (b + c))が成り立つことを具体的に理解できる"",""learning_background"":""具体例や操作活動で、順序・まとまりを変えても結果が変わらないことを確認し、高学年の式操作へつなぐ素地を作る"",""learning_category"":""A"",""learning_grade_level"":""小2"",""learning_url"":""https:\/\/docs.google.com\/spreadsheets\/d\/1W5vaFHcyU_BrMwb1JLZ-DpyFmXXZPaYMTHIITcwLqqY\/edit?gid=0#gid=0&range=13:13""}]",算数,13,A 数と計算 計算の意味・方法 加法や減法に関して成り立つ性質,加法および減法について、交換法則(a + b = b + a)や結合法則((a + b) + c = a + (b + c))が成り立つことを具体的に理解できる,具体例や操作活動で、順序・まとまりを変えても結果が変わらないことを確認し、高学年の式操作へつなぐ素地を作る,A,小2,https://docs.google.com/spreadsheets/d/1W5vaFHcyU_BrMwb1JLZ-DpyFmXXZPaYMTHIITcwLqqY/edit?gid=0#gid=0&range=13:13,200,0,2025-03-17 10:17:54,2025-03-26 02:46:36,

↓{question_sets のパス｝にあるCSVデータの中身
id,unit_id,json_id,version,status,order,generate_question_prompt,generate_question_prompt_file_name,llm_generation_status,consecutive_generation_failures,generation_blocked,created_at,updated_at,deleted_at
059c07b4-0e52-4db9-a12a-0248d9ca1e15,70f2ea84-7219-4260-87f2-09b8710b9027,qset_s1_g3_sec300_u200_v100_100,1.0.0,300,100,"{""ja"":""小学校3年生向けに、わり算の答えをかけ算で確かめる問題を1問作成してください。たとえば「36 ÷ 4 = 9 → 9 × 4 = 36」のように、わり算の結果をかけ算で確認する形式です。もしひき算の問題を出す場合は、途中でマイナスの値を取るような計算（たとえば「(9000 - 4000) + (300 - 800)…」のように途中で負の数が出るもの）は絶対に避けてください。わり算、かけ算、たし算、ひき算という言葉を使いながら、子どもにとってわかりやすい表現にしてください。"",""en"":""Please create one problem for third-grade students in which the learner verifies a division result by using multiplication. For example, “36 ÷ 4 = 9 → 9 × 4 = 36.” If you include subtraction, be sure not to produce any negative intermediate results (e.g., avoid situations like “(9000 - 4000) + (300 - 800)…”). Use the terms division, multiplication, addition, and subtraction in a clear, child-friendly way.""}",fill_in_the_blank,1,0,0,2025-03-24 14:34:02,2025-03-24 14:34:02,
c615e612-56c8-4200-a8d5-0479d4fba898,ca2a5fa3-27ca-4f8e-a62f-cbfcb3004d2e,qset_s1_g3_sec100_u300_v100_400,1.0.0,300,400,"{""ja"":""３桁－２桁のひき算の問題を１問作成してください。くり下がりが必要な場面があると分かりやすいようにし、位ごとの手順を意識できるような簡単な解説も添えてください。小学生向けの問題なのでマイナスの値を取らないようにしてください。例えば、「9361 - 4872 = (9000 - 4000) + (300 - 800) + (60 - 70) + (1 - 2) = ▢ + ▢ + ▢ + ▢ = ▢」のような問題だと、1=2の計算でマイナスが発生しています。こういうことは絶対に避けてください"",""en"":""Please create one subtraction problem that requires subtracting a two-digit number from a three-digit number, ensuring at least one instance of borrowing is needed. Provide a brief explanation to guide elementary learners on the step-by-step place-value process.Because these problems are for elementary school students, please ensure no negative values arise. For example, if you present a problem like “9361 − 4872 = (9000 − 4000) + (300 − 800) + (60 − 70) + (1 − 2) = ▢ + ▢ + ▢ + ▢ = ▢,” it creates a negative result in the calculation “1 − 2.” We must absolutely avoid such cases.""}",fill_in_the_blank,1,0,0,2025-03-18 08:23:26,2025-03-19 06:18:18,
dc850792-ca2d-4dec-bf52-055c2f7fe281,ca2a5fa3-27ca-4f8e-a62f-cbfcb3004d2e,qset_s1_g3_sec100_u300_v100_300,1.0.0,300,300,"{""ja"":""3桁の整数に2桁の整数を足す問題を1問作成してください。くり上がりが起きる場合と起きない場合のいずれでも対応できるよう、位を合わせて正しく計算できるように促す形式にしてください。問題文は小学生に分かりやすい書き方でまとめてください。"",""en"":""Please create a single problem that adds a two-digit number to a three-digit number. Emphasize correct place-value alignment and handling both carrying and non-carrying scenarios. Present the problem in clear, simple language for elementary-level learners.""}",fill_in_the_blank,1,0,0,2025-03-17 10:18:41,2025-03-17 10:18:41,

↓{user_question_sets のパス｝にあるCSVデータの中身
id,user_id,question_set_id,status,score,started_at,finished_at,created_at,updated_at,deleted_at
0c6f6107-005b-466a-a096-007c9437e2d1,3d846b4b-e9f1-445a-aede-50926e9e95ca,97cef556-b94e-4e05-bcc6-809279d1b759,1,,,,2025-03-25 12:28:22,2025-03-25 12:28:22,
83d44b74-e70c-43bf-bad9-16f8b94e2535,3d846b4b-e9f1-445a-aede-50926e9e95ca,e04593de-58f0-41e3-b6b9-a93a7587123b,100,,2025-03-22 12:17:01,2025-03-22 12:23:11,2025-03-22 12:17:00,2025-03-22 12:23:11,
d1928891-eb57-455f-9ded-2754af2d429a,3d846b4b-e9f1-445a-aede-50926e9e95ca,086fc4a7-c776-47e1-8db4-a4b0e713b2d7,100,,2025-03-22 12:23:49,2025-03-22 12:37:23,2025-03-22 12:23:49,2025-03-22 12:37:23,

↓{user_question_set_translations のパス｝にあるCSVデータの中身
id,user_question_set_id,locale,title,description,created_at,updated_at,deleted_at
1,fabe9112-5e54-4388-899f-8d37efd173cb,ja,2桁×1桁、簡単な3桁×1桁のかけ算,2桁の整数や、100の位が簡単な3桁の整数を1桁でかける練習をします。九九を活用しながら、繰り上がりに注意して正確に筆算できる力を身につけましょう。,2025-03-21 09:54:01,2025-03-21 09:54:01,
2,fabe9112-5e54-4388-899f-8d37efd173cb,en,Multiplying Two-Digit and Simple Three-Digit by One-Digit,"In this drill, you will practice multiplying two-digit numbers and simple three-digit numbers by a single digit. Make good use of multiplication tables and be attentive to any carrying as you work toward accurate and efficient written multiplication.",2025-03-21 09:54:01,2025-03-21 09:54:01,
3,c96aa854-ee1b-413b-968d-b39c504a156d,ja,大きな数の大小比較に挑戦しよう,このドリルでは、万や億などを含む大きな数の読み書きや、比較を中心に学びます。＜、＞、＝の記号を正しく使って、大きい数をしっかり比べられるようになりましょう。,2025-03-22 12:13:38,2025-03-22 12:13:38,


↓{user_questions のパス｝にあるCSVデータの中身
id,user_question_set_id,question_id,metadata,version,evaluation_method,checker_method,llm_evaluation_prompt_file_name,evaluation_response_format,question_type,learning_requirement_json,learning_subject,learning_no,learning_requirement,learning_required_competency,learning_background,learning_category,learning_grade_level,learning_url,generated_by_llm,status,answer_data,prompt_for_evaluation,answered_at,order,created_at,updated_at,deleted_at
bfebea9a-60c7-4c8d-8004-00cc7a39917a,23af290c-55c0-4aa4-8a05-29f2e0d89865,da6be600-7c31-4df7-a0bc-4752f2a9595e,"{""question_type"":""FILL_IN_THE_BLANK"",""question_format"":""NUMERIC_ANSWER"",""question_text"":{""ja"":""つぎの ▢ にあてはまる数を答えなさい。"",""en"":""Please answer the numbers that fit in the blanks.""},""explanation"":{""ja"":""4桁どうしの加法を位ごとに整理して計算する練習です。繰り上がりが複数回あるため、丁寧に筆算を行いましょう。"",""en"":""This problem practices adding two four-digit numbers by place value. Perform the column addition carefully as there are multiple carry-overs.""},""background"":{""ja"":""4桁+4桁の足し算を正確に行うための問題です。千、百、十、一の位を区別して合計を求める力を養います。"",""en"":""This problem focuses on accurately adding two four-digit numbers, distinguishing the thousands, hundreds, tens, and ones.""},""question"":{""ja"":""4987 + 3759 = (4000 + 3000) + (900 + 700) + (80 + 50) + (7 + 9) = ▢ + ▢ + ▢ + ▢ = ▢"",""en"":""4987 + 3759 = (4000 + 3000) + (900 + 700) + (80 + 50) + (7 + 9) = ▢ + ▢ + ▢ + ▢ = ▢""},""input_format"":{""type"":""fixed"",""fields"":[{""field_id"":""f_1"",""attribute"":""number"",""user_answer"":""number""},{""field_id"":""f_2"",""attribute"":""number"",""user_answer"":""number""},{""field_id"":""f_3"",""attribute"":""number"",""user_answer"":""number""},{""field_id"":""f_4"",""attribute"":""number"",""user_answer"":""number""},{""field_id"":""f_5"",""attribute"":""number"",""user_answer"":""number""}],""question_components"":[{""type"":""text"",""content"":{""ja"":""4987 + 3759 = "",""en"":""4987 + 3759 = ""},""order"":50},{""type"":""newline"",""order"":100},{""type"":""text"",""content"":{""ja"":""(4000 + 3000) + (900 + 700) + (80 + 50) + (7 + 9) = "",""en"":""(4000 + 3000) + (900 + 700) + (80 + 50) + (7 + 9) = ""},""order"":150},{""type"":""newline"",""order"":200},{""type"":""input_field"",""field_id"":""f_1"",""order"":250},{""type"":""text"",""content"":{""ja"":"" + "",""en"":"" + ""},""order"":300},{""type"":""input_field"",""field_id"":""f_2"",""order"":350},{""type"":""text"",""content"":{""ja"":"" + "",""en"":"" + ""},""order"":400},{""type"":""input_field"",""field_id"":""f_3"",""order"":450},{""type"":""text"",""content"":{""ja"":"" + "",""en"":"" + ""},""order"":500},{""type"":""input_field"",""field_id"":""f_4"",""order"":550},{""type"":""text"",""content"":{""ja"":"" = "",""en"":"" = ""},""order"":600},{""type"":""input_field"",""field_id"":""f_5"",""order"":650}]}}",1.0.0,1,50,,"{""is_correct"":""boolean"",""score"":""number"",""question_text"":{""ja"":""つぎの ▢ にあてはまる数を答えなさい。"",""en"":""Please answer the numbers that fit in the blanks.""},""explanation"":{""ja"":""4桁どうしの加法で、位ごとに丁寧に計算します。複数回のくり上がりが発生するので、1の位から順番に筆算を行いましょう。"",""en"":""This is a four-digit addition problem. Carefully add each place value, noting that multiple carry-overs occur.""},""question"":{""ja"":""4987 + 3759 = (4000 + 3000) + (900 + 700) + (80 + 50) + (7 + 9) = ▢ + ▢ + ▢ + ▢ = ▢"",""en"":""4987 + 3759 = (4000 + 3000) + (900 + 700) + (80 + 50) + (7 + 9) = ▢ + ▢ + ▢ + ▢ = ▢""},""fields"":[{""field_id"":""f_1"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":7000,""en"":7000},""field_explanation"":{""ja"":""千のくらい（4000 + 3000）の合計は7000です。"",""en"":""For the thousands place (4000 + 3000), the sum is 7000.""}},{""field_id"":""f_2"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":1600,""en"":1600},""field_explanation"":{""ja"":""百のくらい（900 + 700）は1600になります。"",""en"":""For the hundreds place (900 + 700), the sum is 1600.""}},{""field_id"":""f_3"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":130,""en"":130},""field_explanation"":{""ja"":""十のくらい（80 + 50）は130です。"",""en"":""For the tens place (80 + 50), the sum is 130.""}},{""field_id"":""f_4"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":16,""en"":16},""field_explanation"":{""ja"":""一のくらい（7 + 9）は16です。"",""en"":""For the ones place (7 + 9), the sum is 16.""}},{""field_id"":""f_5"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":8746,""en"":8746},""field_explanation"":{""ja"":""7000 + 1600 + 130 + 16 を合わせると8746になります。"",""en"":""Summing 7000, 1600, 130, and 16 yields 8746.""}}]}",51,"[{""learning_subject"":""算数"",""learning_no"":37,""learning_requirement"":""計算の意味・方法 大きな数の概念と活用 3位数や4位数の加法及び減法"",""learning_required_competency"":""・3～4桁どうしの足し算・引き算を繰り上がり・繰り下がり含め正確に計算できる。"",""learning_background"":""・筆算の手順をしっかり確立させる。"",""learning_category"":""A"",""learning_grade_level"":""小3""}]",算数,37,計算の意味・方法 大きな数の概念と活用 3位数や4位数の加法及び減法,・3～4桁どうしの足し算・引き算を繰り上がり・繰り下がり含め正確に計算できる。,・筆算の手順をしっかり確立させる。,A,小3,,1,1,,,,6,2025-03-24 01:57:03,2025-03-24 01:57:03,
e95c8286-4ed1-4581-9503-0288d44185a4,9212d375-7972-43bc-b2e6-bb9cbf4b004e,498175be-aede-4458-a41c-4d066b7f078a,"{""question_type"":""FILL_IN_THE_BLANK"",""question"":{""ja"":""42 ÷ 7 = ▢"",""en"":""42 ÷ 7 = ▢""},""question_text"":{""ja"":""つぎの ▢ にあてはまる数を答えなさい。"",""en"":""Please answer the numbers that fit in the blanks.""},""explanation"":{""ja"":""42÷7 の筆算・暗算を通して、わり算の考え方を身につけましょう。7 で 42 を分けたときに、ちょうど 6 回分になります。"",""en"":""Use either written or mental calculation for 42 ÷ 7. This helps you understand the concept of division; you will find that 42 splits into six equal parts of 7.""},""background"":{""ja"":""この問題は、2 桁 ÷ 1 桁のわり算に慣れるための練習です。あまりのないわり算を見つけ、九九を活用して解きましょう。"",""en"":""This problem helps you get comfortable with dividing two-digit numbers by single-digit numbers without remainders. Use your multiplication table to assist you.""},""input_format"":{""type"":""fixed"",""fields"":[{""field_id"":""f_1"",""attribute"":""number"",""user_answer"":""number""}],""question_components"":[{""type"":""text"",""content"":{""ja"":""42 ÷ 7 = "",""en"":""42 ÷ 7 = ""},""order"":50},{""type"":""input_field"",""field_id"":""f_1"",""order"":100,""attribute"":""blank"",""content"":{""ja"":"""",""en"":""""}}]}}",1.0.0,1,50,,"{""is_correct"":""boolean"",""score"":""number"",""question_text"":{""ja"":""つぎの ▢ にあてはまる数を答えなさい。"",""en"":""Please answer the numbers that fit in the blanks.""},""explanation"":{""ja"":""42 を 7 で割ると、7 は 6 回分でちょうど 42 になります。逆に 7 × 6 が 42 になることも確かめましょう。"",""en"":""When dividing 42 by 7, we see that 7 fits exactly 6 times into 42. You can confirm this by checking that 7 × 6 = 42.""},""question"":{""ja"":""42 ÷ 7 = ▢"",""en"":""42 ÷ 7 = ▢""},""fields"":[{""field_id"":""f_1"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":6,""en"":6},""field_explanation"":{""ja"":""7 が何回あるか数えると 6 回になるからです。"",""en"":""Because 7 goes into 42 exactly 6 times.""}}]}",51,"[{""learning_subject"":""算数"",""learning_no"":40,""learning_requirement"":""計算の意味・方法 割り算 1位数などの除法"",""learning_required_competency"":""1桁わり算(27÷3等)を九九を用いてスムーズに解ける。"",""learning_background"":""2桁以上の除法に進む基礎固め。逆算としてかけ算との結び付きも確認。"",""learning_category"":""A"",""learning_grade_level"":""小3"",""learning_url"":""""}]",算数,40,計算の意味・方法 割り算 1位数などの除法,1桁わり算(27÷3等)を九九を用いてスムーズに解ける。,2桁以上の除法に進む基礎固め。逆算としてかけ算との結び付きも確認。,A,小3,,1,1,,,,4,2025-03-25 12:48:27,2025-03-25 12:48:27,
ec3cd623-412b-40d2-96a8-0340d6c76fc8,b4f86bad-2025-4ad9-b86b-481d49746ca3,80621ccb-0528-478d-8b7d-87578ef3f363,"{""question_type"":""FILL_IN_THE_BLANK"",""question"":{""ja"":""8234 × 5 = (8000 × 5) + (200 × 5) + (30 × 5) + (4 × 5) = ▢ + ▢ + ▢ + ▢ = ▢"",""en"":""8234 × 5 = (8000 × 5) + (200 × 5) + (30 × 5) + (4 × 5) = ▢ + ▢ + ▢ + ▢ = ▢""},""question_text"":{""ja"":""つぎの ▢ にあてはまる数を答えなさい。"",""en"":""Please answer the numbers that fit in the blanks.""},""explanation"":{""ja"":""この問題は4桁の数を1桁の数でかけ算する練習です。位ごとに分解して合計する手順を身につけましょう。"",""en"":""In this problem, you will practice multiplying a four-digit number by a single-digit number. Break down each place value and sum them to reinforce accurate calculation.""},""background"":{""ja"":""この問題では4桁×1桁のかけ算を学習し、筆算時の位の管理を確実に行う力を育てます。"",""en"":""In this problem, you will practice a four-digit-by-one-digit multiplication, focusing on correct place value alignment during multiplication.""},""input_format"":{""type"":""fixed"",""fields"":[{""field_id"":""f_1"",""attribute"":""number"",""user_answer"":""number""},{""field_id"":""f_2"",""attribute"":""number"",""user_answer"":""number""},{""field_id"":""f_3"",""attribute"":""number"",""user_answer"":""number""},{""field_id"":""f_4"",""attribute"":""number"",""user_answer"":""number""},{""field_id"":""f_5"",""attribute"":""number"",""user_answer"":""number""}],""question_components"":[{""type"":""text"",""order"":50,""attribute"":""text"",""content"":{""ja"":""8234 × 5 = (8000 × 5) + (200 × 5) + (30 × 5) + (4 × 5) = "",""en"":""8234 × 5 = (8000 × 5) + (200 × 5) + (30 × 5) + (4 × 5) = ""}},{""type"":""input_field"",""field_id"":""f_1"",""order"":100,""attribute"":""blank"",""content"":{""ja"":"""",""en"":""""}},{""type"":""text"",""order"":150,""attribute"":""text"",""content"":{""ja"":"" + "",""en"":"" + ""}},{""type"":""input_field"",""field_id"":""f_2"",""order"":200,""attribute"":""blank"",""content"":{""ja"":"""",""en"":""""}},{""type"":""text"",""order"":250,""attribute"":""text"",""content"":{""ja"":"" + "",""en"":"" + ""}},{""type"":""input_field"",""field_id"":""f_3"",""order"":300,""attribute"":""blank"",""content"":{""ja"":"""",""en"":""""}},{""type"":""text"",""order"":350,""attribute"":""text"",""content"":{""ja"":"" + "",""en"":"" + ""}},{""type"":""input_field"",""field_id"":""f_4"",""order"":400,""attribute"":""blank"",""content"":{""ja"":"""",""en"":""""}},{""type"":""text"",""order"":450,""attribute"":""text"",""content"":{""ja"":"" = "",""en"":"" = ""}},{""type"":""input_field"",""field_id"":""f_5"",""order"":500,""attribute"":""blank"",""content"":{""ja"":"""",""en"":""""}}]}}",1.0.0,1,50,,"{""is_correct"":""boolean"",""score"":""number"",""question_text"":{""ja"":""つぎの ▢ にあてはまる数を答えなさい。"",""en"":""Please answer the numbers that fit in the blanks.""},""explanation"":{""ja"":""4桁の数に1桁の数をかける問題です。位ごとに分解し、合計することで正確に計算する方法を確認しましょう。"",""en"":""This problem involves multiplying a four-digit number by a single-digit number. Break down each place value and sum them up to ensure accurate calculation.""},""question"":{""ja"":""8234 × 5 = (8000 × 5) + (200 × 5) + (30 × 5) + (4 × 5) = ▢ + ▢ + ▢ + ▢ = ▢"",""en"":""8234 × 5 = (8000 × 5) + (200 × 5) + (30 × 5) + (4 × 5) = ▢ + ▢ + ▢ + ▢ = ▢""},""fields"":[{""field_id"":""f_1"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":40000,""en"":40000},""field_explanation"":{""ja"":""8000 × 5 で 40000 になります。"",""en"":""Because 8000 × 5 = 40000.""}},{""field_id"":""f_2"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":1000,""en"":1000},""field_explanation"":{""ja"":""200 × 5 で 1000 になります。"",""en"":""Because 200 × 5 = 1000.""}},{""field_id"":""f_3"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":150,""en"":150},""field_explanation"":{""ja"":""30 × 5 で 150 になります。"",""en"":""Because 30 × 5 = 150.""}},{""field_id"":""f_4"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":20,""en"":20},""field_explanation"":{""ja"":""4 × 5 で 20 になります。"",""en"":""Because 4 × 5 = 20.""}},{""field_id"":""f_5"",""user_answer"":""number"",""is_correct"":""boolean"",""collect_answer"":{""ja"":41170,""en"":41170},""field_explanation"":{""ja"":""40000 + 1000 + 150 + 20 の合計は 41170 です。"",""en"":""Because 40000 + 1000 + 150 + 20 = 41170.""}}]}",51,"[{""learning_subject"":""算数"",""learning_no"":38,""learning_requirement"":""計算の意味・方法 大きな数の概念と活用 2位数や3位数の乗法"",""learning_required_competency"":""2桁×2桁や3桁×1桁など，大きめのかけ算を正確に行える。"",""learning_background"":""計算量が増えるため，桁のそろえ方や筆算手順を重点的に練習する。"",""learning_category"":""A"",""learning_grade_level"":""小3""}]",算数,38,計算の意味・方法 大きな数の概念と活用 2位数や3位数の乗法,2桁×2桁や3桁×1桁など，大きめのかけ算を正確に行える。,計算量が増えるため，桁のそろえ方や筆算手順を重点的に練習する。,A,小3,,1,100,"{""is_correct"":true,""score"":""100"",""question_text"":""\u3064\u304e\u306e \u25a2 \u306b\u3042\u3066\u306f\u307e\u308b\u6570\u3092\u7b54\u3048\u306a\u3055\u3044\u3002"",""explanation"":""4\u6841\u306e\u6570\u306b1\u6841\u306e\u6570\u3092\u304b\u3051\u308b\u554f\u984c\u3067\u3059\u3002\u4f4d\u3054\u3068\u306b\u5206\u89e3\u3057\u3001\u5408\u8a08\u3059\u308b\u3053\u3068\u3067\u6b63\u78ba\u306b\u8a08\u7b97\u3059\u308b\u65b9\u6cd5\u3092\u78ba\u8a8d\u3057\u307e\u3057\u3087\u3046\u3002"",""question"":""8234 \u00d7 5 = (8000 \u00d7 5) + (200 \u00d7 5) + (30 \u00d7 5) + (4 \u00d7 5) = \u25a2 + \u25a2 + \u25a2 + \u25a2 = \u25a2"",""fields"":[{""field_id"":""f_1"",""user_answer"":""40000"",""is_correct"":true,""collect_answer"":40000,""field_explanation"":""8000 \u00d7 5 \u3067 40000 \u306b\u306a\u308a\u307e\u3059\u3002""},{""field_id"":""f_2"",""user_answer"":""1000"",""is_correct"":true,""collect_answer"":1000,""field_explanation"":""200 \u00d7 5 \u3067 1000 \u306b\u306a\u308a\u307e\u3059\u3002""},{""field_id"":""f_3"",""user_answer"":""150"",""is_correct"":true,""collect_answer"":150,""field_explanation"":""30 \u00d7 5 \u3067 150 \u306b\u306a\u308a\u307e\u3059\u3002""},{""field_id"":""f_4"",""user_answer"":""20"",""is_correct"":true,""collect_answer"":20,""field_explanation"":""4 \u00d7 5 \u3067 20 \u306b\u306a\u308a\u307e\u3059\u3002""},{""field_id"":""f_5"",""user_answer"":""41170"",""is_correct"":true,""collect_answer"":41170,""field_explanation"":""40000 + 1000 + 150 + 20 \u306e\u5408\u8a08\u306f 41170 \u3067\u3059\u3002""}]}",,2025-03-24 12:03:40,5,2025-03-24 12:01:00,2025-03-24 12:03:40,


↓{user_question_translations のパス｝にあるCSVデータの中身
id,user_question_id,locale,question_text,explanation,background,created_at,updated_at,deleted_at
1,633cc426-06b5-427b-bd7e-8fda64c1cd63,ja,つぎの ▢ にあてはまる数を答えなさい。,128×4のような3桁未満のかけ算も、位ごとに数を分けてかけ算を行うことで、筆算がスムーズに行えます。百の位・十の位・一の位をそれぞれ計算した後、結果を合計します。,この問題では、2桁〜3桁の整数を1桁の整数でかける筆算の手順を理解することが目的です。繰り上がりを意識しながら処理を行い、確実に答えを導きましょう。,2025-03-21 09:54:01,2025-03-21 09:54:01,
2,633cc426-06b5-427b-bd7e-8fda64c1cd63,en,Please fill in the numbers that fit the blanks.,"When multiplying numbers like 128×4, it's helpful to break the number down by place value—hundreds, tens, and ones. Multiply each digit individually and then sum the partial products.",This problem aims to help you understand how to multiply two- to three-digit numbers by a single-digit number. Pay attention to carrying if needed and carefully compute the total.,2025-03-21 09:54:01,2025-03-21 09:54:01,
3,3a3a7041-93f8-44b0-a141-1dfaeabddd18,ja,つぎの ▢ にあてはまる数を答えなさい。,236×3のような3桁の整数を1桁の整数でかけるときも、位ごとに分けて計算し、最後に結果を合計すると計算が容易になります。,この問題では、2桁〜3桁の整数を1桁の整数でかける筆算の手順を理解することが目的です。繰り上がりを意識しながら正確に答えを導きましょう。,2025-03-21 09:54:01,2025-03-21 09:54:01,


# 実装方針
生成また修正変更するコードはそのコードだけは必ず省略せずに必ず全てアウトプットすること。

修正する場合は修正したコードは全て出力し、修正箇所以外のコードやコメントは現状のままにしてください。

仕様はソースコードにまとめてコメントとして残すこと
コメントを残す時に以下のようにコメントへ処理順を示すようなナンバリングは不用です。
// 7) memo => 空白
// 8) generate_question_prompt => そのまま
// 9) generate_question_prompt_file_name => そのまま

ちゃんと解説してね

変更のブランチ名とコミットメッセージを一行で


ーー DB
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
            $table->integer('llm_evaluation_prompt_file_name')->nullable();
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
        Schema::create('user_question_set_translations', function (Blueprint $table) {
            $table->id();
            $table->unsignedBigInteger('user_question_set_id');
            $table->string('locale', 10);
            $table->string('title')->nullable();
            $table->text('description')->nullable();
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('user_question_set_id')->references('id')->on('user_question_sets')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('user_question_set_translations');
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
        Schema::create('user_question_translations', function (Blueprint $table) {
            $table->id();
            $table->unsignedBigInteger('user_question_id');
            $table->string('locale', 10);
            $table->longText('question_text')->nullable();
            $table->longText('explanation')->nullable();
            $table->longText('background')->nullable();
            $table->timestamps();
            $table->softDeletes();

            $table->foreign('user_question_id')->references('id')->on('user_questions')->onDelete('cascade');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('user_question_translations');
    }
};
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

use App\Models\BaseModel;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

/**
 *
 *
 * @property-read \App\Models\User\TFactory|null $use_factory
 * @property-read \App\Models\User\UserQuestionSet|null $userQuestionSet
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSetTranslation newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSetTranslation newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSetTranslation onlyTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSetTranslation query()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSetTranslation withTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSetTranslation withoutTrashed()
 * @property int $id
 * @property string $user_question_set_id
 * @property string $locale
 * @property string|null $question_text
 * @property string|null $explanation
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property \Illuminate\Support\Carbon|null $deleted_at
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSetTranslation whereCreatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSetTranslation whereDeletedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSetTranslation whereExplanation($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSetTranslation whereId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSetTranslation whereLocale($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSetTranslation whereQuestionText($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSetTranslation whereUpdatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSetTranslation whereUserQuestionSetId($value)
 * @property string|null $title
 * @property string|null $description
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSetTranslation whereDescription($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionSetTranslation whereTitle($value)
 * @mixin \Eloquent
 */
class UserQuestionSetTranslation extends BaseModel
{
    use SoftDeletes, HasFactory;

    protected $fillable = [
        'user_question_set_id',
        'locale',
        'title',
        'description',
    ];

    public function userQuestionSet()
    {
        return $this->belongsTo(UserQuestionSet::class, 'user_question_set_id');
    }
}
<?php

namespace App\Models\User;

use App\Models\BaseModel;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

/**
 *
 *
 * @property int $id
 * @property string $user_question_id
 * @property string $locale
 * @property string|null $question_text
 * @property string|null $explanation
 * @property string|null $background
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property \Illuminate\Support\Carbon|null $deleted_at
 * @property-read \App\Models\User\TFactory|null $use_factory
 * @property-read \App\Models\User\UserQuestion $userQuestion
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionTranslation newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionTranslation newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionTranslation onlyTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionTranslation query()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionTranslation whereBackground($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionTranslation whereCreatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionTranslation whereDeletedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionTranslation whereExplanation($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionTranslation whereId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionTranslation whereLocale($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionTranslation whereQuestionText($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionTranslation whereUpdatedAt($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionTranslation whereUserQuestionId($value)
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionTranslation withTrashed()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|UserQuestionTranslation withoutTrashed()
 * @mixin \Eloquent
 */
class UserQuestionTranslation extends BaseModel
{
    use SoftDeletes, HasFactory;

    protected $fillable = [
        'user_question_id',
        'locale',
        'question_text',
        'explanation',
        'background',
    ];

    public function userQuestion()
    {
        return $this->belongsTo(UserQuestion::class, 'user_question_id');
    }
}

```

---　参考 QuestionJson の取り込みコマンド

```php
<?php

namespace App\Console\Commands\Import\Github;

use App\Enums\EvaluationCheckerMethod;
use App\Enums\EvaluationMethod;
use App\Enums\QuestionStatus;
use App\Enums\QuestionType;
use App\Models\Difficulty\Difficulty;
use App\Models\Grade\Grade;
use App\Models\Level\Level;
use App\Models\Question\Question;
use App\Models\Question\QuestionTranslation;
use App\Services\Utils\Question\QuestionJsonManageService;
use Google_Client;
use Google_Service_Drive;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Str;

/**
 * 【仕様まとめ】
 * 1) staging や local 環境で Google Drive 上の JSON を取り込みたい場合、
 *    サービスアカウント (service-account-credentials.json) で認証し、Drive API を介してダウンロードする。
 * 2) Google Drive でファイルを「service account」に対して共有(閲覧可)し、かつ GCP で Drive API を有効化しておく必要がある。
 * 3) $jsonFileOnGoogleDriveIds にファイルIDを記述しておくと、全て順番にダウンロードし upsertQuestion() へ渡す。
 * 4) GitHub からの取り込み (importFromGitHub) は既存の通り残し、環境が 'staging' なら Drive API、その他はローカル設定で切り替え。
 * 5) 既存の upsertQuestion(), processJsonFile(), fetchAllJsonFilesRecursively() 等のロジックはそのまま利用。
 */

class ImportQuestionsFromGithub extends Command
{
    /**
     * artisan で呼び出すコマンド名
     *
     * 例: php artisan import:questions-from-github --subject=math
     */
    protected $signature = 'import:questions-from-github
                            {--subject= : If specified, only import files under contents/questions/{subject} }';

    protected $description = 'Import question data from GitHub or Google Drive JSON files and sync DB';

    /**
     * Google Drive 上の "ファイルID" 一覧 (サービスアカウント認証で取得する)
     *
     * 例:
     *   https://drive.google.com/file/d/1jPnYxj8VxeUT5JmnAayPUpl-2BnEJ-ax/view?usp=sharing
     *   ↑のURLの「1jPnYxj8VxeUT5JmnAayPUpl-2BnEJ-ax」がファイルID。
     */
    protected array $jsonFileOnGoogleDriveIds = [
        '1o6db1AIwflqfraGwC6IU2re18Zxtv3Kc',
        '1D0AG1gR9Lt7GO5O8hDqlTgRoVxDGCoMH',
        '1OOxfWOTVixHRqTWpQWXVOM1BcdWMBnxi'
    ];
    /**
     * 「取り込もうとしたJSONの総数」
     */
    private int $totalCount = 0;

    /**
     * 「取り込みに失敗またはスキップしたJSONの総数」
     */
    private int $skipCount = 0;

    /**
     * 「取り込みに成功したJSONの総数」
     */
    private int $successCount = 0;

    /**
     * メインのハンドラ
     */
    public function handle()
    {
        // 環境判定
        $env = app()->environment(); // 'local', 'staging', etc
        $useGoogleDrive = false;

        if ($env === 'staging') {
            // staging は必ず Google Drive
            $useGoogleDrive = true;
        } elseif ($env === 'local') {
            // local では config('app.use_google_drive_in_local_for_learning_data_import', true) の値を参照
            $useGoogleDrive = config('app.use_google_drive_in_local_for_learning_data_import', true);
        }

        if ($useGoogleDrive) {
            $this->info("【INFO】Running in '{$env}' environment => Google Drive (Service Account認証) からJSONをダウンロードして処理します。");
            $this->importFromGoogleDriveWithServiceAccount();
        } else {
            $this->info("【INFO】Running in '{$env}' environment => GitHub からJSONをダウンロードして処理します。");
            $this->importFromGitHub();
        }

        // 統計出力
        $this->info("Import process completed.");
        $this->line("=======================================");
        $this->info("取り込もうとした Question JSON の総数: {$this->totalCount}");
        $this->info("取り込みに失敗またはスキップした Question JSON の総数: {$this->skipCount}");
        $this->info("取り込みに成功した Question JSON の総数: {$this->successCount}");
        $this->line("=======================================");

        return 0;
    }

    /**
     * 【B案】Google Drive のファイルを
     * Service Account (service-account-credentials.json) で認証して Drive API 経由でダウンロードする。
     *
     * jsonFileOnGoogleDriveIds に記載された複数ファイルを順に読み取り、DBに反映する。
     */
    private function importFromGoogleDriveWithServiceAccount(): void
    {
        $this->info('Setting up Google Client for Drive API (Questions) ...');

        // 1) Googleクライアントの認証設定
        //    例: storage_path('app/private/service-account-credentials.json')
        $client = new Google_Client();
        $client->setAuthConfig(storage_path('app/private/service-account-credentials.json'));
        $client->addScope(Google_Service_Drive::DRIVE_READONLY);

        // 2) Drive サービスインスタンス
        $driveService = new Google_Service_Drive($client);

        // 3) ファイルID をループし、中身(JSON)をダウンロード => upsert
        foreach ($this->jsonFileOnGoogleDriveIds as $fileId) {
            $this->info("Fetching JSON from Google Drive fileId: {$fileId}");

            // 総数カウント
            $this->totalCount++;

            try {
                // alt=media で取得 (認証済ストリーム)
                $response = $driveService->files->get($fileId, ['alt' => 'media']);
                $jsonContent = $response->getBody()->getContents();

                $decoded = json_decode($jsonContent, true);
                if (!is_array($decoded)) {
                    $this->warn("File ID={$fileId} is not valid JSON. Skipped.");
                    $this->skipCount++;
                    continue;
                }

                // DB upsert
                if ($this->upsertQuestion($decoded)) {
                    $this->successCount++;
                } else {
                    $this->skipCount++;
                }
            } catch (\Exception $e) {
                $this->warn("Failed to fetch from Google Drive: fileId={$fileId}, error=" . $e->getMessage());
                $this->skipCount++;
            }
        }
    }

    /**
     * GitHubリポジトリ上の JSON を取り込み（オリジナルロジックを温存）
     */
    private function importFromGitHub(): void
    {
        $token = config('services.github.api_token');
        if (!$token) {
            $this->error('GitHub API token not found in config/services.php [github.api_token]');
            return;
        }

        $repoOwnerAndName = 'NousContentsManagement/masamo-content';

        // ベースパス: contents/questions
        $basePath = 'contents/questions';

        // subject オプション
        $subjectOption = $this->option('subject');
        if ($subjectOption) {
            $basePath .= '/' . $subjectOption;
            $this->info("Subject specified: {$subjectOption}, path= {$basePath}");
        } else {
            $this->info("No subject specified. Will import from all subdirectories under contents/questions");
        }

        $allJsonFiles = $this->fetchAllJsonFilesRecursively($repoOwnerAndName, $basePath, $token);
        if (empty($allJsonFiles)) {
            $this->warn("No JSON files found under path: {$basePath}");
            return;
        }

        $this->info("Found " . count($allJsonFiles) . " JSON file(s) under {$basePath}");

        // 各ファイルを処理
        foreach ($allJsonFiles as $file) {
            $this->processJsonFile($file, $token);
        }
    }

    /**
     * 再帰的に GitHub 上のディレクトリを走査し、.json ファイル一覧を返す
     */
    private function fetchAllJsonFilesRecursively(string $repo, string $path, string $token): array
    {
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
            $type = $item['type'] ?? null;  // 'file' or 'dir'
            $itemPath = $item['path'] ?? '';
            $name     = $item['name'] ?? '';

            if ($type === 'dir') {
                $descendantFiles = $this->fetchAllJsonFilesRecursively($repo, $itemPath, $token);
                $result = array_merge($result, $descendantFiles);
            } elseif ($type === 'file') {
                if (str_ends_with($name, '.json')) {
                    $result[] = $item;
                }
            }
        }

        return $result;
    }

    /**
     * JSONファイル一件をダウンロードしてパースし、DBに反映 (GitHub用)
     */
    private function processJsonFile(array $file, string $token): void
    {
        $this->totalCount++;

        if (!isset($file['download_url'])) {
            $this->warn("No download_url for file: " . ($file['path'] ?? 'unknown'));
            $this->skipCount++;
            return;
        }

        $this->info("Fetching JSON: " . $file['path']);

        $jsonContent = Http::withToken($token)
            ->get($file['download_url'])
            ->body();

        $decoded = json_decode($jsonContent, true);
        if (!is_array($decoded)) {
            $this->warn("File {$file['name']} is not valid JSON. Skipped.");
            $this->skipCount++;
            return;
        }

        if ($this->upsertQuestion($decoded)) {
            $this->successCount++;
        } else {
            $this->skipCount++;
        }
    }

    /**
     * 単一の問題(JSON)をDBへ upsert
     *
     * @return bool 成功ならtrue、スキップ/失敗ならfalse
     */
    private function upsertQuestion(array $questionJson): bool
    {
        DB::beginTransaction();
        try {
            $jsonId = $questionJson['id'] ?? null;
            if (!$jsonId) {
                $this->warn("Question JSON missing 'id'. Skipped.");
                DB::rollBack();
                return false;
            }

            // metadata が無いならスキップ
            $metadata = $questionJson['metadata'] ?? [];
            if (empty($metadata)) {
                $this->warn("metadata が存在しないためスキップ: json_id={$jsonId}");
                DB::rollBack();
                return false;
            }

            // JSONバリデーション
            try {
                $jsonManager = app(QuestionJsonManageService::class);
                $jsonManager->validateQuestionJson($questionJson);
            } catch (\Illuminate\Validation\ValidationException $ve) {
                $this->warn("バリデーションエラーのため問題(json_id={$jsonId})をスキップ:");
                foreach ($ve->errors() as $field => $msgs) {
                    foreach ($msgs as $msg) {
                        $this->warn("  - {$field}: {$msg}");
                    }
                }
                DB::rollBack();
                return false;
            }

            // 既存レコードを検索 (json_id = $jsonId)
            $question = Question::where('json_id', $jsonId)->first();

            // order, version
            $jsonOrder = $questionJson['order'] ?? 9999;
            $version   = $questionJson['version'] ?? '0.0.1';

            // status
            $rawStatus = $questionJson['status'] ?? null;
            $statusValue = $rawStatus
                ? $this->parseQuestionStatus($rawStatus)
                : QuestionStatus::DRAFT->value;

            // level_id / difficulty_id / grade_id
            $levelUuid      = $this->findLevelUuid($questionJson['level_id'] ?? null);
            $gradeUuid      = $this->findGradeUuid($questionJson['grade_id'] ?? null);
            $difficultyUuid = $this->findDifficultyUuid($questionJson['difficulty_id'] ?? null);

            // 新規or既存
            if (!$question) {
                $question = new Question();
                $question->uuid      = (string) Str::uuid();
                $question->json_id = $jsonId;
            }

            // 項目反映
            $question->order = $questionJson['order'];
            $question->level_id      = $levelUuid;
            $question->grade_id      = $gradeUuid;
            $question->difficulty_id = $difficultyUuid;
            $question->version       = $version;
            $question->status        = $statusValue;

            // question->metadata
            $question->metadata = json_encode($metadata, JSON_UNESCAPED_UNICODE);

            // question_type
            $rawQuestionType = $metadata['question_type'] ?? null;
            $question->question_type = $this->parseQuestionType($rawQuestionType);

            // evaluation_spec
            if (isset($questionJson['evaluation_spec'])) {
                $em = $questionJson['evaluation_spec']['evaluation_method'] ?? null;
                $cm = $questionJson['evaluation_spec']['checker_method'] ?? null;
                $question->evaluation_method = $em
                    ? EvaluationMethod::fromString($em)
                    : null;
                $question->checker_method = $cm
                    ? EvaluationCheckerMethod::fromString($cm)
                    : null;

                $question->llm_evaluation_prompt_file_name = $questionJson['evaluation_spec']['llm_prompt_file_name'] ?? null;

                $rf = $questionJson['evaluation_spec']['response_format'] ?? null;
                $question->evaluation_response_format = $rf
                    ? json_encode($rf, JSON_UNESCAPED_UNICODE)
                    : null;
            }

            // generated_by_llm
            if (array_key_exists('generated_by_llm', $questionJson)) {
                $question->generated_by_llm = (bool)$questionJson['generated_by_llm'];
            }

            // 学習要件
            $this->applyLearningRequirements($question, $questionJson);

            $question->save();

            // question_text / explanation / background
            $this->upsertQuestionTranslations($question, $questionJson);

            // skills → pivot
            $this->applySkillsPivot($question, $questionJson);

            DB::commit();
            $this->info("Upserted question json_id={$jsonId}");
            return true;

        } catch (\Throwable $e) {
            DB::rollBack();
            $this->error("Error upserting question json_id={$jsonId}: " . $e->getMessage());
            return false;
        }
    }

    /**
     * JSON の "skills" 配列 → question_skill ピボットを同期
     */
    private function applySkillsPivot(Question $question, array $questionJson)
    {
        $skillsFromJson = $questionJson['skills'] ?? [];

        $newSkillDbIds = [];
        $skillIndexMap = [];

        foreach ($skillsFromJson as $index => $skillData) {
            $skillJsonId = $skillData['skill_id'] ?? null;
            if (!$skillJsonId) {
                continue;
            }

            // skills テーブルの json_id => id
            $skillDbId = DB::table('skills')
                ->where('json_id', $skillJsonId)
                ->value('id');

            if (!$skillDbId) {
                $this->warn("Skill with json_id='{$skillJsonId}' not found. Skipped linking.");
                continue;
            }

            $newSkillDbIds[] = $skillDbId;
            $skillIndexMap[$skillDbId] = $index + 1;
        }

        // 既存 pivot
        $existingSkillDbIds = DB::table('question_skill')
            ->where('question_id', $question->id)
            ->pluck('skill_id')
            ->toArray();

        // 削除すべきスキル
        $skillIdsToRemove = array_diff($existingSkillDbIds, $newSkillDbIds);
        if (!empty($skillIdsToRemove)) {
            DB::table('question_skill')
                ->where('question_id', $question->id)
                ->whereIn('skill_id', $skillIdsToRemove)
                ->delete();
        }

        // 追加または更新
        foreach ($newSkillDbIds as $skillDbId) {
            $existingPivot = DB::table('question_skill')
                ->where('question_id', $question->id)
                ->where('skill_id', $skillDbId)
                ->first();

            if ($existingPivot) {
                DB::table('question_skill')
                    ->where('id', $existingPivot->id)
                    ->update([
                        'order'      => $skillIndexMap[$skillDbId],
                        'updated_at' => now(),
                    ]);
            } else {
                DB::table('question_skill')->insert([
                    'uuid'          => (string) Str::uuid(),
                    'question_id' => $question->id,
                    'skill_id'    => $skillDbId,
                    'order'       => $skillIndexMap[$skillDbId],
                    'created_at'  => now(),
                    'updated_at'  => now(),
                ]);
            }
        }
    }

    /**
     * question_text, explanation, background を question_translations へ upsert
     */
    private function upsertQuestionTranslations(Question $question, array $questionJson)
    {
        $locales = ['ja','en'];

        foreach ($locales as $locale) {
            $qText = $questionJson['metadata']['question_text'][$locale] ?? null;
            $qExp  = $questionJson['metadata']['explanation'][$locale]   ?? null;
            $qBack = $questionJson['metadata']['background'][$locale]    ?? null;

            if ($qText !== null || $qExp !== null || $qBack !== null) {
                QuestionTranslation::updateOrCreate(
                    [
                        'question_id' => $question->id,
                        'locale'      => $locale,
                    ],
                    [
                        'question_text' => $qText,
                        'explanation'   => $qExp,
                        'background'    => $qBack,
                    ]
                );
            }
        }
    }

    /**
     * JSON の learning_requirements を反映
     */
    private function applyLearningRequirements(Question $question, array $questionJson)
    {
        $items = $questionJson['learning_requirements'] ?? [];
        if (!is_array($items) || empty($items)) {
            return;
        }
        $question->learning_requirement_json = json_encode($items, JSON_UNESCAPED_UNICODE);

        $subjects             = [];
        $nos                  = [];
        $requirements         = [];
        $requiredCompetencies = [];
        $backgrounds          = [];
        $categories           = [];
        $gradeLevels          = [];
        $urls                 = [];

        foreach ($items as $lr) {
            if (isset($lr['learning_subject'])) {
                $subjects[] = $lr['learning_subject'];
            }
            if (isset($lr['learning_no'])) {
                $nos[] = (string)$lr['learning_no'];
            }
            if (isset($lr['learning_requirement'])) {
                $requirements[] = $lr['learning_requirement'];
            }
            if (isset($lr['learning_required_competency'])) {
                $requiredCompetencies[] = $lr['learning_required_competency'];
            }
            if (isset($lr['learning_background'])) {
                $backgrounds[] = $lr['learning_background'];
            }
            if (isset($lr['learning_category'])) {
                $categories[] = $lr['learning_category'];
            }
            if (isset($lr['learning_grade_level'])) {
                $gradeLevels[] = $lr['learning_grade_level'];
            }
            if (isset($lr['learning_url'])) {
                $urls[] = $lr['learning_url'];
            }
        }

        $question->learning_subject             = implode("\n", $subjects);
        $question->learning_no                  = !empty($nos) ? (int)$nos[0] : null;
        $question->learning_requirement         = implode("\n", $requirements);
        $question->learning_required_competency = implode("\n", $requiredCompetencies);
        $question->learning_background          = implode("\n", $backgrounds);
        $question->learning_category            = implode("\n", $categories);
        $question->learning_grade_level         = implode("\n", $gradeLevels);
        $question->learning_url                 = implode("\n", $urls);
    }

    /**
     * JSON の level_id => levelsテーブル json_idカラム => uuid
     */
    private function findLevelUuid(?string $levelJsonId): ?string
    {
        if (!$levelJsonId) {
            return null;
        }
        return Level::where('json_id', $levelJsonId)->value('id') ?: null;
    }

    /**
     * JSON の grade_id => gradesテーブル json_idカラム => uuid
     */
    private function findGradeUuid(?string $gradeJsonId): ?string
    {
        if (!$gradeJsonId) {
            return null;
        }
        return Grade::where('json_id', $gradeJsonId)->value('id') ?: null;
    }

    /**
     * JSON の difficulty_id => difficultiesテーブル json_idカラム => uuid
     */
    private function findDifficultyUuid(?string $difficultyJsonId): ?string
    {
        if (!$difficultyJsonId) {
            return null;
        }
        return Difficulty::where('json_id', $difficultyJsonId)->value('id') ?: null;
    }

    /**
     * question_type 文字列 -> Enum (int値) に変換
     */
    private function parseQuestionType(?string $typeString): int
    {
        if (!$typeString) {
            // デフォルト
            return QuestionType::CALCULATION->value;
        }

        try {
            return QuestionType::fromString($typeString)->value;
        } catch (\InvalidArgumentException) {
            return QuestionType::CALCULATION->value;
        }
    }

    /**
     * status 文字列 -> QuestionStatus enum (int) に変換
     */
    private function parseQuestionStatus(string $statusString): int
    {
        try {
            return QuestionStatus::fromString($statusString)->value;
        } catch (\InvalidArgumentException) {
            return QuestionStatus::DRAFT->value;
        }
    }
}
```

---- 参考 questions CSV の取り込み
```php
<?php

namespace App\Console\Commands\Import\Backup;

use App\Models\Difficulty\Difficulty;
use App\Models\Grade\Grade;
use App\Models\Level\Level;
use App\Models\Question\Question;
use App\Models\Question\QuestionTranslation;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\Str; // ★追加: UUID生成で使う

/**
 * 【仕様まとめ】
 * - levels.csv / grades.csv / difficulties.csv を先に読み込み、
 *   "oldId => json_id" の対応表を作成する。
 * - questions.csv
 *   - CSV上の id(旧uuid)を questionテーブルの uuid に保存する (主キーはauto increment)
 *   - CSV上の level_id, grade_id, difficulty_id は「該当 CSVファイル(levels.csv / grades.csv / difficulties.csv)」の id に対応。
 *     その id から json_id を取得し、DBの levels(grades/difficulties) を where('json_id') で検索して ID を特定。
 *     取得したIDを questionsテーブルの level_id, grade_id, difficulty_id にセット。
 *   - generated_by_llm が true のものだけインサート対象
 *   - そのほかのカラムは CSV列名と同名で questionsテーブルにインサートする (id列は uuid に相当)
 * - question_translations.csv
 *   - CSVの question_id が {questions.csv} の id(旧uuid) に対応している。
 *   - そのため「旧uuid => 新autoIncrementId」のマップを使い、 question_translations.question_id に新IDをセットして保存。
 *   - ほかのカラム(locale, question_text, explanation など)も空文字の場合はNULLに置き換え。
 * - CSVに JSONデータが含まれているため、fgetcsv() で区切り文字`,` / 囲み文字`"` を正しく扱う。
 *
 * ※本サンプルでは最小限のエラーハンドリングです。実運用では適宜バリデーション, トランザクション制御等を追加してください。
 */
class ImportQuestionsCsv extends Command
{
    protected $signature   = 'import:questions-new';
    protected $description = 'Import questions, question_translations from CSV and handle level/grade/difficulty references via CSV.';

    // CSV ファイルパス (例: storage/app/backup/*.csv)
    private string $levelsCsvPath       = 'backup/levels.csv';
    private string $gradesCsvPath       = 'backup/grades.csv';
    private string $difficultiesCsvPath = 'backup/difficulties.csv';

    private string $questionsCsvPath            = 'backup/questions.csv';
    private string $questionTranslationsCsvPath = 'backup/question_translations.csv';

    /**
     * levels.csv の "id => json_id"
     * @var array [oldLevelId => levelJsonId]
     */
    private array $levelsIdToJsonId = [];

    /**
     * grades.csv の "id => json_id"
     * @var array [oldGradeId => gradeJsonId]
     */
    private array $gradesIdToJsonId = [];

    /**
     * difficulties.csv の "id => json_id"
     * @var array [oldDiffId => diffJsonId]
     */
    private array $difficultiesIdToJsonId = [];

    /**
     * questions の 旧uuid => 新autoIncrementId
     * @var array [oldQuestionUuid => newQuestionId]
     */
    private array $questionUuidToNewId = [];

    public function handle()
    {
        $this->info('--- Start Import (Questions) ---');

        DB::beginTransaction();
        try {
            // 1) levels, grades, difficulties の CSV を読み込み => oldId => json_id のマップ
            $this->loadLevelsCsv();
            $this->loadGradesCsv();
            $this->loadDifficultiesCsv();

            // 2) questions.csv の取り込み
            $this->importQuestions();

            // 3) question_translations.csv の取り込み
            $this->importQuestionTranslations();

            // 途中でエラーなく完了した場合はコミット
            DB::commit();

            $this->info('--- Completed Import (Questions) ---');

        } catch (\Throwable $e) {
            // 何らかの例外が起きた場合はロールバック
            DB::rollBack();
            $this->error('!!! Import Failed. Rolled back !!!');
            $this->error($e->getMessage());
            // 必要に応じて return 1; とか throw $e; などで停止処理
            return 1;
        }
        return 0;
    }

    /**
     * CSV上の空文字を null に変換するユーティリティ。
     * 空文字やundefinedを受け取ったら null、それ以外はそのまま返す。
     */
    private function blankToNull(?string $value): ?string
    {
        return ($value === '' || $value === null) ? null : $value;
    }

    /**
     * CSVの int系カラム: 空文字 -> null、さもなければ (int)キャスト
     */
    private function parseIntOrNull(?string $value, ?int $defaultIfNull = null): ?int
    {
        if ($value === '' || $value === null) {
            return $defaultIfNull; // 例: null or 1
        }
        return (int)$value;
    }

    /**
     * CSVの datetime系カラム: 空文字 -> null, それ以外は文字列のままセット (Eloquentがparse)
     */
    private function parseDateTimeOrNull(?string $value): ?string
    {
        return ($value === '' || $value === null) ? null : $value;
    }

    /**
     * levels.csv から "id => json_id" のマップを作成
     */
    private function loadLevelsCsv(): void
    {
        $this->info("Reading: {$this->levelsCsvPath}");
        if (!Storage::exists($this->levelsCsvPath)) {
            $this->warn("File not found: {$this->levelsCsvPath} => skip");
            return;
        }

        $rows = $this->readCsvWithFgetcsv($this->levelsCsvPath);
        foreach ($rows as $row) {
            $oldId  = $this->blankToNull($row['id'] ?? null);
            $jsonId = $this->blankToNull($row['json_id'] ?? null);
            if ($oldId && $jsonId) {
                $this->levelsIdToJsonId[$oldId] = $jsonId;
            }
        }
        $this->info('Loaded ' . count($this->levelsIdToJsonId) . ' levels Id->json_id entries.');
    }

    /**
     * grades.csv から "id => json_id" のマップを作成
     */
    private function loadGradesCsv(): void
    {
        $this->info("Reading: {$this->gradesCsvPath}");
        if (!Storage::exists($this->gradesCsvPath)) {
            $this->warn("File not found: {$this->gradesCsvPath} => skip");
            return;
        }

        $rows = $this->readCsvWithFgetcsv($this->gradesCsvPath);
        foreach ($rows as $row) {
            $oldId  = $this->blankToNull($row['id'] ?? null);
            $jsonId = $this->blankToNull($row['json_id'] ?? null);
            if ($oldId && $jsonId) {
                $this->gradesIdToJsonId[$oldId] = $jsonId;
            }
        }
        $this->info('Loaded ' . count($this->gradesIdToJsonId) . ' grades Id->json_id entries.');
    }

    /**
     * difficulties.csv から "id => json_id" のマップを作成
     */
    private function loadDifficultiesCsv(): void
    {
        $this->info("Reading: {$this->difficultiesCsvPath}");
        if (!Storage::exists($this->difficultiesCsvPath)) {
            $this->warn("File not found: {$this->difficultiesCsvPath} => skip");
            return;
        }

        $rows = $this->readCsvWithFgetcsv($this->difficultiesCsvPath);
        foreach ($rows as $row) {
            $oldId  = $this->blankToNull($row['id'] ?? null);
            $jsonId = $this->blankToNull($row['json_id'] ?? null);
            if ($oldId && $jsonId) {
                $this->difficultiesIdToJsonId[$oldId] = $jsonId;
            }
        }
        $this->info('Loaded ' . count($this->difficultiesIdToJsonId) . ' difficulties Id->json_id entries.');
    }

    /**
     * questions.csv の取り込み
     */
    private function importQuestions(): void
    {
        $this->info("Reading questions from: {$this->questionsCsvPath}");
        if (!Storage::exists($this->questionsCsvPath)) {
            $this->warn("File not found: {$this->questionsCsvPath} => skip");
            return;
        }

        $rows = $this->readCsvWithFgetcsv($this->questionsCsvPath);
        $countInserted = 0;
        $countSkipped  = 0;

        foreach ($rows as $row) {
            $genLlm = $this->blankToNull($row['generated_by_llm'] ?? null);
            // "1" or "true" のみ対象
            if (! in_array($genLlm, ['1','true'], true)) {
                $countSkipped++;
                continue;
            }

            // 旧uuid
            $oldUuid = $this->blankToNull($row['id'] ?? null);
            if (!$oldUuid) {
                $countSkipped++;
                continue;
            }

            $question = new Question();
            $question->uuid = $oldUuid;

            // level_id
            $levelOldId = $this->blankToNull($row['level_id'] ?? null);
            $question->level_id = $this->resolveLevelDbId($levelOldId);

            // grade_id
            $gradeOldId = $this->blankToNull($row['grade_id'] ?? null);
            $question->grade_id = $this->resolveGradeDbId($gradeOldId);

            // difficulty_id
            $diffOldId = $this->blankToNull($row['difficulty_id'] ?? null);
            $question->difficulty_id = $this->resolveDifficultyDbId($diffOldId);

            // そのほか全列
            // 空文字 -> null
            $question->json_id                    = $this->blankToNull($row['json_id'] ?? null);
            $question->metadata                   = $this->blankToNull($row['metadata'] ?? null);
            $question->version                    = $this->blankToNull($row['version'] ?? '0.0.1');

            // numeric
            $question->status                     = $this->parseIntOrNull($row['status'] ?? null, 1);
            $question->evaluation_method          = $this->parseIntOrNull($row['evaluation_method'] ?? null, 1);

            // checker_method は数値かもしれないがNULL可
            // もし空文字ならnull, そうでなければint
            $chk = $this->blankToNull($row['checker_method'] ?? null);
            $question->checker_method = $chk ? (int)$chk : null;

            // llm_evaluation_prompt_file_name
            $lepf = $this->blankToNull($row['llm_evaluation_prompt_file_name'] ?? null);
            $question->llm_evaluation_prompt_file_name = $lepf ? (int)$lepf : null;

            $question->evaluation_response_format = $this->blankToNull($row['evaluation_response_format'] ?? null);

            // question_type
            $question->question_type = $this->parseIntOrNull($row['question_type'] ?? null, 1);

            // learning系
            $question->learning_requirement_json    = $this->blankToNull($row['learning_requirement_json'] ?? null);
            $question->learning_subject             = $this->blankToNull($row['learning_subject']          ?? null);
            $question->learning_no                  = $this->parseIntOrNull($row['learning_no'] ?? null);
            $question->learning_requirement         = $this->blankToNull($row['learning_requirement']      ?? null);
            $question->learning_required_competency = $this->blankToNull($row['learning_required_competency'] ?? null);
            $question->learning_background          = $this->blankToNull($row['learning_background']       ?? null);
            $question->learning_category            = $this->blankToNull($row['learning_category']         ?? null);
            $question->learning_grade_level         = $this->blankToNull($row['learning_grade_level']      ?? null);
            $question->learning_url                 = $this->blankToNull($row['learning_url']             ?? null);

            // order
            $question->order = $this->parseIntOrNull($row['order'] ?? null, 9999);

            // bool
            $question->generated_by_llm = true; // ここまで来てるなら true

            // 日付
            $question->created_at = $this->parseDateTimeOrNull($row['created_at'] ?? null) ?? now();
            $question->updated_at = $this->parseDateTimeOrNull($row['updated_at'] ?? null) ?? now();
            $question->deleted_at = $this->parseDateTimeOrNull($row['deleted_at'] ?? null);

            $question->save();

            // ★追加: question_skill テーブルに skill_id=4 で紐づけを作成
            DB::table('question_skill')->insert([
                'uuid'        => (string) Str::uuid(),  // ランダムUUID
                'question_id' => $question->id,         // 新しく生成された question.id
                'skill_id'    => 4,                     // 固定
                'order'       => 1,                     // 適当な順序
                'created_at'  => now(),
                'updated_at'  => now(),
            ]);
            // ★ここまで

            // 旧uuid => 新id
            $this->questionUuidToNewId[$oldUuid] = $question->id;
            $countInserted++;
        }

        $this->info("questions.csv import done. Inserted={$countInserted}, Skipped={$countSkipped}");
    }

    /**
     * question_translations.csv の取り込み
     */
    private function importQuestionTranslations(): void
    {
        $this->info("Reading question_translations from: {$this->questionTranslationsCsvPath}");
        if (!Storage::exists($this->questionTranslationsCsvPath)) {
            $this->warn("File not found: {$this->questionTranslationsCsvPath} => skip");
            return;
        }

        $rows = $this->readCsvWithFgetcsv($this->questionTranslationsCsvPath);
        $countInserted = 0;
        $countSkipped  = 0;

        foreach ($rows as $row) {
            // 旧uuid
            $oldQid = $this->blankToNull($row['question_id'] ?? null);
            if (!$oldQid || !isset($this->questionUuidToNewId[$oldQid])) {
                $countSkipped++;
                continue;
            }
            $newQuestionId = $this->questionUuidToNewId[$oldQid];

            $tr = new QuestionTranslation();
            $tr->question_id   = $newQuestionId;
            $tr->locale        = $this->blankToNull($row['locale'] ?? 'ja');
            $tr->question_text = $this->blankToNull($row['question_text'] ?? null);
            $tr->explanation   = $this->blankToNull($row['explanation']   ?? null);
            $tr->background    = $this->blankToNull($row['background']    ?? null);

            $tr->created_at = $this->parseDateTimeOrNull($row['created_at'] ?? null) ?? now();
            $tr->updated_at = $this->parseDateTimeOrNull($row['updated_at'] ?? null) ?? now();
            $tr->deleted_at = $this->parseDateTimeOrNull($row['deleted_at'] ?? null);

            $tr->save();
            $countInserted++;
        }

        $this->info("question_translations.csv import done. Inserted={$countInserted}, Skipped={$countSkipped}");
    }

    /**
     * CSV読み込み: fgetcsv() 形式で「カンマ区切り」「ダブルクォート囲み」
     */
    private function readCsvWithFgetcsv(string $storagePath): array
    {
        $content = Storage::get($storagePath);
        $fp = fopen('php://memory', 'r+');
        fwrite($fp, $content);
        rewind($fp);

        // ヘッダー
        $header = fgetcsv($fp, 0, ',', '"');
        if (!$header) {
            fclose($fp);
            return [];
        }

        $rows = [];
        while (($line = fgetcsv($fp, 0, ',', '"')) !== false) {
            // カラム数合わない行は skip
            if (count($line) !== count($header)) {
                continue;
            }
            $rows[] = array_combine($header, $line);
        }
        fclose($fp);

        return $rows;
    }

    /**
     * CSVの level_id => levelsIdToJsonId => DB levels  where('json_id')
     */
    private function resolveLevelDbId(?string $oldLevelId): ?int
    {
        if (!$oldLevelId) {
            return null;
        }
        if (!isset($this->levelsIdToJsonId[$oldLevelId])) {
            return null;
        }
        $jsonId = $this->levelsIdToJsonId[$oldLevelId];

        return Level::where('json_id', $jsonId)->value('id') ?: null;
    }

    /**
     * CSVの grade_id => gradesIdToJsonId => DB grades  where('json_id')
     */
    private function resolveGradeDbId(?string $oldGradeId): ?int
    {
        if (!$oldGradeId) {
            return null;
        }
        if (!isset($this->gradesIdToJsonId[$oldGradeId])) {
            return null;
        }
        $jsonId = $this->gradesIdToJsonId[$oldGradeId];

        return Grade::where('json_id', $jsonId)->value('id') ?: null;
    }

    /**
     * CSVの difficulty_id => difficultiesIdToJsonId => DB difficulties  where('json_id')
     */
    private function resolveDifficultyDbId(?string $oldDiffId): ?int
    {
        if (!$oldDiffId) {
            return null;
        }
        if (!isset($this->difficultiesIdToJsonId[$oldDiffId])) {
            return null;
        }
        $jsonId = $this->difficultiesIdToJsonId[$oldDiffId];

        return Difficulty::where('json_id', $jsonId)->value('id') ?: null;
    }
}

```
