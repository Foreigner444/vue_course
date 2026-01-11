# Lesson 10.1: List Rendering — Theory & Concepts (Отрисовка списков)

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 10.1 of 25 (Part 1 of 2) |
| **Topic** | List Rendering: `v-for` / Отрисовка списков |
| **Module** | Module 4: Rendering Patterns |
| **Prerequisites** | Lessons 1-5 (Core Fundamentals), Lesson 9 (Conditional Rendering) |
| **You Should Know** | JavaScript arrays, `for...of` loops, array methods (`map`, `filter`), object iteration |
| **Unlocks** | Lesson 10.2, then Module 5 (User Interaction) |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 1 (Theory):**
1. **Remember:** Define list rendering and explain the purpose of the `:key` attribute
2. **Understand:** Explain how Vue tracks list items for efficient DOM updates and why keys are critical

---

## 2. Real-World Scenario & Context

**Scenario:** You're building a task management application (like Todoist or Trello). Users have a list of tasks that can be added, removed, reordered, and filtered. The UI must display all tasks from an array, and when tasks change, only the affected items should update — not the entire list.

**Framework Context:** Vue's `v-for` directive solves the problem of rendering collections of data. Instead of manually creating DOM elements in a loop, you declare how a single item should look, and Vue automatically replicates it for every item in your array or object. Combined with Vue's reactivity system, changes to the underlying data automatically reflect in the UI.

---

## 3. Core Concepts Explained (Deep Dive)

### 3.1 How It Actually Works

When you use `v-for`, Vue performs these operations:

1. **Initial Render:** Vue iterates over your array/object and creates a DOM node for each item
2. **Tracking with Keys:** Vue uses the `:key` attribute to create a map between your data items and their DOM nodes
3. **Efficient Updates:** When data changes, Vue compares keys to determine the minimal DOM operations needed

```
┌─────────────────────────────────────────────────────────────┐
│                  v-for RENDERING FLOW                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Source Array                    Rendered DOM              │
│   ┌─────────────┐                ┌─────────────────┐        │
│   │ items = [   │                │ <ul>            │        │
│   │   {id:1,    │   v-for        │   <li key="1">  │        │
│   │    name:"A"}│ ════════════►  │     A           │        │
│   │   {id:2,    │                │   </li>         │        │
│   │    name:"B"}│                │   <li key="2">  │        │
│   │   {id:3,    │                │     B           │        │
│   │    name:"C"}│                │   </li>         │        │
│   │ ]           │                │   <li key="3">  │        │
│   └─────────────┘                │     C           │        │
│                                  │   </li>         │        │
│                                  │ </ul>           │        │
│                                  └─────────────────┘        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**The Key Mechanism (Critical Concept):**

```
┌─────────────────────────────────────────────────────────────┐
│              WHY KEYS MATTER: A VISUAL EXAMPLE              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   BEFORE: items = [{id:1,"A"}, {id:2,"B"}, {id:3,"C"}]      │
│                                                             │
│   WITHOUT KEYS (using index):                               │
│   ┌─────────────────────────────────────────┐               │
│   │ index 0 → "A"                           │               │
│   │ index 1 → "B"                           │               │
│   │ index 2 → "C"                           │               │
│   └─────────────────────────────────────────┘               │
│                                                             │
│   AFTER: items = [{id:2,"B"}, {id:3,"C"}]  (removed "A")    │
│                                                             │
│   WITHOUT KEYS - Vue thinks:                                │
│   ┌─────────────────────────────────────────┐               │
│   │ index 0: "A" → "B" (UPDATE content)     │ ❌ Wasteful   │
│   │ index 1: "B" → "C" (UPDATE content)     │ ❌ Wasteful   │
│   │ index 2: "C" → (REMOVE)                 │               │
│   └─────────────────────────────────────────┘               │
│   Total: 2 updates + 1 removal = 3 operations               │
│                                                             │
│   ──────────────────────────────────────────                │
│                                                             │
│   WITH KEYS (using id):                                     │
│   ┌─────────────────────────────────────────┐               │
│   │ key "1" → "A"                           │               │
│   │ key "2" → "B"                           │               │
│   │ key "3" → "C"                           │               │
│   └─────────────────────────────────────────┘               │
│                                                             │
│   AFTER: items = [{id:2,"B"}, {id:3,"C"}]                   │
│                                                             │
│   WITH KEYS - Vue thinks:                                   │
│   ┌─────────────────────────────────────────┐               │
│   │ key "1": (REMOVE - item gone)           │               │
│   │ key "2": "B" → "B" (no change)          │ ✅ Efficient  │
│   │ key "3": "C" → "C" (no change)          │ ✅ Efficient  │
│   └─────────────────────────────────────────┘               │
│   Total: 1 removal = 1 operation                            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 Mental Model

