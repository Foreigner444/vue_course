# Lesson 5.1: Running Side Effects with `watch()` — Theory & Concepts (Отслеживание изменений с watch())

## 1. Lesson Metadata

| Field               | Value                                                                  |
| :------------------ | :--------------------------------------------------------------------- |
| **Lesson Number**   | 5.1 of 25 (Part 1 of 2)                                                |
| **Topic**           | Running Side Effects with `watch()` / Отслеживание изменений с watch() |
| **Module**          | Module 1: Core Fundamentals                                            |
| **Prerequisites**   | Lessons 1-4 (Components, ref(), reactive(), computed())                |
| **You Should Know** | ref(), reactive(), computed(), async/await, side effects concept       |
| **Unlocks**         | Lesson 5.2, then Module 1 Checkpoint Projects                          |
| **Duration**        | 30-35 minutes                                                          |

**Learning Objectives — Part 1 (Theory):**

1. **Remember:** Define `watch()` and distinguish it from `computed()`
2. **Understand:** Explain when to use watch for side effects vs computed for derived values

---

## 2. Real-World Scenario & Context

**Scenario:** You're building a search feature for a product catalog. As the user types in the search box, you want to:

1. **Debounce** the input (wait 300ms after they stop typing)
2. **Fetch** matching products from an API
3. **Save** the search term to localStorage for history
4. **Log** analytics events

These are all **side effects** — they don't just return a value, they interact with the outside world. You can't do this with `computed()` because computed properties should be pure (no side effects).

**Framework Context:** Vue provides `watch()` to react to reactive data changes and perform side effects. Unlike `computed()`, watchers don't return a value — they execute code when dependencies change.

---

## 3. Core Concepts Explained (Deep Dive)

### How It Actually Works

`watch()` observes reactive sources and runs a callback when they change:

```
┌─────────────────────────────────────────────────────────────────┐
│                    HOW watch() WORKS                            │
│                                                                 │
│   watch(source, callback, options?)                             │
│                                                                 │
│   ┌──────────────┐     ┌──────────────────┐     ┌────────────┐  │
│   │    Source    │     │    Vue Watcher   │     │  Callback  │  │
│   │  (ref/reactive)    │    Observes and  │     │  Executes  │  │
│   │              │ ──▶ │    detects       │ ──▶ │  side      │  │
│   │ searchQuery  │     │    changes       │     │  effects   │  │
│   └──────────────┘     └──────────────────┘     └────────────┘  │
│                                                                 │
│   Timeline:                                                     │
│   ─────────────────────────────────────────────────────────     │
│   searchQuery = ''                                              │
│   searchQuery = 'v'        → callback('v', '')                  │
│   searchQuery = 'vu'       → callback('vu', 'v')                │
│   searchQuery = 'vue'      → callback('vue', 'vu')              │
│                                                                 │
│   Callback receives: (newValue, oldValue)                       │
└─────────────────────────────────────────────────────────────────┘
```

### watch() vs computed()

```
┌─────────────────────────────────────────────────────────────────┐
│               computed() vs watch()                             │
│                                                                 │
│   COMPUTED                           WATCH                      │
│   ────────                           ─────                      │
│   • Returns a value                  • Runs side effects        │
│   • Cached until deps change         • Runs on every change     │
│   • Synchronous only                 • Can be async             │
│   • Used IN templates                • Used FOR reactions       │
│   • Pure (no side effects)           • Meant for side effects   │
│                                                                 │
│   Example:                           Example:                   │
│   ─────────                          ─────────                  │
│   const total = computed(() =>       watch(searchQuery, async   │
│     price.value * qty.value          (newVal) => {              │
│   )                                    await fetchResults(newVal)│
│   // Use: {{ total }}                  saveToHistory(newVal)    │
│                                      })                         │
└─────────────────────────────────────────────────────────────────┘
```

### Mental Model

Think of `watch()` as a **security camera with an alarm system**:

```
┌─────────────────────────────────────────────────────────────────┐
│              SECURITY CAMERA ANALOGY                            │
│                                                                 │
│   computed() = Calculator Display                               │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Shows: price × quantity = total                        │   │
│   │  Just displays a derived value, nothing else            │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   watch() = Security Camera + Alarm + Logger                    │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  📹 Watching: "door" variable                           │   │
│   │                                                         │   │
│   │  When door changes from "closed" to "open":             │   │
│   │    1. 🔔 Sound alarm (side effect!)                     │   │
│   │    2. 📝 Log to security system (side effect!)          │   │
│   │    3. 📧 Send notification (side effect!)               │   │
│   │                                                         │   │
│   │  The camera doesn't "return" anything —                 │   │
│   │  it DOES things when it sees changes                    │   │
│   └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Three Ways to Watch

```typescript
import { ref, reactive, watch, watchEffect } from "vue";

const count = ref(0);
const state = reactive({ name: "Vue", count: 0 });

// 1. Watch a single ref
watch(count, (newVal, oldVal) => {
  console.log(`count: ${oldVal} → ${newVal}`);
});

