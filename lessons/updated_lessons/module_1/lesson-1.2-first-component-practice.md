# Lesson 1.2: Your First Component — Practice & Application (Ваш первый компонент)
---
## 1. Lesson Metadata
| Field              | Value                                                                 |
| :----------------- | :-------------------------------------------------------------------- |
| **Lesson Number**  | 1.2 of 28 (Part 2 of 2)                                               |
| **Topic**          | Your First Component: Template + Script Setup / Ваш первый компонент  |
| **Continues From** | Lesson 1.1                                                            |
| **Duration**       | 30-35 minutes                                                         |
**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Create a complete Vue component with data and template interpolation
2. **Analyze:** Compare different ways to structure component data and identify best practices
---
## 2. Quick Recap
- **Single File Components (SFCs)** bundle template, script, and style in one `.vue` file
- **`<script setup lang="ts">`** automatically exposes all top-level variables and functions to the template
- **Template interpolation** uses `{{ expression }}` to display JavaScript values in HTML
---
## 3. The Variable Frame & Complexity Scale
### Basic Form: Simple Constants
```vue
<script setup lang="ts">
const title: string = "My App";
</script>
<template>
  <h1>{{ title }}</h1>
</template>
```
### With Options: Multiple Data Types
```vue
<script setup lang="ts">
const appName: string = "TaskMaster";
const version: number = 1.0;
const isProduction: boolean = true;
const features: string[] = ["Tasks", "Projects", "Teams"];
</script>
<template>
  <div>
    <h1>{{ appName }} v{{ version }}</h1>
    <p>Production: {{ isProduction ? "Yes" : "No" }}</p>
    <p>Features: {{ features.join(", ") }}</p>
  </div>
</template>
```
### Advanced Form: Objects and Functions
```vue
<script setup lang="ts">
interface User {
  name: string;
  email: string;
  role: "admin" | "user";
}
const user: User = {
  name: "Alice",
  email: "alice@example.com",
  role: "admin",
};
const formatUser = (u: User): string => {
  return `${u.name} (${u.role}) - ${u.email}`;
};
const getInitials = (name: string): string => {
  return name
    .split(" ")
    .map((n) => n[0])
    .join("")
    .toUpperCase();
};
</script>
<template>
  <div class="user-card">
    <div class="avatar">{{ getInitials(user.name) }}</div>
    <h2>{{ user.name }}</h2>
    <p>{{ user.email }}</p>
    <span class="badge">{{ user.role }}</span>
    <footer>{{ formatUser(user) }}</footer>
  </div>
</template>
```
---
## 4. Micro-Implementation Drills
### Drill 1 (Basic): Simple Greeting
Create a component that displays a personalized greeting.
```vue
<script setup lang="ts">
// TODO: Define a constant `name` with your name as a string
// TODO: Define a constant `language` with value "Vue 3"
</script>
<template>
  <div>
    <!-- TODO: Display "Hello, [name]!" using interpolation -->
    <!-- TODO: Display "I'm learning [language]" using interpolation -->
  </div>
</template>
```
**Solution:**
```vue
<script setup lang="ts">
const name: string = "Maria";
const language: string = "Vue 3";
</script>
<template>
  <div>
    <h1>Hello, {{ name }}!</h1>
    <p>I'm learning {{ language }}</p>
  </div>
</template>
```
---
### Drill 2 (Variation): Product Display
Create a component that displays product information with different data types.
```vue
<script setup lang="ts">
// TODO: Define a product object with: name (string), price (number), inStock (boolean)
</script>
<template>
  <div class="product">
    <!-- TODO: Display product name in an h2 -->
    <!-- TODO: Display price formatted as "$XX.XX" -->
    <!-- TODO: Display "In Stock" or "Out of Stock" based on inStock boolean -->
  </div>
</template>
```
**Solution:**
```vue
<script setup lang="ts">
interface Product {
  name: string;
  price: number;
  inStock: boolean;
}
const product: Product = {
  name: "Wireless Headphones",
  price: 79.99,
  inStock: true,
};
</script>
<template>
  <div class="product">
    <h2>{{ product.name }}</h2>
    <p class="price">${{ product.price.toFixed(2) }}</p>
    <p class="status">{{ product.inStock ? "✅ In Stock" : "❌ Out of Stock" }}</p>
  </div>
</template>
```
---
### Drill 3 (Combination): Function + Data
Create a component that uses both data and a helper function.
```vue
<script setup lang="ts">
// TODO: Define an array of strings called `tasks`
// TODO: Create a function `countTasks` that returns the number of tasks
// TODO: Create a function `getFirstTask` that returns the first task or "No tasks"
</script>
<template>
  <div>
    <!-- TODO: Display "You have X tasks" using the function -->
    <!-- TODO: Display "First task: [task]" using the function -->
    <!-- TODO: Display all tasks separated by commas -->
  </div>
</template>
```
**Solution:**
```vue
<script setup lang="ts">
const tasks: string[] = ["Learn Vue", "Build a project", "Deploy to production"];
const countTasks = (): number => {
  return tasks.length;
};
const getFirstTask = (): string => {
  return tasks.length > 0 ? tasks[0] : "No tasks";
};
</script>
<template>
  <div>
    <h2>Task Summary</h2>
    <p>You have {{ countTasks() }} tasks</p>
    <p>First task: {{ getFirstTask() }}</p>
    <p>All tasks: {{ tasks.join(", ") }}</p>
  </div>
</template>
```
---
### Drill 4 (Edge Case): Handling Empty/Null Values
Create a component that gracefully handles potentially missing data.
```vue
<script setup lang="ts">
// TODO: Define a user object where `nickname` might be undefined
// TODO: Create a function that returns the nickname or a default value
</script>
<template>
  <div>
    <!-- TODO: Display the user's name -->
    <!-- TODO: Display nickname with a fallback to "No nickname set" -->
    <!-- TODO: Use optional chaining for a potentially undefined property -->
  </div>
</template>
```
**Solution:**
```vue
<script setup lang="ts">
interface UserProfile {
  name: string;
  nickname?: string;
  socialLinks?: {
    twitter?: string;
  };
}
const user: UserProfile = {
  name: "John Doe",
  // nickname is intentionally undefined
  socialLinks: {
    // twitter is undefined
  },
};
const getDisplayName = (u: UserProfile): string => {
  return u.nickname ?? u.name;
};
</script>
<template>
  <div>
    <h2>{{ user.name }}</h2>
    <p>Nickname: {{ user.nickname ?? "No nickname set" }}</p>
    <p>Display as: {{ getDisplayName(user) }}</p>
    <p>Twitter: {{ user.socialLinks?.twitter ?? "Not connected" }}</p>
  </div>
</template>
```
---
## 5. Common Pitfalls & Anti-Patterns
| ❌ Common Mistake                              | ✅ Correct Approach                           | Why It Matters                                    |
| :--------------------------------------------- | :-------------------------------------------- | :------------------------------------------------ |
| Using Options API syntax in `<script setup>`   | Use direct variable declarations              | `<script setup>` is Composition API only          |
| Forgetting `lang="ts"` for TypeScript          | Always add `lang="ts"` to script tag          | Without it, TypeScript features won't work        |
| Calling functions without `()` in template     | Use `{{ myFunction() }}` for function calls   | Without `()`, you display the function itself     |
### Example of Each Mistake
**Mistake 1: Options API in Script Setup**
```vue
<!-- ❌ WRONG: Options API syntax -->
<script setup lang="ts">
export default {
  data() {
    return { message: "Hello" };
  },
};
</script>
<!-- ✅ CORRECT: Composition API with script setup -->
<script setup lang="ts">
const message: string = "Hello";
</script>
```
**Mistake 2: Missing TypeScript Declaration**
```vue
<!-- ❌ WRONG: No lang="ts" -->
<script setup>
const count: number = 0; // TypeScript syntax won't work!
</script>
<!-- ✅ CORRECT: Include lang="ts" -->
<script setup lang="ts">
const count: number = 0;
</script>
```
**Mistake 3: Forgetting Function Parentheses**
```vue
<script setup lang="ts">
const getMessage = (): string => "Hello World";
</script>
<template>
  <!-- ❌ WRONG: Displays the function object -->
  <p>{{ getMessage }}</p>
  <!-- ✅ CORRECT: Calls the function and displays result -->
  <p>{{ getMessage() }}</p>
</template>
```
---
## 6. Mini-Project: Profile Card Component
Build a complete profile card component that displays user information.
### File Structure
```
src/
├── components/
│   └── ProfileCard.vue
├── App.vue
└── main.ts
```
### ProfileCard.vue
```vue
<script setup lang="ts">
// ============================================
// TYPE DEFINITIONS
// ============================================
// Define the shape of our user data with TypeScript interface
interface SocialLinks {
  github?: string;
  twitter?: string;
  linkedin?: string;
}
interface UserProfile {
  name: string;
  title: string;
  bio: string;
  avatar: string;
  location: string;
  skills: string[];
  social: SocialLinks;
}
// ============================================
// COMPONENT DATA
// ============================================
// Define the user profile data
// In a real app, this might come from props or an API
const user: UserProfile = {
  name: "Sarah Chen",
  title: "Senior Frontend Developer",
  bio: "Passionate about building beautiful, accessible web applications with Vue.js and TypeScript.",
  avatar: "https://api.dicebear.com/7.x/avataaars/svg?seed=Sarah",
  location: "San Francisco, CA",
  skills: ["Vue.js", "TypeScript", "Tailwind CSS", "Node.js"],
  social: {
    github: "sarahchen",
    twitter: "sarahcodes",
    linkedin: "sarahchen-dev",
  },
};
// ============================================
// HELPER FUNCTIONS
// ============================================
// Get user's initials for fallback avatar
const getInitials = (name: string): string => {
  return name
    .split(" ")
    .map((part) => part.charAt(0))
    .join("")
    .toUpperCase();
};
// Format skills as a readable string
const formatSkills = (skills: string[]): string => {
  if (skills.length === 0) return "No skills listed";
  if (skills.length === 1) return skills[0];
  
  const lastSkill = skills[skills.length - 1];
  const otherSkills = skills.slice(0, -1);
  return `${otherSkills.join(", ")} & ${lastSkill}`;
};
// Get social media URL from handle
const getSocialUrl = (platform: keyof SocialLinks, handle?: string): string => {
  if (!handle) return "#";
  
  const baseUrls: Record<keyof SocialLinks, string> = {
    github: "https://github.com/",
    twitter: "https://twitter.com/",
    linkedin: "https://linkedin.com/in/",
  };
  
  return `${baseUrls[platform]}${handle}`;
};
// Check if any social links exist
const hasSocialLinks = (): boolean => {
  return !!(user.social.github || user.social.twitter || user.social.linkedin);
};
</script>
<template>
  <!-- Main card container with Tailwind CSS classes -->
  <div class="card bg-base-100 shadow-xl max-w-sm mx-auto">
    <!-- Avatar section -->
    <figure class="px-10 pt-10">
      <div class="avatar">
        <div class="w-24 rounded-full ring ring-primary ring-offset-base-100 ring-offset-2">
          <!-- Display avatar image, with initials as alt text -->
          <img :src="user.avatar" :alt="getInitials(user.name)" />
        </div>
      </div>
    </figure>
    <!-- Card body with user information -->
    <div class="card-body items-center text-center">
      <!-- User name -->
      <h2 class="card-title text-2xl">{{ user.name }}</h2>
      <!-- Job title with badge styling -->
      <p class="text-primary font-medium">{{ user.title }}</p>
      <!-- Location with icon -->
      <p class="text-sm text-base-content/70">📍 {{ user.location }}</p>
      <!-- Bio text -->
      <p class="text-base-content/80 mt-2">{{ user.bio }}</p>
      <!-- Skills section -->
      <div class="mt-4">
        <h3 class="font-semibold text-sm uppercase tracking-wide text-base-content/60">
          Skills
        </h3>
        <p class="mt-1">{{ formatSkills(user.skills) }}</p>
      </div>
      <!-- Social links (only shown if links exist) -->
      <div class="card-actions mt-4" v-if="hasSocialLinks()">
        <a
          v-if="user.social.github"
          :href="getSocialUrl('github', user.social.github)"
          class="btn btn-circle btn-ghost"
          target="_blank"
        >
          GH
        </a>
        <a
          v-if="user.social.twitter"
          :href="getSocialUrl('twitter', user.social.twitter)"
          class="btn btn-circle btn-ghost"
          target="_blank"
        >
          TW
        </a>
        <a
          v-if="user.social.linkedin"
          :href="getSocialUrl('linkedin', user.social.linkedin)"
          class="btn btn-circle btn-ghost"
          target="_blank"
        >
          LI
        </a>
      </div>
    </div>
  </div>
</template>
<style scoped>
/* Additional custom styles if needed */
/* Most styling is handled by Tailwind/DaisyUI classes in the template */
</style>
```
### App.vue
```vue
<script setup lang="ts">
import ProfileCard from "./components/ProfileCard.vue";
</script>
<template>
  <div class="min-h-screen bg-base-200 py-10">
    <h1 class="text-3xl font-bold text-center mb-8">Team Members</h1>
    <ProfileCard />
  </div>
</template>
```
### How to Test
1. Create a new Vue project: `npm create vue@latest my-first-component`
2. Select TypeScript and choose default options
3. Install dependencies: `npm install`
4. Install DaisyUI: `npm install daisyui` and add to `tailwind.config.js`
5. Replace `src/App.vue` and create `src/components/ProfileCard.vue` with the code above
6. Run the dev server: `npm run dev`
7. Open the browser and verify:
   - The profile card displays with user information
   - The avatar image loads
   - Skills are formatted correctly
   - Social links appear and are clickable
