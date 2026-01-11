# Lesson 17.2: Building a `useFetch` Composable — Practice & Application (Создание композабла useFetch)

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 17.2 of 25 (Part 2 of 2) |
| **Topic** | Building a `useFetch` Composable / Создание композабла useFetch |
| **Continues From** | Lesson 17.1 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Build a complete `useFetch` composable with TypeScript generics and use it across multiple components
2. **Analyze:** Extend the composable to support POST/PUT/DELETE methods and compare it to production libraries

---

## 2. Quick Recap

From Lesson 17.1:
- **Composables** encapsulate reusable reactive logic — `useFetch` extracts the repetitive data fetching pattern
- **TypeScript generics** (`<T>`) allow one function to work with any data type while maintaining type safety
- The basic `useFetch` returns an object with `data`, `loading`, `error`, and `refetch`
- Use `unref()` to handle both plain strings and reactive refs as URLs
- `watch()` enables automatic re-fetching when a reactive URL changes

---

## 3. The Variable Frame & Complexity Scale

### Basic Form: Simple GET-only Composable
```typescript
import { ref, onMounted, type Ref } from 'vue'

export function useFetch<T>(url: string) {
  const data = ref<T | null>(null) as Ref<T | null>
  const loading = ref(false)
  const error = ref<string | null>(null)

  async function execute() {
    loading.value = true
    try {
      const response = await fetch(url)
      data.value = await response.json()
    } catch (e) {
      error.value = (e as Error).message
    } finally {
      loading.value = false
    }
  }

  onMounted(execute)

  return { data, loading, error, refetch: execute }
}
```

### With Options: Configurable Immediate & Reactive URL
```typescript
import { ref, onMounted, watch, isRef, unref, type Ref, type MaybeRef } from 'vue'

interface UseFetchOptions {
  immediate?: boolean
}

export function useFetch<T>(url: MaybeRef<string>, options: UseFetchOptions = {}) {
  const { immediate = true } = options
  
  const data = ref<T | null>(null) as Ref<T | null>
  const loading = ref(false)
  const error = ref<string | null>(null)

  async function execute() {
    loading.value = true
    error.value = null
    try {
      const response = await fetch(unref(url))
      if (!response.ok) throw new Error(`Status: ${response.status}`)
      data.value = await response.json()
    } catch (e) {
      error.value = (e as Error).message
    } finally {
      loading.value = false
    }
  }

  if (immediate) onMounted(execute)
  if (isRef(url)) watch(url, execute)

  return { data, loading, error, refetch: execute }
}
```

### Advanced Form: Full CRUD Support with AbortController
```typescript
import { ref, onMounted, onUnmounted, watch, isRef, unref, type Ref, type MaybeRef } from 'vue'

type HttpMethod = 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE'

interface UseFetchOptions<TBody = unknown> {
  method?: HttpMethod
  body?: TBody
  headers?: Record<string, string>
  immediate?: boolean
}

interface UseFetchReturn<T> {
  data: Ref<T | null>
  loading: Ref<boolean>
  error: Ref<string | null>
  execute: (overrideBody?: unknown) => Promise<void>
  abort: () => void
}

export function useFetch<T, TBody = unknown>(
  url: MaybeRef<string>,
  options: UseFetchOptions<TBody> = {}
): UseFetchReturn<T> {
  const {
    method = 'GET',
    body,
    headers = {},
    immediate = method === 'GET'
  } = options

  const data = ref<T | null>(null) as Ref<T | null>
  const loading = ref(false)
  const error = ref<string | null>(null)
  
  let abortController: AbortController | null = null

  function abort() {
    abortController?.abort()
  }

  async function execute(overrideBody?: unknown): Promise<void> {
    abort()
    abortController = new AbortController()
    
    loading.value = true
    error.value = null

    const requestBody = overrideBody ?? body

    try {
      const response = await fetch(unref(url), {
        method,
        headers: {
          'Content-Type': 'application/json',
          ...headers
        },
        body: requestBody ? JSON.stringify(requestBody) : undefined,
        signal: abortController.signal
      })

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`)
      }

      const contentType = response.headers.get('content-type')
      if (contentType?.includes('application/json')) {
        data.value = await response.json()
      } else {
        data.value = null
      }
    } catch (e) {
      if (e instanceof Error && e.name !== 'AbortError') {
        error.value = e.message
      }
    } finally {
      loading.value = false
    }
  }

  if (immediate) onMounted(execute)
  if (isRef(url)) watch(url, () => execute())
  onUnmounted(abort)

  return { data, loading, error, execute, abort }
}
```

---

## 4. Micro-Implementation Drills

### Drill 1 (Basic): Create the Minimal useFetch

Build the simplest version of `useFetch` that fetches data on mount.

```typescript
// src/composables/useFetch.ts
import { ref, onMounted, type Ref } from 'vue'

