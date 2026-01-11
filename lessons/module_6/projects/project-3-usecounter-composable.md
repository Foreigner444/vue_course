# Guided Project 3: useCounter Composable
---
## Project Overview
| Field | Value |
|:--- |:--- |
| **Module** | Module 6: Reusability & Abstraction |
| **Project** | useCounter Composable |
| **Real-World Scenario** | A dashboard has multiple counters with shared increment/decrement/reset logic |
| **Difficulty** | Beginner-Intermediate |
| **Estimated Time** | 30-40 minutes |
### What You'll Build
A versatile counter composable that can be configured with min/max bounds, custom step values, and optional callbacks. You'll use it to build a product quantity selector, a rating widget, and a game score tracker—demonstrating how one composable can power many different UI patterns.
### Patterns You'll Practice
1. Basic composable structure with `ref()` and `computed()`
2. Configuration through options object
3. Boundary validation (min/max)
4. Optional callback functions
5. Multiple return values (state + actions)
6. Reusing the same composable multiple times
7. TypeScript interfaces for options
---
## Component Structure
```
src/
├── composables/
│   └── useCounter.ts           # The counter composable
├── components/
│   ├── QuantitySelector.vue    # E-commerce quantity picker
│   ├── RatingWidget.vue        # Star rating component
│   └── ScoreTracker.vue        # Game score with history
├── App.vue                     # Demo page
└── main.ts
```
---
## Step-by-Step Guide
### Step 1: Project Setup
```bash
npm create vite@latest counter-demo -- --template vue-ts
cd counter-demo
npm install
npm install -D tailwindcss postcss autoprefixer daisyui@latest
npx tailwindcss init -p
```
Configure `tailwind.config.js`:
```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: ["./index.html", "./src/**/*.{vue,js,ts,jsx,tsx}"],
  theme: { extend: {} },
  plugins: [require("daisyui")],
}
```
Update `src/style.css`:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```
---
### Step 2: Build the useCounter Composable
```typescript
// src/composables/useCounter.ts
import { ref, computed, type Ref, type ComputedRef } from 'vue'
// ─────────────────────────────────────────────────────────────
// OPTIONS INTERFACE: All configuration options for the counter
// ─────────────────────────────────────────────────────────────
export interface UseCounterOptions {
  min?: number
  max?: number
  step?: number
  onChange?: (newValue: number, oldValue: number) => void
  onMin?: () => void
  onMax?: () => void
}
// ─────────────────────────────────────────────────────────────
// RETURN TYPE: Everything the composable exposes
// ─────────────────────────────────────────────────────────────
export interface UseCounterReturn {
  count: Ref<number>
  canIncrement: ComputedRef<boolean>
  canDecrement: ComputedRef<boolean>
  isAtMin: ComputedRef<boolean>
  isAtMax: ComputedRef<boolean>
  percentage: ComputedRef<number>
  increment: () => void
  decrement: () => void
  reset: () => void
  set: (value: number) => void
}
// ─────────────────────────────────────────────────────────────
// MAIN COMPOSABLE: useCounter
// 
// A flexible counter with min/max bounds, custom step, and callbacks.
// Each call creates an independent instance.
// ─────────────────────────────────────────────────────────────
export function useCounter(
  initialValue: number = 0,
  options: UseCounterOptions = {}
): UseCounterReturn {
  
  // ─────────────────────────────────────────────────────────────
  // OPTIONS: Destructure with defaults
  // ─────────────────────────────────────────────────────────────
  const {
    min = -Infinity,
    max = Infinity,
    step = 1,
    onChange,
    onMin,
    onMax
  } = options
  // ─────────────────────────────────────────────────────────────
  // VALIDATION: Ensure initial value is within bounds
  // ─────────────────────────────────────────────────────────────
  const clampedInitial = Math.min(Math.max(initialValue, min), max)
  // ─────────────────────────────────────────────────────────────
  // STATE: The counter value
  // ─────────────────────────────────────────────────────────────
  const count = ref(clampedInitial)
  // ─────────────────────────────────────────────────────────────
  // COMPUTED: Boundary checks
  // ─────────────────────────────────────────────────────────────
  const isAtMin = computed(() => count.value <= min)
  const isAtMax = computed(() => count.value >= max)
  const canIncrement = computed(() => count.value + step <= max)
  const canDecrement = computed(() => count.value - step >= min)
  // ─────────────────────────────────────────────────────────────
  // COMPUTED: Percentage between min and max
  // Useful for progress bars, sliders, etc.
  // ─────────────────────────────────────────────────────────────
  const percentage = computed(() => {
    if (min === -Infinity || max === Infinity) {
      return 0
    }
    const range = max - min
    if (range === 0) return 100
    return ((count.value - min) / range) * 100
  })
  // ─────────────────────────────────────────────────────────────
  // HELPER: Clamp value to bounds and trigger callbacks
  // ─────────────────────────────────────────────────────────────
  function setValue(newValue: number) {
    const oldValue = count.value
    const clampedValue = Math.min(Math.max(newValue, min), max)
    
    if (clampedValue === oldValue) {
      return
    }
    
    count.value = clampedValue
    
    onChange?.(clampedValue, oldValue)
    
    if (clampedValue === min) {
      onMin?.()
    }
    
    if (clampedValue === max) {
      onMax?.()
    }
  }
  // ─────────────────────────────────────────────────────────────
  // ACTION: Increment by step
  // ─────────────────────────────────────────────────────────────
  function increment() {
    setValue(count.value + step)
  }
  // ─────────────────────────────────────────────────────────────
  // ACTION: Decrement by step
  // ─────────────────────────────────────────────────────────────
  function decrement() {
    setValue(count.value - step)
  }
  // ─────────────────────────────────────────────────────────────
  // ACTION: Reset to initial value
  // ─────────────────────────────────────────────────────────────
  function reset() {
    setValue(clampedInitial)
  }
  // ─────────────────────────────────────────────────────────────
  // ACTION: Set to specific value
  // ─────────────────────────────────────────────────────────────
  function set(value: number) {
    setValue(value)
  }
  // ─────────────────────────────────────────────────────────────
  // RETURN: Public API
  // ─────────────────────────────────────────────────────────────
  return {
    count,
    canIncrement,
    canDecrement,
    isAtMin,
    isAtMax,
    percentage,
    increment,
    decrement,
    reset,
    set
  }
}
```
---
### Step 3: Create the Quantity Selector
```vue
<!-- src/components/QuantitySelector.vue -->
<script setup lang="ts">
import { useCounter } from '@/composables/useCounter'
// ─────────────────────────────────────────────────────────────
// PROPS
// ─────────────────────────────────────────────────────────────
const props = withDefaults(defineProps<{
  productName?: string
  price?: number
  maxStock?: number
}>(), {
  productName: 'Product',
  price: 29.99,
  maxStock: 10
})
// ─────────────────────────────────────────────────────────────
// COUNTER: Configured for e-commerce quantity selection
// ─────────────────────────────────────────────────────────────
const {
  count: quantity,
  canIncrement,
  canDecrement,
  isAtMax,
  increment,
  decrement,
  reset
} = useCounter(1, {
  min: 1,
  max: props.maxStock,
  step: 1,
  onMax: () => {
    console.log('Maximum stock reached!')
  }
})
// ─────────────────────────────────────────────────────────────
// COMPUTED: Total price
// ─────────────────────────────────────────────────────────────
import { computed } from 'vue'
const totalPrice = computed(() => {
  return (props.price * quantity.value).toFixed(2)
})
</script>
<template>
  <div class="card bg-base-200 shadow-xl">
    <div class="card-body">
      <h2 class="card-title">🛒 {{ productName }}</h2>
      
      <div class="flex items-center justify-between">
        <span class="text-lg">${{ price.toFixed(2) }} each</span>
        
        <!-- Quantity Controls -->
        <div class="join">
          <button
            class="btn btn-square join-item"
            :disabled="!canDecrement"
            @click="decrement"
          >
            −
          </button>
          
          <div class="btn btn-square join-item no-animation pointer-events-none">
            {{ quantity }}
          </div>
          
          <button
            class="btn btn-square join-item"
            :disabled="!canIncrement"
            @click="increment"
          >
            +
          </button>
        </div>
      </div>
      
      <!-- Stock Warning -->
      <div v-if="isAtMax" class="alert alert-warning mt-2">
        <span>⚠️ Maximum stock reached ({{ maxStock }} available)</span>
      </div>
      
      <!-- Total -->
      <div class="divider"></div>
      
      <div class="flex justify-between items-center">
        <span class="text-base-content/70">Total:</span>
        <span class="text-2xl font-bold">${{ totalPrice }}</span>
      </div>
      
      <!-- Actions -->
      <div class="card-actions justify-end mt-4">
        <button class="btn btn-ghost btn-sm" @click="reset">
          Reset
        </button>
        <button class="btn btn-primary">
          Add to Cart
        </button>
      </div>
    </div>
  </div>
