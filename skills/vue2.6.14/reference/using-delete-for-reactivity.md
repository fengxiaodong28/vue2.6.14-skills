---
title: Using Vue.delete for Removing Reactive Properties
impact: HIGH
impactDescription: Properties deleted with delete operator are not reactive in Vue 2
type: solution
tags:
  - vue2
  - reactivity
  - vue-delete
  - dollar-delete
  - property-deletion
---

# Using Vue.delete for Removing Reactive Properties

Impact: HIGH - Using the `delete` operator to remove properties does not trigger reactivity updates. Use `Vue.delete()` or `this.$delete()` instead.

## Task Checklist

- [ ] Use `this.$delete(obj, key)` to remove reactive properties
- [ ] Use object spread/rest for bulk property removal
- [ ] Avoid using the `delete` operator on reactive objects
- [ ] Consider setting property to `null` if deletion isn't critical

## Problem: Using Delete Operator

```javascript
// ❌ WRONG: delete operator doesn't trigger reactivity
export default {
  data() {
    return {
      user: {
        name: 'John',
        email: 'john@example.com',
        age: 30
      }
    }
  },
  methods: {
    removeEmail() {
      delete this.user.email
      // Property is deleted, but view won't update!
    }
  }
}
```

## Solution: Using $delete

```javascript
// ✅ CORRECT: Use $delete for reactive property removal
export default {
  data() {
    return {
      user: {
        name: 'John',
        email: 'john@example.com',
        age: 30
      }
    }
  },
  methods: {
    removeEmail() {
      this.$delete(this.user, 'email')
      // Property is deleted AND view updates!
    }
  }
}
```

## Deleting Multiple Properties

```javascript
export default {
  data() {
    return {
      user: {
        name: 'John',
        email: 'john@example.com',
        phone: '123-456-7890',
        address: '123 Main St',
        age: 30
      }
    }
  },
  methods: {
    // Delete multiple properties one by one
    removeContactInfo() {
      this.$delete(this.user, 'email')
      this.$delete(this.user, 'phone')
      this.$delete(this.user, 'address')
    },

    // Or use object spread/rest to exclude properties
    removeContactInfoSpread() {
      const { email, phone, address, ...rest } = this.user
      this.user = rest
    }
  }
}
```

## Conditional Deletion

```javascript
export default {
  methods: {
    deleteIfPresent(obj, key) {
      if (key in obj) {
        this.$delete(obj, key)
      }
    },

    deleteByCondition(obj, predicate) {
      Object.keys(obj).forEach(key => {
        if (predicate(obj[key], key)) {
          this.$delete(obj, key)
        }
      })
    },

    // Example usage
    cleanupNullValues(obj) {
      this.deleteByCondition(obj, value => value === null)
    }
  }
}
```

## Array Item Deletion

```javascript
export default {
  data() {
    return {
      items: [
        { id: 1, name: 'Item 1' },
        { id: 2, name: 'Item 2' },
        { id: 3, name: 'Item 3' }
      ]
    }
  },
  methods: {
    // ✅ CORRECT: Use splice for array items
    removeByIndex(index) {
      this.items.splice(index, 1)
    },

    // Or filter to create new array
    removeById(id) {
      this.items = this.items.filter(item => item.id !== id)
    },

    // ❌ WRONG: Don't use delete on arrays (leaves holes)
    removeByIndexWrong(index) {
      delete this.items[index]
      // Creates sparse array with undefined hole!
    }
  }
}
```

## Vue.delete vs this.$delete

```javascript
// Global Vue.delete (when not in component context)
import Vue from 'vue'

const obj = { name: 'John', email: 'john@example.com' }
Vue.delete(obj, 'email')

// Inside component, use this.$delete
export default {
  methods: {
    removeProperty() {
      // Same as Vue.delete but component-scoped
      this.$delete(this.obj, 'key')
    }
  }
}
```

## Safe Deletion Pattern

```javascript
export default {
  methods: {
    // Safe delete with fallback
    safeDelete(obj, key, fallback = null) {
      if (key in obj) {
        this.$delete(obj, key)
        return true
      }
      // Key doesn't exist
      return false
    },

    // Delete and get value
    pop(obj, key) {
      if (key in obj) {
        const value = obj[key]
        this.$delete(obj, key)
        return value
      }
      return undefined
    }
  }
}
```

