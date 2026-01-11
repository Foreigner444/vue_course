# Lesson 1.1: Your First Component — Theory & Concepts (Ваш первый компонент)

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 1.1 of 25 (Part 1 of 2) |
| **Topic** | Your First Component: Template + Script Setup / Ваш первый компонент: Шаблон + Script Setup |
| **Module** | Module 1: Core Fundamentals |
| **Prerequisites** | None |
| **You Should Know** | HTML basics, JavaScript ES6+ (const/let, arrow functions, template literals), basic TypeScript syntax |
| **Unlocks** | Lesson 1.2, then Lessons 2, 3, 4, 5 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 1 (Theory):**
1. **Remember:** Define what a Single File Component (SFC) is and identify its three sections
2. **Understand:** Explain how `<script setup>` simplifies component authoring and why it's the modern standard

---

## 2. Real-World Scenario & Context

**Scenario:** Imagine you're building a personal portfolio website. The first thing you need is a simple "About Me" card that displays your name, title, and a short bio. This card needs to be reusable — you might use it on the homepage, the about page, and in blog post footers. Instead of copying HTML everywhere, you want one component that you can drop in anywhere.

**Framework Context:** Vue solves the problem of UI reusability by letting you encapsulate HTML, JavaScript, and CSS into a single file called a **Single File Component (SFC)**. This component can then be imported and used like a custom HTML tag anywhere in your application.

---

## 3. Core Concepts Explained (Deep Dive)

### How It Actually Works

A Vue Single File Component (`.vue` file) is divided into three distinct sections:

```
┌─────────────────────────────────────┐
│  <script setup lang="ts">           │  ← JavaScript/TypeScript logic
│    // Your component logic here     │
│  </script>                          │
├─────────────────────────────────────┤
│  <template>                         │  ← HTML structure
│    <!-- Your HTML here -->          │
│  </template>                        │
├─────────────────────────────────────┤
│  <style scoped>                     │  ← CSS styles (optional)
│    /* Your styles here */           │
│  </style>                           │
└─────────────────────────────────────┘
```

**The Build Process:**
```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  .vue file   │ ──▶ │  Vite/Vue    │ ──▶ │  JavaScript  │
│  (SFC)       │     │  Compiler    │     │  (Browser)   │
└──────────────┘     └──────────────┘     └──────────────┘
```

When you save a `.vue` file, Vite (the build tool) uses the Vue compiler to transform it into standard JavaScript that browsers understand. The `<script setup>` syntax is a compile-time macro that:

1. Automatically exposes all top-level variables to the template
2. Eliminates boilerplate code (no `export default`, no `setup()` function)
3. Provides better TypeScript inference
4. Enables more efficient runtime performance

### Mental Model

Think of a Vue component as a **custom HTML element factory**:

```
┌─────────────────────────────────────────────────────────┐
│                    COMPONENT FACTORY                     │
│                                                         │
│   Input: Your .vue file                                 │
│   ┌─────────────────────────────────────────────────┐   │
│   │  Blueprint (template) + Logic (script) + Look   │   │
│   │  (style) = Custom Element                       │   │
│   └─────────────────────────────────────────────────┘   │
│                                                         │
│   Output: <MyComponent /> — use it like <div> or <p>   │
└─────────────────────────────────────────────────────────┘
```

Just like HTML has built-in elements (`<button>`, `<input>`, `<div>`), Vue lets you create your own elements (`<AboutCard>`, `<UserProfile>`, `<NavBar>`). Each component is self-contained and can be composed together to build complex UIs.

### In Other Contexts

| Framework | Component Definition |
|:--- |:--- |
| **Vue 3 (Composition API)** | `<script setup>` in `.vue` files |
| **React** | Function components with JSX |
| **Angular** | `@Component` decorator with separate `.ts`, `.html`, `.css` files |
| **Vanilla JS** | Web Components with `customElements.define()` |

Vue's SFC approach keeps everything in one file, making components easier to understand and maintain compared to splitting across multiple files.

### When to Use / When NOT to Use

