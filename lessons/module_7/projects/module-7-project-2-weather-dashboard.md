# Module 7 Project 2: Weather Dashboard

## Project Overview

| Field | Value |
|:--- |:--- |
| **Module** | Module 7: Data Fetching & APIs |
| **Project** | Weather Dashboard |
| **Difficulty** | Intermediate |
| **Estimated Time** | 60-75 minutes |

**What You'll Build:** A weather application that fetches current weather conditions and a 5-day forecast from the OpenWeatherMap API. Users can search for any city and see temperature, humidity, wind speed, and weather conditions with dynamic icons.

**Real-World Scenario:** Weather applications are among the most common API-consuming apps. This project teaches you how to work with query parameters, transform API data, and build a reusable `useFetch` composable.

---

## Patterns You'll Practice

| Pattern | Description |
|:--- |:--- |
| `useFetch` composable | Reusable data fetching logic with generics |
| Query parameters | Building dynamic API URLs with parameters |
| Data transformation | Converting API responses to application-friendly formats |
| Reactive URLs | Re-fetching when computed URLs change |
| TypeScript generics | Type-safe composables that work with any data type |
| Computed properties | Deriving display values from raw data |
| Environment variables | Securely storing API keys |
| Date formatting | Converting timestamps to readable dates |

---

## Component Structure

```
src/
├── composables/
│   └── useFetch.ts            # Reusable fetch composable
├── components/
│   ├── SearchCity.vue         # City search input
│   ├── CurrentWeather.vue     # Current conditions display
│   ├── ForecastCard.vue       # Single day forecast card
│   ├── ForecastList.vue       # 5-day forecast container
│   └── WeatherIcon.vue        # Dynamic weather icon
├── types/
│   └── weather.ts             # TypeScript interfaces
├── utils/
│   └── weatherHelpers.ts      # Data transformation utilities
├── App.vue
└── main.ts
```

---

## Prerequisites

