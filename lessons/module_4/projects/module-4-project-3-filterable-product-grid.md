# Module 4 Project 3: Filterable Product Grid

## Project Overview

| Field | Value |
|:--- |:--- |
| **Module** | Module 4: Rendering Patterns |
| **Project** | Filterable Product Grid |
| **Difficulty** | Intermediate |
| **Estimated Time** | 60-75 minutes |
| **Prerequisites** | Lessons 9.1, 9.2, 10.1, 10.2 |

**What You'll Build:** An online store product catalog with category filters, search functionality, sorting options, and graceful handling of empty results.

**Real-World Scenario:** You're building the product listing page for an e-commerce site like Amazon or Etsy. Customers need to browse products, filter by category, search by name, sort by price or rating, and see a helpful message when no products match their criteria.

---

## Patterns You'll Practice

| Pattern | Where Used |
|:--- |:--- |
| `v-for` with `:key` | Rendering product grid, filter options |
| `v-if` / `v-else` | Empty state vs. product grid |
| `v-show` | Toggle filter panel, quick view elements |
| `computed()` | Filtering, sorting, derived counts |
| Conditional classes | Selected filters, product states |
| `ref()` with arrays/objects | Products, filter state |

---

## Component Structure

```
src/
├── components/
│   ├── ProductGrid.vue        # Main container
│   ├── ProductCard.vue        # Individual product card
│   ├── FilterPanel.vue        # Sidebar filters
│   ├── SearchBar.vue          # Search input
│   ├── SortDropdown.vue       # Sort options
│   └── EmptyState.vue         # No results message
├── types/
│   └── product.ts             # TypeScript interfaces
├── data/
│   └── products.ts            # Sample product data
├── App.vue
└── main.ts
```

---

## Step-by-Step Guide

### Step 1: Project Setup

```bash
npm create vue@latest filterable-product-grid
cd filterable-product-grid
npm install -D tailwindcss postcss autoprefixer daisyui@latest
npx tailwindcss init -p
```

Configure Tailwind and DaisyUI as in previous projects.

---

### Step 2: Define TypeScript Types

Create `src/types/product.ts`:

```typescript
export type Category = 'electronics' | 'clothing' | 'books' | 'home' | 'sports'

export interface Product {
  id: number
  name: string
  description: string
  price: number
  originalPrice?: number
  category: Category
  rating: number
  reviewCount: number
  image: string
  inStock: boolean
  tags: string[]
  featured: boolean
}

export type SortOption = 'featured' | 'price-asc' | 'price-desc' | 'rating' | 'newest'

export interface FilterState {
  categories: Category[]
  priceRange: { min: number; max: number }
  inStockOnly: boolean
  minRating: number
}
```

---

### Step 3: Create Sample Product Data

Create `src/data/products.ts`:

