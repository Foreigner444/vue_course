# Module 4 Project 2: Multi-Step Wizard

## Project Overview

| Field | Value |
|:--- |:--- |
| **Module** | Module 4: Rendering Patterns |
| **Project** | Multi-Step Wizard |
| **Difficulty** | Intermediate |
| **Estimated Time** | 60-75 minutes |
| **Prerequisites** | Lessons 9.1, 9.2, 10.1, 10.2 |

**What You'll Build:** A checkout flow wizard that guides users through multiple form steps with progress indication, step navigation, form validation, and a confirmation summary.

**Real-World Scenario:** You're building the checkout process for an e-commerce platform. Customers need to enter shipping information, select a payment method, review their order, and confirm. Each step should be clearly indicated, and users should be able to navigate back to previous steps to make changes.

---

## Patterns You'll Practice

| Pattern | Where Used |
|:--- |:--- |
| `v-if` / `v-else-if` / `v-else` | Showing different form steps |
| Conditional classes (`:class`) | Active/completed step indicators |
| `computed()` | Step validation, progress calculation |
| `ref()` with objects | Form data management |
| `v-for` | Rendering step indicators, form fields |
| `v-show` | Error messages, optional sections |

---

## Component Structure

```
src/
├── components/
│   ├── CheckoutWizard.vue     # Main wizard container
│   ├── StepIndicator.vue      # Progress steps UI
│   ├── steps/
│   │   ├── ShippingStep.vue   # Step 1: Shipping info
│   │   ├── PaymentStep.vue    # Step 2: Payment method
│   │   ├── ReviewStep.vue     # Step 3: Order review
│   │   └── ConfirmationStep.vue # Success state
│   └── WizardNavigation.vue   # Next/Previous buttons
├── types/
│   └── checkout.ts            # TypeScript interfaces
├── App.vue
└── main.ts
```

---

## Step-by-Step Guide

### Step 1: Project Setup

```bash
npm create vue@latest multi-step-wizard
cd multi-step-wizard
npm install -D tailwindcss postcss autoprefixer daisyui@latest
npx tailwindcss init -p
```

Configure Tailwind and DaisyUI (same as Project 1).

---

### Step 2: Define TypeScript Types

Create `src/types/checkout.ts`:

```typescript
export interface ShippingInfo {
  firstName: string
  lastName: string
  email: string
  address: string
  city: string
  zipCode: string
  country: string
}

export interface PaymentInfo {
  method: 'credit' | 'debit' | 'paypal' | ''
  cardNumber: string
  expiryDate: string
  cvv: string
  cardholderName: string
}

export interface OrderItem {
  id: number
  name: string
  price: number
  quantity: number
  image: string
}

export interface CheckoutData {
  shipping: ShippingInfo
  payment: PaymentInfo
  items: OrderItem[]
}

export interface Step {
  id: number
  name: string
  description: string
  icon: string
}
```

---

### Step 3: Build StepIndicator Component

Create `src/components/StepIndicator.vue`:

```vue
<script setup lang="ts">
import type { Step } from '@/types/checkout'

defineProps<{
  steps: Step[]
  currentStep: number
  completedSteps: number[]
}>()

const emit = defineEmits<{
  'go-to-step': [step: number]
}>()
</script>

<template>
  <div class="w-full py-6">
    <ul class="steps steps-horizontal w-full">
      <!--
        TODO: Use v-for to render each step
        Apply these conditional classes:
        - 'step-primary': step.id <= currentStep OR step.id is in completedSteps
        - Make clickable only if step.id is in completedSteps or is currentStep
      -->
      <li
        v-for="step in steps"
        :key="step.id"
        class="step"
        :class="{
          'step-primary': step.id <= currentStep || completedSteps.includes(step.id)
        }"
      >
        <button
          class="flex flex-col items-center gap-1"
          :disabled="step.id > currentStep && !completedSteps.includes(step.id)"
          @click="emit('go-to-step', step.id)"
        >
          <span class="text-2xl">{{ step.icon }}</span>
          <span class="text-sm font-medium">{{ step.name }}</span>
        </button>
      </li>
    </ul>
  </div>
</template>
```

