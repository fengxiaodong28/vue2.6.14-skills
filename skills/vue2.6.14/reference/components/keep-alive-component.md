---
title: keep-alive Built-in Component in Vue 2
impact: MEDIUM
impactDescription: keep-alive preserves component state and avoids re-rendering when switching between dynamic components or routes
type: capability
tags:
  - vue2.6.14
  - keep-alive
  - built-in-components
  - component-caching
  - performance
  - state-preservation
---

# keep-alive Built-in Component in Vue 2

Impact: MEDIUM - The `<keep-alive>` component preserves component state and avoids re-rendering when switching between dynamic components or routes. It caches component instances instead of destroying them, improving performance and maintaining user input/scroll position.

## Task Checklist

- [ ] Use `<keep-alive>` to preserve component state when toggling
- [ ] Use `include`/`exclude` props to control which components are cached
- [ ] Use `max` prop to limit cache size
- [ ] Access cached lifecycle hooks (`activated`, `deactivated`)
- [ ] Understand when NOT to use keep-alive (memory-intensive components)

## Basic Usage

```vue
<template>
  <div>
    <button @click="currentView = 'home'">Home</button>
    <button @click="currentView = 'profile'">Profile</button>

    <!-- Components will be cached, not destroyed -->
    <keep-alive>
      <component :is="currentView" />
    </keep-alive>
  </div>
</template>

<script>
export default {
  data() {
    return {
      currentView: 'home'
    }
  },
  components: {
    home: {
      data() {
        return { count: 0 }
      },
      template: `
        <div>
          <h2>Home (count preserved: {{ count }})</h2>
          <button @click="count++">Increment</button>
        </div>
      `
    },
    profile: {
      data() {
        return { name: '' }
      },
      template: `
        <div>
          <h2>Profile</h2>
          <input v-model="name" placeholder="Your name (input preserved)">
        </div>
      `
    }
  }
}
</script>
```

**Result:** Switching between Home and Profile preserves the counter value and input text.

## How keep-alive Works

Without `keep-alive`:
```vue
<component :is="currentView" />
<!-- When switching components:
  1. Old component: beforeDestroy → destroyed
  2. New component: beforeCreate → created → beforeMount → mounted
-->
```

With `keep-alive`:
```vue
<keep-alive>
  <component :is="currentView" />
</keep-alive>
<!-- When switching components:
  1. Old component: deactivated (NOT destroyed!)
  2. New component: activated (NOT created again!)
-->
```

## activated and deactivated Lifecycle Hooks

```vue
<template>
  <div class="data-view">
    <h2>{{ title }}</h2>
    <p>Load count: {{ loadCount }}</p>
    <p>Activation count: {{ activationCount }}</p>
  </div>
</template>

<script>
export default {
  name: 'DataView',
  data() {
    return {
      title: 'Data Component',
      loadCount: 0,
      activationCount: 0,
      data: null
    }
  },
  created() {
    // Only runs once on first creation
    console.log('Component created')
    this.loadCount++
  },
  mounted() {
    // Only runs once on first mount
    console.log('Component mounted')
    this.fetchData()
  },
  activated() {
    // Runs every time component is activated (shown)
    console.log('Component activated')
    this.activationCount++
  },
  deactivated() {
    // Runs every time component is deactivated (hidden)
    console.log('Component deactivated')
  },
  beforeDestroy() {
    // Will NOT run when using keep-alive
    console.log('Component destroyed')
  },
  methods: {
    fetchData() {
      // Fetch initial data
      this.data = 'Initial data loaded'
    }
  }
}
</script>
```

## include and exclude Props

```vue
<template>
  <div>
    <button @click="currentView = 'A'">Component A</button>
    <button @click="currentView = 'B'">Component B</button>
    <button @click="currentView = 'C'">Component C</button>

    <!-- Cache only ComponentA and ComponentB -->
    <keep-alive include="ComponentA,ComponentB">
      <component :is="currentView" />
    </keep-alive>

    <!-- Cache all except ComponentC -->
    <keep-alive exclude="ComponentC">
      <component :is="currentView" />
    </keep-alive>

    <!-- Using regex patterns -->
    <keep-alive :include="/Component[AB]/">
      <component :is="currentView" />
    </keep-alive>

    <!-- Using arrays -->
    <keep-alive :include="['ComponentA', 'ComponentB']">
      <component :is="currentView" />
    </keep-alive>
  </div>
</template>

<script>
export default {
  data() {
    return {
      currentView: 'ComponentA'
    }
  },
  components: {
    ComponentA: {
      name: 'ComponentA',
      template: '<div>Component A (cached)</div>'
    },
    ComponentB: {
      name: 'ComponentB',
      template: '<div>Component B (cached)</div>'
    },
    ComponentC: {
      name: 'ComponentC',
      template: '<div>Component C (not cached)</div>'
    }
  }
}
</script>
```

