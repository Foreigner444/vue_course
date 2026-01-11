# Lesson 18.1: Routing I: Basic Navigation with Vue Router — Theory & Concepts
# (Маршрутизация I: Базовая навигация с Vue Router — Теория и концепции)

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 18.1 of 25 (Part 1 of 2) |
| **Topic** | Basic Navigation with Vue Router / Базовая навигация с Vue Router |
| **Module** | Module 8: Navigation |
| **Prerequisites** | Lessons 1-17 (Core fundamentals, Component communication, Composables, Data fetching) |
| **You Should Know** | JavaScript modules, async/await, Vue component basics, composables |
| **Unlocks** | Lesson 18.2, then Lesson 19 (Dynamic Routes & Guards) |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 1 (Theory):**
1. **Remember:** Define what client-side routing is and identify the core Vue Router components (`createRouter`, `RouterView`, `RouterLink`)
2. **Understand:** Explain how Vue Router intercepts navigation and renders components without full page reloads

---

## 2. Real-World Scenario & Context

**Scenario:** You're building a company website with multiple pages: a home page showcasing products, an "About Us" page with team information, and a "Contact" page with a form. Users expect to click navigation links and see different content instantly—without the browser reloading the entire page. They also expect the URL to change so they can bookmark pages or share links with others.

**Framework Context:** Vue Router solves the problem of mapping URLs to components in a Single Page Application (SPA). Instead of the server sending a new HTML page for each URL, Vue Router intercepts navigation events, updates the URL in the browser, and swaps out components dynamically—creating a fast, app-like experience.

---

## 3. Core Concepts Explained (Deep Dive)

### 3.1 How Client-Side Routing Actually Works

In a traditional website, clicking a link sends a request to the server, which responds with a new HTML page. The browser discards the current page and renders the new one—this causes a full page reload.

**Client-side routing** changes this flow:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    CLIENT-SIDE ROUTING FLOW                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  1. User clicks link         2. Router intercepts      3. URL updates       │
│     ("/about")                  (prevents default)        (History API)     │
│         │                            │                        │             │
│         ▼                            ▼                        ▼             │
│  ┌─────────────┐              ┌─────────────┐          ┌─────────────┐      │
│  │  <RouterLink│   ───────►   │ Vue Router  │  ───────►│  Browser    │      │
│  │  to="/about"│              │  matches    │          │  shows      │      │
│  │  >          │              │  route      │          │  /about     │      │
│  └─────────────┘              └─────────────┘          └─────────────┘      │
│                                      │                                      │
│                                      ▼                                      │
│                               4. Component swap                             │
│                               ┌─────────────┐                               │
│                               │<RouterView> │                               │
│                               │  renders    │                               │
│                               │  AboutPage  │                               │
│                               └─────────────┘                               │
│                                                                             │
│  ✓ No server request for HTML                                               │
│  ✓ No full page reload                                                      │
│  ✓ URL changes (bookmarkable)                                               │
│  ✓ Browser back/forward works                                               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Under the hood, Vue Router uses the browser's History API** (`history.pushState`) to update the URL without triggering a page reload. It maintains a mapping of URL paths to Vue components and renders the appropriate component when the URL changes.

### 3.2 Mental Model: The Router as a Switchboard Operator

Think of Vue Router as a **telephone switchboard operator** from the early 1900s:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     MENTAL MODEL: SWITCHBOARD OPERATOR                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│    CALLER (URL)              SWITCHBOARD (Router)         RECIPIENT         │
│    ───────────               ──────────────────           ─────────         │
│                                                                             │
│    "/home"      ─────►  ┌─────────────────────┐  ─────►  HomePage.vue       │
│                         │                     │                             │
│    "/about"     ─────►  │   Route Table:      │  ─────►  AboutPage.vue      │
│                         │   "/" → Home        │                             │
│    "/contact"   ─────►  │   "/about" → About  │  ─────►  ContactPage.vue    │
│                         │   "/contact"→Contact│                             │
│    "/xyz"       ─────►  │   "*" → NotFound    │  ─────►  NotFound.vue       │
│                         │                     │                             │
│                         └─────────────────────┘                             │
│                                                                             │
│  • The operator (router) receives a "call" (URL request)                    │
│  • Looks up the destination in the directory (route table)                  │
│  • Connects the caller to the right recipient (component)                   │
│  • If no match exists, connects to a default (404 page)                     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.3 In Other Contexts: Comparing Routing Solutions

