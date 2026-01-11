# Lesson 2.2: Reactive Primitives with `ref()` — Practice & Application (Реактивные примитивы с ref())

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 2.2 of 25 (Part 2 of 2) |
| **Topic** | Reactive Primitives with `ref()` / Реактивные примитивы с ref() |
| **Continues From** | Lesson 2.1 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Create interactive components with multiple reactive refs that respond to user input
2. **Analyze:** Debug common ref-related issues and identify the correct usage of `.value`

---

## 2. Quick Recap

- **`ref()`** creates a reactive wrapper around a primitive value (string, number, boolean)
- Access and modify the value using **`.value`** in the script section
- In templates, refs are **auto-unwrapped** — write `{{ count }}` not `{{ count.value }}`
- Vue automatically re-renders the template when a ref's value changes
- Import `ref` from Vue: `import { ref } from 'vue'`

---

## 3. The Variable Frame & Complexity Scale

**Basic Form:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const count = ref(0)
</script>

<template>
  <p>{{ count }}</p>
</template>
```

**With Options (Explicit Typing):**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const count = ref<number>(0)
const message = ref<string>('')
const isLoading = ref<boolean>(false)

const increment = (): void => {
  count.value++
}

const updateMessage = (newMsg: string): void => {
  message.value = newMsg
  isLoading.value = true
}
</script>

<template>
  <div>
    <p>Count: {{ count }}</p>
    <p>Message: {{ message || 'No message' }}</p>
    <p>Loading: {{ isLoading ? 'Yes' : 'No' }}</p>
    <button @click="increment">+1</button>
  </div>
</template>
```

**Advanced Form (Multiple Refs with Validation):**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const username = ref<string>('')
const password = ref<string>('')
const errorMessage = ref<string>('')
const isSubmitting = ref<boolean>(false)
const attempts = ref<number>(0)

const MAX_ATTEMPTS = 3

const validateForm = (): boolean => {
  if (username.value.length < 3) {
    errorMessage.value = 'Username must be at least 3 characters'
    return false
  }
  if (password.value.length < 8) {
    errorMessage.value = 'Password must be at least 8 characters'
    return false
  }
  errorMessage.value = ''
  return true
}

const submitForm = async (): Promise<void> => {
  if (attempts.value >= MAX_ATTEMPTS) {
    errorMessage.value = 'Too many attempts. Please try again later.'
    return
  }
  
  if (!validateForm()) {
    attempts.value++
    return
  }
  
  isSubmitting.value = true
  // Simulate API call
  await new Promise(resolve => setTimeout(resolve, 1000))
  isSubmitting.value = false
  attempts.value = 0
}
</script>

<template>
  <form @submit.prevent="submitForm">
    <input v-model="username" placeholder="Username" />
    <input v-model="password" type="password" placeholder="Password" />
    <p v-if="errorMessage" class="text-red-500">{{ errorMessage }}</p>
    <p>Attempts: {{ attempts }} / {{ MAX_ATTEMPTS }}</p>
    <button :disabled="isSubmitting">
      {{ isSubmitting ? 'Submitting...' : 'Submit' }}
    </button>
  </form>
</template>
```

---

## 4. Micro-Implementation Drills

**Drill 1 (Basic): Toggle Switch**

Create a simple toggle that switches between ON and OFF.

```vue
<script setup lang="ts">
import { ref } from 'vue'

// TODO: Create a ref called 'isOn' initialized to false
// TODO: Create a function 'toggle' that flips the boolean value

</script>

<template>
  <div>
    <!-- TODO: Display "ON" or "OFF" based on isOn -->
    <!-- TODO: Create a button that calls toggle when clicked -->
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const isOn = ref<boolean>(false)

const toggle = (): void => {
  isOn.value = !isOn.value
}
</script>

<template>
  <div>
    <p>Status: {{ isOn ? 'ON' : 'OFF' }}</p>
    <button @click="toggle">Toggle</button>
  </div>
</template>
```

---

**Drill 2 (Variation): Name Input with Character Count**

Create an input that shows how many characters have been typed.

```vue
<script setup lang="ts">
import { ref } from 'vue'

