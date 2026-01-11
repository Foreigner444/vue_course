# Lesson 9.2: Conditional Rendering — Practice & Application (Условный рендеринг)

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 9.2 of 25 (Part 2 of 2) |
| **Topic** | Conditional Rendering: `v-if`, `v-else-if`, & `v-show` / Условный рендеринг |
| **Continues From** | Lesson 9.1 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Build components that dynamically show/hide content based on reactive state
2. **Analyze:** Choose between `v-if` and `v-show` based on performance requirements and use case

---

## 2. Quick Recap

- **`v-if`** completely adds/removes elements from the DOM — use for rarely changing conditions
- **`v-show`** toggles CSS `display: none` — use for frequently toggling elements
- **`v-else-if`** and **`v-else`** must immediately follow `v-if` (no elements between them)
- `v-show` does NOT support `v-else` or work on `<template>`

---

## 3. The Variable Frame & Complexity Scale

### Basic Form: Simple Boolean Toggle

```vue
<script setup lang="ts">
import { ref } from 'vue'

const isVisible = ref<boolean>(true)
</script>

<template>
  <button @click="isVisible = !isVisible">Toggle</button>
  <p v-if="isVisible">I appear and disappear from the DOM!</p>
</template>
```

### With Options: Using `v-else` for Alternatives

```vue
<script setup lang="ts">
import { ref } from 'vue'

const isLoading = ref<boolean>(true)

setTimeout(() => {
  isLoading.value = false
}, 2000)
</script>

<template>
  <div v-if="isLoading" class="flex items-center gap-2">
    <span class="loading loading-spinner"></span>
    <span>Loading data...</span>
  </div>
  <div v-else class="alert alert-success">
    Data loaded successfully!
  </div>
</template>
```

### Advanced Form: Multi-Condition Chain with Computed Properties

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

type Status = 'idle' | 'loading' | 'success' | 'error'

const status = ref<Status>('idle')
const errorMessage = ref<string>('')

const isIdle = computed(() => status.value === 'idle')
const isLoading = computed(() => status.value === 'loading')
const isSuccess = computed(() => status.value === 'success')
const isError = computed(() => status.value === 'error')

const fetchData = async () => {
  status.value = 'loading'
  try {
    await new Promise(resolve => setTimeout(resolve, 1500))
    const random = Math.random()
    if (random > 0.5) {
      status.value = 'success'
    } else {
      throw new Error('Random failure for demo')
    }
  } catch (e) {
    status.value = 'error'
    errorMessage.value = (e as Error).message
  }
}
</script>

<template>
  <div class="card bg-base-200 p-6">
    <div v-if="isIdle" class="text-center">
      <p class="mb-4">Click to fetch data</p>
      <button class="btn btn-primary" @click="fetchData">Fetch Data</button>
    </div>
    
    <div v-else-if="isLoading" class="flex justify-center">
      <span class="loading loading-dots loading-lg"></span>
    </div>
    
    <div v-else-if="isSuccess" class="alert alert-success">
      <span>✓ Data fetched successfully!</span>
      <button class="btn btn-sm" @click="status = 'idle'">Reset</button>
    </div>
    
    <div v-else class="alert alert-error">
      <span>✗ Error: {{ errorMessage }}</span>
      <button class="btn btn-sm" @click="fetchData">Retry</button>
    </div>
  </div>
</template>
```

---

## 4. Micro-Implementation Drills

### Drill 1 (Basic): Simple Show/Hide Toggle

Create a component that shows/hides a secret message when a button is clicked.

```vue
<script setup lang="ts">
import { ref } from 'vue'

// TODO: Create a boolean ref called 'showSecret' initialized to false
const showSecret = ref(false)
</script>

<template>
  <div class="p-4">
    <!-- TODO: Add @click handler to toggle showSecret -->
    <button class="btn btn-primary">
      {{ showSecret ? 'Hide' : 'Reveal' }} Secret
    </button>
    
    <!-- TODO: Add v-if directive to conditionally show this paragraph -->
    <p class="mt-4 p-4 bg-warning text-warning-content rounded">
      🤫 The secret code is: VUE3ROCKS
    </p>
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const showSecret = ref<boolean>(false)
</script>

<template>
  <div class="p-4">
    <button class="btn btn-primary" @click="showSecret = !showSecret">
      {{ showSecret ? 'Hide' : 'Reveal' }} Secret
    </button>
    
    <p v-if="showSecret" class="mt-4 p-4 bg-warning text-warning-content rounded">
      🤫 The secret code is: VUE3ROCKS
    </p>
  </div>
