# Lesson 8.2: Styling with Tailwind & DaisyUI — Practice & Application (Стилизация с Tailwind и DaisyUI)

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 8.2 of 25 (Part 2 of 2) |
| **Topic** | Styling with Tailwind & DaisyUI / Стилизация с Tailwind и DaisyUI |
| **Continues From** | Lesson 8.1 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Build styled Vue components using Tailwind utilities and DaisyUI components
2. **Analyze:** Compare utility-first patterns, debug styling issues, and choose appropriate classes for different scenarios

---

## 2. Quick Recap

From Lesson 8.1, remember:

- **Tailwind CSS** uses utility classes (like `p-4`, `text-center`, `bg-blue-500`) applied directly in templates—no custom CSS files needed
- **DaisyUI** provides pre-built component classes (like `btn`, `card`, `badge`) that work with Tailwind and support theming
- **Responsive design** uses breakpoint prefixes (`sm:`, `md:`, `lg:`) in a mobile-first approach
- **State styling** uses prefixes like `hover:`, `focus:`, and `dark:` for interactive and theme-aware styles

---

## 3. The Variable Frame & Complexity Scale

### Basic Form: Simple Utility Styling

```vue
<script setup lang="ts">
// Minimal component—just applying basic utilities
</script>

<template>
  <!-- Simple box with padding, background, and rounded corners -->
  <div class="p-4 bg-gray-100 rounded">
    <p class="text-gray-700">Hello, Tailwind!</p>
  </div>
</template>
```

### With Options: Adding DaisyUI Components

```vue
<script setup lang="ts">
// Using DaisyUI's semantic component classes
</script>

<template>
  <!-- DaisyUI alert component with info style -->
  <div class="alert alert-info">
    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" 
         class="h-6 w-6 shrink-0 stroke-current">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" 
            d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
    </svg>
    <span>New features are available!</span>
  </div>
</template>
```

### Advanced Form: Responsive + States + DaisyUI + Tailwind Combined

```vue
<script setup lang="ts">
import { ref } from 'vue'

// Component with reactive state and full styling
const isHovered = ref(false)
</script>

<template>
  <!-- 
    Complex card combining:
    - DaisyUI card component
    - Tailwind responsive utilities
    - State-based styling (hover)
    - Dark mode support
  -->
  <div 
    class="
      card 
      bg-base-100 
      shadow-md
      w-full
      md:w-96
      transition-all
      duration-300
      hover:shadow-xl
      hover:-translate-y-1
      dark:bg-base-200
    "
    @mouseenter="isHovered = true"
    @mouseleave="isHovered = false"
  >
    <!-- Card image with responsive aspect ratio -->
    <figure class="h-48 md:h-56 overflow-hidden">
      <img 
        src="/api/placeholder/400/300" 
        alt="Card image"
        class="w-full h-full object-cover transition-transform duration-300"
        :class="{ 'scale-110': isHovered }"
      />
    </figure>
    
    <div class="card-body p-4 md:p-6">
      <!-- Title with responsive sizing -->
      <h2 class="card-title text-lg md:text-xl">
        Advanced Card
        <!-- DaisyUI badge -->
        <div class="badge badge-secondary">NEW</div>
      </h2>
      
      <!-- Description with line clamp -->
      <p class="text-sm text-base-content/70 line-clamp-2">
        This card demonstrates combining DaisyUI components with 
        Tailwind utilities for responsive, interactive styling.
      </p>
      
      <!-- Actions with responsive layout -->
      <div class="card-actions justify-end mt-4">
        <button class="btn btn-outline btn-sm md:btn-md">Details</button>
        <button class="btn btn-primary btn-sm md:btn-md">Buy Now</button>
      </div>
    </div>
  </div>
</template>
```

---

## 4. Micro-Implementation Drills

### Drill 1 (Basic): Style a Simple Badge

Create a status badge using Tailwind utilities only (no DaisyUI).

```vue
<script setup lang="ts">
// TODO: No script needed for this drill
</script>

<template>
  <!-- 
    TODO: Create a badge that displays "Active"
    Requirements:
    - Green background (bg-green-100)
    - Green text (text-green-800)
    - Small padding (px-2 py-1)
    - Rounded corners (rounded-full)
    - Small, semibold text (text-xs font-semibold)
  -->
  <span class="_____ _____ _____ _____ _____ _____">
    Active
  </span>
</template>
```

