# Lesson 3.2: Reactive Objects with `reactive()` — Practice & Application (Реактивные объекты с reactive())

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 3.2 of 25 (Part 2 of 2) |
| **Topic** | Reactive Objects with `reactive()` / Реактивные объекты с reactive() |
| **Continues From** | Lesson 3.1 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Build components using `reactive()` for structured state management
2. **Analyze:** Compare `ref()` vs `reactive()` usage and identify reactivity loss scenarios

---

## 2. Quick Recap

- **`reactive()`** creates a reactive proxy around an object — all properties become reactive
- **No `.value` needed** — access and modify properties directly: `state.count++`
- **Deep reactivity** — nested objects are automatically reactive
- **Destructuring breaks reactivity** — use `toRefs()` if you need to destructure
- Best for **grouping related state** (forms, user profiles, settings objects)

---

## 3. The Variable Frame & Complexity Scale

**Basic Form:**
```vue
<script setup lang="ts">
import { reactive } from 'vue'

const state = reactive({
  count: 0,
  message: 'Hello'
})
</script>

<template>
  <p>{{ state.count }} - {{ state.message }}</p>
</template>
```

**With TypeScript Interface:**
```vue
<script setup lang="ts">
import { reactive } from 'vue'

interface FormState {
  username: string
  password: string
  rememberMe: boolean
}

const form = reactive<FormState>({
  username: '',
  password: '',
  rememberMe: false
})

const submitForm = (): void => {
  console.log('Submitting:', form.username, form.rememberMe)
}
</script>

<template>
  <form @submit.prevent="submitForm">
    <input v-model="form.username" placeholder="Username" />
    <input v-model="form.password" type="password" placeholder="Password" />
    <label>
      <input v-model="form.rememberMe" type="checkbox" />
      Remember me
    </label>
    <button type="submit">Login</button>
  </form>
</template>
```

**Advanced Form (Nested Objects + Arrays):**
```vue
<script setup lang="ts">
import { reactive } from 'vue'

interface Todo {
  id: number
  text: string
  completed: boolean
}

interface AppState {
  user: {
    name: string
    settings: {
      theme: 'light' | 'dark'
      fontSize: number
    }
  }
  todos: Todo[]
  filter: 'all' | 'active' | 'completed'
}

const state = reactive<AppState>({
  user: {
    name: 'Guest',
    settings: {
      theme: 'light',
      fontSize: 16
    }
  },
  todos: [],
  filter: 'all'
})

let nextId = 1

const addTodo = (text: string): void => {
  state.todos.push({
    id: nextId++,
    text,
    completed: false
  })
}

const toggleTodo = (id: number): void => {
  const todo = state.todos.find(t => t.id === id)
  if (todo) {
    todo.completed = !todo.completed
  }
}

const removeTodo = (id: number): void => {
  const index = state.todos.findIndex(t => t.id === id)
  if (index > -1) {
    state.todos.splice(index, 1)
  }
}

const filteredTodos = (): Todo[] => {
  switch (state.filter) {
    case 'active':
      return state.todos.filter(t => !t.completed)
    case 'completed':
      return state.todos.filter(t => t.completed)
    default:
      return state.todos
  }
}
</script>

<template>
  <div>
    <h2>{{ state.user.name }}'s Todos</h2>
    <p>Theme: {{ state.user.settings.theme }}, Font: {{ state.user.settings.fontSize }}px</p>
    
    <select v-model="state.filter">
      <option value="all">All</option>
      <option value="active">Active</option>
      <option value="completed">Completed</option>
    </select>
    
    <ul>
      <li v-for="todo in filteredTodos()" :key="todo.id">
        <input type="checkbox" :checked="todo.completed" @change="toggleTodo(todo.id)" />
        <span :class="{ 'line-through': todo.completed }">{{ todo.text }}</span>
        <button @click="removeTodo(todo.id)">×</button>
      </li>
    </ul>
  </div>
</template>
```

