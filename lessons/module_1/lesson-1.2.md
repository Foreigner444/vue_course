# Lesson 1.2: Your First Component — Practice & Application (Ваш первый компонент)

## 1. Lesson Metadata

| Field              | Value                                                                                       |
| :----------------- | :------------------------------------------------------------------------------------------ |
| **Lesson Number**  | 1.2 of 25 (Part 2 of 2)                                                                     |
| **Topic**          | Your First Component: Template + Script Setup / Ваш первый компонент: Шаблон + Script Setup |
| **Continues From** | Lesson 1.1                                                                                  |
| **Duration**       | 30-35 minutes                                                                               |

**Learning Objectives — Part 2 (Practice):**

1. **Apply:** Create a complete Single File Component with template, script, and styles
2. **Analyze:** Compare different ways to structure component data and identify best practices

---

## 2. Quick Recap

- A **Single File Component (SFC)** contains three sections: `<script setup>`, `<template>`, and `<style scoped>`
- **`<script setup lang="ts">`** automatically exposes all top-level variables to the template
- Use **`{{ variableName }}`** (double curly braces) to display JavaScript values in the template
- **`scoped`** styles only affect the current component, preventing CSS leaks

---

## 3. The Variable Frame & Complexity Scale

**Basic Form:**

```vue
<script setup lang="ts">
// Simple string variable
const message: string = "Hello, Vue!";
</script>

<template>
  <p>{{ message }}</p>
</template>
```

**With Options:**

```vue
<script setup lang="ts">
// Multiple variables of different types
const title: string = "Welcome";
const count: number = 42;
const isActive: boolean = true;
const items: string[] = ["Apple", "Banana", "Cherry"];
</script>

<template>
  <div>
    <h1>{{ title }}</h1>
    <p>Count: {{ count }}</p>
    <p>Status: {{ isActive ? "Active" : "Inactive" }}</p>
    <p>First item: {{ items[0] }}</p>
  </div>
</template>
```

**Advanced Form:**

```vue
<script setup lang="ts">
// Interface for type safety
interface User {
  id: number;
  name: string;
  email: string;
  avatar: string;
}

// Object with defined type
const user: User = {
  id: 1,
  name: "Sarah Chen",
  email: "sarah@example.com",
  avatar: "https://api.dicebear.com/7.x/avataaars/svg?seed=sarah",
};

// Helper function (also auto-exposed to template)
const formatEmail = (email: string): string => {
  return email.toLowerCase();
};
</script>

<template>
  <div class="user-card">
    <img :src="user.avatar" :alt="user.name" />
    <h2>{{ user.name }}</h2>
    <p>{{ formatEmail(user.email) }}</p>
  </div>
</template>
```

---

## 4. Micro-Implementation Drills

**Drill 1 (Basic): Simple Greeting Component**

Create a component that displays a greeting message.

```vue
<script setup lang="ts">
// TODO: Declare a constant 'greeting' with the value "Welcome to Vue 3!"
</script>

<template>
  <!-- TODO: Display the greeting inside an <h1> tag -->
</template>
```

**Solution:**

```vue
<script setup lang="ts">
const greeting: string = "Welcome to Vue 3!";
</script>

<template>
  <h1>{{ greeting }}</h1>
</template>
```

---

**Drill 2 (Variation): Product Info Display**

Create a component that displays product information with multiple data types.

```vue
<script setup lang="ts">
// TODO: Create variables for:
// - productName (string): "Wireless Headphones"
// - price (number): 79.99
// - inStock (boolean): true
</script>

<template>
  <div>
    <!-- TODO: Display product name in <h2> -->
    <!-- TODO: Display price formatted as "$79.99" -->
    <!-- TODO: Display "In Stock" or "Out of Stock" based on inStock -->
  </div>
</template>
```

**Solution:**

```vue
<script setup lang="ts">
const productName: string = "Wireless Headphones";
const price: number = 79.99;
const inStock: boolean = true;
</script>

<template>
  <div>
    <h2>{{ productName }}</h2>
    <p>${{ price }}</p>
    <p>{{ inStock ? "In Stock" : "Out of Stock" }}</p>
  </div>
</template>
```