```typescript
import type { Product } from '@/types/product'

export const products: Product[] = [
  {
    id: 1,
    name: 'Wireless Bluetooth Headphones',
    description: 'Premium noise-cancelling headphones with 30-hour battery',
    price: 149.99,
    originalPrice: 199.99,
    category: 'electronics',
    rating: 4.5,
    reviewCount: 2847,
    image: '🎧',
    inStock: true,
    tags: ['wireless', 'bluetooth', 'noise-cancelling'],
    featured: true,
  },
  {
    id: 2,
    name: 'Cotton Crew T-Shirt',
    description: 'Classic fit cotton t-shirt in multiple colors',
    price: 24.99,
    category: 'clothing',
    rating: 4.2,
    reviewCount: 1523,
    image: '👕',
    inStock: true,
    tags: ['cotton', 'casual', 'basics'],
    featured: false,
  },
  {
    id: 3,
    name: 'JavaScript: The Good Parts',
    description: 'Essential guide to JavaScript programming',
    price: 29.99,
    category: 'books',
    rating: 4.8,
    reviewCount: 892,
    image: '📘',
    inStock: true,
    tags: ['programming', 'javascript', 'web development'],
    featured: true,
  },
  {
    id: 4,
    name: 'Smart Watch Pro',
    description: 'Fitness tracking smartwatch with heart rate monitor',
    price: 299.99,
    originalPrice: 349.99,
    category: 'electronics',
    rating: 4.3,
    reviewCount: 3241,
    image: '⌚',
    inStock: false,
    tags: ['fitness', 'smartwatch', 'health'],
    featured: true,
  },
  {
    id: 5,
    name: 'Denim Slim Jeans',
    description: 'Modern slim fit stretch denim jeans',
    price: 59.99,
    category: 'clothing',
    rating: 4.0,
    reviewCount: 2156,
    image: '👖',
    inStock: true,
    tags: ['denim', 'slim fit', 'casual'],
    featured: false,
  },
  {
    id: 6,
    name: 'LED Desk Lamp',
    description: 'Adjustable LED desk lamp with USB charging port',
    price: 45.99,
    category: 'home',
    rating: 4.6,
    reviewCount: 1087,
    image: '💡',
    inStock: true,
    tags: ['led', 'desk', 'office'],
    featured: false,
  },
  {
    id: 7,
    name: 'TypeScript Handbook',
    description: 'Complete guide to TypeScript development',
    price: 39.99,
    category: 'books',
    rating: 4.9,
    reviewCount: 654,
    image: '📕',
    inStock: false,
    tags: ['programming', 'typescript', 'web development'],
    featured: true,
  },
  {
    id: 8,
    name: 'Portable Bluetooth Speaker',
    description: 'Waterproof speaker with 12-hour battery life',
    price: 79.99,
    originalPrice: 99.99,
    category: 'electronics',
    rating: 4.4,
    reviewCount: 1876,
    image: '🔊',
    inStock: true,
    tags: ['bluetooth', 'portable', 'waterproof'],
    featured: false,
  },
  {
    id: 9,
    name: 'Velvet Throw Pillow',
    description: 'Luxury velvet decorative pillow for couch or bed',
    price: 29.99,
    category: 'home',
    rating: 4.5,
    reviewCount: 743,
    image: '🛋️',
    inStock: true,
    tags: ['decor', 'velvet', 'luxury'],
    featured: false,
  },
  {
    id: 10,
    name: 'Winter Puffer Jacket',
    description: 'Warm insulated jacket for cold weather',
    price: 129.99,
    originalPrice: 179.99,
    category: 'clothing',
    rating: 4.7,
    reviewCount: 982,
    image: '🧥',
    inStock: true,
    tags: ['winter', 'warm', 'outdoor'],
    featured: true,
  },
  {
    id: 11,
    name: 'Yoga Mat Premium',
    description: 'Non-slip exercise mat with carrying strap',
    price: 34.99,
    category: 'sports',
    rating: 4.6,
    reviewCount: 2341,
    image: '🧘',
    inStock: true,
    tags: ['yoga', 'fitness', 'exercise'],
    featured: false,
  },
  {
    id: 12,
    name: 'Running Shoes',
    description: 'Lightweight running shoes with cushioned sole',
    price: 89.99,
    originalPrice: 119.99,
    category: 'sports',
    rating: 4.4,
    reviewCount: 1654,
    image: '👟',
    inStock: true,
    tags: ['running', 'athletic', 'lightweight'],
    featured: true,
  },
]
```

---

### Step 4: Build ProductCard Component

Create `src/components/ProductCard.vue`:

