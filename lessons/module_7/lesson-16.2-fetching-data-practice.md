# Lesson 16.2: Fetching Data: Async/Await & Lifecycle — Practice & Application (Получение данных: Async/Await и жизненный цикл)

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 16.2 of 25 (Part 2 of 2) |
| **Topic** | Fetching Data: Async/Await & Lifecycle / Получение данных: Async/Await и жизненный цикл |
| **Continues From** | Lesson 16.1 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Build a component that fetches data from a REST API with proper loading and error states
2. **Analyze:** Debug common async/await issues and compare different data fetching patterns

---

## 2. Quick Recap

From Lesson 16.1:
- **`onMounted()`** is the lifecycle hook for initial data fetching — it runs after the component is inserted into the DOM
- **Three reactive states** are essential: `data` (the response), `loading` (request in progress), and `error` (failure message)
- **async/await** with `try/catch/finally` provides clean error handling for API calls
- Vue's reactivity automatically updates the UI when refs change after fetch completes

---

## 3. The Variable Frame & Complexity Scale

### Basic Form: Simple GET Request
```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'

const data = ref(null)

onMounted(async () => {
  const response = await fetch('https://api.example.com/items')
  data.value = await response.json()
})
</script>
```

### With Options: Full State Management
```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface Item {
  id: number
  title: string
}

const items = ref<Item[]>([])
const loading = ref(false)
const error = ref<string | null>(null)

async function fetchItems() {
  loading.value = true
  error.value = null
  try {
    const response = await fetch('https://api.example.com/items')
    if (!response.ok) throw new Error(`Status: ${response.status}`)
    items.value = await response.json()
  } catch (e) {
    error.value = e instanceof Error ? e.message : 'Unknown error'
  } finally {
    loading.value = false
  }
}

onMounted(fetchItems)
</script>
```

### Advanced Form: With AbortController & Cleanup
```vue
<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

interface Item {
  id: number
  title: string
}

const items = ref<Item[]>([])
const loading = ref(false)
const error = ref<string | null>(null)

let abortController: AbortController | null = null

async function fetchItems() {
  abortController = new AbortController()
  loading.value = true
  error.value = null
  
  try {
    const response = await fetch('https://api.example.com/items', {
      signal: abortController.signal
    })
    if (!response.ok) throw new Error(`Status: ${response.status}`)
    items.value = await response.json()
  } catch (e) {
    if (e instanceof Error && e.name !== 'AbortError') {
      error.value = e.message
    }
  } finally {
    loading.value = false
  }
}

onMounted(fetchItems)

onUnmounted(() => {
  abortController?.abort()
})
</script>
```

---

## 4. Micro-Implementation Drills

### Drill 1 (Basic): Fetch a Single Post

Fetch a single blog post from JSONPlaceholder and display its title and body.

```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface Post {
  id: number
  title: string
  body: string
}

// TODO: Create a ref called 'post' with type Post | null, initial value null
// YOUR CODE HERE

onMounted(async () => {
  // TODO: Fetch from 'https://jsonplaceholder.typicode.com/posts/1'
  // TODO: Parse the JSON and assign to post.value
  // YOUR CODE HERE
})
</script>

<template>
  <div class="p-4">
    <!-- TODO: Display post.title in an h2 and post.body in a p tag -->
    <!-- YOUR CODE HERE -->
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface Post {
  id: number
  title: string
  body: string
}

const post = ref<Post | null>(null)

onMounted(async () => {
  const response = await fetch('https://jsonplaceholder.typicode.com/posts/1')
  post.value = await response.json()
})
</script>

<template>
  <div class="p-4">
    <div v-if="post">
      <h2 class="text-xl font-bold">{{ post.title }}</h2>
      <p class="mt-2">{{ post.body }}</p>
    </div>
  </div>
</template>
```

---

### Drill 2 (Variation): Add Loading State

Extend Drill 1 to show a loading spinner while the request is in progress.

```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface Post {
  id: number
  title: string
  body: string
}

const post = ref<Post | null>(null)
// TODO: Create a 'loading' ref, initial value false
// YOUR CODE HERE

async function fetchPost() {
  // TODO: Set loading to true before fetch
  // YOUR CODE HERE
  
  const response = await fetch('https://jsonplaceholder.typicode.com/posts/1')
  post.value = await response.json()
  
  // TODO: Set loading to false after fetch
  // YOUR CODE HERE
}

onMounted(fetchPost)
</script>

<template>
  <div class="p-4">
    <!-- TODO: Show loading spinner when loading is true -->
    <!-- TODO: Show post content when loading is false -->
    <!-- YOUR CODE HERE -->
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface Post {
  id: number
  title: string
  body: string
}

const post = ref<Post | null>(null)
const loading = ref(false)

async function fetchPost() {
  loading.value = true
  const response = await fetch('https://jsonplaceholder.typicode.com/posts/1')
  post.value = await response.json()
  loading.value = false
}

onMounted(fetchPost)
</script>

<template>
  <div class="p-4">
    <div v-if="loading" class="flex items-center gap-2">
      <span class="loading loading-spinner"></span>
      <span>Loading...</span>
    </div>
    <div v-else-if="post">
      <h2 class="text-xl font-bold">{{ post.title }}</h2>
      <p class="mt-2">{{ post.body }}</p>
    </div>
  </div>
</template>
```

