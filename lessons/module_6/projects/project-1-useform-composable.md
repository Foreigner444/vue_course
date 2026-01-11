# Guided Project 1: useForm Composable
---
## Project Overview
| Field | Value |
|:--- |:--- |
| **Module** | Module 6: Reusability & Abstraction |
| **Project** | useForm Composable |
| **Real-World Scenario** | Multiple forms in an app need shared validation, dirty tracking, and submission logic |
| **Difficulty** | Intermediate |
| **Estimated Time** | 45-60 minutes |
### What You'll Build
A reusable form composable that handles common form concerns: field registration, validation, dirty tracking (detecting unsaved changes), error management, and submission handling. You'll then use this composable in two different forms—a login form and a contact form—demonstrating its flexibility.
### Patterns You'll Practice
1. Composable creation with TypeScript generics
2. Reactive state management with `reactive()` and `ref()`
3. Computed properties for derived form state
4. Watchers for tracking changes
5. Validation pattern with error messages
6. Type-safe form field definitions
7. Async submission handling
8. Exposing a clean public API
---
## Component Structure
```
src/
├── composables/
│   └── useForm.ts           # The main form composable
├── components/
│   ├── LoginForm.vue        # Login form using the composable
│   └── ContactForm.vue      # Contact form using the composable
├── types/
│   └── form.ts              # TypeScript interfaces
├── App.vue                  # Demo page with both forms
└── main.ts
```
---
## Step-by-Step Guide
### Step 1: Project Setup
```bash
# Create a new Vite project with Vue and TypeScript
npm create vite@latest useform-demo -- --template vue-ts
cd useform-demo
# Install dependencies
npm install
# Install Tailwind CSS and DaisyUI
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
npm install -D daisyui@latest
```
Configure Tailwind (`tailwind.config.js`):
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
  plugins: [require("daisyui")],
}
```
Add Tailwind to `src/style.css`:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```
---
### Step 2: Define TypeScript Types
Create the type definitions for the form system:
```typescript
// src/types/form.ts
// ─────────────────────────────────────────────────────────────
// VALIDATION RULE: A function that validates a field value
// Returns null if valid, or an error message string if invalid
// ─────────────────────────────────────────────────────────────
export type ValidationRule<T> = (value: T) => string | null
// ─────────────────────────────────────────────────────────────
// FIELD CONFIG: Configuration for a single form field
// ─────────────────────────────────────────────────────────────
export interface FieldConfig<T> {
  initialValue: T
  rules?: ValidationRule<T>[]
}
// ─────────────────────────────────────────────────────────────
// FORM CONFIG: Configuration for all fields in a form
// Uses a mapped type to create field configs from form shape
// ─────────────────────────────────────────────────────────────
export type FormConfig<T extends Record<string, unknown>> = {
  [K in keyof T]: FieldConfig<T[K]>
}
// ─────────────────────────────────────────────────────────────
// FORM STATE: The current state of the form
// ─────────────────────────────────────────────────────────────
export interface FormState<T> {
  values: T
  errors: Partial<Record<keyof T, string>>
  touched: Partial<Record<keyof T, boolean>>
  isSubmitting: boolean
}
```
---
### Step 3: Build the useForm Composable
```typescript
// src/composables/useForm.ts
import { reactive, computed, watch, type ComputedRef } from 'vue'
import type { FormConfig, FormState, ValidationRule } from '@/types/form'
// ─────────────────────────────────────────────────────────────
// RETURN TYPE: What the composable exposes to components
// ─────────────────────────────────────────────────────────────
interface UseFormReturn<T extends Record<string, unknown>> {
  values: T
  errors: Partial<Record<keyof T, string>>
  touched: Partial<Record<keyof T, boolean>>
  isSubmitting: boolean
  isDirty: ComputedRef<boolean>
  isValid: ComputedRef<boolean>
  setFieldValue: <K extends keyof T>(field: K, value: T[K]) => void
  setFieldTouched: (field: keyof T) => void
  validateField: (field: keyof T) => boolean
  validateAll: () => boolean
  resetForm: () => void
  handleSubmit: (onSubmit: (values: T) => Promise<void> | void) => Promise<void>
}
// ─────────────────────────────────────────────────────────────
// MAIN COMPOSABLE: useForm
// Generic type T represents the shape of the form data
// ─────────────────────────────────────────────────────────────
export function useForm<T extends Record<string, unknown>>(
  config: FormConfig<T>
): UseFormReturn<T> {
  
  // ─────────────────────────────────────────────────────────────
  // EXTRACT INITIAL VALUES: Build initial state from config
  // ─────────────────────────────────────────────────────────────
  const initialValues = Object.keys(config).reduce((acc, key) => {
    const fieldKey = key as keyof T
    acc[fieldKey] = config[fieldKey].initialValue
    return acc
  }, {} as T)
  // ─────────────────────────────────────────────────────────────
  // STORE VALIDATION RULES: Keep rules accessible for validation
  // ─────────────────────────────────────────────────────────────
  const validationRules = Object.keys(config).reduce((acc, key) => {
    const fieldKey = key as keyof T
    acc[fieldKey] = config[fieldKey].rules || []
    return acc
  }, {} as Record<keyof T, ValidationRule<unknown>[]>)
  // ─────────────────────────────────────────────────────────────
  // REACTIVE STATE: The form's current state
  // Using reactive() for the entire state object
  // ─────────────────────────────────────────────────────────────
  const state = reactive<FormState<T>>({
    values: { ...initialValues },
    errors: {},
    touched: {},
    isSubmitting: false
  }) as FormState<T>
  // ─────────────────────────────────────────────────────────────
  // COMPUTED: isDirty - true if any value differs from initial
  // ─────────────────────────────────────────────────────────────
  const isDirty = computed(() => {
    return Object.keys(initialValues).some((key) => {
      const fieldKey = key as keyof T
      return state.values[fieldKey] !== initialValues[fieldKey]
    })
  })
  // ─────────────────────────────────────────────────────────────
  // COMPUTED: isValid - true if no errors exist
  // ─────────────────────────────────────────────────────────────
  const isValid = computed(() => {
    return Object.keys(state.errors).length === 0
  })
  // ─────────────────────────────────────────────────────────────
  // ACTION: Set a field's value
  // ─────────────────────────────────────────────────────────────
  function setFieldValue<K extends keyof T>(field: K, value: T[K]) {
    state.values[field] = value
  }
  // ─────────────────────────────────────────────────────────────
  // ACTION: Mark a field as touched (user has interacted with it)
  // ─────────────────────────────────────────────────────────────
  function setFieldTouched(field: keyof T) {
    state.touched[field] = true
  }
  // ─────────────────────────────────────────────────────────────
  // ACTION: Validate a single field
  // Returns true if valid, false if invalid
  // ─────────────────────────────────────────────────────────────
  function validateField(field: keyof T): boolean {
    const value = state.values[field]
    const rules = validationRules[field]
    
    for (const rule of rules) {
      const error = rule(value)
      if (error) {
        state.errors[field] = error
        return false
      }
    }
    
    delete state.errors[field]
    return true
  }
  // ─────────────────────────────────────────────────────────────
  // ACTION: Validate all fields
  // Returns true if all valid, false if any invalid
  // ─────────────────────────────────────────────────────────────
  function validateAll(): boolean {
    let allValid = true
    
    for (const field of Object.keys(config) as (keyof T)[]) {
      const fieldValid = validateField(field)
      if (!fieldValid) {
        allValid = false
      }
    }
    
    return allValid
  }
  // ─────────────────────────────────────────────────────────────
  // ACTION: Reset form to initial state
  // ─────────────────────────────────────────────────────────────
  function resetForm() {
    state.values = { ...initialValues }
    state.errors = {}
    state.touched = {}
    state.isSubmitting = false
  }
  // ─────────────────────────────────────────────────────────────
  // ACTION: Handle form submission
  // Validates all fields, then calls the provided submit function
  // ─────────────────────────────────────────────────────────────
  async function handleSubmit(
    onSubmit: (values: T) => Promise<void> | void
  ): Promise<void> {
    // Mark all fields as touched
    for (const field of Object.keys(config) as (keyof T)[]) {
      state.touched[field] = true
    }
    
    // Validate all fields
    const isFormValid = validateAll()
    
    if (!isFormValid) {
      return
    }
    
    // Submit the form
    state.isSubmitting = true
    
    try {
      await onSubmit(state.values)
    } finally {
      state.isSubmitting = false
    }
  }
  // ─────────────────────────────────────────────────────────────
  // WATCHER: Validate fields when they change (if touched)
  // This provides real-time validation feedback
  // ─────────────────────────────────────────────────────────────
  watch(
    () => state.values,
    () => {
      for (const field of Object.keys(state.touched) as (keyof T)[]) {
        if (state.touched[field]) {
          validateField(field)
        }
      }
    },
    { deep: true }
  )
  // ─────────────────────────────────────────────────────────────
  // RETURN: Public API
  // ─────────────────────────────────────────────────────────────
  return {
    values: state.values,
    errors: state.errors,
    touched: state.touched,
    isSubmitting: state.isSubmitting,
    isDirty,
    isValid,
    setFieldValue,
    setFieldTouched,
    validateField,
    validateAll,
    resetForm,
    handleSubmit
  }
}
```
---
### Step 4: Create Common Validation Rules
```typescript
// src/composables/validators.ts
import type { ValidationRule } from '@/types/form'
// ─────────────────────────────────────────────────────────────
// REQUIRED: Field must have a non-empty value
// ─────────────────────────────────────────────────────────────
export function required(message = 'This field is required'): ValidationRule<unknown> {
  return (value) => {
    if (value === null || value === undefined || value === '') {
      return message
    }
    return null
  }
}
// ─────────────────────────────────────────────────────────────
// MIN LENGTH: String must have minimum length
// ─────────────────────────────────────────────────────────────
export function minLength(min: number, message?: string): ValidationRule<string> {
  return (value) => {
    if (value.length < min) {
      return message || `Must be at least ${min} characters`
    }
    return null
  }
}
// ─────────────────────────────────────────────────────────────
// MAX LENGTH: String must not exceed maximum length
// ─────────────────────────────────────────────────────────────
export function maxLength(max: number, message?: string): ValidationRule<string> {
  return (value) => {
    if (value.length > max) {
      return message || `Must be no more than ${max} characters`
    }
    return null
  }
}
// ─────────────────────────────────────────────────────────────
// EMAIL: Must be a valid email format
// ─────────────────────────────────────────────────────────────
export function email(message = 'Invalid email address'): ValidationRule<string> {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  return (value) => {
    if (!emailRegex.test(value)) {
      return message
    }
    return null
  }
}
// ─────────────────────────────────────────────────────────────
// PATTERN: Must match a regular expression
// ─────────────────────────────────────────────────────────────
export function pattern(regex: RegExp, message: string): ValidationRule<string> {
  return (value) => {
    if (!regex.test(value)) {
      return message
    }
    return null
  }
}
```
---
### Step 5: Create the Login Form Component
```vue
<!-- src/components/LoginForm.vue -->
<script setup lang="ts">
import { useForm } from '@/composables/useForm'
import { required, email, minLength } from '@/composables/validators'
// ─────────────────────────────────────────────────────────────
// DEFINE FORM SHAPE: TypeScript interface for form data
// ─────────────────────────────────────────────────────────────
interface LoginFormData {
  email: string
  password: string
}
// ─────────────────────────────────────────────────────────────
// INITIALIZE FORM: Use the composable with config
// ─────────────────────────────────────────────────────────────
const {
  values,
  errors,
  touched,
  isSubmitting,
  isDirty,
  isValid,
  setFieldValue,
  setFieldTouched,
  handleSubmit,
  resetForm
} = useForm<LoginFormData>({
  email: {
    initialValue: '',
    rules: [required('Email is required'), email()]
  },
  password: {
    initialValue: '',
    rules: [required('Password is required'), minLength(8, 'Password must be at least 8 characters')]
  }
})
// ─────────────────────────────────────────────────────────────
// SUBMIT HANDLER: Called when form is valid and submitted
// ─────────────────────────────────────────────────────────────
async function onSubmit(data: LoginFormData) {
  // Simulate API call
  await new Promise(resolve => setTimeout(resolve, 1500))
  console.log('Login submitted:', data)
  alert(`Logged in as ${data.email}`)
  resetForm()
}
</script>
<template>
  <div class="card bg-base-200 shadow-xl">
    <div class="card-body">
      <h2 class="card-title">Login</h2>
      
      <!-- Form element - prevent default submit, use our handler -->
      <form @submit.prevent="handleSubmit(onSubmit)" class="space-y-4">
        
        <!-- Email Field -->
        <div class="form-control">
          <label class="label">
            <span class="label-text">Email</span>
          </label>
          <input
            type="email"
            class="input input-bordered"
            :class="{ 'input-error': touched.email && errors.email }"
            :value="values.email"
            @input="setFieldValue('email', ($event.target as HTMLInputElement).value)"
            @blur="setFieldTouched('email')"
            placeholder="you@example.com"
          />
          <!-- Error message -->
          <label v-if="touched.email && errors.email" class="label">
            <span class="label-text-alt text-error">{{ errors.email }}</span>
          </label>
        </div>
        
        <!-- Password Field -->
        <div class="form-control">
          <label class="label">
            <span class="label-text">Password</span>
          </label>
          <input
            type="password"
            class="input input-bordered"
            :class="{ 'input-error': touched.password && errors.password }"
            :value="values.password"
            @input="setFieldValue('password', ($event.target as HTMLInputElement).value)"
            @blur="setFieldTouched('password')"
            placeholder="••••••••"
          />
          <!-- Error message -->
          <label v-if="touched.password && errors.password" class="label">
            <span class="label-text-alt text-error">{{ errors.password }}</span>
          </label>
        </div>
        
        <!-- Form Status Indicators -->
        <div class="flex gap-2 text-sm">
          <span v-if="isDirty" class="badge badge-warning">Unsaved changes</span>
          <span v-if="isValid" class="badge badge-success">Valid</span>
        </div>
        
        <!-- Submit Button -->
        <div class="card-actions justify-end">
          <button
            type="button"
            class="btn btn-ghost"
            @click="resetForm"
            :disabled="!isDirty"
          >
            Reset
          </button>
          <button
            type="submit"
            class="btn btn-primary"
            :disabled="isSubmitting"
          >
            <span v-if="isSubmitting" class="loading loading-spinner loading-sm"></span>
            {{ isSubmitting ? 'Logging in...' : 'Login' }}
          </button>
        </div>
      </form>
    </div>
  </div>
</template>
```
---
### Step 6: Create the Contact Form Component
```vue
<!-- src/components/ContactForm.vue -->
<script setup lang="ts">
import { useForm } from '@/composables/useForm'
import { required, email, minLength, maxLength } from '@/composables/validators'
// ─────────────────────────────────────────────────────────────
// DEFINE FORM SHAPE: Different structure than login form
// ─────────────────────────────────────────────────────────────
interface ContactFormData {
  name: string
  email: string
  subject: string
  message: string
}
// ─────────────────────────────────────────────────────────────
// INITIALIZE FORM: Same composable, different config
// ─────────────────────────────────────────────────────────────
const {
  values,
  errors,
  touched,
  isSubmitting,
  isDirty,
  setFieldValue,
  setFieldTouched,
  handleSubmit,
  resetForm
} = useForm<ContactFormData>({
  name: {
    initialValue: '',
    rules: [required('Name is required'), minLength(2, 'Name must be at least 2 characters')]
  },
  email: {
    initialValue: '',
    rules: [required('Email is required'), email()]
  },
  subject: {
    initialValue: '',
    rules: [required('Subject is required'), maxLength(100, 'Subject too long')]
  },
  message: {
    initialValue: '',
    rules: [
      required('Message is required'),
      minLength(10, 'Message must be at least 10 characters'),
      maxLength(1000, 'Message must be under 1000 characters')
    ]
  }
})
// ─────────────────────────────────────────────────────────────
// SUBMIT HANDLER
// ─────────────────────────────────────────────────────────────
async function onSubmit(data: ContactFormData) {
  await new Promise(resolve => setTimeout(resolve, 1000))
  console.log('Contact form submitted:', data)
  alert('Message sent successfully!')
  resetForm()
}
</script>
<template>
  <div class="card bg-base-200 shadow-xl">
    <div class="card-body">
      <h2 class="card-title">Contact Us</h2>
      
      <form @submit.prevent="handleSubmit(onSubmit)" class="space-y-4">
        
        <!-- Name & Email Row -->
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <!-- Name Field -->
          <div class="form-control">
            <label class="label">
              <span class="label-text">Name</span>
            </label>
            <input
              type="text"
              class="input input-bordered"
              :class="{ 'input-error': touched.name && errors.name }"
              :value="values.name"
              @input="setFieldValue('name', ($event.target as HTMLInputElement).value)"
              @blur="setFieldTouched('name')"
              placeholder="John Doe"
            />
            <label v-if="touched.name && errors.name" class="label">
              <span class="label-text-alt text-error">{{ errors.name }}</span>
            </label>
          </div>
          
          <!-- Email Field -->
          <div class="form-control">
            <label class="label">
              <span class="label-text">Email</span>
            </label>
            <input
              type="email"
              class="input input-bordered"
              :class="{ 'input-error': touched.email && errors.email }"
              :value="values.email"
              @input="setFieldValue('email', ($event.target as HTMLInputElement).value)"
              @blur="setFieldTouched('email')"
              placeholder="you@example.com"
            />
            <label v-if="touched.email && errors.email" class="label">
              <span class="label-text-alt text-error">{{ errors.email }}</span>
            </label>
          </div>
        </div>
        
        <!-- Subject Field -->
        <div class="form-control">
          <label class="label">
            <span class="label-text">Subject</span>
          </label>
          <input
            type="text"
            class="input input-bordered"
            :class="{ 'input-error': touched.subject && errors.subject }"
            :value="values.subject"
            @input="setFieldValue('subject', ($event.target as HTMLInputElement).value)"
            @blur="setFieldTouched('subject')"
            placeholder="How can we help?"
          />
          <label v-if="touched.subject && errors.subject" class="label">
            <span class="label-text-alt text-error">{{ errors.subject }}</span>
          </label>
        </div>
        
        <!-- Message Field -->
        <div class="form-control">
          <label class="label">
            <span class="label-text">Message</span>
          </label>
          <textarea
            class="textarea textarea-bordered h-32"
            :class="{ 'textarea-error': touched.message && errors.message }"
            :value="values.message"
            @input="setFieldValue('message', ($event.target as HTMLTextAreaElement).value)"
            @blur="setFieldTouched('message')"
            placeholder="Tell us more..."
          ></textarea>
          <label class="label">
            <span 
              v-if="touched.message && errors.message" 
              class="label-text-alt text-error"
            >
              {{ errors.message }}
            </span>
            <span class="label-text-alt">
              {{ values.message.length }}/1000
            </span>
          </label>
        </div>
        
        <!-- Submit Button -->
        <div class="card-actions justify-end">
          <button
            type="button"
            class="btn btn-ghost"
            @click="resetForm"
            :disabled="!isDirty"
          >
            Clear
          </button>
          <button
            type="submit"
            class="btn btn-primary"
            :disabled="isSubmitting"
          >
            <span v-if="isSubmitting" class="loading loading-spinner loading-sm"></span>
            {{ isSubmitting ? 'Sending...' : 'Send Message' }}
          </button>
        </div>
      </form>
    </div>
  </div>
</template>
```
---
### Step 7: Create App.vue
```vue
<!-- src/App.vue -->
<script setup lang="ts">
import LoginForm from '@/components/LoginForm.vue'
import ContactForm from '@/components/ContactForm.vue'
</script>
<template>
  <div class="min-h-screen bg-base-100 py-8">
    <div class="container mx-auto px-4">
      <h1 class="text-3xl font-bold text-center mb-8">
        useForm Composable Demo
      </h1>
      
      <div class="grid grid-cols-1 lg:grid-cols-2 gap-8 max-w-5xl mx-auto">
        <LoginForm />
        <ContactForm />
      </div>
      
      <div class="mt-8 text-center text-sm text-base-content/60">
        <p>Both forms use the same <code class="bg-base-200 px-1 rounded">useForm</code> composable</p>
        <p>with different configurations and validation rules.</p>
      </div>
    </div>
  </div>
</template>
```
---
## Stretch Goals
1. **Add async validation:** Create a `uniqueEmail` validator that simulates checking if an email is already registered (returns a Promise)
2. **Add field-level helpers:** Create wrapper components like `<FormInput>` that automatically connect to the form state
3. **Add form persistence:** Use localStorage to save form state and restore it on page refresh
4. **Add confirmation dialog:** Show a "You have unsaved changes" dialog when `isDirty` is true and user tries to navigate away
---
## Complete Solution
The complete solution is provided in the step-by-step guide above. To run:
```bash
npm run dev
```
---
## Self-Assessment Checklist
- [ ] All components use `<script setup>` with TypeScript
- [ ] The `useForm` composable is fully typed with generics
- [ ] Validation runs on blur and re-validates on change
- [ ] `isDirty` correctly detects unsaved changes
- [ ] `isValid` updates based on validation state
- [ ] Form resets properly clear all state
- [ ] Submit handler only runs when form is valid
- [ ] Loading state is shown during submission
- [ ] Error messages appear only for touched fields
- [ ] Both forms work independently with the same composable
