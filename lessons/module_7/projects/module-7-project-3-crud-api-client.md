# Module 7 Project 3: CRUD API Client

## Project Overview

| Field | Value |
|:--- |:--- |
| **Module** | Module 7: Data Fetching & APIs |
| **Project** | CRUD API Client (Task Manager) |
| **Difficulty** | Advanced |
| **Estimated Time** | 75-90 minutes |

**What You'll Build:** A full-featured task manager that performs Create, Read, Update, and Delete operations against a REST API. The app includes task creation, status toggling, editing, deletion with confirmation, and optimistic updates for a smooth user experience.

**Real-World Scenario:** Most web applications need to interact with backend APIs to persist data. This project teaches you how to build a complete CRUD interface with proper error handling, loading states, and optimistic updates that make the UI feel responsive.

---

## Patterns You'll Practice

| Pattern | Description |
|:--- |:--- |
| Full CRUD operations | GET, POST, PUT/PATCH, DELETE HTTP methods |
| `useFetch` with methods | Configurable composable for different request types |
| Optimistic updates | Update UI immediately, rollback on error |
| Confirmation dialogs | Prevent accidental deletions |
| Form validation | Validate input before submission |
| Loading states per action | Show loading for specific operations |
| Error recovery | Graceful error handling with retry options |
| TypeScript strict mode | Full type safety across the application |

---

## Component Structure

```
src/
├── composables/
│   ├── useFetch.ts            # GET requests with auto-fetch
│   ├── useMutation.ts         # POST/PUT/DELETE requests
│   └── useTasks.ts            # Task-specific CRUD operations
├── components/
│   ├── TaskList.vue           # Main task list container
│   ├── TaskItem.vue           # Individual task with actions
│   ├── TaskForm.vue           # Create/Edit task form
│   ├── DeleteConfirm.vue      # Deletion confirmation modal
│   └── EmptyState.vue         # No tasks placeholder
├── types/
│   └── task.ts                # Task interfaces
├── App.vue
└── main.ts
```

---

## API Reference

We'll use [JSONPlaceholder](https://jsonplaceholder.typicode.com/) as our mock API:

| Operation | Method | Endpoint | Body |
|:--- |:--- |:--- |:--- |
| List tasks | GET | `/todos?_limit=10` | - |
| Get task | GET | `/todos/:id` | - |
| Create task | POST | `/todos` | `{ title, completed, userId }` |
| Update task | PUT | `/todos/:id` | `{ id, title, completed, userId }` |
| Toggle status | PATCH | `/todos/:id` | `{ completed }` |
| Delete task | DELETE | `/todos/:id` | - |

**Note:** JSONPlaceholder simulates responses but doesn't persist data. Your app should handle this gracefully.

---

## Step-by-Step Guide

### Step 1: Project Setup

```bash
npm create vite@latest task-manager -- --template vue-ts
cd task-manager
npm install
npm install -D tailwindcss postcss autoprefixer daisyui
npx tailwindcss init -p
```

Configure Tailwind and DaisyUI as in previous projects.

---

### Step 2: Define TypeScript Types

**src/types/task.ts:**
```typescript
// TODO: Define the Task interface with:
// - id: number
// - title: string
// - completed: boolean
// - userId: number

export interface Task {
  // YOUR CODE HERE
}

// TODO: Define CreateTaskDTO (Data Transfer Object) for creating tasks
// Should NOT include 'id' since the server assigns it

export interface CreateTaskDTO {
  // YOUR CODE HERE
}

// TODO: Define UpdateTaskDTO for updating tasks
// All fields optional except 'id'

export interface UpdateTaskDTO {
  // YOUR CODE HERE
}
```

---

### Step 3: Create the Base useFetch Composable

