# Lesson 21.2: State Management II — Pinia Getters & Plugins — Practice & Application
# Управление состоянием II: Геттеры и Плагины Pinia — Практика и Применение

---

## 1. Lesson Metadata

| Field | Value |
|:--- |:--- |
| **Lesson Number** | 21.2 of 25 (Part 2 of 2) |
| **Topic** | Pinia Getters & Plugins / Геттеры и Плагины Pinia |
| **Continues From** | Lesson 21.1 |
| **Duration** | 30-35 minutes |

**Learning Objectives — Part 2 (Practice):**
1. **Apply:** Implement getters for derived state and create a persistence plugin
2. **Analyze:** Debug getter dependencies and plugin timing issues

---

## 2. Quick Recap

- **Getters** are `computed()` values in your store — cached, reactive, derived from state
- **Plugins** are functions that run when stores are created, adding features to all stores
- Use `store.$subscribe()` to react to state changes (for persistence)
- Use `store.$onAction()` to intercept action calls (for logging)
- Plugins receive context with `{ store, pinia, app, options }`

---

## 3. The Variable Frame & Complexity Scale

### Basic Form: Simple Getter

```typescript
// src/stores/counter.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useCounterStore = defineStore('counter', () => {
  const count = ref(0)
  
  // Simple getter: derive a value from state
  const doubled = computed(() => count.value * 2)
  
  function increment() {
    count.value++
  }

  return { count, doubled, increment }
})
```

### With Options: Getters Using Other Getters

```typescript
// src/stores/cart.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

interface CartItem {
  id: number
  name: string
  price: number
  quantity: number
}

export const useCartStore = defineStore('cart', () => {
  const items = ref<CartItem[]>([])
  const discountPercent = ref(0)

  // Base getter
  const subtotal = computed(() => 
    items.value.reduce((sum, item) => sum + item.price * item.quantity, 0)
  )

  // Getter using another getter
  const discountAmount = computed(() => 
    subtotal.value * (discountPercent.value / 100)
  )

  // Getter using multiple getters
  const total = computed(() => 
    subtotal.value - discountAmount.value
  )

  // Getter returning formatted string
  const formattedTotal = computed(() => 
    `$${total.value.toFixed(2)}`
  )

  // Boolean getter
  const isEmpty = computed(() => items.value.length === 0)

  return { 
    items, 
    discountPercent, 
    subtotal, 
    discountAmount, 
    total, 
    formattedTotal, 
    isEmpty 
  }
})
```

### Advanced Form: Getter with Parameters (Getter Factory)

```typescript
// src/stores/products.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

interface Product {
  id: number
  name: string
  price: number
  category: string
}

export const useProductStore = defineStore('products', () => {
  const products = ref<Product[]>([])

  // Standard getter
  const categories = computed(() => 
    [...new Set(products.value.map(p => p.category))]
  )

  // Getter factory: returns a function that takes a parameter
  // This is how you create "parameterized getters"
  const getProductsByCategory = computed(() => {
    return (category: string) => 
      products.value.filter(p => p.category === category)
  })

  // Another getter factory
  const getProductById = computed(() => {
    return (id: number) => 
      products.value.find(p => p.id === id)
  })

  // Price range getter factory
  const getProductsInPriceRange = computed(() => {
    return (min: number, max: number) =>
      products.value.filter(p => p.price >= min && p.price <= max)
  })

  return { 
    products, 
    categories, 
    getProductsByCategory, 
    getProductById,
    getProductsInPriceRange
  }
})

// Usage in component:
// const electronics = productStore.getProductsByCategory('electronics')
// const product = productStore.getProductById(123)
```

---

## 4. Micro-Implementation Drills

### Drill 1 (Basic): Create Getters for a User Store

Create getters for a user profile store.

```typescript
// src/stores/user.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

interface User {
  firstName: string
  lastName: string
  email: string
  age: number
  role: 'user' | 'admin' | 'moderator'
}

export const useUserStore = defineStore('user', () => {
  const user = ref<User | null>(null)

  // TODO 1: Create a getter `fullName` that returns "FirstName LastName"
  // If user is null, return "Guest"
  
  // TODO 2: Create a getter `isLoggedIn` that returns true if user exists
  
  // TODO 3: Create a getter `isAdmin` that returns true if role is 'admin'
  
  // TODO 4: Create a getter `isAdult` that returns true if age >= 18
  // If user is null, return false

  function login(userData: User) {
    user.value = userData
  }

  function logout() {
    user.value = null
  }

  // TODO 5: Return all state, getters, and actions
  return { user, login, logout }
})
```