```vue
<script setup lang="ts">
import { computed } from 'vue'
import type { Product } from '@/types/product'

const props = defineProps<{
  product: Product
}>()

const discountPercent = computed(() => {
  if (!props.product.originalPrice) return 0
  return Math.round(
    ((props.product.originalPrice - props.product.price) / props.product.originalPrice) * 100
  )
})

const ratingStars = computed(() => {
  const full = Math.floor(props.product.rating)
  const hasHalf = props.product.rating % 1 >= 0.5
  return {
    full,
    hasHalf,
    empty: 5 - full - (hasHalf ? 1 : 0),
  }
})

const formatReviewCount = (count: number): string => {
  if (count >= 1000) {
    return (count / 1000).toFixed(1) + 'k'
  }
  return count.toString()
}
</script>

<template>
  <div
    class="card bg-base-100 shadow-md hover:shadow-xl transition-all duration-300 group"
    :class="{ 'opacity-60': !product.inStock }"
  >
    <!-- Image Area -->
    <figure class="relative pt-6 px-6">
      <span class="text-8xl group-hover:scale-110 transition-transform duration-300">
        {{ product.image }}
      </span>

      <!-- Badges -->
      <!--
        TODO: Use v-if to show discount badge only when originalPrice exists
        TODO: Use v-if to show "Featured" badge when product.featured is true
      -->
      <div class="absolute top-2 left-2 flex flex-col gap-1">
        <span
          v-if="product.originalPrice"
          class="badge badge-error"
        >
          -{{ discountPercent }}%
        </span>
        <span
          v-if="product.featured"
          class="badge badge-warning"
        >
          Featured
        </span>
      </div>

      <!-- Out of Stock Overlay -->
      <!--
        TODO: Use v-show for the overlay (it toggles based on stock status)
      -->
      <div
        v-show="!product.inStock"
        class="absolute inset-0 bg-base-300/80 flex items-center justify-center rounded-t-2xl"
      >
        <span class="badge badge-lg badge-error">Out of Stock</span>
      </div>
    </figure>

    <div class="card-body p-4">
      <!-- Category -->
      <span class="badge badge-outline badge-sm capitalize">
        {{ product.category }}
      </span>

      <!-- Product Name -->
      <h3 class="card-title text-base line-clamp-2">
        {{ product.name }}
      </h3>

      <!-- Rating -->
      <div class="flex items-center gap-1 text-sm">
        <span class="text-warning">
          <template v-for="n in ratingStars.full" :key="'full-' + n">★</template>
          <template v-if="ratingStars.hasHalf">½</template>
          <template v-for="n in ratingStars.empty" :key="'empty-' + n">☆</template>
        </span>
        <span class="text-base-content/70">
          {{ product.rating }} ({{ formatReviewCount(product.reviewCount) }})
        </span>
      </div>

      <!-- Price -->
      <div class="flex items-baseline gap-2 mt-2">
        <span class="text-xl font-bold text-primary">
          ${{ product.price.toFixed(2) }}
        </span>
        <!--
          TODO: Use v-if to show original price only when it exists
        -->
        <span
          v-if="product.originalPrice"
          class="text-sm line-through text-base-content/50"
        >
          ${{ product.originalPrice.toFixed(2) }}
        </span>
      </div>

      <!-- Add to Cart Button -->
      <div class="card-actions mt-4">
        <button
          class="btn btn-primary btn-block"
          :disabled="!product.inStock"
        >
          {{ product.inStock ? 'Add to Cart' : 'Notify Me' }}
        </button>
      </div>
    </div>
  </div>
</template>
```

---

### Step 5: Build SearchBar Component

Create `src/components/SearchBar.vue`:

```vue
<script setup lang="ts">
const props = defineProps<{
  modelValue: string
  resultCount: number
}>()

const emit = defineEmits<{
  'update:modelValue': [value: string]
}>()
</script>

<template>
  <div class="form-control">
    <div class="relative">
      <input
        type="text"
        placeholder="Search products..."
        class="input input-bordered w-full pr-10"
        :value="modelValue"
        @input="emit('update:modelValue', ($event.target as HTMLInputElement).value)"
      />
      <span class="absolute right-3 top-1/2 -translate-y-1/2 text-base-content/50">
        🔍
      </span>
    </div>
    <!--
      TODO: Use v-show to display result count only when there's a search query
    -->
    <label v-show="modelValue.trim()" class="label">
      <span class="label-text-alt">
        Found {{ resultCount }} product{{ resultCount !== 1 ? 's' : '' }}
      </span>
    </label>
  </div>
</template>
```

---

### Step 6: Build SortDropdown Component

Create `src/components/SortDropdown.vue`:

```vue
<script setup lang="ts">
import type { SortOption } from '@/types/product'

defineProps<{
  modelValue: SortOption
}>()

const emit = defineEmits<{
  'update:modelValue': [value: SortOption]
}>()

const sortOptions: { value: SortOption; label: string }[] = [
  { value: 'featured', label: 'Featured' },
  { value: 'newest', label: 'Newest' },
  { value: 'price-asc', label: 'Price: Low to High' },
  { value: 'price-desc', label: 'Price: High to Low' },
  { value: 'rating', label: 'Top Rated' },
]
</script>

<template>
  <div class="form-control">
    <select
      class="select select-bordered"
      :value="modelValue"
      @change="emit('update:modelValue', ($event.target as HTMLSelectElement).value as SortOption)"
    >
      <!--
        TODO: Use v-for to render sort options
      -->
      <option
        v-for="option in sortOptions"
        :key="option.value"
        :value="option.value"
      >
        {{ option.label }}
      </option>
    </select>
  </div>
</template>
```

---

### Step 7: Build FilterPanel Component

Create `src/components/FilterPanel.vue`:

```vue
<script setup lang="ts">
import { computed } from 'vue'
import type { Category, FilterState } from '@/types/product'

const props = defineProps<{
  modelValue: FilterState
  categoryCounts: Record<Category, number>
}>()

const emit = defineEmits<{
  'update:modelValue': [value: FilterState]
  reset: []
}>()

const categories: { value: Category; label: string; icon: string }[] = [
  { value: 'electronics', label: 'Electronics', icon: '📱' },
  { value: 'clothing', label: 'Clothing', icon: '👕' },
  { value: 'books', label: 'Books', icon: '📚' },
  { value: 'home', label: 'Home', icon: '🏠' },
  { value: 'sports', label: 'Sports', icon: '⚽' },
]

const ratingOptions = [4, 3, 2, 1]

const toggleCategory = (category: Category) => {
  const current = [...props.modelValue.categories]
  const index = current.indexOf(category)
  if (index > -1) {
    current.splice(index, 1)
  } else {
    current.push(category)
  }
  emit('update:modelValue', { ...props.modelValue, categories: current })
}

const updatePriceRange = (field: 'min' | 'max', value: string) => {
  const numValue = parseFloat(value) || 0
  emit('update:modelValue', {
    ...props.modelValue,
    priceRange: { ...props.modelValue.priceRange, [field]: numValue },
  })
}

const updateInStockOnly = (value: boolean) => {
  emit('update:modelValue', { ...props.modelValue, inStockOnly: value })
}

const updateMinRating = (value: number) => {
  emit('update:modelValue', { ...props.modelValue, minRating: value })
}

const hasActiveFilters = computed(() => {
  return (
    props.modelValue.categories.length > 0 ||
    props.modelValue.priceRange.min > 0 ||
    props.modelValue.priceRange.max < 1000 ||
    props.modelValue.inStockOnly ||
    props.modelValue.minRating > 0
  )
})
</script>

<template>
  <div class="bg-base-200 rounded-lg p-4 space-y-6">
    <div class="flex items-center justify-between">
      <h3 class="font-bold text-lg">Filters</h3>
      <!--
        TODO: Use v-show to only display reset button when filters are active
      -->
      <button
        v-show="hasActiveFilters"
        class="btn btn-ghost btn-xs"
        @click="emit('reset')"
      >
        Reset All
      </button>
    </div>

    <!-- Categories -->
    <div>
      <h4 class="font-semibold mb-3">Categories</h4>
      <div class="space-y-2">
        <!--
          TODO: Use v-for to render category checkboxes
          Apply 'opacity-50' class when count is 0
        -->
        <label
          v-for="cat in categories"
          :key="cat.value"
          class="flex items-center gap-3 cursor-pointer"
          :class="{ 'opacity-50': categoryCounts[cat.value] === 0 }"
        >
          <input
            type="checkbox"
            class="checkbox checkbox-sm checkbox-primary"
            :checked="modelValue.categories.includes(cat.value)"
            @change="toggleCategory(cat.value)"
          />
          <span>{{ cat.icon }} {{ cat.label }}</span>
          <span class="text-sm text-base-content/50 ml-auto">
            ({{ categoryCounts[cat.value] }})
          </span>
        </label>
      </div>
    </div>

    <!-- Price Range -->
    <div>
      <h4 class="font-semibold mb-3">Price Range</h4>
      <div class="flex items-center gap-2">
        <input
          type="number"
          class="input input-bordered input-sm w-24"
          placeholder="Min"
          :value="modelValue.priceRange.min || ''"
          @input="updatePriceRange('min', ($event.target as HTMLInputElement).value)"
        />
        <span>—</span>
        <input
          type="number"
          class="input input-bordered input-sm w-24"
          placeholder="Max"
          :value="modelValue.priceRange.max === 1000 ? '' : modelValue.priceRange.max"
          @input="updatePriceRange('max', ($event.target as HTMLInputElement).value || '1000')"
        />
      </div>
    </div>

    <!-- Rating -->
    <div>
      <h4 class="font-semibold mb-3">Minimum Rating</h4>
      <div class="space-y-2">
        <!--
          TODO: Use v-for to render rating filter options
        -->
        <label
          v-for="rating in ratingOptions"
          :key="rating"
          class="flex items-center gap-3 cursor-pointer"
        >
          <input
            type="radio"
            name="rating"
            class="radio radio-sm radio-primary"
            :checked="modelValue.minRating === rating"
            @change="updateMinRating(rating)"
          />
          <span class="text-warning">
            {{ '★'.repeat(rating) }}{{ '☆'.repeat(5 - rating) }}
          </span>
          <span class="text-sm">& up</span>
        </label>
        <label class="flex items-center gap-3 cursor-pointer">
          <input
            type="radio"
            name="rating"
            class="radio radio-sm radio-primary"
            :checked="modelValue.minRating === 0"
            @change="updateMinRating(0)"
          />
          <span class="text-sm">All ratings</span>
        </label>
      </div>
    </div>

    <!-- Availability -->
    <div>
      <label class="flex items-center gap-3 cursor-pointer">
        <input
          type="checkbox"
          class="checkbox checkbox-sm checkbox-primary"
          :checked="modelValue.inStockOnly"
          @change="updateInStockOnly(($event.target as HTMLInputElement).checked)"
        />
        <span>In Stock Only</span>
      </label>
    </div>
  </div>
</template>
```

