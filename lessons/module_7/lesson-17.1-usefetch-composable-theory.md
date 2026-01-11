# Lesson 17.1: Building a `useFetch` Composable — Theory & Concepts (Создание композабла useFetch)

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 17.1 of 25 (Part 1 of 2) |
| **Topic** | Building a `useFetch` Composable / Создание композабла useFetch |
| **Module** | Module 7: Data Fetching & APIs |
| **Prerequisites** | Lesson 15 (Composables), Lesson 16 (Async/Await & Lifecycle) |
| **You Should Know** | TypeScript generics, async/await, Vue reactivity (`ref`, `watch`) |
| **Unlocks** | Lesson 17.2, Module 7 Checkpoint Projects, Module 8 (Routing) |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 1 (Theory):**
1. **Remember:** List the essential features a data fetching composable should provide (data, loading, error, refetch)
2. **Understand:** Explain how to encapsulate async logic in a reusable composable with TypeScript generics

---

## 2. Real-World Scenario & Context

**Scenario:** Your team is building a dashboard application with multiple pages — a user list, a product catalog, an orders table, and a settings page. Each page needs to fetch data from different API endpoints. You notice you're copying the same loading/error/data pattern across every component. When a bug is found in the error handling, you have to fix it in 10 different places. There must be a better way...

**Framework Context:** Vue's Composition API enables **composables** — reusable functions that encapsulate reactive state and logic. A `useFetch` composable extracts the repetitive data fetching pattern into a single, tested, reusable function. This follows the DRY principle (Don't Repeat Yourself) and makes your codebase more maintainable.

---

## 3. Core Concepts Explained (Deep Dive)

### 3.1 Why Build a Composable for Fetching?

```
┌─────────────────────────────────────────────────────────────────┐
│              WITHOUT useFetch: Code Duplication                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   UserList.vue          ProductGrid.vue       OrdersTable.vue   │
│   ┌──────────────┐      ┌──────────────┐      ┌──────────────┐  │
│   │ const users  │      │ const products│      │ const orders │  │
│   │ const loading│      │ const loading │      │ const loading│  │
│   │ const error  │      │ const error   │      │ const error  │  │
│   │              │      │               │      │              │  │
│   │ async fetch()│      │ async fetch() │      │ async fetch()│  │
│   │   try/catch  │      │   try/catch   │      │   try/catch  │  │
│   │   finally    │      │   finally     │      │   finally    │  │
│   └──────────────┘      └──────────────┘      └──────────────┘  │
│         │                      │                     │          │
│         └──────────────────────┼─────────────────────┘          │
│                                │                                 │
│                    😫 Same code repeated 3x!                     │
│                    😫 Bug fix needed in 3 places                 │
│                    😫 Hard to maintain                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│              WITH useFetch: Single Source of Truth               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│                      useFetch.ts                                 │
│                      ┌──────────────┐                            │
│                      │ export function│                          │
│                      │ useFetch<T>() │                           │
│                      │   data        │                           │
│                      │   loading     │                           │
│                      │   error       │                           │
│                      │   refetch()   │                           │
│                      └──────────────┘                            │
│                             │                                    │
│           ┌─────────────────┼─────────────────┐                  │
│           │                 │                 │                  │
│           ▼                 ▼                 ▼                  │
│   UserList.vue      ProductGrid.vue    OrdersTable.vue          │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│   │const { data }│  │const { data }│  │const { data }│          │
│   │ = useFetch() │  │ = useFetch() │  │ = useFetch() │          │
│   └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                  │
│                    ✅ Logic in ONE place                         │
│                    ✅ Fix bugs once                              │
│                    ✅ Easy to test                               │
│                    ✅ Consistent behavior                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 Mental Model: The Factory Pattern

Think of `useFetch` as a **factory** that produces data fetching machinery:

```
┌────────────────────────────────────────────────────────────────┐
│                   🏭 THE FACTORY ANALOGY                        │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  useFetch() is like a VENDING MACHINE FACTORY                   │
│                                                                 │
│  INPUT: URL + Options (what you want to fetch)                  │
│         │                                                       │
│         ▼                                                       │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    useFetch Factory                      │   │
│  │  ┌─────────────────────────────────────────────────┐    │   │
│  │  │  Creates reactive state containers               │    │   │
│  │  │  Sets up the fetch logic                         │    │   │
│  │  │  Handles loading/error states                    │    │   │
│  │  │  Provides refetch capability                     │    │   │
│  │  └─────────────────────────────────────────────────┘    │   │
│  └─────────────────────────────────────────────────────────┘   │
│         │                                                       │
│         ▼                                                       │
│  OUTPUT: A "vending machine" customized for YOUR data           │
│         ┌────────────────────────────────────────────┐         │
│         │  { data, loading, error, refetch }         │         │
│         │                                            │         │
│         │  data     → The product (your API data)    │         │
│         │  loading  → "Processing..." light          │         │
│         │  error    → "Out of order" message         │         │
│         │  refetch  → Press button to try again      │         │
│         └────────────────────────────────────────────┘         │
│                                                                 │
│  EACH component gets its OWN independent machine!               │
│  UserList gets one, ProductGrid gets another, etc.              │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

