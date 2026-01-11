# Lesson 11.1: Handling User Input: Event Handling — Theory & Concepts (Обработка пользовательского ввода: Обработка событий)

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 11.1 of 25 (Part 1 of 2) |
| **Topic** | Event Handling (`@click`, etc.) / Обработка событий |
| **Module** | Module 5: User Interaction |
| **Prerequisites** | Lessons 1-10 (Core Fundamentals, Component Communication, Rendering Patterns) |
| **You Should Know** | JavaScript event handling basics, arrow functions, DOM events |
| **Unlocks** | Lesson 11.2, then Lessons 12, 13, 14 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 1 (Theory):**
1. **Remember:** Define what events are in Vue, list common event types, and recall the `v-on` directive syntax
2. **Understand:** Explain how Vue's event system works, including event propagation, modifiers, and the relationship between DOM events and Vue handlers

---

## 2. Real-World Scenario & Context

**Scenario:** You're building an interactive dashboard for a project management app. Users need to click buttons to create tasks, hover over cards to see previews, submit forms, and use keyboard shortcuts for power-user workflows. Every interaction—from a simple button click to a complex drag-and-drop—relies on responding to user-generated events.

**Framework Context:** Vue provides a declarative event handling system through the `v-on` directive (shorthand: `@`) that connects DOM events to component methods. This abstraction handles event listener attachment/detachment automatically, preventing memory leaks and simplifying the developer experience compared to manual `addEventListener` management.

---

## 3. Core Concepts Explained (Deep Dive)

### How It Actually Works

When you write `@click="handleClick"` in a Vue template, Vue's compiler transforms this into efficient JavaScript that:

1. **During Mount:** Attaches an event listener to the DOM element
2. **During Event:** Executes your handler function with the event object
3. **During Unmount:** Automatically removes the listener (preventing memory leaks)

```
┌─────────────────────────────────────────────────────────────┐
│                    Vue Event Flow                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Template: <button @click="handleClick">                   │
│                        │                                    │
│                        ▼                                    │
│   ┌─────────────────────────────────────┐                   │
│   │  Vue Compiler transforms to:        │                   │
│   │  element.addEventListener('click',  │                   │
│   │    (event) => handleClick(event))   │                   │
│   └─────────────────────────────────────┘                   │
│                        │                                    │
│                        ▼                                    │
│   ┌─────────────────────────────────────┐                   │
│   │  User clicks button                 │                   │
│   │  → DOM fires 'click' event          │                   │
│   │  → Vue calls handleClick(event)     │                   │
│   │  → Your code executes               │                   │
│   └─────────────────────────────────────┘                   │
│                        │                                    │
│                        ▼                                    │
│   ┌─────────────────────────────────────┐                   │
│   │  Component unmounts                 │                   │
│   │  → Vue calls removeEventListener    │                   │
│   │  → No memory leaks!                 │                   │
│   └─────────────────────────────────────┘                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Mental Model

**Think of Vue events as a "smart receptionist" at a hotel:**

- The receptionist (`v-on` / `@`) listens for guests (events) arriving
- When a specific guest arrives (e.g., "click"), they immediately call the right department (your handler function)
- They pass along any message the guest brought (the event object)
- When the hotel closes (component unmounts), the receptionist automatically goes home (listener removed)

```
┌─────────────────────────────────────────────────────────────┐
│                   🏨 The Vue Hotel                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   👤 Guest "click" arrives                                  │
│          │                                                  │
│          ▼                                                  │
│   ┌─────────────┐      ┌──────────────────┐                │
│   │ Receptionist│ ───► │ handleClick()    │                │
│   │   (@click)  │      │ "Right this way!"│                │
│   └─────────────┘      └──────────────────┘                │
│          │                                                  │
│          │ Passes event info (which button, coordinates)    │
│          ▼                                                  │
│   ┌──────────────────────────────────────┐                  │
│   │  Your code runs with full context    │                  │
│   └──────────────────────────────────────┘                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### In Other Contexts

