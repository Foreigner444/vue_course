# Guided Project 2: User Profile Dashboard

## Project Overview

| Field | Value |
|:--- |:--- |
| **Module** | Module 8: Navigation |
| **Project Number** | 2 of 3 |
| **Difficulty** | Intermediate |
| **Estimated Time** | 60-75 minutes |

**What You'll Build:** A social media-style user profile dashboard with nested routes for different profile sections (overview, posts, settings, activity). Users can navigate between profiles using dynamic routes and explore different tabs within each profile.

**Real-World Scenario:** A social app has nested routes for profile overview, settings, and activity — similar to Twitter/X profiles, LinkedIn user pages, or GitHub profile sections.

---

## Patterns You'll Practice

| Pattern | Description |
|:--- |:--- |
| Dynamic routes (`:userId`) | Match variable URL segments for user IDs |
| `useRoute().params` | Access dynamic route parameters |
| Nested routes | Define child routes that render inside parent layouts |
| Multiple `<RouterView>` | Parent and child RouterViews for layout composition |
| `watch()` for params | React to route parameter changes |
| Named routes with params | Navigate programmatically with `router.push()` |
| Route meta fields | Store custom data on routes |
| Active link styling | Highlight current tab in nested navigation |

---

## Component Structure

```
src/
├── components/
│   ├── NavBar.vue              # Main site navigation
│   ├── UserCard.vue            # Compact user preview card
│   └── ProfileTabs.vue         # Tab navigation for profile sections
├── layouts/
│   └── ProfileLayout.vue       # Parent layout with sidebar + RouterView
├── pages/
│   ├── HomePage.vue            # User listing/discovery page
│   ├── profile/
│   │   ├── OverviewTab.vue     # Default profile tab
│   │   ├── PostsTab.vue        # User's posts
│   │   ├── ActivityTab.vue     # Recent activity feed
│   │   └── SettingsTab.vue     # Profile settings (own profile only)
│   └── NotFoundPage.vue        # 404 page
├── data/
│   └── users.ts                # Mock user data
├── composables/
│   └── useCurrentUser.ts       # Simulated current user state
├── router.ts                   # Router configuration
├── App.vue
├── main.ts
└── style.css
```

---

## Step-by-Step Guide

### Step 1: Project Setup

```bash
npm create vite@latest user-profile-dashboard -- --template vue-ts
cd user-profile-dashboard
npm install
npm install vue-router@4
npm install -D tailwindcss postcss autoprefixer daisyui@latest
npx tailwindcss init -p
```

Configure Tailwind:

```javascript
// tailwind.config.js
export default {
  content: ["./index.html", "./src/**/*.{vue,js,ts,jsx,tsx}"],
  theme: { extend: {} },
  plugins: [require('daisyui')],
  daisyui: { themes: ["light", "dark", "synthwave"] }
}
```

```css
/* src/style.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

---

### Step 2: Create Mock User Data

```typescript
// src/data/users.ts
export interface User {
  id: string
  username: string
  displayName: string
  avatar: string
  bio: string
  location: string
  website: string
  joinedDate: string
  followers: number
  following: number
  isVerified: boolean
}

export interface Post {
  id: string
  userId: string
  content: string
  likes: number
  comments: number
  createdAt: string
}

export interface Activity {
  id: string
  userId: string
  type: 'follow' | 'like' | 'post' | 'comment'
  targetUser?: string
  targetPost?: string
  createdAt: string
}

export const users: User[] = [
  {
    id: '1',
    username: 'sarahdev',
    displayName: 'Sarah Chen',
    avatar: 'https://i.pravatar.cc/150?img=1',
    bio: 'Full-stack developer 👩‍💻 | Vue.js enthusiast | Building cool stuff',
    location: 'San Francisco, CA',
    website: 'https://sarahchen.dev',
    joinedDate: '2021-03-15',
    followers: 12500,
    following: 890,
    isVerified: true
  },
  {
    id: '2',
    username: 'alexcodes',
    displayName: 'Alex Rivera',
    avatar: 'https://i.pravatar.cc/150?img=3',
    bio: 'Open source contributor | TypeScript fanatic | Coffee addict ☕',
    location: 'Austin, TX',
    website: 'https://alexrivera.io',
    joinedDate: '2020-08-22',
    followers: 8200,
    following: 1250,
    isVerified: true
  },
  {
    id: '3',
    username: 'emmadesigns',
    displayName: 'Emma Wilson',
    avatar: 'https://i.pravatar.cc/150?img=5',
    bio: 'UI/UX Designer | Making the web beautiful one pixel at a time ✨',
    location: 'London, UK',
    website: 'https://emmawilson.design',
    joinedDate: '2022-01-10',
    followers: 5600,
    following: 420,
    isVerified: false
  },
  {
    id: '4',
    username: 'marcustech',
    displayName: 'Marcus Johnson',
    avatar: 'https://i.pravatar.cc/150?img=8',
    bio: 'DevOps engineer | Cloud architecture | Kubernetes evangelist',
    location: 'Seattle, WA',
    website: 'https://marcus.tech',
    joinedDate: '2019-11-05',
    followers: 15800,
    following: 560,
    isVerified: true
  },
  {
    id: '5',
    username: 'linaml',
    displayName: 'Lina Martinez',
    avatar: 'https://i.pravatar.cc/150?img=9',
    bio: 'Machine Learning Engineer | AI researcher | Data science mentor',
    location: 'Toronto, Canada',
    website: 'https://linaml.com',
    joinedDate: '2020-05-18',
    followers: 22100,
    following: 310,
    isVerified: true
  }
]