---

### Step 8: Build EmptyState Component

Create `src/components/EmptyState.vue`:

```vue
<script setup lang="ts">
defineProps<{
  searchQuery: string
  hasFilters: boolean
}>()

const emit = defineEmits<{
  'clear-search': []
  'reset-filters': []
}>()
</script>

<template>
  <div class="text-center py-16">
    <div class="text-8xl mb-6">🔍</div>
    <h3 class="text-2xl font-bold mb-2">No products found</h3>

    <!--
      TODO: Use v-if/v-else-if/v-else to show different messages
      - If searchQuery exists: show "No results for [query]"
      - Else if hasFilters: show "No products match your filters"
      - Else: show generic "No products available"
    -->
    <p v-if="searchQuery" class="text-base-content/70 mb-6">
      No results for "<strong>{{ searchQuery }}</strong>"
    </p>
    <p v-else-if="hasFilters" class="text-base-content/70 mb-6">
      No products match your current filters
    </p>
    <p v-else class="text-base-content/70 mb-6">
      No products are currently available
    </p>

    <div class="flex flex-col sm:flex-row gap-3 justify-center">
      <!--
        TODO: Use v-if to show "Clear Search" button only when searchQuery exists
        TODO: Use v-if to show "Reset Filters" button only when hasFilters is true
      -->
      <button
        v-if="searchQuery"
        class="btn btn-outline"
        @click="emit('clear-search')"
      >
        Clear Search
      </button>
      <button
        v-if="hasFilters"
        class="btn btn-outline"
        @click="emit('reset-filters')"
      >
        Reset Filters
      </button>
    </div>
  </div>
</template>
```

---

### Step 9: Build Main ProductGrid Component

