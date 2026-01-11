# Module 4 Project 1: Dynamic Task List

## Project Overview

| Field | Value |
|:--- |:--- |
| **Module** | Module 4: Rendering Patterns |
| **Project** | Dynamic Task List |
| **Difficulty** | Beginner |
| **Estimated Time** | 45-60 minutes |
| **Prerequisites** | Lessons 9.1, 9.2, 10.1, 10.2 |

**What You'll Build:** A to-do application that renders tasks dynamically with status badges, priority indicators, and an empty state message when no tasks exist.

**Real-World Scenario:** You're building the task management section of a productivity app like Todoist or Microsoft To-Do. Users need to see their tasks at a glance, quickly identify task priority and completion status, and receive visual feedback when their task list is empty.

---

## Patterns You'll Practice

| Pattern | Where Used |
|:--- |:--- |
| `v-for` with `:key` | Rendering the task list |
| `v-if` / `v-else` | Empty state vs. task list |
| Conditional classes (`:class`) | Status badges, priority colors, completed styling |
| `computed()` | Filtering tasks, counting stats |
| `ref()` with arrays | Managing the task list |
| Event handling | Adding, removing, toggling tasks |

---

## Component Structure

```
src/
├── components/
│   ├── TaskList.vue        # Main container component
│   ├── TaskItem.vue        # Individual task row
│   ├── TaskInput.vue       # New task input form
│   └── TaskStats.vue       # Task statistics bar
├── types/
│   └── task.ts             # TypeScript interfaces
├── App.vue
└── main.ts
```

---

## Step-by-Step Guide

### Step 1: Project Setup

Create a new Vue project with TypeScript and install dependencies:

```bash
npm create vue@latest dynamic-task-list
# Select: TypeScript, NO JSX, NO Router, NO Pinia, NO Vitest, YES ESLint, YES Prettier
cd dynamic-task-list
npm install -D tailwindcss postcss autoprefixer daisyui@latest
npx tailwindcss init -p
```

Configure `tailwind.config.js`:

```js
/** @type {import('tailwindcss').Config} */
export default {
  content: ['./index.html', './src/**/*.{vue,js,ts,jsx,tsx}'],
  theme: {
    extend: {},
  },
  plugins: [require('daisyui')],
  daisyui: {
    themes: ['light', 'dark', 'cupcake'],
  },
}
```

Update `src/style.css`:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

---

### Step 2: Define TypeScript Types

Create `src/types/task.ts`:

```typescript
export type Priority = 'low' | 'medium' | 'high'

export interface Task {
  id: number
  title: string
  completed: boolean
  priority: Priority
  createdAt: Date
}

export type FilterType = 'all' | 'active' | 'completed'
```

---

### Step 3: Build TaskItem Component

Create `src/components/TaskItem.vue`:

```vue
<script setup lang="ts">
import type { Task } from '@/types/task'

defineProps<{
  task: Task
}>()

const emit = defineEmits<{
  toggle: [id: number]
  remove: [id: number]
}>()

const priorityClasses: Record<string, string> = {
  low: 'badge-info',
  medium: 'badge-warning',
  high: 'badge-error'
}
</script>

<template>
  <!-- 
    TODO: Add conditional class for completed tasks
    When task.completed is true, add 'opacity-60' class
  -->
  <div 
    class="flex items-center gap-3 p-4 bg-base-200 rounded-lg transition-all hover:bg-base-300"
  >
    <!-- Checkbox to toggle completion -->
    <input 
      type="checkbox"
      class="checkbox checkbox-primary"
      :checked="task.completed"
      @change="emit('toggle', task.id)"
    />
    
    <!-- Task title -->
    <!-- TODO: Add conditional 'line-through' class when completed -->
    <span class="flex-1 text-lg">
      {{ task.title }}
    </span>
    
    <!-- Priority badge -->
    <!-- TODO: Use :class with priorityClasses to set badge color based on priority -->
    <span class="badge">
      {{ task.priority }}
    </span>
    
    <!-- Delete button -->
    <button 
      class="btn btn-ghost btn-sm btn-circle"
      @click="emit('remove', task.id)"
    >
      ✕
    </button>
  </div>
</template>
```

**Your Task:** Complete the TODOs:
1. Add `opacity-60` class to the container div when `task.completed` is true
2. Add `line-through` class to the title span when completed
3. Apply dynamic badge classes based on priority

---

### Step 4: Build TaskInput Component

Create `src/components/TaskInput.vue`:

