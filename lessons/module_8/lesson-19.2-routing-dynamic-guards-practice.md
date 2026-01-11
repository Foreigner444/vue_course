# Lesson 19.2: Routing II: Dynamic Routes & Route Guards — Practice & Application
# (Маршрутизация II: Динамические маршруты и охранники маршрутов — Практика и применение)

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 19.2 of 25 (Part 2 of 2) |
| **Topic** | Dynamic Routes & Route Guards / Динамические маршруты и охранники маршрутов |
| **Continues From** | Lesson 19.1 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Build a user profile system with dynamic routes and nested views
2. **Analyze:** Debug navigation guard issues and implement proper redirect flows

---

## 2. Quick Recap

From Lesson 19.1:
- **Dynamic segments** (`:id`) create routes that match variable URL parts
- **Route parameters** are accessed via `useRoute().params`
- **Nested routes** render child components inside parent `<RouterView>`
- **Navigation guards** control access: `beforeEach` (global), `beforeEnter` (per-route), `onBeforeRouteLeave` (component)
- **Route meta** (`meta: { requiresAuth: true }`) stores custom data for guards
- **Component reuse**: Watch params to refetch data when navigating between same-component routes

---

## 3. The Variable Frame & Complexity Scale

### Basic Form: Simple Dynamic Route

```typescript
// src/router.ts
const routes = [
  {
    path: '/user/:id',
    component: () => import('./pages/User.vue')
  }
]
```

```vue
<!-- src/pages/User.vue -->
<script setup lang="ts">
import { useRoute } from 'vue-router'
const route = useRoute()
</script>

<template>
  <h1>User ID: {{ route.params.id }}</h1>
</template>
```

### With Options: Nested Routes and Named Views

```typescript
// src/router.ts
const routes = [
  {
    path: '/user/:id',
    component: () => import('./layouts/UserLayout.vue'),
    children: [
      { path: '', name: 'user-home', component: () => import('./pages/user/Home.vue') },
      { path: 'posts', name: 'user-posts', component: () => import('./pages/user/Posts.vue') },
      { path: 'settings', name: 'user-settings', component: () => import('./pages/user/Settings.vue') }
    ]
  }
]
```

### Advanced Form: Protected Routes with Guards and Redirect

```typescript
// src/router.ts
import { createRouter, createWebHistory } from 'vue-router'
import { useAuthStore } from './stores/auth'

const routes = [
  { path: '/', name: 'home', component: () => import('./pages/Home.vue') },
  { path: '/login', name: 'login', component: () => import('./pages/Login.vue') },
  {
    path: '/dashboard',
    name: 'dashboard',
    component: () => import('./pages/Dashboard.vue'),
    meta: { requiresAuth: true }
  },
  {
    path: '/admin',
    component: () => import('./layouts/AdminLayout.vue'),
    meta: { requiresAuth: true, requiresAdmin: true },
    children: [
      { path: '', name: 'admin-home', component: () => import('./pages/admin/Home.vue') },
      { path: 'users', name: 'admin-users', component: () => import('./pages/admin/Users.vue') },
      { path: 'users/:id', name: 'admin-user-detail', component: () => import('./pages/admin/UserDetail.vue') }
    ]
  },
  { path: '/:pathMatch(.*)*', name: 'not-found', component: () => import('./pages/NotFound.vue') }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

router.beforeEach((to, from) => {
  const auth = useAuthStore()
  
  if (to.meta.requiresAuth && !auth.isLoggedIn) {
    return { name: 'login', query: { redirect: to.fullPath } }
  }
  
  if (to.meta.requiresAdmin && !auth.isAdmin) {
    return { name: 'dashboard' }
  }
})

export default router
```

---

## 4. Micro-Implementation Drills

### Drill 1 (Basic): Access Route Parameters

**Task:** Display the product ID from the URL `/product/abc123`.

