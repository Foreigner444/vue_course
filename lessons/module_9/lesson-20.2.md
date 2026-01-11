# Lesson 20.2: State Management I — Pinia Stores (State & Actions) — Practice & Application
# Управление состоянием I: Хранилища Pinia (Состояние и Действия) — Практика и Применение

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 20.2 of 25 (Part 2 of 2) |
| **Topic** | Pinia Stores (State & Actions) / Хранилища Pinia (Состояние и Действия) |
| **Continues From** | Lesson 20.1 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Create a Pinia store with typed state and multiple actions
2. **Analyze:** Debug common Pinia mistakes and compare store patterns

---

## 2. Quick Recap

- **Pinia stores** are centralized reactive containers accessible from any component
- **State** is defined using `ref()` or `reactive()` inside a setup function
- **Actions** are regular functions that modify state
- The **Setup Store** syntax (`defineStore('id', () => { ... })`) mirrors `<script setup>` patterns
- Components access stores by calling the generated hook: `const store = useMyStore()`

---

## 3. The Variable Frame & Complexity Scale

### Basic Form: Simple Counter Store

```typescript
// src/stores/counter.ts
import { defineStore } from 'pinia'
import { ref } from 'vue'

export const useCounterStore = defineStore('counter', () => {
  const count = ref(0)

  function increment() {
    count.value++
  }

  return { count, increment }
})
```

### With Options: Typed State with Multiple Properties

```typescript
// src/stores/user.ts
import { defineStore } from 'pinia'
import { ref } from 'vue'

interface User {
  id: number
  name: string
  email: string
}

export const useUserStore = defineStore('user', () => {
  const currentUser = ref<User | null>(null)
  const isAuthenticated = ref(false)
  const loginError = ref<string | null>(null)

  function login(user: User) {
    currentUser.value = user
    isAuthenticated.value = true
    loginError.value = null
  }

  function logout() {
    currentUser.value = null
    isAuthenticated.value = false
  }

  function setError(message: string) {
    loginError.value = message
  }

  return { 
    currentUser, 
    isAuthenticated, 
    loginError,
    login, 
    logout,
    setError
  }
})
```

### Advanced Form: Async Actions with Loading States

```typescript
// src/stores/products.ts
import { defineStore } from 'pinia'
import { ref } from 'vue'

interface Product {
  id: number
  name: string
  price: number
  category: string
}

export const useProductStore = defineStore('products', () => {
  const products = ref<Product[]>([])
  const isLoading = ref(false)
  const error = ref<string | null>(null)
  const selectedCategory = ref<string | null>(null)

  async function fetchProducts() {
    isLoading.value = true
    error.value = null
    
    try {
      const response = await fetch('https://api.example.com/products')
      if (!response.ok) throw new Error('Failed to fetch products')
      products.value = await response.json()
    } catch (e) {
      error.value = e instanceof Error ? e.message : 'Unknown error'
    } finally {
      isLoading.value = false
    }
  }

  async function addProduct(product: Omit<Product, 'id'>) {
    isLoading.value = true
    
    try {
      const response = await fetch('https://api.example.com/products', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(product)
      })
      const newProduct = await response.json()
      products.value.push(newProduct)
    } catch (e) {
      error.value = e instanceof Error ? e.message : 'Failed to add product'
    } finally {
      isLoading.value = false
    }
  }

  function setCategory(category: string | null) {
    selectedCategory.value = category
  }

  function clearError() {
    error.value = null
  }

  return {
    products,
    isLoading,
    error,
    selectedCategory,
    fetchProducts,
    addProduct,
    setCategory,
    clearError
  }
})
```

---

## 4. Micro-Implementation Drills

### Drill 1 (Basic): Create a Simple Todo Store

Create a store that manages a list of todo items with add and remove functionality.

```typescript
// src/stores/todo.ts
import { defineStore } from 'pinia'
import { ref } from 'vue'

interface Todo {
  id: number
  text: string
  completed: boolean
}

export const useTodoStore = defineStore('todo', () => {
  // TODO 1: Create a ref for todos array with type Todo[]
  // const todos = ???

  // TODO 2: Create a ref to track the next ID (start at 1)
  // const nextId = ???

  // TODO 3: Create an addTodo action that takes text: string
  // - Creates a new todo with id from nextId, the text, and completed: false
  // - Pushes it to todos array
  // - Increments nextId

  // TODO 4: Create a removeTodo action that takes id: number
  // - Filters out the todo with matching id

  // TODO 5: Return todos, addTodo, and removeTodo
  return {}
})
```

