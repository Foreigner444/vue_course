# Lesson 4.2: Derived State with `computed()` — Practice & Application (Вычисляемые свойства с computed())

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 4.2 of 25 (Part 2 of 2) |
| **Topic** | Derived State with `computed()` / Вычисляемые свойства с computed() |
| **Continues From** | Lesson 4.1 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Create components with multiple computed properties for complex derived state
2. **Analyze:** Compare computed vs functions and implement writable computed properties

---

## 2. Quick Recap

- **`computed()`** creates a cached reactive value derived from other reactive state
- **Automatic caching** — only recalculates when dependencies change
- **Auto-dependency tracking** — Vue detects which refs/reactives are used (no manual array)
- Access with **`.value`** in script, **auto-unwrapped** in template
- **Writable computed** uses `{ get(), set() }` for two-way derived state

---

## 3. The Variable Frame & Complexity Scale

**Basic Form:**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

const count = ref(0)
const doubled = computed(() => count.value * 2)
</script>

<template>
  <p>{{ count }} × 2 = {{ doubled }}</p>
</template>
```

**With TypeScript & Multiple Dependencies:**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

const price = ref(100)
const taxRate = ref(0.08)
const quantity = ref(2)

const subtotal = computed<number>(() => price.value * quantity.value)
const tax = computed<number>(() => subtotal.value * taxRate.value)
const total = computed<number>(() => subtotal.value + tax.value)

const formattedTotal = computed<string>(() => {
  return `$${total.value.toFixed(2)}`
})
</script>

<template>
  <div>
    <p>Subtotal: ${{ subtotal.toFixed(2) }}</p>
    <p>Tax ({{ (taxRate * 100).toFixed(0) }}%): ${{ tax.toFixed(2) }}</p>
    <p>Total: {{ formattedTotal }}</p>
  </div>
</template>
```

**Advanced Form (Writable Computed + Complex Logic):**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

interface Product {
  id: number
  name: string
  price: number
  quantity: number
}

const products = ref<Product[]>([
  { id: 1, name: 'Laptop', price: 999, quantity: 1 },
  { id: 2, name: 'Mouse', price: 49, quantity: 2 }
])

const searchQuery = ref('')
const sortBy = ref<'name' | 'price'>('name')
const sortOrder = ref<'asc' | 'desc'>('asc')