**src/composables/useFetch.ts:**
```typescript
import { ref, onMounted, onUnmounted, watch, isRef, unref, type Ref, type MaybeRef } from 'vue'

interface UseFetchOptions {
  immediate?: boolean
}

interface UseFetchReturn<T> {
  data: Ref<T | null>
  loading: Ref<boolean>
  error: Ref<string | null>
  refetch: () => Promise<void>
}

export function useFetch<T>(
  url: MaybeRef<string>,
  options: UseFetchOptions = {}
): UseFetchReturn<T> {
  const { immediate = true } = options

  const data = ref<T | null>(null) as Ref<T | null>
  const loading = ref(false)
  const error = ref<string | null>(null)

  let abortController: AbortController | null = null

  async function execute(): Promise<void> {
    const resolvedUrl = unref(url)
    if (!resolvedUrl) return

    if (abortController) {
      abortController.abort()
    }
    abortController = new AbortController()

    loading.value = true
    error.value = null

    try {
      const response = await fetch(resolvedUrl, {
        signal: abortController.signal
      })

      if (!response.ok) {
        throw new Error(`HTTP error! Status: ${response.status}`)
      }

      data.value = await response.json()
    } catch (e) {
      if (e instanceof Error && e.name !== 'AbortError') {
        error.value = e.message
      }
    } finally {
      loading.value = false
    }
  }

  if (immediate) {
    onMounted(execute)
  }

  if (isRef(url)) {
    watch(url, execute)
  }

  onUnmounted(() => {
    abortController?.abort()
  })

  return {
    data,
    loading,
    error,
    refetch: execute
  }
}
```

---

### Step 4: Create the useMutation Composable

This composable handles POST, PUT, PATCH, and DELETE requests.

**src/composables/useMutation.ts:**
```typescript
import { ref, type Ref } from 'vue'

type HttpMethod = 'POST' | 'PUT' | 'PATCH' | 'DELETE'

interface UseMutationOptions {
  method: HttpMethod
  headers?: Record<string, string>
}

interface UseMutationReturn<TData, TBody> {
  data: Ref<TData | null>
  loading: Ref<boolean>
  error: Ref<string | null>
  execute: (url: string, body?: TBody) => Promise<TData | null>
  reset: () => void
}

// TODO: Implement useMutation<TData, TBody = unknown>
// Requirements:
// 1. Accept options with method and optional headers
// 2. Create refs for data, loading, error
// 3. Implement execute(url, body?) that:
//    - Sets loading to true
//    - Makes fetch request with method and JSON body
//    - Parses response (handle empty responses for DELETE)
//    - Catches errors and sets error.value
//    - Returns the data or null on error
// 4. Implement reset() to clear all state
// 5. Return { data, loading, error, execute, reset }

export function useMutation<TData, TBody = unknown>(
  options: UseMutationOptions
): UseMutationReturn<TData, TBody> {
  // YOUR CODE HERE
}
```

---

### Step 5: Create the useTasks Composable

A domain-specific composable that combines fetching and mutations for tasks.

**src/composables/useTasks.ts:**
```typescript
import { computed } from 'vue'
import type { Task, CreateTaskDTO, UpdateTaskDTO } from '@/types/task'
import { useFetch } from './useFetch'
import { useMutation } from './useMutation'

const API_BASE = 'https://jsonplaceholder.typicode.com'

// TODO: Implement useTasks composable
// Requirements:
// 1. Use useFetch to get initial task list
// 2. Create mutations for: createTask, updateTask, toggleTask, deleteTask
// 3. Implement optimistic updates:
//    - Add task to list immediately on create
//    - Update task in list immediately on update
//    - Toggle completed status immediately
//    - Remove task from list immediately on delete
// 4. Rollback on error
// 5. Return:
//    - tasks: computed filtered/sorted task list
//    - loading: initial load state
//    - error: error message
//    - creating, updating, deleting: individual operation states
//    - createTask, updateTask, toggleTask, deleteTask: functions
//    - refetch: to reload task list

export function useTasks() {
  // YOUR CODE HERE
}
```

---

### Step 6: Create the Empty State Component

**src/components/EmptyState.vue:**
```vue
<script setup lang="ts">
defineProps<{
  message?: string
}>()
</script>

<template>
  <!-- TODO: Create an empty state display with:
       - Icon (clipboard or similar)
       - Message text (prop or default)
       - Suggestion to add first task -->

</template>
```

---

### Step 7: Create the Task Form Component

**src/components/TaskForm.vue:**
```vue
<script setup lang="ts">
import { ref, watch } from 'vue'
import type { Task } from '@/types/task'

const props = defineProps<{
  task?: Task | null
  loading?: boolean
}>()

const emit = defineEmits<{
  submit: [title: string]
  cancel: []
}>()

const title = ref('')
const error = ref<string | null>(null)

// TODO: Watch props.task and populate title for editing

// TODO: Implement validation (title required, min 3 chars)

// TODO: Implement handleSubmit that validates and emits

// TODO: Implement handleCancel that clears form and emits cancel

</script>

<template>
  <!-- TODO: Create a form with:
       - Text input for task title
       - Validation error display
       - Submit button (changes text based on create/edit)
       - Cancel button (only shown when editing)
       - Loading state on submit button -->

</template>
```