```vue
<!-- src/pages/ProductDetail.vue -->
<script setup lang="ts">
// TODO: Import useRoute from vue-router
_____

// TODO: Get the route object
_____

// TODO: Access the product ID parameter
const productId = _____
</script>

<template>
  <div>
    <h1>Product Details</h1>
    <!-- TODO: Display the product ID -->
    <p>Product ID: _____</p>
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { useRoute } from 'vue-router'

const route = useRoute()

const productId = route.params.id
</script>

<template>
  <div>
    <h1>Product Details</h1>
    <p>Product ID: {{ route.params.id }}</p>
  </div>
</template>
```

---

### Drill 2 (Variation): Create a RouterLink with Dynamic Parameters

**Task:** Create a link that navigates to `/user/42/posts`.

```vue
<template>
  <!-- Method 1: String path -->
  <RouterLink _____>
    View User 42's Posts
  </RouterLink>
  
  <!-- Method 2: Named route with params -->
  <RouterLink _____>
    View User 42's Posts
  </RouterLink>
</template>
```

**Solution:**
```vue
<template>
  <!-- Method 1: String path -->
  <RouterLink to="/user/42/posts">
    View User 42's Posts
  </RouterLink>
  
  <!-- Method 2: Named route with params -->
  <RouterLink :to="{ name: 'user-posts', params: { id: 42 } }">
    View User 42's Posts
  </RouterLink>
</template>
```

---

### Drill 3 (Combination): Watch for Parameter Changes

**Task:** Refetch data when the user ID changes (e.g., /user/1 → /user/2).

```vue
<script setup lang="ts">
import { ref, watch, onMounted } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const userData = ref(null)
const loading = ref(true)

async function fetchUser(id: string) {
  loading.value = true
  const res = await fetch(`/api/users/${id}`)
  userData.value = await res.json()
  loading.value = false
}

// Fetch on initial load
onMounted(() => {
  fetchUser(route.params.id as string)
})

// TODO: Add a watcher that refetches when the id param changes
_____
</script>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref, watch, onMounted } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const userData = ref(null)
const loading = ref(true)

async function fetchUser(id: string) {
  loading.value = true
  const res = await fetch(`/api/users/${id}`)
  userData.value = await res.json()
  loading.value = false
}

onMounted(() => {
  fetchUser(route.params.id as string)
})

// Watch for param changes - component is reused, not remounted!
watch(
  () => route.params.id,
  (newId, oldId) => {
    if (newId && newId !== oldId) {
      fetchUser(newId as string)
    }
  }
)
</script>
```

---

### Drill 4 (Edge Case): Prevent Navigation with Unsaved Changes

**Task:** Show a confirmation dialog when leaving a page with unsaved changes.

```vue
<script setup lang="ts">
import { ref } from 'vue'
// TODO: Import the correct guard composable
import { _____ } from 'vue-router'

const formData = ref('')
const hasChanges = ref(false)

// TODO: Implement the leave guard
_____((to, from) => {
  if (hasChanges.value) {
    const confirmed = window.confirm('You have unsaved changes. Leave anyway?')
    if (!confirmed) {
      // TODO: Cancel navigation
      _____
    }
  }
  // TODO: Allow navigation
  _____
})
</script>

<template>
  <input v-model="formData" @input="hasChanges = true" />
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref } from 'vue'
import { onBeforeRouteLeave } from 'vue-router'

const formData = ref('')
const hasChanges = ref(false)

onBeforeRouteLeave((to, from) => {
  if (hasChanges.value) {
    const confirmed = window.confirm('You have unsaved changes. Leave anyway?')
    if (!confirmed) {
      return false  // Cancel navigation
    }
  }
  return true  // Allow navigation (or just don't return anything)
})
</script>

<template>
  <input v-model="formData" @input="hasChanges = true" />
</template>
```

---

## 5. Common Pitfalls & Anti-Patterns

| ❌ Common Mistake | ✅ Correct Approach | Why It Matters |
|:--- |:--- |:--- |
| Not watching for param changes | `watch(() => route.params.id, fetchData)` | Component is reused when only params change; data becomes stale |
| Using `route.params.id` directly in reactive state | Use inside `computed` or watch callback | `route.params` updates, but primitive assignment doesn't track changes |
| Infinite redirect loop in guards | Check `to.name !== 'login'` before redirecting | Guard runs again after redirect, causing loop if not careful |
| Forgetting `<RouterView>` in nested route parent | Add `<RouterView />` in layout component | Child routes have nowhere to render |
| Calling Pinia store before app is created | Use store inside guard callback, not at module level | Store isn't available until `app.use(pinia)` is called |

