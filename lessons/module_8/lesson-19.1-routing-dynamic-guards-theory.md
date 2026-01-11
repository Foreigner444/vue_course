# Lesson 19.1: Routing II: Dynamic Routes & Route Guards — Theory & Concepts
# (Маршрутизация II: Динамические маршруты и охранники маршрутов — Теория и концепции)

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 19.1 of 25 (Part 1 of 2) |
| **Topic** | Dynamic Routes & Route Guards / Динамические маршруты и охранники маршрутов |
| **Module** | Module 8: Navigation |
| **Prerequisites** | Lesson 18 (Basic Vue Router), Lesson 15 (Composables), Lesson 16 (Data Fetching) |
| **You Should Know** | Basic Vue Router setup, `useRouter()`, `<RouterLink>`, async/await |
| **Unlocks** | Lesson 19.2, then Module 9 (State Management with Pinia) |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 1 (Theory):**
1. **Remember:** Define dynamic route segments, route parameters, and navigation guards
2. **Understand:** Explain how Vue Router matches dynamic URLs and how guards control navigation flow

---

## 2. Real-World Scenario & Context

**Scenario:** You're building a social media application. Each user has a profile page at `/user/123` where `123` is the user's unique ID. The URL structure is the same for every user (`/user/:id`), but the content changes based on the ID. Additionally, some pages like `/settings` and `/admin` should only be accessible to logged-in users—unauthorized visitors should be redirected to the login page.

**Framework Context:** Vue Router solves these problems with **dynamic route segments** (for variable URL parts like user IDs) and **navigation guards** (for controlling access to routes based on conditions like authentication). These features transform a simple URL-to-component mapper into a powerful, secure navigation system.

---

## 3. Core Concepts Explained (Deep Dive)

### 3.1 Dynamic Route Segments: How They Work

Dynamic routes use **parameters** (prefixed with `:`) to match variable URL parts:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    DYNAMIC ROUTE MATCHING                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ROUTE DEFINITION              URL VISITED           MATCHED PARAMS         │
│  ────────────────              ───────────           ──────────────         │
│                                                                             │
│  /user/:id                     /user/123             { id: '123' }          │
│  /user/:id                     /user/456             { id: '456' }          │
│  /user/:id                     /user/abc             { id: 'abc' }          │
│                                                                             │
│  /posts/:year/:month           /posts/2024/01        { year: '2024',        │
│                                                        month: '01' }        │
│                                                                             │
│  /files/:path+                 /files/docs/vue/guide { path: ['docs',       │
│  (+ = one or more segments)                            'vue', 'guide'] }    │
│                                                                             │
│  /search/:query?               /search                { query: undefined }  │
│  (? = optional param)          /search/vue           { query: 'vue' }       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Under the hood:** Vue Router compiles route paths into regular expressions. When a URL is visited, it tests each route pattern until a match is found, then extracts the variable parts as parameters.

### 3.2 Nested Routes: Component Hierarchy

Nested routes allow you to render child components inside parent components:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    NESTED ROUTE STRUCTURE                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  URL: /user/123/profile                                                     │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────┐        │
│  │ App.vue                                                         │        │
│  │ ┌─────────────────────────────────────────────────────────────┐ │        │
│  │ │ <RouterView> → UserLayout.vue                               │ │        │
│  │ │ ┌───────────────────────────────────────────────────────┐   │ │        │
│  │ │ │ User Sidebar  │  <RouterView> → ProfileTab.vue        │   │ │        │
│  │ │ │               │                                       │   │ │        │
│  │ │ │ • Profile     │  User Profile Content Here            │   │ │        │
│  │ │ │ • Settings    │                                       │   │ │        │
│  │ │ │ • Activity    │                                       │   │ │        │
│  │ │ └───────────────────────────────────────────────────────┘   │ │        │
│  │ └─────────────────────────────────────────────────────────────┘ │        │
│  └─────────────────────────────────────────────────────────────────┘        │
│                                                                             │
│  Route Config:                                                              │
│  {                                                                          │
│    path: '/user/:id',                                                       │
│    component: UserLayout,                                                   │
│    children: [                                                              │
│      { path: 'profile', component: ProfileTab },                            │
│      { path: 'settings', component: SettingsTab },                          │
│      { path: 'activity', component: ActivityTab }                           │
│    ]                                                                        │
│  }                                                                          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.3 Mental Model: Route Guards as Security Checkpoints