| Framework | Routing Solution | Key Syntax |
|:--- |:--- |:--- |
| **Vue 3** | Vue Router 4 | `<RouterLink to="/about">`, `<RouterView />` |
| **React** | React Router | `<Link to="/about">`, `<Outlet />` |
| **Angular** | Angular Router | `routerLink="/about"`, `<router-outlet>` |
| **Vanilla JS** | Manual History API | `history.pushState()`, `popstate` event |

Vue Router's declarative approach (`<RouterLink>`, `<RouterView>`) is similar to React Router. The key difference is Vue Router's tight integration with Vue's reactivity system—route params become reactive refs automatically.

### 3.4 When to Use Vue Router

| ✅ Use Vue Router When | ❌ Don't Need Router When |
|:--- |:--- |
| Building a multi-page SPA | Single-page widget/component |
| Users need bookmarkable URLs | Internal tool with no URL requirements |
| SEO matters (with SSR) | Embedded component in another app |
| Complex navigation with guards | Simple tab switching (use `v-if`) |
| Need browser back/forward support | Modal dialogs or overlays |

---

## 4. New Terminology

| Term | Definition |
|:--- |:--- |
| **Client-Side Routing** | Navigation handled entirely in the browser using JavaScript, without requesting new HTML pages from the server |
| **Route** | A mapping between a URL path (e.g., `/about`) and a Vue component that should render when that path is visited |
| **RouterView** | A Vue Router component that acts as a placeholder—it renders whichever component matches the current route |
| **RouterLink** | A Vue Router component that renders an `<a>` tag with special navigation behavior—it updates the URL without a page reload |
| **History Mode** | A router configuration that uses clean URLs (`/about`) instead of hash URLs (`/#/about`) by leveraging the browser's History API |

---

## 5. Algorithmic Thinking (Planning the Solution)

To add routing to a Vue application, follow this mental checklist:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ROUTING SETUP DECISION TREE                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  START: "I need multiple pages in my Vue app"                               │
│         │                                                                   │
│         ▼                                                                   │
│  ┌──────────────────────────────────────────┐                               │
│  │ STEP 1: Install Vue Router               │                               │
│  │ npm install vue-router@4                 │                               │
│  └──────────────────────────────────────────┘                               │
│         │                                                                   │
│         ▼                                                                   │
│  ┌──────────────────────────────────────────┐                               │
│  │ STEP 2: Create page components           │                               │
│  │ - HomePage.vue                           │                               │
│  │ - AboutPage.vue                          │                               │
│  │ - ContactPage.vue                        │                               │
│  └──────────────────────────────────────────┘                               │
│         │                                                                   │
│         ▼                                                                   │
│  ┌──────────────────────────────────────────┐                               │
│  │ STEP 3: Define route table               │                               │
│  │ Map each path to its component           │                               │
│  │ "/" → HomePage                           │                               │
│  │ "/about" → AboutPage                     │                               │
│  └──────────────────────────────────────────┘                               │
│         │                                                                   │
│         ▼                                                                   │
│  ┌──────────────────────────────────────────┐                               │
│  │ STEP 4: Create router instance           │                               │
│  │ createRouter({ history, routes })        │                               │
│  └──────────────────────────────────────────┘                               │
│         │                                                                   │
│         ▼                                                                   │
│  ┌──────────────────────────────────────────┐                               │
│  │ STEP 5: Register router with Vue app     │                               │
│  │ app.use(router)                          │                               │
│  └──────────────────────────────────────────┘                               │
│         │                                                                   │
│         ▼                                                                   │
│  ┌──────────────────────────────────────────┐                               │
│  │ STEP 6: Add RouterView & RouterLink      │                               │
│  │ in App.vue                               │                               │
│  └──────────────────────────────────────────┘                               │
│         │                                                                   │
│         ▼                                                                   │
│  DONE: Navigation works!                                                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Initial Pattern Introduction

### Pattern 1: Creating the Router Instance

| What You Want (Intent) | Code Chunk (The Pattern) | Conceptual Link |
|:--- |:--- |:--- |
| Set up routing for my app | `createRouter({ history, routes })` | The "switchboard" configuration |
| Use clean URLs (not hash) | `createWebHistory()` | History API mode |
| Map URLs to components | `routes: [{ path, component }]` | The routing table |