</template>
```
---
### Step 4: Create the Rating Widget
```vue
<!-- src/components/RatingWidget.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import { useCounter } from '@/composables/useCounter'
// ─────────────────────────────────────────────────────────────
// PROPS & EMITS
// ─────────────────────────────────────────────────────────────
const props = withDefaults(defineProps<{
  maxStars?: number
  initialRating?: number
  readonly?: boolean
}>(), {
  maxStars: 5,
  initialRating: 0,
  readonly: false
})
const emit = defineEmits<{
  change: [rating: number]
}>()
// ─────────────────────────────────────────────────────────────
// COUNTER: Configured as a rating (0 to maxStars)
// ─────────────────────────────────────────────────────────────
const {
  count: rating,
  percentage,
  set: setRating
} = useCounter(props.initialRating, {
  min: 0,
  max: props.maxStars,
  step: 1,
  onChange: (newValue) => {
    emit('change', newValue)
  }
})
// ─────────────────────────────────────────────────────────────
// HOVER STATE: For preview effect
// ─────────────────────────────────────────────────────────────
const hoverRating = ref<number | null>(null)
// ─────────────────────────────────────────────────────────────
// COMPUTED: Which rating to display (hover or actual)
// ─────────────────────────────────────────────────────────────
import { computed } from 'vue'
const displayRating = computed(() => {
  return hoverRating.value !== null ? hoverRating.value : rating.value
})
// ─────────────────────────────────────────────────────────────
// HANDLERS
// ─────────────────────────────────────────────────────────────
function handleClick(star: number) {
  if (props.readonly) return
  
  if (rating.value === star) {
    setRating(0)
  } else {
    setRating(star)
  }
}
function handleMouseEnter(star: number) {
  if (!props.readonly) {
    hoverRating.value = star
  }
}
function handleMouseLeave() {
  hoverRating.value = null
}
// ─────────────────────────────────────────────────────────────
// HELPER: Get rating text
// ─────────────────────────────────────────────────────────────
const ratingLabels = ['Not rated', 'Poor', 'Fair', 'Good', 'Very Good', 'Excellent']
const ratingLabel = computed(() => {
  return ratingLabels[displayRating.value] || ratingLabels[0]
})
</script>
<template>
  <div class="card bg-base-200 shadow-xl">
    <div class="card-body items-center text-center">
      <h2 class="card-title">⭐ Rate This</h2>
      
      <!-- Stars -->
      <div 
        class="flex gap-1 my-4"
        @mouseleave="handleMouseLeave"
      >
        <button
          v-for="star in maxStars"
          :key="star"
          class="text-3xl transition-transform hover:scale-110 focus:outline-none"
          :class="[
            star <= displayRating ? 'text-warning' : 'text-base-300',
            readonly ? 'cursor-default' : 'cursor-pointer'
          ]"
          @click="handleClick(star)"
          @mouseenter="handleMouseEnter(star)"
          :disabled="readonly"
        >
          {{ star <= displayRating ? '★' : '☆' }}
        </button>
      </div>
      
      <!-- Rating Label -->
      <p class="text-lg font-medium">{{ ratingLabel }}</p>
      
      <!-- Progress Bar -->
      <div class="w-full mt-2">
        <progress 
          class="progress progress-warning w-full" 
          :value="percentage" 
          max="100"
        ></progress>
        <p class="text-xs text-base-content/60 mt-1">
          {{ rating }}/{{ maxStars }} stars ({{ percentage.toFixed(0) }}%)
        </p>
      </div>
      
      <!-- Readonly Badge -->
      <div v-if="readonly" class="badge badge-neutral mt-2">
        Read-only
      </div>
    </div>
  </div>
