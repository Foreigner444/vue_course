# Lesson 15.1: Reusable Logic with Composables — Theory & Concepts (Переиспользуемая логика с Composables)
---
## 1. Lesson Metadata
| Field | Value |
|:--- |:--- |
| **Lesson Number** | 15.1 of 25 (Part 1 of 2) |
| **Topic** | Reusable Logic with Composables / Переиспользуемая логика с Composables |
| **Module** | Module 6: Reusability & Abstraction |
| **Prerequisites** | Lessons 2-5 (ref, reactive, computed, watch), Lesson 14 (Slots) |
| **You Should Know** | JavaScript functions, closures, destructuring, TypeScript generics basics |
| **Unlocks** | Lesson 15.2, then Lessons 16-17 (Data Fetching & APIs) |
| **Duration** | 30-35 minutes |
**Learning Objectives — Part 1 (Theory):**
1. **Remember:** Define what a composable is and list its key characteristics
2. **Understand:** Explain how composables enable logic reuse across components without code duplication
---
## 2. Real-World Scenario & Context
### Scenario
Imagine you're building a large e-commerce application. You have a shopping cart page, a wishlist page, and a product comparison page—all of which need to track item counts, handle localStorage persistence, and manage loading states. Without a strategy for reuse, you'd copy-paste the same reactive logic into each component, leading to maintenance nightmares when requirements change.
### Framework Context
Vue's Composition API was designed specifically to solve the "logic reuse" problem. **Composables** are the official pattern for extracting and sharing stateful logic between components—think of them as the Composition API's answer to React's custom hooks or Angular's services.
---
## 3. Core Concepts Explained (Deep Dive)
### 3.1 How Composables Actually Work
A composable is simply a **function** that:
1. Uses Vue's Composition API features (`ref`, `reactive`, `computed`, `watch`, lifecycle hooks)
2. Returns reactive state and/or functions that components can use
3. Can be called from `<script setup>` or other composables
**Under the Hood Flow:**
```
┌─────────────────────────────────────────────────────────────────────┐
│                        Component A                                   │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  <script setup>                                              │   │
│  │    const { count, increment } = useCounter()  ─────────────────────┐
│  │  </script>                                                   │   │ │
│  └─────────────────────────────────────────────────────────────┘   │ │
└─────────────────────────────────────────────────────────────────────┘ │
                                                                        │
┌─────────────────────────────────────────────────────────────────────┐ │
│                        Component B                                   │ │
│  ┌─────────────────────────────────────────────────────────────┐   │ │
│  │  <script setup>                                              │   │ │
│  │    const { count, increment } = useCounter()  ─────────────────────┤
│  │  </script>                                                   │   │ │
│  └─────────────────────────────────────────────────────────────┘   │ │
└─────────────────────────────────────────────────────────────────────┘ │
                                                                        │
                              ▼                                         │
┌─────────────────────────────────────────────────────────────────────┐ │
│                    useCounter.ts (Composable)                        │◄┘
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  export function useCounter(initial = 0) {                   │   │
│  │    const count = ref(initial)        // Each call creates    │   │
│  │    const increment = () => count.value++  // NEW instance    │   │
│  │    return { count, increment }                               │   │
│  │  }                                                           │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```
**Key Insight:** Each component that calls `useCounter()` gets its **own independent instance** of the reactive state. The logic is shared, but the state is isolated per component (unless you intentionally create a singleton pattern).
### 3.2 Mental Model: The "Recipe" Analogy
Think of a composable as a **recipe**, not a pre-cooked meal:
- **The Recipe (Composable Function):** Contains instructions for creating reactive state and behavior
- **Cooking the Recipe (Calling the Composable):** Each time you cook it, you get a fresh, independent dish
- **The Dish (Returned State):** What your component actually uses
```
┌──────────────────┐     Call      ┌──────────────────┐
│   📋 Recipe      │  ──────────►  │   🍳 Fresh Dish  │
│   (useCounter)   │               │   (Independent   │
│                  │               │    State)        │
└──────────────────┘               └──────────────────┘
        │
        │ Call Again
        ▼
┌──────────────────┐
│   🍳 Another     │
│   Fresh Dish     │
│   (Separate      │
│    State)        │
└──────────────────┘
```
### 3.3 Comparison with Other Frameworks
| Framework | Pattern | Key Difference |
|:--- |:--- |:--- |
| **Vue 3** | Composables (`useX`) | Functions returning reactive refs; works with Vue's reactivity |
| **React** | Custom Hooks (`useX`) | Similar pattern, but relies on React's hook rules and re-render cycle |
| **Angular** | Services + DI | Class-based, uses dependency injection; more boilerplate |
| **Vue 2** | Mixins | Object-based, prone to naming collisions; **deprecated** |
| **Vanilla JS** | Factory Functions | No reactivity; must manually trigger UI updates |
### 3.4 When to Use Composables
**✅ Use Composables When:**
- Logic is used in **2+ components** (DRY principle)
- You want to **organize** a large component by feature
- Logic involves **reactive state** (refs, computed, watchers)
- You need to **encapsulate** side effects (API calls, timers, event listeners)
**❌ Don't Use Composables When:**
- Logic is **simple and used once** (inline is fine)
- You only need to share **static utilities** (use plain functions instead)
- You're trying to share **state globally** (use Pinia stores instead)
```
┌─────────────────────────────────────────────────────────────┐
│              Decision Tree: What Pattern to Use?             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Is it reactive state/logic?                                │
│       │                                                     │
│       ├── NO ──► Use plain utility functions                │
│       │                                                     │
│       └── YES                                               │
│            │                                                │
│            ├── Needs global access across many components?  │
│            │       │                                        │
│            │       ├── YES ──► Use Pinia Store              │
│            │       │                                        │
│            │       └── NO                                   │
│            │            │                                   │
│            │            └── Used in 2+ components?          │
│            │                    │                           │
│            │                    ├── YES ──► Use Composable  │
│            │                    │                           │
│            │                    └── NO ──► Inline in        │
│            │                              component         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```
---
## 4. New Terminology
| Term | Definition |
|:--- |:--- |
| **Composable** | A function using Composition API that encapsulates and returns reactive state and logic. Named with `use` prefix by convention (e.g., `useCounter`, `useFetch`). |
| **Logic Extraction** | The practice of moving reactive code out of a component into a reusable composable function. |
| **Stateful Logic** | Code that maintains and modifies state over time (vs. pure functions that just compute values). |
| **Composition over Inheritance** | A design principle favoring small, focused functions that can be combined (composed) rather than class hierarchies. Vue's Composition API embodies this. |
---
## 5. Algorithmic Thinking (Planning the Solution)
When creating a composable, follow this mental process:
### The Plan (Narrative)
```
┌─────────────────────────────────────────────────────────────┐
│           Creating a Composable: Step-by-Step               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Step 1: IDENTIFY THE SHARED LOGIC                          │
│  ────────────────────────────────                           │
│  Look at your components. What reactive state and           │
│  behavior appears in multiple places?                       │
│                                                             │
│              ▼                                              │
│                                                             │
│  Step 2: EXTRACT TO A FUNCTION                              │
│  ─────────────────────────────                              │
│  Create a function named `useXxx` in /composables/          │
│  Move the reactive logic into this function                 │
│                                                             │
│              ▼                                              │
│                                                             │
│  Step 3: DEFINE PARAMETERS                                  │
│  ────────────────────────                                   │
│  What initial values or options does the logic need?        │
│  Accept them as function arguments with sensible defaults   │
│                                                             │
│              ▼                                              │
│                                                             │
│  Step 4: RETURN THE PUBLIC API                              │
│  ─────────────────────────────                              │
│  Return an object with the refs, computed values,           │
│  and functions the component needs to use                   │
│                                                             │
│              ▼                                              │
│                                                             │
│  Step 5: TYPE IT WITH TYPESCRIPT                            │
│  ───────────────────────────────                            │
│  Add types for parameters, return values, and generics      │
│  if the composable needs to be flexible                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```
---
## 6. Initial Pattern Introduction
### Basic Composable Structure
| What You Want (Intent) | Code Chunk (The Pattern) | Conceptual Link |
|:--- |:--- |:--- |
| Create reusable reactive state | `const state = ref(initial)` | Encapsulate state inside function |
| Derive values from state | `const derived = computed(() => ...)` | Computed works inside composables |
| React to state changes | `watch(state, (newVal) => ...)` | Side effects stay encapsulated |
| Expose public API | `return { state, actions }` | Component receives only what it needs |
### Example: A Simple Counter Composable
```typescript
// src/composables/useCounter.ts
// Import Vue's reactivity primitives
import { ref, computed } from 'vue'
// The composable function - named with "use" prefix by convention
export function useCounter(initialValue: number = 0) {
  // ┌─────────────────────────────────────────────────────────┐
  // │ PRIVATE STATE: Created fresh each time function is called│
  // └─────────────────────────────────────────────────────────┘
  const count = ref(initialValue)
  // ┌─────────────────────────────────────────────────────────┐
  // │ DERIVED STATE: Automatically updates when count changes  │
  // └─────────────────────────────────────────────────────────┘
  const doubled = computed(() => count.value * 2)
  const isPositive = computed(() => count.value > 0)
  // ┌─────────────────────────────────────────────────────────┐
  // │ ACTIONS: Functions that modify the state                 │
  // └─────────────────────────────────────────────────────────┘
  function increment() {
    count.value++
  }
  function decrement() {
    count.value--
  }
  function reset() {
    count.value = initialValue
  }
  // ┌─────────────────────────────────────────────────────────┐
  // │ PUBLIC API: Only expose what components need to use      │
  // └─────────────────────────────────────────────────────────┘
  return {
    // State (reactive refs)
    count,
    doubled,
    isPositive,
    // Actions (functions)
    increment,
    decrement,
    reset
  }
}
```
### Using the Composable in a Component
```vue
<!-- src/components/CounterDisplay.vue -->
<script setup lang="ts">
// Import the composable
import { useCounter } from '@/composables/useCounter'
// Call the composable - this creates a fresh instance of all state
// Destructure to get the specific pieces you need
const { count, doubled, increment, decrement, reset } = useCounter(10)
// You can also use multiple independent counters:
const likes = useCounter(0)
const comments = useCounter(0)
</script>
<template>
  <div class="card bg-base-200 p-6">
    <!-- Main counter display -->
    <h2 class="text-2xl font-bold">Count: {{ count }}</h2>
    <p class="text-sm text-gray-500">Doubled: {{ doubled }}</p>
    
    <!-- Action buttons -->
    <div class="flex gap-2 mt-4">
      <button class="btn btn-primary" @click="increment">+</button>
      <button class="btn btn-secondary" @click="decrement">-</button>
      <button class="btn btn-ghost" @click="reset">Reset</button>
    </div>
    <!-- Independent counters -->
    <div class="mt-6 flex gap-4">
      <span>👍 {{ likes.count }}</span>
      <span>💬 {{ comments.count }}</span>
    </div>
  </div>
</template>
```
---
## 7. Comprehension Check
Test your understanding before moving to practice:
1. **Conceptual:** If two different components both call `useCounter()`, do they share the same `count` value, or does each get its own? Explain why.
2. **Conceptual:** Why do composables use the `use` prefix (like `useCounter`, `useFetch`)? What benefit does this naming convention provide?
3. **True or False:** Composables can only return reactive refs—they cannot return computed values or functions.
<details>
<summary>Click to reveal answers</summary>
1. **Each component gets its own independent `count` value.** When you call `useCounter()`, the function executes and creates new `ref()` instances. Each call = new state. The logic (the function code) is shared, but the state is isolated. This is why it's like a "recipe"—each time you cook it, you get a fresh dish.
2. **The `use` prefix is a convention that signals "this function uses Composition API features."** It tells developers: (a) this function contains reactive state, (b) it should be called inside `<script setup>` or another composable, (c) it follows the composable pattern. This is similar to React's hook naming convention.
3. **False.** Composables can return anything: refs, reactive objects, computed values, plain functions, or even non-reactive data. The return value is just a regular JavaScript object—you decide what to expose.
</details>
---
**Reply 'next' for Lesson 15.2 (Practice).**
