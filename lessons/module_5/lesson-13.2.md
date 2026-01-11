# Lesson 13.2: Creating Custom `v-model` Components — Practice & Application (Создание компонентов с пользовательским `v-model`)

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 13.2 of 25 (Part 2 of 2) |
| **Topic** | Creating Custom `v-model` Components / Пользовательский `v-model` |
| **Continues From** | Lesson 13.1 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Build reusable form components with `defineModel()` including typed inputs, toggles, and complex widgets
2. **Analyze:** Debug common issues with custom v-model components and compare implementation approaches

---

## 2. Quick Recap

- **`defineModel<T>()`** creates a ref that syncs bidirectionally with parent's `v-model`
- **Named models:** `defineModel<T>('name')` enables `v-model:name` for multiple bindings
- **Modifiers:** `const [model, modifiers] = defineModel()` gives access to `.lazy`, `.trim`, or custom modifiers
- **Default values:** `defineModel({ default: value })` provides fallback when parent doesn't supply initial value

---

## 3. The Variable Frame & Complexity Scale

**Basic Form:**
```vue
<script setup lang="ts">
const model = defineModel<string>()
</script>

<template>
  <input v-model="model" />
</template>
```

**With Options and Props:**
```vue
<script setup lang="ts">
interface Props {
  placeholder?: string
  disabled?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  placeholder: '',
  disabled: false
})

const model = defineModel<string>({ required: true })
</script>

<template>
  <input 
    v-model="model" 
    :placeholder="props.placeholder"
    :disabled="props.disabled"
    class="input input-bordered"
  />
</template>
```

**Advanced Form (Multiple v-models + Modifiers):**
```vue
<script setup lang="ts">
import { watch } from 'vue'

interface Props {
  label?: string
}

defineProps<Props>()

const [value, valueModifiers] = defineModel<string>('value', { default: '' })
const isValid = defineModel<boolean>('valid', { default: true })

watch(value, (newValue) => {
  if (valueModifiers.required) {
    isValid.value = newValue.trim().length > 0
  }
  if (valueModifiers.email) {
    isValid.value = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(newValue)
  }
})
</script>

<template>
  <div>
    <label v-if="label" class="label">{{ label }}</label>
    <input 
      v-model="value"
      class="input input-bordered"
      :class="{ 'input-error': !isValid }"
    />
  </div>
</template>
```

---

## 4. Micro-Implementation Drills

### Drill 1 (Basic): Custom Text Input Component
Create a styled text input that supports `v-model`.

```vue
<!-- StyledInput.vue -->
<script setup lang="ts">
// TODO: Use defineModel to create a string model
</script>

<template>
  <!-- TODO: Bind the model to the input using v-model -->
  <input
    type="text"
    class="w-full px-4 py-2 border-2 border-gray-300 rounded-lg focus:border-blue-500 focus:outline-none transition-colors"
  />
</template>
```

**Solution:**
```vue
<script setup lang="ts">
const model = defineModel<string>({ default: '' })
</script>

<template>
  <input
    v-model="model"
    type="text"
    class="w-full px-4 py-2 border-2 border-gray-300 rounded-lg focus:border-blue-500 focus:outline-none transition-colors"
  />
</template>
```

---

### Drill 2 (Variation): Toggle Switch Component
Create a toggle switch that works with `v-model` for boolean values.

```vue
<!-- ToggleSwitch.vue -->
<script setup lang="ts">
// TODO: Create a boolean model with default false
// TODO: Create a function to toggle the value
</script>

<template>
  <!-- TODO: Make the button toggle the model value on click -->
  <!-- TODO: Use model value to conditionally style the switch -->
  <button
    type="button"
    class="relative inline-flex h-6 w-11 items-center rounded-full transition-colors"
  >
    <span
      class="inline-block h-4 w-4 transform rounded-full bg-white transition-transform"
    />
  </button>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
const model = defineModel<boolean>({ default: false })

function toggle(): void {
  model.value = !model.value
}
</script>

<template>
  <button
    type="button"
    @click="toggle"
    class="relative inline-flex h-6 w-11 items-center rounded-full transition-colors"
    :class="model ? 'bg-blue-600' : 'bg-gray-300'"
    role="switch"
    :aria-checked="model"
  >
    <span
      class="inline-block h-4 w-4 transform rounded-full bg-white transition-transform"
      :class="model ? 'translate-x-6' : 'translate-x-1'"
    />
  </button>
</template>
```