**Solution:**

```typescript
// src/stores/todo.ts
import { defineStore } from 'pinia'
import { ref } from 'vue'

interface Todo {
  id: number
  text: string
  completed: boolean
}

export const useTodoStore = defineStore('todo', () => {
  const todos = ref<Todo[]>([])
  const nextId = ref(1)

  function addTodo(text: string) {
    todos.value.push({
      id: nextId.value,
      text,
      completed: false
    })
    nextId.value++
  }

  function removeTodo(id: number) {
    todos.value = todos.value.filter(todo => todo.id !== id)
  }

  return { todos, addTodo, removeTodo }
})
```

---

### Drill 2 (Variation): Add Toggle Functionality

Extend the todo store with a toggle action.

```typescript
// Add to the previous store:

// TODO: Create a toggleTodo action that:
// - Takes an id: number
// - Finds the todo with that id
// - Flips its completed property

function toggleTodo(id: number) {
  // Your code here
}
```

**Solution:**

```typescript
function toggleTodo(id: number) {
  const todo = todos.value.find(t => t.id === id)
  if (todo) {
    todo.completed = !todo.completed
  }
}
```

---

### Drill 3 (Combination): Store + Component Integration

Create a component that uses the todo store.

```vue
<!-- src/components/TodoList.vue -->
<script setup lang="ts">
import { ref } from 'vue'
// TODO 1: Import useTodoStore

// TODO 2: Create a store instance

// TODO 3: Create a ref for the new todo input text
const newTodoText = ref('')

// TODO 4: Create a function to handle adding a todo
// - Check if newTodoText is not empty
// - Call the store's addTodo action
// - Clear newTodoText after adding
function handleAdd() {
  // Your code here
}
</script>

<template>
  <div class="p-4 max-w-md mx-auto">
    <h2 class="text-2xl font-bold mb-4">Todo List</h2>
    
    <!-- Input form -->
    <div class="flex gap-2 mb-4">
      <input
        v-model="newTodoText"
        @keyup.enter="handleAdd"
        type="text"
        placeholder="Add a new todo..."
        class="input input-bordered flex-1"
      />
      <button @click="handleAdd" class="btn btn-primary">Add</button>
    </div>
    
    <!-- Todo list -->
    <!-- TODO 5: Use v-for to iterate over store.todos -->
    <!-- TODO 6: Add click handler to toggle todo completion -->
    <!-- TODO 7: Add remove button that calls store.removeTodo -->
    <ul class="space-y-2">
      <!-- Your template here -->
    </ul>
  </div>
</template>
```

**Solution:**

```vue
<!-- src/components/TodoList.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import { useTodoStore } from '@/stores/todo'

const todoStore = useTodoStore()
const newTodoText = ref('')

function handleAdd() {
  if (newTodoText.value.trim()) {
    todoStore.addTodo(newTodoText.value.trim())
    newTodoText.value = ''
  }
}
</script>

<template>
  <div class="p-4 max-w-md mx-auto">
    <h2 class="text-2xl font-bold mb-4">Todo List</h2>
    
    <div class="flex gap-2 mb-4">
      <input
        v-model="newTodoText"
        @keyup.enter="handleAdd"
        type="text"
        placeholder="Add a new todo..."
        class="input input-bordered flex-1"
      />
      <button @click="handleAdd" class="btn btn-primary">Add</button>
    </div>
    
    <ul class="space-y-2">
      <li 
        v-for="todo in todoStore.todos" 
        :key="todo.id"
        class="flex items-center gap-2 p-2 bg-base-200 rounded"
      >
        <input
          type="checkbox"
          :checked="todo.completed"
          @change="todoStore.toggleTodo(todo.id)"
          class="checkbox"
        />
        <span :class="{ 'line-through opacity-50': todo.completed }" class="flex-1">
          {{ todo.text }}
        </span>
        <button 
          @click="todoStore.removeTodo(todo.id)"
          class="btn btn-sm btn-error"
        >
          Delete
        </button>
      </li>
    </ul>
    
    <p v-if="todoStore.todos.length === 0" class="text-gray-500 text-center py-4">
      No todos yet. Add one above!
    </p>
  </div>
</template>
```

