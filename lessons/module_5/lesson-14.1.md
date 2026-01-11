# Lesson 14.1: Flexible Content with Slots — Theory & Concepts (Гибкий контент со слотами)

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 14.1 of 25 (Part 1 of 2) |
| **Topic** | Flexible Content with Slots / Слоты для гибкого контента |
| **Module** | Module 5: User Interaction |
| **Prerequisites** | Lessons 1-13 (Core Fundamentals, Component Communication, Custom v-model) |
| **You Should Know** | Component structure, props, template syntax, TypeScript generics (helpful) |
| **Unlocks** | Lesson 14.2, then Module 6 (Composables) |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 1 (Theory):**
1. **Remember:** Define slots, recall default/named/scoped slot syntax
2. **Understand:** Explain when to use slots vs props, how slot content is compiled, and the data flow in scoped slots

---

## 2. Real-World Scenario & Context

**Scenario:** You're building a reusable `Card` component for your design system. Different pages need cards with vastly different content: one shows a user avatar with actions, another displays a product with a buy button, and a third shows an article preview. Using only props would require dozens of optional properties and complex conditional rendering. Instead, you want the parent to "inject" its own content into predefined areas of your card.

**Framework Context:** Slots are Vue's mechanism for **content distribution**. They allow parent components to pass template content into designated areas of child components. This creates highly flexible, reusable components that define structure and behavior while delegating content decisions to their consumers.

---

## 3. Core Concepts Explained (Deep Dive)

### How It Actually Works

When Vue compiles a component with `<slot>`, it creates a "placeholder" that gets replaced by content from the parent. The content passed to a slot is compiled in the **parent's scope**, not the child's.

```vue
<!-- Card.vue (Child) -->
<template>
  <div class="card">
    <slot></slot>  <!-- This is a placeholder -->
  </div>
</template>

<!-- Parent.vue -->
<template>
  <Card>
    <h1>{{ title }}</h1>  <!-- 'title' comes from Parent's scope -->
  </Card>
</template>
```

```
┌──────────────────────────────────────────────────────────────────┐
│                   Slot Compilation Flow                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│   PARENT COMPONENT                 CHILD COMPONENT               │
│   ┌─────────────────────┐         ┌─────────────────────┐       │
│   │ <Card>              │         │ <div class="card">  │       │
│   │   <h1>Hello</h1>    │ ──────► │   <slot></slot>     │       │
│   │ </Card>             │         │ </div>              │       │
│   └─────────────────────┘         └─────────────────────┘       │
│            │                               │                     │
│            │    Vue Compilation            │                     │
│            ▼                               ▼                     │
│   ┌─────────────────────────────────────────────────────┐       │
│   │              RENDERED OUTPUT                         │       │
│   │  <div class="card">                                  │       │
│   │    <h1>Hello</h1>                                    │       │
│   │  </div>                                              │       │
│   └─────────────────────────────────────────────────────┘       │
│                                                                  │
│   The slot content replaces <slot></slot> in the output          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Types of Slots

```
┌──────────────────────────────────────────────────────────────────┐
│                     Slot Types Overview                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│   1. DEFAULT SLOT                                                │
│   ├── Unnamed, catches all content not targeting named slots     │
│   └── <slot></slot>                                              │
│                                                                  │
│   2. NAMED SLOTS                                                 │
│   ├── Multiple distinct content areas                            │
│   ├── <slot name="header"></slot>                                │
│   └── Parent uses: <template #header>...</template>              │
│                                                                  │
│   3. SCOPED SLOTS                                                │
│   ├── Child passes data back to parent's slot content            │
│   ├── <slot :item="item" :index="i"></slot>                      │
│   └── Parent uses: <template #default="{ item, index }">        │
│                                                                  │
│   4. CONDITIONAL/DYNAMIC SLOTS                                   │
│   ├── Check if slot has content: $slots.header                   │
│   └── Render only if content provided                            │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Mental Model

**Think of slots as "fill-in-the-blank" templates:**

Imagine a greeting card with blank spaces:
- "Dear ________" (named slot: `recipient`)
- "[MESSAGE HERE]" (default slot)
- "From, ________" (named slot: `sender`)

