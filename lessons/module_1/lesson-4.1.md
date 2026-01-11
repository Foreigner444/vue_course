# Lesson 4.1: Derived State with `computed()` — Theory & Concepts (Вычисляемые свойства с computed())

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 4.1 of 25 (Part 1 of 2) |
| **Topic** | Derived State with `computed()` / Вычисляемые свойства с computed() |
| **Module** | Module 1: Core Fundamentals |
| **Prerequisites** | Lessons 1, 2, 3 (Components, ref(), reactive()) |
| **You Should Know** | ref() and reactive() usage, TypeScript generics, getter/setter concept |
| **Unlocks** | Lesson 4.2, then Lesson 5 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 1 (Theory):**
1. **Remember:** Define `computed()` and explain its purpose for derived state
2. **Understand:** Explain how computed properties cache their results and when they recalculate

---

## 2. Real-World Scenario & Context

**Scenario:** You're building an e-commerce shopping cart. For each item, you need to display the subtotal (price × quantity). For the entire cart, you need to show the total price, total items count, and whether the user qualifies for free shipping (orders over $50). These values all **derive from** the cart items — they shouldn't be stored separately because they would get out of sync.

**Framework Context:** Vue provides `computed()` to create derived values that automatically update when their dependencies change. Unlike regular functions, computed properties are **cached** — they only recalculate when one of their dependencies actually changes, making them efficient for expensive calculations.

---

## 3. Core Concepts Explained (Deep Dive)

### How It Actually Works

A computed property is a reactive value that automatically tracks its dependencies:

```
┌─────────────────────────────────────────────────────────────────┐
│                  HOW computed() WORKS                           │
│                                                                 │
│   Source State:                     Computed Value:             │
│   ┌─────────────────┐               ┌─────────────────┐         │
│   │ price = ref(10) │──────────────▶│                 │         │
│   └─────────────────┘               │  total = computed(() =>   │
│   ┌─────────────────┐               │    price.value *          │
│   │ qty = ref(3)    │──────────────▶│    qty.value              │
│   └─────────────────┘               │  )                        │
│                                     │                 │         │
│                                     │  total.value = 30         │
│                                     └─────────────────┘         │
│                                                                 │
│   When price or qty changes → total automatically recalculates  │
│   When nothing changes → total returns cached value (no work!)  │
└─────────────────────────────────────────────────────────────────┘
```

**The Caching Mechanism:**

```
┌─────────────────────────────────────────────────────────────────┐
│                    COMPUTED CACHING                             │
│                                                                 │
│   const expensiveCalc = computed(() => {                        │
│     console.log('Computing...')  // Only runs when needed       │
│     return items.value.reduce((sum, i) => sum + i.price, 0)     │
│   })                                                            │
│                                                                 │
│   Timeline:                                                     │
│   ────────────────────────────────────────────────────────────  │
│                                                                 │
│   1. First access: expensiveCalc.value                          │
│      └─▶ "Computing..." (calculates & caches result)            │
│                                                                 │
│   2. Second access: expensiveCalc.value                         │
│      └─▶ Returns cached value (no computation!)                 │
│                                                                 │
│   3. Third access: expensiveCalc.value                          │
│      └─▶ Returns cached value (still cached!)                   │
│                                                                 │
│   4. items.value changes (dependency update)                    │
│      └─▶ Cache invalidated                                      │
│                                                                 │
│   5. Next access: expensiveCalc.value                           │
│      └─▶ "Computing..." (recalculates & caches new result)      │
└─────────────────────────────────────────────────────────────────┘
```

### Mental Model

Think of `computed()` as a **"smart formula cell" in a spreadsheet**:

```
┌─────────────────────────────────────────────────────────────────┐
│               SPREADSHEET ANALOGY                               │
│                                                                 │
│   ┌───────┬───────┬───────┬───────────────────────┐             │
│   │   A   │   B   │   C   │          D            │             │
│   ├───────┼───────┼───────┼───────────────────────┤             │
│   │ Price │  Qty  │ Total │        Formula        │             │
│   ├───────┼───────┼───────┼───────────────────────┤             │
│   │  $10  │   3   │  $30  │  =A2*B2 (computed!)   │             │
│   ├───────┼───────┼───────┼───────────────────────┤             │
│   │  $25  │   2   │  $50  │  =A3*B3 (computed!)   │             │
│   └───────┴───────┴───────┴───────────────────────┘             │
│                                                                 │
│   • You enter values in A and B (ref/reactive)                  │
│   • C automatically calculates (computed)                       │
│   • Change A2 to $15 → C2 instantly becomes $45                 │
│   • No manual recalculation needed!                             │
│                                                                 │
│   Vue's computed() works the same way:                          │
│   const total = computed(() => price.value * qty.value)         │
└─────────────────────────────────────────────────────────────────┘
```