export const posts: Post[] = [
  { id: 'p1', userId: '1', content: 'Just shipped a new feature using Vue 3 Composition API. The developer experience is incredible! 🚀', likes: 245, comments: 32, createdAt: '2024-01-20T10:30:00Z' },
  { id: 'p2', userId: '1', content: 'Hot take: TypeScript makes you a better JavaScript developer, even if you go back to JS later.', likes: 892, comments: 156, createdAt: '2024-01-18T14:15:00Z' },
  { id: 'p3', userId: '1', content: 'Working on a new open source project. Stay tuned for the announcement! 👀', likes: 567, comments: 89, createdAt: '2024-01-15T09:00:00Z' },
  { id: 'p4', userId: '2', content: 'Spent the weekend refactoring legacy code. It\'s like archaeology, but with more coffee.', likes: 423, comments: 67, createdAt: '2024-01-19T16:45:00Z' },
  { id: 'p5', userId: '2', content: 'PSA: Always write tests before you refactor. Future you will thank present you.', likes: 1205, comments: 201, createdAt: '2024-01-17T11:20:00Z' },
  { id: 'p6', userId: '3', content: 'New design system just dropped! Check out the Figma file in the comments 🎨', likes: 678, comments: 45, createdAt: '2024-01-21T08:30:00Z' },
  { id: 'p7', userId: '4', content: 'Kubernetes tip: Use resource limits. Your cluster will thank you.', likes: 534, comments: 78, createdAt: '2024-01-20T13:00:00Z' },
  { id: 'p8', userId: '5', content: 'Just published my paper on transformer architectures. Link in bio! 📄', likes: 1890, comments: 234, createdAt: '2024-01-19T09:15:00Z' }
]

export const activities: Activity[] = [
  { id: 'a1', userId: '1', type: 'follow', targetUser: '5', createdAt: '2024-01-21T10:00:00Z' },
  { id: 'a2', userId: '1', type: 'like', targetPost: 'p8', createdAt: '2024-01-21T09:30:00Z' },
  { id: 'a3', userId: '1', type: 'post', targetPost: 'p1', createdAt: '2024-01-20T10:30:00Z' },
  { id: 'a4', userId: '1', type: 'comment', targetPost: 'p7', createdAt: '2024-01-20T08:15:00Z' },
  { id: 'a5', userId: '2', type: 'follow', targetUser: '1', createdAt: '2024-01-20T14:00:00Z' },
  { id: 'a6', userId: '2', type: 'like', targetPost: 'p1', createdAt: '2024-01-20T12:00:00Z' },
  { id: 'a7', userId: '3', type: 'post', targetPost: 'p6', createdAt: '2024-01-21T08:30:00Z' },
  { id: 'a8', userId: '4', type: 'like', targetPost: 'p5', createdAt: '2024-01-19T10:00:00Z' }
]

export function getUserById(id: string): User | undefined {
  return users.find(user => user.id === id)
}

export function getUserPosts(userId: string): Post[] {
  return posts.filter(post => post.userId === userId)
    .sort((a, b) => new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime())
}

export function getUserActivity(userId: string): Activity[] {
  return activities.filter(activity => activity.userId === userId)
    .sort((a, b) => new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime())
}

export function formatNumber(num: number): string {
  if (num >= 1000000) return (num / 1000000).toFixed(1) + 'M'
  if (num >= 1000) return (num / 1000).toFixed(1) + 'K'
  return num.toString()
}

export function formatDate(dateString: string): string {
  const date = new Date(dateString)
  return date.toLocaleDateString('en-US', { 
    month: 'short', 
    day: 'numeric',
    year: 'numeric'
  })
}

export function formatRelativeTime(dateString: string): string {
  const date = new Date(dateString)
  const now = new Date()
  const diffMs = now.getTime() - date.getTime()
  const diffMins = Math.floor(diffMs / 60000)
  const diffHours = Math.floor(diffMs / 3600000)
  const diffDays = Math.floor(diffMs / 86400000)
  
  if (diffMins < 60) return `${diffMins}m ago`
  if (diffHours < 24) return `${diffHours}h ago`
  if (diffDays < 7) return `${diffDays}d ago`
  return formatDate(dateString)
}
```

---

### Step 3: Create Current User Composable

```typescript
// src/composables/useCurrentUser.ts
import { ref, computed } from 'vue'
import { getUserById, type User } from '../data/users'

const currentUserId = ref<string>('1')

