
以下の ,FILL_IN_MULTIPART の時に、QuestionJsonManageService　の　validateUserAnswer でバリデートエラーになる問題を修正

回答に、「あまり」という text があるが、attribute チェックに文字列のケースがないので、else にひっかかり強制エラーになる。
このケースでは、

１）文字列であること

２）その文字列が、input_components　で定義された文字列に含まれているか

を確認するように実装できますか？



$metaData

array:5 [ // app/Services/Utils/Question/QuestionJsonManageService.php:1234
"question_type" => 351
"question_text" => "次の ▢ にあてはまる数を答えなさい。"
"explanation" => "25を4で割ると、4×6=24で1つ余るので「6 あまり 1」となります。"
"question" => "25 ÷ 4 = ▢"
"input_format" => array:2 [
"input_components" => array:2 [
0 => array:2 [
"type" => "signed_number_pad"
"order" => 50
]
1 => array:3 [
"type" => "text"
"content" => array:2 [
"ja" => "あまり"
"en" => "remainder"
]
"order" => 100
]
]
"question_components" => array:1 [
0 => array:3 [
"type" => "text"
"content" => "25 ÷ 4 = ▢"
"order" => 50
]
]
]
]



$answerData

array:1 [ // app/Services/Utils/Question/QuestionJsonManageService.php:1233
"fields" => array:3 [
0 => array:3 [
"field_id" => "f_1"
"attribute" => "number"
"user_answer" => 2
]
1 => array:3 [
"field_id" => "f_2"
"attribute" => "text"
"user_answer" => "あまり"
]
2 => array:3 [
"field_id" => "f_3"
"attribute" => "number"
"user_answer" => 3
]
]
]


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


ーーー
<?php

namespace App\Services\Utils\Question;

use App\Enums\EvaluationCheckerMethod;
use App\Enums\EvaluationMethod;
use App\Enums\FieldType;
use App\Enums\QuestionMetadataInputFormatFieldAttribute;
use App\Enums\FillInOperator;
use App\Enums\QuestionStatus;
use App\Enums\QuestionType;
use App\Models\Question\Question;
use App\Models\Question\QuestionTranslation;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Str;
use Illuminate\Validation\ValidationException;

/**
 * Class QuestionJsonManageService
 *
 * 【概要】
 *   このサービスクラスは、問題データ（QuestionJSON）のバリデーションおよび
 *   DBへの登録（upsert）処理を行う共通ロジックを提供します。
 *   もともと ImportQuestionsFromGithub で行っていた JSON のバリデーション/DB登録を移管し、
 *   他の機能（LLM 生成問題の取り込みなど）でも共通利用できるようにしました。
 *
 * 【機能一覧】
 *   1) validateQuestionJson() ：QuestionJSON 全体をバリデーションする
 *   2) upsertQuestionByJson() ：バリデーション後に Question テーブルへ upsert する
 *   3) validateUserAnswer()   ：ユーザー回答データをバリデーションする（サンプル）
 *   4) localizeMetadata()     ：metadata をアプリケーションロケールに合わせて整形（サンプル）
 *   5) localizeLlmResult()    ：LLM応答をアプリケーションロケールに合わせて整形（サンプル）
 *
 *
 * 【備考】
 *   - validateQuestionJson() では、トップレベルルールの他に、アフターコールバックで
 *     level_id / grade_id / difficulty_id / skills / evaluation_spec / metadata など
 *     詳細チェックを行っています。
 *   - upsertQuestionByJson() 内でバリデーション通過後、Question/QuestionTranslation/Skills の
 *     更新を行うことで、GitHubインポートでも LLM生成問題取り込みでも同一メソッドを利用できます。
 */
class QuestionJsonManageService
{
    /**
     * アプリがサポートする言語一覧
     */
    public const LANGUAGES = ['ja', 'en'];


    /**
     * question_components.type でサポートしている属性
     */
    public const VALID_COMPONENT_TYPES = ['text','image','movie','input_field','newline','options'];

    /**
     * question_components のうち content(多言語オブジェクト) が必須な type
     */
    private const COMPONENT_TYPES_REQUIRE_CONTENT = ['text','image','movie','options'];

    /**
     * FILL_IN_OPERATOR のときに必須となる input_components の type/attribute で使える定数
     *  (例: "text","image","movie","blank")
     */
    private const VALID_INPUT_COMPONENTS_TYPES = ['text','image','movie','blank'];

    /**
     * --------------------------------------------------------------------------------
     * QuestionJSON 全体をバリデーション
     * --------------------------------------------------------------------------------
     * 【目的】
     *   1) トップレベルの必須項目（id, order, metadata など）をチェック
     *   2) after コールバックで DB存在チェック (level/grade/difficulty/skills)、
     *      evaluation_spec や metadata構造を追加検証
     *   3) バリデーションに通過しなければ ValidationException を投げる
     *
     * 【戻り値】
     *   - 検証が通ったあとの配列を返す ( Laravel の validate() と同じ )
     *   - 失敗時は例外を投げる
     */
    public function validateQuestionJson(array $json): array
    {
        // トップレベルのルール
        $validator = \Validator::make($json, $this->getTopLevelRules(), $this->messages());

        // after コールバックでさらに詳細検証
        $validator->after(function ($v) use ($json) {
            $this->validateLevelGradeDifficulty($v, $json);
            $this->validateSkills($v, $json);
            $this->validateEvaluationSpec($v, $json);
            $this->validateMetadata($v, $json);  // ← ここで question_type, input_format, etc. の追加検証
        });

        // 通過した場合は結果を返す
        return $validator->validate();
    }

