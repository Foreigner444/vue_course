# Guided Project: Interactive Survey Builder

---

## Project Overview

| Field | Value |
|:--- |:--- |
| **Module** | Module 5: User Interaction |
| **Project Number** | 2 of 3 |
| **Difficulty** | Intermediate-Advanced |
| **Estimated Time** | 60-75 minutes |

**What You'll Build:** An interactive survey builder that lets users create surveys with multiple question types (text, multiple choice, rating). Users can add, remove, and reorder questions, then preview the survey and collect responses.

**Real-World Scenario:** You're building a feedback tool for a SaaS product. Customers need to create custom surveys to gather user opinions. The builder must handle different input types dynamically while maintaining clean data flow through `v-model` bindings.

---

## Patterns You'll Practice

| Pattern | How It's Used |
|:--- |:--- |
| `defineModel()` | Two-way binding for question data |
| Dynamic `v-model` | Binding to dynamically created form fields |
| Named Slots | Custom rendering for different question types |
| Scoped Slots | Passing question data to parent for custom actions |
| Event Handling | Add/remove/reorder questions |
| `v-for` with Complex Objects | Rendering dynamic question list |
| `computed()` | Validation and derived state |
| TypeScript Discriminated Unions | Type-safe question types |

---

## Component Structure

```
src/
├── components/
│   └── survey/
│       ├── SurveyBuilder.vue
│       ├── QuestionEditor.vue
│       ├── QuestionTypeSelector.vue
│       ├── questions/
│       │   ├── TextQuestion.vue
│       │   ├── MultipleChoiceQuestion.vue
│       │   └── RatingQuestion.vue
│       └── SurveyPreview.vue
├── types/
│   └── survey.ts
├── App.vue
└── main.ts
```

---

## Step-by-Step Guide

### Step 1: Project Setup

```bash
npm create vue@latest survey-builder
cd survey-builder
npm install
npm install -D tailwindcss postcss autoprefixer daisyui
npx tailwindcss init -p
```

Configure Tailwind as shown in Project 1.

---

### Step 2: Define TypeScript Types

**File: `src/types/survey.ts`**

```typescript
export type QuestionType = 'text' | 'multiple-choice' | 'rating'

interface BaseQuestion {
  id: string
  title: string
  required: boolean
}

export interface TextQuestion extends BaseQuestion {
  type: 'text'
  placeholder: string
  maxLength: number | null
}

export interface MultipleChoiceQuestion extends BaseQuestion {
  type: 'multiple-choice'
  options: string[]
  allowMultiple: boolean
}

export interface RatingQuestion extends BaseQuestion {
  type: 'rating'
  maxRating: number
  labels: { low: string; high: string }
}

export type Question = TextQuestion | MultipleChoiceQuestion | RatingQuestion

export interface Survey {
  id: string
  title: string
  description: string
  questions: Question[]
}

export interface SurveyResponse {
  surveyId: string
  answers: Record<string, string | string[] | number>
  submittedAt: Date
}

export function createQuestion(type: QuestionType): Question {
  const id = crypto.randomUUID()
  const base = { id, title: '', required: false }
  
  switch (type) {
    case 'text':
      return { ...base, type: 'text', placeholder: '', maxLength: null }
    case 'multiple-choice':
      return { ...base, type: 'multiple-choice', options: ['Option 1', 'Option 2'], allowMultiple: false }
    case 'rating':
      return { ...base, type: 'rating', maxRating: 5, labels: { low: 'Poor', high: 'Excellent' } }
  }
}
```

---

### Step 3: Create Question Type Selector

**File: `src/components/survey/QuestionTypeSelector.vue`**

```vue
<script setup lang="ts">
import type { QuestionType } from '@/types/survey'

interface QuestionTypeOption {
  type: QuestionType
  label: string
  icon: string
  description: string
}

const options: QuestionTypeOption[] = [
  { 
    type: 'text', 
    label: 'Text', 
    icon: '📝',
    description: 'Free-form text response'
  },
  { 
    type: 'multiple-choice', 
    label: 'Multiple Choice', 
    icon: '☑️',
    description: 'Select from predefined options'
  },
  { 
    type: 'rating', 
    label: 'Rating', 
    icon: '⭐',
    description: 'Rate on a numeric scale'
  }
]

const emit = defineEmits<{
  select: [type: QuestionType]
}>()
</script>

<template>
  <div class="grid grid-cols-3 gap-3">
    <button
      v-for="option in options"
      :key="option.type"
      type="button"
      class="flex flex-col items-center p-4 border-2 border-dashed rounded-lg hover:border-primary hover:bg-primary/5 transition-all"
      @click="emit('select', option.type)"
    >
      <span class="text-3xl mb-2">{{ option.icon }}</span>
      <span class="font-medium">{{ option.label }}</span>
      <span class="text-xs text-base-content/50 text-center mt-1">
        {{ option.description }}
      </span>
    </button>
  </div>
</template>
```

