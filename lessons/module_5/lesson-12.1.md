# Lesson 12.1: Basic Form Binding with `v-model` — Theory & Concepts (Базовая привязка форм с `v-model`)

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 12.1 of 25 (Part 1 of 2) |
| **Topic** | Basic Form Binding with `v-model` / Базовая привязка форм |
| **Module** | Module 5: User Interaction |
| **Prerequisites** | Lessons 1-11 (Core Fundamentals, Event Handling) |
| **You Should Know** | HTML form elements, `ref()`, `@input`/`@change` events |
| **Unlocks** | Lesson 12.2, then Lessons 13, 14 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 1 (Theory):**
1. **Remember:** Define two-way binding, recall `v-model` syntax for different form elements
2. **Understand:** Explain how `v-model` works as syntactic sugar, how it differs across input types, and its relationship to `:value` + `@input`

---

## 2. Real-World Scenario & Context

**Scenario:** You're building a user registration form for a web application. As users type their name, email, and password, you need to capture each keystroke in real-time to validate input, show character counts, or enable/disable the submit button based on form completeness. Without a convenient binding mechanism, you'd write repetitive event handlers for every single field.

**Framework Context:** Vue's `v-model` directive creates a two-way binding between form inputs and reactive state. It's syntactic sugar that combines value binding (`:value`) with event listening (`@input` or `@change`), dramatically reducing boilerplate while keeping your data and UI in perfect sync.

---

## 3. Core Concepts Explained (Deep Dive)

### How It Actually Works

`v-model` is **not magic** — it's a compiler shorthand. When Vue sees `v-model="username"`, it expands it differently based on the element type:

**For text inputs:**
```vue
<!-- You write: -->
<input v-model="username" />

<!-- Vue compiles to: -->
<input :value="username" @input="username = $event.target.value" />
```

**For checkboxes:**
```vue
<!-- You write: -->
<input type="checkbox" v-model="isActive" />

<!-- Vue compiles to: -->
<input type="checkbox" :checked="isActive" @change="isActive = $event.target.checked" />
```

```
┌──────────────────────────────────────────────────────────────────┐
│                    v-model Two-Way Flow                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│    ┌──────────────────┐                ┌────────────────────┐    │
│    │                  │   :value       │                    │    │
│    │   Reactive Ref   │ ─────────────► │    <input />       │    │
│    │   username.value │                │    DOM Element     │    │
│    │                  │ ◄───────────── │                    │    │
│    └──────────────────┘   @input       └────────────────────┘    │
│                                                                  │
│    Data → View: When username.value changes, input shows new     │
│                 value automatically                              │
│                                                                  │
│    View → Data: When user types, @input fires and updates        │
│                 username.value                                   │
│                                                                  │
│    This two-way sync is what makes v-model "two-way binding"     │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Mental Model

**Think of `v-model` as a "synchronized mirror":**

Imagine you have a notepad (your reactive data) and a whiteboard (the input field). With `v-model`, they're connected by an invisible assistant who:
1. **Watches the notepad:** Whenever you change the notepad, the assistant immediately updates the whiteboard
2. **Watches the whiteboard:** Whenever someone writes on the whiteboard, the assistant copies it to the notepad

```
┌─────────────────────────────────────────────────────────────┐
│                   📝 The Synchronized Mirror                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────────┐         ┌─────────────┐                  │
│   │  📓 Notepad │ ◄─────► │ 🖥️ Screen  │                   │
│   │   (ref)     │  sync   │  (input)    │                   │
│   │             │         │             │                   │
│   │  "Alice"    │ ═══════ │  Alice█    │                   │
│   └─────────────┘         └─────────────┘                   │
│                                                             │
│   Change the notepad? Screen updates.                       │
│   Type on screen? Notepad updates.                          │
│   Always in sync, always together.                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### In Other Contexts

| Framework | Two-Way Binding | Notes |
|:--- |:--- |:--- |
| **Vue 3** | `v-model="data"` | Single directive, works on native + custom components |
| **React** | `value={data} onChange={handler}` | Explicit, must wire both directions manually |
| **Angular** | `[(ngModel)]="data"` | "Banana in a box" syntax, similar concept |
| **Svelte** | `bind:value={data}` | `bind:` prefix for two-way binding |
| **Vanilla JS** | Manual listeners + DOM updates | Full control, more code |

