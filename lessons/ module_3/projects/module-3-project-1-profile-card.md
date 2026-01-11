# Module 3 Project 1: Profile Card Component

**Scenario:** A team page displays employee cards with avatar, name, role, and social links.

**Key Patterns:** Tailwind utilities, DaisyUI card, responsive design

---

## ProfileCard.vue

```vue
<script setup lang="ts">
interface SocialLinks {
  twitter?: string
  linkedin?: string
  github?: string
}

interface Props {
  name: string
  role: string
  department: string
  avatarUrl: string
  email: string
  socialLinks?: SocialLinks
}

const props = withDefaults(defineProps<Props>(), {
  socialLinks: () => ({})
})
</script>

<template>
  <div class="card bg-base-100 shadow-xl w-full sm:w-72 transition-all duration-300 hover:shadow-2xl hover:-translate-y-2">
    <figure class="pt-6 px-6">
      <div class="avatar">
        <div class="w-24 rounded-full ring ring-primary ring-offset-base-100 ring-offset-2">
          <img :src="props.avatarUrl" :alt="props.name" />
        </div>
      </div>
    </figure>
    
    <div class="card-body items-center text-center pt-4">
      <h2 class="card-title">{{ props.name }}</h2>
      <p class="text-base-content/70 text-sm">{{ props.role }}</p>
      <div class="badge badge-primary badge-outline">{{ props.department }}</div>
      
      <a :href="`mailto:${props.email}`" class="text-sm text-primary hover:underline mt-2">
        {{ props.email }}
      </a>
      
      <div class="divider my-2"></div>
      
      <div class="flex gap-3">
        <a 
          v-if="props.socialLinks?.twitter"
          :href="props.socialLinks.twitter"
          target="_blank"
          class="btn btn-ghost btn-circle btn-sm hover:btn-primary"
        >
          <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
            <path d="M18.244 2.25h3.308l-7.227 8.26 8.502 11.24H16.17l-5.214-6.817L4.99 21.75H1.68l7.73-8.835L1.254 2.25H8.08l4.713 6.231zm-1.161 17.52h1.833L7.084 4.126H5.117z"/>
          </svg>
        </a>
        
        <a 
          v-if="props.socialLinks?.linkedin"
          :href="props.socialLinks.linkedin"
          target="_blank"
          class="btn btn-ghost btn-circle btn-sm hover:btn-primary"
        >
          <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
            <path d="M20.447 20.452h-3.554v-5.569c0-1.328-.027-3.037-1.852-3.037-1.853 0-2.136 1.445-2.136 2.939v5.667H9.351V9h3.414v1.561h.046c.477-.9 1.637-1.85 3.37-1.85 3.601 0 4.267 2.37 4.267 5.455v6.286zM5.337 7.433c-1.144 0-2.063-.926-2.063-2.065 0-1.138.92-2.063 2.063-2.063 1.14 0 2.064.925 2.064 2.063 0 1.139-.925 2.065-2.064 2.065zm1.782 13.019H3.555V9h3.564v11.452zM22.225 0H1.771C.792 0 0 .774 0 1.729v20.542C0 23.227.792 24 1.771 24h20.451C23.2 24 24 23.227 24 22.271V1.729C24 .774 23.2 0 22.222 0h.003z"/>
          </svg>
        </a>
        
        <a 
          v-if="props.socialLinks?.github"
          :href="props.socialLinks.github"
          target="_blank"
          class="btn btn-ghost btn-circle btn-sm hover:btn-primary"
        >
          <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
            <path d="M12 0c-6.626 0-12 5.373-12 12 0 5.302 3.438 9.8 8.207 11.387.599.111.793-.261.793-.577v-2.234c-3.338.726-4.033-1.416-4.033-1.416-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.839 1.237 1.839 1.237 1.07 1.834 2.807 1.304 3.492.997.107-.775.418-1.305.762-1.604-2.665-.305-5.467-1.334-5.467-5.931 0-1.311.469-2.381 1.236-3.221-.124-.303-.535-1.524.117-3.176 0 0 1.008-.322 3.301 1.23.957-.266 1.983-.399 3.003-.404 1.02.005 2.047.138 3.006.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.874.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.609-2.807 5.624-5.479 5.921.43.372.823 1.102.823 2.222v3.293c0 .319.192.694.801.576 4.765-1.589 8.199-6.086 8.199-11.386 0-6.627-5.373-12-12-12z"/>
          </svg>
        </a>
      </div>
      
      <div class="card-actions w-full mt-4">
        <button class="btn btn-primary btn-block">View Profile</button>
      </div>
    </div>
  </div>
</template>
```

---

## TeamPage.vue

```vue
<script setup lang="ts">
import ProfileCard from './ProfileCard.vue'

const team = [
  {
    name: 'Sarah Chen',
    role: 'Senior Frontend Engineer',
    department: 'Engineering',
    avatarUrl: 'https://i.pravatar.cc/150?img=1',
    email: 'sarah.chen@company.com',
    socialLinks: {
      twitter: 'https://twitter.com/sarahchen',
      linkedin: 'https://linkedin.com/in/sarahchen',
      github: 'https://github.com/sarahchen'
    }
  },
  {
    name: 'Marcus Johnson',
    role: 'Product Designer',
    department: 'Design',
    avatarUrl: 'https://i.pravatar.cc/150?img=3',
    email: 'marcus.j@company.com',
    socialLinks: {
      linkedin: 'https://linkedin.com/in/marcusj'
    }
  },
  {
    name: 'Emily Rodriguez',
    role: 'Engineering Manager',
    department: 'Engineering',
    avatarUrl: 'https://i.pravatar.cc/150?img=5',
    email: 'emily.r@company.com',
    socialLinks: {
      github: 'https://github.com/emilyr',
      linkedin: 'https://linkedin.com/in/emilyr'
    }
  },
  {
    name: 'David Kim',
    role: 'Backend Engineer',
    department: 'Engineering',
    avatarUrl: 'https://i.pravatar.cc/150?img=8',
    email: 'david.kim@company.com',
    socialLinks: {
      github: 'https://github.com/davidkim'
    }
  }
]
</script>

<template>
  <div class="min-h-screen bg-base-200 p-6 md:p-12">
    <header class="text-center mb-10">
      <h1 class="text-4xl font-bold text-base-content">Meet Our Team</h1>
      <p class="text-base-content/60 mt-2 max-w-xl mx-auto">
        The talented people behind our success
      </p>
    </header>
    
    <main class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6 justify-items-center max-w-7xl mx-auto">
      <ProfileCard
        v-for="member in team"
        :key="member.email"
        v-bind="member"
      />
    </main>
  </div>
</template>
```

---

## Skills Practiced

- DaisyUI `card`, `avatar`, `badge`, `btn`, `divider` components
- Tailwind spacing (`p-6`, `gap-3`, `mt-4`), sizing (`w-24`, `w-full`)
- Flexbox and grid layout utilities
- Responsive breakpoints (`sm:`, `lg:`, `xl:`)
- Hover transitions (`hover:shadow-2xl`, `hover:-translate-y-2`)
- Conditional rendering with `v-if` for optional social links
- TypeScript props with `withDefaults`