</template>
```
---
### Step 5: Create the Score Tracker
```vue
<!-- src/components/ScoreTracker.vue -->
<script setup lang="ts">
import { ref, computed } from 'vue'
import { useCounter } from '@/composables/useCounter'
// ─────────────────────────────────────────────────────────────
// PROPS
// ─────────────────────────────────────────────────────────────
const props = withDefaults(defineProps<{
  playerName?: string
  pointsPerScore?: number
}>(), {
  playerName: 'Player 1',
  pointsPerScore: 10
})
// ─────────────────────────────────────────────────────────────
// SCORE HISTORY: Track changes over time
// ─────────────────────────────────────────────────────────────
interface ScoreEntry {
  value: number
  change: number
  timestamp: number
}
const history = ref<ScoreEntry[]>([])
// ─────────────────────────────────────────────────────────────
// COUNTER: Configured for game scoring
// ─────────────────────────────────────────────────────────────
const {
  count: score,
  increment: addScore,
  decrement: subtractScore,
  reset,
  set: setScore
} = useCounter(0, {
  min: 0,
  step: props.pointsPerScore,
  onChange: (newValue, oldValue) => {
    history.value.unshift({
      value: newValue,
      change: newValue - oldValue,
      timestamp: Date.now()
    })
    
    if (history.value.length > 10) {
      history.value = history.value.slice(0, 10)
    }
  }
})
// ─────────────────────────────────────────────────────────────
// COMPUTED: Statistics
// ─────────────────────────────────────────────────────────────
const stats = computed(() => {
  if (history.value.length === 0) {
    return { max: 0, average: 0, changes: 0 }
  }
  
  const values = history.value.map(h => h.value)
  const max = Math.max(...values, score.value)
  const average = values.reduce((a, b) => a + b, 0) / values.length
  
  return {
    max,
    average: average.toFixed(1),
    changes: history.value.length
  }
})
// ─────────────────────────────────────────────────────────────
// ACTIONS
// ─────────────────────────────────────────────────────────────
function handleReset() {
  reset()
  history.value = []
}
function handleBonus() {
  setScore(score.value + props.pointsPerScore * 5)
}
// ─────────────────────────────────────────────────────────────
// HELPER: Format timestamp
// ─────────────────────────────────────────────────────────────
function formatTime(timestamp: number): string {
  return new Date(timestamp).toLocaleTimeString('en-US', {
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit'
  })
}
</script>
<template>
  <div class="card bg-base-200 shadow-xl">
    <div class="card-body">
      <h2 class="card-title">🎮 {{ playerName }}</h2>
      
      <!-- Score Display -->
      <div class="text-center py-4">
        <div class="text-6xl font-bold tabular-nums">
          {{ score.toLocaleString() }}
        </div>
        <div class="text-sm text-base-content/60 mt-1">
          points
        </div>
      </div>
      
      <!-- Score Controls -->
      <div class="flex gap-2 justify-center">
        <button 
          class="btn btn-error"
          @click="subtractScore"
          :disabled="score === 0"
        >
          -{{ pointsPerScore }}
        </button>
        <button 
          class="btn btn-success"
          @click="addScore"
        >
          +{{ pointsPerScore }}
        </button>
        <button 
          class="btn btn-warning"
          @click="handleBonus"
        >
          🎁 Bonus
        </button>
      </div>
      
      <!-- Stats -->
      <div class="stats stats-vertical lg:stats-horizontal shadow mt-4">
        <div class="stat place-items-center">
          <div class="stat-title">High Score</div>
          <div class="stat-value text-primary">{{ stats.max }}</div>
        </div>
        <div class="stat place-items-center">
          <div class="stat-title">Average</div>
          <div class="stat-value text-secondary">{{ stats.average }}</div>
        </div>
        <div class="stat place-items-center">
          <div class="stat-title">Changes</div>
          <div class="stat-value">{{ stats.changes }}</div>
        </div>
      </div>
      
      <!-- History -->
      <div v-if="history.length > 0" class="mt-4">
        <h3 class="font-semibold mb-2">Recent History</h3>
        <div class="max-h-32 overflow-y-auto space-y-1">
          <div 
            v-for="(entry, index) in history" 
            :key="index"
            class="flex justify-between text-sm bg-base-100 px-2 py-1 rounded"
          >
            <span :class="entry.change > 0 ? 'text-success' : 'text-error'">
              {{ entry.change > 0 ? '+' : '' }}{{ entry.change }}
            </span>
            <span class="text-base-content/60">
              → {{ entry.value }}
            </span>
            <span class="text-base-content/40 text-xs">
              {{ formatTime(entry.timestamp) }}
            </span>
          </div>
        </div>
      </div>
      
      <!-- Reset -->
      <div class="card-actions justify-end mt-4">
        <button 
          class="btn btn-ghost btn-sm"
          @click="handleReset"
        >
          🔄 Reset All
        </button>
      </div>
    </div>
  </div>
