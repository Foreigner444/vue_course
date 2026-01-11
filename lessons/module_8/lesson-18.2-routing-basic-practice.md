# Lesson 18.2: Routing I: Basic Navigation with Vue Router — Practice & Application
# (Маршрутизация I: Базовая навигация с Vue Router — Практика и применение)

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 18.2 of 25 (Part 2 of 2) |
| **Topic** | Basic Navigation with Vue Router / Базовая навигация с Vue Router |
| **Continues From** | Lesson 18.1 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Create a multi-page Vue application with working navigation using Vue Router
2. **Analyze:** Compare different navigation methods and identify the correct approach for various scenarios

---

## 2. Quick Recap

From Lesson 18.1:
- **Vue Router** enables client-side navigation without page reloads by mapping URL paths to Vue components
- **`createRouter()`** creates the router instance with a history mode and route table
- **`<RouterView>`** is the placeholder where matched components render
- **`<RouterLink>`** creates navigation links that update the URL without server requests
- Vue Router automatically adds `router-link-active` and `router-link-exact-active` classes for styling

---

## 3. The Variable Frame & Complexity Scale

### Basic Form: Simple Route Definition

```typescript
// src/router.ts
import { createRouter, createWebHistory } from 'vue-router'
import HomePage from './pages/HomePage.vue'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    { path: '/', component: HomePage }
  ]
})

export default router
```

### With Options: Named Routes and Aliases

```typescript
// src/router.ts
import { createRouter, createWebHistory } from 'vue-router'
import HomePage from './pages/HomePage.vue'
import AboutPage from './pages/AboutPage.vue'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    { 
      path: '/',
      name: 'home',           // Named route for programmatic navigation
      component: HomePage,
      alias: '/home'          // Alternative URL that shows same component
    },
    { 
      path: '/about',
      name: 'about',
      component: AboutPage
    }
  ]
})

export default router
```

### Advanced Form: Lazy Loading, Redirects, and Catch-All

```typescript
// src/router.ts
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  
  // Scroll behavior: scroll to top on navigation
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      return savedPosition  // Restore position on back/forward
    }
    return { top: 0 }       // Scroll to top for new pages
  },
  
  routes: [
    { 
      path: '/',
      name: 'home',
      // Lazy loading: component is loaded only when route is visited
      component: () => import('./pages/HomePage.vue')
    },
    { 
      path: '/about',
      name: 'about',
      component: () => import('./pages/AboutPage.vue')
    },
    {
      // Redirect: /info goes to /about
      path: '/info',
      redirect: '/about'
    },
    {
      // Redirect using named route
      path: '/home',
      redirect: { name: 'home' }
    },
    {
      // Catch-all 404 route (must be last!)
      // :pathMatch(.*) captures everything, * allows slashes
      path: '/:pathMatch(.*)*',
      name: 'not-found',
      component: () => import('./pages/NotFoundPage.vue')
    }
  ]
})

export default router
```

---

## 4. Micro-Implementation Drills

### Drill 1 (Basic): Create a Simple Route

**Task:** Add a route for a "Services" page.

```typescript
// src/router.ts
import { createRouter, createWebHistory } from 'vue-router'
import HomePage from './pages/HomePage.vue'
import ServicesPage from './pages/ServicesPage.vue'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    { path: '/', component: HomePage },
    // TODO: Add a route for '/services' that renders ServicesPage
    _____
  ]
})

export default router
```

**Solution:**
```typescript
{ path: '/services', component: ServicesPage }
```

---

### Drill 2 (Variation): Add Named Routes

**Task:** Convert these routes to use named routes and create links using the names.

```vue
<!-- src/App.vue -->
<script setup lang="ts">
// Router is already configured with named routes
</script>

<template>
  <nav class="flex gap-4 p-4">
    <!-- TODO: Convert these to use :to="{ name: 'routeName' }" syntax -->
    <RouterLink to="/">Home</RouterLink>
    <RouterLink to="/products">Products</RouterLink>
    <RouterLink to="/contact">Contact</RouterLink>
  </nav>
  
  <RouterView />
</template>
```