export function useCurrentUser() {
  const currentUser = computed<User | undefined>(() => 
    getUserById(currentUserId.value)
  )
  
  const isOwnProfile = (profileUserId: string) => {
    return currentUserId.value === profileUserId
  }
  
  const switchUser = (userId: string) => {
    currentUserId.value = userId
  }
  
  return {
    currentUser,
    currentUserId,
    isOwnProfile,
    switchUser
  }
}
```

---

### Step 4: Create Router Configuration

```typescript
// src/router.ts
import { createRouter, createWebHistory } from 'vue-router'

// TODO: Configure nested routes for the profile section
const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  
  routes: [
    {
      path: '/',
      name: 'home',
      component: () => import('./pages/HomePage.vue'),
      meta: { title: 'Discover' }
    },
    {
      // Dynamic route: matches /profile/1, /profile/2, etc.
      path: '/profile/:userId',
      component: () => import('./layouts/ProfileLayout.vue'),
      // Nested routes render inside ProfileLayout's <RouterView>
      children: [
        {
          // Empty path = default child route
          // Renders at /profile/:userId
          path: '',
          name: 'profile-overview',
          component: () => import('./pages/profile/OverviewTab.vue'),
          meta: { title: 'Profile', tab: 'overview' }
        },
        {
          // Renders at /profile/:userId/posts
          path: 'posts',
          name: 'profile-posts',
          component: () => import('./pages/profile/PostsTab.vue'),
          meta: { title: 'Posts', tab: 'posts' }
        },
        {
          // Renders at /profile/:userId/activity
          path: 'activity',
          name: 'profile-activity',
          component: () => import('./pages/profile/ActivityTab.vue'),
          meta: { title: 'Activity', tab: 'activity' }
        },
        {
          // Renders at /profile/:userId/settings
          path: 'settings',
          name: 'profile-settings',
          component: () => import('./pages/profile/SettingsTab.vue'),
          meta: { title: 'Settings', tab: 'settings', requiresOwner: true }
        }
      ]
    },
    {
      path: '/:pathMatch(.*)*',
      name: 'not-found',
      component: () => import('./pages/NotFoundPage.vue'),
      meta: { title: 'Not Found' }
    }
  ]
})

router.afterEach((to) => {
  document.title = `${to.meta.title} | SocialApp` || 'SocialApp'
})

export default router
```

---

### Step 5: Update Main Entry Point

```typescript
// src/main.ts
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import './style.css'

createApp(App).use(router).mount('#app')
```

---

### Step 6: Create App.vue

```vue
<!-- src/App.vue -->
<script setup lang="ts">
import NavBar from './components/NavBar.vue'
</script>

<template>
  <div class="min-h-screen bg-base-100" data-theme="synthwave">
    <NavBar />
    <RouterView />
  </div>
</template>
```

---

### Step 7: Create Navigation Component

```vue
<!-- src/components/NavBar.vue -->
<script setup lang="ts">
import { useCurrentUser } from '../composables/useCurrentUser'

const { currentUser } = useCurrentUser()
</script>

<template>
  <header class="navbar bg-base-200 shadow-lg sticky top-0 z-50">
    <div class="container mx-auto">
      <div class="flex-1">
        <RouterLink to="/" class="btn btn-ghost text-xl font-bold">
          🌐 SocialApp
        </RouterLink>
      </div>
      
      <div class="flex-none flex items-center gap-4">
        <RouterLink to="/" class="btn btn-ghost btn-sm">
          Discover
        </RouterLink>
        
        <RouterLink 
          v-if="currentUser"
          :to="{ name: 'profile-overview', params: { userId: currentUser.id } }"
          class="flex items-center gap-2 btn btn-ghost btn-sm"
        >
          <img 
            :src="currentUser.avatar" 
            :alt="currentUser.displayName"
            class="w-6 h-6 rounded-full"
          />
          <span class="hidden sm:inline">My Profile</span>
        </RouterLink>
      </div>
    </div>
  </header>
</template>
```

---

### Step 8: Create User Card Component

```vue
<!-- src/components/UserCard.vue -->
<script setup lang="ts">
import { type User, formatNumber } from '../data/users'

defineProps<{
  user: User
}>()
</script>

<template>
  <RouterLink 
    :to="{ name: 'profile-overview', params: { userId: user.id } }"
    class="card bg-base-200 shadow-xl hover:shadow-2xl transition-all hover:-translate-y-1"
  >
    <div class="card-body">
      <div class="flex items-center gap-4">
        <img 
          :src="user.avatar" 
          :alt="user.displayName"
          class="w-16 h-16 rounded-full"
        />
        <div class="flex-1 min-w-0">
          <div class="flex items-center gap-2">
            <h3 class="font-bold truncate">{{ user.displayName }}</h3>
            <span v-if="user.isVerified" class="text-primary">✓</span>
          </div>
          <p class="text-base-content/60 text-sm">@{{ user.username }}</p>
        </div>
      </div>
      
      <p class="text-sm mt-2 line-clamp-2">{{ user.bio }}</p>
      
      <div class="flex gap-4 mt-3 text-sm text-base-content/70">
        <span><strong>{{ formatNumber(user.followers) }}</strong> followers</span>
        <span><strong>{{ formatNumber(user.following) }}</strong> following</span>
      </div>
    </div>
  </RouterLink>