**Get a Free API Key:**
1. Sign up at [OpenWeatherMap](https://openweathermap.org/api)
2. Navigate to "API Keys" in your account
3. Copy your API key (it may take a few minutes to activate)

---

## Step-by-Step Guide

### Step 1: Project Setup

```bash
npm create vite@latest weather-dashboard -- --template vue-ts
cd weather-dashboard
npm install
npm install -D tailwindcss postcss autoprefixer daisyui
npx tailwindcss init -p
```

**Create .env file:**
```
VITE_OPENWEATHER_API_KEY=your_api_key_here
```

**vite-env.d.ts (update):**
```typescript
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_OPENWEATHER_API_KEY: string
}

interface ImportMeta {
  readonly env: ImportMetaEnv
}
```

---

### Step 2: Create the useFetch Composable

Build the reusable data fetching composable.

**src/composables/useFetch.ts:**
```typescript
import { ref, onMounted, onUnmounted, watch, isRef, unref, type Ref, type MaybeRef } from 'vue'

// TODO: Define UseFetchOptions interface with:
// - immediate?: boolean (default true)

// TODO: Define UseFetchReturn<T> interface with:
// - data: Ref<T | null>
// - loading: Ref<boolean>
// - error: Ref<string | null>
// - refetch: () => Promise<void>

// TODO: Implement useFetch<T>(url: MaybeRef<string>, options?: UseFetchOptions)
// Requirements:
// 1. Create refs for data, loading, error
// 2. Create AbortController for request cancellation
// 3. Implement execute() function with try/catch/finally
// 4. Auto-fetch on mount if immediate !== false
// 5. Watch url if it's a Ref and re-fetch on change
// 6. Abort pending request on unmount
// 7. Return { data, loading, error, refetch: execute }

export function useFetch<T>(
  url: MaybeRef<string>,
  options: UseFetchOptions = {}
): UseFetchReturn<T> {
  // YOUR CODE HERE
}
```

---

### Step 3: Define TypeScript Types

**src/types/weather.ts:**
```typescript
// Raw API response types (what OpenWeatherMap returns)
export interface OpenWeatherResponse {
  name: string
  main: {
    temp: number
    feels_like: number
    humidity: number
    pressure: number
    temp_min: number
    temp_max: number
  }
  weather: Array<{
    id: number
    main: string
    description: string
    icon: string
  }>
  wind: {
    speed: number
    deg: number
  }
  sys: {
    country: string
    sunrise: number
    sunset: number
  }
  dt: number
  timezone: number
}

export interface OpenWeatherForecastResponse {
  list: Array<{
    dt: number
    main: {
      temp: number
      temp_min: number
      temp_max: number
      humidity: number
    }
    weather: Array<{
      id: number
      main: string
      description: string
      icon: string
    }>
    wind: {
      speed: number
    }
    dt_txt: string
  }>
  city: {
    name: string
    country: string
  }
}

// TODO: Define transformed types for your application

// CurrentWeather - simplified version of OpenWeatherResponse
export interface CurrentWeather {
  // YOUR CODE HERE
}

// DayForecast - single day's forecast data
export interface DayForecast {
  // YOUR CODE HERE
}
```

---

### Step 4: Create Weather Helper Utilities

**src/utils/weatherHelpers.ts:**
```typescript
import type { OpenWeatherResponse, OpenWeatherForecastResponse, CurrentWeather, DayForecast } from '@/types/weather'

// TODO: Implement transformCurrentWeather(data: OpenWeatherResponse): CurrentWeather
// Transform the raw API response into a cleaner format

// TODO: Implement transformForecast(data: OpenWeatherForecastResponse): DayForecast[]
// Transform 5-day/3-hour forecast into daily forecasts (one per day)
// Hint: Group by date and pick the midday (12:00) forecast for each day

// TODO: Implement getWeatherIconUrl(iconCode: string): string
// Return the OpenWeatherMap icon URL for the given icon code
// Format: https://openweathermap.org/img/wn/${iconCode}@2x.png

// TODO: Implement formatTemperature(temp: number, unit?: 'C' | 'F'): string
// Format temperature with degree symbol (e.g., "23°C")
```

---

### Step 5: Create the Weather Icon Component

**src/components/WeatherIcon.vue:**
```vue
<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{
  iconCode: string
  description: string
  size?: 'sm' | 'md' | 'lg'
}>()

// TODO: Create computed property for icon URL
// TODO: Create computed property for size classes

</script>

<template>
  <!-- TODO: Display weather icon image with alt text -->

</template>
```

---

### Step 6: Create the Search City Component

**src/components/SearchCity.vue:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const emit = defineEmits<{
  search: [city: string]
}>()

const city = ref('')
const recentSearches = ref<string[]>([])

// TODO: Load recent searches from localStorage on mount

// TODO: Implement handleSearch() that:
// 1. Trims and validates the city name
// 2. Emits the search event
// 3. Adds to recent searches (max 5, no duplicates)
// 4. Saves to localStorage

// TODO: Implement selectRecent(city: string) to search a recent city

</script>

<template>
  <!-- TODO: Create search form with:
       - Text input for city name
       - Search button
       - Recent searches as clickable badges -->

</template>
```

---

### Step 7: Create the Current Weather Component

**src/components/CurrentWeather.vue:**
```vue
<script setup lang="ts">
import { computed } from 'vue'
import type { CurrentWeather } from '@/types/weather'
import { useFetch } from '@/composables/useFetch'
import { transformCurrentWeather, getWeatherIconUrl, formatTemperature } from '@/utils/weatherHelpers'
import WeatherIcon from './WeatherIcon.vue'

const props = defineProps<{
  city: string
}>()

const apiKey = import.meta.env.VITE_OPENWEATHER_API_KEY

// TODO: Create computed URL for current weather API
// Format: https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${apiKey}&units=metric

// TODO: Use useFetch with the computed URL

// TODO: Create computed property that transforms the raw data using transformCurrentWeather

</script>

<template>
  <!-- TODO: Display current weather with:
       - Loading spinner
       - Error state
       - City name and country
       - Weather icon and description
       - Current temperature (large)
       - Feels like temperature
       - High/Low temperatures
       - Humidity and wind speed
       - Sunrise/sunset times -->

</template>
```

---

### Step 8: Create the Forecast Card Component

**src/components/ForecastCard.vue:**
```vue
<script setup lang="ts">
import type { DayForecast } from '@/types/weather'
import WeatherIcon from './WeatherIcon.vue'
import { formatTemperature } from '@/utils/weatherHelpers'

defineProps<{
  forecast: DayForecast
}>()

// TODO: Create computed property to format the date (e.g., "Mon, Jan 15")

</script>

<template>
  <!-- TODO: Display forecast card with:
       - Day name
       - Weather icon
       - High/Low temperatures
       - Brief description -->

</template>
```

---

### Step 9: Create the Forecast List Component

**src/components/ForecastList.vue:**
```vue
<script setup lang="ts">
import { computed } from 'vue'
import type { OpenWeatherForecastResponse } from '@/types/weather'
import { useFetch } from '@/composables/useFetch'
import { transformForecast } from '@/utils/weatherHelpers'
import ForecastCard from './ForecastCard.vue'

const props = defineProps<{
  city: string
}>()

const apiKey = import.meta.env.VITE_OPENWEATHER_API_KEY

// TODO: Create computed URL for forecast API
// Format: https://api.openweathermap.org/data/2.5/forecast?q=${city}&appid=${apiKey}&units=metric

// TODO: Use useFetch with the computed URL

// TODO: Create computed property that transforms raw data to DayForecast[]

</script>

<template>
  <!-- TODO: Display 5-day forecast with:
       - Section header
       - Loading state
       - Error state
       - Horizontal scrollable list of ForecastCard components -->

</template>
```

---

### Step 10: Assemble the App

**src/App.vue:**
```vue
<script setup lang="ts">
import { ref } from 'vue'
import SearchCity from './components/SearchCity.vue'
import CurrentWeather from './components/CurrentWeather.vue'
import ForecastList from './components/ForecastList.vue'

// TODO: Create ref for selected city with default value (e.g., "London")

// TODO: Create handler for search event

</script>

<template>
  <!-- TODO: Create app layout with:
       - Gradient background
       - App title
       - SearchCity component
       - CurrentWeather component
       - ForecastList component -->

</template>
```

---

## Complete Solution

### src/composables/useFetch.ts
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
    const resolvedUrl = unref(url)
    if (!resolvedUrl) return

    if (abortController) {
      abortController.abort()
    }
    abortController = new AbortController()

    loading.value = true
    error.value = null

    try {
      const response = await fetch(resolvedUrl, {
        signal: abortController.signal
      })

      if (!response.ok) {
        const errorData = await response.json().catch(() => ({}))
        throw new Error(errorData.message || `HTTP error! Status: ${response.status}`)
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

### src/types/weather.ts
```typescript
export interface OpenWeatherResponse {
  name: string
  main: {
    temp: number
    feels_like: number
    humidity: number
    pressure: number
    temp_min: number
    temp_max: number
  }
  weather: Array<{
    id: number
    main: string
    description: string
    icon: string
  }>
  wind: {
    speed: number
    deg: number
  }
  sys: {
    country: string
    sunrise: number
    sunset: number
  }
  dt: number
  timezone: number
}

export interface OpenWeatherForecastResponse {
  list: Array<{
    dt: number
    main: {
      temp: number
      temp_min: number
      temp_max: number
      humidity: number
    }
    weather: Array<{
      id: number
      main: string
      description: string
      icon: string
    }>
    wind: {
      speed: number
    }
    dt_txt: string
  }>
  city: {
    name: string
    country: string
  }
}

export interface CurrentWeather {
  city: string
  country: string
  temperature: number
  feelsLike: number
  tempMin: number
  tempMax: number
  humidity: number
  windSpeed: number
  description: string
  icon: string
  sunrise: Date
  sunset: Date
}

export interface DayForecast {
  date: Date
  temperature: number
  tempMin: number
  tempMax: number
  humidity: number
  windSpeed: number
  description: string
  icon: string
}
```

### src/utils/weatherHelpers.ts
```typescript
import type { OpenWeatherResponse, OpenWeatherForecastResponse, CurrentWeather, DayForecast } from '@/types/weather'

export function transformCurrentWeather(data: OpenWeatherResponse): CurrentWeather {
  return {
    city: data.name,
    country: data.sys.country,
    temperature: Math.round(data.main.temp),
    feelsLike: Math.round(data.main.feels_like),
    tempMin: Math.round(data.main.temp_min),
    tempMax: Math.round(data.main.temp_max),
    humidity: data.main.humidity,
    windSpeed: Math.round(data.wind.speed * 3.6),
    description: data.weather[0].description,
    icon: data.weather[0].icon,
    sunrise: new Date(data.sys.sunrise * 1000),
    sunset: new Date(data.sys.sunset * 1000)
  }
}

export function transformForecast(data: OpenWeatherForecastResponse): DayForecast[] {
  const dailyMap = new Map<string, DayForecast>()

  data.list.forEach(item => {
    const date = new Date(item.dt * 1000)
    const dateKey = date.toISOString().split('T')[0]
    const hour = date.getHours()

    if (!dailyMap.has(dateKey) || (hour >= 11 && hour <= 14)) {
      dailyMap.set(dateKey, {
        date,
        temperature: Math.round(item.main.temp),
        tempMin: Math.round(item.main.temp_min),
        tempMax: Math.round(item.main.temp_max),
        humidity: item.main.humidity,
        windSpeed: Math.round(item.wind.speed * 3.6),
        description: item.weather[0].description,
        icon: item.weather[0].icon
      })
    }
  })

  return Array.from(dailyMap.values()).slice(0, 5)
}

export function getWeatherIconUrl(iconCode: string): string {
  return `https://openweathermap.org/img/wn/${iconCode}@2x.png`
}

export function formatTemperature(temp: number, unit: 'C' | 'F' = 'C'): string {
  if (unit === 'F') {
    return `${Math.round(temp * 9/5 + 32)}°F`
  }
  return `${Math.round(temp)}°C`
}

export function formatTime(date: Date): string {
  return date.toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit' })
}

export function formatDay(date: Date): string {
  return date.toLocaleDateString('en-US', { weekday: 'short', month: 'short', day: 'numeric' })
}
```

### src/components/WeatherIcon.vue
```vue
<script setup lang="ts">
import { computed } from 'vue'
import { getWeatherIconUrl } from '@/utils/weatherHelpers'

const props = withDefaults(defineProps<{
  iconCode: string
  description: string
  size?: 'sm' | 'md' | 'lg'
}>(), {
  size: 'md'
})

const iconUrl = computed(() => getWeatherIconUrl(props.iconCode))

const sizeClasses = computed(() => ({
  sm: 'w-10 h-10',
  md: 'w-16 h-16',
  lg: 'w-24 h-24'
}[props.size]))
</script>

<template>
  <img 
    :src="iconUrl" 
    :alt="description"
    :class="sizeClasses"
    class="drop-shadow-lg"
  />
</template>
```

### src/components/SearchCity.vue
```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'

const emit = defineEmits<{
  search: [city: string]
}>()

const city = ref('')
const recentSearches = ref<string[]>([])

const STORAGE_KEY = 'weather-recent-searches'

onMounted(() => {
  const stored = localStorage.getItem(STORAGE_KEY)
  if (stored) {
    recentSearches.value = JSON.parse(stored)
  }
})

function saveRecent() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(recentSearches.value))
}