The card (component) defines the layout and structure. The person filling it out (parent) provides the specific content.

```
┌─────────────────────────────────────────────────────────────┐
│              📝 The Fill-in-the-Blank Card                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   CARD TEMPLATE (Child)          FILLED CARD (Rendered)     │
│   ┌───────────────────┐          ┌───────────────────┐     │
│   │ Dear _________    │          │ Dear Mom          │     │
│   │     #recipient    │   ───►   │                   │     │
│   │                   │          │ Happy Birthday!   │     │
│   │ [  MESSAGE HERE ] │          │ 🎂🎉              │     │
│   │     #default      │          │                   │     │
│   │                   │          │ From, Your Son    │     │
│   │ From, _________   │          │                   │     │
│   │     #sender       │          │                   │     │
│   └───────────────────┘          └───────────────────┘     │
│                                                             │
│   Parent provides: #recipient="Mom", #default="Happy...",   │
│                    #sender="Your Son"                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Scoped Slots: Child Passes Data Up

```
┌─────────────────────────────────────────────────────────────┐
│              🔄 Scoped Slots Data Flow                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   CHILD (has the data)           PARENT (renders it)        │
│   ┌───────────────────┐          ┌───────────────────┐     │
│   │                   │          │                   │     │
│   │  items = [...]    │          │  <List>           │     │
│   │                   │          │    <template      │     │
│   │  <slot           ─┼──────────┼─►  #item="{ item, │     │
│   │    :item="item"   │  passes  │        index }">  │     │
│   │    :index="i"     │   data   │      Custom       │     │
│   │  />               │          │      render here  │     │
│   │                   │          │    </template>    │     │
│   └───────────────────┘          └───────────────────┘     │
│                                                             │
│   Child says: "Here's the data, you decide how to show it"  │
│   Parent says: "Thanks! I'll render it my way"              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### In Other Contexts

| Framework | Content Projection | Notes |
|:--- |:--- |:--- |
| **Vue 3** | `<slot>`, `#name`, `v-slot:name` | Named slots, scoped slots, fallback |
| **React** | `props.children`, render props | `children` for default, custom props for "named" |
| **Angular** | `<ng-content>`, `<ng-template>` | Content projection with selectors |
| **Svelte** | `<slot>`, `<slot name="x">` | Similar to Vue, uses `let:` for scoped |
| **Web Components** | `<slot>`, `<slot name="x">` | Native browser API, Vue slots build on this |

### When to Use / When NOT to Use

| ✅ Use Slots When | ❌ Avoid Slots When |
|:--- |:--- |
| Content varies significantly between uses | Content is consistent (use props for data) |
| Parent needs full control over rendered markup | Child can render everything with simple prop data |
| Building layout components (Card, Modal, Layout) | Building leaf components (Button, Input) |
| Creating wrapper/container components | Data, not structure, changes between uses |
| Need to pass complex nested templates | A string prop or simple interpolation suffices |

---

## 4. New Terminology

| Term | Definition |
|:--- |:--- |
| **Slot** | A placeholder in a child component where parent-provided content is inserted. Enables content distribution patterns. |
| **Default Slot** | The unnamed slot that receives any content not explicitly targeting a named slot. Defined with `<slot></slot>`. |
| **Named Slot** | A slot with a specific name, allowing multiple distinct content areas. Defined with `<slot name="header">` and used with `<template #header>`. |
| **Scoped Slot** | A slot that passes data from child to parent, allowing the parent to use child data in its template. Defined with `<slot :data="value">` and consumed with `<template #default="{ data }">`. |
| **Fallback Content** | Default content rendered when the parent provides nothing for a slot. Placed between `<slot>` and `</slot>` tags. |

---

## 5. Algorithmic Thinking (Planning the Solution)

**The Plan:** Building a flexible Modal component with slots for header, body, and footer.

