ありがとう。

以下の、EvaluationCheckService に、
CHECK_BY_ARRAY_ELEMENTS_MATCH_IN_ORDER
の判定を追加したい。

具体的には、
CHECK_BY_UNORDERED_ARRAY_ELEMENTS_MATCH　
では、
*     1) collectAnswer の要素は全て userAnswer 配列に含まれているか
*     2) userAnswer に collectAnswer に存在しない値が入っていないか
の２点について判定していますが、
CHECK_BY_ARRAY_ELEMENTS_MATCH_IN_ORDER
では、これに加えて、
collectAnswer の要素全てと userAnswer 配列の要素全て同じ順番になっているか
を判定してほしい。
要素の一致（順番問わず）だけじゃなくて、順番まで一致しているのかを確認するようにしてほしい


ーー　userAnswer 例
```json
{
  "fields": [
    {
      "field_id": "f_1",
      "attribute": "array",
      "user_answer": "[14, 18, 20, 50]"
    },
    {
      "field_id": "f_2",
      "attribute": "array",
      "user_answer": "[15, 18, 21, 27, 45]"
    },
    {
      "field_id": "f_3",
      "attribute": "array",
      "user_answer": "[15, 20, 25, 35, 45, 50]"
    },
    {
      "field_id": "f_4",
      "attribute": "array",
      "user_answer": "[14, 21, 35]"
    },
    {
      "field_id": "f_5",
      "attribute": "array",
      "user_answer": "[18, 27, 45]"
    }
  ]
}
```

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