---

### Step 4: Create Individual Question Components

**File: `src/components/survey/questions/TextQuestion.vue`**

```vue
<script setup lang="ts">
import type { TextQuestion } from '@/types/survey'

const question = defineModel<TextQuestion>({ required: true })
</script>

<template>
  <div class="space-y-4">
    <div class="form-control">
      <label class="label">
        <span class="label-text">Placeholder Text</span>
      </label>
      <input
        v-model="question.placeholder"
        type="text"
        class="input input-bordered input-sm"
        placeholder="Enter your answer..."
      />
    </div>
    
    <div class="form-control">
      <label class="label">
        <span class="label-text">Max Length (optional)</span>
      </label>
      <input
        v-model.number="question.maxLength"
        type="number"
        class="input input-bordered input-sm w-32"
        min="1"
        placeholder="No limit"
      />
    </div>
  </div>
</template>
```

**File: `src/components/survey/questions/MultipleChoiceQuestion.vue`**

```vue
<script setup lang="ts">
import type { MultipleChoiceQuestion } from '@/types/survey'

const question = defineModel<MultipleChoiceQuestion>({ required: true })

function addOption(): void {
  question.value.options = [
    ...question.value.options, 
    `Option ${question.value.options.length + 1}`
  ]
}

function removeOption(index: number): void {
  question.value.options = question.value.options.filter((_, i) => i !== index)
}

function updateOption(index: number, value: string): void {
  const newOptions = [...question.value.options]
  newOptions[index] = value
  question.value.options = newOptions
}
</script>

<template>
  <div class="space-y-4">
    <div class="form-control">
      <label class="label cursor-pointer justify-start gap-3">
        <input
          v-model="question.allowMultiple"
          type="checkbox"
          class="checkbox checkbox-sm"
        />
        <span class="label-text">Allow multiple selections</span>
      </label>
    </div>
    
    <div class="space-y-2">
      <label class="label">
        <span class="label-text">Options</span>
      </label>
      
      <div
        v-for="(option, index) in question.options"
        :key="index"
        class="flex items-center gap-2"
      >
        <span class="text-base-content/50 w-6">{{ index + 1 }}.</span>
        <input
          :value="option"
          @input="updateOption(index, ($event.target as HTMLInputElement).value)"
          type="text"
          class="input input-bordered input-sm flex-1"
          :placeholder="`Option ${index + 1}`"
        />
        <button
          type="button"
          class="btn btn-ghost btn-sm btn-square"
          :disabled="question.options.length <= 2"
          @click="removeOption(index)"
        >
          ✕
        </button>
      </div>
      
      <button
        type="button"
        class="btn btn-ghost btn-sm"
        @click="addOption"
      >
        + Add Option
      </button>
    </div>
  </div>
</template>
```

**File: `src/components/survey/questions/RatingQuestion.vue`**

