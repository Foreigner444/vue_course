# Lesson 12.2: Basic Form Binding with `v-model` — Practice & Application (Базовая привязка форм с `v-model`)

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 12.2 of 25 (Part 2 of 2) |
| **Topic** | Basic Form Binding with `v-model` / Базовая привязка форм |
| **Continues From** | Lesson 12.1 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Build complete forms using `v-model` with various input types and modifiers
2. **Analyze:** Debug common `v-model` issues and choose appropriate binding strategies

---

## 2. Quick Recap

- **`v-model`** creates two-way binding between form elements and reactive state
- It's syntactic sugar for `:value` + `@input` (or `:checked` + `@change` for checkboxes)
- **Modifiers:** `.lazy` (update on blur), `.number` (cast to number), `.trim` (strip whitespace)
- Different input types bind to different data types (string, boolean, array)

---

## 3. The Variable Frame & Complexity Scale

**Basic Form:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const name = ref('')
</script>

<template>
  <input v-model="name" placeholder="Your name" />
  <p>Hello, {{ name || 'stranger' }}!</p>
</template>
```

**With Modifiers:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const username = ref('')
const age = ref<number | null>(null)
const notes = ref('')
</script>

<template>
  <input v-model.trim="username" placeholder="Username (trimmed)" />
  <input v-model.number="age" type="number" placeholder="Age (as number)" />
  <textarea v-model.lazy="notes" placeholder="Notes (updates on blur)" />
  
  <pre>{{ { username, age, ageType: typeof age, notes } }}</pre>
</template>
```

**Advanced Form (Complete Registration):**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

interface FormData {
  email: string
  password: string
  confirmPassword: string
  age: number
  newsletter: boolean
  interests: string[]
  experience: string
}

const form = ref<FormData>({
  email: '',
  password: '',
  confirmPassword: '',
  age: 18,
  newsletter: true,
  interests: [],
  experience: 'beginner'
})

const passwordsMatch = computed(() => 
  form.value.password === form.value.confirmPassword
)

const isValidEmail = computed(() => 
  /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.value.email)
)

const canSubmit = computed(() =>
  isValidEmail.value && 
  form.value.password.length >= 8 && 
  passwordsMatch.value
)
</script>

<template>
  <form class="space-y-4">
    <input v-model.trim="form.email" type="email" placeholder="Email" />
    <input v-model="form.password" type="password" placeholder="Password (8+ chars)" />
    <input v-model="form.confirmPassword" type="password" placeholder="Confirm" />
    <input v-model.number="form.age" type="range" min="13" max="100" />
    
    <label><input type="checkbox" v-model="form.newsletter" /> Newsletter</label>
    
    <div>
      <label><input type="checkbox" v-model="form.interests" value="vue" /> Vue</label>
      <label><input type="checkbox" v-model="form.interests" value="react" /> React</label>
    </div>
    
    <select v-model="form.experience">
      <option value="beginner">Beginner</option>
      <option value="intermediate">Intermediate</option>
      <option value="expert">Expert</option>
    </select>
    
    <button :disabled="!canSubmit">Submit</button>
  </form>
</template>
```

---

## 4. Micro-Implementation Drills

### Drill 1 (Basic): Live Preview Input
Create an input that shows a live preview of what the user is typing.

```vue
<script setup lang="ts">
import { ref } from 'vue'

// TODO: Create a ref called 'message' initialized to ''
</script>

<template>
  <div class="p-4">
    <!-- TODO: Add v-model to bind the input to message -->
    <input
      type="text"
      class="input input-bordered w-full"
      placeholder="Type a message..."
    />
    
    <!-- Preview area -->
    <div class="mt-4 p-4 bg-base-200 rounded min-h-16">
      <p class="text-lg">{{ message || 'Your message will appear here...' }}</p>
    </div>
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const message = ref('')
</script>