const filteredProducts = computed(() => {
  return products.value.filter(p =>
    p.name.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
})

const sortedProducts = computed(() => {
  return [...filteredProducts.value].sort((a, b) => {
    const modifier = sortOrder.value === 'asc' ? 1 : -1
    if (sortBy.value === 'name') {
      return a.name.localeCompare(b.name) * modifier
    }
    return (a.price - b.price) * modifier
  })
})

const cartTotal = computed(() => {
  return products.value.reduce(
    (sum, p) => sum + p.price * p.quantity,
    0
  )
})

const selectedProductIds = computed({
  get() {
    return products.value.filter(p => p.quantity > 0).map(p => p.id)
  },
  set(ids: number[]) {
    products.value.forEach(p => {
      p.quantity = ids.includes(p.id) ? Math.max(1, p.quantity) : 0
    })
  }
})
</script>

<template>
  <div>
    <input v-model="searchQuery" placeholder="Search..." />
    <select v-model="sortBy">
      <option value="name">Name</option>
      <option value="price">Price</option>
    </select>
    <button @click="sortOrder = sortOrder === 'asc' ? 'desc' : 'asc'">
      {{ sortOrder === 'asc' ? '↑' : '↓' }}
    </button>
    
    <ul>
      <li v-for="product in sortedProducts" :key="product.id">
        {{ product.name }} - ${{ product.price }}
      </li>
    </ul>
    
    <p>Cart Total: ${{ cartTotal }}</p>
  </div>
</template>
```

---

## 4. Micro-Implementation Drills

**Drill 1 (Basic): Character Counter**

Create a text input with computed character count and validation.

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

const MAX_LENGTH = 100

// TODO: Create a ref 'text' initialized to empty string
// TODO: Create computed 'charCount' that returns text length
// TODO: Create computed 'remainingChars' that returns MAX_LENGTH - charCount
// TODO: Create computed 'isOverLimit' that returns boolean

</script>

<template>
  <div>
    <!-- TODO: Create textarea bound to text -->
    <!-- TODO: Display character count and remaining -->
    <!-- TODO: Show warning if over limit -->
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

const MAX_LENGTH = 100

const text = ref('')

const charCount = computed(() => text.value.length)
const remainingChars = computed(() => MAX_LENGTH - charCount.value)
const isOverLimit = computed(() => charCount.value > MAX_LENGTH)
</script>

<template>
  <div class="p-4">
    <textarea
      v-model="text"
      class="w-full border p-2 rounded"
      :class="{ 'border-red-500': isOverLimit }"
      rows="4"
      placeholder="Enter your message..."
    ></textarea>
    <div class="flex justify-between mt-1 text-sm">
      <span :class="isOverLimit ? 'text-red-500' : 'text-gray-500'">
        {{ charCount }} / {{ MAX_LENGTH }} characters
      </span>
      <span v-if="isOverLimit" class="text-red-500">
        {{ Math.abs(remainingChars) }} over limit!
      </span>
      <span v-else class="text-gray-500">
        {{ remainingChars }} remaining
      </span>
    </div>
  </div>
</template>
```

---

**Drill 2 (Variation): Grade Calculator**

Create a grade calculator that computes average and letter grade.

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

// TODO: Create ref 'scores' as an array of numbers [85, 92, 78, 95]
// TODO: Create computed 'average' that calculates mean
// TODO: Create computed 'letterGrade' based on average:
//       A: 90+, B: 80-89, C: 70-79, D: 60-69, F: below 60
// TODO: Create function 'addScore' that pushes a new score

</script>

<template>
  <!-- TODO: Display all scores -->
  <!-- TODO: Display average and letter grade -->
  <!-- TODO: Input to add new score -->
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

const scores = ref<number[]>([85, 92, 78, 95])
const newScore = ref<number>(0)

const average = computed<number>(() => {
  if (scores.value.length === 0) return 0
  const sum = scores.value.reduce((a, b) => a + b, 0)
  return sum / scores.value.length
})

const letterGrade = computed<string>(() => {
  const avg = average.value
  if (avg >= 90) return 'A'
  if (avg >= 80) return 'B'
  if (avg >= 70) return 'C'
  if (avg >= 60) return 'D'
  return 'F'
})

const addScore = (): void => {
  if (newScore.value >= 0 && newScore.value <= 100) {
    scores.value.push(newScore.value)
    newScore.value = 0
  }
}

const removeScore = (index: number): void => {
  scores.value.splice(index, 1)
}
</script>

<template>
  <div class="p-4 max-w-md">
    <h2 class="text-xl font-bold mb-4">Grade Calculator</h2>
    
    <div class="space-y-2 mb-4">
      <div v-for="(score, index) in scores" :key="index" class="flex justify-between items-center bg-gray-100 p-2 rounded">
        <span>Score {{ index + 1 }}: {{ score }}</span>
        <button @click="removeScore(index)" class="text-red-500">×</button>
      </div>
    </div>
    
    <div class="flex gap-2 mb-4">
      <input 
        v-model.number="newScore" 
        type="number" 
        min="0" 
        max="100"
        class="border p-2 rounded flex-1"
        placeholder="Enter score (0-100)"
      />
      <button @click="addScore" class="bg-blue-500 text-white px-4 rounded">Add</button>
    </div>
    
    <div class="bg-gray-100 p-4 rounded">
      <p class="text-lg">Average: <strong>{{ average.toFixed(1) }}</strong></p>
      <p class="text-2xl">
        Grade: <strong :class="{
          'text-green-600': letterGrade === 'A',
          'text-blue-600': letterGrade === 'B',
          'text-yellow-600': letterGrade === 'C',
          'text-orange-600': letterGrade === 'D',
          'text-red-600': letterGrade === 'F'
        }">{{ letterGrade }}</strong>
      </p>
    </div>
  </div>
