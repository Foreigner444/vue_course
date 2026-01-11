# Guided Project 3: Protected Admin Panel

## Project Overview

| Field | Value |
|:--- |:--- |
| **Module** | Module 8: Navigation |
| **Project Number** | 3 of 3 |
| **Difficulty** | Advanced |
| **Estimated Time** | 75-90 minutes |

**What You'll Build:** A secure admin panel with authentication, role-based access control, and protected routes. Unauthorized users are redirected to the login page with return URL handling. Different user roles (admin, editor, viewer) have access to different sections.

**Real-World Scenario:** An admin area requires authentication; unauthorized users are redirected to login — similar to CMS dashboards, e-commerce admin panels, or SaaS application backends.

---

## Patterns You'll Practice

| Pattern | Description |
|:--- |:--- |
| Global `beforeEach` guard | Check authentication on every navigation |
| Route `meta` fields | Store `requiresAuth`, `requiredRole` metadata |
| Redirect handling | Save intended destination, redirect after login |
| Per-route `beforeEnter` | Protect specific routes with custom logic |
| Role-based access | Restrict routes based on user role |
| `onBeforeRouteLeave` | Warn before leaving with unsaved changes |
| Auth composable | Centralized authentication state management |
| Token persistence | Store auth state in localStorage |
| Programmatic navigation | Redirect with `router.push()` after auth actions |

---

## Component Structure

```
src/
├── components/
│   ├── AdminNav.vue            # Admin sidebar navigation
│   ├── TopBar.vue              # Top bar with user info
│   └── StatCard.vue            # Dashboard stat card
├── composables/
│   └── useAuth.ts              # Authentication state and methods
├── layouts/
│   ├── AdminLayout.vue         # Protected admin layout
│   └── AuthLayout.vue          # Login/register layout
├── pages/
│   ├── public/
│   │   └── HomePage.vue        # Public landing page
│   ├── auth/
│   │   ├── LoginPage.vue       # Login form
│   │   └── RegisterPage.vue    # Registration form
│   ├── admin/
│   │   ├── DashboardPage.vue   # Admin dashboard (all roles)
│   │   ├── UsersPage.vue       # User management (admin only)
│   │   ├── UserEditPage.vue    # Edit user (admin only)
│   │   ├── ContentPage.vue     # Content management (admin, editor)
│   │   ├── SettingsPage.vue    # Settings (admin only)
│   │   └── ProfilePage.vue     # Own profile (all roles)
│   ├── UnauthorizedPage.vue    # 403 forbidden page
│   └── NotFoundPage.vue        # 404 page
├── router/
│   ├── index.ts                # Router configuration
│   └── guards.ts               # Navigation guards
├── types/
│   └── auth.ts                 # Auth-related types
├── App.vue
├── main.ts
└── style.css
```

---

## Step-by-Step Guide

### Step 1: Project Setup

```bash
npm create vite@latest protected-admin-panel -- --template vue-ts
cd protected-admin-panel
npm install
npm install vue-router@4
npm install -D tailwindcss postcss autoprefixer daisyui@latest
npx tailwindcss init -p
```

Configure Tailwind:

```javascript
// tailwind.config.js
export default {
  content: ["./index.html", "./src/**/*.{vue,js,ts,jsx,tsx}"],
  theme: { extend: {} },
  plugins: [require('daisyui')],
  daisyui: { themes: ["corporate", "dark", "cupcake"] }
}
```

```css
/* src/style.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

---

### Step 2: Create Auth Types

```typescript
// src/types/auth.ts
export type UserRole = 'admin' | 'editor' | 'viewer'

export interface User {
  id: string
  email: string
  name: string
  role: UserRole
  avatar: string
}

export interface LoginCredentials {
  email: string
  password: string
}

export interface AuthState {
  user: User | null
  token: string | null
  isAuthenticated: boolean
}

// Role hierarchy: admin > editor > viewer
export const ROLE_HIERARCHY: Record<UserRole, number> = {
  admin: 3,
  editor: 2,
  viewer: 1
}

export function hasRequiredRole(userRole: UserRole, requiredRole: UserRole): boolean {
  return ROLE_HIERARCHY[userRole] >= ROLE_HIERARCHY[requiredRole]
}
```

---

### Step 3: Create Auth Composable

```typescript
// src/composables/useAuth.ts
import { ref, computed, readonly } from 'vue'
import { type User, type UserRole, type LoginCredentials, hasRequiredRole } from '../types/auth'

// Mock users database
const MOCK_USERS: (User & { password: string })[] = [
  {
    id: '1',
    email: 'admin@example.com',
    password: 'admin123',
    name: 'Admin User',
    role: 'admin',
    avatar: 'https://i.pravatar.cc/150?img=1'
  },
  {
    id: '2',
    email: 'editor@example.com',
    password: 'editor123',
    name: 'Editor User',
    role: 'editor',
    avatar: 'https://i.pravatar.cc/150?img=2'
  },
  {
    id: '3',
    email: 'viewer@example.com',
    password: 'viewer123',
    name: 'Viewer User',
    role: 'viewer',
    avatar: 'https://i.pravatar.cc/150?img=3'
  }
]

// Global state (singleton)
const user = ref<User | null>(null)
const token = ref<string | null>(null)
const loading = ref(false)
const error = ref<string | null>(null)

// Initialize from localStorage
function initAuth() {
  const storedToken = localStorage.getItem('auth_token')
  const storedUser = localStorage.getItem('auth_user')
  
  if (storedToken && storedUser) {
    token.value = storedToken
    user.value = JSON.parse(storedUser)
  }
}