---

### Step 8: Create the Delete Confirmation Modal

**src/components/DeleteConfirm.vue:**
```vue
<script setup lang="ts">
import type { Task } from '@/types/task'

defineProps<{
  task: Task | null
  loading?: boolean
}>()

const emit = defineEmits<{
  confirm: []
  cancel: []
}>()
</script>

<template>
  <!-- TODO: Create a DaisyUI modal with:
       - Warning icon
       - Task title being deleted
       - Confirm and Cancel buttons
       - Loading state on confirm button -->

</template>
```

---

### Step 9: Create the Task Item Component

**src/components/TaskItem.vue:**
```vue
<script setup lang="ts">
import type { Task } from '@/types/task'

const props = defineProps<{
  task: Task
  updating?: boolean
}>()

const emit = defineEmits<{
  toggle: [task: Task]
  edit: [task: Task]
  delete: [task: Task]
}>()

// TODO: Implement handlers for toggle, edit, delete

</script>

<template>
  <!-- TODO: Create a task item with:
       - Checkbox for completed status
       - Task title (strikethrough if completed)
       - Edit button
       - Delete button
       - Loading indicator when updating -->

</template>
```

---

### Step 10: Create the Task List Component

**src/components/TaskList.vue:**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'
import { useTasks } from '@/composables/useTasks'
import type { Task } from '@/types/task'
import TaskItem from './TaskItem.vue'
import TaskForm from './TaskForm.vue'
import DeleteConfirm from './DeleteConfirm.vue'
import EmptyState from './EmptyState.vue'

const {
  tasks,
  loading,
  error,
  creating,
  updating,
  deleting,
  createTask,
  updateTask,
  toggleTask,
  deleteTask,
  refetch
} = useTasks()

// TODO: Create refs for:
// - editingTask: Task | null (task being edited)
// - deletingTask: Task | null (task pending deletion)
// - filter: 'all' | 'active' | 'completed'

// TODO: Create computed for filtered tasks based on filter

// TODO: Implement handlers:
// - handleCreate(title: string)
// - handleUpdate(title: string) 
// - handleToggle(task: Task)
// - handleDelete() - confirm deletion
// - startEdit(task: Task)
// - startDelete(task: Task)
// - cancelEdit()
// - cancelDelete()

</script>

<template>
  <!-- TODO: Create the task list UI with:
       - Header with title and task count
       - Filter tabs (All, Active, Completed)
       - TaskForm for creating new tasks
       - Loading state for initial load
       - Error state with retry button
       - EmptyState when no tasks match filter
       - List of TaskItem components
       - Edit modal with TaskForm (when editingTask is set)
       - DeleteConfirm modal (when deletingTask is set) -->

</template>
```

---

### Step 11: Assemble the App

**src/App.vue:**
```vue
<script setup lang="ts">
import TaskList from './components/TaskList.vue'
</script>

<template>
  <div class="min-h-screen bg-base-200 py-8">
    <div class="container mx-auto px-4 max-w-2xl">
      <TaskList />
    </div>
  </div>
</template>
```

---

## Complete Solution

### src/types/task.ts
```typescript
export interface Task {
  id: number
  title: string
  completed: boolean
  userId: number
}

export interface CreateTaskDTO {
  title: string
  completed: boolean
  userId: number
}

export interface UpdateTaskDTO {
  id: number
  title?: string
  completed?: boolean
}
```

### src/composables/useMutation.ts
```typescript
import { ref, type Ref } from 'vue'

type HttpMethod = 'POST' | 'PUT' | 'PATCH' | 'DELETE'

interface UseMutationOptions {
  method: HttpMethod
  headers?: Record<string, string>
}

interface UseMutationReturn<TData, TBody> {
  data: Ref<TData | null>
  loading: Ref<boolean>
  error: Ref<string | null>
  execute: (url: string, body?: TBody) => Promise<TData | null>
  reset: () => void
}

