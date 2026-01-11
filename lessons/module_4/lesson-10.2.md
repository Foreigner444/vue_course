# Lesson 10.2: List Rendering — Practice & Application (Отрисовка списков)

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 10.2 of 25 (Part 2 of 2) |
| **Topic** | List Rendering: `v-for` / Отрисовка списков |
| **Continues From** | Lesson 10.1 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Build components that render dynamic lists with proper key management
2. **Analyze:** Combine `v-for` with `v-if`, computed properties, and array methods for filtered/sorted lists

---

## 2. Quick Recap

- **`v-for`** renders a template once for each item in an array or properties of an object
- **`:key`** is required and must be a unique, stable identifier (prefer IDs over indexes)
- **Syntax variants:** `item in items`, `(item, index) in items`, `(value, key) in object`
- **Range:** `n in 10` iterates 1 through 10 (not 0-9)
- **Never use `v-if` and `v-for` on the same element** — use computed properties or `<template>` wrapper

---

## 3. The Variable Frame & Complexity Scale

### Basic Form: Simple Array Rendering

```vue
<script setup lang="ts">
import { ref } from 'vue'

const fruits = ref<string[]>(['Apple', 'Banana', 'Cherry', 'Date'])
</script>

<template>
  <ul>
    <li v-for="fruit in fruits" :key="fruit">
      {{ fruit }}
    </li>
  </ul>
</template>
```

### With Options: Array of Objects with Index

```vue
<script setup lang="ts">
import { ref } from 'vue'

interface Product {
  id: number
  name: string
  price: number
  inStock: boolean
}

const products = ref<Product[]>([
  { id: 1, name: 'Laptop', price: 999, inStock: true },
  { id: 2, name: 'Mouse', price: 29, inStock: true },
  { id: 3, name: 'Keyboard', price: 79, inStock: false },
])
</script>

<template>
  <div class="overflow-x-auto">
    <table class="table">
      <thead>
        <tr>
          <th>#</th>
          <th>Product</th>
          <th>Price</th>
          <th>Status</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(product, index) in products" :key="product.id">
          <td>{{ index + 1 }}</td>
          <td>{{ product.name }}</td>
          <td>${{ product.price }}</td>
          <td>
            <span 
              class="badge"
              :class="product.inStock ? 'badge-success' : 'badge-error'"
            >
              {{ product.inStock ? 'In Stock' : 'Out of Stock' }}
            </span>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>
```

### Advanced Form: Filtered & Sorted Lists with Computed

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

interface Task {
  id: number
  title: string
  completed: boolean
  priority: 'low' | 'medium' | 'high'
  dueDate: string
}

const tasks = ref<Task[]>([
  { id: 1, title: 'Review PR', completed: false, priority: 'high', dueDate: '2024-01-15' },
  { id: 2, title: 'Write tests', completed: true, priority: 'medium', dueDate: '2024-01-14' },
  { id: 3, title: 'Update docs', completed: false, priority: 'low', dueDate: '2024-01-16' },
  { id: 4, title: 'Fix bug', completed: false, priority: 'high', dueDate: '2024-01-13' },
])

type FilterType = 'all' | 'active' | 'completed'
type SortType = 'priority' | 'dueDate' | 'title'

const filterType = ref<FilterType>('all')
const sortType = ref<SortType>('priority')

const priorityOrder = { high: 0, medium: 1, low: 2 }

const filteredAndSortedTasks = computed(() => {
  let result = [...tasks.value]
  
  if (filterType.value === 'active') {
    result = result.filter(t => !t.completed)
  } else if (filterType.value === 'completed') {
    result = result.filter(t => t.completed)
  }
  
  result.sort((a, b) => {
    switch (sortType.value) {
      case 'priority':
        return priorityOrder[a.priority] - priorityOrder[b.priority]
      case 'dueDate':
        return new Date(a.dueDate).getTime() - new Date(b.dueDate).getTime()
      case 'title':
        return a.title.localeCompare(b.title)
      default:
        return 0
    }
  })
  
  return result
})
</script>

