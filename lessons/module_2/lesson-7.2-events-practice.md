# Lesson 7.2: Component Communication: Emitting Events — Practice & Application (Эмиссия Событий)
---
## 1. Lesson Metadata
| Field | Value |
|:--- |:--- |
| **Lesson Number** | 7.2 of 25 (Part 2 of 2) |
| **Topic** | Emitting Events / Эмиссия Событий |
| **Continues From** | Lesson 7.1 |
| **Duration** | 30-35 minutes |
**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Create components that emit typed events with payloads
2. **Analyze:** Debug common event-related issues and compare event patterns
---
## 2. Quick Recap
From Lesson 7.1, remember:
- **Events flow upward**: Child → Parent (the complement to props)
- **`defineEmits<T>()`**: Declares typed events with payload types
- **`emit('eventName', payload)`**: Triggers the event
- **`@event-name="handler"`**: Parent listens in template (kebab-case convention)
---
## 3. The Variable Frame & Complexity Scale
### Basic Form: Simple Event without Payload
```vue
<script setup lang="ts">
// Simplest usage: event with no data
const emit = defineEmits<{
  close: []
  cancel: []
}>()
function handleClose() {
  emit('close')
}
</script>
<template>
  <button @click="handleClose">Close</button>
  <button @click="emit('cancel')">Cancel</button>
</template>
```
### With Options: Events with Typed Payloads
```vue
<script setup lang="ts">
// Adding typed payloads to events
const emit = defineEmits<{
  select: [id: number]
  search: [query: string, filters: { category?: string; minPrice?: number }]
  rate: [rating: number, comment?: string]
}>()
function handleSelect(itemId: number) {
  emit('select', itemId)
}
function handleSearch(searchQuery: string) {
  emit('search', searchQuery, { category: 'electronics', minPrice: 10 })
}
function handleRate(stars: number, feedback?: string) {
  emit('rate', stars, feedback)
}
</script>
```
### Advanced Form: Complex Events with Objects
```vue
<script setup lang="ts">
// Complex events with full type definitions
interface FormData {
  name: string
  email: string
  preferences: {
    newsletter: boolean
    notifications: boolean
  }
}
interface ValidationError {
  field: string
  message: string
}
const emit = defineEmits<{
  submit: [data: FormData]
  validationError: [errors: ValidationError[]]
  fieldChange: [field: keyof FormData, value: unknown]
  cancel: []
}>()
function handleSubmit(formData: FormData) {
  const errors: ValidationError[] = []
  
  if (!formData.email.includes('@')) {
    errors.push({ field: 'email', message: 'Invalid email format' })
  }
  
  if (errors.length > 0) {
    emit('validationError', errors)
    return
  }
  
  emit('submit', formData)
}
</script>
```
---
## 4. Micro-Implementation Drills
### Drill 1 (Basic): Toggle Button
Create a toggle button that emits when its state changes.
```vue
<!-- ToggleButton.vue — Starter Code -->
<script setup lang="ts">
import { ref } from 'vue'
// TODO: Define emit for 'toggle' event that sends boolean (new state)
const isOn = ref(false)
function toggle() {
  isOn.value = !isOn.value
  // TODO: Emit 'toggle' with the new state
}
</script>
<template>
  <button 
    @click="toggle" 
    :class="isOn ? 'btn-primary' : 'btn-ghost'"
    class="btn"
  >
    {{ isOn ? 'ON' : 'OFF' }}
  </button>
</template>
```
**Expected Parent Usage:**
```vue
<ToggleButton @toggle="(state) => console.log('Toggle:', state)" />
```
---
### Drill 2 (Variation): Quantity Selector
Create a component with increment/decrement buttons that emit quantity changes.
```vue
<!-- QuantitySelector.vue — Starter Code -->
<script setup lang="ts">
import { ref, watch } from 'vue'
// Props: receive initial quantity and min/max bounds
const props = withDefaults(defineProps<{
  modelValue?: number
  min?: number
  max?: number
}>(), {
  modelValue: 1,
  min: 0,
  max: 99
})
// TODO: Define emits for:
// - 'update:modelValue' (for v-model compatibility) with number payload
// - 'increment' with no payload
// - 'decrement' with no payload
// - 'limit-reached' with 'min' | 'max' payload
const quantity = ref(props.modelValue)
function increment() {
  if (quantity.value < props.max) {
    quantity.value++
    // TODO: Emit 'update:modelValue' and 'increment'
  } else {
    // TODO: Emit 'limit-reached' with 'max'
  }
}
function decrement() {
  if (quantity.value > props.min) {
    quantity.value--
    // TODO: Emit 'update:modelValue' and 'decrement'
  } else {
    // TODO: Emit 'limit-reached' with 'min'
  }
}
</script>
<template>
  <div class="join">
    <button class="btn join-item" @click="decrement">-</button>
    <span class="btn join-item no-animation">{{ quantity }}</span>
    <button class="btn join-item" @click="increment">+</button>
  </div>
</template>
```
---
### Drill 3 (Combination): Search Input with Props and Events
Combine props (Lesson 6) with events for a search component.
```vue
<!-- SearchInput.vue — Starter Code -->
<script setup lang="ts">
import { ref, watch } from 'vue'
// Props for customization
const props = withDefaults(defineProps<{
  placeholder?: string
  debounceMs?: number
  minLength?: number
}>(), {
  placeholder: 'Search...',
  debounceMs: 300,
  minLength: 2
})
// TODO: Define emits for:
// - 'search' with query string
// - 'clear' with no payload
// - 'focus' with no payload
// - 'blur' with no payload
const query = ref('')
let debounceTimer: ReturnType<typeof setTimeout>
function handleInput() {
  clearTimeout(debounceTimer)
  
  if (query.value.length === 0) {
    // TODO: Emit 'clear'
    return
  }
  
  if (query.value.length >= props.minLength) {
    debounceTimer = setTimeout(() => {
      // TODO: Emit 'search' with query value
    }, props.debounceMs)
  }
}
function clearSearch() {
  query.value = ''
  // TODO: Emit 'clear'
}
</script>
<template>
  <div class="form-control">
    <div class="input-group">
      <input 
        v-model="query"
        type="text"
        :placeholder="placeholder"
        class="input input-bordered"
        @input="handleInput"
      />
      <!-- TODO: Add @focus and @blur handlers that emit events -->
      <button 
        v-if="query.length > 0" 
        class="btn btn-square"
        @click="clearSearch"
      >
        ✕
      </button>
    </div>
  </div>
</template>
```
---
### Drill 4 (Edge Case): Modal with Confirmation
Handle confirm/cancel actions with different payloads.
```vue
<!-- ConfirmModal.vue — Starter Code -->
<script setup lang="ts">
interface ConfirmResult {
  confirmed: boolean
  reason?: string
  timestamp: Date
}
const props = defineProps<{
  title: string
  message: string
  showReasonInput?: boolean
}>()
// TODO: Define emits for:
// - 'confirm' with ConfirmResult payload
// - 'cancel' with ConfirmResult payload
// - 'close' with no payload (for clicking outside)
const reason = ref('')
function handleConfirm() {
  // TODO: Emit 'confirm' with result object
  // Include reason only if showReasonInput is true and reason has value
}
function handleCancel() {
  // TODO: Emit 'cancel' with result object (confirmed: false)
}
function handleBackdropClick() {
  // TODO: Emit 'close'
}
</script>
<template>
  <div class="modal modal-open" @click.self="handleBackdropClick">
    <div class="modal-box">
      <h3 class="font-bold text-lg">{{ title }}</h3>
      <p class="py-4">{{ message }}</p>
      
      <div v-if="showReasonInput" class="form-control">
        <label class="label">
          <span class="label-text">Reason (optional)</span>
        </label>
        <textarea 
          v-model="reason"
          class="textarea textarea-bordered"
          placeholder="Enter reason..."
        />
      </div>
      
      <div class="modal-action">
        <button class="btn" @click="handleCancel">Cancel</button>
        <button class="btn btn-primary" @click="handleConfirm">Confirm</button>
      </div>
    </div>
  </div>
</template>
```
---
## 5. Common Pitfalls & Anti-Patterns
| ❌ Common Mistake | ✅ Correct Approach | Why It Matters |
|:--- |:--- |:--- |
| Forgetting to call emit as a function: `emit 'click'` | Call it properly: `emit('click')` | `emit` is a function, not a keyword; must use parentheses |
| Emitting with wrong event name: `emit('onClick')` but listening with `@on-click` | Be consistent: `emit('click')` and `@click` | Event name in emit should match listener (minus the `@`) |
| Modifying props and emitting: `props.value++; emit('update')` | Emit first, let parent update: `emit('update', newValue)` | Child shouldn't mutate props; parent owns the state |
| Emitting inside computed: `computed(() => { emit('change'); return val })` | Emit in methods or watchers only | Computed should be pure; side effects cause infinite loops |
### Common Debugging Pattern:
```vue
<script setup lang="ts">
const emit = defineEmits<{
  update: [value: string]
}>()
// ❌ WRONG: Event never reaches parent
function handleChange(e: Event) {
  const value = (e.target as HTMLInputElement).value
  // Forgot to emit!
}
// ✅ CORRECT: Emit the event
function handleChange(e: Event) {
  const value = (e.target as HTMLInputElement).value
  emit('update', value) // Don't forget this!
}
// ❌ WRONG: Emitting in computed (side effect)
const processedValue = computed(() => {
  emit('computed', someValue.value) // BAD!
  return someValue.value.toUpperCase()
})
// ✅ CORRECT: Use watch for side effects
watch(someValue, (newVal) => {
  emit('change', newVal)
})
</script>
```
---
## 6. Mini-Project: Star Rating Component
Build a reusable star rating component for the e-commerce scenario.
**File Structure:**
```
src/
├── components/
│   └── StarRating.vue
├── App.vue
└── main.ts
```
**StarRating.vue:**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'
interface RatingPayload {
  rating: number
  previousRating: number | null
  timestamp: Date
}
interface Props {
  modelValue?: number
  maxStars?: number
  readonly?: boolean
  size?: 'sm' | 'md' | 'lg'
  activeColor?: string
  inactiveColor?: string
}
const props = withDefaults(defineProps<Props>(), {
  modelValue: 0,
  maxStars: 5,
  readonly: false,
  size: 'md',
  activeColor: 'text-yellow-400',
  inactiveColor: 'text-gray-300'
})
const emit = defineEmits<{
  'update:modelValue': [rating: number]
  rate: [payload: RatingPayload]
  hover: [star: number | null]
}>()
const hoveredStar = ref<number | null>(null)
const previousRating = ref<number | null>(null)
const displayRating = computed(() => {
  if (hoveredStar.value !== null && !props.readonly) {
    return hoveredStar.value
  }
  return props.modelValue
})
const sizeClasses = computed(() => {
  const sizes = {
    sm: 'w-4 h-4',
    md: 'w-6 h-6',
    lg: 'w-8 h-8'
  }
  return sizes[props.size]
})
function handleClick(star: number) {
  if (props.readonly) return
  
  previousRating.value = props.modelValue
  
  emit('update:modelValue', star)
  
  emit('rate', {
    rating: star,
    previousRating: previousRating.value,
    timestamp: new Date()
  })
}
function handleMouseEnter(star: number) {
  if (props.readonly) return
  hoveredStar.value = star
  emit('hover', star)
}
function handleMouseLeave() {
  if (props.readonly) return
  hoveredStar.value = null
  emit('hover', null)
}
function isStarActive(star: number): boolean {
  return star <= displayRating.value
}
</script>
<template>
  <div 
    class="inline-flex gap-1"
    :class="{ 'cursor-pointer': !readonly, 'cursor-default': readonly }"
    @mouseleave="handleMouseLeave"
  >
    <button
      v-for="star in maxStars"
      :key="star"
      type="button"
      class="transition-colors duration-150 focus:outline-none"
      :class="[
        sizeClasses,
        isStarActive(star) ? activeColor : inactiveColor,
        { 'hover:scale-110': !readonly }
      ]"
      :disabled="readonly"
      @click="handleClick(star)"
      @mouseenter="handleMouseEnter(star)"
    >
      <svg 
        xmlns="http://www.w3.org/2000/svg" 
        viewBox="0 0 24 24" 
        fill="currentColor"
        class="w-full h-full"
      >
        <path 
          fill-rule="evenodd" 
          d="M10.788 3.21c.448-1.077 1.976-1.077 2.424 0l2.082 5.007 5.404.433c1.164.093 1.636 1.545.749 2.305l-4.117 3.527 1.257 5.273c.271 1.136-.964 2.033-1.96 1.425L12 18.354 7.373 21.18c-.996.608-2.231-.29-1.96-1.425l1.257-5.273-4.117-3.527c-.887-.76-.415-2.212.749-2.305l5.404-.433 2.082-5.006z" 
          clip-rule="evenodd" 
        />
      </svg>
    </button>
    
    <span v-if="!readonly" class="ml-2 text-sm text-base-content/60">
      {{ displayRating }} / {{ maxStars }}
    </span>
  </div>
