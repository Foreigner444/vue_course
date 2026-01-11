# Guided Project: Rich Modal Dialog

---

## Project Overview

| Field | Value |
|:--- |:--- |
| **Module** | Module 5: User Interaction |
| **Project Number** | 3 of 3 |
| **Difficulty** | Intermediate |
| **Estimated Time** | 45-60 minutes |

**What You'll Build:** A fully-featured modal dialog component with customizable header, body, and footer using slots. The modal supports confirm/cancel actions, keyboard shortcuts, focus trapping, animations, and multiple sizing options.

**Real-World Scenario:** Every application needs modal dialogs for confirmations, forms, and alerts. You'll build a production-ready modal component that can be reused across the entire application with full customization through slots while maintaining consistent behavior and accessibility.

---

## Patterns You'll Practice

| Pattern | How It's Used |
|:--- |:--- |
| Named Slots | Customizable header, body, footer sections |
| Scoped Slots | Passing close/confirm actions to slot content |
| `defineModel()` | Two-way binding for open/close state |
| Event Handling | Keyboard shortcuts, overlay clicks |
| `Teleport` | Rendering modal outside component tree |
| `Transition` | Smooth open/close animations |
| `useSlots()` | Conditional rendering based on slot content |
| Focus Management | Trapping focus inside modal |

---

## Component Structure

```
src/
├── components/
│   └── modal/
│       ├── Modal.vue
│       ├── ModalHeader.vue
│       ├── ModalBody.vue
│       └── ModalFooter.vue
├── composables/
│   └── useFocusTrap.ts
├── App.vue
└── main.ts
```

---

## Step-by-Step Guide

### Step 1: Project Setup

```bash
npm create vue@latest modal-dialog
cd modal-dialog
npm install
npm install -D tailwindcss postcss autoprefixer daisyui
npx tailwindcss init -p
```

Configure Tailwind as shown in previous projects.

---

### Step 2: Create Focus Trap Composable

**File: `src/composables/useFocusTrap.ts`**

```typescript
import { ref, onMounted, onUnmounted, type Ref } from 'vue'

export function useFocusTrap(containerRef: Ref<HTMLElement | null>) {
  const previousActiveElement = ref<HTMLElement | null>(null)
  
  const focusableSelectors = [
    'button:not([disabled])',
    'input:not([disabled])',
    'select:not([disabled])',
    'textarea:not([disabled])',
    'a[href]',
    '[tabindex]:not([tabindex="-1"])'
  ].join(', ')
  
  function getFocusableElements(): HTMLElement[] {
    if (!containerRef.value) return []
    return Array.from(
      containerRef.value.querySelectorAll<HTMLElement>(focusableSelectors)
    )
  }
  
  function handleKeyDown(event: KeyboardEvent): void {
    if (event.key !== 'Tab') return
    
    const focusable = getFocusableElements()
    if (focusable.length === 0) return
    
    const firstElement = focusable[0]
    const lastElement = focusable[focusable.length - 1]
    
    if (event.shiftKey) {
      if (document.activeElement === firstElement) {
        event.preventDefault()
        lastElement.focus()
      }
    } else {
      if (document.activeElement === lastElement) {
        event.preventDefault()
        firstElement.focus()
      }
    }
  }
  
  function activate(): void {
    previousActiveElement.value = document.activeElement as HTMLElement
    
    const focusable = getFocusableElements()
    if (focusable.length > 0) {
      focusable[0].focus()
    }
    
    document.addEventListener('keydown', handleKeyDown)
  }
  
  function deactivate(): void {
    document.removeEventListener('keydown', handleKeyDown)
    
    if (previousActiveElement.value) {
      previousActiveElement.value.focus()
    }
  }
  
  return {
    activate,
    deactivate
  }
}
```

---

### Step 3: Create Modal Sub-Components

**File: `src/components/modal/ModalHeader.vue`**

```vue
<script setup lang="ts">
interface Props {
  showCloseButton?: boolean
}

withDefaults(defineProps<Props>(), {
  showCloseButton: true
})

const emit = defineEmits<{
  close: []
}>()
</script>

<template>
  <div class="flex items-start justify-between p-4 border-b border-base-300">
    <div class="flex-1">
      <slot></slot>
    </div>
    <button
      v-if="showCloseButton"
      type="button"
      class="btn btn-ghost btn-sm btn-circle ml-4"
      aria-label="Close modal"
      @click="emit('close')"
    >
      <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
      </svg>
    </button>
  </div>
</template>
```