**Solution:**
```vue
<template>
  <nav class="flex gap-4 p-4">
    <RouterLink :to="{ name: 'home' }">Home</RouterLink>
    <RouterLink :to="{ name: 'products' }">Products</RouterLink>
    <RouterLink :to="{ name: 'contact' }">Contact</RouterLink>
  </nav>
  
  <RouterView />
</template>
```

---

### Drill 3 (Combination): Programmatic Navigation with useRouter

**Task:** Create a button that navigates to the home page when clicked.

```vue
<!-- src/components/BackToHomeButton.vue -->
<script setup lang="ts">
// TODO: Import useRouter from vue-router
_____

// TODO: Get the router instance
_____

// TODO: Create a function that navigates to home
function goHome() {
  _____
}
</script>

<template>
  <button @click="goHome" class="btn btn-primary">
    Back to Home
  </button>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { useRouter } from 'vue-router'

const router = useRouter()

function goHome() {
  router.push('/')
  // Or with named route: router.push({ name: 'home' })
}
</script>

<template>
  <button @click="goHome" class="btn btn-primary">
    Back to Home
  </button>
</template>
```

---

### Drill 4 (Edge Case): Handle 404 Not Found

**Task:** Add a catch-all route that displays a NotFound component for any unmatched URL.

```typescript
// src/router.ts
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    { path: '/', name: 'home', component: () => import('./pages/HomePage.vue') },
    { path: '/about', name: 'about', component: () => import('./pages/AboutPage.vue') },
    // TODO: Add a catch-all route for 404 pages
    // Hint: Use path '/:pathMatch(.*)*'
    _____
  ]
})

export default router
```

**Solution:**
```typescript
{
  path: '/:pathMatch(.*)*',
  name: 'not-found',
  component: () => import('./pages/NotFoundPage.vue')
}
```

---

## 5. Common Pitfalls & Anti-Patterns

| ❌ Common Mistake | ✅ Correct Approach | Why It Matters |
|:--- |:--- |:--- |
| Using `<a href="/about">` for internal links | Use `<RouterLink to="/about">` | Regular `<a>` tags cause full page reloads, defeating the purpose of SPA routing |
| Forgetting to call `app.use(router)` in main.ts | Always register router: `app.use(router)` | Without registration, `<RouterView>` and `<RouterLink>` won't work |
| Placing catch-all route before specific routes | Put `/:pathMatch(.*)*` **last** in routes array | Routes are matched in order; catch-all first would match everything |
| Using `router.push()` without importing useRouter | Import and call: `const router = useRouter()` | You can't access the router instance directly in `<script setup>` without the composable |
| Forgetting `<RouterView>` in App.vue | Add `<RouterView />` where pages should render | Without it, matched components have nowhere to display |

---

## 6. Mini-Project: Multi-Page Blog Layout

**What You'll Build:** A blog website with Home, About, and Blog pages with working navigation and active link styling.

### File Structure

```
src/
├── pages/
│   ├── HomePage.vue
│   ├── AboutPage.vue
│   └── BlogPage.vue
├── components/
│   └── NavBar.vue
├── router.ts
├── App.vue
└── main.ts
```

### router.ts

