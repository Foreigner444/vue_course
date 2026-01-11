# Lesson 11.2: Handling User Input: Event Handling — Practice & Application (Обработка пользовательского ввода: Обработка событий)

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 11.2 of 25 (Part 2 of 2) |
| **Topic** | Event Handling (`@click`, etc.) / Обработка событий |
| **Continues From** | Lesson 11.1 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Create interactive components using various event types and modifiers
2. **Analyze:** Debug common event handling issues and choose appropriate event patterns

---

## 2. Quick Recap

- **`@click="handler"`** attaches a click listener that calls `handler` with the event object
- **Event modifiers** (`.prevent`, `.stop`, `.once`) declaratively handle common event operations
- **Key modifiers** (`.enter`, `.escape`, `.ctrl`) filter events by key or system modifier
- Vue **automatically manages listener lifecycle**, preventing memory leaks

---

## 3. The Variable Frame & Complexity Scale

**Basic Form:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const message = ref('Hello')

function showAlert(): void {
  alert(message.value)
}
</script>

<template>
  <button @click="showAlert">Show Message</button>
</template>
```

**With Event Object:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const mouseX = ref(0)
const mouseY = ref(0)

function trackMouse(event: MouseEvent): void {
  mouseX.value = event.clientX
  mouseY.value = event.clientY
}
</script>

<template>
  <div 
    @mousemove="trackMouse"
    class="h-64 bg-gray-100 flex items-center justify-center"
  >
    Position: ({{ mouseX }}, {{ mouseY }})
  </div>
</template>
```

**Advanced Form (Multiple Events & Modifiers):**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const isHovered = ref(false)
const clickCount = ref(0)
const lastKey = ref('')

function handleKeyCombo(event: KeyboardEvent): void {
  if (event.ctrlKey && event.key === 's') {
    event.preventDefault()
    console.log('Save triggered!')
  }
  lastKey.value = event.key
}
</script>

<template>
  <div
    @mouseenter="isHovered = true"
    @mouseleave="isHovered = false"
    @click.stop="clickCount++"
    @keydown="handleKeyCombo"
    tabindex="0"
    class="p-4 border rounded"
    :class="{ 'bg-blue-100': isHovered }"
  >
    <p>Hovered: {{ isHovered }}</p>
    <p>Clicks: {{ clickCount }}</p>
    <p>Last key: {{ lastKey }}</p>
    <small>Press Ctrl+S to test key combo</small>
  </div>
</template>
```

---

## 4. Micro-Implementation Drills

### Drill 1 (Basic): Simple Counter Button
Create a button that increments a counter on each click.

```vue
<script setup lang="ts">
import { ref } from 'vue'

// TODO: Create a ref called 'count' initialized to 0
const count = ref(0)

// TODO: Create a function 'increment' that adds 1 to count
</script>

<template>
  <div class="p-4">
    <!-- TODO: Add @click to call increment -->
    <button class="btn btn-primary">
      Clicked: {{ count }} times
    </button>
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const count = ref(0)

function increment(): void {
  count.value++
}
</script>

<template>
  <div class="p-4">
    <button @click="increment" class="btn btn-primary">
      Clicked: {{ count }} times
    </button>
  </div>
</template>
```

---

### Drill 2 (Variation): Keyboard Event with Enter Key
Create an input that logs its value when Enter is pressed.

```vue
<script setup lang="ts">
import { ref } from 'vue'

const inputValue = ref('')
const submittedValues = ref<string[]>([])

// TODO: Create a function 'submitValue' that:
// 1. Pushes inputValue.value to submittedValues array
// 2. Clears inputValue
</script>

<template>
  <div class="p-4">
    <!-- TODO: Bind inputValue with v-model and add @keyup.enter -->
    <input
      type="text"
      placeholder="Type and press Enter"
      class="input input-bordered"
    />
    
    <ul class="mt-4">
      <li v-for="(value, index) in submittedValues" :key="index">
        {{ value }}
      </li>
    </ul>
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const inputValue = ref('')
const submittedValues = ref<string[]>([])

function submitValue(): void {
  if (inputValue.value.trim()) {
    submittedValues.value.push(inputValue.value)
    inputValue.value = ''
  }
}
</script>

<template>
  <div class="p-4">
    <input
      v-model="inputValue"
      @keyup.enter="submitValue"
      type="text"
      placeholder="Type and press Enter"
      class="input input-bordered"
    />
    
    <ul class="mt-4">
      <li v-for="(value, index) in submittedValues" :key="index">
        {{ value }}
      </li>
    </ul>
  </div>
