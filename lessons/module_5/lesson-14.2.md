# Lesson 14.2: Flexible Content with Slots — Practice & Application (Гибкий контент со слотами)

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 14.2 of 25 (Part 2 of 2) |
| **Topic** | Flexible Content with Slots / Слоты для гибкого контента |
| **Continues From** | Lesson 14.1 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Build reusable components using default, named, and scoped slots
2. **Analyze:** Debug slot-related issues and decide when to use slots vs props

---

## 2. Quick Recap

- **Default slot:** `<slot></slot>` — unnamed placeholder for parent content
- **Named slots:** `<slot name="header">` — multiple distinct content areas
- **Scoped slots:** `<slot :data="value">` — child passes data to parent's slot content
- **Fallback content:** Content between `<slot>` tags shown when parent provides nothing
- **Slot existence check:** `$slots.name` or `useSlots()` for conditional rendering

---

## 3. The Variable Frame & Complexity Scale

**Basic Form:**
```vue
<script setup lang="ts">
</script>

<template>
  <div class="card">
    <slot>Default content here</slot>
  </div>
</template>
```

**With Named Slots:**
```vue
<script setup lang="ts">
import { useSlots } from 'vue'

const slots = useSlots()
</script>

<template>
  <div class="card">
    <header v-if="slots.header" class="card-header">
      <slot name="header"></slot>
    </header>
    <div class="card-body">
      <slot></slot>
    </div>
    <footer v-if="slots.footer" class="card-footer">
      <slot name="footer"></slot>
    </footer>
  </div>
</template>
```

**Advanced Form (Scoped Slots with Actions):**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

interface Props {
  items: string[]
}

const props = defineProps<Props>()

const selectedIndex = ref<number | null>(null)

const selectedItem = computed(() => 
  selectedIndex.value !== null ? props.items[selectedIndex.value] : null
)

function select(index: number): void {
  selectedIndex.value = index
}

function clear(): void {
  selectedIndex.value = null
}
</script>

<template>
  <div>
    <slot 
      name="list" 
      :items="items" 
      :selected-index="selectedIndex"
      :select="select"
    >
      <ul>
        <li v-for="(item, i) in items" :key="i" @click="select(i)">
          {{ item }}
        </li>
      </ul>
    </slot>
    
    <slot 
      name="selected"
      :item="selectedItem"
      :clear="clear"
    >
      <p v-if="selectedItem">Selected: {{ selectedItem }}</p>
    </slot>
  </div>
</template>
```

---

## 4. Micro-Implementation Drills

### Drill 1 (Basic): Alert Component with Slot
Create an alert component that displays slotted content with different styles.

```vue
<!-- Alert.vue -->
<script setup lang="ts">
interface Props {
  type?: 'info' | 'success' | 'warning' | 'error'
}

// TODO: Define props with default type 'info'
</script>

<template>
  <!-- TODO: Create an alert div with dynamic class based on type -->
  <!-- TODO: Add a slot for the alert content -->
  <!-- TODO: Add fallback content: "No message provided" -->
</template>
```

**Solution:**
```vue
<script setup lang="ts">
interface Props {
  type?: 'info' | 'success' | 'warning' | 'error'
}

const props = withDefaults(defineProps<Props>(), {
  type: 'info'
})

const alertClasses: Record<string, string> = {
  info: 'alert-info',
  success: 'alert-success',
  warning: 'alert-warning',
  error: 'alert-error'
}
</script>

<template>
  <div class="alert" :class="alertClasses[type]">
    <slot>No message provided</slot>
  </div>
</template>
```

---

### Drill 2 (Variation): Card with Header and Footer Slots
Create a card component with optional header and footer slots.

```vue
<!-- Card.vue -->
<script setup lang="ts">
import { useSlots } from 'vue'

// TODO: Use useSlots() to check for slot presence
</script>

<template>
  <div class="card bg-base-100 shadow-xl">
    <!-- TODO: Only render header if slot.header exists -->
    
    <div class="card-body">
      <!-- TODO: Default slot for main content -->
    </div>
    
    <!-- TODO: Only render footer if slots.footer exists -->
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { useSlots } from 'vue'

const slots = useSlots()
</script>

<template>
  <div class="card bg-base-100 shadow-xl">
    <div v-if="slots.header" class="card-header p-4 border-b">
      <slot name="header"></slot>
    </div>
    
    <div class="card-body">
      <slot></slot>
    </div>
    
    <div v-if="slots.footer" class="card-footer p-4 border-t">
      <slot name="footer"></slot>
    </div>
  </div>
