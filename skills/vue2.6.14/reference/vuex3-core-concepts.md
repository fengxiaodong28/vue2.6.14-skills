---
title: Vuex 3.x Core Concepts
impact: HIGH
impactDescription: Vuex provides centralized state management with getters, mutations, actions
type: best-practice
tags:
  - vue2
  - vuex
  - state-management
  - getters
  - mutations
  - actions
---

# Vuex 3.x Core Concepts

Impact: HIGH - Vuex provides centralized state management for Vue 2 applications using state, getters, mutations, and actions.

## Task Checklist

- [ ] Define state in store for reactive data
- [ ] Use getters for computed derived state
- [ ] Commit mutations for synchronous state changes
- [ ] Dispatch actions for async operations
- [ ] Never mutate state outside of mutations

## Basic Store Setup

```javascript
// store/index.js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    count: 0,
    user: null,
    todos: []
  },
  getters: {
    doubleCount: state => state.count * 2
  },
  mutations: {
    INCREMENT(state) {
      state.count++
    },
    SET_USER(state, user) {
      state.user = user
    }
  },
  actions: {
    incrementAsync({ commit }) {
      setTimeout(() => {
        commit('INCREMENT')
      }, 1000)
    }
  }
})
```

## State

```javascript
// store/modules/user.js
export default {
  namespaced: true,

  state: {
    currentUser: null,
    isAuthenticated: false,
    token: null,
    permissions: []
  }
}

// Use in component
export default {
  computed: {
    user() {
      return this.$store.state.user.currentUser
    },
    isAuthenticated() {
      return this.$store.state.user.isAuthenticated
    }
  }
}
```

## Getters

```javascript
// store/modules/products.js
export default {
  namespaced: true,

  state: {
    products: [],
    filter: 'all'
  },

  getters: {
    // Basic getter
    productCount: state => state.products.length,

    // Getter with argument (returns function)
    getProductById: state => id => {
      return state.products.find(product => product.id === id)
    },

    // Filtered products
    filteredProducts: (state, getters) => {
      switch (state.filter) {
        case 'active':
          return state.products.filter(p => p.active)
        case 'featured':
          return state.products.filter(p => p.featured)
        default:
          return state.products
      }
    },

    // Using other getters
    activeFeaturedProducts: (state, getters) => {
      return getters.filteredProducts.filter(p => p.featured)
    }
  }
}

// Use in component
export default {
  computed: {
    products() {
      return this.$store.getters['products/filteredProducts']
    },
    productById() {
      return this.$store.getters['products/getProductById']
    }
  },
  methods: {
    getProduct(id) {
      return this.productById(id)
    }
  }
}
```

## Mutations

```javascript
// store/modules/cart.js
export default {
  namespaced: true,

  state: {
    items: [],
    total: 0
  },

  mutations: {
    // Simple mutation
    CLEAR_CART(state) {
      state.items = []
      state.total = 0
    },

    // Mutation with payload
    ADD_ITEM(state, item) {
      state.items.push(item)
    },

    // Mutation with object payload
    UPDATE_QUANTITY(state, { id, quantity }) {
      const item = state.items.find(i => i.id === id)
      if (item) {
        item.quantity = quantity
      }
    },

    // Mutation using Vue.set for reactivity
    SET_ITEM_PROPERTY(state, { id, property, value }) {
      const item = state.items.find(i => i.id === id)
      if (item) {
        Vue.set(item, property, value)
      }
    },

    // Array mutation
    REMOVE_ITEM(state, id) {
      const index = state.items.findIndex(i => i.id === id)
      if (index >= 0) {
        state.items.splice(index, 1)
      }
    }
  }
}

// Commit mutations in component
export default {
  methods: {
    addToCart(product) {
      this.$store.commit('cart/ADD_ITEM', {
        id: product.id,
        name: product.name,
        price: product.price,
        quantity: 1
      })
    },

    updateQuantity(id, quantity) {
      this.$store.commit('cart/UPDATE_QUANTITY', { id, quantity })
    }
  }
}
```

## Actions

```javascript
// store/modules/user.js
import api from '@/api'

export default {
  namespaced: true,

  state: {
    user: null,
    isLoading: false,
    error: null
  },

  mutations: {
    SET_USER(state, user) {
      state.user = user
    },
    SET_LOADING(state, isLoading) {
      state.isLoading = isLoading
    },
    SET_ERROR(state, error) {
      state.error = error
    }
  },

  actions: {
    // Basic async action
    async fetchUser({ commit }, userId) {
      commit('SET_LOADING', true)
      commit('SET_ERROR', null)

      try {
        const response = await api.getUser(userId)
        commit('SET_USER', response.data)
      } catch (error) {
        commit('SET_ERROR', error.message)
        throw error
      } finally {
        commit('SET_LOADING', false)
      }
    },

    // Action with multiple mutations
    async login({ commit, dispatch }, credentials) {
      commit('SET_LOADING', true)

      try {
        const response = await api.login(credentials)
        const { token, user } = response.data

        // Commit local mutations
        commit('SET_USER', user)
        commit('SET_TOKEN', token)

        // Dispatch other module actions
        await dispatch('preferences/fetch', null, { root: true })
      } catch (error) {
        commit('SET_ERROR', error.message)
        throw error
      } finally {
        commit('SET_LOADING', false)
      }
    },

    // Action without API call
    logout({ commit }) {
      commit('SET_USER', null)
      commit('SET_TOKEN', null)
      // Clear local storage
      localStorage.removeItem('token')
    }
  }
}

// Dispatch actions in component
export default {
  methods: {
    async loadUser() {
      try {
        await this.$store.dispatch('user/fetchUser', this.userId)
      } catch (error) {
        this.showError('Failed to load user')
      }
    },

    login() {
      this.$store.dispatch('user/login', {
        username: this.username,
        password: this.password
      })
    }
  }
}
```

