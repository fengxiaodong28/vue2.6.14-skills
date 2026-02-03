---
title: Vuex Integration with Composition API for Vue 2
impact: MEDIUM
impactDescription: Using Vuex store with Composition API composables
type: best-practice
tags:
  - vue2
  - composition-api
  - vuex
  - store
  - state-management
---

# Vuex Integration with Composition API for Vue 2

Impact: MEDIUM - Create composables to integrate Vuex store with Composition API in Vue 2.6.14.

## Task Checklist

- [ ] Create useStore composable for Vuex access
- [ ] Use computed for state getters
- [ ] Use methods for mutations and actions
- [ ] Create typed store accessors for TypeScript
- [ ] Create composables for module-specific access

## Basic useStore Composable

```javascript
// composables/useStore.js
import { computed } from '@vue/composition-api'
import store from '@/store' // Import Vuex store instance

export function useStore() {
  // Access state
  const count = computed(() => store.state.count)
  const user = computed(() => store.state.user)

  // Access getters
  const doubleCount = computed(() => store.getters.doubleCount)
  const isAuthenticated = computed(() => store.getters.isAuthenticated)

  // Mutations
  const increment = () => store.commit('increment')
  const setUser = (user) => store.commit('SET_USER', user)

  // Actions
  const asyncIncrement = () => store.dispatch('asyncIncrement')
  const fetchUser = (id) => store.dispatch('fetchUser', id)

  return {
    // State
    count,
    user,

    // Getters
    doubleCount,
    isAuthenticated,

    // Mutations
    increment,
    setUser,

    // Actions
    asyncIncrement,
    fetchUser
  }
}
```

## Type-Safe Store Accessor (TypeScript)

```typescript
// composables/useStore.ts
import { computed } from '@vue/composition-api'
import { Store } from 'vuex'
import store from '@/store'

// Define store state interface
interface RootState {
  count: number
  user: User | null
  loading: boolean
}

interface User {
  id: number
  name: string
  email: string
}

// Create typed store accessor
export function useStore() {
  // State with typing
  const count = computed<number>(() => store.state.count)
  const user = computed<User | null>(() => store.state.user)
  const loading = computed<boolean>(() => store.state.loading)

  // Getters with typing
  const doubleCount = computed<number>(() => store.getters.doubleCount)
  const isAuthenticated = computed<boolean>(() => store.getters['auth/isAuthenticated'])

  // Mutations
  const increment = () => store.commit('increment')
  const setUser = (user: User) => store.commit('SET_USER', user)
  const setLoading = (loading: boolean) => store.commit('SET_LOADING', loading)

  // Actions
  const asyncIncrement = async () => {
    await store.dispatch('asyncIncrement')
  }

  const fetchUser = async (id: number): Promise<void> => {
    await store.dispatch('fetchUser', id)
  }

  return {
    count,
    user,
    loading,
    doubleCount,
    isAuthenticated,
    increment,
    setUser,
    setLoading,
    asyncIncrement,
    fetchUser
  }
}
```

## Module-Specific Composables

```javascript
// composables/useAuthStore.js
import { computed } from '@vue/composition-api'
import store from '@/store'

export function useAuthStore() {
  // State
  const token = computed(() => store.state.auth.token)
  const user = computed(() => store.state.auth.user)
  const loading = computed(() => store.state.auth.loading)

  // Getters
  const isAuthenticated = computed(() => store.getters['auth/isAuthenticated'])
  const userRole = computed(() => store.getters['auth/userRole'])
  const hasPermission = (permission) => {
    return store.getters['auth/hasPermission'](permission)
  }

  // Mutations
  const setToken = (token) => store.commit('auth/SET_TOKEN', token)
  const setUser = (user) => store.commit('auth/SET_USER', user)
  const setLoading = (loading) => store.commit('auth/SET_LOADING', loading)

  // Actions
  const login = async (credentials) => {
    await store.dispatch('auth/login', credentials)
  }

  const logout = async () => {
    await store.dispatch('auth/logout')
  }

  const fetchUser = async () => {
    await store.dispatch('auth/fetchUser')
  }

  const refreshToken = async () => {
    await store.dispatch('auth/refreshToken')
  }

  return {
    // State
    token,
    user,
    loading,

    // Getters
    isAuthenticated,
    userRole,
    hasPermission,

    // Mutations
    setToken,
    setUser,
    setLoading,

    // Actions
    login,
    logout,
    fetchUser,
    refreshToken
  }
}
```