---

**Drill 3 (Combination): User Profile with Helper Function**

Combine object data with a helper function.

```vue
<script setup lang="ts">
// TODO: Define an interface 'Profile' with: name (string), joinDate (string)
// TODO: Create a profile object with your own data
// TODO: Create a function 'formatDate' that takes a date string and returns "Joined: {date}"
</script>

<template>
  <div>
    <!-- TODO: Display the profile name -->
    <!-- TODO: Use formatDate to display the join date -->
  </div>
</template>
```

**Solution:**

```vue
<script setup lang="ts">
interface Profile {
  name: string;
  joinDate: string;
}

const profile: Profile = {
  name: "Alex Johnson",
  joinDate: "January 2024",
};

const formatDate = (date: string): string => {
  return `Joined: ${date}`;
};
</script>

<template>
  <div>
    <h2>{{ profile.name }}</h2>
    <p>{{ formatDate(profile.joinDate) }}</p>
  </div>
</template>
```

---

**Drill 4 (Edge Case): Handling Empty/Null Values**

Handle cases where data might be missing.

```vue
<script setup lang="ts">
interface Article {
  title: string;
  author: string | null;
  readTime: number | undefined;
}

// TODO: Create an article with a null author and undefined readTime
// TODO: Create a function 'getAuthorDisplay' that returns "Unknown Author" if author is null
</script>

<template>
  <article>
    <!-- TODO: Display title -->
    <!-- TODO: Display author using the helper function -->
    <!-- TODO: Display read time with fallback "N/A" if undefined -->
  </article>
</template>
```

**Solution:**

```vue
<script setup lang="ts">
interface Article {
  title: string;
  author: string | null;
  readTime: number | undefined;
}

const article: Article = {
  title: "Getting Started with Vue 3",
  author: null,
  readTime: undefined,
};

const getAuthorDisplay = (author: string | null): string => {
  return author ?? "Unknown Author";
};
</script>

<template>
  <article>
    <h1>{{ article.title }}</h1>
    <p>By: {{ getAuthorDisplay(article.author) }}</p>
    <p>Read time: {{ article.readTime ?? "N/A" }} min</p>
  </article>
</template>
```

---

## 5. Common Pitfalls & Anti-Patterns

| ❌ Common Mistake                                         | ✅ Correct Approach                                       | Why It Matters                                          |
| :-------------------------------------------------------- | :-------------------------------------------------------- | :------------------------------------------------------ |
| Using Options API syntax (`export default { data() {} }`) | Use `<script setup>` with top-level variables             | Options API is verbose and has worse TypeScript support |
| Forgetting `lang="ts"` on script tag                      | Always add `<script setup lang="ts">`                     | Loses type checking and IDE support                     |
| Using `var` instead of `const`/`let`                      | Use `const` for values that don't change                  | `var` has scoping issues; `const` signals intent        |
| Accessing undefined object properties without checks      | Use optional chaining (`?.`) or nullish coalescing (`??`) | Prevents runtime errors when data is missing            |

---

## 6. Mini-Project: About Me Card

**File Structure:**

```
src/
├── components/
│   └── AboutCard.vue
├── App.vue
└── main.ts
```

**components/AboutCard.vue:**