---

### Step 4: Build ShippingStep Component

Create `src/components/steps/ShippingStep.vue`:

```vue
<script setup lang="ts">
import { computed } from 'vue'
import type { ShippingInfo } from '@/types/checkout'

const props = defineProps<{
  modelValue: ShippingInfo
}>()

const emit = defineEmits<{
  'update:modelValue': [value: ShippingInfo]
}>()

const shipping = computed({
  get: () => props.modelValue,
  set: (value) => emit('update:modelValue', value)
})

const updateField = (field: keyof ShippingInfo, value: string) => {
  emit('update:modelValue', { ...props.modelValue, [field]: value })
}

const countries = [
  'United States',
  'Canada',
  'United Kingdom',
  'Germany',
  'France',
  'Australia',
]
</script>

<template>
  <div class="space-y-6">
    <div class="text-center mb-8">
      <h2 class="text-2xl font-bold">Shipping Information</h2>
      <p class="text-base-content/70">Where should we send your order?</p>
    </div>

    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
      <!-- First Name -->
      <div class="form-control">
        <label class="label">
          <span class="label-text">First Name *</span>
        </label>
        <input
          type="text"
          class="input input-bordered"
          :value="shipping.firstName"
          @input="updateField('firstName', ($event.target as HTMLInputElement).value)"
          placeholder="John"
        />
      </div>

      <!-- Last Name -->
      <div class="form-control">
        <label class="label">
          <span class="label-text">Last Name *</span>
        </label>
        <input
          type="text"
          class="input input-bordered"
          :value="shipping.lastName"
          @input="updateField('lastName', ($event.target as HTMLInputElement).value)"
          placeholder="Doe"
        />
      </div>
    </div>

    <!-- Email -->
    <div class="form-control">
      <label class="label">
        <span class="label-text">Email Address *</span>
      </label>
      <input
        type="email"
        class="input input-bordered"
        :value="shipping.email"
        @input="updateField('email', ($event.target as HTMLInputElement).value)"
        placeholder="john@example.com"
      />
    </div>

    <!-- Address -->
    <div class="form-control">
      <label class="label">
        <span class="label-text">Street Address *</span>
      </label>
      <input
        type="text"
        class="input input-bordered"
        :value="shipping.address"
        @input="updateField('address', ($event.target as HTMLInputElement).value)"
        placeholder="123 Main Street"
      />
    </div>

    <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
      <!-- City -->
      <div class="form-control">
        <label class="label">
          <span class="label-text">City *</span>
        </label>
        <input
          type="text"
          class="input input-bordered"
          :value="shipping.city"
          @input="updateField('city', ($event.target as HTMLInputElement).value)"
          placeholder="New York"
        />
      </div>

      <!-- ZIP Code -->
      <div class="form-control">
        <label class="label">
          <span class="label-text">ZIP Code *</span>
        </label>
        <input
          type="text"
          class="input input-bordered"
          :value="shipping.zipCode"
          @input="updateField('zipCode', ($event.target as HTMLInputElement).value)"
          placeholder="10001"
        />
      </div>

      <!-- Country -->
      <div class="form-control">
        <label class="label">
          <span class="label-text">Country *</span>
        </label>
        <select
          class="select select-bordered"
          :value="shipping.country"
          @change="updateField('country', ($event.target as HTMLSelectElement).value)"
        >
          <option value="">Select country</option>
          <option v-for="country in countries" :key="country" :value="country">
            {{ country }}
          </option>
        </select>
      </div>
    </div>
  </div>
</template>
```

---

### Step 5: Build PaymentStep Component

Create `src/components/steps/PaymentStep.vue`:

```vue
<script setup lang="ts">
import { computed } from 'vue'
import type { PaymentInfo } from '@/types/checkout'

const props = defineProps<{
  modelValue: PaymentInfo
}>()

const emit = defineEmits<{
  'update:modelValue': [value: PaymentInfo]
}>()

const payment = computed(() => props.modelValue)

const updateField = (field: keyof PaymentInfo, value: string) => {
  emit('update:modelValue', { ...props.modelValue, [field]: value })
}

const paymentMethods = [
  { id: 'credit', name: 'Credit Card', icon: '💳', description: 'Visa, Mastercard, Amex' },
  { id: 'debit', name: 'Debit Card', icon: '🏦', description: 'Direct from your bank' },
  { id: 'paypal', name: 'PayPal', icon: '🅿️', description: 'Pay with PayPal account' },
]

const isCardPayment = computed(() => 
  payment.value.method === 'credit' || payment.value.method === 'debit'
)
</script>

<template>
  <div class="space-y-6">
    <div class="text-center mb-8">
      <h2 class="text-2xl font-bold">Payment Method</h2>
      <p class="text-base-content/70">How would you like to pay?</p>
    </div>

    <!-- Payment Method Selection -->
    <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
      <!--
        TODO: Use v-for to render payment method cards
        Add 'ring-2 ring-primary' class when method is selected
      -->
      <div
        v-for="method in paymentMethods"
        :key="method.id"
        class="card bg-base-200 cursor-pointer hover:bg-base-300 transition-all"
        :class="{ 'ring-2 ring-primary': payment.method === method.id }"
        @click="updateField('method', method.id)"
      >
        <div class="card-body items-center text-center p-4">
          <span class="text-4xl">{{ method.icon }}</span>
          <h3 class="font-bold">{{ method.name }}</h3>
          <p class="text-sm text-base-content/70">{{ method.description }}</p>
        </div>
      </div>
    </div>

    <!-- Card Details -->
    <!--
      TODO: Use v-show to only display card fields when credit or debit is selected
      v-show is better here because users might switch between methods
    -->
    <div v-show="isCardPayment" class="space-y-4 mt-6">
      <div class="divider">Card Details</div>

      <!-- Cardholder Name -->
      <div class="form-control">
        <label class="label">
          <span class="label-text">Cardholder Name *</span>
        </label>
        <input
          type="text"
          class="input input-bordered"
          :value="payment.cardholderName"
          @input="updateField('cardholderName', ($event.target as HTMLInputElement).value)"
          placeholder="JOHN DOE"
        />
      </div>

      <!-- Card Number -->
      <div class="form-control">
        <label class="label">
          <span class="label-text">Card Number *</span>
        </label>
        <input
          type="text"
          class="input input-bordered"
          :value="payment.cardNumber"
          @input="updateField('cardNumber', ($event.target as HTMLInputElement).value)"
          placeholder="1234 5678 9012 3456"
          maxlength="19"
        />
      </div>

      <div class="grid grid-cols-2 gap-4">
        <!-- Expiry Date -->
        <div class="form-control">
          <label class="label">
            <span class="label-text">Expiry Date *</span>
          </label>
          <input
            type="text"
            class="input input-bordered"
            :value="payment.expiryDate"
            @input="updateField('expiryDate', ($event.target as HTMLInputElement).value)"
            placeholder="MM/YY"
            maxlength="5"
          />
        </div>

        <!-- CVV -->
        <div class="form-control">
          <label class="label">
            <span class="label-text">CVV *</span>
          </label>
          <input
            type="text"
            class="input input-bordered"
            :value="payment.cvv"
            @input="updateField('cvv', ($event.target as HTMLInputElement).value)"
            placeholder="123"
            maxlength="4"
          />
        </div>
      </div>
    </div>

    <!-- PayPal Message -->
    <!--
      TODO: Use v-if to show PayPal redirect message only when PayPal is selected
    -->
    <div v-if="payment.method === 'paypal'" class="alert alert-info mt-6">
      <span>🅿️ You will be redirected to PayPal to complete your payment after reviewing your order.</span>
    </div>
  </div>
</template>
```