```vue
<script setup lang="ts">
import { ref } from 'vue'
import type { Priority } from '@/types/task'

const emit = defineEmits<{
  add: [title: string, priority: Priority]
}>()

const newTaskTitle = ref<string>('')
const newTaskPriority = ref<Priority>('medium')

const handleSubmit = () => {
  const title = newTaskTitle.value.trim()
  if (!title) return
  
  emit('add', title, newTaskPriority.value)
  newTaskTitle.value = ''
}
</script>

<template>
  <form @submit.prevent="handleSubmit" class="flex gap-2">
    <input
      v-model="newTaskTitle"
      type="text"
      placeholder="What needs to be done?"
      class="input input-bordered flex-1"
    />
    
    <select v-model="newTaskPriority" class="select select-bordered">
      <option value="low">Low</option>
      <option value="medium">Medium</option>
      <option value="high">High</option>
    </select>
    
    <button type="submit" class="btn btn-primary">
      Add Task
    </button>
  </form>
</template>
```

---

### Step 5: Build TaskStats Component

Create `src/components/TaskStats.vue`:

```vue
<script setup lang="ts">
import { computed } from 'vue'
import type { Task, FilterType } from '@/types/task'

const props = defineProps<{
  tasks: Task[]
  currentFilter: FilterType
}>()

const emit = defineEmits<{
  'update:filter': [filter: FilterType]
}>()

const totalCount = computed(() => props.tasks.length)
const completedCount = computed(() => props.tasks.filter(t => t.completed).length)
const activeCount = computed(() => totalCount.value - completedCount.value)

const filters: { value: FilterType; label: string }[] = [
  { value: 'all', label: 'All' },
  { value: 'active', label: 'Active' },
  { value: 'completed', label: 'Completed' },
]
</script>

<template>
  <div class="flex flex-wrap items-center justify-between gap-4 p-4 bg-base-200 rounded-lg">
    <!-- Stats -->
    <div class="flex gap-4 text-sm">
      <span>Total: <strong>{{ totalCount }}</strong></span>
      <span class="text-success">Active: <strong>{{ activeCount }}</strong></span>
      <span class="text-info">Done: <strong>{{ completedCount }}</strong></span>
    </div>
    
    <!-- Filter buttons -->
    <div class="btn-group">
      <!-- 
        TODO: Use v-for to render filter buttons
        Add 'btn-active' class when filter.value === currentFilter
      -->
      <button 
        v-for="filter in filters"
        :key="filter.value"
        class="btn btn-sm"
        @click="emit('update:filter', filter.value)"
      >
        {{ filter.label }}
      </button>
    </div>
  </div>
</template>
```

**Your Task:** Add the `btn-active` class conditionally using `:class`.

---

### Step 6: Build TaskList Main Component

Create `src/components/TaskList.vue`:

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'
import type { Task, Priority, FilterType } from '@/types/task'
import TaskItem from './TaskItem.vue'
import TaskInput from './TaskInput.vue'
import TaskStats from './TaskStats.vue'

const tasks = ref<Task[]>([
  { id: 1, title: 'Learn Vue 3 basics', completed: true, priority: 'high', createdAt: new Date() },
  { id: 2, title: 'Build a task list app', completed: false, priority: 'high', createdAt: new Date() },
  { id: 3, title: 'Master TypeScript', completed: false, priority: 'medium', createdAt: new Date() },
  { id: 4, title: 'Study Pinia state management', completed: false, priority: 'low', createdAt: new Date() },
])

let nextId = 5

const currentFilter = ref<FilterType>('all')

const filteredTasks = computed(() => {
  switch (currentFilter.value) {
    case 'active':
      return tasks.value.filter(t => !t.completed)
    case 'completed':
      return tasks.value.filter(t => t.completed)
    default:
      return tasks.value
  }
})

const addTask = (title: string, priority: Priority) => {
  tasks.value.push({
    id: nextId++,
    title,
    completed: false,
    priority,
    createdAt: new Date()
  })
}

const toggleTask = (id: number) => {
  const task = tasks.value.find(t => t.id === id)
  if (task) {
    task.completed = !task.completed
  }
}

const removeTask = (id: number) => {
  const index = tasks.value.findIndex(t => t.id === id)
  if (index !== -1) {
    tasks.value.splice(index, 1)
  }
}

const clearCompleted = () => {
  tasks.value = tasks.value.filter(t => !t.completed)
}
</script>