**Solution:**
```vue
<template>
  <span class="px-2 py-1 bg-green-100 text-green-800 text-xs font-semibold rounded-full">
    Active
  </span>
</template>
```

---

### Drill 2 (Variation): Create a DaisyUI Button Group

Use DaisyUI button classes with different variants.

```vue
<script setup lang="ts">
// TODO: No script needed for this drill
</script>

<template>
  <!-- 
    TODO: Create a button group with 3 buttons:
    1. Primary button with text "Save"
    2. Secondary button with text "Preview"  
    3. Ghost button with text "Cancel"
    
    Hint: Use btn, btn-primary, btn-secondary, btn-ghost classes
  -->
  <div class="flex gap-2">
    <button class="_____">Save</button>
    <button class="_____">Preview</button>
    <button class="_____">Cancel</button>
  </div>
</template>
```

**Solution:**
```vue
<template>
  <div class="flex gap-2">
    <button class="btn btn-primary">Save</button>
    <button class="btn btn-secondary">Preview</button>
    <button class="btn btn-ghost">Cancel</button>
  </div>
</template>
```

---

### Drill 3 (Combination): Responsive Navigation Bar

Combine layout utilities with DaisyUI for a responsive navbar.

```vue
<script setup lang="ts">
// TODO: No script needed for this drill
</script>

<template>
  <!-- 
    TODO: Create a navbar that:
    - Uses DaisyUI navbar component
    - Has a brand name on the left
    - Has navigation links on the right
    - Links are hidden on mobile, visible on md+ screens
    - Has a hamburger menu button visible only on mobile
    
    Classes to use:
    - navbar, bg-base-100, shadow-md
    - flex-1
    - hidden md:flex (for desktop nav)
    - md:hidden (for mobile menu button)
    - btn btn-ghost btn-square
  -->
  <nav class="navbar _____ _____">
    <!-- Brand -->
    <div class="_____">
      <a class="btn btn-ghost text-xl">MyApp</a>
    </div>
    
    <!-- Desktop Navigation (hidden on mobile) -->
    <div class="_____ gap-2">
      <a class="btn btn-ghost">Home</a>
      <a class="btn btn-ghost">About</a>
      <a class="btn btn-ghost">Contact</a>
    </div>
    
    <!-- Mobile Menu Button (hidden on desktop) -->
    <div class="_____">
      <button class="btn btn-ghost btn-square">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" 
             viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" 
                d="M4 6h16M4 12h16M4 18h16" />
        </svg>
      </button>
    </div>
  </nav>
</template>
```

**Solution:**
```vue
<template>
  <nav class="navbar bg-base-100 shadow-md">
    <div class="flex-1">
      <a class="btn btn-ghost text-xl">MyApp</a>
    </div>
    
    <div class="hidden md:flex gap-2">
      <a class="btn btn-ghost">Home</a>
      <a class="btn btn-ghost">About</a>
      <a class="btn btn-ghost">Contact</a>
    </div>
    
    <div class="md:hidden">
      <button class="btn btn-ghost btn-square">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" 
             viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" 
                d="M4 6h16M4 12h16M4 18h16" />
        </svg>
      </button>
    </div>
  </nav>
</template>
```

---

### Drill 4 (Edge Case): Conditional Styling with Dynamic Classes

Handle conditional styling based on component state.

