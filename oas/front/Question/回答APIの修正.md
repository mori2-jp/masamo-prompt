ありがとう。
以下の、

FillInTheBlank.vue の、handleSubmit　でユーザーの回答を送信しています。
ただし、送信しているデータは、
現在の Answer API へのリクエストデータ
になっており、これを
Answer API が期待している リクエストデータ
に変更したいです。

コードを修正してください


# 実装方針
生成また修正変更するコードはそのコードだけは必ず省略せずに必ず全てアウトプットすること。

仕様はソースコードにまとめてコメントとして残すこと

TypeScript を厳格に使用すること。型が存在しないなら必ず型を作成してください。

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


ーー 現在の Answer API へのリクエストデータ
```json
user_question_id: 7baf0078-785a-4af8-a380-04fd1acf7b5b
answer_data: {"question":"8 + 3 = 11, 11 + 3 = ▢, 14 + 3 = ▢, 17 + 3 = ▢","fields":[{"field_id":"f_1","user_answer":"111"},{"field_id":"f_2","user_answer":"111"},{"field_id":"f_3","user_answer":"111"}]}
```

-- Answer API が期待している リクエストデータ
```json
user_question_id: 7baf0078-785a-4af8-a380-04fd1acf7b5b
answer_data: {
	"fields": [
		{
			"field_id": "f_1",
			"attribute": "number",
			"user_answer": "42"
		},
		{
			"field_id": "f_2",
			"attribute": "number",
			"user_answer": "8"
		}
	]
}
```

---    console.log("metadata", props.questionMetadata); の中身
{ "question_type": "FILL_IN_THE_BLANK", "question": "8 + 3 = 11, 11 + 3 = ▢, 14 + 3 = ▢, 17 + 3 = ▢", "question_text": "つぎの ▢ にあてはまる数を答えなさい。", "explanation": "これは「かけ算＝同じ数をくり返し足す」ということを、足し算のステップで体感できる問題です。8から始めて、毎回3を足していくと、どのように数が増えていくかを確認しましょう。", "input_format": { "type": "fixed", "fields": [ { "field_id": "f_1", "attribute": "number", "user_answer": "number" }, { "field_id": "f_2", "attribute": "number", "user_answer": "number" }, { "field_id": "f_3", "attribute": "number", "user_answer": "number" } ], "question_components": [ { "type": "text", "content": "8 + 3 = 11", "order": 10 }, { "type": "newline", "order": 15 }, { "type": "text", "content": "11 + 3 = ", "order": 16 }, { "type": "input_field", "field_id": "f_1", "order": 20 }, { "type": "newline", "order": 25 }, { "type": "text", "content": "14 + 3 = ", "order": 30 }, { "type": "input_field", "field_id": "f_2", "order": 40 }, { "type": "newline", "order": 45 }, { "type": "text", "content": "17 + 3 = ", "order": 50 }, { "type": "input_field", "field_id": "f_3", "order": 60 } ] } }


---
```vue
<!--/src/components/organisms/Questions/FillInTheBlank.vue-->
<template>
  <div>
    <LoadingDialog :isLoading="isLoading" />

    <h1>{{ metadata?.question_text }}</h1>
    <div>
      <form @submit.prevent="handleSubmit">
        <v-col class="quetions-column">
          <QuestionRender
            :key="question?.question.id"
            :question-metadata="metadata as QuestionMetadataType"
          />
          <v-btn :disabled="isLoading" max-width="150" type="submit" class="submit-btn">
            {{ t('components.organisms.questions.fill_in_the_blank.submit') }}
          </v-btn>
        </v-col>
      </form>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, inject } from "vue";
import { useQuestionIndex, useQuestionIndexKey } from "@/composables/question/useQuestionIndex";
import { useFillInTheBlankQuestion } from "@/composables/question/useFillInTheBlankQuestion";
import QuestionRender from "@/components/atoms/QuestionRender/QuestionRender.vue";
import LoadingDialog from '@/components/molecules/LoadingDialog.vue';
import type { QuestionMetadataType } from "@/types/models/question/question";
import { useI18n } from "vue-i18n";

const { t } = useI18n();

const indexQuestion = inject(useQuestionIndexKey, () => useQuestionIndex(), true);
const { question, isLoading, storeQuestionAnswer } = indexQuestion;

const { checkInputField } = useFillInTheBlankQuestion();

const handleSubmit = async (e: Event) => {
  if (metadata.value && question.value) {
    const answerData = checkInputField(e, metadata.value, question.value);
    storeQuestionAnswer(answerData);
  } else {
    console.error("Metadata or question is null");
  }
};

const metadata = computed(() => question.value?.question.question_data || question.value?.question.metadata);

</script>

<style lang="scss" scoped>
@use "sass:map";

.quetions-column {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.submit-btn {
  background-color: map.get($custom-colors, 'blue-4');
  color: #fff;
}
</style>
```

