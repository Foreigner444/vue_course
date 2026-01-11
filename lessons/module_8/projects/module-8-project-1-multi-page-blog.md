# Guided Project 1: Multi-Page Blog Layout

## Project Overview

| Field | Value |
|:--- |:--- |
| **Module** | Module 8: Navigation |
| **Project Number** | 1 of 3 |
| **Difficulty** | Beginner |
| **Estimated Time** | 45-60 minutes |

**What You'll Build:** A content website with multiple pages including a home page, about page, and dynamic blog post pages. Users can navigate between pages using a responsive navigation bar with active link highlighting.

**Real-World Scenario:** A content site has home, about, and dynamic blog post pages with navigation — similar to a company blog, personal portfolio, or documentation site.

---

## Patterns You'll Practice

| Pattern | Description |
|:--- |:--- |
| `createRouter()` | Create and configure the Vue Router instance |
| `createWebHistory()` | Enable clean URLs with HTML5 History API |
| Route definitions | Map URL paths to Vue components |
| `<RouterView>` | Render matched route components |
| `<RouterLink>` | Navigate without page reloads |
| Active link styling | Highlight current page in navigation |
| Lazy loading routes | Load components on demand with dynamic imports |
| 404 catch-all route | Handle unmatched URLs gracefully |

---

## Component Structure

```
src/
├── components/
│   ├── NavBar.vue          # Site navigation with RouterLinks
│   └── Footer.vue          # Site footer
├── pages/
│   ├── HomePage.vue        # Landing page with hero section
│   ├── AboutPage.vue       # About/team information
│   ├── BlogPage.vue        # List of blog posts
│   ├── BlogPostPage.vue    # Individual blog post (dynamic route)
│   └── NotFoundPage.vue    # 404 error page
├── data/
│   └── posts.ts            # Mock blog post data
├── router.ts               # Router configuration
├── App.vue                 # Root component with RouterView
├── main.ts                 # App entry point
└── style.css               # Tailwind CSS imports
```

---

## Step-by-Step Guide

### Step 1: Project Setup

Create a new Vite project with Vue and TypeScript:

```bash
npm create vite@latest multi-page-blog -- --template vue-ts
cd multi-page-blog
npm install
npm install vue-router@4
npm install -D tailwindcss postcss autoprefixer daisyui@latest
npx tailwindcss init -p
```

Configure Tailwind CSS:

```javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{vue,js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [
    require('daisyui'),
  ],
  daisyui: {
    themes: ["light", "dark", "corporate"],
  },
}
```

```css
/* src/style.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

---

### Step 2: Create Mock Blog Data

```typescript
// src/data/posts.ts
export interface BlogPost {
  id: string
  slug: string
  title: string
  excerpt: string
  content: string
  author: string
  date: string
  tags: string[]
  imageUrl: string
}

