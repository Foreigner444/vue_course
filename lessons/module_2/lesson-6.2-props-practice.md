# Lesson 6.2: Component Communication: Passing Props — Practice & Application (Передача Props)
---
## 1. Lesson Metadata
| Field | Value |
|:--- |:--- |
| **Lesson Number** | 6.2 of 25 (Part 2 of 2) |
| **Topic** | Passing Props / Передача Props |
| **Continues From** | Lesson 6.1 |
| **Duration** | 30-35 minutes |
**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Create components that receive and display typed props
2. **Analyze:** Compare different prop patterns and debug common prop-related issues
---
## 2. Quick Recap
From Lesson 6.1, remember:
- **Props flow one-way**: Parent → Child only (read-only in child)
- **`defineProps<T>()`**: The modern way to declare typed props
- **`:propName` vs `propName`**: Dynamic binding (JS expression) vs static string
- **`withDefaults()`**: Provides default values for optional props
---
## 3. The Variable Frame & Complexity Scale
### Basic Form: Simple Required Props
```vue
<script setup lang="ts">
// Simplest usage: required props with primitive types
const props = defineProps<{
  message: string
  count: number
  isActive: boolean
}>()
</script>
<template>
  <div>{{ message }} - {{ count }} - {{ isActive }}</div>
</template>
```
### With Options: Optional Props and Defaults
```vue
<script setup lang="ts">
// Adding optional parameters with default values
interface Props {
  title: string
  description?: string
  maxItems?: number
  variant?: 'primary' | 'secondary' | 'danger'
}
const props = withDefaults(defineProps<Props>(), {
  description: 'No description available',
  maxItems: 10,
  variant: 'primary'
})
</script>
<template>
  <div :class="`card card-${variant}`">
    <h3>{{ title }}</h3>
    <p>{{ description }}</p>
    <small>Max: {{ maxItems }}</small>
  </div>
</template>
```
### Advanced Form: Complex Types and Objects
```vue
<script setup lang="ts">
// Complex usage with objects, arrays, and union types
interface User {
  id: number
  name: string
  email: string
  avatar?: string
}
interface Props {
  user: User
  permissions: string[]
  status: 'online' | 'offline' | 'away'
  metadata?: Record<string, unknown>
  onError?: (error: Error) => void
}
const props = withDefaults(defineProps<Props>(), {
  metadata: () => ({}),
  onError: () => console.error
})
// Using props in script
const displayName = computed(() => {
  return props.user.name.toUpperCase()
})
</script>
<template>
  <article class="user-card">
    <img :src="user.avatar ?? '/default-avatar.png'" :alt="user.name" />
    <h2>{{ displayName }}</h2>
    <span :class="status">{{ status }}</span>
    <ul>
      <li v-for="permission in permissions" :key="permission">
        {{ permission }}
      </li>
    </ul>
  </article>
</template>
```
---
## 4. Micro-Implementation Drills
### Drill 1 (Basic): Simple Greeting Component
Create a component that displays a personalized greeting.
```vue
<!-- Greeting.vue — Starter Code -->
<script setup lang="ts">
// TODO: Define props to accept:
// - name (string, required)
// - timeOfDay (string, required) — e.g., "morning", "afternoon", "evening"
</script>
<template>
  <!-- TODO: Display "Good [timeOfDay], [name]!" -->
  <p></p>
</template>
```
**Expected Usage:**
```vue
<Greeting name="Alice" timeOfDay="morning" />
<!-- Output: Good morning, Alice! -->
```
---
### Drill 2 (Variation): Product Card with Optional Props
Create a product card with some optional fields.
```vue
<!-- ProductCard.vue — Starter Code -->
<script setup lang="ts">
// TODO: Define props:
// - name (string, required)
// - price (number, required)
// - description (string, optional, default: "No description")
// - inStock (boolean, optional, default: true)
// - imageUrl (string, optional, default: "/placeholder.jpg")
</script>
<template>
  <!-- TODO: Display product information -->
  <!-- Show "Out of Stock" badge if inStock is false -->
  <div class="card bg-base-100 shadow-xl">
    <figure>
      <!-- Image here -->
    </figure>
    <div class="card-body">
      <!-- Name, description, price here -->
      <!-- Stock status badge -->
    </div>
  </div>
</template>
```
---
### Drill 3 (Combination): Stats Card with Computed
Combine props with `computed()` from Lesson 4.
```vue
<!-- StatsCard.vue — Starter Code -->
<script setup lang="ts">
import { computed } from 'vue'
// TODO: Define props:
// - label (string, required)
// - current (number, required)
// - previous (number, required)
// TODO: Create a computed property `percentageChange` that calculates
// the percentage difference between current and previous
// Formula: ((current - previous) / previous) * 100
// TODO: Create a computed property `trend` that returns
// "up" if percentageChange > 0, "down" if < 0, "neutral" if === 0
</script>
<template>
  <div class="stat">
    <div class="stat-title">{{ label }}</div>
    <div class="stat-value">{{ current }}</div>
    <div class="stat-desc" :class="trend">
      <!-- TODO: Show percentage with arrow icon based on trend -->
      <!-- Example: "↑ 25%" or "↓ 10%" or "→ 0%" -->
    </div>
  </div>
</template>
```
---
### Drill 4 (Edge Case): Handling Object Props Safely
Handle potentially undefined nested properties.
```vue
<!-- UserProfile.vue — Starter Code -->
<script setup lang="ts">
interface Address {
  street: string
  city: string
  country: string
}
interface User {
  id: number
  name: string
  address?: Address
  socialLinks?: {
    twitter?: string
    github?: string
  }
}
// TODO: Define props:
// - user (User type, required)
// - showAddress (boolean, optional, default: true)
// TODO: Create a computed property `formattedAddress` that safely
// returns "street, city, country" or "Address not provided"
// TODO: Create a computed property `hasSocialLinks` that returns
// true if at least one social link exists
</script>
<template>
  <div class="card">
    <h2>{{ user.name }}</h2>
    
    <!-- TODO: Only show address section if showAddress is true -->
    <!-- Handle case where user.address might be undefined -->
    
    <!-- TODO: Only show social links if hasSocialLinks is true -->
    <!-- Safely access optional nested properties -->
  </div>
</template>
```
---
## 5. Common Pitfalls & Anti-Patterns
| ❌ Common Mistake | ✅ Correct Approach | Why It Matters |
|:--- |:--- |:--- |
| Mutating props directly: `props.count++` | Emit an event to parent: `emit('update', newValue)` | Props are read-only; mutation causes errors and breaks reactivity chain |
| Forgetting `:` for dynamic props: `<Child count="myVar">` | Use v-bind: `<Child :count="myVar">` | Without `:`, Vue passes the literal string "myVar", not the variable's value |
| Destructuring props loses reactivity: `const { name } = props` | Access via `props.name` or use `toRefs()` | Destructured primitives aren't reactive; changes won't trigger updates |
| Object prop defaults as direct values: `items: []` | Use factory function: `items: () => []` | Objects/arrays are shared across instances; factory creates fresh copy |
### Reactivity Loss Example:
```vue
<script setup lang="ts">
const props = defineProps<{ count: number; user: { name: string } }>()
// ❌ WRONG: Loses reactivity for primitives
const { count } = props
// `count` is now a static value, won't update when parent changes
// ✅ CORRECT: Maintain reactivity
import { toRefs } from 'vue'
const { count } = toRefs(props)
// `count` is now a Ref that stays in sync with parent
// ✅ ALSO CORRECT: Just use props directly
// In template: {{ props.count }}
// In script: props.count
</script>
```
---
## 6. Mini-Project: Profile Header Component
Build a reusable profile header for the social media app scenario.
**File Structure:**
```
src/
├── components/
│   └── ProfileHeader.vue
├── App.vue
└── main.ts
```
**ProfileHeader.vue:**
```vue
<script setup lang="ts">
import { computed } from 'vue'
interface SocialLinks {
  twitter?: string
  github?: string
  linkedin?: string
}
interface ProfileProps {
  name: string
  username: string
  avatarUrl?: string
  bio?: string
  followers: number
  following: number
  isVerified?: boolean
  socialLinks?: SocialLinks
}
const props = withDefaults(defineProps<ProfileProps>(), {
  avatarUrl: '/default-avatar.png',
  bio: 'This user has not added a bio yet.',
  isVerified: false,
  socialLinks: () => ({})
})
const formattedFollowers = computed(() => {
  if (props.followers >= 1_000_000) {
    return `${(props.followers / 1_000_000).toFixed(1)}M`
  }
  if (props.followers >= 1_000) {
    return `${(props.followers / 1_000).toFixed(1)}K`
  }
  return props.followers.toString()
})
const formattedFollowing = computed(() => {
  if (props.following >= 1_000) {
    return `${(props.following / 1_000).toFixed(1)}K`
  }
  return props.following.toString()
})
const hasSocialLinks = computed(() => {
  return Object.values(props.socialLinks ?? {}).some(link => !!link)
})
</script>
<template>
  <div class="card bg-base-100 shadow-xl">
    <figure class="px-10 pt-10">
      <div class="avatar">
        <div class="w-24 rounded-full ring ring-primary ring-offset-base-100 ring-offset-2">
          <img :src="avatarUrl" :alt="`${name}'s avatar`" />
        </div>
      </div>
    </figure>
    
    <div class="card-body items-center text-center">
      <h2 class="card-title">
        {{ name }}
        <span v-if="isVerified" class="badge badge-primary badge-sm">
          ✓ Verified
        </span>
      </h2>
      
      <p class="text-base-content/60">@{{ username }}</p>
      
      <p class="text-sm">{{ bio }}</p>
      
      <div class="stats stats-horizontal shadow mt-4">
        <div class="stat place-items-center">
          <div class="stat-title">Followers</div>
          <div class="stat-value text-lg">{{ formattedFollowers }}</div>
        </div>
        <div class="stat place-items-center">
          <div class="stat-title">Following</div>
          <div class="stat-value text-lg">{{ formattedFollowing }}</div>
        </div>
      </div>
      
      <div v-if="hasSocialLinks" class="flex gap-2 mt-4">
        <a 
          v-if="socialLinks?.twitter" 
          :href="`https://twitter.com/${socialLinks.twitter}`"
          class="btn btn-circle btn-ghost btn-sm"
          target="_blank"
        >
          𝕏
        </a>
        <a 
          v-if="socialLinks?.github" 
          :href="`https://github.com/${socialLinks.github}`"
          class="btn btn-circle btn-ghost btn-sm"
          target="_blank"
        >
          GH
        </a>
        <a 
          v-if="socialLinks?.linkedin" 
          :href="`https://linkedin.com/in/${socialLinks.linkedin}`"
          class="btn btn-circle btn-ghost btn-sm"
          target="_blank"
        >
          in
        </a>
      </div>
    </div>
  </div>
