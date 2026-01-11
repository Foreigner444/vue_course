### Lesson: Reactive Primitives with `ref()`
(Реактивные примитивы с `ref()`)
---

### **Part 1: Foundation & Theory**
*(Est. Time: 15 minutes)*

#### 1. Lesson Metadata
| Field | Value |
|:--- |:--- |
| **Lesson Number** | 2 of 23 |
| **Complexity** | Minimal |
| **Prerequisites** | Lesson 1 (Your First Component) |
| **Core Patterns** | 1 new (`ref()`) + 1 recycled (SFC structure) |
| **Key Pattern-Chunk** | `const myValue = ref('initial');` |
| **Algorithmic Focus** | Managing changeable state |

#### 2. Real-World Scenario & Context
*   **Scenario:** The "Welcome Banner" component from Lesson 1 is great, but it's static. Your team lead now wants it to be dynamic. It needs to display a user's name and a live count of their unread notifications. You need a way to store this information and have the component automatically update when it changes.
*   **Framework Context:** Vue's primary job is to "react" to data changes and update the view (the HTML). A normal JavaScript variable (`let name = 'Alex'`) is not "reactive." If you change it, Vue has no idea it needs to update the template. Vue solves this with its reactivity system, and `ref()` is our primary tool for making simple values reactive.

#### 3. Core Concepts Explained
*   **How It Actually Works:** When you use `ref()`, you aren't just storing a value (like a string or number). Vue wraps that value inside a special object with a `.value` property. This object acts as a "reactive reference." When you use this `ref` in your template, Vue sees it and makes a note: "This part of the HTML depends on this `ref`." Later, when you change the value from your script *by using the `.value` property* (e.g., `myRef.value = 'new value'`), the special object notifies Vue. Vue then finds all the parts of the HTML that depend on that `ref` and updates them automatically.
    *   *Visual Instruction:* Process Flow
        ```
        1. SCRIPT: Create a reactive value
           const count = ref(0);
                 |
                 V
        2. TEMPLATE: Display the value
           <p>{{ count }}</p>  // Vue notes this <p> depends on `count`
                 |
                 V
        3. SCRIPT: A function changes the value
           count.value++;
                 |
                 V
        4. VUE: Reactivity system is notified
                 |
                 V
        5. TEMPLATE: The <p> automatically updates its content to '1'
        ```

*   **Mental Model:** The **"Smart Box"**. Think of `ref()` as creating a smart box.
    1.  You put your value (e.g., the number `10`) inside the box: `const notificationCount = ref(10);`
    2.  In your JavaScript code, if you want to see or change what's in the box, you must explicitly open it using `.value`: `notificationCount.value++`.
    3.  In your `<template>`, Vue is smart enough to automatically open the box for you. You can just write `{{ notificationCount }}` and it will display `11`.
    *   *Visual Instruction:* Conceptual Diagram
        ```
        // In your <script>
           notificationCount   ----->   [ A smart box: { .value: 10 } ]
              (You must use .value to access the content)

        // In your <template>
           {{ notificationCount }} ----> Vue automatically looks inside the box and displays "10"
        ```

*   **In Other Contexts:** This is a core concept in all modern UI frameworks. In **React**, the `useState` hook provides similar functionality: `const [count, setCount] = useState(0);`. Calling `setCount(1)` triggers a re-render, just like changing `.value` does in Vue.

#### 4. New Terminology
*   **Reactivity:** The mechanism by which a framework automatically updates the user interface to match changes in data (state).
*   **State:** The data that your application needs to remember and that can change over time (e.g., usernames, counters, form inputs).
*   **Primitive:** A basic JavaScript data type that is not an object. The main ones are `string`, `number`, `boolean`, `null`, and `undefined`. `ref()` is primarily used for these.
*   **`ref()`:** A Vue function that takes a primitive value and returns a reactive "smart box" (a reactive reference object).

#### 5. Algorithmic Thinking (Planning the Solution)
*   **The Plan (Narrative):**
    1.  "First, our component needs to store a username and a notification count. Since this data will change, we can't use plain JavaScript variables. We need to make them reactive. For simple values like a string and a number, `ref()` is the perfect tool. So, we'll import `ref` from 'vue' and create our state."
    2.  "Second, we need to display this state in our template. We'll use the 'mustache' syntax `{{ }}` to render the `ref` values directly in our HTML."
    3.  "Third, we need a way to change the state. We'll add a button to the template. When clicked, it will call a JavaScript function in our script."
    4.  "Finally, inside that function, we'll modify the notification count. The crucial part is to remember to use `.value` to change the content of our `ref` 'box'."
    *   *Visual Instruction:* A simple flowchart for the notification update.
        ```
        (User) ---clicks--> [Button] --triggers--> [increment() function] --modifies--> [count.value] --notifies--> (Vue) --updates--> [Template]
        ```

---
### **Part 2: Core Pattern Implementation**
*(Est. Time: 25 minutes)*

#### 6. The Intent Map (Concept → Code)
| What You Want (Intent) | Code Chunk (The Pattern) | **Conceptual Link** | Pattern Note |
|:--- |:--- |:--- |:--- |
| Make a primitive value reactive | `const count = ref(0);` | **Reactivity:** Wraps the number `0` in a reactive object. | Pattern-chunk: Use `ref()` for numbers, strings, booleans. |
| Import the `ref` function | `import { ref } from 'vue';` | **Modularity:** Vue provides its core functions as named exports. | This line is essential and always goes at the top of `<script setup>`. |
| Change the value in your script | `count.value++;` | **`.value` property:** The actual value is stored in the `.value` property of the ref object. | **Crucial:** Always use `.value` in the script section. Forgetting this is the #1 mistake. |
| Display the value in the template | `<h1>{{ count }}</h1>` | **Auto-unwrapping:** In the template, Vue automatically accesses `.value` for you. | You do *not* use `.value` in the template. |