</template>
```

---

### Step 9: Create Profile Tabs Component

```vue
<!-- src/components/ProfileTabs.vue -->
<script setup lang="ts">
import { useRoute } from 'vue-router'
import { computed } from 'vue'
import { useCurrentUser } from '../composables/useCurrentUser'

const props = defineProps<{
  userId: string
}>()

const route = useRoute()
const { isOwnProfile } = useCurrentUser()

const currentTab = computed(() => route.meta.tab as string)
const showSettings = computed(() => isOwnProfile(props.userId))

const tabs = computed(() => {
  const baseTabs = [
    { name: 'profile-overview', label: 'Overview', tab: 'overview' },
    { name: 'profile-posts', label: 'Posts', tab: 'posts' },
    { name: 'profile-activity', label: 'Activity', tab: 'activity' }
  ]
  
  if (showSettings.value) {
    baseTabs.push({ name: 'profile-settings', label: 'Settings', tab: 'settings' })
  }
  
  return baseTabs
})
</script>

<template>
  <div class="tabs tabs-boxed bg-base-200 p-1">
    <RouterLink
      v-for="tab in tabs"
      :key="tab.name"
      :to="{ name: tab.name, params: { userId } }"
      :class="[
        'tab',
        currentTab === tab.tab && 'tab-active'
      ]"
    >
      {{ tab.label }}
    </RouterLink>
  </div>
</template>
```

---

### Step 10: Create Profile Layout (Parent with Nested RouterView)

```vue
<!-- src/layouts/ProfileLayout.vue -->
<script setup lang="ts">
import { computed, watch, ref } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { getUserById, formatNumber, formatDate, type User } from '../data/users'
import { useCurrentUser } from '../composables/useCurrentUser'
import ProfileTabs from '../components/ProfileTabs.vue'

const route = useRoute()
const router = useRouter()
const { isOwnProfile, currentUser } = useCurrentUser()

const user = ref<User | null>(null)
const loading = ref(true)

// TODO: Create a computed property to get userId from route params
const userId = computed(() => route.params.userId as string)

// Fetch user data
async function fetchUser(id: string) {
  loading.value = true
  // Simulate API delay
  await new Promise(resolve => setTimeout(resolve, 300))
  
  const foundUser = getUserById(id)
  user.value = foundUser || null
  loading.value = false
  
  // Redirect to 404 if user not found
  if (!foundUser) {
    router.replace({ name: 'not-found' })
  }
}

// Fetch on initial load
fetchUser(userId.value)

// TODO: Watch for userId changes to refetch data
// This is CRITICAL because the component is reused when navigating
// between profiles (e.g., /profile/1 to /profile/2)
watch(userId, (newId) => {
  if (newId) {
    fetchUser(newId)
  }
})

// Check if viewing settings tab without being owner
watch(
  () => route.meta.requiresOwner,
  (requiresOwner) => {
    if (requiresOwner && !isOwnProfile(userId.value)) {
      router.replace({ name: 'profile-overview', params: { userId: userId.value } })
    }
  },
  { immediate: true }
)

const isFollowing = ref(false)
function toggleFollow() {
  isFollowing.value = !isFollowing.value
}
</script>

<template>
  <!-- Loading State -->
  <div v-if="loading" class="flex justify-center items-center min-h-[50vh]">
    <span class="loading loading-spinner loading-lg"></span>
  </div>
  
  <!-- Profile Content -->
  <div v-else-if="user" class="container mx-auto px-4 py-8">
    <!-- Profile Header -->
    <div class="card bg-base-200 shadow-xl mb-8">
      <div class="card-body">
        <div class="flex flex-col md:flex-row gap-6">
          <!-- Avatar -->
          <div class="flex-shrink-0">
            <img 
              :src="user.avatar" 
              :alt="user.displayName"
              class="w-32 h-32 rounded-full ring ring-primary ring-offset-base-100 ring-offset-2"
            />
          </div>
          
          <!-- User Info -->
          <div class="flex-1">
            <div class="flex flex-col sm:flex-row sm:items-center gap-4 mb-4">
              <div>
                <div class="flex items-center gap-2">
                  <h1 class="text-2xl font-bold">{{ user.displayName }}</h1>
                  <span v-if="user.isVerified" class="badge badge-primary">✓ Verified</span>
                </div>
                <p class="text-base-content/60">@{{ user.username }}</p>
              </div>
              
              <!-- Action Button -->
              <div class="sm:ml-auto">
                <RouterLink 
                  v-if="isOwnProfile(userId)"
                  :to="{ name: 'profile-settings', params: { userId } }"
                  class="btn btn-outline"
                >
                  Edit Profile
                </RouterLink>
                <button 
                  v-else
                  @click="toggleFollow"
                  :class="[
                    'btn',
                    isFollowing ? 'btn-outline' : 'btn-primary'
                  ]"
                >
                  {{ isFollowing ? 'Following' : 'Follow' }}
                </button>
              </div>
            </div>
            
            <!-- Bio -->
            <p class="mb-4">{{ user.bio }}</p>
            
            <!-- Meta Info -->
            <div class="flex flex-wrap gap-4 text-sm text-base-content/70">
              <span v-if="user.location" class="flex items-center gap-1">
                📍 {{ user.location }}
              </span>
              <a 
                v-if="user.website" 
                :href="user.website" 
                target="_blank"
                class="flex items-center gap-1 link link-primary"
              >
                🔗 {{ user.website.replace('https://', '') }}
              </a>
              <span class="flex items-center gap-1">
                📅 Joined {{ formatDate(user.joinedDate) }}
              </span>
            </div>
            
            <!-- Stats -->
            <div class="flex gap-6 mt-4">
              <div>
                <span class="font-bold">{{ formatNumber(user.followers) }}</span>
                <span class="text-base-content/60 ml-1">Followers</span>
              </div>
              <div>
                <span class="font-bold">{{ formatNumber(user.following) }}</span>
                <span class="text-base-content/60 ml-1">Following</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Profile Tabs Navigation -->
    <ProfileTabs :userId="userId" class="mb-6" />
    
    <!-- Nested Route Content -->
    <!-- TODO: Add RouterView for child routes to render -->
    <RouterView :user="user" />
  </div>