---

## 6. Mini-Project: Protected Admin Panel

**What You'll Build:** An admin panel with authentication, protected routes, dynamic user management, and nested views.

### File Structure

```
src/
├── composables/
│   └── useAuth.ts
├── layouts/
│   └── AdminLayout.vue
├── pages/
│   ├── HomePage.vue
│   ├── LoginPage.vue
│   ├── admin/
│   │   ├── DashboardPage.vue
│   │   ├── UsersPage.vue
│   │   └── UserDetailPage.vue
│   └── NotFoundPage.vue
├── router.ts
├── App.vue
└── main.ts
```

### useAuth.ts (Simple Auth Composable)

```typescript
// src/composables/useAuth.ts
// A simple auth composable for demo purposes
// In production, use Pinia store with proper token management

import { ref, computed } from 'vue'

// Simulated user state (global singleton)
const currentUser = ref<{ id: string; name: string; role: 'user' | 'admin' } | null>(null)

export function useAuth() {
  // Computed properties for auth state
  const isLoggedIn = computed(() => currentUser.value !== null)
  const isAdmin = computed(() => currentUser.value?.role === 'admin')
  const user = computed(() => currentUser.value)

  // Login function - in real app, this would call an API
  function login(username: string, password: string): boolean {
    // Simulated login logic
    if (username === 'admin' && password === 'admin') {
      currentUser.value = { id: '1', name: 'Admin User', role: 'admin' }
      localStorage.setItem('user', JSON.stringify(currentUser.value))
      return true
    }
    if (username === 'user' && password === 'user') {
      currentUser.value = { id: '2', name: 'Regular User', role: 'user' }
      localStorage.setItem('user', JSON.stringify(currentUser.value))
      return true
    }
    return false
  }

  // Logout function
  function logout() {
    currentUser.value = null
    localStorage.removeItem('user')
  }

  // Initialize from localStorage (for page refresh persistence)
  function initAuth() {
    const stored = localStorage.getItem('user')
    if (stored) {
      currentUser.value = JSON.parse(stored)
    }
  }

  return {
    user,
    isLoggedIn,
    isAdmin,
    login,
    logout,
    initAuth
  }
}
```

### router.ts

```typescript
// src/router.ts
import { createRouter, createWebHistory } from 'vue-router'
import { useAuth } from './composables/useAuth'

// Define route types for better TypeScript support
declare module 'vue-router' {
  interface RouteMeta {
    requiresAuth?: boolean
    requiresAdmin?: boolean
    title?: string
  }
}

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      name: 'home',
      component: () => import('./pages/HomePage.vue'),
      meta: { title: 'Home' }
    },
    {
      path: '/login',
      name: 'login',
      component: () => import('./pages/LoginPage.vue'),
      meta: { title: 'Login' }
    },
    {
      // Admin section - protected with nested routes
      path: '/admin',
      component: () => import('./layouts/AdminLayout.vue'),
      meta: { requiresAuth: true, requiresAdmin: true },
      children: [
        {
          // Default admin page: /admin
          path: '',
          name: 'admin-dashboard',
          component: () => import('./pages/admin/DashboardPage.vue'),
          meta: { title: 'Admin Dashboard' }
        },
        {
          // Users list: /admin/users
          path: 'users',
          name: 'admin-users',
          component: () => import('./pages/admin/UsersPage.vue'),
          meta: { title: 'Manage Users' }
        },
        {
          // User detail: /admin/users/123
          path: 'users/:id',
          name: 'admin-user-detail',
          component: () => import('./pages/admin/UserDetailPage.vue'),
          meta: { title: 'User Details' }
        }
      ]
    },
    {
      // 404 catch-all - must be last!
      path: '/:pathMatch(.*)*',
      name: 'not-found',
      component: () => import('./pages/NotFoundPage.vue'),
      meta: { title: 'Page Not Found' }
    }
  ]
})

// Global navigation guard for authentication
router.beforeEach((to, from) => {
  const { isLoggedIn, isAdmin } = useAuth()

  // Update document title
  document.title = (to.meta.title as string) || 'Admin App'

  // Check if route requires authentication
  if (to.meta.requiresAuth && !isLoggedIn.value) {
    // Redirect to login with return URL
    return {
      name: 'login',
      query: { redirect: to.fullPath }
    }
  }

  // Check if route requires admin role
  if (to.meta.requiresAdmin && !isAdmin.value) {
    // Non-admin logged-in users go to home
    return { name: 'home' }
  }

  // Allow navigation
})

export default router
```

