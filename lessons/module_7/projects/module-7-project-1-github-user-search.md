# Module 7 Project 1: GitHub User Search

## Project Overview

| Field | Value |
|:--- |:--- |
| **Module** | Module 7: Data Fetching & APIs |
| **Project** | GitHub User Search |
| **Difficulty** | Beginner |
| **Estimated Time** | 45-60 minutes |

**What You'll Build:** A developer tool that fetches GitHub user profiles by username and displays the user's avatar, name, bio, repository count, followers, and recent repositories. Users can search for any GitHub username and see their public profile information.

**Real-World Scenario:** Developer portfolios, team directories, and recruitment tools often integrate with GitHub's API to display developer profiles and activity. This project teaches you how to interact with a real-world REST API.

---

## Patterns You'll Practice

| Pattern | Description |
|:--- |:--- |
| `fetch()` with async/await | Making HTTP GET requests to external APIs |
| Loading states | Showing spinners while data is being fetched |
| Error handling | Displaying user-friendly error messages |
| Reactive data binding | Updating the UI when data changes |
| TypeScript interfaces | Defining types for API responses |
| Conditional rendering | Showing different UI based on state |
| Event handling | Responding to user input (search) |
| Computed properties | Deriving values from fetched data |

---

## Component Structure

```
src/
├── components/
│   ├── SearchBar.vue          # Username input with search button
│   ├── UserProfile.vue        # Main profile card display
│   ├── RepoList.vue           # List of user's repositories
│   ├── StatBadge.vue          # Reusable stat display component
│   └── LoadingSpinner.vue     # Reusable loading indicator
├── types/
│   └── github.ts              # TypeScript interfaces for GitHub API
├── App.vue                    # Main application component
└── main.ts                    # Application entry point
```

---

## Step-by-Step Guide

### Step 1: Project Setup

Create a new Vue project with TypeScript and Tailwind CSS:

```bash
npm create vite@latest github-user-search -- --template vue-ts
cd github-user-search
npm install
npm install -D tailwindcss postcss autoprefixer daisyui
npx tailwindcss init -p
```

**tailwind.config.js:**
```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{vue,js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [require("daisyui")],
  daisyui: {
    themes: ["dark", "light"],
  },
}
```

**src/style.css:**
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

---

### Step 2: Define TypeScript Types

Create type definitions for the GitHub API responses.

**src/types/github.ts:**
```typescript
// TODO: Define the GitHubUser interface based on GitHub's API response
// Required fields: login, avatar_url, name, bio, public_repos, followers, following, html_url, created_at

export interface GitHubUser {
  // YOUR CODE HERE
}

// TODO: Define the GitHubRepo interface for repository data
// Required fields: id, name, description, html_url, stargazers_count, forks_count, language

export interface GitHubRepo {
  // YOUR CODE HERE
}
```

---

### Step 3: Create the Loading Spinner Component

A reusable loading indicator component.

**src/components/LoadingSpinner.vue:**
```vue
<script setup lang="ts">
// TODO: Define optional props for 'size' (sm, md, lg) with default 'md'

</script>

<template>
  <!-- TODO: Create a centered loading spinner using DaisyUI classes -->
  <!-- The size should change based on the 'size' prop -->

</template>
```

---

### Step 4: Create the Stat Badge Component

A reusable component for displaying statistics.

**src/components/StatBadge.vue:**
```vue
<script setup lang="ts">
// TODO: Define props for 'label' (string) and 'value' (number)

</script>

<template>
  <!-- TODO: Create a stat display with the label above and value below -->
  <!-- Use DaisyUI stat classes for styling -->

</template>
```

---

### Step 5: Create the Search Bar Component

The search input that emits the username to search.

**src/components/SearchBar.vue:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

// TODO: Define emits for 'search' event that passes the username string
const emit = defineEmits<{
  // YOUR CODE HERE
}>()

const username = ref('')

// TODO: Create a 'handleSearch' function that:
// 1. Trims the username
// 2. Emits the 'search' event if username is not empty
// 3. Does NOT clear the input (user might want to modify their search)