// TODO: Define the generic function useFetch<T> that takes a url string
// TODO: Create refs for data, loading, and error
// TODO: Create an async execute function
// TODO: Call execute in onMounted
// TODO: Return the refs and execute as refetch

export function useFetch<T>(url: string) {
  // YOUR CODE HERE
}
```

**Solution:**
```typescript
import { ref, onMounted, type Ref } from 'vue'

export function useFetch<T>(url: string) {
  const data = ref<T | null>(null) as Ref<T | null>
  const loading = ref(false)
  const error = ref<string | null>(null)

  async function execute() {
    loading.value = true
    error.value = null
    try {
      const response = await fetch(url)
      if (!response.ok) throw new Error(`HTTP ${response.status}`)
      data.value = await response.json()
    } catch (e) {
      error.value = e instanceof Error ? e.message : 'Unknown error'
    } finally {
      loading.value = false
    }
  }

  onMounted(execute)

  return { data, loading, error, refetch: execute }
}
```

---

### Drill 2 (Variation): Add Reactive URL Support

Extend your composable to accept both `string` and `Ref<string>` URLs, re-fetching when the URL changes.

```typescript
import { ref, onMounted, watch, isRef, unref, type Ref, type MaybeRef } from 'vue'

export function useFetch<T>(url: MaybeRef<string>) {
  const data = ref<T | null>(null) as Ref<T | null>
  const loading = ref(false)
  const error = ref<string | null>(null)

  async function execute() {
    loading.value = true
    error.value = null
    try {
      // TODO: Use unref(url) to get the string value
      const response = await fetch(/* YOUR CODE HERE */)
      if (!response.ok) throw new Error(`HTTP ${response.status}`)
      data.value = await response.json()
    } catch (e) {
      error.value = e instanceof Error ? e.message : 'Unknown error'
    } finally {
      loading.value = false
    }
  }

  onMounted(execute)
  
  // TODO: Add a watch that calls execute when url changes (only if url is a Ref)
  // YOUR CODE HERE

  return { data, loading, error, refetch: execute }
}
```

**Solution:**
```typescript
import { ref, onMounted, watch, isRef, unref, type Ref, type MaybeRef } from 'vue'

export function useFetch<T>(url: MaybeRef<string>) {
  const data = ref<T | null>(null) as Ref<T | null>
  const loading = ref(false)
  const error = ref<string | null>(null)

  async function execute() {
    loading.value = true
    error.value = null
    try {
      const response = await fetch(unref(url))
      if (!response.ok) throw new Error(`HTTP ${response.status}`)
      data.value = await response.json()
    } catch (e) {
      error.value = e instanceof Error ? e.message : 'Unknown error'
    } finally {
      loading.value = false
    }
  }

  onMounted(execute)
  
  if (isRef(url)) {
    watch(url, execute)
  }

  return { data, loading, error, refetch: execute }
}
```

---

### Drill 3 (Combination): Use useFetch in a Component with Props

Create a `UserCard` component that uses `useFetch` with a reactive URL based on a prop.

```vue
<script setup lang="ts">
import { computed } from 'vue'
import { useFetch } from '@/composables/useFetch'

interface User {
  id: number
  name: string
  email: string
  phone: string
}

const props = defineProps<{
  userId: number
}>()

// TODO: Create a computed ref for the URL based on props.userId
// TODO: Pass the computed URL to useFetch
// YOUR CODE HERE
</script>

<template>
  <div class="card bg-base-100 shadow-xl">
    <div class="card-body">
      <!-- TODO: Show loading spinner, error state, or user data -->
      <!-- YOUR CODE HERE -->
    </div>
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { computed } from 'vue'
import { useFetch } from '@/composables/useFetch'

interface User {
  id: number
  name: string
  email: string
  phone: string
}

const props = defineProps<{
  userId: number
}>()

const apiUrl = computed(() => `https://jsonplaceholder.typicode.com/users/${props.userId}`)

const { data: user, loading, error, refetch } = useFetch<User>(apiUrl)
</script>