// TODO: Create a ref called 'name' initialized to an empty string
// TODO: Create a ref called 'charCount' initialized to 0
// TODO: Create a function 'updateName' that takes a string, updates 'name', 
//       and sets 'charCount' to the string's length

</script>

<template>
  <div>
    <!-- TODO: Create an input that calls updateName with its value on each input event -->
    <!-- Hint: @input="updateName($event.target.value)" -->
    <!-- TODO: Display the character count -->
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const name = ref<string>('')
const charCount = ref<number>(0)

const updateName = (value: string): void => {
  name.value = value
  charCount.value = value.length
}
</script>

<template>
  <div>
    <input 
      :value="name" 
      @input="updateName(($event.target as HTMLInputElement).value)"
      placeholder="Enter your name"
    />
    <p>Characters: {{ charCount }}</p>
  </div>
</template>
```

---

**Drill 3 (Combination): Like Counter with Limit**

Create a like button that increments but has a maximum limit.

```vue
<script setup lang="ts">
import { ref } from 'vue'

const MAX_LIKES = 10

// TODO: Create 'likes' ref starting at 0
// TODO: Create 'hasReachedLimit' ref starting at false
// TODO: Create 'addLike' function that:
//       - Increments likes if under limit
//       - Sets hasReachedLimit to true when limit is reached

</script>

<template>
  <div>
    <!-- TODO: Display likes count and max -->
    <!-- TODO: Create a button that calls addLike -->
    <!-- TODO: Show a warning message when limit is reached -->
    <!-- TODO: Disable the button when limit is reached -->
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const MAX_LIKES = 10

const likes = ref<number>(0)
const hasReachedLimit = ref<boolean>(false)

const addLike = (): void => {
  if (likes.value < MAX_LIKES) {
    likes.value++
    if (likes.value === MAX_LIKES) {
      hasReachedLimit.value = true
    }
  }
}
</script>

<template>
  <div>
    <p>Likes: {{ likes }} / {{ MAX_LIKES }}</p>
    <button @click="addLike" :disabled="hasReachedLimit">
      ❤️ Like
    </button>
    <p v-if="hasReachedLimit" class="text-orange-500">
      Maximum likes reached!
    </p>
  </div>
</template>
```

---

**Drill 4 (Edge Case): Number Input with Min/Max Validation**

Create a quantity selector that stays within bounds.

```vue
<script setup lang="ts">
import { ref } from 'vue'

const MIN = 1
const MAX = 99

// TODO: Create 'quantity' ref starting at 1
// TODO: Create 'errorMsg' ref as empty string
// TODO: Create 'setQuantity' that takes a number and:
//       - Sets errorMsg if value is out of bounds
//       - Clamps value to min/max range
//       - Updates quantity

</script>

<template>
  <div>
    <button @click="setQuantity(quantity - 1)">-</button>
    <span>{{ quantity }}</span>
    <button @click="setQuantity(quantity + 1)">+</button>
    <!-- TODO: Display error message if any -->
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const MIN = 1
const MAX = 99

const quantity = ref<number>(1)
const errorMsg = ref<string>('')

const setQuantity = (value: number): void => {
  if (value < MIN) {
    errorMsg.value = `Minimum quantity is ${MIN}`
    quantity.value = MIN
  } else if (value > MAX) {
    errorMsg.value = `Maximum quantity is ${MAX}`
    quantity.value = MAX
  } else {
    errorMsg.value = ''
    quantity.value = value
  }
  
  setTimeout(() => {
    errorMsg.value = ''
  }, 2000)
}
</script>

<template>
  <div>
    <button @click="setQuantity(quantity - 1)">-</button>
    <span class="mx-4 text-xl">{{ quantity }}</span>
    <button @click="setQuantity(quantity + 1)">+</button>
    <p v-if="errorMsg" class="text-red-500 mt-2">{{ errorMsg }}</p>
  </div>