Think of navigation guards as **security checkpoints** at an airport:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    MENTAL MODEL: AIRPORT SECURITY                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  TRAVELER                      CHECKPOINT                   DESTINATION     │
│  (User navigating)             (Guard function)             (Route)         │
│                                                                             │
│       ┌───────┐                 ┌──────────┐                ┌─────────┐     │
│       │ User  │ ──────────────► │ GLOBAL   │ ──────────────►│ /home   │     │
│       │       │                 │ GUARD    │    ✓ PASS      │         │     │
│       └───────┘                 └──────────┘                └─────────┘     │
│                                      │                                      │
│                                      │ Check: Is user authenticated?        │
│                                      │                                      │
│       ┌───────┐                 ┌──────────┐                ┌─────────┐     │
│       │ Guest │ ──────────────► │ ROUTE    │ ──────REDIRECT─►│ /login  │     │
│       │       │                 │ GUARD    │    ✗ FAIL      │         │     │
│       └───────┘                 │ /admin   │                └─────────┘     │
│                                 └──────────┘                                │
│                                                                             │
│  Guard Types:                                                               │
│  • Global (beforeEach) - Checks EVERY navigation                            │
│  • Per-Route (beforeEnter) - Checks specific routes only                    │
│  • In-Component (onBeforeRouteLeave) - Checks when leaving a component      │
│                                                                             │
│  Guard Actions:                                                             │
│  • return true / undefined → Allow navigation                               │
│  • return false → Cancel navigation                                         │
│  • return '/path' → Redirect to different route                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.4 Navigation Guard Lifecycle

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    NAVIGATION GUARD SEQUENCE                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Navigation Triggered (e.g., click RouterLink)                              │
│         │                                                                   │
│         ▼                                                                   │
│  ┌─────────────────────────────────────────┐                                │
│  │ 1. onBeforeRouteLeave (in-component)    │ ← Leaving current component    │
│  │    "Are you sure? You have unsaved..."  │                                │
│  └─────────────────────────────────────────┘                                │
│         │                                                                   │
│         ▼                                                                   │
│  ┌─────────────────────────────────────────┐                                │
│  │ 2. beforeEach (global)                  │ ← Every navigation             │
│  │    "Check authentication..."            │                                │
│  └─────────────────────────────────────────┘                                │
│         │                                                                   │
│         ▼                                                                   │
│  ┌─────────────────────────────────────────┐                                │
│  │ 3. beforeEnter (per-route)              │ ← Only for target route        │
│  │    "Check admin permissions..."         │                                │
│  └─────────────────────────────────────────┘                                │
│         │                                                                   │
│         ▼                                                                   │
│  ┌─────────────────────────────────────────┐                                │
│  │ 4. onBeforeRouteUpdate (in-component)   │ ← Same component, new params   │
│  │    "Fetch new user data for :id..."     │   (e.g., /user/1 → /user/2)    │
│  └─────────────────────────────────────────┘                                │
│         │                                                                   │
│         ▼                                                                   │
│  ┌─────────────────────────────────────────┐                                │
│  │ 5. afterEach (global)                   │ ← After navigation complete    │
│  │    "Analytics tracking, scroll reset"   │                                │
│  └─────────────────────────────────────────┘                                │
│         │                                                                   │
│         ▼                                                                   │
│  Navigation Complete! Component renders.                                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.5 In Other Contexts: Comparing Guard Patterns

