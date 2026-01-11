# Module 3 Project 2: Pricing Table

**Scenario:** A SaaS landing page shows 3-tier pricing with highlighted "popular" option.

**Key Patterns:** Grid layout, DaisyUI badges, conditional styling

---

## PricingCard.vue

```vue
<script setup lang="ts">
interface Feature {
  text: string
  included: boolean
}

interface Props {
  name: string
  price: number
  period?: string
  description: string
  features: Feature[]
  isPopular?: boolean
  ctaText?: string
}

const props = withDefaults(defineProps<Props>(), {
  period: 'month',
  isPopular: false,
  ctaText: 'Get Started'
})

const emit = defineEmits<{
  select: [planName: string]
}>()
</script>

<template>
  <div 
    class="card bg-base-100 shadow-xl w-full transition-all duration-300 hover:shadow-2xl"
    :class="{ 
      'border-2 border-primary scale-105 z-10': props.isPopular,
      'hover:-translate-y-1': !props.isPopular
    }"
  >
    <div v-if="props.isPopular" class="absolute -top-4 left-1/2 -translate-x-1/2">
      <div class="badge badge-primary badge-lg gap-1">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" viewBox="0 0 20 20" fill="currentColor">
          <path d="M9.049 2.927c.3-.921 1.603-.921 1.902 0l1.07 3.292a1 1 0 00.95.69h3.462c.969 0 1.371 1.24.588 1.81l-2.8 2.034a1 1 0 00-.364 1.118l1.07 3.292c.3.921-.755 1.688-1.54 1.118l-2.8-2.034a1 1 0 00-1.175 0l-2.8 2.034c-.784.57-1.838-.197-1.539-1.118l1.07-3.292a1 1 0 00-.364-1.118L2.98 8.72c-.783-.57-.38-1.81.588-1.81h3.461a1 1 0 00.951-.69l1.07-3.292z" />
        </svg>
        Most Popular
      </div>
    </div>
    
    <div class="card-body p-6 md:p-8">
      <h3 class="text-xl font-semibold text-base-content">{{ props.name }}</h3>
      <p class="text-base-content/60 text-sm">{{ props.description }}</p>
      
      <div class="my-6">
        <span class="text-5xl font-extrabold text-base-content">${{ props.price }}</span>
        <span class="text-base-content/60">/{{ props.period }}</span>
      </div>
      
      <ul class="space-y-3 mb-6 flex-grow">
        <li 
          v-for="(feature, index) in props.features" 
          :key="index"
          class="flex items-center gap-3"
        >
          <svg 
            v-if="feature.included"
            class="w-5 h-5 text-success flex-shrink-0" 
            fill="none" 
            stroke="currentColor" 
            viewBox="0 0 24 24"
          >
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
          </svg>
          <svg 
            v-else
            class="w-5 h-5 text-base-content/30 flex-shrink-0" 
            fill="none" 
            stroke="currentColor" 
            viewBox="0 0 24 24"
          >
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
          </svg>
          <span :class="feature.included ? 'text-base-content' : 'text-base-content/40'">
            {{ feature.text }}
          </span>
        </li>
      </ul>
      
      <button 
        class="btn btn-block"
        :class="props.isPopular ? 'btn-primary' : 'btn-outline btn-primary'"
        @click="emit('select', props.name)"
      >
        {{ props.ctaText }}
      </button>
    </div>
  </div>
</template>
```

---

## PricingTable.vue

```vue
<script setup lang="ts">
import { ref } from 'vue'
import PricingCard from './PricingCard.vue'

const billingPeriod = ref<'month' | 'year'>('month')

const plans = [
  {
    name: 'Starter',
    monthlyPrice: 9,
    yearlyPrice: 90,
    description: 'Perfect for individuals and small projects',
    features: [
      { text: '5 Projects', included: true },
      { text: '10GB Storage', included: true },
      { text: 'Basic Analytics', included: true },
      { text: 'Email Support', included: true },
      { text: 'API Access', included: false },
      { text: 'Custom Domain', included: false },
      { text: 'Priority Support', included: false }
    ]
  },
  {
    name: 'Professional',
    monthlyPrice: 29,
    yearlyPrice: 290,
    description: 'Best for growing teams and businesses',
    isPopular: true,
    features: [
      { text: 'Unlimited Projects', included: true },
      { text: '100GB Storage', included: true },
      { text: 'Advanced Analytics', included: true },
      { text: 'Priority Email Support', included: true },
      { text: 'API Access', included: true },
      { text: 'Custom Domain', included: true },
      { text: 'Priority Support', included: false }
    ]
  },
  {
    name: 'Enterprise',
    monthlyPrice: 99,
    yearlyPrice: 990,
    description: 'For large organizations with advanced needs',
    ctaText: 'Contact Sales',
    features: [
      { text: 'Unlimited Projects', included: true },
      { text: 'Unlimited Storage', included: true },
      { text: 'Custom Analytics', included: true },
      { text: '24/7 Phone Support', included: true },
      { text: 'API Access', included: true },
      { text: 'Custom Domain', included: true },
      { text: 'Dedicated Account Manager', included: true }
    ]
  }
]

const getPrice = (plan: typeof plans[0]) => {
  return billingPeriod.value === 'month' ? plan.monthlyPrice : plan.yearlyPrice
}

const handleSelect = (planName: string) => {
  console.log(`Selected plan: ${planName}`)
}
</script>

<template>
  <section class="min-h-screen bg-base-200 py-12 px-4 md:py-20">
    <div class="max-w-6xl mx-auto">
      <header class="text-center mb-12">
        <h2 class="text-4xl md:text-5xl font-bold text-base-content mb-4">
          Simple, Transparent Pricing
        </h2>
        <p class="text-base-content/60 max-w-2xl mx-auto mb-8">
          Choose the plan that works best for you. All plans include a 14-day free trial.
        </p>
        
        <div class="flex items-center justify-center gap-4">
          <span :class="billingPeriod === 'month' ? 'text-base-content font-semibold' : 'text-base-content/60'">
            Monthly
          </span>
          <input 
            type="checkbox" 
            class="toggle toggle-primary"
            :checked="billingPeriod === 'year'"
            @change="billingPeriod = billingPeriod === 'month' ? 'year' : 'month'"
          />
          <span :class="billingPeriod === 'year' ? 'text-base-content font-semibold' : 'text-base-content/60'">
            Yearly
            <span class="badge badge-success badge-sm ml-1">Save 20%</span>
          </span>
        </div>
      </header>
      
      <div class="grid grid-cols-1 md:grid-cols-3 gap-6 md:gap-4 items-start pt-6">
        <PricingCard
          v-for="plan in plans"
          :key="plan.name"
          :name="plan.name"
          :price="getPrice(plan)"
          :period="billingPeriod"
          :description="plan.description"
          :features="plan.features"
          :is-popular="plan.isPopular"
          :cta-text="plan.ctaText"
          @select="handleSelect"
        />
      </div>
      
      <footer class="text-center mt-12 text-base-content/60 text-sm">
        <p>All prices in USD. Taxes may apply. Cancel anytime.</p>
      </footer>
    </div>
  </section>
</template>
```

---

## Skills Practiced

- Conditional styling with `:class` bindings
- DaisyUI `badge`, `toggle`, `btn` components
- CSS Grid for responsive 3-column layout
- Tailwind transforms (`scale-105`, `-translate-y-1`, `-translate-x-1/2`)
- Z-index management for overlapping elements
- Dynamic pricing based on billing period toggle
- TypeScript interfaces for feature lists
- Event emission from child to parent
