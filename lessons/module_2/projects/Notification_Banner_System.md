# Project 3: Notification Banner System
**Module:** Module 2: Component Communication  
**Patterns Practiced:** Props for message/type, emit on dismiss, dynamic prop binding, typed events with payloads
---
## Scenario
A dashboard displays dismissible alert banners. Child banner components notify the parent when dismissed so it can remove them from the list.
## What You'll Build
A `NotificationBanner` component and `NotificationContainer` parent that manages a list of notifications with auto-dismiss, actions, and animations.
---
## Component Structure
```
src/
├── components/
│   ├── NotificationBanner.vue
│   └── NotificationContainer.vue
└── App.vue
```
---
## Step 1: NotificationBanner Component
```vue
<!-- components/NotificationBanner.vue -->
<script setup lang="ts">
import { computed } from 'vue'
type NotificationType = 'info' | 'success' | 'warning' | 'error'
interface Props {
  id: string
  type: NotificationType
  title: string
  message?: string
  dismissible?: boolean
  autoDismiss?: number
}
const props = withDefaults(defineProps<Props>(), {
  message: '',
  dismissible: true,
  autoDismiss: 0
})
const emit = defineEmits<{
  dismiss: [id: string]
  action: [id: string, action: string]
}>()
const alertClass = computed(() => ({
  info: 'alert-info',
  success: 'alert-success',
  warning: 'alert-warning',
  error: 'alert-error'
})[props.type])
const iconPath = computed(() => ({
  info: 'M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z',
  success: 'M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z',
  warning: 'M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z',
  error: 'M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z'
})[props.type])
function handleDismiss() {
  emit('dismiss', props.id)
}
function handleAction(action: string) {
  emit('action', props.id, action)
}
if (props.autoDismiss > 0) {
  setTimeout(() => {
    emit('dismiss', props.id)
  }, props.autoDismiss)
}
</script>
<template>
  <div 
    class="alert shadow-lg"
    :class="alertClass"
    role="alert"
  >
    <svg 
      xmlns="http://www.w3.org/2000/svg" 
      class="stroke-current shrink-0 h-6 w-6" 
      fill="none" 
      viewBox="0 0 24 24"
    >
      <path 
        stroke-linecap="round" 
        stroke-linejoin="round" 
        stroke-width="2" 
        :d="iconPath"
      />
    </svg>
    
    <div class="flex-1">
      <h3 class="font-bold">{{ title }}</h3>
      <p v-if="message" class="text-sm">{{ message }}</p>
    </div>
    
    <div class="flex gap-2">
      <slot name="actions">
        <button 
          v-if="type === 'error'"
          class="btn btn-sm"
          @click="handleAction('retry')"
        >
          Retry
        </button>
      </slot>
      
      <button
        v-if="dismissible"
        class="btn btn-sm btn-ghost"
        aria-label="Dismiss"
        @click="handleDismiss"
      >
        ✕
      </button>
    </div>
  </div>
</template>
```
---
## Step 2: NotificationContainer Component
```vue
<!-- components/NotificationContainer.vue -->
<script setup lang="ts">
import { computed } from 'vue'
import NotificationBanner from './NotificationBanner.vue'
type NotificationType = 'info' | 'success' | 'warning' | 'error'
interface Notification {
  id: string
  type: NotificationType
  title: string
  message?: string
  dismissible?: boolean
  autoDismiss?: number
}
interface Props {
  notifications: Notification[]
  position?: 'top-right' | 'top-left' | 'bottom-right' | 'bottom-left' | 'top-center'
  maxVisible?: number
}
const props = withDefaults(defineProps<Props>(), {
  position: 'top-right',
  maxVisible: 5
})
const emit = defineEmits<{
  dismiss: [id: string]
  action: [id: string, action: string]
  clearAll: []
}>()
const visibleNotifications = computed(() => {
  return props.notifications.slice(0, props.maxVisible)
})
const hiddenCount = computed(() => {
  return Math.max(0, props.notifications.length - props.maxVisible)
})
const positionClasses = computed(() => ({
  'top-right': 'top-4 right-4',
  'top-left': 'top-4 left-4',
  'bottom-right': 'bottom-4 right-4',
  'bottom-left': 'bottom-4 left-4',
  'top-center': 'top-4 left-1/2 -translate-x-1/2'
})[props.position])
function handleDismiss(id: string) {
  emit('dismiss', id)
}
function handleAction(id: string, action: string) {
  emit('action', id, action)
}
function handleClearAll() {
  emit('clearAll')
}
</script>
<template>
  <div 
    class="fixed z-50 w-full max-w-sm space-y-2"
    :class="positionClasses"
  >
    <TransitionGroup name="notification">
      <NotificationBanner
        v-for="notification in visibleNotifications"
        :key="notification.id"
        :id="notification.id"
        :type="notification.type"
        :title="notification.title"
        :message="notification.message"
        :dismissible="notification.dismissible"
        :auto-dismiss="notification.autoDismiss"
        @dismiss="handleDismiss"
        @action="handleAction"
      />
    </TransitionGroup>
    
    <div 
      v-if="hiddenCount > 0 || notifications.length > 1" 
      class="flex justify-between items-center text-sm"
    >
      <span v-if="hiddenCount > 0" class="text-base-content/60">
        +{{ hiddenCount }} more notifications
      </span>
      <button 
        v-if="notifications.length > 1"
        class="btn btn-xs btn-ghost"
        @click="handleClearAll"
      >
        Clear all
      </button>
    </div>
  </div>
</template>
<style scoped>
.notification-enter-active,
.notification-leave-active {
  transition: all 0.3s ease;
}
.notification-enter-from {
  opacity: 0;
  transform: translateX(100%);
}
.notification-leave-to {
  opacity: 0;
  transform: translateX(100%);
}
</style>
```
---
## Step 3: App Integration
```vue
<!-- App.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import NotificationContainer from './components/NotificationContainer.vue'
type NotificationType = 'info' | 'success' | 'warning' | 'error'
interface Notification {
  id: string
  type: NotificationType
  title: string
  message?: string
  dismissible?: boolean
  autoDismiss?: number
}
const notifications = ref<Notification[]>([])
let notificationId = 0
function createNotification(
  type: NotificationType,
  title: string,
  message?: string,
  options: Partial<Notification> = {}
) {
  const id = `notification-${++notificationId}`
  notifications.value.push({
    id,
    type,
    title,
    message,
    dismissible: true,
    ...options
  })
}
function handleDismiss(id: string) {
  const index = notifications.value.findIndex(n => n.id === id)
  if (index !== -1) {
    notifications.value.splice(index, 1)
  }
}
function handleAction(id: string, action: string) {
  console.log(`Action "${action}" triggered for notification ${id}`)
  if (action === 'retry') {
    handleDismiss(id)
    createNotification('info', 'Retrying...', 'Please wait while we retry the operation.')
  }
}
function handleClearAll() {
  notifications.value = []
}
function addInfo() {
  createNotification(
    'info',
    'New Feature Available',
    'Check out the new dashboard redesign!'
  )
}
function addSuccess() {
  createNotification(
    'success',
    'Changes Saved',
    'Your profile has been updated successfully.',
    { autoDismiss: 5000 }
  )
}
function addWarning() {
  createNotification(
    'warning',
    'Session Expiring',
    'Your session will expire in 5 minutes.'
  )
}
function addError() {
  createNotification(
    'error',
    'Connection Failed',
    'Unable to connect to the server. Please try again.'
  )
}
</script>
<template>
  <div class="min-h-screen bg-base-200 p-8">
    <div class="max-w-md mx-auto">
      <h1 class="text-3xl font-bold mb-8">Notification System Demo</h1>
      
      <div class="card bg-base-100 shadow-xl">
        <div class="card-body">
          <h2 class="card-title">Trigger Notifications</h2>
          <p class="text-base-content/60">Click buttons to add notifications:</p>
          
          <div class="grid grid-cols-2 gap-2 mt-4">
            <button class="btn btn-info" @click="addInfo">
              Info
            </button>
            <button class="btn btn-success" @click="addSuccess">
              Success (auto-dismiss)
            </button>
            <button class="btn btn-warning" @click="addWarning">
              Warning
            </button>
            <button class="btn btn-error" @click="addError">
              Error
            </button>
          </div>
          
          <div class="divider"></div>
          
          <p class="text-sm text-base-content/60">
            Active notifications: {{ notifications.length }}
          </p>
        </div>
      </div>
    </div>
    
    <NotificationContainer
      :notifications="notifications"
      position="top-right"
      :max-visible="4"
      @dismiss="handleDismiss"
      @action="handleAction"
      @clear-all="handleClearAll"
    />
  </div>
</template>
```
---
## Self-Assessment Checklist
- [ ] `NotificationBanner` receives all display data via props
- [ ] `NotificationBanner` emits `dismiss` with its ID when closed
- [ ] `NotificationBanner` emits `action` for interactive elements
- [ ] `NotificationContainer` manages visibility and passes props to children
- [ ] Parent correctly removes notifications when `dismiss` event received
- [ ] Auto-dismiss feature works using `setTimeout` and emitting `dismiss`
- [ ] Animations work smoothly with `TransitionGroup`
---
## Stretch Goals
- Add sound effects for different notification types
- Implement notification grouping (stack similar notifications)
- Add "undo" action for dismissed notifications (with timer)
- Persist notifications to localStorage
- Add progress bar for auto-dismiss countdown
- Support custom action buttons via slots