## Object Spread Instead of Delete

```javascript
export default {
  data() {
    return {
      user: {
        name: 'John',
        email: 'john@example.com',
        age: 30,
        city: 'New York'
      }
    }
  },
  methods: {
    // Remove specific keys using rest syntax
    removeKeys(keysToRemove) {
      const newObj = { ...this.user }
      keysToRemove.forEach(key => {
        delete newObj[key] // OK on new object
      })
      this.user = newObj
    },

    // More concise version
    removeKeysClean(keysToRemove) {
      const entries = Object.entries(this.user)
        .filter(([key]) => !keysToRemove.includes(key))
      this.user = Object.fromEntries(entries)
    }
  }
}
```

## Handling Nested Properties

```javascript
export default {
  data() {
    return {
      config: {
        api: {
          key: 'abc123',
          secret: 'xyz789',
          endpoint: 'https://api.example.com'
        }
      }
    }
  },
  methods: {
    // Delete nested property
    removeSecret() {
      if (this.config.api) {
        this.$delete(this.config.api, 'secret')
      }
    },

    // Delete entire nested object
    removeApiConfig() {
      this.$delete(this.config, 'api')
    }
  }
}
```

## Form Field Management

```javascript
export default {
  data() {
    return {
      form: {
        username: '',
        email: '',
        password: '',
        confirmPassword: '',
        rememberMe: false
      }
    }
  },
  methods: {
    // Remove temporary form fields
    cleanupForm() {
      this.$delete(this.form, 'confirmPassword')
      this.$delete(this.form, 'rememberMe')
    },

    // Reset form - remove or null values?
    resetForm() {
      // Option 1: Delete keys
      Object.keys(this.form).forEach(key => {
        if (key !== 'username') {
          this.$delete(this.form, key)
        }
      })

      // Option 2: Just set to null/empty (preserves keys)
      this.form = {
        username: '',
        email: '',
        password: ''
      }
    }
  }
}
```

## Dynamic Key Management

```javascript
export default {
  data() {
    return {
      dynamicState: {
        feature1: true,
        feature2: false,
        feature3: true
      }
    }
  },
  methods: {
    toggleFeature(featureName) {
      if (featureName in this.dynamicState) {
        // Toggle value
        this.dynamicState[featureName] = !this.dynamicState[featureName]
      } else {
        // Add feature
        this.$set(this.dynamicState, featureName, true)
      }
    },

    removeFeature(featureName) {
      this.$delete(this.dynamicState, featureName)
    }
  }
}
```

## Performance Consideration

```javascript
export default {
  methods: {
    // Multiple $delete calls trigger multiple updates
    removeManySlow() {
      this.$delete(this.obj, 'a')
      this.$delete(this.obj, 'b')
      this.$delete(this.obj, 'c')
    },

    // Single reactive update - more efficient
    removeManyFast() {
      const { a, b, c, ...rest } = this.obj
      this.obj = rest
    }
  }
}
```

## Common Gotchas

```javascript
export default {
  data() {
    return {
      items: [{ id: 1, value: 'a' }]
    }
  },
  methods: {
    // ❌ WRONG: Delete from array item
    removeValueWrong() {
      delete this.items[0].value
      // Not reactive!
    },

    // ✅ CORRECT: Use $delete
    removeValueRight() {
      this.$delete(this.items[0], 'value')
    },

    // ✅ ALSO CORRECT: Replace object
    removeValueAlternative() {
      const { value, ...rest } = this.items[0]
      this.$set(this.items, 0, rest)
    }
  }
}
```

## Summary Table

| Scenario | Wrong (Not Reactive) | Correct (Reactive) |
|----------|---------------------|-------------------|
| Delete property | `delete obj.key` | `this.$delete(obj, 'key')` |
| Delete multiple | Multiple `delete` | `const {a,b,...rest} = obj; obj = rest` |
| Remove array item | `delete arr[i]` | `arr.splice(i, 1)` |
| Delete nested | `delete obj.a.b` | `this.$delete(obj.a, 'b')` |

## Reference

- [Vue 2 API - Vue.delete](https://v2.vuejs.org/v2/api/#Vue-delete)
- [Vue 2 Reactivity - Change Detection Caveats](https://v2.vuejs.org/v2/guide/reactivity.html#Change-Detection-Caveats)