<template>
  <div class="space-y-4">
    <div class="flex gap-4">
      <select v-model="filterType" class="select select-bordered">
        <option value="all">All Tasks</option>
        <option value="active">Active Only</option>
        <option value="completed">Completed Only</option>
      </select>
      
      <select v-model="sortType" class="select select-bordered">
        <option value="priority">Sort by Priority</option>
        <option value="dueDate">Sort by Due Date</option>
        <option value="title">Sort by Title</option>
      </select>
    </div>
    
    <ul class="space-y-2">
      <li 
        v-for="task in filteredAndSortedTasks" 
        :key="task.id"
        class="p-3 bg-base-200 rounded flex justify-between items-center"
      >
        <span :class="{ 'line-through opacity-50': task.completed }">
          {{ task.title }}
        </span>
        <div class="flex gap-2">
          <span class="badge badge-outline">{{ task.dueDate }}</span>
          <span 
            class="badge"
            :class="{
              'badge-error': task.priority === 'high',
              'badge-warning': task.priority === 'medium',
              'badge-info': task.priority === 'low'
            }"
          >
            {{ task.priority }}
          </span>
        </div>
      </li>
    </ul>
    
    <p v-if="filteredAndSortedTasks.length === 0" class="text-center opacity-50">
      No tasks match the current filter.
    </p>
  </div>
</template>
```

---

## 4. Micro-Implementation Drills

### Drill 1 (Basic): Render a Simple List

Create a component that displays a list of programming languages.

```vue
<script setup lang="ts">
import { ref } from 'vue'

// TODO: Create a ref with an array of programming language names
const languages = ref([])
</script>

<template>
  <div class="p-4">
    <h2 class="text-xl font-bold mb-4">Programming Languages</h2>
    
    <!-- TODO: Use v-for to render each language as a list item -->
    <!-- Remember to add :key -->
    <ul class="list-disc list-inside">
      <li>Language here</li>
    </ul>
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const languages = ref<string[]>([
  'TypeScript',
  'JavaScript', 
  'Python',
  'Rust',
  'Go'
])
</script>

<template>
  <div class="p-4">
    <h2 class="text-xl font-bold mb-4">Programming Languages</h2>
    
    <ul class="list-disc list-inside">
      <li v-for="language in languages" :key="language">
        {{ language }}
      </li>
    </ul>
  </div>
</template>
```

---

### Drill 2 (Variation): Object Iteration with Index

Render a user profile card that displays all properties of a user object.

```vue
<script setup lang="ts">
import { ref } from 'vue'

const userProfile = ref({
  name: 'Alice Johnson',
  email: 'alice@example.com',
  role: 'Senior Developer',
  department: 'Engineering',
  startDate: '2021-03-15',
  location: 'San Francisco'
})
</script>

<template>
  <div class="card bg-base-200 w-96">
    <div class="card-body">
      <h2 class="card-title">User Profile</h2>
      
      <!-- TODO: Use v-for to iterate over the object -->
      <!-- Display each property as "Label: Value" -->
      <!-- Use (value, key, index) syntax -->
      <div class="space-y-2">
        <div class="flex justify-between">
          <span class="font-semibold capitalize">Key:</span>
          <span>Value</span>
        </div>
      </div>
    </div>
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const userProfile = ref({
  name: 'Alice Johnson',
  email: 'alice@example.com',
  role: 'Senior Developer',
  department: 'Engineering',
  startDate: '2021-03-15',
  location: 'San Francisco'
})
</script>

<template>
  <div class="card bg-base-200 w-96">
    <div class="card-body">
      <h2 class="card-title">User Profile</h2>
      
      <div class="space-y-2">
        <div 
          v-for="(value, key, index) in userProfile" 
          :key="key"
          class="flex justify-between"
          :class="{ 'border-t pt-2': index > 0 }"
        >
          <span class="font-semibold capitalize">{{ key }}:</span>
          <span>{{ value }}</span>
        </div>
      </div>
    </div>
  </div>