export function useMutation<TData, TBody = unknown>(
  options: UseMutationOptions
): UseMutationReturn<TData, TBody> {
  const { method, headers = {} } = options

  const data = ref<TData | null>(null) as Ref<TData | null>
  const loading = ref(false)
  const error = ref<string | null>(null)

  async function execute(url: string, body?: TBody): Promise<TData | null> {
    loading.value = true
    error.value = null

    try {
      const response = await fetch(url, {
        method,
        headers: {
          'Content-Type': 'application/json',
          ...headers
        },
        body: body ? JSON.stringify(body) : undefined
      })

      if (!response.ok) {
        throw new Error(`HTTP error! Status: ${response.status}`)
      }

      const contentType = response.headers.get('content-type')
      if (contentType?.includes('application/json')) {
        data.value = await response.json()
      } else {
        data.value = null
      }

      return data.value
    } catch (e) {
      error.value = e instanceof Error ? e.message : 'An unknown error occurred'
      return null
    } finally {
      loading.value = false
    }
  }

  function reset() {
    data.value = null
    loading.value = false
    error.value = null
  }

  return {
    data,
    loading,
    error,
    execute,
    reset
  }
}
```

### src/composables/useTasks.ts
```typescript
import { ref, computed } from 'vue'
import type { Task, CreateTaskDTO } from '@/types/task'
import { useFetch } from './useFetch'
import { useMutation } from './useMutation'

const API_BASE = 'https://jsonplaceholder.typicode.com'

export function useTasks() {
  const { 
    data: fetchedTasks, 
    loading, 
    error, 
    refetch 
  } = useFetch<Task[]>(`${API_BASE}/todos?_limit=10`)

  const localTasks = ref<Task[]>([])
  const nextId = ref(1000)

  const tasks = computed(() => {
    if (localTasks.value.length > 0) {
      return localTasks.value
    }
    return fetchedTasks.value || []
  })

  function initLocalTasks() {
    if (fetchedTasks.value && localTasks.value.length === 0) {
      localTasks.value = [...fetchedTasks.value]
    }
  }

  const createMutation = useMutation<Task, CreateTaskDTO>({ method: 'POST' })
  const updateMutation = useMutation<Task, Partial<Task>>({ method: 'PUT' })
  const patchMutation = useMutation<Task, Partial<Task>>({ method: 'PATCH' })
  const deleteMutation = useMutation<{}>({ method: 'DELETE' })

  async function createTask(title: string): Promise<boolean> {
    initLocalTasks()

    const optimisticTask: Task = {
      id: nextId.value++,
      title,
      completed: false,
      userId: 1
    }

    localTasks.value = [optimisticTask, ...localTasks.value]

    const result = await createMutation.execute(`${API_BASE}/todos`, {
      title,
      completed: false,
      userId: 1
    })

    if (!result) {
      localTasks.value = localTasks.value.filter(t => t.id !== optimisticTask.id)
      return false
    }

    return true
  }

  async function updateTask(id: number, title: string): Promise<boolean> {
    initLocalTasks()

    const taskIndex = localTasks.value.findIndex(t => t.id === id)
    if (taskIndex === -1) return false

    const originalTask = { ...localTasks.value[taskIndex] }
    localTasks.value[taskIndex] = { ...originalTask, title }

    const result = await updateMutation.execute(`${API_BASE}/todos/${id}`, {
      ...localTasks.value[taskIndex]
    })

    if (!result) {
      localTasks.value[taskIndex] = originalTask
      return false
    }

    return true
  }

  async function toggleTask(task: Task): Promise<boolean> {
    initLocalTasks()

    const taskIndex = localTasks.value.findIndex(t => t.id === task.id)
    if (taskIndex === -1) return false

    const originalCompleted = localTasks.value[taskIndex].completed
    localTasks.value[taskIndex] = { 
      ...localTasks.value[taskIndex], 
      completed: !originalCompleted 
    }

    const result = await patchMutation.execute(`${API_BASE}/todos/${task.id}`, {
      completed: !originalCompleted
    })

    if (!result) {
      localTasks.value[taskIndex] = { 
        ...localTasks.value[taskIndex], 
        completed: originalCompleted 
      }
      return false
    }

    return true
  }

  async function deleteTask(id: number): Promise<boolean> {
    initLocalTasks()

    const taskIndex = localTasks.value.findIndex(t => t.id === id)
    if (taskIndex === -1) return false

    const originalTask = localTasks.value[taskIndex]
    localTasks.value = localTasks.value.filter(t => t.id !== id)

    const result = await deleteMutation.execute(`${API_BASE}/todos/${id}`)

    if (deleteMutation.error.value) {
      localTasks.value.splice(taskIndex, 0, originalTask)
      return false
    }

    return true
  }

  return {
    tasks,
    loading,
    error,
    creating: createMutation.loading,
    updating: computed(() => updateMutation.loading.value || patchMutation.loading.value),
    deleting: deleteMutation.loading,
    createError: createMutation.error,
    updateError: updateMutation.error,
    deleteError: deleteMutation.error,
    createTask,
    updateTask,
    toggleTask,
    deleteTask,
    refetch
  }
}
```

### src/components/EmptyState.vue
```vue
<script setup lang="ts">
withDefaults(defineProps<{
  message?: string
}>(), {
  message: 'No tasks yet'
})
</script>