    /**
     * --------------------------------------------------------------------------------
     * QuestionJSON をデータベースへ upsert
     * --------------------------------------------------------------------------------
     * 【目的】
     *   - まず validateQuestionJson() を実施し、問題がなければ Question / QuestionTranslation /
     *     question_skill ピボット等を更新 (upsert) する。
     *   - GitHubインポート処理や LLM 生成問題の保存などに共通利用できる。
     *
     * 【パラメータ】
     *   @param array $questionJson
     *     - 取り込みたい問題 JSON
     *
     * 【戻り値】
     *   @return bool
     *     - true なら登録成功、 false ならバリデーションエラー等でスキップ扱い
     */
    public function upsertQuestionByJson(array $questionJson): bool
    {
        // バリデーション
        try {
            $this->validateQuestionJson($questionJson);
        } catch (\Illuminate\Validation\ValidationException $ve) {
            // バリデーション失敗 → false（呼び出し元でスキップなどの対応）
            return false;
        }

        // JSON 内の必須 key チェック
        $jsonId = $questionJson['id'] ?? null;
        if (!$jsonId) {
            return false;
        }

        $metadata = $questionJson['metadata'] ?? [];
        if (empty($metadata)) {
            // metadata が空なら取り込み不可
            return false;
        }

        // Question の既存 or 新規を検索
        /** @var Question|null $question */
        $question = Question::where('json_id', $jsonId)->first();

        $jsonOrder    = $questionJson['order'] ?? 9999;
        $version      = $questionJson['version'] ?? '0.0.1';
        $rawStatus    = $questionJson['status'] ?? null;
        $statusValue  = $rawStatus
            ? $this->parseQuestionStatus($rawStatus)
            : QuestionStatus::DRAFT->value;

        // level/grade/difficulty
        $levelUuid      = $this->findLevelUuid($questionJson['level_id'] ?? null);
        $gradeUuid      = $this->findGradeUuid($questionJson['grade_id'] ?? null);
        $difficultyUuid = $this->findDifficultyUuid($questionJson['difficulty_id'] ?? null);

        // 新規 or 既存
        if (!$question) {
            // 新規
            $question = new Question();
            $question->uuid      = (string) Str::uuid();
            $question->json_id = $jsonId;

            // order 確定
            $maxOrder = Question::max('order');
            if ($maxOrder === null) {
                $maxOrder = 0;
            }
            $finalOrder = max($maxOrder + 1, $jsonOrder);

            while (Question::where('order', $finalOrder)->exists()) {
                $finalOrder++;
            }
            $question->order = $finalOrder;
        } else {
            // 既存 → order再調整
            $this->reorderQuestion($question->id, $jsonOrder);
        }

        // 基本項目セット
        $question->level_id      = $levelUuid;
        $question->grade_id      = $gradeUuid;
        $question->difficulty_id = $difficultyUuid;
        $question->version       = $version;
        $question->status        = $statusValue;

        // metadata
        $question->metadata = json_encode($metadata, JSON_UNESCAPED_UNICODE);
        $rawQuestionType    = $metadata['question_type'] ?? null;
        $questionTypeValue  = $this->parseQuestionType($rawQuestionType);
        $question->question_type = $questionTypeValue;

        // evaluation_spec
        if (isset($questionJson['evaluation_spec'])) {
            $eval = $questionJson['evaluation_spec'];
            // eval method
            $question->evaluation_method = isset($eval['evaluation_method'])
                ? EvaluationMethod::fromString($eval['evaluation_method'])
                : null;

            // checker_method
            $question->checker_method = isset($eval['checker_method'])
                ? EvaluationCheckerMethod::fromString($eval['checker_method'])
                : null;

            // llm_prompt_file_name
            $question->llm_evaluation_prompt_file_name = $eval['llm_prompt_file_name'] ?? null;

            // response_format
            $question->evaluation_response_format = isset($eval['response_format'])
                ? json_encode($eval['response_format'], JSON_UNESCAPED_UNICODE)
                : null;
        }

        // generated_by_llm
        if (array_key_exists('generated_by_llm', $questionJson)) {
            $question->generated_by_llm = (bool)$questionJson['generated_by_llm'];
        }

        // learning_requirements
        $this->applyLearningRequirements($question, $questionJson);

        // 保存
        $question->save();

        // question_translations
        $this->upsertQuestionTranslations($question, $questionJson);

        // skills
        $this->applySkillsPivot($question, $questionJson);

        return true;
    }

    /**
     * --------------------------------------------------------------------------------
     * reorderQuestion
     * --------------------------------------------------------------------------------
     * 既存 question の order を再設定（衝突回避しながら更新）
     */
    private function reorderQuestion(string $questionId, int $targetOrder)
    {
        $question = Question::find($questionId);
        if (!$question) {
            return;
        }
        while (Question::where('order', $targetOrder)
            ->where('id','!=',$questionId)
            ->exists()) {
            $targetOrder++;
        }
        $question->order = $targetOrder;
        $question->save();
    }