</template>
```

---

### Drill 3 (Combination): v-for with v-if (Proper Pattern)

Build a component that shows a list of users with an "Online" badge, but only displays online users. Demonstrate the correct way to combine `v-for` with `v-if`.

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

interface User {
  id: number
  name: string
  avatar: string
  isOnline: boolean
}

const users = ref<User[]>([
  { id: 1, name: 'Alice', avatar: '👩', isOnline: true },
  { id: 2, name: 'Bob', avatar: '👨', isOnline: false },
  { id: 3, name: 'Charlie', avatar: '🧑', isOnline: true },
  { id: 4, name: 'Diana', avatar: '👩‍🦰', isOnline: false },
  { id: 5, name: 'Eve', avatar: '👱‍♀️', isOnline: true },
])

// TODO: Create a computed property that filters to only online users
// const onlineUsers = computed(() => ...)
</script>

<template>
  <div class="p-4">
    <h2 class="text-xl font-bold mb-4">Online Users</h2>
    
    <!-- 
      ❌ WRONG WAY (don't do this):
      <div v-for="user in users" v-if="user.isOnline" :key="user.id">
      
      ✅ CORRECT WAY: Use computed property to filter first
    -->
    
    <!-- TODO: Use v-for with the computed onlineUsers -->
    <div class="flex flex-wrap gap-4">
      <div class="flex items-center gap-2 p-2 bg-base-200 rounded-lg">
        <span class="text-2xl">Avatar</span>
        <span>Name</span>
        <span class="badge badge-success badge-sm">Online</span>
      </div>
    </div>
    
    <!-- TODO: Show message if no users are online -->
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

interface User {
  id: number
  name: string
  avatar: string
  isOnline: boolean
}

const users = ref<User[]>([
  { id: 1, name: 'Alice', avatar: '👩', isOnline: true },
  { id: 2, name: 'Bob', avatar: '👨', isOnline: false },
  { id: 3, name: 'Charlie', avatar: '🧑', isOnline: true },
  { id: 4, name: 'Diana', avatar: '👩‍🦰', isOnline: false },
  { id: 5, name: 'Eve', avatar: '👱‍♀️', isOnline: true },
])

const onlineUsers = computed(() => users.value.filter(user => user.isOnline))
</script>

<template>
  <div class="p-4">
    <h2 class="text-xl font-bold mb-4">
      Online Users ({{ onlineUsers.length }})
    </h2>
    
    <div class="flex flex-wrap gap-4">
      <div 
        v-for="user in onlineUsers" 
        :key="user.id"
        class="flex items-center gap-2 p-2 bg-base-200 rounded-lg"
      >
        <span class="text-2xl">{{ user.avatar }}</span>
        <span>{{ user.name }}</span>
        <span class="badge badge-success badge-sm">Online</span>
      </div>
    </div>
    
    <p v-if="onlineUsers.length === 0" class="text-center opacity-50 mt-4">
      No users are currently online.
    </p>
  </div>
</template>
```

---

### Drill 4 (Edge Case): Nested v-for with Unique Keys

Build a component that displays a course catalog with modules, where each module contains multiple lessons.

```vue
<script setup lang="ts">
import { ref } from 'vue'

interface Lesson {
  id: number
  title: string
  duration: number
}

interface Module {
  id: number
  name: string
  lessons: Lesson[]
}

const course = ref<Module[]>([
  {
    id: 1,
    name: 'Getting Started',
    lessons: [
      { id: 101, title: 'Introduction', duration: 5 },
      { id: 102, title: 'Setup', duration: 10 },
    ]
  },
  {
    id: 2,
    name: 'Core Concepts',
    lessons: [
      { id: 201, title: 'Reactivity', duration: 15 },
      { id: 202, title: 'Components', duration: 20 },
      { id: 203, title: 'Directives', duration: 12 },
    ]
  },
  {
    id: 3,
    name: 'Advanced Topics',
    lessons: [
      { id: 301, title: 'Composables', duration: 18 },
      { id: 302, title: 'Performance', duration: 25 },
    ]
  },
])
</script>

<template>
  <div class="p-4 space-y-6">
    <h1 class="text-2xl font-bold">Vue.js Course</h1>
    
    <!-- 
      TODO: Create nested v-for loops
      Outer loop: modules
      Inner loop: lessons within each module
      
      IMPORTANT: Each loop needs its own unique :key
      Keys only need to be unique among siblings, not globally
    -->
    
    <div class="card bg-base-200">
      <div class="card-body">
        <h2 class="card-title">Module Name</h2>
        <ul class="space-y-1">
          <li class="flex justify-between">
            <span>Lesson Title</span>
            <span class="opacity-70">X min</span>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

interface Lesson {
  id: number
  title: string
  duration: number
}

interface Module {
  id: number
  name: string
  lessons: Lesson[]
}

const course = ref<Module[]>([
  {
    id: 1,
    name: 'Getting Started',
    lessons: [
      { id: 101, title: 'Introduction', duration: 5 },
      { id: 102, title: 'Setup', duration: 10 },
    ]
  },
  {
    id: 2,
    name: 'Core Concepts',
    lessons: [
      { id: 201, title: 'Reactivity', duration: 15 },
      { id: 202, title: 'Components', duration: 20 },
      { id: 203, title: 'Directives', duration: 12 },
    ]
  },
  {
    id: 3,
    name: 'Advanced Topics',
    lessons: [
      { id: 301, title: 'Composables', duration: 18 },
      { id: 302, title: 'Performance', duration: 25 },
    ]
  },
])
</script>

<template>
  <div class="p-4 space-y-6">
    <h1 class="text-2xl font-bold">Vue.js Course</h1>
    
    <div 
      v-for="(module, moduleIndex) in course" 
      :key="module.id"
      class="card bg-base-200"
    >
      <div class="card-body">
        <h2 class="card-title">
          Module {{ moduleIndex + 1 }}: {{ module.name }}
        </h2>
        <ul class="space-y-1">
          <li 
            v-for="lesson in module.lessons" 
            :key="lesson.id"
            class="flex justify-between p-2 bg-base-100 rounded"
          >
            <span>{{ lesson.title }}</span>
            <span class="opacity-70">{{ lesson.duration }} min</span>
          </li>
        </ul>
        <p class="text-sm opacity-70 mt-2">
          {{ module.lessons.length }} lessons · 
          {{ module.lessons.reduce((sum, l) => sum + l.duration, 0) }} min total
        </p>
      </div>
    </div>
  </div>
</template>
```