</template>
```

---

### Step 11: Create Overview Tab

```vue
<!-- src/pages/profile/OverviewTab.vue -->
<script setup lang="ts">
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import { getUserPosts, getUserActivity, formatRelativeTime, type User } from '../../data/users'

const props = defineProps<{
  user: User
}>()

const route = useRoute()
const userId = computed(() => route.params.userId as string)

const recentPosts = computed(() => getUserPosts(userId.value).slice(0, 2))
const recentActivity = computed(() => getUserActivity(userId.value).slice(0, 3))
</script>

<template>
  <div class="grid md:grid-cols-2 gap-6">
    <!-- Recent Posts -->
    <div class="card bg-base-200">
      <div class="card-body">
        <div class="flex justify-between items-center mb-4">
          <h2 class="card-title">Recent Posts</h2>
          <RouterLink 
            :to="{ name: 'profile-posts', params: { userId } }"
            class="btn btn-ghost btn-sm"
          >
            View All →
          </RouterLink>
        </div>
        
        <div v-if="recentPosts.length > 0" class="space-y-4">
          <div 
            v-for="post in recentPosts" 
            :key="post.id"
            class="p-4 bg-base-100 rounded-lg"
          >
            <p class="mb-2">{{ post.content }}</p>
            <div class="flex gap-4 text-sm text-base-content/60">
              <span>❤️ {{ post.likes }}</span>
              <span>💬 {{ post.comments }}</span>
              <span>{{ formatRelativeTime(post.createdAt) }}</span>
            </div>
          </div>
        </div>
        
        <p v-else class="text-base-content/60">No posts yet.</p>
      </div>
    </div>
    
    <!-- Recent Activity -->
    <div class="card bg-base-200">
      <div class="card-body">
        <div class="flex justify-between items-center mb-4">
          <h2 class="card-title">Recent Activity</h2>
          <RouterLink 
            :to="{ name: 'profile-activity', params: { userId } }"
            class="btn btn-ghost btn-sm"
          >
            View All →
          </RouterLink>
        </div>
        
        <div v-if="recentActivity.length > 0" class="space-y-3">
          <div 
            v-for="activity in recentActivity" 
            :key="activity.id"
            class="flex items-center gap-3 p-3 bg-base-100 rounded-lg"
          >
            <span class="text-2xl">
              {{ activity.type === 'follow' ? '👤' : 
                 activity.type === 'like' ? '❤️' : 
                 activity.type === 'post' ? '📝' : '💬' }}
            </span>
            <div class="flex-1">
              <span class="font-medium">
                {{ activity.type === 'follow' ? 'Followed someone' :
                   activity.type === 'like' ? 'Liked a post' :
                   activity.type === 'post' ? 'Published a post' : 'Commented' }}
              </span>
              <p class="text-sm text-base-content/60">
                {{ formatRelativeTime(activity.createdAt) }}
              </p>
            </div>
          </div>
        </div>
        
        <p v-else class="text-base-content/60">No recent activity.</p>
      </div>
    </div>
    
    <!-- Stats Card -->
    <div class="card bg-base-200 md:col-span-2">
      <div class="card-body">
        <h2 class="card-title mb-4">Profile Stats</h2>
        <div class="stats stats-vertical sm:stats-horizontal shadow w-full">
          <div class="stat">
            <div class="stat-title">Posts</div>
            <div class="stat-value text-primary">{{ getUserPosts(userId).length }}</div>
          </div>
          <div class="stat">
            <div class="stat-title">Total Likes</div>
            <div class="stat-value text-secondary">
              {{ getUserPosts(userId).reduce((sum, p) => sum + p.likes, 0).toLocaleString() }}
            </div>
          </div>
          <div class="stat">
            <div class="stat-title">Comments Received</div>
            <div class="stat-value text-accent">
              {{ getUserPosts(userId).reduce((sum, p) => sum + p.comments, 0) }}
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
```

---

### Step 12: Create Posts Tab

```vue
<!-- src/pages/profile/PostsTab.vue -->
<script setup lang="ts">
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import { getUserPosts, formatRelativeTime, type User } from '../../data/users'

