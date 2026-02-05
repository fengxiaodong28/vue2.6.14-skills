---
title: Lazy Loading Routes in Vue Router 3.x
impact: MEDIUM
impactDescription: Lazy loading routes improves initial load time by code-splitting
type: best-practice
tags:
  - vue2.6.14
  - vue-router
  - lazy-loading
  - code-splitting
  - performance
---

# Lazy Loading Routes in Vue Router 3.x

Impact: MEDIUM - Lazy loading routes with dynamic imports enables code-splitting, reducing initial bundle size and improving load time.

## Task Checklist

- [ ] Use dynamic `import()` for route components
- [ ] Group related routes in same chunk
- [ ] Add loading state for async components
- [ ] Handle component loading errors
- [ ] Use webpack magic comments for chunk naming

## Basic Lazy Loading

```javascript
// router/index.js
import Vue from 'vue'
import Router from 'vue-router'

Vue.use(Router)

function lazyLoad(componentPath) {
  return () => import(`@/views/${componentPath}`).catch(() => {
    // Fallback component if import fails
    return import('@/views/Error.vue')
  })
}

export default new Router({
  routes: [
    {
      path: '/',
      name: 'home',
      component: lazyLoad('Home.vue')
    },
    {
      path: '/about',
      name: 'about',
      component: lazyLoad('About.vue')
    }
  ]
})
```

## Dynamic Import Syntax

```javascript
// ✅ CORRECT: Dynamic import (lazy)
const About = () => import('@/views/About.vue')

// ❌ WRONG: Static import (eager)
import About from '@/views/About.vue'
```

## Named Chunks with Webpack Magic Comments

```javascript
// router/index.js
export default new Router({
  routes: [
    // Single chunk
    {
      path: '/about',
      component: () => import(/* webpackChunkName: "about" */ '@/views/About.vue')
    },

    // Group routes into same chunk
    {
      path: '/user/profile',
      component: () => import(/* webpackChunkName: "user" */ '@/views/user/Profile.vue')
    },
    {
      path: '/user/settings',
      component: () => import(/* webpackChunkName: "user" */ '@/views/user/Settings.vue')
    },

    // Admin routes grouped
    {
      path: '/admin/users',
      component: () => import(/* webpackChunkName: "admin" */ '@/views/admin/Users.vue')
    },
    {
      path: '/admin/content',
      component: () => import(/* webpackChunkName: "admin" */ '@/views/admin/Content.vue')
    }
  ]
})
```

## Component-Level Lazy Loading

```vue
<!-- In component template -->
<template>
  <div>
    <button @click="showModal = true">Open Modal</button>
    <LazyModal v-if="showModal" @close="showModal = false" />
  </div>
</template>

<script>
export default {
  components: {
    LazyModal: () => import('@/components/Modal.vue')
  },
  data() {
    return {
      showModal: false
    }
  }
}
</script>
```

## Async Component with Loading State

```vue
<script>
import LoadingComponent from '@/components/Loading.vue'
import ErrorComponent from '@/components/Error.vue'

export default {
  components: {
    MyAsyncComponent: () => ({
      component: import('@/components/HeavyComponent.vue'),
      loading: LoadingComponent,
      error: ErrorComponent,
      delay: 200,    // Delay before showing loading
      timeout: 10000 // Timeout before showing error
    })
  }
}
</script>
```

## Grouped Route Configuration

```javascript
// router/index.js
const lazyLoad = (path) => () => import(/* webpackChunkName: "[request]" */ `@/views/${path}.vue`)

export default new Router({
  routes: [
    {
      path: '/',
      component: lazyLoad('Home')
    },
    {
      path: '/auth',
      children: [
        {
          path: 'login',
          component: lazyLoad('auth/Login')
        },
        {
          path: 'register',
          component: lazyLoad('auth/Register')
        },
        {
          path: 'forgot-password',
          component: lazyLoad('auth/ForgotPassword')
        }
      ]
    }
  ]
})
```

## Prefetching Routes

```javascript
// Prefetch on hover or visible
export default new Router({
  routes: [
    {
      path: '/dashboard',
      component: () => import(/* webpackPrefetch: true */ '@/views/Dashboard.vue')
    },
    {
      path: '/reports',
      component: () => import(/* webpackPrefetch: true */ '@/views/Reports.vue')
    }
  ]
})
```

```vue
<!-- Manual prefetch on link hover -->
<template>
  <router-link
    to="/dashboard"
    @native.mouseenter="prefetchDashboard"
  >
    Dashboard
  </router-link>
</template>

<script>
export default {
  methods: {
    prefetchDashboard() {
      import('@/views/Dashboard.vue')
    }
  }
}
</script>
```

## Preload Critical Routes

```javascript
export default new Router({
  routes: [
    {
      path: '/checkout',
      component: () => import(/* webpackPreload: true */ '@/views/Checkout.vue')
    }
  ]
})
```