---

### Step 6: Build ReviewStep Component

Create `src/components/steps/ReviewStep.vue`:

```vue
<script setup lang="ts">
import { computed } from 'vue'
import type { CheckoutData } from '@/types/checkout'

const props = defineProps<{
  checkoutData: CheckoutData
}>()

const subtotal = computed(() =>
  props.checkoutData.items.reduce((sum, item) => sum + item.price * item.quantity, 0)
)

const shipping = computed(() => (subtotal.value > 100 ? 0 : 9.99))
const tax = computed(() => subtotal.value * 0.08)
const total = computed(() => subtotal.value + shipping.value + tax.value)

const paymentMethodNames: Record<string, string> = {
  credit: 'Credit Card',
  debit: 'Debit Card',
  paypal: 'PayPal',
}

const maskedCardNumber = computed(() => {
  const num = props.checkoutData.payment.cardNumber.replace(/\s/g, '')
  if (num.length < 4) return num
  return '•••• •••• •••• ' + num.slice(-4)
})
</script>

<template>
  <div class="space-y-6">
    <div class="text-center mb-8">
      <h2 class="text-2xl font-bold">Review Your Order</h2>
      <p class="text-base-content/70">Please confirm your details before placing the order</p>
    </div>

    <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
      <!-- Shipping Summary -->
      <div class="card bg-base-200">
        <div class="card-body">
          <h3 class="card-title text-lg">
            📦 Shipping Address
          </h3>
          <div class="text-base-content/80">
            <p class="font-medium">
              {{ checkoutData.shipping.firstName }} {{ checkoutData.shipping.lastName }}
            </p>
            <p>{{ checkoutData.shipping.address }}</p>
            <p>
              {{ checkoutData.shipping.city }}, {{ checkoutData.shipping.zipCode }}
            </p>
            <p>{{ checkoutData.shipping.country }}</p>
            <p class="mt-2 text-sm">{{ checkoutData.shipping.email }}</p>
          </div>
        </div>
      </div>

      <!-- Payment Summary -->
      <div class="card bg-base-200">
        <div class="card-body">
          <h3 class="card-title text-lg">
            💳 Payment Method
          </h3>
          <div class="text-base-content/80">
            <p class="font-medium">
              {{ paymentMethodNames[checkoutData.payment.method] || 'Not selected' }}
            </p>
            <!--
              TODO: Use v-if to only show card details for credit/debit payments
            -->
            <template v-if="checkoutData.payment.method === 'credit' || checkoutData.payment.method === 'debit'">
              <p>{{ maskedCardNumber }}</p>
              <p>{{ checkoutData.payment.cardholderName }}</p>
              <p>Expires: {{ checkoutData.payment.expiryDate }}</p>
            </template>
            <p v-else-if="checkoutData.payment.method === 'paypal'" class="text-info">
              You will be redirected to PayPal
            </p>
          </div>
        </div>
      </div>
    </div>

    <!-- Order Items -->
    <div class="card bg-base-200">
      <div class="card-body">
        <h3 class="card-title text-lg">🛒 Order Items</h3>
        
        <div class="divide-y divide-base-300">
          <!--
            TODO: Use v-for to render each order item
          -->
          <div
            v-for="item in checkoutData.items"
            :key="item.id"
            class="flex items-center gap-4 py-4"
          >
            <span class="text-4xl">{{ item.image }}</span>
            <div class="flex-1">
              <p class="font-medium">{{ item.name }}</p>
              <p class="text-sm text-base-content/70">Qty: {{ item.quantity }}</p>
            </div>
            <p class="font-bold">${{ (item.price * item.quantity).toFixed(2) }}</p>
          </div>
        </div>

        <!-- Order Totals -->
        <div class="divider"></div>
        <div class="space-y-2">
          <div class="flex justify-between">
            <span>Subtotal</span>
            <span>${{ subtotal.toFixed(2) }}</span>
          </div>
          <div class="flex justify-between">
            <span>Shipping</span>
            <span>
              <template v-if="shipping === 0">
                <span class="text-success">FREE</span>
              </template>
              <template v-else>
                ${{ shipping.toFixed(2) }}
              </template>
            </span>
          </div>
          <div class="flex justify-between">
            <span>Tax (8%)</span>
            <span>${{ tax.toFixed(2) }}</span>
          </div>
          <div class="divider my-2"></div>
          <div class="flex justify-between text-xl font-bold">
            <span>Total</span>
            <span class="text-primary">${{ total.toFixed(2) }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
```