## Modules

```javascript
// store/index.js
import Vue from 'vue'
import Vuex from 'vuex'
import user from './modules/user'
import cart from './modules/cart'
import products from './modules/products'

Vue.use(Vuex)

export default new Vuex.Store({
  modules: {
    user,
    cart,
    products
  },
  strict: process.env.NODE_ENV !== 'production'
})

// store/modules/cart.js
export default {
  namespaced: true, // Important for module isolation

  state: {
    items: []
  },

  getters: {
    itemCount: state => state.items.length,
    total: state => state.items.reduce((sum, item) => sum + item.price, 0)
  },

  mutations: {
    ADD_ITEM(state, item) {
      state.items.push(item)
    }
  },

  actions: {
    async checkout({ commit, state }) {
      const response = await api.checkout(state.items)
      commit('CLEAR_CART')
      return response
    }
  }
}

// Access namespaced module
this.$store.state.cart.items
this.$store.getters['cart/itemCount']
this.$store.commit('cart/ADD_ITEM', item)
this.$store.dispatch('cart/checkout')
```

## Store with TypeScript

```typescript
// store/types.ts
export interface RootState {
  user: UserState
  cart: CartState
}

export interface UserState {
  currentUser: User | null
  isAuthenticated: boolean
}

export interface CartState {
  items: CartItem[]
}

export interface User {
  id: number
  name: string
  email: string
}

export interface CartItem {
  id: number
  name: string
  price: number
  quantity: number
}
```

## Map Helpers

```vue
<script>
import { mapState, mapGetters, mapMutations, mapActions } from 'vuex'

export default {
  computed: {
    // Map state as object
    ...mapState('user', {
      user: 'currentUser',
      isLoggedIn: 'isAuthenticated'
    }),

    // Map state as array
    ...mapState('cart', ['items', 'total']),

    // Map getters
    ...mapGetters('products', [
      'filteredProducts',
      'productById'
    ]),

    // Local computed with getter
    expensiveItems() {
      return this.filteredProducts.filter(p => p.price > 100)
    }
  },

  methods: {
    // Map mutations
    ...mapMutations('cart', [
      'ADD_ITEM', // maps to this.ADD_ITEM()
      'REMOVE_ITEM' // maps to this.REMOVE_ITEM()
    ]),

    // Map with custom name
    ...mapMutations('cart', {
      add: 'ADD_ITEM',
      remove: 'REMOVE_ITEM'
    }),

    // Map actions
    ...mapActions('user', [
      'fetchUser',
      'login',
      'logout'
    ]),

    // Local methods
    handleAddToCart(product) {
      this.add(product)
      this.showNotification('Added to cart')
    }
  }
}
</script>
```

## Composition Pattern

```javascript
// Getters that use other modules
getters: {
  cartWithProducts: (state, getters, rootState) => {
    return state.cartItems.map(item => {
      const product = rootState.products.all.find(p => p.id === item.productId)
      return {
        ...item,
        product
      }
    })
  }
}

// Actions that call other module actions
actions: {
  async purchase({ commit, dispatch, rootState }) {
    // Get user from another module
    const userId = rootState.user.currentUser.id

    // Call action in another module
    await dispatch('user/trackPurchase', { cart: state.cart }, { root: true })

    // Commit local mutation
    commit('CLEAR_CART')
  }
}
```

## Store Registration Pattern

```javascript
// Dynamically register module
store.registerModule('dynamic', {
  namespaced: true,
  state: { ... },
  mutations: { ... }
})

// Unregister module
store.unregisterModule('dynamic')
```

## Hot Reload

```javascript
// Enable hot module replacement for development
if (module.hot) {
  module.hot.accept(['./modules/user', './modules/cart'], () => {
    const newUser = require('./modules/user').default
    const newCart = require('./modules/cart').default

    store.hotUpdate({
      modules: {
        user: newUser,
        cart: newCart
      }
    })
  })
}
```

## Reference

- [Vuex 3 Guide - Getting Started](https://vuex.vuejs.org/guide/)
- [Vuex 3 API Reference](https://vuex.vuejs.org/api/)
