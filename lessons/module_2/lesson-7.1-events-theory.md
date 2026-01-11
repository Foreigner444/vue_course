# Lesson 7.1: Component Communication: Emitting Events — Theory & Concepts (Эмиссия Событий)
---
## 1. Lesson Metadata
| Field | Value |
|:--- |:--- |
| **Lesson Number** | 7.1 of 25 (Part 1 of 2) |
| **Topic** | Emitting Events / Эмиссия Событий |
| **Module** | Module 2: Component Communication |
| **Prerequisites** | Lesson 6 (Passing Props) |
| **You Should Know** | TypeScript function types, callback patterns, event handling basics |
| **Unlocks** | Lesson 7.2, then Module 3 (Styling) |
| **Duration** | 30-35 minutes |
**Learning Objectives — Part 1 (Theory):**
1. **Remember:** Define component events and list the steps to emit typed events
2. **Understand:** Explain how child-to-parent communication completes the data flow pattern
---
## 2. Real-World Scenario & Context
**Scenario:** You're building a product listing page for an e-commerce site. Each product card has an "Add to Cart" button. When clicked, the parent container (which manages the shopping cart state) needs to know which product was added. The child component (ProductCard) has the button, but the cart data lives in the parent. The child needs to send a message upward: "Hey parent, the user wants to add this product!"
**Framework Context:** Vue solves child-to-parent communication through **custom events**. While props flow data *down* (parent → child), events flow data *up* (child → parent). This completes Vue's unidirectional data flow pattern: "props down, events up."
---
## 3. Core Concepts Explained (Deep Dive)
### 3.1 How Events Actually Work
When a child component emits an event, Vue's event system notifies the parent. Here's the full flow:
```
┌─────────────────────────────────────────────────────────────────┐
│                        CHILD COMPONENT                          │
│                                                                 │
│   const emit = defineEmits<{                                   │
│     addToCart: [productId: number, quantity: number]           │
│   }>()                                                          │
│                                                                 │
│   ┌─────────────────────┐                                       │
│   │  Button: "Add"      │ ──click──►  emit('addToCart', 42, 1) │
│   └─────────────────────┘                                       │
│                                                                 │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                      Vue's Event System
                    (bubbles event to parent)
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                        PARENT COMPONENT                         │
│                                                                 │
│   <ProductCard @add-to-cart="handleAdd" />                     │
│                                                                 │
│   function handleAdd(productId: number, quantity: number) {    │
│     cart.value.push({ productId, quantity })                   │
│   }                                                            │
│                                                                 │
│   ✅ Parent receives event with payload                        │
│   ✅ Parent decides what to do with the data                   │
│   ✅ Parent owns and modifies its own state                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```
**Key Points:**
1. Child **declares** what events it can emit using `defineEmits<T>()`
2. Child **triggers** the event using `emit('eventName', ...payload)`
3. Parent **listens** using `@event-name="handler"` in the template
4. Parent's handler **receives** the payload and can update its own state
### 3.2 The Complete Communication Pattern
Props and events together form a complete communication loop:
```
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│   ┌─────────────┐                      ┌─────────────┐        │
│   │   PARENT    │                      │   PARENT    │        │
│   │   STATE     │                      │   HANDLER   │        │
│   │             │                      │             │        │
│   │ product: {} │                      │ addToCart() │        │
│   └──────┬──────┘                      └──────▲──────┘        │
│          │                                    │               │
│          │ Props (down)          Events (up)  │               │
│          │ :product="product"    @add="..."   │               │
│          │                                    │               │
│          ▼                                    │               │
│   ┌──────────────────────────────────────────┴──────┐        │
│   │                                                  │        │
│   │              CHILD COMPONENT                     │        │
│   │                                                  │        │
│   │   Receives prop ──► Displays data               │        │
│   │                                                  │        │
│   │   User clicks ──► emit('add', payload)          │        │
│   │                                                  │        │
│   └──────────────────────────────────────────────────┘        │
│                                                                │
│         "Props Down, Events Up" — Vue's Data Flow             │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```
### 3.3 Mental Model: The "Walkie-Talkie" Pattern
Think of props and events like a walkie-talkie system:
```
┌──────────────────┐         ┌──────────────────┐
│     PARENT       │         │      CHILD       │
│   (Base Camp)    │         │   (Field Team)   │
│                  │         │                  │
│  Sends orders    │ ──────► │  Receives orders │
│  via Props       │  Props  │  (read-only)     │
│                  │         │                  │
│  Receives        │ ◄────── │  Reports back    │
│  reports         │ Events  │  via Events      │
│                  │         │                  │
│  Makes decisions │         │  Cannot make     │
│  & updates state │         │  state decisions │
└──────────────────┘         └──────────────────┘
```
- **Parent (Base Camp):** Has authority, owns the state, makes decisions
- **Child (Field Team):** Executes tasks, reports observations, doesn't decide
- **Props:** Orders/instructions flowing down
- **Events:** Reports/observations flowing up
### 3.4 In Other Contexts
| Framework | Events Equivalent | Key Difference |
|:--- |:--- |:--- |
| **React** | Callback props: `onAdd={() => {...}}` | No special emit system; just pass functions as props |
| **Angular** | `@Output()` + `EventEmitter` | Similar pattern, uses decorator and RxJS-style emitter |
| **Svelte** | `createEventDispatcher()` | Similar to Vue's emit pattern |
| **Vanilla JS** | `CustomEvent` + `dispatchEvent()` | Manual event creation and DOM event bubbling |
**Note:** React doesn't distinguish between "props" and "events" — callback functions are just props. Vue's explicit `emit` pattern makes the intent clearer and enables better TypeScript integration.
### 3.5 When to Use / When NOT to Use Events
**✅ Use Events When:**
- Child needs to notify parent of user actions (clicks, form submissions)
- Child needs to send data back to parent
- You want to maintain one-way data flow
- The parent needs to decide what happens with the data
**❌ Don't Use Events When:**
- You need two-way binding (use `v-model` with `defineModel()` — Lesson 13)
- Communicating with deeply nested components (use `provide/inject` or Pinia)
- Siblings need to communicate (use shared state — Module 9)
- The action is purely internal to the child (handle it locally)
---
## 4. New Terminology
| Term | Definition |
|:--- |:--- |
| **Custom Events** | User-defined events that components can emit to communicate with their parents, distinct from native DOM events |
| **`defineEmits<T>()`** | Vue 3's macro for declaring typed custom events a component can emit |
| **Event Payload** | Data sent along with an event; can be any type (primitive, object, array) |
| **Event Handler** | A function in the parent component that runs when the child emits an event |
| **Kebab-case Convention** | Vue automatically converts `camelCase` event names to `kebab-case` in templates (`@add-to-cart`) |
---
## 5. Algorithmic Thinking (Planning the Solution)
**The Plan for Implementing Events:**
```
START: Child needs to notify parent
           │
           ▼
┌─────────────────────────────────────┐
│ STEP 1: Define the Events Interface │
│ What events will this component     │
│ emit? What data does each carry?    │
└─────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────┐
│ STEP 2: Declare Emits in Child      │
│ Use defineEmits<EventInterface>()   │
│ TypeScript enforces payload types   │
└─────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────┐
│ STEP 3: Emit at the Right Moment    │
│ Call emit('eventName', payload)     │
│ Usually in response to user action  │
└─────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────┐
│ STEP 4: Listen in Parent Template   │
│ Use @event-name="handlerFunction"   │
│ Note: camelCase → kebab-case        │
└─────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────┐
│ STEP 5: Handle in Parent Script     │
│ Write handler that receives payload │
│ Update parent state as needed       │
└─────────────────────────────────────┘
           │
           ▼
END: Parent reacts to child's event
```
---
## 6. Initial Pattern Introduction
| What You Want (Intent) | Code Chunk (The Pattern) | Conceptual Link |
|:--- |:--- |:--- |
| Declare what events child can emit | `defineEmits<{ eventName: [arg: Type] }>()` | Establishes the event contract |
| Trigger an event from child | `emit('eventName', payload)` | Sends the message upward |
| Listen for event in parent | `<Child @event-name="handler" />` | Receives the message |
| Handle event with payload | `function handler(payload: Type) { }` | Processes the message |
### Basic Events Pattern
```vue
<!-- ChildButton.vue -->
<script setup lang="ts">
// STEP 1 & 2: Define the events interface using TypeScript
// defineEmits<T>() tells Vue what events this component can emit
// The type describes event names and their payload types
// Modern syntax (Vue 3.3+): Use call signature format
const emit = defineEmits<{
  // Event 'click' carries no payload
  click: []
  // Event 'submit' carries a string payload
  submit: [value: string]
  // Event 'update' carries a number payload
  update: [id: number, newValue: number]
}>()
// STEP 3: Emit events in response to user actions
function handleClick() {
  // Emit 'click' with no payload
  emit('click')
}
function handleSubmit() {
  // Emit 'submit' with the input value
  emit('submit', 'Hello from child!')
}
function handleUpdate() {
  // Emit 'update' with multiple arguments
  emit('update', 42, 100)
}
</script>
<template>
  <div class="flex gap-2">
    <button @click="handleClick" class="btn">
      Click Me
    </button>
    <button @click="handleSubmit" class="btn btn-primary">
      Submit
    </button>
    <button @click="handleUpdate" class="btn btn-secondary">
      Update
    </button>
  </div>
</template>
```
```vue
<!-- ParentComponent.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import ChildButton from './ChildButton.vue'
const clickCount = ref(0)
const lastMessage = ref('')
const lastUpdate = ref({ id: 0, value: 0 })
// STEP 4 & 5: Handle events from child
// Handler function names are arbitrary — choose descriptive names
function onChildClick() {
  clickCount.value++
  console.log('Child was clicked!')
}
function onChildSubmit(value: string) {
  // `value` is the payload from emit('submit', value)
  lastMessage.value = value
  console.log('Received:', value)
}
function onChildUpdate(id: number, newValue: number) {
  // Multiple arguments come as separate parameters
  lastUpdate.value = { id, value: newValue }
  console.log('Update:', id, newValue)
}
</script>
<template>
  <div class="space-y-4">
    <!-- Listen using @event-name syntax -->
    <!-- Note: camelCase in emit → kebab-case in template is optional -->
    <!-- Both @submit and @Submit work, but kebab-case is convention -->
    <ChildButton 
      @click="onChildClick"
      @submit="onChildSubmit"
      @update="onChildUpdate"
    />
    
    <div class="stats">
      <p>Clicks: {{ clickCount }}</p>
      <p>Last message: {{ lastMessage }}</p>
      <p>Last update: ID {{ lastUpdate.id }}, Value {{ lastUpdate.value }}</p>
    </div>
  </div>
</template>
```
### Events with Validation (Runtime Checks)
```vue
<script setup lang="ts">
// For runtime validation, use object syntax
// This runs validation BEFORE the event is emitted
const emit = defineEmits({
  // Validator function returns true if valid
  submit: (email: string) => {
    if (!email.includes('@')) {
      console.warn('Invalid email format')
      return false // Event still emits, but warns in dev
    }
    return true
  },
  // Simple declaration (no validation)
  cancel: null
})
function handleSubmit(email: string) {
  emit('submit', email) // Validation runs here
}
</script>
```
### Inline Event Handling in Parent
```vue
<!-- Parent can handle events inline for simple cases -->
<template>
  <!-- Direct expression -->
  <Counter @increment="count++" />
  
  <!-- Arrow function for payload access -->
  <SearchBox @search="(query) => performSearch(query)" />
  
  <!-- $event gives access to payload in inline handlers -->
  <RatingStars @rate="console.log('Rated:', $event)" />
</template>
```
---
## 7. Comprehension Check
1. **What's the purpose of `defineEmits()` and why is it necessary?**
   - Think about TypeScript integration and Vue's need to know what events exist.
2. **In the template, you write `@add-to-cart="handler"`. In the child's emit call, you write `emit('addToCart', ...)`. Why the difference?**
   - Consider Vue's naming conventions and HTML attribute limitations.
3. **True or False: When emitting an event, the child component should also update the parent's state directly if it has access.**
   - Consider the separation of concerns and who "owns" the state.
---
**Reply 'next' for Lesson 7.2 (Practice).**