</template>
```

---

**Drill 3 (Combination): Filterable List**

Create a list with search, filter, and sort using multiple computed properties.

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

interface Task {
  id: number
  title: string
  completed: boolean
  priority: 'low' | 'medium' | 'high'
}

// TODO: Create ref 'tasks' with sample data
// TODO: Create ref 'searchQuery' for text search
// TODO: Create ref 'filterStatus' ('all' | 'active' | 'completed')
// TODO: Create computed 'filteredTasks' that combines search and status filter
// TODO: Create computed 'taskStats' with counts for all/active/completed

</script>

<template>
  <!-- TODO: Search input -->
  <!-- TODO: Filter buttons for status -->
  <!-- TODO: Display filtered tasks -->
  <!-- TODO: Display stats -->
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

interface Task {
  id: number
  title: string
  completed: boolean
  priority: 'low' | 'medium' | 'high'
}

const tasks = ref<Task[]>([
  { id: 1, title: 'Learn Vue 3', completed: true, priority: 'high' },
  { id: 2, title: 'Build a project', completed: false, priority: 'high' },
  { id: 3, title: 'Write tests', completed: false, priority: 'medium' },
  { id: 4, title: 'Deploy to production', completed: false, priority: 'low' },
  { id: 5, title: 'Setup CI/CD', completed: true, priority: 'medium' }
])

const searchQuery = ref('')
const filterStatus = ref<'all' | 'active' | 'completed'>('all')

const filteredTasks = computed(() => {
  return tasks.value
    .filter(task => {
      const matchesSearch = task.title.toLowerCase().includes(searchQuery.value.toLowerCase())
      const matchesStatus = 
        filterStatus.value === 'all' ||
        (filterStatus.value === 'active' && !task.completed) ||
        (filterStatus.value === 'completed' && task.completed)
      return matchesSearch && matchesStatus
    })
})

const taskStats = computed(() => ({
  total: tasks.value.length,
  active: tasks.value.filter(t => !t.completed).length,
  completed: tasks.value.filter(t => t.completed).length
}))

const toggleTask = (id: number): void => {
  const task = tasks.value.find(t => t.id === id)
  if (task) task.completed = !task.completed
}
</script>

<template>
  <div class="p-4 max-w-lg">
    <input
      v-model="searchQuery"
      placeholder="Search tasks..."
      class="w-full border p-2 rounded mb-4"
    />
    
    <div class="flex gap-2 mb-4">
      <button
        v-for="status in ['all', 'active', 'completed'] as const"
        :key="status"
        @click="filterStatus = status"
        :class="[
          'px-4 py-2 rounded capitalize',
          filterStatus === status ? 'bg-blue-500 text-white' : 'bg-gray-200'
        ]"
      >
        {{ status }} ({{ status === 'all' ? taskStats.total : status === 'active' ? taskStats.active : taskStats.completed }})
      </button>
    </div>
    
    <ul class="space-y-2">
      <li
        v-for="task in filteredTasks"
        :key="task.id"
        class="flex items-center gap-2 p-2 bg-gray-50 rounded"
      >
        <input
          type="checkbox"
          :checked="task.completed"
          @change="toggleTask(task.id)"
        />
        <span :class="{ 'line-through text-gray-400': task.completed }">
          {{ task.title }}
        </span>
        <span :class="[
          'text-xs px-2 py-1 rounded ml-auto',
          task.priority === 'high' ? 'bg-red-100 text-red-700' :
          task.priority === 'medium' ? 'bg-yellow-100 text-yellow-700' :
          'bg-green-100 text-green-700'
        ]">
          {{ task.priority }}
        </span>
      </li>
    </ul>
    
    <p v-if="filteredTasks.length === 0" class="text-gray-500 text-center py-4">
      No tasks found
    </p>
  </div>
</template>
```

---

**Drill 4 (Edge Case): Writable Computed for Two-Way Binding**

Create a temperature converter using writable computed.

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

// TODO: Create ref 'celsius' starting at 0
// TODO: Create writable computed 'fahrenheit' that:
//       - get: converts celsius to fahrenheit
//       - set: converts input back to celsius and updates the ref

</script>

<template>
  <!-- TODO: Input for Celsius -->
  <!-- TODO: Input for Fahrenheit (bound to writable computed) -->
  <!-- Both should sync in real-time -->
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

const celsius = ref(0)

const fahrenheit = computed({
  get() {
    return (celsius.value * 9/5) + 32
  },
  set(value: number) {
    celsius.value = (value - 32) * 5/9
  }
})

const roundedCelsius = computed(() => Math.round(celsius.value * 100) / 100)
const roundedFahrenheit = computed(() => Math.round(fahrenheit.value * 100) / 100)
</script>