</template>
```

---

## 5. Common Pitfalls & Anti-Patterns

| ❌ Common Mistake | ✅ Correct Approach | Why It Matters |
|:--- |:--- |:--- |
| `count.value` in template | `{{ count }}` | Templates auto-unwrap refs; using `.value` is redundant and confusing |
| `count = 5` in script | `count.value = 5` | You can't reassign the ref itself; you must modify `.value` |
| Forgetting `import { ref } from 'vue'` | Always import ref | Without import, `ref` is undefined — common error for beginners |
| `let count = ref(0)` then `count = ref(1)` | Keep `const`, modify `.value` | Reassigning the ref breaks reactivity; the template loses track |
| Comparing refs directly: `if (count === 5)` | `if (count.value === 5)` | Comparing ref object to primitive always returns false |

**Common Error Messages:**

```typescript
// Error: Cannot assign to 'count' because it is a constant
const count = ref(0)
count = ref(5)  // ❌ Wrong!
count.value = 5 // ✅ Correct!

// Error: ref is not defined
const x = ref(0)  // ❌ Forgot import!
// Fix: import { ref } from 'vue'

// Subtle bug: condition never true
if (count === 0) { }  // ❌ count is an object, not a number
if (count.value === 0) { }  // ✅ Compare the value
```

---

## 6. Mini-Project: Step Counter Dashboard

**File Structure:**
```
src/
├── components/
│   └── StepCounter.vue
├── App.vue
└── main.ts
```

**components/StepCounter.vue:**
```vue
<script setup lang="ts">
// ============================================
// STEP COUNTER COMPONENT
// A fitness-style step counter with goal tracking
// ============================================

import { ref } from 'vue'

// ============================================
// REACTIVE STATE
// All values that can change and affect the UI
// ============================================

const steps = ref<number>(0)
const dailyGoal = ref<number>(10000)
const userName = ref<string>('Fitness Fan')

const lastUpdated = ref<string>('')
const isEditing = ref<boolean>(false)
const tempGoal = ref<string>('')

// ============================================
// HELPER FUNCTIONS
// Pure functions that compute values
// ============================================

const formatNumber = (num: number): string => {
  return num.toLocaleString()
}

const getTimeString = (): string => {
  return new Date().toLocaleTimeString()
}

const calculateProgress = (): number => {
  const progress = (steps.value / dailyGoal.value) * 100
  return Math.min(progress, 100)
}

const getMotivationalMessage = (): string => {
  const progress = calculateProgress()
  if (progress === 0) return "Let's get moving! 🚶"
  if (progress < 25) return 'Good start! Keep going! 💪'
  if (progress < 50) return "You're making progress! 🌟"
  if (progress < 75) return 'More than halfway there! 🔥'
  if (progress < 100) return 'Almost at your goal! 🏃'
  return 'Goal achieved! Amazing! 🎉'
}

// ============================================
// ACTIONS
// Functions that modify reactive state
// ============================================

const addSteps = (amount: number): void => {
  steps.value += amount
  lastUpdated.value = getTimeString()
}

const removeSteps = (amount: number): void => {
  steps.value = Math.max(0, steps.value - amount)
  lastUpdated.value = getTimeString()
}

const resetSteps = (): void => {
  steps.value = 0
  lastUpdated.value = ''
}

const startEditingGoal = (): void => {
  tempGoal.value = dailyGoal.value.toString()
  isEditing.value = true
}

const saveGoal = (): void => {
  const newGoal = parseInt(tempGoal.value, 10)
  if (!isNaN(newGoal) && newGoal > 0) {
    dailyGoal.value = newGoal
  }
  isEditing.value = false
}

const cancelEditingGoal = (): void => {
  isEditing.value = false
  tempGoal.value = ''
}
</script>

