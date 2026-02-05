---
title: Can't Detect Array Index Assignment
impact: HIGH
impactDescription: Direct array index assignment (arr[index] = value) doesn't trigger reactivity in Vue 2
type: capability
tags:
  - vue2.6.14
  - reactivity
  - arrays
  - this-set
  - vue2.6.14-6-14
---

# Can't Detect Array Index Assignment

Impact: HIGH - Vue 2 cannot detect when you directly assign a value to an array index (e.g., `arr[0] = 'new value'`). The array will update internally, but Vue won't know to re-render templates.

## Problem

Vue 2 wraps array methods like `push()`, `pop()`, `splice()`, etc., but direct index assignment bypasses Vue's reactivity system.

## Task Checklist

- [ ] Use `this.$set(arr, index, value)` for index assignment
- [ ] Use `arr.splice(index, 1, newValue)` as alternative
- [ ] Use array methods like `push()`, `pop()`, `shift()`, `unshift()` which are wrapped

## Incorrect

```vue
<template>
  <div>
    <li v-for="(item, index) in items" :key="index">
      {{ item }}
    </li>
    <button @click="updateFirst">Update First Item</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: ['apple', 'banana', 'cherry']
    }
  },
  methods: {
    updateFirst() {
      // WRONG: Direct index assignment - NOT reactive
      this.items[0] = 'orange'
      console.log(this.items[0]) // 'orange' - value changed
      // But template won't update!
    }
  }
}
</script>
```

## Correct

```vue
<template>
  <div>
    <li v-for="(item, index) in items" :key="index">
      {{ item }}
    </li>
    <button @click="updateFirst">Update First Item</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: ['apple', 'banana', 'cherry']
    }
  },
  methods: {
    // CORRECT: Use this.$set()
    updateFirst() {
      this.$set(this.items, 0, 'orange')
      // Template updates!
    },

    // ALSO CORRECT: Use splice()
    updateFirstAlternative() {
      this.items.splice(0, 1, 'orange')
      // Template updates!
    }
  }
}
</script>
```

## Array Methods That ARE Reactive

These methods are wrapped by Vue 2 and will trigger updates:

```javascript
// All of these ARE reactive:
this.items.push('new item')
this.items.pop()
this.items.shift()
this.items.unshift('first item')
this.items.splice(start, deleteCount, ...items)
this.items.sort()
this.items.reverse()
```

## Reference

- [Vue 2 Reactivity - Change Detection Caveats](https://vue2.docs.vuejs.org/v2/guide/reactivity.html#Change-Detection-Caveats)
- [Vue 2 API - vm.$set](https://vue2.docs.vuejs.org/v2/api/#vm-set)