---

### Drill 4 (Edge Case): Handling Async Actions

Create a store that fetches data and handles loading/error states.

```typescript
// src/stores/posts.ts
import { defineStore } from 'pinia'
import { ref } from 'vue'

interface Post {
  id: number
  title: string
  body: string
}

export const usePostStore = defineStore('posts', () => {
  const posts = ref<Post[]>([])
  const isLoading = ref(false)
  const error = ref<string | null>(null)

  // TODO: Implement fetchPosts as an async action
  // 1. Set isLoading to true and clear error
  // 2. Try to fetch from 'https://jsonplaceholder.typicode.com/posts?_limit=5'
  // 3. Parse JSON and assign to posts
  // 4. Catch any errors and set error message
  // 5. Always set isLoading to false in finally block

  async function fetchPosts() {
    // Your code here
  }

  return { posts, isLoading, error, fetchPosts }
})
```

**Solution:**

```typescript
// src/stores/posts.ts
import { defineStore } from 'pinia'
import { ref } from 'vue'

interface Post {
  id: number
  title: string
  body: string
}

export const usePostStore = defineStore('posts', () => {
  const posts = ref<Post[]>([])
  const isLoading = ref(false)
  const error = ref<string | null>(null)

  async function fetchPosts() {
    isLoading.value = true
    error.value = null
    
    try {
      const response = await fetch('https://jsonplaceholder.typicode.com/posts?_limit=5')
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`)
      }
      
      posts.value = await response.json()
    } catch (e) {
      error.value = e instanceof Error ? e.message : 'Failed to fetch posts'
    } finally {
      isLoading.value = false
    }
  }

  return { posts, isLoading, error, fetchPosts }
})
```

---

## 5. Common Pitfalls & Anti-Patterns

| ❌ Common Mistake | ✅ Correct Approach | Why It Matters |
|:--- |:--- |:--- |
| Destructuring reactive state: `const { count } = useCounterStore()` | Keep the store reference: `const store = useCounterStore()` and use `store.count`, OR use `storeToRefs()`: `const { count } = storeToRefs(store)` | Destructuring breaks reactivity — changes won't update your component |
| Modifying state directly outside actions: `store.items.value.push(...)` | Always use actions: `store.addItem(...)` | Direct mutation bypasses Pinia's tracking and devtools, making debugging harder |
| Creating multiple store instances in one component: `const store1 = useStore(); const store2 = useStore()` | Create once and reuse: `const store = useStore()` | Each `useStore()` call returns the same singleton, but multiple calls are confusing and wasteful |

### Destructuring Fix with `storeToRefs()`

```vue
<script setup lang="ts">
import { storeToRefs } from 'pinia'
import { useCounterStore } from '@/stores/counter'

const counterStore = useCounterStore()

// ❌ BAD: This breaks reactivity!
// const { count } = counterStore

// ✅ GOOD: Use storeToRefs for reactive destructuring
const { count } = storeToRefs(counterStore)

// Note: Actions don't need storeToRefs — they're just functions
const { increment } = counterStore
</script>

<template>
  <!-- count is now reactive! -->
  <p>{{ count }}</p>
  <button @click="increment">+1</button>
</template>
```

---

## 6. Mini-Project: Task Manager Store

Build a complete task management store with categories and priority levels.

**File Structure:**
```
src/
├── stores/
│   └── tasks.ts
├── components/
│   └── TaskManager.vue
├── App.vue
└── main.ts
```

**src/stores/tasks.ts:**
```typescript
import { defineStore } from 'pinia'
import { ref } from 'vue'

// Define the possible priority levels as a TypeScript union type
// This ensures type safety when assigning priorities
type Priority = 'low' | 'medium' | 'high'