---

### Drill 3 (Combination): Add Error Handling

Combine loading state with proper error handling using try/catch/finally.

```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface Post {
  id: number
  title: string
  body: string
}

const post = ref<Post | null>(null)
const loading = ref(false)
// TODO: Create an 'error' ref with type string | null, initial value null
// YOUR CODE HERE

async function fetchPost() {
  loading.value = true
  // TODO: Reset error to null before fetching
  // YOUR CODE HERE
  
  // TODO: Wrap fetch in try/catch/finally
  // - In try: fetch and check response.ok
  // - In catch: set error.value to the error message
  // - In finally: set loading to false
  // YOUR CODE HERE
}

onMounted(fetchPost)
</script>

<template>
  <div class="p-4">
    <div v-if="loading" class="flex items-center gap-2">
      <span class="loading loading-spinner"></span>
      <span>Loading...</span>
    </div>
    <!-- TODO: Add error state display with alert styling -->
    <!-- YOUR CODE HERE -->
    <div v-else-if="post">
      <h2 class="text-xl font-bold">{{ post.title }}</h2>
      <p class="mt-2">{{ post.body }}</p>
    </div>
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface Post {
  id: number
  title: string
  body: string
}

const post = ref<Post | null>(null)
const loading = ref(false)
const error = ref<string | null>(null)

async function fetchPost() {
  loading.value = true
  error.value = null
  
  try {
    const response = await fetch('https://jsonplaceholder.typicode.com/posts/1')
    if (!response.ok) {
      throw new Error(`HTTP error! Status: ${response.status}`)
    }
    post.value = await response.json()
  } catch (e) {
    error.value = e instanceof Error ? e.message : 'An unknown error occurred'
  } finally {
    loading.value = false
  }
}

onMounted(fetchPost)
</script>

<template>
  <div class="p-4">
    <div v-if="loading" class="flex items-center gap-2">
      <span class="loading loading-spinner"></span>
      <span>Loading...</span>
    </div>
    <div v-else-if="error" class="alert alert-error">
      <span>{{ error }}</span>
    </div>
    <div v-else-if="post">
      <h2 class="text-xl font-bold">{{ post.title }}</h2>
      <p class="mt-2">{{ post.body }}</p>
    </div>
  </div>
</template>
```

---

### Drill 4 (Edge Case): Fetch with Dynamic ID from Props

Create a component that fetches a post based on a prop, and re-fetches when the prop changes.

```vue
<script setup lang="ts">
import { ref, watch, onMounted } from 'vue'

interface Post {
  id: number
  title: string
  body: string
}

// TODO: Define props with a 'postId' of type number
// YOUR CODE HERE

const post = ref<Post | null>(null)
const loading = ref(false)
const error = ref<string | null>(null)

async function fetchPost(id: number) {
  loading.value = true
  error.value = null
  
  try {
    const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`)
    if (!response.ok) throw new Error(`Status: ${response.status}`)
    post.value = await response.json()
  } catch (e) {
    error.value = e instanceof Error ? e.message : 'Unknown error'
  } finally {
    loading.value = false
  }
}

// TODO: Use onMounted to fetch with initial postId
// YOUR CODE HERE

// TODO: Use watch to re-fetch when postId prop changes
// YOUR CODE HERE
</script>

<template>
  <div class="p-4">
    <div v-if="loading" class="flex items-center gap-2">
      <span class="loading loading-spinner"></span>
    </div>
    <div v-else-if="error" class="alert alert-error">{{ error }}</div>
    <div v-else-if="post">
      <h2 class="font-bold">{{ post.title }}</h2>
      <p>{{ post.body }}</p>
    </div>
  </div>
</template>
```

**Solution:**
```vue
<script setup lang="ts">
import { ref, watch, onMounted } from 'vue'

interface Post {
  id: number
  title: string
  body: string
}

const props = defineProps<{
  postId: number
}>()

const post = ref<Post | null>(null)
const loading = ref(false)
const error = ref<string | null>(null)