---

## 4. Micro-Implementation Drills

**Drill 1 (Basic): Simple Counter Object**

Create a counter using `reactive()` with value and step properties.

```vue
<script setup lang="ts">
import { reactive } from 'vue'

// TODO: Create a reactive object 'counter' with:
// - value: number (start at 0)
// - step: number (start at 1)

// TODO: Create 'increment' function that adds step to value
// TODO: Create 'decrement' function that subtracts step from value
// TODO: Create 'setStep' function that changes the step value

</script>

<template>
  <div>
    <!-- TODO: Display current value -->
    <!-- TODO: Display current step -->
    <!-- TODO: Create buttons for increment/decrement -->
    <!-- TODO: Create input to change step size -->
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { reactive } from 'vue'

interface Counter {
  value: number
  step: number
}

const counter = reactive<Counter>({
  value: 0,
  step: 1
})

const increment = (): void => {
  counter.value += counter.step
}

const decrement = (): void => {
  counter.value -= counter.step
}

const setStep = (newStep: number): void => {
  counter.step = Math.max(1, newStep)
}
</script>

<template>
  <div class="p-4">
    <p class="text-2xl">Value: {{ counter.value }}</p>
    <p>Step: {{ counter.step }}</p>
    <div class="flex gap-2 mt-2">
      <button @click="decrement">- {{ counter.step }}</button>
      <button @click="increment">+ {{ counter.step }}</button>
    </div>
    <input 
      type="number" 
      :value="counter.step" 
      @input="setStep(Number(($event.target as HTMLInputElement).value))"
      min="1"
      class="mt-2 border p-1"
    />
  </div>
</template>
```

---

**Drill 2 (Variation): Contact Form State**

Create a contact form with validation state.

```vue
<script setup lang="ts">
import { reactive } from 'vue'

// TODO: Define interface 'ContactForm' with:
// - name: string
// - email: string
// - message: string
// - errors: { name?: string; email?: string; message?: string }
// - isSubmitting: boolean

// TODO: Create reactive 'form' object
// TODO: Create 'validate' function that sets errors
// TODO: Create 'submit' function that validates and simulates submission

</script>

<template>
  <!-- TODO: Create form with inputs for name, email, message -->
  <!-- TODO: Display error messages below each field -->
  <!-- TODO: Disable submit button while submitting -->
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { reactive } from 'vue'

interface FormErrors {
  name?: string
  email?: string
  message?: string
}

interface ContactForm {
  name: string
  email: string
  message: string
  errors: FormErrors
  isSubmitting: boolean
}

const form = reactive<ContactForm>({
  name: '',
  email: '',
  message: '',
  errors: {},
  isSubmitting: false
})

const validate = (): boolean => {
  form.errors = {}
  
  if (!form.name.trim()) {
    form.errors.name = 'Name is required'
  }
  
  if (!form.email.trim()) {
    form.errors.email = 'Email is required'
  } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.email)) {
    form.errors.email = 'Invalid email format'
  }
  
  if (!form.message.trim()) {
    form.errors.message = 'Message is required'
  } else if (form.message.length < 10) {
    form.errors.message = 'Message must be at least 10 characters'
  }
  
  return Object.keys(form.errors).length === 0
}

const submit = async (): Promise<void> => {
  if (!validate()) return
  
  form.isSubmitting = true
  await new Promise(resolve => setTimeout(resolve, 1500))
  form.isSubmitting = false
  
  alert(`Thank you, ${form.name}! Your message has been sent.`)
  form.name = ''
  form.email = ''
  form.message = ''
}
</script>

<template>
  <form @submit.prevent="submit" class="max-w-md space-y-4">
    <div>
      <input 
        v-model="form.name" 
        placeholder="Your name" 
        class="w-full border p-2 rounded"
        :class="{ 'border-red-500': form.errors.name }"
      />
      <p v-if="form.errors.name" class="text-red-500 text-sm">{{ form.errors.name }}</p>
    </div>
    
    <div>
      <input 
        v-model="form.email" 
        type="email" 
        placeholder="Your email"
        class="w-full border p-2 rounded"
        :class="{ 'border-red-500': form.errors.email }"
      />
      <p v-if="form.errors.email" class="text-red-500 text-sm">{{ form.errors.email }}</p>
    </div>
    
    <div>
      <textarea 
        v-model="form.message" 
        placeholder="Your message"
        rows="4"
        class="w-full border p-2 rounded"
        :class="{ 'border-red-500': form.errors.message }"
      ></textarea>
      <p v-if="form.errors.message" class="text-red-500 text-sm">{{ form.errors.message }}</p>
    </div>
    
    <button 
      type="submit" 
      :disabled="form.isSubmitting"
      class="bg-blue-500 text-white px-4 py-2 rounded disabled:opacity-50"
    >
      {{ form.isSubmitting ? 'Sending...' : 'Send Message' }}
    </button>
  </form>
</template>
```

