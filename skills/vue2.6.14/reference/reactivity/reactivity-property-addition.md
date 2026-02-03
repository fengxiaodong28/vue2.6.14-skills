---
title: Can't Detect Property Addition on Objects
impact: HIGH
impactDescription: Vue 2's Object.defineProperty reactivity cannot detect new properties added to existing objects
type: capability
tags:
  - vue2.6.14
  - reactivity
  - object-property-addition
  - this-set
  - vue2.6.14-6-14
---

# Can't Detect Property Addition on Objects

Impact: HIGH - Vue 2's `Object.defineProperty` reactivity system cannot detect when new properties are added to an existing object after initialization. This means newly added properties will not trigger reactivity updates in templates or computed properties.

## Problem

Vue 2 uses `Object.defineProperty` to add getters and setters to each property during component initialization. When you add a new property to an object after initialization, Vue cannot intercept access to that property, so it won't be reactive.

## Task Checklist

- [ ] Always use `this.$set()` to add new properties to reactive objects
- [ ] Use object spread `return { ...this.obj, newProp: value }` for complete object replacement
- [ ] Initialize all possible properties in `data()` with default values when possible

## Incorrect

```vue
<template>
  <div>
    <p>User: {{ user.name }}</p>
    <p>Email: {{ user.email }}</p>
    <button @click="addEmail">Add Email</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      user: {
        name: 'John'
      }
    }
  },
  methods: {
    addEmail() {
      // WRONG: Direct property addition - NOT reactive
      this.user.email = 'john@example.com'
      // Template won't update!
    }
  }
}
</script>
```

## Correct

```vue
<template>
  <div>
    <p>User: {{ user.name }}</p>
    <p>Email: {{ user.email }}</p>
    <button @click="addEmail">Add Email</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      user: {
        name: 'John',
        email: null // Initialize with null/undefined
      }
    }
  },
  methods: {
    // CORRECT: Use this.$set() for reactivity
    addEmail() {
      this.$set(this.user, 'email', 'john@example.com')
      // Template updates correctly!
    },

    // ALSO CORRECT: Object spread for complete replacement (Vue 2.6+)
    addEmailAlternative() {
      this.user = {
        ...this.user,
        email: 'john@example.com'
      }
    }
  }
}
</script>
```

## Alternative: Initialize with Default Values

```javascript
data() {
  return {
    // Initialize all known properties upfront
    user: {
      name: 'John',
      email: null,
      phone: null,
      address: null
    }
  }
}
```

## Reference

- [Vue 2 Reactivity - Change Detection Caveats](https://vue2.docs.vuejs.org/v2/guide/reactivity.html#Change-Detection-Caveats)
- [Vue 2 API - vm.$set](https://vue2.docs.vuejs.org/v2/api/#vm-set)
