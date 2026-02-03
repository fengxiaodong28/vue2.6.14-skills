---
title: Key Syntax Differences Between Vue 2 and Vue 3
impact: HIGH
impactDescription: Understanding syntax differences prevents using Vue 3 patterns in Vue 2 code
type: reference
tags:
  - vue2
  - vue3
  - migration
  - syntax-differences
  - compatibility
---

# Key Syntax Differences Between Vue 2 and Vue 3

Impact: HIGH - Knowing the key syntax differences between Vue 2 and Vue 3 prevents accidentally using incompatible patterns and helps with migration planning.

## Task Checklist

- [ ] Use Options API (Vue 2) not Composition API (Vue 3)
- [ ] Use `beforeDestroy/destroyed` (Vue 2) not `beforeUnmount/unmounted` (Vue 3)
- [ ] Use `$listeners` and `$attrs` correctly in Vue 2
- [ ] Use Filters only in Vue 2 (removed in Vue 3)

## Lifecycle Hooks

| Vue 2 | Vue 3 | Notes |
|-------|-------|-------|
| `beforeCreate` | `beforeCreate` | Same |
| `created` | `created` | Same |
| `beforeMount` | `beforeMount` | Same |
| `mounted` | `mounted` | Same |
| `beforeUpdate` | `beforeUpdate` | Same |
| `updated` | `updated` | Same |
| `beforeDestroy` | `beforeUnmount` | Renamed |
| `destroyed` | `unmounted` | Renamed |

```javascript
// Vue 2
export default {
  beforeDestroy() {
    // Cleanup
  },
  destroyed() {
    // Final cleanup
  }
}

// Vue 3
export default {
  beforeUnmount() {
    // Cleanup
  },
  unmounted() {
    // Final cleanup
  }
}
```

## v-for and v-if Priority

```vue
<!-- Vue 2: v-for has HIGHER priority -->
<li v-for="item in items" v-if="item.isActive">
  {{ item.name }}
</li>

<!-- Vue 3: v-if has HIGHER priority -->
<li v-for="item in items" v-if="item.isActive">
  {{ item.name }}
</li>

<!-- Solution for both: Use template or computed -->
<template v-for="item in items">
  <li v-if="item.isActive">{{ item.name }}</li>
</template>
```

## Filters

```vue
<!-- Vue 2: Filters work -->
{{ message | capitalize }}
{{ price | currency('USD') }}

<!-- Vue 3: Filters REMOVED -->
<!-- Use methods or computed instead -->
{{ capitalize(message) }}
{{ formatCurrency(price, 'USD') }}
```

## $listeners and $attrs

```vue
<!-- Vue 2 -->
<script>
export default {
  inheritAttrs: false,
  computed: {
    // All listeners passed to component
    listeners() {
      return this.$listeners
    }
  }
}
</script>

<template>
  <!-- Pass down all attrs and listeners -->
  <div v-bind="$attrs" v-on="$listeners">
    <input v-bind="$attrs" v-on="$listeners" />
  </div>
</template>

<!-- Vue 3: $listeners removed, merged into $attrs -->
<template>
  <!-- Everything in $attrs now -->
  <div v-bind="$attrs">
    <input v-bind="$attrs" />
  </div>
</template>
```

## .sync Modifier (Vue 2) vs v-model (Vue 3)

```vue
<!-- Vue 2: .sync modifier -->
<ChildComponent :value.sync="localValue" />

<!-- Child emits -->
this.$emit('update:value', newValue)

<!-- Vue 3: .sync removed, use v-model -->
<ChildComponent v-model:value="localValue" />

<!-- Child emits -->
this.$emit('update:modelValue', newValue)
```

## Functional Components

```javascript
// Vue 2: Functional components with 'functional' option
export default {
  functional: true,
  render(h, { props, children, slots }) {
    return h('div', props, children)
  }
}

// Vue 3: Functional components are just functions
import { h } from 'vue'

export default function MyComponent(props, { slots }) {
  return h('div', {}, slots.default())
}
```

## Global API

```javascript
// Vue 2
import Vue from 'vue'
Vue.use(plugin)
Vue.component('comp', Component)
Vue.directive('dir', directive)
Vue.mixin(mixin)
new Vue({ ... }).$mount('#app')

// Vue 3
import { createApp } from 'vue'
const app = createApp({ ... })
app.use(plugin)
app.component('comp', Component)
app.directive('dir', directive)
app.mixin(mixin)
app.mount('#app')
```

## Multiple Root Nodes

```vue
<!-- Vue 2: Only single root node -->
<template>
  <div>
    <header>...</header>
    <main>...</main>
    <footer>...</footer>
  </div>
</template>

<!-- Vue 3: Multiple root nodes supported -->
<template>
  <header>...</header>
  <main>...</main>
  <footer>...</footer>
</template>
```

## Async Components

```javascript
// Vue 2
const AsyncComp = () => import('./AsyncComp.vue')

// Vue 3
import { defineAsyncComponent } from 'vue'
const AsyncComp = defineAsyncComponent(() =>
  import('./AsyncComp.vue')
)
```

## v-model

```vue
<!-- Vue 2 -->
<input v-model="value" />
<!-- Compiles to: -->
<input :value="value" @input="value = $event.target.value" />

<!-- Vue 3 (similar but more configurable) -->
<input v-model="value" />
```

## Reference

- [Vue 3 Migration Guide](https://v3-migration.vuejs.org/)
- [Vue 2 Documentation](https://vue2.docs.vuejs.org/)
