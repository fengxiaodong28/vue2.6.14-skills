---
title: Vue.config Global Configuration Options in Vue 2
impact: MEDIUM
impactDescription: Vue.config contains global configuration options that must be set before bootstrapping the application
type: capability
tags:
  - vue2.6.14
  - global-config
  - Vue.config
  - configuration
  - silent
  - devtools
  - errorHandler
---

# Vue.config Global Configuration Options in Vue 2

Impact: MEDIUM - `Vue.config` is an object containing Vue's global configurations. These options must be modified before bootstrapping your application (before `new Vue()`).

## Task Checklist

- [ ] Set `Vue.config` options before creating the root Vue instance
- [ ] Use `silent` to suppress all Vue logs and warnings
- [ ] Use `errorHandler` for global error tracking
- [ ] Use `devtools` to enable vue-devtools in production
- [ ] Use `warnHandler` for custom warning handlers in development

## When to Set Vue.config

```javascript
// ✅ CORRECT: Set config before new Vue()
import Vue from 'vue'

// Set global configuration
Vue.config.silent = true
Vue.config.devtools = true
Vue.config.errorHandler = (err, vm, info) => {
  console.error('Vue error:', err, vm, info)
}

// Now create the Vue instance
new Vue({
  el: '#app'
})

// ❌ WRONG: Setting config after Vue instance creation
new Vue({ el: '#app' })
Vue.config.silent = true // Too late! Instance already created
```

## silent

Suppress all Vue logs and warnings:

```javascript
Vue.config.silent = true
```

**Use cases:**
- Production environments
- Testing environments
- Suppressing warnings during data migrations

```javascript
// main.js
import Vue from 'vue'

if (process.env.NODE_ENV === 'production') {
  Vue.config.silent = true
}

new Vue({ el: '#app' })
```

## optionMergeStrategies

Define custom merging strategies for options:

```javascript
Vue.config.optionMergeStrategies._my_option = function (parent, child, vm) {
  // Return the merged value
  return child + 1
}

const Profile = Vue.extend({
  _my_option: 1
})

// Profile.options._my_option = 2
```

**Example with custom strategy:**

```javascript
// Custom merge strategy for API configs
Vue.config.optionMergeStrategies.apiConfig = function (toVal, fromVal) {
  if (!toVal) return fromVal
  if (!fromVal) return toVal

  // Deep merge API configs
  const merged = {}
  const keys = new Set([...Object.keys(toVal), ...Object.keys(fromVal)])

  keys.forEach(key => {
    const toValue = toVal[key]
    const fromValue = fromVal[key]

    if (typeof toValue === 'object' && typeof fromValue === 'object') {
      merged[key] = { ...fromValue, ...toValue }
    } else {
      merged[key] = toValue
    }
  })

  return merged
}

// Usage
Vue.mixin({
  beforeCreate() {
    console.log(this.$options.apiConfig)
  }
})
```

## devtools

Configure whether to allow vue-devtools inspection:

```javascript
// make sure to set this synchronously immediately after loading Vue
Vue.config.devtools = true
```

**Use cases:**
- Enable devtools in production builds
- Disable devtools in specific environments

```javascript
// Enable devtools in production for debugging
if (process.env.VUE_APP_ENABLE_DEVTOOLS === 'true') {
  Vue.config.devtools = true
}
```

## errorHandler

Assign a handler for uncaught errors:

```javascript
Vue.config.errorHandler = function (err, vm, info) {
  // err: Error object
  // vm: Vue instance that encountered the error
  // info: Vue-specific error information (e.g., which lifecycle hook)
  console.error('Vue error:', err)
  console.error('Component:', vm)
  console.error('Info:', info)

  // Send to error tracking service
  if (typeof Sentry !== 'undefined' && Sentry.captureException) {
    Sentry.captureException(err, {
      tags: {
        vue_component: vm.$options.name || 'anonymous',
        lifecycle: info
      }
    })
  }
}
```

**Error handler coverage:**
- Component render function and watcher errors (2.2.0+)
- Component lifecycle hook errors (2.2.0+)
- Vue custom event handler errors (2.4.0+)
- v-on DOM listener errors (2.6.0+)
- Promise chain errors in async functions

## warnHandler

Assign a custom handler for runtime warnings:

```javascript
// New in 2.4.0+
Vue.config.warnHandler = function (msg, vm, trace) {
  // msg: Warning message
  // vm: Vue instance
  // trace: Component hierarchy trace

  // Log warnings differently in development
  if (process.env.NODE_ENV === 'development') {
    console.group('Vue Warning')
    console.warn(msg)
    console.log('Component:', vm)
    console.log('Trace:', trace)
    console.groupEnd()
  }
}
```

