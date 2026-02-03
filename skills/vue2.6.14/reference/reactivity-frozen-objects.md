---
title: Frozen Objects Can't Be Made Reactive in Vue 2
impact: HIGH
impactDescription: Object.defineProperty cannot make frozen objects reactive
type: gotcha
tags:
  - vue2
  - reactivity
  - frozen-objects
  - object-freeze
  - limitations
---

# Frozen Objects Can't Be Made Reactive in Vue 2

Impact: HIGH - Frozen objects cannot be made reactive in Vue 2 because Object.defineProperty cannot define properties on frozen objects.

## Task Checklist

- [ ] Don't use `Object.freeze()` on data you want to be reactive
- [ ] Freeze objects only after you're done with mutations
- [ ] Use frozen objects for static configuration
- [ ] Use separate reactive state for dynamic data

## Problem: Freezing Breaks Reactivity

```javascript
// ❌ WRONG: Frozen objects can't be reactive
export default {
  data() {
    const config = {
      title: 'My App',
      theme: 'dark'
    }
    Object.freeze(config) // FREEZING prevents reactivity!

    return {
      config // This will NOT be reactive
    }
  }
}
```

## Solution: Don't Freeze Reactive Data

```javascript
// ✅ CORRECT: Don't freeze reactive state
export default {
  data() {
    return {
      config: {
        title: 'My App',
        theme: 'dark'
      }
    }
  }
}
```

## When to Use Frozen Objects

```javascript
// ✅ OK: Freeze static constants that don't need reactivity
export default {
  data() {
    return {
      // Static config - can be frozen
      staticConfig: Object.freeze({
        API_BASE_URL: 'https://api.example.com',
        VERSION: '1.0.0',
        DEBUG: false
      }),

      // Dynamic state - must NOT be frozen
      user: {
        name: '',
        email: ''
      },

      // List of options that won't change
      options: Object.freeze([
        { value: 'en', label: 'English' },
        { value: 'zh', label: 'Chinese' }
      ])
    }
  }
}
```

## Freezing After Data Initialization

```javascript
// ✅ CORRECT: Freeze after setting up reactivity
export default {
  data() {
    return {
      items: [1, 2, 3, 4, 5]
    }
  },
  mounted() {
    // After component is mounted, freeze if you want read-only
    // But then you can't make it reactive anymore!
    Object.freeze(this.items)
  }
}
```

## Immutable State Pattern

```javascript
// Create new objects instead of mutations
export default {
  data() {
    return {
      user: {
        name: 'John',
        age: 30
      }
    }
  },
  methods: {
    updateName(newName) {
      // Immutable update - create new object
      this.user = {
        ...this.user,
        name: newName
      }
    },
    haveBirthday() {
      // Immutable update
      this.user = {
        ...this.user,
        age: this.user.age + 1
      }
    }
  }
}
```

## Readonly Computed Properties

```javascript
// Instead of frozen data, use computed
export default {
  data() {
    return {
      firstName: 'John',
      lastName: 'Doe'
    }
  },
  computed: {
    // Computed is always read-only from outside
    fullName() {
      return `${this.firstName} ${this.lastName}`
    },

    // Derived data that should not be modified directly
    userInfo() {
      return {
        name: `${this.firstName} ${this.lastName}`,
        initials: `${this.firstName[0]}${this.lastName[0]}`
      }
    }
  }
}
```

## Static Configuration Pattern

```javascript
// Define static config outside component
const APP_CONFIG = Object.freeze({
  API_URL: 'https://api.example.com',
  MAX_UPLOAD_SIZE: 10 * 1024 * 1024,
  DEFAULTS: Object.freeze({
    THEME: 'light',
    LANGUAGE: 'en'
  })
})

export default {
  data() {
    return {
      // Use frozen config
      config: APP_CONFIG,

      // Dynamic settings that can change
      userSettings: {
        theme: APP_CONFIG.DEFAULTS.THEME,
        language: APP_CONFIG.DEFAULTS.LANGUAGE
      }
    }
  },
  methods: {
    updateSettings(newSettings) {
      // Merge with defaults
      this.userSettings = {
        theme: newSettings.theme || APP_CONFIG.DEFAULTS.THEME,
        language: newSettings.language || APP_CONFIG.DEFAULTS.LANGUAGE
      }
    }
  }
}
```

## Frozen Arrays

```javascript
export default {
  data() {
    return {
      // Static list of options
      constellations: Object.freeze([
        'Aries', 'Taurus', 'Gemini', 'Cancer',
        'Leo', 'Virgo', 'Libra', 'Scorpio',
        'Sagittarius', 'Capricorn', 'Aquarius', 'Pisces'
      ]),

      // Dynamic user selections
      selectedConstellations: []
    }
  },
  methods: {
    toggleSelection(constellation) {
      const index = this.selectedConstellations.indexOf(constellation)
      if (index >= 0) {
        this.selectedConstellations.splice(index, 1)
      } else {
        this.selectedConstellations.push(constellation)
      }
    }
  }
}
```

## Deep Freeze

```javascript
// Helper function for deep freeze
function deepFreeze(obj) {
  Object.keys(obj).forEach(key => {
    const value = obj[key]
    if (value && typeof value === 'object') {
      deepFreeze(value)
    }
  })
  return Object.freeze(obj)
}

// Usage
export default {
  data() {
    return {
      // Deeply frozen config
      config: deepFreeze({
        api: {
          baseUrl: 'https://api.example.com',
          timeout: 5000
        },
        features: {
          darkMode: true,
          notifications: false
        }
      })
    }
  }
}
```

## Vuex with Frozen State

```javascript
// In Vuex, state can be frozen in production for performance
const store = new Vuex.Store({
  state: {
    items: []
  },
  mutations: {
    // Mutations are the only way to change state
    addItem(state, item) {
      state.items.push(item)
    }
  },
  strict: process.env.NODE_ENV !== 'production' // Warns against direct mutation
})

// In production, you can freeze state for performance
if (process.env.NODE_ENV === 'production') {
  // Deep freeze initial state
  deepFreeze(store.state)
}
```

## Detecting Frozen Objects

```javascript
export default {
  methods: {
    isFrozen(obj) {
      return Object.isFrozen(obj)
    },
    tryUpdate(obj, key, value) {
      if (Object.isFrozen(obj)) {
        console.warn('Cannot update frozen object')
        return false
      }
      obj[key] = value
      return true
    }
  }
}
```

## Summary

| Scenario | Use Frozen? | Why |
|----------|-------------|-----|
| Static config constants | Yes | Never changes |
| Component data properties | No | Must be reactive |
| Computed properties | N/A | Already read-only |
| Large static lists | Yes | Performance |
| User input state | No | Needs reactivity |
| API response data | No | May need updates |

## Reference

- [Vue 2 Reactivity - Change Detection Caveats](https://v2.vuejs.org/v2/guide/reactivity.html#Change-Detection-Caveats)
- [MDN - Object.freeze()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)