export const posts: BlogPost[] = [
  {
    id: '1',
    slug: 'getting-started-with-vue-3',
    title: 'Getting Started with Vue 3',
    excerpt: 'Learn the fundamentals of Vue 3 and the Composition API in this comprehensive beginner guide.',
    content: `
      <p>Vue 3 represents a major evolution in the Vue.js framework, bringing improved performance, 
      better TypeScript support, and the powerful Composition API.</p>
      
      <h2>Why Vue 3?</h2>
      <p>Vue 3 offers several advantages over its predecessor:</p>
      <ul>
        <li>Smaller bundle size</li>
        <li>Better performance with the new reactivity system</li>
        <li>Improved TypeScript integration</li>
        <li>The Composition API for better code organization</li>
      </ul>
      
      <h2>Getting Started</h2>
      <p>To create a new Vue 3 project, you can use Vite:</p>
      <pre><code>npm create vite@latest my-app -- --template vue-ts</code></pre>
      
      <p>This will scaffold a new project with Vue 3 and TypeScript configured out of the box.</p>
    `,
    author: 'Jane Developer',
    date: '2024-01-15',
    tags: ['Vue', 'JavaScript', 'Frontend'],
    imageUrl: 'https://images.unsplash.com/photo-1555066931-4365d14bab8c?w=800'
  },
  {
    id: '2',
    slug: 'mastering-vue-router',
    title: 'Mastering Vue Router',
    excerpt: 'Deep dive into Vue Router 4: navigation, dynamic routes, guards, and best practices.',
    content: `
      <p>Vue Router is the official routing library for Vue.js applications. It enables 
      navigation between different views without page reloads.</p>
      
      <h2>Core Concepts</h2>
      <p>Understanding these fundamentals is essential:</p>
      <ul>
        <li><strong>Routes:</strong> Map URLs to components</li>
        <li><strong>RouterView:</strong> Renders the matched component</li>
        <li><strong>RouterLink:</strong> Declarative navigation</li>
        <li><strong>Navigation Guards:</strong> Control access to routes</li>
      </ul>
      
      <h2>Dynamic Routes</h2>
      <p>Dynamic segments allow you to match patterns:</p>
      <pre><code>{ path: '/user/:id', component: UserProfile }</code></pre>
      
      <p>This matches /user/1, /user/2, /user/abc, etc.</p>
    `,
    author: 'John Coder',
    date: '2024-01-22',
    tags: ['Vue Router', 'Vue', 'SPA'],
    imageUrl: 'https://images.unsplash.com/photo-1517694712202-14dd9538aa97?w=800'
  },
  {
    id: '3',
    slug: 'state-management-with-pinia',
    title: 'State Management with Pinia',
    excerpt: 'Explore Pinia, the intuitive and type-safe state management solution for Vue applications.',
    content: `
      <p>Pinia is the recommended state management library for Vue 3, offering a simpler 
      and more intuitive API compared to Vuex.</p>
      
      <h2>Why Pinia?</h2>
      <ul>
        <li>First-class TypeScript support</li>
        <li>No mutations, only actions</li>
        <li>Devtools integration</li>
        <li>Hot module replacement</li>
        <li>Plugin system for extensibility</li>
      </ul>
      
      <h2>Creating a Store</h2>
      <pre><code>import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  state: () => ({ count: 0 }),
  actions: {
    increment() { this.count++ }
  }
})</code></pre>
    `,
    author: 'Jane Developer',
    date: '2024-02-01',
    tags: ['Pinia', 'Vue', 'State Management'],
    imageUrl: 'https://images.unsplash.com/photo-1516116216624-53e697fedbea?w=800'
  },
  {
    id: '4',
    slug: 'building-responsive-layouts',
    title: 'Building Responsive Layouts with Tailwind CSS',
    excerpt: 'Create beautiful, responsive designs using Tailwind CSS utility classes and DaisyUI components.',
    content: `
      <p>Tailwind CSS is a utility-first CSS framework that enables rapid UI development 
      without leaving your HTML.</p>
      
      <h2>Responsive Design</h2>
      <p>Tailwind makes responsive design intuitive with breakpoint prefixes:</p>
      <pre><code>&lt;div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3"&gt;</code></pre>
      
      <h2>DaisyUI Components</h2>
      <p>DaisyUI adds component classes on top of Tailwind:</p>
      <ul>
        <li>Buttons: <code>btn btn-primary</code></li>
        <li>Cards: <code>card bg-base-100</code></li>
        <li>Navigation: <code>navbar bg-base-200</code></li>
      </ul>
    `,
    author: 'Alex Designer',
    date: '2024-02-10',
    tags: ['CSS', 'Tailwind', 'Design'],
    imageUrl: 'https://images.unsplash.com/photo-1507003211169-0a1dd7228f2d?w=800'
  }
]

export function getPostBySlug(slug: string): BlogPost | undefined {
  return posts.find(post => post.slug === slug)
}

export function getRecentPosts(count: number = 3): BlogPost[] {
  return [...posts]
    .sort((a, b) => new Date(b.date).getTime() - new Date(a.date).getTime())
    .slice(0, count)
}
```

---

### Step 3: Create Router Configuration

```typescript
// src/router.ts
// TODO: Import createRouter and createWebHistory from vue-router
import { createRouter, createWebHistory } from 'vue-router'

// TODO: Create the router with history mode and routes array
const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      return savedPosition
    }
    return { top: 0 }
  },
  
  routes: [
    // TODO: Add route for home page at '/'
    {
      path: '/',
      name: 'home',
      component: () => import('./pages/HomePage.vue'),
      meta: { title: 'Home' }
    },
    
    // TODO: Add route for about page at '/about'
    {
      path: '/about',
      name: 'about',
      component: () => import('./pages/AboutPage.vue'),
      meta: { title: 'About Us' }
    },
    
    // TODO: Add route for blog listing at '/blog'
    {
      path: '/blog',
      name: 'blog',
      component: () => import('./pages/BlogPage.vue'),
      meta: { title: 'Blog' }
    },
    
    // TODO: Add dynamic route for blog posts at '/blog/:slug'
    {
      path: '/blog/:slug',
      name: 'blog-post',
      component: () => import('./pages/BlogPostPage.vue'),
      meta: { title: 'Blog Post' }
    },
    
    // TODO: Add catch-all 404 route
    {
      path: '/:pathMatch(.*)*',
      name: 'not-found',
      component: () => import('./pages/NotFoundPage.vue'),
      meta: { title: 'Page Not Found' }
    }
  ]
})