---

**Drill 3 (Combination): Shopping Cart Item**

Combine `reactive()` with computed-like derived values.

```vue
<script setup lang="ts">
import { reactive } from 'vue'

// TODO: Define interface 'CartItem' with:
// - name: string
// - price: number
// - quantity: number

// TODO: Create reactive 'item' object
// TODO: Create functions: increaseQty, decreaseQty, getTotal

</script>

<template>
  <!-- TODO: Display item name, price, quantity -->
  <!-- TODO: Display total (price × quantity) -->
  <!-- TODO: Add +/- buttons for quantity -->
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { reactive } from 'vue'

interface CartItem {
  name: string
  price: number
  quantity: number
}

const item = reactive<CartItem>({
  name: 'Wireless Headphones',
  price: 79.99,
  quantity: 1
})

const increaseQty = (): void => {
  item.quantity++
}

const decreaseQty = (): void => {
  if (item.quantity > 1) {
    item.quantity--
  }
}

const getTotal = (): string => {
  return (item.price * item.quantity).toFixed(2)
}
</script>

<template>
  <div class="p-4 border rounded max-w-sm">
    <h3 class="font-bold">{{ item.name }}</h3>
    <p class="text-gray-600">${{ item.price.toFixed(2) }} each</p>
    
    <div class="flex items-center gap-2 mt-2">
      <button @click="decreaseQty" :disabled="item.quantity <= 1" class="btn">-</button>
      <span class="px-4">{{ item.quantity }}</span>
      <button @click="increaseQty" class="btn">+</button>
    </div>
    
    <p class="mt-4 text-lg font-semibold">
      Total: ${{ getTotal() }}
    </p>
  </div>
</template>
```

---

**Drill 4 (Edge Case): Handling Reactivity Loss**

Fix a component where reactivity is broken due to destructuring.

```vue
<script setup lang="ts">
import { reactive, toRefs } from 'vue'

interface Settings {
  volume: number
  brightness: number
  wifi: boolean
}

const settings = reactive<Settings>({
  volume: 50,
  brightness: 75,
  wifi: true
})

// TODO: This destructuring breaks reactivity! Fix it using toRefs()
// const { volume, brightness, wifi } = settings

// TODO: Create functions to modify each setting

</script>

<template>
  <!-- TODO: Display and modify settings while maintaining reactivity -->
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { reactive, toRefs } from 'vue'

interface Settings {
  volume: number
  brightness: number
  wifi: boolean
}

const settings = reactive<Settings>({
  volume: 50,
  brightness: 75,
  wifi: true
})

const { volume, brightness, wifi } = toRefs(settings)

const setVolume = (val: number): void => {
  volume.value = Math.min(100, Math.max(0, val))
}

const setBrightness = (val: number): void => {
  brightness.value = Math.min(100, Math.max(0, val))
}

const toggleWifi = (): void => {
  wifi.value = !wifi.value
}
</script>

<template>
  <div class="p-4 space-y-4">
    <div>
      <label>Volume: {{ volume }}</label>
      <input 
        type="range" 
        :value="volume" 
        @input="setVolume(Number(($event.target as HTMLInputElement).value))"
        min="0" 
        max="100"
        class="w-full"
      />
    </div>
    
    <div>
      <label>Brightness: {{ brightness }}</label>
      <input 
        type="range" 
        :value="brightness" 
        @input="setBrightness(Number(($event.target as HTMLInputElement).value))"
        min="0" 
        max="100"
        class="w-full"
      />
    </div>
    
    <div>
      <label>
        <input type="checkbox" :checked="wifi" @change="toggleWifi" />
        WiFi: {{ wifi ? 'On' : 'Off' }}
      </label>
    </div>
  </div>
</template>
```

