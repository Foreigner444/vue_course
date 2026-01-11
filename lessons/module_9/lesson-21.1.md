# Lesson 21.1: State Management II — Pinia Getters & Plugins — Theory & Concepts
# Управление состоянием II: Геттеры и Плагины Pinia — Теория и Концепции

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 21.1 of 25 (Part 1 of 2) |
| **Topic** | Pinia Getters & Plugins / Геттеры и Плагины Pinia |
| **Module** | Module 9: State Management |
| **Prerequisites** | Lesson 20 (Pinia Stores: State & Actions) |
| **You Should Know** | `computed()`, `watch()`, localStorage API, TypeScript generics |
| **Unlocks** | Lesson 21.2, then Module 10 (Advanced Patterns) |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 1 (Theory):**
1. **Remember:** Define what getters are and list common use cases for Pinia plugins
2. **Understand:** Explain how getters optimize derived state and how plugins extend store functionality

---

## 2. Real-World Scenario & Context

**Scenario:** You've built a shopping cart store with items and quantities. Now you need to:
1. Display the **total price** (sum of all items × quantities) in multiple places
2. Show a **formatted currency string** (e.g., "$45.99")
3. Check if the cart **qualifies for free shipping** (total > $50)
4. **Persist the cart** to localStorage so it survives page refreshes

Currently, you're recalculating the total in every component that needs it, and you've written the same localStorage save/load logic in multiple places.

**Framework Context:** Pinia **getters** are computed values derived from state — calculate once, use everywhere. Pinia **plugins** let you add functionality (like persistence) to all stores with zero repetition.

---

## 3. Core Concepts Explained (Deep Dive)

### Part A: Getters — Derived State

#### How Getters Actually Work

Getters are **computed properties for your store**. They:
1. Derive values from state (e.g., total from items)
2. **Cache** results — only recalculate when dependencies change
3. Are **reactive** — components using them auto-update

```
┌──────────────────────────────────────────────────────────────────┐
│                         PINIA STORE                              │
│                                                                  │
│  ┌─────────────────┐         ┌─────────────────┐                │
│  │     STATE       │         │     GETTERS     │                │
│  │                 │ ──────► │   (computed)    │                │
│  │  items: [...]   │         │                 │                │
│  │  taxRate: 0.08  │         │  totalPrice     │                │
│  │                 │         │  formattedTotal │                │
│  │                 │         │  itemCount      │                │
│  └─────────────────┘         └─────────────────┘                │
│         │                            │                           │
│         │                            │                           │
│         ▼                            ▼                           │
│  ┌─────────────────┐         ┌─────────────────┐                │
│  │    ACTIONS      │         │   COMPONENTS    │                │
│  │                 │         │                 │                │
│  │  Actions modify │         │  Read state     │                │
│  │  state, which   │         │  AND getters    │                │
│  │  triggers       │         │  (both reactive)│                │
│  │  getter updates │         │                 │                │
│  └─────────────────┘         └─────────────────┘                │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

#### Mental Model: Getters as "Smart Labels"

Imagine your store's state is a warehouse full of products:

- **State** = The actual products on shelves (items, quantities, prices)
- **Getters** = Smart labels that automatically update:
  - "Total Inventory Value: $12,450"
  - "Items Low in Stock: 3"
  - "Best Seller: Widget X"

When products (state) change, the labels (getters) automatically update. You don't manually recalculate — the labels are "live."

```
┌─────────────────────────────────────────────────────────────────┐
│                    🏭 WAREHOUSE (Store)                         │
│                                                                 │
│   ┌─────────┐  ┌─────────┐  ┌─────────┐                        │
│   │ Product │  │ Product │  │ Product │     STATE              │
│   │    A    │  │    B    │  │    C    │     (raw data)         │
│   │  $10×5  │  │  $20×3  │  │  $15×2  │                        │
│   └─────────┘  └─────────┘  └─────────┘                        │
│                                                                 │
│   ════════════════════════════════════════════════════════════ │
│                                                                 │
│   📊 SMART LABELS (Getters - auto-calculated)                  │
│                                                                 │
│   ┌────────────────┐  ┌────────────────┐  ┌────────────────┐   │
│   │ Total Value:   │  │ Item Count:    │  │ Avg Price:     │   │
│   │    $140.00     │  │      10        │  │    $14.00      │   │
│   │ (auto-updated) │  │ (auto-updated) │  │ (auto-updated) │   │
│   └────────────────┘  └────────────────┘  └────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### When to Use Getters vs. Inline Computation