```vue
<script setup lang="ts">
import { ref } from 'vue'

// Status can be: 'success', 'warning', 'error', 'info'
const status = ref<'success' | 'warning' | 'error' | 'info'>('success')

// TODO: Create a computed or method to return the appropriate classes
// based on the status value
</script>

<template>
  <!-- 
    TODO: Create a status card that changes colors based on status:
    - success: bg-green-50, border-green-500, text-green-700
    - warning: bg-yellow-50, border-yellow-500, text-yellow-700
    - error: bg-red-50, border-red-500, text-red-700
    - info: bg-blue-50, border-blue-500, text-blue-700
    
    The card should have:
    - Left border (border-l-4)
    - Padding (p-4)
    - Rounded corners (rounded-r)
  -->
  <div 
    class="border-l-4 p-4 rounded-r"
    :class="_____"
  >
    <p class="font-semibold">Status: {{ status }}</p>
    <p>This is a status message.</p>
  </div>
  
  <!-- Buttons to change status for testing -->
  <div class="flex gap-2 mt-4">
    <button class="btn btn-success btn-sm" @click="status = 'success'">Success</button>
    <button class="btn btn-warning btn-sm" @click="status = 'warning'">Warning</button>
    <button class="btn btn-error btn-sm" @click="status = 'error'">Error</button>
    <button class="btn btn-info btn-sm" @click="status = 'info'">Info</button>
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

const status = ref<'success' | 'warning' | 'error' | 'info'>('success')

const statusClasses = computed(() => {
  const classMap = {
    success: 'bg-green-50 border-green-500 text-green-700',
    warning: 'bg-yellow-50 border-yellow-500 text-yellow-700',
    error: 'bg-red-50 border-red-500 text-red-700',
    info: 'bg-blue-50 border-blue-500 text-blue-700'
  }
  return classMap[status.value]
})
</script>

<template>
  <div 
    class="border-l-4 p-4 rounded-r"
    :class="statusClasses"
  >
    <p class="font-semibold">Status: {{ status }}</p>
    <p>This is a status message.</p>
  </div>
  
  <div class="flex gap-2 mt-4">
    <button class="btn btn-success btn-sm" @click="status = 'success'">Success</button>
    <button class="btn btn-warning btn-sm" @click="status = 'warning'">Warning</button>
    <button class="btn btn-error btn-sm" @click="status = 'error'">Error</button>
    <button class="btn btn-info btn-sm" @click="status = 'info'">Info</button>
  </div>
</template>
```

---

## 5. Common Pitfalls & Anti-Patterns

| ❌ Common Mistake | ✅ Correct Approach | Why It Matters |
|:--- |:--- |:--- |
| Using arbitrary values everywhere: `class="w-[327px] mt-[13px]"` | Use Tailwind's scale: `class="w-80 mt-3"` | Arbitrary values bypass the design system, leading to inconsistent spacing and harder maintenance. Use the spacing scale (4, 8, 12, 16...) for consistency. |
| Forgetting mobile-first: `class="lg:text-sm text-lg"` | Mobile-first: `class="text-lg lg:text-sm"` | Tailwind is mobile-first. Base classes apply to ALL screens; prefixes apply to that breakpoint AND UP. The wrong order leads to unexpected mobile styles. |
| Mixing Tailwind colors with DaisyUI theme colors: `class="btn bg-blue-500"` | Use DaisyUI's semantic colors: `class="btn btn-primary"` | Hardcoded colors ignore the active theme. If a user switches to dark mode or another theme, `bg-blue-500` won't adapt, but `btn-primary` will. |

### Additional Anti-Patterns to Avoid

**❌ Creating custom CSS for things Tailwind already provides:**
```css
/* Don't do this */
.my-card {
  padding: 1rem;
  border-radius: 0.5rem;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
}
```

**✅ Just use utilities:**
```vue
<div class="p-4 rounded-lg shadow-md">
```

---

**❌ Overriding DaisyUI components with too many custom utilities:**
```vue
<!-- Don't fight the framework -->
<button class="btn btn-primary px-8 py-4 rounded-none shadow-2xl text-2xl">
```

**✅ Use DaisyUI's built-in modifiers or create a custom theme:**
```vue
<!-- Use size modifiers -->
<button class="btn btn-primary btn-lg">
```

---

## 6. Mini-Project (Putting It Together)

Build a **Team Member Profile Card** component that displays employee information with responsive design and interactive hover states.

### File Structure

```
src/
├── components/
│   └── ProfileCard.vue
├── App.vue
└── main.ts
```

### ProfileCard.vue