---

## 5. Common Pitfalls & Anti-Patterns

| ❌ Common Mistake | ✅ Correct Approach | Why It Matters |
|:--- |:--- |:--- |
| `reactive(5)` or `reactive('text')` | Use `ref()` for primitives | `reactive()` only works with objects/arrays |
| `const { name } = reactive({name: 'Vue'})` | Keep object reference: `state.name` or use `toRefs()` | Destructuring extracts plain values, losing reactivity |
| Reassigning the whole object: `state = newState` | Assign properties: `Object.assign(state, newState)` | Reassigning breaks the proxy connection |
| `reactive()` for data that might be null | Use `ref<Type \| null>(null)` | `reactive(null)` throws an error |

**Code Examples of Mistakes:**

```typescript
// ❌ reactive() with primitive
const count = reactive(0)  // Error: value cannot be made reactive: 0

// ✅ Use ref() for primitives
const count = ref(0)

// ❌ Reassigning reactive object
let state = reactive({ count: 0 })
state = reactive({ count: 5 })  // Breaks template reactivity!

// ✅ Modify properties instead
const state = reactive({ count: 0 })
state.count = 5  // Works!

// ✅ Or use Object.assign for bulk updates
Object.assign(state, { count: 5, name: 'new' })
```

---

## 6. Mini-Project: Temperature Converter Widget

**File Structure:**
```
src/
├── components/
│   └── TemperatureConverter.vue
├── App.vue
└── main.ts
```