```typescript
// /src/composables/question/useFillInTheBlankQuestion.ts
import type { AnswerFieldRequest, SubmitAnswerRequest } from "@/types/models/question/answer"
import type { BeginQuestionResponse, QuestionMetadataType } from "@/types/models/question/question"

export function useFillInTheBlankQuestion() {

  const checkInputField = (
    event: Event,
    metadata: QuestionMetadataType,
    question: BeginQuestionResponse
  ) => {
    removeClassNames()

    const formData = new FormData(event.target as HTMLFormElement)
    const questionAnswers = Object.fromEntries(
      Array.from(formData.entries()).map(([key, value]) => [key, value.toString()])
    )

    const results = checkCorrectAnswers(metadata.input_format.fields, questionAnswers)

    const answerData = generateAnswerData(results, question)
    updateInputStyles(results)

    return answerData
  }

  const removeClassNames = () => {
    const inputs = document.querySelectorAll('input')
    inputs.forEach((input) => input.classList.remove('pass', 'wrong'))
  }

  const updateInputStyles = (validationResults: { isCorrect: boolean; field_id: string }[]) => {
    validationResults.forEach(({ field_id, isCorrect }) => {
      const input = document.querySelector(`input[name="${field_id}"]`)
      if (input) {
        input.classList.remove('pass', 'wrong')
        if (isCorrect) {
          input.classList.add('pass')
        } else {
          input.classList.add('wrong')
        }
      }
    })
  }

  const checkCorrectAnswers = (
    questionAnswerFields: AnswerFieldRequest[],
    userAnswers: { [key: string]: string }
  ): (AnswerFieldRequest & { isCorrect: boolean })[] => {
    return questionAnswerFields.map(field => {
      const currentKey = field.field_id
      const userAnswer = userAnswers[currentKey] || ""
      const isCorrect = userAnswer.trim() !== ""
      return {
        field_id: field.field_id,
        user_answer: userAnswer,
        isCorrect
      }
    })
  }

  const generateAnswerData = (
    fields: (AnswerFieldRequest & { isCorrect: boolean })[],
    question: BeginQuestionResponse
  ): SubmitAnswerRequest => {
    const ansData = {
      user_question_id: question.user_question_id,
      answer_data: {
        question_text: question.question_text,
        explanation: question.explanation,
        question: question.question.question_data?.question ?? question.question.metadata?.question ?? null,
        fields: fields.map(field => ({
          field_id: field.field_id,
          user_answer: field.user_answer
        }))
      }
    }
    return ansData
  }

  return {
    checkInputField,
    checkCorrectAnswers,
  }
}

```

```typescript
import { QuestionService } from "@/services/question/questionService"
import { useQuestionStore } from "@/store/questionStore"
import { computed, defineAsyncComponent, ref } from "vue";
import { useRoute } from "vue-router";
import QuestionType from "@/types/enums/questionType";
import type { SubmitAnswerRequest } from "@/types/models/question/answer";

export const useQuestionIndexKey = Symbol('useQuestionIndex')
const questionService = new QuestionService()

// Dynamically import components for each problem type
const questionComponentMap: Record<QuestionType, ReturnType<typeof defineAsyncComponent>> = {
    [QuestionType.FILL_IN_THE_BLANK]: defineAsyncComponent(() => import('@/components/organisms/Questions/FillInTheBlank.vue')),
    [QuestionType.SCENARIO]: defineAsyncComponent(() => import('@/components/organisms/Questions/Scenario.vue')),
    [QuestionType.MULTIPLE_CHOICE]: defineAsyncComponent(() => import('@/components/organisms/Questions/Scenario.vue')),
    [QuestionType.SIMPLE_ARITHMETIC]: defineAsyncComponent(() => import('@/components/organisms/Questions/Scenario.vue')),
    [QuestionType.COMBINATION]: defineAsyncComponent(() => import('@/components/organisms/Questions/Scenario.vue')),
    [QuestionType.HELL]: defineAsyncComponent(() => import('@/components/organisms/Questions/Scenario.vue')),
    [QuestionType.CALCULATION]: defineAsyncComponent(() => import('@/components/organisms/Questions/Scenario.vue')),
    [QuestionType.DATA_INTERPRETATION]: defineAsyncComponent(() => import('@/components/organisms/Questions/Scenario.vue')),
    [QuestionType.DRILL]: defineAsyncComponent(() => import('@/components/organisms/Questions/Scenario.vue')),
    [QuestionType.STORY]: defineAsyncComponent(() => import('@/components/organisms/Questions/Scenario.vue')),
    [QuestionType.SIMULATION]: defineAsyncComponent(() => import('@/components/organisms/Questions/Scenario.vue')),
    [QuestionType.REASONING]: defineAsyncComponent(() => import('@/components/organisms/Questions/Scenario.vue')),
    [QuestionType.PROOF]: defineAsyncComponent(() => import('@/components/organisms/Questions/Scenario.vue')),
    [QuestionType.MULTI_STEP_COMPARATIVE]: defineAsyncComponent(() => import('@/components/organisms/Questions/Scenario.vue')),
    [QuestionType.LOGIC]: defineAsyncComponent(() => import('@/components/organisms/Questions/Scenario.vue')),
    [QuestionType.FUSION]: defineAsyncComponent(() => import('@/components/organisms/Questions/Scenario.vue')),
    [QuestionType.DEBUG]: defineAsyncComponent(() => import('@/components/organisms/Questions/Scenario.vue')),
    [QuestionType.FREE_TEXT]: defineAsyncComponent(() => import('@/components/organisms/Questions/Scenario.vue')),
    [QuestionType.INFERENCE]: defineAsyncComponent(() => import('@/components/organisms/Questions/Scenario.vue')),
};

export function useQuestionIndex() {
    const questionStore = useQuestionStore()
    const route = useRoute()
    const isInitialized = ref(false)
    const isLoading = ref(false)
    const questionId = route.query.question_id ? Number(route.query.question_id) : 1
    const question = computed(() => {
        return questionStore.question
    })
    const error = computed(() => {
        return questionStore.question
    })
    const answer = computed(() => {
        return questionStore.answer
    })
    const isCorrect = computed(() => {
        return questionStore.isCorrect
    })


    const fetchQuestion = async () => {
        isLoading.value = true
        const result = await questionService.fetchQuestion(questionId)
        if (result) {
            questionStore.setQuestion(result.data)
            isLoading.value = false
        }
    }

    const initialize = async ()  => {
        isInitialized.value = true
    }

    const targetQuestionComponentName = computed(() => {
        if (!question.value || !question.value.question.question_type) return null
        return questionComponentMap[question.value.question.question_type];
    })

    const storeQuestionAnswer = async (request: SubmitAnswerRequest) => {
        isLoading.value = true
        const res = await questionService.storeAnswer(request)
        if (res) {
            questionStore.setAnswer(res.data)
            isLoading.value = false
            // TODO Action after process next question
        }
    }

    const fetchNextQuestion = async (userQuestionId: string) => {
        isLoading.value = true
        const result = await questionService.fetchNextQuestion(userQuestionId)
        if (result) {
            questionStore.setQuestion(result.data)
            isLoading.value = false
        }
    }
    return {
        fetchQuestion,
        isInitialized,
        isLoading,
        initialize,
        fetchNextQuestion,
        question,
        answer,
        error,
        targetQuestionComponentName,
        storeQuestionAnswer,
        isCorrect,
        clearAnswer: questionStore.clearAnswer
    }
}

```

