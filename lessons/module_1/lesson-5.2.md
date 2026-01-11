# Lesson 5.2: Running Side Effects with `watch()` — Practice & Application (Отслеживание изменений с watch())

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 5.2 of 25 (Part 2 of 2) |
| **Topic** | Running Side Effects with `watch()` / Отслеживание изменений с watch() |
| **Continues From** | Lesson 5.1 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Implement watchers for API calls, localStorage sync, and input validation
2. **Analyze:** Debug watcher issues and optimize watcher performance with debouncing

---

## 2. Quick Recap

- **`watch(source, callback)`** — runs callback when source changes
- **`watchEffect(callback)`** — auto-tracks dependencies, runs immediately
- Use **getter function** `() => state.prop` to watch reactive object properties
- **`{ immediate: true }`** — run callback immediately on setup
- **`{ deep: true }`** — watch nested object/array changes
- Watchers are for **side effects** (API calls, storage, logging), not derived values

---

## 3. The Variable Frame & Complexity Scale

**Basic Form:**
```vue
<script setup lang="ts">
import { ref, watch } from 'vue'

const count = ref(0)

watch(count, (newVal) => {
  console.log(`Count is now: ${newVal}`)
})
</script>
```

**With Options:**
```vue
<script setup lang="ts">
import { ref, watch } from 'vue'

const userId = ref(1)
const userData = ref(null)

watch(
  userId,
  async (newId) => {
    const response = await fetch(`/api/users/${newId}`)
    userData.value = await response.json()
  },
  { 
    immediate: true,
    flush: 'post'
  }
)
</script>
```

**Advanced Form (Debounced + Cleanup):**
```vue
<script setup lang="ts">
import { ref, watch, onWatcherCleanup } from 'vue'

const searchQuery = ref('')
const results = ref<string[]>([])
const isLoading = ref(false)
const error = ref<string | null>(null)

watch(searchQuery, async (newQuery) => {
  error.value = null
  
  if (!newQuery.trim()) {
    results.value = []
    return
  }
  
  let timeoutId: number
  const controller = new AbortController()
  
  onWatcherCleanup(() => {
    clearTimeout(timeoutId)
    controller.abort()
  })
  
  await new Promise<void>((resolve) => {
    timeoutId = window.setTimeout(resolve, 300)
  })
  
  isLoading.value = true
  
  try {
    const response = await fetch(`/api/search?q=${encodeURIComponent(newQuery)}`, {
      signal: controller.signal
    })
    
    if (!response.ok) throw new Error('Search failed')
    
    results.value = await response.json()
  } catch (e) {
    if ((e as Error).name !== 'AbortError') {
      error.value = (e as Error).message
      results.value = []
    }
  } finally {
    isLoading.value = false
  }
})
</script>
```

---

## 4. Micro-Implementation Drills

**Drill 1 (Basic): Auto-Save to localStorage**

Create a notes input that automatically saves to localStorage.

```vue
<script setup lang="ts">
import { ref, watch, onMounted } from 'vue'

const STORAGE_KEY = 'user-notes'

// TODO: Create ref 'notes' initialized to empty string
// TODO: Create ref 'lastSaved' for timestamp display
// TODO: On mount, load notes from localStorage
// TODO: Watch notes and save to localStorage (with timestamp update)

</script>

<template>
  <div>
    <!-- TODO: Textarea bound to notes -->
    <!-- TODO: Display last saved time -->
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref, watch, onMounted } from 'vue'

const STORAGE_KEY = 'user-notes'

const notes = ref('')
const lastSaved = ref<string | null>(null)

onMounted(() => {
  const saved = localStorage.getItem(STORAGE_KEY)
  if (saved) {
    notes.value = saved
  }
})

watch(notes, (newNotes) => {
  localStorage.setItem(STORAGE_KEY, newNotes)
  lastSaved.value = new Date().toLocaleTimeString()
})
</script>

<template>
  <div class="p-4">
    <textarea
      v-model="notes"
      class="w-full border p-3 rounded-lg"
      rows="6"
      placeholder="Start typing your notes..."
    ></textarea>
    <p v-if="lastSaved" class="text-sm text-gray-500 mt-2">
      Auto-saved at {{ lastSaved }}
    </p>
  </div>
</template>
```

