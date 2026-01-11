# Project 1: Parent-Child Form
**Module:** Module 2: Component Communication  
**Patterns Practiced:** Props down, events up, `defineProps<T>()`, `withDefaults()`, `defineEmits<T>()`, prop validation, v-model pattern
---
## Scenario
A settings page has a parent container managing form state, with child input components that report changes upward.
## What You'll Build
A settings form with reusable `TextInput` and `ToggleSwitch` child components that receive values via props and emit changes to the parent.
---
## Component Structure
```
src/
├── components/
│   ├── TextInput.vue
│   ├── ToggleSwitch.vue
│   └── SettingsForm.vue
└── App.vue
```
---
## Step 1: TextInput Component
Create a reusable text input that receives its value via prop and emits changes.
```vue
<!-- components/TextInput.vue -->
<script setup lang="ts">
interface Props {
  label: string
  modelValue: string
  placeholder?: string
  type?: 'text' | 'email' | 'password'
  error?: string
  required?: boolean
}
const props = withDefaults(defineProps<Props>(), {
  placeholder: '',
  type: 'text',
  error: '',
  required: false
})
const emit = defineEmits<{
  'update:modelValue': [value: string]
  blur: []
  focus: []
}>()
function handleInput(event: Event) {
  const target = event.target as HTMLInputElement
  emit('update:modelValue', target.value)
}
</script>
<template>
  <div class="form-control w-full">
    <label class="label">
      <span class="label-text">
        {{ label }}
        <span v-if="required" class="text-error">*</span>
      </span>
    </label>
    <input
      :type="type"
      :value="modelValue"
      :placeholder="placeholder"
      class="input input-bordered w-full"
      :class="{ 'input-error': error }"
      @input="handleInput"
      @blur="emit('blur')"
      @focus="emit('focus')"
    />
    <label v-if="error" class="label">
      <span class="label-text-alt text-error">{{ error }}</span>
    </label>
  </div>
</template>
```
---
## Step 2: ToggleSwitch Component
Create a toggle switch that emits its new state when clicked.
```vue
<!-- components/ToggleSwitch.vue -->
<script setup lang="ts">
interface Props {
  label: string
  modelValue: boolean
  description?: string
  disabled?: boolean
}
const props = withDefaults(defineProps<Props>(), {
  description: '',
  disabled: false
})
const emit = defineEmits<{
  'update:modelValue': [value: boolean]
  change: [value: boolean]
}>()
function toggle() {
  if (props.disabled) return
  const newValue = !props.modelValue
  emit('update:modelValue', newValue)
  emit('change', newValue)
}
</script>
<template>
  <div 
    class="form-control cursor-pointer"
    :class="{ 'opacity-50 cursor-not-allowed': disabled }"
    @click="toggle"
  >
    <label class="label cursor-pointer justify-start gap-4">
      <input
        type="checkbox"
        class="toggle toggle-primary"
        :checked="modelValue"
        :disabled="disabled"
        @click.stop="toggle"
      />
      <div>
        <span class="label-text font-medium">{{ label }}</span>
        <p v-if="description" class="text-xs text-base-content/60">
          {{ description }}
        </p>
      </div>
    </label>
  </div>
</template>
```
---
## Step 3: SettingsForm Component
Parent component that manages all form state and handles child events.
```vue
<!-- components/SettingsForm.vue -->
<script setup lang="ts">
import { reactive, computed } from 'vue'
import TextInput from './TextInput.vue'
import ToggleSwitch from './ToggleSwitch.vue'
interface Settings {
  displayName: string
  email: string
  notifications: boolean
  darkMode: boolean
  publicProfile: boolean
}
interface ValidationErrors {
  displayName?: string
  email?: string
}
const emit = defineEmits<{
  save: [settings: Settings]
  cancel: []
}>()
const settings = reactive<Settings>({
  displayName: '',
  email: '',
  notifications: true,
  darkMode: false,
  publicProfile: false
})
const errors = reactive<ValidationErrors>({})
const touched = reactive({ displayName: false, email: false })
const isValid = computed(() => {
  return settings.displayName.length >= 2 && 
         settings.email.includes('@') &&
         !errors.displayName &&
         !errors.email
})
function validateField(field: keyof ValidationErrors) {
  if (field === 'displayName') {
    errors.displayName = settings.displayName.length < 2 
      ? 'Display name must be at least 2 characters' 
      : undefined
  }
  if (field === 'email') {
    errors.email = !settings.email.includes('@')
      ? 'Please enter a valid email address'
      : undefined
  }
}
function handleBlur(field: keyof ValidationErrors) {
  touched[field] = true
  validateField(field)
}
function handleToggleChange(setting: keyof Settings, value: boolean) {
  console.log(`Setting "${setting}" changed to:`, value)
}
function handleSave() {
  Object.keys(touched).forEach(key => {
    touched[key as keyof typeof touched] = true
    validateField(key as keyof ValidationErrors)
  })
  
  if (isValid.value) {
    emit('save', { ...settings })
  }
}
function handleCancel() {
  emit('cancel')
}
</script>
<template>
  <div class="card bg-base-100 shadow-xl">
    <div class="card-body">
      <h2 class="card-title">Account Settings</h2>
      
      <div class="divider">Profile</div>
      
      <TextInput
        v-model="settings.displayName"
        label="Display Name"
        placeholder="Enter your display name"
        :error="touched.displayName ? errors.displayName : ''"
        required
        @blur="handleBlur('displayName')"
      />
      
      <TextInput
        v-model="settings.email"
        label="Email Address"
        type="email"
        placeholder="you@example.com"
        :error="touched.email ? errors.email : ''"
        required
        @blur="handleBlur('email')"
      />
      
      <div class="divider">Preferences</div>
      
      <ToggleSwitch
        v-model="settings.notifications"
        label="Email Notifications"
        description="Receive updates about your account activity"
        @change="handleToggleChange('notifications', $event)"
      />
      
      <ToggleSwitch
        v-model="settings.darkMode"
        label="Dark Mode"
        description="Use dark theme across the application"
        @change="handleToggleChange('darkMode', $event)"
      />
      
      <ToggleSwitch
        v-model="settings.publicProfile"
        label="Public Profile"
        description="Allow others to see your profile information"
        @change="handleToggleChange('publicProfile', $event)"
      />
      
      <div class="card-actions justify-end mt-6">
        <button class="btn btn-ghost" @click="handleCancel">Cancel</button>
        <button 
          class="btn btn-primary" 
          :disabled="!isValid"
          @click="handleSave"
        >
          Save Changes
        </button>
      </div>
    </div>
  </div>
</template>
```
---
## Step 4: App Integration
```vue
<!-- App.vue -->
<script setup lang="ts">
import SettingsForm from './components/SettingsForm.vue'
interface Settings {
  displayName: string
  email: string
  notifications: boolean
  darkMode: boolean
  publicProfile: boolean
}
function handleSave(settings: Settings) {
  console.log('Saving settings:', settings)
  alert(`Settings saved!\n${JSON.stringify(settings, null, 2)}`)
}
function handleCancel() {
  console.log('Settings cancelled')
}
</script>
<template>
  <div class="container mx-auto p-8 max-w-md">
    <SettingsForm @save="handleSave" @cancel="handleCancel" />
  </div>
</template>
```
---
## Self-Assessment Checklist
- [ ] `TextInput` receives value via `modelValue` prop and emits `update:modelValue`
- [ ] `ToggleSwitch` emits both `update:modelValue` and `change` events
- [ ] Parent form validates fields on blur using emitted events
- [ ] All components use `<script setup>` with TypeScript
- [ ] Props have proper types and defaults
---
## Stretch Goals
- Add password field with show/hide toggle
- Implement form reset functionality
- Add "unsaved changes" warning on navigation
- Create a `SelectInput` component with the same patterns