</template>
```

**Parent Usage:**
```vue
<template>
  <Card>
    <template #header>
      <h2 class="font-bold">Card Title</h2>
    </template>
    
    <p>This is the main content of the card.</p>
    
    <template #footer>
      <button class="btn btn-primary">Action</button>
    </template>
  </Card>
</template>
```

---

### Drill 3 (Combination): Tabs Component with Scoped Slots
Create a tabs component where parent controls tab content rendering.

```vue
<!-- Tabs.vue -->
<script setup lang="ts">
import { ref } from 'vue'

interface Tab {
  id: string
  label: string
}

interface Props {
  tabs: Tab[]
}

const props = defineProps<Props>()

// TODO: Create activeTab ref initialized to first tab's id
// TODO: Create setActiveTab function
// TODO: Create isActive helper function
</script>

<template>
  <div>
    <!-- Tab buttons -->
    <div class="tabs tabs-boxed">
      <!-- TODO: v-for over tabs, highlight active, call setActiveTab on click -->
    </div>
    
    <!-- Tab content with scoped slot -->
    <div class="p-4">
      <!-- TODO: Slot that passes activeTab and setActiveTab to parent -->
    </div>
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

interface Tab {
  id: string
  label: string
}

interface Props {
  tabs: Tab[]
}

const props = defineProps<Props>()

const activeTab = ref(props.tabs[0]?.id ?? '')

function setActiveTab(id: string): void {
  activeTab.value = id
}

function isActive(id: string): boolean {
  return activeTab.value === id
}
</script>

<template>
  <div>
    <div class="tabs tabs-boxed mb-4">
      <button
        v-for="tab in tabs"
        :key="tab.id"
        class="tab"
        :class="{ 'tab-active': isActive(tab.id) }"
        @click="setActiveTab(tab.id)"
      >
        {{ tab.label }}
      </button>
    </div>
    
    <div class="p-4 bg-base-200 rounded-lg">
      <slot 
        :active-tab="activeTab" 
        :set-active-tab="setActiveTab"
      ></slot>
    </div>
  </div>
</template>
```

**Parent Usage:**
```vue
<script setup lang="ts">
const tabs = [
  { id: 'home', label: 'Home' },
  { id: 'profile', label: 'Profile' },
  { id: 'settings', label: 'Settings' }
]
</script>

<template>
  <Tabs :tabs="tabs">
    <template #default="{ activeTab }">
      <div v-if="activeTab === 'home'">
        <h2>Welcome Home!</h2>
      </div>
      <div v-else-if="activeTab === 'profile'">
        <h2>Your Profile</h2>
      </div>
      <div v-else-if="activeTab === 'settings'">
        <h2>Settings Panel</h2>
      </div>
    </template>
  </Tabs>
</template>
```

---

### Drill 4 (Edge Case): Render Function Slots for Dynamic Components
Create a component that uses scoped slots for a sortable list.

```vue
<!-- SortableList.vue -->
<script setup lang="ts" generic="T">
import { ref, computed } from 'vue'

interface Props {
  items: T[]
  keyExtractor: (item: T) => string | number
}

const props = defineProps<Props>()

const sortDirection = ref<'asc' | 'desc'>('asc')

// TODO: Create sorted items computed that reverses based on direction
// TODO: Create toggleSort function
</script>

<template>
  <div>
    <!-- Header slot with sort controls -->
    <slot 
      name="header" 
      :direction="sortDirection" 
      :toggle="toggleSort"
    >
      <button @click="toggleSort" class="btn btn-sm mb-2">
        Sort {{ sortDirection === 'asc' ? '↑' : '↓' }}
      </button>
    </slot>
    
    <!-- List items with scoped slot -->
    <ul class="space-y-2">
      <li v-for="(item, index) in sortedItems" :key="keyExtractor(item)">
        <slot 
          name="item"
          :item="item" 
          :index="index"
          :original-index="items.indexOf(item)"
        >
          {{ item }}
        </slot>
      </li>
    </ul>
    
    <!-- Empty state slot -->
    <slot v-if="items.length === 0" name="empty">
      <p class="text-gray-500">No items to display</p>
    </slot>
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts" generic="T">
import { ref, computed } from 'vue'

interface Props {
  items: T[]
  keyExtractor: (item: T) => string | number
}

const props = defineProps<Props>()

const sortDirection = ref<'asc' | 'desc'>('asc')