---

**Drill 2 (Variation): Form Validation on Change**

Create an email input with real-time validation.

```vue
<script setup lang="ts">
import { ref, watch } from 'vue'

// TODO: Create ref 'email' for input
// TODO: Create ref 'isValid' for validation state
// TODO: Create ref 'errorMessage' for error display
// TODO: Watch email and validate with regex
// TODO: Show appropriate error messages for different cases

</script>

<template>
  <div>
    <!-- TODO: Email input with visual validation feedback -->
    <!-- TODO: Error message display -->
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref, watch } from 'vue'

const email = ref('')
const isValid = ref<boolean | null>(null)
const errorMessage = ref('')

const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/

watch(email, (newEmail) => {
  if (!newEmail) {
    isValid.value = null
    errorMessage.value = ''
    return
  }
  
  if (!newEmail.includes('@')) {
    isValid.value = false
    errorMessage.value = 'Email must contain @'
    return
  }
  
  if (!emailRegex.test(newEmail)) {
    isValid.value = false
    errorMessage.value = 'Please enter a valid email address'
    return
  }
  
  isValid.value = true
  errorMessage.value = ''
})
</script>

<template>
  <div class="p-4 max-w-md">
    <label class="block text-sm font-medium mb-1">Email</label>
    <div class="relative">
      <input
        v-model="email"
        type="email"
        class="w-full border p-2 rounded pr-10"
        :class="{
          'border-red-500': isValid === false,
          'border-green-500': isValid === true
        }"
        placeholder="you@example.com"
      />
      <span 
        v-if="isValid !== null" 
        class="absolute right-3 top-1/2 -translate-y-1/2"
      >
        {{ isValid ? '✓' : '✗' }}
      </span>
    </div>
    <p 
      v-if="errorMessage" 
      class="text-red-500 text-sm mt-1"
    >
      {{ errorMessage }}
    </p>
    <p 
      v-else-if="isValid" 
      class="text-green-500 text-sm mt-1"
    >
      Valid email address
    </p>
  </div>
</template>
```

---

**Drill 3 (Combination): Debounced Search with Loading State**

Create a search input that debounces API calls and shows loading state.