<template>
  <div class="p-4 max-w-md">
    <h2 class="text-xl font-bold mb-4">Temperature Converter</h2>
    
    <div class="grid grid-cols-2 gap-4">
      <div>
        <label class="block text-sm font-medium mb-1">Celsius</label>
        <input
          v-model.number="celsius"
          type="number"
          step="0.1"
          class="w-full border p-2 rounded text-center text-xl"
        />
      </div>
      
      <div>
        <label class="block text-sm font-medium mb-1">Fahrenheit</label>
        <input
          v-model.number="fahrenheit"
          type="number"
          step="0.1"
          class="w-full border p-2 rounded text-center text-xl"
        />
      </div>
    </div>
    
    <p class="text-center mt-4 text-gray-600">
      {{ roundedCelsius }}°C = {{ roundedFahrenheit }}°F
    </p>
  </div>
</template>
```

---

## 5. Common Pitfalls & Anti-Patterns

| ❌ Common Mistake | ✅ Correct Approach | Why It Matters |
|:--- |:--- |:--- |
| Calling computed as function: `{{ total() }}` | Access as property: `{{ total }}` | Computed is already evaluated; calling it again is wrong syntax |
| Using computed for side effects | Use `watch()` for side effects | Computed should be pure — only calculate and return |
| Modifying source data inside getter | Keep getters pure, use setter for mutations | Side effects in getters cause unpredictable behavior |
| Creating computed for static values | Use regular `const` for non-reactive values | Computed overhead not needed for values that never change |

**Code Examples:**

```typescript
// ❌ Side effect in computed getter
const badComputed = computed(() => {
  localStorage.setItem('count', count.value.toString())  // Side effect!
  return count.value * 2
})

// ✅ Use watch for side effects
const doubled = computed(() => count.value * 2)
watch(count, (newVal) => {
  localStorage.setItem('count', newVal.toString())  // Side effect here
})

// ❌ Mutating source data in getter
const badMutation = computed(() => {
  items.value.sort()  // Mutates the original array!
  return items.value
})

// ✅ Return new array, don't mutate
const safeSorted = computed(() => {
  return [...items.value].sort()  // Spread creates a copy
})
```

---

## 6. Mini-Project: Live Character Counter

**File Structure:**
```
src/
├── components/
│   └── CharacterCounter.vue
├── App.vue
└── main.ts
```

**components/CharacterCounter.vue:**
```vue
<script setup lang="ts">
// ============================================
// LIVE CHARACTER COUNTER COMPONENT
// Social media post composer with character limits
// ============================================

import { ref, computed } from 'vue'

// ============================================
// CONFIGURATION
// ============================================

interface PlatformConfig {
  name: string
  maxChars: number
  icon: string
}

const platforms: PlatformConfig[] = [
  { name: 'Twitter', maxChars: 280, icon: '🐦' },
  { name: 'LinkedIn', maxChars: 3000, icon: '💼' },
  { name: 'SMS', maxChars: 160, icon: '📱' }
]

// ============================================
// SOURCE STATE
// ============================================

const text = ref('')
const selectedPlatform = ref<PlatformConfig>(platforms[0])

// ============================================
// COMPUTED PROPERTIES (Derived State)
// ============================================

const charCount = computed(() => text.value.length)

const maxChars = computed(() => selectedPlatform.value.maxChars)

const remainingChars = computed(() => maxChars.value - charCount.value)

const percentUsed = computed(() => {
  return Math.min((charCount.value / maxChars.value) * 100, 100)
})

const isOverLimit = computed(() => charCount.value > maxChars.value)

const isNearLimit = computed(() => {
  const threshold = maxChars.value * 0.9
  return charCount.value >= threshold && !isOverLimit.value
})

const wordCount = computed(() => {
  if (!text.value.trim()) return 0
  return text.value.trim().split(/\s+/).length
})

const estimatedReadTime = computed(() => {
  const wordsPerMinute = 200
  const minutes = wordCount.value / wordsPerMinute
  if (minutes < 1) return 'Less than 1 min'
  return `${Math.ceil(minutes)} min read`
})

const progressBarColor = computed(() => {
  if (isOverLimit.value) return 'bg-red-500'
  if (isNearLimit.value) return 'bg-yellow-500'
  return 'bg-blue-500'
})

const statusMessage = computed(() => {
  if (charCount.value === 0) {
    return 'Start typing your message...'
  }
  if (isOverLimit.value) {
    return `⚠️ ${Math.abs(remainingChars.value)} characters over limit!`
  }
  if (isNearLimit.value) {
    return `Almost there! ${remainingChars.value} characters remaining.`
  }
  return `${remainingChars.value} characters remaining`
})

