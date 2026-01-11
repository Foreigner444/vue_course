# Guided Project 2: useLocalStorage Composable
---
## Project Overview
| Field | Value |
|:--- |:--- |
| **Module** | Module 6: Reusability & Abstraction |
| **Project** | useLocalStorage Composable |
| **Real-World Scenario** | A notes app persists user preferences and drafts to localStorage with reactive sync |
| **Difficulty** | Intermediate |
| **Estimated Time** | 40-50 minutes |
### What You'll Build
A robust localStorage composable that provides reactive two-way binding between Vue refs and browser localStorage. The composable will handle JSON serialization, SSR safety, type inference, and automatic syncing across browser tabs. You'll use it to build a simple notes app that persists user data.
### Patterns You'll Practice
1. Generic TypeScript for flexible typing
2. `ref()` with type parameters
3. `watch()` with deep option for object tracking
4. SSR-safe browser API access
5. Event listeners for cross-tab sync
6. Cleanup with `onUnmounted`
7. Default value handling
8. Error handling for JSON parsing
---
## Component Structure
```
src/
├── composables/
│   └── useLocalStorage.ts    # The localStorage composable
├── components/
│   ├── NoteEditor.vue        # Note editing component
│   ├── SettingsPanel.vue     # User settings component
│   └── NotesList.vue         # Display saved notes
├── types/
│   └── notes.ts              # TypeScript interfaces
├── App.vue                   # Main app layout
└── main.ts
```
---
## Step-by-Step Guide
### Step 1: Project Setup
```bash
npm create vite@latest localstorage-demo -- --template vue-ts
cd localstorage-demo
npm install
npm install -D tailwindcss postcss autoprefixer daisyui@latest
npx tailwindcss init -p
```
Configure `tailwind.config.js`:
```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: ["./index.html", "./src/**/*.{vue,js,ts,jsx,tsx}"],
  theme: { extend: {} },
  plugins: [require("daisyui")],
  daisyui: {
    themes: ["light", "dark", "cupcake", "forest"],
  },
}
```
Update `src/style.css`:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```
---
### Step 2: Define TypeScript Types
```typescript
// src/types/notes.ts
export interface Note {
  id: string
  title: string
  content: string
  createdAt: number
  updatedAt: number
}
export interface UserSettings {
  theme: 'light' | 'dark' | 'cupcake' | 'forest'
  fontSize: 'sm' | 'base' | 'lg' | 'xl'
  autoSave: boolean
}
export interface AppState {
  notes: Note[]
  settings: UserSettings
  draftNote: Partial<Note> | null
}
```
---
### Step 3: Build the useLocalStorage Composable
```typescript
// src/composables/useLocalStorage.ts
import { ref, watch, onMounted, onUnmounted, type Ref } from 'vue'
// ─────────────────────────────────────────────────────────────
// OPTIONS INTERFACE: Configuration for the composable
// ─────────────────────────────────────────────────────────────
interface UseLocalStorageOptions<T> {
  deep?: boolean
  listenToStorageChanges?: boolean
  writeDefaults?: boolean
  onError?: (error: Error) => void
}
// ─────────────────────────────────────────────────────────────
// MAIN COMPOSABLE: useLocalStorage
// 
// Creates a ref that automatically syncs with localStorage.
// Changes to the ref are persisted, and the ref updates when
// localStorage changes (even from other tabs).
// ─────────────────────────────────────────────────────────────
export function useLocalStorage<T>(
  key: string,
  defaultValue: T,
  options: UseLocalStorageOptions<T> = {}
): Ref<T> {
  
  // ─────────────────────────────────────────────────────────────
  // OPTIONS: Destructure with defaults
  // ─────────────────────────────────────────────────────────────
  const {
    deep = true,
    listenToStorageChanges = true,
    writeDefaults = true,
    onError = console.error
  } = options
  // ─────────────────────────────────────────────────────────────
  // HELPER: Safely serialize value to JSON string
  // ─────────────────────────────────────────────────────────────
  function serialize(value: T): string {
    try {
      return JSON.stringify(value)
    } catch (error) {
      onError(new Error(`Failed to serialize value for key "${key}": ${error}`))
      return JSON.stringify(defaultValue)
    }
  }
  // ─────────────────────────────────────────────────────────────
  // HELPER: Safely deserialize JSON string to value
  // ─────────────────────────────────────────────────────────────
  function deserialize(value: string | null): T {
    if (value === null) {
      return defaultValue
    }
    
    try {
      return JSON.parse(value) as T
    } catch (error) {
      onError(new Error(`Failed to parse stored value for key "${key}": ${error}`))
      return defaultValue
    }
  }
  // ─────────────────────────────────────────────────────────────
  // HELPER: Read current value from localStorage
  // SSR-safe: checks if window exists
  // ─────────────────────────────────────────────────────────────
  function read(): T {
    if (typeof window === 'undefined') {
      return defaultValue
    }
    
    try {
      const stored = localStorage.getItem(key)
      
      if (stored === null && writeDefaults) {
        localStorage.setItem(key, serialize(defaultValue))
        return defaultValue
      }
      
      return deserialize(stored)
    } catch (error) {
      onError(new Error(`Failed to read from localStorage: ${error}`))
      return defaultValue
    }
  }
  // ─────────────────────────────────────────────────────────────
  // HELPER: Write value to localStorage
  // ─────────────────────────────────────────────────────────────
  function write(value: T): void {
    if (typeof window === 'undefined') {
      return
    }
    
    try {
      localStorage.setItem(key, serialize(value))
    } catch (error) {
      onError(new Error(`Failed to write to localStorage: ${error}`))
    }
  }
  // ─────────────────────────────────────────────────────────────
  // STATE: Create the reactive ref with initial value from storage
  // ─────────────────────────────────────────────────────────────
  const data = ref<T>(read()) as Ref<T>
  // ─────────────────────────────────────────────────────────────
  // WATCHER: Sync changes to localStorage
  // Uses deep watching for objects/arrays
  // ─────────────────────────────────────────────────────────────
  watch(
    data,
    (newValue) => {
      write(newValue)
    },
    { deep }
  )
  // ─────────────────────────────────────────────────────────────
  // STORAGE EVENT: Listen for changes from other tabs/windows
  // This enables cross-tab synchronization
  // ─────────────────────────────────────────────────────────────
  function handleStorageChange(event: StorageEvent) {
    if (event.key !== key) {
      return
    }
    
    if (event.newValue === null) {
      data.value = defaultValue
    } else {
      data.value = deserialize(event.newValue)
    }
  }
  // ─────────────────────────────────────────────────────────────
  // LIFECYCLE: Set up and tear down storage event listener
  // ─────────────────────────────────────────────────────────────
  if (listenToStorageChanges && typeof window !== 'undefined') {
    onMounted(() => {
      window.addEventListener('storage', handleStorageChange)
    })
    
    onUnmounted(() => {
      window.removeEventListener('storage', handleStorageChange)
    })
  }
  // ─────────────────────────────────────────────────────────────
  // RETURN: The reactive ref that syncs with localStorage
  // ─────────────────────────────────────────────────────────────
  return data
}
// ─────────────────────────────────────────────────────────────
// UTILITY: Remove item from localStorage
// ─────────────────────────────────────────────────────────────
export function removeStorageItem(key: string): void {
  if (typeof window !== 'undefined') {
    localStorage.removeItem(key)
  }
}
// ─────────────────────────────────────────────────────────────
// UTILITY: Clear all app-related localStorage items
// ─────────────────────────────────────────────────────────────
export function clearStorage(prefix?: string): void {
  if (typeof window === 'undefined') {
    return
  }
  
  if (prefix) {
    const keysToRemove: string[] = []
    for (let i = 0; i < localStorage.length; i++) {
      const key = localStorage.key(i)
      if (key?.startsWith(prefix)) {
        keysToRemove.push(key)
      }
    }
    keysToRemove.forEach(key => localStorage.removeItem(key))
  } else {
    localStorage.clear()
  }
}
```
---
### Step 4: Create the Settings Panel
```vue
<!-- src/components/SettingsPanel.vue -->
<script setup lang="ts">
import { useLocalStorage } from '@/composables/useLocalStorage'
import type { UserSettings } from '@/types/notes'
// ─────────────────────────────────────────────────────────────
// SETTINGS STATE: Persisted to localStorage automatically
// ─────────────────────────────────────────────────────────────
const settings = useLocalStorage<UserSettings>('app-settings', {
  theme: 'light',
  fontSize: 'base',
  autoSave: true
})
// ─────────────────────────────────────────────────────────────
// THEME OPTIONS: Available themes from DaisyUI
// ─────────────────────────────────────────────────────────────
const themes = [
  { value: 'light', label: '☀️ Light' },
  { value: 'dark', label: '🌙 Dark' },
  { value: 'cupcake', label: '🧁 Cupcake' },
  { value: 'forest', label: '🌲 Forest' }
] as const
const fontSizes = [
  { value: 'sm', label: 'Small' },
  { value: 'base', label: 'Medium' },
  { value: 'lg', label: 'Large' },
  { value: 'xl', label: 'Extra Large' }
] as const
// ─────────────────────────────────────────────────────────────
// EFFECT: Apply theme to document when it changes
// ─────────────────────────────────────────────────────────────
import { watchEffect } from 'vue'
watchEffect(() => {
  document.documentElement.setAttribute('data-theme', settings.value.theme)
})
</script>
<template>
  <div class="card bg-base-200">
    <div class="card-body">
      <h2 class="card-title text-lg">⚙️ Settings</h2>
      
      <!-- Theme Selector -->
      <div class="form-control">
        <label class="label">
          <span class="label-text">Theme</span>
        </label>
        <select 
          v-model="settings.theme" 
          class="select select-bordered select-sm"
        >
          <option 
            v-for="theme in themes" 
            :key="theme.value" 
            :value="theme.value"
          >
            {{ theme.label }}
          </option>
        </select>
      </div>
      
      <!-- Font Size Selector -->
      <div class="form-control">
        <label class="label">
          <span class="label-text">Font Size</span>
        </label>
        <select 
          v-model="settings.fontSize" 
          class="select select-bordered select-sm"
        >
          <option 
            v-for="size in fontSizes" 
            :key="size.value" 
            :value="size.value"
          >
            {{ size.label }}
          </option>
        </select>
      </div>
      
      <!-- Auto-Save Toggle -->
      <div class="form-control">
        <label class="label cursor-pointer">
          <span class="label-text">Auto-save drafts</span>
          <input 
            type="checkbox" 
            v-model="settings.autoSave" 
            class="toggle toggle-primary"
          />
        </label>
      </div>
      
      <!-- Current Settings Display -->
      <div class="mt-2 text-xs text-base-content/60">
        <p>Settings are automatically saved to localStorage</p>
      </div>
    </div>
  </div>