**`v-for` is like a cookie cutter for DOM elements:**

Imagine you're making cookies from dough (your array data). You have a single cookie cutter (your template). Vue takes that cutter and stamps out one cookie (DOM element) for each piece of dough (array item).

```
┌─────────────────────────────────────────────────────────────┐
│                   COOKIE CUTTER ANALOGY                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Your Array (Dough pieces):                                │
│   ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐                        │
│   │  1  │  │  2  │  │  3  │  │  4  │                        │
│   └─────┘  └─────┘  └─────┘  └─────┘                        │
│                                                             │
│   Your Template (Cookie Cutter):                            │
│   ┌─────────────────────┐                                   │
│   │  <li>{{ item }}</li>│  ← Single template                │
│   └─────────────────────┘                                   │
│                                                             │
│   Vue Stamps Out (Rendered Cookies):                        │
│   ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐           │
│   │ <li>1   │ │ <li>2   │ │ <li>3   │ │ <li>4   │           │
│   │ </li>   │ │ </li>   │ │ </li>   │ │ </li>   │           │
│   └─────────┘ └─────────┘ └─────────┘ └─────────┘           │
│                                                             │
│   Each cookie has a unique KEY (like a name tag)            │
│   so Vue knows which is which when rearranging!             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Keys are like name tags:** If you shuffle your cookies, the name tags tell you which cookie is which — you don't have to taste each one to figure it out!

### 3.3 In Other Contexts

| Framework | List Rendering Equivalent |
|:--- |:--- |
| **Vue** | `v-for="item in items" :key="item.id"` |
| **React** | `{items.map(item => <li key={item.id}>{item.name}</li>)}` |
| **Angular** | `*ngFor="let item of items; trackBy: trackById"` |
| **Vanilla JS** | `items.forEach(item => container.appendChild(createElement(...)))` |

**Key insight:** All modern frameworks require unique keys/track-by for efficient list rendering. This is a universal concept!

### 3.4 When to Use / When NOT to Use

```
┌─────────────────────────────────────────────────────────────┐
│               v-for USAGE DECISION GUIDE                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   USE v-for WHEN:                                           │
│   ┌─────────────────────────────────────────────────────┐   │
│   │ ✅ Rendering arrays of data (tasks, users, products)│   │
│   │ ✅ Iterating over object properties                 │   │
│   │ ✅ Generating repeated UI elements (grid items)     │   │
│   │ ✅ Creating navigation from route configs           │   │
│   │ ✅ Rendering form fields from schema                │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                             │
│   AVOID v-for WHEN:                                         │
│   ┌─────────────────────────────────────────────────────┐   │
│   │ ❌ You only have 2-3 static items (just write them) │   │
│   │ ❌ Combined with v-if on same element (use computed)│   │
│   │ ❌ Deeply nested loops (consider flattening data)   │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                             │
│   KEY SELECTION PRIORITY:                                   │
│   ┌─────────────────────────────────────────────────────┐   │
│   │ 1st: Unique ID from database (item.id, item.uuid)   │   │
│   │ 2nd: Unique business identifier (item.sku, email)   │   │
│   │ 3rd: Generated unique ID (crypto.randomUUID())      │   │
│   │ ❌ AVOID: Array index (except static, never-change) │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 4. New Terminology

