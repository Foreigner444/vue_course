# Lesson 16.1: Fetching Data: Async/Await & Lifecycle — Theory & Concepts (Получение данных: Async/Await и жизненный цикл)

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 16.1 of 25 (Part 1 of 2) |
| **Topic** | Fetching Data: Async/Await & Lifecycle / Получение данных: Async/Await и жизненный цикл |
| **Module** | Module 7: Data Fetching & APIs |
| **Prerequisites** | Lessons 1-5 (Core Fundamentals), Lesson 15 (Composables) |
| **You Should Know** | JavaScript Promises, async/await syntax, try/catch error handling, HTTP basics (GET, POST) |
| **Unlocks** | Lesson 16.2, then Lesson 17 (useFetch Composable) |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 1 (Theory):**
1. **Remember:** Define the Vue 3 component lifecycle hooks and identify which hook is appropriate for data fetching
2. **Understand:** Explain how async/await works with Vue's reactivity system and why lifecycle timing matters for API calls

---

## 2. Real-World Scenario & Context

**Scenario:** You're building a GitHub user profile viewer. When a user visits the page, the app needs to fetch profile data from GitHub's API and display the user's avatar, name, bio, and repository count. The data doesn't exist in your app — it lives on GitHub's servers. Your component must reach out over the network, wait for the response, and then display it.

**Framework Context:** Vue components need a way to perform asynchronous operations (like API calls) at the right moment in their lifecycle. Vue provides lifecycle hooks that let you "hook into" specific moments — like when a component is first created or mounted to the DOM — to trigger these operations safely.

---

## 3. Core Concepts Explained (Deep Dive)

### 3.1 How It Actually Works: The Component Lifecycle

When Vue creates a component, it goes through a series of phases. Think of it like building a house:

```
┌─────────────────────────────────────────────────────────────────┐
│                    VUE COMPONENT LIFECYCLE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   1. SETUP PHASE                                                 │
│   ┌──────────────────────────────────────────────────────┐      │
│   │  <script setup> runs                                  │      │
│   │  • refs and reactive objects created                  │      │
│   │  • computed properties defined                        │      │
│   │  • watchers registered                                │      │
│   │  ⚠️ DOM does NOT exist yet!                           │      │
│   └──────────────────────────────────────────────────────┘      │
│                            │                                     │
│                            ▼                                     │
│   2. MOUNT PHASE                                                 │
│   ┌──────────────────────────────────────────────────────┐      │
│   │  onBeforeMount() → Component about to be inserted     │      │
│   │                            │                          │      │
│   │                            ▼                          │      │
│   │  onMounted() → Component IS in the DOM ✅             │      │
│   │  • Safe to access DOM elements                        │      │
│   │  • ⭐ IDEAL for fetching data                         │      │
│   └──────────────────────────────────────────────────────┘      │
│                            │                                     │
│                            ▼                                     │
│   3. UPDATE PHASE (when reactive data changes)                   │
│   ┌──────────────────────────────────────────────────────┐      │
│   │  onBeforeUpdate() → About to re-render                │      │
│   │                            │                          │      │
│   │                            ▼                          │      │
│   │  onUpdated() → DOM has been updated                   │      │
│   └──────────────────────────────────────────────────────┘      │
│                            │                                     │
│                            ▼                                     │
│   4. UNMOUNT PHASE                                               │
│   ┌──────────────────────────────────────────────────────┐      │
│   │  onBeforeUnmount() → Component about to be destroyed  │      │
│   │                            │                          │      │
│   │                            ▼                          │      │
│   │  onUnmounted() → Component removed from DOM           │      │
│   │  • Clean up timers, subscriptions, etc.               │      │
│   └──────────────────────────────────────────────────────┘      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Key Insight:** `onMounted()` is the primary hook for data fetching because:
1. The component is fully initialized
2. The DOM exists (in case you need to show loading states)
3. It runs exactly once when the component appears

### 3.2 Mental Model: The Restaurant Analogy

Think of your Vue component as a restaurant:

```
┌────────────────────────────────────────────────────────────────┐
│                   🍽️ THE RESTAURANT ANALOGY                    │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SETUP = Setting up the kitchen                                 │
│  ├── Prepare all equipment (refs, reactive)                     │
│  ├── Plan the menu (computed, methods)                          │
│  └── Kitchen isn't open yet (no DOM)                            │
│                                                                 │
│  onMounted = Restaurant opens for business ✅                   │
│  ├── Customers can now enter (DOM exists)                       │
│  ├── Start taking orders (fetch data)                           │
│  └── Kitchen starts cooking (process responses)                 │
│                                                                 │
│  onUnmounted = Restaurant closes                                │
│  ├── Clean up everything                                        │
│  └── Cancel pending orders (abort fetch requests)               │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