const canPost = computed(() => {
  return charCount.value > 0 && !isOverLimit.value
})

// ============================================
// ACTIONS
// ============================================

const clearText = (): void => {
  text.value = ''
}

const selectPlatform = (platform: PlatformConfig): void => {
  selectedPlatform.value = platform
}

const simulatePost = (): void => {
  if (canPost.value) {
    alert(`Posted to ${selectedPlatform.value.name}:\n\n${text.value}`)
    clearText()
  }
}
</script>

<template>
  <div class="max-w-xl mx-auto bg-white rounded-2xl shadow-xl overflow-hidden">
    <div class="bg-gradient-to-r from-blue-500 to-purple-600 p-6 text-white">
      <h2 class="text-xl font-bold">{{ selectedPlatform.icon }} {{ selectedPlatform.name }} Composer</h2>
      <p class="text-blue-100">Max {{ maxChars }} characters</p>
    </div>

    <div class="p-6">
      <div class="flex gap-2 mb-4">
        <button
          v-for="platform in platforms"
          :key="platform.name"
          @click="selectPlatform(platform)"
          :class="[
            'px-4 py-2 rounded-lg transition',
            selectedPlatform.name === platform.name
              ? 'bg-blue-500 text-white'
              : 'bg-gray-100 hover:bg-gray-200'
          ]"
        >
          {{ platform.icon }} {{ platform.name }}
        </button>
      </div>

      <textarea
        v-model="text"
        :class="[
          'w-full border-2 rounded-lg p-4 text-lg resize-none transition-colors',
          isOverLimit ? 'border-red-500 bg-red-50' : 
          isNearLimit ? 'border-yellow-500 bg-yellow-50' : 
          'border-gray-200 focus:border-blue-500'
        ]"
        rows="5"
        :placeholder="`What's on your mind? (${maxChars} chars max)`"
      ></textarea>

      <div class="mt-2 mb-4">
        <div class="w-full bg-gray-200 rounded-full h-2 overflow-hidden">
          <div
            :class="['h-full transition-all duration-300', progressBarColor]"
            :style="{ width: `${percentUsed}%` }"
          ></div>
        </div>
      </div>

      <div class="flex justify-between items-center text-sm mb-4">
        <span :class="[
          isOverLimit ? 'text-red-500 font-semibold' :
          isNearLimit ? 'text-yellow-600' : 'text-gray-500'
        ]">
          {{ statusMessage }}
        </span>
        <span class="text-gray-400">
          {{ charCount }} / {{ maxChars }}
        </span>
      </div>

      <div class="grid grid-cols-2 gap-4 text-sm text-gray-600 mb-6 bg-gray-50 p-4 rounded-lg">
        <div>
          <span class="font-medium">Words:</span> {{ wordCount }}
        </div>
        <div>
          <span class="font-medium">Read time:</span> {{ estimatedReadTime }}
        </div>
        <div>
          <span class="font-medium">Characters:</span> {{ charCount }}
        </div>
        <div>
          <span class="font-medium">Progress:</span> {{ percentUsed.toFixed(0) }}%
        </div>
      </div>

      <div class="flex gap-2">
        <button
          @click="simulatePost"
          :disabled="!canPost"
          :class="[
            'flex-1 py-3 px-6 rounded-lg font-semibold transition',
            canPost
              ? 'bg-blue-500 hover:bg-blue-600 text-white'
              : 'bg-gray-200 text-gray-400 cursor-not-allowed'
          ]"
        >
          Post to {{ selectedPlatform.name }}
        </button>
        <button
          @click="clearText"
          class="py-3 px-6 rounded-lg bg-gray-100 hover:bg-gray-200 transition"
        >
          Clear
        </button>
      </div>
    </div>
  </div>
</template>
```

**App.vue:**
```vue
<script setup lang="ts">
import CharacterCounter from './components/CharacterCounter.vue'
</script>

<template>
  <div class="min-h-screen bg-gray-100 py-12">
    <h1 class="text-3xl font-bold text-center text-gray-800 mb-8">
      Social Media Composer
    </h1>
    <CharacterCounter />
  </div>