</template>
```

---

### Drill 3 (Combination): Form with Prevent Default
Create a form that prevents page reload on submit.

```vue
<script setup lang="ts">
import { ref } from 'vue'

const email = ref('')
const submitted = ref(false)

// TODO: Create 'handleSubmit' that sets submitted to true
// Note: You should NOT need to call preventDefault manually!
</script>

<template>
  <div class="p-4">
    <!-- TODO: Add @submit with .prevent modifier -->
    <form class="space-y-4">
      <input
        v-model="email"
        type="email"
        placeholder="Enter email"
        class="input input-bordered w-full"
      />
      <button type="submit" class="btn btn-primary">
        Subscribe
      </button>
    </form>
    
    <div v-if="submitted" class="alert alert-success mt-4">
      Thanks for subscribing with {{ email }}!
    </div>
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const email = ref('')
const submitted = ref(false)

function handleSubmit(): void {
  submitted.value = true
}
</script>

<template>
  <div class="p-4">
    <form @submit.prevent="handleSubmit" class="space-y-4">
      <input
        v-model="email"
        type="email"
        placeholder="Enter email"
        class="input input-bordered w-full"
      />
      <button type="submit" class="btn btn-primary">
        Subscribe
      </button>
    </form>
    
    <div v-if="submitted" class="alert alert-success mt-4">
      Thanks for subscribing with {{ email }}!
    </div>
  </div>
</template>
```

---

### Drill 4 (Edge Case): Stop Propagation in Nested Elements
Create nested clickable elements where inner clicks don't trigger outer handlers.

```vue
<script setup lang="ts">
import { ref } from 'vue'

const outerClicks = ref(0)
const innerClicks = ref(0)

function handleOuterClick(): void {
  outerClicks.value++
}

// TODO: Create handleInnerClick
</script>

<template>
  <div class="p-4">
    <!-- Outer clickable div -->
    <div
      @click="handleOuterClick"
      class="p-8 bg-blue-200 rounded cursor-pointer"
    >
      <p>Outer: {{ outerClicks }} clicks</p>
      
      <!-- TODO: Add @click with .stop modifier to prevent bubbling -->
      <button class="btn btn-secondary mt-2">
        Inner: {{ innerClicks }} clicks
      </button>
    </div>
    
    <p class="mt-4 text-sm text-gray-600">
      Without .stop, clicking inner button would also increment outer!
    </p>
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const outerClicks = ref(0)
const innerClicks = ref(0)

function handleOuterClick(): void {
  outerClicks.value++
}

function handleInnerClick(): void {
  innerClicks.value++
}
</script>

<template>
  <div class="p-4">
    <div
      @click="handleOuterClick"
      class="p-8 bg-blue-200 rounded cursor-pointer"
    >
      <p>Outer: {{ outerClicks }} clicks</p>
      
      <button 
        @click.stop="handleInnerClick" 
        class="btn btn-secondary mt-2"
      >
        Inner: {{ innerClicks }} clicks
      </button>
    </div>
    
    <p class="mt-4 text-sm text-gray-600">
      Without .stop, clicking inner button would also increment outer!
    </p>
  </div>
</template>
```

---

## 5. Common Pitfalls & Anti-Patterns

| ❌ Common Mistake | ✅ Correct Approach | Why It Matters |
|:--- |:--- |:--- |
| `@click="handler()"` (with parentheses but no args) | `@click="handler"` or `@click="handler($event)"` | Empty parentheses call the function immediately during render, not on click. Without parens, Vue calls it on the event. |
| Calling `event.preventDefault()` manually for every form | `@submit.prevent="handler"` | Modifiers are more declarative, easier to read, and less error-prone. |
| Adding event listeners with `addEventListener` in `onMounted` | Use `@event` directive in template | Vue handles cleanup automatically. Manual listeners can cause memory leaks if not removed in `onUnmounted`. |

### Detailed Examples

**Mistake 1: Immediate Invocation**
```vue
<!-- ❌ WRONG: handler() executes during render, not on click -->
<button @click="showMessage()">Click me</button>

<!-- ✅ CORRECT: handler executes on click -->
<button @click="showMessage">Click me</button>

<!-- ✅ ALSO CORRECT: when you need to pass arguments -->
<button @click="showMessage('Hello')">Click me</button>
```

**Mistake 2: Manual preventDefault**
```vue
<!-- ❌ WRONG: Verbose and easy to forget -->
<script setup lang="ts">
function handleSubmit(event: Event): void {
  event.preventDefault()
  // ... rest of logic
}
</script>
<template>
  <form @submit="handleSubmit">...</form>
</template>