defineProps<{
  user: User
}>()

const route = useRoute()
const userId = computed(() => route.params.userId as string)
const posts = computed(() => getUserPosts(userId.value))
</script>

<template>
  <div class="card bg-base-200">
    <div class="card-body">
      <h2 class="card-title mb-6">All Posts ({{ posts.length }})</h2>
      
      <div v-if="posts.length > 0" class="space-y-4">
        <article 
          v-for="post in posts" 
          :key="post.id"
          class="p-6 bg-base-100 rounded-xl shadow"
        >
          <!-- Post Header -->
          <div class="flex items-center gap-3 mb-4">
            <img 
              :src="user.avatar" 
              :alt="user.displayName"
              class="w-10 h-10 rounded-full"
            />
            <div>
              <p class="font-bold">{{ user.displayName }}</p>
              <p class="text-sm text-base-content/60">
                @{{ user.username }} · {{ formatRelativeTime(post.createdAt) }}
              </p>
            </div>
          </div>
          
          <!-- Post Content -->
          <p class="text-lg mb-4">{{ post.content }}</p>
          
          <!-- Post Actions -->
          <div class="flex gap-6 text-base-content/70">
            <button class="flex items-center gap-2 hover:text-error transition-colors">
              ❤️ <span>{{ post.likes.toLocaleString() }}</span>
            </button>
            <button class="flex items-center gap-2 hover:text-primary transition-colors">
              💬 <span>{{ post.comments }}</span>
            </button>
            <button class="flex items-center gap-2 hover:text-success transition-colors">
              🔄 <span>Share</span>
            </button>
          </div>
        </article>
      </div>
      
      <!-- Empty State -->
      <div v-else class="text-center py-12">
        <div class="text-6xl mb-4">📝</div>
        <h3 class="text-xl font-bold mb-2">No posts yet</h3>
        <p class="text-base-content/60">
          {{ user.displayName }} hasn't published any posts.
        </p>
      </div>
    </div>
  </div>
</template>
```

---

### Step 13: Create Activity Tab

```vue
<!-- src/pages/profile/ActivityTab.vue -->
<script setup lang="ts">
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import { getUserActivity, getUserById, formatRelativeTime, type User } from '../../data/users'

defineProps<{
  user: User
}>()

const route = useRoute()
const userId = computed(() => route.params.userId as string)
const activities = computed(() => getUserActivity(userId.value))

function getActivityIcon(type: string): string {
  switch (type) {
    case 'follow': return '👤'
    case 'like': return '❤️'
    case 'post': return '📝'
    case 'comment': return '💬'
    default: return '📌'
  }
}

function getActivityText(type: string, targetUser?: string): string {
  switch (type) {
    case 'follow': 
      const user = targetUser ? getUserById(targetUser) : null
      return user ? `Followed ${user.displayName}` : 'Followed someone'
    case 'like': return 'Liked a post'
    case 'post': return 'Published a new post'
    case 'comment': return 'Commented on a post'
    default: return 'Did something'
  }
}
</script>

<template>
  <div class="card bg-base-200">
    <div class="card-body">
      <h2 class="card-title mb-6">Activity Feed</h2>
      
      <div v-if="activities.length > 0" class="relative">
        <!-- Timeline Line -->
        <div class="absolute left-4 top-0 bottom-0 w-0.5 bg-base-300"></div>
        
        <!-- Activity Items -->
        <div class="space-y-6">
          <div 
            v-for="activity in activities" 
            :key="activity.id"
            class="flex gap-4 relative"
          >
            <!-- Icon -->
            <div class="w-8 h-8 rounded-full bg-base-100 flex items-center justify-center z-10 shadow">
              {{ getActivityIcon(activity.type) }}
            </div>
            
            <!-- Content -->
            <div class="flex-1 pb-6">
              <div class="p-4 bg-base-100 rounded-lg shadow-sm">
                <p class="font-medium">
                  {{ getActivityText(activity.type, activity.targetUser) }}
                </p>
                <p class="text-sm text-base-content/60 mt-1">
                  {{ formatRelativeTime(activity.createdAt) }}
                </p>
                
                <!-- Show target user card for follows -->
                <div 
                  v-if="activity.type === 'follow' && activity.targetUser"
                  class="mt-3 p-3 bg-base-200 rounded-lg"
                >
                  <RouterLink 
                    :to="{ name: 'profile-overview', params: { userId: activity.targetUser } }"
                    class="flex items-center gap-3 hover:opacity-80"
                  >
                    <img 
                      :src="getUserById(activity.targetUser)?.avatar" 
                      class="w-10 h-10 rounded-full"
                    />
                    <div>
                      <p class="font-medium">{{ getUserById(activity.targetUser)?.displayName }}</p>
                      <p class="text-sm text-base-content/60">
                        @{{ getUserById(activity.targetUser)?.username }}
                      </p>
                    </div>
                  </RouterLink>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      
      <!-- Empty State -->
      <div v-else class="text-center py-12">
        <div class="text-6xl mb-4">📊</div>
        <h3 class="text-xl font-bold mb-2">No activity yet</h3>
        <p class="text-base-content/60">
          Activity will appear here once {{ user.displayName }} starts interacting.
        </p>
      </div>
    </div>
  </div>
