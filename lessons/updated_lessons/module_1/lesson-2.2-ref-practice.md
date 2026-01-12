# Lesson 2.2: Reactive Primitives with `ref()` — Practice & Application (Реактивные примитивы с ref())
---
## 1. Lesson Metadata
| Field              | Value                                                           |
| :----------------- | :-------------------------------------------------------------- |
| **Lesson Number**  | 2.2 of 28 (Part 2 of 2)                                         |
| **Topic**          | Reactive Primitives with `ref()` / Реактивные примитивы с ref() |
| **Continues From** | Lesson 2.1                                                      |
| **Duration**       | 30-35 minutes                                                   |
**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Create interactive components with reactive state using `ref()`
2. **Analyze:** Debug common reactivity issues and identify correct `.value` usage
---
## 2. Quick Recap
- **`ref()`** creates a reactive container for primitive values (strings, numbers, booleans)
- Access the value with **`.value`** in `<script setup>`
- Vue **auto-unwraps** refs in templates — no `.value` needed there
- When `.value` changes, Vue automatically updates the DOM
---
## 3. The Variable Frame & Complexity Scale
### Basic Form: Simple Counter
```vue
<script setup lang="ts">
import { ref } from "vue";
const count = ref(0);
const increment = () => {
  count.value++;
};
</script>
<template>
  <button @click="increment">Count: {{ count }}</button>
</template>
```
### With Options: Typed Refs with Multiple Actions
```vue
<script setup lang="ts">
import { ref } from "vue";
const count = ref<number>(0);
const step = ref<number>(1);
const increment = () => {
  count.value += step.value;
};
const decrement = () => {
  count.value -= step.value;
};
const setStep = (newStep: number) => {
  step.value = newStep;
};
</script>
<template>
  <div>
    <p>Count: {{ count }} (Step: {{ step }})</p>
    <button @click="decrement">-{{ step }}</button>
    <button @click="increment">+{{ step }}</button>
    <button @click="setStep(5)">Step = 5</button>
    <button @click="setStep(10)">Step = 10</button>
  </div>
</template>
```
### Advanced Form: Multiple Refs with Conditional Logic
```vue
<script setup lang="ts">
import { ref } from "vue";
const count = ref<number>(0);
const min = ref<number>(0);
const max = ref<number>(10);
const history = ref<number[]>([]);
const increment = () => {
  if (count.value < max.value) {
    history.value.push(count.value);
    count.value++;
  }
};
const decrement = () => {
  if (count.value > min.value) {
    history.value.push(count.value);
    count.value--;
  }
};
const reset = () => {
  history.value.push(count.value);
  count.value = 0;
};
const undo = () => {
  if (history.value.length > 0) {
    count.value = history.value.pop()!;
  }
};
</script>
<template>
  <div>
    <p>Count: {{ count }} (Range: {{ min }}-{{ max }})</p>
    <button @click="decrement" :disabled="count <= min">-</button>
    <button @click="increment" :disabled="count >= max">+</button>
    <button @click="reset">Reset</button>
    <button @click="undo" :disabled="history.length === 0">Undo</button>
    <p>History: {{ history.join(" → ") || "Empty" }}</p>
  </div>
</template>
```
---
## 4. Micro-Implementation Drills
### Drill 1 (Basic): Toggle Switch
Create a component with a boolean ref that toggles between true and false.
```vue
<script setup lang="ts">
import { ref } from "vue";
// TODO: Create a ref called `isOn` initialized to false
// TODO: Create a function `toggle` that flips the value
</script>
<template>
  <div>
    <!-- TODO: Display "ON" or "OFF" based on isOn -->
    <!-- TODO: Create a button that calls toggle -->
  </div>
</template>
```
**Solution:**
```vue
<script setup lang="ts">
import { ref } from "vue";
const isOn = ref<boolean>(false);
const toggle = (): void => {
  isOn.value = !isOn.value;
};
</script>
<template>
  <div>
    <p>Status: {{ isOn ? "ON" : "OFF" }}</p>
    <button @click="toggle">Toggle</button>
  </div>
</template>
```
---
### Drill 2 (Variation): Text Input Binding
Create a component that tracks text input and shows character count.
```vue
<script setup lang="ts">
import { ref } from "vue";
// TODO: Create a ref called `text` initialized to empty string
// TODO: Create a function `clearText` that resets to empty
</script>
<template>
  <div>
    <!-- TODO: Bind input to text using v-model -->
    <!-- TODO: Display "Characters: X" showing text length -->
    <!-- TODO: Button to clear the text -->
  </div>
</template>
```
**Solution:**
```vue
<script setup lang="ts">
import { ref } from "vue";
const text = ref<string>("");
const clearText = (): void => {
  text.value = "";
};
</script>
<template>
  <div>
    <input v-model="text" placeholder="Type something..." />
    <p>Characters: {{ text.length }}</p>
    <button @click="clearText">Clear</button>
  </div>
</template>
```
---
### Drill 3 (Combination): Counter with Step Control
Combine multiple refs to create a counter with configurable step.
```vue
<script setup lang="ts">
import { ref } from "vue";
// TODO: Create `count` ref initialized to 0
// TODO: Create `step` ref initialized to 1
// TODO: Create functions: incrementByStep, decrementByStep, doubleStep
</script>
<template>
  <div>
    <!-- TODO: Display current count -->
    <!-- TODO: Display current step -->
    <!-- TODO: Buttons for +step, -step, and double step -->
  </div>
</template>
```
**Solution:**
```vue
<script setup lang="ts">
import { ref } from "vue";
const count = ref<number>(0);
const step = ref<number>(1);
const incrementByStep = (): void => {
  count.value += step.value;
};
const decrementByStep = (): void => {
  count.value -= step.value;
};
const doubleStep = (): void => {
  step.value *= 2;
};
</script>
<template>
  <div>
    <p>Count: {{ count }}</p>
    <p>Step: {{ step }}</p>
    <button @click="decrementByStep">-{{ step }}</button>
    <button @click="incrementByStep">+{{ step }}</button>
    <button @click="doubleStep">Double Step</button>
  </div>
</template>
```
---
### Drill 4 (Edge Case): Handling Limits and Validation
Create a counter that respects minimum and maximum bounds.
```vue
<script setup lang="ts">
import { ref } from "vue";
// TODO: Create count, min (0), and max (100) refs
// TODO: Create safeIncrement that only increments if under max
// TODO: Create safeDecrement that only decrements if above min
// TODO: Create a function to check if at min/max limit
</script>
<template>
  <div>
    <!-- TODO: Display count with min/max info -->
    <!-- TODO: Disable buttons when at limits -->
    <!-- TODO: Show warning when at a limit -->
  </div>
</template>
```
**Solution:**
```vue
<script setup lang="ts">
import { ref } from "vue";
const count = ref<number>(50);
const min = ref<number>(0);
const max = ref<number>(100);
const isAtMin = (): boolean => count.value <= min.value;
const isAtMax = (): boolean => count.value >= max.value;
const safeIncrement = (): void => {
  if (!isAtMax()) {
    count.value++;
  }
};
const safeDecrement = (): void => {
  if (!isAtMin()) {
    count.value--;
  }
};
const getLimitWarning = (): string => {
  if (isAtMin()) return "⚠️ Minimum reached!";
  if (isAtMax()) return "⚠️ Maximum reached!";
  return "";
};
</script>
<template>
  <div>
    <p>Count: {{ count }} (Range: {{ min }} - {{ max }})</p>
    <button @click="safeDecrement" :disabled="isAtMin()">-1</button>
    <button @click="safeIncrement" :disabled="isAtMax()">+1</button>
    <p v-if="getLimitWarning()">{{ getLimitWarning() }}</p>
  </div>
</template>
```
---
## 5. Common Pitfalls & Anti-Patterns
| ❌ Common Mistake                        | ✅ Correct Approach                      | Why It Matters                                      |
| :--------------------------------------- | :--------------------------------------- | :-------------------------------------------------- |
| Forgetting `.value` in script            | Always use `.value` to read/write        | Without `.value`, you're working with the ref object, not the value |
| Using `.value` in template               | Just use the ref name directly           | Vue auto-unwraps; `.value` in template is redundant |
| Reassigning the entire ref               | Mutate `.value` instead                  | Reassigning breaks reactivity connection            |
### Example of Each Mistake
**Mistake 1: Forgetting `.value` in Script**
```vue
<script setup lang="ts">
import { ref } from "vue";
const count = ref(0);
const increment = () => {
  // ❌ WRONG: This doesn't change the reactive value
  count++;  // Error: Cannot apply ++ to Ref<number>
  
  // ✅ CORRECT: Access .value
  count.value++;
};
</script>
```
**Mistake 2: Using `.value` in Template**
```vue
<script setup lang="ts">
import { ref } from "vue";
const count = ref(0);
</script>
<template>
  <!-- ❌ WRONG: Unnecessary .value (works but redundant) -->
  <p>{{ count.value }}</p>
  
  <!-- ✅ CORRECT: Vue auto-unwraps refs -->
  <p>{{ count }}</p>
</template>
```
**Mistake 3: Reassigning the Ref Object**
```vue
<script setup lang="ts">
import { ref } from "vue";
let count = ref(0);  // Using let instead of const
const reset = () => {
  // ❌ WRONG: Creates new ref, breaks template binding
  count = ref(0);
  
  // ✅ CORRECT: Mutate the existing ref's value
  count.value = 0;
};
</script>
```
---
## 6. Mini-Project: Click Counter Dashboard
Build a fitness step counter dashboard with increment, decrement, and goal tracking.
### File Structure
```
src/
├── components/
│   └── StepCounter.vue
├── App.vue
└── main.ts
```
### StepCounter.vue
```vue
<script setup lang="ts">
// ============================================
// IMPORTS
// ============================================
import { ref } from "vue";
// ============================================
// REACTIVE STATE
// ============================================
// Current step count - starts at 0
const steps = ref<number>(0);
// Daily goal - user can adjust this
const goal = ref<number>(10000);
// Step increment amount for quick-add buttons
const quickAddAmount = ref<number>(100);
// Whether the goal has been celebrated (to show message once)
const hasReachedGoal = ref<boolean>(false);
// ============================================
// COMPUTED-LIKE HELPERS
// ============================================
// Calculate progress percentage (capped at 100)
const getProgress = (): number => {
  const percentage = (steps.value / goal.value) * 100;
  return Math.min(percentage, 100);
};
// Format step count with commas (e.g., 10,000)
const formatNumber = (num: number): string => {
  return num.toLocaleString();
};
// Get progress bar color based on percentage
const getProgressColor = (): string => {
  const progress = getProgress();
  if (progress >= 100) return "progress-success";
  if (progress >= 75) return "progress-info";
  if (progress >= 50) return "progress-warning";
  return "progress-error";
};
// ============================================
// ACTIONS
// ============================================
// Add steps with validation
const addSteps = (amount: number): void => {
  steps.value += amount;
  
  // Check if goal reached for the first time
  if (steps.value >= goal.value && !hasReachedGoal.value) {
    hasReachedGoal.value = true;
  }
};
// Remove steps (can't go below 0)
const removeSteps = (amount: number): void => {
  steps.value = Math.max(0, steps.value - amount);
  
  // Reset goal celebration if we drop below goal
  if (steps.value < goal.value) {
    hasReachedGoal.value = false;
  }
};
// Reset everything
const resetCounter = (): void => {
  steps.value = 0;
  hasReachedGoal.value = false;
};
// Update the goal
const setGoal = (newGoal: number): void => {
  goal.value = Math.max(1000, newGoal); // Minimum goal of 1000
  
  // Re-evaluate goal reached status
  hasReachedGoal.value = steps.value >= goal.value;
};
</script>
<template>
  <!-- Main container with card styling -->
  <div class="card bg-base-100 shadow-xl w-full max-w-md mx-auto">
    <div class="card-body">
      <!-- Header -->
      <h2 class="card-title justify-center text-2xl">
        🏃 Step Counter
      </h2>
      <!-- Step Count Display -->
      <div class="text-center my-6">
        <p class="text-5xl font-bold text-primary">
          {{ formatNumber(steps) }}
        </p>
        <p class="text-base-content/60">
          of {{ formatNumber(goal) }} steps
        </p>
      </div>
      <!-- Progress Bar -->
      <div class="w-full mb-4">
        <progress
          class="progress w-full"
          :class="getProgressColor()"
          :value="getProgress()"
          max="100"
        ></progress>
        <p class="text-center text-sm mt-1">
          {{ getProgress().toFixed(1) }}% complete
        </p>
      </div>
      <!-- Goal Reached Celebration -->
      <div
        v-if="hasReachedGoal"
        class="alert alert-success mb-4"
      >
        <span>🎉 Congratulations! You've reached your daily goal!</span>
      </div>
      <!-- Quick Add Buttons -->
      <div class="flex justify-center gap-2 mb-4">
        <button
          class="btn btn-primary btn-sm"
          @click="addSteps(quickAddAmount)"
        >
          +{{ quickAddAmount }}
        </button>
        <button
          class="btn btn-primary"
          @click="addSteps(quickAddAmount * 10)"
        >
          +{{ quickAddAmount * 10 }}
        </button>
        <button
          class="btn btn-secondary btn-sm"
          @click="removeSteps(quickAddAmount)"
        >
          -{{ quickAddAmount }}
        </button>
      </div>
      <!-- Fine Control -->
      <div class="flex justify-center gap-2 mb-4">
        <button class="btn btn-outline btn-sm" @click="addSteps(1)">
          +1
        </button>
        <button class="btn btn-outline btn-sm" @click="addSteps(10)">
          +10
        </button>
        <button class="btn btn-outline btn-sm" @click="removeSteps(1)">
          -1
        </button>
        <button class="btn btn-outline btn-sm" @click="removeSteps(10)">
          -10
        </button>
      </div>
      <!-- Goal Adjustment -->
      <div class="form-control mb-4">
        <label class="label">
          <span class="label-text">Adjust Goal</span>
        </label>
        <div class="flex gap-2">
          <button
            class="btn btn-ghost btn-sm"
            @click="setGoal(goal - 1000)"
          >
            -1000
          </button>
          <input
            type="number"
            v-model.number="goal"
            class="input input-bordered flex-1 text-center"
            min="1000"
            step="1000"
          />
          <button
            class="btn btn-ghost btn-sm"
            @click="setGoal(goal + 1000)"
          >
            +1000
          </button>
        </div>
      </div>
      <!-- Reset Button -->
      <div class="card-actions justify-center">
        <button class="btn btn-error btn-outline" @click="resetCounter">
          Reset Counter
        </button>
      </div>
    </div>
  </div>
</template>
<style scoped>
/* Custom transition for progress bar */
.progress {
  transition: all 0.3s ease;
}
</style>
```
### App.vue
```vue
<script setup lang="ts">
import StepCounter from "./components/StepCounter.vue";
</script>
<template>
  <div class="min-h-screen bg-base-200 py-10 px-4">
    <h1 class="text-3xl font-bold text-center mb-8">
      Fitness Dashboard
    </h1>
    <StepCounter />
  </div>
</template>
```
### How to Test
1. Run the dev server: `npm run dev`
2. Open the browser
3. Verify these interactions:
   - Click "+100" → steps increase by 100
   - Click "-100" → steps decrease by 100 (not below 0)
   - Progress bar updates in real-time
   - When steps ≥ goal, celebration message appears
   - Adjust goal using buttons or input
   - Reset button clears steps to 0