---

### Step 7: Build ConfirmationStep Component

Create `src/components/steps/ConfirmationStep.vue`:

```vue
<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{
  orderNumber: string
  email: string
}>()

const emit = defineEmits<{
  'start-new': []
}>()

const estimatedDelivery = computed(() => {
  const date = new Date()
  date.setDate(date.getDate() + 5)
  return date.toLocaleDateString('en-US', { 
    weekday: 'long', 
    month: 'long', 
    day: 'numeric' 
  })
})
</script>

<template>
  <div class="text-center space-y-6 py-8">
    <div class="text-8xl">🎉</div>
    
    <div>
      <h2 class="text-3xl font-bold text-success">Order Confirmed!</h2>
      <p class="text-base-content/70 mt-2">
        Thank you for your purchase
      </p>
    </div>

    <div class="card bg-base-200 max-w-md mx-auto">
      <div class="card-body">
        <div class="space-y-4">
          <div>
            <p class="text-sm text-base-content/70">Order Number</p>
            <p class="text-xl font-mono font-bold">{{ orderNumber }}</p>
          </div>
          
          <div class="divider my-0"></div>
          
          <div>
            <p class="text-sm text-base-content/70">Confirmation sent to</p>
            <p class="font-medium">{{ email }}</p>
          </div>
          
          <div>
            <p class="text-sm text-base-content/70">Estimated Delivery</p>
            <p class="font-medium">{{ estimatedDelivery }}</p>
          </div>
        </div>
      </div>
    </div>

    <div class="flex flex-col sm:flex-row gap-4 justify-center mt-8">
      <button class="btn btn-primary" @click="emit('start-new')">
        Start New Order
      </button>
      <button class="btn btn-outline">
        Track Order
      </button>
    </div>
  </div>
</template>
```

---

### Step 8: Build WizardNavigation Component

Create `src/components/WizardNavigation.vue`:

```vue
<script setup lang="ts">
defineProps<{
  currentStep: number
  totalSteps: number
  canGoNext: boolean
  isLastStep: boolean
  isSubmitting: boolean
}>()

const emit = defineEmits<{
  previous: []
  next: []
  submit: []
}>()
</script>

<template>
  <div class="flex justify-between items-center pt-6 border-t border-base-300">
    <!-- Previous Button -->
    <button
      class="btn btn-outline"
      :class="{ invisible: currentStep === 1 }"
      :disabled="currentStep === 1"
      @click="emit('previous')"
    >
      ← Previous
    </button>

    <!-- Step Counter -->
    <span class="text-sm text-base-content/70">
      Step {{ currentStep }} of {{ totalSteps }}
    </span>

    <!-- Next/Submit Button -->
    <!--
      TODO: Use v-if/v-else to show different buttons
      - If NOT last step: show "Next" button
      - If last step: show "Place Order" button with loading state
    -->
    <button
      v-if="!isLastStep"
      class="btn btn-primary"
      :disabled="!canGoNext"
      @click="emit('next')"
    >
      Next →
    </button>
    
    <button
      v-else
      class="btn btn-success"
      :disabled="!canGoNext || isSubmitting"
      @click="emit('submit')"
    >
      <span v-if="isSubmitting" class="loading loading-spinner loading-sm"></span>
      <span v-else>Place Order ✓</span>
    </button>
  </div>
</template>
```

---