**Solution:**

```typescript
// src/stores/user.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

interface User {
  firstName: string
  lastName: string
  email: string
  age: number
  role: 'user' | 'admin' | 'moderator'
}

export const useUserStore = defineStore('user', () => {
  const user = ref<User | null>(null)

  const fullName = computed(() => {
    if (!user.value) return 'Guest'
    return `${user.value.firstName} ${user.value.lastName}`
  })

  const isLoggedIn = computed(() => user.value !== null)

  const isAdmin = computed(() => user.value?.role === 'admin')

  const isAdult = computed(() => {
    if (!user.value) return false
    return user.value.age >= 18
  })

  function login(userData: User) {
    user.value = userData
  }

  function logout() {
    user.value = null
  }

  return { 
    user, 
    fullName, 
    isLoggedIn, 
    isAdmin, 
    isAdult, 
    login, 
    logout 
  }
})
```

---

### Drill 2 (Variation): Chained Getters for Shopping Cart

Create a store with getters that build on each other.

```typescript
// src/stores/shopping.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

interface Item {
  id: number
  name: string
  price: number
  quantity: number
}

export const useShoppingStore = defineStore('shopping', () => {
  const items = ref<Item[]>([])
  const taxRate = ref(0.0825) // 8.25%
  const shippingThreshold = ref(75) // Free shipping over $75
  const shippingCost = ref(9.99)

  // TODO 1: Create `itemCount` getter - sum of all quantities

  // TODO 2: Create `subtotal` getter - sum of (price × quantity)

  // TODO 3: Create `taxAmount` getter - subtotal × taxRate

  // TODO 4: Create `qualifiesForFreeShipping` getter - subtotal >= threshold

  // TODO 5: Create `shippingAmount` getter - 0 if free shipping, else shippingCost

  // TODO 6: Create `grandTotal` getter - subtotal + tax + shipping

  return { 
    items, 
    taxRate, 
    shippingThreshold, 
    shippingCost
    // Add all getters here
  }
})
```

**Solution:**

```typescript
// src/stores/shopping.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

interface Item {
  id: number
  name: string
  price: number
  quantity: number
}

export const useShoppingStore = defineStore('shopping', () => {
  const items = ref<Item[]>([])
  const taxRate = ref(0.0825)
  const shippingThreshold = ref(75)
  const shippingCost = ref(9.99)

  const itemCount = computed(() => 
    items.value.reduce((sum, item) => sum + item.quantity, 0)
  )

  const subtotal = computed(() => 
    items.value.reduce((sum, item) => sum + item.price * item.quantity, 0)
  )

  const taxAmount = computed(() => 
    subtotal.value * taxRate.value
  )

  const qualifiesForFreeShipping = computed(() => 
    subtotal.value >= shippingThreshold.value
  )

  const shippingAmount = computed(() => 
    qualifiesForFreeShipping.value ? 0 : shippingCost.value
  )

  const grandTotal = computed(() => 
    subtotal.value + taxAmount.value + shippingAmount.value
  )

  return { 
    items, 
    taxRate, 
    shippingThreshold, 
    shippingCost,
    itemCount,
    subtotal,
    taxAmount,
    qualifiesForFreeShipping,
    shippingAmount,
    grandTotal
  }
})
```

---

### Drill 3 (Combination): Create a Persistence Plugin

Build a localStorage persistence plugin from scratch.

```typescript
// src/plugins/persist.ts
import type { PiniaPluginContext } from 'pinia'

// TODO: Implement the persistence plugin
// Requirements:
// 1. On store creation, check localStorage for saved state
// 2. If found, use $patch to restore the state
// 3. Subscribe to state changes and save to localStorage
// 4. Use the store's $id as part of the storage key

export function persistPlugin(context: PiniaPluginContext) {
  // Your code here
}
```

**Solution:**

