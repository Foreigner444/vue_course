# Lesson 9.1: Conditional Rendering — Theory & Concepts (Условный рендеринг)

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 9.1 of 25 (Part 1 of 2) |
| **Topic** | Conditional Rendering: `v-if`, `v-else-if`, & `v-show` / Условный рендеринг |
| **Module** | Module 4: Rendering Patterns |
| **Prerequisites** | Lessons 1-5 (Core Fundamentals), Lesson 8 (Styling) |
| **You Should Know** | JavaScript conditionals (`if`/`else`), truthy/falsy values, boolean logic |
| **Unlocks** | Lesson 9.2, then Lesson 10 (List Rendering) |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 1 (Theory):**
1. **Remember:** Define what conditional rendering means and list the three Vue directives used for it (`v-if`, `v-else-if`, `v-else`, `v-show`)
2. **Understand:** Explain how Vue adds/removes elements from the DOM vs. toggling visibility, and when to choose each approach

---

## 2. Real-World Scenario & Context

**Scenario:** Imagine you're building a user dashboard for an online learning platform. When a user logs in, they should see their profile information and course progress. When they're logged out, they should see a login form instead. Additionally, premium users should see extra features that free users cannot access. The interface needs to adapt dynamically based on authentication status and subscription tier.

**Framework Context:** Vue's conditional rendering directives solve the problem of showing or hiding parts of your UI based on application state. Instead of manually manipulating the DOM with JavaScript (like `element.style.display = 'none'`), Vue provides declarative directives that automatically sync your template with your reactive data.

---

## 3. Core Concepts Explained (Deep Dive)

### 3.1 How It Actually Works

Vue provides two fundamentally different approaches to conditional rendering:

**Approach 1: `v-if` / `v-else-if` / `v-else` — True Conditional Rendering**