</template>
```
---
### Step 6: Create App.vue
```vue
<!-- src/App.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import QuantitySelector from '@/components/QuantitySelector.vue'
import RatingWidget from '@/components/RatingWidget.vue'
import ScoreTracker from '@/components/ScoreTracker.vue'
const selectedRating = ref(0)
function handleRatingChange(rating: number) {
  selectedRating.value = rating
  console.log('Rating changed to:', rating)
}
</script>
<template>
  <div class="min-h-screen bg-base-100 py-8">
    <div class="container mx-auto px-4">
      <!-- Header -->
      <div class="text-center mb-8">
        <h1 class="text-3xl font-bold">useCounter Composable Demo</h1>
        <p class="text-base-content/60 mt-2">
          One composable, three different use cases
        </p>
      </div>
      
      <!-- Demo Grid -->
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 max-w-6xl mx-auto">
        
        <!-- Quantity Selector Demo -->
        <QuantitySelector
          product-name="Wireless Headphones"
          :price="79.99"
          :max-stock="5"
        />
        
        <!-- Rating Widget Demo -->
        <RatingWidget
          :max-stars="5"
          :initial-rating="3"
          @change="handleRatingChange"
        />
        
        <!-- Score Tracker Demo -->
        <ScoreTracker
          player-name="Player 1"
          :points-per-score="100"
        />
        
        <!-- Additional Examples -->
        <QuantitySelector
          product-name="USB-C Cable"
          :price="12.99"
          :max-stock="20"
        />
        
        <RatingWidget
          :max-stars="10"
          :initial-rating="7"
          readonly
        />
        
        <ScoreTracker
          player-name="Player 2"
          :points-per-score="50"
        />
      </div>
      
      <!-- Info Section -->
      <div class="mt-12 text-center">
        <div class="card bg-base-200 max-w-2xl mx-auto">
          <div class="card-body">
            <h2 class="card-title justify-center">
              💡 How It Works
            </h2>
            <p class="text-left">
              All three components use the same <code class="bg-base-300 px-1 rounded">useCounter</code> 
              composable with different configurations:
            </p>
            <ul class="text-left list-disc list-inside space-y-2 mt-4">
              <li>
                <strong>Quantity Selector:</strong> 
                <code class="text-xs">min: 1, max: stock, step: 1</code>
              </li>
              <li>
                <strong>Rating Widget:</strong> 
                <code class="text-xs">min: 0, max: stars, step: 1</code>
              </li>
              <li>
                <strong>Score Tracker:</strong> 
                <code class="text-xs">min: 0, step: points, onChange callback</code>
              </li>
            </ul>
            <p class="text-left mt-4 text-base-content/70">
              Each component instance has its own independent state, but they all 
              share the same reusable logic from the composable.
            </p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