const sortedItems = computed(() => {
  const copy = [...props.items]
  return sortDirection.value === 'desc' ? copy.reverse() : copy
})

function toggleSort(): void {
  sortDirection.value = sortDirection.value === 'asc' ? 'desc' : 'asc'
}
</script>

<template>
  <div>
    <slot 
      name="header" 
      :direction="sortDirection" 
      :toggle="toggleSort"
    >
      <button @click="toggleSort" class="btn btn-sm mb-2">
        Sort {{ sortDirection === 'asc' ? '↑' : '↓' }}
      </button>
    </slot>
    
    <ul class="space-y-2">
      <li v-for="(item, index) in sortedItems" :key="keyExtractor(item)">
        <slot 
          name="item"
          :item="item" 
          :index="index"
          :original-index="items.indexOf(item)"
        >
          {{ item }}
        </slot>
      </li>
    </ul>
    
    <slot v-if="items.length === 0" name="empty">
      <p class="text-gray-500">No items to display</p>
    </slot>
  </div>
</template>
```

---

## 5. Common Pitfalls & Anti-Patterns

| ❌ Common Mistake | ✅ Correct Approach | Why It Matters |
|:--- |:--- |:--- |
| Trying to access child data in slot without scoped slot | Use `<slot :data="childData">` and `<template #default="{ data }">` | Slot content is compiled in parent scope; must explicitly pass data |
| Using `v-slot` on regular elements | Use `v-slot` only on `<template>` or component tags | `v-slot` directive only works on templates and components |
| Checking `slots.default` for emptiness | Slot always exists; check content with `slots.default?.()?.length` | `$slots.default` is a function, not the rendered content |

### Detailed Examples

**Mistake 1: Accessing Child Data**
```vue
<!-- ❌ WRONG: 'item' is not defined in parent scope -->
<!-- Child: ItemList.vue -->
<template>
  <div v-for="item in items">
    <slot></slot>  <!-- No data passed -->
  </div>
</template>

<!-- Parent -->
<template>
  <ItemList :items="items">
    <p>{{ item.name }}</p>  <!-- ERROR: 'item' is undefined -->
  </ItemList>
</template>

<!-- ✅ CORRECT: Pass data via scoped slot -->
<!-- Child: ItemList.vue -->
<template>
  <div v-for="item in items" :key="item.id">
    <slot :item="item"></slot>  <!-- Pass item to parent -->
  </div>
</template>

<!-- Parent -->
<template>
  <ItemList :items="items">
    <template #default="{ item }">
      <p>{{ item.name }}</p>  <!-- Works! -->
    </template>
  </ItemList>
</template>
```

**Mistake 2: v-slot on Wrong Element**
```vue
<!-- ❌ WRONG: v-slot on a div -->
<template>
  <MyComponent>
    <div v-slot:header>Content</div>  <!-- ERROR -->
  </MyComponent>
</template>

<!-- ✅ CORRECT: v-slot on template -->
<template>
  <MyComponent>
    <template #header>
      <div>Content</div>
    </template>
  </MyComponent>
</template>

<!-- ✅ ALSO CORRECT: v-slot directly on component for default slot only -->
<template>
  <MyComponent v-slot="{ data }">
    <div>{{ data }}</div>
  </MyComponent>
</template>
```

**Mistake 3: Checking Slot Existence**
```vue
<!-- ❌ WRONG: This doesn't check if slot has content -->
<script setup lang="ts">
import { useSlots } from 'vue'
const slots = useSlots()

// This is always truthy even if parent passes empty content
const hasHeader = !!slots.header
</script>

<!-- ✅ CORRECT: Check if slot function returns content -->
<script setup lang="ts">
import { useSlots, computed } from 'vue'
const slots = useSlots()

const hasHeader = computed(() => {
  const slotContent = slots.header?.()
  return slotContent && slotContent.length > 0
})
</script>
```

---

## 6. Mini-Project (Putting It Together)

**Modal Dialog Component**

A fully featured modal with slots for header, body, footer, and scoped actions.

**File Structure:**
```
src/
├── components/
│   └── Modal.vue
├── App.vue
└── main.ts
```