### Computed vs Regular Function

```typescript
// ❌ Regular function: recalculates EVERY time it's called
const getTotal = (): number => {
  console.log('Calculating...')  // Runs on EVERY call
  return price.value * qty.value
}

// In template: {{ getTotal() }} — calls function every render!

// ✅ Computed: recalculates only when dependencies change
const total = computed(() => {
  console.log('Calculating...')  // Runs only when price/qty change
  return price.value * qty.value
})

// In template: {{ total }} — uses cached value if unchanged!
```

**Performance Comparison:**

```
┌─────────────────────────────────────────────────────────────────┐
│              Function vs Computed Performance                    │
│                                                                 │
│   Scenario: Template re-renders 100 times, data unchanged       │
│                                                                 │
│   getTotal()  →  100 calculations  →  Slow! 🐢                  │
│   computed    →    1 calculation   →  Fast! 🚀                  │
│                                                                 │
│   Scenario: Data changes once, template renders 100 times       │
│                                                                 │
│   getTotal()  →  100 calculations  →  Slow! 🐢                  │
│   computed    →    1 calculation   →  Fast! 🚀                  │
└─────────────────────────────────────────────────────────────────┘
```

### In Other Contexts

| Framework | Equivalent to Vue's `computed()` |
|:--- |:--- |
| **Vue 3** | `computed()` |
| **React** | `useMemo()` hook |
| **Angular** | Pipes or getters |
| **MobX** | `@computed` decorator |
| **Svelte** | `$:` reactive declarations |
| **Solid.js** | `createMemo()` |

```typescript
// Vue 3
const doubled = computed(() => count.value * 2)

// React (for comparison)
const doubled = useMemo(() => count * 2, [count])
// Note: React requires manual dependency array!

// Vue tracks dependencies automatically — no array needed!
```

### When to Use / When NOT to Use

| ✅ Use `computed()` When | ❌ Don't Use `computed()` When |
|:--- |:--- |
| Value derives from other reactive state | Value doesn't depend on reactive state |
| Same derived value needed in multiple places | One-time calculation (just use const) |
| Expensive calculations that benefit from caching | Simple calculations (overhead might not be worth it) |
| Filtering/sorting lists | Async operations (use watch + ref instead) |
| Formatting data for display | Side effects needed (use watch) |

---

## 4. New Terminology

| Term | Definition |
|:--- |:--- |
| **`computed()`** | Vue function that creates a cached, reactive derived value |
| **Derived State** | Values that are calculated from other state (not stored directly) |
| **Caching** | Storing the result of a calculation to avoid redundant work |
| **Dependency Tracking** | Vue automatically detecting which reactive values a computed uses |
| **Writable Computed** | A computed property with both getter and setter (two-way derived state) |

---

## 5. Algorithmic Thinking (Planning the Solution)

**The Plan (Narrative):** To build our shopping cart with derived values:

```
┌─────────────────────────────────────────────────────────────────┐
│ STEP 1: Identify source state (what the user directly modifies) │
│   └──▶ items: Array<{ name, price, quantity }>                  │
│        These are stored in ref() or reactive()                  │
│                                                                 │
│ STEP 2: Identify derived state (calculated from source)         │
│   └──▶ totalItems: sum of all quantities                        │
│        subtotal: sum of (price × quantity) for each item        │
│        tax: subtotal × taxRate                                  │
│        total: subtotal + tax                                    │
│        freeShipping: subtotal > 50                              │
│                                                                 │
│ STEP 3: Create computed properties for each derived value       │
│   └──▶ const totalItems = computed(() =>                        │
│          items.value.reduce((sum, i) => sum + i.quantity, 0)    │
│        )                                                        │
│                                                                 │
│ STEP 4: Use in template without ()                              │
│   └──▶ {{ totalItems }} not {{ totalItems() }}                  │
│                                                                 │
│ STEP 5: Enjoy automatic updates!                                │
│   └──▶ When items change, all computed values update            │
└─────────────────────────────────────────────────────────────────┘
```