---

### Drill 3 (Combination): Quantity Selector with Min/Max
Create a quantity selector with increment/decrement buttons and bounds.

```vue
<!-- QuantitySelector.vue -->
<script setup lang="ts">
import { computed } from 'vue'

interface Props {
  min?: number
  max?: number
}

// TODO: Define props with defaults (min: 0, max: 99)
// TODO: Create a number model with default of 1
// TODO: Create increment and decrement functions that respect min/max
// TODO: Create computed properties to check if at min/max bounds
</script>

<template>
  <div class="flex items-center gap-2">
    <!-- TODO: Disable button when at minimum -->
    <button class="btn btn-circle btn-sm">−</button>
    
    <!-- TODO: Bind model to input, use .number modifier -->
    <input
      type="number"
      class="input input-bordered w-20 text-center"
    />
    
    <!-- TODO: Disable button when at maximum -->
    <button class="btn btn-circle btn-sm">+</button>
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { computed } from 'vue'

interface Props {
  min?: number
  max?: number
}

const props = withDefaults(defineProps<Props>(), {
  min: 0,
  max: 99
})

const model = defineModel<number>({ default: 1 })

const isAtMin = computed(() => model.value <= props.min)
const isAtMax = computed(() => model.value >= props.max)

function increment(): void {
  if (model.value < props.max) {
    model.value++
  }
}

function decrement(): void {
  if (model.value > props.min) {
    model.value--
  }
}
</script>

<template>
  <div class="flex items-center gap-2">
    <button 
      @click="decrement"
      :disabled="isAtMin"
      class="btn btn-circle btn-sm"
    >
      −
    </button>
    
    <input
      v-model.number="model"
      type="number"
      :min="min"
      :max="max"
      class="input input-bordered w-20 text-center"
    />
    
    <button 
      @click="increment"
      :disabled="isAtMax"
      class="btn btn-circle btn-sm"
    >
      +
    </button>
  </div>
</template>
```

---

### Drill 4 (Edge Case): Tag Input with Multiple v-models
Create a tag input that exposes both the current input text and the list of tags.

```vue
<!-- TagInput.vue -->
<script setup lang="ts">
// TODO: Create named model 'input' for current text being typed
// TODO: Create named model 'tags' for the array of added tags
// TODO: Function to add current input as tag (on Enter)
// TODO: Function to remove a tag by index
</script>

<template>
  <div class="border rounded-lg p-2">
    <!-- Display existing tags -->
    <div class="flex flex-wrap gap-2 mb-2">
      <!-- TODO: v-for over tags, show remove button -->
    </div>
    
    <!-- Input for new tags -->
    <!-- TODO: Bind to 'input' model, add tag on Enter key -->
    <input
      type="text"
      placeholder="Type and press Enter"
      class="input input-sm w-full"
    />
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
const inputText = defineModel<string>('input', { default: '' })
const tags = defineModel<string[]>('tags', { default: () => [] })

function addTag(): void {
  const trimmed = inputText.value.trim()
  if (trimmed && !tags.value.includes(trimmed)) {
    tags.value = [...tags.value, trimmed]
    inputText.value = ''
  }
}

function removeTag(index: number): void {
  tags.value = tags.value.filter((_, i) => i !== index)
}
</script>

<template>
  <div class="border rounded-lg p-2">
    <div class="flex flex-wrap gap-2 mb-2">
      <span
        v-for="(tag, index) in tags"
        :key="tag"
        class="badge badge-primary gap-1"
      >
        {{ tag }}
        <button 
          @click="removeTag(index)"
          class="btn btn-ghost btn-xs"
        >
          ×
        </button>
      </span>
    </div>
    
    <input
      v-model="inputText"
      @keyup.enter="addTag"
      type="text"
      placeholder="Type and press Enter"
      class="input input-sm w-full"
    />
  </div>
</template>
```