</template>
```

---

### Step 14: Create Settings Tab

```vue
<!-- src/pages/profile/SettingsTab.vue -->
<script setup lang="ts">
import { ref, computed } from 'vue'
import { useRoute } from 'vue-router'
import { onBeforeRouteLeave } from 'vue-router'
import { type User } from '../../data/users'

const props = defineProps<{
  user: User
}>()

const route = useRoute()

// Form state - copy from props to allow editing
const form = ref({
  displayName: props.user.displayName,
  bio: props.user.bio,
  location: props.user.location,
  website: props.user.website
})

const hasChanges = ref(false)
const saveSuccess = ref(false)

function handleInputChange() {
  hasChanges.value = true
  saveSuccess.value = false
}

function handleSave() {
  // In real app, call API to save
  console.log('Saving:', form.value)
  hasChanges.value = false
  saveSuccess.value = true
  
  setTimeout(() => {
    saveSuccess.value = false
  }, 3000)
}

function handleReset() {
  form.value = {
    displayName: props.user.displayName,
    bio: props.user.bio,
    location: props.user.location,
    website: props.user.website
  }
  hasChanges.value = false
}

// TODO: Implement route leave guard to warn about unsaved changes
onBeforeRouteLeave((to, from) => {
  if (hasChanges.value) {
    const answer = window.confirm(
      'You have unsaved changes. Do you really want to leave?'
    )
    if (!answer) return false
  }
})
</script>

<template>
  <div class="card bg-base-200">
    <div class="card-body">
      <h2 class="card-title mb-6">Profile Settings</h2>
      
      <!-- Success Alert -->
      <div v-if="saveSuccess" class="alert alert-success mb-6">
        <span>✓ Profile updated successfully!</span>
      </div>
      
      <!-- Unsaved Changes Warning -->
      <div v-if="hasChanges" class="alert alert-warning mb-6">
        <span>⚠️ You have unsaved changes</span>
      </div>
      
      <form @submit.prevent="handleSave" class="space-y-6 max-w-lg">
        <!-- Avatar -->
        <div class="form-control">
          <label class="label">
            <span class="label-text">Profile Picture</span>
          </label>
          <div class="flex items-center gap-4">
            <img 
              :src="user.avatar" 
              :alt="user.displayName"
              class="w-20 h-20 rounded-full"
            />
            <button type="button" class="btn btn-outline btn-sm">
              Change Photo
            </button>
          </div>
        </div>
        
        <!-- Display Name -->
        <div class="form-control">
          <label class="label">
            <span class="label-text">Display Name</span>
          </label>
          <input 
            v-model="form.displayName"
            @input="handleInputChange"
            type="text" 
            class="input input-bordered"
            placeholder="Your display name"
          />
        </div>
        
        <!-- Bio -->
        <div class="form-control">
          <label class="label">
            <span class="label-text">Bio</span>
          </label>
          <textarea 
            v-model="form.bio"
            @input="handleInputChange"
            class="textarea textarea-bordered h-24"
            placeholder="Tell us about yourself"
          ></textarea>
          <label class="label">
            <span class="label-text-alt">{{ form.bio.length }}/160 characters</span>
          </label>
        </div>
        
        <!-- Location -->
        <div class="form-control">
          <label class="label">
            <span class="label-text">Location</span>
          </label>
          <input 
            v-model="form.location"
            @input="handleInputChange"
            type="text" 
            class="input input-bordered"
            placeholder="Where are you based?"
          />
        </div>
        
        <!-- Website -->
        <div class="form-control">
          <label class="label">
            <span class="label-text">Website</span>
          </label>
          <input 
            v-model="form.website"
            @input="handleInputChange"
            type="url" 
            class="input input-bordered"
            placeholder="https://yourwebsite.com"
          />
        </div>
        
        <!-- Actions -->
        <div class="flex gap-4 pt-4">
          <button 
            type="submit" 
            class="btn btn-primary"
            :disabled="!hasChanges"
          >
            Save Changes
          </button>
          <button 
            type="button" 
            class="btn btn-ghost"
            :disabled="!hasChanges"
            @click="handleReset"
          >
            Reset
          </button>
        </div>
      </form>
      
      <!-- Danger Zone -->
      <div class="divider mt-12"></div>
      
      <div class="p-4 border border-error rounded-lg">
        <h3 class="font-bold text-error mb-2">Danger Zone</h3>
        <p class="text-sm text-base-content/70 mb-4">
          Once you delete your account, there is no going back. Please be certain.
        </p>
        <button class="btn btn-error btn-outline btn-sm">
          Delete Account
        </button>
      </div>
    </div>
  </div>
