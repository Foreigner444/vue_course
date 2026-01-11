<script setup lang="ts">
/**
 * TemperatureConverter.vue - Temperature Converter Widget
 * 
 * This component demonstrates advanced Vue 3 reactivity patterns:
 * - reactive() for grouping related state
 * - Writable computed() for two-way derived values
 * - watch() for validation and side effects
 * 
 * Real-world scenario: Weather app temperature unit conversion
 */

import { reactive, ref, computed, watch } from 'vue'

// ============================================
// TYPE DEFINITIONS
// ============================================
interface TemperatureState {
  celsius: number
  lastEdited: 'celsius' | 'fahrenheit'
}

interface Preset {
  label: string
  celsius: number
  icon: string
}

// ============================================
// REACTIVE STATE with reactive()
// ============================================
// reactive() is ideal for objects with multiple related properties
// Unlike ref(), you access properties directly without .value

const temperatures = reactive<TemperatureState>({
  celsius: 20,
  lastEdited: 'celsius'
})

// ref() for simple UI state
const showKelvin = ref<boolean>(false)
const showWarning = ref<boolean>(false)
const warningMessage = ref<string>('')

// ============================================
// WRITABLE COMPUTED PROPERTIES
// ============================================
// Writable computed() allows v-model binding to derived values
// The getter reads, the setter writes back to the source

const celsiusModel = computed({
  get: (): number => {
    return temperatures.celsius
  },
  set: (value: number): void => {
    temperatures.celsius = value
    temperatures.lastEdited = 'celsius'
  }
})

const fahrenheitModel = computed({
  get: (): number => {
    // Convert Celsius to Fahrenheit: F = C × 9/5 + 32
    return Math.round((temperatures.celsius * 9/5 + 32) * 10) / 10
  },
  set: (value: number): void => {
    // Convert Fahrenheit to Celsius: C = (F - 32) × 5/9
    temperatures.celsius = Math.round(((value - 32) * 5/9) * 10) / 10
    temperatures.lastEdited = 'fahrenheit'
  }
})

// Read-only computed for Kelvin (no setter needed)
const kelvin = computed<number>(() => {
  // Kelvin = Celsius + 273.15
  return Math.round((temperatures.celsius + 273.15) * 10) / 10
})

// ============================================
// DERIVED STATE with computed()
// ============================================

const temperatureDescription = computed<string>(() => {
  const c = temperatures.celsius
  if (c <= -40) return 'Extreme Cold ❄️'
  if (c <= -20) return 'Severe Cold 🥶'
  if (c <= 0) return 'Freezing 🧊'
  if (c <= 10) return 'Cold 🌡️'
  if (c <= 20) return 'Cool 🍃'
  if (c <= 25) return 'Comfortable 😊'
  if (c <= 30) return 'Warm ☀️'
  if (c <= 35) return 'Hot 🔥'
  if (c <= 40) return 'Very Hot 🥵'
  return 'Extreme Heat ⚠️'
})

const temperatureColor = computed<string>(() => {
  const c = temperatures.celsius
  if (c <= 0) return 'text-info'
  if (c <= 15) return 'text-primary'
  if (c <= 25) return 'text-success'
  if (c <= 35) return 'text-warning'
  return 'text-error'
})

const gaugePercentage = computed<number>(() => {
  // Map temperature range -40 to 50 onto 0-100 scale
  const min = -40
  const max = 50
  const clamped = Math.max(min, Math.min(max, temperatures.celsius))
  return ((clamped - min) / (max - min)) * 100
})

const gaugeColor = computed<string>(() => {
  const c = temperatures.celsius
  if (c <= 0) return 'radial-progress text-info'
  if (c <= 20) return 'radial-progress text-success'
  if (c <= 30) return 'radial-progress text-warning'
  return 'radial-progress text-error'
})

const presets: Preset[] = [
  { label: 'Water Freezes', celsius: 0, icon: '🧊' },
  { label: 'Room Temp', celsius: 22, icon: '🏠' },
  { label: 'Body Temp', celsius: 37, icon: '🌡️' },
  { label: 'Water Boils', celsius: 100, icon: '♨️' },
]

// ============================================
// SIDE EFFECTS with watch()
// ============================================

// Watch for extreme temperatures
watch(() => temperatures.celsius, (newValue) => {
  if (newValue <= 0) {
    showWarning.value = true
    warningMessage.value = '⚠️ Freezing temperatures! Risk of ice and frost.'
  } else if (newValue >= 35) {
    showWarning.value = true
    warningMessage.value = '⚠️ High temperatures! Stay hydrated and avoid sun exposure.'
  } else {
    showWarning.value = false
    warningMessage.value = ''
  }
})