</template>
```

**How to Test:**

1. Type in the textarea and watch character count update in real-time
2. Switch between platforms and observe the limit change
3. Type past 90% of the limit to see the yellow warning state
4. Exceed the limit to see the red error state
5. Notice the progress bar color changes with state
6. Check word count and read time update as you type
7. Try posting — button is disabled when over limit or empty

---

## 7. Implementation Exercises (Progressive)

### Level 1 (Recall): Fill in the Blanks

```vue
<script setup lang="ts">
import { ref, _____ } from 'vue'

const price = ref(50)
const quantity = ref(3)

const total = _____(() => {
  return _____.value * quantity._____
})
</script>

<template>
  <p>Total: ${{ _____ }}</p>
</template>
```

**Answer:** `computed`, `computed`, `price`, `value`, `total`

---

### Level 2 (Apply): Small Coding Tasks

**Exercise 2.1:** Create a `PasswordStrength.vue` component with:
- A `password` ref
- Computed properties for: `length`, `hasUppercase`, `hasLowercase`, `hasNumber`, `hasSpecial`
- A computed `strength` that returns 'weak', 'medium', 'strong' based on criteria met
- Visual strength indicator

**Exercise 2.2:** Create a `ShoppingCart.vue` component with:
- A `cartItems` ref (array of { name, price, quantity })
- Computed: `itemCount`, `subtotal`, `tax` (8%), `total`
- Computed: `isEmpty`, `freeShipping` (if total > 50)
- Functions to add/remove items

---

### Level 3 (Debug): Fix the Broken Code

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

const numbers = ref([1, 2, 3, 4, 5])

const doubled = computed(() => {
  numbers.value.forEach((n, i) => {
    numbers.value[i] = n * 2
  })
  return numbers.value
})

const sum = computed(() => {
  return numbers.reduce((a, b) => a + b, 0)
})

const addNumber = () => {
  numbers.push(6)
}
</script>

<template>
  <p>Numbers: {{ numbers }}</p>
  <p>Doubled: {{ doubled() }}</p>
  <p>Sum: {{ sum }}</p>
  <button @click="addNumber">Add 6</button>
</template>
```

**Bugs to find:**
1. Mutating source array inside computed getter
2. Missing `.value` when accessing `numbers` in `sum`
3. Missing `.value` when pushing to `numbers`
4. Calling computed as function with `()`

**Fixed Version:**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

const numbers = ref([1, 2, 3, 4, 5])

const doubled = computed(() => {
  return numbers.value.map(n => n * 2)
})

const sum = computed(() => {
  return numbers.value.reduce((a, b) => a + b, 0)
})

const addNumber = (): void => {
  numbers.value.push(6)
}
</script>

<template>
  <p>Numbers: {{ numbers }}</p>
  <p>Doubled: {{ doubled }}</p>
  <p>Sum: {{ sum }}</p>
  <button @click="addNumber">Add 6</button>
</template>
```

---

## Self-Check Solutions

**Exercise 2.1 Solution:**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

const password = ref('')

const length = computed(() => password.value.length)
const hasUppercase = computed(() => /[A-Z]/.test(password.value))
const hasLowercase = computed(() => /[a-z]/.test(password.value))
const hasNumber = computed(() => /[0-9]/.test(password.value))
const hasSpecial = computed(() => /[!@#$%^&*]/.test(password.value))

const criteriaCount = computed(() => {
  let count = 0
  if (length.value >= 8) count++
  if (hasUppercase.value) count++
  if (hasLowercase.value) count++
  if (hasNumber.value) count++
  if (hasSpecial.value) count++
  return count
})

const strength = computed<'weak' | 'medium' | 'strong'>(() => {
  if (criteriaCount.value <= 2) return 'weak'
  if (criteriaCount.value <= 4) return 'medium'
  return 'strong'
})

const strengthColor = computed(() => ({
  weak: 'bg-red-500',
  medium: 'bg-yellow-500',
  strong: 'bg-green-500'
}[strength.value]))
</script>

<template>
  <div class="p-4 max-w-md">
    <input
      v-model="password"
      type="password"
      placeholder="Enter password"
      class="w-full border p-2 rounded mb-4"
    />
    
    <div class="h-2 bg-gray-200 rounded overflow-hidden mb-2">
      <div
        :class="['h-full transition-all', strengthColor]"
        :style="{ width: `${(criteriaCount / 5) * 100}%` }"
      ></div>
    </div>
    
    <p class="font-semibold capitalize mb-2">Strength: {{ strength }}</p>
    
    <ul class="text-sm space-y-1">
      <li :class="length >= 8 ? 'text-green-600' : 'text-gray-400'">
        {{ length >= 8 ? '✓' : '○' }} At least 8 characters ({{ length }})
      </li>
      <li :class="hasUppercase ? 'text-green-600' : 'text-gray-400'">
        {{ hasUppercase ? '✓' : '○' }} Uppercase letter
      </li>
      <li :class="hasLowercase ? 'text-green-600' : 'text-gray-400'">
        {{ hasLowercase ? '✓' : '○' }} Lowercase letter
      </li>
      <li :class="hasNumber ? 'text-green-600' : 'text-gray-400'">
        {{ hasNumber ? '✓' : '○' }} Number
      </li>
      <li :class="hasSpecial ? 'text-green-600' : 'text-gray-400'">
        {{ hasSpecial ? '✓' : '○' }} Special character (!@#$%^&*)
      </li>
    </ul>
  </div>
</template>
```