| Use Getters When | Compute Inline When |
|:--- |:--- |
| Multiple components need the same derived value | Only one component needs it |
| The computation is expensive | The computation is trivial (e.g., `items.length`) |
| You want to test the logic independently | It's a simple template expression |
| The derived value is part of your "domain model" | It's purely presentational |

---

### Part B: Plugins — Store Superpowers

#### How Plugins Actually Work

A Pinia plugin is a function that runs when each store is created. It can:
1. **Add properties** to stores (e.g., `$router`)
2. **Add methods** to stores (e.g., `$reset()`)
3. **Wrap actions** (e.g., logging, error tracking)
4. **React to state changes** (e.g., sync to localStorage)

```
┌─────────────────────────────────────────────────────────────────┐
│                        PLUGIN LIFECYCLE                          │
│                                                                  │
│   createPinia()                                                  │
│        │                                                         │
│        ▼                                                         │
│   ┌─────────────────────────────────────┐                       │
│   │  pinia.use(myPlugin)                │  Register plugins     │
│   │  pinia.use(persistPlugin)           │                       │
│   └─────────────────────────────────────┘                       │
│        │                                                         │
│        ▼                                                         │
│   ┌─────────────────────────────────────┐                       │
│   │  useCartStore() is called           │  First store access   │
│   └─────────────────────────────────────┘                       │
│        │                                                         │
│        ▼                                                         │
│   ┌─────────────────────────────────────┐                       │
│   │  FOR EACH PLUGIN:                   │                       │
│   │    plugin({ store, pinia, app })    │  Plugins run in order │
│   │                                     │                       │
│   │  - Can read store.$id               │                       │
│   │  - Can add properties to store      │                       │
│   │  - Can subscribe to state changes   │                       │
│   └─────────────────────────────────────┘                       │
│        │                                                         │
│        ▼                                                         │
│   ┌─────────────────────────────────────┐                       │
│   │  Store is ready to use              │                       │
│   └─────────────────────────────────────┘                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Mental Model: Plugins as "Store Upgrades"

Think of plugins like **browser extensions** for your stores:

- **Base store** = A basic web browser (Chrome, Firefox)
- **Persistence plugin** = An extension that auto-saves your tabs
- **Logging plugin** = DevTools extension that tracks all activity
- **Undo plugin** = An extension that adds undo/redo to every page

You install extensions once, and they enhance ALL browser windows (stores).

```
┌─────────────────────────────────────────────────────────────────┐
│                    🌐 BROWSER (Pinia)                            │
│                                                                  │
│   Installed Extensions:                                          │
│   ┌─────────────┐ ┌─────────────┐ ┌─────────────┐               │
│   │ 💾 AutoSave │ │ 📋 Logger   │ │ ↩️ Undo     │               │
│   │   Plugin    │ │   Plugin    │ │   Plugin    │               │
│   └─────────────┘ └─────────────┘ └─────────────┘               │
│                                                                  │
│   ════════════════════════════════════════════════════════════  │
│                                                                  │
│   All tabs (stores) get ALL extensions automatically:            │
│                                                                  │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐                  │
│   │ Cart Tab │    │ User Tab │    │ UI Tab   │                  │
│   │ 💾 📋 ↩️  │    │ 💾 📋 ↩️  │    │ 💾 📋 ↩️  │                  │
│   └──────────┘    └──────────┘    └──────────┘                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Common Plugin Use Cases