```javascript
// composables/useCartStore.js
import { computed } from '@vue/composition-api'
import store from '@/store'

export function useCartStore() {
  // State
  const items = computed(() => store.state.cart.items)
  const total = computed(() => store.state.cart.total)

  // Getters
  const itemCount = computed(() => store.getters['cart/itemCount'])
  const subtotal = computed(() => store.getters['cart/subtotal'])
  const tax = computed(() => store.getters['cart/tax'])
  const grandTotal = computed(() => store.getters['cart/grandTotal'])

  // Mutations
  const addItem = (item) => store.commit('cart/ADD_ITEM', item)
  const removeItem = (itemId) => store.commit('cart/REMOVE_ITEM', itemId)
  const updateQuantity = (itemId, quantity) => {
    store.commit('cart/UPDATE_QUANTITY', { itemId, quantity })
  }
  const clearCart = () => store.commit('cart/CLEAR_CART')

  // Actions
  const fetchCart = async () => {
    await store.dispatch('cart/fetchCart')
  }

  const addToCart = async (productId, quantity = 1) => {
    await store.dispatch('cart/addToCart', { productId, quantity })
  }

  const removeFromCart = async (itemId) => {
    await store.dispatch('cart/removeFromCart', itemId)
  }

  const updateItemQuantity = async (itemId, quantity) => {
    await store.dispatch('cart/updateQuantity', { itemId, quantity })
  }

  const checkout = async (paymentDetails) => {
    await store.dispatch('cart/checkout', paymentDetails)
  }

  return {
    // State
    items,
    total,

    // Getters
    itemCount,
    subtotal,
    tax,
    grandTotal,

    // Mutations
    addItem,
    removeItem,
    updateQuantity,
    clearCart,

    // Actions
    fetchCart,
    addToCart,
    removeFromCart,
    updateItemQuantity,
    checkout
  }
}
```

## Form State with Vuex

```javascript
// composables/useFormState.js
import { ref, watch, onMounted } from '@vue/composition-api'
import store from '@/store'

export function useFormState(module, initialState = {}) {
  // Local form state
  const formData = ref({ ...initialState })
  const errors = ref({})
  const touched = ref({})

  // Initialize from store
  onMounted(() => {
    if (store.state[module]) {
      formData.value = { ...store.state[module] }
    }
  })

  // Watch for external store changes
  watch(
    () => store.state[module],
    (storeState) => {
      if (storeState && !Object.keys(touched.value).length) {
        formData.value = { ...storeState }
      }
    },
    { deep: true }
  )

  // Update store on form changes
  const updateField = (field, value) => {
    formData.value[field] = value
    touched.value[field] = true
    store.commit(`${module}/UPDATE_FIELD`, { field, value })
  }

  // Save to store
  const save = () => {
    store.commit(`${module}/SET_STATE`, formData.value)
  }

  // Reset form
  const reset = () => {
    formData.value = { ...initialState }
    errors.value = {}
    touched.value = {}
  }

  // Sync with store action
  const syncWithStore = async () => {
    try {
      await store.dispatch(`${module}/sync`, formData.value)
    } catch (error) {
      errors.value = error.response?.data?.errors || {}
      throw error
    }
  }

  return {
    formData,
    errors,
    touched,
    updateField,
    save,
    reset,
    syncWithStore
  }
}
```

