---
title: Filters are Removed in Vue 3
impact: MEDIUM
impactDescription: Filters feature exists only in Vue 2 and is completely removed in Vue 3
type: capability
tags:
  - vue2
  - vue3
  - filters
  - deprecated
  - migration
---

# Filters are Removed in Vue 3

Impact: MEDIUM - Filters are a Vue 2 only feature that has been completely removed in Vue 3. While filters work fine in Vue 2.6.14, be aware this syntax won't migrate to Vue 3.

## Problem

Filters use the pipe `|` syntax which doesn't work well with TypeScript and creates ambiguity. Vue 3 removes this feature entirely in favor of methods or computed properties.

## Task Checklist

- [ ] Use filters for simple text transformations in Vue 2
- [ ] Be aware filters won't work in Vue 3
- [ ] Consider using methods or computed properties for better TypeScript support

## Vue 2 Filters (Current)

```vue
<template>
  <div>
    <!-- Local filter -->
    <p>{{ message | capitalize }}</p>

    <!-- Chained filters -->
    <p>{{ message | capitalize | reverse }}</p>

    <!-- Filter with arguments -->
    <p>{{ price | currency('USD') }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: 'hello world',
      price: 99.99
    }
  },
  filters: {
    capitalize(value) {
      if (!value) return ''
      value = value.toString()
      return value.charAt(0).toUpperCase() + value.slice(1)
    },
    reverse(value) {
      if (!value) return ''
      return value.toString().split('').reverse().join('')
    },
    currency(value, symbol = '$') {
      return symbol + value.toFixed(2)
    }
  }
}
</script>
```

## Vue 3 / Alternative Approach

```vue
<template>
  <div>
    <!-- Use computed property -->
    <p>{{ capitalizedMessage }}</p>

    <!-- Use method -->
    <p{{ reverseMessage(message) }}</p>

    <!-- Use method with arguments -->
    <p>{{ formatCurrency(price, 'USD') }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: 'hello world',
      price: 99.99
    }
  },
  computed: {
    capitalizedMessage() {
      if (!this.message) return ''
      return this.message.charAt(0).toUpperCase() + this.message.slice(1)
    }
  },
  methods: {
    reverseMessage(value) {
      if (!value) return ''
      return value.toString().split('').reverse().join('')
    },
    formatCurrency(value, symbol = '$') {
      return symbol + value.toFixed(2)
    }
  }
}
</script>
```

## Global Filters (Vue 2 Only)

```javascript
// Register global filter (Vue 2 only)
Vue.filter('capitalize', function(value) {
  if (!value) return ''
  value = value.toString()
  return value.charAt(0).toUpperCase() + value.slice(1)
})

// Use in any component
{{ message | capitalize }}
```

## Migration Path

When migrating from Vue 2 to Vue 3, replace filters with:

1. **Computed properties** - For transformations without arguments
2. **Methods** - For transformations with arguments
3. **Global properties** - Replace global filters with app.config.globalProperties

## Reference

- [Vue 2 Filters](https://vue2.docs.vuejs.org/v2/guide/filters.html)
- [Vue 3 Migration Guide - Filters](https://v3-migration.vuejs.org/breaking-changes/filters.html)