export function useAuth() {
  // Computed properties
  const isAuthenticated = computed(() => !!token.value && !!user.value)
  const isAdmin = computed(() => user.value?.role === 'admin')
  const isEditor = computed(() => user.value?.role === 'editor' || isAdmin.value)
  const currentRole = computed(() => user.value?.role || null)
  
  // Check if user has required role
  const canAccess = (requiredRole: UserRole): boolean => {
    if (!user.value) return false
    return hasRequiredRole(user.value.role, requiredRole)
  }
  
  // Login
  async function login(credentials: LoginCredentials): Promise<boolean> {
    loading.value = true
    error.value = null
    
    // Simulate API delay
    await new Promise(resolve => setTimeout(resolve, 800))
    
    // Find user
    const foundUser = MOCK_USERS.find(
      u => u.email === credentials.email && u.password === credentials.password
    )
    
    if (!foundUser) {
      error.value = 'Invalid email or password'
      loading.value = false
      return false
    }
    
    // Create mock token
    const mockToken = `mock_token_${foundUser.id}_${Date.now()}`
    
    // Set state
    const { password, ...userWithoutPassword } = foundUser
    user.value = userWithoutPassword
    token.value = mockToken
    
    // Persist to localStorage
    localStorage.setItem('auth_token', mockToken)
    localStorage.setItem('auth_user', JSON.stringify(userWithoutPassword))
    
    loading.value = false
    return true
  }
  
  // Logout
  function logout() {
    user.value = null
    token.value = null
    localStorage.removeItem('auth_token')
    localStorage.removeItem('auth_user')
  }
  
  // Update profile
  async function updateProfile(updates: Partial<User>): Promise<boolean> {
    if (!user.value) return false
    
    loading.value = true
    await new Promise(resolve => setTimeout(resolve, 500))
    
    user.value = { ...user.value, ...updates }
    localStorage.setItem('auth_user', JSON.stringify(user.value))
    
    loading.value = false
    return true
  }
  
  return {
    // State (readonly to prevent direct mutation)
    user: readonly(user),
    token: readonly(token),
    loading: readonly(loading),
    error: readonly(error),
    
    // Computed
    isAuthenticated,
    isAdmin,
    isEditor,
    currentRole,
    
    // Methods
    initAuth,
    login,
    logout,
    updateProfile,
    canAccess
  }
}
```

---

### Step 4: Create Navigation Guards

```typescript
// src/router/guards.ts
import type { NavigationGuardNext, RouteLocationNormalized } from 'vue-router'
import { useAuth } from '../composables/useAuth'
import type { UserRole } from '../types/auth'

// Extend route meta types
declare module 'vue-router' {
  interface RouteMeta {
    requiresAuth?: boolean
    requiredRole?: UserRole
    title?: string
    guestOnly?: boolean
  }
}

export function setupGuards(router: ReturnType<typeof import('vue-router').createRouter>) {
  const { isAuthenticated, canAccess } = useAuth()
  
  // TODO: Implement global beforeEach guard
  router.beforeEach((to, from) => {
    // Update document title
    document.title = `${to.meta.title || 'Page'} | Admin Panel`
    
    // Check if route requires authentication
    if (to.meta.requiresAuth && !isAuthenticated.value) {
      // Redirect to login with return URL
      return {
        name: 'login',
        query: { redirect: to.fullPath }
      }
    }
    
    // Check if route requires specific role
    if (to.meta.requiredRole && !canAccess(to.meta.requiredRole)) {
      // User is logged in but doesn't have permission
      return { name: 'unauthorized' }
    }
    
    // Prevent authenticated users from accessing guest-only routes
    if (to.meta.guestOnly && isAuthenticated.value) {
      return { name: 'admin-dashboard' }
    }
    
    // Allow navigation
  })
  
  // After each navigation
  router.afterEach((to) => {
    // Could add analytics tracking here
    console.log(`Navigated to: ${to.fullPath}`)
  })
}
```

---

### Step 5: Create Router Configuration

```typescript
// src/router/index.ts
import { createRouter, createWebHistory } from 'vue-router'
import { setupGuards } from './guards'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  
  routes: [
    // Public routes
    {
      path: '/',
      name: 'home',
      component: () => import('../pages/public/HomePage.vue'),
      meta: { title: 'Home' }
    },
    
    // Auth routes (guest only)
    {
      path: '/auth',
      component: () => import('../layouts/AuthLayout.vue'),
      children: [
        {
          path: 'login',
          name: 'login',
          component: () => import('../pages/auth/LoginPage.vue'),
          meta: { title: 'Login', guestOnly: true }
        },
        {
          path: 'register',
          name: 'register',
          component: () => import('../pages/auth/RegisterPage.vue'),
          meta: { title: 'Register', guestOnly: true }
        }
      ]
    },
    
    // Admin routes (protected)
    {
      path: '/admin',
      component: () => import('../layouts/AdminLayout.vue'),
      meta: { requiresAuth: true },
      children: [
        {
          path: '',
          name: 'admin-dashboard',
          component: () => import('../pages/admin/DashboardPage.vue'),
          meta: { title: 'Dashboard', requiresAuth: true }
        },
        {
          path: 'users',
          name: 'admin-users',
          component: () => import('../pages/admin/UsersPage.vue'),
          meta: { title: 'Users', requiresAuth: true, requiredRole: 'admin' }
        },
        {
          path: 'users/:id/edit',
          name: 'admin-user-edit',
          component: () => import('../pages/admin/UserEditPage.vue'),
          meta: { title: 'Edit User', requiresAuth: true, requiredRole: 'admin' }
        },
        {
          path: 'content',
          name: 'admin-content',
          component: () => import('../pages/admin/ContentPage.vue'),
          meta: { title: 'Content', requiresAuth: true, requiredRole: 'editor' }
        },
        {
          path: 'settings',
          name: 'admin-settings',
          component: () => import('../pages/admin/SettingsPage.vue'),
          meta: { title: 'Settings', requiresAuth: true, requiredRole: 'admin' }
        },
        {
          path: 'profile',
          name: 'admin-profile',
          component: () => import('../pages/admin/ProfilePage.vue'),
          meta: { title: 'Profile', requiresAuth: true }
        }
      ]
    },
    
    // Error pages
    {
      path: '/unauthorized',
      name: 'unauthorized',
      component: () => import('../pages/UnauthorizedPage.vue'),
      meta: { title: 'Unauthorized' }
    },
    {
      path: '/:pathMatch(.*)*',
      name: 'not-found',
      component: () => import('../pages/NotFoundPage.vue'),
      meta: { title: 'Not Found' }
    }
  ]
})