// 2. Watch a specific property of reactive object (use getter)
watch(
  () => state.name,
  (newName, oldName) => {
    console.log(`name: ${oldName} → ${newName}`);
  }
);

// 3. Watch multiple sources
watch([count, () => state.name], ([newCount, newName], [oldCount, oldName]) => {
  console.log(`count: ${oldCount} → ${newCount}`);
  console.log(`name: ${oldName} → ${newName}`);
});
```

### watchEffect() — The Auto-Tracking Alternative

```typescript
import { watchEffect } from "vue";

// watchEffect automatically tracks ALL reactive deps used inside
watchEffect(() => {
  console.log(`count is ${count.value}`); // Tracks count
  console.log(`name is ${state.name}`); // Tracks state.name
  // Runs immediately, then on any change to count or state.name
});

// Comparison:
// watch() = You specify what to watch
// watchEffect() = Vue figures out what you're using
```

### In Other Contexts

| Framework   | Equivalent to Vue's `watch()`       |
| :---------- | :---------------------------------- |
| **Vue 3**   | `watch()` / `watchEffect()`         |
| **React**   | `useEffect()` hook                  |
| **Angular** | `ngOnChanges` or RxJS subscriptions |
| **MobX**    | `reaction()` / `autorun()`          |
| **Svelte**  | `$:` reactive statements            |

```typescript
// Vue 3
watch(count, (newVal) => {
  document.title = `Count: ${newVal}`;
});

// React equivalent
useEffect(() => {
  document.title = `Count: ${count}`;
}, [count]); // Must specify deps manually!

// Vue's watch auto-tracks deps when using watchEffect()
```

### When to Use / When NOT to Use

| ✅ Use `watch()` When                   | ❌ Don't Use `watch()` When              |
| :-------------------------------------- | :--------------------------------------- |
| Making API calls based on data changes  | Computing derived values (use computed)  |
| Saving to localStorage/sessionStorage   | Simple data transformations              |
| Logging/analytics events                | Modifying other reactive state (usually) |
| DOM manipulation (rarely needed in Vue) | Synchronous calculations                 |
| Debouncing user input                   | Values needed in template                |

---

## 4. New Terminology

| Term                | Definition                                                                                  |
| :------------------ | :------------------------------------------------------------------------------------------ |
| **`watch()`**       | Vue function that runs a callback when watched reactive sources change                      |
| **`watchEffect()`** | Variant that auto-tracks dependencies and runs immediately                                  |
| **Side Effect**     | Any operation that affects something outside the function (API calls, DOM changes, logging) |
| **Immediate**       | Option to run the watcher callback immediately on setup                                     |
| **Deep**            | Option to watch nested properties of objects/arrays                                         |
| **Flush**           | Controls when the callback runs relative to Vue's update cycle                              |

---

## 5. Algorithmic Thinking (Planning the Solution)

**The Plan (Narrative):** To build our search feature with side effects:

```
┌─────────────────────────────────────────────────────────────────┐
│ STEP 1: Identify the reactive source (what triggers effects)   │
│   └──▶ searchQuery (ref) — user's input                        │
│                                                                 │
│ STEP 2: Identify the side effects needed                        │
│   └──▶ • Fetch products from API                                │
│        • Save to search history (localStorage)                  │
│        • Track analytics event                                  │
│                                                                 │
│ STEP 3: Consider timing/optimization                            │
│   └──▶ • Debounce to avoid excessive API calls                  │
│        • Don't fetch for empty queries                          │
│        • Show loading state during fetch                        │
│                                                                 │
│ STEP 4: Set up the watcher                                      │
│   └──▶ watch(searchQuery, async (newQuery) => {                 │
│          if (!newQuery) return                                  │
│          isLoading.value = true                                 │
│          results.value = await fetchProducts(newQuery)          │
│          saveToHistory(newQuery)                                │
│          trackSearch(newQuery)                                  │
│          isLoading.value = false                                │
│        })                                                       │
│                                                                 │
│ STEP 5: Handle cleanup if needed                                │
│   └──▶ Return cleanup function or use onWatcherCleanup          │
└─────────────────────────────────────────────────────────────────┘
```

**Decision Tree: computed vs watch?**

```
What do you need to do when data changes?

Return a calculated value?
├── YES ──▶ Use computed() ✅
│
└── NO ──▶ Perform side effects? (API, storage, logging, DOM)
            │
            ├── YES ──▶ Need to know old value?
            │           │
            │           ├── YES ──▶ Use watch() ✅
            │           │
            │           └── NO ──▶ Want auto-tracking?
            │                     │
            │                     ├── YES ──▶ Use watchEffect() ✅
            │                     │
            │                     └── NO ──▶ Use watch() ✅
            │
            └── NO ──▶ Maybe you don't need a watcher!