### When to Use / When NOT to Use

| ✅ Use `v-model` When | ❌ Avoid `v-model` When |
|:--- |:--- |
| Building forms with text inputs, checkboxes, radios, selects | You only need to display data (use `:value` or text interpolation) |
| You need real-time sync between input and state | You need to transform data before storing (use separate `:value` + `@input`) |
| Creating controlled form components | The input is purely decorative or disabled |
| Working with form validation in real-time | You're working with contenteditable (complex edge cases) |

---

## 4. New Terminology

| Term | Definition |
|:--- |:--- |
| **Two-Way Binding** | A synchronization pattern where changes to either the model (data) or the view (UI) automatically update the other. `v-model` implements this in Vue. |
| **Syntactic Sugar** | Code syntax that makes something easier to write but compiles to a more verbose form. `v-model` is sugar for `:value` + `@input`. |
| **Controlled Input** | A form input whose value is driven by reactive state rather than its own internal state. All `v-model` inputs are controlled. |
| **v-model Modifiers** | Suffixes like `.lazy`, `.number`, `.trim` that modify how `v-model` processes input values. |

---

## 5. Algorithmic Thinking (Planning the Solution)

**The Plan:** Building a registration form with real-time validation feedback.

```
┌─────────────────────────────────────────────────────────────┐
│         Decision Tree: Form Field Implementation            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Step 1: Identify form fields needed                       │
│   └── Text: username, email, password                       │
│   └── Checkbox: agree to terms                              │
│   └── Select: country                                       │
│       └── Create a ref for each field                       │
│                                                             │
│   Step 2: Choose appropriate v-model usage                  │
│   └── Text inputs: v-model="fieldRef"                       │
│   └── Password: v-model with type="password"                │
│   └── Checkbox: v-model binds to boolean                    │
│   └── Select: v-model binds to selected option value        │
│                                                             │
│   Step 3: Add modifiers if needed                           │
│   └── .trim for username/email (remove whitespace)          │
│   └── .lazy for password (update on blur, not input)        │
│   └── .number for age field (convert to number)             │
│                                                             │
│   Step 4: Create computed properties for validation         │
│   └── isUsernameValid, isEmailValid, etc.                   │
│   └── isFormValid combines all checks                       │
│                                                             │
│   Step 5: Bind validation state to UI feedback              │
│   └── Conditional classes for valid/invalid states          │
│   └── Error messages shown with v-if                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. Initial Pattern Introduction

| What You Want (Intent) | Code Chunk (The Pattern) | Conceptual Link |
|:--- |:--- |:--- |
| Bind text input | `<input v-model="text">` | Syncs with string ref |
| Bind number input | `<input v-model.number="age">` | Auto-converts to number |
| Trim whitespace | `<input v-model.trim="name">` | Removes leading/trailing spaces |
| Update on blur only | `<input v-model.lazy="email">` | Uses `@change` instead of `@input` |
| Bind checkbox | `<input type="checkbox" v-model="agreed">` | Syncs with boolean ref |
| Bind radio group | `<input type="radio" v-model="picked" value="A">` | Syncs with string ref |
| Bind select | `<select v-model="selected">` | Syncs with option value |
| Bind textarea | `<textarea v-model="content">` | Same as text input |

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

// Text input binding - stores the raw text value
const username = ref<string>('')

// Number input - .number modifier ensures numeric type
const age = ref<number>(0)

// Checkbox binding - stores boolean true/false
const agreedToTerms = ref<boolean>(false)

// Radio button group - stores the selected option's value
const selectedPlan = ref<string>('free')

// Select dropdown - stores the selected option's value
const country = ref<string>('')

// Textarea - works just like text input
const bio = ref<string>('')

// Computed validation example
const isUsernameValid = computed<boolean>(() => {
  return username.value.length >= 3
})

// Computed to check if form is complete
const isFormComplete = computed<boolean>(() => {
  return (
    username.value.length > 0 &&
    age.value > 0 &&
    agreedToTerms.value &&
    country.value !== ''
  )
})
</script>

<template>
  <form class="space-y-4 p-4">
    <!-- Text input with .trim modifier -->
    <div>
      <label class="label">Username</label>
      <input
        v-model.trim="username"
        type="text"
        class="input input-bordered w-full"
        :class="{ 'input-success': isUsernameValid, 'input-error': !isUsernameValid && username }"
        placeholder="Enter username"
      />
      <p v-if="username && !isUsernameValid" class="text-error text-sm mt-1">
        Username must be at least 3 characters
      </p>
    </div>
    
    <!-- Number input with .number modifier -->
    <div>
      <label class="label">Age</label>
      <input
        v-model.number="age"
        type="number"
        class="input input-bordered w-full"
        placeholder="Enter age"
      />
      <p class="text-sm text-gray-500">Type: {{ typeof age }}</p>
    </div>
    
    <!-- Checkbox binding -->
    <div class="form-control">
      <label class="label cursor-pointer justify-start gap-2">
        <input
          v-model="agreedToTerms"
          type="checkbox"
          class="checkbox"
        />
        <span>I agree to the terms and conditions</span>
      </label>
    </div>
    
    <!-- Radio button group -->
    <div>
      <label class="label">Select Plan</label>
      <div class="flex gap-4">
        <label class="label cursor-pointer gap-2">
          <input
            v-model="selectedPlan"
            type="radio"
            value="free"
            class="radio"
          />
          <span>Free</span>
        </label>
        <label class="label cursor-pointer gap-2">
          <input
            v-model="selectedPlan"
            type="radio"
            value="pro"
            class="radio"
          />
          <span>Pro</span>
        </label>
        <label class="label cursor-pointer gap-2">
          <input
            v-model="selectedPlan"
            type="radio"
            value="enterprise"
            class="radio"
          />
          <span>Enterprise</span>
        </label>
      </div>
      <p class="text-sm">Selected: {{ selectedPlan }}</p>
    </div>
    
    <!-- Select dropdown -->
    <div>
      <label class="label">Country</label>
      <select v-model="country" class="select select-bordered w-full">
        <option value="" disabled>Choose a country</option>
        <option value="us">United States</option>
        <option value="uk">United Kingdom</option>
        <option value="ca">Canada</option>
        <option value="au">Australia</option>
      </select>
    </div>
    
    <!-- Textarea with .lazy modifier -->
    <div>
      <label class="label">Bio</label>
      <textarea
        v-model.lazy="bio"
        class="textarea textarea-bordered w-full"
        rows="3"
        placeholder="Tell us about yourself (updates on blur)"
      />
      <p class="text-sm text-gray-500">{{ bio.length }} characters</p>
    </div>
    
    <!-- Submit button -->
    <button
      type="submit"
      class="btn btn-primary"
      :disabled="!isFormComplete"
    >
      Submit
    </button>
  </form>
</template>
```