```vue
<script setup lang="ts">
// ============================================================
// ProfileCard.vue
// A reusable team member profile card component
// Demonstrates: Tailwind utilities, DaisyUI components,
//               responsive design, hover states, props
// ============================================================

// ------------------------------------------------------------
// IMPORTS
// ------------------------------------------------------------
// No imports needed for basic styling!
// Tailwind classes work directly in the template

// ------------------------------------------------------------
// PROPS DEFINITION
// ------------------------------------------------------------
// Using TypeScript interface for type-safe props
// This defines what data the component accepts from its parent

interface Props {
  name: string
  role: string
  department: string
  avatarUrl: string
  email: string
  socialLinks?: {
    twitter?: string
    linkedin?: string
    github?: string
  }
}

// defineProps receives the Props interface
// withDefaults allows us to set default values for optional props
const props = withDefaults(defineProps<Props>(), {
  socialLinks: () => ({})
})
</script>

<template>
  <!-- ========================================================
       MAIN CARD CONTAINER
       
       DaisyUI classes:
       - card: Base card component styling
       - bg-base-100: Background color from theme (adapts to dark mode)
       - shadow-xl: Large shadow for depth
       
       Tailwind classes:
       - w-full: Full width on mobile
       - md:w-80: Fixed 320px width on medium screens and up
       - transition-all duration-300: Smooth transitions for all properties
       - hover:shadow-2xl: Larger shadow on hover
       - hover:-translate-y-2: Lift effect on hover
       ======================================================== -->
  <div class="
    card 
    bg-base-100 
    shadow-xl
    w-full
    md:w-80
    transition-all
    duration-300
    hover:shadow-2xl
    hover:-translate-y-2
  ">
    <!-- ======================================================
         AVATAR SECTION
         
         - figure: Semantic HTML for images
         - pt-6: Padding top for spacing
         - px-6: Horizontal padding
         ====================================================== -->
    <figure class="pt-6 px-6">
      <!-- ====================================================
           AVATAR CONTAINER
           
           DaisyUI classes:
           - avatar: Avatar component wrapper
           
           Tailwind classes:
           - w-24 h-24: 96px square size
           - rounded-full: Circular shape
           - ring ring-primary ring-offset-base-100 ring-offset-2:
             Creates a decorative ring around the avatar using
             the theme's primary color
           - overflow-hidden: Clips the image to the rounded shape
           ==================================================== -->
      <div class="
        avatar
        w-24 
        h-24 
        rounded-full 
        ring 
        ring-primary 
        ring-offset-base-100 
        ring-offset-2
        overflow-hidden
      ">
        <!-- Avatar image with cover fit -->
        <img 
          :src="props.avatarUrl" 
          :alt="`${props.name}'s avatar`"
          class="w-full h-full object-cover"
        />
      </div>
    </figure>

    <!-- ======================================================
         CARD BODY
         
         DaisyUI classes:
         - card-body: Automatic padding and flex layout
         - items-center: Center children horizontally
         - text-center: Center text alignment
         ====================================================== -->
    <div class="card-body items-center text-center">
      
      <!-- ==================================================
           NAME AND ROLE
           
           - card-title: DaisyUI title styling
           - text-lg md:text-xl: Responsive font size
           - mb-1: Small bottom margin
           ================================================== -->
      <h2 class="card-title text-lg md:text-xl mb-1">
        {{ props.name }}
      </h2>
      
      <!-- Role with muted color -->
      <p class="text-base-content/70 text-sm">
        {{ props.role }}
      </p>
      
      <!-- ==================================================
           DEPARTMENT BADGE
           
           DaisyUI classes:
           - badge: Base badge component
           - badge-primary: Primary theme color
           - badge-outline: Outlined style (not filled)
           
           Tailwind classes:
           - mt-2: Top margin for spacing
           ================================================== -->
      <div class="badge badge-primary badge-outline mt-2">
        {{ props.department }}
      </div>
      
      <!-- ==================================================
           EMAIL LINK
           
           Tailwind classes:
           - mt-3: Top margin
           - text-sm: Small font size
           - text-primary: Primary theme color for link
           - hover:underline: Underline on hover
           - transition-colors: Smooth color transition
           ================================================== -->
      <a 
        :href="`mailto:${props.email}`"
        class="mt-3 text-sm text-primary hover:underline transition-colors"
      >
        {{ props.email }}
      </a>
      
      <!-- ==================================================
           SOCIAL LINKS
           
           - divider: DaisyUI horizontal line
           - flex gap-4: Horizontal layout with spacing
           - mt-2: Top margin
           ================================================== -->
      <div class="divider my-2"></div>
      
      <div class="flex gap-4 mt-2">
        <!-- Twitter Link (only shown if provided) -->
        <a 
          v-if="props.socialLinks?.twitter"
          :href="props.socialLinks.twitter"
          target="_blank"
          rel="noopener noreferrer"
          class="btn btn-ghost btn-circle btn-sm hover:btn-primary transition-all"
          aria-label="Twitter"
        >
          <!-- Twitter/X Icon (SVG) -->
          <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
            <path d="M18.244 2.25h3.308l-7.227 8.26 8.502 11.24H16.17l-5.214-6.817L4.99 21.75H1.68l7.73-8.835L1.254 2.25H8.08l4.713 6.231zm-1.161 17.52h1.833L7.084 4.126H5.117z"/>
          </svg>
        </a>
        
        <!-- LinkedIn Link -->
        <a 
          v-if="props.socialLinks?.linkedin"
          :href="props.socialLinks.linkedin"
          target="_blank"
          rel="noopener noreferrer"
          class="btn btn-ghost btn-circle btn-sm hover:btn-primary transition-all"
          aria-label="LinkedIn"
        >
          <!-- LinkedIn Icon -->
          <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
            <path d="M20.447 20.452h-3.554v-5.569c0-1.328-.027-3.037-1.852-3.037-1.853 0-2.136 1.445-2.136 2.939v5.667H9.351V9h3.414v1.561h.046c.477-.9 1.637-1.85 3.37-1.85 3.601 0 4.267 2.37 4.267 5.455v6.286zM5.337 7.433c-1.144 0-2.063-.926-2.063-2.065 0-1.138.92-2.063 2.063-2.063 1.14 0 2.064.925 2.064 2.063 0 1.139-.925 2.065-2.064 2.065zm1.782 13.019H3.555V9h3.564v11.452zM22.225 0H1.771C.792 0 0 .774 0 1.729v20.542C0 23.227.792 24 1.771 24h20.451C23.2 24 24 23.227 24 22.271V1.729C24 .774 23.2 0 22.222 0h.003z"/>
          </svg>
        </a>
        
        <!-- GitHub Link -->
        <a 
          v-if="props.socialLinks?.github"
          :href="props.socialLinks.github"
          target="_blank"
          rel="noopener noreferrer"
          class="btn btn-ghost btn-circle btn-sm hover:btn-primary transition-all"
          aria-label="GitHub"
        >
          <!-- GitHub Icon -->
          <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
            <path d="M12 0c-6.626 0-12 5.373-12 12 0 5.302 3.438 9.8 8.207 11.387.599.111.793-.261.793-.577v-2.234c-3.338.726-4.033-1.416-4.033-1.416-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.839 1.237 1.839 1.237 1.07 1.834 2.807 1.304 3.492.997.107-.775.418-1.305.762-1.604-2.665-.305-5.467-1.334-5.467-5.931 0-1.311.469-2.381 1.236-3.221-.124-.303-.535-1.524.117-3.176 0 0 1.008-.322 3.301 1.23.957-.266 1.983-.399 3.003-.404 1.02.005 2.047.138 3.006.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.874.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.609-2.807 5.624-5.479 5.921.43.372.823 1.102.823 2.222v3.293c0 .319.192.694.801.576 4.765-1.589 8.199-6.086 8.199-11.386 0-6.627-5.373-12-12-12z"/>
          </svg>
        </a>
      </div>
      
      <!-- ==================================================
           ACTION BUTTON
           
           DaisyUI classes:
           - card-actions: Container for card buttons
           - btn btn-primary btn-block: Full-width primary button
           
           Tailwind classes:
           - mt-4: Top margin
           ================================================== -->
      <div class="card-actions w-full mt-4">
        <button class="btn btn-primary btn-block">
          View Full Profile
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* 
  No custom styles needed! 
  Everything is handled by Tailwind utilities and DaisyUI components.
  
  If you need custom styles that Tailwind doesn't cover,
  you can add them here. But for 95% of use cases,
  utilities are sufficient.
