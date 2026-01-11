# Project 2: Star Rating Component
**Module:** Module 2: Component Communication  
**Patterns Practiced:** Props for initial value, emit on selection, visual feedback, computed properties, v-model pattern
---
## Scenario
An e-commerce product page needs a reusable rating component that shows the current rating and allows users to select a new score.
## What You'll Build
A `StarRating` component with hover preview, click selection, and optional read-only mode. A `ProductCard` parent demonstrates usage.
---
## Component Structure
```
src/
├── components/
│   ├── StarRating.vue
│   └── ProductCard.vue
└── App.vue
```
---
## Step 1: StarRating Component
```vue
<!-- components/StarRating.vue -->
<script setup lang="ts">
import { ref, computed } from 'vue'
interface Props {
  modelValue: number
  max?: number
  readonly?: boolean
  size?: 'sm' | 'md' | 'lg'
}
const props = withDefaults(defineProps<Props>(), {
  max: 5,
  readonly: false,
  size: 'md'
})
const emit = defineEmits<{
  'update:modelValue': [rating: number]
  rate: [rating: number]
}>()
const hoverIndex = ref<number | null>(null)
const displayValue = computed(() => {
  if (!props.readonly && hoverIndex.value !== null) {
    return hoverIndex.value
  }
  return props.modelValue
})
const sizeClass = computed(() => ({
  sm: 'w-4 h-4',
  md: 'w-6 h-6',
  lg: 'w-8 h-8'
})[props.size])
function handleClick(index: number) {
  if (props.readonly) return
  emit('update:modelValue', index)
  emit('rate', index)
}
function handleMouseEnter(index: number) {
  if (props.readonly) return
  hoverIndex.value = index
}
function handleMouseLeave() {
  hoverIndex.value = null
}
</script>
<template>
  <div 
    class="inline-flex items-center gap-0.5"
    @mouseleave="handleMouseLeave"
  >
    <button
      v-for="i in max"
      :key="i"
      type="button"
      :disabled="readonly"
      class="transition-transform"
      :class="[
        sizeClass,
        readonly ? 'cursor-default' : 'cursor-pointer hover:scale-110'
      ]"
      @click="handleClick(i)"
      @mouseenter="handleMouseEnter(i)"
    >
      <svg
        viewBox="0 0 24 24"
        :class="i <= displayValue ? 'text-yellow-400' : 'text-gray-300'"
        fill="currentColor"
      >
        <path d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"/>
      </svg>
    </button>
    <span 
      v-if="!readonly" 
      class="ml-2 text-sm text-base-content/60 min-w-[3ch]"
    >
      {{ displayValue }}/{{ max }}
    </span>
  </div>
</template>
```
---
## Step 2: ProductCard Component
```vue
<!-- components/ProductCard.vue -->
<script setup lang="ts">
import { computed } from 'vue'
import StarRating from './StarRating.vue'
interface Product {
  id: number
  name: string
  price: number
  image: string
  rating: number
  reviewCount: number
}
const props = defineProps<{
  product: Product
}>()
const emit = defineEmits<{
  rate: [productId: number, rating: number]
  addToCart: [productId: number]
}>()
const formattedPrice = computed(() => {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD'
  }).format(props.product.price)
})
function handleRate(rating: number) {
  emit('rate', props.product.id, rating)
}
function handleAddToCart() {
  emit('addToCart', props.product.id)
}
</script>
<template>
  <div class="card bg-base-100 shadow-xl">
    <figure class="px-4 pt-4">
      <img 
        :src="product.image" 
        :alt="product.name"
        class="rounded-xl h-48 w-full object-cover"
      />
    </figure>
    <div class="card-body">
      <h2 class="card-title">{{ product.name }}</h2>
      
      <div class="flex items-center gap-2">
        <StarRating
          :model-value="product.rating"
          size="sm"
          readonly
        />
        <span class="text-sm text-base-content/60">
          ({{ product.reviewCount }} reviews)
        </span>
      </div>
      
      <p class="text-xl font-bold text-primary">{{ formattedPrice }}</p>
      
      <div class="divider my-2">Rate this product</div>
      
      <div class="flex justify-center">
        <StarRating
          :model-value="0"
          size="lg"
          @rate="handleRate"
        />
      </div>
      
      <div class="card-actions justify-end mt-4">
        <button class="btn btn-primary" @click="handleAddToCart">
          Add to Cart
        </button>
      </div>
    </div>
  </div>
</template>
```
---
## Step 3: App Integration
```vue
<!-- App.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import ProductCard from './components/ProductCard.vue'
interface Product {
  id: number
  name: string
  price: number
  image: string
  rating: number
  reviewCount: number
}
const products = ref<Product[]>([
  {
    id: 1,
    name: 'Wireless Headphones',
    price: 149.99,
    image: 'https://picsum.photos/seed/headphones/400/300',
    rating: 4,
    reviewCount: 128
  },
  {
    id: 2,
    name: 'Mechanical Keyboard',
    price: 199.99,
    image: 'https://picsum.photos/seed/keyboard/400/300',
    rating: 5,
    reviewCount: 256
  },
  {
    id: 3,
    name: 'USB-C Hub',
    price: 79.99,
    image: 'https://picsum.photos/seed/hub/400/300',
    rating: 3,
    reviewCount: 64
  }
])
const userRatings = ref<Record<number, number>>({})
function handleRate(productId: number, rating: number) {
  userRatings.value[productId] = rating
  console.log(`Product ${productId} rated: ${rating} stars`)
}
function handleAddToCart(productId: number) {
  const product = products.value.find(p => p.id === productId)
  console.log(`Added to cart: ${product?.name}`)
}
</script>
<template>
  <div class="container mx-auto p-8">
    <h1 class="text-3xl font-bold mb-8">Featured Products</h1>
    
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      <ProductCard
        v-for="product in products"
        :key="product.id"
        :product="product"
        @rate="handleRate"
        @add-to-cart="handleAddToCart"
      />
    </div>
    
    <div v-if="Object.keys(userRatings).length > 0" class="mt-8">
      <h2 class="text-xl font-semibold mb-4">Your Ratings</h2>
      <div class="flex gap-4">
        <div 
          v-for="(rating, productId) in userRatings" 
          :key="productId"
          class="badge badge-lg badge-primary"
        >
          Product #{{ productId }}: {{ rating }}★
        </div>
      </div>
    </div>
  </div>
</template>
```
---
## Self-Assessment Checklist
- [ ] `StarRating` shows hover preview when not readonly
- [ ] `StarRating` emits `rate` event with selected value on click
- [ ] `ProductCard` passes product data via props
- [ ] `ProductCard` emits `rate` and `addToCart` with product ID
- [ ] Parent tracks user ratings separately from product data
---
## Stretch Goals
- Allow half-star ratings (0.5 increments)
- Add rating distribution display (bar chart showing 1-5 star counts)
- Implement "Was this review helpful?" voting
- Add animation when rating changes
- Support keyboard navigation (arrow keys to select)