### main.ts

```typescript
// src/main.ts
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import { useAuth } from './composables/useAuth'
import './style.css'

// Initialize auth state from localStorage before mounting
const { initAuth } = useAuth()
initAuth()

const app = createApp(App)
app.use(router)
app.mount('#app')
```

### App.vue

```vue
<!-- src/App.vue -->
<script setup lang="ts">
// Root component - just renders RouterView
</script>

<template>
  <RouterView />
</template>
```

### LoginPage.vue

```vue
<!-- src/pages/LoginPage.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { useAuth } from '../composables/useAuth'

const router = useRouter()
const route = useRoute()
const { login, isLoggedIn } = useAuth()

// Form state
const username = ref('')
const password = ref('')
const error = ref('')

// If already logged in, redirect away
if (isLoggedIn.value) {
  router.replace({ name: 'home' })
}

async function handleLogin() {
  error.value = ''
  
  const success = login(username.value, password.value)
  
  if (success) {
    // Redirect to intended destination or home
    const redirectPath = route.query.redirect as string
    router.push(redirectPath || { name: 'home' })
  } else {
    error.value = 'Invalid credentials. Try admin/admin or user/user'
  }
}
</script>

<template>
  <div class="min-h-screen flex items-center justify-center bg-base-200">
    <div class="card w-96 bg-base-100 shadow-xl">
      <div class="card-body">
        <h2 class="card-title justify-center">Login</h2>
        
        <!-- Error message -->
        <div v-if="error" class="alert alert-error">
          <span>{{ error }}</span>
        </div>
        
        <!-- Login form -->
        <form @submit.prevent="handleLogin">
          <div class="form-control">
            <label class="label">
              <span class="label-text">Username</span>
            </label>
            <input 
              v-model="username"
              type="text" 
              placeholder="Enter username"
              class="input input-bordered"
              required
            />
          </div>
          
          <div class="form-control mt-4">
            <label class="label">
              <span class="label-text">Password</span>
            </label>
            <input 
              v-model="password"
              type="password" 
              placeholder="Enter password"
              class="input input-bordered"
              required
            />
          </div>
          
          <div class="form-control mt-6">
            <button type="submit" class="btn btn-primary">
              Login
            </button>
          </div>
        </form>
        
        <!-- Demo credentials hint -->
        <div class="text-sm text-base-content/60 mt-4">
          <p>Demo credentials:</p>
          <p>Admin: admin / admin</p>
          <p>User: user / user</p>
        </div>
        
        <!-- Back to home link -->
        <div class="mt-4">
          <RouterLink to="/" class="link link-primary">
            ← Back to Home
          </RouterLink>
        </div>
      </div>
    </div>
  </div>
</template>
```

### AdminLayout.vue