</script>

<template>
  <!-- TODO: Create a search form with:
       - Text input bound to 'username' with v-model
       - Search button that triggers handleSearch
       - Form should submit on Enter key
       - Use DaisyUI 'join' classes for connected input+button -->

</template>
```

---

### Step 6: Create the Repository List Component

Displays a list of the user's repositories.

**src/components/RepoList.vue:**
```vue
<script setup lang="ts">
import type { GitHubRepo } from '@/types/github'

// TODO: Define props for 'repos' (array of GitHubRepo)

</script>

<template>
  <!-- TODO: Create a list of repository cards showing:
       - Repository name (linked to html_url)
       - Description (or "No description" if null)
       - Language badge (if language exists)
       - Star count and fork count
       - Use v-for with :key="repo.id" -->

</template>
```

---

### Step 7: Create the User Profile Component

The main component that fetches and displays user data.

**src/components/UserProfile.vue:**
```vue
<script setup lang="ts">
import { ref, watch } from 'vue'
import type { GitHubUser, GitHubRepo } from '@/types/github'
import LoadingSpinner from './LoadingSpinner.vue'
import StatBadge from './StatBadge.vue'
import RepoList from './RepoList.vue'

const props = defineProps<{
  username: string
}>()

// TODO: Create refs for:
// - user: GitHubUser | null
// - repos: GitHubRepo[]
// - loading: boolean
// - error: string | null

// TODO: Create an async function 'fetchUserData' that:
// 1. Sets loading to true and clears previous error
// 2. Fetches user data from: https://api.github.com/users/${username}
// 3. Fetches repos from: https://api.github.com/users/${username}/repos?sort=updated&per_page=6
// 4. Handles errors (404 = "User not found", other = error message)
// 5. Sets loading to false in finally block

// TODO: Watch the 'username' prop and call fetchUserData when it changes
// Use { immediate: true } to fetch on initial render

</script>

<template>
  <div class="w-full max-w-2xl">
    <!-- Loading State -->
    <!-- TODO: Show LoadingSpinner when loading is true -->

    <!-- Error State -->
    <!-- TODO: Show error alert when error is not null -->

    <!-- User Profile -->
    <!-- TODO: Show user profile when user is not null, including:
         - Avatar image
         - Name and @username
         - Bio
         - Stats (repos, followers, following)
         - "View Profile" link
         - RepoList component with the repos -->

  </div>
</template>
```

---

### Step 8: Assemble the App

Put all components together in the main App component.

**src/App.vue:**
```vue
<script setup lang="ts">
import { ref } from 'vue'
import SearchBar from './components/SearchBar.vue'
import UserProfile from './components/UserProfile.vue'

// TODO: Create a ref 'searchedUsername' with initial value 'octocat' (GitHub's mascot)

// TODO: Create a function 'handleSearch' that updates searchedUsername

</script>

<template>
  <!-- TODO: Create the app layout with:
       - Header with title "GitHub User Search"
       - SearchBar component that calls handleSearch on search
       - UserProfile component that receives searchedUsername as prop
       - Use a gradient background and center the content -->

</template>
```

---

## Complete Solution

### src/types/github.ts
```typescript
export interface GitHubUser {
  login: string
  avatar_url: string
  name: string | null
  bio: string | null
  public_repos: number
  followers: number
  following: number
  html_url: string
  created_at: string
  location: string | null
  company: string | null
  blog: string | null
}

export interface GitHubRepo {
  id: number
  name: string
  description: string | null
  html_url: string
  stargazers_count: number
  forks_count: number
  language: string | null
  updated_at: string
}
```

### src/components/LoadingSpinner.vue
```vue
<script setup lang="ts">
withDefaults(defineProps<{
  size?: 'sm' | 'md' | 'lg'
}>(), {
  size: 'md'
})
</script>

<template>
  <div class="flex justify-center items-center py-8">
    <span 
      class="loading loading-spinner"
      :class="{
        'loading-sm': size === 'sm',
        'loading-md': size === 'md',
        'loading-lg': size === 'lg'
      }"
    ></span>
  </div>