---
## 7. Implementation Exercises (Progressive)
### Level 1 (Recall): Fill in the Blanks
Complete the missing parts of this component:
```vue
<script setup _____="ts">
const productName: _____ = "Vue.js Course";
const price: number = _____;
const isAvailable: _____ = true;
const formatPrice = (amount: number): string => {
  return `$${amount._____()}`;
};
</script>
<template>
  <div>
    <h1>{{ _____ }}</h1>
    <p>Price: {{ _____(_____) }}</p>
    <p>{{ isAvailable ? "Available" : "_____ _____" }}</p>
  </div>
</template>
```
**Answer:**
```vue
<script setup lang="ts">
const productName: string = "Vue.js Course";
const price: number = 49.99;
const isAvailable: boolean = true;
const formatPrice = (amount: number): string => {
  return `$${amount.toFixed(2)}`;
};
</script>
<template>
  <div>
    <h1>{{ productName }}</h1>
    <p>Price: {{ formatPrice(price) }}</p>
    <p>{{ isAvailable ? "Available" : "Sold Out" }}</p>
  </div>
</template>
```
---
### Level 2 (Apply): Independent Coding Tasks
**Exercise 2.1: Recipe Card**
Create a `RecipeCard.vue` component that displays:
- Recipe name
- Cooking time (in minutes)
- Difficulty level ("Easy", "Medium", "Hard")
- List of main ingredients (as an array)
- A function that returns "Quick recipe!" if cooking time is under 30 minutes
**Exercise 2.2: Weather Display**
Create a `WeatherDisplay.vue` component that displays:
- City name
- Temperature (number)
- Conditions (string like "Sunny", "Cloudy", "Rainy")
- A function that converts Celsius to Fahrenheit
- Display temperature in both units
---
### Level 3 (Debug): Find and Fix the Bugs
This component has **3 bugs**. Find and fix them:
```vue
<script setup lang="ts">
interface Book {
  title: string
  author: string
  pages: number
}
const book: Book = {
  title: "Learning Vue 3"
  author: "John Smith",
  pages: 350,
}
const getReadingTime = (pages: number): string {
  const hours = Math.ceil(pages / 50);
  return `${hours} hours`;
}
</script>
<template>
  <div>
    <h1>{{ book.title }}</h1>
    <p>By {{ book.Author }}</p>
    <p>{{ book.pages }} pages</p>
    <p>Estimated reading time: {{ getReadingTime }}</p>
  </div>
</template>
```
**Bugs:**
1. Missing comma after `title: "Learning Vue 3"`
2. `book.Author` should be `book.author` (case-sensitive)
3. `getReadingTime` should be `getReadingTime(book.pages)` (missing function call with argument)
**Corrected Code:**
```vue
<script setup lang="ts">
interface Book {
  title: string;
  author: string;
  pages: number;
}
const book: Book = {
  title: "Learning Vue 3",
  author: "John Smith",
  pages: 350,
};
const getReadingTime = (pages: number): string => {
  const hours = Math.ceil(pages / 50);
  return `${hours} hours`;
};
</script>
<template>
  <div>
    <h1>{{ book.title }}</h1>
    <p>By {{ book.author }}</p>
    <p>{{ book.pages }} pages</p>
    <p>Estimated reading time: {{ getReadingTime(book.pages) }}</p>
  </div>
</template>
```
---
**Reply 'next' for Lesson 2.1 (Theory).**