```typescript
// src/plugins/persist.ts
import type { PiniaPluginContext } from 'pinia'

export function persistPlugin(context: PiniaPluginContext) {
  // Get the store's unique ID
  const storeId = context.store.$id
  
  // Create a unique key for localStorage
  const storageKey = `pinia-store-${storeId}`

  // HYDRATE: Restore state from localStorage when store is created
  const savedStateJson = localStorage.getItem(storageKey)
  
  if (savedStateJson) {
    try {
      const savedState = JSON.parse(savedStateJson)
      // $patch merges the saved state into current state
      context.store.$patch(savedState)
    } catch (error) {
      console.error(`Failed to parse saved state for ${storeId}:`, error)
      // Remove corrupted data
      localStorage.removeItem(storageKey)
    }
  }

  // PERSIST: Save state to localStorage whenever it changes
  context.store.$subscribe(
    (mutation, state) => {
      try {
        localStorage.setItem(storageKey, JSON.stringify(state))
      } catch (error) {
        console.error(`Failed to save state for ${storeId}:`, error)
      }
    },
    { detached: true } // Continue even if component unmounts
  )
}
```

---

### Drill 4 (Edge Case): Selective Persistence Plugin

Create a plugin that only persists certain stores.

```typescript
// src/plugins/selectivePersist.ts
import type { PiniaPluginContext } from 'pinia'

// Only persist stores whose IDs are in this list
const STORES_TO_PERSIST = ['cart', 'user', 'settings']

export function selectivePersistPlugin(context: PiniaPluginContext) {
  // TODO: Implement plugin that:
  // 1. Checks if store.$id is in STORES_TO_PERSIST
  // 2. If not, return early (don't persist)
  // 3. If yes, apply persistence logic
  
  // Your code here
}
```

**Solution:**

```typescript
// src/plugins/selectivePersist.ts
import type { PiniaPluginContext } from 'pinia'

const STORES_TO_PERSIST = ['cart', 'user', 'settings']

export function selectivePersistPlugin(context: PiniaPluginContext) {
  const storeId = context.store.$id

  // Skip stores not in the list
  if (!STORES_TO_PERSIST.includes(storeId)) {
    return
  }

  const storageKey = `pinia-${storeId}`

  // Hydrate
  const saved = localStorage.getItem(storageKey)
  if (saved) {
    try {
      context.store.$patch(JSON.parse(saved))
    } catch {
      localStorage.removeItem(storageKey)
    }
  }

  // Persist
  context.store.$subscribe(
    (_, state) => {
      localStorage.setItem(storageKey, JSON.stringify(state))
    },
    { detached: true }
  )
}
```

---

## 5. Common Pitfalls & Anti-Patterns

| ❌ Common Mistake | ✅ Correct Approach | Why It Matters |
|:--- |:--- |:--- |
| Using `function` instead of `computed` for getters: `function getTotal() { return ... }` | Use `computed()`: `const total = computed(() => ...)` | Regular functions recalculate every call; `computed` caches and only recalculates when dependencies change |
| Mutating state inside a getter: `const doubled = computed(() => { count.value = count.value * 2; return count.value })` | Getters should be pure (no side effects): `const doubled = computed(() => count.value * 2)` | Mutations inside getters cause infinite loops and unpredictable behavior |
| Forgetting `{ detached: true }` in plugin subscriptions | Add it: `store.$subscribe(callback, { detached: true })` | Without `detached`, the subscription is tied to the component lifecycle and may be cleaned up too early |

### Example: Function vs. Computed Getter

```typescript
// ❌ BAD: Regular function (not cached)
function getTotal() {
  console.log('Calculating...') // This logs EVERY time
  return items.value.reduce((sum, i) => sum + i.price, 0)
}

// ✅ GOOD: Computed (cached)
const total = computed(() => {
  console.log('Calculating...') // This logs only when items change
  return items.value.reduce((sum, i) => sum + i.price, 0)
})
```

---

## 6. Mini-Project: E-Commerce Cart with Persistence

Build a complete shopping cart store with getters and persistence.

**File Structure:**
```
src/
├── plugins/
│   └── persist.ts
├── stores/
│   └── cart.ts
├── components/
│   └── ShoppingCart.vue
├── App.vue
└── main.ts
```