```vue
<script setup lang="ts">
import { ref, watch, onWatcherCleanup } from 'vue'

// TODO: Create refs for: searchQuery, results, isLoading, error
// TODO: Watch searchQuery with debounce (300ms delay)
// TODO: Simulate API call with setTimeout
// TODO: Handle cleanup when query changes during delay

</script>

<template>
  <div>
    <!-- TODO: Search input -->
    <!-- TODO: Loading indicator -->
    <!-- TODO: Error message -->
    <!-- TODO: Results list -->
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref, watch, onWatcherCleanup } from 'vue'

const searchQuery = ref('')
const results = ref<string[]>([])
const isLoading = ref(false)
const error = ref<string | null>(null)

const simulateSearch = async (query: string): Promise<string[]> => {
  await new Promise(resolve => setTimeout(resolve, 500))
  
  if (query.toLowerCase() === 'error') {
    throw new Error('Search failed')
  }
  
  return [
    `${query} - Result 1`,
    `${query} - Result 2`,
    `${query} - Result 3`
  ].filter(r => r.toLowerCase().includes(query.toLowerCase()))
}

watch(searchQuery, async (newQuery) => {
  error.value = null
  
  if (!newQuery.trim()) {
    results.value = []
    isLoading.value = false
    return
  }
  
  let cancelled = false
  let timeoutId: number
  
  onWatcherCleanup(() => {
    cancelled = true
    clearTimeout(timeoutId)
  })
  
  await new Promise<void>(resolve => {
    timeoutId = window.setTimeout(resolve, 300)
  })
  
  if (cancelled) return
  
  isLoading.value = true
  
  try {
    const searchResults = await simulateSearch(newQuery)
    if (!cancelled) {
      results.value = searchResults
    }
  } catch (e) {
    if (!cancelled) {
      error.value = (e as Error).message
      results.value = []
    }
  } finally {
    if (!cancelled) {
      isLoading.value = false
    }
  }
})
</script>

<template>
  <div class="p-4 max-w-md">
    <div class="relative">
      <input
        v-model="searchQuery"
        class="w-full border p-3 rounded-lg pr-10"
        placeholder="Search... (try 'error' to test error handling)"
      />
      <span 
        v-if="isLoading" 
        class="absolute right-3 top-1/2 -translate-y-1/2 animate-spin"
      >
        ⏳
      </span>
    </div>
    
    <p v-if="error" class="text-red-500 mt-2">{{ error }}</p>
    
    <ul v-else-if="results.length > 0" class="mt-4 space-y-2">
      <li 
        v-for="result in results" 
        :key="result"
        class="p-2 bg-gray-100 rounded"
      >
        {{ result }}
      </li>
    </ul>
    
    <p 
      v-else-if="searchQuery && !isLoading" 
      class="text-gray-500 mt-4"
    >
      No results found
    </p>
  </div>
</template>
```

---

**Drill 4 (Edge Case): watchEffect with Cleanup**

Create a component that tracks mouse position using watchEffect.

```vue
<script setup lang="ts">
import { ref, watchEffect } from 'vue'

// TODO: Create refs for: x, y, isTracking
// TODO: Use watchEffect to add/remove mousemove listener based on isTracking
// TODO: Properly clean up the event listener

</script>

<template>
  <div>
    <!-- TODO: Toggle button for tracking -->
    <!-- TODO: Display coordinates when tracking -->
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref, watchEffect } from 'vue'

const x = ref(0)
const y = ref(0)
const isTracking = ref(false)

watchEffect((onCleanup) => {
  if (!isTracking.value) return
  
  const handleMouseMove = (event: MouseEvent) => {
    x.value = event.clientX
    y.value = event.clientY
  }
  
  window.addEventListener('mousemove', handleMouseMove)
  
  onCleanup(() => {
    window.removeEventListener('mousemove', handleMouseMove)
  })
})
</script>

<template>
  <div class="p-4">
    <button
      @click="isTracking = !isTracking"
      :class="[
        'px-4 py-2 rounded-lg transition',
        isTracking ? 'bg-red-500 text-white' : 'bg-blue-500 text-white'
      ]"
    >
      {{ isTracking ? 'Stop Tracking' : 'Start Tracking' }}
    </button>
    
    <div v-if="isTracking" class="mt-4 p-4 bg-gray-100 rounded-lg">
      <p class="font-mono">
        Mouse Position: ({{ x }}, {{ y }})
      </p>
    </div>
    
    <p v-else class="mt-4 text-gray-500">
      Click the button to start tracking mouse position
    </p>
  </div>
</template>
```

---

## 5. Common Pitfalls & Anti-Patterns

| ❌ Common Mistake | ✅ Correct Approach | Why It Matters |
|:--- |:--- |:--- |
| Using watch for derived values | Use `computed()` for derived values | Watch is for side effects, computed is for calculations |
| Forgetting async/await in async watchers | Add `async` to callback function | Without async, promises won't be awaited properly |
| No cleanup for pending operations | Use `onWatcherCleanup()` | Race conditions occur when query changes during async operation |
| Watching reactive object directly for specific prop | Use getter: `() => state.prop` | Watching whole object triggers on ANY property change |

**Code Examples:**