### 3.3 Anatomy of a Fetch Composable

```
┌─────────────────────────────────────────────────────────────────┐
│                  useFetch COMPOSABLE ANATOMY                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  export function useFetch<T>(url: string, options?: Options)     │
│                         ▲                          ▲             │
│                         │                          │             │
│                    Generic Type              Optional config     │
│                    (what shape is             (headers, etc.)    │
│                     the response?)                               │
│                                                                  │
│  INSIDE THE COMPOSABLE:                                          │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                                                          │    │
│  │  1. REACTIVE STATE                                       │    │
│  │     const data = ref<T | null>(null)                     │    │
│  │     const loading = ref(false)                           │    │
│  │     const error = ref<string | null>(null)               │    │
│  │                                                          │    │
│  │  2. FETCH FUNCTION                                       │    │
│  │     async function execute() {                           │    │
│  │       loading.value = true                               │    │
│  │       try {                                              │    │
│  │         const response = await fetch(url)                │    │
│  │         data.value = await response.json()               │    │
│  │       } catch (e) {                                      │    │
│  │         error.value = e.message                          │    │
│  │       } finally {                                        │    │
│  │         loading.value = false                            │    │
│  │       }                                                  │    │
│  │     }                                                    │    │
│  │                                                          │    │
│  │  3. AUTO-FETCH ON MOUNT (optional)                       │    │
│  │     onMounted(execute)                                   │    │
│  │                                                          │    │
│  │  4. WATCH FOR URL CHANGES (optional)                     │    │
│  │     watch(() => url, execute)                            │    │
│  │                                                          │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
│  RETURN VALUE:                                                   │
│  return {                                                        │
│    data,        // Ref<T | null> - the fetched data             │
│    loading,     // Ref<boolean> - is request in progress?       │
│    error,       // Ref<string | null> - error message           │
│    refetch: execute  // Function to manually re-fetch           │
│  }                                                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 3.4 TypeScript Generics: The Power of `<T>`

Generics allow your composable to work with ANY data type while maintaining type safety:

```typescript
// WITHOUT generics - you'd need a different function for each type
function useFetchUsers(): { data: Ref<User[] | null>, ... }
function useFetchProducts(): { data: Ref<Product[] | null>, ... }
function useFetchOrders(): { data: Ref<Order[] | null>, ... }

// WITH generics - one function works for ALL types
function useFetch<T>(url: string): { data: Ref<T | null>, ... }

// Usage - TypeScript knows the exact type!
const { data: users } = useFetch<User[]>('/api/users')
//     ^-- TypeScript knows: users is Ref<User[] | null>

const { data: product } = useFetch<Product>('/api/products/1')
//     ^-- TypeScript knows: product is Ref<Product | null>
```

### 3.5 In Other Contexts

| Framework/Library | Equivalent Pattern |
|:--- |:--- |
| **Vue 3 (this lesson)** | `useFetch()` composable |
| **React** | Custom `useFetch` hook, or libraries like `react-query`, `SWR` |
| **Angular** | Services with RxJS, or `@ngneat/query` |
| **Svelte** | Store-based fetching or `@tanstack/svelte-query` |
| **VueUse** | `useFetch()` — a production-ready version of what we're building |

### 3.6 When to Use / When NOT to Use

| ✅ Use a `useFetch` Composable When... | ❌ Consider Alternatives When... |
|:--- |:--- |
| Multiple components fetch data the same way | You need advanced caching (use TanStack Query / VueQuery) |
| You want consistent loading/error handling | You need real-time updates (use WebSockets) |
| You want to reduce code duplication | Server-side rendering is critical (use Nuxt's `useFetch`) |
| Your app has simple data fetching needs | You need request deduplication and background refresh |

---

## 4. New Terminology

| Term | Definition |
|:--- |:--- |
| **Composable** | A function that uses Vue's Composition API to encapsulate and reuse stateful logic. By convention, composable function names start with `use` (e.g., `useFetch`, `useCounter`). |
| **Generic Type (`<T>`)** | A TypeScript feature that allows you to create components/functions that work with multiple types while maintaining type safety. `T` is a placeholder that gets replaced with an actual type when the function is called. |
| **Return Object Pattern** | A pattern where composables return an object with named properties (like `{ data, loading, error }`), allowing consumers to destructure only what they need. |
| **Immediate Fetch** | A composable option that determines whether data is fetched automatically when the composable is called, or only when manually triggered via a function. |

---

## 5. Algorithmic Thinking (Planning the Solution)

**The Plan for Building `useFetch`:**

```
┌─────────────────────────────────────────────────────────────────┐
│              PLANNING: Building useFetch Step-by-Step            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  STEP 1: Define the Return Type Interface                        │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  What should useFetch return?                            │    │
│  │  • data: Ref<T | null>       → The fetched data          │    │
│  │  • loading: Ref<boolean>     → Request in progress?      │    │
│  │  • error: Ref<string | null> → Error message             │    │
│  │  • refetch: () => Promise    → Manual re-fetch function  │    │
│  └─────────────────────────────────────────────────────────┘    │
│                         │                                        │
│                         ▼                                        │
│  STEP 2: Define the Function Signature                           │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  What inputs does useFetch need?                         │    │
│  │  • url: string | Ref<string>  → The endpoint             │    │
│  │  • options?: { immediate?: boolean } → Config            │    │
│  │                                                          │    │
│  │  function useFetch<T>(url, options): UseFetchReturn<T>   │    │
│  └─────────────────────────────────────────────────────────┘    │
│                         │                                        │
│                         ▼                                        │
│  STEP 3: Create Reactive State Inside                            │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  const data = ref<T | null>(null)                        │    │
│  │  const loading = ref(false)                              │    │
│  │  const error = ref<string | null>(null)                  │    │
│  └─────────────────────────────────────────────────────────┘    │
│                         │                                        │
│                         ▼                                        │
│  STEP 4: Create the Execute Function                             │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  async function execute() {                              │    │
│  │    loading.value = true                                  │    │
│  │    error.value = null                                    │    │
│  │    try {                                                 │    │
│  │      const response = await fetch(resolvedUrl)           │    │
│  │      if (!response.ok) throw new Error(...)              │    │
│  │      data.value = await response.json()                  │    │
│  │    } catch (e) { error.value = ... }                     │    │
│  │    finally { loading.value = false }                     │    │
│  │  }                                                       │    │
│  └─────────────────────────────────────────────────────────┘    │
│                         │                                        │
│                         ▼                                        │
│  STEP 5: Handle Auto-Fetch and URL Watching                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  if (options.immediate !== false) {                      │    │
│  │    onMounted(execute)  // Fetch on mount by default      │    │
│  │  }                                                       │    │
│  │                                                          │    │
│  │  if (isRef(url)) {                                       │    │
│  │    watch(url, execute)  // Re-fetch when URL changes     │    │
│  │  }                                                       │    │
│  └─────────────────────────────────────────────────────────┘    │
│                         │                                        │
│                         ▼                                        │
│  STEP 6: Return the Object                                       │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  return { data, loading, error, refetch: execute }       │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Initial Pattern Introduction