**Modal.vue:**
```vue
<script setup lang="ts">
import { ref, watch, onMounted, onUnmounted, useSlots } from 'vue'

interface Props {
  modelValue?: boolean
  title?: string
  closable?: boolean
  closeOnOverlay?: boolean
  size?: 'sm' | 'md' | 'lg' | 'full'
}

const props = withDefaults(defineProps<Props>(), {
  modelValue: false,
  title: '',
  closable: true,
  closeOnOverlay: true,
  size: 'md'
})

const emit = defineEmits<{
  'update:modelValue': [value: boolean]
  'close': []
  'confirm': []
}>()

const slots = useSlots()

const isOpen = ref(props.modelValue)

watch(() => props.modelValue, (newVal) => {
  isOpen.value = newVal
})

function close(): void {
  isOpen.value = false
  emit('update:modelValue', false)
  emit('close')
}

function confirm(): void {
  emit('confirm')
  close()
}

function handleOverlayClick(): void {
  if (props.closeOnOverlay) {
    close()
  }
}

function handleEscape(event: KeyboardEvent): void {
  if (event.key === 'Escape' && props.closable) {
    close()
  }
}

onMounted(() => {
  document.addEventListener('keydown', handleEscape)
})

onUnmounted(() => {
  document.removeEventListener('keydown', handleEscape)
})

const sizeClasses: Record<string, string> = {
  sm: 'max-w-sm',
  md: 'max-w-md',
  lg: 'max-w-2xl',
  full: 'max-w-full mx-4'
}
</script>

<template>
  <Teleport to="body">
    <Transition name="modal">
      <div 
        v-if="isOpen" 
        class="fixed inset-0 z-50 flex items-center justify-center"
      >
        <div 
          class="absolute inset-0 bg-black/50"
          @click="handleOverlayClick"
        />
        
        <div 
          class="relative bg-base-100 rounded-lg shadow-xl w-full"
          :class="sizeClasses[size]"
          role="dialog"
          aria-modal="true"
        >
          <div 
            v-if="slots.header || title || closable" 
            class="flex items-center justify-between p-4 border-b"
          >
            <slot name="header" :close="close">
              <h3 class="text-lg font-semibold">{{ title }}</h3>
            </slot>
            
            <button
              v-if="closable"
              @click="close"
              class="btn btn-ghost btn-sm btn-circle"
              aria-label="Close modal"
            >
              ✕
            </button>
          </div>
          
          <div class="p-4 max-h-[60vh] overflow-y-auto">
            <slot :close="close" :confirm="confirm">
              <p class="text-base-content/70">Modal content goes here</p>
            </slot>
          </div>
          
          <div 
            v-if="slots.footer" 
            class="flex justify-end gap-2 p-4 border-t"
          >
            <slot name="footer" :close="close" :confirm="confirm">
              <button @click="close" class="btn btn-ghost">Cancel</button>
              <button @click="confirm" class="btn btn-primary">Confirm</button>
            </slot>
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<style scoped>
.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.2s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-active .relative,
.modal-leave-active .relative {
  transition: transform 0.2s ease;
}

.modal-enter-from .relative,
.modal-leave-to .relative {
  transform: scale(0.95);
}
</style>
```

**App.vue Usage:**
```vue
<script setup lang="ts">
import { ref } from 'vue'
import Modal from './components/Modal.vue'

const isDeleteModalOpen = ref(false)
const isFormModalOpen = ref(false)

const itemToDelete = ref('Document.pdf')

function handleDeleteConfirm(): void {
  console.log(`Deleting ${itemToDelete.value}`)
}
</script>

<template>
  <div class="p-8 space-y-4">
    <h1 class="text-2xl font-bold">Modal Examples</h1>
    
    <div class="flex gap-4">
      <button @click="isDeleteModalOpen = true" class="btn btn-error">
        Delete Item
      </button>
      <button @click="isFormModalOpen = true" class="btn btn-primary">
        Open Form
      </button>
    </div>
    
    <!-- Confirmation Modal -->
    <Modal 
      v-model="isDeleteModalOpen" 
      title="Confirm Delete"
      size="sm"
      @confirm="handleDeleteConfirm"
    >
      <template #default>
        <p>Are you sure you want to delete <strong>{{ itemToDelete }}</strong>?</p>
        <p class="text-sm text-error mt-2">This action cannot be undone.</p>
      </template>
      
      <template #footer="{ close, confirm }">
        <button @click="close" class="btn btn-ghost">Cancel</button>
        <button @click="confirm" class="btn btn-error">Delete</button>
      </template>
    </Modal>
    
    <!-- Form Modal with Custom Header -->
    <Modal 
      v-model="isFormModalOpen"
      size="lg"
      :close-on-overlay="false"
    >
      <template #header="{ close }">
        <div class="flex items-center gap-2">
          <span class="text-2xl">📝</span>
          <div>
            <h3 class="font-semibold">Edit Profile</h3>
            <p class="text-sm text-base-content/70">Update your information</p>
          </div>
        </div>
      </template>
      
      <template #default="{ close }">
        <form @submit.prevent class="space-y-4">
          <div>
            <label class="label">Name</label>
            <input type="text" class="input input-bordered w-full" />
          </div>
          <div>
            <label class="label">Email</label>
            <input type="email" class="input input-bordered w-full" />
          </div>
          <div>
            <label class="label">Bio</label>
            <textarea class="textarea textarea-bordered w-full" rows="3"></textarea>
          </div>
        </form>
      </template>
      
      <template #footer="{ close }">
        <button @click="close" class="btn btn-ghost">Discard</button>
        <button @click="close" class="btn btn-primary">Save Changes</button>
      </template>
    </Modal>
  </div>
</template>
```

