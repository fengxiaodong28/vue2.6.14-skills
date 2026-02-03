---
title: Global Component Registration with Vue.component
impact: MEDIUM
impactDescription: Vue.component() registers components globally for use throughout the application
type: capability
tags:
  - vue2.6.14
  - component-registration
  - global-components
  - Vue.component
  - component-naming
---

# Global Component Registration with Vue.component

Impact: MEDIUM - Global component registration with `Vue.component()` makes components available throughout the application without explicit imports. Use sparingly as it increases initial bundle size and may cause naming conflicts.

## Task Checklist

- [ ] Use `Vue.component()` for frequently used UI components (buttons, inputs, etc.)
- [ ] Follow kebab-case naming convention for global components
- [ ] Register globally before creating the root Vue instance
- [ ] Use local registration for feature-specific components
- [ ] Be aware of tree-shaking limitations with global components

## Basic Registration

```javascript
import Vue from 'vue'
import MyComponent from './MyComponent.vue'

// Register globally before creating the Vue instance
Vue.component('my-component', MyComponent)

// Now can be used in any component without import
new Vue({
  el: '#app'
})
```

**Usage in template:**

```vue
<template>
  <div>
    <!-- No import needed -->
    <my-component></my-component>
  </div>
</template>
```

## Registration with Vue.extend

```javascript
import Vue from 'vue'

// Using Vue.extend for component definition
const AlertBox = Vue.extend({
  name: 'AlertBox',
  props: {
    type: {
      type: String,
      default: 'info'
    },
    message: String
  },
  template: `
    <div :class="['alert', 'alert-' + type]">
      {{ message }}
    </div>
  `
})

// Register
Vue.component('alert-box', AlertBox)
```

## Auto-Global Registration Pattern

```javascript
// src/components/global.js
import Vue from 'vue'

// Automatically import and register all base components
function registerGlobalComponents() {
  const requireComponent = require.context(
    // Look for files in the components/base directory
    './components/base',
    // Do not look in subdirectories
    false,
    // Only include .vue files
    /[A-Z]\w+\.(vue)$/
  )

  requireComponent.keys().forEach(fileName => {
    // Get component config
    const componentConfig = requireComponent(fileName)

    // Get PascalCase name of component
    const componentName = fileName
      .split('/')
      .pop()
      .replace(/\.\w+$/, '')

    // Register component globally
    Vue.component(componentName, componentConfig.default || componentConfig)
  })
}

export default registerGlobalComponents
```

**Directory structure:**

```
src/
├── components/
│   └── base/
│       ├── BaseButton.vue
│       ├── BaseInput.vue
│       ├── BaseCard.vue
│       └── BaseModal.vue
├── main.js
```

**Usage in main.js:**

```javascript
import Vue from 'vue'
import App from './App.vue'
import registerGlobalComponents from './components/global'

registerGlobalComponents()

new Vue({
  render: h => h(App)
}).$mount('#app')
```

## Naming Conventions

```javascript
import Vue from 'vue'
import MyComponent from './MyComponent.vue'

// Use kebab-case in template, PascalCase in JavaScript
Vue.component('my-component', MyComponent)

// All these work in template:
// <my-component>
// <MyComponent>  // NOT recommended but works
// <myComponent>  // NOT recommended but works

// Recommended: Always use kebab-case in templates
```

## Plugin-Based Registration

```javascript
// src/plugins/components.js
import BaseButton from '@/components/base/BaseButton.vue'
import BaseInput from '@/components/base/BaseInput.vue'
import BaseCard from '@/components/base/BaseCard.vue'

export default {
  install(Vue) {
    Vue.component('BaseButton', BaseButton)
    Vue.component('BaseInput', BaseInput)
    Vue.component('BaseCard', BaseCard)
  }
}
```

**Register the plugin:**

```javascript
// main.js
import Vue from 'vue'
import App from './App.vue'
import ComponentPlugin from './plugins/components'

Vue.use(ComponentPlugin)

new Vue({
  render: h => h(App)
}).$mount('#app')
```

## When to Use Global Registration

### Good Use Cases

```javascript
// 1. Frequently used UI primitives
Vue.component('base-button', BaseButton)
Vue.component('base-input', BaseInput)
Vue.component('base-icon', BaseIcon)
Vue.component('base-badge', BaseBadge)

// 2. Form components
Vue.component('form-field', FormField)
Vue.component('form-select', FormSelect)
Vue.component('form-checkbox', FormCheckbox)

// 3. Layout components
Vue.component('app-header', AppHeader)
Vue.component('app-footer', AppFooter)
Vue.component('app-sidebar', AppSidebar)
```

### Avoid Global Registration For

```javascript
// ❌ Avoid: Feature-specific components
Vue.component('UserProfileCard', UserProfileCard)
Vue.component('DashboardChart', DashboardChart)
Vue.component('AdminPanel', AdminPanel)

// ✅ Better: Import these locally where needed
```

## Example: Base UI Components