<!-- ✅ CORRECT: Declarative modifier -->
<script setup lang="ts">
function handleSubmit(): void {
  // No need to handle event at all!
}
</script>
<template>
  <form @submit.prevent="handleSubmit">...</form>
</template>
```

**Mistake 3: Manual Event Listeners**
```vue
<!-- ❌ WRONG: Memory leak risk -->
<script setup lang="ts">
import { onMounted, onUnmounted } from 'vue'

function handleResize(): void {
  console.log('resized')
}

onMounted(() => {
  window.addEventListener('resize', handleResize)
})

// Easy to forget this!
onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
})
</script>

<!-- ✅ BETTER: Use a composable like @vueuse/core -->
<script setup lang="ts">
import { useWindowSize } from '@vueuse/core'

const { width, height } = useWindowSize()
</script>
```

---

## 6. Mini-Project (Putting It Together)

**Interactive Click Tracker Panel**

A component that tracks mouse clicks, displays click positions, and supports keyboard shortcuts.

**File Structure:**
```
src/
├── components/
│   └── ClickTracker.vue
├── App.vue
└── main.ts
```

**ClickTracker.vue:**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

// Define a type for our click data
interface ClickPoint {
  id: number
  x: number
  y: number
  timestamp: Date
}

// Reactive state for tracking clicks
const clicks = ref<ClickPoint[]>([])
const isTracking = ref<boolean>(true)
let nextId = 1

// Computed property for click count
const clickCount = computed<number>(() => clicks.value.length)

// Computed property for average position
const averagePosition = computed<{ x: number; y: number } | null>(() => {
  if (clicks.value.length === 0) return null
  
  const sumX = clicks.value.reduce((sum, click) => sum + click.x, 0)
  const sumY = clicks.value.reduce((sum, click) => sum + click.y, 0)
  
  return {
    x: Math.round(sumX / clicks.value.length),
    y: Math.round(sumY / clicks.value.length)
  }
})

// Handler for tracking clicks in the panel
function handlePanelClick(event: MouseEvent): void {
  // Only track if tracking is enabled
  if (!isTracking.value) return
  
  // Get click position relative to the panel
  const target = event.currentTarget as HTMLElement
  const rect = target.getBoundingClientRect()
  
  const newClick: ClickPoint = {
    id: nextId++,
    x: Math.round(event.clientX - rect.left),
    y: Math.round(event.clientY - rect.top),
    timestamp: new Date()
  }
  
  clicks.value.push(newClick)
}

// Handler to remove a specific click
function removeClick(id: number): void {
  clicks.value = clicks.value.filter(click => click.id !== id)
}

// Handler to clear all clicks
function clearAllClicks(): void {
  clicks.value = []
}

// Toggle tracking on/off
function toggleTracking(): void {
  isTracking.value = !isTracking.value
}

// Keyboard shortcut handler
function handleKeyboard(event: KeyboardEvent): void {
  // 'c' key clears all clicks
  if (event.key === 'c' && !event.ctrlKey) {
    clearAllClicks()
  }
  // 't' key toggles tracking
  if (event.key === 't') {
    toggleTracking()
  }
  // Escape removes the last click
  if (event.key === 'Escape' && clicks.value.length > 0) {
    clicks.value.pop()
  }
}
</script>

<template>
  <!-- 
    Main container with tabindex to receive keyboard events
    @keydown listens for keyboard shortcuts
  -->
  <div 
    @keydown="handleKeyboard"
    tabindex="0"
    class="p-4 outline-none"
  >
    <!-- Header with controls -->
    <div class="flex items-center justify-between mb-4">
      <h2 class="text-xl font-bold">Click Tracker</h2>
      
      <div class="flex gap-2">
        <!-- Toggle button with dynamic text and styling -->
        <button
          @click="toggleTracking"
          class="btn btn-sm"
          :class="isTracking ? 'btn-success' : 'btn-warning'"
        >
          {{ isTracking ? 'Tracking ON' : 'Tracking OFF' }}
        </button>
        
        <!-- Clear button with .stop to prevent panel click -->
        <button
          @click.stop="clearAllClicks"
          class="btn btn-sm btn-error"
        >
          Clear All
        </button>
      </div>
    </div>
    
    <!-- Stats display -->
    <div class="stats shadow mb-4">
      <div class="stat">
        <div class="stat-title">Total Clicks</div>
        <div class="stat-value">{{ clickCount }}</div>
      </div>
      <div class="stat">
        <div class="stat-title">Average Position</div>
        <div class="stat-value text-lg">
          <template v-if="averagePosition">
            ({{ averagePosition.x }}, {{ averagePosition.y }})
          </template>
          <template v-else>
            N/A
          </template>
        </div>
      </div>
    </div>
    
    <!-- Clickable tracking panel -->
    <div
      @click="handlePanelClick"
      class="relative h-64 bg-base-200 rounded-lg border-2 border-dashed cursor-crosshair overflow-hidden"
      :class="{ 'border-success': isTracking, 'border-warning': !isTracking }"
    >
      <!-- Render click markers at their positions -->
      <div
        v-for="click in clicks"
        :key="click.id"
        class="absolute w-4 h-4 -ml-2 -mt-2 bg-primary rounded-full cursor-pointer hover:bg-error transition-colors"
        :style="{ left: `${click.x}px`, top: `${click.y}px` }"
        @click.stop="removeClick(click.id)"
        title="Click to remove"
      />
      
      <!-- Empty state message -->
      <div
        v-if="clicks.length === 0"
        class="absolute inset-0 flex items-center justify-center text-base-content/50"
      >
        Click anywhere in this area to track clicks
      </div>
    </div>
    
    <!-- Click history list -->
    <div class="mt-4">
      <h3 class="font-semibold mb-2">Click History</h3>
      <ul class="space-y-1 max-h-32 overflow-y-auto">
        <li
          v-for="click in clicks"
          :key="click.id"
          class="flex items-center justify-between p-2 bg-base-200 rounded"
        >
          <span>
            Click #{{ click.id }}: ({{ click.x }}, {{ click.y }})
          </span>
          <button
            @click="removeClick(click.id)"
            class="btn btn-xs btn-ghost"
          >
            ✕
          </button>
        </li>
      </ul>
    </div>
    
    <!-- Keyboard shortcuts help -->
    <div class="mt-4 text-sm text-base-content/70">
      <strong>Keyboard Shortcuts:</strong>
      <kbd class="kbd kbd-sm">T</kbd> Toggle tracking |
      <kbd class="kbd kbd-sm">C</kbd> Clear all |
      <kbd class="kbd kbd-sm">Esc</kbd> Undo last
    </div>
  </div>
</template>
```