```vue
<script setup lang="ts">
// ============================================
// ABOUT CARD COMPONENT
// A reusable card displaying personal information
// ============================================

// Define the shape of our data with TypeScript interface
// This helps catch errors and provides autocomplete
interface SocialLink {
  platform: string;
  url: string;
  icon: string;
}

interface PersonInfo {
  name: string;
  title: string;
  bio: string;
  avatarUrl: string;
  socialLinks: SocialLink[];
}

// Component data - these are static values for now
// In future lessons, we'll learn to make these dynamic with props
const person: PersonInfo = {
  name: "Sarah Chen",
  title: "Senior Frontend Developer",
  bio: "Passionate about building beautiful, accessible user interfaces. Vue.js enthusiast with 5+ years of experience.",
  avatarUrl: "https://api.dicebear.com/7.x/avataaars/svg?seed=sarah",
  socialLinks: [
    { platform: "GitHub", url: "https://github.com", icon: "🐙" },
    { platform: "LinkedIn", url: "https://linkedin.com", icon: "💼" },
    { platform: "Twitter", url: "https://twitter.com", icon: "🐦" },
  ],
};

// Helper function to truncate bio if too long
// Functions defined here are automatically available in the template
const truncateBio = (text: string, maxLength: number = 100): string => {
  if (text.length <= maxLength) return text;
  return text.slice(0, maxLength).trim() + "...";
};

// Get initials from name for fallback avatar
const getInitials = (name: string): string => {
  return name
    .split(" ")
    .map((word) => word[0])
    .join("")
    .toUpperCase();
};
</script>

<template>
  <!-- Main card container using Tailwind CSS classes -->
  <div class="max-w-sm mx-auto bg-white rounded-xl shadow-lg overflow-hidden">
    <!-- Avatar section -->
    <div class="flex justify-center pt-6">
      <img
        :src="person.avatarUrl"
        :alt="`${person.name}'s avatar`"
        class="w-24 h-24 rounded-full border-4 border-blue-100"
      />
    </div>

    <!-- Content section -->
    <div class="p-6 text-center">
      <!-- Name - using interpolation {{ }} to display the value -->
      <h2 class="text-xl font-bold text-gray-800">
        {{ person.name }}
      </h2>

      <!-- Title - styled differently from name -->
      <p class="text-blue-600 font-medium mt-1">
        {{ person.title }}
      </p>

      <!-- Bio - using our helper function to potentially truncate -->
      <p class="text-gray-600 mt-3 text-sm">
        {{ truncateBio(person.bio, 150) }}
      </p>

      <!-- Social links - we'll learn v-for in a later lesson -->
      <!-- For now, showing them manually -->
      <div class="flex justify-center gap-4 mt-4">
        <a
          v-for="link in person.socialLinks"
          :key="link.platform"
          :href="link.url"
          :title="link.platform"
          class="text-2xl hover:scale-110 transition-transform"
          target="_blank"
          rel="noopener noreferrer"
        >
          {{ link.icon }}
        </a>
      </div>
    </div>

    <!-- Footer with initials badge -->
    <div class="bg-gray-50 px-6 py-3 text-center">
      <span
        class="inline-block bg-blue-100 text-blue-800 text-xs font-semibold px-2.5 py-0.5 rounded"
      >
        {{ getInitials(person.name) }}
      </span>
    </div>
  </div>
</template>

<style scoped>
/* Additional custom styles if needed beyond Tailwind */
/* The 'scoped' attribute ensures these styles only apply to this component */

/* Example: Custom hover effect for the card */
.max-w-sm {
  transition: transform 0.2s ease-in-out;
}

.max-w-sm:hover {
  transform: translateY(-4px);
}
</style>
```

**App.vue:**

```vue
<script setup lang="ts">
// Import the AboutCard component
// Vue components are imported just like any other JavaScript module
import AboutCard from "./components/AboutCard.vue";
</script>

<template>
  <!-- Use Tailwind classes for page layout -->
  <div class="min-h-screen bg-gray-100 py-12">
    <div class="container mx-auto">
      <h1 class="text-3xl font-bold text-center text-gray-800 mb-8">
        Team Members
      </h1>

      <!-- Use the component like a custom HTML element -->
      <!-- Self-closing syntax when there's no content inside -->
      <AboutCard />
    </div>
  </div>
</template>
```

**main.ts:**

```typescript
// Entry point of the Vue application
import { createApp } from "vue";
import App from "./App.vue";

// Import Tailwind CSS (assuming it's configured in the project)
import "./style.css";

// Create and mount the Vue application
// This connects Vue to the #app element in index.html
createApp(App).mount("#app");
```

**How to Test:**

1. Create a new Vue project: `npm create vite@latest my-project -- --template vue-ts`
2. Navigate to project: `cd my-project`
3. Install dependencies: `npm install`
4. Install Tailwind CSS: `npm install -D tailwindcss postcss autoprefixer`
5. Initialize Tailwind: `npx tailwindcss init -p`
6. Add Tailwind to `src/style.css`:
   ```css
   @tailwind base;
   @tailwind components;
   @tailwind utilities;
   ```
7. Configure `tailwind.config.js`:
   ```javascript
   export default {
     content: ["./index.html", "./src/**/*.{vue,js,ts,jsx,tsx}"],
     theme: { extend: {} },
     plugins: [],
   };
   ```
8. Replace `src/components/` and `src/App.vue` with the code above
9. Run: `npm run dev`
10. Open `http://localhost:5173` in your browser