```vue
<script setup lang="ts">
import type { RatingQuestion } from '@/types/survey'

const question = defineModel<RatingQuestion>({ required: true })

const ratingOptions = [3, 5, 7, 10]
</script>

<template>
  <div class="space-y-4">
    <div class="form-control">
      <label class="label">
        <span class="label-text">Rating Scale</span>
      </label>
      <div class="flex gap-2">
        <button
          v-for="rating in ratingOptions"
          :key="rating"
          type="button"
          class="btn btn-sm"
          :class="question.maxRating === rating ? 'btn-primary' : 'btn-ghost'"
          @click="question.maxRating = rating"
        >
          1-{{ rating }}
        </button>
      </div>
    </div>
    
    <div class="grid grid-cols-2 gap-4">
      <div class="form-control">
        <label class="label">
          <span class="label-text">Low Label</span>
        </label>
        <input
          v-model="question.labels.low"
          type="text"
          class="input input-bordered input-sm"
          placeholder="Poor"
        />
      </div>
      
      <div class="form-control">
        <label class="label">
          <span class="label-text">High Label</span>
        </label>
        <input
          v-model="question.labels.high"
          type="text"
          class="input input-bordered input-sm"
          placeholder="Excellent"
        />
      </div>
    </div>
    
    <!-- Preview -->
    <div class="p-3 bg-base-200 rounded-lg">
      <p class="text-sm text-base-content/70 mb-2">Preview:</p>
      <div class="flex items-center justify-between">
        <span class="text-xs">{{ question.labels.low }}</span>
        <div class="flex gap-1">
          <button
            v-for="n in question.maxRating"
            :key="n"
            type="button"
            class="btn btn-circle btn-sm btn-ghost"
          >
            {{ n }}
          </button>
        </div>
        <span class="text-xs">{{ question.labels.high }}</span>
      </div>
    </div>
  </div>
</template>
```

---

### Step 5: Create Question Editor Component

**File: `src/components/survey/QuestionEditor.vue`**

```vue
<script setup lang="ts">
import { computed } from 'vue'
import type { Question } from '@/types/survey'
import TextQuestion from './questions/TextQuestion.vue'
import MultipleChoiceQuestion from './questions/MultipleChoiceQuestion.vue'
import RatingQuestion from './questions/RatingQuestion.vue'

interface Props {
  index: number
  totalQuestions: number
}

const props = defineProps<Props>()

const question = defineModel<Question>({ required: true })

const emit = defineEmits<{
  remove: []
  moveUp: []
  moveDown: []
  duplicate: []
}>()

const typeLabels: Record<string, string> = {
  'text': '📝 Text',
  'multiple-choice': '☑️ Multiple Choice',
  'rating': '⭐ Rating'
}

const canMoveUp = computed(() => props.index > 0)
const canMoveDown = computed(() => props.index < props.totalQuestions - 1)
</script>

<template>
  <div class="card bg-base-100 border">
    <div class="card-body p-4">
      <!-- Header -->
      <div class="flex items-start justify-between gap-4">
        <div class="flex items-center gap-3">
          <span class="badge badge-outline">Q{{ index + 1 }}</span>
          <span class="text-sm text-base-content/50">{{ typeLabels[question.type] }}</span>
        </div>
        
        <div class="flex items-center gap-1">
          <button
            type="button"
            class="btn btn-ghost btn-xs"
            :disabled="!canMoveUp"
            title="Move up"
            @click="emit('moveUp')"
          >
            ↑
          </button>
          <button
            type="button"
            class="btn btn-ghost btn-xs"
            :disabled="!canMoveDown"
            title="Move down"
            @click="emit('moveDown')"
          >
            ↓
          </button>
          <button
            type="button"
            class="btn btn-ghost btn-xs"
            title="Duplicate"
            @click="emit('duplicate')"
          >
            📋
          </button>
          <button
            type="button"
            class="btn btn-ghost btn-xs text-error"
            title="Remove"
            @click="emit('remove')"
          >
            🗑️
          </button>
        </div>
      </div>
      
      <!-- Question Title -->
      <div class="form-control mt-2">
        <input
          v-model="question.title"
          type="text"
          class="input input-bordered"
          placeholder="Enter your question..."
        />
      </div>
      
      <!-- Required Toggle -->
      <label class="flex items-center gap-2 cursor-pointer mt-2">
        <input
          v-model="question.required"
          type="checkbox"
          class="checkbox checkbox-sm checkbox-primary"
        />
        <span class="label-text">Required</span>
      </label>
      
      <!-- Type-Specific Options -->
      <div class="mt-4 pt-4 border-t">
        <TextQuestion
          v-if="question.type === 'text'"
          v-model="(question as any)"
        />
        <MultipleChoiceQuestion
          v-else-if="question.type === 'multiple-choice'"
          v-model="(question as any)"
        />
        <RatingQuestion
          v-else-if="question.type === 'rating'"
          v-model="(question as any)"
        />
      </div>
    </div>
  </div>
</template>
```

---

### Step 6: Create Survey Preview Component

**File: `src/components/survey/SurveyPreview.vue`**

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'
import type { Survey, Question } from '@/types/survey'

interface Props {
  survey: Survey
}

const props = defineProps<Props>()