async function fetchPost(id: number) {
  loading.value = true
  error.value = null
  
  try {
    const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`)
    if (!response.ok) throw new Error(`Status: ${response.status}`)
    post.value = await response.json()
  } catch (e) {
    error.value = e instanceof Error ? e.message : 'Unknown error'
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  fetchPost(props.postId)
})

watch(() => props.postId, (newId) => {
  fetchPost(newId)
})
</script>

<template>
  <div class="p-4">
    <div v-if="loading" class="flex items-center gap-2">
      <span class="loading loading-spinner"></span>
    </div>
    <div v-else-if="error" class="alert alert-error">{{ error }}</div>
    <div v-else-if="post">
      <h2 class="font-bold">{{ post.title }}</h2>
      <p>{{ post.body }}</p>
    </div>
  </div>
</template>
```

---

## 5. Common Pitfalls & Anti-Patterns

| ❌ Common Mistake | ✅ Correct Approach | Why It Matters |
|:--- |:--- |:--- |
| Making `onMounted` callback async directly: `onMounted(async () => { await fetch() })` | Create a separate async function and call it: `onMounted(() => { fetchData() })` | While the async callback works, Vue doesn't await it. Separating the function makes the code clearer and allows reuse (e.g., for refresh buttons). |
| Forgetting to check `response.ok` before parsing JSON | Always check: `if (!response.ok) throw new Error(...)` | `fetch()` doesn't throw on 404/500 errors — it only throws on network failures. You must manually check HTTP status codes. |
| Not handling the case where component unmounts during fetch | Use `AbortController` and cancel in `onUnmounted` | If the component unmounts while a request is pending, updating state will cause a memory leak and potential errors. |

---

## 6. Mini-Project: GitHub User Search

Build a component that fetches and displays a GitHub user's profile based on username input.

**File Structure:**
```
src/
├── components/
│   └── GitHubProfile.vue
├── App.vue
└── main.ts
```

**GitHubProfile.vue:**
```vue
<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from 'vue'

interface GitHubUser {
  login: string
  avatar_url: string
  name: string | null
  bio: string | null
  public_repos: number
  followers: number
  following: number
  html_url: string
}

const props = defineProps<{
  username: string
}>()

const user = ref<GitHubUser | null>(null)
const loading = ref(false)
const error = ref<string | null>(null)

let abortController: AbortController | null = null

async function fetchUser(username: string) {
  if (abortController) {
    abortController.abort()
  }
  abortController = new AbortController()
  
  loading.value = true
  error.value = null
  user.value = null
  
  try {
    const response = await fetch(
      `https://api.github.com/users/${username}`,
      { signal: abortController.signal }
    )
    
    if (!response.ok) {
      if (response.status === 404) {
        throw new Error(`User "${username}" not found`)
      }
      throw new Error(`GitHub API error: ${response.status}`)
    }
    
    user.value = await response.json()
  } catch (e) {
    if (e instanceof Error && e.name !== 'AbortError') {
      error.value = e.message
    }
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  if (props.username) {
    fetchUser(props.username)
  }
})

watch(() => props.username, (newUsername) => {
  if (newUsername) {
    fetchUser(newUsername)
  }
})

onUnmounted(() => {
  abortController?.abort()
})
</script>

<template>
  <div class="card bg-base-100 shadow-xl max-w-md">
    <div class="card-body">
      <div v-if="loading" class="flex justify-center py-8">
        <span class="loading loading-spinner loading-lg"></span>
      </div>
      
      <div v-else-if="error" class="alert alert-error">
        <svg xmlns="http://www.w3.org/2000/svg" class="stroke-current shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
        <span>{{ error }}</span>
      </div>
      
      <div v-else-if="user" class="flex flex-col items-center">
        <div class="avatar">
          <div class="w-24 rounded-full ring ring-primary ring-offset-base-100 ring-offset-2">
            <img :src="user.avatar_url" :alt="`${user.login}'s avatar`" />
          </div>
        </div>
        
        <h2 class="card-title mt-4">{{ user.name || user.login }}</h2>
        <p class="text-base-content/60">@{{ user.login }}</p>
        
        <p v-if="user.bio" class="text-center mt-2">{{ user.bio }}</p>
        
        <div class="stats stats-vertical lg:stats-horizontal shadow mt-4">
          <div class="stat place-items-center">
            <div class="stat-title">Repos</div>
            <div class="stat-value text-lg">{{ user.public_repos }}</div>
          </div>
          <div class="stat place-items-center">
            <div class="stat-title">Followers</div>
            <div class="stat-value text-lg">{{ user.followers }}</div>
          </div>
          <div class="stat place-items-center">
            <div class="stat-title">Following</div>
            <div class="stat-value text-lg">{{ user.following }}</div>
          </div>
        </div>
        
        <div class="card-actions mt-4">
          <a :href="user.html_url" target="_blank" class="btn btn-primary">
            View Profile
          </a>
        </div>
      </div>
      
      <div v-else class="text-center py-8 text-base-content/60">
        Enter a username to search
      </div>
    </div>
  </div>