**src/plugins/persist.ts:**
```typescript
import type { PiniaPluginContext } from 'pinia'

export function persistPlugin({ store }: PiniaPluginContext) {
  const key = `pinia-${store.$id}`

  const saved = localStorage.getItem(key)
  if (saved) {
    try {
      store.$patch(JSON.parse(saved))
    } catch {
      localStorage.removeItem(key)
    }
  }

  store.$subscribe(
    (_, state) => {
      localStorage.setItem(key, JSON.stringify(state))
    },
    { detached: true }
  )
}
```

**src/stores/cart.ts:**
```typescript
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

// Type definition for a product that can be added to cart
interface Product {
  id: number
  name: string
  price: number
  image: string
}

// Type definition for a cart item (product + quantity)
interface CartItem extends Product {
  quantity: number
}

export const useCartStore = defineStore('cart', () => {
  // ============ STATE ============
  // Array of items currently in the cart
  const items = ref<CartItem[]>([])
  
  // Tax rate as a decimal (8% = 0.08)
  const taxRate = ref(0.08)
  
  // Minimum subtotal for free shipping
  const freeShippingThreshold = ref(50)
  
  // Cost of shipping if threshold not met
  const shippingCost = ref(5.99)
  
  // Coupon code currently applied
  const appliedCoupon = ref<string | null>(null)
  
  // Discount percentage from coupon (10 = 10%)
  const couponDiscount = ref(0)

  // ============ GETTERS ============
  
  // Total number of items in cart (sum of quantities)
  const itemCount = computed(() => {
    return items.value.reduce((total, item) => total + item.quantity, 0)
  })

  // Whether the cart is empty
  const isEmpty = computed(() => items.value.length === 0)

  // Subtotal before tax, shipping, and discounts
  const subtotal = computed(() => {
    return items.value.reduce(
      (total, item) => total + item.price * item.quantity,
      0
    )
  })

  // Discount amount based on coupon percentage
  const discountAmount = computed(() => {
    return subtotal.value * (couponDiscount.value / 100)
  })

  // Subtotal after coupon discount
  const subtotalAfterDiscount = computed(() => {
    return subtotal.value - discountAmount.value
  })

  // Tax amount (calculated on discounted subtotal)
  const taxAmount = computed(() => {
    return subtotalAfterDiscount.value * taxRate.value
  })

  // Whether order qualifies for free shipping
  const qualifiesForFreeShipping = computed(() => {
    return subtotalAfterDiscount.value >= freeShippingThreshold.value
  })

  // Actual shipping cost (0 if free shipping applies)
  const shippingAmount = computed(() => {
    return qualifiesForFreeShipping.value ? 0 : shippingCost.value
  })

  // Amount needed to reach free shipping
  const amountUntilFreeShipping = computed(() => {
    if (qualifiesForFreeShipping.value) return 0
    return freeShippingThreshold.value - subtotalAfterDiscount.value
  })

  // Grand total: subtotal - discount + tax + shipping
  const grandTotal = computed(() => {
    return subtotalAfterDiscount.value + taxAmount.value + shippingAmount.value
  })

  // Formatted currency strings for display
  const formattedSubtotal = computed(() => formatCurrency(subtotal.value))
  const formattedDiscount = computed(() => formatCurrency(discountAmount.value))
  const formattedTax = computed(() => formatCurrency(taxAmount.value))
  const formattedShipping = computed(() => 
    qualifiesForFreeShipping.value ? 'FREE' : formatCurrency(shippingAmount.value)
  )
  const formattedTotal = computed(() => formatCurrency(grandTotal.value))

  // Helper function to format numbers as currency
  function formatCurrency(amount: number): string {
    return new Intl.NumberFormat('en-US', {
      style: 'currency',
      currency: 'USD'
    }).format(amount)
  }

  // Getter factory: get a specific item by ID
  const getItemById = computed(() => {
    return (id: number) => items.value.find(item => item.id === id)
  })

  // ============ ACTIONS ============
  
  // Add a product to the cart
  function addItem(product: Product) {
    const existingItem = items.value.find(item => item.id === product.id)
    
    if (existingItem) {
      existingItem.quantity++
    } else {
      items.value.push({ ...product, quantity: 1 })
    }
  }

  // Remove an item completely from cart
  function removeItem(productId: number) {
    const index = items.value.findIndex(item => item.id === productId)
    if (index > -1) {
      items.value.splice(index, 1)
    }
  }

  // Update quantity of a specific item
  function updateQuantity(productId: number, quantity: number) {
    const item = items.value.find(item => item.id === productId)
    if (item) {
      if (quantity <= 0) {
        removeItem(productId)
      } else {
        item.quantity = quantity
      }
    }
  }

  // Increment quantity by 1
  function incrementQuantity(productId: number) {
    const item = items.value.find(item => item.id === productId)
    if (item) {
      item.quantity++
    }
  }

  // Decrement quantity by 1 (removes if quantity becomes 0)
  function decrementQuantity(productId: number) {
    const item = items.value.find(item => item.id === productId)
    if (item) {
      if (item.quantity <= 1) {
        removeItem(productId)
      } else {
        item.quantity--
      }
    }
  }

  // Apply a coupon code
  function applyCoupon(code: string) {
    // Simulated coupon validation
    const coupons: Record<string, number> = {
      'SAVE10': 10,
      'SAVE20': 20,
      'HALFOFF': 50
    }
    
    const discount = coupons[code.toUpperCase()]
    if (discount) {
      appliedCoupon.value = code.toUpperCase()
      couponDiscount.value = discount
      return true
    }
    return false
  }

  // Remove applied coupon
  function removeCoupon() {
    appliedCoupon.value = null
    couponDiscount.value = 0
  }

  // Clear entire cart
  function clearCart() {
    items.value = []
    removeCoupon()
  }

  // ============ RETURN ============
  return {
    // State
    items,
    taxRate,
    freeShippingThreshold,
    shippingCost,
    appliedCoupon,
    couponDiscount,
    // Getters
    itemCount,
    isEmpty,
    subtotal,
    discountAmount,
    subtotalAfterDiscount,
    taxAmount,
    qualifiesForFreeShipping,
    shippingAmount,
    amountUntilFreeShipping,
    grandTotal,
    formattedSubtotal,
    formattedDiscount,
    formattedTax,
    formattedShipping,
    formattedTotal,
    getItemById,
    // Actions
    addItem,
    removeItem,
    updateQuantity,
    incrementQuantity,
    decrementQuantity,
    applyCoupon,
    removeCoupon,
    clearCart
  }
})
```