```typescript
// ❌ Using watch for derived value
watch(items, () => {
  total.value = items.value.reduce((s, i) => s + i.price, 0)
})

// ✅ Use computed instead
const total = computed(() => 
  items.value.reduce((s, i) => s + i.price, 0)
)

// ❌ Forgetting cleanup
watch(query, async (newQuery) => {
  const data = await fetch(`/api?q=${newQuery}`)
  results.value = await data.json()
  // If query changes mid-fetch, stale results could be displayed!
})

// ✅ With cleanup
watch(query, async (newQuery) => {
  const controller = new AbortController()
  onWatcherCleanup(() => controller.abort())
  
  try {
    const response = await fetch(`/api?q=${newQuery}`, { 
      signal: controller.signal 
    })
    results.value = await response.json()
  } catch (e) {
    if ((e as Error).name !== 'AbortError') throw e
  }
})

// ❌ Watching whole object when you need one property
watch(settings, () => {
  // Runs when ANY setting changes!
  localStorage.setItem('theme', settings.theme)
})

// ✅ Watch specific property
watch(() => settings.theme, (newTheme) => {
  localStorage.setItem('theme', newTheme)
})
```

---

## 6. Mini-Project: Temperature Converter with Validation

**File Structure:**
```
src/
├── components/
│   └── TemperatureWatch.vue
├── App.vue
└── main.ts
```

