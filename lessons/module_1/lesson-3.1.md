# Lesson 3.1: Reactive Objects with `reactive()` — Theory & Concepts (Реактивные объекты с reactive())

## 1. Lesson Metadata

| Field               | Value                                                                 |
| :------------------ | :-------------------------------------------------------------------- |
| **Lesson Number**   | 3.1 of 25 (Part 1 of 2)                                               |
| **Topic**           | Reactive Objects with `reactive()` / Реактивные объекты с reactive()  |
| **Module**          | Module 1: Core Fundamentals                                           |
| **Prerequisites**   | Lessons 1, 2 (Components and ref())                                   |
| **You Should Know** | JavaScript objects, TypeScript interfaces, ref() usage, destructuring |
| **Unlocks**         | Lesson 3.2, then Lessons 4, 5                                         |
| **Duration**        | 30-35 minutes                                                         |

**Learning Objectives — Part 1 (Theory):**

1. **Remember:** Define `reactive()` and distinguish it from `ref()`
2. **Understand:** Explain how Vue makes objects reactive through Proxy and when to choose `reactive()` over `ref()`

---

## 2. Real-World Scenario & Context

**Scenario:** You're building a user profile settings page. The user's profile has multiple related fields: name, email, bio, avatar URL, preferences (theme, notifications), and account status. Using individual `ref()` calls for each field would be cumbersome and make it harder to manage related data together.

**Framework Context:** Vue provides `reactive()` to make entire objects reactive. Instead of wrapping each value individually, you can create a reactive object where all properties are automatically tracked — changes to any property trigger UI updates.

---

## 3. Core Concepts Explained (Deep Dive)

### How It Actually Works

`reactive()` uses JavaScript's `Proxy` to intercept all property access and mutations:

```
┌─────────────────────────────────────────────────────────────────┐
│                  WHAT reactive() CREATES                        │
│                                                                 │
│   const state = reactive({ count: 0, name: 'Vue' })             │
│                                                                 │
│   Creates a Proxy wrapper:                                      │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Proxy {                                                │   │
│   │    target: { count: 0, name: 'Vue' },  // Original obj  │   │
│   │                                                         │   │
│   │    get(target, key) {                                   │   │
│   │      track(target, key)  // Record dependency           │   │
│   │      return target[key]                                 │   │
│   │    },                                                   │   │
│   │                                                         │   │
│   │    set(target, key, value) {                            │   │
│   │      target[key] = value                                │   │
│   │      trigger(target, key)  // Notify watchers           │   │
│   │      return true                                        │   │
│   │    }                                                    │   │
│   │  }                                                      │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   Access: state.count → 0 (no .value needed!)                   │
│   Mutate: state.count = 5 (directly assign!)                    │
└─────────────────────────────────────────────────────────────────┘
```

**The Key Difference: ref() vs reactive()**

```
┌─────────────────────────────────────────────────────────────────┐
│                    ref() vs reactive()                          │
│                                                                 │
│   ref(0)                          reactive({ count: 0 })        │
│   ┌─────────────────────┐         ┌─────────────────────┐       │
│   │ RefImpl {           │         │ Proxy {             │       │
│   │   _value: 0,        │         │   count: 0          │       │
│   │   get value() {...} │         │   /* All props are  │       │
│   │   set value() {...} │         │      automatically  │       │
│   │ }                   │         │      tracked */     │       │
│   └─────────────────────┘         └─────────────────────┘       │
│                                                                 │
│   count.value = 5                 state.count = 5               │
│   ↑ .value required               ↑ Direct access!              │
│                                                                 │
│   {{ count }}                     {{ state.count }}             │
│   ↑ Auto-unwrapped in template    ↑ Access like normal object   │
└─────────────────────────────────────────────────────────────────┘
```

### Mental Model

Think of `reactive()` as putting your object into a **"surveillance room"** where every property access is monitored:

```
┌─────────────────────────────────────────────────────────────────┐
│                   REACTIVE SURVEILLANCE ROOM                    │
│                                                                 │
│   Your original object:           │   Vue's reactive proxy:     │
│   { name: 'John', age: 25 }       │   Watches EVERYTHING        │
│                                   │                             │
│   ┌─────────────────────────┐     │   📹 "name was read"        │
│   │  name: 'John'           │◀────┼── 📹 "age was read"         │
│   │  age: 25                │     │   📹 "name changed to Jane" │
│   └─────────────────────────┘     │   📹 "new prop 'email' added│
│                                   │                             │
│   Every READ records a dependency │                             │
│   Every WRITE triggers updates    │                             │
└─────────────────────────────────────────────────────────────────┘
```