```vue
<!-- src/layouts/AdminLayout.vue -->
<script setup lang="ts">
import { useRouter, useRoute } from 'vue-router'
import { useAuth } from '../composables/useAuth'

const router = useRouter()
const route = useRoute()
const { user, logout } = useAuth()

function handleLogout() {
  logout()
  router.push({ name: 'login' })
}

// Helper to check if a route is active
function isActive(routeName: string): boolean {
  return route.name === routeName
}
</script>

<template>
  <div class="min-h-screen flex">
    <!-- Sidebar Navigation -->
    <aside class="w-64 bg-base-200 p-4">
      <div class="mb-8">
        <h1 class="text-xl font-bold">Admin Panel</h1>
        <p class="text-sm text-base-content/60">
          Welcome, {{ user?.name }}
        </p>
      </div>
      
      <!-- Navigation Links -->
      <nav class="flex flex-col gap-2">
        <RouterLink 
          :to="{ name: 'admin-dashboard' }"
          :class="[
            'btn btn-ghost justify-start',
            isActive('admin-dashboard') && 'btn-active'
          ]"
        >
          📊 Dashboard
        </RouterLink>
        
        <RouterLink 
          :to="{ name: 'admin-users' }"
          :class="[
            'btn btn-ghost justify-start',
            (isActive('admin-users') || isActive('admin-user-detail')) && 'btn-active'
          ]"
        >
          👥 Users
        </RouterLink>
        
        <div class="divider"></div>
        
        <RouterLink 
          :to="{ name: 'home' }"
          class="btn btn-ghost justify-start"
        >
          🏠 Back to Site
        </RouterLink>
        
        <button 
          @click="handleLogout"
          class="btn btn-ghost justify-start text-error"
        >
          🚪 Logout
        </button>
      </nav>
    </aside>
    
    <!-- Main Content Area -->
    <main class="flex-1 p-8 bg-base-100">
      <!-- Nested routes render here -->
      <RouterView />
    </main>
  </div>
</template>
```

### DashboardPage.vue

```vue
<!-- src/pages/admin/DashboardPage.vue -->
<script setup lang="ts">
import { useAuth } from '../../composables/useAuth'

const { user } = useAuth()
</script>

<template>
  <div>
    <h1 class="text-3xl font-bold mb-8">Admin Dashboard</h1>
    
    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
      <!-- Stats Cards -->
      <div class="stat bg-base-200 rounded-box">
        <div class="stat-title">Total Users</div>
        <div class="stat-value">1,234</div>
        <div class="stat-desc">↗︎ 12% from last month</div>
      </div>
      
      <div class="stat bg-base-200 rounded-box">
        <div class="stat-title">Active Sessions</div>
        <div class="stat-value">89</div>
        <div class="stat-desc">Currently online</div>
      </div>
      
      <div class="stat bg-base-200 rounded-box">
        <div class="stat-title">Revenue</div>
        <div class="stat-value">$12,345</div>
        <div class="stat-desc">↗︎ 5% from last month</div>
      </div>
    </div>
    
    <div class="mt-8 p-4 bg-base-200 rounded-box">
      <h2 class="font-bold mb-2">Quick Actions</h2>
      <div class="flex gap-4">
        <RouterLink :to="{ name: 'admin-users' }" class="btn btn-primary">
          Manage Users
        </RouterLink>
      </div>
    </div>
  </div>
</template>
```

### UsersPage.vue

```vue
<!-- src/pages/admin/UsersPage.vue -->
<script setup lang="ts">
import { ref } from 'vue'

// Mock user data - in real app, fetch from API
const users = ref([
  { id: '1', name: 'John Doe', email: 'john@example.com', role: 'admin' },
  { id: '2', name: 'Jane Smith', email: 'jane@example.com', role: 'user' },
  { id: '3', name: 'Bob Wilson', email: 'bob@example.com', role: 'user' },
  { id: '4', name: 'Alice Brown', email: 'alice@example.com', role: 'moderator' },
])
</script>

<template>
  <div>
    <div class="flex justify-between items-center mb-8">
      <h1 class="text-3xl font-bold">Users</h1>
      <button class="btn btn-primary">+ Add User</button>
    </div>
    
    <!-- Users Table -->
    <div class="overflow-x-auto">
      <table class="table table-zebra">
        <thead>
          <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Email</th>
            <th>Role</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="user in users" :key="user.id">
            <td>{{ user.id }}</td>
            <td>{{ user.name }}</td>
            <td>{{ user.email }}</td>
            <td>
              <span 
                :class="[
                  'badge',
                  user.role === 'admin' ? 'badge-primary' : 
                  user.role === 'moderator' ? 'badge-secondary' : 
                  'badge-ghost'
                ]"
              >
                {{ user.role }}
              </span>
            </td>
            <td>
              <!-- Dynamic route link with user ID -->
              <RouterLink 
                :to="{ name: 'admin-user-detail', params: { id: user.id } }"
                class="btn btn-sm btn-ghost"
              >
                View Details
              </RouterLink>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>
```