</template>
```

### src/components/StatBadge.vue
```vue
<script setup lang="ts">
defineProps<{
  label: string
  value: number
}>()
</script>

<template>
  <div class="stat place-items-center p-2">
    <div class="stat-title text-xs">{{ label }}</div>
    <div class="stat-value text-lg">{{ value.toLocaleString() }}</div>
  </div>
</template>
```

### src/components/SearchBar.vue
```vue
<script setup lang="ts">
import { ref } from 'vue'

const emit = defineEmits<{
  search: [username: string]
}>()

const username = ref('')

function handleSearch() {
  const trimmed = username.value.trim()
  if (trimmed) {
    emit('search', trimmed)
  }
}
</script>

<template>
  <form @submit.prevent="handleSearch" class="w-full max-w-md">
    <div class="join w-full">
      <input
        v-model="username"
        type="text"
        placeholder="Enter GitHub username..."
        class="input input-bordered join-item flex-1"
      />
      <button type="submit" class="btn btn-primary join-item">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
        </svg>
        Search
      </button>
    </div>
  </form>
</template>
```

### src/components/RepoList.vue
```vue
<script setup lang="ts">
import type { GitHubRepo } from '@/types/github'

defineProps<{
  repos: GitHubRepo[]
}>()
</script>

<template>
  <div class="space-y-3">
    <h3 class="text-lg font-semibold">Recent Repositories</h3>
    
    <div v-if="repos.length === 0" class="text-base-content/60 text-center py-4">
      No public repositories
    </div>
    
    <a
      v-for="repo in repos"
      :key="repo.id"
      :href="repo.html_url"
      target="_blank"
      rel="noopener noreferrer"
      class="card bg-base-200 hover:bg-base-300 transition-colors cursor-pointer block"
    >
      <div class="card-body p-4">
        <div class="flex items-start justify-between gap-2">
          <div class="flex-1 min-w-0">
            <h4 class="font-medium text-primary truncate">{{ repo.name }}</h4>
            <p class="text-sm text-base-content/60 line-clamp-2">
              {{ repo.description || 'No description' }}
            </p>
          </div>
          <div v-if="repo.language" class="badge badge-outline badge-sm shrink-0">
            {{ repo.language }}
          </div>
        </div>
        
        <div class="flex gap-4 mt-2 text-sm text-base-content/60">
          <span class="flex items-center gap-1">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11.049 2.927c.3-.921 1.603-.921 1.902 0l1.519 4.674a1 1 0 00.95.69h4.915c.969 0 1.371 1.24.588 1.81l-3.976 2.888a1 1 0 00-.363 1.118l1.518 4.674c.3.922-.755 1.688-1.538 1.118l-3.976-2.888a1 1 0 00-1.176 0l-3.976 2.888c-.783.57-1.838-.197-1.538-1.118l1.518-4.674a1 1 0 00-.363-1.118l-3.976-2.888c-.784-.57-.38-1.81.588-1.81h4.914a1 1 0 00.951-.69l1.519-4.674z" />
            </svg>
            {{ repo.stargazers_count }}
          </span>
          <span class="flex items-center gap-1">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8.684 13.342C8.886 12.938 9 12.482 9 12c0-.482-.114-.938-.316-1.342m0 2.684a3 3 0 110-2.684m0 2.684l6.632 3.316m-6.632-6l6.632-3.316m0 0a3 3 0 105.367-2.684 3 3 0 00-5.367 2.684zm0 9.316a3 3 0 105.368 2.684 3 3 0 00-5.368-2.684z" />
            </svg>
            {{ repo.forks_count }}
          </span>
        </div>
      </div>
    </a>
  </div>
</template>
```

### src/components/UserProfile.vue
```vue
<script setup lang="ts">
import { ref, watch } from 'vue'
import type { GitHubUser, GitHubRepo } from '@/types/github'
import LoadingSpinner from './LoadingSpinner.vue'
import StatBadge from './StatBadge.vue'
import RepoList from './RepoList.vue'