| Plugin Type | What It Does | Example |
|:--- |:--- |:--- |
| **Persistence** | Saves state to localStorage/sessionStorage | `pinia-plugin-persistedstate` |
| **Logging** | Logs all actions and state changes | Custom debug plugin |
| **Undo/Redo** | Tracks state history for time-travel | `pinia-undo` |
| **Sync** | Syncs state across browser tabs | Broadcast Channel plugin |
| **Encryption** | Encrypts sensitive data before storage | Custom security plugin |

---

### In Other Contexts

| Concept | Vue/Pinia | React | Angular |
|:--- |:--- |:--- |:--- |
| **Getters** | `computed()` in store | `useSelector` with memoization (Redux) | Selectors in NgRx |
| **Plugins** | `pinia.use(plugin)` | Redux middleware | NgRx meta-reducers |
| **Persistence** | `pinia-plugin-persistedstate` | `redux-persist` | Custom service |

---

## 4. New Terminology

| Term | Definition |
|:--- |:--- |
| **Getter** | A computed value derived from store state; cached and only recalculates when dependencies change |
| **Plugin** | A function that runs when stores are created, used to add features to all stores |
| **Persistence** | Saving state to a storage medium (localStorage, IndexedDB) so it survives page refreshes |
| **Subscription** | A callback that runs whenever store state changes, used by plugins to react to updates |

---

## 5. Algorithmic Thinking (Planning the Solution)

**The Plan:** Adding getters and persistence to our cart store

```
STEP 1: Identify Derived Values
─────────────────────────────────
What values can be computed from state?

  items: [{ price: 10, qty: 2 }, { price: 20, qty: 1 }]
                           │
                           ▼
  ┌─────────────────────────────────────────────────┐
  │ GETTERS TO CREATE:                              │
  │                                                 │
  │ • totalItems → sum of all quantities (3)        │
  │ • subtotal → sum of (price × qty) ($40)         │
  │ • tax → subtotal × taxRate ($3.20)              │
  │ • total → subtotal + tax ($43.20)               │
  │ • formattedTotal → "$43.20"                     │
  │ • freeShipping → total > 50                     │
  └─────────────────────────────────────────────────┘

STEP 2: Define Getters in Store
─────────────────────────────────
  const totalItems = computed(() => 
    items.value.reduce((sum, item) => sum + item.qty, 0)
  )

STEP 3: Add Persistence Plugin
─────────────────────────────────
  
  ┌─────────────────────────────────────────────────┐
  │           PERSISTENCE FLOW                       │
  │                                                  │
  │   Store Created                                  │
  │        │                                         │
  │        ▼                                         │
  │   Check localStorage for 'cart' key              │
  │        │                                         │
  │        ├── Found? ──► Hydrate store with data    │
  │        │                                         │
  │        └── Not found? ──► Use default state      │
  │                                                  │
  │   ═══════════════════════════════════════════   │
  │                                                  │
  │   State Changes (via actions)                    │
  │        │                                         │
  │        ▼                                         │
  │   Plugin subscription fires                      │
  │        │                                         │
  │        ▼                                         │
  │   Save state to localStorage                     │
  │                                                  │
  └─────────────────────────────────────────────────┘

STEP 4: Register Plugin with Pinia
─────────────────────────────────
  // main.ts
  const pinia = createPinia()
  pinia.use(persistPlugin)
  app.use(pinia)
```

---

## 6. Initial Pattern Introduction

### Getters with `computed()`

In Setup Stores, getters are just `computed()` properties that you return:

| What You Want (Intent) | Code Chunk (The Pattern) | Conceptual Link |
|:--- |:--- |:--- |
| Create a derived value | `const total = computed(() => ...)` | Same `computed()` from components |
| Access other getters | Reference them directly in the function | Getters can depend on other getters |
| Expose the getter | Include in the return statement | Makes it available to components |