### UserDetailPage.vue

```vue
<!-- src/pages/admin/UserDetailPage.vue -->
<script setup lang="ts">
import { ref, watch, onMounted } from 'vue'
import { useRoute, useRouter, onBeforeRouteLeave } from 'vue-router'

const route = useRoute()
const router = useRouter()

// User data state
const user = ref<{ id: string; name: string; email: string; role: string } | null>(null)
const loading = ref(true)
const hasUnsavedChanges = ref(false)

// Mock API call to fetch user
async function fetchUser(id: string) {
  loading.value = true
  
  // Simulate API delay
  await new Promise(resolve => setTimeout(resolve, 500))
  
  // Mock data based on ID
  const mockUsers: Record<string, { id: string; name: string; email: string; role: string }> = {
    '1': { id: '1', name: 'John Doe', email: 'john@example.com', role: 'admin' },
    '2': { id: '2', name: 'Jane Smith', email: 'jane@example.com', role: 'user' },
    '3': { id: '3', name: 'Bob Wilson', email: 'bob@example.com', role: 'user' },
    '4': { id: '4', name: 'Alice Brown', email: 'alice@example.com', role: 'moderator' },
  }
  
  user.value = mockUsers[id] || null
  loading.value = false
}

// Fetch on mount
onMounted(() => {
  fetchUser(route.params.id as string)
})

// IMPORTANT: Watch for param changes!
// When navigating from /admin/users/1 to /admin/users/2,
// the component is REUSED, so we must watch and refetch
watch(
  () => route.params.id,
  (newId) => {
    if (newId) {
      hasUnsavedChanges.value = false
      fetchUser(newId as string)
    }
  }
)

// Prevent accidental navigation with unsaved changes
onBeforeRouteLeave((to, from) => {
  if (hasUnsavedChanges.value) {
    const answer = window.confirm(
      'You have unsaved changes. Do you really want to leave?'
    )
    if (!answer) return false
  }
})

function handleInputChange() {
  hasUnsavedChanges.value = true
}

function handleSave() {
  // In real app, call API to save
  hasUnsavedChanges.value = false
  alert('User saved successfully!')
}

function goBack() {
  router.push({ name: 'admin-users' })
}
</script>

<template>
  <div>
    <!-- Header with back button -->
    <div class="flex items-center gap-4 mb-8">
      <button @click="goBack" class="btn btn-ghost btn-sm">
        ← Back
      </button>
      <h1 class="text-3xl font-bold">
        User Details
        <span v-if="hasUnsavedChanges" class="text-warning text-sm">
          (unsaved changes)
        </span>
      </h1>
    </div>
    
    <!-- Loading state -->
    <div v-if="loading" class="flex justify-center py-8">
      <span class="loading loading-spinner loading-lg"></span>
    </div>
    
    <!-- User not found -->
    <div v-else-if="!user" class="alert alert-error">
      <span>User not found with ID: {{ route.params.id }}</span>
    </div>
    
    <!-- User form -->
    <div v-else class="max-w-md">
      <div class="form-control mb-4">
        <label class="label">
          <span class="label-text">User ID</span>
        </label>
        <input 
          :value="user.id"
          type="text" 
          class="input input-bordered"
          disabled
        />
      </div>
      
      <div class="form-control mb-4">
        <label class="label">
          <span class="label-text">Name</span>
        </label>
        <input 
          v-model="user.name"
          @input="handleInputChange"
          type="text" 
          class="input input-bordered"
        />
      </div>
      
      <div class="form-control mb-4">
        <label class="label">
          <span class="label-text">Email</span>
        </label>
        <input 
          v-model="user.email"
          @input="handleInputChange"
          type="email" 
          class="input input-bordered"
        />
      </div>
      
      <div class="form-control mb-4">
        <label class="label">
          <span class="label-text">Role</span>
        </label>
        <select 
          v-model="user.role"
          @change="handleInputChange"
          class="select select-bordered"
        >
          <option value="user">User</option>
          <option value="moderator">Moderator</option>
          <option value="admin">Admin</option>
        </select>
      </div>
      
      <div class="flex gap-4 mt-6">
        <button 
          @click="handleSave"
          class="btn btn-primary"
          :disabled="!hasUnsavedChanges"
        >
          Save Changes
        </button>
        <button 
          @click="goBack"
          class="btn btn-ghost"
        >
          Cancel
        </button>
      </div>
      
      <!-- Navigation hint -->
      <div class="mt-8 p-4 bg-base-200 rounded-box">
        <p class="text-sm text-base-content/60">
          Try navigating to another user to see param watching in action:
        </p>
        <div class="flex gap-2 mt-2">
          <RouterLink 
            v-for="id in ['1', '2', '3', '4']"
            :key="id"
            :to="{ name: 'admin-user-detail', params: { id } }"
            :class="[
              'btn btn-xs',
              route.params.id === id ? 'btn-primary' : 'btn-ghost'
            ]"
          >
            User {{ id }}
          </RouterLink>
        </div>
      </div>
    </div>
  </div>
</template>
```