## max Prop - Cache Size Limit

```vue
<template>
  <div>
    <button @click="currentView = 'A'">A</button>
    <button @click="currentView = 'B'">B</button>
    <button @click="currentView = 'C'">C</button>
    <button @click="currentView = 'D'">D</button>

    <!-- Cache最多3个组件实例 -->
    <keep-alive :max="3">
      <component :is="currentView" :key="currentView" />
    </keep-alive>
  </div>
</template>

<script>
export default {
  data() {
    return {
      currentView: 'A'
    }
  },
  components: {
    A: {
      name: 'ComponentA',
      data() {
        return { mounted: false }
      },
      mounted() {
        this.mounted = true
        console.log('Component A mounted')
      },
      activated() {
        console.log('Component A activated')
      },
      deactivated() {
        console.log('Component A deactivated')
      },
      template: '<div>Component A - Mounted: {{ mounted }}</div>'
    },
    B: {
      name: 'ComponentB',
      template: '<div>Component B</div>'
    },
    C: {
      name: 'ComponentC',
      template: '<div>Component C</div>'
    },
    D: {
      name: 'ComponentD',
      template: '<div>Component D</div>'
    }
  }
}
</script>
```

**Behavior with `max="3"`:**
- When activating A, B, C: All cached
- When activating D: Oldest (A) is destroyed and removed from cache
- Cache follows LRU (Least Recently Used) strategy

## keep-alive with Vue Router

```vue
<!-- App.vue -->
<template>
  <div id="app">
    <nav>
      <router-link to="/">Home</router-link>
      <router-link to="/about">About</router-link>
      <router-link to="/contact">Contact</router-link>
    </nav>

    <!-- Cache all route components -->
    <keep-alive>
      <router-view />
    </keep-alive>

    <!-- Cache specific routes -->
    <keep-alive include="HomePage,AboutPage">
      <router-view />
    </keep-alive>
  </div>
</template>

<style>
nav a {
  margin-right: 10px;
}
</style>
```

## Router with Conditional Caching

```vue
<!-- App.vue -->
<template>
  <div id="app">
    <keep-alive>
      <router-view v-if="$route.meta.keepAlive" />
    </keep-alive>
    <router-view v-if="!$route.meta.keepAlive" />
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>
```

```javascript
// router.js
import Vue from 'vue'
import Router from 'vue-router'

import Home from './views/Home.vue'
import About from './views/About.vue'
import Login from './views/Login.vue'

Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/',
      name: 'home',
      component: Home,
      meta: { keepAlive: true } // This route will be cached
    },
    {
      path: '/about',
      name: 'about',
      component: About,
      meta: { keepAlive: true } // This route will be cached
    },
    {
      path: '/login',
      name: 'login',
      component: Login,
      meta: { keepAlive: false } // This route will NOT be cached
    }
  ]
})
```

## Combining with transition

```vue
<template>
  <div>
    <keep-alive>
      <transition name="slide" mode="out-in">
        <component :is="currentView" :key="currentView" />
      </transition>
    </keep-alive>
  </div>
</template>

<style>
.slide-enter-active, .slide-leave-active {
  transition: transform 0.3s;
}
.slide-enter, .slide-leave-to {
  transform: translateX(100%);
}
</style>
```

## Data Refreshing on Activation

```vue
<template>
  <div class="product-list">
    <h2>Products</h2>
    <p>Last updated: {{ lastUpdated }}</p>
    <ul>
      <li v-for="product in products" :key="product.id">
        {{ product.name }} - ${{ product.price }}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'ProductList',
  data() {
    return {
      products: [],
      lastUpdated: null
    }
  },
  created() {
    this.fetchProducts()
  },
  activated() {
    // Refresh data when component is activated
    this.fetchProducts()
  },
  methods: {
    fetchProducts() {
      // Simulate API call
      this.products = [
        { id: 1, name: 'Product A', price: 100 },
        { id: 2, name: 'Product B', price: 200 }
      ]
      this.lastUpdated = new Date().toLocaleTimeString()
    }
  }
}
</script>
```

## Cache Invalidation Pattern