function handleSearch() {
  const trimmed = city.value.trim()
  if (!trimmed) return

  emit('search', trimmed)

  const filtered = recentSearches.value.filter(
    s => s.toLowerCase() !== trimmed.toLowerCase()
  )
  recentSearches.value = [trimmed, ...filtered].slice(0, 5)
  saveRecent()

  city.value = ''
}

function selectRecent(selectedCity: string) {
  emit('search', selectedCity)
}

function clearRecent() {
  recentSearches.value = []
  localStorage.removeItem(STORAGE_KEY)
}
</script>

<template>
  <div class="w-full max-w-md">
    <form @submit.prevent="handleSearch" class="mb-4">
      <div class="join w-full">
        <input
          v-model="city"
          type="text"
          placeholder="Search city..."
          class="input input-bordered join-item flex-1"
        />
        <button type="submit" class="btn btn-primary join-item">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
          </svg>
        </button>
      </div>
    </form>

    <div v-if="recentSearches.length > 0" class="flex flex-wrap gap-2 items-center">
      <span class="text-sm text-base-content/60">Recent:</span>
      <button
        v-for="recent in recentSearches"
        :key="recent"
        @click="selectRecent(recent)"
        class="badge badge-outline badge-lg cursor-pointer hover:badge-primary transition-colors"
      >
        {{ recent }}
      </button>
      <button 
        @click="clearRecent" 
        class="btn btn-ghost btn-xs text-base-content/40"
      >
        Clear
      </button>
    </div>
  </div>