// Setup navigation guards
setupGuards(router)

export default router
```

---

### Step 6: Update Main Entry Point

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

createApp(App).use(router).mount('#app')
```

---

### Step 7: Create App.vue

```vue
<!-- src/App.vue -->
<template>
  <div class="min-h-screen bg-base-100" data-theme="corporate">
    <RouterView />
  </div>
</template>
```

---

### Step 8: Create Auth Layout

```vue
<!-- src/layouts/AuthLayout.vue -->
<script setup lang="ts">
</script>

<template>
  <div class="min-h-screen flex items-center justify-center bg-gradient-to-br from-primary to-secondary p-4">
    <div class="w-full max-w-md">
      <!-- Logo -->
      <div class="text-center mb-8">
        <RouterLink to="/" class="text-4xl font-bold text-white">
          🔐 AdminPanel
        </RouterLink>
      </div>
      
      <!-- Auth Card -->
      <div class="card bg-base-100 shadow-2xl">
        <div class="card-body">
          <RouterView />
        </div>
      </div>
      
      <!-- Back Link -->
      <div class="text-center mt-6">
        <RouterLink to="/" class="link link-primary text-white">
          ← Back to Home
        </RouterLink>
      </div>
    </div>
  </div>
</template>
```

---

### Step 9: Create Login Page

```vue
<!-- src/pages/auth/LoginPage.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { useAuth } from '../../composables/useAuth'

const router = useRouter()
const route = useRoute()
const { login, loading, error } = useAuth()

const form = ref({
  email: '',
  password: ''
})

async function handleSubmit() {
  const success = await login({
    email: form.value.email,
    password: form.value.password
  })
  
  if (success) {
    // TODO: Redirect to intended destination or default dashboard
    const redirect = route.query.redirect as string
    router.push(redirect || { name: 'admin-dashboard' })
  }
}

// Demo credentials for quick testing
function fillDemo(role: 'admin' | 'editor' | 'viewer') {
  const credentials = {
    admin: { email: 'admin@example.com', password: 'admin123' },
    editor: { email: 'editor@example.com', password: 'editor123' },
    viewer: { email: 'viewer@example.com', password: 'viewer123' }
  }
  form.value = { ...credentials[role] }
}
</script>

<template>
  <div>
    <h2 class="text-2xl font-bold text-center mb-6">Sign In</h2>
    
    <!-- Demo Credentials -->
    <div class="bg-base-200 rounded-lg p-4 mb-6">
      <p class="text-sm font-medium mb-2">Demo Accounts:</p>
      <div class="flex flex-wrap gap-2">
        <button @click="fillDemo('admin')" class="btn btn-xs btn-primary">
          Admin
        </button>
        <button @click="fillDemo('editor')" class="btn btn-xs btn-secondary">
          Editor
        </button>
        <button @click="fillDemo('viewer')" class="btn btn-xs btn-accent">
          Viewer
        </button>
      </div>
    </div>
    
    <!-- Error Alert -->
    <div v-if="error" class="alert alert-error mb-4">
      <span>{{ error }}</span>
    </div>
    
    <!-- Redirect Notice -->
    <div v-if="route.query.redirect" class="alert alert-info mb-4">
      <span>Please log in to access that page.</span>
    </div>
    
    <!-- Login Form -->
    <form @submit.prevent="handleSubmit" class="space-y-4">
      <div class="form-control">
        <label class="label">
          <span class="label-text">Email</span>
        </label>
        <input 
          v-model="form.email"
          type="email" 
          placeholder="you@example.com"
          class="input input-bordered"
          required
        />
      </div>
      
      <div class="form-control">
        <label class="label">
          <span class="label-text">Password</span>
        </label>
        <input 
          v-model="form.password"
          type="password" 
          placeholder="••••••••"
          class="input input-bordered"
          required
        />
      </div>
      
      <button 
        type="submit" 
        class="btn btn-primary w-full"
        :disabled="loading"
      >
        <span v-if="loading" class="loading loading-spinner loading-sm"></span>
        <span v-else>Sign In</span>
      </button>
    </form>
    
    <div class="divider">OR</div>
    
    <p class="text-center text-sm">
      Don't have an account?
      <RouterLink to="/auth/register" class="link link-primary">
        Sign Up
      </RouterLink>
    </p>
  </div>
</template>
```

---

### Step 10: Create Register Page

```vue
<!-- src/pages/auth/RegisterPage.vue -->
<script setup lang="ts">
import { ref } from 'vue'

const form = ref({
  name: '',
  email: '',
  password: '',
  confirmPassword: ''
})

function handleSubmit() {
  // Registration would be implemented here
  alert('Registration is disabled in this demo. Please use the demo accounts.')
}
</script>

<template>
  <div>
    <h2 class="text-2xl font-bold text-center mb-6">Create Account</h2>
    
    <form @submit.prevent="handleSubmit" class="space-y-4">
      <div class="form-control">
        <label class="label">
          <span class="label-text">Name</span>
        </label>
        <input 
          v-model="form.name"
          type="text" 
          placeholder="John Doe"
          class="input input-bordered"
          required
        />
      </div>
      
      <div class="form-control">
        <label class="label">
          <span class="label-text">Email</span>
        </label>
        <input 
          v-model="form.email"
          type="email" 
          placeholder="you@example.com"
          class="input input-bordered"
          required
        />
      </div>
      
      <div class="form-control">
        <label class="label">
          <span class="label-text">Password</span>
        </label>
        <input 
          v-model="form.password"
          type="password" 
          placeholder="••••••••"
          class="input input-bordered"
          required
        />
      </div>
      
      <div class="form-control">
        <label class="label">
          <span class="label-text">Confirm Password</span>
        </label>
        <input 
          v-model="form.confirmPassword"
          type="password" 
          placeholder="••••••••"
          class="input input-bordered"
          required
        />
      </div>
      
      <button type="submit" class="btn btn-primary w-full">
        Create Account
      </button>
    </form>
    
    <div class="divider">OR</div>
    
    <p class="text-center text-sm">
      Already have an account?
      <RouterLink to="/auth/login" class="link link-primary">
        Sign In
      </RouterLink>
    </p>
  </div>
</template>
```