**Parent Usage:**
```vue
<script setup lang="ts">
import { ref } from 'vue'
import TagInput from './TagInput.vue'

const currentInput = ref('')
const selectedTags = ref<string[]>(['vue', 'typescript'])
</script>

<template>
  <TagInput v-model:input="currentInput" v-model:tags="selectedTags" />
  <p>Typing: {{ currentInput }}</p>
  <p>Tags: {{ selectedTags.join(', ') }}</p>
</template>
```

---

## 5. Common Pitfalls & Anti-Patterns

| ❌ Common Mistake | ✅ Correct Approach | Why It Matters |
|:--- |:--- |:--- |
| Mutating model array directly with `.push()` | Create new array: `model.value = [...model.value, newItem]` | Vue's reactivity may not detect mutations on the same array reference; new reference ensures update |
| Using `defineModel` without proper TypeScript type | Always type: `defineModel<string>()` | Without type, you get `unknown`, losing autocomplete and type safety |
| Forgetting default for optional v-model | Use `defineModel({ default: value })` | If parent doesn't provide v-model, the model will be `undefined`, causing errors |

### Detailed Examples

**Mistake 1: Array Mutation**
```vue
<!-- ❌ WRONG: Vue might not detect this mutation -->
<script setup lang="ts">
const items = defineModel<string[]>({ default: () => [] })

function addItem(item: string): void {
  items.value.push(item)  // Mutating existing array
}
</script>

<!-- ✅ CORRECT: Create new array reference -->
<script setup lang="ts">
const items = defineModel<string[]>({ default: () => [] })

function addItem(item: string): void {
  items.value = [...items.value, item]  // New array
}
</script>
```

**Mistake 2: Missing Type Parameter**
```vue
<!-- ❌ WRONG: model is typed as Ref<unknown> -->
<script setup lang="ts">
const model = defineModel()  // No type!

function process(): void {
  console.log(model.value.toUpperCase())  // TS error: unknown has no toUpperCase
}
</script>

<!-- ✅ CORRECT: Explicitly type the model -->
<script setup lang="ts">
const model = defineModel<string>()

function process(): void {
  console.log(model.value?.toUpperCase())  // Works with type safety
}
</script>
```

**Mistake 3: No Default Value**
```vue
<!-- ❌ WRONG: Will crash if parent doesn't provide v-model -->
<script setup lang="ts">
const count = defineModel<number>()  // No default
</script>

<template>
  <!-- ERROR: Cannot read properties of undefined -->
  <p>{{ count.toFixed(2) }}</p>
</template>

<!-- ✅ CORRECT: Provide default value -->
<script setup lang="ts">
const count = defineModel<number>({ default: 0 })
</script>

<template>
  <p>{{ count.toFixed(2) }}</p>  <!-- Works: "0.00" -->
</template>
```

---

## 6. Mini-Project (Putting It Together)

**Color Picker Component**

A complete color picker with hex input, sliders for RGB, and preview.

**File Structure:**
```
src/
├── components/
│   └── ColorPicker.vue
├── App.vue
└── main.ts
```

