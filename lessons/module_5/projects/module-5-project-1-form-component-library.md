# Guided Project: Custom Form Component Library

---

## Project Overview

| Field | Value |
|:--- |:--- |
| **Module** | Module 5: User Interaction |
| **Project Number** | 1 of 3 |
| **Difficulty** | Intermediate |
| **Estimated Time** | 45-60 minutes |

**What You'll Build:** A reusable form component library featuring custom Input, Select, and Checkbox components that support `v-model`, validation states, and flexible labeling through slots. This mirrors real-world design system development.

**Real-World Scenario:** Your company's design team has created a unified look for all form elements. Instead of styling native inputs repeatedly across the application, you'll build a component library that enforces consistent styling, handles validation states, and provides a clean API for developers.

---

## Patterns You'll Practice

| Pattern | How It's Used |
|:--- |:--- |
| `defineModel()` | Two-way binding for all form components |
| Named Slots | Flexible label and helper text areas |
| Slot Props | Passing validation state to slot content |
| Event Modifiers | `.trim`, `.number` support in custom components |
| `defineProps` with TypeScript | Strongly typed component APIs |
| `computed()` | Derived validation states |
| Conditional Styling | Dynamic classes based on state |
| `useSlots()` | Conditional slot rendering |

---

## Component Structure

```
src/
├── components/
│   └── form/
│       ├── FormInput.vue
│       ├── FormSelect.vue
│       ├── FormCheckbox.vue
│       └── FormGroup.vue
├── App.vue
└── main.ts
```

---

## Step-by-Step Guide

### Step 1: Project Setup

Create a new Vue project with TypeScript and Tailwind CSS:

```bash
npm create vue@latest form-component-library
# ✔ TypeScript: Yes
# ✔ JSX: No
# ✔ Vue Router: No
# ✔ Pinia: No
# ✔ Vitest: No
# ✔ ESLint: Yes
# ✔ Prettier: Yes

cd form-component-library
npm install
npm install -D tailwindcss postcss autoprefixer daisyui
npx tailwindcss init -p
```

Configure `tailwind.config.js`:
```js
/** @type {import('tailwindcss').Config} */
export default {
  content: ['./index.html', './src/**/*.{vue,js,ts,jsx,tsx}'],
  theme: {
    extend: {},
  },
  plugins: [require('daisyui')],
  daisyui: {
    themes: ['light', 'dark'],
  },
}
```

Add Tailwind to `src/style.css`:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

---

### Step 2: Create FormGroup Component

The `FormGroup` component wraps form elements with consistent spacing and label handling.

**File: `src/components/form/FormGroup.vue`**

```vue
<script setup lang="ts">
import { useSlots, computed } from 'vue'

interface Props {
  label?: string
  htmlFor?: string
  error?: string
  hint?: string
  required?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  label: '',
  htmlFor: '',
  error: '',
  hint: '',
  required: false
})

const slots = useSlots()

const hasLabel = computed(() => props.label || slots.label)
const hasHint = computed(() => props.hint || slots.hint)
const hasError = computed(() => !!props.error)
</script>

<template>
  <div class="form-control w-full">
    <!-- Label Area -->
    <label 
      v-if="hasLabel" 
      :for="htmlFor"
      class="label"
    >
      <span class="label-text">
        <slot name="label">
          {{ label }}
          <span v-if="required" class="text-error ml-1">*</span>
        </slot>
      </span>
    </label>
    
    <!-- Main Content (Form Element) -->
    <slot :has-error="hasError"></slot>
    
    <!-- Helper Text / Error Area -->
    <label v-if="hasHint || hasError" class="label">
      <span 
        class="label-text-alt"
        :class="hasError ? 'text-error' : 'text-base-content/70'"
      >
        <slot name="hint">
          {{ error || hint }}
        </slot>
      </span>
    </label>
  </div>
</template>
```

---

### Step 3: Create FormInput Component

The main text input component with full `v-model` support and validation states.

**File: `src/components/form/FormInput.vue`**

