# Lesson 2.1: Reactive Primitives with `ref()` — Theory & Concepts (Реактивные примитивы с ref())

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 2.1 of 25 (Part 1 of 2) |
| **Topic** | Reactive Primitives with `ref()` / Реактивные примитивы с ref() |
| **Module** | Module 1: Core Fundamentals |
| **Prerequisites** | Lesson 1 (Your First Component) |
| **You Should Know** | JavaScript primitives (string, number, boolean), variable declaration (const/let), template syntax |
| **Unlocks** | Lesson 2.2, then Lessons 3, 4, 5 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 1 (Theory):**
1. **Remember:** Define what reactivity means in Vue and identify when to use `ref()`
2. **Understand:** Explain how Vue's reactivity system tracks and updates the DOM automatically

---

## 2. Real-World Scenario & Context

**Scenario:** You're building a fitness tracking app. The home screen shows a step counter that updates as the user walks. Every time the step count increases, the display should update immediately — you shouldn't have to manually tell the browser to redraw the number. The user should also be able to set a daily goal and see a progress bar that fills up in real-time.

**Framework Context:** Vue solves the problem of keeping the UI in sync with data through its **reactivity system**. When you use `ref()`, Vue automatically tracks the value and re-renders any part of the template that depends on it — no manual DOM manipulation required.

---

## 3. Core Concepts Explained (Deep Dive)

### How It Actually Works

When you create a `ref()`, Vue wraps your value in a special reactive object:

```
┌─────────────────────────────────────────────────────────────────┐
│                    WHAT ref() CREATES                           │
│                                                                 │
│   const count = ref(0)                                          │
│                                                                 │
│   Creates this object internally:                               │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  {                                                      │   │
│   │    __v_isRef: true,     // Marker that this is a ref    │   │
│   │    _value: 0,           // The actual stored value      │   │
│   │    get value() {...},   // Getter with tracking         │   │
│   │    set value() {...}    // Setter with triggering       │   │
│   │  }                                                      │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   Access in script:  count.value  → 0                           │
│   Access in template: {{ count }} → 0 (auto-unwrapped!)         │
└─────────────────────────────────────────────────────────────────┘
```

**The Reactivity Flow:**

```
┌──────────────┐     ┌──────────────────┐     ┌──────────────┐
│  You change  │     │  Vue detects the │     │  Vue updates │
│  ref.value   │ ──▶ │  change via the  │ ──▶ │  the DOM     │
│              │     │  setter trap     │     │  automatically│
└──────────────┘     └──────────────────┘     └──────────────┘
        │                                             │
        │         ┌──────────────────┐                │
        └────────▶│  Template reads  │◀───────────────┘
                  │  the ref, Vue    │
                  │  tracks this     │
                  │  dependency      │
                  └──────────────────┘
```

**Step-by-step breakdown:**

1. **Creation:** `ref(0)` creates a reactive wrapper around `0`
2. **Reading:** When the template accesses `{{ count }}`, Vue records that this component depends on `count`
3. **Writing:** When you do `count.value = 5`, Vue's setter detects the change
4. **Re-rendering:** Vue schedules a DOM update for all components that depend on `count`
5. **Efficiency:** Vue batches multiple changes together and only updates what changed

### Mental Model

Think of `ref()` as a **"smart container"** with a notification system:

```
┌─────────────────────────────────────────────────────────────────┐
│                    ref() = SMART CONTAINER                       │
│                                                                 │
│   ┌─────────────────────────────┐                               │
│   │         📦 ref()            │                               │
│   │   ┌─────────────────────┐   │                               │
│   │   │                     │   │    Subscribers (watchers):    │
│   │   │    value: 42        │───┼──▶ • Template: {{ count }}    │
│   │   │                     │   │    • watch() callbacks        │
│   │   └─────────────────────┘   │    • computed() dependencies  │
│   │                             │                               │
│   │   🔔 "Hey! My value just   │                               │
│   │       changed from 42 to   │                               │
│   │       43! Update yourselves│                               │
│   │       if you care!"        │                               │
│   └─────────────────────────────┘                               │
│                                                                 │
│   Normal variable: const x = 42                                 │
│   → Just a value, no notifications, no tracking                 │
│                                                                 │
│   Reactive ref: const x = ref(42)                               │
│   → Wrapped value with change detection and notifications       │
└─────────────────────────────────────────────────────────────────┘
```

### Why `.value`?

JavaScript primitives (string, number, boolean) are **passed by value**, not by reference. You can't track changes to a raw primitive:

```typescript
// This CANNOT be tracked
let count = 0
count = 1  // JavaScript has no way to detect this happened

// This CAN be tracked
const count = ref(0)
count.value = 1  // Vue's setter intercepts this
```

The `.value` property is the key that allows Vue to intercept reads and writes.

### In Other Contexts

| Framework | Equivalent to Vue's `ref()` |
|:--- |:--- |
| **Vue 3** | `ref()` for primitives |
| **React** | `useState()` hook |
| **Angular** | Signals or RxJS BehaviorSubject |
| **Svelte** | `let` with `$:` reactive statements |
| **Solid.js** | `createSignal()` |

```typescript
// Vue 3
const count = ref(0)
count.value++

// React (for comparison)
const [count, setCount] = useState(0)
setCount(count + 1)

// The key difference: Vue refs are mutable, React state requires setter functions
```