**components/TemperatureWatch.vue:**
```vue
<script setup lang="ts">
// ============================================
// TEMPERATURE CONVERTER WITH WATCH
// Demonstrates watchers for real-time validation,
// localStorage sync, and conversion history
// ============================================

import { ref, watch, watchEffect, onMounted } from 'vue'

// ============================================
// TYPE DEFINITIONS
// ============================================

interface ConversionEntry {
  celsius: number
  fahrenheit: number
  timestamp: string
}

interface ValidationState {
  isValid: boolean
  message: string
}

// ============================================
// CONSTANTS
// ============================================

const MIN_TEMP = -273.15
const MAX_TEMP = 1000
const STORAGE_KEY = 'temp-converter-history'

// ============================================
// REACTIVE STATE
// ============================================

const celsiusInput = ref('')
const fahrenheit = ref<number | null>(null)
const validation = ref<ValidationState>({ isValid: true, message: '' })
const history = ref<ConversionEntry[]>([])
const isConverting = ref(false)
const lastUpdated = ref<string | null>(null)

// ============================================
// WATCHERS
// ============================================

watch(celsiusInput, (newValue) => {
  if (!newValue.trim()) {
    validation.value = { isValid: true, message: '' }
    fahrenheit.value = null
    return
  }
  
  const parsed = parseFloat(newValue)
  
  if (isNaN(parsed)) {
    validation.value = {
      isValid: false,
      message: 'Please enter a valid number'
    }
    fahrenheit.value = null
    return
  }
  
  if (parsed < MIN_TEMP) {
    validation.value = {
      isValid: false,
      message: `Temperature cannot be below absolute zero (${MIN_TEMP}°C)`
    }
    fahrenheit.value = null
    return
  }
  
  if (parsed > MAX_TEMP) {
    validation.value = {
      isValid: false,
      message: `Temperature too high (max ${MAX_TEMP}°C)`
    }
    fahrenheit.value = null
    return
  }
  
  validation.value = { isValid: true, message: '' }
  
  isConverting.value = true
  setTimeout(() => {
    fahrenheit.value = (parsed * 9/5) + 32
    isConverting.value = false
    lastUpdated.value = new Date().toLocaleTimeString()
  }, 200)
})

watch(fahrenheit, (newFahrenheit) => {
  if (newFahrenheit === null) return
  
  const celsius = parseFloat(celsiusInput.value)
  if (isNaN(celsius)) return
  
  const entry: ConversionEntry = {
    celsius: Math.round(celsius * 100) / 100,
    fahrenheit: Math.round(newFahrenheit * 100) / 100,
    timestamp: new Date().toLocaleTimeString()
  }
  
  const existingIndex = history.value.findIndex(
    h => h.celsius === entry.celsius
  )
  
  if (existingIndex === -1) {
    history.value.unshift(entry)
    if (history.value.length > 10) {
      history.value.pop()
    }
  }
})

watch(
  history,
  (newHistory) => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(newHistory))
  },
  { deep: true }
)

watchEffect(() => {
  const celsius = parseFloat(celsiusInput.value)
  if (!isNaN(celsius) && celsius >= -40 && celsius <= 40) {
    document.documentElement.style.setProperty(
      '--temp-hue',
      String(240 - ((celsius + 40) / 80) * 240)
    )
  }
})

// ============================================
// LIFECYCLE
// ============================================

onMounted(() => {
  const saved = localStorage.getItem(STORAGE_KEY)
  if (saved) {
    try {
      history.value = JSON.parse(saved)
    } catch {
      history.value = []
    }
  }
})

// ============================================
// ACTIONS
// ============================================

const loadFromHistory = (entry: ConversionEntry): void => {
  celsiusInput.value = entry.celsius.toString()
}

const clearHistory = (): void => {
  history.value = []
  localStorage.removeItem(STORAGE_KEY)
}

const clearInput = (): void => {
  celsiusInput.value = ''
  fahrenheit.value = null
}

// ============================================
// COMPUTED HELPERS
// ============================================

const getTemperatureEmoji = (): string => {
  const celsius = parseFloat(celsiusInput.value)
  if (isNaN(celsius)) return '🌡️'
  if (celsius < 0) return '❄️'
  if (celsius < 10) return '🥶'
  if (celsius < 20) return '😊'
  if (celsius < 30) return '☀️'
  if (celsius < 40) return '🥵'
  return '🔥'
}
</script>

<template>
  <div class="max-w-lg mx-auto">
    <div class="bg-white rounded-2xl shadow-xl overflow-hidden">
      <div 
        class="p-6 text-white transition-colors duration-500"
        :style="{ 
          backgroundColor: `hsl(var(--temp-hue, 200), 70%, 50%)` 
        }"
      >
        <h2 class="text-2xl font-bold flex items-center gap-2">
          {{ getTemperatureEmoji() }} Temperature Converter
        </h2>
        <p class="opacity-80">Real-time validation with watch()</p>
      </div>

      <div class="p-6 space-y-6">
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1">
            Celsius (°C)
          </label>
          <div class="relative">
            <input
              v-model="celsiusInput"
              type="text"
              inputmode="decimal"
              class="w-full border-2 rounded-lg p-3 text-xl transition-colors"
              :class="{
                'border-red-500 bg-red-50': !validation.isValid,
                'border-green-500 bg-green-50': validation.isValid && fahrenheit !== null,
                'border-gray-200': validation.isValid && fahrenheit === null
              }"
              placeholder="Enter temperature in Celsius"
            />
            <button
              v-if="celsiusInput"
              @click="clearInput"
              class="absolute right-3 top-1/2 -translate-y-1/2 text-gray-400 hover:text-gray-600"
            >
              ✕
            </button>
          </div>
          <p 
            v-if="!validation.isValid" 
            class="text-red-500 text-sm mt-1"
          >
            {{ validation.message }}
          </p>
        </div>

        <div 
          v-if="fahrenheit !== null || isConverting"
          class="bg-gray-50 rounded-lg p-4"
        >
          <div class="flex items-center justify-between">
            <span class="text-gray-600">Fahrenheit (°F)</span>
            <span v-if="isConverting" class="animate-pulse">Converting...</span>
            <span v-else class="text-2xl font-bold">
              {{ fahrenheit?.toFixed(2) }}°F
            </span>
          </div>
          <p 
            v-if="lastUpdated" 
            class="text-xs text-gray-400 mt-1"
          >
            Converted at {{ lastUpdated }}
          </p>
        </div>

        <div class="text-sm text-gray-500 bg-blue-50 p-3 rounded-lg">
          <strong>Validation Rules:</strong>
          <ul class="mt-1 space-y-1">
            <li>• Must be a valid number</li>
            <li>• Minimum: {{ MIN_TEMP }}°C (absolute zero)</li>
            <li>• Maximum: {{ MAX_TEMP }}°C</li>
          </ul>
        </div>

        <div v-if="history.length > 0">
          <div class="flex justify-between items-center mb-2">
            <h3 class="font-semibold text-gray-700">Conversion History</h3>
            <button
              @click="clearHistory"
              class="text-sm text-red-500 hover:text-red-700"
            >
              Clear All
            </button>
          </div>
          <ul class="space-y-1">
            <li
              v-for="(entry, index) in history"
              :key="index"
              @click="loadFromHistory(entry)"
              class="flex justify-between p-2 bg-gray-50 rounded cursor-pointer hover:bg-gray-100 transition"
            >
              <span>{{ entry.celsius }}°C = {{ entry.fahrenheit }}°F</span>
              <span class="text-gray-400 text-sm">{{ entry.timestamp }}</span>
            </li>
          </ul>
        </div>
      </div>
    </div>

    <div class="mt-6 bg-white rounded-lg shadow p-4">
      <h3 class="font-semibold mb-2">Watch Patterns Used:</h3>
      <ul class="text-sm text-gray-600 space-y-1">
        <li>✓ <code>watch(input)</code> — Input validation</li>
        <li>✓ <code>watch(fahrenheit)</code> — History recording</li>
        <li>✓ <code>watch(history, {deep})</code> — localStorage sync</li>
        <li>✓ <code>watchEffect()</code> — Dynamic background color</li>
      </ul>
    </div>
  </div>
</template>

<style>
:root {
  --temp-hue: 200;
}
</style>
```