</template>
```
**App.vue:**
```vue
<script setup lang="ts">
import { ref } from 'vue'
import ProfileHeader from './components/ProfileHeader.vue'
const user = ref({
  name: 'Alice Johnson',
  username: 'alicej',
  avatarUrl: 'https://i.pravatar.cc/150?u=alice',
  bio: 'Full-stack developer passionate about Vue.js and TypeScript. Building the future, one component at a time.',
  followers: 12500,
  following: 340,
  isVerified: true,
  socialLinks: {
    twitter: 'alicej_dev',
    github: 'alicej'
  }
})
const minimalUser = ref({
  name: 'Bob Smith',
  username: 'bobsmith',
  followers: 45,
  following: 120
})
</script>
<template>
  <div class="container mx-auto p-8">
    <h1 class="text-3xl font-bold mb-8">Profile Headers Demo</h1>
    
    <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
      <div>
        <h2 class="text-xl mb-4">Full Profile</h2>
        <ProfileHeader
          :name="user.name"
          :username="user.username"
          :avatar-url="user.avatarUrl"
          :bio="user.bio"
          :followers="user.followers"
          :following="user.following"
          :is-verified="user.isVerified"
          :social-links="user.socialLinks"
        />
      </div>
      
      <div>
        <h2 class="text-xl mb-4">Minimal Profile (using defaults)</h2>
        <ProfileHeader
          :name="minimalUser.name"
          :username="minimalUser.username"
          :followers="minimalUser.followers"
          :following="minimalUser.following"
        />
      </div>
    </div>
  </div>