---

## 5. Common Pitfalls & Anti-Patterns

| ❌ Common Mistake | ✅ Correct Approach | Why It Matters |
|:--- |:--- |:--- |
| Using array index as key when items can change | Use unique ID from data (`item.id`) | Index-based keys cause incorrect updates when items are reordered, added, or removed from the middle |
| Using `v-if` and `v-for` on the same element | Use computed property to filter, or wrap with `<template v-for>` | `v-if` has higher priority, evaluated for each iteration, causing performance issues and unexpected behavior |
| Mutating the array inside `v-for` | Mutate outside the loop or use array methods that return new arrays | Mutating while iterating can cause infinite loops or skipped items |
| Forgetting `:key` attribute | Always provide `:key` with a unique value | Vue will warn, and the app may have subtle rendering bugs during updates |

**Detailed Example: Index Key Problem**

```vue
<!-- ❌ PROBLEMATIC: Using index as key -->
<script setup lang="ts">
import { ref } from 'vue'

const items = ref(['A', 'B', 'C'])

const removeFirst = () => {
  items.value.shift()
}
</script>

<template>
  <div v-for="(item, index) in items" :key="index">
    <input :placeholder="item" />
  </div>
  <button @click="removeFirst">Remove First</button>
</template>

<!-- 
  BUG: Type something in the first input, then click "Remove First"
  The input VALUE stays in the first position even though the item changed!
  
  Why? Vue sees:
  - Before: key=0 has "A", key=1 has "B", key=2 has "C"
  - After:  key=0 has "B", key=1 has "C"
  Vue reuses the DOM nodes by key, so the input with user's text stays at key=0
-->

<!-- ✅ CORRECT: Use unique identifier -->
<script setup lang="ts">
import { ref } from 'vue'

const items = ref([
  { id: 1, value: 'A' },
  { id: 2, value: 'B' },
  { id: 3, value: 'C' }
])

const removeFirst = () => {
  items.value.shift()
}
</script>

<template>
  <div v-for="item in items" :key="item.id">
    <input :placeholder="item.value" />
  </div>
  <button @click="removeFirst">Remove First</button>
</template>

<!-- Now Vue correctly removes the first input because it tracks by id -->
```

---

## 6. Mini-Project: Filterable Product Grid

**File Structure:**
```
src/
├── components/
│   └── ProductGrid.vue
├── App.vue
└── main.ts
```

**ProductGrid.vue:**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

// Step 1: Define the product interface
// This gives us type safety and autocomplete for our product data
interface Product {
  id: number
  name: string
  category: 'electronics' | 'clothing' | 'books' | 'home'
  price: number
  rating: number
  inStock: boolean
  image: string
}