</template>
```

---

### Step 15: Create Home Page (User Discovery)

```vue
<!-- src/pages/HomePage.vue -->
<script setup lang="ts">
import { ref, computed } from 'vue'
import { users } from '../data/users'
import { useCurrentUser } from '../composables/useCurrentUser'
import UserCard from '../components/UserCard.vue'

const { currentUser, switchUser } = useCurrentUser()

const searchQuery = ref('')

const filteredUsers = computed(() => {
  if (!searchQuery.value) return users
  
  const query = searchQuery.value.toLowerCase()
  return users.filter(user => 
    user.displayName.toLowerCase().includes(query) ||
    user.username.toLowerCase().includes(query) ||
    user.bio.toLowerCase().includes(query)
  )
})
</script>

<template>
  <div class="container mx-auto px-4 py-8">
    <!-- Header -->
    <div class="text-center mb-12">
      <h1 class="text-4xl font-bold mb-4">Discover People</h1>
      <p class="text-base-content/70 max-w-xl mx-auto">
        Find and connect with developers, designers, and creators from around the world.
      </p>
    </div>
    
    <!-- Current User Switcher (Demo) -->
    <div class="card bg-base-200 mb-8">
      <div class="card-body">
        <h2 class="card-title text-sm">Demo: Switch Current User</h2>
        <div class="flex flex-wrap gap-2">
          <button 
            v-for="user in users" 
            :key="user.id"
            @click="switchUser(user.id)"
            :class="[
              'btn btn-sm',
              currentUser?.id === user.id ? 'btn-primary' : 'btn-ghost'
            ]"
          >
            <img :src="user.avatar" class="w-5 h-5 rounded-full" />
            {{ user.username }}
          </button>
        </div>
        <p class="text-xs text-base-content/60 mt-2">
          Current user: <strong>{{ currentUser?.displayName }}</strong> — 
          Settings tab only visible on your own profile.
        </p>
      </div>
    </div>
    
    <!-- Search -->
    <div class="form-control mb-8">
      <input 
        v-model="searchQuery"
        type="text" 
        placeholder="Search by name, username, or bio..." 
        class="input input-bordered input-lg w-full"
      />
    </div>
    
    <!-- User Grid -->
    <div class="grid md:grid-cols-2 lg:grid-cols-3 gap-6">
      <UserCard 
        v-for="user in filteredUsers" 
        :key="user.id"
        :user="user"
      />
    </div>
    
    <!-- Empty State -->
    <div v-if="filteredUsers.length === 0" class="text-center py-16">
      <div class="text-6xl mb-4">🔍</div>
      <h3 class="text-xl font-bold mb-2">No users found</h3>
      <p class="text-base-content/60">
        Try a different search term.
      </p>
    </div>
  </div>
</template>
```

---

### Step 16: Create Not Found Page

```vue
<!-- src/pages/NotFoundPage.vue -->
<script setup lang="ts">
import { useRoute } from 'vue-router'

const route = useRoute()
</script>

<template>
  <div class="min-h-[70vh] flex items-center justify-center px-4">
    <div class="text-center">
      <h1 class="text-9xl font-bold text-primary">404</h1>
      <h2 class="text-3xl font-bold mt-4 mb-2">Page Not Found</h2>
      <p class="text-base-content/70 mb-8">
        The page <code class="bg-base-200 px-2 py-1 rounded">{{ route.fullPath }}</code> doesn't exist.
      </p>
      <div class="flex gap-4 justify-center">
        <RouterLink to="/" class="btn btn-primary">
          Go Home
        </RouterLink>
      </div>
    </div>
  </div>
</template>
```

---

## Complete Solution

All files above form the complete solution.

---

## How to Test

1. **Dynamic routes:** Click different user cards, verify URL changes to `/profile/1`, `/profile/2`, etc.
2. **Nested routes:** Click tabs (Overview, Posts, Activity), verify URL updates and content changes
3. **Param watching:** Navigate from one profile to another (use user cards in activity), verify data updates
4. **Settings protection:** Only visible on your own profile (use demo switcher to test)
5. **Route leave guard:** Make changes in Settings, try to navigate away, verify confirmation dialog
6. **Search:** Type in search on home page, verify filtering works
7. **404 handling:** Visit `/profile/999`, verify redirect to 404

---

## Stretch Goals

1. **Follow system:** Implement real follow/unfollow with state persistence
2. **Infinite scroll:** Add pagination to posts and activity feeds
3. **Profile editing:** Actually save profile changes to localStorage
4. **Direct messages:** Add a messages tab with mock conversations
5. **Notifications:** Add a notifications system with real-time updates

---

## Self-Assessment Checklist

- [ ] Dynamic route parameters are accessed via `useRoute().params`
- [ ] Nested routes render inside parent layout's `<RouterView>`
- [ ] `watch()` is used to react to param changes (component reuse)
- [ ] Settings tab is only accessible on own profile
- [ ] `onBeforeRouteLeave` prevents accidental navigation with unsaved changes
- [ ] Tab navigation uses named routes with params
- [ ] Active tab is visually highlighted
- [ ] User not found redirects to 404 page