**components/TemperatureConverter.vue:**
```vue
<script setup lang="ts">
// ============================================
// TEMPERATURE CONVERTER COMPONENT
// Real-time conversion between Celsius and Fahrenheit
// ============================================

import { reactive, watch } from 'vue'

// ============================================
// TYPE DEFINITIONS
// ============================================

type TemperatureUnit = 'celsius' | 'fahrenheit'

interface TemperatureState {
  celsius: number
  fahrenheit: number
  lastEdited: TemperatureUnit
  history: Array<{
    celsius: number
    fahrenheit: number
    timestamp: string
  }>
}

interface ConversionInfo {
  formula: string
  description: string
}

// ============================================
// REACTIVE STATE
// Using reactive() because we have multiple related properties
// ============================================

const state = reactive<TemperatureState>({
  celsius: 0,
  fahrenheit: 32,
  lastEdited: 'celsius',
  history: []
})

// ============================================
// CONVERSION FUNCTIONS
// Pure functions for temperature math
// ============================================

const celsiusToFahrenheit = (c: number): number => {
  return (c * 9/5) + 32
}

const fahrenheitToCelsius = (f: number): number => {
  return (f - 32) * 5/9
}

const roundToDecimal = (num: number, decimals: number = 2): number => {
  return Math.round(num * Math.pow(10, decimals)) / Math.pow(10, decimals)
}

// ============================================
// UPDATE HANDLERS
// Modify reactive state and trigger conversions
// ============================================

const updateCelsius = (value: string): void => {
  const parsed = parseFloat(value)
  if (!isNaN(parsed)) {
    state.celsius = parsed
    state.fahrenheit = roundToDecimal(celsiusToFahrenheit(parsed))
    state.lastEdited = 'celsius'
  }
}

const updateFahrenheit = (value: string): void => {
  const parsed = parseFloat(value)
  if (!isNaN(parsed)) {
    state.fahrenheit = parsed
    state.celsius = roundToDecimal(fahrenheitToCelsius(parsed))
    state.lastEdited = 'fahrenheit'
  }
}

const saveToHistory = (): void => {
  state.history.unshift({
    celsius: state.celsius,
    fahrenheit: state.fahrenheit,
    timestamp: new Date().toLocaleTimeString()
  })
  if (state.history.length > 5) {
    state.history.pop()
  }
}

const loadFromHistory = (index: number): void => {
  const entry = state.history[index]
  if (entry) {
    state.celsius = entry.celsius
    state.fahrenheit = entry.fahrenheit
  }
}

const reset = (): void => {
  state.celsius = 0
  state.fahrenheit = 32
  state.lastEdited = 'celsius'
}

// ============================================
// DISPLAY HELPERS
// Functions for UI presentation
// ============================================

const getTemperatureDescription = (): string => {
  const c = state.celsius
  if (c <= -20) return '🥶 Extremely Cold'
  if (c <= 0) return '❄️ Freezing'
  if (c <= 10) return '🧥 Cold'
  if (c <= 20) return '🌤️ Cool'
  if (c <= 25) return '😊 Comfortable'
  if (c <= 30) return '☀️ Warm'
  if (c <= 35) return '🔥 Hot'
  return '🌡️ Extremely Hot'
}

const getConversionInfo = (): ConversionInfo => {
  if (state.lastEdited === 'celsius') {
    return {
      formula: `${state.celsius}°C × 9/5 + 32 = ${state.fahrenheit}°F`,
      description: 'Celsius to Fahrenheit: multiply by 9/5, then add 32'
    }
  }
  return {
    formula: `(${state.fahrenheit}°F - 32) × 5/9 = ${state.celsius}°C`,
    description: 'Fahrenheit to Celsius: subtract 32, then multiply by 5/9'
  }
}

const getBackgroundClass = (): string => {
  const c = state.celsius
  if (c <= 0) return 'bg-blue-100'
  if (c <= 15) return 'bg-cyan-100'
  if (c <= 25) return 'bg-green-100'
  if (c <= 30) return 'bg-yellow-100'
  return 'bg-red-100'
}
</script>

<template>
  <div 
    class="max-w-md mx-auto rounded-2xl shadow-xl overflow-hidden transition-colors duration-300"
    :class="getBackgroundClass()"
  >
    <div class="bg-gradient-to-r from-blue-500 to-purple-600 p-6 text-white">
      <h2 class="text-xl font-bold">Temperature Converter</h2>
      <p class="text-blue-100">{{ getTemperatureDescription() }}</p>
    </div>

    <div class="p-6">
      <div class="grid grid-cols-2 gap-4 mb-6">
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1">
            Celsius (°C)
          </label>
          <input
            type="number"
            :value="state.celsius"
            @input="updateCelsius(($event.target as HTMLInputElement).value)"
            class="w-full px-3 py-2 border rounded-lg text-xl text-center focus:ring-2 focus:ring-blue-500 focus:border-transparent"
            step="0.1"
          />
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1">
            Fahrenheit (°F)
          </label>
          <input
            type="number"
            :value="state.fahrenheit"
            @input="updateFahrenheit(($event.target as HTMLInputElement).value)"
            class="w-full px-3 py-2 border rounded-lg text-xl text-center focus:ring-2 focus:ring-blue-500 focus:border-transparent"
            step="0.1"
          />
        </div>
      </div>

      <div class="bg-white bg-opacity-50 rounded-lg p-4 mb-4">
        <p class="text-sm font-mono text-gray-700">{{ getConversionInfo().formula }}</p>
        <p class="text-xs text-gray-500 mt-1">{{ getConversionInfo().description }}</p>
      </div>

      <div class="flex gap-2 mb-6">
        <button
          @click="saveToHistory"
          class="flex-1 bg-blue-500 hover:bg-blue-600 text-white py-2 px-4 rounded-lg transition"
        >
          Save to History
        </button>
        <button
          @click="reset"
          class="bg-gray-200 hover:bg-gray-300 text-gray-700 py-2 px-4 rounded-lg transition"
        >
          Reset
        </button>
      </div>

      <div v-if="state.history.length > 0">
        <h3 class="text-sm font-medium text-gray-700 mb-2">Recent Conversions</h3>
        <ul class="space-y-1">
          <li
            v-for="(entry, index) in state.history"
            :key="index"
            @click="loadFromHistory(index)"
            class="text-sm bg-white bg-opacity-50 rounded px-3 py-2 cursor-pointer hover:bg-opacity-75 flex justify-between"
          >
            <span>{{ entry.celsius }}°C = {{ entry.fahrenheit }}°F</span>
            <span class="text-gray-400">{{ entry.timestamp }}</span>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>
```