// Step 2: Create our product data
// In a real app, this would come from an API
const products = ref<Product[]>([
  { id: 1, name: 'Wireless Headphones', category: 'electronics', price: 149.99, rating: 4.5, inStock: true, image: '🎧' },
  { id: 2, name: 'Cotton T-Shirt', category: 'clothing', price: 29.99, rating: 4.2, inStock: true, image: '👕' },
  { id: 3, name: 'JavaScript Guide', category: 'books', price: 39.99, rating: 4.8, inStock: true, image: '📘' },
  { id: 4, name: 'Smart Watch', category: 'electronics', price: 299.99, rating: 4.3, inStock: false, image: '⌚' },
  { id: 5, name: 'Denim Jeans', category: 'clothing', price: 59.99, rating: 4.0, inStock: true, image: '👖' },
  { id: 6, name: 'Desk Lamp', category: 'home', price: 45.99, rating: 4.6, inStock: true, image: '💡' },
  { id: 7, name: 'TypeScript Handbook', category: 'books', price: 44.99, rating: 4.9, inStock: false, image: '📕' },
  { id: 8, name: 'Bluetooth Speaker', category: 'electronics', price: 79.99, rating: 4.1, inStock: true, image: '🔊' },
  { id: 9, name: 'Throw Pillow', category: 'home', price: 24.99, rating: 4.4, inStock: true, image: '🛋️' },
  { id: 10, name: 'Winter Jacket', category: 'clothing', price: 129.99, rating: 4.7, inStock: true, image: '🧥' },
])

// Step 3: Create filter state
// These refs control what products are displayed
const selectedCategory = ref<string>('all')
const showInStockOnly = ref<boolean>(false)
const searchQuery = ref<string>('')
const sortBy = ref<'price-asc' | 'price-desc' | 'rating' | 'name'>('name')

// Step 4: Extract unique categories for the filter dropdown
// Using Set to get unique values, then converting to array
const categories = computed(() => {
  const cats = new Set(products.value.map(p => p.category))
  return ['all', ...Array.from(cats)]
})

// Step 5: Create the filtered and sorted products computed property
// This is the KEY pattern: filter with computed, then use v-for on the result
const filteredProducts = computed(() => {
  let result = [...products.value]
  
  // Apply category filter
  if (selectedCategory.value !== 'all') {
    result = result.filter(p => p.category === selectedCategory.value)
  }
  
  // Apply in-stock filter
  if (showInStockOnly.value) {
    result = result.filter(p => p.inStock)
  }
  
  // Apply search filter (case-insensitive)
  if (searchQuery.value.trim()) {
    const query = searchQuery.value.toLowerCase()
    result = result.filter(p => 
      p.name.toLowerCase().includes(query)
    )
  }
  
  // Apply sorting
  switch (sortBy.value) {
    case 'price-asc':
      result.sort((a, b) => a.price - b.price)
      break
    case 'price-desc':
      result.sort((a, b) => b.price - a.price)
      break
    case 'rating':
      result.sort((a, b) => b.rating - a.rating)
      break
    case 'name':
      result.sort((a, b) => a.name.localeCompare(b.name))
      break
  }
  
  return result
})

// Step 6: Helper function to render star ratings
const getStars = (rating: number): string => {
  const fullStars = Math.floor(rating)
  const hasHalf = rating % 1 >= 0.5
  return '★'.repeat(fullStars) + (hasHalf ? '½' : '') + '☆'.repeat(5 - fullStars - (hasHalf ? 1 : 0))
}
</script>