```

---

## 6. Initial Pattern Introduction

| What You Want (Intent)  | Code Chunk (The Pattern)                             | Conceptual Link  |
| :---------------------- | :--------------------------------------------------- | :--------------- |
| Watch a ref             | `watch(myRef, (new, old) => {...})`                  | Basic watcher    |
| Watch reactive property | `watch(() => state.prop, cb)`                        | Getter function  |
| Watch multiple sources  | `watch([ref1, ref2], ([n1, n2], [o1, o2]) => {...})` | Array of sources |
| Run immediately         | `watch(src, cb, { immediate: true })`                | Execute on setup |
| Watch nested objects    | `watch(src, cb, { deep: true })`                     | Deep observation |
| Auto-track dependencies | `watchEffect(() => {...})`                           | Runs immediately |

```vue
<script setup lang="ts">
import { ref, reactive, watch, watchEffect, onWatcherCleanup } from "vue";

// ============================================
// REACTIVE STATE
// ============================================

const searchQuery = ref("");
const userId = ref(1);
const settings = reactive({
  theme: "light",
  notifications: true,
});

const results = ref<string[]>([]);
const isLoading = ref(false);

// ============================================
// BASIC WATCH (single ref)
// ============================================

watch(searchQuery, async (newQuery, oldQuery) => {
  console.log(`Search changed: "${oldQuery}" → "${newQuery}"`);

  if (!newQuery.trim()) {
    results.value = [];
    return;
  }

  isLoading.value = true;

  // Simulate API call
  await new Promise((resolve) => setTimeout(resolve, 500));
  results.value = [`Result for: ${newQuery}`];

  isLoading.value = false;
});

// ============================================
// WATCH WITH IMMEDIATE (runs on mount)
// ============================================

watch(
  userId,
  async (newId) => {
    console.log(`Fetching user ${newId}...`);
    // Fetch user data immediately when component mounts
  },
  { immediate: true } // Run callback right away with current value
);

// ============================================
// WATCH REACTIVE PROPERTY (use getter)
// ============================================

watch(
  () => settings.theme, // Getter function for reactive property
  (newTheme, oldTheme) => {
    console.log(`Theme changed: ${oldTheme} → ${newTheme}`);
    document.documentElement.setAttribute("data-theme", newTheme);
  }
);

// ============================================
// WATCH MULTIPLE SOURCES
// ============================================

watch([searchQuery, userId], ([newQuery, newUserId], [oldQuery, oldUserId]) => {
  console.log("Multiple values changed:");
  console.log(`  Query: ${oldQuery} → ${newQuery}`);
  console.log(`  User ID: ${oldUserId} → ${newUserId}`);
});

// ============================================
// DEEP WATCH (nested objects)
// ============================================

watch(
  settings,
  (newSettings) => {
    console.log("Settings changed:", newSettings);
    localStorage.setItem("settings", JSON.stringify(newSettings));
  },
  { deep: true } // Watch ALL nested properties
);

// ============================================
// WATCHEFFECT (auto-track dependencies)
// ============================================

watchEffect(() => {
  // This automatically tracks searchQuery and userId
  // because they're accessed inside
  console.log(
    `Auto-tracked: search="${searchQuery.value}", user=${userId.value}`
  );
});

// ============================================
// CLEANUP (cancel pending operations)
// ============================================

watch(searchQuery, async (newQuery) => {
  const controller = new AbortController();

  // Register cleanup for when watcher re-runs or component unmounts
  onWatcherCleanup(() => {
    controller.abort(); // Cancel the fetch if query changes again
  });

  try {
    const response = await fetch(`/api/search?q=${newQuery}`, {
      signal: controller.signal,
    });
    results.value = await response.json();
  } catch (e) {
    if ((e as Error).name !== "AbortError") {
      console.error("Search failed:", e);
    }
  }
});
</script>

<template>
  <div class="p-6 space-y-4">
    <div>
      <label class="block mb-1">Search:</label>
      <input
        v-model="searchQuery"
        class="border p-2 rounded w-full"
        placeholder="Type to search..."
      />
    </div>

    <div>
      <label class="block mb-1">User ID:</label>
      <input v-model.number="userId" type="number" class="border p-2 rounded" />
    </div>

    <div>
      <label class="block mb-1">Theme:</label>
      <select v-model="settings.theme" class="border p-2 rounded">
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </select>
    </div>

    <div v-if="isLoading" class="text-gray-500">Loading...</div>
    <ul v-else>
      <li v-for="result in results" :key="result">{{ result }}</li>
    </ul>
  </div>
</template>
```

**Key Points:**

- `watch(source, callback, options)` — explicit source declaration
- `watchEffect(callback)` — auto-tracks dependencies
- Use getter function `() => state.prop` for reactive object properties
- `{ immediate: true }` — run callback on setup
- `{ deep: true }` — watch nested changes
- `onWatcherCleanup()` — clean up side effects

---

## 7. Comprehension Check

1. **Why can't you use `computed()` to make an API call when a search query changes?**

   - Think about what computed properties are designed for.

2. **What's the difference between `watch(myRef, cb)` and `watch(() => state.prop, cb)`?**

   - Consider how Vue tracks dependencies.

3. **True or False: `watchEffect()` requires you to specify which reactive values to watch.**
   - Think about how watchEffect differs from watch.

---

_Reply 'next' for Lesson 5.2 (Practice)._