```php
<?php

namespace App\Services\Utils\Evaluation;

use Illuminate\Support\Facades\App;
use Illuminate\Validation\ValidationException;

/**
 * Class EvaluationCheckService
 *
 * 【概要】
 * - evaluation_spec.checker_method で指定されたメソッド名を呼び出し、CODE の正誤判定を行うためのサービス
 * - question_type=FILL_IN_THE_BLANK をはじめ、将来的に様々なチェックロジックを増やす想定
 *
 */
class EvaluationCheckService
{
    /**
     * 穴埋め問題(FILL_IN_THE_BLANK) の「完全一致」チェック
     *
     * @param array $userAnswerData   ユーザー回答( { fields: [...] } 形式 )
     * @param array $evaluationResponseFormat   userQuestion.evaluation_response_format (JSONをdecodeした配列)
     * @return array レスポンス( evaluation_spec.response_format と同じ構造 )
     *
     * 【仕様】
     * - evaluation_response_format.fields[*].collect_answer と userAnswerData.fields[*].user_answer を照合
     * - 一致したら is_correct=true、そうでなければfalse
     * - 全フィールドが正解なら response_format.is_correct=true,そうでなければfalse
     * - score = 100 * (正解数 / fields総数)
     * - question_text, explanation, question は、App::getLocale() をキーとして evaluation_response_format の該当言語を取得
     * - field_explanation も同様に言語ごと
     */
    public function checkByExactMatch(array $userAnswerData, array $evaluationResponseFormat): array
    {
        // TODO: 多言語化（現在はサンプルで固定メッセージを入れています）

        // 1) fieldsリスト
        $respFields = $evaluationResponseFormat['fields'] ?? [];
        if (!is_array($respFields)) {
            // evaluation_response_format が不正
            throw ValidationException::withMessages([
                'response_format.fields' => "evaluation_response_format の fields が配列ではありません。"
            ]);
        }

        // ユーザー回答 fields
        $answerFields = $userAnswerData['fields'] ?? [];
        if (!is_array($answerFields)) {
            throw ValidationException::withMessages([
                'answer_data.fields' => "ユーザー回答の fields が配列ではありません。"
            ]);
        }

        // field_id => userAnswer
        $userFieldMap = [];
        foreach ($answerFields as $af) {
            $fid = $af['field_id'] ?? null;
            if ($fid) {
                $userFieldMap[$fid] = $af['user_answer'] ?? null;
            }
        }

        $correctCount = 0;
        $totalCount = count($respFields);

        // 現在のアプリケーションロケール(ja/enなど)
        $locale = App::getLocale();

        // 2) fields をループし、is_correctを判定
        $newFields = [];
        foreach ($respFields as $f) {
            $fid = $f['field_id'] ?? null;
            if (!$fid) {
                // 不正
                continue;
            }

            // collect_answer は多言語オブジェクトになったため、現在の言語を取得
            $collectAnswerMultiLang = $f['collect_answer'] ?? [];
            $collectValue = null;
            if (is_array($collectAnswerMultiLang)) {
                // 該当ロケールの正解値
                $collectValue = $collectAnswerMultiLang[$locale] ?? null;
            }

            // user_answer
            $userAns = $userFieldMap[$fid] ?? null;

            // 同じなら正解
            $isCorrect = ($userAns == $collectValue);  // 厳密型ではなく "==" でよいか要件次第
            // field_explanation
            $fieldExp = $f['field_explanation'] ?? [];
            if (!is_array($fieldExp)) {
                $fieldExp = [];
            }

            // フィールドごとの explanation も "ja"=>"...","en"=>"..." が入ってる前提
            // もしUI表示で全言語返すなら「field_explanationそのまま返却」でもOK
            // ここでは fieldsごとに "field_explanation.{locale}" のみ返す例
            $explanationForLocale = $fieldExp[$locale] ?? '';

            if ($isCorrect) {
                $correctCount++;
            }

            // 新しい fields配列を作成
            $newFields[] = [
                'field_id'          => $fid,
                'user_answer'       => $userAns,
                'is_correct'        => $isCorrect ? true : false, // "boolean"文字列ではだめ
                'collect_answer'    => $collectValue,  // collect_answerはそのまま(ユーザーには隠す場合は外す)
                'field_explanation' => $explanationForLocale
            ];
        }

        // 全体 is_correct
        $allCorrect = ($correctCount === $totalCount);
        $scoreEach = 100 / max($totalCount, 1);
        $scoreVal = $scoreEach * $correctCount;

        // question_text / explanation / question は LLM形式なら "text"固定だが
        // CODE形式なら userQuestion.metadata.question_text などを流用、と要件次第
        // 今回は "localeごとにevaluation_response_format" と書いているので
        // -> "ja":"text","en":"text" があったはず
        // -> そのままロケールに合わせて文字列を返す(あるいは "text"固定）
        // 例：evaluationResponseFormat["question_text"]["ja"] = "text"
        // => "text" → ここでは UI表示向けに question_text=... という文字列にするかどうかは要件次第

        $locale = App::getLocale();
        $qText = $evaluationResponseFormat['question_text'][$locale] ?? '';
        $expText = $evaluationResponseFormat['explanation'][$locale] ?? '';
        $qBody = $evaluationResponseFormat['question'][$locale] ?? '';

        // 最終的に evaluation_spec.response_format と同じ構造で返す
        //   is_correct, score, question_text, explanation, question, fields[*]
        //   ただし fields[*].is_correct は "boolean"(文字列) になっているので `'true'/'false'`
        return [
            'is_correct'   => $allCorrect ? true : false,
            'score'        => (string) $scoreVal,  // "number"文字列
            'question_text'=> $qText,
            'explanation'  => $expText,
            'question'     => $qBody,
            'fields'       => $newFields,
        ];
    }

    /**
     * CHECK_BY_UNORDERED_ARRAY_ELEMENTS_MATCH など、順不同の配列要素が一致しているかを判定する新規メソッド
     *
     * 【仕様】
     * - ユーザー回答データ (fields の中に array形式の値 "[10,12]" など) をJSONデコードして配列に変換
     * - evaluation_response_format.fields[*].collect_answer (多言語オブジェクト) を取り出し、locale別の配列に変換
     * - 判定:
     *     1) collectAnswer の要素は全て userAnswer 配列に含まれているか
     *     2) userAnswer に collectAnswer に存在しない値が入っていないか
     *   上記2条件を両方満たせば is_correct=true (順序は問わない)
     * - 全フィールドが正解なら is_correct=true, score = 100 * (正解数 / fields数)
     * - question_text, explanation, question は現在のロケールに対応した文字列を返す
     * - field_explanation も同様に多言語対応
     */
    public function checkByUnorderedArrayElementsMatch(array $userAnswerData, array $evaluationResponseFormat): array
    {
        // 1) fieldsリスト取得
        $respFields = $evaluationResponseFormat['fields'] ?? [];
        if (!is_array($respFields)) {
            throw ValidationException::withMessages([
                'response_format.fields' => "evaluation_response_format の fields が配列ではありません。"
            ]);
        }

        // ユーザー回答 fields
        $answerFields = $userAnswerData['fields'] ?? [];
        if (!is_array($answerFields)) {
            throw ValidationException::withMessages([
                'answer_data.fields' => "ユーザー回答の fields が配列ではありません。"
            ]);
        }

        // userの回答 (field_id => JSON文字列)
        $userFieldMap = [];
        foreach ($answerFields as $af) {
            $fid = $af['field_id'] ?? null;
            if ($fid !== null) {
                $userFieldMap[$fid] = $af['user_answer'] ?? null;
            }
        }

        $correctCount = 0;
        $totalCount   = count($respFields);

        // 現在のロケール
        $locale = App::getLocale();

        $newFields = [];
        foreach ($respFields as $f) {
            $fid = $f['field_id'] ?? null;
            if (!$fid) {
                continue;
            }
            // collect_answer 多言語から現在のロケール値を取得
            $collectAnswerMultiLang = $f['collect_answer'] ?? [];
            $collectRaw = null;
            if (is_array($collectAnswerMultiLang)) {
                $collectRaw = $collectAnswerMultiLang[$locale] ?? null;
            }

            // user_answer のJSON文字列を配列に変換
            $userRaw = $userFieldMap[$fid] ?? '[]';

            // field_explanation
            $fieldExp = $f['field_explanation'] ?? [];
            if (!is_array($fieldExp)) {
                $fieldExp = [];
            }
            $explanationForLocale = $fieldExp[$locale] ?? '';

            // JSONを配列にパース
            // 例: "[14,15,18]" => [14,15,18]
            $userArr = [];
            if (is_string($userRaw)) {
                $decoded = @json_decode($userRaw, true);
                if (is_array($decoded)) {
                    $userArr = $decoded;
                }
            }

            $collectArr = [];
            if (is_string($collectRaw)) {
                $decoded2 = @json_decode($collectRaw, true);
                if (is_array($decoded2)) {
                    $collectArr = $decoded2;
                }
            }

            // 順不同で要素が一致しているか (set equality)
            // 1) collectにあるものが全てユーザー回答に含まれているか
            // 2) ユーザー回答に collectに無い要素が含まれていないか
            // ここでは重複は考慮せず「要素集合」として扱う実装例
            $userUnique    = array_unique($userArr);
            $collectUnique = array_unique($collectArr);

            // 条件1) collectの要素すべてが userUnique に含まれている
            $allIncluded = !array_diff($collectUnique, $userUnique);
            // 条件2) userUniqueに collectUniqueに無い要素が存在しない
            $noExtra = !array_diff($userUnique, $collectUnique);

            $isCorrect = ($allIncluded && $noExtra);

            if ($isCorrect) {
                $correctCount++;
            }

            // field出力
            $newFields[] = [
                'field_id'          => $fid,
                'user_answer'       => $userRaw,         // JSON文字列
                'is_correct'        => $isCorrect ? true : false,
                'collect_answer'    => $collectRaw,      // JSON文字列
                'field_explanation' => $explanationForLocale,
            ];
        }

        // 全体正解
        $allCorrect = ($correctCount === $totalCount);
        $scoreEach  = 100 / max($totalCount, 1);
        $scoreVal   = $scoreEach * $correctCount;

        // question_text / explanation / question
        $qText = $evaluationResponseFormat['question_text'][$locale] ?? '';
        $expText = $evaluationResponseFormat['explanation'][$locale] ?? '';
        $qBody = $evaluationResponseFormat['question'][$locale] ?? '';

        return [
            'is_correct'   => $allCorrect ? true : false,
            'score'        => (string) $scoreVal,  // "number"文字列
            'question_text'=> $qText,
            'explanation'  => $expText,
            'question'     => $qBody,
            'fields'       => $newFields,
        ];
    }
}


```
