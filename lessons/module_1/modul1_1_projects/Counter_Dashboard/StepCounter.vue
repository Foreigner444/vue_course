<script setup lang="ts">
/**
 * StepCounter.vue - Counter Dashboard Component
 * 
 * This component demonstrates core Vue 3 reactivity patterns:
 * - ref() for primitive reactive state
 * - computed() for derived values
 * - watch() for side effects
 * 
 * Real-world scenario: A fitness app step counter with progress tracking
 */

import { ref, computed, watch } from 'vue'

// ============================================
// REACTIVE STATE with ref()
// ============================================
// ref() wraps primitive values (numbers, strings, booleans) in a reactive container
// Access the value using .value in script, but directly in template

const stepCount = ref<number>(0)
const dailyGoal = ref<number>(10000)
const showCelebration = ref<boolean>(false)

// ============================================
// DERIVED STATE with computed()
// ============================================
// computed() creates values that automatically update when dependencies change
// They are cached and only recalculate when necessary

const progressPercentage = computed<number>(() => {
  // Calculate percentage, capped at 100%
  const percentage = (stepCount.value / dailyGoal.value) * 100
  return Math.min(percentage, 100)
})

const statusMessage = computed<string>(() => {
  const remaining = dailyGoal.value - stepCount.value
  
  if (remaining <= 0) {
    return '🎉 Goal achieved! Great job!'
  } else if (remaining <= 1000) {
    return `Almost there! Only ${remaining.toLocaleString()} steps to go!`
  } else if (remaining <= 5000) {
    return `Keep going! ${remaining.toLocaleString()} steps remaining.`
  } else {
    return `${remaining.toLocaleString()} steps to reach your goal.`
  }
})

const progressColor = computed<string>(() => {
  const pct = progressPercentage.value
  
  if (pct >= 100) return 'progress-success'
  if (pct >= 75) return 'progress-info'
  if (pct >= 50) return 'progress-warning'
  return 'progress-error'
})

const statsCards = computed(() => [
  { 
    label: 'Current Steps', 
    value: stepCount.value.toLocaleString(),
    icon: '👟'
  },
  { 
    label: 'Daily Goal', 
    value: dailyGoal.value.toLocaleString(),
    icon: '🎯'
  },
  { 
    label: 'Progress', 
    value: `${progressPercentage.value.toFixed(1)}%`,
    icon: '📊'
  },
])

// ============================================
// SIDE EFFECTS with watch()
// ============================================
// watch() runs a callback when reactive values change
// Use for side effects like API calls, logging, or animations

watch(progressPercentage, (newValue, oldValue) => {
  // Trigger celebration when goal is first reached
  if (newValue >= 100 && oldValue < 100) {
    showCelebration.value = true
    
    // Hide celebration after 3 seconds
    setTimeout(() => {
      showCelebration.value = false
    }, 3000)
  }
})

// ============================================
// EVENT HANDLERS
// ============================================
// Functions that modify reactive state

const increment = (amount: number = 100): void => {
  stepCount.value += amount
}

const decrement = (amount: number = 100): void => {
  stepCount.value = Math.max(0, stepCount.value - amount)
}

const reset = (): void => {
  stepCount.value = 0
}

const setGoal = (goal: number): void => {
  dailyGoal.value = goal
}
</script>

<template>
  <!-- Main Card Container -->
  <div class="card w-96 bg-base-100 shadow-xl">
    <div class="card-body">
      <!-- Header with celebration animation -->
      <h2 class="card-title justify-center text-2xl">
        <span>Step Counter</span>
        <span 
          v-if="showCelebration" 
          class="animate-bounce"
        >
          🎊
        </span>
      </h2>
      
      <!-- Stats Display Grid -->
      <div class="stats stats-vertical shadow mt-4">
        <div 
          v-for="stat in statsCards" 
          :key="stat.label"
          class="stat"
        >
          <div class="stat-figure text-2xl">{{ stat.icon }}</div>
          <div class="stat-title">{{ stat.label }}</div>
          <div class="stat-value text-primary">{{ stat.value }}</div>
        </div>
      </div>

      <!-- Progress Bar -->
      <div class="mt-4">
        <progress 
          class="progress w-full h-4"
          :class="progressColor"
          :value="progressPercentage" 
          max="100"
        ></progress>
      </div>

      <!-- Status Message -->
      <p class="text-center text-sm mt-2 font-medium">
        {{ statusMessage }}
      </p>

      <!-- Control Buttons -->
      <div class="card-actions justify-center mt-4">
        <div class="btn-group">
          <button 
            class="btn btn-outline btn-sm"
            @click="decrement(1000)"
          >
            -1000
          </button>
          <button 
            class="btn btn-outline btn-sm"
            @click="decrement(100)"
          >
            -100
          </button>
          <button 
            class="btn btn-primary btn-sm"
            @click="increment(100)"
          >
            +100
          </button>
          <button 
            class="btn btn-primary btn-sm"
            @click="increment(1000)"
          >
            +1000
          </button>
        </div>
      </div>

      <!-- Quick Add Buttons -->
      <div class="flex justify-center gap-2 mt-2">
        <button 
          class="btn btn-ghost btn-xs"
          @click="increment(2500)"
        >
          +2.5K Walk
        </button>
        <button 
          class="btn btn-ghost btn-xs"
          @click="increment(5000)"
        >
          +5K Run
        </button>
      </div>

      <!-- Reset and Goal Adjustment -->
      <div class="divider"></div>
      
      <div class="flex justify-between items-center">
        <button 
          class="btn btn-error btn-sm btn-outline"
          @click="reset"
        >
          Reset
        </button>
        
        <div class="dropdown dropdown-end">
          <label tabindex="0" class="btn btn-ghost btn-sm">
            Set Goal ▾
          </label>
          <ul 
            tabindex="0" 
            class="dropdown-content z-[1] menu p-2 shadow bg-base-100 rounded-box w-32"
          >
            <li><a @click="setGoal(5000)">5,000</a></li>
            <li><a @click="setGoal(7500)">7,500</a></li>
            <li><a @click="setGoal(10000)">10,000</a></li>
            <li><a @click="setGoal(15000)">15,000</a></li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>
