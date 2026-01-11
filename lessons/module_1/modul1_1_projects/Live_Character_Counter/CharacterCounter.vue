<script setup lang="ts">
/**
 * CharacterCounter.vue - Live Character Counter Component
 * 
 * This component demonstrates Vue 3 reactivity with user input:
 * - ref() for managing text state
 * - computed() for deriving character counts and validation state
 * - watch() for triggering side effects based on thresholds
 * - v-model for two-way data binding
 * 
 * Real-world scenario: Social media post composer with character limit
 */

import { ref, computed, watch } from 'vue'

// ============================================
// REACTIVE STATE with ref()
// ============================================
// The message text that the user is composing
const message = ref<string>('')

// Configurable character limit (like Twitter's 280)
const maxLength = ref<number>(280)

// UI state flags
const showWarning = ref<boolean>(false)
const lastSaved = ref<string>('')

// ============================================
// DERIVED STATE with computed()
// ============================================
// These values automatically update whenever `message` or `maxLength` changes

const characterCount = computed<number>(() => {
  return message.value.length
})

const remainingChars = computed<number>(() => {
  return maxLength.value - characterCount.value
})

const isOverLimit = computed<boolean>(() => {
  return remainingChars.value < 0
})

const isNearLimit = computed<boolean>(() => {
  // Warn when less than 20% of characters remain
  return remainingChars.value <= maxLength.value * 0.2 && remainingChars.value >= 0
})

const isEmpty = computed<boolean>(() => {
  return characterCount.value === 0
})

const progressPercentage = computed<number>(() => {
  const pct = (characterCount.value / maxLength.value) * 100
  return Math.min(pct, 100)
})

const counterColor = computed<string>(() => {
  if (isOverLimit.value) return 'text-error'
  if (isNearLimit.value) return 'text-warning'
  return 'text-base-content'
})

const progressBarColor = computed<string>(() => {
  if (isOverLimit.value) return 'progress-error'
  if (isNearLimit.value) return 'progress-warning'
  if (progressPercentage.value > 50) return 'progress-info'
  return 'progress-success'
})

const textareaClasses = computed<string>(() => {
  let base = 'textarea textarea-bordered w-full h-32 resize-none'
  if (isOverLimit.value) return `${base} textarea-error`
  if (isNearLimit.value) return `${base} textarea-warning`
  return base
})

const statusMessage = computed<string>(() => {
  if (isEmpty.value) return 'Start typing your message...'
  if (isOverLimit.value) return `⚠️ ${Math.abs(remainingChars.value)} characters over the limit!`
  if (isNearLimit.value) return `⚡ Only ${remainingChars.value} characters left!`
  return `✓ ${remainingChars.value} characters remaining`
})

const wordCount = computed<number>(() => {
  if (isEmpty.value) return 0
  return message.value.trim().split(/\s+/).filter(word => word.length > 0).length
})

// ============================================
// SIDE EFFECTS with watch()
// ============================================

// Watch for when user approaches the limit
watch(remainingChars, (newValue, oldValue) => {
  // Trigger warning animation when crossing the 20% threshold
  if (oldValue > maxLength.value * 0.2 && newValue <= maxLength.value * 0.2) {
    showWarning.value = true
    setTimeout(() => {
      showWarning.value = false
    }, 1000)
  }
})

// Auto-save draft (simulated)
watch(message, (newValue) => {
  // In a real app, this would save to localStorage or an API
  if (newValue.length > 0 && newValue.length % 50 === 0) {
    lastSaved.value = new Date().toLocaleTimeString()
    console.log('Draft auto-saved:', newValue.substring(0, 50) + '...')
  }
})

// ============================================
// EVENT HANDLERS
// ============================================

const clearMessage = (): void => {
  message.value = ''
}

const insertEmoji = (emoji: string): void => {
  message.value += emoji
}

const setLimit = (limit: number): void => {
  maxLength.value = limit
}

const handleSubmit = (): void => {
  if (!isEmpty.value && !isOverLimit.value) {
    alert(`Message posted: "${message.value}"`)
    clearMessage()
  }
}
</script>

<template>
  <div class="card w-full max-w-md bg-base-100 shadow-xl">
    <div class="card-body">
      <!-- Header -->
      <div class="flex justify-between items-center">
        <h2 class="card-title">
          📝 Compose Post
          <span 
            v-if="showWarning" 
            class="badge badge-warning animate-pulse"
          >
            Limit approaching!
          </span>
        </h2>
        
        <!-- Limit Selector -->
        <div class="dropdown dropdown-end">
          <label tabindex="0" class="btn btn-ghost btn-xs">
            {{ maxLength }} limit ▾
          </label>
          <ul 
            tabindex="0" 
            class="dropdown-content z-[1] menu p-2 shadow bg-base-100 rounded-box w-28"
          >
            <li><a @click="setLimit(140)">140 (SMS)</a></li>
            <li><a @click="setLimit(280)">280 (Tweet)</a></li>
            <li><a @click="setLimit(500)">500 (Post)</a></li>
            <li><a @click="setLimit(1000)">1000 (Long)</a></li>
          </ul>
        </div>
      </div>

      <!-- Textarea with v-model binding -->
      <div class="form-control mt-2">
        <textarea
          v-model="message"
          :class="textareaClasses"
          placeholder="What's on your mind?"
        ></textarea>
      </div>

      <!-- Progress Bar -->
      <progress 
        class="progress w-full h-2 mt-2"
        :class="progressBarColor"
        :value="progressPercentage" 
        max="100"
      ></progress>

      <!-- Character Count Display -->
      <div class="flex justify-between items-center mt-1">
        <span class="text-xs text-base-content/60">
          {{ wordCount }} word{{ wordCount !== 1 ? 's' : '' }}
        </span>
        <span 
          class="font-mono text-sm font-bold transition-colors"
          :class="counterColor"
        >
          {{ characterCount }} / {{ maxLength }}
        </span>
      </div>

      <!-- Status Message -->
      <div 
        class="alert py-2 mt-2"
        :class="{
          'alert-error': isOverLimit,
          'alert-warning': isNearLimit && !isOverLimit,
          'alert-info': !isNearLimit && !isOverLimit && !isEmpty,
          'bg-base-200': isEmpty
        }"
      >
        <span class="text-sm">{{ statusMessage }}</span>
      </div>

      <!-- Emoji Quick Insert -->
      <div class="flex gap-1 mt-2">
        <span class="text-xs text-base-content/60 self-center mr-1">Quick add:</span>
        <button 
          v-for="emoji in ['😊', '🎉', '❤️', '🔥', '👍', '✨']"
          :key="emoji"
          class="btn btn-ghost btn-xs"
          @click="insertEmoji(emoji)"
        >
          {{ emoji }}
        </button>
      </div>

      <!-- Action Buttons -->
      <div class="card-actions justify-between mt-4">
        <button 
          class="btn btn-ghost btn-sm"
          :disabled="isEmpty"
          @click="clearMessage"
        >
          Clear
        </button>
        
        <div class="flex gap-2 items-center">
          <span 
            v-if="lastSaved" 
            class="text-xs text-base-content/50"
          >
            Saved {{ lastSaved }}
          </span>
          <button 
            class="btn btn-primary btn-sm"
            :disabled="isEmpty || isOverLimit"
            @click="handleSubmit"
          >
            Post
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
