### Lesson: Your First Component: Template + Script Setup
(Первый компонент)
---

### **Part 1: Foundation & Planning**
*(Est. Time: 10 minutes)*

#### 1. Lesson Metadata
*A table summarizing the lesson's scope and goals.*
| Field | Value |
|:--- |:--- |
| **Lesson Number** | 1 of 15 |
| **Complexity** | Minimal |
| **Prerequisites** | Basic JavaScript/TypeScript & HTML knowledge |
| **Core Patterns** | 3 new + 0 recycled |
| **Key Pattern-Chunk** | The Single File Component (SFC) structure |
| **Algorithmic Focus** | UI Decomposition |

#### 2. Real-World Scenario & Context
*   **Scenario:** You've just joined a new project. Your first task is to create a static, reusable "Welcome Banner" for the main dashboard page. The lead developer tells you, "Just build the basic component file; we'll wire up the data later." The goal is to create the component structure correctly so others can build upon it.
*   **Framework Context:** Vue 3 excels at this because its Single File Component (SFC) system lets you organize a component's HTML, JavaScript, and CSS in one clean, intuitive file. "Production-ready" at this stage means creating a well-formed, correctly named component file that follows project conventions.

#### 3. How to Approach This Lesson
*   **Pragmatist Path (Recommended):** Follow the parts in order (1-4) to build practical skills quickly.
*   **Theorist Path:** If you prefer to understand "why" before "how," follow this order: **Part 1**, then jump to **Section 12 (Deep Dive)** in Part 4, then return to **Part 2** and finish the lesson from there.

#### 4. Algorithmic Thinking (Problem Decomposition)
*   **The Problem:** Create a self-contained UI component that displays a static welcome message.
*   **Decomposition Exercise (3-5 steps):
    1.  **Identify the boundary:** What is the component? → The "Welcome Banner".
    2.  **Define the structure (view):** What will it look like? → A container `<div>` with a title `<h1>` and a paragraph `<p>`. This maps to the `<template>` block.
    3.  **Define the logic (state):** What data does it need? → For now, none. All content is static. This maps to an empty `<script setup>` block.
    4.  **Encapsulate:** How do we package it? → A single `.vue` file.

*   **Input/Output Analysis:**
| Aspect | Details |
|:--- |:--- |
| **Input source** | None (all content is static within the component). |
| **Input type** | N/A. |
| **Validation needs** | N/A. |
| **Output shape** | A block of rendered HTML (`<div><h1>...</h1><p>...</p></div>`). |
| **Success case** | The component renders the static HTML correctly. |
| **Error cases** | Mismatched tags, syntax errors in the template. |

*   **Edge Cases Checklist:**
| Edge Case | What Happens | How to Handle |
|:--- |:--- |:--- |
| Component file misnamed | Build tools may fail; imports become confusing. | Use `PascalCase` for component filenames (e.g., `WelcomeBanner.vue`). |
| No `<template>` block | Vue compiler throws an error. | Every component must have a template. |
| Multiple root elements | Works in Vue 3, but a single root is often cleaner. | Wrap template content in one root `<div>` for clarity and styling. |
| Missing `<script>` block | Perfectly valid for static components. | Only add `<script setup>` when you need logic or to import something. |

*   **Decision Tree:**
```
Do I need to create a reusable piece of UI?
└── Yes: Create a Single File Component (.vue file)
    ├── Does it need any JavaScript logic or state?
    │   ├── Yes: Use <script setup> and <template>
    │   └── No: You can use just a <template> block (but adding an empty <script setup> is good practice)
```

---
### **Part 2: Core Pattern Implementation**
*(Est. Time: 25 minutes)*

#### 5. The Intent Map (Goal → Code Bridge)
*A table with 5-7 rows demonstrating the core patterns.*
| What You Want (Intent) | Code Chunk (The Pattern) | Defensive Version | Pattern Note |
|:--- |:--- |:--- |:--- |
| Create a basic component | `(See A below)` | `(See B below)` | Pattern-chunk: The basic SFC. Always have a root element. |
| Add TypeScript-ready logic | `<script setup lang="ts"></script>` | `---` | Pattern-chunk: The script block. `lang="ts"` is essential for TypeScript. |
| Combine logic and view | `(See C below)` | `(See D below)` | The complete, minimal, well-formed SFC. |
| Add a title | `<h1>Welcome!</h1>` | `<h1 class="text-2xl font-bold">Welcome!</h1>` | Use semantic HTML for content. |
| Add descriptive text | `<p>Your dashboard is ready.</p>` | `<p class="text-base-content/70 mt-2">Your dashboard is ready.</p>` | Tailwind/DaisyUI classes add styling. |

**Full Code Chunks for Intent Map:**
*   **A. Basic Component:**
    ```vue
    <template>
      <h1>Hi</h1>
    </template>
    ```
*   **B. Defensive Component:**
    ```vue
    <template>
      <div class="my-component">
        <h1>Hi</h1>
      </div>
    </template>
    ```
*   **C. Combine Logic and View (Minimal):**
    ```vue
    <script setup lang="ts"></script>
    <template>
      <div>...</div>
    </template>
    ```
*   **D. Combine Logic and View (Defensive/Production):**
    ```vue
    <!-- WelcomeBanner.vue -->
    <script setup lang="ts"></script>

    <template>
      <div class="p-4">
        ...
      </div>
    </template>
    ```


#### 6. The Variable Frame & Complexity Scale
*Show how patterns are composed of fixed and variable parts ("slots").*