### 3.3 Async/Await with Vue Reactivity

When you fetch data, you're dealing with **asynchronous operations** — operations that take time and don't block the rest of your code. Here's how it flows:

```
┌─────────────────────────────────────────────────────────────────┐
│                    ASYNC DATA FETCHING FLOW                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Component Mounts                                               │
│         │                                                        │
│         ▼                                                        │
│   ┌─────────────┐                                                │
│   │ loading = true │  ← Show spinner                             │
│   └─────────────┘                                                │
│         │                                                        │
│         ▼                                                        │
│   ┌─────────────────────────┐                                    │
│   │ await fetch(url)        │  ← Request sent to server          │
│   │    ... waiting ...      │  ← JavaScript continues            │
│   │    ... waiting ...      │  ← User sees loading state         │
│   │ response arrives!       │  ← Server responds                 │
│   └─────────────────────────┘                                    │
│         │                                                        │
│         ├──────────────────────┬───────────────────┐             │
│         ▼                      ▼                   ▼             │
│   ┌──────────┐          ┌──────────┐        ┌──────────┐         │
│   │ SUCCESS  │          │  ERROR   │        │ ABORTED  │         │
│   │ data.value│          │ error.value│      │ (unmount)│         │
│   │ = result │          │ = message│        │          │         │
│   └──────────┘          └──────────┘        └──────────┘         │
│         │                      │                   │             │
│         ▼                      ▼                   ▼             │
│   ┌─────────────┐                                                │
│   │ loading = false │  ← Hide spinner                            │
│   └─────────────┘                                                │
│         │                                                        │
│         ▼                                                        │
│   Vue re-renders with new data                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 3.4 In Other Contexts

| Framework | Data Fetching Approach |
|:--- |:--- |
| **Vue 3 (Composition API)** | `onMounted()` + `async/await` + reactive refs |
| **React** | `useEffect()` with empty dependency array |
| **Angular** | `ngOnInit()` lifecycle hook with RxJS Observables |
| **Vanilla JavaScript** | `DOMContentLoaded` event + `fetch()` |

### 3.5 When to Use / When NOT to Use

| ✅ Use `onMounted` for Fetching When... | ❌ Don't Use When... |
|:--- |:--- |
| Initial data load when component appears | Data depends on user input (use `watch` instead) |
| Component needs external data to render | Data is static and can be imported |
| One-time fetch on component creation | Polling/real-time updates (use `setInterval` in `onMounted`, clean up in `onUnmounted`) |
| Server-side rendering isn't required | SSR apps (use server-side data fetching solutions) |

---

## 4. New Terminology

| Term | Definition |
|:--- |:--- |
| **Lifecycle Hook** | A function that Vue calls at specific moments during a component's existence (creation, mounting, updating, destruction). In Composition API, these are imported functions like `onMounted`, `onUnmounted`. |
| **Asynchronous Operation** | An operation that doesn't complete immediately and doesn't block code execution. API calls, file reads, and timers are async. JavaScript uses Promises to handle them. |
| **Loading State** | A reactive variable (usually a boolean) that tracks whether an async operation is in progress. Used to show/hide loading indicators in the UI. |
| **Error State** | A reactive variable that stores error information when an async operation fails. Used to display error messages to users. |

---

## 5. Algorithmic Thinking (Planning the Solution)

**The Plan for Fetching Data in a Component:**

```
┌─────────────────────────────────────────────────────────────────┐
│              PLANNING: DATA FETCHING IN VUE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  STEP 1: Define Reactive State                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  What data will we receive?     → data = ref(null)      │    │
│  │  Are we currently loading?      → loading = ref(false)  │    │
│  │  Did something go wrong?        → error = ref(null)     │    │
│  └─────────────────────────────────────────────────────────┘    │
│                         │                                        │
│                         ▼                                        │
│  STEP 2: Create the Fetch Function                               │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  async function fetchData() {                            │    │
│  │    loading.value = true     // Start loading            │    │
│  │    error.value = null       // Clear previous errors    │    │
│  │    try {                                                 │    │
│  │      const response = await fetch(url)                  │    │
│  │      data.value = await response.json()                 │    │
│  │    } catch (e) {                                         │    │
│  │      error.value = e.message // Capture error           │    │
│  │    } finally {                                           │    │
│  │      loading.value = false  // Stop loading             │    │
│  │    }                                                     │    │
│  │  }                                                       │    │
│  └─────────────────────────────────────────────────────────┘    │
│                         │                                        │
│                         ▼                                        │
│  STEP 3: Call Fetch in onMounted                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  onMounted(() => {                                       │    │
│  │    fetchData()  // Trigger when component appears       │    │
│  │  })                                                      │    │
│  └─────────────────────────────────────────────────────────┘    │
│                         │                                        │
│                         ▼                                        │
│  STEP 4: Render Based on State                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  <template>                                              │    │
│  │    <div v-if="loading">Loading...</div>                 │    │
│  │    <div v-else-if="error">{{ error }}</div>             │    │
│  │    <div v-else>{{ data }}</div>                         │    │
│  │  </template>                                             │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Initial Pattern Introduction