---
## 7. Implementation Exercises (Progressive)
### Level 1 (Recall): Fill in the Blanks
Complete the missing parts:
```vue
<script setup lang="ts">
import { _____ } from "vue";
const temperature = _____<number>(20);
const increaseTemp = (): void => {
  temperature._____ += 5;
};
const decreaseTemp = (): void => {
  temperature._____ -= 5;
};
</script>
<template>
  <div>
    <p>Temperature: {{ _____ }}°C</p>
    <button @click="_____">+5°C</button>
    <button @click="decreaseTemp">-5°C</button>
  </div>
</template>
```
**Answer:**
```vue
<script setup lang="ts">
import { ref } from "vue";
const temperature = ref<number>(20);
const increaseTemp = (): void => {
  temperature.value += 5;
};
const decreaseTemp = (): void => {
  temperature.value -= 5;
};
</script>
<template>
  <div>
    <p>Temperature: {{ temperature }}°C</p>
    <button @click="increaseTemp">+5°C</button>
    <button @click="decreaseTemp">-5°C</button>
  </div>
</template>
```
---
### Level 2 (Apply): Independent Coding Tasks
**Exercise 2.1: Password Visibility Toggle**
Create a component with:
- A password input field
- A ref `showPassword` (boolean) 
- A button that toggles between showing/hiding the password
- Input type should change between "password" and "text"
**Exercise 2.2: Simple Timer**
Create a component with:
- A ref `seconds` starting at 0
- A ref `isRunning` boolean
- Start button that begins incrementing seconds every second
- Stop button that pauses the timer
- Reset button that sets seconds to 0 and stops
Hint: Use `setInterval` and `clearInterval`
---
### Level 3 (Debug): Find and Fix the Bugs
This component has **4 bugs**. Find and fix them:
```vue
<script setup lang="ts">
import { ref } from "vue";
let likes = ref(0);
const addLike = () => {
  likes++;
};
const removeLike = () => {
  if (likes > 0) {
    likes.value--;
  }
};
const resetLikes = () => {
  likes = ref(0);
};
</script>
<template>
  <div>
    <p>Likes: {{ likes.value }}</p>
    <button @click="addLike">👍</button>
    <button @click="removeLike">👎</button>
    <button @click="resetLikes">Reset</button>
  </div>
</template>
```
**Bugs:**
1. `let likes = ref(0)` should be `const likes = ref(0)` — refs should be const
2. `likes++` should be `likes.value++` — forgot `.value`
3. `if (likes > 0)` should be `if (likes.value > 0)` — forgot `.value` in condition
4. `likes = ref(0)` should be `likes.value = 0` — reassigning breaks reactivity
5. `{{ likes.value }}` should be `{{ likes }}` — `.value` auto-unwrapped in template
**Corrected Code:**
```vue
<script setup lang="ts">
import { ref } from "vue";
const likes = ref(0);
const addLike = () => {
  likes.value++;
};
const removeLike = () => {
  if (likes.value > 0) {
    likes.value--;
  }
};
const resetLikes = () => {
  likes.value = 0;
};
</script>
<template>
  <div>
    <p>Likes: {{ likes }}</p>
    <button @click="addLike">👍</button>
    <button @click="removeLike">👎</button>
    <button @click="resetLikes">Reset</button>
  </div>
</template>
```
---
**Reply 'next' for Lesson 3.1 (Theory).**