</template>
```

### src/components/CurrentWeather.vue
```vue
<script setup lang="ts">
import { computed } from 'vue'
import type { OpenWeatherResponse } from '@/types/weather'
import { useFetch } from '@/composables/useFetch'
import { transformCurrentWeather, formatTemperature, formatTime } from '@/utils/weatherHelpers'
import WeatherIcon from './WeatherIcon.vue'

const props = defineProps<{
  city: string
}>()

const apiKey = import.meta.env.VITE_OPENWEATHER_API_KEY

const apiUrl = computed(() => 
  `https://api.openweathermap.org/data/2.5/weather?q=${encodeURIComponent(props.city)}&appid=${apiKey}&units=metric`
)

const { data: rawData, loading, error, refetch } = useFetch<OpenWeatherResponse>(apiUrl)

const weather = computed(() => {
  if (!rawData.value) return null
  return transformCurrentWeather(rawData.value)
})
</script>

<template>
  <div class="card bg-base-100/80 backdrop-blur-sm shadow-xl w-full max-w-md">
    <div class="card-body">
      <div v-if="loading" class="flex justify-center py-12">
        <span class="loading loading-spinner loading-lg"></span>
      </div>

      <div v-else-if="error" class="alert alert-error">
        <svg xmlns="http://www.w3.org/2000/svg" class="stroke-current shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
        <div>
          <p>{{ error }}</p>
          <button class="btn btn-sm btn-ghost mt-2" @click="refetch">Try Again</button>
        </div>
      </div>

      <template v-else-if="weather">
        <div class="flex justify-between items-start">
          <div>
            <h2 class="text-2xl font-bold">{{ weather.city }}</h2>
            <p class="text-base-content/60">{{ weather.country }}</p>
          </div>
          <button @click="refetch" class="btn btn-ghost btn-sm btn-circle">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
            </svg>
          </button>
        </div>

        <div class="flex items-center justify-center gap-4 my-4">
          <WeatherIcon :icon-code="weather.icon" :description="weather.description" size="lg" />
          <div class="text-center">
            <p class="text-6xl font-bold">{{ formatTemperature(weather.temperature) }}</p>
            <p class="capitalize text-lg">{{ weather.description }}</p>
          </div>
        </div>

        <p class="text-center text-base-content/60 mb-4">
          Feels like {{ formatTemperature(weather.feelsLike) }}
        </p>

        <div class="grid grid-cols-2 gap-4">
          <div class="stat bg-base-200 rounded-lg p-3">
            <div class="stat-title text-xs">High / Low</div>
            <div class="stat-value text-lg">
              {{ formatTemperature(weather.tempMax) }} / {{ formatTemperature(weather.tempMin) }}
            </div>
          </div>
          <div class="stat bg-base-200 rounded-lg p-3">
            <div class="stat-title text-xs">Humidity</div>
            <div class="stat-value text-lg">{{ weather.humidity }}%</div>
          </div>
          <div class="stat bg-base-200 rounded-lg p-3">
            <div class="stat-title text-xs">Wind</div>
            <div class="stat-value text-lg">{{ weather.windSpeed }} km/h</div>
          </div>
          <div class="stat bg-base-200 rounded-lg p-3">
            <div class="stat-title text-xs">Sunrise / Sunset</div>
            <div class="stat-value text-sm">
              {{ formatTime(weather.sunrise) }} / {{ formatTime(weather.sunset) }}
            </div>
          </div>
        </div>
      </template>
    </div>
  </div>