</template>
```
**App.vue:**
```vue
<script setup lang="ts">
import { ref, reactive } from 'vue'
import StarRating from './components/StarRating.vue'
interface RatingPayload {
  rating: number
  previousRating: number | null
  timestamp: Date
}
const productRating = ref(0)
const reviewRating = ref(3)
const ratingHistory = reactive<RatingPayload[]>([])
const currentHover = ref<number | null>(null)
function handleRate(payload: RatingPayload) {
  ratingHistory.push(payload)
  console.log('Rating submitted:', payload)
}
function handleHover(star: number | null) {
  currentHover.value = star
}
function formatTime(date: Date): string {
  return date.toLocaleTimeString()
}
</script>
<template>
  <div class="container mx-auto p-8 space-y-8">
    <h1 class="text-3xl font-bold">Star Rating Component Demo</h1>
    
    <div class="card bg-base-100 shadow-xl">
      <div class="card-body">
        <h2 class="card-title">Interactive Rating</h2>
        <p class="text-base-content/60">Click to rate this product:</p>
        
        <StarRating 
          v-model="productRating"
          size="lg"
          @rate="handleRate"
          @hover="handleHover"
        />
        
        <p class="mt-2">
          Current rating: <strong>{{ productRating }}</strong>
          <span v-if="currentHover !== null" class="text-info">
            (hovering: {{ currentHover }})
          </span>
        </p>
      </div>
    </div>
    
    <div class="card bg-base-100 shadow-xl">
      <div class="card-body">
        <h2 class="card-title">Pre-filled Rating</h2>
        <p class="text-base-content/60">Edit an existing review:</p>
        
        <StarRating 
          v-model="reviewRating"
          size="md"
          active-color="text-orange-500"
          @rate="handleRate"
        />
        
        <p class="mt-2">Review rating: <strong>{{ reviewRating }}</strong></p>
      </div>
    </div>
    
    <div class="card bg-base-100 shadow-xl">
      <div class="card-body">
        <h2 class="card-title">Read-only Display</h2>
        <p class="text-base-content/60">Average customer rating:</p>
        
        <StarRating 
          :model-value="4"
          readonly
          size="sm"
        />
      </div>
    </div>
    
    <div class="card bg-base-100 shadow-xl">
      <div class="card-body">
        <h2 class="card-title">Rating History</h2>
        
        <div v-if="ratingHistory.length === 0" class="text-base-content/60">
          No ratings yet. Click on stars above to rate!
        </div>
        
        <ul v-else class="space-y-2">
          <li 
            v-for="(entry, index) in ratingHistory" 
            :key="index"
            class="flex items-center gap-2"
          >
            <span class="badge badge-primary">{{ entry.rating }} ★</span>
            <span v-if="entry.previousRating !== null" class="text-sm">
              (was {{ entry.previousRating }})
            </span>
            <span class="text-xs text-base-content/50">
              at {{ formatTime(entry.timestamp) }}
            </span>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>