## ignoredElements

Make Vue ignore custom elements:

```javascript
Vue.config.ignoredElements = [
  'my-custom-web-component',
  'another-web-component',
  // Use a RegExp to ignore all elements that start with "ion-"
  /^ion-/
]
```

**Use cases:**
- Web Components
- Third-party component libraries
- Custom elements from legacy code

## keyCodes

Define custom key alias(es) for v-on:

```javascript
Vue.config.keyCodes = {
  v: 86,
  f1: 112,
  // camelCase won't work
  mediaPlayPause: 179,
  // use kebab-case with double quotation marks
  "media-play-pause": 179,
  up: [38, 87] // Multiple keys
}

// Usage
<input @keyup.media-play-pause="method">
```

**Common key codes:**

```javascript
Vue.config.keyCodes = {
  // Media keys
  mediaPlayPause: 179,
  mediaStop: 178,
  mediaTrackNext: 176,
  mediaTrackPrev: 177,

  // Function keys
  f1: 112, f2: 113, f3: 114, f4: 115, f5: 116,
  f6: 117, f7: 118, f8: 119, f9: 120, f10: 121,
  f11: 122, f12: 123,

  // Special keys
  enter: 13,
  tab: 9,
  delete: 46,
  esc: 27,
  space: 32,
  up: 38,
  down: 40,
  left: 37,
  right: 39
}
```

## performance

Enable component performance tracing:

```javascript
// New in 2.2.0+
Vue.config.performance = true
```

**Use cases:**
- Development performance profiling
- Identifying slow components
- Debugging render performance

**Enabling only in development:**

```javascript
if (process.env.NODE_ENV === 'development') {
  Vue.config.performance = true
}
```

## productionTip

Suppress the production tip on Vue startup:

```javascript
// New in 2.2.0+
Vue.config.productionTip = false
```

**Use cases:**
- Production builds
- Reducing console output
- Cleaner application startup

## Complete Configuration Example

```javascript
// src/config/vue.js
import Vue from 'vue'
import * as Sentry from '@sentry/vue'

export function configureVue() {
  // Silent mode for production
  if (process.env.NODE_ENV === 'production') {
    Vue.config.silent = process.env.VUE_APP_SILENT === 'true'
    Vue.config.productionTip = false
  }

  // DevTools
  if (process.env.NODE_ENV === 'development') {
    Vue.config.devtools = true
    Vue.config.performance = true
  }

  // Error handling
  Vue.config.errorHandler = (err, vm, info) => {
    console.error('Vue error:', err)

    // Log component information
    if (vm) {
      console.error('Component name:', vm.$options.name)
      console.error('Component data:', vm.$data)
    }
    console.error('Info:', info)

    // Send to error tracking
    Sentry.captureException(err, {
      tags: {
        vue: true,
        lifecycle: info
      },
      extra: {
        componentName: vm?.$options.name,
        componentData: vm?.$data
      }
    })
  }

  // Warning handler (development only)
  Vue.config.warnHandler = (msg, vm, trace) => {
    console.warn('Vue warning:', msg)
    if (vm) {
      console.warn('Component:', vm.$options.name)
    }
    if (trace) {
      console.warn('Trace:', trace)
    }
  }

  // Ignore custom elements
  Vue.config.ignoredElements = [
    'my-component',
    /^ion-/
  ]

  // Custom key codes
  Vue.config.keyCodes = {
    mediaPlayPause: 179,
    mediaStop: 178
  }

  return Vue
}
```

**Usage:**

```javascript
// main.js
import { configureVue } from './config/vue'

const Vue = configureVue()

new Vue({
  el: '#app',
  render: h => h(App)
})
```

## Per-Environment Configuration

```javascript
// Development configuration
if (process.env.NODE_ENV === 'development') {
  Vue.config.devtools = true
  Vue.config.performance = true
  Vue.config.warnHandler = (msg, vm, trace) => {
    console.warn(msg)
  }
}

// Production configuration
if (process.env.NODE_ENV === 'production') {
  Vue.config.silent = false // Keep logs for debugging
  Vue.config.productionTip = false
  Vue.config.errorHandler = (err, vm, info) => {
    // Send to error service
    errorService.log(err, {
      component: vm.$options.name,
      info
    })
  }
}
```

## Reactivity System Configuration

Note: These options cannot be modified at runtime in production builds.

## Reference

- [Vue 2 API - Global Config](https://vue2.docs.vuejs.org/v2/api/#Global-Config)
- [Vue 2 Guide - Reactivity - Change Detection Caveats](https://vue2.docs.vuejs.org/v2/guide/reactivity.html)