</template>
```
---
### Step 5: Create the Note Editor
```vue
<!-- src/components/NoteEditor.vue -->
<script setup lang="ts">
import { computed, watch } from 'vue'
import { useLocalStorage } from '@/composables/useLocalStorage'
import type { Note, UserSettings } from '@/types/notes'
// ─────────────────────────────────────────────────────────────
// PROPS & EMITS
// ─────────────────────────────────────────────────────────────
const props = defineProps<{
  editingNote?: Note | null
}>()
const emit = defineEmits<{
  save: [note: Note]
  cancel: []
}>()
// ─────────────────────────────────────────────────────────────
// DRAFT STATE: Auto-saved to localStorage
// ─────────────────────────────────────────────────────────────
const draft = useLocalStorage<{ title: string; content: string }>('note-draft', {
  title: '',
  content: ''
})
// ─────────────────────────────────────────────────────────────
// SETTINGS: Read settings for auto-save behavior
// ─────────────────────────────────────────────────────────────
const settings = useLocalStorage<UserSettings>('app-settings', {
  theme: 'light',
  fontSize: 'base',
  autoSave: true
})
// ─────────────────────────────────────────────────────────────
// FONT SIZE CLASS: Computed from settings
// ─────────────────────────────────────────────────────────────
const fontSizeClass = computed(() => {
  const sizes = {
    sm: 'text-sm',
    base: 'text-base',
    lg: 'text-lg',
    xl: 'text-xl'
  }
  return sizes[settings.value.fontSize]
})
// ─────────────────────────────────────────────────────────────
// EFFECT: Load editing note into draft
// ─────────────────────────────────────────────────────────────
watch(
  () => props.editingNote,
  (note) => {
    if (note) {
      draft.value = {
        title: note.title,
        content: note.content
      }
    }
  },
  { immediate: true }
)
// ─────────────────────────────────────────────────────────────
// COMPUTED: Check if draft has content
// ─────────────────────────────────────────────────────────────
const hasDraft = computed(() => {
  return draft.value.title.trim() !== '' || draft.value.content.trim() !== ''
})
// ─────────────────────────────────────────────────────────────
// ACTIONS
// ─────────────────────────────────────────────────────────────
function saveNote() {
  if (!draft.value.title.trim()) {
    alert('Please enter a title')
    return
  }
  
  const now = Date.now()
  const note: Note = {
    id: props.editingNote?.id || crypto.randomUUID(),
    title: draft.value.title.trim(),
    content: draft.value.content.trim(),
    createdAt: props.editingNote?.createdAt || now,
    updatedAt: now
  }
  
  emit('save', note)
  clearDraft()
}
function clearDraft() {
  draft.value = { title: '', content: '' }
}
function cancel() {
  clearDraft()
  emit('cancel')
}
</script>
<template>
  <div class="card bg-base-200">
    <div class="card-body">
      <h2 class="card-title text-lg">
        {{ editingNote ? '✏️ Edit Note' : '📝 New Note' }}
      </h2>
      
      <!-- Title Input -->
      <div class="form-control">
        <input
          v-model="draft.title"
          type="text"
          placeholder="Note title..."
          class="input input-bordered"
          :class="fontSizeClass"
        />
      </div>
      
      <!-- Content Textarea -->
      <div class="form-control">
        <textarea
          v-model="draft.content"
          placeholder="Write your note..."
          class="textarea textarea-bordered h-40"
          :class="fontSizeClass"
        ></textarea>
      </div>
      
      <!-- Auto-save indicator -->
      <div v-if="settings.autoSave && hasDraft" class="text-xs text-success">
        ✓ Draft auto-saved
      </div>
      
      <!-- Actions -->
      <div class="card-actions justify-end mt-2">
        <button 
          v-if="hasDraft"
          class="btn btn-ghost btn-sm"
          @click="cancel"
        >
          {{ editingNote ? 'Cancel' : 'Clear' }}
        </button>
        <button 
          class="btn btn-primary btn-sm"
          @click="saveNote"
          :disabled="!draft.title.trim()"
        >
          {{ editingNote ? 'Update' : 'Save' }} Note
        </button>
      </div>
    </div>
  </div>
