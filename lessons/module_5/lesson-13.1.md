# Lesson 13.1: Creating Custom `v-model` Components — Theory & Concepts (Создание компонентов с пользовательским `v-model`)

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 13.1 of 25 (Part 1 of 2) |
| **Topic** | Creating Custom `v-model` Components / Пользовательский `v-model` |
| **Module** | Module 5: User Interaction |
| **Prerequisites** | Lessons 1-12 (Core Fundamentals, Event Handling, Basic v-model) |
| **You Should Know** | `v-model` on native inputs, `defineProps`, `defineEmits`, component communication |
| **Unlocks** | Lesson 13.2, then Lesson 14 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 1 (Theory):**
1. **Remember:** Recall the `defineModel()` macro syntax and its relationship to props/emits
2. **Understand:** Explain how custom `v-model` components work internally and when to create reusable form components

---

## 2. Real-World Scenario & Context

**Scenario:** Your design team has created a custom date picker, star rating widget, and color selector that appear throughout the application. Each needs to work seamlessly with `v-model` just like native `<input>` elements. Instead of forcing every parent component to manage `:value` + `@update` separately, you want users of your components to simply write `<StarRating v-model="rating" />`.

**Framework Context:** Vue 3.4+ introduced `defineModel()`, a compiler macro that dramatically simplifies creating components that support `v-model`. Previously, developers had to manually wire up props and emits with specific naming conventions. `defineModel()` handles all of this automatically, making custom form components as easy to create as they are to use.

---

## 3. Core Concepts Explained (Deep Dive)

### How It Actually Works

When you use `v-model` on a custom component, Vue transforms it into a prop + event pattern:

```vue
<!-- Parent uses v-model -->
<CustomInput v-model="username" />

<!-- Vue compiles this to: -->
<CustomInput :modelValue="username" @update:modelValue="username = $event" />
```

**The Old Way (Before Vue 3.4):**
```vue
<script setup lang="ts">
const props = defineProps<{ modelValue: string }>()
const emit = defineEmits<{ 'update:modelValue': [value: string] }>()

function updateValue(event: Event) {
  emit('update:modelValue', (event.target as HTMLInputElement).value)
}
</script>

<template>
  <input :value="modelValue" @input="updateValue" />
</template>
```

**The Modern Way (Vue 3.4+):**
```vue
<script setup lang="ts">
const model = defineModel<string>()
</script>

<template>
  <input v-model="model" />
</template>
```

```
┌──────────────────────────────────────────────────────────────────┐
│              defineModel() Under the Hood                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│   What you write:                                                │
│   ┌────────────────────────────────────┐                        │
│   │ const model = defineModel<string>()│                        │
│   └────────────────────────────────────┘                        │
│                         │                                        │
│                         ▼                                        │
│   What Vue generates:                                            │
│   ┌────────────────────────────────────────────────────────┐    │
│   │ const props = defineProps<{ modelValue: string }>()    │    │
│   │ const emit = defineEmits<{                             │    │
│   │   'update:modelValue': [value: string]                 │    │
│   │ }>()                                                   │    │
│   │                                                        │    │
│   │ const model = computed({                               │    │
│   │   get: () => props.modelValue,                         │    │
│   │   set: (val) => emit('update:modelValue', val)         │    │
│   │ })                                                     │    │
│   └────────────────────────────────────────────────────────┘    │
│                                                                  │
│   Result: `model` is a ref you can use with v-model!             │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Mental Model

**Think of `defineModel()` as a "two-way portal":**

Imagine your component has a portal door. When the parent pushes a value through the door, it appears inside your component. When your component changes the value, it's automatically pushed back out to the parent. The `defineModel()` ref is that magical portal door.

```
┌─────────────────────────────────────────────────────────────┐
│                   🚪 The Portal Door                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   PARENT COMPONENT              CHILD COMPONENT             │
│   ┌─────────────────┐          ┌─────────────────┐         │
│   │                 │          │                 │         │
│   │  rating = 4     │◄────────►│  model = 4      │         │
│   │                 │   🚪     │                 │         │
│   │                 │  PORTAL  │  (click star)   │         │
│   │  (auto-updates) │◄─────────│  model = 5      │         │
│   │  rating = 5     │          │                 │         │
│   │                 │          │                 │         │
│   └─────────────────┘          └─────────────────┘         │
│                                                             │
│   defineModel() creates the portal automatically!           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### In Other Contexts

