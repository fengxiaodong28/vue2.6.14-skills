---
title: Provide/Inject is NOT Reactive by Default in Vue 2
impact: HIGH
impactDescription: Provide/inject in Vue 2 doesn't automatically make provided values reactive to changes
type: capability
tags:
  - vue2
  - provide-inject
  - reactivity
  - component-communication
  - cross-component
---

# Provide/Inject is NOT Reactive by Default in Vue 2

Impact: HIGH - In Vue 2, provide/inject does NOT automatically create reactive connections. When the provider's value changes, injected values in descendants won't update unless you use specific patterns.

## Task Checklist

- [ ] Use object/function wrapper for reactive provide/inject
- [ ] Use Vue.observable() for reactive shared state (Vue 2.6+)
- [ ] Consider using event bus or Vuex for complex state sharing

## Problem (Not Reactive)

```javascript
// Ancestor.vue
export default {
  data() {
    return {
      theme: 'dark'
    }
  },
  provide() {
    return {
      // WRONG: Direct value - NOT reactive
      theme: this.theme
    }
  },
  methods: {
    toggleTheme() {
      this.theme = this.theme === 'dark' ? 'light' : 'dark'
      // Descendants won't see the change!
    }
  }
}

// Descendant.vue
export default {
  inject: ['theme'],
  computed: {
    currentTheme() {
      return this.theme // Will NOT update when ancestor changes it
    }
  }
}
```

## Solution 1: Use Function Wrapper

```javascript
// Ancestor.vue
export default {
  data() {
    return {
      theme: 'dark'
    }
  },
  provide() {
    return {
      // CORRECT: Function wrapper - reactive
      theme: () => this.theme
    }
  }
}

// Descendant.vue
export default {
  inject: ['theme'],
  computed: {
    currentTheme() {
      // Call the function to get current value
      return this.theme() // NOW reactive!
    }
  }
}
```

## Solution 2: Use Vue.observable (Vue 2.6+)

```javascript
// Ancestor.vue
import Vue from 'vue'

// Create reactive object
const reactiveState = Vue.observable({
  theme: 'dark',
  user: null
})

export default {
  provide() {
    return {
      // CORRECT: Observable object - reactive
      reactiveState
    }
  },
  methods: {
    toggleTheme() {
      reactiveState.theme = reactiveState.theme === 'dark' ? 'light' : 'dark'
      // Descendants will see the change!
    }
  }
}

// Descendant.vue
export default {
  inject: ['reactiveState'],
  computed: {
    currentTheme() {
      return this.reactiveState.theme // NOW reactive!
    }
  }
}
```

## Solution 3: Provide Computed Property

```javascript
// Ancestor.vue
export default {
  data() {
    return {
      theme: 'dark'
    }
  },
  provide() {
    return {
      // CORRECT: Computed property - reactive
      theme: {
        from: 'ancestor',
        get: () => this.theme
      }
    }
  }
}

// Descendant.vue
export default {
  inject: ['theme'],
  computed: {
    currentTheme() {
      return this.theme.get() // NOW reactive!
    }
  }
}
```

## Accessing Methods via Provide/Inject

```javascript
// Ancestor.vue
export default {
  methods: {
    showMessage(msg) {
      alert(msg)
    }
  },
  provide() {
    return {
      // Methods work fine (they're references)
      showMessage: this.showMessage
    }
  }
}

// Descendant.vue
export default {
  inject: ['showMessage'],
  methods: {
    handleClick() {
      this.showMessage('Hello from descendant')
    }
  }
}
```

## Provide Symbol Keys (Avoid Naming Conflicts)

```javascript
// keys.js
export const ThemeKey = Symbol('theme')
export const UserKey = Symbol('user')

// Ancestor.vue
import { ThemeKey, UserKey } from './keys'

export default {
  provide() {
    return {
      [ThemeKey]: this.theme,
      [UserKey]: this.user
    }
  }
}

// Descendant.vue
import { ThemeKey } from './keys'

export default {
  inject: {
    theme: ThemeKey,
    user: { from: UserKey, default: null }
  }
}
```

## Reference

- [Vue 2 Provide/Inject](https://vue2.docs.vuejs.org/v2/api/#provide-inject)
- [Vue 2 Reactivity - Vue.observable](https://vue2.docs.vuejs.org/v2/api/#vue-observable)
