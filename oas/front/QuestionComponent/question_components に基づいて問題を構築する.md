ありがとう。

input_format.question_components は、UI上の問題を構成する各パーツを表しています。
現在の、QuestionRender.vue は、question_text の、▢ 　を文字列変換で input フィールドに置き換える処理になっているのでこれは間違っています。

現状存在している type ですと、
type="text" は、<p>{{ content }}</p>
type="image" は、<img>
type="movie" は、<video>
type="newline" は、改行
type="input_field"は、field_id と対応する input_format.fields を探索して、一致する field の attribute に指定されている値で input を作成
例）input_format.question_components.type="input_field" で、field_id : 1 で、input_format.fields の field_id : 1 の attribute : "number" だったら、
<input type="number" class="input-answer-box" autocomplete="off" required> を作成

が仕様です。
他のコンポーネントでも使用するアプリケーション全体での共通処理なので、
composable に切り出す必要がありそう。


# 実装方針
生成また修正変更するコードはそのコードだけは必ず省略せずに必ず全てアウトプットすること。

仕様はソースコードにまとめてコメントとして残すこと

TypeScript を厳格に使用すること。型が存在しないなら必ず型を作成してください。型を作成する時は /src/types 以下に別ファイルで切り出してください


# ディレクトリ構成
front-app
├─ .vscode
├─ docs
├─ e2e
├─ node_modules
├─ public
└─ src
├─ api
│  ├─ core
│  ├─ endpoints
│  └─ index.ts
├─ assets
│  ├─ images
│  ├─ base.scss
│  ├─ main.scss
│  └─ variables.scss
├─ components
├─ composables
├─ locales
├─ pages
├─ router
├─ services
├─ store
├─ types
│  ├─ api
│  ├─ common
│  ├─ enums
│  ├─ models
│  └─ payloads
├─ App.vue
├─ i18n.ts
├─ main.ts
├─ auto-imports.d.ts
├─ components.d.ts
├─ env.d.ts
├─ eslint.config.ts
├─ index.html
├─ package.json
├─ package-lock.json
├─ playwright.config.ts
├─ README.md
├─ tsconfig.app.json
├─ tsconfig.json
├─ tsconfig.node.json
└─ tsconfig.vitest.json


## ディレクトリ構成（説明）
- src/
    - assets/
        - 静的リソース（画像やフォントなど）を配置
    - components/
        - 再利用可能な Vue コンポーネントを配置
    - composables/
        - UI 周りのロジックを中心とした再利用可能なロジックをまとめる。API コールなどのService層の呼び出しや、Storeへのデータの保存や取り出しは　composable から行う。
        - 例）フォームバリデーション、表示用ユーティリティなど
    - constants/
        - 共通で使用する定数を配置
    - layouts/
        - アプリ全体で使うレイアウトコンポーネントを配置（ヘッダー、フッターなど）
    - pages/
        - 画面（ページ）コンポーネントを配置
    - router/
        - ルーティング設定を配置（Vue Router の設定ファイルなど）
    - services/
        - API コールなどのビジネスロジックや外部サービスとの連携を担当
    - store/
        - Pinia などグローバルステート管理の設定ファイルやストアを配置
    - styles/
        - グローバルな CSS/Sass ファイルを配置
    - types/
        - TypeScript の型定義（インターフェース、型エイリアス）などを配置
    - utils/
        - 汎用的なユーティリティ関数を配置
- public/
    - ビルド前に直接参照される静的ファイル（favicon, manifest など）を配置
- tests/
    - 単体テストや統合テストなどを配置


