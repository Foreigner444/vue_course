# Lesson 3.2: Reactive Objects with `reactive()` — Practice & Application (Реактивные объекты с reactive())
---
## 1. Lesson Metadata
| Field              | Value                                                               |
| :----------------- | :------------------------------------------------------------------ |
| **Lesson Number**  | 3.2 of 28 (Part 2 of 2)                                             |
| **Topic**          | Reactive Objects with `reactive()` / Реактивные объекты с reactive()|
| **Continues From** | Lesson 3.1                                                          |
| **Duration**       | 30-35 minutes                                                       |
**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Create components using reactive objects with nested data and arrays
2. **Analyze:** Debug reactivity loss issues and use `toRefs()` for safe destructuring
---
## 2. Quick Recap
- **`reactive()`** creates a deeply reactive proxy for objects and arrays
- **No `.value` needed** — access properties directly
- **Reactivity breaks** when you: reassign the object, destructure, or pass primitives
- **Use `toRefs()`** to safely destructure while keeping reactivity
---
## 3. The Variable Frame & Complexity Scale
### Basic Form: Simple Object
```vue
<script setup lang="ts">
import { reactive } from "vue";
const form = reactive({
  username: "",
  email: "",
});
</script>
<template>
  <input v-model="form.username" placeholder="Username" />
  <input v-model="form.email" placeholder="Email" />
</template>
```
### With Options: Typed Object with Methods
```vue
<script setup lang="ts">
import { reactive } from "vue";
interface TodoItem {
  id: number;
  text: string;
  completed: boolean;
}
interface TodoState {
  items: TodoItem[];
  filter: "all" | "active" | "completed";
}
const state = reactive<TodoState>({
  items: [],
  filter: "all",
});
const addTodo = (text: string): void => {
  state.items.push({
    id: Date.now(),
    text,
    completed: false,
  });
};
const toggleTodo = (id: number): void => {
  const todo = state.items.find((item) => item.id === id);
  if (todo) {
    todo.completed = !todo.completed;
  }
};
const removeTodo = (id: number): void => {
  const index = state.items.findIndex((item) => item.id === id);
  if (index > -1) {
    state.items.splice(index, 1);
  }
};
</script>
```
### Advanced Form: Nested Objects with `toRefs()`
```vue
<script setup lang="ts">
import { reactive, toRefs } from "vue";
interface UserSettings {
  profile: {
    name: string;
    bio: string;
  };
  preferences: {
    theme: "light" | "dark";
    notifications: boolean;
    language: string;
  };
}
const settings = reactive<UserSettings>({
  profile: {
    name: "Alice",
    bio: "Vue developer",
  },
  preferences: {
    theme: "dark",
    notifications: true,
    language: "en",
  },
});
// Safe destructuring with toRefs
const { profile, preferences } = toRefs(settings);
// Now profile and preferences are Refs that stay reactive
const toggleTheme = (): void => {
  preferences.value.theme = 
    preferences.value.theme === "light" ? "dark" : "light";
};
</script>
<template>
  <div>
    <h2>{{ profile.name }}</h2>
    <p>Theme: {{ preferences.theme }}</p>
    <button @click="toggleTheme">Toggle Theme</button>
  </div>
</template>
```
---
## 4. Micro-Implementation Drills
### Drill 1 (Basic): Form Data Object
Create a reactive form object with validation state.
```vue
<script setup lang="ts">
import { reactive } from "vue";
// TODO: Create a reactive object called `form` with:
//   - email: string (empty)
//   - password: string (empty)
//   - rememberMe: boolean (false)
// TODO: Create a function `resetForm` that clears all fields
</script>
<template>
  <form @submit.prevent>
    <!-- TODO: Bind inputs to form properties with v-model -->
    <!-- TODO: Add a submit button and reset button -->
  </form>
</template>
```
**Solution:**
```vue
<script setup lang="ts">
import { reactive } from "vue";
interface LoginForm {
  email: string;
  password: string;
  rememberMe: boolean;
}
const form = reactive<LoginForm>({
  email: "",
  password: "",
  rememberMe: false,
});
const resetForm = (): void => {
  form.email = "";
  form.password = "";
  form.rememberMe = false;
};
const handleSubmit = (): void => {
  console.log("Submitting:", form);
};
</script>
<template>
  <form @submit.prevent="handleSubmit" class="space-y-4">
    <input
      v-model="form.email"
      type="email"
      placeholder="Email"
      class="input input-bordered w-full"
    />
    <input
      v-model="form.password"
      type="password"
      placeholder="Password"
      class="input input-bordered w-full"
    />
    <label class="flex items-center gap-2">
      <input v-model="form.rememberMe" type="checkbox" class="checkbox" />
      Remember me
    </label>
    <div class="flex gap-2">
      <button type="submit" class="btn btn-primary">Login</button>
      <button type="button" @click="resetForm" class="btn btn-ghost">Reset</button>
    </div>
  </form>
</template>
```
---
### Drill 2 (Variation): Shopping Cart with Array
Create a reactive shopping cart with add/remove functionality.
```vue
<script setup lang="ts">
import { reactive } from "vue";
// TODO: Define CartItem interface with: id, name, price, quantity
// TODO: Create reactive cart with items array and methods:
//   - addItem(item): add or increment quantity
//   - removeItem(id): remove item completely
//   - getTotal(): calculate total price
</script>
<template>
  <!-- TODO: Display cart items with quantity and price -->
  <!-- TODO: Show total at bottom -->
</template>
```
**Solution:**
```vue
<script setup lang="ts">
import { reactive } from "vue";
interface CartItem {
  id: number;
  name: string;
  price: number;
  quantity: number;
}
interface Cart {
  items: CartItem[];
}
const cart = reactive<Cart>({
  items: [],
});
const addItem = (item: Omit<CartItem, "quantity">): void => {
  const existing = cart.items.find((i) => i.id === item.id);
  if (existing) {
    existing.quantity++;
  } else {
    cart.items.push({ ...item, quantity: 1 });
  }
};
const removeItem = (id: number): void => {
  const index = cart.items.findIndex((i) => i.id === id);
  if (index > -1) {
    cart.items.splice(index, 1);
  }
};
const decrementItem = (id: number): void => {
  const item = cart.items.find((i) => i.id === id);
  if (item) {
    if (item.quantity > 1) {
      item.quantity--;
    } else {
      removeItem(id);
    }
  }
};
const getTotal = (): number => {
  return cart.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
};
const clearCart = (): void => {
  cart.items.splice(0, cart.items.length);
};
// Sample products for testing
const sampleProducts = [
  { id: 1, name: "Vue.js Book", price: 29.99 },
  { id: 2, name: "TypeScript Course", price: 49.99 },
  { id: 3, name: "Sticker Pack", price: 9.99 },
];
</script>
<template>
  <div class="max-w-md mx-auto">
    <h2 class="text-xl font-bold mb-4">Shopping Cart</h2>
    <!-- Products to Add -->
    <div class="mb-6">
      <h3 class="font-semibold mb-2">Products</h3>
      <div class="flex flex-wrap gap-2">
        <button
          v-for="product in sampleProducts"
          :key="product.id"
          @click="addItem(product)"
          class="btn btn-sm btn-outline"
        >
          Add {{ product.name }} (${{ product.price }})
        </button>
      </div>
    </div>
    <!-- Cart Items -->
    <div v-if="cart.items.length > 0" class="space-y-2">
      <div
        v-for="item in cart.items"
        :key="item.id"
        class="flex justify-between items-center p-2 bg-base-200 rounded"
      >
        <span>{{ item.name }} (x{{ item.quantity }})</span>
        <div class="flex items-center gap-2">
          <span>${{ (item.price * item.quantity).toFixed(2) }}</span>
          <button @click="decrementItem(item.id)" class="btn btn-xs">-</button>
          <button @click="addItem(item)" class="btn btn-xs">+</button>
          <button @click="removeItem(item.id)" class="btn btn-xs btn-error">×</button>
        </div>
      </div>
      <!-- Total -->
      <div class="text-right font-bold text-lg pt-2 border-t">
        Total: ${{ getTotal().toFixed(2) }}
      </div>
      <button @click="clearCart" class="btn btn-outline btn-error w-full">
        Clear Cart
      </button>
    </div>
    <p v-else class="text-center text-base-content/60">Cart is empty</p>
  </div>
</template>
```
---
### Drill 3 (Combination): Nested Settings with `toRefs()`
Use `toRefs()` to safely work with nested reactive objects.
```vue
<script setup lang="ts">
import { reactive, toRefs } from "vue";
// TODO: Create nested settings object with:
//   - display: { brightness, contrast, colorMode }
//   - audio: { volume, muted }
// TODO: Use toRefs to extract display and audio
// TODO: Create functions to modify settings
</script>
<template>
  <!-- TODO: Display and modify settings using the extracted refs -->
</template>
```
**Solution:**
```vue
<script setup lang="ts">
import { reactive, toRefs } from "vue";
interface DisplaySettings {
  brightness: number;
  contrast: number;
  colorMode: "light" | "dark" | "auto";
}
interface AudioSettings {
  volume: number;
  muted: boolean;
}
interface AppSettings {
  display: DisplaySettings;
  audio: AudioSettings;
}
const settings = reactive<AppSettings>({
  display: {
    brightness: 100,
    contrast: 50,
    colorMode: "auto",
  },
  audio: {
    volume: 80,
    muted: false,
  },
});
// Extract refs for cleaner template access
const { display, audio } = toRefs(settings);
// Display actions
const setBrightness = (value: number): void => {
  display.value.brightness = Math.max(0, Math.min(100, value));
};
const setColorMode = (mode: DisplaySettings["colorMode"]): void => {
  display.value.colorMode = mode;
};
// Audio actions
const setVolume = (value: number): void => {
  audio.value.volume = Math.max(0, Math.min(100, value));
  if (value > 0) {
    audio.value.muted = false;
  }
};
const toggleMute = (): void => {
  audio.value.muted = !audio.value.muted;
};
const resetToDefaults = (): void => {
  display.value.brightness = 100;
  display.value.contrast = 50;
  display.value.colorMode = "auto";
  audio.value.volume = 80;
  audio.value.muted = false;
};
</script>
<template>
  <div class="max-w-md mx-auto space-y-6">
    <h2 class="text-xl font-bold">Settings</h2>
    <!-- Display Settings -->
    <div class="card bg-base-200 p-4">
      <h3 class="font-semibold mb-3">Display</h3>
      <div class="space-y-3">
        <div>
          <label class="label">Brightness: {{ display.brightness }}%</label>
          <input
            type="range"
            v-model.number="display.brightness"
            min="0"
            max="100"
            class="range"
          />
        </div>
        <div>
          <label class="label">Contrast: {{ display.contrast }}%</label>
          <input
            type="range"
            v-model.number="display.contrast"
            min="0"
            max="100"
            class="range"
          />
        </div>
        <div>
          <label class="label">Color Mode</label>
          <div class="btn-group">
            <button
              v-for="mode in ['light', 'dark', 'auto'] as const"
              :key="mode"
              :class="['btn btn-sm', display.colorMode === mode ? 'btn-active' : '']"
              @click="setColorMode(mode)"
            >
              {{ mode }}
            </button>
          </div>
        </div>
      </div>
    </div>
    <!-- Audio Settings -->
    <div class="card bg-base-200 p-4">
      <h3 class="font-semibold mb-3">Audio</h3>
      <div class="space-y-3">
        <div>
          <label class="label">
            Volume: {{ audio.muted ? "Muted" : `${audio.volume}%` }}
          </label>
          <input
            type="range"
            v-model.number="audio.volume"
            min="0"
            max="100"
            :disabled="audio.muted"
            class="range"
          />
        </div>
        <button @click="toggleMute" class="btn btn-sm">
          {{ audio.muted ? "Unmute" : "Mute" }}
        </button>
      </div>
    </div>
    <button @click="resetToDefaults" class="btn btn-outline w-full">
      Reset to Defaults
    </button>
  </div>
</template>
```
---
### Drill 4 (Edge Case): Avoiding Reactivity Loss
Identify and fix reactivity issues in this code.
```vue
<script setup lang="ts">
import { reactive } from "vue";
// This code has reactivity problems — identify and fix them
const user = reactive({
  name: "Alice",
  age: 25,
});
// Problem 1: Destructuring
const { name, age } = user;
// Problem 2: Reassigning
let settings = reactive({ theme: "dark" });
const resetSettings = () => {
  settings = reactive({ theme: "light" });
};
// Problem 3: Passing primitive to function
const logAge = (age: number) => {
  console.log(`Age is: ${age}`);
};
// Called somewhere: logAge(user.age)
</script>
```
**Solution:**
```vue
<script setup lang="ts">
import { reactive, toRefs } from "vue";
const user = reactive({
  name: "Alice",
  age: 25,
});
// Fix 1: Use toRefs for safe destructuring
const { name, age } = toRefs(user);
// Now name and age are Refs that stay connected to user
// Fix 2: Don't reassign, mutate properties instead
const settings = reactive({ theme: "dark" as "dark" | "light" });
const resetSettings = () => {
  settings.theme = "light"; // Mutate, don't reassign
};
// Alternative Fix 2: Use ref if you need to replace entirely
import { ref } from "vue";
const settingsAlt = ref({ theme: "dark" as "dark" | "light" });
const resetSettingsAlt = () => {
  settingsAlt.value = { theme: "light" }; // This is fine with ref
};
// Fix 3: Pass the reactive object or use computed
const logAge = () => {
  console.log(`Age is: ${user.age}`); // Access from reactive object
};
// Or pass the entire user object
const logUserAge = (u: typeof user) => {
  console.log(`Age is: ${u.age}`);
};
</script>
<template>
  <div>
    <!-- With toRefs, access via .value in script but auto-unwrapped in template -->
    <p>Name: {{ name }}</p>
    <p>Age: {{ age }}</p>
    <p>Theme: {{ settings.theme }}</p>
    <button @click="resetSettings">Reset Theme</button>
  </div>
</template>
```
---
## 5. Common Pitfalls & Anti-Patterns
| ❌ Common Mistake                        | ✅ Correct Approach                      | Why It Matters                                      |
| :--------------------------------------- | :--------------------------------------- | :-------------------------------------------------- |
| Destructuring reactive object            | Use `toRefs()` before destructuring      | Plain destructuring creates non-reactive copies     |
| Reassigning entire reactive object       | Mutate properties with `Object.assign()` | Reassignment breaks the proxy connection            |
| Using `reactive()` for primitives        | Use `ref()` for primitives               | `reactive()` only works with objects                |
### Example of Each Mistake
**Mistake 1: Breaking Reactivity with Destructuring**
```vue
<script setup lang="ts">
import { reactive, toRefs } from "vue";
const state = reactive({ count: 0, name: "Test" });
// ❌ WRONG: These are now plain, non-reactive values
const { count, name } = state;
// ✅ CORRECT: Use toRefs to maintain reactivity
const { count: countRef, name: nameRef } = toRefs(state);
// Now countRef.value and nameRef.value are reactive
</script>
```
**Mistake 2: Reassigning the Reactive Object**
```vue
<script setup lang="ts">
import { reactive } from "vue";
let form = reactive({ email: "", password: "" });
const clearForm = () => {
  // ❌ WRONG: Creates new proxy, template loses connection
  form = reactive({ email: "", password: "" });
  
  // ✅ CORRECT: Mutate existing object
  form.email = "";
  form.password = "";
  
  // Or use Object.assign for multiple properties
  Object.assign(form, { email: "", password: "" });
};
</script>
```
**Mistake 3: Reactive with Primitives**
```vue
<script setup lang="ts">
import { reactive, ref } from "vue";
// ❌ WRONG: Primitives can't be made reactive
const count = reactive(0); // TypeScript error!
// ✅ CORRECT: Use ref for primitives
const count = ref(0);
// ✅ CORRECT: Or wrap in an object
const counter = reactive({ value: 0 });
</script>
```
---
## 6. Mini-Project: Temperature Converter Widget
Build a real-time temperature converter with reactive form state.
### File Structure
```
src/
├── components/
│   └── TemperatureConverter.vue
├── App.vue
└── main.ts
```
### TemperatureConverter.vue
```vue
<script setup lang="ts">
// ============================================
// IMPORTS
// ============================================
import { reactive, watch } from "vue";
// ============================================
// TYPES
// ============================================
type TemperatureUnit = "celsius" | "fahrenheit" | "kelvin";
interface TemperatureState {
  value: number;
  unit: TemperatureUnit;
  lastEdited: TemperatureUnit;
}
interface ConversionResult {
  celsius: number;
  fahrenheit: number;
  kelvin: number;
}
// ============================================
// REACTIVE STATE
// ============================================
const state = reactive<TemperatureState>({
  value: 20,
  unit: "celsius",
  lastEdited: "celsius",
});
const conversions = reactive<ConversionResult>({
  celsius: 20,
  fahrenheit: 68,
  kelvin: 293.15,
});
const history = reactive<{ from: string; to: string; timestamp: Date }[]>([]);
// ============================================
// CONVERSION FUNCTIONS
// ============================================
const celsiusToFahrenheit = (c: number): number => (c * 9) / 5 + 32;
const fahrenheitToCelsius = (f: number): number => ((f - 32) * 5) / 9;
const celsiusToKelvin = (c: number): number => c + 273.15;
const kelvinToCelsius = (k: number): number => k - 273.15;
const convertFromCelsius = (celsius: number): void => {
  conversions.celsius = celsius;
  conversions.fahrenheit = celsiusToFahrenheit(celsius);
  conversions.kelvin = celsiusToKelvin(celsius);
};
const convertFromFahrenheit = (fahrenheit: number): void => {
  const celsius = fahrenheitToCelsius(fahrenheit);
  conversions.celsius = celsius;
  conversions.fahrenheit = fahrenheit;
  conversions.kelvin = celsiusToKelvin(celsius);
};
const convertFromKelvin = (kelvin: number): void => {
  const celsius = kelvinToCelsius(kelvin);
  conversions.celsius = celsius;
  conversions.fahrenheit = celsiusToFahrenheit(celsius);
  conversions.kelvin = kelvin;
};
// ============================================
// ACTIONS
// ============================================
const updateTemperature = (value: number, unit: TemperatureUnit): void => {
  state.value = value;
  state.unit = unit;
  state.lastEdited = unit;
  switch (unit) {
    case "celsius":
      convertFromCelsius(value);
      break;
    case "fahrenheit":
      convertFromFahrenheit(value);
      break;
    case "kelvin":
      convertFromKelvin(value);
      break;
  }
  // Add to history
  history.unshift({
    from: `${value.toFixed(1)}° ${unit.charAt(0).toUpperCase()}`,
    to: `${conversions.celsius.toFixed(1)}°C / ${conversions.fahrenheit.toFixed(1)}°F / ${conversions.kelvin.toFixed(1)}K`,
    timestamp: new Date(),
  });
  // Keep only last 5 entries
  if (history.length > 5) {
    history.pop();
  }
};
const setPreset = (celsius: number, name: string): void => {
  updateTemperature(celsius, "celsius");
};
const clearHistory = (): void => {
  history.splice(0, history.length);
};
// ============================================
// HELPERS
// ============================================
const formatNumber = (n: number): string => n.toFixed(2);
const getTemperatureClass = (celsius: number): string => {
  if (celsius <= 0) return "text-blue-500";
  if (celsius <= 15) return "text-cyan-500";
  if (celsius <= 25) return "text-green-500";
  if (celsius <= 35) return "text-orange-500";
  return "text-red-500";
};
// ============================================
// PRESETS
// ============================================
const presets = [
  { name: "Freezing", celsius: 0 },
  { name: "Room Temp", celsius: 20 },
  { name: "Body Temp", celsius: 37 },
  { name: "Boiling", celsius: 100 },
];
</script>
<template>
  <div class="card bg-base-100 shadow-xl max-w-lg mx-auto">
    <div class="card-body">
      <h2 class="card-title justify-center text-2xl mb-4">
        🌡️ Temperature Converter
      </h2>
      <!-- Temperature Display -->
      <div class="text-center mb-6">
        <p
          class="text-5xl font-bold"
          :class="getTemperatureClass(conversions.celsius)"
        >
          {{ formatNumber(conversions.celsius) }}°C
        </p>
        <p class="text-base-content/60 mt-2">
          {{ formatNumber(conversions.fahrenheit) }}°F |
          {{ formatNumber(conversions.kelvin) }}K
        </p>
      </div>
      <!-- Input Fields -->
      <div class="grid grid-cols-3 gap-4 mb-6">
        <div class="form-control">
          <label class="label">
            <span class="label-text">Celsius</span>
          </label>
          <input
            type="number"
            :value="conversions.celsius.toFixed(2)"
            @input="updateTemperature(Number(($event.target as HTMLInputElement).value), 'celsius')"
            class="input input-bordered"
            step="0.1"
          />
        </div>
        <div class="form-control">
          <label class="label">
            <span class="label-text">Fahrenheit</span>
          </label>
          <input
            type="number"
            :value="conversions.fahrenheit.toFixed(2)"
            @input="updateTemperature(Number(($event.target as HTMLInputElement).value), 'fahrenheit')"
            class="input input-bordered"
            step="0.1"
          />
        </div>
        <div class="form-control">
          <label class="label">
            <span class="label-text">Kelvin</span>
          </label>
          <input
            type="number"
            :value="conversions.kelvin.toFixed(2)"
            @input="updateTemperature(Number(($event.target as HTMLInputElement).value), 'kelvin')"
            class="input input-bordered"
            step="0.1"
          />
        </div>
      </div>
      <!-- Presets -->
      <div class="mb-6">
        <p class="text-sm font-semibold mb-2">Quick Presets</p>
        <div class="flex flex-wrap gap-2">
          <button
            v-for="preset in presets"
            :key="preset.name"
            @click="setPreset(preset.celsius, preset.name)"
            class="btn btn-sm btn-outline"
          >
            {{ preset.name }} ({{ preset.celsius }}°C)
          </button>
        </div>
      </div>
      <!-- History -->
      <div v-if="history.length > 0">
        <div class="flex justify-between items-center mb-2">
          <p class="text-sm font-semibold">Recent Conversions</p>
          <button @click="clearHistory" class="btn btn-xs btn-ghost">
            Clear
          </button>
        </div>
        <div class="space-y-1">
          <div
            v-for="(entry, index) in history"
            :key="index"
            class="text-xs bg-base-200 p-2 rounded flex justify-between"
          >
            <span>{{ entry.from }} → {{ entry.to }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<style scoped>
input[type="number"] {
  -moz-appearance: textfield;
}
input[type="number"]::-webkit-outer-spin-button,
input[type="number"]::-webkit-inner-spin-button {
  -webkit-appearance: none;
}
</style>
```
### App.vue
```vue
<script setup lang="ts">
import TemperatureConverter from "./components/TemperatureConverter.vue";
</script>
<template>
  <div class="min-h-screen bg-base-200 py-10 px-4">
    <TemperatureConverter />
  </div>
</template>
```
### How to Test
1. Run `npm run dev`
2. Open the browser
3. Verify:
   - Entering a value in any input updates all three
   - Preset buttons set the temperature correctly
   - History tracks recent conversions
   - Temperature color changes based on value
   - Clear button removes history
