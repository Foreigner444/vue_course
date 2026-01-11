# Module 3 Project 3: Dark Mode Toggle

**Scenario:** A portfolio site switches between light/dark themes with smooth transition.

**Key Patterns:** CSS variables, Tailwind dark mode, DaisyUI themes

---

## useTheme.ts

```typescript
import { ref, watch, onMounted } from 'vue'

type Theme = 'light' | 'dark' | 'cupcake' | 'forest' | 'cyberpunk'

const STORAGE_KEY = 'app-theme'
const DEFAULT_THEME: Theme = 'light'

const currentTheme = ref<Theme>(DEFAULT_THEME)

export function useTheme() {
  const setTheme = (theme: Theme) => {
    currentTheme.value = theme
    document.documentElement.setAttribute('data-theme', theme)
    localStorage.setItem(STORAGE_KEY, theme)
  }

  const toggleDarkMode = () => {
    const newTheme = currentTheme.value === 'light' ? 'dark' : 'light'
    setTheme(newTheme)
  }

  const initTheme = () => {
    const stored = localStorage.getItem(STORAGE_KEY) as Theme | null
    const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches
    
    if (stored) {
      setTheme(stored)
    } else if (prefersDark) {
      setTheme('dark')
    } else {
      setTheme(DEFAULT_THEME)
    }
  }

  onMounted(() => {
    initTheme()
  })

  return {
    currentTheme,
    setTheme,
    toggleDarkMode
  }
}
```

---

## ThemeToggle.vue

```vue
<script setup lang="ts">
import { useTheme } from '../composables/useTheme'

const { currentTheme, toggleDarkMode } = useTheme()
</script>

<template>
  <label class="swap swap-rotate btn btn-ghost btn-circle">
    <input 
      type="checkbox" 
      :checked="currentTheme === 'dark'"
      @change="toggleDarkMode"
    />
    
    <svg 
      class="swap-on fill-current w-6 h-6" 
      xmlns="http://www.w3.org/2000/svg" 
      viewBox="0 0 24 24"
    >
      <path d="M5.64,17l-.71.71a1,1,0,0,0,0,1.41,1,1,0,0,0,1.41,0l.71-.71A1,1,0,0,0,5.64,17ZM5,12a1,1,0,0,0-1-1H3a1,1,0,0,0,0,2H4A1,1,0,0,0,5,12Zm7-7a1,1,0,0,0,1-1V3a1,1,0,0,0-2,0V4A1,1,0,0,0,12,5ZM5.64,7.05a1,1,0,0,0,.7.29,1,1,0,0,0,.71-.29,1,1,0,0,0,0-1.41l-.71-.71A1,1,0,0,0,4.93,6.34Zm12,.29a1,1,0,0,0,.7-.29l.71-.71a1,1,0,1,0-1.41-1.41L17,5.64a1,1,0,0,0,0,1.41A1,1,0,0,0,17.66,7.34ZM21,11H20a1,1,0,0,0,0,2h1a1,1,0,0,0,0-2Zm-9,8a1,1,0,0,0-1,1v1a1,1,0,0,0,2,0V20A1,1,0,0,0,12,19ZM18.36,17A1,1,0,0,0,17,18.36l.71.71a1,1,0,0,0,1.41,0,1,1,0,0,0,0-1.41ZM12,6.5A5.5,5.5,0,1,0,17.5,12,5.51,5.51,0,0,0,12,6.5Zm0,9A3.5,3.5,0,1,1,15.5,12,3.5,3.5,0,0,1,12,15.5Z"/>
    </svg>
    
    <svg 
      class="swap-off fill-current w-6 h-6" 
      xmlns="http://www.w3.org/2000/svg" 
      viewBox="0 0 24 24"
    >
      <path d="M21.64,13a1,1,0,0,0-1.05-.14,8.05,8.05,0,0,1-3.37.73A8.15,8.15,0,0,1,9.08,5.49a8.59,8.59,0,0,1,.25-2A1,1,0,0,0,8,2.36,10.14,10.14,0,1,0,22,14.05,1,1,0,0,0,21.64,13Zm-9.5,6.69A8.14,8.14,0,0,1,7.08,5.22v.27A10.15,10.15,0,0,0,17.22,15.63a9.79,9.79,0,0,0,2.1-.22A8.11,8.11,0,0,1,12.14,19.73Z"/>
    </svg>
  </label>
</template>
```

---

## ThemeSelector.vue