**src/components/ShoppingCart.vue:**
```vue
<script setup lang="ts">
import { ref } from 'vue'
import { useCartStore } from '@/stores/cart'

// Get the cart store instance
const cart = useCartStore()

// Local state for coupon input
const couponInput = ref('')
const couponError = ref('')

// Sample products to add
const sampleProducts = [
  { id: 1, name: 'Vue.js T-Shirt', price: 24.99, image: '👕' },
  { id: 2, name: 'TypeScript Mug', price: 14.99, image: '☕' },
  { id: 3, name: 'Pinia Sticker Pack', price: 4.99, image: '🏷️' },
  { id: 4, name: 'Vite Hoodie', price: 49.99, image: '🧥' }
]

// Handle coupon submission
function handleApplyCoupon() {
  couponError.value = ''
  
  if (!couponInput.value.trim()) {
    couponError.value = 'Please enter a coupon code'
    return
  }
  
  const success = cart.applyCoupon(couponInput.value)
  
  if (success) {
    couponInput.value = ''
  } else {
    couponError.value = 'Invalid coupon code'
  }
}
</script>

<template>
  <div class="p-6 max-w-4xl mx-auto">
    <h1 class="text-3xl font-bold mb-6">Shopping Cart</h1>
    
    <!-- Products to Add -->
    <section class="mb-8">
      <h2 class="text-xl font-semibold mb-4">Available Products</h2>
      <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
        <div
          v-for="product in sampleProducts"
          :key="product.id"
          class="card bg-base-200 p-4 text-center"
        >
          <div class="text-4xl mb-2">{{ product.image }}</div>
          <h3 class="font-medium">{{ product.name }}</h3>
          <p class="text-sm text-gray-500">${{ product.price.toFixed(2) }}</p>
          <button
            @click="cart.addItem(product)"
            class="btn btn-sm btn-primary mt-2"
          >
            Add to Cart
          </button>
        </div>
      </div>
    </section>
    
    <!-- Cart Contents -->
    <section class="mb-8">
      <div class="flex items-center justify-between mb-4">
        <h2 class="text-xl font-semibold">
          Your Cart ({{ cart.itemCount }} item{{ cart.itemCount !== 1 ? 's' : '' }})
        </h2>
        <button
          v-if="!cart.isEmpty"
          @click="cart.clearCart"
          class="btn btn-sm btn-outline btn-error"
        >
          Clear Cart
        </button>
      </div>
      
      <!-- Empty State -->
      <div v-if="cart.isEmpty" class="text-center py-8 bg-base-200 rounded-lg">
        <p class="text-gray-500">Your cart is empty</p>
        <p class="text-sm text-gray-400">Add some products above!</p>
      </div>
      
      <!-- Cart Items -->
      <ul v-else class="space-y-4">
        <li
          v-for="item in cart.items"
          :key="item.id"
          class="flex items-center gap-4 bg-base-200 p-4 rounded-lg"
        >
          <span class="text-3xl">{{ item.image }}</span>
          
          <div class="flex-1">
            <h3 class="font-medium">{{ item.name }}</h3>
            <p class="text-sm text-gray-500">${{ item.price.toFixed(2) }} each</p>
          </div>
          
          <!-- Quantity Controls -->
          <div class="flex items-center gap-2">
            <button
              @click="cart.decrementQuantity(item.id)"
              class="btn btn-sm btn-circle"
            >
              -
            </button>
            <span class="w-8 text-center font-medium">{{ item.quantity }}</span>
            <button
              @click="cart.incrementQuantity(item.id)"
              class="btn btn-sm btn-circle"
            >
              +
            </button>
          </div>
          
          <!-- Item Total -->
          <div class="text-right min-w-24">
            <p class="font-semibold">${{ (item.price * item.quantity).toFixed(2) }}</p>
          </div>
          
          <!-- Remove Button -->
          <button
            @click="cart.removeItem(item.id)"
            class="btn btn-sm btn-ghost text-error"
          >
            ✕
          </button>
        </li>
      </ul>
    </section>
    
    <!-- Coupon Section -->
    <section v-if="!cart.isEmpty" class="mb-8">
      <h3 class="font-semibold mb-2">Have a coupon?</h3>
      
      <!-- Applied Coupon -->
      <div v-if="cart.appliedCoupon" class="flex items-center gap-2 text-success">
        <span class="badge badge-success">{{ cart.appliedCoupon }}</span>
        <span>{{ cart.couponDiscount }}% off applied!</span>
        <button @click="cart.removeCoupon" class="btn btn-xs btn-ghost">
          Remove
        </button>
      </div>
      
      <!-- Coupon Input -->
      <div v-else class="flex gap-2">
        <input
          v-model="couponInput"
          type="text"
          placeholder="Enter coupon code"
          class="input input-bordered flex-1 max-w-xs"
          @keyup.enter="handleApplyCoupon"
        />
        <button @click="handleApplyCoupon" class="btn btn-outline">
          Apply
        </button>
      </div>
      <p v-if="couponError" class="text-error text-sm mt-1">{{ couponError }}</p>
      <p class="text-xs text-gray-400 mt-1">Try: SAVE10, SAVE20, or HALFOFF</p>
    </section>
    
    <!-- Order Summary -->
    <section v-if="!cart.isEmpty" class="card bg-base-200 p-6">
      <h2 class="text-xl font-semibold mb-4">Order Summary</h2>
      
      <div class="space-y-2">
        <div class="flex justify-between">
          <span>Subtotal</span>
          <span>{{ cart.formattedSubtotal }}</span>
        </div>
        
        <div v-if="cart.discountAmount > 0" class="flex justify-between text-success">
          <span>Discount ({{ cart.couponDiscount }}%)</span>
          <span>-{{ cart.formattedDiscount }}</span>
        </div>
        
        <div class="flex justify-between">
          <span>Tax ({{ (cart.taxRate * 100).toFixed(1) }}%)</span>
          <span>{{ cart.formattedTax }}</span>
        </div>
        
        <div class="flex justify-between">
          <span>Shipping</span>
          <span :class="{ 'text-success': cart.qualifiesForFreeShipping }">
            {{ cart.formattedShipping }}
          </span>
        </div>
        
        <!-- Free Shipping Progress -->
        <div v-if="!cart.qualifiesForFreeShipping" class="text-sm text-gray-500">
          Add ${{ cart.amountUntilFreeShipping.toFixed(2) }} more for free shipping!
        </div>
        
        <div class="divider my-2"></div>
        
        <div class="flex justify-between text-xl font-bold">
          <span>Total</span>
          <span>{{ cart.formattedTotal }}</span>
        </div>
      </div>
      
      <button class="btn btn-primary w-full mt-6">
        Proceed to Checkout
      </button>
    </section>
    
    <!-- Persistence Notice -->
    <p class="text-center text-xs text-gray-400 mt-6">
      Your cart is automatically saved. Refresh the page to test persistence!
    </p>
  </div>
</template>
```