---
## 7. Implementation Exercises (Progressive)
### Level 1 (Recall): Fill in the Blanks
```vue
<script setup lang="ts">
import { _____, toRefs } from "vue";
interface Product {
  name: string;
  price: number;
  quantity: number;
}
const product = _____<Product>({
  name: "Widget",
  price: 29.99,
  quantity: 10,
});
const { name, price, quantity } = _____(product);
const decreaseStock = (): void => {
  if (quantity._____ > 0) {
    quantity._____--;
  }
};
</script>
<template>
  <div>
    <h2>{{ _____ }}</h2>
    <p>Price: ${{ price }}</p>
    <p>In Stock: {{ _____ }}</p>
    <button @click="_____">Sell One</button>
  </div>
</template>
```
**Answer:**
```vue
<script setup lang="ts">
import { reactive, toRefs } from "vue";
interface Product {
  name: string;
  price: number;
  quantity: number;
}
const product = reactive<Product>({
  name: "Widget",
  price: 29.99,
  quantity: 10,
});
const { name, price, quantity } = toRefs(product);
const decreaseStock = (): void => {
  if (quantity.value > 0) {
    quantity.value--;
  }
};
</script>
<template>
  <div>
    <h2>{{ name }}</h2>
    <p>Price: ${{ price }}</p>
    <p>In Stock: {{ quantity }}</p>
    <button @click="decreaseStock">Sell One</button>
  </div>
</template>
```
---
### Level 2 (Apply): Independent Coding Tasks
**Exercise 2.1: Contact Form**
Create a contact form component with:
- Reactive object with: name, email, subject, message
- Character counter for message (max 500)
- Submit handler that logs the form data
- Reset button that clears all fields
**Exercise 2.2: Todo List**
Create a todo list with:
- Reactive array of todo items (id, text, completed)
- Add todo functionality
- Toggle completion status
- Delete individual todos
- Filter options (all, active, completed)
---
### Level 3 (Debug): Find and Fix the Bugs
This component has **4 bugs** related to reactivity. Find and fix them:
```vue
<script setup lang="ts">
import { reactive } from "vue";
let config = reactive({
  theme: "light",
  fontSize: 16,
  language: "en",
});
const { theme, fontSize } = config;
const setTheme = (newTheme: string) => {
  theme = newTheme;
};
const increaseFontSize = () => {
  fontSize++;
};
const resetConfig = () => {
  config = reactive({
    theme: "light",
    fontSize: 16,
    language: "en",
  });
};
</script>
<template>
  <div>
    <p>Theme: {{ theme }}</p>
    <p>Font Size: {{ fontSize }}px</p>
    <button @click="setTheme('dark')">Dark Mode</button>
    <button @click="increaseFontSize">Larger Text</button>
    <button @click="resetConfig">Reset</button>
  </div>
</template>
```
**Bugs:**
1. Destructuring `config` creates non-reactive plain values
2. `theme = newTheme` tries to reassign a const (and wouldn't be reactive anyway)
3. `fontSize++` modifies a non-reactive copy
4. `config = reactive({...})` reassigns the reactive object, breaking template binding
**Corrected Code:**
```vue
<script setup lang="ts">
import { reactive, toRefs } from "vue";
const config = reactive({
  theme: "light" as "light" | "dark",
  fontSize: 16,
  language: "en",
});
// Use toRefs for safe destructuring
const { theme, fontSize } = toRefs(config);
const setTheme = (newTheme: "light" | "dark") => {
  config.theme = newTheme; // Modify original reactive object
  // Or: theme.value = newTheme; (since we used toRefs)
};
const increaseFontSize = () => {
  config.fontSize++; // Modify original reactive object
  // Or: fontSize.value++; (since we used toRefs)
};
const resetConfig = () => {
  // Mutate properties, don't reassign
  Object.assign(config, {
    theme: "light",
    fontSize: 16,
    language: "en",
  });
};
</script>
<template>
  <div>
    <p>Theme: {{ config.theme }}</p>
    <p>Font Size: {{ config.fontSize }}px</p>
    <button @click="setTheme('dark')">Dark Mode</button>
    <button @click="increaseFontSize">Larger Text</button>
    <button @click="resetConfig">Reset</button>
  </div>
</template>
```
---
**Reply 'next' for Lesson 4.1 (Theory).**