Unlike `ref()` which only wraps a single value, `reactive()` makes the entire object structure reactive — including nested objects!

### Deep Reactivity

```typescript
const state = reactive({
  user: {
    profile: {
      name: "John",
      settings: {
        theme: "dark",
      },
    },
  },
});

// ALL of these trigger reactivity:
state.user.profile.name = "Jane"; // ✅ Tracked
state.user.profile.settings.theme = "light"; // ✅ Tracked (nested!)
```

### In Other Contexts

| Framework    | Equivalent to Vue's `reactive()`                              |
| :----------- | :------------------------------------------------------------ |
| **Vue 3**    | `reactive()` for objects/arrays                               |
| **React**    | `useState()` with object (but requires spreading for updates) |
| **MobX**     | `observable()`                                                |
| **Svelte**   | Plain objects with `$:` reactive statements                   |
| **Solid.js** | `createStore()`                                               |

```typescript
// Vue 3 - Direct mutation
const state = reactive({ count: 0 });
state.count++; // Just works!

// React - Must create new object (immutable pattern)
const [state, setState] = useState({ count: 0 });
setState((prev) => ({ ...prev, count: prev.count + 1 })); // Spread required
```

Vue's approach allows direct mutation, which many developers find more intuitive.

### When to Use / When NOT to Use

| ✅ Use `reactive()` When                           | ❌ Don't Use `reactive()` When              |
| :------------------------------------------------- | :------------------------------------------ |
| Grouping related state (form fields, user profile) | Single primitive value (use `ref()`)        |
| Object with multiple properties that change        | Passing to composables that expect `ref()`  |
| You want direct property access without `.value`   | You need to replace the entire object       |
| Complex nested data structures                     | You want to destructure and keep reactivity |

### The Reactivity Loss Trap ⚠️

```typescript
// ⚠️ DANGER: Destructuring breaks reactivity!
const state = reactive({ count: 0, name: "Vue" });

// ❌ WRONG - These are now plain values, not reactive!
const { count, name } = state;
count++; // Does NOT update the UI!

// ✅ CORRECT - Keep the object reference
state.count++; // Updates the UI!

// ✅ CORRECT - Use toRefs() if you need destructuring
import { toRefs } from "vue";
const { count, name } = toRefs(state);
count.value++; // Now it works! (but back to .value)
```

---

## 4. New Terminology

| Term                | Definition                                                                     |
| :------------------ | :----------------------------------------------------------------------------- |
| **`reactive()`**    | Vue function that makes an object reactive by wrapping it in a Proxy           |
| **Proxy**           | JavaScript feature that intercepts operations on objects (get, set, delete)    |
| **Deep Reactivity** | Nested objects inside a reactive object are also automatically reactive        |
| **Reactivity Loss** | When destructuring or reassigning breaks the connection to the reactive source |
| **`toRefs()`**      | Converts each property of a reactive object to a ref, preserving reactivity    |

---

## 5. Algorithmic Thinking (Planning the Solution)

**The Plan (Narrative):** To build our user profile settings page:

```
┌─────────────────────────────────────────────────────────────────┐
│ STEP 1: Identify related data that should be grouped            │
│   └──▶ User profile fields: name, email, bio, avatar            │
│        Preferences: theme, notifications                        │
│        Account: status, createdAt                               │
│                                                                 │
│ STEP 2: Design the TypeScript interface                         │
│   └──▶ interface UserProfile {                                  │
│          name: string                                           │
│          email: string                                          │
│          preferences: { theme: string; notifications: boolean } │
│        }                                                        │
│                                                                 │
│ STEP 3: Create the reactive object                              │
│   └──▶ const profile = reactive<UserProfile>({ ... })           │
│                                                                 │
│ STEP 4: Create update functions                                 │
│   └──▶ Modify properties directly: profile.name = 'New Name'    │
│        No .value needed!                                        │
│                                                                 │
│ STEP 5: Bind to template                                        │
│   └──▶ {{ profile.name }}, {{ profile.preferences.theme }}      │
└─────────────────────────────────────────────────────────────────┘
```