*/
</style>
```

### App.vue

```vue
<script setup lang="ts">
// ============================================================
// App.vue
// Main application component demonstrating the ProfileCard
// ============================================================

import ProfileCard from './components/ProfileCard.vue'

// Sample team data
const teamMembers = [
  {
    name: 'Sarah Chen',
    role: 'Senior Frontend Engineer',
    department: 'Engineering',
    avatarUrl: 'https://i.pravatar.cc/150?img=1',
    email: 'sarah.chen@company.com',
    socialLinks: {
      twitter: 'https://twitter.com/sarahchen',
      linkedin: 'https://linkedin.com/in/sarahchen',
      github: 'https://github.com/sarahchen'
    }
  },
  {
    name: 'Marcus Johnson',
    role: 'Product Designer',
    department: 'Design',
    avatarUrl: 'https://i.pravatar.cc/150?img=3',
    email: 'marcus.j@company.com',
    socialLinks: {
      linkedin: 'https://linkedin.com/in/marcusj',
      twitter: 'https://twitter.com/marcusj'
    }
  },
  {
    name: 'Emily Rodriguez',
    role: 'Engineering Manager',
    department: 'Engineering',
    avatarUrl: 'https://i.pravatar.cc/150?img=5',
    email: 'emily.r@company.com',
    socialLinks: {
      github: 'https://github.com/emilyr',
      linkedin: 'https://linkedin.com/in/emilyr'
    }
  }
]
</script>