```
┌─────────────────────────────────────────────────────────────┐
│        Decision Tree: Designing a Slotted Component         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Step 1: Identify fixed vs flexible parts                  │
│   └── Fixed: overlay, animation, close behavior             │
│   └── Flexible: header content, body content, footer buttons│
│       └── These become slots!                               │
│                                                             │
│   Step 2: Determine slot types needed                       │
│   └── Header: named slot (optional, with fallback)          │
│   └── Body: default slot (required)                         │
│   └── Footer: named slot (optional, with fallback)          │
│       └── Consider: Does footer need data from modal?       │
│       └── Yes: Use scoped slot to pass close function       │
│                                                             │
│   Step 3: Design slot interfaces                            │
│   └── #header - no special data needed                      │
│   └── #default - main content area                          │
│   └── #footer="{ close }" - receives close function         │
│                                                             │
│   Step 4: Add fallback content                              │
│   └── Header fallback: "Modal Title"                        │
│   └── Footer fallback: Close button                         │
│                                                             │
│   Step 5: Check for slot presence                           │
│   └── Use $slots.header to conditionally render wrapper     │
│   └── Hide header section if no content and no fallback     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. Initial Pattern Introduction

| What You Want (Intent) | Code Chunk (The Pattern) | Conceptual Link |
|:--- |:--- |:--- |
| Basic content placeholder | `<slot></slot>` | Default slot |
| Fallback content | `<slot>Default text</slot>` | Shown when parent provides nothing |
| Named slot | `<slot name="header"></slot>` | Specific content area |
| Use named slot in parent | `<template #header>...</template>` | `#` is shorthand for `v-slot:` |
| Scoped slot (pass data) | `<slot :item="item"></slot>` | Child exposes data |
| Consume scoped slot | `<template #default="{ item }">` | Parent receives and uses data |
| Check slot existence | `$slots.header` or `useSlots()` | Conditional rendering |

### Basic Pattern: Card with Default Slot

```vue
<!-- Card.vue -->
<script setup lang="ts">
interface Props {
  variant?: 'default' | 'bordered' | 'elevated'
}

withDefaults(defineProps<Props>(), {
  variant: 'default'
})
</script>

<template>
  <div 
    class="rounded-lg p-4"
    :class="{
      'bg-base-100': variant === 'default',
      'border border-base-300': variant === 'bordered',
      'shadow-xl': variant === 'elevated'
    }"
  >
    <!-- Default slot: parent content goes here -->
    <slot>
      <!-- Fallback content if parent provides nothing -->
      <p class="text-base-content/50">No content provided</p>
    </slot>
  </div>
</template>
```

**Parent Usage:**
```vue
<template>
  <!-- With content -->
  <Card variant="elevated">
    <h2>Welcome!</h2>
    <p>This is my custom card content.</p>
  </Card>
  
  <!-- Without content (shows fallback) -->
  <Card />
</template>
```

### Named Slots Pattern: Layout Component

```vue
<!-- PageLayout.vue -->
<script setup lang="ts">
import { useSlots } from 'vue'

const slots = useSlots()
const hasSidebar = computed(() => !!slots.sidebar)
</script>

<template>
  <div class="min-h-screen flex flex-col">
    <!-- Header slot -->
    <header class="bg-primary text-primary-content p-4">
      <slot name="header">
        <h1>Default Header</h1>
      </slot>
    </header>
    
    <!-- Main area with optional sidebar -->
    <div class="flex flex-1">
      <!-- Sidebar (only if content provided) -->
      <aside v-if="hasSidebar" class="w-64 bg-base-200 p-4">
        <slot name="sidebar"></slot>
      </aside>
      
      <!-- Main content (default slot) -->
      <main class="flex-1 p-4">
        <slot></slot>
      </main>
    </div>
    
    <!-- Footer slot -->
    <footer class="bg-neutral text-neutral-content p-4">
      <slot name="footer">
        <p>© 2024 My App</p>
      </slot>
    </footer>
  </div>
</template>
```