```typescript
import { questionApi } from '@/api'
import type { Either } from 'fp-ts/lib/Either'
import type { ApiErrorResponse } from '@/api/core'
import type { GetResponse } from '@/types/api/response'
// import type { Me } from '@/types/models/auth/me'
import type { BeginQuestionResponse } from '@/types/models/question/question'
import type { AnswerDataResponse, SubmitAnswerRequest } from '@/types/models/question/answer'

export class QuestionService {
  // ユーザー情報を取得
  // TODO 型は仮置き
  async fetchQuestion(questionId: number): Promise<GetResponse<BeginQuestionResponse> | null> {
    const result: Either<ApiErrorResponse, GetResponse<BeginQuestionResponse>> = await questionApi.fetchQuestion(questionId)

    if (result._tag === 'Left') {
      throw new Error(`Failed to get fetch question: ${result.left.type}`)
    }
    return result.right
  }

  async storeAnswer(request: SubmitAnswerRequest): Promise<GetResponse<AnswerDataResponse> | null> {
    const result: Either<ApiErrorResponse, GetResponse<AnswerDataResponse>> = await questionApi.storeAnswer(request)

    if (result._tag === 'Left') {
      throw new Error(`Failed to get store question: ${result.left.type}`)
    }
    return result.right
  }

  async fetchNextQuestion(userQuestionId: string): Promise<GetResponse<BeginQuestionResponse> | null> {
    const result: Either<ApiErrorResponse, GetResponse<BeginQuestionResponse>> = await questionApi.fetchNextQuestion(userQuestionId)

    if (result._tag === 'Left') {
      throw new Error(`Failed to get fetch next question: ${result.left.type}`)
    }
    return result.right
  }
}

```

```typescript
// /src/types/payloads/question/submitAnswerPayload.ts
import type { AnswerDataRequest } from "@/types/models/question/answer.ts";

export interface SubmitAnswerRequest {
  user_question_id: string;
  answer_data: AnswerDataRequest;
}
```

```typescript
export interface AnswerFieldRequest {
  field_id: string;
  user_answer: string;
}

export interface AnswerDataRequest {
  question_text: string | null;
  question: string | null;
  fields: AnswerFieldRequest[];
}



export interface AnswerFieldResponse {
  field_id: string;
  user_answer: string;
  is_correct: boolean;
  collect_answer: string;
  field_explanation: string;
}

export interface AnswerDataResponse {
  question_text: string | LocalizedString;
  explanation: string | LocalizedString;
  fields: AnswerFieldResponse[];
  next_user_question_id?: string | null;
}

export interface SubmitAnswerResponse {
  data: AnswerDataResponse;
}

export interface LocalizedString {
  en: string;
  ja: string;
}

```