---    console.log("metadata", props.questionMetadata); の中身
{ "question_type": "FILL_IN_THE_BLANK", "question": "8 + 3 = 11, 11 + 3 = ▢, 14 + 3 = ▢, 17 + 3 = ▢", "question_text": "つぎの ▢ にあてはまる数を答えなさい。", "explanation": "これは「かけ算＝同じ数をくり返し足す」ということを、足し算のステップで体感できる問題です。8から始めて、毎回3を足していくと、どのように数が増えていくかを確認しましょう。", "input_format": { "type": "fixed", "fields": [ { "field_id": "f_1", "attribute": "number", "user_answer": "number" }, { "field_id": "f_2", "attribute": "number", "user_answer": "number" }, { "field_id": "f_3", "attribute": "number", "user_answer": "number" } ], "question_components": [ { "type": "text", "content": "8 + 3 = 11", "order": 10 }, { "type": "newline", "order": 15 }, { "type": "text", "content": "11 + 3 = ", "order": 16 }, { "type": "input_field", "field_id": "f_1", "order": 20 }, { "type": "newline", "order": 25 }, { "type": "text", "content": "14 + 3 = ", "order": 30 }, { "type": "input_field", "field_id": "f_2", "order": 40 }, { "type": "newline", "order": 45 }, { "type": "text", "content": "17 + 3 = ", "order": 50 }, { "type": "input_field", "field_id": "f_3", "order": 60 } ] } }

ーーー
```typescript

import type QuestionType from "@/types/enums/questionType";
import type { AnswerDataRequest, AnswerDataResponse, AnswerFieldRequest } from "./answer";

export interface MeatadataOption {
    option_id: string;
    name_ja: string;
    price: number;
}

export interface BeginQuestionResponse {
  user_question_id: string;
  question_id: string;
  status: number;
  answer_data: AnswerDataResponse | AnswerDataRequest | null;
  answered_at: string | null;
  question: Question;
  question_text: string;
  explanation: string;
}

export interface Question {
  id: string;
  question_text: string;
  explanation: string;
  question_data?: QuestionMetadataType;
  metadata?: QuestionMetadataType;
  version: string;
  question_type: QuestionType;
}

export interface QuestionMetadataType {
  question: string;
  question_text: string;
  input_format: InputFormat;
  evaluationMethod: number;
  options?: MeatadataOption[];
}

export interface InputFormat {
  type: string;
  fields: AnswerFieldRequest[];
  question_components: QuestionComponent[];
}

export interface QuestionComponent {
  type: "text" | "blank";
  content?: string;
  field_id?: string;
}

```

ーーー
<!--/src/components/atoms/QuestionRender/QuestionRender.vue-->
<template>
  <div id="question-render-section" v-html="questionHTML"></div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import type { QuestionMetadataType } from "@/types/models/question/question.ts";
const props = defineProps<{ questionMetadata: QuestionMetadataType }>();
const questionHTML = ref("");


const createHTMLFromJson = (metadata:QuestionMetadataType) => {
  //full json question
  let question = metadata.question
  //input section
  const searchValue = '▢'
  //input detais
  const inputFields = metadata.input_format.fields

  const inputBoxes = []

  for (let i = 0; i <= question.length - searchValue.length; i++) {
    if (question.substring(i, i + searchValue.length) === searchValue) {
      const inputPosition = [i,i+searchValue.length]
      inputBoxes.push(inputPosition)
    }

  let inputIndex = 0;


  question = question.replace(/▢/g, () => {
    if (inputIndex < inputFields.length) {
      const field = inputFields[inputIndex];
      inputIndex++;
      // TODO number を期待しているが、Array など type に損座しない値が入ってきた時に対応する
      return `<input type="${type}" id="${field.field_id}" name="${field.field_id}" class="input-answer-box" autocomplete="off" required>`;
    }
    return searchValue;
  });
}

questionHTML.value = question
}

onMounted(() => {
  console.log("metadata", props.questionMetadata);
  createHTMLFromJson(props.questionMetadata);
});


</script>

<style>
.input-answer-box{
  width: 35px;
  height: 35px;
  border-width: 1px;
  aspect-ratio: 1;
  border-style: solid;
  padding: 4px;
  font-size: 14px;
  text-align: center;
  outline: none;
}

.input-answer-box:focus{
  border-color: #009FE8;
}

.pass{
  border: 2px solid green;;
}

.wrong {
  border: 2px solid red;
  background-color: #ffe6e6;
  animation: shake 0.3s ease-in-out;
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  20%, 80% { transform: translateX(-5px); }
  40%, 60% { transform: translateX(5px); }
}

</style>