| Framework | Guard Concept | Syntax |
|:--- |:--- |:--- |
| **Vue Router** | Navigation Guards | `router.beforeEach((to, from) => { })` |
| **React Router** | Loader + Protected Route pattern | `<Route element={<ProtectedRoute><Page /></ProtectedRoute>} />` |
| **Angular** | Route Guards (CanActivate, etc.) | `canActivate: [AuthGuard]` |
| **Next.js** | Middleware | `middleware.ts` with `NextResponse.redirect()` |

Vue Router's guard system is more declarative than React's component-based approach, and more flexible than Angular's class-based guards.

### 3.6 When to Use Each Guard Type

| Guard Type | Use When | Example |
|:--- |:--- |:--- |
| **Global `beforeEach`** | Every route needs the same check | Authentication, loading indicators |
| **Per-route `beforeEnter`** | Specific routes need protection | Admin-only pages, premium features |
| **`onBeforeRouteLeave`** | Prevent accidental data loss | Unsaved form warning |
| **`onBeforeRouteUpdate`** | Same component, different params | Refetch data when ID changes |
| **Global `afterEach`** | Post-navigation side effects | Analytics, scroll to top |

---

## 4. New Terminology

| Term | Definition |
|:--- |:--- |
| **Dynamic Segment** | A variable part of a URL path, defined with `:paramName` syntax (e.g., `/user/:id`) |
| **Route Parameters** | The extracted values from dynamic segments, accessible via `useRoute().params` |
| **Nested Routes** | Child routes that render inside a parent route's `<RouterView>`, creating component hierarchies |
| **Navigation Guard** | A function that runs before, during, or after navigation to control access or perform side effects |
| **Route Meta** | Custom data attached to route definitions (e.g., `meta: { requiresAuth: true }`) used by guards |

---

## 5. Algorithmic Thinking (Planning the Solution)

### Planning a Protected Route System

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    PROTECTED ROUTES DECISION TREE                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  START: User navigates to /admin/dashboard                                  │
│         │                                                                   │
│         ▼                                                                   │
│  ┌──────────────────────────────────────────┐                               │
│  │ STEP 1: Define route with meta field     │                               │
│  │ { path: '/admin', meta: { requiresAuth } }│                              │
│  └──────────────────────────────────────────┘                               │
│         │                                                                   │
│         ▼                                                                   │
│  ┌──────────────────────────────────────────┐                               │
│  │ STEP 2: Create global beforeEach guard   │                               │
│  │ router.beforeEach((to, from) => { ... }) │                               │
│  └──────────────────────────────────────────┘                               │
│         │                                                                   │
│         ▼                                                                   │
│  ┌──────────────────────────────────────────┐                               │
│  │ STEP 3: Check if route requires auth     │                               │
│  │ if (to.meta.requiresAuth)                │                               │
│  └──────────────────────────────────────────┘                               │
│         │                                                                   │
│         ├─── NO ──► Allow navigation (return true)                          │
│         │                                                                   │
│         ▼ YES                                                               │
│  ┌──────────────────────────────────────────┐                               │
│  │ STEP 4: Check if user is authenticated   │                               │
│  │ if (isAuthenticated())                   │                               │
│  └──────────────────────────────────────────┘                               │
│         │                                                                   │
│         ├─── YES ──► Allow navigation (return true)                         │
│         │                                                                   │
│         ▼ NO                                                                │
│  ┌──────────────────────────────────────────┐                               │
│  │ STEP 5: Redirect to login                │                               │
│  │ return { name: 'login', query: {         │                               │
│  │   redirect: to.fullPath                  │                               │
│  │ }}                                       │                               │
│  └──────────────────────────────────────────┘                               │
│         │                                                                   │
│         ▼                                                                   │
│  User sees login page with redirect param                                   │
│  After login, redirect back to original page                                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Initial Pattern Introduction

### Pattern 1: Dynamic Route Parameters

| What You Want (Intent) | Code Chunk (The Pattern) | Conceptual Link |
|:--- |:--- |:--- |
| URL with variable part | `path: '/user/:id'` | Dynamic segment definition |
| Access the parameter | `useRoute().params.id` | Extract matched value |
| React to param changes | `watch(() => route.params.id, ...)` | Reactive param tracking |