router.afterEach((to) => {
  document.title = `${to.meta.title} | Vue Blog` || 'Vue Blog'
})

export default router
```

---

### Step 4: Update Main Entry Point

```typescript
// src/main.ts
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import './style.css'

const app = createApp(App)

// TODO: Register the router with app.use()
app.use(router)

app.mount('#app')
```

---

### Step 5: Create Navigation Component

```vue
<!-- src/components/NavBar.vue -->
<script setup lang="ts">
import { ref } from 'vue'

const mobileMenuOpen = ref(false)

function toggleMobileMenu() {
  mobileMenuOpen.value = !mobileMenuOpen.value
}

function closeMobileMenu() {
  mobileMenuOpen.value = false
}
</script>

<template>
  <header class="navbar bg-base-200 shadow-lg sticky top-0 z-50">
    <div class="container mx-auto">
      <!-- Logo -->
      <div class="flex-1">
        <RouterLink 
          to="/" 
          class="btn btn-ghost text-xl font-bold"
          @click="closeMobileMenu"
        >
          📝 Vue Blog
        </RouterLink>
      </div>
      
      <!-- Desktop Navigation -->
      <nav class="flex-none hidden md:block">
        <ul class="menu menu-horizontal px-1 gap-1">
          <li>
            <!-- TODO: Use RouterLink with active styling -->
            <RouterLink 
              to="/"
              class="btn btn-ghost btn-sm"
              active-class="btn-primary"
              exact
            >
              Home
            </RouterLink>
          </li>
          <li>
            <RouterLink 
              to="/blog"
              class="btn btn-ghost btn-sm"
              active-class="btn-primary"
            >
              Blog
            </RouterLink>
          </li>
          <li>
            <RouterLink 
              to="/about"
              class="btn btn-ghost btn-sm"
              active-class="btn-primary"
            >
              About
            </RouterLink>
          </li>
        </ul>
      </nav>
      
      <!-- Mobile Menu Button -->
      <div class="flex-none md:hidden">
        <button 
          class="btn btn-ghost btn-square"
          @click="toggleMobileMenu"
          aria-label="Toggle menu"
        >
          <svg 
            v-if="!mobileMenuOpen"
            xmlns="http://www.w3.org/2000/svg" 
            class="h-6 w-6" 
            fill="none" 
            viewBox="0 0 24 24" 
            stroke="currentColor"
          >
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
          </svg>
          <svg 
            v-else
            xmlns="http://www.w3.org/2000/svg" 
            class="h-6 w-6" 
            fill="none" 
            viewBox="0 0 24 24" 
            stroke="currentColor"
          >
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
          </svg>
        </button>
      </div>
    </div>
    
    <!-- Mobile Menu Dropdown -->
    <div 
      v-if="mobileMenuOpen"
      class="absolute top-full left-0 right-0 bg-base-200 shadow-lg md:hidden"
    >
      <ul class="menu p-4">
        <li>
          <RouterLink to="/" @click="closeMobileMenu">
            Home
          </RouterLink>
        </li>
        <li>
          <RouterLink to="/blog" @click="closeMobileMenu">
            Blog
          </RouterLink>
        </li>
        <li>
          <RouterLink to="/about" @click="closeMobileMenu">
            About
          </RouterLink>
        </li>
      </ul>
    </div>
  </header>
</template>

<style scoped>
.router-link-active.btn-ghost {
  @apply bg-primary text-primary-content;
}
</style>
```

---

### Step 6: Create Footer Component

```vue
<!-- src/components/Footer.vue -->
<script setup lang="ts">
const currentYear = new Date().getFullYear()
</script>