| Term | Definition |
|:--- |:--- |
| **List Rendering** | The process of generating multiple DOM elements from an array or object of data |
| **`:key` Attribute** | A special Vue attribute that provides a unique identifier for each item in a `v-for` loop, enabling Vue's efficient DOM diffing algorithm |
| **DOM Diffing** | The algorithm Vue uses to compare the previous and current virtual DOM to determine the minimal changes needed |
| **Iteration Variable** | The temporary variable (e.g., `item` in `v-for="item in items"`) that holds the current item's value during each loop iteration |

---

## 5. Algorithmic Thinking (Planning the Solution)

**The Plan:** Building the task list scenario step-by-step.

```
┌─────────────────────────────────────────────────────────────┐
│          PLANNING FLOWCHART: Task List Rendering            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   START                                                     │
│     │                                                       │
│     ▼                                                       │
│   ┌─────────────────────────────────┐                       │
│   │ Step 1: Define the data shape   │                       │
│   │ • Task interface with id, title,│                       │
│   │   completed, priority           │                       │
│   └─────────────────┬───────────────┘                       │
│                     │                                       │
│                     ▼                                       │
│   ┌─────────────────────────────────┐                       │
│   │ Step 2: Create reactive array   │                       │
│   │ • ref<Task[]>([...])            │                       │
│   │ • Initialize with sample data   │                       │
│   └─────────────────┬───────────────┘                       │
│                     │                                       │
│                     ▼                                       │
│   ┌─────────────────────────────────┐                       │
│   │ Step 3: Design the item template│                       │
│   │ • How should ONE task look?     │                       │
│   │ • What data does it display?    │                       │
│   └─────────────────┬───────────────┘                       │
│                     │                                       │
│                     ▼                                       │
│   ┌─────────────────────────────────┐                       │
│   │ Step 4: Apply v-for directive   │                       │
│   │ • v-for="task in tasks"         │                       │
│   │ • :key="task.id" (critical!)    │                       │
│   └─────────────────┬───────────────┘                       │
│                     │                                       │
│                     ▼                                       │
│   ┌─────────────────────────────────┐                       │
│   │ Step 5: Add mutation methods    │                       │
│   │ • addTask(), removeTask()       │                       │
│   │ • toggleComplete(), etc.        │                       │
│   └─────────────────────────────────┘                       │
│                     │                                       │
│                     ▼                                       │
│                   END                                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Narrative:**

1. **Define the data shape:** Before writing any Vue code, define a TypeScript interface for your task. What properties does each task have? (`id`, `title`, `completed`, `priority`, `createdAt`, etc.)

2. **Create reactive array:** Use `ref<Task[]>([])` to create a reactive array that Vue will track. When items are added/removed/modified, the UI updates automatically.

3. **Design the item template:** Think about how a single task should appear. Write the HTML/CSS for ONE task without thinking about the loop yet.

4. **Apply v-for directive:** Wrap your single-item template with `v-for`. Always provide a `:key` using the item's unique identifier.

5. **Add mutation methods:** Create functions that modify the array (add, remove, update items). Vue's reactivity handles the rest!

---

## 6. Initial Pattern Introduction

| What You Want (Intent) | Code Chunk (The Pattern) | Conceptual Link |
|:--- |:--- |:--- |
| Render array of items | `v-for="item in items" :key="item.id"` | One DOM node per array element |
| Access item index | `v-for="(item, index) in items"` | Index is 0-based position |
| Iterate object properties | `v-for="(value, key) in object"` | Iterates over key-value pairs |
| Iterate with index on object | `v-for="(value, key, index) in object"` | All three available |
| Render a range of numbers | `v-for="n in 10"` | Renders 1 through 10 |

```vue
<script setup lang="ts">
// Step 1: Import ref for reactive state
import { ref } from 'vue'

// Step 2: Define the shape of our data with TypeScript
interface Task {
  id: number
  title: string
  completed: boolean
  priority: 'low' | 'medium' | 'high'
}

