# Lesson 20.1: State Management I — Pinia Stores (State & Actions) — Theory & Concepts
# Управление состоянием I: Хранилища Pinia (Состояние и Действия) — Теория и Концепции

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 20.1 of 25 (Part 1 of 2) |
| **Topic** | Pinia Stores (State & Actions) / Хранилища Pinia (Состояние и Действия) |
| **Module** | Module 9: State Management |
| **Prerequisites** | Lessons 15 (Composables), 18-19 (Vue Router) |
| **You Should Know** | TypeScript interfaces, `ref()` and `reactive()`, composables, async/await |
| **Unlocks** | Lesson 20.2, then Lesson 21 (Getters & Plugins) |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 1 (Theory):**
1. **Remember:** Define what a Pinia store is and identify its three core parts (state, getters, actions)
2. **Understand:** Explain why centralized state management solves prop drilling and component coupling problems

---

## 2. Real-World Scenario & Context

**Scenario:** You're building an e-commerce application with a shopping cart. The cart needs to be accessible from multiple pages: the product listing, the product detail page, the header (showing item count), and the checkout page. Currently, you're passing cart data through props across 4+ levels of components, and every component in between needs to forward the props even if it doesn't use them. When a user adds an item from a deeply nested component, you need to emit events up through multiple parent components to update the cart.

**Framework Context:** Pinia is Vue's official state management library that provides a centralized "single source of truth" for data that multiple components need to access. Instead of prop drilling or event chains, any component can directly read from and write to a shared store.

---

## 3. Core Concepts Explained (Deep Dive)

### How It Actually Works

Pinia stores are reactive containers that live outside of your component tree. When you define a store, Pinia creates a singleton instance that:

1. **Holds reactive state** — Data wrapped in Vue's reactivity system
2. **Provides actions** — Methods to modify that state
3. **Exposes getters** — Computed values derived from state (covered in Lesson 21)

```
┌─────────────────────────────────────────────────────────────────┐
│                        PINIA STORE                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │   STATE     │  │   GETTERS   │  │   ACTIONS   │              │
│  │  (reactive  │  │  (computed  │  │  (methods   │              │
│  │   data)     │  │   values)   │  │   to modify │              │
│  │             │  │             │  │   state)    │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
└─────────────────────────────────────────────────────────────────┘
          ▲                 │                 │
          │                 │                 │
          │    ┌────────────┴─────────────────┘
          │    │
          │    ▼
┌─────────┴────────────────────────────────────────────────────────┐
│                      COMPONENT TREE                               │
│                                                                   │
│    ┌─────────┐      ┌─────────┐      ┌─────────┐                │
│    │ Header  │      │ Product │      │ Checkout│                │
│    │  Cart   │      │  Page   │      │  Page   │                │
│    │ Counter │      │         │      │         │                │
│    └─────────┘      └─────────┘      └─────────┘                │
│         │                │                │                      │
│         └────────────────┴────────────────┘                      │
│                          │                                       │
│              All components directly access                      │
│              the same store instance                             │
└─────────────────────────────────────────────────────────────────┘
```

When any component calls an action that modifies state, all components using that state automatically re-render — no manual event emission or prop updates needed.

### Mental Model

Think of a Pinia store as a **shared whiteboard in an office**:

- **State** = The information written on the whiteboard (e.g., "Cart items: 3")
- **Actions** = The markers and erasers anyone can use to update the whiteboard
- **Getters** = Calculations based on what's written (e.g., "Total items × Price = $45")

Any team member (component) can walk up to the whiteboard and read it. Any team member can use the markers (actions) to update it. When someone changes the whiteboard, everyone looking at it sees the change immediately.

```
┌──────────────────────────────────────────────────────┐
│              🏢 OFFICE WHITEBOARD                    │
│                  (Pinia Store)                       │
│                                                      │
│   ┌────────────────────────────────────────────┐    │
│   │  CART ITEMS: 🛒 🛒 🛒                       │    │  ← STATE
│   │  TOTAL: $45.00                              │    │  ← GETTER
│   └────────────────────────────────────────────┘    │
│                                                      │
│   [Add Item ✏️]  [Remove Item 🧽]  [Clear All 🗑️]   │  ← ACTIONS
│                                                      │
└──────────────────────────────────────────────────────┘
         ▲              ▲              ▲
         │              │              │
     ┌───┴───┐     ┌────┴───┐    ┌────┴────┐
     │Header │     │Product │    │Checkout │
     │  👀   │     │Page 👀 │    │ Page 👀 │
     └───────┘     └────────┘    └─────────┘
     
     All team members can see and update the whiteboard
```

### In Other Contexts