<template>
  <footer class="footer footer-center p-10 bg-base-200 text-base-content">
    <nav class="grid grid-flow-col gap-4">
      <RouterLink to="/about" class="link link-hover">About</RouterLink>
      <RouterLink to="/blog" class="link link-hover">Blog</RouterLink>
      <a href="#" class="link link-hover">Contact</a>
    </nav>
    <nav class="grid grid-flow-col gap-4">
      <a href="https://github.com" target="_blank" class="link link-hover">
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="currentColor">
          <path d="M12 0c-6.626 0-12 5.373-12 12 0 5.302 3.438 9.8 8.207 11.387.599.111.793-.261.793-.577v-2.234c-3.338.726-4.033-1.416-4.033-1.416-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.839 1.237 1.839 1.237 1.07 1.834 2.807 1.304 3.492.997.107-.775.418-1.305.762-1.604-2.665-.305-5.467-1.334-5.467-5.931 0-1.311.469-2.381 1.236-3.221-.124-.303-.535-1.524.117-3.176 0 0 1.008-.322 3.301 1.23.957-.266 1.983-.399 3.003-.404 1.02.005 2.047.138 3.006.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.874.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.609-2.807 5.624-5.479 5.921.43.372.823 1.102.823 2.222v3.293c0 .319.192.694.801.576 4.765-1.589 8.199-6.086 8.199-11.386 0-6.627-5.373-12-12-12z"/>
        </svg>
      </a>
      <a href="https://twitter.com" target="_blank" class="link link-hover">
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="currentColor">
          <path d="M24 4.557c-.883.392-1.832.656-2.828.775 1.017-.609 1.798-1.574 2.165-2.724-.951.564-2.005.974-3.127 1.195-.897-.957-2.178-1.555-3.594-1.555-3.179 0-5.515 2.966-4.797 6.045-4.091-.205-7.719-2.165-10.148-5.144-1.29 2.213-.669 5.108 1.523 6.574-.806-.026-1.566-.247-2.229-.616-.054 2.281 1.581 4.415 3.949 4.89-.693.188-1.452.232-2.224.084.626 1.956 2.444 3.379 4.6 3.419-2.07 1.623-4.678 2.348-7.29 2.04 2.179 1.397 4.768 2.212 7.548 2.212 9.142 0 14.307-7.721 13.995-14.646.962-.695 1.797-1.562 2.457-2.549z"/>
        </svg>
      </a>
    </nav>
    <aside>
      <p>© {{ currentYear }} Vue Blog. Built with Vue 3 + Vue Router.</p>
    </aside>
  </footer>
</template>
```

---

### Step 7: Create App.vue

```vue
<!-- src/App.vue -->
<script setup lang="ts">
import NavBar from './components/NavBar.vue'
import Footer from './components/Footer.vue'
</script>

<template>
  <div class="min-h-screen flex flex-col bg-base-100" data-theme="corporate">
    <!-- Navigation -->
    <NavBar />
    
    <!-- Main Content Area -->
    <!-- TODO: Add RouterView where page components will render -->
    <main class="flex-1">
      <RouterView />
    </main>
    
    <!-- Footer -->
    <Footer />
  </div>
</template>
```

---

### Step 8: Create Home Page

```vue
<!-- src/pages/HomePage.vue -->
<script setup lang="ts">
import { getRecentPosts } from '../data/posts'

const recentPosts = getRecentPosts(3)
</script>