#### 7. The Variable Frame & Complexity Scale
*   **Frame 1: Creating a Reactive Primitive**
    *   **Fixed:** `const [name] = ref<[Type]>([initialValue]);`
    *   **Slot (`name`):** The variable name for your state (e.g., `userName`, `score`, `isActive`).
    *   **Slot (`Type`):** The TypeScript type (e.g., `string`, `number`, `boolean`).
    *   **Slot (`initialValue`):** The starting value (e.g., `'Guest'`, `0`, `false`).
    *   **Examples:**
        1.  `const userName = ref<string>('Guest');`
        2.  `const score = ref<number>(0);`
        3.  `const isActive = ref<boolean>(false);`

*   **Complexity Scale: A Reactive Counter**
    ```
    ├── Quick & dirty: `<p>{{ count }}</p>` (Assumes `count` is somehow reactive)
    ├── Standard: `const count = ref(0);` and `<p>{{ count }}</p>` (Working, but no way to change it)
    ├── Robust: `const count = ref(0); function inc() { count.value++ }` and `<button @click="inc">` (Full implementation)
    ├── Production: (Same as robust, but the composable logic might be extracted for reuse - we'll learn this later).
    ```

#### 8. Micro-Implementation Drills
*   **Drill A (Sequencing): Create a reactive username.**
    1.  `import { ref } from 'vue';` // First, import what you need.
    2.  `const username = ref('Alice');` // Second, declare your reactive state.
    3.  `function changeName() { username.value = 'Bob'; }` // Third, create a function to change it.
    4.  `<p>{{ username }}</p><button @click="changeName">Change</button>` // Finally, use it in the template.

*   **Drill B (Data Flow): Show a live character count for an input.**
    ```vue
    <script setup lang="ts">
    import { ref } from 'vue';
    const message = ref(''); // User types here
    </script>
    <template>
      <input v-model="message"> <!-- We will learn v-model soon! -->
      <p>Character count: {{ message.length }}</p> <!-- Template automatically updates as message changes -->
    </template>
    ```

#### 9. Code Collocations & Fixed Patterns
| Import | Usage Pattern |
|:--- |:--- |
| `import { ref } from 'vue';` | `const myVar = ref(...);` |
| `const myVar = ref(...);` | In script: `myVar.value` |
| `const myVar = ref(...);` | In template: `{{ myVar }}` |

---
### **Part 3: Application & Refinement**
*(Est. Time: 20 minutes)*

#### 10. Mini-Project (Putting It Together)
*A complete, runnable "Notification Counter" component.*
```vue
<!-- NotificationCounter.vue -->

<script setup lang="ts">
// 1. Import `ref` from 'vue' so we can create reactive state.
import { ref } from 'vue';

// 2. Create a reactive variable `notificationCount`.
//    We start it at 0. `ref()` puts it in a "smart box".
const notificationCount = ref(0);

// 3. Create a function that will be called by our button.
function addNotification() {
  // 4. To change the value inside the `ref` box, we must use the `.value` property.
  notificationCount.value++;
}

function clearNotifications() {
  // We can also set the value directly.
  notificationCount.value = 0;
}
</script>

<template>
  <!-- 5. This whole div is styled with Tailwind and DaisyUI classes. -->
  <div class="card bg-base-200 shadow-lg">
    <div class="card-body items-center text-center">
      <h2 class="card-title">Notifications</h2>
      
      <!-- 6. We display the reactive `notificationCount` here. -->
      <!--    Vue automatically "unboxes" the value for us. No .value needed! -->
      <p class="text-5xl font-bold">{{ notificationCount }}</p>

      <div class="card-actions justify-end">
        <!-- 7. When this button is clicked, it calls our `addNotification` function. -->
        <button class="btn btn-success" @click="addNotification">Add Notification</button>
        
        <!-- 8. This button calls the `clearNotifications` function. -->
        <button class="btn btn-ghost" @click="clearNotifications">Clear</button>
      </div>
    </div>
  </div>
</template>
```

#### 11. Implementation Exercise
*   **Phase 1: Write:**
    1.  **Task (★):** Import `ref` from `vue`.
    2.  **Task (★):** Create a reactive variable named `score` and initialize it to `100`.
    3.  **Task (★★):** Display the `score` inside an `<h2>` tag in your template.
    4.  **Task (★★):** Create a function `increaseScore` that adds 10 to the score. Remember to use `.value`.
    5.  **Task (★★★):** Create a button that calls `increaseScore` when you click it.

*   **Phase 2: Self-Check:**
    ```vue
    <script setup lang="ts">
    import { ref } from 'vue';

    // Creates the reactive score state
    const score = ref(100);

    // Creates the function to modify the state
    function increaseScore() {
      score.value += 10;
    }
    </script>

    <template>
      <div>
        <!-- Displays the reactive state -->
        <h2>Score: {{ score }}</h2>
        
        <!-- Button to trigger the state change -->
        <button class="btn" @click="increaseScore">Add 10 Points</button>
      </div>
    </template>
    ```