    /**
     * --------------------------------------------------------------------------------
     * applySkillsPivot
     * --------------------------------------------------------------------------------
     * skills 配列を DBの question_skill ピボットに反映
     */
    private function applySkillsPivot(Question $question, array $questionJson)
    {
        $skillsFromJson = $questionJson['skills'] ?? [];
        $newSkillDbIds  = [];
        $skillIndexMap  = [];

        foreach ($skillsFromJson as $index => $sk) {
            $skillJsonId = $sk['skill_id'] ?? null;
            if (!$skillJsonId) {
                continue;
            }
            $skillDbId = DB::table('skills')
                ->where('json_id', $skillJsonId)
                ->value('id');
            if (!$skillDbId) {
                continue;
            }
            $newSkillDbIds[] = $skillDbId;
            $skillIndexMap[$skillDbId] = $index + 1;
        }

        $existingSkillDbIds = DB::table('question_skill')
            ->where('question_id', $question->id)
            ->pluck('skill_id')
            ->toArray();

        $skillIdsToRemove = array_diff($existingSkillDbIds, $newSkillDbIds);
        if (!empty($skillIdsToRemove)) {
            DB::table('question_skill')
                ->where('question_id', $question->id)
                ->whereIn('skill_id', $skillIdsToRemove)
                ->delete();
        }

        foreach ($newSkillDbIds as $skillDbId) {
            $pivotRow = DB::table('question_skill')
                ->where('question_id', $question->id)
                ->where('skill_id', $skillDbId)
                ->first();
            if ($pivotRow) {
                DB::table('question_skill')
                    ->where('id', $pivotRow->id)
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
     * --------------------------------------------------------------------------------
     * upsertQuestionTranslations
     * --------------------------------------------------------------------------------
     * question_translations に多言語のquestion_text等を格納
     */
    private function upsertQuestionTranslations(Question $question, array $questionJson)
    {
        $locales = self::LANGUAGES;
        $meta = $questionJson['metadata'] ?? [];

        foreach ($locales as $locale) {
            $qText = $meta['question_text'][$locale] ?? null;
            $qExp  = $meta['explanation'][$locale]   ?? null;
            $qBack = $meta['background'][$locale]    ?? null;

            // いずれか存在すれば upsert
            if ($qText !== null || $qExp !== null) {
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
     * --------------------------------------------------------------------------------
     * applyLearningRequirements
     * --------------------------------------------------------------------------------
     * learning_requirements (配列) を Question に格納 (改行区切り)
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

        // 改行結合
        $question->learning_subject             = implode("\n", $subjects);
        $question->learning_no                  = !empty($nos) ? (int)$nos[0] : null;
        $question->learning_requirement         = implode("\n", $requirements);
        $question->learning_required_competency = implode("\n", $requiredCompetencies);
        $question->learning_background          = implode("\n", $backgrounds);
        $question->learning_category            = implode("\n", $categories);
        $question->learning_grade_level         = implode("\n", $gradeLevels);
        $question->learning_url                 = implode("\n", $urls);
    }

    //==================================================
    // バリデーション関連
    //==================================================

    /**
     * トップレベルのバリデーションルール
     */
    private function getTopLevelRules(): array
    {
        return [
            // 基本情報
            'order'         => ['required','integer'],
            'id'            => ['required','string'],
            'level_id'      => ['required','string'],
            'grade_id'      => ['required','string'],
            'difficulty_id' => ['required','string'],
            'version'       => ['required','regex:/^\d+\.\d+\.\d+$/'],
            'status'        => ['required','string'],
            'generated_by_llm' => ['required','boolean'],
            'created_at'    => ['required','date_format:Y-m-d H:i:s'],
            'updated_at'    => ['required','date_format:Y-m-d H:i:s'],

            // skills
            'skills'        => ['required','array'],

            // learning_requirements
            'learning_requirements' => ['required','array'],
            'learning_requirements.*.learning_subject'            => ['required','string'],
            'learning_requirements.*.learning_no'                 => ['required','integer'],
            'learning_requirements.*.learning_requirement'        => ['required','string'],
            'learning_requirements.*.learning_required_competency'=> ['required','string'],
            'learning_requirements.*.learning_background'         => ['required','string'],
            'learning_requirements.*.learning_category'           => ['required','string'],
            'learning_requirements.*.learning_grade_level'        => ['required','string'],
            'learning_requirements.*.learning_url'                => ['sometimes','url'],

            // evaluation_spec
            'evaluation_spec'                   => ['required','array'],
            'evaluation_spec.evaluation_method' => ['required','string'],

            // metadata
            'metadata'                          => ['required','array'],
            'metadata.question_type'            => ['required'],

            // question_text / explanation / background / question => 多言語
            'metadata.question_text'            => ['required','array'],
            'metadata.question_text.ja'         => ['required','string'],
            'metadata.question_text.en'         => ['required','string'],
            'metadata.explanation'              => ['required','array'],
            'metadata.explanation.ja'           => ['required','string'],
            'metadata.explanation.en'           => ['required','string'],
            'metadata.background'               => ['required','array'],
            'metadata.background.ja'            => ['required','string'],
            'metadata.background.en'            => ['required','string'],
            'metadata.question'                 => ['required','array'],
            'metadata.question.ja'              => ['required','string'],
            'metadata.question.en'              => ['required','string'],

            // input_format
            'metadata.input_format'             => ['required','array'],
            'metadata.input_format.fields'      => ['required','array'],
            'metadata.input_format.question_components' => ['required','array'],
        ];
    }

    /**
     * バリデーションエラーメッセージ（日本語）
     */
    private function messages(): array
    {
        return [
            'required'    => ':attribute は必須項目です。',
            'integer'     => ':attribute は数値を指定してください。',
            'string'      => ':attribute は文字列である必要があります。',
            'boolean'     => ':attribute は true/false を指定してください。',
            'date_format' => ':attribute は :format 形式で指定してください。',
            'regex'       => ':attribute の形式が不正です。(例: 1.0.0)',
            'array'       => ':attribute は配列である必要があります。',
            'in'          => ':attribute に不正な値が指定されました( :values )。',
            'url'         => ':attribute は有効なURL形式で指定してください。',
        ];
    }

    /**
     * level_id, grade_id, difficulty_id が DB 上に存在するか確認
     */
    private function validateLevelGradeDifficulty($validator, array $json)
    {
        $levelId = $json['level_id'] ?? null;
        $gradeId = $json['grade_id'] ?? null;
        $diffId  = $json['difficulty_id'] ?? null;

        if ($levelId) {
            $exists = DB::table('levels')->where('json_id', $levelId)->exists();
            if (!$exists) {
                $validator->errors()->add('level_id',
                    "指定された level_id='{$levelId}' はDBに存在しません。"
                );
            }
        }
        if ($gradeId) {
            $exists = DB::table('grades')->where('json_id', $gradeId)->exists();
            if (!$exists) {
                $validator->errors()->add('grade_id',
                    "指定された grade_id='{$gradeId}' はDBに存在しません。"
                );
            }
        }
        if ($diffId) {
            $exists = DB::table('difficulties')->where('json_id', $diffId)->exists();
            if (!$exists) {
                $validator->errors()->add('difficulty_id',
                    "指定された difficulty_id='{$diffId}' はDBに存在しません。"
                );
            }
        }
    }

    /**
     * skills 配列の内容を DBの skills テーブルと照合
     */
    private function validateSkills($validator, array $json)
    {
        $skills = $json['skills'] ?? [];
        if (!is_array($skills)) {
            return;
        }
        foreach ($skills as $idx => $sk) {
            $sid   = $sk['skill_id'] ?? null;
            $sname = $sk['name']     ?? null;
            if (!$sid || !$sname) {
                continue;
            }
            $row = DB::table('skills')->where('json_id', $sid)->first();
            if (!$row) {
                $validator->errors()->add("skills.{$idx}.skill_id",
                    "skill_id='{$sid}' はDBに存在しません。"
                );
                continue;
            }
            if ($row->display_name !== $sname) {
                $validator->errors()->add("skills.{$idx}.name",
                    "skill_id='{$sid}' の display_name と name='{$sname}' が一致しません。"
                );
            }
        }
    }

    /**
     * evaluation_spec の詳細チェック
     * - evaluation_method (CODE/LLM)
     * - checker_method (CODE時必須)
     * - llm_prompt_file_name (LLM時必須)
     * - response_format, fields 等
     *
     * - metadata.question_type が CLASSIFY_THE_OPTIONS の場合、
     *   evaluation_method=CODE のときは checker_method を CHECK_BY_UNORDERED_ARRAY_ELEMENTS_MATCH に限定する
     */
    private function validateEvaluationSpec($validator, array $json)
    {
        $eval = $json['evaluation_spec'] ?? [];
        $methodRaw = $eval['evaluation_method'] ?? null;
        if (!$methodRaw) {
            return;
        }
        // CODE / LLM
        try {
            $method = EvaluationMethod::fromString($methodRaw);
        } catch (\InvalidArgumentException) {
            $validator->errors()->add('evaluation_spec.evaluation_method',
                "evaluation_method='{$methodRaw}' は無効です (CODE/LLM)。"
            );
            return;
        }

        // ここで question_type を取得し、後でチェックに利用
        $qtRaw = $json['metadata']['question_type'] ?? null;
        $questionTypeValue = null;
        try {
            $questionTypeValue = QuestionType::fromString((string)$qtRaw)->value;
        } catch (\InvalidArgumentException) {
            // 既に metadata.question_type でエラーになるはずなので ここでは何もしない
        }

        // CODE
        if ($method === EvaluationMethod::CODE) {
            // checker_method 必須
            if (empty($eval['checker_method']) || !is_string($eval['checker_method'])) {
                $validator->errors()->add('evaluation_spec.checker_method',
                    "evaluation_method=CODE のため checker_method が必須です。"
                );
            } else {
                try {
                    EvaluationCheckerMethod::fromString($eval['checker_method']);
                } catch (\InvalidArgumentException) {
                    $validator->errors()->add('evaluation_spec.checker_method',
                        "checker_method='{$eval['checker_method']}' は未定義です。"
                    );
                }
            }
            // response_format 必須
            if (!isset($eval['response_format']) || !is_array($eval['response_format'])) {
                $validator->errors()->add('evaluation_spec.response_format',
                    "evaluation_method=CODE のため response_format が必須です。"
                );
                return;
            }
            // CODE時でもfieldsがあれば検証するが必須ではない想定
            if (isset($eval['response_format']['fields']) && is_array($eval['response_format']['fields'])) {
                $this->validateResponseFormatFields($validator, $eval['response_format']['fields'], $json);
            }

            // CLASSIFY_THE_OPTIONS の場合、checker_method を限定
            if ($questionTypeValue === QuestionType::CLASSIFY_THE_OPTIONS->value) {
                $actualChecker = $eval['checker_method'] ?? null;
                if ($actualChecker !== EvaluationCheckerMethod::CHECK_BY_UNORDERED_ARRAY_ELEMENTS_MATCH->label()) {
                    $validator->errors()->add(
                        'evaluation_spec.checker_method',
                        "CLASSIFY_THE_OPTIONS の場合、evaluation_method=CODE では checker_method='".EvaluationCheckerMethod::CHECK_BY_UNORDERED_ARRAY_ELEMENTS_MATCH->label()."' が必須です。"
                    );
                }
            }

            // [ADD for ORDER_THE_OPTIONS start]
            // ORDER_THE_OPTIONS の場合、checker_method を CHECK_BY_ARRAY_ELEMENTS_MATCH_IN_ORDER に限定
            if ($questionTypeValue === QuestionType::ORDER_THE_OPTIONS->value) {
                $actualChecker = $eval['checker_method'] ?? null;
                if ($actualChecker !== EvaluationCheckerMethod::CHECK_BY_ARRAY_ELEMENTS_MATCH_IN_ORDER->label()) {
                    $validator->errors()->add(
                        'evaluation_spec.checker_method',
                        "ORDER_THE_OPTIONS の場合、evaluation_method=CODE では checker_method='".EvaluationCheckerMethod::CHECK_BY_ARRAY_ELEMENTS_MATCH_IN_ORDER->label()."' が必須です。"
                    );
                }
            }
            // [ADD for ORDER_THE_OPTIONS end]
        }
        // LLM
        else {
            // llm_prompt_file_name 必須
            if (!isset($eval['llm_prompt_file_name'])) {
                $validator->errors()->add('evaluation_spec.llm_prompt_file_name',
                    "evaluation_method=LLM のため llm_prompt_file_name(文字列) が必須です。"
                );
            } else {
                $path = resource_path("prompts/evaluation/{$eval['llm_prompt_file_name']}.txt");
                if (!file_exists($path)) {
                    $validator->errors()->add('evaluation_spec.llm_prompt_file_name',
                        "LLM promptファイルが見つかりません。(path={$path})"
                    );
                }
            }
            // response_format 必須
            if (!isset($eval['response_format']) || !is_array($eval['response_format'])) {
                $validator->errors()->add('evaluation_spec.response_format',
                    "evaluation_method=LLM のため response_format が必須です。"
                );
                return;
            }
            // LLM時 => fields 配列必須
            if (!isset($eval['response_format']['fields']) || !is_array($eval['response_format']['fields'])) {
                $validator->errors()->add('evaluation_spec.response_format.fields',
                    "evaluation_method=LLM のため fields(配列) が必須です。"
                );
                return;
            }
            $this->validateResponseFormatFields($validator, $eval['response_format']['fields'], $json);
        }
    }

    /**
     * 新規追加メソッド
     * evaluation_spec.response_format.fields.* の collect_answer を
     * question_type に合わせてバリデーションする
     */
    private function validateResponseFormatFields($validator, array $fieldsArr, array $json)
    {
        // question_type を参照する
        $meta       = $json['metadata'] ?? [];
        $rawQt      = $meta['question_type'] ?? '';
        $questionTp = null;
        try {
            $questionTp = QuestionType::fromString((string)$rawQt)->value;
        } catch (\InvalidArgumentException) {
            // 不明なら何もしない
        }

        foreach ($fieldsArr as $i => $field) {
            // field_id
            if (empty($field['field_id']) || !is_string($field['field_id'])) {
                $validator->errors()->add("evaluation_spec.response_format.fields.{$i}.field_id",
                    "field_id は必須の文字列です。"
                );
            }
            // user_answer
            if (empty($field['user_answer']) || !is_string($field['user_answer'])) {
                $validator->errors()->add("evaluation_spec.response_format.fields.{$i}.user_answer",
                    "user_answer は必須の文字列です。"
                );
            } else {
                // CLASSIFY_THE_OPTIONS の場合は 'sequence' を必須
                if ($questionTp === QuestionType::CLASSIFY_THE_OPTIONS->value) {
                    if ($field['user_answer'] !== FieldType::SEQUENCE->value) {
                        $validator->errors()->add(
                            "evaluation_spec.response_format.fields.{$i}.user_answer",
                            "CLASSIFY_THE_OPTIONS の場合 user_answer は 'sequence' である必要があります。"
                        );
                    }
                }
                // [ADD for ORDER_THE_OPTIONS start]
                // ORDER_THE_OPTIONS の場合は 'sequence' を必須
                if ($questionTp === QuestionType::ORDER_THE_OPTIONS->value) {
                    if ($field['user_answer'] !== FieldType::SEQUENCE->value) {
                        $validator->errors()->add(
                            "evaluation_spec.response_format.fields.{$i}.user_answer",
                            "ORDER_THE_OPTIONS の場合 user_answer は 'sequence' である必要があります。"
                        );
                    }
                }
                // [ADD for ORDER_THE_OPTIONS end]
            }

            // is_correct
            if (!isset($field['is_correct']) || $field['is_correct'] !== 'boolean') {
                $validator->errors()->add("evaluation_spec.response_format.fields.{$i}.is_correct",
                    "is_correct は 'boolean' の文字列である必要があります。"
                );
            }
            // collect_answer => 多言語オブジェクト
            if (!isset($field['collect_answer']) || !is_array($field['collect_answer'])) {
                $validator->errors()->add("evaluation_spec.response_format.fields.{$i}.collect_answer",
                    "collect_answer は多言語オブジェクトが必須です。"
                );
            } else {
                // LANGUAGES に対応しているか
                foreach (self::LANGUAGES as $lang) {
                    if (!array_key_exists($lang, $field['collect_answer'])) {
                        $validator->errors()->add(
                            "evaluation_spec.response_format.fields.{$i}.collect_answer",
                            "collect_answer に言語キー'{$lang}'がありません。"
                        );
                    } else {
                        // question_type によって値の型チェック
                        $val = $field['collect_answer'][$lang];
                        if ($questionTp === QuestionType::FILL_IN_THE_BLANK->value) {
                            // "number" の場合は数値（例示）
                            if (!is_numeric($val)) {
                                $validator->errors()->add(
                                    "evaluation_spec.response_format.fields.{$i}.collect_answer.{$lang}",
                                    "FILL_IN_THE_BLANK の場合 collect_answer.{$lang} は数値である必要があります。"
                                );
                            }
                        }
                        elseif ($questionTp === QuestionType::FILL_IN_OPERATOR->value) {
                            // FillInOperator enumに存在するか
                            try {
                                FillInOperator::from($val); // '>', '<', '=' 等
                            } catch (\ValueError $ve) {
                                $validator->errors()->add(
                                    "evaluation_spec.response_format.fields.{$i}.collect_answer.{$lang}",
                                    "FILL_IN_OPERATOR の場合 collect_answer.{$lang} は演算子（>,<,=）のいずれかが必要です。"
                                );
                            }
                        }
                        // ORDER_THE_OPTIONS や CLASSIFY_THE_OPTIONS の場合は、ここでは特に型指定なし
                        //（実際の正解値は配列などの形式を想定しうるが、ここでは多言語オブジェクトで文字列保存）
                    }
                }
            }

            // field_explanation => 多言語オブジェクト
            if (!isset($field['field_explanation']) || !is_array($field['field_explanation'])) {
                $validator->errors()->add("evaluation_spec.response_format.fields.{$i}.field_explanation",
                    "field_explanation は多言語オブジェクトが必須です。"
                );
            } else {
                foreach (self::LANGUAGES as $lang) {
                    $txt = $field['field_explanation'][$lang] ?? null;
                    if (!is_string($txt) || trim($txt) === '') {
                        $validator->errors()->add(
                            "evaluation_spec.response_format.fields.{$i}.field_explanation.{$lang}",
                            "field_explanation.{$lang} は空文字不可の文字列が必須です。"
                        );
                    }
                }
            }
        }
    }

    /**
     * metadata.question_type / input_format などを検証
     */
    private function validateMetadata($validator, array $json)
    {
        $meta = $json['metadata'] ?? [];
        $qtRaw = $meta['question_type'] ?? null;

        // question_type が有効な Enumか
        $questionTypeValue = null;
        try {
            $questionTypeValue = QuestionType::fromString((string)$qtRaw)->value;
        } catch (\InvalidArgumentException) {
            $validator->errors()->add('metadata.question_type',
                "question_type='{$qtRaw}' は未定義です。"
            );
        }

        // fields
        $fieldsArr = $meta['input_format']['fields'] ?? [];
        if (is_array($fieldsArr)) {
            $fieldIds = [];
            foreach ($fieldsArr as $idx => $f) {
                // field_id='f_数字'形式
                if (empty($f['field_id']) || !preg_match('/^f_\d+$/', $f['field_id'])) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.field_id",
                        "field_id='f_数字'形式が必須です。"
                    );
                }
                // 重複チェック
                if (in_array($f['field_id'] ?? '', $fieldIds, true)) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.field_id",
                        "field_id='{$f['field_id']}' が重複しています。"
                    );
                }
                $fieldIds[] = $f['field_id'] ?? '';

                // attribute
                $attrRaw = $f['attribute'] ?? null;
                if (!$attrRaw) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.attribute",
                        "attribute が指定されていません。"
                    );
                } else {
                    // enumチェック
                    try {
                        QuestionMetadataInputFormatFieldAttribute::fromString($attrRaw);
                    } catch (\InvalidArgumentException) {
                        $validator->errors()->add("metadata.input_format.fields.{$idx}.attribute",
                            "attribute='{$attrRaw}' はサポート対象外です。"
                        );
                    }
                }

                // user_answer => enumチェック
                if (!isset($f['user_answer']) || !is_string($f['user_answer'])) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.user_answer",
                        "user_answer は必須の文字列です。"
                    );
                } else {
                    try {
                        FieldType::from($f['user_answer']);
                    } catch (\ValueError $ve) {
                        // Enumに無い文字列が渡された場合
                        $validator->errors()->add("metadata.input_format.fields.{$idx}.user_answer",
                            "user_answer='{$f['user_answer']}' は 'number','string','operator','sequence' のいずれかのみ有効です。"
                        );
                    }
                }