When you use `v-if`, Vue performs **real DOM manipulation**:
- When the condition is `true`: Vue creates the element and inserts it into the DOM
- When the condition is `false`: Vue **completely removes** the element from the DOM (it doesn't exist at all)

```
┌─────────────────────────────────────────────────────────────┐
│                    v-if FLOW DIAGRAM                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Reactive State Change                                     │
│          │                                                  │
│          ▼                                                  │
│   ┌──────────────┐                                          │
│   │  Condition   │                                          │
│   │  Evaluated   │                                          │
│   └──────┬───────┘                                          │
│          │                                                  │
│    ┌─────┴─────┐                                            │
│    │           │                                            │
│    ▼           ▼                                            │
│  TRUE        FALSE                                          │
│    │           │                                            │
│    ▼           ▼                                            │
│ ┌──────┐   ┌──────────┐                                     │
│ │CREATE│   │ DESTROY  │                                     │
│ │ DOM  │   │   DOM    │                                     │
│ │ Node │   │   Node   │                                     │
│ └──────┘   └──────────┘                                     │
│                                                             │
│ Element EXISTS        Element DOES NOT EXIST                │
│ in the DOM            in the DOM at all                     │
└─────────────────────────────────────────────────────────────┘
```

**Approach 2: `v-show` — CSS-Based Toggling**

When you use `v-show`, the element is **always rendered** and stays in the DOM:
- When the condition is `true`: Element is visible (`display: block/flex/etc.`)
- When the condition is `false`: Vue adds `display: none` inline style (element is hidden but still in DOM)

```
┌─────────────────────────────────────────────────────────────┐
│                   v-show FLOW DIAGRAM                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Reactive State Change                                     │
│          │                                                  │
│          ▼                                                  │
│   ┌──────────────┐                                          │
│   │  Condition   │                                          │
│   │  Evaluated   │                                          │
│   └──────┬───────┘                                          │
│          │                                                  │
│    ┌─────┴─────┐                                            │
│    │           │                                            │
│    ▼           ▼                                            │
│  TRUE        FALSE                                          │
│    │           │                                            │
│    ▼           ▼                                            │
│ ┌──────────┐  ┌──────────────────┐                          │
│ │ display: │  │ display: none    │                          │
│ │ (normal) │  │ (CSS property)   │                          │
│ └──────────┘  └──────────────────┘                          │
│                                                             │
│ Element VISIBLE       Element HIDDEN                        │
│ (always in DOM)       (still in DOM, just invisible)        │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 Mental Model

Think of `v-if` and `v-show` like two different ways to manage furniture in a room:

**`v-if` = Moving furniture in and out of storage**
- When you need a chair (`v-if="true"`), you go to storage, get the chair, and bring it into the room
- When you don't need it (`v-if="false"`), you completely remove the chair and put it back in storage
- The chair doesn't exist in the room when not needed
- Moving furniture takes effort (performance cost)

**`v-show` = Covering furniture with a sheet**
- The chair is always in the room
- When you need it (`v-show="true"`), you remove the sheet and use it
- When you don't need it (`v-show="false"`), you cover it with a sheet (it's still there, just hidden)
- Covering/uncovering is quick and easy

```
┌─────────────────────────────────────────────────────────────┐
│                  MENTAL MODEL ILLUSTRATION                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   v-if (Moving Furniture)          v-show (Sheet Cover)     │
│                                                             │
│   ┌─────────┐                      ┌─────────┐              │
│   │  ROOM   │                      │  ROOM   │              │
│   │         │ ◄── Chair moved      │  🪑     │ ◄── Always   │
│   │   🪑    │     in when needed   │ (sheet) │     present  │
│   │         │                      │         │              │
│   └─────────┘                      └─────────┘              │
│                                                             │
│   ┌─────────┐                                               │
│   │ STORAGE │                                               │
│   │   🪑    │ ◄── Chair stored                              │
│   │         │     when not needed                           │
│   └─────────┘                                               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 3.3 In Other Contexts

| Framework | Conditional Rendering Equivalent |
|:--- |:--- |
| **Vue** | `v-if`, `v-else-if`, `v-else`, `v-show` |
| **React** | Ternary operators `{condition ? <A/> : <B/>}`, logical AND `{condition && <A/>}` |
| **Angular** | `*ngIf`, `[hidden]` attribute binding |
| **Vanilla JS** | `if/else` + `document.createElement()` / `element.remove()` or `element.style.display` |

**Key Difference:** Vue's approach is declarative—you describe *what* should appear based on state, not *how* to manipulate the DOM.

### 3.4 When to Use / When NOT to Use

```
┌─────────────────────────────────────────────────────────────┐
│              DECISION TREE: v-if vs v-show                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   "How often will this toggle?"                             │
│                │                                            │
│        ┌───────┴───────┐                                    │
│        │               │                                    │
│        ▼               ▼                                    │
│    RARELY          FREQUENTLY                               │
│   (once or         (many times                              │
│    twice)           per session)                            │
│        │               │                                    │
│        ▼               ▼                                    │
│   ┌─────────┐     ┌─────────┐                               │
│   │  v-if   │     │ v-show  │                               │
│   └─────────┘     └─────────┘                               │
│                                                             │
│   Examples:        Examples:                                │
│   • Login/logout   • Dropdown menus                         │
│   • Error states   • Tooltips                               │
│   • Feature flags  • Tab panels                             │
│   • Modal dialogs  • Accordions                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

| Scenario | Use `v-if` | Use `v-show` |
|:--- |:---:|:---:|
| Content rarely changes (login state) | ✅ | |
| Content toggles frequently (dropdown) | | ✅ |
| Heavy component with expensive setup | ✅ | |
| Simple element that toggles often | | ✅ |
| Need to avoid initial render cost | ✅ | |
| Need instant toggle performance | | ✅ |
| Using with `v-else` or `v-else-if` | ✅ | ❌ Not supported |
| Using on `<template>` wrapper | ✅ | ❌ Not supported |

---

## 4. New Terminology

| Term | Definition |
|:--- |:--- |
| **Conditional Rendering** | The technique of showing or hiding parts of the UI based on the application's state or data conditions |
| **Directive** | Special attributes in Vue templates (prefixed with `v-`) that apply reactive behavior to the DOM |
| **DOM Mounting/Unmounting** | The process of adding (mounting) or removing (unmounting) elements from the Document Object Model |
| **Truthy/Falsy** | JavaScript values that evaluate to `true` or `false` in a boolean context. Falsy: `false`, `0`, `""`, `null`, `undefined`, `NaN`. All other values are truthy. |

---

## 5. Algorithmic Thinking (Planning the Solution)

**The Plan:** Building the user dashboard scenario step-by-step.

```
┌─────────────────────────────────────────────────────────────┐
│           PLANNING FLOWCHART: Dashboard UI                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   START                                                     │
│     │                                                       │
│     ▼                                                       │
│   ┌─────────────────────────────────┐                       │
│   │ Step 1: Define the state        │                       │
│   │ • isLoggedIn (boolean)          │                       │
│   │ • userTier ('free'|'premium')   │                       │
│   └─────────────────┬───────────────┘                       │
│                     │                                       │
│                     ▼                                       │
│   ┌─────────────────────────────────┐                       │
│   │ Step 2: Identify conditions     │                       │
│   │ • Not logged in → Show login    │                       │
│   │ • Logged in + free → Basic UI   │                       │
│   │ • Logged in + premium → Full UI │                       │
│   └─────────────────┬───────────────┘                       │
│                     │                                       │
│                     ▼                                       │
│   ┌─────────────────────────────────┐                       │
│   │ Step 3: Choose directive        │                       │
│   │ • Auth state changes rarely     │                       │
│   │   → Use v-if/v-else-if/v-else   │                       │
│   └─────────────────┬───────────────┘                       │
│                     │                                       │
│                     ▼                                       │
│   ┌─────────────────────────────────┐                       │
│   │ Step 4: Structure the template  │                       │
│   │ • v-if="!isLoggedIn" → Login    │                       │
│   │ • v-else-if="isPremium" → Full  │                       │
│   │ • v-else → Basic dashboard      │                       │
│   └─────────────────┬───────────────┘                       │
│                     │                                       │
│                     ▼                                       │
│   ┌─────────────────────────────────┐                       │
│   │ Step 5: Add reactive triggers   │                       │
│   │ • Buttons to toggle login       │                       │
│   │ • Buttons to change tier        │                       │
│   └─────────────────────────────────┘                       │
│                     │                                       │
│                     ▼                                       │
│                   END                                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Narrative:**

1. **Define the state:** First, identify what reactive data controls the UI. We need `isLoggedIn` (a boolean) and `userTier` (a string like `'free'` or `'premium'`).

2. **Identify all conditions:** Map out every possible state combination and what UI should appear for each. This prevents missing edge cases.

3. **Choose the right directive:** Since authentication state doesn't toggle frequently (users don't log in/out repeatedly), `v-if` is the right choice for better performance.

