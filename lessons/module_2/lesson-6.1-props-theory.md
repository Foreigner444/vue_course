# Lesson 6.1: Component Communication: Passing Props — Theory & Concepts (Передача Props)
---
## 1. Lesson Metadata
| Field | Value |
|:--- |:--- |
| **Lesson Number** | 6.1 of 25 (Part 1 of 2) |
| **Topic** | Passing Props / Передача Props |
| **Module** | Module 2: Component Communication |
| **Prerequisites** | Lessons 1-5 (Core Fundamentals) |
| **You Should Know** | TypeScript interfaces, generic types, object destructuring |
| **Unlocks** | Lesson 6.2, then Lesson 7 (Emitting Events) |
| **Duration** | 30-35 minutes |
**Learning Objectives — Part 1 (Theory):**
1. **Remember:** Define what props are and list the rules for prop flow in Vue
2. **Understand:** Explain how Vue's one-way data flow works and why it exists
---
## 2. Real-World Scenario & Context
**Scenario:** You're building a user profile page for a social media application. The main page component holds user data fetched from an API, but you need to display this information in multiple child components: a `ProfileHeader` showing the avatar and name, a `ProfileStats` showing follower counts, and a `ProfileBio` showing the user's description. Each child component needs access to specific pieces of the user data, but they shouldn't fetch it themselves or modify it directly.
**Framework Context:** Vue solves the problem of passing data from parent to child components through **props** — a mechanism that establishes a clear, predictable, one-way data flow. This pattern prevents chaotic data mutations and makes your application easier to debug and reason about.
---
## 3. Core Concepts Explained (Deep Dive)
### 3.1 How Props Actually Work
When you define props in a child component, Vue creates a **one-way binding** from the parent's data to the child's props. Here's what happens under the hood:
```
┌─────────────────────────────────────────────────────────────────┐
│                        PARENT COMPONENT                         │
│                                                                 │
│   const user = ref({ name: 'Alice', followers: 1000 })         │
│                           │                                     │
│                           ▼                                     │
│   <ProfileHeader :name="user.name" :followers="user.followers"/>│
│                           │                                     │
└───────────────────────────┼─────────────────────────────────────┘
                            │
                   Vue's Reactivity System
                   (creates one-way binding)
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                        CHILD COMPONENT                          │
│                                                                 │
│   defineProps<{ name: string; followers: number }>()           │
│                                                                 │
│   Props are READ-ONLY reactive references                       │
│   ✅ Can read: props.name → "Alice"                            │
│   ❌ Cannot write: props.name = "Bob" → ERROR                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```
**The Reactivity Chain:**
1. Parent's reactive data changes (e.g., `user.name` updates)
2. Vue detects the change through its reactivity system
3. Vue automatically updates the prop value in the child
4. Child component re-renders with the new prop value
5. Child **cannot** send changes back up through the prop
### 3.2 Mental Model: The "Read-Only Contract"
Think of props like a **newspaper subscription**:
```
┌──────────────────┐         ┌──────────────────┐
│     PARENT       │         │      CHILD       │
│   (Publisher)    │         │   (Subscriber)   │
│                  │         │                  │
│  Creates the     │ ──────► │  Receives and    │
│  content         │  Props  │  reads content   │
│                  │         │                  │
│  Can update      │         │  Cannot edit     │
│  anytime         │         │  the newspaper   │
└──────────────────┘         └──────────────────┘
```
- The **parent** (publisher) creates and owns the data
- The **child** (subscriber) receives a read-only copy
- When the parent updates, all subscribers get the new version
- Subscribers cannot write back to the publisher through the newspaper
### 3.3 In Other Contexts
| Framework | Props Equivalent | Key Difference |
|:--- |:--- |:--- |
| **React** | `props` object, destructured in function params | Nearly identical pattern, also read-only |
| **Angular** | `@Input()` decorator | Similar concept, uses decorator syntax |
| **Svelte** | `export let propName` | Props are mutable by default in child |
| **Vanilla JS** | Function parameters | No reactivity, manual DOM updates needed |
### 3.4 When to Use / When NOT to Use Props
**✅ Use Props When:**
- Passing data from parent to immediate child
- The child only needs to **read** the data
- You want clear, traceable data flow
- Data is owned and managed by a parent component
**❌ Don't Use Props When:**
- Data needs to flow from child to parent (use **events** instead — Lesson 7)
- Passing data through many nested levels (use **provide/inject** or **Pinia**)
- Siblings need to share state (use a **shared store** — Module 9)
- The child needs to modify the data (use **v-model** — Lesson 13)
---
## 4. New Terminology
| Term | Definition |
|:--- |:--- |
| **Props** | Short for "properties" — custom attributes you register on a component to pass data from parent to child |
| **One-Way Data Flow** | A design pattern where data flows in a single direction (parent → child), making state changes predictable |
| **Prop Validation** | Vue's built-in mechanism to declare expected types, required status, and default values for props |
| **Type Inference** | TypeScript's ability to automatically determine types from `defineProps<T>()` generic parameter |
---
## 5. Algorithmic Thinking (Planning the Solution)
**The Plan for Implementing Props:**
```
START: Parent has data that child needs
           │
           ▼
┌─────────────────────────────────────┐
│ STEP 1: Define the Props Interface  │
│ What data does the child need?      │
│ What are the types?                 │
│ Which props are required/optional?  │
└─────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────┐
│ STEP 2: Declare Props in Child      │
│ Use defineProps<Interface>()        │
│ TypeScript enforces the contract    │
└─────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────┐
│ STEP 3: Pass Props from Parent      │
│ Use :propName="value" syntax        │
│ Static: propName="value"            │
│ Dynamic: :propName="variable"       │
└─────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────┐
│ STEP 4: Use Props in Child Template │
│ Access via props.propName           │
│ Or destructure (with care)          │
└─────────────────────────────────────┘
           │
           ▼
END: Child displays parent's data
```
---
## 6. Initial Pattern Introduction
| What You Want (Intent) | Code Chunk (The Pattern) | Conceptual Link |
|:--- |:--- |:--- |
| Declare what data the child accepts | `defineProps<{ name: string }>()` | Establishes the contract |
| Pass data from parent to child | `<Child :name="userName" />` | Fulfills the contract |
| Access prop value in child template | `{{ name }}` or `{{ props.name }}` | Uses the received data |
| Make a prop optional with default | `name?: string` + `withDefaults()` | Flexible contracts |
### Basic Props Pattern
```vue
<!-- ChildComponent.vue -->
<script setup lang="ts">
// STEP 1 & 2: Define the props interface using TypeScript generics
// defineProps<T>() tells Vue what props this component accepts
// The generic type T describes the shape of the props object
const props = defineProps<{
  // Required prop: must be provided by parent
  title: string
  // Required prop with specific type
  count: number
}>()
// Props are now available as `props.title` and `props.count`
// They are reactive — template re-renders when parent updates them
</script>
<template>
  <!-- STEP 4: Use props in the template -->
  <!-- You can access props directly by name in the template -->
  <div class="card">
    <h2>{{ title }}</h2>
    <p>Count: {{ count }}</p>
  </div>
</template>
```
```vue
<!-- ParentComponent.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import ChildComponent from './ChildComponent.vue'
// Parent owns the data
const pageTitle = ref('Welcome')
const itemCount = ref(42)
</script>
<template>
  <!-- STEP 3: Pass props using v-bind (shorthand :) -->
  <!-- :title means v-bind:title — binds the JavaScript expression -->
  <ChildComponent 
    :title="pageTitle" 
    :count="itemCount" 
  />
  
  <!-- Static prop (no colon) — passes literal string "Hello" -->
  <ChildComponent 
    title="Hello" 
    :count="10" 
  />
</template>
```
### Props with Defaults (Optional Props)
```vue
<script setup lang="ts">
// For optional props with defaults, use withDefaults()
// This is the Vue 3.4+ recommended pattern
interface Props {
  // Required — no ?
  title: string
  // Optional — has ?
  subtitle?: string
  // Optional with complex default
  tags?: string[]
}
// withDefaults() wraps defineProps to provide default values
const props = withDefaults(defineProps<Props>(), {
  // Only optional props need defaults
  subtitle: 'No subtitle provided',
  // For arrays/objects, use a factory function
  tags: () => ['general']
})
</script>
<template>
  <article>
    <h1>{{ title }}</h1>
    <h2>{{ subtitle }}</h2>
    <span v-for="tag in tags" :key="tag">{{ tag }}</span>
  </article>
</template>
```
---
## 7. Comprehension Check
1. **Why does Vue enforce one-way data flow for props?**
   - Think about what would happen if any child could modify parent data directly.
2. **What's the difference between `:title="pageTitle"` and `title="pageTitle"`?**
   - One passes a JavaScript expression, the other passes a literal string.
3. **True or False: A child component can modify a prop value directly to update the parent's data.**
   - Consider the "newspaper subscription" mental model.
---
**Reply 'next' for Lesson 6.2 (Practice).**