<template>
  <div class="card bg-base-100 shadow-xl">
    <div class="card-body">
      <div v-if="loading" class="flex justify-center">
        <span class="loading loading-spinner loading-lg"></span>
      </div>
      <div v-else-if="error" class="alert alert-error">
        {{ error }}
        <button class="btn btn-sm" @click="refetch">Retry</button>
      </div>
      <template v-else-if="user">
        <h2 class="card-title">{{ user.name }}</h2>
        <p>{{ user.email }}</p>
        <p>{{ user.phone }}</p>
      </template>
    </div>
  </div>
</template>
```

---

### Drill 4 (Edge Case): Create a POST Request Composable

Build a variant of `useFetch` that supports POST requests for creating new resources.

```typescript
// src/composables/usePost.ts
import { ref, type Ref } from 'vue'

interface UsePostReturn<T> {
  data: Ref<T | null>
  loading: Ref<boolean>
  error: Ref<string | null>
  execute: (body: unknown) => Promise<void>
}

// TODO: Create usePost<T>(url: string) that returns UsePostReturn<T>
// The execute function should accept a body parameter and POST it as JSON
export function usePost<T>(url: string): UsePostReturn<T> {
  // YOUR CODE HERE
}
```

**Solution:**
```typescript
import { ref, type Ref } from 'vue'

interface UsePostReturn<T> {
  data: Ref<T | null>
  loading: Ref<boolean>
  error: Ref<string | null>
  execute: (body: unknown) => Promise<void>
}