```
**How to Test:**
1. Run `npm create vite@latest events-demo -- --template vue-ts`
2. Install and configure Tailwind CSS with DaisyUI
3. Replace files with the code above
4. Run `npm run dev`
5. Test: Click stars, observe rating changes and history
6. Test: Hover over stars, see preview updates
7. Test: Read-only component doesn't respond to clicks
---
## 7. Implementation Exercises (Progressive)
### Level 1 (Recall): Fill in the Blanks
```vue
<script setup lang="ts">
// Fill in the blanks to create a color picker that emits the selected color
const emit = _____<{
  _____: [color: string]
  clear: _____
}>()
const colors = ['red', 'blue', 'green', 'yellow', 'purple']
function selectColor(color: string) {
  _____('select', _____)
}
function clearSelection() {
  emit('_____')
}
</script>
<template>
  <div class="flex gap-2">
    <button 
      v-for="color in colors" 
      :key="color"
      :style="{ backgroundColor: color }"
      class="w-8 h-8 rounded"
      @click="_____"
    />
    <button @click="clearSelection">Clear</button>
  </div>
</template>
```
---
### Level 2 (Apply): Independent Coding Tasks
**Exercise 2.1: Expandable Panel**
Create a `CollapsiblePanel.vue` component with:
- Props: `title` (string), `initialExpanded` (boolean, default: false)
- Events: `expand` (no payload), `collapse` (no payload), `toggle` (expanded: boolean)
- Clicking the header toggles the panel open/closed
- Emit appropriate events when state changes
**Exercise 2.2: Pagination Controls**
Create a `Pagination.vue` component with:
- Props: `currentPage` (number), `totalPages` (number), `showFirstLast` (boolean, default: true)
- Events: `page-change` (page: number), `next`, `previous`, `first`, `last`
- Buttons for: First, Previous, Next, Last (all emit their respective events)
- Disable buttons appropriately at boundaries
---
### Level 3 (Debug): Fix the Broken Code
```vue
<!-- BrokenCounter.vue — Find and fix 5 bugs -->
<script setup lang="ts">
import { ref, computed } from 'vue'
const props = defineProps<{
  initialValue: number
  step: number
}>()
// Bug 1: Something wrong with defineEmits syntax
const emit = defineEmits({
  change: Number,
  reset: null,
})
const count = ref(props.initialValue)
// Bug 2: Something wrong with this computed
const doubled = computed(() => {
  emit('computed-change', count.value * 2)
  return count.value * 2
})
function increment() {
  count.value += props.step
  // Bug 3: Something missing here
}
function decrement() {
  count.value -= props.step
  emit(change, count.value) // Bug 4: Syntax error
}
function reset() {
  count.value = props.initialValue
  emit('reset')
}
</script>
<template>
  <div>
    <p>Count: {{ count }} (Doubled: {{ doubled }})</p>
    <button @click="increment">+{{ step }}</button>
    <!-- Bug 5: Event listener syntax -->
    <button click="decrement">-{{ step }}</button>
    <button @click="reset">Reset</button>
  </div>