**App.vue:**
```vue
<script setup lang="ts">
import TemperatureConverter from './components/TemperatureConverter.vue'
</script>

<template>
  <div class="min-h-screen bg-gray-100 py-12">
    <h1 class="text-3xl font-bold text-center text-gray-800 mb-8">
      Weather Tools
    </h1>
    <TemperatureConverter />
  </div>
</template>
```

**How to Test:**

1. Enter a value in the Celsius field — Fahrenheit updates automatically
2. Enter a value in the Fahrenheit field — Celsius updates automatically
3. Observe the background color change based on temperature
4. Click "Save to History" to record conversions
5. Click on a history entry to restore those values
6. Check the formula display updates based on which field was last edited

---

## 7. Implementation Exercises (Progressive)

### Level 1 (Recall): Fill in the Blanks

```vue
<script setup lang="ts">
import { _____ } from 'vue'

interface User {
  name: string
  age: number
}

const user = _____<User>({
  name: 'John',
  _____: 25
})

const birthday = (): void => {
  user._____ += 1
}
</script>

<template>
  <p>{{ user._____ }} is {{ user.age }} years old</p>
  <button @click="_____">Happy Birthday!</button>
</template>
```

**Answer:** `reactive`, `reactive`, `age`, `age`, `name`, `birthday`

---

### Level 2 (Apply): Small Coding Tasks

**Exercise 2.1:** Create a `BookInfo.vue` component using `reactive()` with:
- Interface: `Book { title: string; author: string; pages: number; currentPage: number }`
- Display book info and reading progress percentage
- Functions to turn page forward/backward (stay within bounds)

**Exercise 2.2:** Create a `PollWidget.vue` component using `reactive()` with:
- Interface: `Poll { question: string; options: Array<{ text: string; votes: number }> }`
- Display question and options with vote counts
- Allow voting (increment vote count for an option)
- Show percentage for each option

---

### Level 3 (Debug): Fix the Broken Code

```vue
<script setup lang="ts">
import { reactive } from 'vue'

interface Profile {
  name: string
  email: string
}

let profile = reactive<Profile>({
  name: 'John',
  email: 'john@example.com'
})

const updateProfile = (newProfile: Profile): void => {
  profile = reactive(newProfile)
}

const { name, email } = profile

const updateName = (newName: string): void => {
  name = newName
}
</script>

<template>
  <p>{{ name }} - {{ email }}</p>
  <button @click="updateName('Jane')">Change Name</button>
</template>
```

**Bugs to find:**
1. Reassigning `profile` breaks reactivity
2. Destructuring loses reactivity
3. Assigning to destructured `name` doesn't work