**Decision Tree: ref() or reactive()?**

```
What type of data are you storing?

Primitive (string, number, boolean)?
├── YES ──▶ Use ref()
│
└── NO (object/array) ──▶ Will you replace the whole object?
                          │
                          ├── YES ──▶ Use ref()
                          │           const user = ref<User>(null)
                          │           user.value = newUser  // Replace whole thing
                          │
                          └── NO ──▶ Will you destructure it?
                                    │
                                    ├── YES ──▶ Use ref() or reactive() + toRefs()
                                    │
                                    └── NO ──▶ Use reactive()
                                              Direct property access, no .value!
```

---

## 6. Initial Pattern Introduction

| What You Want (Intent) | Code Chunk (The Pattern)          | Conceptual Link     |
| :--------------------- | :-------------------------------- | :------------------ |
| Create reactive object | `const state = reactive({ ... })` | Proxy wrapper       |
| Access property        | `state.property`                  | Direct access       |
| Modify property        | `state.property = value`          | No .value needed    |
| Add new property       | `state.newProp = value`           | Dynamically tracked |
| Delete property        | `delete state.prop`               | Triggers reactivity |
| Type the object        | `reactive<Interface>({ ... })`    | TypeScript safety   |

```vue
<script setup lang="ts">
import { reactive } from "vue";

// Define the shape of our data with TypeScript
interface UserProfile {
  name: string;
  email: string;
  bio: string;
  preferences: {
    theme: "light" | "dark";
    notifications: boolean;
    language: string;
  };
}

// Create a reactive object with the defined type
// All properties and nested objects are automatically reactive
const profile = reactive<UserProfile>({
  name: "Sarah Chen",
  email: "sarah@example.com",
  bio: "Frontend developer passionate about Vue.js",
  preferences: {
    theme: "dark",
    notifications: true,
    language: "en",
  },
});

// Functions to modify the reactive state
// Note: No .value needed! Direct property access.
const updateName = (newName: string): void => {
  profile.name = newName; // Direct assignment
};

const toggleNotifications = (): void => {
  profile.preferences.notifications = !profile.preferences.notifications;
};

const switchTheme = (): void => {
  profile.preferences.theme =
    profile.preferences.theme === "dark" ? "light" : "dark";
};

// Reading reactive properties (also no .value)
const getDisplayName = (): string => {
  return profile.name || "Anonymous User";
};
</script>

<template>
  <div class="p-6">
    <!-- Access reactive properties directly -->
    <h2 class="text-2xl font-bold">{{ profile.name }}</h2>
    <p class="text-gray-600">{{ profile.email }}</p>
    <p class="mt-2">{{ profile.bio }}</p>

    <!-- Nested properties also reactive -->
    <div class="mt-4 p-4 bg-gray-100 rounded">
      <h3 class="font-semibold">Preferences</h3>
      <p>Theme: {{ profile.preferences.theme }}</p>
      <p>
        Notifications: {{ profile.preferences.notifications ? "On" : "Off" }}
      </p>
      <p>Language: {{ profile.preferences.language }}</p>
    </div>

    <!-- Buttons to modify state -->
    <div class="mt-4 flex gap-2">
      <button @click="switchTheme" class="btn">
        Switch to {{ profile.preferences.theme === "dark" ? "Light" : "Dark" }}
      </button>
      <button @click="toggleNotifications" class="btn">
        {{
          profile.preferences.notifications ? "Disable" : "Enable"
        }}
        Notifications
      </button>
    </div>
  </div>
</template>
```

**Key Differences from ref():**

- Import `reactive` instead of `ref`
- No `.value` anywhere — access properties directly
- Best for objects with multiple related properties
- Deep reactivity — nested objects are also reactive

---

## 7. Comprehension Check

1. **What JavaScript feature does `reactive()` use under the hood to track changes?**

   - Think about how Vue intercepts property access.

2. **Why does destructuring a reactive object break reactivity?**

   - Consider what happens when you extract a primitive from an object.

3. **True or False: You can use `reactive()` to wrap a primitive like `reactive(5)`.**
   - Think about what reactive() is designed for.

---

_Reply 'next' for Lesson 3.2 (Practice)._