```typescript
// src/router.ts
// This file configures all the routes for our blog application

import { createRouter, createWebHistory } from 'vue-router'

// Define routes with lazy loading for better performance
// Each route maps a URL path to a page component
const router = createRouter({
  // Use HTML5 history mode for clean URLs (no # in URL)
  history: createWebHistory(import.meta.env.BASE_URL),
  
  // Define the route table
  routes: [
    {
      path: '/',                    // URL: http://localhost:5173/
      name: 'home',                 // Name for programmatic navigation
      component: () => import('./pages/HomePage.vue')  // Lazy loaded
    },
    {
      path: '/about',               // URL: http://localhost:5173/about
      name: 'about',
      component: () => import('./pages/AboutPage.vue')
    },
    {
      path: '/blog',                // URL: http://localhost:5173/blog
      name: 'blog',
      component: () => import('./pages/BlogPage.vue')
    },
    {
      // Catch-all route for 404 errors
      // This MUST be the last route in the array
      path: '/:pathMatch(.*)*',
      name: 'not-found',
      component: () => import('./pages/NotFoundPage.vue')
    }
  ]
})

export default router
```

### main.ts

```typescript
// src/main.ts
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import './style.css'  // Tailwind CSS

const app = createApp(App)

// Register the router plugin
// This enables <RouterView> and <RouterLink> globally
app.use(router)

app.mount('#app')
```

### NavBar.vue

```vue
<!-- src/components/NavBar.vue -->
<script setup lang="ts">
// No imports needed - RouterLink is globally available
// We'll use the v-slot API for custom active styling
</script>

<template>
  <!-- Navigation bar with DaisyUI styling -->
  <nav class="navbar bg-base-200 shadow-lg">
    <div class="container mx-auto">
      <!-- Logo/Brand -->
      <div class="flex-1">
        <RouterLink to="/" class="btn btn-ghost text-xl font-bold">
          My Blog
        </RouterLink>
      </div>
      
      <!-- Navigation Links -->
      <div class="flex-none">
        <ul class="menu menu-horizontal px-1 gap-2">
          <!-- Home Link with custom active styling using v-slot -->
          <li>
            <RouterLink 
              to="/"
              v-slot="{ isExactActive }"
              custom
            >
              <a 
                href="/"
                :class="[
                  'btn btn-sm',
                  isExactActive ? 'btn-primary' : 'btn-ghost'
                ]"
                @click.prevent="$router.push('/')"
              >
                Home
              </a>
            </RouterLink>
          </li>
          
          <!-- About Link -->
          <li>
            <RouterLink 
              to="/about"
              v-slot="{ isActive }"
              custom
            >
              <a 
                href="/about"
                :class="[
                  'btn btn-sm',
                  isActive ? 'btn-primary' : 'btn-ghost'
                ]"
                @click.prevent="$router.push('/about')"
              >
                About
              </a>
            </RouterLink>
          </li>
          
          <!-- Blog Link -->
          <li>
            <RouterLink 
              to="/blog"
              v-slot="{ isActive }"
              custom
            >
              <a 
                href="/blog"
                :class="[
                  'btn btn-sm',
                  isActive ? 'btn-primary' : 'btn-ghost'
                ]"
                @click.prevent="$router.push('/blog')"
              >
                Blog
              </a>
            </RouterLink>
          </li>
        </ul>
      </div>
    </div>
  </nav>
</template>
```

### App.vue

```vue
<!-- src/App.vue -->
<script setup lang="ts">
// Import our NavBar component
import NavBar from './components/NavBar.vue'
</script>

<template>
  <div class="min-h-screen bg-base-100">
    <!-- Navigation bar appears on all pages -->
    <NavBar />
    
    <!-- RouterView: This is where page components render -->
    <!-- When URL changes, the matched component appears here -->
    <main class="container mx-auto p-6">
      <RouterView />
    </main>
    
    <!-- Footer appears on all pages -->
    <footer class="footer footer-center p-4 bg-base-200 text-base-content">
      <p>© 2024 My Blog. Built with Vue 3 + Vue Router.</p>
    </footer>
  </div>
</template>
```

### HomePage.vue