**App.vue:**
```vue
<script setup lang="ts">
import TemperatureWatch from './components/TemperatureWatch.vue'
</script>

<template>
  <div class="min-h-screen bg-gray-100 py-12">
    <h1 class="text-3xl font-bold text-center text-gray-800 mb-8">
      Vue Watchers Demo
    </h1>
    <TemperatureWatch />
  </div>
</template>
```

**How to Test:**

1. Type a valid number and see the Fahrenheit conversion appear
2. Type "abc" or other invalid input to see validation errors
3. Try typing a number below -273.15 or above 1000
4. Notice the background color changes with temperature
5. Check that conversion history persists after page refresh
6. Click a history entry to load it back

---

## 7. Implementation Exercises (Progressive)

### Level 1 (Recall): Fill in the Blanks

```vue
<script setup lang="ts">
import { ref, _____ } from 'vue'

const count = ref(0)

_____(count, (newVal, _____) => {
  console.log(`Changed from ${oldVal} to ${_____}`)
})
</script>
```

**Answer:** `watch`, `watch`, `oldVal`, `newVal`

---

### Level 2 (Apply): Small Coding Tasks

**Exercise 2.1:** Create a `DarkModeToggle.vue` component with:
- A `isDarkMode` ref (boolean)
- Watch that updates `document.body.classList` when mode changes
- Watch that saves preference to localStorage
- Load preference from localStorage on mount

**Exercise 2.2:** Create a `WindowSize.vue` component with:
- Refs for `width` and `height`
- A `watchEffect` that adds/removes resize event listener
- Proper cleanup when component unmounts
- Display current window dimensions

---

### Level 3 (Debug): Fix the Broken Code