**Decision Tree: Function vs Computed?**

```
Does the value derive from reactive state?
│
├── NO ──▶ Regular function or constant
│
└── YES ──▶ Is it used multiple times in template/component?
            │
            ├── NO ──▶ Function might be fine
            │
            └── YES ──▶ Is the calculation expensive?
                        │
                        ├── YES ──▶ Definitely use computed() ✅
                        │
                        └── NO ──▶ Use computed() anyway (best practice) ✅
```

---

## 6. Initial Pattern Introduction

| What You Want (Intent) | Code Chunk (The Pattern) | Conceptual Link |
|:--- |:--- |:--- |
| Create read-only computed | `computed(() => expression)` | Cached getter |
| Create writable computed | `computed({ get(), set() })` | Two-way derived |
| Access computed value | `myComputed.value` (script) or `{{ myComputed }}` (template) | Like ref |
| Type a computed | `computed<Type>(() => ...)` | TypeScript inference |

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

// ============================================
// SOURCE STATE (what can be directly modified)
// ============================================
const firstName = ref('John')
const lastName = ref('Doe')
const price = ref(29.99)
const quantity = ref(2)
const items = ref([
  { name: 'Apple', price: 1.5, qty: 3 },
  { name: 'Banana', price: 0.75, qty: 5 }
])

// ============================================
// COMPUTED PROPERTIES (derived from source)
// ============================================

// Simple computed: combines two refs
const fullName = computed(() => {
  return `${firstName.value} ${lastName.value}`
})

// Computed with calculation
const subtotal = computed(() => {
  return price.value * quantity.value
})

// Computed with array method
const totalItems = computed(() => {
  return items.value.reduce((sum, item) => sum + item.qty, 0)
})

// Computed with formatting
const formattedPrice = computed(() => {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD'
  }).format(subtotal.value)
})

// Computed boolean (condition)
const freeShipping = computed(() => {
  return subtotal.value >= 50
})

// ============================================
// WRITABLE COMPUTED (get + set)
// ============================================

// fullName can now be set, and it updates firstName/lastName
const fullNameWritable = computed({
  get() {
    return `${firstName.value} ${lastName.value}`
  },
  set(newValue: string) {
    const parts = newValue.split(' ')
    firstName.value = parts[0] || ''
    lastName.value = parts.slice(1).join(' ') || ''
  }
})

// ============================================
// USAGE NOTE
// ============================================
// In script: use .value (like ref)
console.log(fullName.value)  // "John Doe"

// In template: auto-unwrapped (no .value)
// {{ fullName }} displays "John Doe"
</script>

<template>
  <div class="p-6 space-y-4">
    <!-- Source inputs -->
    <div>
      <input v-model="firstName" placeholder="First name" class="border p-2 mr-2" />
      <input v-model="lastName" placeholder="Last name" class="border p-2" />
    </div>
    
    <!-- Computed display (read-only) -->
    <p>Full Name: <strong>{{ fullName }}</strong></p>
    
    <!-- Writable computed -->
    <div>
      <input v-model="fullNameWritable" placeholder="Full name" class="border p-2" />
      <p class="text-sm text-gray-500">Edit full name directly!</p>
    </div>
    
    <!-- Shopping example -->
    <div class="border-t pt-4">
      <p>Price: ${{ price }} × {{ quantity }} = {{ formattedPrice }}</p>
      <p v-if="freeShipping" class="text-green-600">✓ Free shipping!</p>
      <p v-else class="text-gray-500">Add ${{ (50 - subtotal).toFixed(2) }} for free shipping</p>
    </div>
    
    <!-- Array computed -->
    <p>Total items in cart: {{ totalItems }}</p>
  </div>
</template>
```

**Key Points:**
- Import `computed` from Vue
- Computed returns a `ComputedRef` — use `.value` in script
- Template auto-unwraps — no `.value` needed
- Dependencies are automatically tracked
- Writable computed uses `{ get(), set() }` syntax

---

## 7. Comprehension Check

1. **Why is `computed()` more efficient than a regular function for derived values?**
   - Think about what happens when the template re-renders.

2. **What triggers a computed property to recalculate its value?**
   - Consider how Vue knows when to invalidate the cache.

3. **True or False: You must manually specify the dependencies of a computed property like in React's `useMemo`.**
   - Think about Vue's reactivity system.

---

*Reply 'next' for Lesson 4.2 (Practice).*