```vue
<script setup lang="ts">
import { computed, useSlots } from 'vue'

interface Props {
  type?: 'text' | 'email' | 'password' | 'number' | 'tel' | 'url' | 'search'
  placeholder?: string
  disabled?: boolean
  readonly?: boolean
  error?: string
  size?: 'xs' | 'sm' | 'md' | 'lg'
  variant?: 'bordered' | 'ghost'
}

const props = withDefaults(defineProps<Props>(), {
  type: 'text',
  placeholder: '',
  disabled: false,
  readonly: false,
  error: '',
  size: 'md',
  variant: 'bordered'
})

const model = defineModel<string | number>({ default: '' })

const slots = useSlots()

const hasError = computed(() => !!props.error)
const hasPrefix = computed(() => !!slots.prefix)
const hasSuffix = computed(() => !!slots.suffix)

const inputClasses = computed(() => {
  const classes = ['input', 'w-full']
  
  if (props.variant === 'bordered') classes.push('input-bordered')
  if (props.variant === 'ghost') classes.push('input-ghost')
  
  if (props.size === 'xs') classes.push('input-xs')
  if (props.size === 'sm') classes.push('input-sm')
  if (props.size === 'lg') classes.push('input-lg')
  
  if (hasError.value) classes.push('input-error')
  
  return classes
})
</script>

<template>
  <div class="relative flex items-center">
    <!-- Prefix Slot -->
    <div 
      v-if="hasPrefix" 
      class="absolute left-3 flex items-center pointer-events-none text-base-content/50"
    >
      <slot name="prefix"></slot>
    </div>
    
    <!-- Input Element -->
    <input
      v-model="model"
      :type="type"
      :placeholder="placeholder"
      :disabled="disabled"
      :readonly="readonly"
      :class="[
        ...inputClasses,
        { 'pl-10': hasPrefix, 'pr-10': hasSuffix }
      ]"
    />
    
    <!-- Suffix Slot -->
    <div 
      v-if="hasSuffix" 
      class="absolute right-3 flex items-center text-base-content/50"
    >
      <slot name="suffix"></slot>
    </div>
  </div>
</template>
```

---

### Step 4: Create FormSelect Component

A custom select dropdown with `v-model` support.

**File: `src/components/form/FormSelect.vue`**

```vue
<script setup lang="ts">
import { computed } from 'vue'

interface Option {
  value: string | number
  label: string
  disabled?: boolean
}

interface Props {
  options: Option[]
  placeholder?: string
  disabled?: boolean
  error?: string
  size?: 'xs' | 'sm' | 'md' | 'lg'
}

const props = withDefaults(defineProps<Props>(), {
  options: () => [],
  placeholder: 'Select an option',
  disabled: false,
  error: '',
  size: 'md'
})

const model = defineModel<string | number | null>({ default: null })

const hasError = computed(() => !!props.error)

const selectClasses = computed(() => {
  const classes = ['select', 'select-bordered', 'w-full']
  
  if (props.size === 'xs') classes.push('select-xs')
  if (props.size === 'sm') classes.push('select-sm')
  if (props.size === 'lg') classes.push('select-lg')
  
  if (hasError.value) classes.push('select-error')
  
  return classes
})
</script>

<template>
  <select
    v-model="model"
    :disabled="disabled"
    :class="selectClasses"
  >
    <option 
      v-if="placeholder" 
      :value="null" 
      disabled
    >
      {{ placeholder }}
    </option>
    
    <option
      v-for="option in options"
      :key="option.value"
      :value="option.value"
      :disabled="option.disabled"
    >
      {{ option.label }}
    </option>
  </select>
</template>
```

---

### Step 5: Create FormCheckbox Component

A styled checkbox with label slot support.

**File: `src/components/form/FormCheckbox.vue`**

```vue
<script setup lang="ts">
import { computed, useSlots } from 'vue'

interface Props {
  label?: string
  disabled?: boolean
  error?: string
  size?: 'xs' | 'sm' | 'md' | 'lg'
  color?: 'primary' | 'secondary' | 'accent' | 'success' | 'warning' | 'error'
  indeterminate?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  label: '',
  disabled: false,
  error: '',
  size: 'md',
  color: 'primary',
  indeterminate: false
})

const model = defineModel<boolean>({ default: false })

const slots = useSlots()

const hasLabel = computed(() => props.label || slots.default)

const checkboxClasses = computed(() => {
  const classes = ['checkbox']
  
  if (props.size === 'xs') classes.push('checkbox-xs')
  if (props.size === 'sm') classes.push('checkbox-sm')
  if (props.size === 'lg') classes.push('checkbox-lg')
  
  classes.push(`checkbox-${props.color}`)
  
  return classes
})
</script>

<template>
  <label 
    class="flex items-center gap-3 cursor-pointer"
    :class="{ 'opacity-50 cursor-not-allowed': disabled }"
  >
    <input
      v-model="model"
      type="checkbox"
      :disabled="disabled"
      :indeterminate="indeterminate"
      :class="checkboxClasses"
    />
    
    <span v-if="hasLabel" class="label-text">
      <slot>{{ label }}</slot>
    </span>
  </label>
</template>
```