<template>
  <div class="p-6 max-w-6xl mx-auto">
    <!-- Header -->
    <h1 class="text-3xl font-bold mb-6">Product Catalog</h1>
    
    <!-- Filters Section -->
    <div class="bg-base-200 p-4 rounded-lg mb-6">
      <div class="flex flex-wrap gap-4 items-end">
        <!-- Search Input -->
        <div class="form-control">
          <label class="label">
            <span class="label-text">Search</span>
          </label>
          <input 
            v-model="searchQuery"
            type="text" 
            placeholder="Search products..." 
            class="input input-bordered w-64"
          />
        </div>
        
        <!-- Category Filter -->
        <div class="form-control">
          <label class="label">
            <span class="label-text">Category</span>
          </label>
          <select v-model="selectedCategory" class="select select-bordered">
            <option 
              v-for="category in categories" 
              :key="category" 
              :value="category"
            >
              {{ category === 'all' ? 'All Categories' : category }}
            </option>
          </select>
        </div>
        
        <!-- Sort Options -->
        <div class="form-control">
          <label class="label">
            <span class="label-text">Sort By</span>
          </label>
          <select v-model="sortBy" class="select select-bordered">
            <option value="name">Name (A-Z)</option>
            <option value="price-asc">Price (Low to High)</option>
            <option value="price-desc">Price (High to Low)</option>
            <option value="rating">Rating (Best First)</option>
          </select>
        </div>
        
        <!-- In-Stock Toggle -->
        <div class="form-control">
          <label class="label cursor-pointer gap-2">
            <input 
              v-model="showInStockOnly"
              type="checkbox" 
              class="checkbox checkbox-primary"
            />
            <span class="label-text">In Stock Only</span>
          </label>
        </div>
      </div>
      
      <!-- Results Count -->
      <p class="mt-4 text-sm opacity-70">
        Showing {{ filteredProducts.length }} of {{ products.length }} products
      </p>
    </div>
    
    <!-- Products Grid -->
    <!-- Using v-for on the COMPUTED filtered array, not the original -->
    <div 
      v-if="filteredProducts.length > 0"
      class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4"
    >
      <div 
        v-for="product in filteredProducts" 
        :key="product.id"
        class="card bg-base-100 shadow-md hover:shadow-lg transition-shadow"
      >
        <div class="card-body">
          <!-- Product Image (emoji placeholder) -->
          <div class="text-6xl text-center mb-2">{{ product.image }}</div>
          
          <!-- Product Name -->
          <h2 class="card-title text-lg">{{ product.name }}</h2>
          
          <!-- Category Badge -->
          <div class="flex gap-2">
            <span class="badge badge-outline capitalize">
              {{ product.category }}
            </span>
            <span 
              v-if="!product.inStock" 
              class="badge badge-error"
            >
              Out of Stock
            </span>
          </div>
          
          <!-- Rating -->
          <div class="text-yellow-500">
            {{ getStars(product.rating) }}
            <span class="text-base-content opacity-70 ml-1">
              ({{ product.rating }})
            </span>
          </div>
          
          <!-- Price -->
          <div class="text-2xl font-bold text-primary">
            ${{ product.price.toFixed(2) }}
          </div>
          
          <!-- Add to Cart Button -->
          <div class="card-actions justify-end mt-2">
            <button 
              class="btn btn-primary btn-sm"
              :disabled="!product.inStock"
            >
              {{ product.inStock ? 'Add to Cart' : 'Unavailable' }}
            </button>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Empty State -->
    <!-- Using v-else after v-if for the empty state -->
    <div v-else class="text-center py-12">
      <div class="text-6xl mb-4">🔍</div>
      <h3 class="text-xl font-semibold mb-2">No products found</h3>
      <p class="opacity-70 mb-4">
        Try adjusting your filters or search query.
      </p>
      <button 
        class="btn btn-outline"
        @click="selectedCategory = 'all'; showInStockOnly = false; searchQuery = ''"
      >
        Clear All Filters
      </button>
    </div>
  </div>
</template>
```

**How to Test:**
1. Create a new Vue project with Tailwind CSS and DaisyUI
2. Copy `ProductGrid.vue` to `src/components/`
3. Update `App.vue` to import and use the component
4. Run `npm run dev` and test the following:
   - Search for "wireless" — only matching products should appear
   - Select "electronics" category — only electronics products shown
   - Check "In Stock Only" — out-of-stock items hidden
   - Change sort order — verify products reorder
   - Combine filters — verify they work together
   - Clear filters when showing "No products found"

---

## 7. Implementation Exercises (Progressive)

### Level 1 (Recall): Fill in the Blanks

Complete the v-for syntax:

```vue
<script setup lang="ts">
import { ref } from 'vue'

const colors = ref([
  { id: 1, name: 'Red', hex: '#FF0000' },
  { id: 2, name: 'Green', hex: '#00FF00' },
  { id: 3, name: 'Blue', hex: '#0000FF' },
])
</script>