<template>
  <div>
    <!-- Hero Section -->
    <section class="hero min-h-[70vh] bg-gradient-to-br from-primary to-secondary text-primary-content">
      <div class="hero-content text-center">
        <div class="max-w-2xl">
          <h1 class="text-5xl md:text-6xl font-bold mb-6">
            Welcome to Vue Blog
          </h1>
          <p class="text-xl mb-8 opacity-90">
            Discover articles about Vue.js, web development, and modern frontend technologies. 
            Learn from practical examples and level up your skills.
          </p>
          <div class="flex gap-4 justify-center flex-wrap">
            <RouterLink to="/blog" class="btn btn-lg bg-white text-primary hover:bg-gray-100">
              Read Articles
            </RouterLink>
            <RouterLink to="/about" class="btn btn-lg btn-outline border-white text-white hover:bg-white hover:text-primary">
              About Us
            </RouterLink>
          </div>
        </div>
      </div>
    </section>
    
    <!-- Recent Posts Section -->
    <section class="py-16 px-4">
      <div class="container mx-auto">
        <h2 class="text-3xl font-bold text-center mb-12">Recent Articles</h2>
        
        <div class="grid md:grid-cols-2 lg:grid-cols-3 gap-8">
          <article 
            v-for="post in recentPosts" 
            :key="post.id"
            class="card bg-base-200 shadow-xl hover:shadow-2xl transition-shadow"
          >
            <figure>
              <img 
                :src="post.imageUrl" 
                :alt="post.title"
                class="h-48 w-full object-cover"
              />
            </figure>
            <div class="card-body">
              <div class="flex gap-2 flex-wrap">
                <span 
                  v-for="tag in post.tags.slice(0, 2)" 
                  :key="tag"
                  class="badge badge-primary badge-sm"
                >
                  {{ tag }}
                </span>
              </div>
              <h3 class="card-title">{{ post.title }}</h3>
              <p class="text-base-content/70">{{ post.excerpt }}</p>
              <div class="card-actions justify-between items-center mt-4">
                <span class="text-sm text-base-content/50">{{ post.date }}</span>
                <!-- TODO: Use RouterLink with dynamic :to for blog post -->
                <RouterLink 
                  :to="{ name: 'blog-post', params: { slug: post.slug } }"
                  class="btn btn-primary btn-sm"
                >
                  Read More →
                </RouterLink>
              </div>
            </div>
          </article>
        </div>
        
        <div class="text-center mt-12">
          <RouterLink to="/blog" class="btn btn-outline btn-lg">
            View All Articles
          </RouterLink>
        </div>
      </div>
    </section>
    
    <!-- Features Section -->
    <section class="py-16 px-4 bg-base-200">
      <div class="container mx-auto">
        <h2 class="text-3xl font-bold text-center mb-12">What You'll Learn</h2>
        
        <div class="grid md:grid-cols-3 gap-8">
          <div class="card bg-base-100 shadow-md">
            <div class="card-body items-center text-center">
              <div class="text-5xl mb-4">⚡</div>
              <h3 class="card-title">Vue 3 Fundamentals</h3>
              <p>Master the Composition API, reactivity, and component patterns.</p>
            </div>
          </div>
          
          <div class="card bg-base-100 shadow-md">
            <div class="card-body items-center text-center">
              <div class="text-5xl mb-4">🛣️</div>
              <h3 class="card-title">Vue Router</h3>
              <p>Build SPAs with navigation, dynamic routes, and guards.</p>
            </div>
          </div>
          
          <div class="card bg-base-100 shadow-md">
            <div class="card-body items-center text-center">
              <div class="text-5xl mb-4">🏪</div>
              <h3 class="card-title">Pinia</h3>
              <p>Manage application state with the modern Vue store.</p>
            </div>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>
```

---

### Step 9: Create Blog List Page

```vue
<!-- src/pages/BlogPage.vue -->
<script setup lang="ts">
import { ref, computed } from 'vue'
import { posts, type BlogPost } from '../data/posts'

const searchQuery = ref('')
const selectedTag = ref<string | null>(null)

const allTags = computed(() => {
  const tags = new Set<string>()
  posts.forEach(post => post.tags.forEach(tag => tags.add(tag)))
  return Array.from(tags).sort()
})

const filteredPosts = computed(() => {
  return posts.filter(post => {
    const matchesSearch = post.title.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
                         post.excerpt.toLowerCase().includes(searchQuery.value.toLowerCase())
    const matchesTag = !selectedTag.value || post.tags.includes(selectedTag.value)
    return matchesSearch && matchesTag
  })
})

function clearFilters() {
  searchQuery.value = ''
  selectedTag.value = null
}
</script>