### HomePage.vue

```vue
<!-- src/pages/HomePage.vue -->
<script setup lang="ts">
import { useAuth } from '../composables/useAuth'
import { useRouter } from 'vue-router'

const { isLoggedIn, isAdmin, user, logout } = useAuth()
const router = useRouter()

function handleLogout() {
  logout()
  router.push({ name: 'home' })
}
</script>

<template>
  <div class="min-h-screen bg-base-100">
    <!-- Navigation -->
    <nav class="navbar bg-base-200">
      <div class="flex-1">
        <span class="text-xl font-bold px-4">My App</span>
      </div>
      <div class="flex-none">
        <template v-if="isLoggedIn">
          <span class="mr-4">Hello, {{ user?.name }}</span>
          <RouterLink 
            v-if="isAdmin" 
            :to="{ name: 'admin-dashboard' }" 
            class="btn btn-ghost"
          >
            Admin Panel
          </RouterLink>
          <button @click="handleLogout" class="btn btn-ghost">
            Logout
          </button>
        </template>
        <template v-else>
          <RouterLink :to="{ name: 'login' }" class="btn btn-primary">
            Login
          </RouterLink>
        </template>
      </div>
    </nav>
    
    <!-- Hero Content -->
    <div class="hero min-h-[80vh]">
      <div class="hero-content text-center">
        <div class="max-w-md">
          <h1 class="text-5xl font-bold">Welcome!</h1>
          <p class="py-6">
            This is a demo of Vue Router with dynamic routes and navigation guards.
            Try logging in as admin (admin/admin) to access the protected admin panel.
          </p>
          <div class="flex gap-4 justify-center">
            <RouterLink 
              v-if="!isLoggedIn"
              :to="{ name: 'login' }" 
              class="btn btn-primary"
            >
              Get Started
            </RouterLink>
            <RouterLink 
              v-if="isAdmin"
              :to="{ name: 'admin-dashboard' }" 
              class="btn btn-secondary"
            >
              Go to Admin
            </RouterLink>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
```

### NotFoundPage.vue

```vue
<!-- src/pages/NotFoundPage.vue -->
<script setup lang="ts">
import { useRoute } from 'vue-router'

const route = useRoute()
</script>

<template>
  <div class="min-h-screen flex items-center justify-center bg-base-100">
    <div class="text-center">
      <h1 class="text-9xl font-bold text-error">404</h1>
      <h2 class="text-3xl font-bold mt-4">Page Not Found</h2>
      <p class="py-6 text-base-content/60">
        The page <code class="bg-base-200 px-2 py-1 rounded">{{ route.fullPath }}</code> 
        doesn't exist.
      </p>
      <RouterLink :to="{ name: 'home' }" class="btn btn-primary">
        Go Home
      </RouterLink>
    </div>
  </div>
</template>
```

### How to Test