---

## 7. Implementation Exercises (Progressive)

### Level 1 (Recall): Fill in the Blanks

```vue
<script setup ______="ts">
const title: ______ = "My First Vue App";
const version: number = ______;
</script>

<template>
  <div>
    <h1>{{ ______ }}</h1>
    <p>Version: {{ version }}</p>
  </div>
</template>
```

**Answer:** `lang`, `string`, `3`, `title`

---

### Level 2 (Apply): Small Coding Tasks

**Exercise 2.1:** Create a `BookCard.vue` component that displays:

- Book title (string)
- Author name (string)
- Page count (number)
- A function that returns "Quick Read" if pages < 200, "Standard" if 200-400, "Long Read" if > 400

**Exercise 2.2:** Create a `WeatherWidget.vue` component that displays:

- City name (string)
- Temperature (number)
- Weather condition (string: "sunny", "cloudy", "rainy")
- A function that returns an emoji based on the condition (☀️, ☁️, 🌧️)

---

### Level 3 (Debug): Fix the Broken Code

```vue
<script lang="ts">
export default {
  data() {
    return {
      username: "john_doe",
      email: "john@example.com",
    };
  },
};
</script>

<template>
  <div>
    <p>Username: { username }</p>
    <p>Email: {{ this.email }}</p>
  </div>
</template>
```

**Bugs to find:**

1. Using Options API instead of `<script setup>`
2. Missing `setup` in script tag
3. Single curly braces instead of double
4. Using `this.` in template (not needed in Vue 3 templates)

**Fixed Version:**

```vue
<script setup lang="ts">
const username: string = "john_doe";
const email: string = "john@example.com";
</script>

<template>
  <div>
    <p>Username: {{ username }}</p>
    <p>Email: {{ email }}</p>
  </div>
</template>
```

---

## Self-Check Solutions

**Exercise 2.1 Solution:**

```vue
<script setup lang="ts">
interface Book {
  title: string;
  author: string;
  pages: number;
}

const book: Book = {
  title: "The Vue.js Handbook",
  author: "Jane Developer",
  pages: 350,
};

const getReadingCategory = (pages: number): string => {
  if (pages < 200) return "Quick Read";
  if (pages <= 400) return "Standard";
  return "Long Read";
};
</script>

<template>
  <div class="p-4 border rounded-lg">
    <h2 class="text-lg font-bold">{{ book.title }}</h2>
    <p class="text-gray-600">by {{ book.author }}</p>
    <p>{{ book.pages }} pages</p>
    <span class="badge">{{ getReadingCategory(book.pages) }}</span>
  </div>
</template>
```

**Exercise 2.2 Solution:**

```vue
<script setup lang="ts">
type WeatherCondition = "sunny" | "cloudy" | "rainy";

interface Weather {
  city: string;
  temperature: number;
  condition: WeatherCondition;
}

const weather: Weather = {
  city: "San Francisco",
  temperature: 18,
  condition: "cloudy",
};

const getWeatherEmoji = (condition: WeatherCondition): string => {
  const emojis: Record<WeatherCondition, string> = {
    sunny: "☀️",
    cloudy: "☁️",
    rainy: "🌧️",
  };
  return emojis[condition];
};
</script>

<template>
  <div class="p-4 bg-blue-50 rounded-lg">
    <h2 class="font-bold">{{ weather.city }}</h2>
    <p class="text-4xl">{{ getWeatherEmoji(weather.condition) }}</p>
    <p>{{ weather.temperature }}°C</p>
    <p class="capitalize">{{ weather.condition }}</p>
  </div>
</template>
```

---

_Reply 'next' for Lesson 2.1 (Theory)._
