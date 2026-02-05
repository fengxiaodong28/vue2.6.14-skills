---
title: v-for Has Higher Priority Than v-if in Vue 2
impact: HIGH
impactDescription: v-if doesn't have access to v-for variables when used on the same element
type: capability
tags:
  - vue2.6.14
  - directives
  - v-for
  - v-if
  - template
  - computed
---

# v-for Has Higher Priority Than v-if in Vue 2

Impact: HIGH - In Vue 2, `v-for` has higher priority than `v-if`, meaning `v-if` cannot access the variables defined by `v-for` when used on the same element. This is reversed from Vue 3 behavior.

## Problem

When `v-for` and `v-if` are used on the same element in Vue 2, `v-for` is evaluated first. The `v-if` condition will not have access to the loop variables (like `item` or `index`), often resulting in errors or unexpected behavior.

## Task Checklist

- [ ] Use `<template>` element with `v-if` and inner `v-for`
- [ ] Use computed properties to filter arrays before `v-for`
- [ ] Never use `v-if` and `v-for` on the same element

## Incorrect

```vue
<template>
  <div>
    <!-- WRONG: v-if doesn't have access to 'item' from v-for -->
    <li v-for="item in items" v-if="item.isActive" :key="item.id">
      {{ item.name }}
    </li>

    <!-- WRONG: Trying to filter with v-if -->
    <li v-for="user in users" v-if="user.age >= 18" :key="user.id">
      {{ user.name }} (Adult)
    </li>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: [
        { id: 1, name: 'Item 1', isActive: true },
        { id: 2, name: 'Item 2', isActive: false }
      ],
      users: [
        { id: 1, name: 'John', age: 25 },
        { id: 2, name: 'Jane', age: 17 }
      ]
    }
  }
}
</script>
```

## Correct: Using Template Element

```vue
<template>
  <div>
    <!-- CORRECT: Use template with v-if, v-for on inner element -->
    <template v-for="item in items">
      <li v-if="item.isActive" :key="item.id">
        {{ item.name }}
      </li>
    </template>

    <!-- CORRECT: Template with v-if for filtering -->
    <template v-for="user in users">
      <li v-if="user.age >= 18" :key="user.id">
        {{ user.name }} (Adult)
      </li>
    </template>
  </div>
</template>
```

## Correct: Using Computed Properties (Recommended)

```vue
<template>
  <div>
    <!-- CORRECT: Use computed property for filtering -->
    <li v-for="item in activeItems" :key="item.id">
      {{ item.name }}
    </li>

    <!-- CORRECT: Use computed property for filtered users -->
    <li v-for="user in adultUsers" :key="user.id">
      {{ user.name }} (Adult)
    </li>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: [
        { id: 1, name: 'Item 1', isActive: true },
        { id: 2, name: 'Item 2', isActive: false }
      ],
      users: [
        { id: 1, name: 'John', age: 25 },
        { id: 2, name: 'Jane', age: 17 }
      ]
    }
  },
  computed: {
    activeItems() {
      return this.items.filter(item => item.isActive)
    },
    adultUsers() {
      return this.users.filter(user => user.age >= 18)
    }
  }
}
</script>
```

## Vue 2 vs Vue 3 Difference

**Important**: This behavior is reversed in Vue 3. In Vue 3, `v-if` has higher priority than `v-for`, meaning the code examples marked as "WRONG" above would actually work in Vue 3 (though it's still not recommended for performance reasons).

## Reference

- [Vue 2 List Rendering - v-for with v-if](https://vue2.docs.vuejs.org/v2/guide/list.html#v-for-with-v-if)
- [Vue 3 Migration Guide - v-if and v-for on the Same Element](https://v3-migration.vuejs.org/breaking-changes/#v-if-and-v-for-on-the-same-element)