---

### Step 6: Create Demo Application

**File: `src/App.vue`**

```vue
<script setup lang="ts">
import { ref, computed, reactive } from 'vue'
import FormGroup from './components/form/FormGroup.vue'
import FormInput from './components/form/FormInput.vue'
import FormSelect from './components/form/FormSelect.vue'
import FormCheckbox from './components/form/FormCheckbox.vue'

interface FormData {
  username: string
  email: string
  password: string
  country: string | null
  agreeToTerms: boolean
  newsletter: boolean
}

const form = reactive<FormData>({
  username: '',
  email: '',
  password: '',
  country: null,
  agreeToTerms: false,
  newsletter: true
})

const countries = [
  { value: 'us', label: 'United States' },
  { value: 'uk', label: 'United Kingdom' },
  { value: 'ca', label: 'Canada' },
  { value: 'au', label: 'Australia' },
  { value: 'de', label: 'Germany' },
]

const showPassword = ref(false)

const errors = reactive({
  username: '',
  email: '',
  password: '',
  country: '',
  agreeToTerms: ''
})

const isUsernameValid = computed(() => form.username.length >= 3)
const isEmailValid = computed(() => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.email))
const isPasswordValid = computed(() => form.password.length >= 8)
const isFormValid = computed(() => 
  isUsernameValid.value && 
  isEmailValid.value && 
  isPasswordValid.value && 
  form.country !== null &&
  form.agreeToTerms
)

function validateField(field: keyof typeof errors): void {
  switch (field) {
    case 'username':
      errors.username = form.username && !isUsernameValid.value 
        ? 'Username must be at least 3 characters' 
        : ''
      break
    case 'email':
      errors.email = form.email && !isEmailValid.value 
        ? 'Please enter a valid email address' 
        : ''
      break
    case 'password':
      errors.password = form.password && !isPasswordValid.value 
        ? 'Password must be at least 8 characters' 
        : ''
      break
    case 'country':
      errors.country = form.country === null 
        ? 'Please select a country' 
        : ''
      break
    case 'agreeToTerms':
      errors.agreeToTerms = !form.agreeToTerms 
        ? 'You must agree to the terms' 
        : ''
      break
  }
}

function handleSubmit(): void {
  Object.keys(errors).forEach(field => validateField(field as keyof typeof errors))
  
  if (isFormValid.value) {
    alert('Form submitted successfully!\n\n' + JSON.stringify(form, null, 2))
  }
}

function resetForm(): void {
  form.username = ''
  form.email = ''
  form.password = ''
  form.country = null
  form.agreeToTerms = false
  form.newsletter = true
  
  Object.keys(errors).forEach(key => {
    errors[key as keyof typeof errors] = ''
  })
}
</script>

<template>
  <div class="min-h-screen bg-base-200 py-8">
    <div class="max-w-md mx-auto">
      <div class="card bg-base-100 shadow-xl">
        <div class="card-body">
          <h1 class="card-title text-2xl mb-6">Create Account</h1>
          
          <form @submit.prevent="handleSubmit" class="space-y-4">
            <!-- Username Field -->
            <FormGroup
              label="Username"
              :error="errors.username"
              hint="Choose a unique username"
              required
            >
              <FormInput
                v-model="form.username"
                placeholder="johndoe"
                :error="errors.username"
                @blur="validateField('username')"
              >
                <template #prefix>
                  <span class="text-lg">@</span>
                </template>
              </FormInput>
            </FormGroup>
            
            <!-- Email Field -->
            <FormGroup
              label="Email Address"
              :error="errors.email"
              required
            >
              <FormInput
                v-model="form.email"
                type="email"
                placeholder="john@example.com"
                :error="errors.email"
                @blur="validateField('email')"
              >
                <template #prefix>
                  <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 8l7.89 5.26a2 2 0 002.22 0L21 8M5 19h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z" />
                  </svg>
                </template>
              </FormInput>
            </FormGroup>
            
            <!-- Password Field -->
            <FormGroup
              label="Password"
              :error="errors.password"
              hint="At least 8 characters"
              required
            >
              <FormInput
                v-model="form.password"
                :type="showPassword ? 'text' : 'password'"
                placeholder="••••••••"
                :error="errors.password"
                @blur="validateField('password')"
              >
                <template #prefix>
                  <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z" />
                  </svg>
                </template>
                <template #suffix>
                  <button 
                    type="button"
                    class="btn btn-ghost btn-xs btn-circle"
                    @click="showPassword = !showPassword"
                  >
                    <svg v-if="showPassword" class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13.875 18.825A10.05 10.05 0 0112 19c-4.478 0-8.268-2.943-9.543-7a9.97 9.97 0 011.563-3.029m5.858.908a3 3 0 114.243 4.243M9.878 9.878l4.242 4.242M9.88 9.88l-3.29-3.29m7.532 7.532l3.29 3.29M3 3l3.59 3.59m0 0A9.953 9.953 0 0112 5c4.478 0 8.268 2.943 9.543 7a10.025 10.025 0 01-4.132 5.411m0 0L21 21" />
                    </svg>
                    <svg v-else class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z" />
                    </svg>
                  </button>
                </template>
              </FormInput>
            </FormGroup>
            
            <!-- Country Select -->
            <FormGroup
              label="Country"
              :error="errors.country"
              required
            >
              <FormSelect
                v-model="form.country"
                :options="countries"
                placeholder="Select your country"
                :error="errors.country"
                @change="validateField('country')"
              />
            </FormGroup>
            
            <!-- Checkboxes -->
            <div class="space-y-3 pt-2">
              <FormCheckbox
                v-model="form.agreeToTerms"
                color="primary"
                @change="validateField('agreeToTerms')"
              >
                I agree to the 
                <a href="#" class="link link-primary">Terms of Service</a>
                and
                <a href="#" class="link link-primary">Privacy Policy</a>
              </FormCheckbox>
              
              <p v-if="errors.agreeToTerms" class="text-error text-sm ml-8">
                {{ errors.agreeToTerms }}
              </p>
              
              <FormCheckbox
                v-model="form.newsletter"
                color="secondary"
                label="Send me product updates and newsletters"
              />
            </div>
            
            <!-- Form Actions -->
            <div class="flex gap-3 pt-4">
              <button
                type="submit"
                class="btn btn-primary flex-1"
                :disabled="!isFormValid"
              >
                Create Account
              </button>
              <button
                type="button"
                class="btn btn-ghost"
                @click="resetForm"
              >
                Reset
              </button>
            </div>
          </form>
          
          <!-- Debug Panel -->
          <details class="mt-6">
            <summary class="cursor-pointer text-sm text-base-content/50">
              Debug: Form State
            </summary>
            <pre class="mt-2 p-3 bg-base-200 rounded text-xs overflow-auto">{{ { form, errors, isFormValid } }}</pre>
          </details>
        </div>
      </div>
    </div>
  </div>
</template>
```

---

## How to Run

```bash
npm run dev
```

Open `http://localhost:5173` in your browser.

---

## Stretch Goals

1. **Add FormTextarea Component:** Create a textarea variant with character count display
2. **Add FormRadioGroup Component:** Create a radio button group with horizontal/vertical layout options
3. **Add Form-Level Validation:** Implement a composable `useFormValidation` that handles all validation logic
4. **Add Loading State:** Add a loading spinner to inputs during async validation
5. **Add Animations:** Add transition animations for error message appearance

---

## Self-Assessment Checklist

- [ ] All components use `<script setup>` with TypeScript
- [ ] `defineModel()` is used correctly for two-way binding
- [ ] Slots are used for flexible content (labels, icons, custom content)
- [ ] Props are properly typed with interfaces
- [ ] Components support different sizes and states
- [ ] Error states are visually distinct
- [ ] Form validation provides clear feedback
- [ ] Components are accessible (labels, ARIA attributes)
- [ ] Code follows Vue 3.4+ best practices