---

### Step 11: Create Admin Layout

```vue
<!-- src/layouts/AdminLayout.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import AdminNav from '../components/AdminNav.vue'
import TopBar from '../components/TopBar.vue'

const sidebarCollapsed = ref(false)

function toggleSidebar() {
  sidebarCollapsed.value = !sidebarCollapsed.value
}
</script>

<template>
  <div class="min-h-screen flex">
    <!-- Sidebar -->
    <AdminNav :collapsed="sidebarCollapsed" />
    
    <!-- Main Content -->
    <div class="flex-1 flex flex-col">
      <!-- Top Bar -->
      <TopBar @toggle-sidebar="toggleSidebar" />
      
      <!-- Page Content -->
      <main class="flex-1 p-6 bg-base-200 overflow-auto">
        <RouterView />
      </main>
    </div>
  </div>
</template>
```

---

### Step 12: Create Admin Navigation

```vue
<!-- src/components/AdminNav.vue -->
<script setup lang="ts">
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import { useAuth } from '../composables/useAuth'

defineProps<{
  collapsed: boolean
}>()

const route = useRoute()
const { canAccess, currentRole } = useAuth()

const menuItems = computed(() => [
  { 
    name: 'admin-dashboard', 
    label: 'Dashboard', 
    icon: '📊',
    show: true 
  },
  { 
    name: 'admin-content', 
    label: 'Content', 
    icon: '📝',
    show: canAccess('editor') 
  },
  { 
    name: 'admin-users', 
    label: 'Users', 
    icon: '👥',
    show: canAccess('admin') 
  },
  { 
    name: 'admin-settings', 
    label: 'Settings', 
    icon: '⚙️',
    show: canAccess('admin') 
  },
  { 
    name: 'admin-profile', 
    label: 'Profile', 
    icon: '👤',
    show: true 
  }
].filter(item => item.show))

function isActive(routeName: string): boolean {
  return route.name === routeName
}
</script>

<template>
  <aside 
    :class="[
      'bg-base-100 border-r border-base-300 transition-all duration-300',
      collapsed ? 'w-16' : 'w-64'
    ]"
  >
    <!-- Logo -->
    <div class="p-4 border-b border-base-300">
      <RouterLink 
        to="/admin" 
        class="flex items-center gap-3"
      >
        <span class="text-2xl">🔐</span>
        <span 
          v-if="!collapsed" 
          class="font-bold text-lg"
        >
          AdminPanel
        </span>
      </RouterLink>
    </div>
    
    <!-- Role Badge -->
    <div v-if="!collapsed" class="p-4 border-b border-base-300">
      <span 
        :class="[
          'badge',
          currentRole === 'admin' ? 'badge-primary' :
          currentRole === 'editor' ? 'badge-secondary' :
          'badge-accent'
        ]"
      >
        {{ currentRole?.toUpperCase() }}
      </span>
    </div>
    
    <!-- Navigation -->
    <nav class="p-2">
      <ul class="menu">
        <li v-for="item in menuItems" :key="item.name">
          <RouterLink 
            :to="{ name: item.name }"
            :class="[
              'flex items-center gap-3 rounded-lg',
              isActive(item.name) && 'active'
            ]"
          >
            <span class="text-xl">{{ item.icon }}</span>
            <span v-if="!collapsed">{{ item.label }}</span>
          </RouterLink>
        </li>
      </ul>
    </nav>
  </aside>
</template>
```

---

### Step 13: Create Top Bar

```vue
<!-- src/components/TopBar.vue -->
<script setup lang="ts">
import { useRouter } from 'vue-router'
import { useAuth } from '../composables/useAuth'

const emit = defineEmits<{
  toggleSidebar: []
}>()

const router = useRouter()
const { user, logout } = useAuth()

function handleLogout() {
  logout()
  router.push({ name: 'login' })
}
</script>

<template>
  <header class="navbar bg-base-100 border-b border-base-300 px-4">
    <!-- Sidebar Toggle -->
    <div class="flex-none">
      <button 
        @click="emit('toggleSidebar')"
        class="btn btn-ghost btn-square"
      >
        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
        </svg>
      </button>
    </div>
    
    <!-- Breadcrumb placeholder -->
    <div class="flex-1">
      <div class="text-sm breadcrumbs">
        <ul>
          <li><RouterLink to="/admin">Admin</RouterLink></li>
          <li>{{ $route.meta.title }}</li>
        </ul>
      </div>
    </div>
    
    <!-- User Menu -->
    <div class="flex-none">
      <div class="dropdown dropdown-end">
        <label tabindex="0" class="btn btn-ghost btn-circle avatar">
          <div class="w-10 rounded-full">
            <img :src="user?.avatar" :alt="user?.name" />
          </div>
        </label>
        <ul tabindex="0" class="dropdown-content z-10 menu p-2 shadow-lg bg-base-100 rounded-box w-52">
          <li class="menu-title">
            <span>{{ user?.name }}</span>
          </li>
          <li><RouterLink :to="{ name: 'admin-profile' }">Profile</RouterLink></li>
          <li><RouterLink to="/">View Site</RouterLink></li>
          <li class="text-error"><a @click="handleLogout">Logout</a></li>
        </ul>
      </div>
    </div>
  </header>
</template>
```

---

### Step 14: Create Stat Card Component