**ColorPicker.vue:**
```vue
<script setup lang="ts">
import { computed, watch } from 'vue'

interface Props {
  showPreview?: boolean
  showHex?: boolean
  showRgb?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  showPreview: true,
  showHex: true,
  showRgb: true
})

const color = defineModel<string>({ default: '#3b82f6' })

const rgb = computed(() => {
  const hex = color.value.replace('#', '')
  return {
    r: parseInt(hex.substring(0, 2), 16) || 0,
    g: parseInt(hex.substring(2, 4), 16) || 0,
    b: parseInt(hex.substring(4, 6), 16) || 0
  }
})

function toHex(value: number): string {
  return value.toString(16).padStart(2, '0')
}

function updateFromRgb(channel: 'r' | 'g' | 'b', value: number): void {
  const clamped = Math.max(0, Math.min(255, value))
  const newRgb = { ...rgb.value, [channel]: clamped }
  color.value = `#${toHex(newRgb.r)}${toHex(newRgb.g)}${toHex(newRgb.b)}`
}

function handleHexInput(event: Event): void {
  const input = (event.target as HTMLInputElement).value
  if (/^#[0-9A-Fa-f]{6}$/.test(input)) {
    color.value = input.toLowerCase()
  }
}

const presetColors = [
  '#ef4444', '#f97316', '#eab308', '#22c55e', 
  '#3b82f6', '#8b5cf6', '#ec4899', '#6b7280'
]

function selectPreset(preset: string): void {
  color.value = preset
}
</script>

<template>
  <div class="p-4 border rounded-lg bg-base-100 w-80">
    <div v-if="showPreview" class="mb-4">
      <div
        class="h-16 rounded-lg border-2 border-base-300 transition-colors"
        :style="{ backgroundColor: color }"
      />
    </div>
    
    <div v-if="showHex" class="mb-4">
      <label class="label text-sm font-medium">Hex Color</label>
      <div class="flex gap-2">
        <input
          type="color"
          :value="color"
          @input="color = ($event.target as HTMLInputElement).value"
          class="w-12 h-10 cursor-pointer rounded"
        />
        <input
          type="text"
          :value="color"
          @input="handleHexInput"
          maxlength="7"
          class="input input-bordered flex-1 font-mono uppercase"
          placeholder="#000000"
        />
      </div>
    </div>
    
    <div v-if="showRgb" class="space-y-3 mb-4">
      <div>
        <label class="flex justify-between text-sm">
          <span class="text-red-500 font-medium">Red</span>
          <span class="font-mono">{{ rgb.r }}</span>
        </label>
        <input
          type="range"
          :value="rgb.r"
          @input="updateFromRgb('r', Number(($event.target as HTMLInputElement).value))"
          min="0"
          max="255"
          class="range range-error range-sm w-full"
        />
      </div>
      
      <div>
        <label class="flex justify-between text-sm">
          <span class="text-green-500 font-medium">Green</span>
          <span class="font-mono">{{ rgb.g }}</span>
        </label>
        <input
          type="range"
          :value="rgb.g"
          @input="updateFromRgb('g', Number(($event.target as HTMLInputElement).value))"
          min="0"
          max="255"
          class="range range-success range-sm w-full"
        />
      </div>
      
      <div>
        <label class="flex justify-between text-sm">
          <span class="text-blue-500 font-medium">Blue</span>
          <span class="font-mono">{{ rgb.b }}</span>
        </label>
        <input
          type="range"
          :value="rgb.b"
          @input="updateFromRgb('b', Number(($event.target as HTMLInputElement).value))"
          min="0"
          max="255"
          class="range range-info range-sm w-full"
        />
      </div>
    </div>
    
    <div class="space-y-2">
      <label class="text-sm font-medium">Presets</label>
      <div class="flex flex-wrap gap-2">
        <button
          v-for="preset in presetColors"
          :key="preset"
          @click="selectPreset(preset)"
          class="w-8 h-8 rounded-full border-2 transition-transform hover:scale-110"
          :class="{ 'ring-2 ring-offset-2 ring-primary': color === preset }"
          :style="{ backgroundColor: preset }"
          :aria-label="`Select ${preset}`"
        />
      </div>
    </div>
    
    <div class="mt-4 pt-4 border-t text-sm font-mono text-center">
      {{ color }}
      <span class="text-base-content/50 ml-2">
        rgb({{ rgb.r }}, {{ rgb.g }}, {{ rgb.b }})
      </span>
    </div>
  </div>