*   **Frame 1: The Single File Component (SFC) Boilerplate**
    *   **Fixed:** `<script setup lang="ts">
  [...Script Content...]
</script>

<template>
  [...Template Content...]
</template>`
    *   **Slot (`Script Content`):** Imports, variable definitions, functions (empty for now).
    *   **Slot (`Template Content`):** The HTML that defines your component's structure.
    *   **Example:**
        *   `Script Content` = `// No logic yet`
        *   `Template Content` = `<div>My App</div>`

*   **Complexity Scale: Your First Component**
    ```
    ├── Quick & dirty: `<template><h1>Hello</h1></template>`
    ├── Standard: `<script setup lang="ts"></script><template><div><h1>Hello</h1></div></template>`
    ├── Robust: `<!-- MyComponent.vue -->
<script setup lang="ts"></script>
<template><div>...</div></template>` (Filename comment)
    ├── Production: (Standard version, but formatted with Prettier, following project linting rules).
    ```

#### 7. Micro-Implementation Drills
*Teach step-by-step implementation with 4 short, scenario-specific drills.*

*   **Drill A (Sequencing): Create a `UserProfile.vue` component.**
    ```vue
    // 1. Create the file: UserProfile.vue

    // 2. Add the script block for TypeScript
    <script setup lang="ts">
    </script>

    // 3. Add the template block with a root div
    <template>
      <div>
      </div>
    </template>

    // 4. Add static content inside the template
    <template>
      <div>
        <h2>User Profile</h2>
        <p>Name: John Doe</p>
      </div>
    </template>
    ```

*   **Drill B (Data Flow): N/A for this lesson, as there is no data flow yet.**

*   **Drill C (State-Change): N/A for this lesson, as there is no state yet.**

*   **Drill D (Debug Flow): Your component shows a blank page.**
    1.  **Check View:** Is there any content inside the `<template>` block? Is it saved?
    2.  **Check Structure:** Does the `<template>` have exactly one root element? (A common issue in older Vue versions, still good practice).
    3.  **Check Import:** Did you import and use the component correctly in your main `App.vue`? (We'll cover this in the next lessons).

#### 8. Code Collocations & Fixed Patterns
*Show memorized combinations that always go together.*
*   **A. Import + Usage Pairs:** N/A for this lesson.
*   **B. Script Setup + Template Patterns:**
| Script Setup | Template Usage | Example |
|:--- |:--- |:--- |
| `<script setup lang="ts">` | `<template>` | A component always contains both, even if the script part is empty. |

*   **C. Type + Component Combinations:** N/A for this lesson.

---
### **Part 3: Application & Refinement**
*(Est. Time: 20 minutes)*

#### 10. Mini-Project (Putting It Together)
*A complete, runnable code example implementing a simple "Notification" component.*
```vue
<!-- NotificationBanner.vue -->
<!-- A static banner component styled with DaisyUI -->

<script setup lang="ts">
// No logic is needed for this static component yet.
// We use <script setup> to declare we are using Composition API and TypeScript.
</script>

<template>
  <!-- DaisyUI 'alert' component with info color -->
  <div role="alert" class="alert alert-info shadow-lg">
    <!-- Icon -->
    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" class="stroke-current shrink-0 w-6 h-6"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg>
    
    <!-- Content -->
    <div>
      <h3 class="font-bold">New Feature!</h3>
      <div class="text-xs">You can now customize your dashboard.</div>
    </div>

    <!-- Action button -->
    <button class="btn btn-sm">See</button>
  </div>
</template>
```

#### 11. Implementation Exercise
*   **Phase 1: Write:**
    1.  **Task (★):** Create a new file named `PageFooter.vue`.
    2.  **Task (★):** Inside `PageFooter.vue`, add a `<template>` block containing a `<footer>` element.
    3.  **Task (★★):** Add a `<p>` tag inside the footer with the text "© 2026 Acme Corp. All rights reserved.".
    4.  **Task (★★):** Add an empty `<script setup lang="ts">` block to the file.
    5.  **Task (★★★):** Use DaisyUI/Tailwind classes to give the footer a `bg-neutral`, `text-neutral-content`, and `p-4` style.

*   **Phase 2: Self-Check:**
    ```vue
    <!-- Reference Implementation for PageFooter.vue -->
    <script setup lang="ts">
    </script>

    <template>
      <footer class="footer footer-center p-4 bg-neutral text-neutral-content">
        <div>
          <p>© 2026 Acme Corp. All rights reserved.</p>
        </div>
      </footer>
    </template>
    ```

---
### **Part 4: Deeper Knowledge & Review**
*(Est. Time: 10 minutes + Optional Deep Dive)*

#### 12. Deep Dive (Optional)
*   **How It Actually Works:** A `.vue` file is a custom file format. When you build your application, Vue's compiler separates it into three parts: the HTML in `<template>` is compiled into a highly efficient JavaScript `render` function, the logic in `<script setup>` is transformed into a `setup()` function for the component, and the CSS in `<style>` is processed and injected. `<script setup>` is syntactic sugar that makes the Composition API much cleaner to use.
*   **Mental Model:** Think of a `.vue` file as a self-contained blueprint for a piece of your UI. It has the instructions for the structure (`<template>`), the logic (`<script>`), and the appearance (`<style>`). The Vue compiler is the factory that reads the blueprint and builds the final product.
*   **When This Pattern Breaks:** This pattern *is* Vue. It doesn't break. The only time you wouldn't use an SFC is if you are in a non-build environment (using Vue directly from a CDN) or if you are programmatically generating components using render functions, which is a very advanced use case.
*   **In Other Contexts:**
    *   **React:** The closest equivalent is a JSX file (`.jsx` or `.tsx`) which combines markup (JSX) and logic in a single JavaScript file.
    *   **Svelte:** Very similar. Svelte also uses `.svelte` files with `<script>`, markup, and `<style>` blocks.
    *   **General principle:** This is an implementation of "Component-Based Architecture."