1. **Start the development server:**
   ```bash
   npm run dev
   ```

2. **Test authentication flow:**
   - Visit `/admin` while logged out → Should redirect to `/login?redirect=/admin`
   - Login with `admin/admin` → Should redirect back to `/admin`
   - Login with `user/user` → Should redirect to home (not admin)

3. **Test dynamic routes:**
   - Go to Admin → Users → Click "View Details" on any user
   - URL should show `/admin/users/1`, `/admin/users/2`, etc.
   - Click the user buttons at the bottom to navigate between users without page reload

4. **Test navigation guards:**
   - On User Detail page, modify a field
   - Try to navigate away → Should see confirmation dialog
   - Click Cancel → Should stay on page
   - Click OK → Should navigate away

5. **Test 404 handling:**
   - Visit `/nonexistent` → Should see 404 page

---

## 7. Implementation Exercises (Progressive)

### Level 1 (Recall): Fill in the Blanks

```typescript
// Complete this navigation guard that checks for authentication
router.beforeEach((to, from) => {
  const { isLoggedIn } = useAuth()
  
  if (to.meta._____ && !isLoggedIn.value) {
    return { 
      name: '_____',
      query: { redirect: to._____ }
    }
  }
})
```

**Answer:**
```typescript
router.beforeEach((to, from) => {
  const { isLoggedIn } = useAuth()
  
  if (to.meta.requiresAuth && !isLoggedIn.value) {
    return { 
      name: 'login',
      query: { redirect: to.fullPath }
    }
  }
})
```

---

### Level 2 (Apply): Small Independent Tasks

**Exercise 2.1:** Create a route configuration for a blog with the pattern `/blog/:year/:month/:slug` (e.g., `/blog/2024/01/my-first-post`).

**Solution:**
```typescript
{
  path: '/blog/:year/:month/:slug',
  name: 'blog-post',
  component: () => import('./pages/BlogPost.vue')
}
```

---

**Exercise 2.2:** In a component, write code to programmatically navigate to the blog post route with params `year: '2024'`, `month: '01'`, `slug: 'hello-world'`.

**Solution:**
```vue
<script setup lang="ts">
import { useRouter } from 'vue-router'

const router = useRouter()

function goToPost() {
  router.push({
    name: 'blog-post',
    params: {
      year: '2024',
      month: '01',
      slug: 'hello-world'
    }
  })
}
</script>
```

---

### Level 3 (Debug): Fix the Broken Code

```typescript
// This router guard has 3 bugs. Find and fix them.
router.beforeEach((to, from) => {
  // Bug 1: Wrong property access
  if (to.requiresAuth) {
    // Bug 2: Checking wrong value
    if (!isLoggedIn) {
      // Bug 3: Causes infinite redirect loop
      return { name: 'login' }
    }
  }
})
```

**Fixed Code:**
```typescript
router.beforeEach((to, from) => {
  const { isLoggedIn } = useAuth()  // Need to get isLoggedIn from composable
  
  // Fix 1: Access meta properties through to.meta
  if (to.meta.requiresAuth) {
    // Fix 2: isLoggedIn is a ref, access .value
    if (!isLoggedIn.value) {
      // Fix 3: Prevent loop by checking if already going to login
      if (to.name !== 'login') {
        return { name: 'login', query: { redirect: to.fullPath } }
      }
    }
  }
})
```

---

## Self-Check: Key Takeaways

After completing this lesson, you should be able to:

- [ ] Define dynamic routes with `:param` syntax
- [ ] Access route parameters using `useRoute().params`
- [ ] Watch for parameter changes to refetch data
- [ ] Create nested routes with child `<RouterView>`
- [ ] Implement global navigation guards with `beforeEach`
- [ ] Use route `meta` fields for custom data (like `requiresAuth`)
- [ ] Redirect users based on authentication state
- [ ] Prevent navigation with `onBeforeRouteLeave` for unsaved changes
- [ ] Handle 404s with catch-all routes

---

**Module 8 Complete!** 🎉

**Reply 'next' for Lesson 20.1 (Theory) — State Management I: Pinia Stores.**