</template>
```
**How to Test:**
1. Run `npm create vite@latest props-demo -- --template vue-ts`
2. Install DaisyUI: `npm install -D tailwindcss postcss autoprefixer daisyui`
3. Configure Tailwind and DaisyUI in `tailwind.config.js`
4. Replace `App.vue` and create `ProfileHeader.vue` with the code above
5. Run `npm run dev` and verify both profile cards render correctly
6. Verify the minimal profile shows default values for missing props
---
## 7. Implementation Exercises (Progressive)
### Level 1 (Recall): Fill in the Blanks
```vue
<script setup lang="ts">
// Fill in the blanks to create a Button component with props
interface _____ {
  label: string
  variant?: 'primary' | 'secondary'
  disabled?: _____
}
const props = _____(defineProps<Props>(), {
  variant: '_____',
  disabled: false
})
</script>
<template>
  <button 
    :class="['btn', `btn-${_____}`]"
    :disabled="_____"
  >
    {{ _____ }}
  </button>
</template>
```
---
### Level 2 (Apply): Independent Coding Tasks
**Exercise 2.1: Alert Component**
Create an `AlertMessage.vue` component with these props:
- `type`: 'info' | 'success' | 'warning' | 'error' (required)
- `title`: string (required)
- `message`: string (optional, default: '')
- `dismissible`: boolean (optional, default: false)
Use DaisyUI's alert component classes (`alert`, `alert-info`, etc.).
**Exercise 2.2: Data Table Row**
Create a `TableRow.vue` component that accepts:
- `item`: object with `id`, `name`, `email`, `status` (required)
- `striped`: boolean (optional, default: false)
- `highlighted`: boolean (optional, default: false)
Render a `<tr>` element with appropriate styling.
---
### Level 3 (Debug): Fix the Broken Code
```vue
<!-- BrokenUserCard.vue — Find and fix 4 bugs -->
<script setup lang="ts">
import { computed } from 'vue'
interface Props {
  name: string
  age: number
  hobbies: string[]
  isAdmin: boolean
}
// Bug 1: Something wrong with defaults
const props = withDefaults(defineProps<Props>(), {
  hobbies: [],
  isAdmin: false
})
// Bug 2: Reactivity issue
const { name, age } = props
const birthYear = computed(() => {
  return new Date().getFullYear() - age
})
</script>
<template>
  <div class="card">
    <!-- Bug 3: Something wrong with binding -->
    <h2>name</h2>
    <p>Age: {{ age }} (Born: {{ birthYear }})</p>
    
    <!-- Bug 4: Something wrong with list rendering -->
    <ul>
      <li v-for="hobby in hobbies">{{ hobby }}</li>
    </ul>
    
    <span v-if="isAdmin" class="badge">Admin</span>
  </div>
</template>
```
---
### Self-Check: Reference Implementations
**Drill 1 Solution:**
```vue
<script setup lang="ts">
const props = defineProps<{
  name: string
  timeOfDay: string
}>()
</script>
<template>
  <p>Good {{ timeOfDay }}, {{ name }}!</p>
</template>
```
**Level 1 Solution:**
```vue
<script setup lang="ts">
interface Props {
  label: string
  variant?: 'primary' | 'secondary'
  disabled?: boolean
}
const props = withDefaults(defineProps<Props>(), {
  variant: 'primary',
  disabled: false
})
</script>
<template>
  <button 
    :class="['btn', `btn-${variant}`]"
    :disabled="disabled"
  >
    {{ label }}
  </button>
</template>
```
**Level 3 Bugs Fixed:**
1. `hobbies: []` → `hobbies: () => []` (array needs factory function)
2. `const { name, age } = props` → use `props.name` and `props.age` directly, or `toRefs(props)`
3. `<h2>name</h2>` → `<h2>{{ name }}</h2>` (missing interpolation)
4. `v-for="hobby in hobbies"` → `v-for="hobby in hobbies" :key="hobby"` (missing key)
---
**Reply 'next' for Lesson 7.1 (Theory).**
