# Lesson 4.2: Derived State with `computed()` — Practice & Application (Производное состояние с computed())
---
## 1. Lesson Metadata
| Field              | Value                                                                   |
| :----------------- | :---------------------------------------------------------------------- |
| **Lesson Number**  | 4.2 of 28 (Part 2 of 2)                                                 |
| **Topic**          | Derived State with `computed()` / Производное состояние с computed()    |
| **Continues From** | Lesson 4.1                                                              |
| **Duration**       | 30-35 minutes                                                           |
**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Create computed properties for filtering, sorting, and transforming data
2. **Analyze:** Debug caching issues and choose between computed and methods appropriately
---
## 2. Quick Recap
- **`computed()`** creates reactive derived values that auto-update when dependencies change
- **Caching:** Computed values only recalculate when their dependencies change
- **Access:** Use `.value` in script, auto-unwrapped in template
- **Writable computed:** Use `{ get, set }` syntax for two-way derived values
---
## 3. The Variable Frame & Complexity Scale
### Basic Form: Simple Derivation
```vue
<script setup lang="ts">
import { ref, computed } from "vue";
const count = ref(5);
const doubled = computed(() => count.value * 2);
</script>
<template>
  <p>{{ count }} × 2 = {{ doubled }}</p>
</template>
```
### With Options: Filtering and Sorting
```vue
<script setup lang="ts">
import { ref, computed } from "vue";
interface Task {
  id: number;
  title: string;
  completed: boolean;
  priority: "low" | "medium" | "high";
}
const tasks = ref<Task[]>([
  { id: 1, title: "Learn Vue", completed: false, priority: "high" },
  { id: 2, title: "Build app", completed: false, priority: "medium" },
  { id: 3, title: "Deploy", completed: true, priority: "low" },
]);
const filter = ref<"all" | "active" | "completed">("all");
const sortBy = ref<"priority" | "title">("priority");
// Filtered tasks based on current filter
const filteredTasks = computed(() => {
  switch (filter.value) {
    case "active":
      return tasks.value.filter((t) => !t.completed);
    case "completed":
      return tasks.value.filter((t) => t.completed);
    default:
      return tasks.value;
  }
});
// Sorted and filtered tasks
const sortedTasks = computed(() => {
  const priorityOrder = { high: 0, medium: 1, low: 2 };
  return [...filteredTasks.value].sort((a, b) => {
    if (sortBy.value === "priority") {
      return priorityOrder[a.priority] - priorityOrder[b.priority];
    }
    return a.title.localeCompare(b.title);
  });
});
</script>
```
### Advanced Form: Chained Computeds with Statistics
```vue
<script setup lang="ts">
import { ref, computed } from "vue";
interface Sale {
  id: number;
  product: string;
  amount: number;
  date: string;
  region: string;
}
const sales = ref<Sale[]>([/* ... */]);
const selectedRegion = ref<string>("all");
const dateRange = ref({ start: "2024-01-01", end: "2024-12-31" });
// Layer 1: Filter by region
const regionFilteredSales = computed(() => {
  if (selectedRegion.value === "all") return sales.value;
  return sales.value.filter((s) => s.region === selectedRegion.value);
});
// Layer 2: Filter by date (depends on Layer 1)
const dateFilteredSales = computed(() => {
  return regionFilteredSales.value.filter((s) => {
    return s.date >= dateRange.value.start && s.date <= dateRange.value.end;
  });
});
// Layer 3: Statistics (depends on Layer 2)
const stats = computed(() => ({
  count: dateFilteredSales.value.length,
  total: dateFilteredSales.value.reduce((sum, s) => sum + s.amount, 0),
  average: dateFilteredSales.value.length > 0
    ? dateFilteredSales.value.reduce((sum, s) => sum + s.amount, 0) / dateFilteredSales.value.length
    : 0,
  max: Math.max(...dateFilteredSales.value.map((s) => s.amount), 0),
  min: Math.min(...dateFilteredSales.value.map((s) => s.amount), Infinity),
}));
</script>
```
---
## 4. Micro-Implementation Drills
### Drill 1 (Basic): Message Reverser
Create a computed property that reverses a string.
```vue
<script setup lang="ts">
import { ref, computed } from "vue";
// TODO: Create a ref called `message` with initial value "Hello Vue"
// TODO: Create a computed `reversedMessage` that reverses the string
</script>
<template>
  <div>
    <input v-model="message" />
    <!-- TODO: Display the reversed message -->
  </div>
</template>
```
**Solution:**
```vue
<script setup lang="ts">
import { ref, computed } from "vue";
const message = ref("Hello Vue");
const reversedMessage = computed((): string => {
  return message.value.split("").reverse().join("");
});
</script>
<template>
  <div>
    <input v-model="message" placeholder="Type a message" class="input input-bordered" />
    <p>Original: {{ message }}</p>
    <p>Reversed: {{ reversedMessage }}</p>
  </div>
</template>
```
---
### Drill 2 (Variation): Filtered List
Create computed properties to filter and count items.
```vue
<script setup lang="ts">
import { ref, computed } from "vue";
interface Todo {
  id: number;
  text: string;
  done: boolean;
}
const todos = ref<Todo[]>([
  { id: 1, text: "Learn Vue", done: true },
  { id: 2, text: "Build project", done: false },
  { id: 3, text: "Deploy", done: false },
]);
// TODO: Create computed `activeTodos` - filter incomplete items
// TODO: Create computed `completedTodos` - filter complete items
// TODO: Create computed `activeCount` - count of incomplete items
// TODO: Create computed `progress` - percentage complete (0-100)
</script>
<template>
  <!-- TODO: Display progress bar and filtered lists -->
</template>
```
**Solution:**
```vue
<script setup lang="ts">
import { ref, computed } from "vue";
interface Todo {
  id: number;
  text: string;
  done: boolean;
}
const todos = ref<Todo[]>([
  { id: 1, text: "Learn Vue", done: true },
  { id: 2, text: "Build project", done: false },
  { id: 3, text: "Deploy", done: false },
]);
const activeTodos = computed((): Todo[] => {
  return todos.value.filter((todo) => !todo.done);
});
const completedTodos = computed((): Todo[] => {
  return todos.value.filter((todo) => todo.done);
});
const activeCount = computed((): number => {
  return activeTodos.value.length;
});
const progress = computed((): number => {
  if (todos.value.length === 0) return 0;
  return Math.round((completedTodos.value.length / todos.value.length) * 100);
});
const toggleTodo = (id: number): void => {
  const todo = todos.value.find((t) => t.id === id);
  if (todo) {
    todo.done = !todo.done;
  }
};
</script>
<template>
  <div class="space-y-4">
    <div>
      <p class="font-bold">Progress: {{ progress }}%</p>
      <progress class="progress progress-primary w-full" :value="progress" max="100"></progress>
    </div>
    <div>
      <h3 class="font-bold">Active ({{ activeCount }})</h3>
      <ul>
        <li v-for="todo in activeTodos" :key="todo.id" class="flex gap-2">
          <input type="checkbox" @change="toggleTodo(todo.id)" />
          {{ todo.text }}
        </li>
      </ul>
    </div>
    <div>
      <h3 class="font-bold">Completed ({{ completedTodos.length }})</h3>
      <ul>
        <li v-for="todo in completedTodos" :key="todo.id" class="flex gap-2 line-through opacity-50">
          <input type="checkbox" checked @change="toggleTodo(todo.id)" />
          {{ todo.text }}
        </li>
      </ul>
    </div>
  </div>
</template>
```
---
### Drill 3 (Combination): Search and Sort
Combine multiple computeds for search, filter, and sort.
```vue
<script setup lang="ts">
import { ref, computed } from "vue";
interface Product {
  id: number;
  name: string;
  price: number;
  category: string;
  inStock: boolean;
}
const products = ref<Product[]>([
  { id: 1, name: "Laptop", price: 999, category: "Electronics", inStock: true },
  { id: 2, name: "Desk", price: 299, category: "Furniture", inStock: true },
  { id: 3, name: "Mouse", price: 29, category: "Electronics", inStock: false },
  { id: 4, name: "Chair", price: 199, category: "Furniture", inStock: true },
]);
const searchQuery = ref("");
const selectedCategory = ref("all");
const sortOrder = ref<"asc" | "desc">("asc");
const showInStockOnly = ref(false);
// TODO: Create computed `categories` - unique list of categories
// TODO: Create computed `filteredProducts` - apply all filters
// TODO: Create computed `sortedProducts` - sort filtered results
// TODO: Create computed `resultCount` - count of final results
</script>
<template>
  <!-- TODO: Search input, category dropdown, sort toggle, product list -->
</template>
```
**Solution:**
```vue
<script setup lang="ts">
import { ref, computed } from "vue";
interface Product {
  id: number;
  name: string;
  price: number;
  category: string;
  inStock: boolean;
}
const products = ref<Product[]>([
  { id: 1, name: "Laptop", price: 999, category: "Electronics", inStock: true },
  { id: 2, name: "Desk", price: 299, category: "Furniture", inStock: true },
  { id: 3, name: "Mouse", price: 29, category: "Electronics", inStock: false },
  { id: 4, name: "Chair", price: 199, category: "Furniture", inStock: true },
  { id: 5, name: "Monitor", price: 399, category: "Electronics", inStock: true },
]);
const searchQuery = ref("");
const selectedCategory = ref("all");
const sortOrder = ref<"asc" | "desc">("asc");
const showInStockOnly = ref(false);
// Unique categories from products
const categories = computed((): string[] => {
  const cats = products.value.map((p) => p.category);
  return [...new Set(cats)];
});
// Apply all filters
const filteredProducts = computed((): Product[] => {
  return products.value.filter((product) => {
    // Search filter
    const matchesSearch = product.name
      .toLowerCase()
      .includes(searchQuery.value.toLowerCase());
    // Category filter
    const matchesCategory =
      selectedCategory.value === "all" || product.category === selectedCategory.value;
    // Stock filter
    const matchesStock = !showInStockOnly.value || product.inStock;
    return matchesSearch && matchesCategory && matchesStock;
  });
});
// Sort filtered results
const sortedProducts = computed((): Product[] => {
  return [...filteredProducts.value].sort((a, b) => {
    const modifier = sortOrder.value === "asc" ? 1 : -1;
    return (a.price - b.price) * modifier;
  });
});
// Result count
const resultCount = computed((): number => {
  return sortedProducts.value.length;
});
const toggleSortOrder = (): void => {
  sortOrder.value = sortOrder.value === "asc" ? "desc" : "asc";
};
</script>
<template>
  <div class="space-y-4">
    <!-- Filters -->
    <div class="flex flex-wrap gap-4">
      <input
        v-model="searchQuery"
        type="text"
        placeholder="Search products..."
        class="input input-bordered"
      />
      <select v-model="selectedCategory" class="select select-bordered">
        <option value="all">All Categories</option>
        <option v-for="cat in categories" :key="cat" :value="cat">{{ cat }}</option>
      </select>
      <button @click="toggleSortOrder" class="btn btn-outline">
        Price: {{ sortOrder === "asc" ? "↑ Low to High" : "↓ High to Low" }}
      </button>
      <label class="flex items-center gap-2">
        <input v-model="showInStockOnly" type="checkbox" class="checkbox" />
        In Stock Only
      </label>
    </div>
    <!-- Results -->
    <p class="text-sm text-base-content/60">
      Showing {{ resultCount }} of {{ products.length }} products
    </p>
    <!-- Product List -->
    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
      <div
        v-for="product in sortedProducts"
        :key="product.id"
        class="card bg-base-200 p-4"
      >
        <h3 class="font-bold">{{ product.name }}</h3>
        <p>{{ product.category }}</p>
        <p class="text-lg">${{ product.price }}</p>
        <span
          :class="product.inStock ? 'text-success' : 'text-error'"
        >
          {{ product.inStock ? "In Stock" : "Out of Stock" }}
        </span>
      </div>
    </div>
    <p v-if="resultCount === 0" class="text-center text-base-content/60">
      No products match your filters
    </p>
  </div>
</template>
```
---
### Drill 4 (Edge Case): Writable Computed
Create a two-way computed for temperature conversion.
```vue
<script setup lang="ts">
import { ref, computed } from "vue";
// TODO: Create ref `celsius` with initial value 0
// TODO: Create writable computed `fahrenheit` that:
//   - GET: converts celsius to fahrenheit
//   - SET: converts fahrenheit input back to celsius
</script>
<template>
  <!-- TODO: Two-way bind inputs to both celsius and fahrenheit -->
</template>
```
**Solution:**
```vue
<script setup lang="ts">
import { ref, computed } from "vue";
const celsius = ref(0);
const fahrenheit = computed({
  get: (): number => {
    return (celsius.value * 9) / 5 + 32;
  },
  set: (newFahrenheit: number): void => {
    celsius.value = ((newFahrenheit - 32) * 5) / 9;
  },
});
const kelvin = computed({
  get: (): number => {
    return celsius.value + 273.15;
  },
  set: (newKelvin: number): void => {
    celsius.value = newKelvin - 273.15;
  },
});
</script>
<template>
  <div class="space-y-4 max-w-md">
    <h2 class="text-xl font-bold">Temperature Converter</h2>
    <div class="form-control">
      <label class="label">Celsius</label>
      <input
        v-model.number="celsius"
        type="number"
        step="0.1"
        class="input input-bordered"
      />
    </div>
    <div class="form-control">
      <label class="label">Fahrenheit</label>
      <input
        v-model.number="fahrenheit"
        type="number"
        step="0.1"
        class="input input-bordered"
      />
    </div>
    <div class="form-control">
      <label class="label">Kelvin</label>
      <input
        v-model.number="kelvin"
        type="number"
        step="0.1"
        class="input input-bordered"
      />
    </div>
    <p class="text-sm text-base-content/60">
      Editing any field automatically updates the others!
    </p>
  </div>
</template>
```
---
## 5. Common Pitfalls & Anti-Patterns
| ❌ Common Mistake                             | ✅ Correct Approach                           | Why It Matters                                         |
| :-------------------------------------------- | :-------------------------------------------- | :----------------------------------------------------- |
| Using computed for side effects               | Use `watch()` for side effects                | Computed should be pure functions                      |
| Mutating source data inside computed          | Only read data, compute new values            | Mutations cause unpredictable behavior                 |
| Creating computed with parameters             | Use a function or a computed returning function | Computed doesn't accept parameters directly          |
### Example of Each Mistake
**Mistake 1: Side Effects in Computed**
```vue
<script setup lang="ts">
import { ref, computed, watch } from "vue";
const count = ref(0);
// ❌ WRONG: Side effect in computed
const doubledWithLog = computed(() => {
  console.log("Calculating..."); // Side effect!
  localStorage.setItem("count", String(count.value)); // Side effect!
  return count.value * 2;
});
// ✅ CORRECT: Pure computed + separate watch for side effects
const doubled = computed(() => count.value * 2);
watch(count, (newVal) => {
  console.log("Count changed to:", newVal);
  localStorage.setItem("count", String(newVal));
});
</script>
```
**Mistake 2: Mutating Inside Computed**
```vue
<script setup lang="ts">
import { ref, computed } from "vue";
const items = ref([3, 1, 4, 1, 5]);
// ❌ WRONG: Mutates the original array
const sortedItems = computed(() => {
  return items.value.sort((a, b) => a - b); // sort() mutates!
});
// ✅ CORRECT: Create a copy before sorting
const sortedItemsCorrect = computed(() => {
  return [...items.value].sort((a, b) => a - b);
});
</script>
```
**Mistake 3: Computed with Parameters**
```vue
<script setup lang="ts">
import { ref, computed } from "vue";
const basePrice = ref(100);
// ❌ WRONG: Computed doesn't take parameters
// const priceWithTax = computed((taxRate) => basePrice.value * (1 + taxRate));
// ✅ CORRECT Option 1: Use a function
const getPriceWithTax = (taxRate: number): number => {
  return basePrice.value * (1 + taxRate);
};
// ✅ CORRECT Option 2: Make taxRate a ref
const taxRate = ref(0.1);
const priceWithTax = computed(() => basePrice.value * (1 + taxRate.value));
// ✅ CORRECT Option 3: Return a function from computed (memoized factory)
const createPriceCalculator = computed(() => {
  return (taxRate: number) => basePrice.value * (1 + taxRate);
});
// Usage: createPriceCalculator.value(0.1)
</script>
```
---
## 6. Mini-Project: Live Character Counter
Build a social media post composer with real-time character counting and limit warnings.
### File Structure
```
src/
├── components/
│   └── PostComposer.vue
├── App.vue
└── main.ts
```
### PostComposer.vue
```vue
<script setup lang="ts">
// ============================================
// IMPORTS
// ============================================
import { ref, computed } from "vue";
// ============================================
// CONSTANTS
// ============================================
const MAX_CHARS = 280;
const WARNING_THRESHOLD = 260;
const DANGER_THRESHOLD = 270;
// ============================================
// SOURCE STATE
// ============================================
const content = ref("");
const username = ref("@user");
// ============================================
// COMPUTED PROPERTIES
// ============================================
// Character count
const charCount = computed((): number => {
  return content.value.length;
});
// Remaining characters
const remainingChars = computed((): number => {
  return MAX_CHARS - charCount.value;
});
// Progress percentage for circular indicator
const progress = computed((): number => {
  return Math.min((charCount.value / MAX_CHARS) * 100, 100);
});
// Status: ok, warning, danger, over
const status = computed((): "ok" | "warning" | "danger" | "over" => {
  if (charCount.value > MAX_CHARS) return "over";
  if (charCount.value >= DANGER_THRESHOLD) return "danger";
  if (charCount.value >= WARNING_THRESHOLD) return "warning";
  return "ok";
});
// Status color for UI
const statusColor = computed((): string => {
  switch (status.value) {
    case "over":
      return "text-error";
    case "danger":
      return "text-error";
    case "warning":
      return "text-warning";
    default:
      return "text-base-content";
  }
});
// Progress bar color
const progressColor = computed((): string => {
  switch (status.value) {
    case "over":
    case "danger":
      return "progress-error";
    case "warning":
      return "progress-warning";
    default:
      return "progress-primary";
  }
});
// Can submit?
const canSubmit = computed((): boolean => {
  return charCount.value > 0 && charCount.value <= MAX_CHARS;
});
// Word count
const wordCount = computed((): number => {
  const text = content.value.trim();
  if (!text) return 0;
  return text.split(/\s+/).length;
});
// Hashtags found
const hashtags = computed((): string[] => {
  const matches = content.value.match(/#\w+/g);
  return matches ? [...new Set(matches)] : [];
});
// Mentions found
const mentions = computed((): string[] => {
  const matches = content.value.match(/@\w+/g);
  return matches ? [...new Set(matches)] : [];
});
// Preview text with highlighted hashtags/mentions (simplified)
const previewHtml = computed((): string => {
  let html = content.value
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/(#\w+)/g, '<span class="text-primary">$1</span>')
    .replace(/(@\w+)/g, '<span class="text-secondary">$1</span>');
  return html;
});
// ============================================
// ACTIONS
// ============================================
const handleSubmit = (): void => {
  if (!canSubmit.value) return;
  
  console.log("Posting:", {
    content: content.value,
    hashtags: hashtags.value,
    mentions: mentions.value,
    charCount: charCount.value,
  });
  
  // Clear after submit
  content.value = "";
};
const clearContent = (): void => {
  content.value = "";
};
</script>
<template>
  <div class="card bg-base-100 shadow-xl max-w-xl mx-auto">
    <div class="card-body">
      <!-- Header -->
      <div class="flex items-center gap-3 mb-4">
        <div class="avatar placeholder">
          <div class="bg-primary text-primary-content rounded-full w-12">
            <span>{{ username.slice(1, 3).toUpperCase() }}</span>
          </div>
        </div>
        <span class="font-bold">{{ username }}</span>
      </div>
      <!-- Textarea -->
      <div class="relative">
        <textarea
          v-model="content"
          class="textarea textarea-bordered w-full h-32 resize-none"
          placeholder="What's happening?"
          :class="{ 'textarea-error': status === 'over' }"
        ></textarea>
      </div>
      <!-- Stats Row -->
      <div class="flex justify-between items-center mt-2">
        <div class="flex gap-4 text-sm text-base-content/60">
          <span>{{ wordCount }} words</span>
          <span v-if="hashtags.length">{{ hashtags.length }} hashtags</span>
          <span v-if="mentions.length">{{ mentions.length }} mentions</span>
        </div>
        <div class="flex items-center gap-3">
          <!-- Progress indicator -->
          <div class="w-24">
            <progress
              class="progress w-full"
              :class="progressColor"
              :value="progress"
              max="100"
            ></progress>
          </div>
          <!-- Character count -->
          <span
            class="font-mono text-sm min-w-[3rem] text-right"
            :class="statusColor"
          >
            {{ remainingChars }}
          </span>
        </div>
      </div>
      <!-- Warning message -->
      <div
        v-if="status === 'over'"
        class="alert alert-error mt-2"
      >
        <span>Your post is {{ -remainingChars }} characters over the limit!</span>
      </div>
      <!-- Preview -->
      <div v-if="content" class="mt-4">
        <p class="text-sm font-semibold text-base-content/60 mb-2">Preview:</p>
        <div
          class="bg-base-200 p-3 rounded-lg"
          v-html="previewHtml"
        ></div>
      </div>
      <!-- Detected hashtags/mentions -->
      <div v-if="hashtags.length || mentions.length" class="flex gap-4 mt-2">
        <div v-if="hashtags.length">
          <span class="text-xs text-base-content/60">Hashtags: </span>
          <span
            v-for="tag in hashtags"
            :key="tag"
            class="badge badge-primary badge-sm mr-1"
          >
            {{ tag }}
          </span>
        </div>
        <div v-if="mentions.length">
          <span class="text-xs text-base-content/60">Mentions: </span>
          <span
            v-for="mention in mentions"
            :key="mention"
            class="badge badge-secondary badge-sm mr-1"
          >
            {{ mention }}
          </span>
        </div>
      </div>
      <!-- Actions -->
      <div class="card-actions justify-end mt-4">
        <button
          @click="clearContent"
          class="btn btn-ghost"
          :disabled="!content"
        >
          Clear
        </button>
        <button
          @click="handleSubmit"
          class="btn btn-primary"
          :disabled="!canSubmit"
        >
          Post
        </button>
      </div>
    </div>
  </div>
</template>
<style scoped>
.textarea {
  font-size: 1.125rem;
}
</style>
```
### App.vue
```vue
<script setup lang="ts">
import PostComposer from "./components/PostComposer.vue";
</script>
<template>
  <div class="min-h-screen bg-base-200 py-10 px-4">
    <h1 class="text-2xl font-bold text-center mb-6">Compose Post</h1>
    <PostComposer />
  </div>
</template>
```
### How to Test
1. Run `npm run dev`
2. Open the browser
3. Verify:
   - Character count updates as you type
   - Progress bar color changes at thresholds
   - Hashtags and mentions are detected and highlighted
   - Post button disables when over limit or empty
   - Clear button removes all content
   - Word count updates correctly