<template>
  <div class="text-center py-12">
    <svg xmlns="http://www.w3.org/2000/svg" class="h-16 w-16 mx-auto text-base-content/30" fill="none" viewBox="0 0 24 24" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-6 9l2 2 4-4" />
    </svg>
    <p class="mt-4 text-base-content/60">{{ message }}</p>
    <p class="text-sm text-base-content/40">Add your first task above to get started</p>
  </div>
</template>
```

### src/components/TaskForm.vue
```vue
<script setup lang="ts">
import { ref, watch } from 'vue'
import type { Task } from '@/types/task'

const props = withDefaults(defineProps<{
  task?: Task | null
  loading?: boolean
}>(), {
  task: null,
  loading: false
})

const emit = defineEmits<{
  submit: [title: string]
  cancel: []
}>()

const title = ref('')
const error = ref<string | null>(null)

watch(() => props.task, (newTask) => {
  if (newTask) {
    title.value = newTask.title
  } else {
    title.value = ''
  }
  error.value = null
}, { immediate: true })

function validate(): boolean {
  if (!title.value.trim()) {
    error.value = 'Task title is required'
    return false
  }
  if (title.value.trim().length < 3) {
    error.value = 'Task title must be at least 3 characters'
    return false
  }
  error.value = null
  return true
}

function handleSubmit() {
  if (!validate()) return
  emit('submit', title.value.trim())
  if (!props.task) {
    title.value = ''
  }
}

function handleCancel() {
  title.value = ''
  error.value = null
  emit('cancel')
}
</script>

<template>
  <form @submit.prevent="handleSubmit" class="space-y-2">
    <div class="join w-full">
      <input
        v-model="title"
        type="text"
        :placeholder="task ? 'Edit task...' : 'Add a new task...'"
        class="input input-bordered join-item flex-1"
        :class="{ 'input-error': error }"
        :disabled="loading"
      />
      <button 
        type="submit" 
        class="btn btn-primary join-item"
        :disabled="loading"
      >
        <span v-if="loading" class="loading loading-spinner loading-sm"></span>
        <span v-else>{{ task ? 'Update' : 'Add' }}</span>
      </button>
      <button 
        v-if="task"
        type="button"
        class="btn btn-ghost join-item"
        @click="handleCancel"
        :disabled="loading"
      >
        Cancel
      </button>
    </div>
    <p v-if="error" class="text-error text-sm">{{ error }}</p>
  </form>
</template>
```

### src/components/DeleteConfirm.vue
```vue
<script setup lang="ts">
import type { Task } from '@/types/task'

defineProps<{
  task: Task | null
  loading?: boolean
}>()

const emit = defineEmits<{
  confirm: []
  cancel: []
}>()
</script>

<template>
  <dialog class="modal" :class="{ 'modal-open': task }">
    <div class="modal-box">
      <h3 class="font-bold text-lg flex items-center gap-2">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-warning" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
        </svg>
        Delete Task
      </h3>
      <p class="py-4">
        Are you sure you want to delete "<strong>{{ task?.title }}</strong>"? This action cannot be undone.
      </p>
      <div class="modal-action">
        <button 
          class="btn btn-ghost" 
          @click="emit('cancel')"
          :disabled="loading"
        >
          Cancel
        </button>
        <button 
          class="btn btn-error" 
          @click="emit('confirm')"
          :disabled="loading"
        >
          <span v-if="loading" class="loading loading-spinner loading-sm"></span>
          <span v-else>Delete</span>
        </button>
      </div>
    </div>
    <form method="dialog" class="modal-backdrop">
      <button @click="emit('cancel')">close</button>
    </form>
  </dialog>