```
---
## Key Takeaways
1. **One composable, many uses:** The same `useCounter` powers a quantity selector, a star rating, and a game score tracker.
2. **Configuration over code:** Instead of writing three different counter implementations, we configure one flexible composable.
3. **Independent instances:** Each component gets its own state when it calls `useCounter()`.
4. **TypeScript for clarity:** The `UseCounterOptions` and `UseCounterReturn` interfaces document exactly what the composable accepts and returns.
5. **Callbacks for side effects:** The `onChange`, `onMin`, and `onMax` callbacks let components react to counter changes without polling.
---
## Stretch Goals
1. **Add animation:** Animate the score changes using CSS transitions or a library like `@vueuse/motion`
2. **Add keyboard support:** Let users increment/decrement with arrow keys when focused
3. **Add sound effects:** Play a sound when incrementing/decrementing (use `useSound` composable pattern)
4. **Add undo/redo:** Track history in the composable and add `undo()` and `redo()` actions
---
## Self-Assessment Checklist
- [ ] The `useCounter` composable accepts configuration options
- [ ] Min/max boundaries are enforced correctly
- [ ] The `step` option controls increment/decrement amount
- [ ] Callbacks (`onChange`, `onMin`, `onMax`) fire at the right times
- [ ] `canIncrement` and `canDecrement` disable buttons properly
- [ ] `percentage` computed works for bounded counters
- [ ] Each component instance has independent state
- [ ] TypeScript types are defined for options and return value
- [ ] All three demo components work correctly
- [ ] The composable can be used in additional contexts easily