Create `src/components/ProductGrid.vue`:

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'
import type { Product, Category, SortOption, FilterState } from '@/types/product'
import { products as productData } from '@/data/products'
import ProductCard from './ProductCard.vue'
import SearchBar from './SearchBar.vue'
import SortDropdown from './SortDropdown.vue'
import FilterPanel from './FilterPanel.vue'
import EmptyState from './EmptyState.vue'

const products = ref<Product[]>(productData)

const searchQuery = ref<string>('')
const sortBy = ref<SortOption>('featured')
const showFilters = ref<boolean>(true)

const filters = ref<FilterState>({
  categories: [],
  priceRange: { min: 0, max: 1000 },
  inStockOnly: false,
  minRating: 0,
})

const categoryCounts = computed(() => {
  const counts: Record<Category, number> = {
    electronics: 0,
    clothing: 0,
    books: 0,
    home: 0,
    sports: 0,
  }
  products.value.forEach((p) => {
    counts[p.category]++
  })
  return counts
})

const filteredProducts = computed(() => {
  let result = [...products.value]

  if (searchQuery.value.trim()) {
    const query = searchQuery.value.toLowerCase()
    result = result.filter(
      (p) =>
        p.name.toLowerCase().includes(query) ||
        p.description.toLowerCase().includes(query) ||
        p.tags.some((tag) => tag.toLowerCase().includes(query))
    )
  }

  if (filters.value.categories.length > 0) {
    result = result.filter((p) => filters.value.categories.includes(p.category))
  }

  result = result.filter(
    (p) =>
      p.price >= filters.value.priceRange.min && p.price <= filters.value.priceRange.max
  )

  if (filters.value.inStockOnly) {
    result = result.filter((p) => p.inStock)
  }

  if (filters.value.minRating > 0) {
    result = result.filter((p) => p.rating >= filters.value.minRating)
  }

  switch (sortBy.value) {
    case 'price-asc':
      result.sort((a, b) => a.price - b.price)
      break
    case 'price-desc':
      result.sort((a, b) => b.price - a.price)
      break
    case 'rating':
      result.sort((a, b) => b.rating - a.rating)
      break
    case 'newest':
      result.sort((a, b) => b.id - a.id)
      break
    case 'featured':
    default:
      result.sort((a, b) => (b.featured ? 1 : 0) - (a.featured ? 1 : 0))
  }

  return result
})

const hasActiveFilters = computed(() => {
  return (
    filters.value.categories.length > 0 ||
    filters.value.priceRange.min > 0 ||
    filters.value.priceRange.max < 1000 ||
    filters.value.inStockOnly ||
    filters.value.minRating > 0
  )
})

const clearSearch = () => {
  searchQuery.value = ''
}

const resetFilters = () => {
  filters.value = {
    categories: [],
    priceRange: { min: 0, max: 1000 },
    inStockOnly: false,
    minRating: 0,
  }
}
</script>

<template>
  <div class="max-w-7xl mx-auto p-6">
    <!-- Header -->
    <div class="mb-8">
      <h1 class="text-4xl font-bold mb-2">Shop All Products</h1>
      <p class="text-base-content/70">
        Browse our collection of {{ products.length }} products
      </p>
    </div>

    <!-- Controls Bar -->
    <div class="flex flex-col sm:flex-row gap-4 mb-6">
      <div class="flex-1">
        <SearchBar v-model="searchQuery" :result-count="filteredProducts.length" />
      </div>
      <div class="flex gap-2">
        <SortDropdown v-model="sortBy" />
        <button
          class="btn btn-outline"
          :class="{ 'btn-active': showFilters }"
          @click="showFilters = !showFilters"
        >
          {{ showFilters ? 'Hide' : 'Show' }} Filters
        </button>
      </div>
    </div>

    <!-- Main Content -->
    <div class="flex gap-6">
      <!-- Filter Panel Sidebar -->
      <!--
        TODO: Use v-show for the filter panel (toggle visibility without destroying)
      -->
      <aside v-show="showFilters" class="w-64 flex-shrink-0">
        <FilterPanel
          v-model="filters"
          :category-counts="categoryCounts"
          @reset="resetFilters"
        />
      </aside>

      <!-- Product Grid Area -->
      <main class="flex-1">
        <!--
          TODO: Use v-if/v-else to show either:
          - Product grid when filteredProducts.length > 0
          - EmptyState when no products match
        -->
        <div
          v-if="filteredProducts.length > 0"
          class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6"
        >
          <!--
            TODO: Use v-for to render ProductCard components
            Remember to add :key="product.id"
          -->
          <ProductCard
            v-for="product in filteredProducts"
            :key="product.id"
            :product="product"
          />
        </div>

        <EmptyState
          v-else
          :search-query="searchQuery"
          :has-filters="hasActiveFilters"
          @clear-search="clearSearch"
          @reset-filters="resetFilters"
        />
      </main>
    </div>

    <!-- Results Summary -->
    <!--
      TODO: Use v-show to display summary only when there are results
    -->
    <div v-show="filteredProducts.length > 0" class="mt-8 text-center text-sm text-base-content/70">
      Showing {{ filteredProducts.length }} of {{ products.length }} products
    </div>
  </div>