</template>
```

### src/components/TaskItem.vue
```vue
<script setup lang="ts">
import type { Task } from '@/types/task'

const props = withDefaults(defineProps<{
  task: Task
  updating?: boolean
}>(), {
  updating: false
})

const emit = defineEmits<{
  toggle: [task: Task]
  edit: [task: Task]
  delete: [task: Task]
}>()
</script>

<template>
  <div 
    class="flex items-center gap-3 p-3 bg-base-100 rounded-lg shadow-sm hover:shadow transition-shadow"
    :class="{ 'opacity-60': task.completed }"
  >
    <input
      type="checkbox"
      :checked="task.completed"
      class="checkbox checkbox-primary"
      :disabled="updating"
      @change="emit('toggle', task)"
    />
    
    <span 
      class="flex-1"
      :class="{ 'line-through text-base-content/50': task.completed }"
    >
      {{ task.title }}
    </span>

    <div v-if="updating" class="loading loading-spinner loading-sm"></div>
    
    <template v-else>
      <button
        class="btn btn-ghost btn-sm btn-square"
        @click="emit('edit', task)"
        title="Edit task"
      >
        <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
        </svg>
      </button>
      
      <button
        class="btn btn-ghost btn-sm btn-square text-error"
        @click="emit('delete', task)"
        title="Delete task"
      >
        <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
        </svg>
      </button>
    </template>
  </div>
</template>
```

### src/components/TaskList.vue
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'
import { useTasks } from '@/composables/useTasks'
import type { Task } from '@/types/task'
import TaskItem from './TaskItem.vue'
import TaskForm from './TaskForm.vue'
import DeleteConfirm from './DeleteConfirm.vue'
import EmptyState from './EmptyState.vue'

const {
  tasks,
  loading,
  error,
  creating,
  updating,
  deleting,
  createTask,
  updateTask,
  toggleTask,
  deleteTask,
  refetch
} = useTasks()

type FilterType = 'all' | 'active' | 'completed'

const editingTask = ref<Task | null>(null)
const deletingTask = ref<Task | null>(null)
const filter = ref<FilterType>('all')
const updatingId = ref<number | null>(null)

const filteredTasks = computed(() => {
  const taskList = tasks.value
  switch (filter.value) {
    case 'active':
      return taskList.filter(t => !t.completed)
    case 'completed':
      return taskList.filter(t => t.completed)
    default:
      return taskList
  }
})

const taskCounts = computed(() => ({
  all: tasks.value.length,
  active: tasks.value.filter(t => !t.completed).length,
  completed: tasks.value.filter(t => t.completed).length
}))

async function handleCreate(title: string) {
  await createTask(title)
}

async function handleUpdate(title: string) {
  if (!editingTask.value) return
  updatingId.value = editingTask.value.id
  await updateTask(editingTask.value.id, title)
  updatingId.value = null
  editingTask.value = null
}

async function handleToggle(task: Task) {
  updatingId.value = task.id
  await toggleTask(task)
  updatingId.value = null
}

async function handleDelete() {
  if (!deletingTask.value) return
  await deleteTask(deletingTask.value.id)
  deletingTask.value = null
}

function startEdit(task: Task) {
  editingTask.value = task
}

function startDelete(task: Task) {
  deletingTask.value = task
}

function cancelEdit() {
  editingTask.value = null
}

function cancelDelete() {
  deletingTask.value = null
}
</script>

<template>
  <div class="card bg-base-100 shadow-xl">
    <div class="card-body">
      <div class="flex justify-between items-center mb-4">
        <h2 class="card-title">
          📋 Task Manager
          <span class="badge badge-primary">{{ taskCounts.all }}</span>
        </h2>
        <button 
          class="btn btn-ghost btn-sm btn-circle"
          @click="refetch"
          :disabled="loading"
          title="Refresh"
        >
          <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" :class="{ 'animate-spin': loading }" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
          </svg>
        </button>
      </div>

      <div class="tabs tabs-boxed mb-4">
        <button 
          class="tab" 
          :class="{ 'tab-active': filter === 'all' }"
          @click="filter = 'all'"
        >
          All ({{ taskCounts.all }})
        </button>
        <button 
          class="tab" 
          :class="{ 'tab-active': filter === 'active' }"
          @click="filter = 'active'"
        >
          Active ({{ taskCounts.active }})
        </button>
        <button 
          class="tab" 
          :class="{ 'tab-active': filter === 'completed' }"
          @click="filter = 'completed'"
        >
          Completed ({{ taskCounts.completed }})
        </button>
      </div>

      <TaskForm 
        :loading="creating" 
        @submit="handleCreate" 
      />

      <div class="divider"></div>

      <div v-if="loading" class="flex justify-center py-8">
        <span class="loading loading-spinner loading-lg"></span>
      </div>

      <div v-else-if="error" class="alert alert-error">
        <span>{{ error }}</span>
        <button class="btn btn-sm" @click="refetch">Retry</button>
      </div>

      <EmptyState 
        v-else-if="filteredTasks.length === 0"
        :message="filter === 'all' ? 'No tasks yet' : `No ${filter} tasks`"
      />

      <div v-else class="space-y-2">
        <TaskItem
          v-for="task in filteredTasks"
          :key="task.id"
          :task="task"
          :updating="updatingId === task.id"
          @toggle="handleToggle"
          @edit="startEdit"
          @delete="startDelete"
        />
      </div>
    </div>
  </div>

  <dialog class="modal" :class="{ 'modal-open': editingTask }">
    <div class="modal-box">
      <h3 class="font-bold text-lg mb-4">Edit Task</h3>
      <TaskForm 
        :task="editingTask"
        :loading="updating"
        @submit="handleUpdate"
        @cancel="cancelEdit"
      />
    </div>
    <form method="dialog" class="modal-backdrop">
      <button @click="cancelEdit">close</button>
    </form>
  </dialog>

  <DeleteConfirm
    :task="deletingTask"
    :loading="deleting"
    @confirm="handleDelete"
    @cancel="cancelDelete"
  />
</template>
```