<template>
  <div class="py-12 px-4">
    <div class="container mx-auto">
      <!-- Header -->
      <div class="text-center mb-12">
        <h1 class="text-4xl font-bold mb-4">Blog</h1>
        <p class="text-lg text-base-content/70 max-w-2xl mx-auto">
          Explore our collection of articles on Vue.js, web development, and modern frontend practices.
        </p>
      </div>
      
      <!-- Filters -->
      <div class="flex flex-col md:flex-row gap-4 mb-8">
        <!-- Search -->
        <div class="form-control flex-1">
          <input 
            v-model="searchQuery"
            type="text" 
            placeholder="Search articles..." 
            class="input input-bordered w-full"
          />
        </div>
        
        <!-- Tag Filter -->
        <div class="flex gap-2 flex-wrap">
          <button 
            v-for="tag in allTags" 
            :key="tag"
            @click="selectedTag = selectedTag === tag ? null : tag"
            :class="[
              'btn btn-sm',
              selectedTag === tag ? 'btn-primary' : 'btn-outline'
            ]"
          >
            {{ tag }}
          </button>
          
          <button 
            v-if="searchQuery || selectedTag"
            @click="clearFilters"
            class="btn btn-sm btn-ghost"
          >
            Clear
          </button>
        </div>
      </div>
      
      <!-- Posts Grid -->
      <div v-if="filteredPosts.length > 0" class="grid md:grid-cols-2 gap-8">
        <article 
          v-for="post in filteredPosts" 
          :key="post.id"
          class="card card-side bg-base-200 shadow-xl hover:shadow-2xl transition-shadow"
        >
          <figure class="w-1/3 hidden sm:block">
            <img 
              :src="post.imageUrl" 
              :alt="post.title"
              class="h-full w-full object-cover"
            />
          </figure>
          <div class="card-body w-full sm:w-2/3">
            <div class="flex gap-2 flex-wrap">
              <span 
                v-for="tag in post.tags" 
                :key="tag"
                class="badge badge-primary badge-sm"
              >
                {{ tag }}
              </span>
            </div>
            <h2 class="card-title">{{ post.title }}</h2>
            <p class="text-base-content/70 text-sm">{{ post.excerpt }}</p>
            <div class="flex justify-between items-center mt-2">
              <div class="text-sm text-base-content/50">
                <span>{{ post.author }}</span>
                <span class="mx-2">•</span>
                <span>{{ post.date }}</span>
              </div>
              <RouterLink 
                :to="{ name: 'blog-post', params: { slug: post.slug } }"
                class="btn btn-primary btn-sm"
              >
                Read
              </RouterLink>
            </div>
          </div>
        </article>
      </div>
      
      <!-- Empty State -->
      <div v-else class="text-center py-16">
        <div class="text-6xl mb-4">🔍</div>
        <h3 class="text-xl font-bold mb-2">No articles found</h3>
        <p class="text-base-content/70 mb-4">
          Try adjusting your search or filter criteria.
        </p>
        <button @click="clearFilters" class="btn btn-primary">
          Clear Filters
        </button>
      </div>
    </div>
  </div>
</template>
```

---

### Step 10: Create Blog Post Page (Dynamic Route)

```vue
<!-- src/pages/BlogPostPage.vue -->
<script setup lang="ts">
import { computed } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { getPostBySlug, getRecentPosts } from '../data/posts'

const route = useRoute()
const router = useRouter()

// TODO: Get the slug parameter from the route
const slug = computed(() => route.params.slug as string)

const post = computed(() => getPostBySlug(slug.value))

const relatedPosts = computed(() => 
  getRecentPosts(3).filter(p => p.slug !== slug.value).slice(0, 2)
)

function goBack() {
  router.push({ name: 'blog' })
}
</script>

