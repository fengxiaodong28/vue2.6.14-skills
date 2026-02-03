---
title: Navigation Guards in Vue Router 3.x
impact: HIGH
impactDescription: Navigation guards control route access and navigation flow
type: capability
tags:
  - vue2
  - vue-router
  - navigation-guards
  - route-protection
  - authentication
---

# Navigation Guards in Vue Router 3.x

Impact: HIGH - Navigation guards control route access, protect authenticated routes, and handle navigation logic.

## Task Checklist

- [ ] Use `beforeEnter` for route-specific guards
- [ ] Use `beforeRouteEnter` when component needs data before render
- [ ] Use `beforeRouteUpdate` for same-component parameter changes
- [ ] Use `beforeRouteLeave` for unsaved changes warnings
- [ ] Always call `next()` to continue navigation

## Global Guards

```javascript
// router/index.js
import Vue from 'vue'
import Router from 'vue-router'
import store from '@/store'

Vue.use(Router)

const router = new Router({
  routes: [
    // Your routes here
  ]
})

// Global beforeEach guard
router.beforeEach((to, from, next) => {
  // Check authentication
  const isAuthenticated = store.getters.isAuthenticated

  if (to.meta.requiresAuth && !isAuthenticated) {
    next({
      name: 'login',
      query: { redirect: to.fullPath }
    })
  } else {
    next()
  }
})

// Global beforeResolve guard
router.beforeResolve((to, from, next) => {
  // Called after in-component guards but before navigation confirmed
  next()
})

// Global afterEach hook
router.afterEach((to, from) => {
  // Navigation completed - no next() function
  document.title = to.meta.title || 'My App'

  // Track page view
  if (window.gtag) {
    window.gtag('event', 'page_view', {
      page_path: to.path
    })
  }
})

export default router
```

## Per-Route Guards

```javascript
// router/index.js
export default new Router({
  routes: [
    {
      path: '/admin',
      component: AdminPanel,
      meta: {
        requiresAuth: true,
        requiresAdmin: true
      },
      beforeEnter: (to, from, next) => {
        // Route-specific guard
        const isAdmin = store.getters.isAdmin

        if (!isAdmin) {
          next({ name: 'forbidden' })
        } else {
          next()
        }
      }
    },

    {
      path: '/checkout',
      component: Checkout,
      beforeEnter: (to, from, next) => {
        // Check if cart has items
        if (store.state.cart.items.length === 0) {
          next({ name: 'cart' })
        } else {
          next()
        }
      }
    }
  ]
})
```

## In-Component Guards

```vue
<script>
export default {
  name: 'UserProfile',
  data() {
    return {
      user: null,
      isLoading: false,
      hasChanges: false
    }
  },

  // Called before route that renders this component is confirmed
  beforeRouteEnter(to, from, next) {
    // Cannot access component instance via `this`
    next(vm => {
      // Access component instance through callback
      vm.fetchUserData()
    })
  },

  // Called when route changes but component is reused
  beforeRouteUpdate(to, from, next) {
    // Component already exists
    this.fetchUserData(to.params.id)
    next()
  },

  // Called when navigation away from this route
  beforeRouteLeave(to, from, next) {
    // Prevent navigation with unsaved changes
    if (this.hasChanges) {
      const answer = window.confirm(
        'You have unsaved changes. Are you sure you want to leave?'
      )
      if (answer) {
        next()
      } else {
        next(false)
      }
    } else {
      next()
    }
  },

  methods: {
    async fetchUserData() {
      this.isLoading = true
      try {
        const response = await fetch(`/api/users/${this.$route.params.id}`)
        this.user = await response.json()
      } finally {
        this.isLoading = false
      }
    }
  }
}
</script>
```

## Authentication Guard Pattern

```javascript
// router/index.js
import Vue from 'vue'
import Router from 'vue-router'
import store from '@/store'
import { AuthService } from '@/services/auth'

Vue.use(Router)

const router = new Router({
  mode: 'history',
  routes: [
    {
      path: '/login',
      name: 'login',
      component: () => import('@/views/Login.vue'),
      meta: { guest: true }
    },
    {
      path: '/dashboard',
      name: 'dashboard',
      component: () => import('@/views/Dashboard.vue'),
      meta: { requiresAuth: true }
    },
    {
      path: '/admin',
      name: 'admin',
      component: () => import('@/views/Admin.vue'),
      meta: {
        requiresAuth: true,
        roles: ['admin']
      }
    },
    {
      path: '/profile',
      name: 'profile',
      component: () => import('@/views/Profile.vue'),
      meta: {
        requiresAuth: true,
        roles: ['user', 'admin']
      }
    }
  ]
})

// Auth guard
router.beforeEach(async (to, from, next) => {
  // Check if route requires authentication
  const requiresAuth = to.matched.some(record => record.meta.requiresAuth)
  const isGuestRoute = to.matched.some(record => record.meta.guest)

  // Get current auth state
  const isAuthenticated = await AuthService.checkAuth()

  if (requiresAuth && !isAuthenticated) {
    // Redirect to login with return URL
    next({
      name: 'login',
      query: { redirect: to.fullPath }
    })
  } else if (isGuestRoute && isAuthenticated) {
    // Redirect authenticated users away from guest routes
    next({ name: 'dashboard' })
  } else {
    // Check role-based access
    const requiredRoles = to.meta.roles
    if (requiredRoles) {
      const userRole = store.getters.userRole
      if (!requiredRoles.includes(userRole)) {
        next({ name: 'forbidden' })
        return
      }
    }
    next()
  }
})

export default router
```