### src/App.vue
```vue
<script setup lang="ts">
import TaskList from './components/TaskList.vue'
</script>

<template>
  <div class="min-h-screen bg-gradient-to-br from-slate-900 via-slate-800 to-slate-900 py-8">
    <div class="container mx-auto px-4 max-w-2xl">
      <header class="text-center mb-8">
        <h1 class="text-3xl font-bold text-white">Task Manager</h1>
        <p class="text-white/60">Manage your tasks with full CRUD operations</p>
      </header>
      <TaskList />
    </div>
  </div>
</template>
```

---

## Stretch Goals

1. **Drag and Drop:** Reorder tasks using drag and drop
2. **Due Dates:** Add due date field with date picker and overdue highlighting
3. **Categories/Tags:** Add categories or tags to tasks with filtering
4. **Bulk Actions:** Select multiple tasks for bulk complete/delete
5. **Undo Delete:** Show toast with "Undo" option after deletion
6. **Offline Support:** Cache tasks in localStorage and sync when online
7. **Search:** Add search functionality to filter tasks by title

---

## Self-Assessment Checklist

- [ ] All CRUD operations work correctly (Create, Read, Update, Delete)
- [ ] Optimistic updates provide immediate UI feedback
- [ ] Errors trigger rollback of optimistic updates
- [ ] Loading states show for each operation individually
- [ ] Delete confirmation prevents accidental deletions
- [ ] Form validation prevents empty or short task titles
- [ ] Filter tabs correctly filter the task list
- [ ] Task counts update in real-time
- [ ] The useMutation composable is reusable for any API
- [ ] TypeScript types are properly defined and used throughout

---

## Testing Your Implementation

1. **Create Task:** Add a new task — should appear immediately at top of list
2. **Toggle Task:** Click checkbox — should toggle completed state immediately
3. **Edit Task:** Click edit, change title — should update immediately
4. **Delete Task:** Click delete, confirm — should remove immediately
5. **Filter Tasks:** Test all three filter tabs
6. **Error Handling:** Disconnect network and try operations — should show errors
7. **Validation:** Try to create empty task or task with 1-2 characters
8. **Refresh:** Click refresh button — should reload tasks from API