```vue
<!-- src/components/StatCard.vue -->
<script setup lang="ts">
defineProps<{
  title: string
  value: string | number
  change?: string
  icon: string
  color?: 'primary' | 'secondary' | 'accent' | 'success' | 'warning' | 'error'
}>()
</script>

<template>
  <div class="card bg-base-100 shadow-md">
    <div class="card-body">
      <div class="flex items-center justify-between">
        <div>
          <p class="text-sm text-base-content/60">{{ title }}</p>
          <p class="text-3xl font-bold mt-1">{{ value }}</p>
          <p v-if="change" class="text-sm text-success mt-1">
            {{ change }}
          </p>
        </div>
        <div 
          :class="[
            'w-12 h-12 rounded-full flex items-center justify-center text-2xl',
            color ? `bg-${color}/20` : 'bg-primary/20'
          ]"
        >
          {{ icon }}
        </div>
      </div>
    </div>
  </div>
</template>
```

---

### Step 15: Create Dashboard Page

```vue
<!-- src/pages/admin/DashboardPage.vue -->
<script setup lang="ts">
import { useAuth } from '../../composables/useAuth'
import StatCard from '../../components/StatCard.vue'

const { user, currentRole, canAccess } = useAuth()
</script>

<template>
  <div>
    <h1 class="text-3xl font-bold mb-2">Dashboard</h1>
    <p class="text-base-content/60 mb-8">
      Welcome back, {{ user?.name }}!
    </p>
    
    <!-- Stats Grid -->
    <div class="grid sm:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
      <StatCard 
        title="Total Users" 
        value="1,234" 
        change="+12% from last month"
        icon="👥"
        color="primary"
      />
      <StatCard 
        title="Active Sessions" 
        value="89" 
        icon="🟢"
        color="success"
      />
      <StatCard 
        title="Content Items" 
        value="456" 
        change="+5 new today"
        icon="📄"
        color="secondary"
      />
      <StatCard 
        title="Pending Tasks" 
        value="12" 
        icon="📋"
        color="warning"
      />
    </div>
    
    <!-- Role-based content -->
    <div class="grid lg:grid-cols-2 gap-6">
      <!-- Recent Activity (all roles) -->
      <div class="card bg-base-100 shadow-md">
        <div class="card-body">
          <h2 class="card-title">Recent Activity</h2>
          <div class="space-y-4">
            <div v-for="i in 5" :key="i" class="flex items-center gap-4">
              <div class="w-10 h-10 rounded-full bg-base-200 flex items-center justify-center">
                {{ ['📝', '👤', '💬', '🔔', '📊'][i-1] }}
              </div>
              <div class="flex-1">
                <p class="font-medium">Activity item {{ i }}</p>
                <p class="text-sm text-base-content/60">{{ i }} hours ago</p>
              </div>
            </div>
          </div>
        </div>
      </div>
      
      <!-- Quick Actions (role-based) -->
      <div class="card bg-base-100 shadow-md">
        <div class="card-body">
          <h2 class="card-title">Quick Actions</h2>
          <div class="grid grid-cols-2 gap-4">
            <RouterLink 
              :to="{ name: 'admin-profile' }"
              class="btn btn-outline"
            >
              👤 Edit Profile
            </RouterLink>
            
            <RouterLink 
              v-if="canAccess('editor')"
              :to="{ name: 'admin-content' }"
              class="btn btn-outline"
            >
              📝 New Content
            </RouterLink>
            
            <RouterLink 
              v-if="canAccess('admin')"
              :to="{ name: 'admin-users' }"
              class="btn btn-outline"
            >
              👥 Manage Users
            </RouterLink>
            
            <RouterLink 
              v-if="canAccess('admin')"
              :to="{ name: 'admin-settings' }"
              class="btn btn-outline"
            >
              ⚙️ Settings
            </RouterLink>
          </div>
          
          <!-- Role info -->
          <div class="alert mt-4">
            <span>
              Your role: <strong>{{ currentRole }}</strong>.
              {{ currentRole === 'admin' ? 'You have full access.' :
                 currentRole === 'editor' ? 'You can manage content.' :
                 'You have view-only access.' }}
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
```

---

### Step 16: Create Users Page (Admin Only)

```vue
<!-- src/pages/admin/UsersPage.vue -->
<script setup lang="ts">
import { ref } from 'vue'

const users = ref([
  { id: '1', name: 'Admin User', email: 'admin@example.com', role: 'admin', status: 'active' },
  { id: '2', name: 'Editor User', email: 'editor@example.com', role: 'editor', status: 'active' },
  { id: '3', name: 'Viewer User', email: 'viewer@example.com', role: 'viewer', status: 'active' },
  { id: '4', name: 'Jane Doe', email: 'jane@example.com', role: 'editor', status: 'inactive' },
  { id: '5', name: 'Bob Smith', email: 'bob@example.com', role: 'viewer', status: 'active' }
])

function getRoleBadgeClass(role: string): string {
  switch (role) {
    case 'admin': return 'badge-primary'
    case 'editor': return 'badge-secondary'
    default: return 'badge-accent'
  }
}
</script>

<template>
  <div>
    <div class="flex justify-between items-center mb-8">
      <div>
        <h1 class="text-3xl font-bold">Users</h1>
        <p class="text-base-content/60">Manage user accounts and permissions</p>
      </div>
      <button class="btn btn-primary">
        + Add User
      </button>
    </div>
    
    <!-- Users Table -->
    <div class="card bg-base-100 shadow-md">
      <div class="card-body">
        <div class="overflow-x-auto">
          <table class="table">
            <thead>
              <tr>
                <th>User</th>
                <th>Email</th>
                <th>Role</th>
                <th>Status</th>
                <th>Actions</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="user in users" :key="user.id">
                <td>
                  <div class="flex items-center gap-3">
                    <div class="avatar placeholder">
                      <div class="bg-base-200 text-base-content w-10 rounded-full">
                        <span>{{ user.name.charAt(0) }}</span>
                      </div>
                    </div>
                    <span class="font-medium">{{ user.name }}</span>
                  </div>
                </td>
                <td>{{ user.email }}</td>
                <td>
                  <span :class="['badge', getRoleBadgeClass(user.role)]">
                    {{ user.role }}
                  </span>
                </td>
                <td>
                  <span :class="[
                    'badge',
                    user.status === 'active' ? 'badge-success' : 'badge-ghost'
                  ]">
                    {{ user.status }}
                  </span>
                </td>
                <td>
                  <div class="flex gap-2">
                    <RouterLink 
                      :to="{ name: 'admin-user-edit', params: { id: user.id } }"
                      class="btn btn-ghost btn-sm"
                    >
                      Edit
                    </RouterLink>
                    <button class="btn btn-ghost btn-sm text-error">
                      Delete
                    </button>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>
```

