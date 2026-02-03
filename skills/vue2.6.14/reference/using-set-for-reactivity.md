---
title: Using Vue.set for Adding Reactive Properties
impact: HIGH
impactDescription: New properties added to objects are not reactive unless using Vue.set or this.$set
type: solution
tags:
  - vue2
  - reactivity
  - vue-set
  - dollar-set
  - object-properties
---

# Using Vue.set for Adding Reactive Properties

Impact: HIGH - New properties added directly to objects after creation are not reactive. Use `Vue.set()` or `this.$set()` to add reactive properties.

## Task Checklist

- [ ] Use `this.$set(obj, key, value)` to add reactive properties
- [ ] Use object spread `this.obj = { ...this.obj, newProp: value }` for bulk updates
- [ ] Initialize all possible properties in `data()` when possible
- [ ] Never assign directly to new object keys for reactivity

## Problem: Direct Property Addition

```javascript
// ❌ WRONG: New properties are not reactive
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
      // Direct assignment - NOT reactive!
      this.user.email = 'john@example.com'
      // View won't update!
    }
  }
}
```

## Solution: Using $set

```javascript
// ✅ CORRECT: Use $set for reactive property addition
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
      // Use $set - this IS reactive!
      this.$set(this.user, 'email', 'john@example.com')
    }
  }
}
```

## Adding Nested Properties

```javascript
export default {
  data() {
    return {
      config: {}
    }
  },
  methods: {
    // Adding nested properties
    addApiKey() {
      // For deep nested properties
      this.$set(this.config, 'api', {
        key: 'abc123',
        endpoint: 'https://api.example.com'
      })
    },

    // Or create nested object step by step
    addEndpoint() {
      if (!this.config.api) {
        this.$set(this.config, 'api', {})
      }
      this.$set(this.config.api, 'endpoint', 'https://api.example.com')
    }
  }
}
```

## Array Item Updates

```javascript
export default {
  data() {
    return {
      items: [
        { id: 1, name: 'Item 1' },
        { id: 2, name: 'Item 2' }
      ]
    }
  },
  methods: {
    updateItemName(index, newName) {
      // ❌ WRONG: Not reactive for array items
      this.items[index].name = newName

      // ✅ CORRECT: Use $set for array items
      this.$set(this.items, index, {
        ...this.items[index],
        name: newName
      })
    }
  }
}
```

## Initializing with Null Values

```javascript
// Best practice: Initialize all possible properties
export default {
  data() {
    return {
      user: {
        name: '',
        email: '',    // Initialize with empty string
        age: null,    // Initialize with null
        roles: []     // Initialize with empty array
      }
    }
  }
}
```

## Bulk Property Updates with Object Spread

```javascript
export default {
  data() {
    return {
      user: {
        name: 'John'
      }
    }
  },
  methods: {
    // Add multiple properties at once
    updateUser() {
      // ✅ CORRECT: Replace entire object
      this.user = {
        ...this.user,
        email: 'john@example.com',
        age: 30,
        city: 'New York'
      }
    }
  }
}
```

## Dynamic Form Fields

```javascript
export default {
  data() {
    return {
      form: {
        username: ''
      },
      // Optional fields that may be added
      optionalFields: {
        email: null,
        phone: null,
        address: null
      }
    }
  },
  methods: {
    addOptionalField(fieldName) {
      if (this.optionalFields[fieldName] === undefined) {
        // Add the field
        this.$set(this.form, fieldName, '')
      }
    },

    removeOptionalField(fieldName) {
      // Can't use delete, replace object instead
      const { [fieldName]: removed, ...rest } = this.form
      this.form = rest
    }
  }
}
```

## Conditional Property Addition

```javascript
export default {
  data() {
    return {
      user: {
        name: 'John'
      }
    }
  },
  methods: {
    loadData() {
      // From API
      const apiData = {
        email: 'john@example.com',
        settings: {
          theme: 'dark'
        }
      }

      // Add each property reactively
      Object.keys(apiData).forEach(key => {
        this.$set(this.user, key, apiData[key])
      })
    },

    // Or simpler - replace entire object
    loadDataSimple() {
      const apiData = fetchUserData()
      this.user = { ...this.user, ...apiData }
    }
  }
}
```