<template>
  <div class="p-4">
    <input
      v-model="message"
      type="text"
      class="input input-bordered w-full"
      placeholder="Type a message..."
    />
    
    <div class="mt-4 p-4 bg-base-200 rounded min-h-16">
      <p class="text-lg">{{ message || 'Your message will appear here...' }}</p>
    </div>
  </div>
</template>
```

---

### Drill 2 (Variation): Number Input with Range Slider
Create synchronized number input and range slider.

```vue
<script setup lang="ts">
import { ref } from 'vue'

// TODO: Create a ref called 'volume' initialized to 50
// Make sure it's typed as number
</script>

<template>
  <div class="p-4 space-y-4">
    <!-- TODO: Bind both inputs to the same 'volume' ref -->
    <!-- Use .number modifier on the number input -->
    
    <div class="flex items-center gap-4">
      <span>🔈</span>
      <input
        type="range"
        min="0"
        max="100"
        class="range range-primary flex-1"
      />
      <span>🔊</span>
    </div>
    
    <div class="flex items-center gap-2">
      <input
        type="number"
        min="0"
        max="100"
        class="input input-bordered w-24"
      />
      <span>%</span>
    </div>
    
    <p>Current volume: {{ volume }}</p>
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const volume = ref<number>(50)
</script>

<template>
  <div class="p-4 space-y-4">
    <div class="flex items-center gap-4">
      <span>🔈</span>
      <input
        v-model.number="volume"
        type="range"
        min="0"
        max="100"
        class="range range-primary flex-1"
      />
      <span>🔊</span>
    </div>
    
    <div class="flex items-center gap-2">
      <input
        v-model.number="volume"
        type="number"
        min="0"
        max="100"
        class="input input-bordered w-24"
      />
      <span>%</span>
    </div>
    
    <p>Current volume: {{ volume }}</p>
  </div>
</template>
```

---

### Drill 3 (Combination): Multi-Select with Checkboxes
Create a skills selector with multiple checkboxes bound to an array.

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

const availableSkills = ['JavaScript', 'TypeScript', 'Vue', 'React', 'Node.js', 'Python']

// TODO: Create a ref called 'selectedSkills' as an empty array of strings
// TODO: Create a computed property 'skillCount' that returns the number of selected skills
</script>

<template>
  <div class="p-4">
    <h3 class="font-bold mb-4">Select Your Skills ({{ skillCount }} selected)</h3>
    
    <div class="space-y-2">
      <!-- TODO: Use v-for to render checkboxes for each skill -->
      <!-- Each checkbox should have v-model bound to selectedSkills -->
      <!-- Use :value to set each checkbox's value to the skill name -->
    </div>
    
    <div class="mt-4 p-4 bg-base-200 rounded">
      <p class="font-semibold">Selected:</p>
      <p>{{ selectedSkills.join(', ') || 'None selected' }}</p>
    </div>
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

const availableSkills = ['JavaScript', 'TypeScript', 'Vue', 'React', 'Node.js', 'Python']
const selectedSkills = ref<string[]>([])

const skillCount = computed(() => selectedSkills.value.length)
</script>

<template>
  <div class="p-4">
    <h3 class="font-bold mb-4">Select Your Skills ({{ skillCount }} selected)</h3>
    
    <div class="space-y-2">
      <label
        v-for="skill in availableSkills"
        :key="skill"
        class="flex items-center gap-2 cursor-pointer"
      >
        <input
          v-model="selectedSkills"
          :value="skill"
          type="checkbox"
          class="checkbox"
        />
        <span>{{ skill }}</span>
      </label>
    </div>
    
    <div class="mt-4 p-4 bg-base-200 rounded">
      <p class="font-semibold">Selected:</p>
      <p>{{ selectedSkills.join(', ') || 'None selected' }}</p>
    </div>
  </div>
</template>
```

---

### Drill 4 (Edge Case): Dynamic Object Form
Create a form that edits properties of a reactive object.

