---
title: Async Component Error Handling in Vue 2.6
impact: MEDIUM
impactDescription: Vue 2.6.0+ improved async component error handling with errorCaptured hook and error handler option
type: capability
tags:
  - vue2.6.14
  - async-components
  - error-handling
  - error-captured
  - component-lifecycle
---

# Async Component Error Handling in Vue 2.6

Impact: MEDIUM - Vue 2.6.0+ introduced improved error handling for async components with the `errorCaptured` lifecycle hook and `errorHandler` option. These features allow graceful error recovery and better user experience.

> **Note**: Async component error handling improvements were introduced in Vue 2.6.0. Earlier versions had limited error handling capabilities for async components.

## Task Checklist

- [ ] Use `errorCaptured` hook to catch errors from descendant components
- [ ] Use `errorHandler` option for component-specific error handling
- [ ] Return `false` from errorCaptured to propagate errors upward
- [ ] Implement fallback UI for failed async components
- [ ] Handle async component loading states with v-if

## errorCaptured Lifecycle Hook

The `errorCaptured` hook is called when an error from any descendant component is captured.

```vue
<template>
  <div class="error-boundary">
    <!-- Display error if caught -->
    <div v-if="error" class="error-message">
      <h3>Something went wrong</h3>
      <p>{{ error.message }}</p>
      <button @click="error = null">Try Again</button>
    </div>

    <!-- Child components -->
    <async-component v-if="!error" />
  </div>
</template>

<script>
import AsyncComponent from './AsyncComponent.vue'

export default {
  name: 'ErrorBoundary',
  components: {
    AsyncComponent
  },
  data() {
    return {
      error: null
    }
  },
  errorCaptured(err, vm, info) {
    // err: Error object
    // vm: Component that caused the error
    // info: Vue-specific error information (e.g., which lifecycle hook)

    console.error('Error captured:', err)
    console.error('Component:', vm)
    console.error('Info:', info)

    // Store error for display
    this.error = err

    // Return false to stop error propagation
    // Return true (or undefined) to continue propagating
    return false
  }
}
</script>

<style scoped>
.error-message {
  padding: 20px;
  background: #fee;
  border: 1px solid #f88;
  border-radius: 4px;
}
</style>
```

## errorHandler Component Option

Vue 2.6.0+ supports the `errorHandler` option for component-specific error handling.

```vue
<template>
  <div class="error-handling-component">
    <p>{{ data }}</p>
  </div>
</template>

<script>
export default {
  name: 'ErrorHandlingComponent',
  // Component-specific error handler
  errorHandler(err, vm, info) {
    // err: Error object
    // vm: Component instance
    // info: Error information string

    console.error('Component error:', err)
    console.error('Info:', info)

    // Log to external service
    if (typeof window.trackJs !== 'undefined') {
      window.trackJs.track(err)
    }

    // Don't propagate error further
    return false
  },
  data() {
    return {
      data: 'Initial data'
    }
  },
  mounted() {
    // This might throw an error
    this.mightFail()
  },
  methods: {
    mightFail() {
      // Simulate error
      throw new Error('Something went wrong!')
    }
  }
}
</script>
```

## Async Component with Error Handling

```vue
<template>
  <div>
    <!-- Async component with loading, error, and delay states -->
    <async-component
      v-if="!hasError"
      @hook:error="handleError"
    />

    <!-- Error fallback -->
    <div v-if="hasError" class="error-state">
      <p>Failed to load component</p>
      <button @click="retry">Retry</button>
    </div>
  </div>
</template>

<script>
import LoadingComponent from './LoadingComponent.vue'
import ErrorComponent from './ErrorComponent.vue'

// Async component factory
const AsyncComponent = () => ({
  component: import('./HeavyComponent.vue'),
  loading: LoadingComponent,
  error: ErrorComponent,
  delay: 200, // Show loading after 200ms
  timeout: 10000 // Timeout after 10 seconds
})

export default {
  name: 'AsyncComponentWrapper',
  components: {
    AsyncComponent
  },
  data() {
    return {
      hasError: false
    }
  },
  errorCaptured(err, vm, info) {
    // Catch errors from async component
    console.error('Async component error:', err)
    this.hasError = true
    return false
  },
  methods: {
    handleError(err) {
      console.error('Component error:', err)
      this.hasError = true
    },
    retry() {
      // Clear error and retry
      this.hasError = false
      // Force re-render
      this.$forceUpdate()
    }
  }
}
</script>
```

## Error Propagation Control

```vue
<template>
  <div>
    <!-- Parent Error Boundary -->
    <parent-boundary>
      <child-component />
    </parent-boundary>
  </div>
</template>

<script>
export default {
  name: 'App',
  components: {
    ParentBoundary: {
      name: 'ParentBoundary',
      data() {
        return {
          errors: []
        }
      },
      errorCaptured(err, vm, info) {
        // Log error
        this.errors.push({
          error: err,
          component: vm.$options.name,
          info: info,
          timestamp: new Date()
        })

        // Return false to stop propagation
        // Error won't reach ancestor components
        return false
      }
    },
    ChildComponent: {
      name: 'ChildComponent',
      template: `
        <div>
          <grandchild-component />
        </div>
      `,
      components: {
        GrandchildComponent: {
          name: 'GrandchildComponent',
          template: `
            <div>
              <button @click="throwError">Throw Error</button>
            </div>
          `,
          methods: {
            throwError() {
              throw new Error('Error from grandchild')
            }
          }
        }
      }
    }
  }
}
</script>
```