**src/main.ts:**
```typescript
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import { persistPlugin } from './plugins/persist'
import App from './App.vue'
import './style.css'

const app = createApp(App)

const pinia = createPinia()
pinia.use(persistPlugin)

app.use(pinia)
app.mount('#app')
```

**How to Test:**
1. Add items to the cart
2. Adjust quantities
3. Apply coupon codes (SAVE10, SAVE20, HALFOFF)
4. **Refresh the page** — cart should persist!
5. Check browser DevTools → Application → Local Storage

---

## 7. Implementation Exercises (Progressive)

### Level 1 (Recall): Fill in the Blanks

```typescript
import { defineStore } from 'pinia'
import { ref, _____ } from 'vue'

export const useStore = defineStore('example', () => {
  const count = ref(0)
  
  const doubled = _____(_____ => count._____ * 2)
  
  return { count, doubled }
})
```

**Answer:** `computed`, `computed`, `()`, `value`

---

### Level 2 (Apply): Small Independent Tasks

**Exercise 2.1:** Create a getter factory that filters products by minimum rating:
```typescript
// Returns a function that takes minRating and returns filtered products
const getProductsByMinRating = computed(() => {
  // Your code: return (minRating: number) => ...
})
```

**Exercise 2.2:** Create a logging plugin that logs the store ID and all action names when a store is created:
```typescript
export function loggingPlugin(context: PiniaPluginContext) {
  // Your code: log store.$id and listen for action calls
}
```