**Parent Usage:**
```vue
<template>
  <PageLayout>
    <!-- Named slot: header -->
    <template #header>
      <nav class="flex justify-between">
        <span class="font-bold">MyApp</span>
        <a href="/login">Login</a>
      </nav>
    </template>
    
    <!-- Named slot: sidebar -->
    <template #sidebar>
      <ul class="menu">
        <li><a>Dashboard</a></li>
        <li><a>Settings</a></li>
      </ul>
    </template>
    
    <!-- Default slot: main content -->
    <h1>Welcome to the Dashboard</h1>
    <p>Your main content here...</p>
    
    <!-- Named slot: footer -->
    <template #footer>
      <div class="flex justify-between">
        <span>Custom Footer</span>
        <a href="/privacy">Privacy Policy</a>
      </div>
    </template>
  </PageLayout>
</template>
```

### Scoped Slots Pattern: List with Custom Rendering

```vue
<!-- DataList.vue -->
<script setup lang="ts" generic="T">
interface Props {
  items: T[]
  keyField?: keyof T
}

const props = withDefaults(defineProps<Props>(), {
  keyField: 'id' as keyof T
})
</script>

<template>
  <ul class="space-y-2">
    <li 
      v-for="(item, index) in items" 
      :key="(item as any)[keyField] ?? index"
    >
      <!-- Scoped slot: passes item data to parent -->
      <slot 
        :item="item" 
        :index="index"
        :is-first="index === 0"
        :is-last="index === items.length - 1"
      >
        <!-- Fallback: simple JSON display -->
        <pre>{{ item }}</pre>
      </slot>
    </li>
  </ul>
</template>
```

**Parent Usage:**
```vue
<script setup lang="ts">
interface User {
  id: number
  name: string
  email: string
  role: 'admin' | 'user'
}

const users: User[] = [
  { id: 1, name: 'Alice', email: 'alice@example.com', role: 'admin' },
  { id: 2, name: 'Bob', email: 'bob@example.com', role: 'user' },
]
</script>

<template>
  <DataList :items="users" key-field="id">
    <!-- Scoped slot with destructured props -->
    <template #default="{ item, index, isLast }">
      <div 
        class="flex justify-between p-2 bg-base-200 rounded"
        :class="{ 'mb-0': isLast }"
      >
        <span class="font-medium">{{ index + 1 }}. {{ item.name }}</span>
        <span class="badge" :class="item.role === 'admin' ? 'badge-primary' : 'badge-ghost'">
          {{ item.role }}
        </span>
      </div>
    </template>
  </DataList>
</template>
```

### Slot with Exposed Actions

```vue
<!-- Expandable.vue -->
<script setup lang="ts">
import { ref } from 'vue'

const isExpanded = ref(false)

function toggle(): void {
  isExpanded.value = !isExpanded.value
}

function open(): void {
  isExpanded.value = true
}

function close(): void {
  isExpanded.value = false
}
</script>

<template>
  <div class="border rounded-lg overflow-hidden">
    <!-- Header slot with actions -->
    <div class="p-4 bg-base-200 cursor-pointer" @click="toggle">
      <slot name="header" :is-expanded="isExpanded" :toggle="toggle">
        <span>Click to {{ isExpanded ? 'collapse' : 'expand' }}</span>
      </slot>
    </div>
    
    <!-- Content (conditionally rendered) -->
    <div v-show="isExpanded" class="p-4">
      <slot :close="close"></slot>
    </div>
  </div>
</template>
```

**Parent Usage:**
```vue
<template>
  <Expandable>
    <template #header="{ isExpanded }">
      <div class="flex justify-between items-center">
        <h3>FAQ Item</h3>
        <span>{{ isExpanded ? '▼' : '▶' }}</span>
      </div>
    </template>
    
    <template #default="{ close }">
      <p>Here's the answer to the question...</p>
      <button @click="close" class="btn btn-sm mt-2">Got it!</button>
    </template>
  </Expandable>
</template>
```

---

## 7. Comprehension Check

1. **What is the difference between a named slot and a scoped slot?**
   - *Hint: Think about what data flows in which direction.*

2. **If a parent component doesn't provide content for a slot, what happens?**
   - *Hint: Consider fallback content and the difference between an empty slot and no slot.*

3. **True or False: Content passed to a slot is compiled in the child component's scope, giving it access to the child's data.**
   - *Hint: Think about where the template content is written and whose variables it can access.*

---

**Reply 'next' for Lesson 14.2 (Practice).**