<template>
  <div class="max-w-md mx-auto bg-white rounded-2xl shadow-xl overflow-hidden">
    
    <div class="bg-gradient-to-r from-blue-500 to-purple-600 p-6 text-white">
      <h2 class="text-xl font-bold">{{ userName }}'s Steps</h2>
      <p class="text-blue-100 text-sm">Daily Progress Tracker</p>
    </div>

    <div class="p-6">
      <div class="text-center mb-6">
        <p class="text-5xl font-bold text-gray-800">
          {{ formatNumber(steps) }}
        </p>
        
        <div class="flex items-center justify-center gap-2 mt-2">
          <span class="text-gray-500">Goal:</span>
          
          <template v-if="isEditing">
            <input
              v-model="tempGoal"
              type="number"
              min="1"
              class="w-24 px-2 py-1 border rounded text-center"
              @keyup.enter="saveGoal"
              @keyup.escape="cancelEditingGoal"
            />
            <button @click="saveGoal" class="text-green-500">✓</button>
            <button @click="cancelEditingGoal" class="text-red-500">✗</button>
          </template>
          
          <template v-else>
            <span 
              class="text-blue-600 font-medium cursor-pointer hover:underline"
              @click="startEditingGoal"
            >
              {{ formatNumber(dailyGoal) }}
            </span>
          </template>
        </div>
      </div>

      <div class="mb-6">
        <div class="flex justify-between text-sm text-gray-600 mb-1">
          <span>Progress</span>
          <span>{{ calculateProgress().toFixed(1) }}%</span>
        </div>
        <div class="w-full bg-gray-200 rounded-full h-4 overflow-hidden">
          <div
            class="h-full bg-gradient-to-r from-blue-500 to-purple-600 transition-all duration-300"
            :style="{ width: `${calculateProgress()}%` }"
          ></div>
        </div>
      </div>

      <p class="text-center text-gray-600 mb-6">
        {{ getMotivationalMessage() }}
      </p>

      <div class="grid grid-cols-3 gap-2 mb-4">
        <button
          @click="addSteps(100)"
          class="bg-blue-500 hover:bg-blue-600 text-white py-2 px-4 rounded-lg transition"
        >
          +100
        </button>
        <button
          @click="addSteps(500)"
          class="bg-blue-500 hover:bg-blue-600 text-white py-2 px-4 rounded-lg transition"
        >
          +500
        </button>
        <button
          @click="addSteps(1000)"
          class="bg-blue-500 hover:bg-blue-600 text-white py-2 px-4 rounded-lg transition"
        >
          +1K
        </button>
      </div>

      <div class="grid grid-cols-3 gap-2 mb-4">
        <button
          @click="removeSteps(100)"
          class="bg-gray-200 hover:bg-gray-300 text-gray-700 py-2 px-4 rounded-lg transition"
        >
          -100
        </button>
        <button
          @click="removeSteps(500)"
          class="bg-gray-200 hover:bg-gray-300 text-gray-700 py-2 px-4 rounded-lg transition"
        >
          -500
        </button>
        <button
          @click="resetSteps"
          class="bg-red-100 hover:bg-red-200 text-red-600 py-2 px-4 rounded-lg transition"
        >
          Reset
        </button>
      </div>

      <p v-if="lastUpdated" class="text-center text-xs text-gray-400">
        Last updated: {{ lastUpdated }}
      </p>
    </div>
  </div>
</template>
```

**App.vue:**
```vue
<script setup lang="ts">
import StepCounter from './components/StepCounter.vue'
</script>

<template>
  <div class="min-h-screen bg-gray-100 py-12">
    <h1 class="text-3xl font-bold text-center text-gray-800 mb-8">
      Fitness Dashboard
    </h1>
    <StepCounter />
  </div>
</template>
```

**How to Test:**

1. Run the development server: `npm run dev`
2. Click the +100, +500, +1K buttons and watch the count update
3. Observe the progress bar filling up
4. Click on the goal number to edit it
5. Try the -100, -500, and Reset buttons
6. Notice the motivational message changes as you progress

---

## 7. Implementation Exercises (Progressive)

### Level 1 (Recall): Fill in the Blanks

```vue
<script setup lang="ts">
import { _____ } from 'vue'

const score = _____(0)

const addPoint = (): void => {
  score._____ += 1
}
</script>

<template>
  <p>Score: {{ _____ }}</p>
  <button @click="_____">Add Point</button>