</template>
```

### src/components/ForecastCard.vue
```vue
<script setup lang="ts">
import { computed } from 'vue'
import type { DayForecast } from '@/types/weather'
import WeatherIcon from './WeatherIcon.vue'
import { formatTemperature, formatDay } from '@/utils/weatherHelpers'

const props = defineProps<{
  forecast: DayForecast
}>()

const dayName = computed(() => formatDay(props.forecast.date))
</script>

<template>
  <div class="card bg-base-200 min-w-[120px]">
    <div class="card-body items-center p-4">
      <p class="font-semibold text-sm">{{ dayName }}</p>
      <WeatherIcon :icon-code="forecast.icon" :description="forecast.description" size="sm" />
      <p class="font-bold">{{ formatTemperature(forecast.tempMax) }}</p>
      <p class="text-base-content/60 text-sm">{{ formatTemperature(forecast.tempMin) }}</p>
    </div>
  </div>
</template>
```

### src/components/ForecastList.vue
```vue
<script setup lang="ts">
import { computed } from 'vue'
import type { OpenWeatherForecastResponse } from '@/types/weather'
import { useFetch } from '@/composables/useFetch'
import { transformForecast } from '@/utils/weatherHelpers'
import ForecastCard from './ForecastCard.vue'

const props = defineProps<{
  city: string
}>()