<template>
  <div class="max-w-2xl mx-auto p-6 space-y-6">
    <!-- Header -->
    <div class="text-center">
      <h1 class="text-4xl font-bold mb-2">📝 Task List</h1>
      <p class="text-base-content/70">Stay organized and get things done</p>
    </div>
    
    <!-- Task Input -->
    <TaskInput @add="addTask" />
    
    <!-- Stats & Filters -->
    <TaskStats 
      :tasks="tasks" 
      :current-filter="currentFilter"
      @update:filter="currentFilter = $event"
    />
    
    <!-- Task List -->
    <!-- 
      TODO: Implement conditional rendering
      - If filteredTasks has items: render the list with v-for
      - If filteredTasks is empty: show empty state message
    -->
    <div class="space-y-3">
      <!-- Use v-if to check if there are tasks -->
      <!-- Use v-for to render TaskItem components -->
      <!-- Remember :key="task.id" -->
      
      <TaskItem
        v-for="task in filteredTasks"
        :key="task.id"
        :task="task"
        @toggle="toggleTask"
        @remove="removeTask"
      />
    </div>
    
    <!-- Empty State -->
    <!-- TODO: Add v-if/v-else to show this when filteredTasks is empty -->
    <div class="text-center py-12 bg-base-200 rounded-lg">
      <div class="text-6xl mb-4">🎉</div>
      <h3 class="text-xl font-semibold mb-2">No tasks here!</h3>
      <p class="text-base-content/70">
        <!-- Show different messages based on filter -->
        Add a new task to get started.
      </p>
    </div>
    
    <!-- Clear Completed Button -->
    <!-- TODO: Only show this button if there are completed tasks -->
    <div class="text-center">
      <button 
        class="btn btn-outline btn-error btn-sm"
        @click="clearCompleted"
      >
        Clear Completed Tasks
      </button>
    </div>
  </div>
</template>
```

**Your Tasks:**
1. Add `v-if` to the task list div to only show when `filteredTasks.length > 0`
2. Add `v-else` to the empty state div
3. Update empty state message based on `currentFilter`
4. Only show "Clear Completed" button when there are completed tasks

---

### Step 7: Update App.vue

```vue
<script setup lang="ts">
import TaskList from './components/TaskList.vue'
</script>

<template>
  <div class="min-h-screen bg-base-300 py-8">
    <TaskList />
  </div>
</template>
```

---

## Complete Solution

### TaskItem.vue (Complete)

```vue
<script setup lang="ts">
import type { Task } from '@/types/task'

defineProps<{
  task: Task
}>()

const emit = defineEmits<{
  toggle: [id: number]
  remove: [id: number]
}>()

const priorityClasses: Record<string, string> = {
  low: 'badge-info',
  medium: 'badge-warning',
  high: 'badge-error'
}
</script>

<template>
  <div 
    class="flex items-center gap-3 p-4 bg-base-200 rounded-lg transition-all hover:bg-base-300"
    :class="{ 'opacity-60': task.completed }"
  >
    <input 
      type="checkbox"
      class="checkbox checkbox-primary"
      :checked="task.completed"
      @change="emit('toggle', task.id)"
    />
    
    <span 
      class="flex-1 text-lg"
      :class="{ 'line-through': task.completed }"
    >
      {{ task.title }}
    </span>
    
    <span 
      class="badge"
      :class="priorityClasses[task.priority]"
    >
      {{ task.priority }}
    </span>
    
    <button 
      class="btn btn-ghost btn-sm btn-circle"
      @click="emit('remove', task.id)"
    >
      ✕
    </button>
  </div>
</template>
```

### TaskStats.vue (Complete)

```vue
<script setup lang="ts">
import { computed } from 'vue'
import type { Task, FilterType } from '@/types/task'

const props = defineProps<{
  tasks: Task[]
  currentFilter: FilterType
}>()

const emit = defineEmits<{
  'update:filter': [filter: FilterType]
}>()

const totalCount = computed(() => props.tasks.length)
const completedCount = computed(() => props.tasks.filter(t => t.completed).length)
const activeCount = computed(() => totalCount.value - completedCount.value)

const filters: { value: FilterType; label: string }[] = [
  { value: 'all', label: 'All' },
  { value: 'active', label: 'Active' },
  { value: 'completed', label: 'Completed' },
]
</script>

<template>
  <div class="flex flex-wrap items-center justify-between gap-4 p-4 bg-base-200 rounded-lg">
    <div class="flex gap-4 text-sm">
      <span>Total: <strong>{{ totalCount }}</strong></span>
      <span class="text-success">Active: <strong>{{ activeCount }}</strong></span>
      <span class="text-info">Done: <strong>{{ completedCount }}</strong></span>
    </div>
    
    <div class="btn-group">
      <button 
        v-for="filter in filters"
        :key="filter.value"
        class="btn btn-sm"
        :class="{ 'btn-active': filter.value === currentFilter }"
        @click="emit('update:filter', filter.value)"
      >
        {{ filter.label }}
      </button>
    </div>
  </div>