// Define the shape of a single task
// All tasks will conform to this structure
interface Task {
  id: number           // Unique identifier for each task
  title: string        // The task's title/name
  description: string  // Detailed description of the task
  priority: Priority   // How urgent the task is
  completed: boolean   // Whether the task is done
  createdAt: Date      // When the task was created
}

// Define the shape of data needed to create a new task
// Omit excludes fields that will be auto-generated
type NewTask = Omit<Task, 'id' | 'completed' | 'createdAt'>

// Create and export the task store
// 'tasks' is the unique identifier for this store in Pinia devtools
export const useTaskStore = defineStore('tasks', () => {
  // ============== STATE ==============
  // All reactive data that components can read
  
  // Array to hold all tasks
  // ref<Task[]>([]) creates a reactive array typed as Task[]
  const tasks = ref<Task[]>([])
  
  // Counter to generate unique IDs for new tasks
  // Starts at 1 and increments with each new task
  const nextId = ref(1)
  
  // Optional filter for viewing tasks by priority
  // null means "show all priorities"
  const filterPriority = ref<Priority | null>(null)

  // ============== ACTIONS ==============
  // Functions that modify state
  
  // Add a new task to the list
  // Takes title, description, and priority; auto-generates id, completed, createdAt
  function addTask(newTask: NewTask) {
    // Create the full task object
    const task: Task = {
      id: nextId.value,              // Use current nextId
      title: newTask.title,
      description: newTask.description,
      priority: newTask.priority,
      completed: false,               // New tasks start uncompleted
      createdAt: new Date()          // Timestamp when created
    }
    
    // Add to the tasks array
    tasks.value.push(task)
    
    // Increment ID for next task
    nextId.value++
  }
  
  // Remove a task by its ID
  function removeTask(id: number) {
    // Find the index of the task with matching ID
    const index = tasks.value.findIndex(task => task.id === id)
    
    // If found (index !== -1), remove it using splice
    if (index > -1) {
      tasks.value.splice(index, 1)
    }
  }
  
  // Toggle a task's completed status
  function toggleTask(id: number) {
    // Find the task with matching ID
    const task = tasks.value.find(t => t.id === id)
    
    // If found, flip the completed boolean
    if (task) {
      task.completed = !task.completed
    }
  }
  
  // Update a task's priority level
  function updatePriority(id: number, priority: Priority) {
    const task = tasks.value.find(t => t.id === id)
    if (task) {
      task.priority = priority
    }
  }
  
  // Set the filter for displaying tasks
  // Pass null to show all tasks, or a priority to filter
  function setFilter(priority: Priority | null) {
    filterPriority.value = priority
  }
  
  // Clear all completed tasks from the list
  function clearCompleted() {
    // Filter keeps only tasks where completed is false
    tasks.value = tasks.value.filter(task => !task.completed)
  }

  // ============== RETURN ==============
  // Expose state and actions to components
  return {
    // State
    tasks,
    filterPriority,
    // Actions
    addTask,
    removeTask,
    toggleTask,
    updatePriority,
    setFilter,
    clearCompleted
  }
})
```

**src/components/TaskManager.vue:**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'
import { useTaskStore } from '@/stores/tasks'

// Get the store instance
// This gives us access to all state and actions defined in the store
const taskStore = useTaskStore()

// Local component state for the "new task" form
// These don't need to be in the store because they're only used here
const newTitle = ref('')
const newDescription = ref('')
const newPriority = ref<'low' | 'medium' | 'high'>('medium')

// Computed property to filter tasks based on selected priority
// This runs automatically when tasks or filterPriority changes
const filteredTasks = computed(() => {
  // If no filter is set, return all tasks
  if (!taskStore.filterPriority) {
    return taskStore.tasks
  }
  // Otherwise, return only tasks matching the selected priority
  return taskStore.tasks.filter(task => task.priority === taskStore.filterPriority)
})

// Computed property to count remaining (incomplete) tasks
const remainingCount = computed(() => {
  return taskStore.tasks.filter(task => !task.completed).length
})

// Handle form submission to add a new task
function handleAddTask() {
  // Validate: don't add empty tasks
  if (!newTitle.value.trim()) return
  
  // Call the store action with task data
  taskStore.addTask({
    title: newTitle.value.trim(),
    description: newDescription.value.trim(),
    priority: newPriority.value
  })
  
  // Reset form fields after adding
  newTitle.value = ''
  newDescription.value = ''
  newPriority.value = 'medium'
}

// Helper function to get CSS classes for priority badges
function getPriorityClass(priority: string): string {
  switch (priority) {
    case 'high': return 'badge-error'    // Red for high priority
    case 'medium': return 'badge-warning' // Yellow for medium
    case 'low': return 'badge-success'    // Green for low
    default: return 'badge-ghost'
  }
}
</script>

<template>
  <div class="p-6 max-w-2xl mx-auto">
    <h1 class="text-3xl font-bold mb-6">Task Manager</h1>
    
    <!-- New Task Form -->
    <div class="card bg-base-200 p-4 mb-6">
      <h2 class="text-xl font-semibold mb-4">Add New Task</h2>
      
      <div class="space-y-4">
        <!-- Title input -->
        <input
          v-model="newTitle"
          type="text"
          placeholder="Task title"
          class="input input-bordered w-full"
          @keyup.enter="handleAddTask"
        />
        
        <!-- Description textarea -->
        <textarea
          v-model="newDescription"
          placeholder="Task description (optional)"
          class="textarea textarea-bordered w-full"
          rows="2"
        ></textarea>
        
        <!-- Priority selector and submit button -->
        <div class="flex gap-4 items-center">
          <select v-model="newPriority" class="select select-bordered">
            <option value="low">Low Priority</option>
            <option value="medium">Medium Priority</option>
            <option value="high">High Priority</option>
          </select>
          
          <button @click="handleAddTask" class="btn btn-primary">
            Add Task
          </button>
        </div>
      </div>
    </div>
    
    <!-- Filter Controls -->
    <div class="flex gap-2 mb-4 flex-wrap">
      <button
        @click="taskStore.setFilter(null)"
        class="btn btn-sm"
        :class="{ 'btn-active': !taskStore.filterPriority }"
      >
        All
      </button>
      <button
        @click="taskStore.setFilter('high')"
        class="btn btn-sm"
        :class="{ 'btn-active': taskStore.filterPriority === 'high' }"
      >
        High
      </button>
      <button
        @click="taskStore.setFilter('medium')"
        class="btn btn-sm"
        :class="{ 'btn-active': taskStore.filterPriority === 'medium' }"
      >
        Medium
      </button>
      <button
        @click="taskStore.setFilter('low')"
        class="btn btn-sm"
        :class="{ 'btn-active': taskStore.filterPriority === 'low' }"
      >
        Low
      </button>
      
      <div class="flex-1"></div>
      
      <button
        @click="taskStore.clearCompleted"
        class="btn btn-sm btn-outline btn-error"
      >
        Clear Completed
      </button>
    </div>
    
    <!-- Task Stats -->
    <p class="text-sm text-gray-500 mb-4">
      {{ remainingCount }} task{{ remainingCount !== 1 ? 's' : '' }} remaining
    </p>
    
    <!-- Task List -->
    <ul class="space-y-3">
      <li
        v-for="task in filteredTasks"
        :key="task.id"
        class="card bg-base-100 shadow-sm p-4"
      >
        <div class="flex items-start gap-3">
          <!-- Completion checkbox -->
          <input
            type="checkbox"
            :checked="task.completed"
            @change="taskStore.toggleTask(task.id)"
            class="checkbox checkbox-primary mt-1"
          />
          
          <!-- Task content -->
          <div class="flex-1 min-w-0">
            <div class="flex items-center gap-2 mb-1">
              <h3
                class="font-semibold"
                :class="{ 'line-through opacity-50': task.completed }"
              >
                {{ task.title }}
              </h3>
              <span class="badge" :class="getPriorityClass(task.priority)">
                {{ task.priority }}
              </span>
            </div>
            
            <p
              v-if="task.description"
              class="text-sm text-gray-600"
              :class="{ 'line-through opacity-50': task.completed }"
            >
              {{ task.description }}
            </p>
            
            <p class="text-xs text-gray-400 mt-1">
              Created: {{ task.createdAt.toLocaleDateString() }}
            </p>
          </div>
          
          <!-- Priority selector -->
          <select
            :value="task.priority"
            @change="taskStore.updatePriority(task.id, ($event.target as HTMLSelectElement).value as 'low' | 'medium' | 'high')"
            class="select select-sm select-bordered"
          >
            <option value="low">Low</option>
            <option value="medium">Medium</option>
            <option value="high">High</option>
          </select>
          
          <!-- Delete button -->
          <button
            @click="taskStore.removeTask(task.id)"
            class="btn btn-sm btn-ghost text-error"
          >
            ✕
          </button>
        </div>
      </li>
    </ul>
    
    <!-- Empty state -->
    <div
      v-if="filteredTasks.length === 0"
      class="text-center py-8 text-gray-500"
    >
      <p v-if="taskStore.tasks.length === 0">
        No tasks yet. Add your first task above!
      </p>
      <p v-else>
        No tasks match the current filter.
      </p>
    </div>
  </div>
</template>
```