</template>
```

**Answer:** `ref`, `ref`, `value`, `score`, `addPoint`

---

### Level 2 (Apply): Small Coding Tasks

**Exercise 2.1:** Create a `VolumeSlider.vue` component with:
- A `volume` ref (number, 0-100)
- A `isMuted` ref (boolean)
- Functions to increase/decrease volume by 10
- A toggle mute function
- Display volume level and mute status

**Exercise 2.2:** Create a `Timer.vue` component with:
- A `seconds` ref starting at 0
- An `isRunning` ref (boolean)
- Start, pause, and reset functions
- Display time in MM:SS format
- (Hint: use `setInterval` in start, `clearInterval` in pause)

---

### Level 3 (Debug): Fix the Broken Code

```vue
<script setup lang="ts">
const count = ref(0)
const doubled = ref(0)

const increment = () => {
  count = count + 1
  doubled.value = count * 2
}
</script>

<template>
  <p>Count: {{ count.value }}</p>
  <p>Doubled: {{ doubled.value }}</p>
  <button @click="increment">Increment</button>
</template>
```

**Bugs to find:**
1. Missing import for `ref`
2. Trying to reassign `count` instead of `count.value`
3. Using `count` instead of `count.value` in calculation
4. Using `.value` in template (not needed)

**Fixed Version:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const count = ref(0)
const doubled = ref(0)

const increment = (): void => {
  count.value = count.value + 1
  doubled.value = count.value * 2
}
</script>

<template>
  <p>Count: {{ count }}</p>
  <p>Doubled: {{ doubled }}</p>
  <button @click="increment">Increment</button>
</template>
```

---

## Self-Check Solutions

**Exercise 2.1 Solution:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const volume = ref<number>(50)
const isMuted = ref<boolean>(false)

const increaseVolume = (): void => {
  if (volume.value < 100) {
    volume.value = Math.min(100, volume.value + 10)
  }
}

const decreaseVolume = (): void => {
  if (volume.value > 0) {
    volume.value = Math.max(0, volume.value - 10)
  }
}

const toggleMute = (): void => {
  isMuted.value = !isMuted.value
}

const getVolumeIcon = (): string => {
  if (isMuted.value || volume.value === 0) return '🔇'
  if (volume.value < 30) return '🔈'
  if (volume.value < 70) return '🔉'
  return '🔊'
}
</script>

<template>
  <div class="p-4">
    <p class="text-2xl mb-4">{{ getVolumeIcon() }} {{ isMuted ? 'Muted' : volume + '%' }}</p>
    <div class="flex gap-2">
      <button @click="decreaseVolume" :disabled="volume === 0">-10</button>
      <button @click="increaseVolume" :disabled="volume === 100">+10</button>
      <button @click="toggleMute">{{ isMuted ? 'Unmute' : 'Mute' }}</button>
    </div>
  </div>
</template>
```

**Exercise 2.2 Solution:**
```vue
<script setup lang="ts">
import { ref, onUnmounted } from 'vue'

const seconds = ref<number>(0)
const isRunning = ref<boolean>(false)
let intervalId: number | null = null

const formatTime = (totalSeconds: number): string => {
  const mins = Math.floor(totalSeconds / 60)
  const secs = totalSeconds % 60
  return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`
}

const start = (): void => {
  if (!isRunning.value) {
    isRunning.value = true
    intervalId = window.setInterval(() => {
      seconds.value++
    }, 1000)
  }
}

const pause = (): void => {
  if (intervalId !== null) {
    clearInterval(intervalId)
    intervalId = null
  }
  isRunning.value = false
}

const reset = (): void => {
  pause()
  seconds.value = 0
}

onUnmounted(() => {
  if (intervalId !== null) {
    clearInterval(intervalId)
  }
})
</script>

<template>
  <div class="p-4 text-center">
    <p class="text-4xl font-mono mb-4">{{ formatTime(seconds) }}</p>
    <div class="flex gap-2 justify-center">
      <button @click="start" :disabled="isRunning">Start</button>
      <button @click="pause" :disabled="!isRunning">Pause</button>
      <button @click="reset">Reset</button>
    </div>
  </div>
</template>
```

---

*Reply 'next' for Lesson 3.1 (Theory).*