4. **Structure the template:** Use `v-if` for the first condition, `v-else-if` for subsequent conditions, and `v-else` as the fallback. The order matters!

5. **Add reactive triggers:** Create buttons or actions that modify the state, which will automatically trigger the conditional rendering.

---

## 6. Initial Pattern Introduction

| What You Want (Intent) | Code Chunk (The Pattern) | Conceptual Link |
|:--- |:--- |:--- |
| Show element only when condition is true | `<div v-if="condition">` | DOM created/destroyed based on boolean |
| Show alternative when first condition is false | `<div v-else>` | Must immediately follow `v-if` |
| Chain multiple conditions | `<div v-else-if="otherCondition">` | Evaluated in order, first true wins |
| Toggle visibility without DOM changes | `<div v-show="condition">` | Uses CSS `display: none` |

```vue
<script setup lang="ts">
// Step 1: Import ref to create reactive state
import { ref, computed } from 'vue'

// Step 2: Define reactive state that controls what we show
const isLoggedIn = ref<boolean>(false)
const userTier = ref<'free' | 'premium'>('free')

// Step 3: Create a computed property for cleaner template logic
const isPremium = computed(() => userTier.value === 'premium')

// Step 4: Functions to change state (triggers re-render)
const login = () => {
  isLoggedIn.value = true
}

const logout = () => {
  isLoggedIn.value = false
}

const upgradeToPremium = () => {
  userTier.value = 'premium'
}
</script>

<template>
  <!-- 
    v-if / v-else-if / v-else chain
    - Only ONE of these will be in the DOM at any time
    - They must be adjacent siblings (no elements between them)
  -->
  
  <!-- Condition 1: User is NOT logged in -->
  <div v-if="!isLoggedIn" class="login-form">
    <h2>Please Log In</h2>
    <button @click="login">Log In</button>
  </div>
  
  <!-- Condition 2: User IS logged in AND is premium -->
  <div v-else-if="isPremium" class="premium-dashboard">
    <h2>Welcome, Premium User!</h2>
    <p>You have access to all features.</p>
    <button @click="logout">Log Out</button>
  </div>
  
  <!-- Condition 3: User IS logged in but NOT premium (fallback) -->
  <div v-else class="basic-dashboard">
    <h2>Welcome, Free User!</h2>
    <p>Upgrade to unlock more features.</p>
    <button @click="upgradeToPremium">Upgrade</button>
    <button @click="logout">Log Out</button>
  </div>
</template>
```

**Key Syntax Rules:**
1. `v-else-if` and `v-else` must immediately follow a `v-if` or `v-else-if` (no other elements between them)
2. `v-else` takes no value (it's the fallback)
3. Conditions are evaluated top-to-bottom; first `true` wins
4. `v-show` does NOT support `v-else`

---

## 7. Comprehension Check

1. **Conceptual:** What happens to a DOM element when its `v-if` condition changes from `true` to `false`? How does this differ from `v-show`?

2. **Conceptual:** You're building a tooltip that appears when hovering over a button and disappears when the mouse leaves. Should you use `v-if` or `v-show`? Explain your reasoning.

3. **True/False:** The following code is valid Vue syntax:
   ```vue
   <div v-if="conditionA">A</div>
   <span>Some text</span>
   <div v-else>B</div>
   ```

---

*Reply 'next' for Lesson 9.2 (Practice).*