### Step 9: Build Main CheckoutWizard Component

Create `src/components/CheckoutWizard.vue`:

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'
import type { Step, CheckoutData, ShippingInfo, PaymentInfo } from '@/types/checkout'
import StepIndicator from './StepIndicator.vue'
import WizardNavigation from './WizardNavigation.vue'
import ShippingStep from './steps/ShippingStep.vue'
import PaymentStep from './steps/PaymentStep.vue'
import ReviewStep from './steps/ReviewStep.vue'
import ConfirmationStep from './steps/ConfirmationStep.vue'

const steps: Step[] = [
  { id: 1, name: 'Shipping', description: 'Delivery address', icon: '📦' },
  { id: 2, name: 'Payment', description: 'Payment method', icon: '💳' },
  { id: 3, name: 'Review', description: 'Confirm order', icon: '✅' },
]

const currentStep = ref<number>(1)
const completedSteps = ref<number[]>([])
const isSubmitting = ref<boolean>(false)
const isOrderComplete = ref<boolean>(false)
const orderNumber = ref<string>('')

const checkoutData = ref<CheckoutData>({
  shipping: {
    firstName: '',
    lastName: '',
    email: '',
    address: '',
    city: '',
    zipCode: '',
    country: '',
  },
  payment: {
    method: '',
    cardNumber: '',
    expiryDate: '',
    cvv: '',
    cardholderName: '',
  },
  items: [
    { id: 1, name: 'Wireless Headphones', price: 149.99, quantity: 1, image: '🎧' },
    { id: 2, name: 'Phone Case', price: 29.99, quantity: 2, image: '📱' },
    { id: 3, name: 'USB-C Cable', price: 19.99, quantity: 3, image: '🔌' },
  ],
})

const isShippingValid = computed(() => {
  const s = checkoutData.value.shipping
  return (
    s.firstName.trim() !== '' &&
    s.lastName.trim() !== '' &&
    s.email.trim() !== '' &&
    s.email.includes('@') &&
    s.address.trim() !== '' &&
    s.city.trim() !== '' &&
    s.zipCode.trim() !== '' &&
    s.country !== ''
  )
})

const isPaymentValid = computed(() => {
  const p = checkoutData.value.payment
  if (!p.method) return false
  if (p.method === 'paypal') return true
  return (
    p.cardholderName.trim() !== '' &&
    p.cardNumber.replace(/\s/g, '').length >= 13 &&
    p.expiryDate.length === 5 &&
    p.cvv.length >= 3
  )
})

const canGoNext = computed(() => {
  switch (currentStep.value) {
    case 1:
      return isShippingValid.value
    case 2:
      return isPaymentValid.value
    case 3:
      return true
    default:
      return false
  }
})

const isLastStep = computed(() => currentStep.value === steps.length)

const goToStep = (step: number) => {
  if (step <= currentStep.value || completedSteps.value.includes(step)) {
    currentStep.value = step
  }
}

const goToPrevious = () => {
  if (currentStep.value > 1) {
    currentStep.value--
  }
}

const goToNext = () => {
  if (canGoNext.value && currentStep.value < steps.length) {
    if (!completedSteps.value.includes(currentStep.value)) {
      completedSteps.value.push(currentStep.value)
    }
    currentStep.value++
  }
}

const submitOrder = async () => {
  isSubmitting.value = true
  
  await new Promise(resolve => setTimeout(resolve, 2000))
  
  orderNumber.value = 'ORD-' + Math.random().toString(36).substring(2, 10).toUpperCase()
  isOrderComplete.value = true
  isSubmitting.value = false
}

const startNewOrder = () => {
  currentStep.value = 1
  completedSteps.value = []
  isOrderComplete.value = false
  orderNumber.value = ''
  checkoutData.value.shipping = {
    firstName: '',
    lastName: '',
    email: '',
    address: '',
    city: '',
    zipCode: '',
    country: '',
  }
  checkoutData.value.payment = {
    method: '',
    cardNumber: '',
    expiryDate: '',
    cvv: '',
    cardholderName: '',
  }
}
</script>