const props = defineProps<{
  username: string
}>()

const user = ref<GitHubUser | null>(null)
const repos = ref<GitHubRepo[]>([])
const loading = ref(false)
const error = ref<string | null>(null)

async function fetchUserData() {
  if (!props.username) return
  
  loading.value = true
  error.value = null
  user.value = null
  repos.value = []
  
  try {
    const userResponse = await fetch(`https://api.github.com/users/${props.username}`)
    
    if (!userResponse.ok) {
      if (userResponse.status === 404) {
        throw new Error(`User "${props.username}" not found`)
      }
      throw new Error(`GitHub API error: ${userResponse.status}`)
    }
    
    user.value = await userResponse.json()
    
    const reposResponse = await fetch(
      `https://api.github.com/users/${props.username}/repos?sort=updated&per_page=6`
    )
    
    if (reposResponse.ok) {
      repos.value = await reposResponse.json()
    }
  } catch (e) {
    error.value = e instanceof Error ? e.message : 'An unknown error occurred'
  } finally {
    loading.value = false
  }
}

watch(() => props.username, fetchUserData, { immediate: true })
</script>

<template>
  <div class="w-full max-w-2xl">
    <LoadingSpinner v-if="loading" size="lg" />
    
    <div v-else-if="error" class="alert alert-error">
      <svg xmlns="http://www.w3.org/2000/svg" class="stroke-current shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z" />
      </svg>
      <span>{{ error }}</span>
    </div>
    
    <div v-else-if="user" class="card bg-base-100 shadow-xl">
      <div class="card-body">
        <div class="flex flex-col sm:flex-row items-center sm:items-start gap-6">
          <div class="avatar">
            <div class="w-32 rounded-full ring ring-primary ring-offset-base-100 ring-offset-2">
              <img :src="user.avatar_url" :alt="`${user.login}'s avatar`" />
            </div>
          </div>
          
          <div class="flex-1 text-center sm:text-left">
            <h2 class="text-2xl font-bold">{{ user.name || user.login }}</h2>
            <p class="text-base-content/60">@{{ user.login }}</p>
            
            <p v-if="user.bio" class="mt-2">{{ user.bio }}</p>
            
            <div v-if="user.location || user.company" class="flex flex-wrap gap-4 mt-2 text-sm text-base-content/60">
              <span v-if="user.location" class="flex items-center gap-1">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z" />
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 11a3 3 0 11-6 0 3 3 0 016 0z" />
                </svg>
                {{ user.location }}
              </span>
              <span v-if="user.company" class="flex items-center gap-1">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 21V5a2 2 0 00-2-2H7a2 2 0 00-2 2v16m14 0h2m-2 0h-5m-9 0H3m2 0h5M9 7h1m-1 4h1m4-4h1m-1 4h1m-5 10v-5a1 1 0 011-1h2a1 1 0 011 1v5m-4 0h4" />
                </svg>
                {{ user.company }}
              </span>
            </div>
          </div>
        </div>
        
        <div class="stats stats-vertical sm:stats-horizontal shadow my-4">
          <StatBadge label="Repositories" :value="user.public_repos" />
          <StatBadge label="Followers" :value="user.followers" />
          <StatBadge label="Following" :value="user.following" />
        </div>
        
        <div class="card-actions justify-center mb-4">
          <a :href="user.html_url" target="_blank" rel="noopener noreferrer" class="btn btn-primary">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="currentColor" viewBox="0 0 24 24">
              <path d="M12 0c-6.626 0-12 5.373-12 12 0 5.302 3.438 9.8 8.207 11.387.599.111.793-.261.793-.577v-2.234c-3.338.726-4.033-1.416-4.033-1.416-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.839 1.237 1.839 1.237 1.07 1.834 2.807 1.304 3.492.997.107-.775.418-1.305.762-1.604-2.665-.305-5.467-1.334-5.467-5.931 0-1.311.469-2.381 1.236-3.221-.124-.303-.535-1.524.117-3.176 0 0 1.008-.322 3.301 1.23.957-.266 1.983-.399 3.003-.404 1.02.005 2.047.138 3.006.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.874.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.609-2.807 5.624-5.479 5.921.43.372.823 1.102.823 2.222v3.293c0 .319.192.694.801.576 4.765-1.589 8.199-6.086 8.199-11.386 0-6.627-5.373-12-12-12z"/>
            </svg>
            View GitHub Profile
          </a>
        </div>
        
        <div class="divider"></div>
        
        <RepoList :repos="repos" />
      </div>
    </div>
  </div>