## Pagination with Vuex

```javascript
// composables/usePagination.js
import { ref, computed } from '@vue/composition-api'
import store from '@/store'

export function usePagination(module = 'users') {
  const page = ref(1)
  const pageSize = ref(20)

  // Get from store
  const items = computed(() => store.state[module].items)
  const total = computed(() => store.state[module].total)
  const loading = computed(() => store.state[module].loading)

  // Computed
  const totalPages = computed(() => Math.ceil(total.value / pageSize.value))

  // Methods
  const fetchPage = async (pageNum = page.value) => {
    page.value = pageNum
    await store.dispatch(`${module}/fetchPage`, {
      page: page.value,
      pageSize: pageSize.value
    })
  }

  const nextPage = async () => {
    if (page.value < totalPages.value) {
      await fetchPage(page.value + 1)
    }
  }

  const prevPage = async () => {
    if (page.value > 1) {
      await fetchPage(page.value - 1)
    }
  }

  const goToPage = async (pageNum) => {
    if (pageNum >= 1 && pageNum <= totalPages.value) {
      await fetchPage(pageNum)
    }
  }

  const refresh = () => fetchPage(page.value)

  // Load initial page
  const init = () => fetchPage(1)

  return {
    page,
    pageSize,
    items,
    total,
    loading,
    totalPages,
    fetchPage,
    nextPage,
    prevPage,
    goToPage,
    refresh,
    init
  }
}
```

## Usage in Component

```vue
<template>
  <div>
    <h1>User Profile</h1>
    <div v-if="loading">Loading...</div>
    <div v-else-if="user">
      <p>Name: {{ user.name }}</p>
      <p>Email: {{ user.email }}</p>
      <button @click="handleLogout">Logout</button>
    </div>
    <div v-else>
      <p>Please login</p>
    </div>
  </div>
</template>

<script>
import { useAuthStore } from '@/composables/useAuthStore'

export default {
  setup() {
    const {
      user,
      loading,
      isAuthenticated,
      logout
    } = useAuthStore()

    const handleLogout = async () => {
      await logout()
    }

    return {
      user,
      loading,
      isAuthenticated,
      handleLogout
    }
  }
}
</script>
```

## Dynamic Module Registration

```javascript
// composables/useDynamicModule.js
import { onUnmounted } from '@vue/composition-api'
import store from '@/store'

export function useDynamicModule(moduleName, module) {
  // Register module
  if (!store.hasModule(moduleName)) {
    store.registerModule(moduleName, module)
  }

  // Unregister on unmount
  onUnmounted(() => {
    if (store.hasModule(moduleName)) {
      store.unregisterModule(moduleName)
    }
  })

  return store
}

// Usage
export default {
  setup() {
    const store = useDynamicModule('tempData', {
      namespaced: true,
      state: {
        items: []
      },
      mutations: {
        addItem(state, item) {
          state.items.push(item)
        }
      }
    })

    const addItem = (item) => {
      store.commit('tempData/addItem', item)
    }

    return {
      addItem
    }
  }
}
```

## Store Watcher Composable

```javascript
// composables/useStoreWatcher.js
import { watch, onUnmounted } from '@vue/composition-api'
import store from '@/store'

export function useStoreWatcher(getter, callback, options = {}) {
  // Create watcher for store getter
  const unwatch = watch(
    () => getter(store),
    (newValue, oldValue) => {
      callback(newValue, oldValue)
    },
    options
  )

  // Clean up
  onUnmounted(() => {
    unwatch()
  })

  return unwatch
}

// Usage
export default {
  setup() {
    useStoreWatcher(
      (store) => store.state.user,
      (user) => {
        console.log('User changed:', user)
      },
      { immediate: true }
    )
  }
}
```

## Reference

- [Vuex 3 Documentation](https://vuex.vuejs.org/)
- [@vue/composition-api GitHub](https://github.com/vuejs/composition-api)