                // collect_answer は基本的に metadata では非公開(禁止)
                if (array_key_exists('collect_answer', $f)) {
                    $validator->errors()->add("metadata.input_format.fields.{$idx}.collect_answer",
                        "collect_answer は指定できません。"
                    );
                }
            }
        }

        // question_components
        $comps = $meta['input_format']['question_components'] ?? [];
        if (is_array($comps)) {
            $inputFieldCount = 0;
            $orders = [];
            foreach ($comps as $cidx => $comp) {
                $ctype = $comp['type'] ?? '';
                if (!in_array($ctype, self::VALID_COMPONENT_TYPES, true)) {
                    $validator->errors()->add("metadata.input_format.question_components.{$cidx}.type",
                        "不正なコンポーネントtype='{$ctype}'です。"
                    );
                }
                // input_field => field_id必須 & カウント
                if ($ctype === 'input_field') {
                    $inputFieldCount++;
                    if (!isset($comp['field_id'])) {
                        $validator->errors()->add("metadata.input_format.question_components.{$cidx}.field_id",
                            "type=input_field の場合 field_id が必須です。"
                        );
                    }
                }
                // content が必須な type
                elseif (in_array($ctype, self::COMPONENT_TYPES_REQUIRE_CONTENT, true)) {
                    if (empty($comp['content']) || !is_array($comp['content'])) {
                        $validator->errors()->add(
                            "metadata.input_format.question_components.{$cidx}.content",
                            "type='{$ctype}' の場合 content(オブジェクト) が必須です。"
                        );
                    } else {
                        // 多言語キーがあるか
                        foreach (self::LANGUAGES as $lang) {
                            if (!isset($comp['content'][$lang])) {
                                $validator->errors()->add(
                                    "metadata.input_format.question_components.{$cidx}.content.{$lang}",
                                    "content.{$lang} がありません。"
                                );
                            }
                        }
                    }
                }
                // order => 数値 & 重複チェック
                if (!isset($comp['order']) || !is_numeric($comp['order'])) {
                    $validator->errors()->add("metadata.input_format.question_components.{$cidx}.order",
                        "order(数値) は必須です。"
                    );
                } else {
                    if (in_array($comp['order'], $orders, true)) {
                        $validator->errors()->add("metadata.input_format.question_components.{$cidx}.order",
                            "order='{$comp['order']}' が重複しています。"
                        );
                    }
                    $orders[] = $comp['order'];
                }
            }
            // input_field数 と fields数 が一致するか
            $fieldsCount = count($fieldsArr);
            if ($inputFieldCount !== $fieldsCount) {
                $validator->errors()->add("metadata.input_format.question_components",
                    "input_field要素数({$inputFieldCount}) と fields数({$fieldsCount}) が一致しません。"
                );
            }
        }

        // --------------------------------------------------------------------------------
        // ここから FILL_IN_OPERATOR / CLASSIFY_THE_OPTIONS 向けバリデーション
        // --------------------------------------------------------------------------------
        // FILL_IN_OPERATOR なら input_components 必須
        if ($questionTypeValue === QuestionType::FILL_IN_OPERATOR->value) {
            $icKey = 'metadata.input_format.input_components';
            if (!isset($meta['input_format']['input_components'])) {
                $validator->errors()->add($icKey,
                    "FILL_IN_OPERATOR のため input_components が必須です。"
                );
                return;
            }
            $inputComps = $meta['input_format']['input_components'];
            if (!is_array($inputComps)) {
                $validator->errors()->add($icKey,
                    "FILL_IN_OPERATOR のため input_components は配列である必要があります。"
                );
                return;
            }

            $icOrders = [];
            foreach ($inputComps as $idx => $ic) {
                $typeVal = $ic['type'] ?? null;
                if (!$typeVal) {
                    $validator->errors()->add("{$icKey}.{$idx}.type",
                        "type が必須です。"
                    );
                } elseif (!in_array($typeVal, self::VALID_INPUT_COMPONENTS_TYPES, true)) {
                    $validator->errors()->add("{$icKey}.{$idx}.type",
                        "不正な type='{$typeVal}' です。"
                    );
                }

                if (empty($ic['content']) || !is_array($ic['content'])) {
                    $validator->errors()->add("{$icKey}.{$idx}.content",
                        "content(オブジェクト) が必須です。"
                    );
                } else {
                    foreach (self::LANGUAGES as $lang) {
                        if (!isset($ic['content'][$lang])) {
                            $validator->errors()->add("{$icKey}.{$idx}.content.{$lang}",
                                "content.{$lang} がありません。"
                            );
                        } else {
                            // FillInOperator の値(> < =)になっているか
                            $val = $ic['content'][$lang];
                            try {
                                FillInOperator::from($val);
                            } catch (\ValueError) {
                                $validator->errors()->add("{$icKey}.{$idx}.content.{$lang}",
                                    "不正な演算子 '{$val}' です。(>,<,= のいずれか)"
                                );
                            }
                        }
                    }
                }
                // order
                if (!isset($ic['order']) || !is_numeric($ic['order'])) {
                    $validator->errors()->add("{$icKey}.{$idx}.order",
                        "order(数値) は必須です。"
                    );
                } else {
                    if (in_array($ic['order'], $icOrders, true)) {
                        $validator->errors()->add("{$icKey}.{$idx}.order",
                            "order='{$ic['order']}' が重複しています。"
                        );
                    }
                    $icOrders[] = $ic['order'];
                }
            }
        }

        // --------------------------------------------------------------------------------
        // CLASSIFY_THE_OPTIONS 向けのバリデーション
        // --------------------------------------------------------------------------------
        if ($questionTypeValue === QuestionType::CLASSIFY_THE_OPTIONS->value) {
            // 1) question がカンマ区切りか（言語ごとにチェック）
            foreach (self::LANGUAGES as $lang) {
                $qStr = $meta['question'][$lang] ?? '';
                if (!str_contains($qStr, ',')) {
                    $validator->errors()->add("metadata.question.{$lang}",
                        "CLASSIFY_THE_OPTIONS の場合、question.{$lang} はカンマ区切りの文字列が必須です。"
                    );
                }
            }

            // 2) input_format.input_components が必須
            $icKey = 'metadata.input_format.input_components';
            if (!isset($meta['input_format']['input_components']) || !is_array($meta['input_format']['input_components'])) {
                $validator->errors()->add($icKey,
                    "CLASSIFY_THE_OPTIONS のため input_components (配列) が必須です。"
                );
            } else {
                foreach (self::LANGUAGES as $lang) {
                    $qStr = $meta['question'][$lang] ?? '';
                    $arrQ = array_map('trim', explode(',', $qStr));
                    $arrQ = array_filter($arrQ, fn($v) => $v !== '');
                    $countQ = count($arrQ);

                    $icComps = $meta['input_format']['input_components'];
                    $icContents = [];
                    foreach ($icComps as $ic) {
                        $icContent = $ic['content'][$lang] ?? null;
                        if ($icContent !== null) {
                            $icContents[] = trim($icContent);
                        }
                    }
                    if ($countQ !== count($icContents)) {
                        $validator->errors()->add(
                            $icKey,
                            "CLASSIFY_THE_OPTIONS の場合、question.{$lang} と同じ数の input_components が必要です。"
                        );
                    } else {
                        foreach ($arrQ as $token) {
                            if (!in_array($token, $icContents, true)) {
                                $validator->errors()->add(
                                    $icKey,
                                    "CLASSIFY_THE_OPTIONS の場合、question.{$lang} の要素 '{$token}' が input_components にありません。"
                                );
                            }
                        }
                    }
                }
            }

            // 3) metadata.input_format.fields[*].user_answer と attribute
            if (is_array($fieldsArr)) {
                foreach ($fieldsArr as $idx => $f) {
                    if (($f['user_answer'] ?? '') !== FieldType::SEQUENCE->value) {
                        $validator->errors()->add("metadata.input_format.fields.{$idx}.user_answer",
                            "CLASSIFY_THE_OPTIONS の場合、user_answer は 'sequence' である必要があります。"
                        );
                    }
                    if (($f['attribute'] ?? '') !== QuestionMetadataInputFormatFieldAttribute::ARRAY->value) {
                        $validator->errors()->add("metadata.input_format.fields.{$idx}.attribute",
                            "CLASSIFY_THE_OPTIONS の場合、attribute は 'array' である必要があります。"
                        );
                    }
                }
            }
        }

        // [ADD for ORDER_THE_OPTIONS start]
        // --------------------------------------------------------------------------------
        // ORDER_THE_OPTIONS 向けのバリデーション
        // --------------------------------------------------------------------------------
        if ($questionTypeValue === QuestionType::ORDER_THE_OPTIONS->value) {
            // 1) question がカンマ区切りか（言語ごとにチェック）
            foreach (self::LANGUAGES as $lang) {
                $qStr = $meta['question'][$lang] ?? '';
                if (!str_contains($qStr, ',')) {
                    $validator->errors()->add("metadata.question.{$lang}",
                        "ORDER_THE_OPTIONS の場合、question.{$lang} はカンマ区切りの文字列が必須です。"
                    );
                }
            }

            // 2) input_format.input_components が必須
            $icKey = 'metadata.input_format.input_components';
            if (!isset($meta['input_format']['input_components']) || !is_array($meta['input_format']['input_components'])) {
                $validator->errors()->add($icKey,
                    "ORDER_THE_OPTIONS のため input_components (配列) が必須です。"
                );
            } else {
                // question内のカンマ区切り要素がすべて input_components に含まれているか、数が一致するか
                foreach (self::LANGUAGES as $lang) {
                    $qStr = $meta['question'][$lang] ?? '';
                    $arrQ = array_map('trim', explode(',', $qStr));
                    $arrQ = array_filter($arrQ, fn($v) => $v !== '');
                    $countQ = count($arrQ);

                    $icComps = $meta['input_format']['input_components'];
                    $icContents = [];
                    foreach ($icComps as $ic) {
                        $icContent = $ic['content'][$lang] ?? null;
                        if ($icContent !== null) {
                            $icContents[] = trim($icContent);
                        }
                    }
                    if ($countQ !== count($icContents)) {
                        $validator->errors()->add(
                            $icKey,
                            "ORDER_THE_OPTIONS の場合、question.{$lang} と同じ数の input_components が必要です。"
                        );
                    } else {
                        foreach ($arrQ as $token) {
                            if (!in_array($token, $icContents, true)) {
                                $validator->errors()->add(
                                    $icKey,
                                    "ORDER_THE_OPTIONS の場合、question.{$lang} の要素 '{$token}' が input_components にありません。"
                                );
                            }
                        }
                    }
                }
            }

            // 3) metadata.input_format.fields[*].user_answer と attribute
            if (is_array($fieldsArr)) {
                foreach ($fieldsArr as $idx => $f) {
                    if (($f['user_answer'] ?? '') !== FieldType::SEQUENCE->value) {
                        $validator->errors()->add("metadata.input_format.fields.{$idx}.user_answer",
                            "ORDER_THE_OPTIONS の場合、user_answer は 'sequence' である必要があります。"
                        );
                    }
                    if (($f['attribute'] ?? '') !== QuestionMetadataInputFormatFieldAttribute::ARRAY->value) {
                        $validator->errors()->add("metadata.input_format.fields.{$idx}.attribute",
                            "ORDER_THE_OPTIONS の場合、attribute は 'array' である必要があります。"
                        );
                    }
                }
            }
        }
        // [ADD for ORDER_THE_OPTIONS end]
    }

    /**
     * 文字列→QuestionType enum の int値に変換（未定義なら CALCULATION）
     */
    private function parseQuestionType(?string $typeString): int
    {
        if (!$typeString) {
            return QuestionType::CALCULATION->value;
        }
        try {
            return QuestionType::fromString($typeString)->value;
        } catch (\InvalidArgumentException) {
            return QuestionType::CALCULATION->value;
        }
    }

    /**
     * 文字列→QuestionStatus enum の int値に変換（未定義なら DRAFT）
     */
    private function parseQuestionStatus(string $statusString): int
    {
        try {
            return QuestionStatus::fromString($statusString)->value;
        } catch (\InvalidArgumentException) {
            return QuestionStatus::DRAFT->value;
        }
    }

    /**
     * level_id → levelsテーブル(json_id)検索 → UUID
     */
    private function findLevelUuid(?string $levelJsonId): ?string
    {
        if (!$levelJsonId) {
            return null;
        }
        return DB::table('levels')
            ->where('json_id', $levelJsonId)
            ->value('id') ?: null;
    }

    /**
     * grade_id → gradesテーブル(json_id)検索 → UUID
     */
    private function findGradeUuid(?string $gradeJsonId): ?string
    {
        if (!$gradeJsonId) {
            return null;
        }
        return DB::table('grades')
            ->where('json_id', $gradeJsonId)
            ->value('id') ?: null;
    }

    /**
     * difficulty_id → difficultiesテーブル(json_id)検索 → UUID
     */
    private function findDifficultyUuid(?string $difficultyJsonId): ?string
    {
        if (!$difficultyJsonId) {
            return null;
        }
        return DB::table('difficulties')
            ->where('json_id', $difficultyJsonId)
            ->value('id') ?: null;
    }

    /**
     * --------------------------------------------------------------------------------
     * ユーザー回答のバリデーション
     * --------------------------------------------------------------------------------
     * 【目的】
     *   - question_type='FILL_IN_THE_BLANK' を例にした回答検証
     *   - metadata.input_format との整合性 (fields数/attribute/field_id) をチェック
     *   - ここでは一部簡易実装のみ示している。 (実運用では他 question_type にも対応)
     */
    public function validateUserAnswer(array $answerData, array $metadata): void
    {
        // metadata は localizeMetadata() で、ローカライズされている前提
        // question_type は数値になっている
        $qt = $metadata['question_type'] ?? null;

        $inFmt = $metadata['input_format'] ?? [];
        if (!is_array($inFmt)) {
            throw ValidationException::withMessages([
                'metadata.input_format' => [__('errors.api.answer.mismatch_answer_format')]
            ]);
        }

        $metaFields = $inFmt['fields'] ?? [];
        if (!is_array($metaFields)) {
            throw ValidationException::withMessages([
                'metadata.input_format.fields' => [__('errors.api.answer.mismatch_answer_format')]
            ]);
        }

        $answerFields = $answerData['fields'] ?? null;
        if (!is_array($answerFields)) {
            throw ValidationException::withMessages([
                'answerData.fields' => [__('errors.api.answer.mismatch_answer_format')]
            ]);
        }


        // FILL_IN_THE_BLANK / FILL_IN_OPERATOR のみ
        // MetaData の数と回答数があっている。
        if ($qt == QuestionType::FILL_IN_THE_BLANK->value
            || $qt == QuestionType::FILL_IN_OPERATOR->value) {
            if (count($metaFields) !== count($answerFields)) {
                throw ValidationException::withMessages([
                    'fields.count' => [__('errors.api.answer.mismatch_answer_format')]
                ]);
            }
        }


        // CLASSIFY_THE_OPTIONS 対応
        //    userAnswer fields の数が metadata fields の数と一致しているか
        if ($qt == QuestionType::CLASSIFY_THE_OPTIONS->value) {
            if (count($metaFields) !== count($answerFields)) {
                throw ValidationException::withMessages([
                    'fields.count' => ["CLASSIFY_THE_OPTIONS の場合、ユーザーの fields数が問題定義と一致しません。"]
                ]);
            }
        }

        // ★ ORDER_THE_OPTIONS 対応
        //    userAnswer fields の数が metadata fields の数と一致しているか
        if ($qt == QuestionType::ORDER_THE_OPTIONS->value) {
            if (count($metaFields) !== count($answerFields)) {
                throw ValidationException::withMessages([
                    'fields.count' => ["ORDER_THE_OPTIONS の場合、ユーザーの fields数が問題定義と一致しません。"]
                ]);
            }
        }


        $metaFieldMap = [];
        foreach ($metaFields as $mf) {
            if (!empty($mf['field_id'])) {
                $metaFieldMap[$mf['field_id']] = $mf;
            }
        }

        foreach ($answerFields as $idx => $af) {
            $fid = $af['field_id'] ?? '';
            if (!$fid) {
                throw ValidationException::withMessages([
                    "fields.{$idx}.field_id" => [__('errors.api.answer.mismatch_answer_format')]
                ]);
            }

            $attr = $af['attribute'] ?? null;
            $uAns = $af['user_answer'] ?? null;

            if (!$attr) {
                throw ValidationException::withMessages([
                    "fields.{$idx}.attribute" => [__('errors.api.answer.mismatch_answer_format')]
                ]);
            }

            // FILL_IN_THE_BLANK の場合にmetadataに存在しない field_id はNG
            if ($qt == QuestionType::FILL_IN_THE_BLANK->value && !isset($metaFieldMap[$fid])) {
                throw ValidationException::withMessages([
                    "fields.{$idx}.field_id" => [__('errors.api.answer.mismatch_answer_format')]
                ]);
            }

            // 既存チェック: number / operator
            if ($attr === FieldType::NUMBER->value) {
                if (!is_numeric($uAns)) {
                    throw ValidationException::withMessages([
                        "fields.{$idx}.user_answer" => [__('errors.api.answer.mismatch_answer_format_to_number')]
                    ]);
                }
            }
            else if ($attr === QuestionMetadataInputFormatFieldAttribute::OPERATOR->value) {
                try {
                    FillInOperator::from($uAns);
                } catch (\ValueError) {
                    throw ValidationException::withMessages([
                        "fields.{$idx}.user_answer" => [__('errors.api.answer.mismatch_answer_format_operator')],
                    ]);
                }

            }
            // ★ CLASSIFY_THE_OPTIONS 用ロジック: attribute='array' なら、user_answer が JSON配列文字列かチェック
            else if ($qt == QuestionType::CLASSIFY_THE_OPTIONS->value
                && $attr === QuestionMetadataInputFormatFieldAttribute::ARRAY->value) {
                // user_answer が JSON配列文字列かどうかを簡易チェック
                $decoded = @json_decode($uAns, true);
                if (!is_array($decoded)) {
                    throw ValidationException::withMessages([
                        "fields.{$idx}.user_answer" => [__('errors.api.answer.mismatch_answer_format_classify_the_options')],
                    ]);
                }
            }

            // ★ ORDER_THE_OPTIONS 用ロジック: attribute='array' なら、user_answer が JSON配列文字列かチェック
            else if ($qt == QuestionType::ORDER_THE_OPTIONS->value
                && $attr === QuestionMetadataInputFormatFieldAttribute::ARRAY->value) {
                // user_answer が JSON配列文字列かどうかを簡易チェック
                $decoded = @json_decode($uAns, true);
                if (!is_array($decoded)) {
                    throw ValidationException::withMessages([
                        "fields.{$idx}.user_answer" => [__('errors.api.answer.mismatch_answer_format_order_the_options')],
                    ]);
                }
            }
            else {
                throw ValidationException::withMessages([
                    "fields.{$idx}.attribute" => [__('errors.api.answer.mismatch_answer_format')]
                ]);
            }
        }
    }

    /**
     * --------------------------------------------------------------------------------
     * localizeMetadata
     * --------------------------------------------------------------------------------
     * metadata を現在のアプリケーションロケールに合わせて整形（サンプル）
     */
    public function localizeMetadata(array $metadata): array
    {
        $locale = \App::getLocale();
        if (isset($metadata['question_text']) && is_array($metadata['question_text'])) {
            $metadata['question_text'] = $metadata['question_text'][$locale] ?? '';
        }
        if (isset($metadata['explanation']) && is_array($metadata['explanation'])) {
            $metadata['explanation']   = $metadata['explanation'][$locale] ?? '';
        }
        // background を表示しない
        if (isset($metadata['background']) && is_array($metadata['background'])) {
            unset($metadata['background']);
        }
        // metadata の　question_type は表示しない
        if (isset($metadata['question_type'])) {
            try {
                $metadata['question_type'] = QuestionType::fromString($metadata['question_type'])->value;
            } catch (\Exception) {
                $metadata['question_type'] = null;
            }
        }
        if (isset($metadata['question']) && is_array($metadata['question'])) {
            $metadata['question'] = $metadata['question'][$locale] ?? '';
        }
        if (isset($metadata['input_format']['question_components'])
            && is_array($metadata['input_format']['question_components'])) {
            foreach ($metadata['input_format']['question_components'] as $i => $comp) {
                if (($comp['type'] ?? '') === 'text'
                    && isset($comp['content'])
                    && is_array($comp['content'])) {
                    $metadata['input_format']['question_components'][$i]['content']
                        = $comp['content'][$locale] ?? '';
                }
            }
        }
        return $metadata;
    }

    /**
     * --------------------------------------------------------------------------------
     * localizeLlmResult
     * --------------------------------------------------------------------------------
     * LLM応答を現在のロケールに合わせて整形
     */
    public function localizeLlmResult(array $llmResult): array
    {
        $locale = \App::getLocale();

        $mapped = [];
        $rawIsCorrect = data_get($llmResult, 'is_correct', false);
        $mapped['is_correct'] = ($rawIsCorrect === true || $rawIsCorrect === "true") ? "true" : "false";

        $mapped['score'] = data_get($llmResult, 'score', 0);

        $qtArr = data_get($llmResult, 'question_text', []);
        $mapped['question_text'] = is_array($qtArr)
            ? data_get($qtArr, $locale, '')
            : '';

        $expArr = data_get($llmResult, 'explanation', []);
        $mapped['explanation'] = is_array($expArr)
            ? data_get($expArr, $locale, '')
            : '';

        $qArr = data_get($llmResult, 'question', []);
        $mapped['question'] = is_array($qArr)
            ? data_get($qArr, $locale, '')
            : '';

        $mapped['fields'] = [];
        $fieldsArr = data_get($llmResult, 'fields', []);
        if (is_array($fieldsArr)) {
            foreach ($fieldsArr as $f) {
                $mf = [];
                $mf['field_id']     = (string) data_get($f, 'field_id', '');
                $mf['user_answer']  = (string) data_get($f, 'user_answer', '');
                $rCorrect           = data_get($f, 'is_correct', false);
                $mf['is_correct']   = ($rCorrect === true || $rCorrect === "true") ? "true" : "false";
                $mf['collect_answer'] = data_get($f, 'collect_answer', '');

                $fieldExpArr = data_get($f, 'field_explanation', []);
                $mf['field_explanation'] = is_array($fieldExpArr)
                    ? data_get($fieldExpArr, $locale, '')
                    : '';

                $mapped['fields'][] = $mf;
            }
        }
        return $mapped;
    }
}