| Framework | Custom Two-Way Binding | Notes |
|:--- |:--- |:--- |
| **Vue 3.4+** | `defineModel()` | Compiler macro, simplest syntax |
| **Vue 3.0-3.3** | `defineProps` + `defineEmits` | Manual wiring required |
| **React** | `value` + `onChange` props | Always explicit, no sugar |
| **Angular** | `[(ngModel)]` with `ControlValueAccessor` | Complex interface implementation |
| **Svelte** | `bind:value` with exported `let value` | Simple export creates binding |

### When to Use / When NOT to Use

| ✅ Create Custom v-model When | ❌ Avoid When |
|:--- |:--- |
| Building reusable form components (inputs, selects, pickers) | The component just displays data (use props only) |
| Wrapping third-party input libraries | You need one-way data flow for predictability |
| Creating complex input widgets (date pickers, rich text editors) | Simple components that emit events without returning values |
| Standardizing form field styling across your app | Internal components that don't need external value binding |

---

## 4. New Terminology

| Term | Definition |
|:--- |:--- |
| **`defineModel()`** | A Vue 3.4+ compiler macro that creates a ref automatically synchronized with the parent's `v-model` binding. Replaces manual prop/emit wiring. |
| **`modelValue`** | The default prop name Vue uses for `v-model` bindings. When parent writes `v-model="x"`, child receives it as `modelValue` prop. |
| **Named v-model** | Using `v-model:name="value"` to bind to a prop other than `modelValue`. Enables multiple v-model bindings on one component. |
| **Model Modifiers** | Custom processing options passed via `v-model.modifier="value"`. Accessible in the component to alter binding behavior (e.g., `.trim`, `.capitalize`). |

---

## 5. Algorithmic Thinking (Planning the Solution)

**The Plan:** Building a reusable StarRating component with `v-model` support.

```
┌─────────────────────────────────────────────────────────────┐
│        Decision Tree: Custom v-model Component              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Step 1: Identify the value type                           │
│   └── Rating: number (1-5)                                  │
│       └── defineModel<number>()                             │
│                                                             │
│   Step 2: Consider default value                            │
│   └── Should work without initial v-model?                  │
│       └── Yes: defineModel<number>({ default: 0 })          │
│                                                             │
│   Step 3: Design the internal UI                            │
│   └── 5 star buttons                                        │
│   └── Click sets rating to that star's position             │
│   └── Visual feedback for selected stars                    │
│                                                             │
│   Step 4: Bind internal state to model                      │
│   └── model.value = clickedStarIndex                        │
│   └── Vue auto-emits to parent                              │
│                                                             │
│   Step 5: Add optional features                             │
│   └── Hover preview?                                        │
│   └── Disabled state via prop?                              │
│   └── Custom star count via prop?                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. Initial Pattern Introduction

| What You Want (Intent) | Code Chunk (The Pattern) | Conceptual Link |
|:--- |:--- |:--- |
| Basic v-model support | `const model = defineModel<T>()` | Creates synced ref |
| With default value | `defineModel<T>({ default: value })` | Fallback when parent provides nothing |
| Required v-model | `defineModel<T>({ required: true })` | TypeScript will enforce |
| Named v-model | `defineModel<T>('rating')` | Parent uses `v-model:rating` |
| Multiple v-models | Multiple `defineModel` calls with names | e.g., `v-model:start` + `v-model:end` |
| Access modifiers | `const [model, modifiers] = defineModel()` | Get `.lazy`, `.trim`, custom modifiers |

### Basic Pattern: Simple Text Input Wrapper

```vue
<script setup lang="ts">
// defineModel creates a ref that syncs with parent's v-model
// When parent does <MyInput v-model="name" />, this ref connects to 'name'
const model = defineModel<string>()
</script>

<template>
  <!-- Use the model ref directly with v-model on native input -->
  <input 
    v-model="model" 
    class="input input-bordered w-full"
  />
</template>
```

**Parent Usage:**
```vue
<script setup lang="ts">
import { ref } from 'vue'
import MyInput from './MyInput.vue'