## Error Handling for Lazy Routes

```javascript
const lazyLoadWithError = (componentPath) => {
  return () => import(`@/views/${componentPath}`)
    .catch(() => {
      // Log error
      console.error(`Failed to load component: ${componentPath}`)
      // Return error component
      return import('@/views/RouteError.vue')
    })
}

export default new Router({
  routes: [
    {
      path: '/feature',
      component: lazyLoadWithError('Feature')
    }
  ]
})
```

## Bundle Analysis

```bash
# Install bundle analyzer
npm install --save-dev webpack-bundle-analyzer

# Add to vue.config.js
```

```javascript
// vue.config.js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin

module.exports = {
  configureWebpack: {
    plugins: [
      new BundleAnalyzerPlugin({
        analyzerMode: 'static',
        openAnalyzer: false,
        reportFilename: 'bundle-report.html'
      })
    ]
  }
}
```

```bash
# Build and analyze
npm run build

# View the report
open bundle-report.html
```

## Chunk Splitting Strategy

```javascript
// webpack config in vue.config.js
module.exports = {
  configureWebpack: {
    optimization: {
      splitChunks: {
        chunks: 'all',
        maxSize: 244 * 1024, // 244 KB max chunk size
        cacheGroups: {
          vendor: {
            test: /[\\/]node_modules[\\/]/,
            name: 'vendors',
            priority: 10
          },
          common: {
            minChunks: 2,
            priority: 5,
            reuseExistingChunk: true
          }
        }
      }
    }
  }
}
```

## Route-Based Code Splitting Pattern

```javascript
// router/routes.js
export const routes = [
  {
    path: '/',
    name: 'home',
    chunkName: 'home',
    component: 'Home'
  },
  {
    path: '/products',
    name: 'products',
    chunkName: 'products',
    component: 'Products'
  },
  {
    path: '/product/:id',
    name: 'product-detail',
    chunkName: 'products',
    component: 'ProductDetail'
  }
]

// router/index.js
const lazyLoad = (component, chunkName) => () => import(
  /* webpackChunkName: "[request]" */
  `@/views/${component}.vue`
)

export default new Router({
  routes: routes.map(route => ({
    ...route,
    component: lazyLoad(route.component, route.chunkName)
  }))
})
```

## Progressive Web App Pattern

```javascript
// Register service worker for offline support
if (process.env.NODE_ENV === 'production') {
  registerServiceWorker()
}

// Pre-cache critical chunks
workbox.precaching.precacheAndRoute([
  '/app.js',
  '/home.js',
  '/vendors.js'
])
```

## Loading Indicator

```vue
<!-- App.vue -->
<template>
  <div id="app">
    <div v-if="isLoading" class="loading">
      <ProgressLoader />
    </div>
    <router-view v-else />
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      isLoading: false
    }
  },
  created() {
    // Show loading on route change
    this.$router.beforeEach((to, from, next) => {
      this.isLoading = true
      next()
    })

    this.$router.afterEach(() => {
      this.isLoading = false
    })

    this.$router.onError((error) => {
      this.isLoading = false
      console.error('Route error:', error)
    })
  }
}
</script>
```

## Bundle Size Optimization

```javascript
// vue.config.js
module.exports = {
  chainWebpack: config => {
    // Remove moment.js locales if not needed
    config.plugin('ignore')
      .use(new webpack.IgnorePlugin(/^\.\/locale$/, /moment$/))

    // Use production builds
    config.resolve.alias.set('vue$', 'vue/dist/vue.runtime.esm.js')
  }
}
```

## Common Chunk Patterns

```javascript
// 1. Feature-based chunks
{
  path: '/checkout',
  component: () => import(/* webpackChunkName: "checkout" */ '@/views/Checkout.vue')
}

// 2. User account chunks
{
  path: '/profile',
  component: () => import(/* webpackChunkName: "account" */ '@/views/Profile.vue')
}

// 3. Admin chunks
{
  path: '/admin/dashboard',
  component: () => import(/* webpackChunkName: "admin" */ '@/views/admin/Dashboard.vue')
}

// 4. Report chunks (usually heavy)
{
  path: '/reports/analytics',
  component: () => import(/* webpackChunkName: "reports" */ '@/views/reports/Analytics.vue')
}
```

## Lazy Loading Best Practices

| Pattern | Description | Use When |
|---------|-------------|----------|
| Named chunks | Group related routes | Routes share functionality |
| Prefetch | Load before user navigates | High-probability next page |
| Preload | Load with high priority | Critical but off-critical-path |
| Error component | Fallback for failed loads | Flaky network or large bundles |
| Loading delay | Show loading after delay | Fast networks, avoid flash |

## Reference

- [Vue Router 3 - Lazy Loading](https://v3.router.vuejs.org/guide/advanced/lazy-loading.html)
- [Webpack - Code Splitting](https://webpack.js.org/guides/code-splitting/)