```vue
<script setup lang="ts">
import { reactive } from 'vue'

// TODO: Create a reactive object 'user' with name, email, and role properties
// Use reactive() instead of ref() for the object
</script>

<template>
  <div class="p-4 space-y-4">
    <!-- TODO: Bind inputs directly to user.name, user.email, user.role -->
    <!-- Note: No .value needed when using reactive()! -->
    
    <div>
      <label class="label">Name</label>
      <input type="text" class="input input-bordered w-full" />
    </div>
    
    <div>
      <label class="label">Email</label>
      <input type="email" class="input input-bordered w-full" />
    </div>
    
    <div>
      <label class="label">Role</label>
      <select class="select select-bordered w-full">
        <option value="user">User</option>
        <option value="admin">Admin</option>
        <option value="moderator">Moderator</option>
      </select>
    </div>
    
    <pre class="bg-base-200 p-4 rounded">{{ user }}</pre>
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { reactive } from 'vue'

const user = reactive({
  name: '',
  email: '',
  role: 'user'
})
</script>

<template>
  <div class="p-4 space-y-4">
    <div>
      <label class="label">Name</label>
      <input v-model="user.name" type="text" class="input input-bordered w-full" />
    </div>
    
    <div>
      <label class="label">Email</label>
      <input v-model.trim="user.email" type="email" class="input input-bordered w-full" />
    </div>
    
    <div>
      <label class="label">Role</label>
      <select v-model="user.role" class="select select-bordered w-full">
        <option value="user">User</option>
        <option value="admin">Admin</option>
        <option value="moderator">Moderator</option>
      </select>
    </div>
    
    <pre class="bg-base-200 p-4 rounded">{{ user }}</pre>
  </div>
</template>
```

---

## 5. Common Pitfalls & Anti-Patterns

| ❌ Common Mistake | ✅ Correct Approach | Why It Matters |
|:--- |:--- |:--- |
| Using `:value` and `@input` separately when `v-model` would work | Use `v-model` for two-way binding | Reduces boilerplate and potential bugs from mismatched handlers |
| Forgetting `.number` modifier and getting strings from number inputs | Use `v-model.number="age"` | `<input type="number">` still returns strings without the modifier |
| Binding multiple checkboxes to a boolean instead of an array | Use `ref<string[]>([])` for checkbox groups | Single boolean only works for one checkbox; groups need arrays |

### Detailed Examples

**Mistake 1: Reinventing v-model**
```vue
<!-- ❌ WRONG: Unnecessarily verbose -->
<script setup lang="ts">
import { ref } from 'vue'

const name = ref('')

function updateName(event: Event): void {
  name.value = (event.target as HTMLInputElement).value
}
</script>

<template>
  <input :value="name" @input="updateName" />
</template>

<!-- ✅ CORRECT: Let v-model do the work -->
<script setup lang="ts">
import { ref } from 'vue'

const name = ref('')
</script>

<template>
  <input v-model="name" />
</template>
```

**Mistake 2: String Numbers**
```vue
<!-- ❌ WRONG: age will be a string "25", not number 25 -->
<script setup lang="ts">
import { ref } from 'vue'

const age = ref(0)
</script>

<template>
  <input v-model="age" type="number" />
  <p>Age + 1 = {{ age + 1 }}</p> <!-- Shows "251" not 26! -->
</template>

<!-- ✅ CORRECT: Use .number modifier -->
<script setup lang="ts">
import { ref } from 'vue'

const age = ref(0)
</script>

<template>
  <input v-model.number="age" type="number" />
  <p>Age + 1 = {{ age + 1 }}</p> <!-- Shows 26 -->
</template>
```