</template>
```

### TaskList.vue (Complete)

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'
import type { Task, Priority, FilterType } from '@/types/task'
import TaskItem from './TaskItem.vue'
import TaskInput from './TaskInput.vue'
import TaskStats from './TaskStats.vue'

const tasks = ref<Task[]>([
  { id: 1, title: 'Learn Vue 3 basics', completed: true, priority: 'high', createdAt: new Date() },
  { id: 2, title: 'Build a task list app', completed: false, priority: 'high', createdAt: new Date() },
  { id: 3, title: 'Master TypeScript', completed: false, priority: 'medium', createdAt: new Date() },
  { id: 4, title: 'Study Pinia state management', completed: false, priority: 'low', createdAt: new Date() },
])

let nextId = 5

const currentFilter = ref<FilterType>('all')

const filteredTasks = computed(() => {
  switch (currentFilter.value) {
    case 'active':
      return tasks.value.filter(t => !t.completed)
    case 'completed':
      return tasks.value.filter(t => t.completed)
    default:
      return tasks.value
  }
})

const hasCompletedTasks = computed(() => tasks.value.some(t => t.completed))

const emptyStateMessage = computed(() => {
  switch (currentFilter.value) {
    case 'active':
      return 'All tasks are completed! Great job! 🎉'
    case 'completed':
      return 'No completed tasks yet. Keep working!'
    default:
      return 'Add a new task to get started.'
  }
})

const addTask = (title: string, priority: Priority) => {
  tasks.value.push({
    id: nextId++,
    title,
    completed: false,
    priority,
    createdAt: new Date()
  })
}

const toggleTask = (id: number) => {
  const task = tasks.value.find(t => t.id === id)
  if (task) {
    task.completed = !task.completed
  }
}

const removeTask = (id: number) => {
  const index = tasks.value.findIndex(t => t.id === id)
  if (index !== -1) {
    tasks.value.splice(index, 1)
  }
}

const clearCompleted = () => {
  tasks.value = tasks.value.filter(t => !t.completed)
}
</script>

<template>
  <div class="max-w-2xl mx-auto p-6 space-y-6">
    <div class="text-center">
      <h1 class="text-4xl font-bold mb-2">📝 Task List</h1>
      <p class="text-base-content/70">Stay organized and get things done</p>
    </div>
    
    <TaskInput @add="addTask" />
    
    <TaskStats 
      :tasks="tasks" 
      :current-filter="currentFilter"
      @update:filter="currentFilter = $event"
    />
    
    <div v-if="filteredTasks.length > 0" class="space-y-3">
      <TaskItem
        v-for="task in filteredTasks"
        :key="task.id"
        :task="task"
        @toggle="toggleTask"
        @remove="removeTask"
      />
    </div>
    
    <div v-else class="text-center py-12 bg-base-200 rounded-lg">
      <div class="text-6xl mb-4">🎉</div>
      <h3 class="text-xl font-semibold mb-2">No tasks here!</h3>
      <p class="text-base-content/70">
        {{ emptyStateMessage }}
      </p>
    </div>
    
    <div v-if="hasCompletedTasks" class="text-center">
      <button 
        class="btn btn-outline btn-error btn-sm"
        @click="clearCompleted"
      >
        Clear Completed Tasks
      </button>
    </div>
  </div>
</template>
```

---

## Stretch Goals

1. **Persist to LocalStorage:** Save tasks to localStorage and load on page refresh
2. **Drag & Drop Reordering:** Add drag-and-drop functionality to reorder tasks
3. **Due Dates:** Add due date field with overdue highlighting
4. **Categories/Tags:** Allow tasks to be tagged and filtered by category
5. **Animations:** Add transition animations when tasks are added/removed

---

## Self-Assessment Checklist

- [ ] All components use `<script setup>` with TypeScript
- [ ] `v-for` is used with unique `:key` attributes (task.id, not index)
- [ ] `v-if` / `v-else` properly shows task list OR empty state
- [ ] Conditional classes work correctly for completed tasks and priority badges
- [ ] Filter buttons have active state styling
- [ ] "Clear Completed" only shows when there are completed tasks
- [ ] Adding tasks works with title and priority
- [ ] Toggling task completion updates the UI
- [ ] Removing tasks works correctly
- [ ] Filter switches between all/active/completed views
- [ ] Empty state shows appropriate message per filter