**How to Test:**

1. Create a new Vite project: `npm create vite@latest task-app -- --template vue-ts`
2. Install dependencies: `npm install pinia`
3. Install Tailwind + DaisyUI: Follow setup guides
4. Add Pinia to `main.ts`:
   ```typescript
   import { createPinia } from 'pinia'
   app.use(createPinia())
   ```
5. Copy the store and component files
6. Import `TaskManager` in `App.vue`
7. Run: `npm run dev`
8. Test: Add tasks, toggle completion, filter by priority, clear completed

---

## 7. Implementation Exercises (Progressive)

### Level 1 (Recall): Fill in the Blanks

```typescript
import { _____ } from 'pinia'
import { ref } from 'vue'

export const useCounterStore = _____('counter', () => {
  const count = _____(0)

  function increment() {
    count._____ ++
  }

  return { count, _____ }
})
```

**Answer:** `defineStore`, `defineStore`, `ref`, `value`, `increment`

---

### Level 2 (Apply): Small Independent Tasks

**Exercise 2.1:** Create a `useThemeStore` that manages light/dark mode:
- State: `theme` (type: `'light' | 'dark'`, default: `'light'`)
- Actions: `toggleTheme()`, `setTheme(newTheme: 'light' | 'dark')`

**Exercise 2.2:** Create a `useNotificationStore` that manages toast notifications:
- State: `notifications` (array of `{ id: number, message: string, type: 'success' | 'error' }`)
- Actions: `addNotification(message, type)`, `removeNotification(id)`
- The `addNotification` should auto-generate IDs