**Mistake 3: Wrong Type for Checkbox Groups**
```vue
<!-- ❌ WRONG: Can't track multiple selections with boolean -->
<script setup lang="ts">
import { ref } from 'vue'

const selected = ref(false) // Wrong type!
</script>

<template>
  <input type="checkbox" v-model="selected" value="a" /> A
  <input type="checkbox" v-model="selected" value="b" /> B
</template>

<!-- ✅ CORRECT: Use array for multiple checkboxes -->
<script setup lang="ts">
import { ref } from 'vue'

const selected = ref<string[]>([])
</script>

<template>
  <input type="checkbox" v-model="selected" value="a" /> A
  <input type="checkbox" v-model="selected" value="b" /> B
  <p>Selected: {{ selected }}</p>
</template>
```

---

## 6. Mini-Project (Putting It Together)

**User Profile Editor**

A complete form for editing a user profile with validation and real-time preview.

**File Structure:**
```
src/
├── components/
│   └── ProfileEditor.vue
├── App.vue
└── main.ts
```

**ProfileEditor.vue:**
```vue
<script setup lang="ts">
import { reactive, computed } from 'vue'

interface UserProfile {
  displayName: string
  email: string
  bio: string
  website: string
  birthYear: number
  isPublic: boolean
  notifications: string[]
  theme: string
}

const profile = reactive<UserProfile>({
  displayName: '',
  email: '',
  bio: '',
  website: '',
  birthYear: 2000,
  isPublic: true,
  notifications: ['email'],
  theme: 'system'
})

const notificationOptions = [
  { value: 'email', label: 'Email notifications' },
  { value: 'push', label: 'Push notifications' },
  { value: 'sms', label: 'SMS notifications' }
]

const themeOptions = [
  { value: 'light', label: '☀️ Light' },
  { value: 'dark', label: '🌙 Dark' },
  { value: 'system', label: '💻 System' }
]

const age = computed(() => new Date().getFullYear() - profile.birthYear)

const isValidEmail = computed(() => {
  if (!profile.email) return true
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(profile.email)
})

const isValidWebsite = computed(() => {
  if (!profile.website) return true
  try {
    new URL(profile.website)
    return true
  } catch {
    return false
  }
})

const bioCharCount = computed(() => profile.bio.length)
const bioMaxLength = 200

const canSave = computed(() => {
  return (
    profile.displayName.trim().length >= 2 &&
    isValidEmail.value &&
    isValidWebsite.value &&
    profile.bio.length <= bioMaxLength
  )
})

function handleSave(): void {
  if (!canSave.value) return
  console.log('Saving profile:', { ...profile })
  alert('Profile saved successfully!')
}

function handleReset(): void {
  Object.assign(profile, {
    displayName: '',
    email: '',
    bio: '',
    website: '',
    birthYear: 2000,
    isPublic: true,
    notifications: ['email'],
    theme: 'system'
  })
}
</script>

<template>
  <div class="max-w-2xl mx-auto p-6">
    <h1 class="text-2xl font-bold mb-6">Edit Profile</h1>
    
    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
      <!-- Form Section -->
      <div class="space-y-4">
        <!-- Display Name -->
        <div class="form-control">
          <label class="label">
            <span class="label-text">Display Name *</span>
          </label>
          <input
            v-model.trim="profile.displayName"
            type="text"
            class="input input-bordered"
            :class="{ 'input-error': profile.displayName && profile.displayName.length < 2 }"
            placeholder="Your display name"
          />
          <label v-if="profile.displayName && profile.displayName.length < 2" class="label">
            <span class="label-text-alt text-error">Minimum 2 characters</span>
          </label>
        </div>
        
        <!-- Email -->
        <div class="form-control">
          <label class="label">
            <span class="label-text">Email</span>
          </label>
          <input
            v-model.trim="profile.email"
            type="email"
            class="input input-bordered"
            :class="{ 'input-error': !isValidEmail }"
            placeholder="you@example.com"
          />
          <label v-if="!isValidEmail" class="label">
            <span class="label-text-alt text-error">Invalid email format</span>
          </label>
        </div>
        
        <!-- Website -->
        <div class="form-control">
          <label class="label">
            <span class="label-text">Website</span>
          </label>
          <input
            v-model.trim="profile.website"
            type="url"
            class="input input-bordered"
            :class="{ 'input-error': !isValidWebsite }"
            placeholder="https://yourwebsite.com"
          />
          <label v-if="!isValidWebsite" class="label">
            <span class="label-text-alt text-error">Invalid URL format</span>
          </label>
        </div>
        
        <!-- Bio with character count -->
        <div class="form-control">
          <label class="label">
            <span class="label-text">Bio</span>
            <span 
              class="label-text-alt"
              :class="{ 'text-error': bioCharCount > bioMaxLength }"
            >
              {{ bioCharCount }}/{{ bioMaxLength }}
            </span>
          </label>
          <textarea
            v-model="profile.bio"
            class="textarea textarea-bordered h-24"
            :class="{ 'textarea-error': bioCharCount > bioMaxLength }"
            placeholder="Tell us about yourself..."
          />
        </div>
        
        <!-- Birth Year with Slider -->
        <div class="form-control">
          <label class="label">
            <span class="label-text">Birth Year</span>
            <span class="label-text-alt">Age: {{ age }}</span>
          </label>
          <div class="flex items-center gap-4">
            <input
              v-model.number="profile.birthYear"
              type="range"
              min="1920"
              max="2010"
              class="range range-sm flex-1"
            />
            <input
              v-model.number="profile.birthYear"
              type="number"
              min="1920"
              max="2010"
              class="input input-bordered input-sm w-24"
            />
          </div>
        </div>
        
        <!-- Public Profile Toggle -->
        <div class="form-control">
          <label class="label cursor-pointer">
            <span class="label-text">Public Profile</span>
            <input
              v-model="profile.isPublic"
              type="checkbox"
              class="toggle toggle-primary"
            />
          </label>
        </div>
        
        <!-- Notification Preferences (checkbox group) -->
        <div class="form-control">
          <label class="label">
            <span class="label-text">Notifications</span>
          </label>
          <div class="space-y-2">
            <label
              v-for="option in notificationOptions"
              :key="option.value"
              class="flex items-center gap-2 cursor-pointer"
            >
              <input
                v-model="profile.notifications"
                :value="option.value"
                type="checkbox"
                class="checkbox checkbox-sm"
              />
              <span class="text-sm">{{ option.label }}</span>
            </label>
          </div>
        </div>
        
        <!-- Theme Selection (radio group) -->
        <div class="form-control">
          <label class="label">
            <span class="label-text">Theme</span>
          </label>
          <div class="flex gap-4">
            <label
              v-for="option in themeOptions"
              :key="option.value"
              class="flex items-center gap-2 cursor-pointer"
            >
              <input
                v-model="profile.theme"
                :value="option.value"
                type="radio"
                class="radio radio-sm"
              />
              <span class="text-sm">{{ option.label }}</span>
            </label>
          </div>
        </div>
        
        <!-- Action Buttons -->
        <div class="flex gap-2 pt-4">
          <button
            @click="handleSave"
            class="btn btn-primary flex-1"
            :disabled="!canSave"
          >
            Save Profile
          </button>
          <button
            @click="handleReset"
            class="btn btn-ghost"
          >
            Reset
          </button>
        </div>
      </div>
      
      <!-- Live Preview Section -->
      <div class="bg-base-200 rounded-lg p-4">
        <h2 class="font-semibold mb-4">Live Preview</h2>
        
        <div class="card bg-base-100 shadow-xl">
          <div class="card-body">
            <h3 class="card-title">
              {{ profile.displayName || 'Your Name' }}
              <span v-if="profile.isPublic" class="badge badge-success badge-sm">Public</span>
              <span v-else class="badge badge-warning badge-sm">Private</span>
            </h3>
            
            <p v-if="profile.email" class="text-sm opacity-70">
              {{ profile.email }}
            </p>
            
            <p class="text-sm">
              {{ profile.bio || 'No bio yet...' }}
            </p>
            
            <p v-if="profile.website" class="text-sm">
              <a :href="profile.website" class="link link-primary" target="_blank">
                {{ profile.website }}
              </a>
            </p>
            
            <div class="flex flex-wrap gap-1 mt-2">
              <span class="badge badge-outline">Age: {{ age }}</span>
              <span class="badge badge-outline">Theme: {{ profile.theme }}</span>
            </div>
            
            <div v-if="profile.notifications.length" class="text-xs opacity-70 mt-2">
              Notifications: {{ profile.notifications.join(', ') }}
            </div>
          </div>
        </div>
        
        <!-- Debug Data -->
        <details class="mt-4">
          <summary class="cursor-pointer text-sm opacity-70">Raw Data</summary>
          <pre class="text-xs mt-2 overflow-auto">{{ profile }}</pre>
        </details>
      </div>
    </div>
  </div>
</template>
```