<template>
  <div class="py-12 px-4">
    <div class="container mx-auto max-w-4xl">
      <!-- Back Button -->
      <button @click="goBack" class="btn btn-ghost mb-8">
        ← Back to Blog
      </button>
      
      <!-- Post Content -->
      <article v-if="post">
        <!-- Featured Image -->
        <figure class="mb-8 rounded-xl overflow-hidden">
          <img 
            :src="post.imageUrl" 
            :alt="post.title"
            class="w-full h-64 md:h-96 object-cover"
          />
        </figure>
        
        <!-- Tags -->
        <div class="flex gap-2 flex-wrap mb-4">
          <span 
            v-for="tag in post.tags" 
            :key="tag"
            class="badge badge-primary"
          >
            {{ tag }}
          </span>
        </div>
        
        <!-- Title -->
        <h1 class="text-4xl md:text-5xl font-bold mb-4">
          {{ post.title }}
        </h1>
        
        <!-- Meta -->
        <div class="flex items-center gap-4 text-base-content/60 mb-8">
          <span>By {{ post.author }}</span>
          <span>•</span>
          <span>{{ post.date }}</span>
        </div>
        
        <!-- Content -->
        <div 
          class="prose prose-lg max-w-none"
          v-html="post.content"
        ></div>
        
        <!-- Share Section -->
        <div class="divider my-12"></div>
        
        <div class="flex flex-col md:flex-row justify-between items-center gap-4">
          <p class="text-lg font-medium">Enjoyed this article? Share it!</p>
          <div class="flex gap-2">
            <button class="btn btn-circle btn-outline">
              <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
                <path d="M24 4.557c-.883.392-1.832.656-2.828.775 1.017-.609 1.798-1.574 2.165-2.724-.951.564-2.005.974-3.127 1.195-.897-.957-2.178-1.555-3.594-1.555-3.179 0-5.515 2.966-4.797 6.045-4.091-.205-7.719-2.165-10.148-5.144-1.29 2.213-.669 5.108 1.523 6.574-.806-.026-1.566-.247-2.229-.616-.054 2.281 1.581 4.415 3.949 4.89-.693.188-1.452.232-2.224.084.626 1.956 2.444 3.379 4.6 3.419-2.07 1.623-4.678 2.348-7.29 2.04 2.179 1.397 4.768 2.212 7.548 2.212 9.142 0 14.307-7.721 13.995-14.646.962-.695 1.797-1.562 2.457-2.549z"/>
              </svg>
            </button>
            <button class="btn btn-circle btn-outline">
              <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
                <path d="M19 0h-14c-2.761 0-5 2.239-5 5v14c0 2.761 2.239 5 5 5h14c2.762 0 5-2.239 5-5v-14c0-2.761-2.238-5-5-5zm-11 19h-3v-11h3v11zm-1.5-12.268c-.966 0-1.75-.79-1.75-1.764s.784-1.764 1.75-1.764 1.75.79 1.75 1.764-.783 1.764-1.75 1.764zm13.5 12.268h-3v-5.604c0-3.368-4-3.113-4 0v5.604h-3v-11h3v1.765c1.396-2.586 7-2.777 7 2.476v6.759z"/>
              </svg>
            </button>
          </div>
        </div>
      </article>
      
      <!-- Post Not Found -->
      <div v-else class="text-center py-16">
        <div class="text-6xl mb-4">📄</div>
        <h2 class="text-2xl font-bold mb-4">Post Not Found</h2>
        <p class="text-base-content/70 mb-6">
          We couldn't find a post with the slug "{{ slug }}".
        </p>
        <RouterLink to="/blog" class="btn btn-primary">
          Browse All Posts
        </RouterLink>
      </div>
      
      <!-- Related Posts -->
      <div v-if="post && relatedPosts.length > 0" class="mt-16">
        <h2 class="text-2xl font-bold mb-8">Related Articles</h2>
        <div class="grid md:grid-cols-2 gap-6">
          <RouterLink 
            v-for="related in relatedPosts" 
            :key="related.id"
            :to="{ name: 'blog-post', params: { slug: related.slug } }"
            class="card card-side bg-base-200 shadow-md hover:shadow-lg transition-shadow"
          >
            <figure class="w-1/3">
              <img 
                :src="related.imageUrl" 
                :alt="related.title"
                class="h-full w-full object-cover"
              />
            </figure>
            <div class="card-body w-2/3">
              <h3 class="card-title text-base">{{ related.title }}</h3>
              <p class="text-sm text-base-content/60">{{ related.date }}</p>
            </div>
          </RouterLink>
        </div>
      </div>
    </div>
  </div>
</template>
```

---

### Step 11: Create About Page

```vue
<!-- src/pages/AboutPage.vue -->
<script setup lang="ts">
const team = [
  {
    name: 'Jane Developer',
    role: 'Lead Author',
    bio: 'Full-stack developer with 8+ years of experience in Vue.js ecosystem.',
    avatar: 'https://i.pravatar.cc/150?img=1'
  },
  {
    name: 'John Coder',
    role: 'Technical Writer',
    bio: 'Passionate about making complex topics accessible to beginners.',
    avatar: 'https://i.pravatar.cc/150?img=3'
  },
  {
    name: 'Alex Designer',
    role: 'UI/UX Expert',
    bio: 'Creating beautiful interfaces and sharing design knowledge.',
    avatar: 'https://i.pravatar.cc/150?img=5'
  }
]
</script>

