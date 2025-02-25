OpenAI のAPIを利用して、JSONデータを生成するスクリプトを生成してください。

# 条件
1
TypeScript で実装してください

2
コードのディレクトリは ./src

3 
プロンプトはtxtファイルで保存され ./prompt にあり、スクリプトを実行するコマンドの引数でパスを指定出来るようにしてください。（コードにデフォルト値を設定してください）

4
出力フォーマットはtxtファイルで保存され ./format にあり、スクリプトを実行するコマンドの引数でパスを指定出来るようにしてください。（コードにデフォルト値を設定してください）

5
生成された JSONは、./output に出力してください。オプションで、/output以下のディレクトリをコマンドの引数で指定できるようにしてください。

6
OPENAI API Key は、.env に設定します。

# 参考コード（PHP）
/**
* LLM評価用のAPI呼び出し
*/
private function callLLMEvaluationApi(array $answerData, Question $question): array
{
$responseFormat = $question->llm_evaluation_response_format;
$learningRequirement = $question->learning_requirement_json;
$userAnswerJson = json_encode($answerData, JSON_UNESCAPED_UNICODE | JSON_PRETTY_PRINT);
$promptId = $question->llm_evaluation_prompt_number;

        $prompt = $this->buildPrompt($promptId, $userAnswerJson, $responseFormat, $learningRequirement);
        $model = 'gpt-4o-mini';
//        $model = 'o3-mini';
$response = Http::withToken(config('services.openai.api_key'))
->post('https://api.openai.com/v1/chat/completions', [
'model' => $model,
'messages' => [
[
'role' => 'system',
'content' => 'You are a helpful assistant. Return your answer in valid JSON format only, with no extra text.'
],
[
'role' => 'assistant',
'content' => $responseFormat
],
[
'role' => 'user',
'content' => $prompt
]
],
'response_format' => [
'type' => 'json_object'
],
'temperature' => 0.0,
]);

        $responseBody = $response->json();

        $rawContent = data_get($responseBody, 'choices.0.message.content');

        $parsed = json_decode($rawContent, true);

        return $parsed ?? [];
    }