---

### Step 17: Create User Edit Page (Admin Only)

```vue
<!-- src/pages/admin/UserEditPage.vue -->
<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import { useRoute, useRouter, onBeforeRouteLeave } from 'vue-router'

const route = useRoute()
const router = useRouter()

const userId = computed(() => route.params.id as string)
const loading = ref(true)
const hasChanges = ref(false)

const form = ref({
  name: '',
  email: '',
  role: 'viewer'
})

// Simulate fetching user
async function fetchUser(id: string) {
  loading.value = true
  await new Promise(resolve => setTimeout(resolve, 500))
  
  // Mock data
  const mockUsers: Record<string, typeof form.value> = {
    '1': { name: 'Admin User', email: 'admin@example.com', role: 'admin' },
    '2': { name: 'Editor User', email: 'editor@example.com', role: 'editor' },
    '3': { name: 'Viewer User', email: 'viewer@example.com', role: 'viewer' }
  }
  
  if (mockUsers[id]) {
    form.value = { ...mockUsers[id] }
  }
  loading.value = false
}

fetchUser(userId.value)

// TODO: Watch for userId changes (component reuse)
watch(userId, (newId) => {
  hasChanges.value = false
  fetchUser(newId)
})

// TODO: Implement route leave guard
onBeforeRouteLeave((to, from) => {
  if (hasChanges.value) {
    return window.confirm('You have unsaved changes. Discard them?')
  }
})

function handleSave() {
  hasChanges.value = false
  alert('User saved!')
  router.push({ name: 'admin-users' })
}
</script>

<template>
  <div>
    <div class="flex items-center gap-4 mb-8">
      <RouterLink :to="{ name: 'admin-users' }" class="btn btn-ghost">
        ← Back
      </RouterLink>
      <div>
        <h1 class="text-3xl font-bold">Edit User</h1>
        <p class="text-base-content/60">User ID: {{ userId }}</p>
      </div>
    </div>
    
    <div v-if="loading" class="flex justify-center py-12">
      <span class="loading loading-spinner loading-lg"></span>
    </div>
    
    <div v-else class="card bg-base-100 shadow-md max-w-xl">
      <div class="card-body">
        <!-- Unsaved Warning -->
        <div v-if="hasChanges" class="alert alert-warning mb-4">
          You have unsaved changes
        </div>
        
        <form @submit.prevent="handleSave" class="space-y-4">
          <div class="form-control">
            <label class="label">
              <span class="label-text">Name</span>
            </label>
            <input 
              v-model="form.name"
              @input="hasChanges = true"
              type="text" 
              class="input input-bordered"
            />
          </div>
          
          <div class="form-control">
            <label class="label">
              <span class="label-text">Email</span>
            </label>
            <input 
              v-model="form.email"
              @input="hasChanges = true"
              type="email" 
              class="input input-bordered"
            />
          </div>
          
          <div class="form-control">
            <label class="label">
              <span class="label-text">Role</span>
            </label>
            <select 
              v-model="form.role"
              @change="hasChanges = true"
              class="select select-bordered"
            >
              <option value="viewer">Viewer</option>
              <option value="editor">Editor</option>
              <option value="admin">Admin</option>
            </select>
          </div>
          
          <div class="flex gap-4 pt-4">
            <button type="submit" class="btn btn-primary">
              Save Changes
            </button>
            <RouterLink :to="{ name: 'admin-users' }" class="btn btn-ghost">
              Cancel
            </RouterLink>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>
```

---

### Step 18: Create Content Page (Editor+ Only)

```vue
<!-- src/pages/admin/ContentPage.vue -->
<script setup lang="ts">
import { ref } from 'vue'

const content = ref([
  { id: '1', title: 'Getting Started Guide', type: 'article', status: 'published', date: '2024-01-20' },
  { id: '2', title: 'Product Announcement', type: 'post', status: 'draft', date: '2024-01-19' },
  { id: '3', title: 'Team Updates', type: 'article', status: 'published', date: '2024-01-18' },
  { id: '4', title: 'FAQ Page', type: 'page', status: 'published', date: '2024-01-15' }
])
</script>

<template>
  <div>
    <div class="flex justify-between items-center mb-8">
      <div>
        <h1 class="text-3xl font-bold">Content</h1>
        <p class="text-base-content/60">Manage articles, posts, and pages</p>
      </div>
      <button class="btn btn-primary">
        + New Content
      </button>
    </div>
    
    <div class="card bg-base-100 shadow-md">
      <div class="card-body">
        <div class="overflow-x-auto">
          <table class="table">
            <thead>
              <tr>
                <th>Title</th>
                <th>Type</th>
                <th>Status</th>
                <th>Date</th>
                <th>Actions</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in content" :key="item.id">
                <td class="font-medium">{{ item.title }}</td>
                <td>
                  <span class="badge badge-ghost">{{ item.type }}</span>
                </td>
                <td>
                  <span :class="[
                    'badge',
                    item.status === 'published' ? 'badge-success' : 'badge-warning'
                  ]">
                    {{ item.status }}
                  </span>
                </td>
                <td>{{ item.date }}</td>
                <td>
                  <div class="flex gap-2">
                    <button class="btn btn-ghost btn-sm">Edit</button>
                    <button class="btn btn-ghost btn-sm text-error">Delete</button>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>
```

---

### Step 19: Create Settings Page (Admin Only)