</template>
```

**App.vue:**
```vue
<script setup lang="ts">
import { ref } from 'vue'
import GitHubProfile from './components/GitHubProfile.vue'

const username = ref('')
const searchedUsername = ref('octocat')

function handleSearch() {
  if (username.value.trim()) {
    searchedUsername.value = username.value.trim()
  }
}
</script>

<template>
  <div class="min-h-screen bg-base-200 p-8">
    <div class="max-w-md mx-auto">
      <h1 class="text-2xl font-bold text-center mb-6">GitHub User Search</h1>
      
      <div class="join w-full mb-6">
        <input
          v-model="username"
          type="text"
          placeholder="Enter GitHub username"
          class="input input-bordered join-item flex-1"
          @keyup.enter="handleSearch"
        />
        <button class="btn btn-primary join-item" @click="handleSearch">
          Search
        </button>
      </div>
      
      <GitHubProfile :username="searchedUsername" />
    </div>
  </div>
</template>
```

**How to Test:**
1. Run `npm run dev` to start the development server
2. The app loads with "octocat" (GitHub's mascot) by default
3. Type a different username (e.g., "vuejs", "yyx990803") and press Enter or click Search
4. Observe the loading spinner while fetching
5. Try an invalid username (e.g., "thisisnotarealuser12345") to see the error state
6. Open DevTools Network tab to see the API requests

---

## 7. Implementation Exercises (Progressive)

### Level 1 (Recall): Fill in the Blanks

```vue
<script setup lang="ts">
import { _____, onMounted } from 'vue'

interface Todo {
  id: number
  title: string
  completed: boolean
}

const todos = ref<_____[]>([])
const loading = ref(_____)

async function fetchTodos() {
  loading.value = _____
  const response = await _____('https://jsonplaceholder.typicode.com/todos?_limit=5')
  todos.value = await response._____()
  loading.value = _____
}

_____(fetchTodos)
</script>
```

**Answer:** `ref`, `Todo`, `false`, `true`, `fetch`, `json`, `false`, `onMounted`

---

### Level 2 (Apply): Small Coding Tasks

**Exercise 2.1:** Create a component that fetches and displays a list of 5 photos from `https://jsonplaceholder.typicode.com/photos?_limit=5`. Each photo has `id`, `title`, `url`, and `thumbnailUrl`. Display the photos in a grid with their titles.

**Exercise 2.2:** Add a "Refresh" button to your photo gallery that re-fetches the data when clicked. The button should be disabled while loading.

---

### Level 3 (Debug): Fix the Broken Code

```vue
<script setup lang="ts">
import { ref } from 'vue'

interface Comment {
  id: number
  body: string
  email: string
}

const comments = ref<Comment[]>([])
const loading = ref(false)
const error = ref(null)

// BUG 1: Where should this function be called?
async function fetchComments() {
  loading = true  // BUG 2: Something wrong here
  
  // BUG 3: Missing error handling
  const response = await fetch('https://jsonplaceholder.typicode.com/comments?_limit=3')
  comments.value = response.json()  // BUG 4: Something wrong here
  
  loading = false  // BUG 5: Same issue as BUG 2
}

fetchComments()  // BUG 6: This runs during setup, not mount
</script>

<template>
  <div>
    <div v-if="loading">Loading...</div>
    <div v-for="comment in comments">  <!-- BUG 7: Missing key -->
      <p>{{ comment.body }}</p>
    </div>
  </div>
</template>
```

**Bugs to Fix:**
1. Missing `onMounted` import and usage
2. `loading = true` should be `loading.value = true`
3. No try/catch for error handling
4. Missing `await` before `response.json()`
5. Same as BUG 2
6. Should be inside `onMounted`
7. Missing `:key` attribute in `v-for`

**Corrected Code:**
```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface Comment {
  id: number
  body: string
  email: string
}

const comments = ref<Comment[]>([])
const loading = ref(false)
const error = ref<string | null>(null)

async function fetchComments() {
  loading.value = true
  error.value = null
  
  try {
    const response = await fetch('https://jsonplaceholder.typicode.com/comments?_limit=3')
    if (!response.ok) throw new Error(`Status: ${response.status}`)
    comments.value = await response.json()
  } catch (e) {
    error.value = e instanceof Error ? e.message : 'Unknown error'
  } finally {
    loading.value = false
  }
}

onMounted(fetchComments)
</script>

<template>
  <div>
    <div v-if="loading">Loading...</div>
    <div v-else-if="error" class="text-red-500">{{ error }}</div>
    <div v-else v-for="comment in comments" :key="comment.id">
      <p>{{ comment.body }}</p>
    </div>
  </div>
</template>
```

---

*Reply 'next' for Lesson 17.1 (Theory).*