| What You Want (Intent) | Code Chunk (The Pattern) | Conceptual Link |
|:--- |:--- |:--- |
| Store fetched data | `const data = ref<T \| null>(null)` | Reactive container for API response |
| Track loading status | `const loading = ref(false)` | Boolean flag for UI feedback |
| Capture errors | `const error = ref<string \| null>(null)` | Store error messages for display |
| Run code when mounted | `onMounted(() => { ... })` | Lifecycle hook for initialization |
| Make async request | `await fetch(url)` | Native browser API for HTTP calls |
| Parse JSON response | `await response.json()` | Convert response body to object |
| Handle errors | `try { } catch (e) { }` | Error boundary for async code |

```vue
<script setup lang="ts">
// Import lifecycle hook and reactivity functions
import { ref, onMounted } from 'vue'

// Define TypeScript interface for the expected data shape
interface User {
  id: number
  name: string
  email: string
}

// STEP 1: Create reactive state for all possible states
const user = ref<User | null>(null)      // Will hold the fetched user
const loading = ref(false)                // Tracks if request is in progress
const error = ref<string | null>(null)   // Stores error message if request fails

// STEP 2: Define async function to fetch data
async function fetchUser() {
  loading.value = true           // Indicate loading has started
  error.value = null             // Clear any previous errors
  
  try {
    // Make the HTTP GET request
    const response = await fetch('https://jsonplaceholder.typicode.com/users/1')
    
    // Check if the request was successful (status 200-299)
    if (!response.ok) {
      throw new Error(`HTTP error! Status: ${response.status}`)
    }
    
    // Parse the JSON body and store in reactive ref
    user.value = await response.json()
  } catch (e) {
    // Capture any errors (network issues, parsing errors, etc.)
    error.value = e instanceof Error ? e.message : 'An unknown error occurred'
  } finally {
    // Always set loading to false when done (success or failure)
    loading.value = false
  }
}

// STEP 3: Call the fetch function when component mounts
onMounted(() => {
  fetchUser()
})
</script>

<template>
  <!-- STEP 4: Render different UI based on current state -->
  <div class="p-4">
    <!-- Loading State -->
    <div v-if="loading" class="flex items-center gap-2">
      <span class="loading loading-spinner loading-md"></span>
      <span>Loading user data...</span>
    </div>
    
    <!-- Error State -->
    <div v-else-if="error" class="alert alert-error">
      <span>Error: {{ error }}</span>
    </div>
    
    <!-- Success State -->
    <div v-else-if="user" class="card bg-base-100 shadow-xl">
      <div class="card-body">
        <h2 class="card-title">{{ user.name }}</h2>
        <p>{{ user.email }}</p>
      </div>
    </div>
  </div>
</template>
```

---

## 7. Comprehension Check

1. **Why do we use `onMounted()` instead of calling `fetch()` directly in `<script setup>`?**
   - Think about: When does `<script setup>` run? Is the DOM available then?

2. **What are the three reactive states typically needed when fetching data, and why is each important?**
   - Think about: What does the user need to see during different phases of the request?

3. **True or False: You should use `onUpdated()` for fetching data that needs to refresh when a prop changes.**
   - Think about: What is `onUpdated` designed for? Is there a better alternative?

---

*Reply 'next' for Lesson 16.2 (Practice).*