```vue
<!-- src/pages/HomePage.vue -->
<script setup lang="ts">
import { useRouter } from 'vue-router'

// Get the router instance for programmatic navigation
const router = useRouter()

// Function to navigate to blog page
function goToBlog() {
  router.push({ name: 'blog' })
}
</script>

<template>
  <div class="hero min-h-[60vh]">
    <div class="hero-content text-center">
      <div class="max-w-md">
        <h1 class="text-5xl font-bold">Welcome to My Blog</h1>
        <p class="py-6">
          Discover articles about web development, Vue.js, and modern 
          JavaScript. Start exploring our content today!
        </p>
        
        <!-- Two ways to navigate: RouterLink and programmatic -->
        <div class="flex gap-4 justify-center">
          <!-- Declarative: using RouterLink -->
          <RouterLink to="/about" class="btn btn-outline">
            Learn About Us
          </RouterLink>
          
          <!-- Programmatic: using router.push() -->
          <button @click="goToBlog" class="btn btn-primary">
            Read Blog Posts
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
```

### AboutPage.vue

```vue
<!-- src/pages/AboutPage.vue -->
<script setup lang="ts">
// This page doesn't need any reactive state
// It's purely presentational
</script>

<template>
  <div class="prose lg:prose-xl mx-auto">
    <h1>About Us</h1>
    
    <p>
      We are a team of passionate developers dedicated to sharing knowledge
      about modern web development. Our blog covers topics including Vue.js,
      TypeScript, and frontend best practices.
    </p>
    
    <h2>Our Mission</h2>
    <p>
      To make web development accessible and enjoyable for everyone, from
      beginners to experienced developers looking to stay current with
      the latest technologies.
    </p>
    
    <div class="not-prose mt-8">
      <RouterLink to="/blog" class="btn btn-primary">
        Check Out Our Blog
      </RouterLink>
    </div>
  </div>
</template>
```

### BlogPage.vue

```vue
<!-- src/pages/BlogPage.vue -->
<script setup lang="ts">
import { ref } from 'vue'

// Sample blog post data
// In a real app, this would come from an API
const posts = ref([
  {
    id: 1,
    title: 'Getting Started with Vue 3',
    excerpt: 'Learn the basics of Vue 3 and the Composition API.',
    date: '2024-01-15'
  },
  {
    id: 2,
    title: 'Understanding Vue Router',
    excerpt: 'Master client-side routing in your Vue applications.',
    date: '2024-01-20'
  },
  {
    id: 3,
    title: 'State Management with Pinia',
    excerpt: 'Explore the modern state management solution for Vue.',
    date: '2024-01-25'
  }
])
</script>

<template>
  <div>
    <h1 class="text-4xl font-bold mb-8">Blog Posts</h1>
    
    <!-- Grid of blog post cards -->
    <div class="grid md:grid-cols-2 lg:grid-cols-3 gap-6">
      <article 
        v-for="post in posts" 
        :key="post.id"
        class="card bg-base-200 shadow-xl"
      >
        <div class="card-body">
          <h2 class="card-title">{{ post.title }}</h2>
          <p class="text-sm text-base-content/70">{{ post.date }}</p>
          <p>{{ post.excerpt }}</p>
          <div class="card-actions justify-end">
            <!-- In Lesson 19, we'll make this link to dynamic routes -->
            <button class="btn btn-primary btn-sm">
              Read More
            </button>
          </div>
        </div>
      </article>
    </div>
  </div>
</template>
```

### NotFoundPage.vue

```vue
<!-- src/pages/NotFoundPage.vue -->
<script setup lang="ts">
import { useRoute, useRouter } from 'vue-router'

// useRoute gives us information about the current route
const route = useRoute()
const router = useRouter()

function goHome() {
  router.push({ name: 'home' })
}
</script>

<template>
  <div class="hero min-h-[60vh]">
    <div class="hero-content text-center">
      <div class="max-w-md">
        <h1 class="text-9xl font-bold text-error">404</h1>
        <h2 class="text-3xl font-bold mt-4">Page Not Found</h2>
        <p class="py-6">
          Sorry, we couldn't find the page 
          <code class="bg-base-200 px-2 py-1 rounded">{{ route.fullPath }}</code>
        </p>
        <button @click="goHome" class="btn btn-primary">
          Go Back Home
        </button>
      </div>
    </div>
  </div>
</template>
```