</template>
```
---
### Self-Check: Reference Implementations
**Drill 1 Solution:**
```vue
<script setup lang="ts">
import { ref } from 'vue'
const emit = defineEmits<{
  toggle: [state: boolean]
}>()
const isOn = ref(false)
function toggle() {
  isOn.value = !isOn.value
  emit('toggle', isOn.value)
}
</script>
<template>
  <button 
    @click="toggle" 
    :class="isOn ? 'btn-primary' : 'btn-ghost'"
    class="btn"
  >
    {{ isOn ? 'ON' : 'OFF' }}
  </button>
</template>
```
**Level 1 Solution:**
```vue
<script setup lang="ts">
const emit = defineEmits<{
  select: [color: string]
  clear: []
}>()
const colors = ['red', 'blue', 'green', 'yellow', 'purple']
function selectColor(color: string) {
  emit('select', color)
}
function clearSelection() {
  emit('clear')
}
</script>
<template>
  <div class="flex gap-2">
    <button 
      v-for="color in colors" 
      :key="color"
      :style="{ backgroundColor: color }"
      class="w-8 h-8 rounded"
      @click="selectColor(color)"
    />
    <button @click="clearSelection">Clear</button>
  </div>
</template>
```
**Level 3 Bugs Fixed:**
1. `defineEmits` should use TypeScript generic syntax: `defineEmits<{ change: [value: number]; reset: [] }>()`
2. Computed should NOT have side effects (emit). Remove the emit from computed, use `watch` instead
3. Missing emit after increment: `emit('change', count.value)`
4. `emit(change, ...)` → `emit('change', ...)` (event name must be a string)
5. `click="decrement"` → `@click="decrement"` (missing `@` for event listener)
---
**Reply 'next' for Lesson 8.1 (Theory) — Styling with Tailwind & DaisyUI.**