**File: `src/components/modal/ModalBody.vue`**

```vue
<script setup lang="ts">
interface Props {
  padding?: boolean
}

withDefaults(defineProps<Props>(), {
  padding: true
})
</script>

<template>
  <div 
    class="flex-1 overflow-y-auto"
    :class="{ 'p-4': padding }"
  >
    <slot></slot>
  </div>
</template>
```

**File: `src/components/modal/ModalFooter.vue`**

```vue
<script setup lang="ts">
interface Props {
  align?: 'left' | 'center' | 'right' | 'between'
}

withDefaults(defineProps<Props>(), {
  align: 'right'
})

const alignClasses = {
  left: 'justify-start',
  center: 'justify-center',
  right: 'justify-end',
  between: 'justify-between'
}
</script>

<template>
  <div 
    class="flex items-center gap-2 p-4 border-t border-base-300 bg-base-200/50"
    :class="alignClasses[align]"
  >
    <slot></slot>
  </div>
</template>
```

---

### Step 4: Create Main Modal Component

**File: `src/components/modal/Modal.vue`**

```vue
<script setup lang="ts">
import { ref, watch, computed, onMounted, onUnmounted, useSlots, nextTick } from 'vue'
import { useFocusTrap } from '@/composables/useFocusTrap'
import ModalHeader from './ModalHeader.vue'
import ModalBody from './ModalBody.vue'
import ModalFooter from './ModalFooter.vue'

interface Props {
  title?: string
  size?: 'sm' | 'md' | 'lg' | 'xl' | 'full'
  closable?: boolean
  closeOnOverlay?: boolean
  closeOnEscape?: boolean
  persistent?: boolean
  showFooter?: boolean
  confirmText?: string
  cancelText?: string
  confirmVariant?: 'primary' | 'secondary' | 'accent' | 'error' | 'success' | 'warning'
  loading?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  title: '',
  size: 'md',
  closable: true,
  closeOnOverlay: true,
  closeOnEscape: true,
  persistent: false,
  showFooter: true,
  confirmText: 'Confirm',
  cancelText: 'Cancel',
  confirmVariant: 'primary',
  loading: false
})

const emit = defineEmits<{
  confirm: []
  cancel: []
  close: []
  'after-enter': []
  'after-leave': []
}>()

const isOpen = defineModel<boolean>({ default: false })

const slots = useSlots()

const modalRef = ref<HTMLElement | null>(null)
const { activate: activateFocusTrap, deactivate: deactivateFocusTrap } = useFocusTrap(modalRef)

const hasHeader = computed(() => props.title || slots.header)
const hasFooter = computed(() => props.showFooter || slots.footer)

const sizeClasses: Record<string, string> = {
  sm: 'max-w-sm',
  md: 'max-w-md',
  lg: 'max-w-2xl',
  xl: 'max-w-4xl',
  full: 'max-w-full mx-4'
}

function close(): void {
  if (props.persistent) {
    shakeModal()
    return
  }
  isOpen.value = false
  emit('close')
}

function handleConfirm(): void {
  emit('confirm')
  if (!props.persistent) {
    close()
  }
}

function handleCancel(): void {
  emit('cancel')
  close()
}

function handleOverlayClick(): void {
  if (props.closeOnOverlay) {
    close()
  } else if (props.persistent) {
    shakeModal()
  }
}

function handleKeyDown(event: KeyboardEvent): void {
  if (event.key === 'Escape' && props.closeOnEscape) {
    close()
  }
}

const isShaking = ref(false)

function shakeModal(): void {
  isShaking.value = true
  setTimeout(() => {
    isShaking.value = false
  }, 300)
}

function handleAfterEnter(): void {
  activateFocusTrap()
  emit('after-enter')
}

function handleAfterLeave(): void {
  deactivateFocusTrap()
  emit('after-leave')
}

watch(isOpen, (newValue) => {
  if (newValue) {
    document.body.style.overflow = 'hidden'
    document.addEventListener('keydown', handleKeyDown)
  } else {
    document.body.style.overflow = ''
    document.removeEventListener('keydown', handleKeyDown)
  }
})

onUnmounted(() => {
  document.body.style.overflow = ''
  document.removeEventListener('keydown', handleKeyDown)
})
</script>

<template>
  <Teleport to="body">
    <Transition
      name="modal-overlay"
      @after-enter="handleAfterEnter"
      @after-leave="handleAfterLeave"
    >
      <div
        v-if="isOpen"
        class="fixed inset-0 z-50 flex items-center justify-center p-4"
      >
        <!-- Overlay -->
        <div
          class="absolute inset-0 bg-black/60 backdrop-blur-sm"
          @click="handleOverlayClick"
        />
        
        <!-- Modal Container -->
        <Transition name="modal-content">
          <div
            v-if="isOpen"
            ref="modalRef"
            class="relative bg-base-100 rounded-xl shadow-2xl w-full flex flex-col max-h-[90vh]"
            :class="[
              sizeClasses[size],
              { 'animate-shake': isShaking }
            ]"
            role="dialog"
            aria-modal="true"
            :aria-labelledby="title ? 'modal-title' : undefined"
          >
            <!-- Header -->
            <slot 
              name="header" 
              :close="close"
            >
              <ModalHeader
                v-if="hasHeader"
                :show-close-button="closable"
                @close="close"
              >
                <h2 id="modal-title" class="text-lg font-semibold">
                  {{ title }}
                </h2>
              </ModalHeader>
            </slot>
            
            <!-- Body -->
            <ModalBody>
              <slot :close="close" :confirm="handleConfirm"></slot>
            </ModalBody>
            
            <!-- Footer -->
            <slot 
              name="footer" 
              :close="close" 
              :confirm="handleConfirm"
              :cancel="handleCancel"
              :loading="loading"
            >
              <ModalFooter v-if="hasFooter">
                <button
                  type="button"
                  class="btn btn-ghost"
                  :disabled="loading"
                  @click="handleCancel"
                >
                  {{ cancelText }}
                </button>
                <button
                  type="button"
                  class="btn"
                  :class="`btn-${confirmVariant}`"
                  :disabled="loading"
                  @click="handleConfirm"
                >
                  <span v-if="loading" class="loading loading-spinner loading-sm"></span>
                  {{ confirmText }}
                </button>
              </ModalFooter>
            </slot>
          </div>
        </Transition>
      </div>
    </Transition>
  </Teleport>
</template>

<style scoped>
.modal-overlay-enter-active,
.modal-overlay-leave-active {
  transition: opacity 0.2s ease;
}

.modal-overlay-enter-from,
.modal-overlay-leave-to {
  opacity: 0;
}

.modal-content-enter-active {
  transition: all 0.2s ease-out;
}

.modal-content-leave-active {
  transition: all 0.15s ease-in;
}

.modal-content-enter-from {
  opacity: 0;
  transform: scale(0.95) translateY(-10px);
}

.modal-content-leave-to {
  opacity: 0;
  transform: scale(0.95);
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-5px); }
  75% { transform: translateX(5px); }
}

.animate-shake {
  animation: shake 0.3s ease-in-out;
}
</style>
```