| Framework | Event Syntax | Notes |
|:--- |:--- |:--- |
| **Vue 3** | `@click="handler"` | Declarative, auto-cleanup |
| **React** | `onClick={handler}` | camelCase, synthetic events |
| **Angular** | `(click)="handler()"` | Parentheses syntax |
| **Vanilla JS** | `el.addEventListener('click', handler)` | Manual cleanup required |
| **Svelte** | `on:click={handler}` | Similar to Vue |

### When to Use / When NOT to Use

| ✅ Use Event Handlers When | ❌ Avoid When |
|:--- |:--- |
| Responding to user actions (clicks, typing, scrolling) | Data should update automatically (use `computed()` instead) |
| Triggering side effects (API calls, navigation) | You need to watch data changes (use `watch()` instead) |
| Validating input before processing | The "event" is a prop change from parent (use `watch` on prop) |
| Coordinating multiple UI updates | You're tempted to use events for parent-to-child communication (use props) |

---

## 4. New Terminology

| Term | Definition |
|:--- |:--- |
| **Event Handler** | A function that executes in response to a specific event (e.g., click, keypress). In Vue, these are defined in `<script setup>` and referenced in the template. |
| **Event Modifier** | A suffix added to the event directive (e.g., `.prevent`, `.stop`) that automatically applies common event manipulations like `preventDefault()` or `stopPropagation()`. |
| **Event Object** | The native DOM event passed to your handler, containing information about the event (target element, coordinates, key pressed, etc.). Accessed as the first parameter or via `$event`. |
| **Inline Handler** | An expression written directly in the template (e.g., `@click="count++"`), useful for simple one-liners but discouraged for complex logic. |

---

## 5. Algorithmic Thinking (Planning the Solution)

**The Plan:** Building an interactive button that tracks click count and shows the last click position.

```
┌─────────────────────────────────────────────────────────────┐
│           Decision Tree: Handling a Click Event             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Step 1: Identify the interaction                          │
│   └── "User clicks a button"                                │
│       └── Event type: 'click'                               │
│                                                             │
│   Step 2: Decide what data to track                         │
│   └── Click count (number)                                  │
│   └── Last click position (x, y coordinates)                │
│       └── Create reactive refs for each                     │
│                                                             │
│   Step 3: Write the handler function                        │
│   └── Function receives MouseEvent                          │
│   └── Extract clientX, clientY from event                   │
│   └── Increment count                                       │
│   └── Update position state                                 │
│                                                             │
│   Step 4: Connect handler to template                       │
│   └── Use @click="handlerName" on the element               │
│                                                             │
│   Step 5: Display reactive data                             │
│   └── Interpolate {{ count }} and {{ position }}            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. Initial Pattern Introduction

| What You Want (Intent) | Code Chunk (The Pattern) | Conceptual Link |
|:--- |:--- |:--- |
| Listen for a click | `@click="handler"` | Shorthand for `v-on:click` |
| Pass the event object | `@click="handler($event)"` or just `@click="handler"` | Event auto-passed as first arg |
| Inline simple logic | `@click="count++"` | Direct expression in template |
| Prevent default behavior | `@click.prevent="handler"` | Replaces `e.preventDefault()` |
| Stop event bubbling | `@click.stop="handler"` | Replaces `e.stopPropagation()` |
| Listen for specific key | `@keyup.enter="handler"` | Key modifier for Enter key |
| Combine modifiers | `@click.stop.prevent="handler"` | Chain modifiers together |

```vue
<script setup lang="ts">
import { ref } from 'vue'

// Step 1: Create reactive state to track
const clickCount = ref<number>(0)
const lastPosition = ref<{ x: number; y: number } | null>(null)

// Step 2: Define the event handler function
// The MouseEvent type provides full autocomplete for event properties
function handleClick(event: MouseEvent): void {
  // Step 3: Update reactive state based on event data
  clickCount.value++
  lastPosition.value = {
    x: event.clientX,
    y: event.clientY
  }
}