| Context | Equivalent |
|:--- |:--- |
| **React** | Redux, Zustand, or React Context + useReducer |
| **Angular** | Services with RxJS BehaviorSubject |
| **Vanilla JS** | A global object with event emitters |
| **Vue 2** | Vuex (Pinia's predecessor) |

**Key difference:** Pinia is simpler than Vuex — no mutations, no modules boilerplate, full TypeScript support out of the box.

### When to Use / When NOT to Use

| ✅ Use Pinia When | ❌ Don't Use Pinia When |
|:--- |:--- |
| Data is needed by multiple unrelated components | Data is only used by a single component |
| You have deeply nested components sharing state | Parent-child communication (use props/emits) |
| State needs to persist across route changes | Temporary UI state (modal open/closed) |
| You need to share state between routes | Form data that resets on navigation |
| Complex state logic with multiple actions | Simple derived values (use `computed()`) |

---

## 4. New Terminology

| Term | Definition |
|:--- |:--- |
| **Store** | A reactive container holding state, getters, and actions that can be accessed from any component |
| **State** | The reactive data properties defined in a store — the "single source of truth" |
| **Action** | A method defined in a store that can modify state, perform async operations, or call other actions |
| **Prop Drilling** | The anti-pattern of passing props through many intermediate components that don't use them, just to reach a deeply nested child |

---

## 5. Algorithmic Thinking (Planning the Solution)

**The Plan:** Converting our shopping cart from prop drilling to Pinia

```
BEFORE (Prop Drilling):
─────────────────────────
App.vue (owns cart state)
    │
    ├── passes cart as prop ──► Header.vue
    │                               │
    │                               └── passes cart as prop ──► CartCounter.vue
    │
    └── passes cart as prop ──► ProductPage.vue
                                    │
                                    └── passes cart as prop ──► AddToCartButton.vue
                                              │
                                              └── emits "add" event up through ALL parents

AFTER (Pinia):
──────────────
┌─────────────────┐
│  useCartStore   │◄───── All components import directly
└─────────────────┘
        ▲
        │
   ┌────┴─────┬──────────────┬─────────────────┐
   │          │              │                 │
Header    CartCounter   ProductPage    AddToCartButton
   │          │              │                 │
   └──────────┴──────────────┴─────────────────┘
   
   Each component reads/writes directly to the store
```

**Step-by-Step Thinking:**

1. **Identify shared state** — What data do multiple components need? (cart items, quantities)
2. **Define the store** — Create a file with `defineStore()` containing state and actions
3. **Remove prop chains** — Delete all cart-related props from intermediate components
4. **Import store in consumers** — Each component that needs cart data imports `useCartStore()`
5. **Call actions directly** — Components call `store.addItem()` instead of emitting events

---

## 6. Initial Pattern Introduction

### The Two Store Syntaxes

Pinia offers two ways to define stores. We'll focus on the **Setup Store** syntax because it matches what you learned with `<script setup>` and composables.

| What You Want (Intent) | Code Chunk (The Pattern) | Conceptual Link |
|:--- |:--- |:--- |
| Create a store | `defineStore('id', () => { ... })` | Like creating a composable that Pinia manages |
| Define state | `const items = ref<Type[]>([])` | Same `ref()` you already know |
| Define an action | `function addItem(item: Type) { ... }` | Regular function that modifies refs |
| Expose store API | `return { items, addItem }` | Same as composable return pattern |
| Use store in component | `const store = useCartStore()` | Call the function Pinia created |

### Basic Store Definition

```typescript
// src/stores/cart.ts
import { defineStore } from 'pinia'
import { ref } from 'vue'

// Interface defining the shape of a cart item
interface CartItem {
  id: number
  name: string
  price: number
  quantity: number
}

// defineStore takes two arguments:
// 1. A unique string ID for this store ('cart')
// 2. A setup function (like <script setup>) that returns state and actions
export const useCartStore = defineStore('cart', () => {
  // STATE: Reactive data using ref() — just like in components!
  const items = ref<CartItem[]>([])

  // ACTION: A function that modifies state
  function addItem(product: Omit<CartItem, 'quantity'>) {
    // Check if item already exists in cart
    const existingItem = items.value.find(item => item.id === product.id)
    
    if (existingItem) {
      // If exists, increment quantity
      existingItem.quantity++
    } else {
      // If new, add with quantity 1
      items.value.push({ ...product, quantity: 1 })
    }
  }

  // ACTION: Remove an item from cart
  function removeItem(productId: number) {
    const index = items.value.findIndex(item => item.id === productId)
    if (index > -1) {
      items.value.splice(index, 1)
    }
  }

  // ACTION: Clear all items
  function clearCart() {
    items.value = []
  }

  // Return everything that should be accessible from components
  return {
    items,      // State (readable and reactive)
    addItem,    // Action
    removeItem, // Action
    clearCart   // Action
  }
})
```

### Using the Store in a Component

```vue
<script setup lang="ts">
// Import the store hook (not the store itself!)
import { useCartStore } from '@/stores/cart'

// Call the hook to get the store instance
// This gives you access to all state and actions returned by the store
const cartStore = useCartStore()

// Example product to add
const sampleProduct = {
  id: 1,
  name: 'Vue.js T-Shirt',
  price: 29.99
}
</script>

<template>
  <div class="p-4">
    <!-- Access state directly from the store -->
    <p class="text-lg">Items in cart: {{ cartStore.items.length }}</p>
    
    <!-- List all items -->
    <ul class="my-4">
      <li v-for="item in cartStore.items" :key="item.id" class="flex justify-between">
        <span>{{ item.name }} (x{{ item.quantity }})</span>
        <span>${{ (item.price * item.quantity).toFixed(2) }}</span>
      </li>
    </ul>
    
    <!-- Call actions as methods on the store -->
    <button 
      @click="cartStore.addItem(sampleProduct)"
      class="btn btn-primary mr-2"
    >
      Add Sample Product
    </button>
    
    <button 
      @click="cartStore.clearCart()"
      class="btn btn-error"
    >
      Clear Cart
    </button>
  </div>
</template>
```

### Setting Up Pinia in Your App

```typescript
// src/main.ts
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

const app = createApp(App)

// Create and install Pinia
const pinia = createPinia()
app.use(pinia)

app.mount('#app')
```

---

## 7. Comprehension Check

1. **Conceptual:** What problem does Pinia solve that `ref()` and `reactive()` alone cannot?

2. **Conceptual:** In the Setup Store syntax, why do we use `ref()` for state instead of just declaring a regular variable?

3. **True/False:** When you call an action in one component that modifies store state, other components using that state must manually refresh to see the change.

---

**Reply 'next' for Lesson 20.2 (Practice).**