<template>
  <div>
    <!-- Hero Section -->
    <section class="py-20 px-4 bg-gradient-to-br from-secondary to-accent text-secondary-content">
      <div class="container mx-auto text-center">
        <h1 class="text-5xl font-bold mb-6">About Vue Blog</h1>
        <p class="text-xl max-w-2xl mx-auto opacity-90">
          We're a team of passionate developers dedicated to sharing knowledge 
          and helping others learn modern web development.
        </p>
      </div>
    </section>
    
    <!-- Mission Section -->
    <section class="py-16 px-4">
      <div class="container mx-auto max-w-4xl">
        <div class="grid md:grid-cols-2 gap-12 items-center">
          <div>
            <h2 class="text-3xl font-bold mb-6">Our Mission</h2>
            <p class="text-lg text-base-content/70 mb-4">
              We believe that quality education should be accessible to everyone. 
              Our mission is to create comprehensive, practical tutorials that help 
              developers of all skill levels master Vue.js and modern frontend technologies.
            </p>
            <p class="text-lg text-base-content/70">
              Every article is written with beginners in mind, but packed with 
              insights that even experienced developers will find valuable.
            </p>
          </div>
          <div class="flex justify-center">
            <div class="stats stats-vertical shadow">
              <div class="stat">
                <div class="stat-title">Articles Published</div>
                <div class="stat-value text-primary">50+</div>
              </div>
              <div class="stat">
                <div class="stat-title">Monthly Readers</div>
                <div class="stat-value text-secondary">10K+</div>
              </div>
              <div class="stat">
                <div class="stat-title">Happy Learners</div>
                <div class="stat-value text-accent">5K+</div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>
    
    <!-- Team Section -->
    <section class="py-16 px-4 bg-base-200">
      <div class="container mx-auto">
        <h2 class="text-3xl font-bold text-center mb-12">Meet Our Team</h2>
        
        <div class="grid md:grid-cols-3 gap-8 max-w-4xl mx-auto">
          <div 
            v-for="member in team" 
            :key="member.name"
            class="card bg-base-100 shadow-xl"
          >
            <figure class="px-10 pt-10">
              <img 
                :src="member.avatar" 
                :alt="member.name"
                class="rounded-full w-32 h-32 object-cover"
              />
            </figure>
            <div class="card-body items-center text-center">
              <h3 class="card-title">{{ member.name }}</h3>
              <p class="text-primary font-medium">{{ member.role }}</p>
              <p class="text-base-content/70">{{ member.bio }}</p>
            </div>
          </div>
        </div>
      </div>
    </section>
    
    <!-- CTA Section -->
    <section class="py-16 px-4">
      <div class="container mx-auto text-center">
        <h2 class="text-3xl font-bold mb-6">Ready to Start Learning?</h2>
        <p class="text-lg text-base-content/70 mb-8 max-w-xl mx-auto">
          Explore our collection of tutorials and take your Vue.js skills to the next level.
        </p>
        <RouterLink to="/blog" class="btn btn-primary btn-lg">
          Browse Articles
        </RouterLink>
      </div>
    </section>
  </div>
</template>
```

---

### Step 12: Create 404 Page

```vue
<!-- src/pages/NotFoundPage.vue -->
<script setup lang="ts">
import { useRoute } from 'vue-router'

const route = useRoute()
</script>

<template>
  <div class="min-h-[70vh] flex items-center justify-center px-4">
    <div class="text-center">
      <h1 class="text-9xl font-bold text-primary">404</h1>
      <h2 class="text-3xl font-bold mt-4 mb-2">Page Not Found</h2>
      <p class="text-base-content/70 mb-8 max-w-md">
        Sorry, we couldn't find the page 
        <code class="bg-base-200 px-2 py-1 rounded text-sm">{{ route.fullPath }}</code>
      </p>
      <div class="flex gap-4 justify-center">
        <RouterLink to="/" class="btn btn-primary">
          Go Home
        </RouterLink>
        <RouterLink to="/blog" class="btn btn-outline">
          Browse Blog
        </RouterLink>
      </div>
    </div>
  </div>
</template>
```

---

## Complete Solution

All files above form the complete solution. Run the project with:

```bash
npm run dev
```

---

## How to Test

1. **Navigation:** Click all nav links and verify URL changes without page reload
2. **Active state:** Current page link should be highlighted in the navbar
3. **Blog listing:** Search and filter posts, verify they update instantly
4. **Dynamic routes:** Click "Read More" on posts, verify URL shows the slug
5. **Related posts:** On a blog post, click related articles, verify navigation works
6. **404 handling:** Visit `/nonexistent`, verify 404 page shows with attempted path
7. **Mobile menu:** Resize browser, test mobile hamburger menu functionality
8. **Browser history:** Use back/forward buttons, verify navigation works correctly

---

## Stretch Goals

1. **Pagination:** Add pagination to the blog list page
2. **Reading time:** Calculate and display estimated reading time for posts
3. **Search highlighting:** Highlight search terms in results
4. **Transitions:** Add page transition animations using Vue's `<Transition>` component
5. **Bookmarks:** Allow users to bookmark posts (save to localStorage)
6. **Dark mode:** Add a theme toggle using DaisyUI's theme system

---

## Self-Assessment Checklist

- [ ] All components use `<script setup>` with TypeScript
- [ ] Router is configured with `createWebHistory()` for clean URLs
- [ ] All internal links use `<RouterLink>` instead of `<a href>`
- [ ] Dynamic route parameter is accessed via `useRoute().params`
- [ ] 404 catch-all route is placed last in the routes array
- [ ] Active navigation links are visually distinguished
- [ ] Pages scroll to top on navigation
- [ ] Document title updates based on current route