<template>
  <!-- 
    Main container with:
    - min-h-screen: Full viewport height minimum
    - bg-base-200: Slightly darker background (theme-aware)
    - p-4 md:p-8: Responsive padding
  -->
  <div class="min-h-screen bg-base-200 p-4 md:p-8">
    
    <!-- Page header -->
    <header class="text-center mb-8">
      <h1 class="text-3xl md:text-4xl font-bold text-base-content">
        Our Team
      </h1>
      <p class="text-base-content/60 mt-2">
        Meet the amazing people behind our products
      </p>
    </header>
    
    <!-- 
      Card grid with:
      - grid: CSS Grid layout
      - grid-cols-1: Single column on mobile
      - md:grid-cols-2: Two columns on medium screens
      - lg:grid-cols-3: Three columns on large screens
      - gap-6: Consistent spacing between cards
      - justify-items-center: Center cards in their grid cells
      - max-w-6xl mx-auto: Constrain width and center
    -->
    <main class="
      grid 
      grid-cols-1 
      md:grid-cols-2 
      lg:grid-cols-3 
      gap-6
      justify-items-center
      max-w-6xl 
      mx-auto
    ">
      <!-- Loop through team members and render ProfileCards -->
      <ProfileCard
        v-for="member in teamMembers"
        :key="member.email"
        :name="member.name"
        :role="member.role"
        :department="member.department"
        :avatar-url="member.avatarUrl"
        :email="member.email"
        :social-links="member.socialLinks"
      />
    </main>
  </div>
</template>
```

### main.ts

```typescript
// ============================================================
// main.ts
// Application entry point
// ============================================================

import { createApp } from 'vue'
import App from './App.vue'

// Import Tailwind CSS (includes DaisyUI)
import './style.css'

createApp(App).mount('#app')
```

### style.css (Tailwind Entry Point)

```css
/* ============================================================
   style.css
   Tailwind CSS entry point with DaisyUI
   ============================================================ */

/* Tailwind base styles (resets, defaults) */
@tailwind base;

/* Tailwind component styles (DaisyUI components loaded here) */
@tailwind components;

/* Tailwind utility classes */
@tailwind utilities;
```

### tailwind.config.js

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
  plugins: [
    require('daisyui'),
  ],
  daisyui: {
    themes: ["light", "dark", "cupcake", "cyberpunk"],
  },
}
```

### How to Test

1. Create a new Vite + Vue + TypeScript project:
   ```bash
   npm create vite@latest my-profile-cards -- --template vue-ts
   cd my-profile-cards
   ```

2. Install Tailwind CSS and DaisyUI:
   ```bash
   npm install -D tailwindcss postcss autoprefixer daisyui
   npx tailwindcss init -p
   ```