**How to Test:**
1. Create new Vue project with TypeScript and Tailwind/DaisyUI
2. Add the `Modal.vue` component
3. Test: Open modals with buttons, try closing with overlay, X button, and Escape key
4. Verify: Different slot configurations work correctly
5. Confirm: Scoped slot actions (close, confirm) function as expected

---

## 7. Implementation Exercises (Progressive)

### Level 1 (Recall): Fill in the Blanks
```vue
<!-- Complete the named slot syntax -->
<template>
  <Card>
    <_______ _____="header">
      <h1>My Title</h1>
    </_______>
    
    <p>Main content here</p>
  </Card>
</template>
```

**Answer:** `<template #header>` or `<template v-slot:header>`

---

### Level 2 (Apply): Independent Coding Tasks

**Exercise 2.1: Accordion Component**
Create an accordion component that:
- Accepts an array of items with `title` and `id`
- Uses named slot `title` for custom title rendering
- Uses scoped slot for content with `item` and `isOpen` passed
- Only one item can be open at a time

**Exercise 2.2: Data Table with Slot Columns**
Create a table component that:
- Accepts `columns` and `data` props
- Uses scoped slot for each column to allow custom cell rendering
- Provides `row`, `column`, and `value` in the slot scope

---

### Level 3 (Debug): Fix the Broken Code
```vue
<!-- Child: Dropdown.vue -->
<script setup lang="ts">
import { ref } from 'vue'

const isOpen = ref(false)
const selected = ref(null)

function select(item) {
  selected.value = item
  isOpen.value = false
}
</script>

<template>
  <div class="dropdown">
    <button @click="isOpen = !isOpen">
      <slot name="trigger">Select...</slot>
    </button>
    
    <ul v-if="isOpen">
      <slot :select="select" :selected="selected"></slot>
    </ul>
  </div>
</template>

<!-- Parent usage (has bugs!) -->
<template>
  <Dropdown>
    <div v-slot:trigger>
      {{ selected || 'Choose option' }}
    </div>
    
    <template #default="slotProps">
      <li 
        v-for="opt in options" 
        @click="slotProps.select(opt)"
      >
        {{ opt }}
      </li>
    </template>
  </Dropdown>
</template>
```

**Bugs to Find:**
1. `v-slot:trigger` cannot be on a `<div>`, must be on `<template>`
2. `selected` in parent references nothing (should use scoped slot's selected)
3. Missing `:key` in v-for

**Fixed Code:**
```vue
<!-- Parent usage -->
<script setup lang="ts">
const options = ['Apple', 'Banana', 'Cherry']
</script>

<template>
  <Dropdown>
    <template #trigger>
      Choose option
    </template>
    
    <template #default="{ select, selected }">
      <li 
        v-for="opt in options"
        :key="opt"
        @click="select(opt)"
        :class="{ 'font-bold': selected === opt }"
      >
        {{ opt }}
      </li>
    </template>
  </Dropdown>
</template>
```

---

## Module 5 Checkpoint

Congratulations! You've completed **Module 5: User Interaction**. You now know how to:

- ✅ Handle user events with `@click`, modifiers, and keyboard shortcuts
- ✅ Create two-way bindings with `v-model` on native form elements
- ✅ Build custom `v-model` components using `defineModel()`
- ✅ Create flexible, reusable components with slots

**Next Steps:** Module 6 will teach you how to extract and share logic across components using **Composables**.

---

**Reply 'next' for Lesson 15.1 (Theory) — Reusable Logic with Composables.**
