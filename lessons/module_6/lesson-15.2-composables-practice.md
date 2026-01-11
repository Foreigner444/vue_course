# Lesson 15.2: Reusable Logic with Composables — Practice & Application (Переиспользуемая логика с Composables)
---
## 1. Lesson Metadata
| Field | Value |
|:--- |:--- |
| **Lesson Number** | 15.2 of 25 (Part 2 of 2) |
| **Topic** | Reusable Logic with Composables / Переиспользуемая логика с Composables |
| **Continues From** | Lesson 15.1 |
| **Duration** | 30-35 minutes |
**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Create composables that encapsulate reactive state, computed values, and actions
2. **Analyze:** Identify when logic should be extracted into a composable vs. kept inline
---
## 2. Quick Recap
From Lesson 15.1, remember:
- **Composables** are functions that use Vue's Composition API and return reactive state/logic
- Each call to a composable creates **independent state** (like a recipe producing fresh dishes)
- Name composables with the `use` prefix (`useCounter`, `useFetch`, `useAuth`)
- Return an object containing refs, computed values, and functions that components need
---
## 3. The Variable Frame & Complexity Scale
### Basic Form: Simple State + Action
```typescript
// src/composables/useToggle.ts
import { ref } from 'vue'
export function useToggle(initial: boolean = false) {
  const state = ref(initial)
  
  function toggle() {
    state.value = !state.value
  }
  
  return { state, toggle }
}
```
### With Options: Configurable Behavior
```typescript
// src/composables/useToggle.ts
import { ref } from 'vue'
interface UseToggleOptions {
  initial?: boolean
  onToggle?: (newValue: boolean) => void
}
export function useToggle(options: UseToggleOptions = {}) {
  const { initial = false, onToggle } = options
  
  const state = ref(initial)
  
  function toggle() {
    state.value = !state.value
    onToggle?.(state.value)
  }
  
  function setTrue() {
    state.value = true
    onToggle?.(true)
  }
  
  function setFalse() {
    state.value = false
    onToggle?.(false)
  }
  
  return { state, toggle, setTrue, setFalse }
}
```
### Advanced Form: With Computed, Watch, and Cleanup
```typescript
// src/composables/useMousePosition.ts
import { ref, computed, onMounted, onUnmounted } from 'vue'
export function useMousePosition() {
  const x = ref(0)
  const y = ref(0)
  
  const position = computed(() => ({ x: x.value, y: y.value }))
  const isInUpperHalf = computed(() => y.value < window.innerHeight / 2)
  
  function updatePosition(event: MouseEvent) {
    x.value = event.clientX
    y.value = event.clientY
  }
  
  onMounted(() => {
    window.addEventListener('mousemove', updatePosition)
  })
  
  onUnmounted(() => {
    window.removeEventListener('mousemove', updatePosition)
  })
  
  return { x, y, position, isInUpperHalf }
}
```
---
## 4. Micro-Implementation Drills
### Drill 1 (Basic): Create a useCounter Composable
Complete the composable that tracks a count with increment/decrement:
```typescript
// src/composables/useCounter.ts
import { ref } from 'vue'
export function useCounter(initial: number = 0) {
  // TODO: Create a ref for the count
  const count = _____
  
  // TODO: Create increment function
  function increment() {
    _____
  }
  
  // TODO: Create decrement function
  function decrement() {
    _____
  }
  
  // TODO: Return the public API
  return { _____ }
}
```
<details>
<summary>Solution</summary>
```typescript
import { ref } from 'vue'
export function useCounter(initial: number = 0) {
  const count = ref(initial)
  
  function increment() {
    count.value++
  }
  
  function decrement() {
    count.value--
  }
  
  return { count, increment, decrement }
}
```
</details>
---
### Drill 2 (Variation): Add Computed Values
Extend the counter with derived state:
```typescript
// src/composables/useCounter.ts
import { ref, computed } from 'vue'
export function useCounter(initial: number = 0, step: number = 1) {
  const count = ref(initial)
  
  // TODO: Create computed that returns true if count is even
  const isEven = computed(() => _____)
  
  // TODO: Create computed for absolute value
  const absolute = computed(() => _____)
  
  function increment() {
    count.value += step
  }
  
  function decrement() {
    count.value -= step
  }
  
  return { count, isEven, absolute, increment, decrement }
}
```
<details>
<summary>Solution</summary>
```typescript
import { ref, computed } from 'vue'
export function useCounter(initial: number = 0, step: number = 1) {
  const count = ref(initial)
  
  const isEven = computed(() => count.value % 2 === 0)
  const absolute = computed(() => Math.abs(count.value))
  
  function increment() {
    count.value += step
  }
  
  function decrement() {
    count.value -= step
  }
  
  return { count, isEven, absolute, increment, decrement }
}
```
</details>
---
### Drill 3 (Combination): Composable with Watch
Create a composable that syncs with localStorage using `watch`:
```typescript
// src/composables/useStoredValue.ts
import { ref, watch } from 'vue'
export function useStoredValue<T>(key: string, defaultValue: T) {
  // TODO: Initialize ref - try to read from localStorage first
  const stored = localStorage.getItem(key)
  const value = ref<T>(stored ? JSON.parse(stored) : defaultValue)
  
  // TODO: Add a watcher that saves to localStorage whenever value changes
  watch(_____, (newValue) => {
    _____
  })
  
  return value
}
```
<details>
<summary>Solution</summary>
```typescript
import { ref, watch } from 'vue'
export function useStoredValue<T>(key: string, defaultValue: T) {
  const stored = localStorage.getItem(key)
  const value = ref<T>(stored ? JSON.parse(stored) : defaultValue) as Ref<T>
  
  watch(value, (newValue) => {
    localStorage.setItem(key, JSON.stringify(newValue))
  }, { deep: true })
  
  return value
}
```
</details>
---
### Drill 4 (Edge Case): Cleanup on Unmount
Create a composable for a timer that cleans up properly:
```typescript
// src/composables/useInterval.ts
import { ref, onUnmounted } from 'vue'
export function useInterval(callback: () => void, delay: number) {
  const isRunning = ref(false)
  let intervalId: number | null = null
  
  function start() {
    if (isRunning.value) return
    // TODO: Start the interval and update isRunning
    _____
    _____
  }
  
  function stop() {
    // TODO: Clear the interval and update isRunning
    if (intervalId !== null) {
      _____
      _____
      _____
    }
  }
  
  // TODO: Clean up when component unmounts
  onUnmounted(() => {
    _____
  })
  
  return { isRunning, start, stop }
}
```
<details>
<summary>Solution</summary>
```typescript
import { ref, onUnmounted } from 'vue'
export function useInterval(callback: () => void, delay: number) {
  const isRunning = ref(false)
  let intervalId: number | null = null
  
  function start() {
    if (isRunning.value) return
    intervalId = window.setInterval(callback, delay)
    isRunning.value = true
  }
  
  function stop() {
    if (intervalId !== null) {
      clearInterval(intervalId)
      intervalId = null
      isRunning.value = false
    }
  }
  
  onUnmounted(() => {
    stop()
  })
  
  return { isRunning, start, stop }
}
```
</details>
---
## 5. Common Pitfalls & Anti-Patterns
| ❌ Common Mistake | ✅ Correct Approach | Why It Matters |
|:--- |:--- |:--- |
| Destructuring refs loses reactivity: `let { count } = useCounter(); count = 5` | Use `.value` on the ref: `const { count } = useCounter(); count.value = 5` | Reassigning a destructured variable doesn't update the original ref—you're just reassigning a local variable |
| Calling composables outside `<script setup>`: `onClick() { const x = useCounter() }` | Call composables at the top level of `<script setup>` only | Composables rely on Vue's setup context; calling them in event handlers or callbacks breaks lifecycle hooks |
| Creating global state accidentally: `const count = ref(0); export function useCounter() { return { count } }` | Create state inside the function: `export function useCounter() { const count = ref(0); return { count } }` | If state is declared outside the function, all components share the same instance—usually not intended |
---
## 6. Mini-Project: Dark Mode Toggle with Persistence
Build a complete dark mode composable that persists user preference.
### File Structure
```
src/
├── composables/
│   └── useDarkMode.ts
├── components/
│   └── ThemeToggle.vue
├── App.vue
└── main.ts
```
### useDarkMode.ts
```typescript
// src/composables/useDarkMode.ts
import { ref, watch, onMounted } from 'vue'
export function useDarkMode() {
  // ─────────────────────────────────────────────────────────────
  // STATE: Track whether dark mode is active
  // We start with false and will check localStorage/system preference on mount
  // ─────────────────────────────────────────────────────────────
  const isDark = ref(false)
  // ─────────────────────────────────────────────────────────────
  // CONSTANTS: Key for localStorage
  // ─────────────────────────────────────────────────────────────
  const STORAGE_KEY = 'theme-dark-mode'
  // ─────────────────────────────────────────────────────────────
  // HELPER: Apply the theme to the document
  // This adds/removes the 'dark' class on the <html> element
  // Tailwind and DaisyUI use this class to switch themes
  // ─────────────────────────────────────────────────────────────
  function applyTheme(dark: boolean) {
    if (dark) {
      document.documentElement.classList.add('dark')
      document.documentElement.setAttribute('data-theme', 'dark')
    } else {
      document.documentElement.classList.remove('dark')
      document.documentElement.setAttribute('data-theme', 'light')
    }
  }
  // ─────────────────────────────────────────────────────────────
  // ACTION: Toggle between dark and light mode
  // ─────────────────────────────────────────────────────────────
  function toggle() {
    isDark.value = !isDark.value
  }
  // ─────────────────────────────────────────────────────────────
  // ACTION: Explicitly enable dark mode
  // ─────────────────────────────────────────────────────────────
  function enableDark() {
    isDark.value = true
  }
  // ─────────────────────────────────────────────────────────────
  // ACTION: Explicitly enable light mode
  // ─────────────────────────────────────────────────────────────
  function enableLight() {
    isDark.value = false
  }
  // ─────────────────────────────────────────────────────────────
  // WATCHER: Persist preference and apply theme when isDark changes
  // ─────────────────────────────────────────────────────────────
  watch(isDark, (newValue) => {
    // Save to localStorage so preference persists across sessions
    localStorage.setItem(STORAGE_KEY, JSON.stringify(newValue))
    // Apply the theme to the DOM
    applyTheme(newValue)
  })
  // ─────────────────────────────────────────────────────────────
  // LIFECYCLE: On mount, initialize from stored preference or system
  // ─────────────────────────────────────────────────────────────
  onMounted(() => {
    // First, check if user has a saved preference
    const stored = localStorage.getItem(STORAGE_KEY)
    
    if (stored !== null) {
      // User has a saved preference - use it
      isDark.value = JSON.parse(stored)
    } else {
      // No saved preference - check system preference
      // window.matchMedia checks the user's OS-level dark mode setting
      const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches
      isDark.value = prefersDark
    }
    
    // Apply the determined theme
    applyTheme(isDark.value)
  })
  // ─────────────────────────────────────────────────────────────
  // PUBLIC API: Return what components need
  // ─────────────────────────────────────────────────────────────
  return {
    isDark,
    toggle,
    enableDark,
    enableLight
  }
}
```
### ThemeToggle.vue
```vue
<!-- src/components/ThemeToggle.vue -->
<script setup lang="ts">
// ─────────────────────────────────────────────────────────────
// IMPORTS: Bring in the dark mode composable
// ─────────────────────────────────────────────────────────────
import { useDarkMode } from '@/composables/useDarkMode'
// ─────────────────────────────────────────────────────────────
// USE COMPOSABLE: Destructure the pieces we need
// - isDark: reactive boolean for current state
// - toggle: function to switch between dark/light
// ─────────────────────────────────────────────────────────────
const { isDark, toggle } = useDarkMode()
</script>
<template>
  <!-- 
    TOGGLE BUTTON
    - Uses DaisyUI's swap component for smooth icon transition
    - Clicking anywhere on the button calls toggle()
    - The checkbox is controlled by isDark state
  -->
  <label class="swap swap-rotate btn btn-ghost btn-circle">
    <!-- Hidden checkbox controls the swap state -->
    <input 
      type="checkbox" 
      :checked="isDark"
      @change="toggle"
    />
    
    <!-- Sun icon - shown when in dark mode (click to go light) -->
    <svg 
      class="swap-on fill-current w-6 h-6" 
      xmlns="http://www.w3.org/2000/svg" 
      viewBox="0 0 24 24"
    >
      <path d="M5.64,17l-.71.71a1,1,0,0,0,0,1.41,1,1,0,0,0,1.41,0l.71-.71A1,1,0,0,0,5.64,17ZM5,12a1,1,0,0,0-1-1H3a1,1,0,0,0,0,2H4A1,1,0,0,0,5,12Zm7-7a1,1,0,0,0,1-1V3a1,1,0,0,0-2,0V4A1,1,0,0,0,12,5ZM5.64,7.05a1,1,0,0,0,.7.29,1,1,0,0,0,.71-.29,1,1,0,0,0,0-1.41l-.71-.71A1,1,0,0,0,4.93,6.34Zm12,.29a1,1,0,0,0,.7-.29l.71-.71a1,1,0,1,0-1.41-1.41L17,5.64a1,1,0,0,0,0,1.41A1,1,0,0,0,17.66,7.34ZM21,11H20a1,1,0,0,0,0,2h1a1,1,0,0,0,0-2Zm-9,8a1,1,0,0,0-1,1v1a1,1,0,0,0,2,0V20A1,1,0,0,0,12,19ZM18.36,17A1,1,0,0,0,17,18.36l.71.71a1,1,0,0,0,1.41,0,1,1,0,0,0,0-1.41ZM12,6.5A5.5,5.5,0,1,0,17.5,12,5.51,5.51,0,0,0,12,6.5Zm0,9A3.5,3.5,0,1,1,15.5,12,3.5,3.5,0,0,1,12,15.5Z"/>
    </svg>
    
    <!-- Moon icon - shown when in light mode (click to go dark) -->
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
### App.vue
```vue
<!-- src/App.vue -->
<script setup lang="ts">
import ThemeToggle from '@/components/ThemeToggle.vue'
</script>
<template>
  <div class="min-h-screen bg-base-100 transition-colors duration-300">
    <!-- Navbar with theme toggle -->
    <nav class="navbar bg-base-200">
      <div class="flex-1">
        <span class="text-xl font-bold">My App</span>
      </div>
      <div class="flex-none">
        <ThemeToggle />
      </div>
    </nav>
    
    <!-- Main content -->
    <main class="container mx-auto p-8">
      <div class="card bg-base-200 shadow-xl">
        <div class="card-body">
          <h2 class="card-title">Welcome!</h2>
          <p>Toggle the theme using the button in the navbar.</p>
          <p>Your preference is automatically saved.</p>
        </div>
      </div>
    </main>
  </div>