---

### Level 3 (Debug): Find and Fix the Bugs

```typescript
// This store has 3 bugs. Find and fix them.
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useBuggyStore = defineStore('buggy', () => {
  const items = ref([1, 2, 3, 4, 5])

  // Bug 1: Why won't this update reactively?
  const total = items.value.reduce((a, b) => a + b, 0)

  // Bug 2: Why does this cause an infinite loop?
  const doubled = computed(() => {
    items.value.push(items.value.length)
    return items.value.map(n => n * 2)
  })

  // Bug 3: This getter factory doesn't work correctly
  const getItemsGreaterThan = (threshold: number) => {
    return items.value.filter(n => n > threshold)
  }

  return { items, total, doubled, getItemsGreaterThan }
})
```

**Bugs and Fixes:**

1. **Bug 1:** `total` is calculated once and never updates because it's not wrapped in `computed()`
   - Fix: `const total = computed(() => items.value.reduce((a, b) => a + b, 0))`

2. **Bug 2:** Getters must be pure — mutating state (`push`) inside a getter causes infinite reactivity loops
   - Fix: Remove the `push` call: `const doubled = computed(() => items.value.map(n => n * 2))`

3. **Bug 3:** Regular functions aren't cached. Should be a getter factory wrapped in `computed()`
   - Fix: 
   ```typescript
   const getItemsGreaterThan = computed(() => {
     return (threshold: number) => items.value.filter(n => n > threshold)
   })
   ```

**Corrected Code:**
```typescript
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useFixedStore = defineStore('fixed', () => {
  const items = ref([1, 2, 3, 4, 5])

  const total = computed(() => 
    items.value.reduce((a, b) => a + b, 0)
  )

  const doubled = computed(() => 
    items.value.map(n => n * 2)
  )

  const getItemsGreaterThan = computed(() => {
    return (threshold: number) => items.value.filter(n => n > threshold)
  })

  return { items, total, doubled, getItemsGreaterThan }
})
```

---

**Reply 'next' for Lesson 22.1 (Theory) — Async Components & Suspense.**