## Global Error Handler

```javascript
// main.js
import Vue from 'vue'

// Set global error handler
Vue.config.errorHandler = function (err, vm, info) {
  // Handle all Vue errors globally
  console.error('Global error handler:', err)
  console.error('Component:', vm?.$options.name)
  console.error('Info:', info)

  // Send to error tracking service
  if (process.env.NODE_ENV === 'production') {
    // Example: Send to Sentry
    if (window.Sentry) {
      window.Sentry.captureException(err, {
        tags: {
          vue_component: vm?.$options.name,
          lifecycle: info
        },
        extra: {
          componentName: vm?.$options.name,
          componentData: vm?.$data
        }
      })
    }
  }
}

// Create Vue instance
new Vue({
  el: '#app',
  render: h => h(App)
})
```

## Fallback UI for Failed Components

```vue
<template>
  <div>
    <!-- Async component with error handling -->
    <component
      :is="loadedComponent"
      v-if="!hasError"
      @error="handleComponentError"
    />

    <!-- Fallback UI when component fails -->
    <div v-else class="component-failed">
      <div class="error-icon">⚠️</div>
      <h3>Component Unavailable</h3>
      <p>{{ errorMessage }}</p>
      <div class="error-actions">
        <button @click="reload">Reload</button>
        <button @click="useFallback">Use Alternative</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ComponentWithFallback',
  data() {
    return {
      loadedComponent: null,
      hasError: false,
      errorMessage: 'The component failed to load.'
    }
  },
  created() {
    this.loadComponent()
  },
  errorCaptured(err, vm, info) {
    console.error('Component error captured:', err)
    this.hasError = true
    this.errorMessage = err.message || 'Unknown error occurred'
    return false
  },
  methods: {
    async loadComponent() {
      try {
        // Dynamic import with error handling
        const component = await import('./HeavyComponent.vue')
        this.loadedComponent = component.default
      } catch (error) {
        console.error('Failed to load component:', error)
        this.hasError = true
        this.errorMessage = 'Failed to load component. Please try again.'
      }
    },
    handleComponentError(error) {
      this.hasError = true
      this.errorMessage = error.message
    },
    reload() {
      this.hasError = false
      this.loadComponent()
    },
    useFallback() {
      // Load a simpler fallback component
      this.loadedComponent = {
        template: '<div>Simple fallback component</div>'
      }
      this.hasError = false
    }
  }
}
</script>

<style scoped>
.component-failed {
  padding: 40px;
  text-align: center;
  background: #fff5f5;
  border: 1px solid #fed7d7;
  border-radius: 8px;
}

.error-icon {
  font-size: 48px;
  margin-bottom: 20px;
}

.error-actions {
  margin-top: 20px;
}

.error-actions button {
  margin: 0 10px;
  padding: 10px 20px;
}
</style>
```

## errorCaptured Return Values

```vue
<script>
export default {
  name: 'ErrorCapturedDemo',
  errorCaptured(err, vm, info) {
    console.error('Error:', err)

    // Return false: Stop error propagation
    // Error is considered handled and won't propagate further
    return false

    // Return true or undefined: Continue propagation
    // Error will propagate to parent components
    // return true

    // Don't return: Same as returning true
    // Error continues to propagate
  }
}
</script>
```

## Best Practices

1. **Always implement error boundaries** - Wrap async components in error boundaries
2. **Provide fallback UI** - Show user-friendly messages when components fail
3. **Log errors appropriately** - Use different logging for development vs production
4. **Don't suppress all errors** - Only handle errors you can recover from
5. **Test error scenarios** - Simulate errors to test your error handling

## Common Patterns

### 1. Error Boundary Component

```vue
<!-- ErrorBoundary.vue -->
<template>
  <div>
    <slot v-if="!hasError" />
    <div v-else class="error-fallback">
      <slot name="error" :error="error">
        <p>Something went wrong</p>
        <button @click="reset">Try Again</button>
      </slot>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ErrorBoundary',
  data() {
    return {
      hasError: false,
      error: null
    }
  },
  errorCaptured(err, vm, info) {
    this.hasError = true
    this.error = err
    return false
  },
  methods: {
    reset() {
      this.hasError = false
      this.error = null
    }
  }
}
</script>
```

### 2. Async Component with Retry

```vue
<template>
  <div>
    <div v-if="loading">Loading...</div>
    <div v-else-if="error">
      <p>Error: {{ error.message }}</p>
      <button @click="fetch">Retry</button>
    </div>
    <div v-else>
      <slot :data="data" />
    </div>
  </div>
</template>

<script>
export default {
  name: 'AsyncDataLoader',
  props: {
    url: String
  },
  data() {
    return {
      data: null,
      loading: false,
      error: null
    }
  },
  created() {
    this.fetch()
  },
  errorCaptured(err) {
    this.error = err
    return false
  },
  methods: {
    async fetch() {
      this.loading = true
      this.error = null
      try {
        const response = await fetch(this.url)
        this.data = await response.json()
      } catch (err) {
        this.error = err
      } finally {
        this.loading = false
      }
    }
  }
}
</script>
```

## Reference

- [Vue 2 API - errorCaptured](https://vue2.docs.vuejs.org/v2/api/#errorCaptured)
- [Vue 2 API - errorHandler](https://vue2.docs.vuejs.org/v2/api/#errorHandler)
- [Vue 2 Guide - Error Handling](https://vue2.docs.vuejs.org/v2/guide/components-edge-cases.html)