**Exercise 2.2 Solution:**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

interface CartItem {
  id: number
  name: string
  price: number
  quantity: number
}

const cartItems = ref<CartItem[]>([
  { id: 1, name: 'T-Shirt', price: 25, quantity: 2 },
  { id: 2, name: 'Jeans', price: 50, quantity: 1 }
])

const TAX_RATE = 0.08
const FREE_SHIPPING_THRESHOLD = 50

const itemCount = computed(() => 
  cartItems.value.reduce((sum, item) => sum + item.quantity, 0)
)

const subtotal = computed(() =>
  cartItems.value.reduce((sum, item) => sum + item.price * item.quantity, 0)
)

const tax = computed(() => subtotal.value * TAX_RATE)

const total = computed(() => subtotal.value + tax.value)

const isEmpty = computed(() => cartItems.value.length === 0)

const freeShipping = computed(() => subtotal.value >= FREE_SHIPPING_THRESHOLD)

const removeItem = (id: number): void => {
  const index = cartItems.value.findIndex(item => item.id === id)
  if (index > -1) cartItems.value.splice(index, 1)
}

const updateQuantity = (id: number, delta: number): void => {
  const item = cartItems.value.find(i => i.id === id)
  if (item) {
    item.quantity = Math.max(1, item.quantity + delta)
  }
}
</script>

<template>
  <div class="p-4 max-w-md">
    <h2 class="text-xl font-bold mb-4">Shopping Cart</h2>
    
    <div v-if="isEmpty" class="text-center text-gray-500 py-8">
      Your cart is empty
    </div>
    
    <div v-else class="space-y-4">
      <div v-for="item in cartItems" :key="item.id" class="flex items-center gap-4 bg-gray-50 p-3 rounded">
        <div class="flex-1">
          <p class="font-medium">{{ item.name }}</p>
          <p class="text-sm text-gray-500">${{ item.price }} each</p>
        </div>
        <div class="flex items-center gap-2">
          <button @click="updateQuantity(item.id, -1)" class="px-2 bg-gray-200 rounded">-</button>
          <span>{{ item.quantity }}</span>
          <button @click="updateQuantity(item.id, 1)" class="px-2 bg-gray-200 rounded">+</button>
        </div>
        <button @click="removeItem(item.id)" class="text-red-500">×</button>
      </div>
      
      <div class="border-t pt-4 space-y-2">
        <div class="flex justify-between">
          <span>Items ({{ itemCount }})</span>
          <span>${{ subtotal.toFixed(2) }}</span>
        </div>
        <div class="flex justify-between text-gray-600">
          <span>Tax (8%)</span>
          <span>${{ tax.toFixed(2) }}</span>
        </div>
        <div class="flex justify-between text-gray-600">
          <span>Shipping</span>
          <span :class="freeShipping ? 'text-green-600' : ''">
            {{ freeShipping ? 'FREE' : '$5.00' }}
          </span>
        </div>
        <div class="flex justify-between font-bold text-lg border-t pt-2">
          <span>Total</span>
          <span>${{ (total + (freeShipping ? 0 : 5)).toFixed(2) }}</span>
        </div>
      </div>
      
      <p v-if="!freeShipping" class="text-sm text-gray-500">
        Add ${{ (FREE_SHIPPING_THRESHOLD - subtotal).toFixed(2) }} more for free shipping!
      </p>
    </div>
  </div>
</template>
```

---

*Reply 'next' for Lesson 5.1 (Theory).*