### When to Use / When NOT to Use

| ✅ Use `ref()` When | ❌ Don't Use `ref()` When |
|:--- |:--- |
| Storing primitive values (string, number, boolean) | Value never changes (use `const`) |
| Value needs to update the UI when changed | Storing complex nested objects (consider `reactive()`) |
| Passing reactive data to composables | Temporary calculation results |
| Storing single DOM element references | Large datasets that don't need reactivity |

---

## 4. New Terminology

| Term | Definition |
|:--- |:--- |
| **Reactivity** | The ability for the UI to automatically update when underlying data changes |
| **`ref()`** | A Vue function that creates a reactive reference to a value, enabling change detection |
| **`.value`** | The property used to access or modify the actual value inside a ref (required in script, auto-unwrapped in template) |
| **Dependency Tracking** | Vue's mechanism to record which components depend on which reactive data |
| **Auto-unwrapping** | Vue automatically extracts `.value` in templates, so you write `{{ count }}` not `{{ count.value }}` |

---

## 5. Algorithmic Thinking (Planning the Solution)

**The Plan (Narrative):** To build our step counter with reactive updates:

```
┌─────────────────────────────────────────────────────────────────┐
│ STEP 1: Identify the changing data                              │
│   └──▶ What values will change over time?                       │
│        • steps (number) — changes with each step                │
│        • goal (number) — user can modify                        │
│        These need to be refs!                                   │
│                                                                 │
│ STEP 2: Create reactive references                              │
│   └──▶ const steps = ref(0)                                     │
│        const goal = ref(10000)                                  │
│                                                                 │
│ STEP 3: Create functions that modify the refs                   │
│   └──▶ function addSteps(amount: number) {                      │
│          steps.value += amount  // Use .value to modify         │
│        }                                                        │
│                                                                 │
│ STEP 4: Bind to template                                        │
│   └──▶ {{ steps }} displays current value                       │
│        @click="addSteps(100)" calls function on button click    │
│                                                                 │
│ STEP 5: UI auto-updates                                         │
│   └──▶ Vue handles DOM updates automatically!                   │
└─────────────────────────────────────────────────────────────────┘
```

**Decision Tree: Should This Be a `ref()`?**

```
Is this value...

Used in the template?
├── NO ──▶ Regular variable is fine
│
└── YES ──▶ Does it change after component mounts?
            │
            ├── NO ──▶ Regular const is fine
            │
            └── YES ──▶ Is it a primitive (string, number, boolean)?
                        │
                        ├── YES ──▶ Use ref()
                        │
                        └── NO (object/array) ──▶ Consider reactive()
                                                   (Lesson 3)
```

---

## 6. Initial Pattern Introduction

| What You Want (Intent) | Code Chunk (The Pattern) | Conceptual Link |
|:--- |:--- |:--- |
| Create a reactive number | `const count = ref(0)` | Wrap primitive in ref |
| Create a reactive string | `const name = ref('default')` | Same pattern, different type |
| Read the value in script | `count.value` | Access via .value |
| Read the value in template | `{{ count }}` | Auto-unwrapped |
| Modify the value | `count.value = 5` or `count.value++` | Assign via .value |

```vue
<script setup lang="ts">
// Import ref from Vue
// This is a named import - ref is one of many functions Vue provides
import { ref } from 'vue'

// Create reactive references for our step counter
// TypeScript infers the type from the initial value
const steps = ref(0)           // Type: Ref<number>
const goal = ref(10000)        // Type: Ref<number>
const userName = ref('Guest')  // Type: Ref<string>

// You can also explicitly type the ref
const isActive = ref<boolean>(true)

// Functions that modify reactive state
// Always use .value when reading or writing in script
const addSteps = (amount: number): void => {
  steps.value += amount
}

const resetSteps = (): void => {
  steps.value = 0
}

// Reading a ref value in script
const logCurrentSteps = (): void => {
  console.log(`Current steps: ${steps.value}`)  // .value required here
}
</script>

<template>
  <div class="p-6">
    <!-- In template, refs are auto-unwrapped - no .value needed! -->
    <h2>Hello, {{ userName }}!</h2>
    
    <!-- Display the reactive step count -->
    <p class="text-2xl font-bold">{{ steps }} / {{ goal }} steps</p>
    
    <!-- Buttons that call our functions to modify state -->
    <div class="flex gap-2 mt-4">
      <button @click="addSteps(100)" class="btn btn-primary">
        +100 Steps
      </button>
      <button @click="addSteps(1000)" class="btn btn-secondary">
        +1000 Steps
      </button>
      <button @click="resetSteps" class="btn btn-ghost">
        Reset
      </button>
    </div>
  </div>
</template>
```

**Key Points:**
- `import { ref } from 'vue'` — must import ref
- `ref(initialValue)` — creates the reactive reference
- `.value` — required in `<script>`, not in `<template>`
- Functions modify `.value`, not the ref itself

---

## 7. Comprehension Check

1. **Why can't Vue track changes to a regular `const count = 0` variable?**
   - Think about how JavaScript handles primitives.

2. **When do you need to use `.value` and when can you omit it?**
   - Consider where the code is running (script vs template).

3. **True or False: `const count = ref(0)` means `count` can never be reassigned.**
   - Think carefully about what `const` protects and what `.value` allows.

---

*Reply 'next' for Lesson 2.2 (Practice).*
