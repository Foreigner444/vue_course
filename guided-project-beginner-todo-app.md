# Guided Project: Simple To-Do List Application

**Module:** Beginner (Lessons 1-13)
**Estimated Time:** 2-3 hours
**Difficulty:** Medium

### What You'll Build
You will build a complete, interactive To-Do list application from scratch. It will allow you to add new tasks, mark tasks as complete, filter between all, active, and completed tasks, and delete tasks. This project will combine everything you've learned about components, state, props, events, and styling.

![Todo App Preview](https://i.imgur.com/gY8vj3G.png)

### Patterns You'll Practice
*   Component creation (`<script setup>`)
*   Reactive state with `ref()`
*   Derived state with `computed()`
*   List rendering with `v-for`
*   Conditional rendering with `v-if`
*   Event handling with `@click`
*   Form input binding with `v-model`
*   Passing data down with `props`
*   Sending events up with `defineEmits`
*   Styling with Tailwind CSS & DaisyUI

---

## Step-by-Step Guide

### Step 1: Project Setup (10 min)
First, set up a new Vue project and add Tailwind CSS and DaisyUI.

```bash
# 1. Create a new Vue project (choose TypeScript and other options as you like)
npm create vue@latest todo-app
cd todo-app

# 2. Install dev dependencies for Tailwind CSS
npm install -D tailwindcss postcss autoprefixer

# 3. Initialize Tailwind CSS
npx tailwindcss init -p

# 4. Configure Tailwind to scan your Vue files
# Open tailwind.config.js and replace its content with this:
# module.exports = {
#   content: [
#     "./index.html",
#     "./src/**/*.{vue,js,ts,jsx,tsx}",
#   ],
#   theme: {
#     extend: {},
#   },
#   plugins: [],
# }

# 5. Add Tailwind directives to your main CSS file
# Open src/assets/main.css and replace its content with this:
# @tailwind base;
# @tailwind components;
# @tailwind utilities;

# 6. Install and configure DaisyUI
npm i -D daisyui@latest

# 7. Add DaisyUI to your Tailwind config
# Open tailwind.config.js again and add the plugin:
# module.exports = {
#   // ... other config
#   plugins: [require("daisyui")],
# }

# 8. Run the development server
npm run dev
```
You should now have a running Vue application with styling capabilities.

### Step 2: Define the `Todo` Type (5 min)
It's good practice to define the "shape" of your data. Create a new file `src/types/Todo.ts`.

```typescript
// src/types/Todo.ts
export interface Todo {
  id: number;
  text: string;
  completed: boolean;
}
```

### Step 3: Set Up The Main App State (15 min)
Let's clear out `src/App.vue` and set up the core state for our application. We need a list of todos, a way to track the next todo ID, and the current filter.

```vue
<!-- src/App.vue -->
<script setup lang="ts">
import { ref, computed } from 'vue';
import type { Todo } from './types/Todo';

// The main list of all todos
const todos = ref<Todo[]>([
  { id: 1, text: 'Learn Vue Basics', completed: true },
  { id: 2, text: 'Build a Todo App', completed: false },
  { id: 3, text: 'Master Vue', completed: false },
]);

// A ref to track the next available ID
const nextId = ref(4);

// TODO: We will add functions to add/remove/toggle todos here
</script>

<template>
  <main class="max-w-xl mx-auto py-12 px-4">
    <h1 class="text-4xl font-bold text-center mb-8">My To-Do List</h1>
    
    <!-- TODO: We will add the components here -->
    <div class="card bg-base-100 shadow-xl">
      <div class="card-body">
        <p>Your components will go here soon!</p>
      </div>
    </div>
  </main>
</template>
```

### Step 4: Create the `TodoItem` Component (20 min)
This component will be responsible for displaying a single todo item. It needs to accept a `todo` as a prop and emit events when the user wants to delete it or toggle its status.

Create a new file `src/components/TodoItem.vue`.

```vue
<!-- src/components/TodoItem.vue -->
<script setup lang="ts">
import type { Todo } from '../types/Todo';

// This component receives a 'todo' object as a prop
defineProps<{
  todo: Todo;
}>();

// This component can emit two types of events
const emit = defineEmits<{
  (e: 'toggle'): void;
  (e: 'delete'): void;
}>();
</script>

<template>
  <div class="flex items-center gap-4 p-2 rounded-lg hover:bg-base-200">
    <input 
      type="checkbox" 
      :checked="todo.completed"
      class="checkbox checkbox-primary"
      @change="emit('toggle')" 
    />
    <span class="flex-1" :class="{ 'line-through text-base-content/50': todo.completed }">
      {{ todo.text }}
    </span>
    <button class="btn btn-ghost btn-square btn-sm" @click="emit('delete')">
      <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" /></svg>
    </button>
  </div>
</template>
```

### Step 5: Create `TodoList` and `TodoInput` Components (25 min)

**`TodoInput.vue`**: This component will have a form for adding new todos.

```vue
<!-- src/components/TodoInput.vue -->
<script setup lang="ts">
import { ref } from 'vue';

const newTodoText = ref('');
const emit = defineEmits<{
  (e: 'add-todo', text: string): void;
}>();

function addTodo() {
  if (newTodoText.value.trim()) {
    emit('add-todo', newTodoText.value.trim());
    newTodoText.value = ''; // Clear the input
  }
}
</script>

<template>
  <form class="flex gap-2" @submit.prevent="addTodo">
    <input 
      v-model="newTodoText"
      type="text" 
      placeholder="Add a new task..." 
      class="input input-bordered w-full" 
    />
    <button type="submit" class="btn btn-primary">Add</button>
  </form>
</template>
```

**`TodoList.vue`**: This component will display the list of todos and handle filtering.

```vue
<!-- src/components/TodoList.vue -->
<script setup lang="ts">
import { ref, computed } from 'vue';
import type { Todo } from '../types/Todo';
import TodoItem from './TodoItem.vue';

const props = defineProps<{
  todos: Todo[];
}>();

const emit = defineEmits<{
  (e: 'toggle', id: number): void;
  (e: 'delete', id: number): void;
}>();

const filter = ref<'all' | 'active' | 'completed'>('all');

const filteredTodos = computed(() => {
  switch (filter.value) {
    case 'active':
      return props.todos.filter(t => !t.completed);
    case 'completed':
      return props.todos.filter(t => t.completed);
    default:
      return props.todos;
  }
});
</script>

<template>
  <div>
    <!-- Filter Buttons -->
    <div class="tabs tabs-boxed mb-4">
      <a class="tab" :class="{ 'tab-active': filter === 'all' }" @click="filter = 'all'">All</a> 
      <a class="tab" :class="{ 'tab-active': filter === 'active' }" @click="filter = 'active'">Active</a> 
      <a class="tab" :class="{ 'tab-active': filter === 'completed' }" @click="filter = 'completed'">Completed</a>
    </div>

    <!-- Todo List -->
    <div class="space-y-2">
      <TodoItem 
        v-for="todo in filteredTodos"
        :key="todo.id"
        :todo="todo"
        @toggle="emit('toggle', todo.id)"
        @delete="emit('delete', todo.id)"
      />
      <p v-if="filteredTodos.length === 0" class="text-center text-base-content/50">
        No tasks here.
      </p>
    </div>
  </div>
</template>
```

### Step 6: Assemble Everything in `App.vue` (20 min)
Now, let's put all the pieces together in `App.vue` and add the logic to handle the events.

```vue
<!-- src/App.vue -->
<script setup lang="ts">
import { ref, computed } from 'vue';
import type { Todo } from './types/Todo';
import TodoInput from './components/TodoInput.vue';
import TodoList from './components/TodoList.vue';

const todos = ref<Todo[]>([
  { id: 1, text: 'Learn Vue Basics', completed: true },
  { id: 2, text: 'Build a Todo App', completed: false },
  { id: 3, text: 'Master Vue', completed: false },
]);

const nextId = ref(4);

// Function to add a new todo
function handleAddTodo(text: string) {
  todos.value.push({
    id: nextId.value++,
    text: text,
    completed: false,
  });
}

// Function to toggle a todo's completed status
function handleToggle(id: number) {
  const todo = todos.value.find(t => t.id === id);
  if (todo) {
    todo.completed = !todo.completed;
  }
}

// Function to delete a todo
function handleDelete(id: number) {
  todos.value = todos.value.filter(t => t.id !== id);
}
</script>

<template>
  <main class="max-w-xl mx-auto py-12 px-4">
    <h1 class="text-4xl font-bold text-center mb-8">My To-Do List</h1>
    
    <div class="card bg-base-100 shadow-xl">
      <div class="card-body">
        <!-- Input Component -->
        <TodoInput @add-todo="handleAddTodo" class="mb-6" />

        <!-- List Component -->
        <TodoList 
          :todos="todos"
          @toggle="handleToggle"
          @delete="handleDelete"
        />
      </div>
    </div>
  </main>
</template>
```

Your application should now be fully functional!

---

### Stretch Goals (Optional)
*   **[ ] Persist to LocalStorage:** Use a `watch` function on the `todos` ref to save its contents to `localStorage` whenever it changes. When the app loads, try to read from `localStorage` first.
*   **[ ] Add a "Clear Completed" Button:** Add a button that filters out all completed todos from the main `todos` array.
*   **[ ] Animate List Items:** Wrap the `TodoItem` in the `v-for` loop with Vue's built-in `<TransitionGroup>` component to add animations when items are added or removed.

---

### Self-Assessment
*   [ ] My application correctly adds new tasks to the list.
*   [ ] I can toggle a task's completed status using the checkbox.
*   [ ] The task text gets a line-through style when it's completed.
*   [ ] I can delete tasks from the list.
*   [ ] The filter buttons correctly show/hide tasks based on their status.
*   [ ] My code is split into at least three components (`App`, `TodoInput`, `TodoList`, `TodoItem`).
*   [ ] `props` are used to pass data down to child components.
*   [ ] `emits` are used to send information from child components back up to `App.vue`.
*   [ ] A `computed` property is used for the filtering logic.