</template>
```
### How to Test
1. **Run the dev server:** `npm run dev`
2. **Click the theme toggle** in the navbar - the page should switch between light and dark
3. **Refresh the page** - your preference should persist
4. **Clear localStorage** and refresh - the app should detect your system preference
---
## 7. Implementation Exercises (Progressive)
### Level 1 (Recall): Fill in the Blanks
Complete this composable for tracking window size:
```typescript
// src/composables/useWindowSize.ts
import { _____, onMounted, onUnmounted } from 'vue'
export function useWindowSize() {
  const width = _____(window.innerWidth)
  const height = _____(window.innerHeight)
  
  function update() {
    width._____ = window.innerWidth
    height._____ = window.innerHeight
  }
  
  onMounted(() => {
    window._____('resize', update)
  })
  
  onUnmounted(() => {
    window._____('resize', update)
  })
  
  return { width, height }
}
```
<details>
<summary>Solution</summary>
```typescript
import { ref, onMounted, onUnmounted } from 'vue'
export function useWindowSize() {
  const width = ref(window.innerWidth)
  const height = ref(window.innerHeight)
  
  function update() {
    width.value = window.innerWidth
    height.value = window.innerHeight
  }
  
  onMounted(() => {
    window.addEventListener('resize', update)
  })
  
  onUnmounted(() => {
    window.removeEventListener('resize', update)
  })
  
  return { width, height }
}
```
</details>
---
### Level 2 (Apply): Build Your Own Composables
**Exercise 2.1:** Create a `useOnline` composable that tracks whether the user is online or offline.
*Hints:*
- Use `navigator.onLine` for initial state
- Listen to `'online'` and `'offline'` window events
- Clean up listeners on unmount
**Exercise 2.2:** Create a `useClipboard` composable with `copy(text)` function and `copied` state.
*Hints:*
- Use `navigator.clipboard.writeText()`
- Set `copied` to true after copying
- Reset `copied` to false after 2 seconds (use `setTimeout`)
<details>
<summary>Solution 2.1</summary>
```typescript
// src/composables/useOnline.ts
import { ref, onMounted, onUnmounted } from 'vue'
export function useOnline() {
  const isOnline = ref(navigator.onLine)
  
  function updateOnlineStatus() {
    isOnline.value = navigator.onLine
  }
  
  onMounted(() => {
    window.addEventListener('online', updateOnlineStatus)
    window.addEventListener('offline', updateOnlineStatus)
  })
  
  onUnmounted(() => {
    window.removeEventListener('online', updateOnlineStatus)
    window.removeEventListener('offline', updateOnlineStatus)
  })
  
  return { isOnline }
}
```
</details>
<details>
<summary>Solution 2.2</summary>
```typescript
// src/composables/useClipboard.ts
import { ref } from 'vue'
export function useClipboard() {
  const copied = ref(false)
  let timeoutId: number | null = null
  
  async function copy(text: string) {
    try {
      await navigator.clipboard.writeText(text)
      copied.value = true
      
      if (timeoutId) clearTimeout(timeoutId)
      timeoutId = window.setTimeout(() => {
        copied.value = false
      }, 2000)
    } catch (error) {
      console.error('Failed to copy:', error)
    }
  }
  
  return { copied, copy }
}
```
</details>
---
### Level 3 (Debug): Fix the Broken Composable
This composable has 3 bugs. Find and fix them:
```typescript
// src/composables/useDebounce.ts - BROKEN
import { ref, watch } from 'vue'
export function useDebounce(value: Ref<string>, delay: number = 300) {
  const debouncedValue = ref(value)  // Bug 1
  let timeoutId: number
  
  watch(value, (newValue) => {
    timeoutId = setTimeout(() => {  // Bug 2
      debouncedValue.value = newValue
    }, delay)
  })
  
  return debouncedValue  // Bug 3
}
```
<details>
<summary>Solution</summary>
```typescript
// src/composables/useDebounce.ts - FIXED
import { ref, watch, type Ref } from 'vue'
export function useDebounce(value: Ref<string>, delay: number = 300) {
  // Bug 1 Fix: Initialize with value.value, not the ref itself
  const debouncedValue = ref(value.value)
  let timeoutId: number | null = null
  
  watch(value, (newValue) => {
    // Bug 2 Fix: Clear previous timeout to actually debounce
    if (timeoutId !== null) {
      clearTimeout(timeoutId)
    }
    timeoutId = window.setTimeout(() => {
      debouncedValue.value = newValue
    }, delay)
  })
  
  // Bug 3 Fix: Return an object for consistency (optional but recommended)
  // Original was returning just the ref, which works but breaks convention
  return { debouncedValue }
}
```
**Bugs explained:**
1. `ref(value)` wraps the Ref object itself, not its value. Use `ref(value.value)` to get the actual string.
2. Without clearing the previous timeout, every keystroke queues a new update instead of debouncing.
3. Returning just the ref works but breaks the composable convention. Returning an object `{ debouncedValue }` is more flexible.
</details>
---
**Reply 'next' for Lesson 16.1 (Theory) — Fetching Data: Async/Await & Lifecycle.**