```typescript
// src/router.ts
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      // :id is a dynamic segment - it matches any value
      path: '/user/:id',
      name: 'user-profile',
      component: () => import('./pages/UserProfile.vue')
    },
    {
      // Multiple dynamic segments
      path: '/posts/:year/:month/:slug',
      name: 'blog-post',
      component: () => import('./pages/BlogPost.vue')
    },
    {
      // Optional parameter (? makes it optional)
      path: '/search/:query?',
      name: 'search',
      component: () => import('./pages/SearchResults.vue')
    }
  ]
})
```

```vue
<!-- src/pages/UserProfile.vue -->
<script setup lang="ts">
import { useRoute } from 'vue-router'
import { ref, watch, onMounted } from 'vue'

// useRoute() gives access to the current route information
const route = useRoute()

// Access the dynamic parameter
// Note: params are always strings!
const userId = route.params.id  // Type: string | string[]

// Reactive state for user data
const user = ref<{ name: string; email: string } | null>(null)
const loading = ref(true)

// Function to fetch user data
async function fetchUser(id: string) {
  loading.value = true
  // Simulated API call
  const response = await fetch(`/api/users/${id}`)
  user.value = await response.json()
  loading.value = false
}

// Fetch on initial mount
onMounted(() => {
  fetchUser(route.params.id as string)
})

// IMPORTANT: Watch for parameter changes!
// If user navigates from /user/1 to /user/2, the component is REUSED
// We must watch for param changes and refetch data
watch(
  () => route.params.id,
  (newId) => {
    if (newId) {
      fetchUser(newId as string)
    }
  }
)
</script>

<template>
  <div>
    <p v-if="loading">Loading user {{ route.params.id }}...</p>
    <div v-else-if="user">
      <h1>{{ user.name }}</h1>
      <p>{{ user.email }}</p>
    </div>
  </div>
</template>
```

### Pattern 2: Nested Routes

```typescript
// src/router.ts
const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      path: '/user/:id',
      component: () => import('./layouts/UserLayout.vue'),
      // Children routes render inside UserLayout's <RouterView>
      children: [
        {
          // Empty path = default child route
          // Renders at /user/:id
          path: '',
          name: 'user-overview',
          component: () => import('./pages/user/Overview.vue')
        },
        {
          // Renders at /user/:id/posts
          path: 'posts',
          name: 'user-posts',
          component: () => import('./pages/user/Posts.vue')
        },
        {
          // Renders at /user/:id/settings
          path: 'settings',
          name: 'user-settings',
          component: () => import('./pages/user/Settings.vue')
        }
      ]
    }
  ]
})
```

```vue
<!-- src/layouts/UserLayout.vue -->
<script setup lang="ts">
import { useRoute } from 'vue-router'

const route = useRoute()
</script>

<template>
  <div class="flex">
    <!-- Sidebar - visible on all user pages -->
    <aside class="w-64 bg-base-200 p-4">
      <h2 class="font-bold mb-4">User {{ route.params.id }}</h2>
      <nav class="flex flex-col gap-2">
        <RouterLink :to="{ name: 'user-overview', params: { id: route.params.id } }">
          Overview
        </RouterLink>
        <RouterLink :to="{ name: 'user-posts', params: { id: route.params.id } }">
          Posts
        </RouterLink>
        <RouterLink :to="{ name: 'user-settings', params: { id: route.params.id } }">
          Settings
        </RouterLink>
      </nav>
    </aside>
    
    <!-- Child route content renders here -->
    <main class="flex-1 p-4">
      <RouterView />
    </main>
  </div>
</template>
```

### Pattern 3: Global Navigation Guard

| What You Want (Intent) | Code Chunk (The Pattern) | Conceptual Link |
|:--- |:--- |:--- |
| Check every navigation | `router.beforeEach((to, from) => {})` | Global checkpoint |
| Mark routes as protected | `meta: { requiresAuth: true }` | Route metadata |
| Redirect unauthorized users | `return { name: 'login' }` | Guard action |