</template>
```

### src/App.vue
```vue
<script setup lang="ts">
import { ref } from 'vue'
import SearchBar from './components/SearchBar.vue'
import UserProfile from './components/UserProfile.vue'

const searchedUsername = ref('octocat')

function handleSearch(username: string) {
  searchedUsername.value = username
}
</script>

<template>
  <div class="min-h-screen bg-gradient-to-br from-slate-900 via-purple-900 to-slate-900">
    <div class="container mx-auto px-4 py-8">
      <header class="text-center mb-8">
        <h1 class="text-4xl font-bold text-white mb-2">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-10 w-10 inline-block mr-2 -mt-1" fill="currentColor" viewBox="0 0 24 24">
            <path d="M12 0c-6.626 0-12 5.373-12 12 0 5.302 3.438 9.8 8.207 11.387.599.111.793-.261.793-.577v-2.234c-3.338.726-4.033-1.416-4.033-1.416-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.839 1.237 1.839 1.237 1.07 1.834 2.807 1.304 3.492.997.107-.775.418-1.305.762-1.604-2.665-.305-5.467-1.334-5.467-5.931 0-1.311.469-2.381 1.236-3.221-.124-.303-.535-1.524.117-3.176 0 0 1.008-.322 3.301 1.23.957-.266 1.983-.399 3.003-.404 1.02.005 2.047.138 3.006.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.874.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.609-2.807 5.624-5.479 5.921.43.372.823 1.102.823 2.222v3.293c0 .319.192.694.801.576 4.765-1.589 8.199-6.086 8.199-11.386 0-6.627-5.373-12-12-12z"/>
          </svg>
          GitHub User Search
        </h1>
        <p class="text-white/60">Search for any GitHub user to view their profile</p>
      </header>
      
      <div class="flex flex-col items-center gap-8">
        <SearchBar @search="handleSearch" />
        <UserProfile :username="searchedUsername" />
      </div>
    </div>
  </div>
</template>
```

### src/main.ts
```typescript
import { createApp } from 'vue'
import App from './App.vue'
import './style.css'

createApp(App).mount('#app')
```

---

## Stretch Goals

1. **Search History:** Store recent searches in localStorage and display them as quick-select buttons
2. **Rate Limit Display:** Show GitHub API rate limit status (60 requests/hour for unauthenticated)
3. **Repository Filtering:** Add tabs to filter repos by language or sort by stars/forks
4. **User Comparison:** Allow searching two users and comparing their stats side-by-side
5. **Follower/Following Lists:** Add expandable sections to view follower and following lists
6. **Activity Graph:** Fetch and display the user's contribution activity

---

## Self-Assessment Checklist

- [ ] All components use `<script setup>` with TypeScript
- [ ] TypeScript interfaces properly define all API response shapes
- [ ] Loading states display while fetching data
- [ ] Error states handle both 404 (user not found) and other errors
- [ ] The UI updates reactively when searching for new users
- [ ] All links open in new tabs with proper security attributes
- [ ] The design is responsive and works on mobile devices
- [ ] Code follows Vue 3 Composition API patterns

---

## Testing Your Implementation

1. **Default Load:** App should load with "octocat" profile displayed
2. **Search Valid User:** Search for "vuejs" — should display Vue.js organization profile
3. **Search Invalid User:** Search for "thisisnotarealuser12345678" — should show error message
4. **Rapid Searches:** Quickly search multiple users — only the last search should display
5. **Mobile View:** Resize browser to mobile width — layout should stack vertically
6. **External Links:** Click "View GitHub Profile" — should open GitHub in new tab