| ✅ Use Components When | ❌ Don't Use Components When |
|:--- |:--- |
| UI element is used more than once | One-off, unique page layout |
| Element has its own state or logic | Simple static HTML with no interactivity |
| You want to isolate styles | Rapid prototyping (initially) |
| Team collaboration (clear boundaries) | Extremely simple apps (single file is fine) |

---

## 4. New Terminology

| Term | Definition |
|:--- |:--- |
| **Single File Component (SFC)** | A `.vue` file that contains template, script, and style sections in one place |
| **`<script setup>`** | A Vue 3.2+ compile-time syntax that simplifies component authoring by auto-exposing variables to the template |
| **Template** | The HTML-like section of a component that defines its structure |
| **Composition API** | Vue 3's function-based API for organizing component logic (as opposed to the Options API) |

---

## 5. Algorithmic Thinking (Planning the Solution)

**The Plan (Narrative):** To create our "About Me" card component, we follow this thinking process:

```
┌─────────────────────────────────────────────────────────┐
│ STEP 1: Create the file                                 │
│   └──▶ Create AboutCard.vue in src/components/          │
│                                                         │
│ STEP 2: Define the data                                 │
│   └──▶ What information does this card display?         │
│        • name (string)                                  │
│        • title (string)                                 │
│        • bio (string)                                   │
│                                                         │
│ STEP 3: Write the script                                │
│   └──▶ Declare variables in <script setup>              │
│                                                         │
│ STEP 4: Build the template                              │
│   └──▶ Create HTML structure using {{ }} for variables  │
│                                                         │
│ STEP 5: Add styles (optional)                           │
│   └──▶ Use Tailwind classes or <style scoped>           │
└─────────────────────────────────────────────────────────┘
```

**Decision Tree for Component Creation:**

```
Do I need to create a component?
│
├── Is this UI reused? ─── YES ──▶ Create a component
│
├── Does it have its own state? ─── YES ──▶ Create a component
│
├── Is it logically separate? ─── YES ──▶ Create a component
│
└── All NO ──▶ Keep it inline in parent
```

---

## 6. Initial Pattern Introduction

| What You Want (Intent) | Code Chunk (The Pattern) | Conceptual Link |
|:--- |:--- |:--- |
| Store data in the component | `const name = 'Value'` | Variable declaration |
| Display data in HTML | `{{ variableName }}` | Template interpolation |
| Use TypeScript | `lang="ts"` attribute | Type safety |
| Keep styles local | `<style scoped>` | CSS encapsulation |

```vue
<script setup lang="ts">
// All top-level variables are automatically available in the template
// No need for `export default` or `return` statements

const name: string = 'Sarah Chen'
const title: string = 'Frontend Developer'
const bio: string = 'Building beautiful user interfaces with Vue.js'
</script>

<template>
  <!-- The template is where your HTML structure lives -->
  <!-- Use {{ }} to insert JavaScript expressions -->
  <div class="card">
    <h2>{{ name }}</h2>
    <p class="title">{{ title }}</p>
    <p class="bio">{{ bio }}</p>
  </div>
</template>

<style scoped>
/* 'scoped' means these styles only apply to THIS component */
/* They won't leak out to parent or child components */
.card {
  padding: 1rem;
  border-radius: 0.5rem;
  background: white;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.title {
  color: #666;
  font-style: italic;
}

.bio {
  margin-top: 0.5rem;
}
</style>
```

**Using the Component:**

```vue
<!-- In App.vue or any parent component -->
<script setup lang="ts">
// Import the component like any JavaScript module
import AboutCard from './components/AboutCard.vue'
</script>

<template>
  <!-- Use it like a custom HTML tag -->
  <AboutCard />
</template>
```

---

## 7. Comprehension Check

1. **What are the three sections of a Vue Single File Component?**
   - Think about what each section is responsible for.

2. **Why does `<script setup>` not require an `export default` or `return` statement?**
   - Consider what the Vue compiler does at build time.

3. **True or False: Styles in `<style scoped>` will affect all components in the application.**
   - Think about what "scoped" means.

---

*Reply 'next' for Lesson 1.2 (Practice).*