| What You Want (Intent) | Code Chunk (The Pattern) | Conceptual Link |
|:--- |:--- |:--- |
| Create typed composable | `function useFetch<T>(url: string)` | Generic type for flexible return |
| Reactive data storage | `const data = ref<T \| null>(null)` | Holds the API response |
| Track request status | `const loading = ref(false)` | UI shows spinner when true |
| Store error messages | `const error = ref<string \| null>(null)` | UI shows error when present |
| Expose refetch ability | `return { ..., refetch: execute }` | Allow manual data refresh |
| Auto-fetch on mount | `onMounted(execute)` | Fetch immediately by default |
| Handle reactive URLs | `watch(url, execute)` | Re-fetch when URL changes |

```typescript
// src/composables/useFetch.ts

import { ref, onMounted, watch, isRef, unref, type Ref } from 'vue'

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
  url: string | Ref<string>,
  options: UseFetchOptions = {}
): UseFetchReturn<T> {
  const { immediate = true } = options

  const data = ref<T | null>(null) as Ref<T | null>
  const loading = ref(false)
  const error = ref<string | null>(null)

  async function execute(): Promise<void> {
    loading.value = true
    error.value = null

    try {
      const resolvedUrl = unref(url)
      const response = await fetch(resolvedUrl)

      if (!response.ok) {
        throw new Error(`HTTP error! Status: ${response.status}`)
      }

      data.value = await response.json()
    } catch (e) {
      error.value = e instanceof Error ? e.message : 'An unknown error occurred'
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

  return {
    data,
    loading,
    error,
    refetch: execute
  }
}
```

**Usage in a Component:**
```vue
<script setup lang="ts">
import { useFetch } from '@/composables/useFetch'

interface User {
  id: number
  name: string
  email: string
}

const { data: users, loading, error, refetch } = useFetch<User[]>(
  'https://jsonplaceholder.typicode.com/users'
)
</script>

<template>
  <div class="p-4">
    <button @click="refetch" class="btn btn-primary mb-4" :disabled="loading">
      Refresh
    </button>

    <div v-if="loading" class="loading loading-spinner"></div>
    <div v-else-if="error" class="alert alert-error">{{ error }}</div>
    <ul v-else-if="users">
      <li v-for="user in users" :key="user.id">{{ user.name }}</li>
    </ul>
  </div>
</template>
```

---

## 7. Comprehension Check

1. **Why do we use a generic type `<T>` in `useFetch<T>` instead of a specific type like `User[]`?**
   - Think about: How many different API endpoints might use this composable? What types might they return?

2. **What is the purpose of `unref(url)` in the execute function?**
   - Think about: What happens if `url` is a `Ref<string>` vs a plain `string`?

3. **True or False: A composable creates a single shared state that all components using it will share.**
   - Think about: When you call `useFetch()` in two different components, do they get the same `data` ref or different ones?

---

*Reply 'next' for Lesson 17.2 (Practice).*