```vue
<script setup lang="ts">
import { useTheme } from '../composables/useTheme'

const { currentTheme, setTheme } = useTheme()

const themes = [
  { name: 'light', label: 'Light', emoji: '☀️' },
  { name: 'dark', label: 'Dark', emoji: '🌙' },
  { name: 'cupcake', label: 'Cupcake', emoji: '🧁' },
  { name: 'forest', label: 'Forest', emoji: '🌲' },
  { name: 'cyberpunk', label: 'Cyberpunk', emoji: '🤖' }
] as const
</script>

<template>
  <div class="dropdown dropdown-end">
    <div tabindex="0" role="button" class="btn btn-ghost gap-2">
      <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 21a4 4 0 01-4-4V5a2 2 0 012-2h4a2 2 0 012 2v12a4 4 0 01-4 4zm0 0h12a2 2 0 002-2v-4a2 2 0 00-2-2h-2.343M11 7.343l1.657-1.657a2 2 0 012.828 0l2.829 2.829a2 2 0 010 2.828l-8.486 8.485M7 17h.01" />
      </svg>
      Theme
      <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" />
      </svg>
    </div>
    
    <ul tabindex="0" class="dropdown-content z-50 menu p-2 shadow-lg bg-base-100 rounded-box w-52 mt-2">
      <li v-for="theme in themes" :key="theme.name">
        <button 
          class="flex items-center justify-between"
          :class="{ 'active': currentTheme === theme.name }"
          @click="setTheme(theme.name)"
        >
          <span>{{ theme.emoji }} {{ theme.label }}</span>
          <svg 
            v-if="currentTheme === theme.name"
            class="w-4 h-4 text-success" 
            fill="none" 
            stroke="currentColor" 
            viewBox="0 0 24 24"
          >
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
          </svg>
        </button>
      </li>
    </ul>
  </div>
</template>
```

---

## PortfolioPage.vue

```vue
<script setup lang="ts">
import ThemeToggle from './ThemeToggle.vue'
import ThemeSelector from './ThemeSelector.vue'
import { useTheme } from '../composables/useTheme'

useTheme()

const projects = [
  { title: 'E-commerce Platform', category: 'Web App', image: 'https://picsum.photos/seed/1/400/300' },
  { title: 'Mobile Banking', category: 'Mobile', image: 'https://picsum.photos/seed/2/400/300' },
  { title: 'Analytics Dashboard', category: 'Web App', image: 'https://picsum.photos/seed/3/400/300' }
]
</script>

<template>
  <div class="min-h-screen bg-base-200 transition-colors duration-300">
    <nav class="navbar bg-base-100 shadow-md sticky top-0 z-50">
      <div class="navbar-start">
        <a class="btn btn-ghost text-xl font-bold">Portfolio</a>
      </div>
      
      <div class="navbar-center hidden md:flex">
        <ul class="menu menu-horizontal px-1 gap-2">
          <li><a class="hover:text-primary">Work</a></li>
          <li><a class="hover:text-primary">About</a></li>
          <li><a class="hover:text-primary">Contact</a></li>
        </ul>
      </div>
      
      <div class="navbar-end gap-2">
        <ThemeToggle />
        <ThemeSelector />
      </div>
    </nav>

    <main>
      <section class="hero min-h-[60vh] bg-base-100">
        <div class="hero-content text-center">
          <div class="max-w-2xl">
            <h1 class="text-5xl md:text-6xl font-bold text-base-content">
              Creative Developer
            </h1>
            <p class="py-6 text-lg text-base-content/70">
              I build beautiful, functional, and accessible web experiences.
              Currently focused on Vue.js and modern frontend technologies.
            </p>
            <div class="flex gap-4 justify-center">
              <button class="btn btn-primary">View Work</button>
              <button class="btn btn-outline">Contact Me</button>
            </div>
          </div>
        </div>
      </section>

      <section class="py-16 px-4 md:px-8">
        <div class="max-w-6xl mx-auto">
          <h2 class="text-3xl font-bold text-base-content mb-8 text-center">
            Featured Projects
          </h2>
          
          <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            <div 
              v-for="project in projects" 
              :key="project.title"
              class="card bg-base-100 shadow-xl hover:shadow-2xl transition-all duration-300 hover:-translate-y-2 overflow-hidden group"
            >
              <figure class="h-48 overflow-hidden">
                <img 
                  :src="project.image" 
                  :alt="project.title"
                  class="w-full h-full object-cover transition-transform duration-500 group-hover:scale-110"
                />
              </figure>
              <div class="card-body">
                <div class="badge badge-outline badge-sm">{{ project.category }}</div>
                <h3 class="card-title mt-2">{{ project.title }}</h3>
                <div class="card-actions justify-end mt-4">
                  <button class="btn btn-primary btn-sm">View Project</button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>

      <section class="py-16 bg-base-100">
        <div class="max-w-4xl mx-auto px-4 text-center">
          <h2 class="text-3xl font-bold text-base-content mb-4">Let's Work Together</h2>
          <p class="text-base-content/70 mb-8">
            Have a project in mind? I'd love to hear about it.
          </p>
          <button class="btn btn-primary btn-lg">Get In Touch</button>
        </div>
      </section>
    </main>

    <footer class="footer footer-center p-10 bg-base-100 text-base-content border-t border-base-300">
      <div>
        <p>© 2024 Portfolio. Built with Vue.js, Tailwind CSS, and DaisyUI.</p>
      </div>
    </footer>
  </div>
</template>
```

---

## tailwind.config.js (Theme Configuration)

```javascript
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
    themes: ["light", "dark", "cupcake", "forest", "cyberpunk"],
    darkTheme: "dark",
  },
}
```

---

## Skills Practiced

- DaisyUI theming system with `data-theme` attribute
- `swap` component for animated icon transitions
- `dropdown` component for theme selector menu
- Composables for shared state (`useTheme`)
- localStorage persistence for user preference
- System preference detection with `matchMedia`
- Smooth transitions with `transition-colors duration-300`
- DaisyUI semantic colors (`bg-base-100`, `text-base-content`, `btn-primary`)
- Responsive navbar with hidden mobile elements
- Group hover effects (`group-hover:scale-110`)