## Vue.set vs this.$set

```javascript
// Global Vue.set (when not in component context)
import Vue from 'vue'

const obj = { name: 'John' }
Vue.set(obj, 'email', 'john@example.com')

// Inside component, use this.$set
export default {
  methods: {
    addProperty() {
      // Same as Vue.set but component-scoped
      this.$set(this.obj, 'key', 'value')
    }
  }
}
```

## Checking Property Existence

```javascript
export default {
  methods: {
    addPropertyIfNotExists(obj, key, value) {
      if (!(key in obj)) {
        this.$set(obj, key, value)
      } else {
        console.log(`Property ${key} already exists`)
      }
    },

    ensureProperty(obj, key, defaultValue) {
      if (!(key in obj)) {
        this.$set(obj, key, defaultValue)
      }
      return obj[key]
    }
  }
}
```

## Reactive Maps (Alternative Pattern)

```javascript
// For truly dynamic key-value storage, use an array of objects
export default {
  data() {
    return {
      // Instead of dynamic object properties
      dynamicData: [
        { key: 'name', value: 'John' },
        { key: 'email', value: 'john@example.com' }
      ]
    }
  },
  computed: {
    // Convert to object when needed
    asObject() {
      return this.dynamicData.reduce((acc, item) => {
        acc[item.key] = item.value
        return acc
      }, {})
    }
  },
  methods: {
    addField(key, value) {
      this.dynamicData.push({ key, value })
    },
    updateField(key, newValue) {
      const index = this.dynamicData.findIndex(item => item.key === key)
      if (index >= 0) {
        this.$set(this.dynamicData, index, { key, value: newValue })
      }
    }
  }
}
```

## Common Patterns

```javascript
export default {
  data() {
    return {
      items: []
    }
  },
  methods: {
    // Add to array
    addItem(item) {
      this.items.push(item)
    },

    // Update array item by index
    updateItem(index, newItem) {
      this.$set(this.items, index, newItem)
    },

    // Add property to object in array
    addPropertyToItem(index, key, value) {
      this.$set(this.items[index], key, value)
    },

    // Update nested property
    updateNested(obj, path, value) {
      const keys = path.split('.')
      let target = obj
      for (let i = 0; i < keys.length - 1; i++) {
        if (!target[keys[i]]) {
          this.$set(target, keys[i], {})
        }
        target = target[keys[i]]
      }
      this.$set(target, keys[keys.length - 1], value)
    }
  }
}
```

## Performance Consideration

```javascript
// For many property additions, object spread is more efficient
export default {
  methods: {
    // Less efficient for many properties
    addManyPropertiesSlow() {
      this.$set(this.obj, 'a', 1)
      this.$set(this.obj, 'b', 2)
      this.$set(this.obj, 'c', 3)
      // ... many more
    },

    // More efficient - one reactive update
    addManyPropertiesFast() {
      this.obj = {
        ...this.obj,
        a: 1,
        b: 2,
        c: 3
        // ... many more
      }
    }
  }
}
```

## Summary Table

| Scenario | Wrong | Correct |
|----------|-------|---------|
| Add new property | `obj.new = val` | `this.$set(obj, 'new', val)` |
| Update array item | `arr[i] = val` | `this.$set(arr, i, val)` |
| Multiple properties | Multiple `$set` calls | `obj = { ...obj, ...newProps }` |
| Nested properties | `obj.a.b = val` | `this.$set(obj.a, 'b', val)` |

## Reference

- [Vue 2 Reactivity - Change Detection Caveats](https://v2.vuejs.org/v2/guide/reactivity.html#Change-Detection-Caveats)
- [Vue 2 API - Vue.set](https://v2.vuejs.org/v2/api/#Vue-set)