```vue
<template>
  <div>
    <button @click="invalidateCache">Invalidate Cache</button>

    <keep-alive>
      <component :is="currentView" :cache-key="cacheKey" />
    </keep-alive>
  </div>
</template>

<script>
export default {
  data() {
    return {
      currentView: 'DataView',
      cacheKey: 1
    }
  },
  methods: {
    invalidateCache() {
      // Changing the key forces component recreation
      this.cacheKey++
    }
  }
}
</script>
```

## keep-alive Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `include` | String \| RegExp \| Array | - | Components to cache (matched by `name` option) |
| `exclude` | String \| RegExp \| Array | - | Components NOT to cache |
| `max` | Number | - | Maximum number of component instances to cache |

## Important: Component Name Requirement

```vue
<!-- ❌ WRONG: Component without name won't be cached properly -->
<script>
export default {
  // No name option
  data() {
    return { message: 'Hello' }
  }
}
</script>

<!-- ✅ CORRECT: Component with name -->
<script>
export default {
  name: 'MyComponent', // Required for include/exclude to work
  data() {
    return { message: 'Hello' }
  }
}
</script>
```

## Common Patterns

### 1. Tab Navigation with State Preservation

```vue
<template>
  <div class="tabs">
    <div class="tab-buttons">
      <button
        v-for="tab in tabs"
        :key="tab.id"
        :class="{ active: currentTab === tab.id }"
        @click="currentTab = tab.id"
      >
        {{ tab.name }}
      </button>
    </div>

    <keep-alive>
      <component
        :is="getTabComponent(currentTab)"
        :key="currentTab"
      />
    </keep-alive>
  </div>
</template>

<script>
export default {
  data() {
    return {
      currentTab: 'profile',
      tabs: [
        { id: 'profile', name: 'Profile' },
        { id: 'settings', name: 'Settings' },
        { id: 'activity', name: 'Activity' }
      ]
    }
  },
  methods: {
    getTabComponent(tabId) {
      return tabId + '-tab'
    }
  },
  components: {
    'profile-tab': {
      name: 'ProfileTab',
      data() {
        return { formData: {} }
      },
      template: `
        <div class="tab-content">
          <h3>Profile</h3>
          <input v-model="formData.name" placeholder="Name">
          <input v-model="formData.email" placeholder="Email">
        </div>
      `
    },
    'settings-tab': {
      name: 'SettingsTab',
      data() {
        return { settings: { notifications: true } }
      },
      template: `
        <div class="tab-content">
          <h3>Settings</h3>
          <label>
            <input type="checkbox" v-model="settings.notifications">
            Enable notifications
          </label>
        </div>
      `
    },
    'activity-tab': {
      name: 'ActivityTab',
      template: `
        <div class="tab-content">
          <h3>Activity</h3>
          <p>Your recent activity...</p>
        </div>
      `
    }
  }
}
</script>

<style>
.tabs {
  width: 100%;
  max-width: 600px;
}
.tab-buttons {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}
.tab-buttons button {
  padding: 10px 20px;
  border: 1px solid #ddd;
  background: white;
  cursor: pointer;
}
.tab-buttons button.active {
  background: #42b983;
  color: white;
}
.tab-content {
  padding: 20px;
  border: 1px solid #ddd;
}
</style>
```

### 2. Search Results Cache

```vue
<template>
  <div class="search-container">
    <input
      v-model="searchQuery"
      @input="handleSearch"
      placeholder="Search..."
    >

    <keep-alive>
      <search-results
        v-if="showResults"
        :query="searchQuery"
        :key="searchQuery"
      />
    </keep-alive>
  </div>
</template>

<script>
export default {
  data() {
    return {
      searchQuery: '',
      showResults: false
    }
  },
  methods: {
    handleSearch() {
      this.showResults = this.searchQuery.length > 2
    }
  },
  components: {
    SearchResults: {
      name: 'SearchResults',
      props: ['query'],
      data() {
        return { results: [], loading: false }
      },
      watch: {
        query(newQuery) {
          this.fetchResults(newQuery)
        }
      },
      activated() {
        // Refresh results when coming back to this view
        if (this.query) {
          this.fetchResults(this.query)
        }
      },
      methods: {
        fetchResults(query) {
          this.loading = true
          // Simulate API call
          setTimeout(() => {
            this.results = [
              { id: 1, title: `${query} result 1` },
              { id: 2, title: `${query} result 2` }
            ]
            this.loading = false
          }, 500)
        }
      },
      template: `
        <div class="results">
          <div v-if="loading">Loading...</div>
          <ul v-else>
            <li v-for="result in results" :key="result.id">
              {{ result.title }}
            </li>
          </ul>
        </div>
      `
    }
  }
}
</script>
```