export function usePost<T>(url: string): UsePostReturn<T> {
  const data = ref<T | null>(null) as Ref<T | null>
  const loading = ref(false)
  const error = ref<string | null>(null)

  async function execute(body: unknown): Promise<void> {
    loading.value = true
    error.value = null

    try {
      const response = await fetch(url, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(body)
      })

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`)
      }

      data.value = await response.json()
    } catch (e) {
      error.value = e instanceof Error ? e.message : 'Unknown error'
    } finally {
      loading.value = false
    }
  }

  return { data, loading, error, execute }
}
```

---

## 5. Common Pitfalls & Anti-Patterns

| ❌ Common Mistake | ✅ Correct Approach | Why It Matters |
|:--- |:--- |:--- |
| Forgetting the type assertion: `const data = ref<T \| null>(null)` | Add `as Ref<T \| null>`: `const data = ref<T \| null>(null) as Ref<T \| null>` | TypeScript infers `Ref<null>` without the assertion, losing your generic type. The cast preserves the full union type. |
| Watching a plain string URL: `watch(url, execute)` when `url` is not a Ref | Guard with `isRef()`: `if (isRef(url)) { watch(url, execute) }` | Watching a non-reactive value does nothing and may cause confusion. Only watch refs. |
| Not resetting error before a new fetch: `async function execute() { loading.value = true; try { ... }` | Always reset: `error.value = null` at the start of execute | Stale errors from previous requests will display alongside new data, confusing users. |

---

## 6. Mini-Project: Weather Dashboard with useFetch

Build a weather dashboard that fetches current weather data using your `useFetch` composable.

**File Structure:**
```
src/
├── composables/
│   └── useFetch.ts
├── components/
│   └── WeatherCard.vue
├── App.vue
└── main.ts
```

**useFetch.ts:**
```typescript
import { ref, onMounted, onUnmounted, watch, isRef, unref, type Ref, type MaybeRef } from 'vue'

interface UseFetchOptions {
  immediate?: boolean
}

interface UseFetchReturn<T> {
  data: Ref<T | null>
  loading: Ref<boolean>
  error: Ref<string | null>
  refetch: () => Promise<void>
}

export function useFetch<T>(
  url: MaybeRef<string>,
  options: UseFetchOptions = {}
): UseFetchReturn<T> {
  const { immediate = true } = options

  const data = ref<T | null>(null) as Ref<T | null>
  const loading = ref(false)
  const error = ref<string | null>(null)

  let abortController: AbortController | null = null

  async function execute(): Promise<void> {
    if (abortController) {
      abortController.abort()
    }
    abortController = new AbortController()

    loading.value = true
    error.value = null

    try {
      const resolvedUrl = unref(url)
      const response = await fetch(resolvedUrl, {
        signal: abortController.signal
      })

      if (!response.ok) {
        throw new Error(`HTTP error! Status: ${response.status}`)
      }

      data.value = await response.json()
    } catch (e) {
      if (e instanceof Error && e.name !== 'AbortError') {
        error.value = e.message
      }
    } finally {
      loading.value = false
    }
  }

  if (immediate) {
    onMounted(execute)
  }

  if (isRef(url)) {
    watch(url, execute)
  }

  onUnmounted(() => {
    abortController?.abort()
  })

  return {
    data,
    loading,
    error,
    refetch: execute
  }
}
```

**WeatherCard.vue:**
```vue
<script setup lang="ts">
import { computed } from 'vue'
import { useFetch } from '@/composables/useFetch'

interface WeatherData {
  name: string
  main: {
    temp: number
    feels_like: number
    humidity: number
  }
  weather: Array<{
    main: string
    description: string
    icon: string
  }>
  wind: {
    speed: number
  }
}

const props = defineProps<{
  city: string
  apiKey: string
}>()

const apiUrl = computed(
  () => `https://api.openweathermap.org/data/2.5/weather?q=${props.city}&appid=${props.apiKey}&units=metric`
)

const { data: weather, loading, error, refetch } = useFetch<WeatherData>(apiUrl)

const weatherIcon = computed(() => {
  if (!weather.value?.weather[0]) return ''
  return `https://openweathermap.org/img/wn/${weather.value.weather[0].icon}@2x.png`
})
</script>

<template>
  <div class="card bg-base-100 shadow-xl w-80">
    <div class="card-body">
      <div v-if="loading" class="flex flex-col items-center py-8">
        <span class="loading loading-spinner loading-lg"></span>
        <span class="mt-2">Loading weather...</span>
      </div>

      <div v-else-if="error" class="alert alert-error">
        <div class="flex flex-col gap-2">
          <span>{{ error }}</span>
          <button class="btn btn-sm btn-ghost" @click="refetch">Try Again</button>
        </div>
      </div>

      <template v-else-if="weather">
        <div class="flex items-center justify-between">
          <div>
            <h2 class="card-title text-2xl">{{ weather.name }}</h2>
            <p class="text-base-content/60 capitalize">
              {{ weather.weather[0].description }}
            </p>
          </div>
          <img :src="weatherIcon" :alt="weather.weather[0].description" class="w-16 h-16" />
        </div>

        <div class="divider"></div>

        <div class="text-center">
          <span class="text-5xl font-bold">{{ Math.round(weather.main.temp) }}°C</span>
          <p class="text-base-content/60">
            Feels like {{ Math.round(weather.main.feels_like) }}°C
          </p>
        </div>

        <div class="stats stats-vertical shadow mt-4">
          <div class="stat py-2">
            <div class="stat-title">Humidity</div>
            <div class="stat-value text-lg">{{ weather.main.humidity }}%</div>
          </div>
          <div class="stat py-2">
            <div class="stat-title">Wind Speed</div>
            <div class="stat-value text-lg">{{ weather.wind.speed }} m/s</div>
          </div>
        </div>

        <div class="card-actions justify-end mt-4">
          <button class="btn btn-sm btn-ghost" @click="refetch" :disabled="loading">
            ↻ Refresh
          </button>
        </div>
      </template>
    </div>
  </div>
</template>
```

**App.vue:**
```vue
<script setup lang="ts">
import { ref } from 'vue'
import WeatherCard from './components/WeatherCard.vue'

const cityInput = ref('')
const selectedCity = ref('London')
const apiKey = 'YOUR_OPENWEATHERMAP_API_KEY'

function searchCity() {
  if (cityInput.value.trim()) {
    selectedCity.value = cityInput.value.trim()
    cityInput.value = ''
  }
}

const quickCities = ['London', 'New York', 'Tokyo', 'Paris', 'Sydney']
</script>

<template>
  <div class="min-h-screen bg-gradient-to-br from-blue-400 to-purple-500 p-8">
    <div class="max-w-md mx-auto">
      <h1 class="text-3xl font-bold text-white text-center mb-6">
        Weather Dashboard
      </h1>

      <div class="join w-full mb-4">
        <input
          v-model="cityInput"
          type="text"
          placeholder="Enter city name..."
          class="input input-bordered join-item flex-1"
          @keyup.enter="searchCity"
        />
        <button class="btn btn-primary join-item" @click="searchCity">
          Search
        </button>
      </div>

      <div class="flex flex-wrap gap-2 justify-center mb-6">
        <button
          v-for="city in quickCities"
          :key="city"
          class="btn btn-sm"
          :class="selectedCity === city ? 'btn-primary' : 'btn-ghost bg-white/20'"
          @click="selectedCity = city"
        >
          {{ city }}
        </button>
      </div>

      <div class="flex justify-center">
        <WeatherCard :city="selectedCity" :api-key="apiKey" />
      </div>
    </div>
  </div>
</template>
```

**How to Test:**
1. Sign up for a free API key at [OpenWeatherMap](https://openweathermap.org/api)
2. Replace `YOUR_OPENWEATHERMAP_API_KEY` in `App.vue`
3. Run `npm run dev`
4. The app loads with London's weather by default
5. Click the quick city buttons to see the weather update automatically
6. Type a city name and press Enter to search
7. Click the refresh button to re-fetch the current city's weather
8. Try an invalid city name to see the error state

---

## 7. Implementation Exercises (Progressive)

### Level 1 (Recall): Fill in the Blanks

```typescript
import { ref, onMounted, type _____ } from 'vue'

export function useFetch<_____>(url: string) {
  const data = _____<T | null>(null) as Ref<T | null>
  const loading = ref(_____)
  const error = ref<_____ | null>(null)

  async function _____() {
    loading._____ = true
    try {
      const response = await _____(url)
      data.value = await response._____()
    } catch (e) {
      error.value = (e as Error).message
    } finally {
      loading.value = _____
    }
  }

  _____(execute)

  return { data, loading, error, refetch: execute }
}
```

**Answer:** `Ref`, `T`, `ref`, `false`, `string`, `execute`, `value`, `fetch`, `json`, `false`, `onMounted`

---

### Level 2 (Apply): Small Coding Tasks

**Exercise 2.1:** Create a `useFetch` variant called `useLazyFetch` that does NOT fetch automatically on mount. Instead, it only fetches when the returned `execute` function is called. Test it with a button that triggers the fetch.

**Exercise 2.2:** Extend your `useFetch` composable to accept an optional `transform` function that processes the data before storing it. For example:
```typescript
const { data } = useFetch<User[]>(url, {
  transform: (users) => users.filter(u => u.active)
})
```

---

### Level 3 (Debug): Fix the Broken Code

```typescript
// This composable has several bugs. Find and fix them.
import { ref, onMounted } from 'vue'

export function useFetch(url: string) {  // BUG 1: Missing generic type
  const data = ref(null)  // BUG 2: Missing type parameter
  const loading = ref(false)
  const error = ref(null)  // BUG 3: Missing type parameter

  async function execute() {
    loading = true  // BUG 4: Missing .value
    
    const response = await fetch(url)  // BUG 5: No error handling
    data.value = response.json()  // BUG 6: Missing await
    
    loading = false  // BUG 7: Missing .value
  }

  execute()  // BUG 8: Should be in onMounted

  return { data, loading, error }  // BUG 9: Missing refetch
}
```

**Corrected Code:**
```typescript
import { ref, onMounted, type Ref } from 'vue'

export function useFetch<T>(url: string) {
  const data = ref<T | null>(null) as Ref<T | null>
  const loading = ref(false)
  const error = ref<string | null>(null)

  async function execute() {
    loading.value = true
    error.value = null
    
    try {
      const response = await fetch(url)
      if (!response.ok) throw new Error(`HTTP ${response.status}`)
      data.value = await response.json()
    } catch (e) {
      error.value = e instanceof Error ? e.message : 'Unknown error'
    } finally {
      loading.value = false
    }
  }

  onMounted(execute)

  return { data, loading, error, refetch: execute }
}
```

---

## 8. Module 7 Checkpoint: What You've Learned

Congratulations on completing Module 7! You now have the skills to:

- ✅ Fetch data from REST APIs using `async/await`
- ✅ Handle loading and error states properly
- ✅ Use Vue lifecycle hooks (`onMounted`, `onUnmounted`) for data fetching
- ✅ Clean up pending requests with `AbortController`
- ✅ Build reusable composables with TypeScript generics
- ✅ Support both static and reactive URLs in composables
- ✅ Re-fetch data when dependencies change using `watch`

**Ready for the Module 7 Checkpoint Projects?** Try building:
1. **GitHub User Search** — Search and display GitHub profiles
2. **Weather Dashboard** — Fetch weather data with location switching  
3. **CRUD API Client** — Full Create, Read, Update, Delete operations

---

*Reply 'next' for Lesson 18.1 (Theory) — Routing I: Basic Navigation with Vue Router.*