</template>
```

---

### Step 10: Update App.vue

```vue
<script setup lang="ts">
import ProductGrid from './components/ProductGrid.vue'
</script>

<template>
  <div class="min-h-screen bg-base-300">
    <ProductGrid />
  </div>
</template>
```

---

## Key Concepts Demonstrated

### 1. `v-for` with Computed Filtering

```vue
<ProductCard
  v-for="product in filteredProducts"
  :key="product.id"
  :product="product"
/>
```

**Why it works:** Instead of using `v-if` inside `v-for`, we pre-filter with a computed property. This is:
- More performant (filter runs once, not per item)
- Cleaner template code
- Easier to debug

### 2. `v-show` vs `v-if` Usage

```vue
<!-- v-show for frequently toggled UI -->
<aside v-show="showFilters" class="w-64">
  <FilterPanel />
</aside>

<!-- v-if for conditional content that changes less often -->
<div v-if="filteredProducts.length > 0">
  <!-- Product grid -->
</div>
<EmptyState v-else />
```

**Why:** Filter panel toggles frequently, so `v-show` (CSS toggle) is more efficient. Product grid vs. empty state changes less frequently and has different structure, so `v-if` is appropriate.

### 3. Empty State Pattern

```vue
<p v-if="searchQuery">
  No results for "<strong>{{ searchQuery }}</strong>"
</p>
<p v-else-if="hasFilters">
  No products match your current filters
</p>
<p v-else>
  No products are currently available
</p>
```

**Why:** Provides context-aware messaging to help users understand why there are no results and what action to take.

---

## Stretch Goals

1. **Pagination:** Add pagination or infinite scroll for large product lists
2. **Quick View Modal:** Add a modal that shows product details on hover/click
3. **Wishlist:** Allow users to save products to a wishlist with localStorage
4. **Filter Tags:** Show active filters as removable tags above the grid
5. **View Toggle:** Add grid/list view toggle with different layouts
6. **Product Comparison:** Allow selecting products to compare side-by-side

---

## Self-Assessment Checklist

- [ ] All components use `<script setup>` with TypeScript
- [ ] Product grid uses `v-for` with unique `:key` attributes
- [ ] Filtering uses computed property (not `v-if` inside `v-for`)
- [ ] Filter panel toggles with `v-show` (efficient for frequent toggles)
- [ ] Product grid vs. empty state uses `v-if`/`v-else`
- [ ] Empty state shows different messages based on search/filters
- [ ] Category checkboxes render with `v-for`
- [ ] Sort options render with `v-for`
- [ ] Discount badges only show when `originalPrice` exists (`v-if`)
- [ ] Out-of-stock overlay uses `v-show`
- [ ] Search result count only shows when there's a query (`v-show`)
- [ ] Reset filters button only shows when filters are active (`v-show`)
- [ ] All filters work correctly together
- [ ] Sorting works in all directions
- [ ] Product cards show correct conditional styling