### How to Test

1. **Start the development server:**
   ```bash
   npm run dev
   ```

2. **Test navigation:**
   - Click "Home", "About", "Blog" links — URL should change, page content should update
   - Active link should have different styling (btn-primary)
   - Browser back/forward buttons should work

3. **Test 404 handling:**
   - Navigate to `http://localhost:5173/nonexistent`
   - Should see the 404 page with the attempted path displayed

4. **Test programmatic navigation:**
   - On home page, click "Read Blog Posts" button
   - Should navigate to /blog

---

## 7. Implementation Exercises (Progressive)

### Level 1 (Recall): Fill in the Blanks

```typescript
// Fill in the blanks to complete this router configuration
import { _____, createWebHistory } from 'vue-router'

const router = _____({
  history: _____(),
  routes: [
    { path: '/', component: () => import('./pages/Home.vue') },
    { path: '/about', component: () => import('./pages/About.vue') }
  ]
})

export default router
```

**Answer:**
```typescript
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    { path: '/', component: () => import('./pages/Home.vue') },
    { path: '/about', component: () => import('./pages/About.vue') }
  ]
})

export default router
```

---

### Level 2 (Apply): Small Independent Tasks

**Exercise 2.1:** Create a `<RouterLink>` that navigates to `/products` and displays "View Products" text. Use the `class="btn btn-secondary"` for styling.

**Solution:**
```vue
<RouterLink to="/products" class="btn btn-secondary">
  View Products
</RouterLink>
```

---

**Exercise 2.2:** Write a function using `useRouter` that navigates to the "contact" named route and also passes a query parameter `subject=support`.

**Solution:**
```vue
<script setup lang="ts">
import { useRouter } from 'vue-router'

const router = useRouter()

function goToSupport() {
  router.push({ 
    name: 'contact', 
    query: { subject: 'support' } 
  })
  // This navigates to: /contact?subject=support
}
</script>
```

---

### Level 3 (Debug): Fix the Broken Code

```vue
<!-- This code has 3 bugs. Find and fix them. -->
<script setup lang="ts">
// Bug 1: Missing import
const router = useRouter()
</script>

<template>
  <nav>
    <!-- Bug 2: Wrong element for SPA navigation -->
    <a href="/home">Home</a>
    <a href="/about">About</a>
  </nav>
  
  <!-- Bug 3: Missing something important -->
  <main>
    <!-- Where do pages render? -->
  </main>
</template>
```

**Fixed Code:**
```vue
<script setup lang="ts">
// Fix 1: Import useRouter from vue-router
import { useRouter } from 'vue-router'

const router = useRouter()
</script>

<template>
  <nav>
    <!-- Fix 2: Use RouterLink instead of <a> for SPA navigation -->
    <RouterLink to="/home">Home</RouterLink>
    <RouterLink to="/about">About</RouterLink>
  </nav>
  
  <!-- Fix 3: Add RouterView so page components can render -->
  <main>
    <RouterView />
  </main>
</template>
```

---

## Self-Check: Key Takeaways

After completing this lesson, you should be able to:

- [ ] Set up Vue Router in a new project with `createRouter` and `createWebHistory`
- [ ] Define routes that map URL paths to Vue components
- [ ] Use `<RouterLink>` for navigation that doesn't cause page reloads
- [ ] Place `<RouterView>` where you want page components to render
- [ ] Style active navigation links using built-in CSS classes or v-slot API
- [ ] Navigate programmatically using `useRouter().push()`
- [ ] Create a catch-all route for 404 pages
- [ ] Use lazy loading (`() => import()`) for route components

---

**Reply 'next' for Lesson 19.1 (Theory) — Routing II: Dynamic Routes & Route Guards.**