### 3. Multi-Step Form

```vue
<template>
  <div class="wizard">
    <div class="steps">
      <div
        v-for="(step, index) in steps"
        :key="index"
        :class="{ active: currentStep === index }"
        class="step-indicator"
      >
        Step {{ index + 1 }}
      </div>
    </div>

    <keep-alive>
      <component
        :is="currentStepComponent"
        :data="formData"
        @next="nextStep"
        @prev="prevStep"
      />
    </keep-alive>
  </div>
</template>

<script>
export default {
  data() {
    return {
      currentStep: 0,
      steps: ['personal', 'address', 'confirm'],
      formData: {}
    }
  },
  computed: {
    currentStepComponent() {
      return `step-${this.steps[this.currentStep]}`
    }
  },
  methods: {
    nextStep(data) {
      this.formData = { ...this.formData, ...data }
      if (this.currentStep < this.steps.length - 1) {
        this.currentStep++
      }
    },
    prevStep() {
      if (this.currentStep > 0) {
        this.currentStep--
      }
    }
  },
  components: {
    'step-personal': {
      name: 'StepPersonal',
      props: ['data'],
      template: `
        <div>
          <h3>Personal Information</h3>
          <input v-model="localData.name" placeholder="Name">
          <input v-model="localData.email" placeholder="Email">
          <button @click="$emit('next', localData)">Next</button>
        </div>
      `,
      data() {
        return {
          localData: { ...this.data }
        }
      }
    },
    'step-address': {
      name: 'StepAddress',
      props: ['data'],
      template: `
        <div>
          <h3>Address</h3>
          <input v-model="localData.address" placeholder="Address">
          <button @click="$emit('prev')">Previous</button>
          <button @click="$emit('next', localData)">Next</button>
        </div>
      `,
      data() {
        return {
          localData: { ...this.data }
        }
      }
    },
    'step-confirm': {
      name: 'StepConfirm',
      props: ['data'],
      template: `
        <div>
          <h3>Confirm</h3>
          <pre>{{ data }}</pre>
          <button @click="$emit('prev')">Previous</button>
          <button @click="$emit('submit')">Submit</button>
        </div>
      `
    }
  }
}
</script>
```

## When NOT to Use keep-alive

```vue
<!-- ❌ AVOID: Caching memory-intensive components -->
<keep-alive>
  <heavy-chart-component /> <!-- Large chart, high memory usage -->
</keep-alive>

<!-- ❌ AVOID: Caching components with real-time data -->
<keep-alive>
  <live-stock-ticker /> <!-- Shows stale data when reactivated -->
</keep-alive>

<!-- ❌ AVOID: Caching login/security pages -->
<keep-alive>
  <login-form /> <!-- May show sensitive data or cause security issues -->
</keep-alive>

<!-- ✅ CORRECT: Don't cache these components -->
<heavy-chart-component />
<live-stock-ticker />
<login-form />
```

## Memory Management

```vue
<template>
  <div>
    <!-- Limit cache size to prevent memory bloat -->
    <keep-alive :max="5">
      <router-view />
    </keep-alive>
  </div>
</template>
```

## Debugging keep-alive

```vue
<script>
export default {
  name: 'MyComponent',
  created() {
    console.log('Component created')
  },
  mounted() {
    console.log('Component mounted')
  },
  activated() {
    console.log('Component activated - from keep-alive cache')
  },
  deactivated() {
    console.log('Component deactivated - moved to keep-alive cache')
  },
  beforeDestroy() {
    console.log('Component destroyed - removed from cache')
  }
}
</script>
```

## Best Practices

1. **Always set component `name`** - Required for `include`/`exclude` to work
2. **Use `max` prop** - Prevent unlimited cache growth
3. **Refresh data in `activated`** - Update stale data when component reappears
4. **Avoid caching heavy components** - Charts, videos, large lists
5. **Don't cache sensitive pages** - Login, payment, admin pages
6. **Clear cache when needed** - Use `:key` change to force recreation
7. **Test memory usage** - Monitor app memory with large caches

## Reference

- [Vue 2 API - keep-alive](https://v2.vuejs.org/v2/api/#keep-alive)
- [Vue 2 Guide - Dynamic Components with keep-alive](https://v2.vuejs.org/v2/guide/components.html#Dynamic-Components)