---

### Step 5: Create Demo Application

**File: `src/App.vue`**

```vue
<script setup lang="ts">
import { ref, reactive } from 'vue'
import Modal from '@/components/modal/Modal.vue'

const isBasicModalOpen = ref(false)
const isConfirmModalOpen = ref(false)
const isFormModalOpen = ref(false)
const isCustomModalOpen = ref(false)
const isPersistentModalOpen = ref(false)
const isLargeModalOpen = ref(false)

const isDeleting = ref(false)
const itemToDelete = ref('Important Document.pdf')

async function handleDeleteConfirm(): Promise<void> {
  isDeleting.value = true
  await new Promise(resolve => setTimeout(resolve, 1500))
  isDeleting.value = false
  isConfirmModalOpen.value = false
  alert(`Deleted: ${itemToDelete.value}`)
}

const formData = reactive({
  name: '',
  email: '',
  message: ''
})

function handleFormSubmit(): void {
  console.log('Form submitted:', formData)
  alert('Message sent!')
  isFormModalOpen.value = false
  formData.name = ''
  formData.email = ''
  formData.message = ''
}

const notifications = ref([
  { id: 1, title: 'New message from Alice', time: '2 min ago', read: false },
  { id: 2, title: 'Your order has shipped', time: '1 hour ago', read: false },
  { id: 3, title: 'Password changed successfully', time: '3 hours ago', read: true },
  { id: 4, title: 'Welcome to our platform!', time: '1 day ago', read: true }
])

function markAllAsRead(): void {
  notifications.value.forEach(n => n.read = true)
}
</script>

<template>
  <div class="min-h-screen bg-gradient-to-br from-base-200 to-base-300 py-12">
    <div class="max-w-4xl mx-auto px-4">
      <div class="text-center mb-12">
        <h1 class="text-4xl font-bold mb-4">Rich Modal Dialog</h1>
        <p class="text-base-content/70 text-lg">
          A fully-featured modal component with slots, animations, and accessibility.
        </p>
      </div>
      
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
        <!-- Basic Modal -->
        <div class="card bg-base-100 shadow-lg">
          <div class="card-body">
            <h2 class="card-title">Basic Modal</h2>
            <p class="text-sm text-base-content/70">Simple modal with title and content.</p>
            <div class="card-actions justify-end mt-4">
              <button class="btn btn-primary" @click="isBasicModalOpen = true">
                Open
              </button>
            </div>
          </div>
        </div>
        
        <!-- Confirmation Modal -->
        <div class="card bg-base-100 shadow-lg">
          <div class="card-body">
            <h2 class="card-title">Confirmation</h2>
            <p class="text-sm text-base-content/70">Delete confirmation with loading state.</p>
            <div class="card-actions justify-end mt-4">
              <button class="btn btn-error" @click="isConfirmModalOpen = true">
                Delete Item
              </button>
            </div>
          </div>
        </div>
        
        <!-- Form Modal -->
        <div class="card bg-base-100 shadow-lg">
          <div class="card-body">
            <h2 class="card-title">Form Modal</h2>
            <p class="text-sm text-base-content/70">Contact form inside a modal.</p>
            <div class="card-actions justify-end mt-4">
              <button class="btn btn-secondary" @click="isFormModalOpen = true">
                Contact Us
              </button>
            </div>
          </div>
        </div>
        
        <!-- Custom Slots Modal -->
        <div class="card bg-base-100 shadow-lg">
          <div class="card-body">
            <h2 class="card-title">Custom Slots</h2>
            <p class="text-sm text-base-content/70">Fully customized header and footer.</p>
            <div class="card-actions justify-end mt-4">
              <button class="btn btn-accent" @click="isCustomModalOpen = true">
                Notifications
              </button>
            </div>
          </div>
        </div>
        
        <!-- Persistent Modal -->
        <div class="card bg-base-100 shadow-lg">
          <div class="card-body">
            <h2 class="card-title">Persistent</h2>
            <p class="text-sm text-base-content/70">Cannot be closed by clicking outside.</p>
            <div class="card-actions justify-end mt-4">
              <button class="btn btn-warning" @click="isPersistentModalOpen = true">
                Terms & Conditions
              </button>
            </div>
          </div>
        </div>
        
        <!-- Large Modal -->
        <div class="card bg-base-100 shadow-lg">
          <div class="card-body">
            <h2 class="card-title">Large Content</h2>
            <p class="text-sm text-base-content/70">Scrollable content in XL modal.</p>
            <div class="card-actions justify-end mt-4">
              <button class="btn btn-info" @click="isLargeModalOpen = true">
                View Article
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Basic Modal -->
    <Modal
      v-model="isBasicModalOpen"
      title="Welcome!"
      confirm-text="Got it"
      :show-footer="true"
      @confirm="isBasicModalOpen = false"
    >
      <div class="space-y-4">
        <p>
          This is a basic modal with a title, content area, and default footer buttons.
        </p>
        <p class="text-base-content/70">
          You can close it by clicking the X button, pressing Escape, clicking the overlay,
          or using the footer buttons.
        </p>
      </div>
    </Modal>
    
    <!-- Confirmation Modal -->
    <Modal
      v-model="isConfirmModalOpen"
      title="Confirm Delete"
      size="sm"
      confirm-text="Delete"
      confirm-variant="error"
      :loading="isDeleting"
      :persistent="isDeleting"
      @confirm="handleDeleteConfirm"
    >
      <div class="text-center space-y-4">
        <div class="text-6xl">🗑️</div>
        <p>
          Are you sure you want to delete
          <strong class="text-error">{{ itemToDelete }}</strong>?
        </p>
        <p class="text-sm text-base-content/70">
          This action cannot be undone.
        </p>
      </div>
    </Modal>
    
    <!-- Form Modal -->
    <Modal
      v-model="isFormModalOpen"
      title="Contact Us"
      size="md"
      confirm-text="Send Message"
      @confirm="handleFormSubmit"
    >
      <form class="space-y-4" @submit.prevent="handleFormSubmit">
        <div class="form-control">
          <label class="label">
            <span class="label-text">Name</span>
          </label>
          <input
            v-model="formData.name"
            type="text"
            class="input input-bordered"
            placeholder="Your name"
            required
          />
        </div>
        
        <div class="form-control">
          <label class="label">
            <span class="label-text">Email</span>
          </label>
          <input
            v-model="formData.email"
            type="email"
            class="input input-bordered"
            placeholder="your@email.com"
            required
          />
        </div>
        
        <div class="form-control">
          <label class="label">
            <span class="label-text">Message</span>
          </label>
          <textarea
            v-model="formData.message"
            class="textarea textarea-bordered"
            rows="4"
            placeholder="How can we help?"
            required
          ></textarea>
        </div>
      </form>
    </Modal>
    
    <!-- Custom Slots Modal -->
    <Modal
      v-model="isCustomModalOpen"
      size="md"
      :show-footer="false"
    >
      <template #header="{ close }">
        <div class="flex items-center justify-between p-4 border-b bg-primary text-primary-content rounded-t-xl">
          <div class="flex items-center gap-3">
            <span class="text-2xl">🔔</span>
            <div>
              <h2 class="font-bold">Notifications</h2>
              <p class="text-sm opacity-80">
                {{ notifications.filter(n => !n.read).length }} unread
              </p>
            </div>
          </div>
          <button
            class="btn btn-ghost btn-sm btn-circle"
            @click="close"
          >
            ✕
          </button>
        </div>
      </template>
      
      <template #default>
        <div class="divide-y">
          <div
            v-for="notification in notifications"
            :key="notification.id"
            class="flex items-start gap-3 p-4 hover:bg-base-200 transition-colors cursor-pointer"
            :class="{ 'opacity-60': notification.read }"
          >
            <div 
              class="w-2 h-2 rounded-full mt-2"
              :class="notification.read ? 'bg-base-300' : 'bg-primary'"
            ></div>
            <div class="flex-1">
              <p class="font-medium">{{ notification.title }}</p>
              <p class="text-sm text-base-content/50">{{ notification.time }}</p>
            </div>
          </div>
        </div>
      </template>
      
      <template #footer="{ close }">
        <div class="flex justify-between p-4 border-t bg-base-200/50 rounded-b-xl">
          <button class="btn btn-ghost btn-sm" @click="markAllAsRead">
            Mark all as read
          </button>
          <button class="btn btn-primary btn-sm" @click="close">
            Done
          </button>
        </div>
      </template>
    </Modal>
    
    <!-- Persistent Modal -->
    <Modal
      v-model="isPersistentModalOpen"
      title="Terms & Conditions"
      size="lg"
      persistent
      :close-on-overlay="false"
      :close-on-escape="false"
      :closable="false"
      confirm-text="I Accept"
      cancel-text="Decline"
      @confirm="isPersistentModalOpen = false"
      @cancel="alert('You must accept to continue')"
    >
      <div class="prose prose-sm max-w-none">
        <p>Please read and accept our terms and conditions to continue.</p>
        
        <h3>1. Introduction</h3>
        <p>
          These terms and conditions govern your use of our website and services.
          By using our website, you accept these terms in full.
        </p>
        
        <h3>2. Intellectual Property</h3>
        <p>
          All content on this website is protected by copyright and other
          intellectual property laws.
        </p>
        
        <h3>3. Limitations of Liability</h3>
        <p>
          We will not be liable for any indirect, special, or consequential
          damages arising out of or in connection with your use of our services.
        </p>
        
        <div class="form-control mt-6">
          <label class="cursor-pointer flex items-center gap-3">
            <input type="checkbox" class="checkbox checkbox-primary" />
            <span>I have read and agree to the terms and conditions</span>
          </label>
        </div>
      </div>
    </Modal>
    
    <!-- Large Content Modal -->
    <Modal
      v-model="isLargeModalOpen"
      title="The History of Web Development"
      size="xl"
      confirm-text="Close"
      :show-footer="true"
      @confirm="isLargeModalOpen = false"
    >
      <article class="prose prose-sm max-w-none">
        <p class="lead">
          Web development has come a long way since the early days of the internet.
          Let's explore its fascinating evolution.
        </p>
        
        <h2>The Early Days (1990s)</h2>
        <p>
          The World Wide Web was invented by Tim Berners-Lee in 1989 while working
          at CERN. The first website went live in 1991, and it was a simple
          text-based page explaining what the World Wide Web was.
        </p>
        <p>
          Early websites were built using only HTML. There was no CSS for styling
          or JavaScript for interactivity. Pages were static and looked quite
          plain by today's standards.
        </p>
        
        <h2>The Rise of Dynamic Content (Late 1990s - Early 2000s)</h2>
        <p>
          JavaScript was introduced in 1995, allowing developers to add
          interactivity to web pages. CSS followed in 1996, separating content
          from presentation.
        </p>
        <p>
          Server-side technologies like PHP, ASP, and Java Servlets enabled
          dynamic content generation. Websites could now interact with databases
          and generate personalized content for users.
        </p>
        
        <h2>Web 2.0 Era (2004-2010)</h2>
        <p>
          The term "Web 2.0" was coined to describe a new generation of websites
          that emphasized user-generated content, usability, and interoperability.
          AJAX (Asynchronous JavaScript and XML) enabled rich, interactive
          experiences without page reloads.
        </p>
        <p>
          Social media platforms, blogs, and wikis became popular. jQuery
          simplified JavaScript development and became nearly ubiquitous.
        </p>
        
        <h2>The Modern Era (2010-Present)</h2>
        <p>
          Modern web development has been transformed by:
        </p>
        <ul>
          <li><strong>Single Page Applications (SPAs)</strong> - Frameworks like Angular, React, and Vue.js</li>
          <li><strong>Responsive Design</strong> - Mobile-first approaches and CSS frameworks</li>
          <li><strong>Component-Based Architecture</strong> - Reusable UI components</li>
          <li><strong>TypeScript</strong> - Type-safe JavaScript development</li>
          <li><strong>Build Tools</strong> - Webpack, Vite, and modern bundlers</li>
        </ul>
        
        <h2>The Future</h2>
        <p>
          Web development continues to evolve with technologies like WebAssembly,
          Web Components, and AI-assisted development. The future promises even
          more powerful and accessible web experiences.
        </p>
      </article>
    </Modal>
  </div>
</template>

<style>
.prose h2 {
  @apply text-lg font-semibold mt-6 mb-2;
}

.prose h3 {
  @apply text-base font-semibold mt-4 mb-2;
}

.prose p {
  @apply mb-3;
}

.prose ul {
  @apply list-disc pl-5 space-y-1;
}

.prose .lead {
  @apply text-lg text-base-content/80;
}
</style>
```

---

## How to Run

```bash
npm run dev
```

---

## Stretch Goals

1. **Nested Modals:** Support opening modals from within modals with proper z-index management
2. **Drawer Variant:** Create a slide-in drawer variant of the modal
3. **Image Lightbox:** Build a specialized modal for viewing images with zoom and navigation
4. **Form Validation Integration:** Add automatic form validation before confirm
5. **Undo Support:** Add an undo toast notification after destructive actions

---

## Self-Assessment Checklist

- [ ] Modal uses named slots for header, body, and footer customization
- [ ] Scoped slots pass `close`, `confirm`, and `cancel` actions
- [ ] `defineModel()` controls open/close state
- [ ] Keyboard shortcuts (Escape) work correctly
- [ ] Focus is trapped inside the modal when open
- [ ] Previous focus is restored when modal closes
- [ ] Animations are smooth for open/close
- [ ] `Teleport` renders modal at document body
- [ ] Persistent mode prevents accidental closing
- [ ] Loading state disables interactions appropriately
- [ ] Accessibility attributes are properly set
- [ ] Body scroll is locked when modal is open