```typescript
// src/router.ts
// This file creates and configures the Vue Router instance

// Import the router creation functions from vue-router
import { createRouter, createWebHistory } from 'vue-router'

// Import the page components that will be rendered for each route
import HomePage from './pages/HomePage.vue'
import AboutPage from './pages/AboutPage.vue'
import ContactPage from './pages/ContactPage.vue'

// Define the route table: an array of route objects
// Each object maps a URL path to a Vue component
const routes = [
  {
    path: '/',           // The URL path (root/home)
    name: 'home',        // Optional: a name for programmatic navigation
    component: HomePage  // The component to render at this path
  },
  {
    path: '/about',
    name: 'about',
    component: AboutPage
  },
  {
    path: '/contact',
    name: 'contact',
    component: ContactPage
  }
]

// Create the router instance
const router = createRouter({
  // createWebHistory() enables "history mode" with clean URLs
  // Pass the base URL from Vite's environment variables
  history: createWebHistory(import.meta.env.BASE_URL),
  
  // Pass the routes array we defined above
  routes
})

// Export the router so we can use it in main.ts
export default router
```

### Pattern 2: Registering the Router

```typescript
// src/main.ts
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'  // Import our router configuration

const app = createApp(App)

// Register the router with the Vue app
// This makes <RouterView> and <RouterLink> available globally
app.use(router)

app.mount('#app')
```

### Pattern 3: Using RouterView and RouterLink

| What You Want (Intent) | Code Chunk (The Pattern) | Conceptual Link |
|:--- |:--- |:--- |
| Display the current page | `<RouterView />` | The "display screen" |
| Create a navigation link | `<RouterLink to="/path">` | The "phone dial" |
| Style the active link | `:class="{ active: isActive }"` or CSS `.router-link-active` | Visual feedback |

```vue
<!-- src/App.vue -->
<script setup lang="ts">
// No imports needed! RouterView and RouterLink are globally available
// after calling app.use(router) in main.ts
</script>

<template>
  <div class="min-h-screen bg-base-100">
    <!-- Navigation bar with links -->
    <nav class="navbar bg-base-200 px-4">
      <div class="flex gap-4">
        <!-- RouterLink renders an <a> tag but intercepts clicks -->
        <!-- The "to" prop specifies the destination path -->
        <RouterLink 
          to="/" 
          class="btn btn-ghost"
        >
          Home
        </RouterLink>
        
        <RouterLink 
          to="/about" 
          class="btn btn-ghost"
        >
          About
        </RouterLink>
        
        <RouterLink 
          to="/contact" 
          class="btn btn-ghost"
        >
          Contact
        </RouterLink>
      </div>
    </nav>
    
    <!-- RouterView is the "slot" where page components render -->
    <!-- When URL is "/about", AboutPage.vue renders here -->
    <main class="container mx-auto p-4">
      <RouterView />
    </main>
  </div>
</template>
```

### Pattern 4: Active Link Styling

Vue Router automatically adds CSS classes to `<RouterLink>` components:

- `router-link-active` — Added when the link's path **partially matches** the current route
- `router-link-exact-active` — Added when the link's path **exactly matches** the current route

```vue
<!-- Styling active links with Tailwind -->
<template>
  <nav class="flex gap-4">
    <!-- Using the slot API for custom active styling -->
    <RouterLink 
      to="/" 
      v-slot="{ isActive, isExactActive }"
      custom
    >
      <a 
        :class="[
          'btn',
          isExactActive ? 'btn-primary' : 'btn-ghost'
        ]"
      >
        Home
      </a>
    </RouterLink>
    
    <!-- Or use global CSS targeting the automatic classes -->
    <RouterLink to="/about" class="nav-link">
      About
    </RouterLink>
  </nav>
</template>

<style>
/* These classes are automatically applied by Vue Router */
.nav-link.router-link-active {
  @apply text-primary font-bold;
}

.nav-link.router-link-exact-active {
  @apply border-b-2 border-primary;
}
</style>
```

---

## 7. Comprehension Check

1. **Conceptual:** What is the difference between server-side routing (traditional websites) and client-side routing (SPAs)? What are the benefits of client-side routing?

2. **Conceptual:** In the Vue Router architecture, what is the role of `<RouterView>` and where should it be placed in your application?

3. **True/False:** When using Vue Router with `createWebHistory()`, clicking a `<RouterLink>` causes the browser to send an HTTP request to the server for the new page. (Explain your answer.)

---

**Reply 'next' for Lesson 18.2 (Practice).**
