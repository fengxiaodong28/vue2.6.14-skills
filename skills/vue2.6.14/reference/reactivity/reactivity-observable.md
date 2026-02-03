---
title: Vue.observable() for Reactive State (Vue 2.6.0+)
impact: MEDIUM
impactDescription: Vue.observable() creates minimal reactive state without full Vuex store
type: capability
tags:
  - vue2.6.14
  - reactivity
  - vue-observable
  - state-management
  - vue2.6.14-6-plus
---

# Vue.observable() for Reactive State (Vue 2.6.0+)

Impact: MEDIUM - `Vue.observable()` makes an object reactive without needing a full Vuex store. Perfect for simple cross-component state sharing in Vue 2.6.0+.

## Task Checklist

- [ ] Use `Vue.observable()` for simple state management
- [ ] Pass observable object via provide/inject
- [ ] Remember it directly mutates the original object (Vue 2 behavior)
- [ ] Not a replacement for complex state (use Vuex instead)

## Basic Usage

```javascript
import Vue from 'vue'

// Create reactive state
const state = Vue.observable({
  count: 0,
  user: null
})

// Can be used in render functions and computed properties
const Component = {
  render(h) {
    return h('div', `Count: ${state.count}`)
  }
}
```

## With Provide/Inject Pattern

```javascript
// store/state.js
import Vue from 'vue'

// Create observable state
export const state = Vue.observable({
  theme: 'light',
  language: 'en',
  isLoggedIn: false
})
```

```vue
<!-- App.vue -->
<template>
  <div id="app">
    <router-view />
  </div>
</template>

<script>
import { state } from './store/state'

export default {
  name: 'App',
  provide() {
    return {
      // Provide the observable state
      appState: state
    }
  }
}
</script>
```

```vue
<!-- AnyChildComponent.vue -->
<template>
  <div>
    <p>Theme: {{ appState.theme }}</p>
    <button @click="toggleTheme">Toggle Theme</button>
  </div>
</template>

<script>
export default {
  name: 'AnyChildComponent',
  inject: ['appState'],
  methods: {
    toggleTheme() {
      // Direct mutation is reactive
      this.appState.theme = this.appState.theme === 'light' ? 'dark' : 'light'
    }
  }
}
</script>
```

## Vue 2 vs Vue 3 Difference

**Important**: In Vue 2.x, `Vue.observable()` directly mutates the object passed to it:

```javascript
// Vue 2 behavior
const original = { count: 0 }
const reactive = Vue.observable(original)

original.count = 1 // Reactive! (same reference)
reactive.count = 2 // Reactive! (same reference)
```

In Vue 3.x, `reactive()` returns a proxy and leaves the original non-reactive:

```javascript
// Vue 3 behavior (for comparison)
import { reactive } from 'vue'

const original = { count: 0 }
const reactive = reactive(original)

original.count = 1 // NOT reactive
reactive.count = 2 // Reactive
```

## For Future Compatibility

To ensure your code works with both Vue 2 and Vue 3:

```javascript
import Vue from 'vue'

const original = { count: 0 }
const reactiveState = Vue.observable(original)

// Always work with the returned object
reactiveState.count = 1 // ✅ Works in both Vue 2 and Vue 3

// Avoid direct mutation of original in Vue 3
original.count = 1 // ⚠️ Works in Vue 2, but NOT in Vue 3
```

## When to Use vs Vuex

```javascript
// Use Vue.observable() for:
const simpleState = Vue.observable({
  // Simple cross-component state
  theme: 'light',
  sidebarOpen: false,
  notifications: []
})

// Use Vuex for:
const complexStore = new Vuex.Store({
  modules: {
    // Multiple modules
    user: { /* ... */ },
    posts: { /* ... */ },
    // Many actions, mutations, getters
  }
})
```

## Computed Properties with Observable

```javascript
const state = Vue.observable({
  items: []
})

// Create component that uses observable
const ListComponent = Vue.extend({
  computed: {
    filteredItems() {
      return this.state.items.filter(item => item.active)
    }
  },
  data() {
    return {
      state
    }
  }
})
```

## Reference

- [Vue 2.6.0 Release Notes - Vue.observable](https://github.com/vuejs/vue/releases/tag/v2.6.0)
- [Vue 2 API - Vue.observable](https://v2.vuejs.org/v2/api/#Vue-observable)