const emit = defineEmits<{
  submit: [answers: Record<string, any>]
  close: []
}>()

const answers = ref<Record<string, any>>({})

function initializeAnswers(): void {
  props.survey.questions.forEach(q => {
    if (q.type === 'multiple-choice' && q.allowMultiple) {
      answers.value[q.id] = []
    } else if (q.type === 'rating') {
      answers.value[q.id] = 0
    } else {
      answers.value[q.id] = ''
    }
  })
}

initializeAnswers()

const isValid = computed(() => {
  return props.survey.questions.every(q => {
    if (!q.required) return true
    const answer = answers.value[q.id]
    if (Array.isArray(answer)) return answer.length > 0
    if (typeof answer === 'number') return answer > 0
    return answer && answer.trim().length > 0
  })
})

function handleSubmit(): void {
  emit('submit', { ...answers.value })
}

function toggleMultipleChoice(questionId: string, option: string): void {
  const current = answers.value[questionId] as string[]
  if (current.includes(option)) {
    answers.value[questionId] = current.filter(o => o !== option)
  } else {
    answers.value[questionId] = [...current, option]
  }
}
</script>

<template>
  <div class="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4">
    <div class="bg-base-100 rounded-xl shadow-2xl w-full max-w-2xl max-h-[90vh] overflow-hidden flex flex-col">
      <!-- Header -->
      <div class="p-6 border-b">
        <div class="flex justify-between items-start">
          <div>
            <h2 class="text-xl font-bold">{{ survey.title || 'Untitled Survey' }}</h2>
            <p v-if="survey.description" class="text-base-content/70 mt-1">
              {{ survey.description }}
            </p>
          </div>
          <button
            type="button"
            class="btn btn-ghost btn-sm btn-circle"
            @click="emit('close')"
          >
            ✕
          </button>
        </div>
      </div>
      
      <!-- Questions -->
      <div class="flex-1 overflow-y-auto p-6 space-y-6">
        <div
          v-for="(question, index) in survey.questions"
          :key="question.id"
          class="space-y-3"
        >
          <label class="block">
            <span class="font-medium">
              {{ index + 1 }}. {{ question.title || 'Untitled Question' }}
              <span v-if="question.required" class="text-error">*</span>
            </span>
          </label>
          
          <!-- Text Question -->
          <template v-if="question.type === 'text'">
            <textarea
              v-model="answers[question.id]"
              class="textarea textarea-bordered w-full"
              :placeholder="question.placeholder"
              :maxlength="question.maxLength || undefined"
              rows="3"
            />
            <p v-if="question.maxLength" class="text-xs text-base-content/50 text-right">
              {{ (answers[question.id] as string).length }}/{{ question.maxLength }}
            </p>
          </template>
          
          <!-- Multiple Choice (Single) -->
          <template v-else-if="question.type === 'multiple-choice' && !question.allowMultiple">
            <div class="space-y-2">
              <label
                v-for="option in question.options"
                :key="option"
                class="flex items-center gap-3 cursor-pointer"
              >
                <input
                  v-model="answers[question.id]"
                  type="radio"
                  :value="option"
                  class="radio radio-primary"
                />
                <span>{{ option }}</span>
              </label>
            </div>
          </template>
          
          <!-- Multiple Choice (Multiple) -->
          <template v-else-if="question.type === 'multiple-choice' && question.allowMultiple">
            <div class="space-y-2">
              <label
                v-for="option in question.options"
                :key="option"
                class="flex items-center gap-3 cursor-pointer"
              >
                <input
                  type="checkbox"
                  :checked="(answers[question.id] as string[]).includes(option)"
                  class="checkbox checkbox-primary"
                  @change="toggleMultipleChoice(question.id, option)"
                />
                <span>{{ option }}</span>
              </label>
            </div>
          </template>
          
          <!-- Rating Question -->
          <template v-else-if="question.type === 'rating'">
            <div class="flex items-center justify-between">
              <span class="text-sm text-base-content/70">{{ question.labels.low }}</span>
              <div class="flex gap-1">
                <button
                  v-for="n in question.maxRating"
                  :key="n"
                  type="button"
                  class="btn btn-circle"
                  :class="answers[question.id] === n ? 'btn-primary' : 'btn-ghost'"
                  @click="answers[question.id] = n"
                >
                  {{ n }}
                </button>
              </div>
              <span class="text-sm text-base-content/70">{{ question.labels.high }}</span>
            </div>
          </template>
        </div>
        
        <div v-if="survey.questions.length === 0" class="text-center py-8 text-base-content/50">
          No questions in this survey yet.
        </div>
      </div>
      
      <!-- Footer -->
      <div class="p-6 border-t bg-base-200">
        <div class="flex justify-between items-center">
          <span class="text-sm text-base-content/50">
            {{ survey.questions.filter(q => q.required).length }} required question(s)
          </span>
          <div class="flex gap-2">
            <button
              type="button"
              class="btn btn-ghost"
              @click="emit('close')"
            >
              Cancel
            </button>
            <button
              type="button"
              class="btn btn-primary"
              :disabled="!isValid"
              @click="handleSubmit"
            >
              Submit Response
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
```

---

### Step 7: Create Main Survey Builder Component

**File: `src/components/survey/SurveyBuilder.vue`**

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'
import type { Survey, Question, QuestionType } from '@/types/survey'
import { createQuestion } from '@/types/survey'
import QuestionTypeSelector from './QuestionTypeSelector.vue'
import QuestionEditor from './QuestionEditor.vue'
import SurveyPreview from './SurveyPreview.vue'

const survey = defineModel<Survey>({ required: true })

const showTypeSelector = ref(false)
const showPreview = ref(false)

const questionCount = computed(() => survey.value.questions.length)
const requiredCount = computed(() => survey.value.questions.filter(q => q.required).length)

function addQuestion(type: QuestionType): void {
  const newQuestion = createQuestion(type)
  survey.value.questions = [...survey.value.questions, newQuestion]
  showTypeSelector.value = false
}

function removeQuestion(index: number): void {
  survey.value.questions = survey.value.questions.filter((_, i) => i !== index)
}

function moveQuestion(fromIndex: number, toIndex: number): void {
  const questions = [...survey.value.questions]
  const [removed] = questions.splice(fromIndex, 1)
  questions.splice(toIndex, 0, removed)
  survey.value.questions = questions
}

function duplicateQuestion(index: number): void {
  const original = survey.value.questions[index]
  const duplicate: Question = {
    ...JSON.parse(JSON.stringify(original)),
    id: crypto.randomUUID()
  }
  const questions = [...survey.value.questions]
  questions.splice(index + 1, 0, duplicate)
  survey.value.questions = questions
}

function updateQuestion(index: number, question: Question): void {
  const questions = [...survey.value.questions]
  questions[index] = question
  survey.value.questions = questions
}

function handlePreviewSubmit(answers: Record<string, any>): void {
  console.log('Survey submitted:', answers)
  alert('Survey response recorded!\n\n' + JSON.stringify(answers, null, 2))
  showPreview.value = false
}
</script>

<template>
  <div class="space-y-6">
    <!-- Survey Header -->
    <div class="card bg-base-100 border">
      <div class="card-body">
        <div class="form-control">
          <input
            v-model="survey.title"
            type="text"
            class="input input-lg input-ghost font-bold text-2xl px-0"
            placeholder="Untitled Survey"
          />
        </div>
        <div class="form-control">
          <textarea
            v-model="survey.description"
            class="textarea textarea-ghost px-0 resize-none"
            placeholder="Add a description..."
            rows="2"
          />
        </div>
        
        <div class="flex items-center justify-between mt-4 pt-4 border-t">
          <div class="flex gap-4 text-sm text-base-content/70">
            <span>{{ questionCount }} question(s)</span>
            <span>{{ requiredCount }} required</span>
          </div>
          <button
            type="button"
            class="btn btn-primary btn-sm"
            :disabled="questionCount === 0"
            @click="showPreview = true"
          >
            👁️ Preview Survey
          </button>
        </div>
      </div>
    </div>
    
    <!-- Questions List -->
    <div class="space-y-4">
      <TransitionGroup name="list">
        <QuestionEditor
          v-for="(question, index) in survey.questions"
          :key="question.id"
          :model-value="question"
          :index="index"
          :total-questions="questionCount"
          @update:model-value="updateQuestion(index, $event)"
          @remove="removeQuestion(index)"
          @move-up="moveQuestion(index, index - 1)"
          @move-down="moveQuestion(index, index + 1)"
          @duplicate="duplicateQuestion(index)"
        />
      </TransitionGroup>
    </div>
    
    <!-- Add Question Button / Type Selector -->
    <div class="card bg-base-200 border-2 border-dashed">
      <div class="card-body">
        <div v-if="showTypeSelector" class="space-y-4">
          <div class="flex items-center justify-between">
            <h3 class="font-medium">Choose Question Type</h3>
            <button
              type="button"
              class="btn btn-ghost btn-sm"
              @click="showTypeSelector = false"
            >
              Cancel
            </button>
          </div>
          <QuestionTypeSelector @select="addQuestion" />
        </div>
        
        <button
          v-else
          type="button"
          class="btn btn-ghost w-full"
          @click="showTypeSelector = true"
        >
          <span class="text-xl mr-2">+</span>
          Add Question
        </button>
      </div>
    </div>
    
    <!-- Preview Modal -->
    <SurveyPreview
      v-if="showPreview"
      :survey="survey"
      @submit="handlePreviewSubmit"
      @close="showPreview = false"
    />
  </div>
</template>

<style scoped>
.list-enter-active,
.list-leave-active {
  transition: all 0.3s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(-30px);
}
.list-move {
  transition: transform 0.3s ease;
}
</style>
```