// Step 3: Create a reactive array of tasks
// The ref() makes the entire array reactive
const tasks = ref<Task[]>([
  { id: 1, title: 'Learn Vue basics', completed: true, priority: 'high' },
  { id: 2, title: 'Build a todo app', completed: false, priority: 'high' },
  { id: 3, title: 'Master TypeScript', completed: false, priority: 'medium' },
  { id: 4, title: 'Study Pinia', completed: false, priority: 'low' },
])

// Step 4: Create a counter for generating unique IDs
let nextId = 5

// Step 5: Function to add a new task
const addTask = (title: string) => {
  tasks.value.push({
    id: nextId++,
    title,
    completed: false,
    priority: 'medium'
  })
}

// Step 6: Function to remove a task by ID
const removeTask = (id: number) => {
  const index = tasks.value.findIndex(task => task.id === id)
  if (index !== -1) {
    tasks.value.splice(index, 1)
  }
}

// Step 7: Function to toggle task completion
const toggleTask = (id: number) => {
  const task = tasks.value.find(task => task.id === id)
  if (task) {
    task.completed = !task.completed
  }
}
</script>

<template>
  <div class="p-4">
    <h1 class="text-2xl font-bold mb-4">My Tasks</h1>
    
    <!--
      v-for SYNTAX BREAKDOWN:
      - "task" = iteration variable (each item in the loop)
      - "in" = separator keyword
      - "tasks" = source array (from ref)
      - ":key" = REQUIRED unique identifier for each item
      
      Vue uses :key to track which DOM elements correspond to which
      array items. This enables efficient updates when the array changes.
    -->
    <ul class="space-y-2">
      <li 
        v-for="task in tasks" 
        :key="task.id"
        class="flex items-center gap-3 p-3 bg-base-200 rounded-lg"
      >
        <!-- Checkbox to toggle completion -->
        <input 
          type="checkbox" 
          class="checkbox"
          :checked="task.completed"
          @change="toggleTask(task.id)"
        />
        
        <!-- Task title with conditional styling -->
        <span 
          class="flex-1"
          :class="{ 'line-through opacity-50': task.completed }"
        >
          {{ task.title }}
        </span>
        
        <!-- Priority badge -->
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
        
        <!-- Delete button -->
        <button 
          class="btn btn-ghost btn-sm"
          @click="removeTask(task.id)"
        >
          ✕
        </button>
      </li>
    </ul>
    
    <!-- Add task button (simplified for this example) -->
    <button 
      class="btn btn-primary mt-4"
      @click="addTask('New Task ' + nextId)"
    >
      + Add Task
    </button>
  </div>
</template>
```

**Key Syntax Forms:**

```vue
<!-- Basic array iteration -->
<li v-for="item in items" :key="item.id">{{ item.name }}</li>

<!-- With index -->
<li v-for="(item, index) in items" :key="item.id">
  {{ index + 1 }}. {{ item.name }}
</li>

<!-- Object iteration (value, key, index) -->
<div v-for="(value, key, index) in userProfile" :key="key">
  {{ index }}: {{ key }} = {{ value }}
</div>

<!-- Range iteration (1 to n) -->
<span v-for="n in 5" :key="n">{{ n }}</span>
<!-- Output: 1 2 3 4 5 -->

<!-- Using <template> for multiple root elements -->
<template v-for="item in items" :key="item.id">
  <dt>{{ item.term }}</dt>
  <dd>{{ item.definition }}</dd>
</template>
```

---

## 7. Comprehension Check

1. **Conceptual:** What is the purpose of the `:key` attribute in `v-for`, and what happens if you use array index as the key when items can be reordered or removed?

2. **Conceptual:** Given `const user = { name: 'Alice', age: 30, role: 'Developer' }`, what would `v-for="(value, key) in user"` iterate over? List all iterations.

3. **True/False:** The following code will render numbers 0 through 9:
   ```vue
   <span v-for="n in 10" :key="n">{{ n }}</span>
   ```

---

*Reply 'next' for Lesson 10.2 (Practice).*