```vue
<!-- src/pages/admin/SettingsPage.vue -->
<script setup lang="ts">
import { ref } from 'vue'

const settings = ref({
  siteName: 'Admin Panel',
  siteDescription: 'A secure admin dashboard',
  maintenanceMode: false,
  allowRegistration: true,
  defaultRole: 'viewer'
})

function handleSave() {
  alert('Settings saved!')
}
</script>

<template>
  <div>
    <h1 class="text-3xl font-bold mb-2">Settings</h1>
    <p class="text-base-content/60 mb-8">Configure application settings</p>
    
    <div class="card bg-base-100 shadow-md max-w-2xl">
      <div class="card-body">
        <form @submit.prevent="handleSave" class="space-y-6">
          <!-- General Settings -->
          <h3 class="font-bold text-lg">General</h3>
          
          <div class="form-control">
            <label class="label">
              <span class="label-text">Site Name</span>
            </label>
            <input 
              v-model="settings.siteName"
              type="text" 
              class="input input-bordered"
            />
          </div>
          
          <div class="form-control">
            <label class="label">
              <span class="label-text">Site Description</span>
            </label>
            <textarea 
              v-model="settings.siteDescription"
              class="textarea textarea-bordered"
            ></textarea>
          </div>
          
          <div class="divider"></div>
          
          <!-- Access Settings -->
          <h3 class="font-bold text-lg">Access Control</h3>
          
          <div class="form-control">
            <label class="label cursor-pointer justify-start gap-4">
              <input 
                v-model="settings.maintenanceMode"
                type="checkbox" 
                class="toggle toggle-warning"
              />
              <span class="label-text">Maintenance Mode</span>
            </label>
          </div>
          
          <div class="form-control">
            <label class="label cursor-pointer justify-start gap-4">
              <input 
                v-model="settings.allowRegistration"
                type="checkbox" 
                class="toggle toggle-primary"
              />
              <span class="label-text">Allow New Registrations</span>
            </label>
          </div>
          
          <div class="form-control">
            <label class="label">
              <span class="label-text">Default Role for New Users</span>
            </label>
            <select 
              v-model="settings.defaultRole"
              class="select select-bordered"
            >
              <option value="viewer">Viewer</option>
              <option value="editor">Editor</option>
            </select>
          </div>
          
          <button type="submit" class="btn btn-primary">
            Save Settings
          </button>
        </form>
      </div>
    </div>
  </div>
</template>
```

---

### Step 20: Create Profile Page

```vue
<!-- src/pages/admin/ProfilePage.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import { onBeforeRouteLeave } from 'vue-router'
import { useAuth } from '../../composables/useAuth'

const { user, updateProfile, loading } = useAuth()

const form = ref({
  name: user.value?.name || '',
  email: user.value?.email || ''
})

const hasChanges = ref(false)
const showSuccess = ref(false)

onBeforeRouteLeave(() => {
  if (hasChanges.value) {
    return window.confirm('You have unsaved changes. Leave anyway?')
  }
})

async function handleSave() {
  await updateProfile({
    name: form.value.name,
    email: form.value.email
  })
  hasChanges.value = false
  showSuccess.value = true
  setTimeout(() => showSuccess.value = false, 3000)
}
</script>

<template>
  <div>
    <h1 class="text-3xl font-bold mb-2">Profile</h1>
    <p class="text-base-content/60 mb-8">Manage your account settings</p>
    
    <div class="grid lg:grid-cols-3 gap-6">
      <!-- Profile Card -->
      <div class="card bg-base-100 shadow-md">
        <div class="card-body items-center text-center">
          <div class="avatar">
            <div class="w-24 rounded-full ring ring-primary ring-offset-base-100 ring-offset-2">
              <img :src="user?.avatar" :alt="user?.name" />
            </div>
          </div>
          <h2 class="card-title mt-4">{{ user?.name }}</h2>
          <p class="text-base-content/60">{{ user?.email }}</p>
          <div class="badge badge-primary mt-2">
            {{ user?.role?.toUpperCase() }}
          </div>
        </div>
      </div>
      
      <!-- Edit Form -->
      <div class="card bg-base-100 shadow-md lg:col-span-2">
        <div class="card-body">
          <h3 class="font-bold text-lg mb-4">Edit Profile</h3>
          
          <div v-if="showSuccess" class="alert alert-success mb-4">
            Profile updated successfully!
          </div>
          
          <div v-if="hasChanges" class="alert alert-warning mb-4">
            You have unsaved changes
          </div>
          
          <form @submit.prevent="handleSave" class="space-y-4">
            <div class="form-control">
              <label class="label">
                <span class="label-text">Name</span>
              </label>
              <input 
                v-model="form.name"
                @input="hasChanges = true"
                type="text" 
                class="input input-bordered"
              />
            </div>
            
            <div class="form-control">
              <label class="label">
                <span class="label-text">Email</span>
              </label>
              <input 
                v-model="form.email"
                @input="hasChanges = true"
                type="email" 
                class="input input-bordered"
              />
            </div>
            
            <div class="form-control">
              <label class="label">
                <span class="label-text">Role</span>
              </label>
              <input 
                :value="user?.role"
                type="text" 
                class="input input-bordered"
                disabled
              />
              <label class="label">
                <span class="label-text-alt text-base-content/60">
                  Contact an admin to change your role
                </span>
              </label>
            </div>
            
            <button 
              type="submit" 
              class="btn btn-primary"
              :disabled="!hasChanges || loading"
            >
              <span v-if="loading" class="loading loading-spinner loading-sm"></span>
              Save Changes
            </button>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>
```

---

### Step 21: Create Error Pages

```vue
<!-- src/pages/UnauthorizedPage.vue -->
<script setup lang="ts">
import { useAuth } from '../composables/useAuth'

const { currentRole } = useAuth()
</script>

<template>
  <div class="min-h-screen flex items-center justify-center bg-base-200 px-4">
    <div class="text-center">
      <h1 class="text-9xl font-bold text-error">403</h1>
      <h2 class="text-3xl font-bold mt-4 mb-2">Access Denied</h2>
      <p class="text-base-content/70 mb-4">
        You don't have permission to access this page.
      </p>
      <p class="text-sm text-base-content/60 mb-8">
        Your current role: <strong>{{ currentRole }}</strong>
      </p>
      <div class="flex gap-4 justify-center">
        <RouterLink :to="{ name: 'admin-dashboard' }" class="btn btn-primary">
          Go to Dashboard
        </RouterLink>
        <RouterLink to="/" class="btn btn-ghost">
          Back to Home
        </RouterLink>
      </div>
    </div>
  </div>
</template>
```