**How to Test:**
1. Create a new Vue project with Tailwind and DaisyUI
2. Add the `ProfileEditor.vue` component
3. Import and use in `App.vue`
4. Test: Edit each field and observe real-time preview updates
5. Verify: Validation messages appear for invalid inputs
6. Confirm: Save button is disabled until form is valid

---

## 7. Implementation Exercises (Progressive)

### Level 1 (Recall): Fill in the Blanks
```vue
<script setup lang="ts">
import { ref } from 'vue'

const searchQuery = ref('')
</script>

<template>
  <!-- Fill in the blank to enable two-way binding with whitespace trimming -->
  <input _____._____="searchQuery" placeholder="Search..." />
</template>
```

**Answer:** `v-model.trim`

---

### Level 2 (Apply): Independent Coding Tasks

**Exercise 2.1: Temperature Converter**
Create a component with two inputs:
- Celsius input
- Fahrenheit input
- Both update each other in real-time
- Hint: Use `computed` with getter/setter or watchers

**Exercise 2.2: Todo Quick Add**
Create a form with:
- Text input for todo title
- Priority select (low, medium, high)
- Press Enter or click button to add to a list
- Clear inputs after adding

---

### Level 3 (Debug): Fix the Broken Code
```vue
<script setup lang="ts">
import { ref } from 'vue'

const quantity = ref(1)
const items = ref(['apples', 'bananas'])
const selectedItem = ref()
</script>

<template>
  <div>
    <!-- Bug 1: Quantity is showing as string concatenation -->
    <input v-model="quantity" type="number" />
    <p>Total: {{ quantity + 10 }}</p>  <!-- Shows "110" instead of 11 -->
    
    <!-- Bug 2: selectedItem is undefined even after selection -->
    <select v-model="selected">
      <option v-for="item in items" :key="item">{{ item }}</option>
    </select>
    <p>You selected: {{ selectedItem }}</p>
    
    <!-- Bug 3: Radio buttons don't work -->
    <label><input type="radio" v-model="size" value="small" /> Small</label>
    <label><input type="radio" v-model="size" value="large" /> Large</label>
  </div>
</template>
```

**Bugs to Find:**
1. Missing `.number` modifier on quantity input
2. `v-model="selected"` should be `v-model="selectedItem"` (typo)
3. `size` ref is never declared

**Fixed Code:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const quantity = ref(1)
const items = ref(['apples', 'bananas'])
const selectedItem = ref('')
const size = ref('small')
</script>

<template>
  <div>
    <input v-model.number="quantity" type="number" />
    <p>Total: {{ quantity + 10 }}</p>
    
    <select v-model="selectedItem">
      <option v-for="item in items" :key="item" :value="item">{{ item }}</option>
    </select>
    <p>You selected: {{ selectedItem }}</p>
    
    <label><input type="radio" v-model="size" value="small" /> Small</label>
    <label><input type="radio" v-model="size" value="large" /> Large</label>
    <p>Size: {{ size }}</p>
  </div>
</template>
```

---

**Reply 'next' for Lesson 13.1 (Theory).**