</template>
```

---

### Drill 2 (Variation): v-if vs v-show Comparison

Create a component that demonstrates the difference between `v-if` and `v-show` by toggling both.

```vue
<script setup lang="ts">
import { ref } from 'vue'

const showBoth = ref<boolean>(true)
</script>

<template>
  <div class="p-4 space-y-4">
    <button class="btn btn-secondary" @click="showBoth = !showBoth">
      Toggle Both (currently: {{ showBoth }})
    </button>
    
    <div class="grid grid-cols-2 gap-4">
      <div class="border p-4 rounded">
        <h3 class="font-bold mb-2">v-if Element</h3>
        <!-- TODO: Add v-if directive -->
        <div class="bg-primary text-primary-content p-4 rounded">
          I use v-if (check DOM in DevTools - I disappear!)
        </div>
      </div>
      
      <div class="border p-4 rounded">
        <h3 class="font-bold mb-2">v-show Element</h3>
        <!-- TODO: Add v-show directive -->
        <div class="bg-secondary text-secondary-content p-4 rounded">
          I use v-show (check DOM in DevTools - I'm always there!)
        </div>
      </div>
    </div>
    
    <p class="text-sm opacity-70">
      Open browser DevTools → Elements tab → Toggle and observe the DOM
    </p>
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const showBoth = ref<boolean>(true)
</script>

<template>
  <div class="p-4 space-y-4">
    <button class="btn btn-secondary" @click="showBoth = !showBoth">
      Toggle Both (currently: {{ showBoth }})
    </button>
    
    <div class="grid grid-cols-2 gap-4">
      <div class="border p-4 rounded">
        <h3 class="font-bold mb-2">v-if Element</h3>
        <div v-if="showBoth" class="bg-primary text-primary-content p-4 rounded">
          I use v-if (check DOM in DevTools - I disappear!)
        </div>
      </div>
      
      <div class="border p-4 rounded">
        <h3 class="font-bold mb-2">v-show Element</h3>
        <div v-show="showBoth" class="bg-secondary text-secondary-content p-4 rounded">
          I use v-show (check DOM in DevTools - I'm always there!)
        </div>
      </div>
    </div>
    
    <p class="text-sm opacity-70">
      Open browser DevTools → Elements tab → Toggle and observe the DOM
    </p>
  </div>
</template>
```

---

### Drill 3 (Combination): Conditional Rendering + Computed Properties

Build a user role display that shows different badges based on the user's role.

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

type UserRole = 'guest' | 'user' | 'moderator' | 'admin'

const currentRole = ref<UserRole>('guest')

// TODO: Create computed properties for role checks
// const isGuest = computed(() => ...)
// const isUser = computed(() => ...)
// const isModerator = computed(() => ...)
// const isAdmin = computed(() => ...)

const roles: UserRole[] = ['guest', 'user', 'moderator', 'admin']
</script>

<template>
  <div class="p-4 space-y-4">
    <div class="flex gap-2">
      <button 
        v-for="role in roles" 
        :key="role"
        class="btn btn-sm"
        :class="{ 'btn-active': currentRole === role }"
        @click="currentRole = role"
      >
        {{ role }}
      </button>
    </div>
    
    <!-- TODO: Add v-if/v-else-if/v-else chain to show appropriate badge -->
    <div class="badge badge-lg badge-ghost">Guest - Limited Access</div>
    <div class="badge badge-lg badge-info">User - Standard Access</div>
    <div class="badge badge-lg badge-warning">Moderator - Can manage content</div>
    <div class="badge badge-lg badge-error">Admin - Full Access</div>
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

type UserRole = 'guest' | 'user' | 'moderator' | 'admin'

const currentRole = ref<UserRole>('guest')

const isGuest = computed(() => currentRole.value === 'guest')
const isUser = computed(() => currentRole.value === 'user')
const isModerator = computed(() => currentRole.value === 'moderator')
const isAdmin = computed(() => currentRole.value === 'admin')

const roles: UserRole[] = ['guest', 'user', 'moderator', 'admin']
</script>

<template>
  <div class="p-4 space-y-4">
    <div class="flex gap-2">
      <button 
        v-for="role in roles" 
        :key="role"
        class="btn btn-sm"
        :class="{ 'btn-active': currentRole === role }"
        @click="currentRole = role"
      >
        {{ role }}
      </button>
    </div>
    
    <div v-if="isGuest" class="badge badge-lg badge-ghost">
      Guest - Limited Access
    </div>
    <div v-else-if="isUser" class="badge badge-lg badge-info">
      User - Standard Access
    </div>
    <div v-else-if="isModerator" class="badge badge-lg badge-warning">
      Moderator - Can manage content
    </div>
    <div v-else class="badge badge-lg badge-error">
      Admin - Full Access
    </div>
  </div>
</template>
```

---

### Drill 4 (Edge Case): Using `<template>` for Grouping

Sometimes you need to conditionally render multiple elements without a wrapper. Use `<template v-if>` for this.

```vue
<script setup lang="ts">
import { ref } from 'vue'

const showDetails = ref<boolean>(false)

const user = {
  name: 'Alice Johnson',
  email: 'alice@example.com',
  role: 'Developer',
  department: 'Engineering'
}
</script>

<template>
  <div class="p-4">
    <h2 class="text-xl font-bold">{{ user.name }}</h2>
    
    <button class="btn btn-sm my-2" @click="showDetails = !showDetails">
      {{ showDetails ? 'Hide' : 'Show' }} Details
    </button>
    
    <!-- 
      TODO: Use <template v-if> to conditionally show these elements
      without adding an extra wrapper div to the DOM
    -->
    <p><strong>Email:</strong> {{ user.email }}</p>
    <p><strong>Role:</strong> {{ user.role }}</p>
    <p><strong>Department:</strong> {{ user.department }}</p>
    
    <!-- Note: v-show does NOT work on <template> -->
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const showDetails = ref<boolean>(false)

const user = {
  name: 'Alice Johnson',
  email: 'alice@example.com',
  role: 'Developer',
  department: 'Engineering'
}
</script>

<template>
  <div class="p-4">
    <h2 class="text-xl font-bold">{{ user.name }}</h2>
    
    <button class="btn btn-sm my-2" @click="showDetails = !showDetails">
      {{ showDetails ? 'Hide' : 'Show' }} Details
    </button>
    
    <template v-if="showDetails">
      <p><strong>Email:</strong> {{ user.email }}</p>
      <p><strong>Role:</strong> {{ user.role }}</p>
      <p><strong>Department:</strong> {{ user.department }}</p>
    </template>
  </div>
</template>
```

---

## 5. Common Pitfalls & Anti-Patterns

| ❌ Common Mistake | ✅ Correct Approach | Why It Matters |
|:--- |:--- |:--- |
| Putting elements between `v-if` and `v-else` | Keep `v-if`/`v-else-if`/`v-else` as adjacent siblings | Vue cannot associate the `v-else` with its `v-if` if there are elements between them |
| Using `v-if` and `v-for` on the same element | Use `<template v-for>` with `v-if` inside, or filter with computed | `v-if` has higher priority than `v-for`, causing unexpected behavior and performance issues |
| Using `v-show` on `<template>` | Use `v-if` on `<template>` instead | `v-show` toggles CSS `display`, but `<template>` doesn't render to DOM, so it has no effect |
| Using `v-show` when you need `v-else` | Switch to `v-if`/`v-else` | `v-show` doesn't support `v-else`; you'd need two separate `v-show` directives with opposite conditions |

**Example of v-if/v-for conflict:**

```vue
<!-- ❌ BAD: v-if and v-for on same element -->
<li v-for="item in items" v-if="item.isActive" :key="item.id">
  {{ item.name }}
</li>

<!-- ✅ GOOD: Use computed to filter first -->
<script setup lang="ts">
const activeItems = computed(() => items.value.filter(item => item.isActive))
</script>
<template>
  <li v-for="item in activeItems" :key="item.id">
    {{ item.name }}
  </li>
</template>

<!-- ✅ ALSO GOOD: Use template wrapper -->
<template v-for="item in items" :key="item.id">
  <li v-if="item.isActive">
    {{ item.name }}
  </li>
</template>
```

---

## 6. Mini-Project: Multi-Step Form Wizard

**File Structure:**
```
src/
├── components/
│   └── FormWizard.vue
├── App.vue
└── main.ts
```

**FormWizard.vue:**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

// Step 1: Define the current step state
// We use a number to track which step the user is on (1, 2, or 3)
const currentStep = ref<number>(1)

// Step 2: Define form data that persists across steps
// This reactive object holds all the form values
const formData = ref({
  // Step 1 fields
  firstName: '',
  lastName: '',
  email: '',
  // Step 2 fields
  plan: '' as 'basic' | 'pro' | 'enterprise' | '',
  billingCycle: 'monthly' as 'monthly' | 'yearly',
  // Step 3 is confirmation (no additional fields)
})

// Step 3: Define the total number of steps
// This makes it easy to adjust if we add more steps later
const totalSteps = 3

// Step 4: Create computed properties for step checks
// These make the template cleaner and more readable
const isFirstStep = computed(() => currentStep.value === 1)
const isLastStep = computed(() => currentStep.value === totalSteps)

// Step 5: Compute progress percentage for the progress bar
const progressPercent = computed(() => {
  return ((currentStep.value - 1) / (totalSteps - 1)) * 100
})

// Step 6: Navigation functions
const goToNextStep = () => {
  if (currentStep.value < totalSteps) {
    currentStep.value++
  }
}

const goToPreviousStep = () => {
  if (currentStep.value > 1) {
    currentStep.value--
  }
}

const goToStep = (step: number) => {
  if (step >= 1 && step <= totalSteps) {
    currentStep.value = step
  }
}

// Step 7: Form submission handler
const isSubmitted = ref<boolean>(false)

const submitForm = () => {
  console.log('Form submitted:', formData.value)
  isSubmitted.value = true
}

// Step 8: Reset function to start over
const resetForm = () => {
  currentStep.value = 1
  isSubmitted.value = false
  formData.value = {
    firstName: '',
    lastName: '',
    email: '',
    plan: '',
    billingCycle: 'monthly',
  }
}
</script>

<template>
  <div class="card bg-base-200 shadow-xl max-w-2xl mx-auto">
    <div class="card-body">
      <!-- Header with title -->
      <h2 class="card-title text-2xl mb-4">Account Setup Wizard</h2>
      
      <!-- Progress indicator - always visible -->
      <div class="mb-6">
        <!-- Step indicators -->
        <ul class="steps steps-horizontal w-full">
          <li 
            class="step cursor-pointer"
            :class="{ 'step-primary': currentStep >= 1 }"
            @click="goToStep(1)"
          >
            Personal
          </li>
          <li 
            class="step cursor-pointer"
            :class="{ 'step-primary': currentStep >= 2 }"
            @click="goToStep(2)"
          >
            Plan
          </li>
          <li 
            class="step cursor-pointer"
            :class="{ 'step-primary': currentStep >= 3 }"
            @click="goToStep(3)"
          >
            Confirm
          </li>
        </ul>
        
        <!-- Progress bar -->
        <progress 
          class="progress progress-primary w-full mt-2" 
          :value="progressPercent" 
          max="100"
        ></progress>
      </div>
      
      <!-- 
        CONDITIONAL CONTENT AREA
        Using v-if/v-else-if/v-else to show different steps
        Only ONE step is in the DOM at a time
      -->
      
      <!-- Success State - shown after form submission -->
      <div v-if="isSubmitted" class="text-center py-8">
        <div class="text-6xl mb-4">🎉</div>
        <h3 class="text-xl font-bold text-success mb-2">Registration Complete!</h3>
        <p class="mb-4">
          Welcome, {{ formData.firstName }}! Your {{ formData.plan }} plan is now active.
        </p>
        <button class="btn btn-primary" @click="resetForm">
          Start New Registration
        </button>
      </div>
      
      <!-- Step 1: Personal Information -->
      <div v-else-if="currentStep === 1" class="space-y-4">
        <h3 class="text-lg font-semibold">Personal Information</h3>
        
        <div class="form-control">
          <label class="label">
            <span class="label-text">First Name</span>
          </label>
          <input 
            v-model="formData.firstName"
            type="text" 
            placeholder="John" 
            class="input input-bordered"
          />
        </div>
        
        <div class="form-control">
          <label class="label">
            <span class="label-text">Last Name</span>
          </label>
          <input 
            v-model="formData.lastName"
            type="text" 
            placeholder="Doe" 
            class="input input-bordered"
          />
        </div>
        
        <div class="form-control">
          <label class="label">
            <span class="label-text">Email Address</span>
          </label>
          <input 
            v-model="formData.email"
            type="email" 
            placeholder="john@example.com" 
            class="input input-bordered"
          />
        </div>
      </div>
      
      <!-- Step 2: Plan Selection -->
      <div v-else-if="currentStep === 2" class="space-y-4">
        <h3 class="text-lg font-semibold">Choose Your Plan</h3>
        
        <!-- Plan cards -->
        <div class="grid grid-cols-3 gap-4">
          <div 
            class="card bg-base-100 cursor-pointer transition-all hover:shadow-lg"
            :class="{ 'ring-2 ring-primary': formData.plan === 'basic' }"
            @click="formData.plan = 'basic'"
          >
            <div class="card-body items-center text-center p-4">
              <h4 class="font-bold">Basic</h4>
              <p class="text-2xl font-bold">$9</p>
              <p class="text-sm opacity-70">/month</p>
            </div>
          </div>
          
          <div 
            class="card bg-base-100 cursor-pointer transition-all hover:shadow-lg"
            :class="{ 'ring-2 ring-primary': formData.plan === 'pro' }"
            @click="formData.plan = 'pro'"
          >
            <div class="card-body items-center text-center p-4">
              <div class="badge badge-secondary mb-1">Popular</div>
              <h4 class="font-bold">Pro</h4>
              <p class="text-2xl font-bold">$29</p>
              <p class="text-sm opacity-70">/month</p>
            </div>
          </div>
          
          <div 
            class="card bg-base-100 cursor-pointer transition-all hover:shadow-lg"
            :class="{ 'ring-2 ring-primary': formData.plan === 'enterprise' }"
            @click="formData.plan = 'enterprise'"
          >
            <div class="card-body items-center text-center p-4">
              <h4 class="font-bold">Enterprise</h4>
              <p class="text-2xl font-bold">$99</p>
              <p class="text-sm opacity-70">/month</p>
            </div>
          </div>
        </div>
        
        <!-- Billing cycle toggle -->
        <div class="form-control">
          <label class="label cursor-pointer justify-start gap-4">
            <span class="label-text">Billing Cycle:</span>
            <input 
              type="radio" 
              name="billing" 
              class="radio radio-primary" 
              value="monthly"
              v-model="formData.billingCycle"
            />
            <span>Monthly</span>
            <input 
              type="radio" 
              name="billing" 
              class="radio radio-primary" 
              value="yearly"
              v-model="formData.billingCycle"
            />
            <span>Yearly (Save 20%)</span>
          </label>
        </div>
      </div>
      
      <!-- Step 3: Confirmation -->
      <div v-else class="space-y-4">
        <h3 class="text-lg font-semibold">Confirm Your Details</h3>
        
        <div class="bg-base-100 rounded-lg p-4 space-y-2">
          <div class="flex justify-between">
            <span class="opacity-70">Name:</span>
            <span class="font-medium">{{ formData.firstName }} {{ formData.lastName }}</span>
          </div>
          <div class="flex justify-between">
            <span class="opacity-70">Email:</span>
            <span class="font-medium">{{ formData.email }}</span>
          </div>
          <div class="divider my-2"></div>
          <div class="flex justify-between">
            <span class="opacity-70">Plan:</span>
            <span class="font-medium capitalize">{{ formData.plan || 'Not selected' }}</span>
          </div>
          <div class="flex justify-between">
            <span class="opacity-70">Billing:</span>
            <span class="font-medium capitalize">{{ formData.billingCycle }}</span>
          </div>
        </div>
        
        <!-- Warning if plan not selected -->
        <div v-if="!formData.plan" class="alert alert-warning">
          <span>⚠️ Please go back and select a plan before submitting.</span>
        </div>
      </div>
      
      <!-- Navigation buttons - always visible except after submission -->
      <div v-show="!isSubmitted" class="card-actions justify-between mt-6">
        <button 
          class="btn btn-outline"
          :class="{ 'btn-disabled opacity-50': isFirstStep }"
          :disabled="isFirstStep"
          @click="goToPreviousStep"
        >
          ← Previous
        </button>
        
        <button 
          v-if="!isLastStep"
          class="btn btn-primary"
          @click="goToNextStep"
        >
          Next →
        </button>
        
        <button 
          v-else
          class="btn btn-success"
          :disabled="!formData.plan"
          @click="submitForm"
        >
          Submit ✓
        </button>
      </div>
    </div>
  </div>
</template>
```

**How to Test:**
1. Create a new Vue project: `npm create vue@latest form-wizard-demo`
2. Install Tailwind CSS and DaisyUI following their setup guides
3. Replace the contents of `App.vue` with: `<script setup><FormWizard /></script><template><div class="min-h-screen bg-base-300 p-8"><FormWizard /></div></template>`
4. Copy `FormWizard.vue` to `src/components/`
5. Run `npm run dev` and navigate through the steps
6. Verify: Only one step content is visible at a time, progress updates, form data persists across steps

---

## 7. Implementation Exercises (Progressive)

### Level 1 (Recall): Fill in the Blanks

Complete the conditional rendering logic:

```vue
<script setup lang="ts">
import { ref } from 'vue'

const temperature = ref<number>(25)
</script>

<template>
  <div>
    <input v-model.number="temperature" type="range" min="-20" max="50" />
    <p>Temperature: {{ temperature }}°C</p>
    
    <!-- Fill in the blanks to show appropriate message -->
    <p ___="temperature < 0" class="text-blue-500">
      ❄️ Freezing! Stay warm!
    </p>
    <p ___="temperature < 15" class="text-cyan-500">
      🧥 It's cold. Wear a jacket.
    </p>
    <p ___="temperature < 25" class="text-green-500">
      😊 Nice weather!
    </p>
    <p ___ class="text-red-500">
      🔥 It's hot! Stay hydrated!
    </p>
  </div>
</template>
```

**Solution:**
```vue
<p v-if="temperature < 0" class="text-blue-500">
<p v-else-if="temperature < 15" class="text-cyan-500">
<p v-else-if="temperature < 25" class="text-green-500">
<p v-else class="text-red-500">
```

---

### Level 2 (Apply): Independent Coding Tasks

**Exercise 2.1:** Create a `PasswordStrength.vue` component that shows different strength indicators (Weak/Medium/Strong) based on password length. Use `v-if`/`v-else-if`/`v-else` with appropriate styling.

**Requirements:**
- Password < 6 characters → Red "Weak" badge
- Password 6-10 characters → Yellow "Medium" badge  
- Password > 10 characters → Green "Strong" badge
- Empty password → Show nothing

**Exercise 2.2:** Create a `FeatureFlag.vue` component that uses `v-show` for a frequently-toggled tooltip and `v-if` for a modal dialog. Demonstrate why `v-show` is better for the tooltip (toggles on hover) and `v-if` is better for the modal (rarely opened).

---

### Level 3 (Debug): Fix the Broken Code

Find and fix the bugs in this code:

```vue
<script setup lang="ts">
import { ref } from 'vue'

const userStatus = ref<'online' | 'away' | 'offline'>('online')
</script>

<template>
  <div class="p-4">
    <select v-model="userStatus" class="select select-bordered">
      <option value="online">Online</option>
      <option value="away">Away</option>
      <option value="offline">Offline</option>
    </select>
    
    <!-- Bug 1: Something is wrong with the v-if chain -->
    <span v-if="userStatus === 'online'" class="badge badge-success">
      🟢 Online
    </span>
    <p>Some unrelated text here</p>
    <span v-else-if="userStatus === 'away'" class="badge badge-warning">
      🟡 Away
    </span>
    <span v-else class="badge badge-error">
      🔴 Offline
    </span>
    
    <!-- Bug 2: This won't work as expected -->
    <template v-show="userStatus === 'online'">
      <p>You can receive messages</p>
      <p>You appear active to others</p>
    </template>
  </div>
</template>
```

**Solution:**

```vue
<script setup lang="ts">
import { ref } from 'vue'

const userStatus = ref<'online' | 'away' | 'offline'>('online')
</script>

<template>
  <div class="p-4">
    <select v-model="userStatus" class="select select-bordered">
      <option value="online">Online</option>
      <option value="away">Away</option>
      <option value="offline">Offline</option>
    </select>
    
    <!-- Fix 1: Remove the element between v-if and v-else-if -->
    <span v-if="userStatus === 'online'" class="badge badge-success">
      🟢 Online
    </span>
    <span v-else-if="userStatus === 'away'" class="badge badge-warning">
      🟡 Away
    </span>
    <span v-else class="badge badge-error">
      🔴 Offline
    </span>
    
    <p>Some unrelated text here</p>
    
    <!-- Fix 2: v-show doesn't work on <template>, use v-if instead -->
    <template v-if="userStatus === 'online'">
      <p>You can receive messages</p>
      <p>You appear active to others</p>
    </template>
  </div>
</template>
```

**Bugs explained:**
1. **Bug 1:** There was a `<p>` element between the `v-if` and `v-else-if`, breaking the chain. The `v-else-if` couldn't find its corresponding `v-if`.
2. **Bug 2:** `v-show` does not work on `<template>` elements because `<template>` doesn't render an actual DOM element, so there's nothing to apply `display: none` to. Use `v-if` instead.

---

*Reply 'next' for Lesson 10.1 (Theory).*