---

### Step 8: Create Main App

**File: `src/App.vue`**

```vue
<script setup lang="ts">
import { reactive } from 'vue'
import type { Survey } from '@/types/survey'
import SurveyBuilder from '@/components/survey/SurveyBuilder.vue'

const survey = reactive<Survey>({
  id: crypto.randomUUID(),
  title: '',
  description: '',
  questions: []
})

function exportSurvey(): void {
  const json = JSON.stringify(survey, null, 2)
  const blob = new Blob([json], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = `survey-${survey.id.slice(0, 8)}.json`
  a.click()
  URL.revokeObjectURL(url)
}

function clearSurvey(): void {
  if (confirm('Are you sure you want to clear the entire survey?')) {
    survey.title = ''
    survey.description = ''
    survey.questions = []
  }
}
</script>

<template>
  <div class="min-h-screen bg-base-200">
    <!-- Navbar -->
    <nav class="navbar bg-base-100 shadow-sm sticky top-0 z-40">
      <div class="flex-1">
        <span class="text-xl font-bold px-4">📋 Survey Builder</span>
      </div>
      <div class="flex-none gap-2">
        <button
          class="btn btn-ghost btn-sm"
          :disabled="survey.questions.length === 0"
          @click="exportSurvey"
        >
          📥 Export JSON
        </button>
        <button
          class="btn btn-ghost btn-sm text-error"
          :disabled="survey.questions.length === 0"
          @click="clearSurvey"
        >
          🗑️ Clear
        </button>
      </div>
    </nav>
    
    <!-- Main Content -->
    <main class="max-w-3xl mx-auto py-8 px-4">
      <SurveyBuilder v-model="survey" />
    </main>
    
    <!-- Debug Panel -->
    <div class="fixed bottom-4 right-4">
      <details class="bg-base-100 rounded-lg shadow-lg p-2">
        <summary class="cursor-pointer text-sm font-medium px-2">
          Debug State
        </summary>
        <pre class="text-xs mt-2 p-2 bg-base-200 rounded max-h-64 overflow-auto max-w-md">{{ survey }}</pre>
      </details>
    </div>
  </div>
</template>
```

---

## How to Run

```bash
npm run dev
```

---

## Stretch Goals

1. **Drag-and-Drop Reordering:** Use `@vueuse/core`'s `useDraggable` for drag-and-drop
2. **Question Branching:** Add conditional logic to show/hide questions based on answers
3. **Survey Templates:** Create pre-built survey templates (NPS, Feedback, etc.)
4. **Response Analytics:** Build a dashboard to visualize collected responses
5. **Import/Export:** Add ability to import surveys from JSON files

---

## Self-Assessment Checklist

- [ ] All components use `<script setup>` with TypeScript
- [ ] `defineModel()` is used for question data binding
- [ ] Dynamic v-model works with different question types
- [ ] Questions can be added, removed, reordered, and duplicated
- [ ] Survey preview accurately renders all question types
- [ ] Form validation prevents submitting incomplete required fields
- [ ] TypeScript discriminated unions provide type safety
- [ ] Transitions animate question list changes
- [ ] Code follows Vue 3.4+ best practices