</template>
```
---
### Step 6: Create the Notes List
```vue
<!-- src/components/NotesList.vue -->
<script setup lang="ts">
import { computed } from 'vue'
import type { Note } from '@/types/notes'
// ─────────────────────────────────────────────────────────────
// PROPS & EMITS
// ─────────────────────────────────────────────────────────────
const props = defineProps<{
  notes: Note[]
}>()
const emit = defineEmits<{
  edit: [note: Note]
  delete: [id: string]
}>()
// ─────────────────────────────────────────────────────────────
// COMPUTED: Sort notes by updated date (newest first)
// ─────────────────────────────────────────────────────────────
const sortedNotes = computed(() => {
  return [...props.notes].sort((a, b) => b.updatedAt - a.updatedAt)
})
// ─────────────────────────────────────────────────────────────
// HELPER: Format date for display
// ─────────────────────────────────────────────────────────────
function formatDate(timestamp: number): string {
  return new Date(timestamp).toLocaleDateString('en-US', {
    month: 'short',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  })
}
// ─────────────────────────────────────────────────────────────
// HELPER: Truncate content for preview
// ─────────────────────────────────────────────────────────────
function truncate(text: string, length: number = 100): string {
  if (text.length <= length) return text
  return text.slice(0, length) + '...'
}
// ─────────────────────────────────────────────────────────────
// ACTION: Confirm and delete note
// ─────────────────────────────────────────────────────────────
function confirmDelete(note: Note) {
  if (confirm(`Delete "${note.title}"?`)) {
    emit('delete', note.id)
  }
}
</script>
<template>
  <div class="card bg-base-200">
    <div class="card-body">
      <h2 class="card-title text-lg">📚 My Notes ({{ notes.length }})</h2>
      
      <!-- Empty State -->
      <div v-if="notes.length === 0" class="text-center py-8 text-base-content/60">
        <p class="text-4xl mb-2">📭</p>
        <p>No notes yet. Create your first note!</p>
      </div>
      
      <!-- Notes List -->
      <div v-else class="space-y-2 max-h-96 overflow-y-auto">
        <div
          v-for="note in sortedNotes"
          :key="note.id"
          class="card bg-base-100 shadow-sm hover:shadow-md transition-shadow"
        >
          <div class="card-body p-4">
            <!-- Title & Actions -->
            <div class="flex justify-between items-start gap-2">
              <h3 class="font-semibold">{{ note.title }}</h3>
              <div class="flex gap-1">
                <button 
                  class="btn btn-ghost btn-xs"
                  @click="emit('edit', note)"
                  title="Edit"
                >
                  ✏️
                </button>
                <button 
                  class="btn btn-ghost btn-xs text-error"
                  @click="confirmDelete(note)"
                  title="Delete"
                >
                  🗑️
                </button>
              </div>
            </div>
            
            <!-- Content Preview -->
            <p v-if="note.content" class="text-sm text-base-content/70">
              {{ truncate(note.content) }}
            </p>
            
            <!-- Metadata -->
            <div class="text-xs text-base-content/50 mt-1">
              Updated: {{ formatDate(note.updatedAt) }}
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
```
---
### Step 7: Create App.vue
```vue
<!-- src/App.vue -->
<script setup lang="ts">
import { ref, computed } from 'vue'
import { useLocalStorage, clearStorage } from '@/composables/useLocalStorage'
import type { Note } from '@/types/notes'
import SettingsPanel from '@/components/SettingsPanel.vue'
import NoteEditor from '@/components/NoteEditor.vue'
import NotesList from '@/components/NotesList.vue'
// ─────────────────────────────────────────────────────────────
// NOTES STATE: Persisted to localStorage
// ─────────────────────────────────────────────────────────────
const notes = useLocalStorage<Note[]>('app-notes', [])
// ─────────────────────────────────────────────────────────────
// LOCAL STATE: Currently editing note (not persisted)
// ─────────────────────────────────────────────────────────────
const editingNote = ref<Note | null>(null)
// ─────────────────────────────────────────────────────────────
// COMPUTED: Stats for display
// ─────────────────────────────────────────────────────────────
const stats = computed(() => ({
  total: notes.value.length,
  totalWords: notes.value.reduce((acc, note) => {
    return acc + note.content.split(/\s+/).filter(Boolean).length
  }, 0)
}))
// ─────────────────────────────────────────────────────────────
// ACTIONS
// ─────────────────────────────────────────────────────────────
function saveNote(note: Note) {
  const index = notes.value.findIndex(n => n.id === note.id)
  
  if (index >= 0) {
    notes.value[index] = note
  } else {
    notes.value.push(note)
  }
  
  editingNote.value = null
}
function deleteNote(id: string) {
  notes.value = notes.value.filter(n => n.id !== id)
  if (editingNote.value?.id === id) {
    editingNote.value = null
  }
}
function editNote(note: Note) {
  editingNote.value = note
}
function cancelEdit() {
  editingNote.value = null
}
function clearAllData() {
  if (confirm('Clear all notes and settings? This cannot be undone.')) {
    clearStorage('app-')
    clearStorage('note-')
    notes.value = []
    editingNote.value = null
    window.location.reload()
  }
}
</script>
<template>
  <div class="min-h-screen bg-base-100 transition-colors duration-300">
    <!-- Header -->
    <header class="navbar bg-base-200 shadow-sm">
      <div class="flex-1">
        <span class="text-xl font-bold">📓 Notes App</span>
      </div>
      <div class="flex-none">
        <div class="badge badge-primary badge-outline mr-2">
          {{ stats.total }} notes · {{ stats.totalWords }} words
        </div>
        <button 
          class="btn btn-ghost btn-sm text-error"
          @click="clearAllData"
          title="Clear all data"
        >
          🗑️ Clear All
        </button>
      </div>
    </header>
    
    <!-- Main Content -->
    <main class="container mx-auto p-4">
      <div class="grid grid-cols-1 lg:grid-cols-3 gap-4">
        
        <!-- Left Column: Editor -->
        <div class="lg:col-span-2 space-y-4">
          <NoteEditor
            :editing-note="editingNote"
            @save="saveNote"
            @cancel="cancelEdit"
          />
          
          <NotesList
            :notes="notes"
            @edit="editNote"
            @delete="deleteNote"
          />
        </div>
        
        <!-- Right Column: Settings -->
        <div>
          <SettingsPanel />
          
          <!-- Info Card -->
          <div class="card bg-base-200 mt-4">
            <div class="card-body text-sm">
              <h3 class="font-semibold">💡 Tips</h3>
              <ul class="list-disc list-inside space-y-1 text-base-content/70">
                <li>Your notes are saved automatically</li>
                <li>Settings persist across sessions</li>
                <li>Open in another tab to see sync!</li>
                <li>Drafts are saved as you type</li>
              </ul>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>
```
---
## Testing Cross-Tab Sync
To test the cross-tab synchronization feature:
1. Open the app in two browser tabs
2. Create a note in Tab 1
3. Watch Tab 2 automatically update with the new note
4. Change the theme in Tab 2
5. Watch Tab 1 theme update automatically
This works because the composable listens to the `storage` event, which fires when localStorage changes in another tab.
---
## Stretch Goals
1. **Add expiration:** Implement a `useLocalStorageWithExpiry` variant that auto-clears data after a specified time
2. **Add compression:** For large data, compress JSON before storing using a library like `lz-string`
3. **Add migration:** Handle schema changes by implementing versioning and migration logic
4. **Add encryption:** For sensitive data, encrypt values before storing
---
## Self-Assessment Checklist
- [ ] The `useLocalStorage` composable works with any serializable type
- [ ] Values persist after page refresh
- [ ] Changes sync across browser tabs
- [ ] Default values are used when no stored value exists
- [ ] JSON parsing errors are handled gracefully
- [ ] SSR safety is implemented (checks for `window`)
- [ ] Event listeners are cleaned up on unmount
- [ ] Settings changes are reflected immediately in the UI
- [ ] Draft auto-save works when enabled in settings
- [ ] Notes can be created, edited, and deleted