```vue
<script setup lang="ts">
import { reactive, watch } from 'vue'

const state = reactive({
  search: '',
  filter: 'all'
})

const results = ref([])

watch(state.search, async (newSearch) => {
  const data = await fetch(`/api?q=${newSearch}`)
  results = await data.json()
})

watch(state, () => {
  console.log('State changed')
}, { deep: false })
</script>

<template>
  <input v-model="state.search" />
  <ul>
    <li v-for="r in results">{{ r }}</li>
  </ul>
</template>
```

**Bugs to find:**
1. `results` not imported from vue
2. Watching `state.search` directly instead of using getter
3. Assigning to `results` instead of `results.value`
4. Missing `key` in v-for
5. `deep: false` is redundant (default behavior)

**Fixed Version:**
```vue
<script setup lang="ts">
import { reactive, ref, watch } from 'vue'

const state = reactive({
  search: '',
  filter: 'all'
})

const results = ref<string[]>([])

watch(
  () => state.search,
  async (newSearch) => {
    const response = await fetch(`/api?q=${newSearch}`)
    results.value = await response.json()
  }
)

watch(state, () => {
  console.log('State changed')
}, { deep: true })
</script>

<template>
  <input v-model="state.search" />
  <ul>
    <li v-for="r in results" :key="r">{{ r }}</li>
  </ul>
</template>
```

---

## Self-Check Solutions

**Exercise 2.1 Solution:**
```vue
<script setup lang="ts">
import { ref, watch, onMounted } from 'vue'

const STORAGE_KEY = 'dark-mode-preference'
const isDarkMode = ref(false)

onMounted(() => {
  const saved = localStorage.getItem(STORAGE_KEY)
  if (saved !== null) {
    isDarkMode.value = saved === 'true'
  } else {
    isDarkMode.value = window.matchMedia('(prefers-color-scheme: dark)').matches
  }
})

watch(isDarkMode, (newValue) => {
  if (newValue) {
    document.body.classList.add('dark')
  } else {
    document.body.classList.remove('dark')
  }
  
  localStorage.setItem(STORAGE_KEY, String(newValue))
}, { immediate: true })
</script>

<template>
  <button
    @click="isDarkMode = !isDarkMode"
    class="px-4 py-2 rounded-lg transition"
    :class="isDarkMode ? 'bg-gray-800 text-white' : 'bg-gray-200 text-gray-800'"
  >
    {{ isDarkMode ? '🌙 Dark' : '☀️ Light' }} Mode
  </button>
</template>
```

**Exercise 2.2 Solution:**
```vue
<script setup lang="ts">
import { ref, watchEffect } from 'vue'

const width = ref(window.innerWidth)
const height = ref(window.innerHeight)

watchEffect((onCleanup) => {
  const handleResize = () => {
    width.value = window.innerWidth
    height.value = window.innerHeight
  }
  
  window.addEventListener('resize', handleResize)
  
  onCleanup(() => {
    window.removeEventListener('resize', handleResize)
  })
})

const getSizeCategory = (): string => {
  if (width.value < 640) return 'Mobile'
  if (width.value < 1024) return 'Tablet'
  return 'Desktop'
}
</script>

<template>
  <div class="p-4 bg-gray-100 rounded-lg">
    <h3 class="font-bold mb-2">Window Size</h3>
    <p class="font-mono">{{ width }} × {{ height }}</p>
    <p class="text-sm text-gray-600">
      Category: {{ getSizeCategory() }}
    </p>
  </div>
</template>
```

---

## 🎉 Module 1 Complete!

Congratulations! You've completed Module 1: Core Fundamentals. You now understand:

1. **Components** — Single File Components with `<script setup>`
2. **`ref()`** — Reactive primitives
3. **`reactive()`** — Reactive objects
4. **`computed()`** — Derived state with caching
5. **`watch()`** — Side effects on data changes

**Next Steps:**
- Complete the Module 1 Checkpoint Projects
- Move on to Module 2: Component Communication (Props & Emits)

---

*Reply 'next' for Module 1 Checkpoint Projects or Lesson 6.1 (Component Communication: Props).*