```typescript
// src/stores/cart.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

interface CartItem {
  id: number
  name: string
  price: number
  quantity: number
}

export const useCartStore = defineStore('cart', () => {
  // ============ STATE ============
  const items = ref<CartItem[]>([])
  const taxRate = ref(0.08) // 8% tax

  // ============ GETTERS ============
  // Getter: Total number of items in cart
  const totalItems = computed(() => {
    return items.value.reduce((sum, item) => sum + item.quantity, 0)
  })

  // Getter: Subtotal before tax
  const subtotal = computed(() => {
    return items.value.reduce(
      (sum, item) => sum + item.price * item.quantity,
      0
    )
  })

  // Getter: Tax amount (uses another getter!)
  const tax = computed(() => {
    return subtotal.value * taxRate.value
  })

  // Getter: Final total (uses other getters)
  const total = computed(() => {
    return subtotal.value + tax.value
  })

  // Getter: Formatted currency string
  const formattedTotal = computed(() => {
    return new Intl.NumberFormat('en-US', {
      style: 'currency',
      currency: 'USD'
    }).format(total.value)
  })

  // Getter: Boolean check
  const qualifiesForFreeShipping = computed(() => {
    return subtotal.value >= 50
  })

  // ============ ACTIONS ============
  function addItem(item: Omit<CartItem, 'quantity'>) {
    const existing = items.value.find(i => i.id === item.id)
    if (existing) {
      existing.quantity++
    } else {
      items.value.push({ ...item, quantity: 1 })
    }
  }

  // ============ RETURN ============
  return {
    // State
    items,
    taxRate,
    // Getters
    totalItems,
    subtotal,
    tax,
    total,
    formattedTotal,
    qualifiesForFreeShipping,
    // Actions
    addItem
  }
})
```

### Basic Plugin Structure

```typescript
// src/plugins/logger.ts
import type { PiniaPluginContext } from 'pinia'

// A plugin is a function that receives context about the store
export function loggerPlugin(context: PiniaPluginContext) {
  // context.store - the store being created
  // context.pinia - the pinia instance
  // context.app - the Vue app instance
  // context.options - options passed to defineStore

  const storeName = context.store.$id

  // Subscribe to all state changes
  context.store.$subscribe((mutation, state) => {
    console.log(`[${storeName}] State changed:`, mutation.type)
    console.log('New state:', JSON.stringify(state, null, 2))
  })

  // Subscribe to all actions
  context.store.$onAction(({ name, args, after, onError }) => {
    console.log(`[${storeName}] Action "${name}" called with:`, args)

    after((result) => {
      console.log(`[${storeName}] Action "${name}" completed with:`, result)
    })

    onError((error) => {
      console.error(`[${storeName}] Action "${name}" failed:`, error)
    })
  })
}

// Register in main.ts:
// const pinia = createPinia()
// pinia.use(loggerPlugin)
```

### Persistence Plugin Pattern

```typescript
// src/plugins/persist.ts
import type { PiniaPluginContext } from 'pinia'

export function persistPlugin(context: PiniaPluginContext) {
  const storeName = context.store.$id
  const storageKey = `pinia-${storeName}`

  // HYDRATE: Load saved state when store is created
  const savedState = localStorage.getItem(storageKey)
  if (savedState) {
    // $patch updates the store's state
    context.store.$patch(JSON.parse(savedState))
  }

  // PERSIST: Save state whenever it changes
  context.store.$subscribe((mutation, state) => {
    localStorage.setItem(storageKey, JSON.stringify(state))
  })
}
```

---

## 7. Comprehension Check

1. **Conceptual:** What's the difference between recalculating a value in every component vs. using a getter? Why does the difference matter?

2. **Conceptual:** When does a Pinia plugin's function execute — when you call `pinia.use()`, when the app mounts, or when a store is first accessed?

3. **True/False:** Getters in Pinia can access and depend on other getters defined in the same store.

---

**Reply 'next' for Lesson 21.2 (Practice).**