---

### Level 3 (Debug): Find and Fix the Bugs

```typescript
// This store has 3 bugs. Find and fix them.
import { defineStore } from 'pinia'
import { ref } from 'vue'

interface Item {
  id: number
  name: string
}

export const useItemStore = defineStore('items', () => {
  const items = ref<Item[]>([])
  let nextId = 1  // Bug 1: What's wrong here?

  function addItem(name: string) {
    items.push({  // Bug 2: What's wrong here?
      id: nextId,
      name
    })
    nextId++
  }

  function removeItem(id: number) {
    items = items.filter(item => item.id !== id)  // Bug 3: What's wrong here?
  }

  return { items, addItem, removeItem }
})
```

**Bugs and Fixes:**

1. **Bug 1:** `nextId` should be a `ref()` to be reactive and persistent
   - Fix: `const nextId = ref(1)`

2. **Bug 2:** Must use `.value` to access/modify ref contents
   - Fix: `items.value.push({ ... })`

3. **Bug 3:** Must use `.value` and can't reassign a ref's value like a variable
   - Fix: `items.value = items.value.filter(item => item.id !== id)`

**Corrected Code:**
```typescript
import { defineStore } from 'pinia'
import { ref } from 'vue'

interface Item {
  id: number
  name: string
}

export const useItemStore = defineStore('items', () => {
  const items = ref<Item[]>([])
  const nextId = ref(1)

  function addItem(name: string) {
    items.value.push({
      id: nextId.value,
      name
    })
    nextId.value++
  }

  function removeItem(id: number) {
    items.value = items.value.filter(item => item.id !== id)
  }

  return { items, addItem, removeItem }
})
```

---

**Reply 'next' for Lesson 21.1 (Theory).**