3. Configure `tailwind.config.js` as shown above

4. Replace `src/style.css` with the Tailwind directives

5. Copy the component files to your project

6. Run the development server:
   ```bash
   npm run dev
   ```

7. Open `http://localhost:5173` in your browser

8. **Verify:**
   - Cards display correctly with avatar, name, role, and department badge
   - Hover over cards to see the lift and shadow animation
   - Resize the browser to see responsive behavior (1 → 2 → 3 columns)
   - Social links show only when provided in the data

---

## 7. Implementation Exercises (Progressive)

### Level 1 (Recall): Fill in the Blanks

Complete the Tailwind classes to create a pricing card header:

```vue
<template>
  <!-- 
    Create a pricing card header with:
    - White background
    - Padding of 6 (1.5rem) on all sides
    - Centered text
    - Bottom border (gray-200)
  -->
  <div class="_____ _____ _____ _____">
    <h3 class="text-2xl font-bold">Pro Plan</h3>
    <p class="text-4xl font-extrabold mt-2">$29<span class="text-lg font-normal">/mo</span></p>
  </div>
</template>
```

**Answer:** `bg-white p-6 text-center border-b border-gray-200`

---

### Level 2 (Apply): Small Independent Tasks

**Exercise 2.1:** Create a DaisyUI stat component that displays:
- A large number (e.g., "31K")
- A label below (e.g., "Downloads")
- An icon to the left
- Use the `stat` component classes from DaisyUI

**Exercise 2.2:** Build a responsive image gallery grid that:
- Shows 2 columns on mobile
- Shows 3 columns on tablet (md:)
- Shows 4 columns on desktop (lg:)
- Has consistent gap spacing
- Images have rounded corners and hover:scale effect

---

### Level 3 (Debug): Find and Fix the Bugs

This code has 3 styling bugs. Find and fix them:

```vue
<template>
  <!-- BUG 1: Button won't show hover effect properly -->
  <button class="btn btn-primary btn-hover:btn-secondary">
    Click Me
  </button>
  
  <!-- BUG 2: Text should be large on desktop, small on mobile, but it's reversed -->
  <p class="lg:text-sm text-lg">
    Responsive text
  </p>
  
  <!-- BUG 3: This should be a flex container with centered items, but items aren't centered -->
  <div class="flex align-center justify-center">
    <span>Item 1</span>
    <span>Item 2</span>
  </div>
</template>
```

**Solutions:**

```vue
<template>
  <!-- FIX 1: Correct hover syntax uses hover: prefix before the class -->
  <button class="btn btn-primary hover:btn-secondary">
    Click Me
  </button>
  
  <!-- FIX 2: Mobile-first means base class is for mobile, prefix is for larger -->
  <p class="text-sm lg:text-lg">
    Responsive text
  </p>
  
  <!-- FIX 3: Tailwind uses items-center, not align-center -->
  <div class="flex items-center justify-center">
    <span>Item 1</span>
    <span>Item 2</span>
  </div>
</template>
```

---

### Self-Check Solutions

**Exercise 2.1 Solution:**
```vue
<template>
  <div class="stat">
    <div class="stat-figure text-primary">
      <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" 
           class="inline-block w-8 h-8 stroke-current">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" 
              d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z" />
      </svg>
    </div>
    <div class="stat-value">31K</div>
    <div class="stat-title">Downloads</div>
  </div>
</template>
```

**Exercise 2.2 Solution:**
```vue
<template>
  <div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
    <div v-for="i in 8" :key="i" class="overflow-hidden rounded-lg">
      <img 
        :src="`https://picsum.photos/300/300?random=${i}`"
        :alt="`Gallery image ${i}`"
        class="w-full h-full object-cover transition-transform duration-300 hover:scale-110"
      />
    </div>
  </div>
</template>
```

---

**✅ End of Lesson 8.2**

You've now practiced applying Tailwind utilities and DaisyUI components to build real Vue components. You can create responsive layouts, interactive hover states, and theme-aware designs.

---

*Reply 'next' for Lesson 9.1 (Theory) — Conditional Rendering: `v-if`, `v-else-if`, & `v-show`*