## Async Data Fetching Guard

```javascript
// router/index.js
export default new Router({
  routes: [
    {
      path: '/product/:id',
      component: ProductDetail,
      props: true,
      meta: {
        fetchProduct: true
      },
      beforeEnter: async (to, from, next) => {
        try {
          // Fetch data before entering route
          const product = await fetch(`/api/products/${to.params.id}`)
            .then(res => {
              if (!res.ok) throw new Error('Product not found')
              return res.json()
            })

          // Store data for component
          to.params.product = product
          next()
        } catch (error) {
          next({ name: 'not-found' })
        }
      }
    }
  ]
})
```

## Progress Bar on Navigation

```javascript
// router/index.js
import NProgress from 'nprogress'
import 'nprogress/nprogress.css'

router.beforeEach((to, from, next) => {
  NProgress.start()
  next()
})

router.afterEach(() => {
  NProgress.done()
})
```

## Scroll Behavior with Guards

```javascript
// router/index.js
const router = new Router({
  routes: [
    // routes...
  ],
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      return savedPosition
    } else if (to.hash) {
      return {
        selector: to.hash,
        behavior: 'smooth'
      }
    } else {
      return { x: 0, y: 0 }
    }
  }
})
```

## Confirmation Dialog Pattern

```vue
<script>
export default {
  data() {
    return {
      form: {
        name: '',
        email: '',
        message: ''
      },
      isDirty: false
    }
  },

  watch: {
    form: {
      handler() {
        this.isDirty = true
      },
      deep: true
    }
  },

  beforeRouteLeave(to, from, next) {
    if (this.isDirty) {
      this.$dialog
        .confirm('You have unsaved changes. Leave anyway?')
        .then(() => next())
        .catch(() => next(false))
    } else {
      next()
    }
  }
}
</script>
```

## Loading State with Guards

```javascript
// store/modules/route.js
export default {
  namespaced: true,
  state: {
    isLoading: false,
    error: null
  },
  mutations: {
    setLoading(state, isLoading) {
      state.isLoading = isLoading
    },
    setError(state, error) {
      state.error = error
    }
  }
}

// router/index.js
router.beforeEach((to, from, next) => {
  store.commit('route/setLoading', true)
  store.commit('route/setError', null)
  next()
})

router.afterEach(() => {
  store.commit('route/setLoading', false)
})

router.onError((error) => {
  store.commit('route/setLoading', false)
  store.commit('route/setError', error)
})
```

## Redirect Based on Route Params

```javascript
// router/index.js
export default new Router({
  routes: [
    {
      path: '/old-post/:id',
      redirect: to => {
        // Lookup new slug from ID
        const postSlug = postSlugMap[to.params.id]
        return {
          name: 'blog-post',
          params: { slug: postSlug }
        }
      }
    }
  ]
})
```

## Lazy Load Guards

```javascript
// Separate guard file
// guards/auth.js
export function requireAuth(store) {
  return (to, from, next) => {
    if (store.getters.isAuthenticated) {
      next()
    } else {
      next({
        path: '/login',
        query: { redirect: to.fullPath }
      })
    }
  }
}

// guards/role.js
export function requireRole(store, allowedRoles) {
  return (to, from, next) => {
    const userRole = store.getters.userRole
    if (allowedRoles.includes(userRole)) {
      next()
    } else {
      next({ name: 'forbidden' })
    }
  }
}

// Use in routes
import { requireAuth, requireRole } from '@/guards'

export default new Router({
  routes: [
    {
      path: '/admin',
      component: Admin,
      beforeEnter: requireRole(store, ['admin'])
    }
  ]
})
```

## Navigation Failures

```javascript
// Detect and handle navigation failures
this.$router.push({ name: 'user', params: { id: 123 } })
  .catch(failure => {
    if (NavigationDuplicated.isNavigationFailure(failure)) {
      // Ignore duplicate navigation
      return
    }
    // Handle other errors
    console.error('Navigation failed:', failure)
  })
```

## Common Guard Patterns

```javascript
// 1. Feature flag guard
const featureFlagGuard = (flag) => (to, from, next) => {
  if (featureFlags[flag]) {
    next()
  } else {
    next({ name: 'feature-not-available' })
  }
}

// 2. Maintenance mode guard
const maintenanceGuard = (to, from, next) => {
  if (isMaintenanceMode && !isAdminRoute(to)) {
    next({ name: 'maintenance' })
  } else {
    next()
  }
}

// 3. Terms acceptance guard
const termsGuard = (store) => (to, from, next) => {
  if (store.getters.hasAcceptedTerms) {
    next()
  } else if (to.name !== 'terms') {
    next({ name: 'terms', query: { redirect: to.fullPath } })
  } else {
    next()
  }
}

// 4. Email verification guard
const verifiedGuard = (store) => (to, from, next) => {
  const user = store.state.user
  if (user && !user.emailVerified) {
    next({ name: 'verify-email' })
  } else {
    next()
  }
}
```

## Guard Execution Order

```
1. Global beforeEach
2. Route-specific beforeEnter
3. beforeRouteEnter (component)
4. Global beforeResolve
5. Navigation confirmed
6. Global afterEach
7. DOM updates
8. beforeRouteEnter callback (with vm access)
```

## Reference

- [Vue Router 3 - Navigation Guards](https://v3.router.vuejs.org/guide/advanced/navigation-guards.html)