// Log temperature changes (simulating analytics)
watch(() => temperatures.celsius, (newValue, oldValue) => {
  const direction = newValue > oldValue ? 'increased' : 'decreased'
  console.log(`Temperature ${direction}: ${oldValue}°C → ${newValue}°C`)
}, { immediate: false })

// ============================================
// EVENT HANDLERS
// ============================================

const setPreset = (celsius: number): void => {
  temperatures.celsius = celsius
  temperatures.lastEdited = 'celsius'
}

const reset = (): void => {
  temperatures.celsius = 20
  temperatures.lastEdited = 'celsius'
}

const toggleKelvin = (): void => {
  showKelvin.value = !showKelvin.value
}
</script>

<template>
  <div class="card w-full max-w-md bg-base-100 shadow-xl">
    <div class="card-body">
      <!-- Header -->
      <h2 class="card-title justify-center text-2xl mb-2">
        🌡️ Temperature Converter
      </h2>

      <!-- Temperature Gauge -->
      <div class="flex justify-center my-4">
        <div 
          :class="gaugeColor"
          :style="`--value: ${gaugePercentage}; --size: 10rem; --thickness: 0.8rem;`"
          role="progressbar"
        >
          <div class="flex flex-col items-center">
            <span class="text-3xl font-bold" :class="temperatureColor">
              {{ temperatures.celsius }}°C
            </span>
            <span class="text-sm opacity-70">
              {{ fahrenheitModel }}°F
            </span>
          </div>
        </div>
      </div>

      <!-- Description -->
      <div class="text-center mb-4">
        <span class="badge badge-lg" :class="temperatureColor">
          {{ temperatureDescription }}
        </span>
      </div>

      <!-- Warning Alert -->
      <div 
        v-if="showWarning" 
        class="alert alert-warning py-2 mb-4"
      >
        <span class="text-sm">{{ warningMessage }}</span>
      </div>

      <!-- Input Fields -->
      <div class="grid grid-cols-2 gap-4">
        <!-- Celsius Input -->
        <div class="form-control">
          <label class="label">
            <span class="label-text font-medium">Celsius (°C)</span>
          </label>
          <input 
            v-model.number="celsiusModel"
            type="number"
            step="0.1"
            class="input input-bordered w-full text-center text-lg"
            :class="{ 
              'input-primary': temperatures.lastEdited === 'celsius',
              'input-info': temperatures.celsius <= 0
            }"
          />
        </div>

        <!-- Fahrenheit Input -->
        <div class="form-control">
          <label class="label">
            <span class="label-text font-medium">Fahrenheit (°F)</span>
          </label>
          <input 
            v-model.number="fahrenheitModel"
            type="number"
            step="0.1"
            class="input input-bordered w-full text-center text-lg"
            :class="{ 
              'input-primary': temperatures.lastEdited === 'fahrenheit'
            }"
          />
        </div>
      </div>

      <!-- Kelvin Display (Optional) -->
      <div class="collapse collapse-arrow bg-base-200 mt-4">
        <input 
          type="checkbox" 
          :checked="showKelvin"
          @change="toggleKelvin"
        />
        <div class="collapse-title text-sm font-medium">
          Show Kelvin (Scientific Unit)
        </div>
        <div class="collapse-content">
          <div class="stat py-2">
            <div class="stat-title">Kelvin</div>
            <div class="stat-value text-2xl">{{ kelvin }} K</div>
            <div class="stat-desc">Absolute temperature scale</div>
          </div>
        </div>
      </div>

      <!-- Preset Buttons -->
      <div class="divider text-xs">Quick Presets</div>
      
      <div class="grid grid-cols-2 gap-2">
        <button 
          v-for="preset in presets"
          :key="preset.label"
          class="btn btn-sm btn-outline"
          :class="{ 'btn-active': temperatures.celsius === preset.celsius }"
          @click="setPreset(preset.celsius)"
        >
          {{ preset.icon }} {{ preset.label }}
        </button>
      </div>

      <!-- Formula Reference -->
      <div class="mt-4 p-3 bg-base-200 rounded-lg text-xs font-mono">
        <div class="opacity-70">Formulas:</div>
        <div>°F = °C × 9/5 + 32</div>
        <div>°C = (°F - 32) × 5/9</div>
        <div>K = °C + 273.15</div>
      </div>

      <!-- Reset Button -->
      <div class="card-actions justify-center mt-4">
        <button 
          class="btn btn-ghost btn-sm"
          @click="reset"
        >
          Reset to 20°C
        </button>
      </div>
    </div>
  </div>
</template>