```typescript
// src/router.ts
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      path: '/',
      name: 'home',
      component: () => import('./pages/Home.vue')
    },
    {
      path: '/login',
      name: 'login',
      component: () => import('./pages/Login.vue')
    },
    {
      path: '/dashboard',
      name: 'dashboard',
      component: () => import('./pages/Dashboard.vue'),
      // Meta field marks this route as requiring authentication
      meta: { requiresAuth: true }
    },
    {
      path: '/admin',
      name: 'admin',
      component: () => import('./pages/Admin.vue'),
      // Multiple meta flags
      meta: { requiresAuth: true, requiresAdmin: true }
    }
  ]
})

// Simulated auth check (in real app, check Pinia store or token)
function isAuthenticated(): boolean {
  return localStorage.getItem('token') !== null
}

function isAdmin(): boolean {
  return localStorage.getItem('role') === 'admin'
}

// Global navigation guard - runs BEFORE every navigation
router.beforeEach((to, from) => {
  // to: the target route
  // from: the current route
  
  // Check if route requires authentication
  if (to.meta.requiresAuth && !isAuthenticated()) {
    // Redirect to login, preserving the intended destination
    return {
      name: 'login',
      query: { redirect: to.fullPath }
    }
  }
  
  // Check if route requires admin role
  if (to.meta.requiresAdmin && !isAdmin()) {
    // Redirect to home if not admin
    return { name: 'home' }
  }
  
  // Allow navigation (return true is implicit)
})

// Global after guard - runs AFTER navigation completes
router.afterEach((to, from) => {
  // Good for analytics or page title updates
  document.title = (to.meta.title as string) || 'My App'
})

export default router
```

### Pattern 4: Per-Route Guard

```typescript
// src/router.ts
const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      path: '/premium',
      name: 'premium',
      component: () => import('./pages/Premium.vue'),
      // Per-route guard - only runs when entering THIS route
      beforeEnter: (to, from) => {
        const hasSubscription = checkSubscription()
        if (!hasSubscription) {
          return { name: 'pricing' }  // Redirect to pricing page
        }
        // Allow access
      }
    }
  ]
})
```

### Pattern 5: In-Component Guards

```vue
<!-- src/pages/EditPost.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import { onBeforeRouteLeave, onBeforeRouteUpdate } from 'vue-router'

const hasUnsavedChanges = ref(false)
const postContent = ref('')

// Runs when trying to leave this route
onBeforeRouteLeave((to, from) => {
  if (hasUnsavedChanges.value) {
    const answer = window.confirm(
      'You have unsaved changes. Do you really want to leave?'
    )
    // Cancel navigation if user clicks "Cancel"
    if (!answer) return false
  }
})

// Runs when route params change (same component, different data)
// e.g., /edit/1 → /edit/2
onBeforeRouteUpdate((to, from) => {
  // Reset form when navigating to different post
  postContent.value = ''
  hasUnsavedChanges.value = false
  // Fetch new post data here...
})
</script>

<template>
  <div>
    <h1>Edit Post</h1>
    <textarea 
      v-model="postContent"
      @input="hasUnsavedChanges = true"
    ></textarea>
    <button @click="hasUnsavedChanges = false">Save</button>
  </div>
</template>
```

---

## 7. Comprehension Check

1. **Conceptual:** What is the difference between a "global" navigation guard (`beforeEach`) and a "per-route" guard (`beforeEnter`)? When would you choose one over the other?

2. **Conceptual:** When navigating from `/user/1` to `/user/2`, Vue Router **reuses** the same `UserProfile` component instead of destroying and recreating it. Why is this a problem for data fetching, and how do you solve it?

3. **True/False:** If a navigation guard returns `false`, the navigation is cancelled and the user stays on the current page. If it returns `{ name: 'login' }`, the user is redirected to the login route. (Explain the behavior.)

---

**Reply 'next' for Lesson 19.2 (Practice) — Dynamic Routes & Route Guards.**