```vue
<!-- src/pages/NotFoundPage.vue -->
<script setup lang="ts">
import { useRoute } from 'vue-router'

const route = useRoute()
</script>

<template>
  <div class="min-h-screen flex items-center justify-center bg-base-200 px-4">
    <div class="text-center">
      <h1 class="text-9xl font-bold text-primary">404</h1>
      <h2 class="text-3xl font-bold mt-4 mb-2">Page Not Found</h2>
      <p class="text-base-content/70 mb-8">
        The page <code class="bg-base-300 px-2 py-1 rounded">{{ route.fullPath }}</code> doesn't exist.
      </p>
      <div class="flex gap-4 justify-center">
        <RouterLink to="/" class="btn btn-primary">
          Go Home
        </RouterLink>
      </div>
    </div>
  </div>
</template>
```

---

### Step 22: Create Public Home Page

```vue
<!-- src/pages/public/HomePage.vue -->
<script setup lang="ts">
import { useAuth } from '../../composables/useAuth'

const { isAuthenticated } = useAuth()
</script>

<template>
  <div class="min-h-screen">
    <!-- Hero -->
    <section class="hero min-h-screen bg-gradient-to-br from-primary to-secondary text-primary-content">
      <div class="hero-content text-center">
        <div class="max-w-lg">
          <h1 class="text-6xl font-bold mb-6">🔐 AdminPanel</h1>
          <p class="text-xl mb-8 opacity-90">
            A secure, role-based admin dashboard built with Vue 3 and Vue Router.
            Features authentication, navigation guards, and access control.
          </p>
          <div class="flex gap-4 justify-center">
            <RouterLink 
              v-if="isAuthenticated"
              :to="{ name: 'admin-dashboard' }"
              class="btn btn-lg bg-white text-primary hover:bg-gray-100"
            >
              Go to Dashboard
            </RouterLink>
            <RouterLink 
              v-else
              :to="{ name: 'login' }"
              class="btn btn-lg bg-white text-primary hover:bg-gray-100"
            >
              Sign In
            </RouterLink>
            <RouterLink 
              v-if="!isAuthenticated"
              :to="{ name: 'register' }"
              class="btn btn-lg btn-outline border-white text-white hover:bg-white hover:text-primary"
            >
              Sign Up
            </RouterLink>
          </div>
        </div>
      </div>
    </section>
    
    <!-- Features -->
    <section class="py-20 px-4 bg-base-100">
      <div class="container mx-auto">
        <h2 class="text-3xl font-bold text-center mb-12">Features</h2>
        <div class="grid md:grid-cols-3 gap-8">
          <div class="card bg-base-200 shadow-md">
            <div class="card-body">
              <div class="text-4xl mb-4">🛡️</div>
              <h3 class="card-title">Route Protection</h3>
              <p>Navigation guards prevent unauthorized access to protected pages.</p>
            </div>
          </div>
          <div class="card bg-base-200 shadow-md">
            <div class="card-body">
              <div class="text-4xl mb-4">👥</div>
              <h3 class="card-title">Role-Based Access</h3>
              <p>Admin, Editor, and Viewer roles with hierarchical permissions.</p>
            </div>
          </div>
          <div class="card bg-base-200 shadow-md">
            <div class="card-body">
              <div class="text-4xl mb-4">🔄</div>
              <h3 class="card-title">Redirect Handling</h3>
              <p>Users are redirected back to their intended destination after login.</p>
            </div>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>
```

---

## Complete Solution

All files above form the complete solution.

---

## How to Test

### Authentication Flow
1. Visit `/admin` while logged out → Redirects to `/auth/login?redirect=/admin`
2. Login with any demo account → Redirects back to `/admin`
3. Logout → Redirects to login page

### Role-Based Access
| Route | Admin | Editor | Viewer |
|:------|:-----:|:------:|:------:|
| `/admin` (Dashboard) | ✅ | ✅ | ✅ |
| `/admin/content` | ✅ | ✅ | ❌ (403) |
| `/admin/users` | ✅ | ❌ (403) | ❌ (403) |
| `/admin/settings` | ✅ | ❌ (403) | ❌ (403) |
| `/admin/profile` | ✅ | ✅ | ✅ |

### Navigation Guards
1. Edit a user, make changes, try to navigate away → Confirmation dialog
2. Edit profile, make changes, try to navigate away → Confirmation dialog
3. Visit `/admin/users/1/edit`, then `/admin/users/2/edit` → Data refreshes (param watching)

### Guest-Only Routes
1. Login, then visit `/auth/login` → Redirects to dashboard
2. Login, then visit `/auth/register` → Redirects to dashboard

---

## Stretch Goals

1. **Remember me:** Add "remember me" checkbox that uses longer-lived tokens
2. **Session timeout:** Automatically logout after period of inactivity
3. **Password change:** Add password change functionality with confirmation
4. **Activity logging:** Track user actions and display in admin audit log
5. **Two-factor auth:** Add 2FA simulation with code verification

---

## Self-Assessment Checklist

- [ ] Global `beforeEach` guard checks authentication on every navigation
- [ ] Route `meta` fields store `requiresAuth` and `requiredRole`
- [ ] Unauthorized users are redirected to login with `redirect` query param
- [ ] After login, users are redirected to their intended destination
- [ ] Role hierarchy is enforced (admin > editor > viewer)
- [ ] 403 page shown when user lacks required role
- [ ] `guestOnly` routes redirect authenticated users away
- [ ] `onBeforeRouteLeave` warns about unsaved changes
- [ ] Auth state persists in localStorage across page refreshes
- [ ] Param changes trigger data refetch (component reuse)