// Alternative: Handler that doesn't need the event object
function handleSimpleClick(): void {
  clickCount.value++
}
</script>

<template>
  <!-- Step 4: Connect handler using @click directive -->
  <div class="p-4">
    <!-- Full handler with event access -->
    <button 
      @click="handleClick"
      class="btn btn-primary"
    >
      Click me! ({{ clickCount }} times)
    </button>
    
    <!-- Display event data -->
    <p v-if="lastPosition" class="mt-2">
      Last click at: ({{ lastPosition.x }}, {{ lastPosition.y }})
    </p>
    
    <!-- Inline handler for simple operations -->
    <button 
      @click="clickCount = 0"
      class="btn btn-secondary mt-2"
    >
      Reset
    </button>
  </div>
</template>
```

### Common Event Types Reference

```vue
<template>
  <!-- Mouse Events -->
  <button @click="onClick">Click</button>
  <button @dblclick="onDoubleClick">Double Click</button>
  <div @mouseenter="onHover">Hover me</div>
  <div @mouseleave="onLeave">Leave me</div>
  
  <!-- Keyboard Events -->
  <input @keyup="onKeyUp" />
  <input @keydown.enter="onEnter" />
  <input @keyup.escape="onEscape" />
  
  <!-- Form Events -->
  <form @submit.prevent="onSubmit">
    <input @input="onInput" />
    <input @change="onChange" />
    <input @focus="onFocus" />
    <input @blur="onBlur" />
  </form>
  
  <!-- Other Events -->
  <div @scroll="onScroll">Scrollable</div>
  <img @load="onImageLoad" />
  <img @error="onImageError" />
</template>
```

### Event Modifiers Cheat Sheet

```vue
<template>
  <!-- .prevent - calls event.preventDefault() -->
  <form @submit.prevent="onSubmit">...</form>
  
  <!-- .stop - calls event.stopPropagation() -->
  <button @click.stop="onClick">Won't bubble up</button>
  
  <!-- .once - handler fires only once -->
  <button @click.once="onFirstClick">Only once</button>
  
  <!-- .self - only trigger if event target is this element -->
  <div @click.self="onDivClick">
    <button>Click button won't trigger div handler</button>
  </div>
  
  <!-- .capture - use capture mode instead of bubble -->
  <div @click.capture="onCapture">...</div>
  
  <!-- .passive - for scroll performance -->
  <div @scroll.passive="onScroll">...</div>
  
  <!-- Key modifiers -->
  <input @keyup.enter="onEnter" />
  <input @keyup.tab="onTab" />
  <input @keyup.delete="onDelete" />
  <input @keyup.esc="onEscape" />
  <input @keyup.space="onSpace" />
  <input @keyup.up="onArrowUp" />
  <input @keyup.down="onArrowDown" />
  
  <!-- System modifiers (must be held) -->
  <button @click.ctrl="onCtrlClick">Ctrl+Click</button>
  <button @click.shift="onShiftClick">Shift+Click</button>
  <button @click.alt="onAltClick">Alt+Click</button>
  <button @click.meta="onMetaClick">Cmd/Win+Click</button>
  
  <!-- Mouse button modifiers -->
  <button @click.left="onLeftClick">Left click only</button>
  <button @click.right="onRightClick">Right click only</button>
  <button @click.middle="onMiddleClick">Middle click only</button>
</template>
```

---

## 7. Comprehension Check

1. **What is the difference between `@click="handler"` and `@click="handler()"`?**
   - *Hint: Think about when the handler executes and what arguments it receives.*

2. **Why would you use `@submit.prevent` instead of calling `event.preventDefault()` inside your handler?**
   - *Hint: Consider code readability and the declarative nature of Vue templates.*

3. **True or False: Event listeners added via `@click` in Vue templates must be manually removed when the component unmounts to prevent memory leaks.**
   - *Hint: Consider what Vue handles automatically.*

---

**Reply 'next' for Lesson 11.2 (Practice).**