**Fixed Version:**
```vue
<script setup lang="ts">
import { reactive } from 'vue'

interface Profile {
  name: string
  email: string
}

const profile = reactive<Profile>({
  name: 'John',
  email: 'john@example.com'
})

const updateProfile = (newProfile: Profile): void => {
  Object.assign(profile, newProfile)
}

const updateName = (newName: string): void => {
  profile.name = newName
}
</script>

<template>
  <p>{{ profile.name }} - {{ profile.email }}</p>
  <button @click="updateName('Jane')">Change Name</button>
</template>
```

---

## Self-Check Solutions

**Exercise 2.1 Solution:**
```vue
<script setup lang="ts">
import { reactive } from 'vue'

interface Book {
  title: string
  author: string
  pages: number
  currentPage: number
}

const book = reactive<Book>({
  title: 'Vue.js in Action',
  author: 'Erik Hanchett',
  pages: 350,
  currentPage: 1
})

const nextPage = (): void => {
  if (book.currentPage < book.pages) {
    book.currentPage++
  }
}

const prevPage = (): void => {
  if (book.currentPage > 1) {
    book.currentPage--
  }
}

const getProgress = (): number => {
  return Math.round((book.currentPage / book.pages) * 100)
}
</script>

<template>
  <div class="p-4 border rounded">
    <h2 class="font-bold">{{ book.title }}</h2>
    <p class="text-gray-600">by {{ book.author }}</p>
    <p>Page {{ book.currentPage }} of {{ book.pages }}</p>
    <div class="w-full bg-gray-200 rounded h-2 mt-2">
      <div class="bg-blue-500 h-2 rounded" :style="{ width: getProgress() + '%' }"></div>
    </div>
    <p class="text-sm text-gray-500">{{ getProgress() }}% complete</p>
    <div class="flex gap-2 mt-2">
      <button @click="prevPage" :disabled="book.currentPage <= 1">Previous</button>
      <button @click="nextPage" :disabled="book.currentPage >= book.pages">Next</button>
    </div>
  </div>
</template>
```

**Exercise 2.2 Solution:**
```vue
<script setup lang="ts">
import { reactive } from 'vue'

interface PollOption {
  text: string
  votes: number
}

interface Poll {
  question: string
  options: PollOption[]
}

const poll = reactive<Poll>({
  question: 'What is your favorite Vue feature?',
  options: [
    { text: 'Composition API', votes: 0 },
    { text: 'Reactivity System', votes: 0 },
    { text: 'Single File Components', votes: 0 },
    { text: 'Vue Router', votes: 0 }
  ]
})

const vote = (index: number): void => {
  poll.options[index].votes++
}

const getTotalVotes = (): number => {
  return poll.options.reduce((sum, opt) => sum + opt.votes, 0)
}

const getPercentage = (votes: number): number => {
  const total = getTotalVotes()
  return total === 0 ? 0 : Math.round((votes / total) * 100)
}
</script>

<template>
  <div class="p-4 max-w-md">
    <h2 class="font-bold mb-4">{{ poll.question }}</h2>
    <div class="space-y-2">
      <div v-for="(option, index) in poll.options" :key="index" class="flex items-center gap-2">
        <button @click="vote(index)" class="px-3 py-1 bg-blue-500 text-white rounded">
          Vote
        </button>
        <div class="flex-1">
          <div class="flex justify-between">
            <span>{{ option.text }}</span>
            <span>{{ option.votes }} votes ({{ getPercentage(option.votes) }}%)</span>
          </div>
          <div class="w-full bg-gray-200 rounded h-2">
            <div 
              class="bg-blue-500 h-2 rounded transition-all" 
              :style="{ width: getPercentage(option.votes) + '%' }"
            ></div>
          </div>
        </div>
      </div>
    </div>
    <p class="mt-4 text-gray-500">Total votes: {{ getTotalVotes() }}</p>
  </div>
</template>
```

---

*Reply 'next' for Lesson 4.1 (Theory).*