</template>
```

**App.vue Usage:**
```vue
<script setup lang="ts">
import { ref } from 'vue'
import ColorPicker from './components/ColorPicker.vue'

const primaryColor = ref('#3b82f6')
const accentColor = ref('#ec4899')
</script>

<template>
  <div class="p-8 space-y-8">
    <h1 class="text-2xl font-bold">Theme Colors</h1>
    
    <div class="flex gap-8">
      <div>
        <h2 class="mb-2 font-semibold">Primary Color</h2>
        <ColorPicker v-model="primaryColor" />
      </div>
      
      <div>
        <h2 class="mb-2 font-semibold">Accent Color</h2>
        <ColorPicker v-model="accentColor" :show-rgb="false" />
      </div>
    </div>
    
    <div 
      class="p-6 rounded-lg text-white"
      :style="{ backgroundColor: primaryColor }"
    >
      <button 
        class="px-4 py-2 rounded font-semibold"
        :style="{ backgroundColor: accentColor }"
      >
        Preview Button
      </button>
    </div>
  </div>
</template>
```

**How to Test:**
1. Create new Vue project with TypeScript and Tailwind/DaisyUI
2. Add the `ColorPicker.vue` component
3. Use in `App.vue` with `v-model`
4. Test: Adjust sliders, pick presets, type hex values
5. Verify: All controls stay in sync with the model
6. Confirm: Parent component receives updates immediately

---

## 7. Implementation Exercises (Progressive)

### Level 1 (Recall): Fill in the Blanks
```vue
<script setup lang="ts">
// Create a v-model for a boolean value with default false
const isActive = ___________<_______>({ _______: false })
</script>

<template>
  <button @click="isActive = !isActive">
    {{ isActive ? 'Active' : 'Inactive' }}
  </button>
</template>
```

**Answer:** `defineModel<boolean>({ default: false })`

---

### Level 2 (Apply): Independent Coding Tasks

**Exercise 2.1: Rating Slider**
Create a component that:
- Accepts `v-model` for a number 1-10
- Shows a slider and numeric display
- Has optional `label` prop
- Emits value changes in real-time

**Exercise 2.2: Dual-Handle Range**
Create a component that:
- Accepts `v-model:min` and `v-model:max` for a range
- Ensures min never exceeds max
- Shows both values and a visual range bar

---

### Level 3 (Debug): Fix the Broken Code
```vue
<script setup lang="ts">
import { ref } from 'vue'

// Custom select component
const selected = defineModel()

const options = ['Apple', 'Banana', 'Cherry']

function selectOption(option) {
  selected = option
}
</script>

<template>
  <div class="dropdown">
    <button class="btn">{{ selected || 'Select...' }}</button>
    <ul class="menu">
      <li v-for="option in options" :key="option">
        <a @click="selectOption(option)">{{ option }}</a>
      </li>
    </ul>
  </div>
</template>
```

**Bugs to Find:**
1. `defineModel()` missing type parameter
2. `selected = option` should be `selected.value = option` (model is a ref)
3. `selectOption(option)` parameter missing type annotation

**Fixed Code:**
```vue
<script setup lang="ts">
const selected = defineModel<string>({ default: '' })

const options = ['Apple', 'Banana', 'Cherry']

function selectOption(option: string): void {
  selected.value = option
}
</script>

<template>
  <div class="dropdown">
    <button class="btn">{{ selected || 'Select...' }}</button>
    <ul class="menu">
      <li v-for="option in options" :key="option">
        <a @click="selectOption(option)">{{ option }}</a>
      </li>
    </ul>
  </div>
</template>
```

---

**Reply 'next' for Lesson 14.1 (Theory).**