**How to Test:**
1. Create a new Vue project: `npm create vue@latest click-tracker`
2. Install dependencies: `npm install && npm install -D tailwindcss daisyui`
3. Copy `ClickTracker.vue` to `src/components/`
4. Import and use in `App.vue`: `<ClickTracker />`
5. Run: `npm run dev`
6. Test: Click in the panel, use keyboard shortcuts, try removing individual clicks

---

## 7. Implementation Exercises (Progressive)

### Level 1 (Recall): Fill in the Blanks
```vue
<script setup lang="ts">
import { ref } from 'vue'

const likes = ref(0)

function handleLike(): void {
  likes.value++
}
</script>

<template>
  <!-- Fill in the blanks to make the button work -->
  <button _____="_____" class="btn">
    ❤️ {{ likes }}
  </button>
</template>
```

**Answer:** `@click="handleLike"`

---

### Level 2 (Apply): Independent Coding Tasks

**Exercise 2.1: Double Click Counter**
Create a button that:
- Increments by 1 on single click
- Increments by 5 on double click
- Shows the current count

**Exercise 2.2: Hover Card**
Create a card component that:
- Shows basic info by default
- Reveals additional details when hovered
- Has a button that stops propagation when clicked

---

### Level 3 (Debug): Fix the Broken Code
```vue
<script setup lang="ts">
import { ref } from 'vue'

const formData = ref({ name: '', email: '' })
const isSubmitted = ref(false)

function submitForm(event) {
  event.preventDefault()  // This works but is not ideal
  isSubmitted.value = true
}
</script>

<template>
  <form @submit="submitForm()">
    <input v-model="formData.name" placeholder="Name" />
    <input v-model="formData.email" placeholder="Email" />
    <button type="submit">Submit</button>
  </form>
  
  <div v-show="isSubmitted">
    Form submitted!
  </div>
</template>
```

**Bugs to Find:**
1. `@submit="submitForm()"` — parentheses cause immediate invocation
2. Should use `.prevent` modifier instead of manual `preventDefault`
3. Function parameter type should be `Event` or omitted entirely

**Fixed Code:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const formData = ref({ name: '', email: '' })
const isSubmitted = ref(false)

function submitForm(): void {
  isSubmitted.value = true
}
</script>

<template>
  <form @submit.prevent="submitForm">
    <input v-model="formData.name" placeholder="Name" />
    <input v-model="formData.email" placeholder="Email" />
    <button type="submit">Submit</button>
  </form>
  
  <div v-show="isSubmitted">
    Form submitted!
  </div>
</template>
```

---

**Reply 'next' for Lesson 12.1 (Theory).**