const apiKey = import.meta.env.VITE_OPENWEATHER_API_KEY

const apiUrl = computed(() =>
  `https://api.openweathermap.org/data/2.5/forecast?q=${encodeURIComponent(props.city)}&appid=${apiKey}&units=metric`
)

const { data: rawData, loading, error } = useFetch<OpenWeatherForecastResponse>(apiUrl)

const forecast = computed(() => {
  if (!rawData.value) return []
  return transformForecast(rawData.value)
})
</script>

<template>
  <div class="w-full max-w-md">
    <h3 class="text-lg font-semibold mb-4 text-white">5-Day Forecast</h3>
    
    <div v-if="loading" class="flex justify-center py-8">
      <span class="loading loading-spinner"></span>
    </div>

    <div v-else-if="error" class="alert alert-warning">
      <span>Could not load forecast</span>
    </div>

    <div v-else class="flex gap-3 overflow-x-auto pb-2">
      <ForecastCard
        v-for="day in forecast"
        :key="day.date.toISOString()"
        :forecast="day"
      />
    </div>
  </div>
</template>
```

### src/App.vue
```vue
<script setup lang="ts">
import { ref } from 'vue'
import SearchCity from './components/SearchCity.vue'
import CurrentWeather from './components/CurrentWeather.vue'
import ForecastList from './components/ForecastList.vue'

const selectedCity = ref('London')

function handleSearch(city: string) {
  selectedCity.value = city
}
</script>

<template>
  <div class="min-h-screen bg-gradient-to-br from-blue-500 via-purple-500 to-pink-500">
    <div class="container mx-auto px-4 py-8">
      <header class="text-center mb-8">
        <h1 class="text-4xl font-bold text-white mb-2">
          ☀️ Weather Dashboard
        </h1>
        <p class="text-white/80">Search for any city to see current weather and forecast</p>
      </header>

      <div class="flex flex-col items-center gap-8">
        <SearchCity @search="handleSearch" />
        <CurrentWeather :city="selectedCity" />
        <ForecastList :city="selectedCity" />
      </div>
    </div>
  </div>
</template>
```

### src/main.ts
```typescript
import { createApp } from 'vue'
import App from './App.vue'
import './style.css'

createApp(App).mount('#app')
```

---

## Stretch Goals

1. **Unit Toggle:** Add a button to switch between Celsius and Fahrenheit
2. **Geolocation:** Add "Use My Location" button using browser's Geolocation API
3. **Hourly Forecast:** Display hourly forecast for the current day
4. **Weather Alerts:** Fetch and display any active weather alerts for the location
5. **Animated Backgrounds:** Change the gradient background based on weather conditions
6. **Multiple Cities:** Allow users to save multiple cities and view them as cards

---

## Self-Assessment Checklist

- [ ] The `useFetch` composable is generic and reusable
- [ ] API key is stored in environment variable, not hardcoded
- [ ] Loading states display while fetching both current weather and forecast
- [ ] Error handling provides meaningful messages to users
- [ ] Data is properly transformed from API response to application types
- [ ] Recent searches are persisted to localStorage
- [ ] The UI updates reactively when searching for new cities
- [ ] Weather icons display correctly for all conditions
- [ ] Temperature and time values are formatted for readability

---

## Testing Your Implementation

1. **Default Load:** App should load with London weather displayed
2. **Search Valid City:** Search for "Tokyo" — should display Tokyo weather
3. **Search Invalid City:** Search for "asdfghjkl" — should show error message
4. **Recent Searches:** Search multiple cities — recent searches should appear and persist on reload
5. **Forecast Display:** 5-day forecast should show below current weather
6. **Refresh Button:** Click refresh icon — should re-fetch current data
7. **API Key Missing:** Remove API key — should show appropriate error