---
## 7. Implementation Exercises (Progressive)
### Level 1 (Recall): Fill in the Blanks
```vue
<script setup lang="ts">
import { ref, _____ } from "vue";
const price = ref(100);
const taxRate = ref(0.1);
const tax = _____(() => {
  return price._____ * taxRate.value;
});
const total = computed((): number => {
  return price.value + _____.value;
});
</script>
<template>
  <div>
    <p>Price: ${{ _____ }}</p>
    <p>Tax: ${{ tax.toFixed(2) }}</p>
    <p>Total: ${{ _____.toFixed(2) }}</p>
  </div>
</template>
```
**Answer:**
```vue
<script setup lang="ts">
import { ref, computed } from "vue";
const price = ref(100);
const taxRate = ref(0.1);
const tax = computed(() => {
  return price.value * taxRate.value;
});
const total = computed((): number => {
  return price.value + tax.value;
});
</script>
<template>
  <div>
    <p>Price: ${{ price }}</p>
    <p>Tax: ${{ tax.toFixed(2) }}</p>
    <p>Total: ${{ total.toFixed(2) }}</p>
  </div>
</template>
```
---
### Level 2 (Apply): Independent Coding Tasks
**Exercise 2.1: Grade Calculator**
Create a component with:
- An array of scores (refs)
- Computed average score
- Computed letter grade (A: 90+, B: 80+, C: 70+, D: 60+, F: below)
- Computed pass/fail status
**Exercise 2.2: Shopping Cart Summary**
Create a component with:
- Array of cart items (name, price, quantity)
- Computed subtotal
- Computed discount (10% if subtotal > $100)
- Computed final total
- Computed item count
---
### Level 3 (Debug): Find and Fix the Bugs
This component has **3 bugs**. Find and fix them:
```vue
<script setup lang="ts">
import { ref, computed } from "vue";
const numbers = ref([5, 2, 8, 1, 9]);
// Bug 1: Mutates source array
const sortedNumbers = computed(() => {
  return numbers.value.sort((a, b) => a - b);
});
// Bug 2: Side effect in computed
const sum = computed(() => {
  console.log("Calculating sum...");
  return numbers.value.reduce((a, b) => a + b, 0);
});
// Bug 3: Trying to pass parameter to computed
const multiplyBy = computed((factor: number) => {
  return sum.value * factor;
});
</script>
<template>
  <div>
    <p>Sorted: {{ sortedNumbers }}</p>
    <p>Sum: {{ sum }}</p>
    <p>Times 2: {{ multiplyBy(2) }}</p>
  </div>
</template>
```
**Corrected Code:**
```vue
<script setup lang="ts">
import { ref, computed } from "vue";
const numbers = ref([5, 2, 8, 1, 9]);
// Fix 1: Create copy before sorting
const sortedNumbers = computed(() => {
  return [...numbers.value].sort((a, b) => a - b);
});
// Fix 2: Remove side effect (or move to watch if needed)
const sum = computed(() => {
  return numbers.value.reduce((a, b) => a + b, 0);
});
// Fix 3: Use a regular function for parameterized calculations
const multiplyBy = (factor: number): number => {
  return sum.value * factor;
};
// Alternative Fix 3: Use a ref for the factor
const factor = ref(2);
const multiplied = computed(() => sum.value * factor.value);
</script>
<template>
  <div>
    <p>Sorted: {{ sortedNumbers.join(", ") }}</p>
    <p>Sum: {{ sum }}</p>
    <p>Times 2: {{ multiplyBy(2) }}</p>
  </div>
</template>
```
---
**Reply 'next' for Lesson 5.1 (Theory).**