<template>
  <div class="max-w-4xl mx-auto p-6">
    <div class="card bg-base-100 shadow-xl">
      <div class="card-body">
        <!-- Header -->
        <h1 class="text-3xl font-bold text-center mb-2">Checkout</h1>
        
        <!--
          MAIN CONDITIONAL RENDERING LOGIC
          Use v-if to check if order is complete
          Use v-else to show the wizard steps
        -->
        
        <!-- Order Complete State -->
        <ConfirmationStep
          v-if="isOrderComplete"
          :order-number="orderNumber"
          :email="checkoutData.shipping.email"
          @start-new="startNewOrder"
        />
        
        <!-- Wizard Steps -->
        <template v-else>
          <!-- Step Indicator -->
          <StepIndicator
            :steps="steps"
            :current-step="currentStep"
            :completed-steps="completedSteps"
            @go-to-step="goToStep"
          />
          
          <!-- Step Content -->
          <!--
            TODO: Use v-if / v-else-if / v-else chain to show correct step
            - Step 1: ShippingStep
            - Step 2: PaymentStep  
            - Step 3: ReviewStep
          -->
          <div class="min-h-[400px]">
            <ShippingStep
              v-if="currentStep === 1"
              v-model="checkoutData.shipping"
            />
            
            <PaymentStep
              v-else-if="currentStep === 2"
              v-model="checkoutData.payment"
            />
            
            <ReviewStep
              v-else
              :checkout-data="checkoutData"
            />
          </div>
          
          <!-- Navigation -->
          <WizardNavigation
            :current-step="currentStep"
            :total-steps="steps.length"
            :can-go-next="canGoNext"
            :is-last-step="isLastStep"
            :is-submitting="isSubmitting"
            @previous="goToPrevious"
            @next="goToNext"
            @submit="submitOrder"
          />
        </template>
      </div>
    </div>
  </div>
</template>
```

---

### Step 10: Update App.vue

```vue
<script setup lang="ts">
import CheckoutWizard from './components/CheckoutWizard.vue'
</script>

<template>
  <div class="min-h-screen bg-base-300 py-8">
    <CheckoutWizard />
  </div>
</template>
```

---

## Complete Solution

All components above are already complete solutions. The key patterns demonstrated:

1. **`v-if` / `v-else-if` / `v-else` chain** in `CheckoutWizard.vue` for showing different steps
2. **`v-if` / `v-else`** for order complete vs. wizard view
3. **`v-show`** in `PaymentStep.vue` for toggling card details (better performance for frequent toggles)
4. **`v-for`** for rendering step indicators, payment methods, and order items
5. **Conditional classes** for active/selected states throughout
6. **Computed properties** for form validation and derived calculations

---

## Stretch Goals

1. **Form Validation UI:** Show inline error messages under invalid fields
2. **Address Autocomplete:** Integrate Google Places API for address suggestions
3. **Save Progress:** Store checkout data in localStorage
4. **Animations:** Add slide transitions between steps using `<Transition>`
5. **Coupon Codes:** Add a coupon input with discount calculation
6. **Order History:** Store completed orders and show order history page

---

## Self-Assessment Checklist

- [ ] All components use `<script setup>` with TypeScript
- [ ] Step indicator shows correct active/completed states
- [ ] Only one step content is visible at a time (`v-if`/`v-else-if`/`v-else`)
- [ ] Card payment details toggle correctly with `v-show`
- [ ] PayPal message only shows when PayPal is selected (`v-if`)
- [ ] Navigation buttons appear/hide correctly based on step
- [ ] Form validation prevents advancing with empty required fields
- [ ] Review step displays all entered information
- [ ] Order items render correctly with `v-for`
- [ ] Order confirmation replaces wizard after successful submission
- [ ] "Start New Order" resets the entire form
- [ ] Step indicators allow navigation to previously completed steps