### v-model on Different Input Types

```vue
<script setup lang="ts">
import { ref } from 'vue'

// Different types require different ref types
const textValue = ref<string>('')           // text, email, password, search, url, tel
const numberValue = ref<number>(0)          // number, range (with .number modifier)
const booleanValue = ref<boolean>(false)    // single checkbox
const arrayValue = ref<string[]>([])        // multiple checkboxes
const singleValue = ref<string>('')         // radio buttons, single select
const multiValue = ref<string[]>([])        // multi-select
</script>

<template>
  <!-- Multiple checkboxes bind to array -->
  <div>
    <label>
      <input type="checkbox" v-model="arrayValue" value="vue" />
      Vue
    </label>
    <label>
      <input type="checkbox" v-model="arrayValue" value="react" />
      React
    </label>
    <label>
      <input type="checkbox" v-model="arrayValue" value="angular" />
      Angular
    </label>
    <p>Selected: {{ arrayValue.join(', ') }}</p>
  </div>
  
  <!-- Multi-select dropdown -->
  <select v-model="multiValue" multiple>
    <option value="red">Red</option>
    <option value="green">Green</option>
    <option value="blue">Blue</option>
  </select>
  <p>Selected: {{ multiValue }}</p>
</template>
```

---

## 7. Comprehension Check

1. **What is the expanded form of `<input v-model="name" />`?**
   - *Hint: Think about what property is bound and what event is listened to.*

2. **What is the difference between `v-model` and `v-model.lazy`?**
   - *Hint: Consider when the data updates — on every keystroke or only when the user leaves the field.*

3. **True or False: When using `v-model` on a checkbox, the bound value must always be a boolean.**
   - *Hint: Think about what happens with multiple checkboxes sharing the same model.*

---

**Reply 'next' for Lesson 12.2 (Practice).**