<template>
  <ul>
    <!-- Fill in the blanks -->
    <li 
      v-___="_____ in colors" 
      :___="color.id"
      :style="{ backgroundColor: color.hex }"
      class="p-2 text-white"
    >
      {{ color.name }}: {{ color.hex }}
    </li>
  </ul>
</template>
```

**Solution:**
```vue
<li 
  v-for="color in colors" 
  :key="color.id"
  :style="{ backgroundColor: color.hex }"
  class="p-2 text-white"
>
```

---

### Level 2 (Apply): Independent Coding Tasks

**Exercise 2.1:** Create a `NumberRange.vue` component that uses `v-for="n in X"` to render a row of numbered buttons. The user can input a number (1-20), and that many buttons should appear.

**Requirements:**
- Input field for the range limit
- Buttons rendered using range iteration
- Each button shows its number
- Clicking a button logs "Button X clicked" to console

**Exercise 2.2:** Create a `TagList.vue` component with an input to add tags and a list to display them. Users should be able to add and remove tags.

**Requirements:**
- Input + button to add new tags
- Tags displayed using `v-for` with unique keys
- Each tag has a remove button (✕)
- Prevent duplicate tags
- Show empty state when no tags exist

---

### Level 3 (Debug): Fix the Broken Code

Find and fix the bugs in this code:

```vue
<script setup lang="ts">
import { ref } from 'vue'

interface Todo {
  id: number
  text: string
  done: boolean
}

const todos = ref<Todo[]>([
  { id: 1, text: 'Learn Vue', done: false },
  { id: 2, text: 'Build app', done: false },
  { id: 3, text: 'Deploy', done: false },
])

let nextId = 4

const addTodo = () => {
  todos.value.push({ id: nextId++, text: 'New todo', done: false })
}

const removeTodo = (index: number) => {
  todos.value.splice(index, 1)
}
</script>

<template>
  <div class="p-4">
    <!-- Bug 1: Missing something important -->
    <div v-for="(todo, index) in todos">
      <input type="checkbox" v-model="todo.done" />
      <span :class="{ 'line-through': todo.done }">
        {{ todo.text }}
      </span>
      <button @click="removeTodo(index)">Delete</button>
    </div>
    
    <!-- Bug 2: This v-if/v-for combo is problematic -->
    <div v-for="todo in todos" v-if="!todo.done" :key="todo.id">
      Active: {{ todo.text }}
    </div>
    
    <button @click="addTodo">Add Todo</button>
  </div>
</template>
```

**Solution:**

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

interface Todo {
  id: number
  text: string
  done: boolean
}

const todos = ref<Todo[]>([
  { id: 1, text: 'Learn Vue', done: false },
  { id: 2, text: 'Build app', done: false },
  { id: 3, text: 'Deploy', done: false },
])

let nextId = 4

const addTodo = () => {
  todos.value.push({ id: nextId++, text: 'New todo', done: false })
}

const removeTodo = (id: number) => {
  const index = todos.value.findIndex(t => t.id === id)
  if (index !== -1) {
    todos.value.splice(index, 1)
  }
}

const activeTodos = computed(() => todos.value.filter(t => !t.done))
</script>

<template>
  <div class="p-4">
    <!-- Fix 1: Added :key="todo.id" -->
    <div v-for="todo in todos" :key="todo.id">
      <input type="checkbox" v-model="todo.done" />
      <span :class="{ 'line-through': todo.done }">
        {{ todo.text }}
      </span>
      <!-- Fix 1b: Use id instead of index for removal -->
      <button @click="removeTodo(todo.id)">Delete</button>
    </div>
    
    <!-- Fix 2: Separated v-if from v-for using computed property -->
    <div v-for="todo in activeTodos" :key="todo.id">
      Active: {{ todo.text }}
    </div>
    
    <button @click="addTodo">Add Todo</button>
  </div>
</template>
```

**Bugs explained:**
1. **Bug 1a:** Missing `:key` attribute on the first `v-for`. Always required for proper DOM tracking.
2. **Bug 1b:** Using `index` for removal is fragile — after removing an item, indexes shift. Using `id` is more reliable.
3. **Bug 2:** `v-for` and `v-if` on the same element is an anti-pattern. Vue evaluates `v-if` on every iteration, and it's confusing which has priority. Use a computed property to filter first.

---

*Reply 'next' for Lesson 11.1 (Theory) — Module 5: User Interaction begins!*