**BaseButton.vue**

```vue
<template>
  <button
    :class="classes"
    :disabled="disabled || loading"
    @click="handleClick"
  >
    <span v-if="loading" class="spinner"></span>
    <slot></slot>
  </button>
</template>

<script>
export default {
  name: 'BaseButton',
  props: {
    type: {
      type: String,
      default: 'default',
      validator: (value) => ['default', 'primary', 'danger', 'success'].includes(value)
    },
    size: {
      type: String,
      default: 'medium',
      validator: (value) => ['small', 'medium', 'large'].includes(value)
    },
    disabled: Boolean,
    loading: Boolean
  },
  computed: {
    classes() {
      return [
        'base-button',
        `base-button--${this.type}`,
        `base-button--${this.size}`,
        { 'base-button--loading': this.loading }
      ]
    }
  },
  methods: {
    handleClick(event) {
      if (!this.disabled && !this.loading) {
        this.$emit('click', event)
      }
    }
  }
}
</script>

<style scoped>
.base-button {
  /* Base styles */
}
.base-button--primary {
  /* Primary styles */
}
</style>
```

**Register globally:**

```javascript
import Vue from 'vue'
import BaseButton from '@/components/base/BaseButton.vue'
import BaseInput from '@/components/base/BaseInput.vue'
import BaseModal from '@/components/base/BaseModal.vue'

Vue.component('BaseButton', BaseButton)
Vue.component('BaseInput', BaseInput)
Vue.component('BaseModal', BaseModal)
```

**Use anywhere:**

```vue
<template>
  <div>
    <base-button type="primary" @click="submit">Submit</base-button>
    <base-button type="danger" @click="cancel">Cancel</base-button>
    <base-button loading>Loading...</base-button>
  </div>
</template>

<script>
// No imports needed!
export default {
  methods: {
    submit() {
      // Handle submit
    },
    cancel() {
      // Handle cancel
    }
  }
}
</script>
```

## Component Name Resolution

```javascript
// Registering with PascalCase
Vue.component('MyComponent', MyComponent)

// All these work in template:
<my-component>     <!-- Recommended -->
<MyComponent>     <!-- Works but not recommended -->
<myComponent>     <!-- Works but not recommended -->

// Best practice: Always use kebab-case in templates for consistency
```

## Recursive Components

```javascript
// Tree component that renders itself
Vue.component('tree-node', {
  name: 'TreeNode',  // Required for recursive components
  props: {
    node: Object
  },
  template: `
    <div class="tree-node">
      <span>{{ node.name }}</span>
      <tree-node
        v-for="child in node.children"
        :key="child.id"
        :node="child"
        v-if="node.children"
      />
    </div>
  `
})
```

## Extending Registered Components

```javascript
// Base component
Vue.component('base-button', {
  props: ['type', 'size'],
  template: '<button :class="classes"><slot></slot></button>',
  computed: {
    classes() {
      return [`btn-${this.type}`, `btn-${this.size}`]
    }
  }
})

// Extend the base component
const PrimaryButton = Vue.component('base-button').extend({
  computed: {
    classes() {
      return ['btn-primary', `btn-${this.size}`]
    }
  }
})

Vue.component('primary-button', PrimaryButton)
```

## Async Global Components

```javascript
// Register component that loads on demand
Vue.component('async-component', function (resolve) {
  require(['./AsyncComponent.vue'], resolve)
})

// Or with dynamic import (Vue 2.6+)
Vue.component('async-component', () => import('./AsyncComponent.vue'))
```

## Testing with Global Components

```javascript
// tests/unit/example.spec.js
import Vue from 'vue'
import { mount } from '@vue/test-utils'

// Register global components for testing
import BaseButton from '@/components/base/BaseButton.vue'
import BaseInput from '@/components/base/BaseInput.vue'

Vue.component('BaseButton', BaseButton)
Vue.component('BaseInput', BaseInput)

// Now can use in tests without importing
describe('MyComponent', () => {
  it('renders base-button', () => {
    const wrapper = mount(MyComponent)
    expect(wrapper.find('.base-button').exists()).toBe(true)
  })
})
```

## Pros and Cons

### Pros
- No import statements needed
- Convenient for frequently used components
- Cleaner component code

### Cons
- Not tree-shaken (included in bundle even if unused)
- Potential naming conflicts
- Less explicit dependency tracking
- Harder to track component usage

## Best Practices

1. **Global**: Base UI primitives (buttons, inputs, icons)
2. **Local**: Feature-specific components (user cards, dashboards)
3. Use **prefixes** for global components to avoid conflicts (`Base-`, `App-`)
4. **Auto-register** base components from a dedicated directory
5. Consider **lazy loading** for large global components

## Reference

- [Vue 2 Guide - Component Registration](https://vue2.docs.vuejs.org/v2/guide/components-registration.html)
- [Vue 2 API - Vue.component](https://vue2.docs.vuejs.org/v2/api/#Vue-component)