const username = ref('')
</script>

<template>
  <!-- Works exactly like native input! -->
  <MyInput v-model="username" />
  <p>You typed: {{ username }}</p>
</template>
```

### Named v-model Pattern: Date Range Picker

```vue
<script setup lang="ts">
// Named models allow multiple v-model bindings on one component
// Parent will use: v-model:start="startDate" v-model:end="endDate"
const startDate = defineModel<string>('start')
const endDate = defineModel<string>('end')
</script>

<template>
  <div class="flex gap-2">
    <input v-model="startDate" type="date" class="input input-bordered" />
    <span>to</span>
    <input v-model="endDate" type="date" class="input input-bordered" />
  </div>
</template>
```

**Parent Usage:**
```vue
<script setup lang="ts">
import { ref } from 'vue'
import DateRangePicker from './DateRangePicker.vue'

const start = ref('2024-01-01')
const end = ref('2024-12-31')
</script>

<template>
  <DateRangePicker v-model:start="start" v-model:end="end" />
</template>
```

### With Modifiers Pattern: Custom Modifier Handling

```vue
<script setup lang="ts">
// Destructure to get both the model and any modifiers applied
const [model, modifiers] = defineModel<string>()

// modifiers is an object like { capitalize: true, trim: true }
// when parent uses v-model.capitalize.trim="value"

function handleInput(event: Event): void {
  let value = (event.target as HTMLInputElement).value
  
  // Apply custom 'capitalize' modifier
  if (modifiers.capitalize) {
    value = value.charAt(0).toUpperCase() + value.slice(1)
  }
  
  model.value = value
}
</script>

<template>
  <input 
    :value="model" 
    @input="handleInput"
    class="input input-bordered"
  />
</template>
```

**Parent Usage:**
```vue
<template>
  <!-- Custom .capitalize modifier will uppercase first letter -->
  <MyInput v-model.capitalize="name" />
</template>
```

### Complete Example: Star Rating Component

```vue
<script setup lang="ts">
import { computed } from 'vue'

// Props for customization (separate from v-model)
interface Props {
  max?: number
  disabled?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  max: 5,
  disabled: false
})

// The v-model binding
const rating = defineModel<number>({ default: 0 })

// Generate array for v-for rendering
const stars = computed(() => 
  Array.from({ length: props.max }, (_, i) => i + 1)
)

// Handler for clicking a star
function setRating(value: number): void {
  if (props.disabled) return
  rating.value = value
}

// Check if a star should be filled
function isStarFilled(starNumber: number): boolean {
  return starNumber <= (rating.value ?? 0)
}
</script>

<template>
  <div 
    class="flex gap-1"
    :class="{ 'opacity-50 cursor-not-allowed': disabled }"
  >
    <button
      v-for="star in stars"
      :key="star"
      type="button"
      @click="setRating(star)"
      :disabled="disabled"
      class="text-2xl transition-transform hover:scale-110 focus:outline-none"
      :class="isStarFilled(star) ? 'text-yellow-400' : 'text-gray-300'"
      :aria-label="`Rate ${star} out of ${max}`"
    >
      ★
    </button>
  </div>
</template>
```

**Parent Usage:**
```vue
<script setup lang="ts">
import { ref } from 'vue'
import StarRating from './StarRating.vue'

const productRating = ref(3)
</script>

<template>
  <StarRating v-model="productRating" :max="5" />
  <p>Current rating: {{ productRating }}/5</p>
  
  <!-- Disabled state -->
  <StarRating v-model="productRating" disabled />
</template>
```

---

## 7. Comprehension Check

1. **What does `defineModel<string>()` return, and how is it different from a regular `ref()`?**
   - *Hint: Think about what happens when you modify its `.value` property.*

2. **How would you create a component that accepts two different `v-model` bindings, such as `v-model:min` and `v-model:max`?**
   - *Hint: Consider the first argument to `defineModel()`.*

3. **True or False: When using `defineModel()`, you still need to manually call `defineEmits()` to emit the update event.**
   - *Hint: Consider what "compiler macro" means and what it generates for you.*

---

**Reply 'next' for Lesson 13.2 (Practice).**
