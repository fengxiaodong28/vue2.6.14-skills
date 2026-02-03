---
title: Vue 2 Lifecycle Hooks (beforeDestroy vs destroyed)
impact: MEDIUM
impactDescription: Vue 2 uses different lifecycle names than Vue 3 for destruction hooks
type: capability
tags:
  - vue2
  - lifecycle
  - beforeDestroy
  - destroyed
  - migration
  - vue3-differences
---

# Vue 2 Lifecycle Hooks (beforeDestroy vs destroyed)

Impact: MEDIUM - Vue 2 uses `beforeDestroy` and `destroyed` for cleanup hooks. These are renamed to `beforeUnmount` and `unmounted` in Vue 3.

## Task Checklist

- [ ] Use `beforeDestroy` for cleanup before component removal
- [ ] Use `destroyed` for cleanup after component removal (rarely needed)
- [ ] Remember Vue 3 uses different names (`beforeUnmount`, `unmounted`)

## All Vue 2 Lifecycle Hooks (In Order)

```javascript
export default {
  // 1. Instance initialization
  beforeCreate() {
    // Cannot access data, computed, methods
    // Good for: plugins initialization
  },

  created() {
    // Can access data, computed, methods
    // DOM not mounted yet
    // Good for: API calls, initial data setup
  },

  // 2. DOM mounting
  beforeMount() {
    // DOM compile complete, not mounted
    // Rarely used
  },

  mounted() {
    // DOM ready and mounted
    // Good for: DOM manipulation, third-party init
  },

  // 3. Data changes
  beforeUpdate() {
    // Data changed, DOM not yet updated
  },

  updated() {
    // DOM updated after data change
  },

  // 4. Destruction (VUE 2 SPECIFIC NAMES)
  beforeDestroy() {
    // Before instance destruction
    // Good for: cleanup (event listeners, timers, etc.)
    // Renamed to beforeUnmount in Vue 3
  },

  destroyed() {
    // After instance destruction
    // Good for: final cleanup notifications
    // Renamed to unmounted in Vue 3
  }
}
```

## Common Cleanup Pattern

```vue
<script>
export default {
  data() {
    return {
      timer: null,
      eventHandler: null
    }
  },

  mounted() {
    // Set up resources
    this.timer = setInterval(this.fetchData, 5000)
    window.addEventListener('resize', this.handleResize)
    this.eventHandler = this.$refs.myInput.addEventListener('input', this.handleInput)
  },

  // VUE 2: Use beforeDestroy for cleanup
  beforeDestroy() {
    // Clean up timer
    if (this.timer) {
      clearInterval(this.timer)
      this.timer = null
    }

    // Clean up event listeners
    window.removeEventListener('resize', this.handleResize)
  },

  methods: {
    fetchData() {
      // ...
    },
    handleResize() {
      // ...
    },
    handleInput() {
      // ...
    }
  }
}
</script>
```

## Lifecycle Comparison Table

| Vue 2 | Vue 3 | Description |
|-------|-------|-------------|
| `beforeCreate` | `beforeCreate` | Instance initialization |
| `created` | `created` | Options setup complete |
| `beforeMount` | `beforeMount` | Before DOM mount |
| `mounted` | `mounted` | After DOM mount |
| `beforeUpdate` | `beforeUpdate` | Before DOM update |
| `updated` | `updated` | After DOM update |
| **`beforeDestroy`** | **`beforeUnmount`** | Before component removal |
| **`destroyed`** | **`unmounted`** | After component removal |
| `activated` | `activated` | Keep-alive component activated |
| `deactivated` | `deactivated` | Keep-alive component deactivated |
| `errorCaptured` | `errorCaptured` | Error from descendant caught |

## Keep-Alive Lifecycle

```vue
<template>
  <keep-alive>
    <component :is="currentComponent"></component>
  </keep-alive>
</template>

<script>
export default {
  // Called when cached component is activated
  activated() {
    console.log('Component activated')
    // Good for: refreshing data, starting timers
  },

  // Called when cached component is deactivated
  deactivated() {
    console.log('Component deactivated')
    // Good for: pausing timers, saving state
  }
}
</script>
```

## Error Capturing

```javascript
export default {
  // Capture errors from descendant components
  errorCaptured(err, vm, info) {
    console.error('Error captured:', err)
    console.error('Component:', vm)
    console.error('Info:', info)

    // Return false to prevent error from propagating
    return false
  }
}
```

## Reference

- [Vue 2 Lifecycle Hooks](https://vue2.docs.vuejs.org/v2/api/#Options-Lifecycle-Hooks)
- [Vue 3 Migration Guide - Lifecycle Hooks](https://v3-migration.vuejs.org/breaking-changes/#options-api-destruction-hooks)
