---
title: Use Vue.extend() for TypeScript Type Inference
impact: HIGH
impactDescription: Vue.extend() enables proper TypeScript type inference for Options API components
type: best-practice
tags:
  - vue2.6.14
  - typescript
  - vue-extend
  - options-api
  - type-inference
---

# Use Vue.extend() for TypeScript Type Inference

Impact: HIGH - Using `Vue.extend()` provides proper TypeScript type inference for Vue 2 Options API components, enabling type-safe access to `data`, `computed`, `methods`, and props.

## Problem

Without `Vue.extend()`, TypeScript cannot properly infer component types, leading to missing type checking and autocomplete for component properties.

## Task Checklist

- [ ] Always use `Vue.extend()` when defining Vue 2 components with TypeScript
- [ ] Use `<script lang="ts">` in SFC files
- [ ] Create shims-vue.d.ts for `.vue` file type declarations

## Incorrect

```vue
<script lang="ts">
// WRONG: No type inference
export default {
  data() {
    return {
      count: 0,
      message: 'Hello'
    }
  },
  computed: {
    doubleCount() {
      return this.count * 2 // TypeScript doesn't know `this.count` exists
    }
  },
  methods: {
    increment() {
      this.count++ // TypeScript error: Property 'count' does not exist
    }
  }
}
</script>
```

## Correct

```vue
<script lang="ts">
import Vue from 'vue'

// CORRECT: Use Vue.extend() for type inference
export default Vue.extend({
  name: 'MyComponent',
  data() {
    return {
      count: 0,
      message: 'Hello'
    }
  },
  computed: {
    doubleCount(): number {
      return this.count * 2 // TypeScript knows this.count is number
    }
  },
  methods: {
    increment(): void {
      this.count++ // TypeScript knows this.count exists
    },
    getMessage(): string {
      return this.message // TypeScript knows this.message is string
    }
  }
})
</script>
```

## With Props and Interfaces

```vue
<script lang="ts">
import Vue from 'vue'
import { PropType } from 'vue'

interface User {
  id: number
  name: string
  email: string
}

export default Vue.extend({
  name: 'UserProfile',
  props: {
    user: {
      type: Object as PropType<User>,
      required: true
    }
  },
  data() {
    return {
      isLoading: false
    }
  },
  computed: {
    displayName(): string {
      return this.user.name // TypeScript knows this.user is User
    }
  },
  methods: {
    async fetchData(): Promise<void> {
      this.isLoading = true
      // ... fetch logic
    }
  }
})
</script>
```

## Required: shims-vue.d.ts

```typescript
// src/shims-vue.d.ts
declare module '*.vue' {
  import Vue from 'vue'
  export default Vue
}
```

## Reference

- [Vue 2 TypeScript Support](https://vue2.docs.vuejs.org/v2/guide/typescript.html)
- [TypeScript - Component Options](https://github.com/vuejs/vue-class-component/blob/master/README.md)
