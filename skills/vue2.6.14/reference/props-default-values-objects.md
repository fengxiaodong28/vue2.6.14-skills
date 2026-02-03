---
title: Default Values for Object and Array Props Must Use Factory Functions
impact: HIGH
impactDescription: Object and array props require factory functions to avoid shared reference across component instances
type: best-practice
tags:
  - vue2
  - props
  - default-values
  - factory-functions
  - common-mistake
---

# Default Values for Object and Array Props Must Use Factory Functions

Impact: HIGH - Object and array props default values must be returned from a function. Using plain objects/arrays causes all component instances to share the same data reference, leading to state contamination bugs.

## Task Checklist

- [ ] Always use factory functions for object/array prop defaults
- [ ] Return a new object/array for each component instance
- [ ] Never use plain object/array literals for defaults

## Incorrect: Shared Reference

```vue
<script>
export default {
  props: {
    // WRONG: All instances share the same object!
    user: {
      type: Object,
      default: { name: '', email: '' }
    },
    // WRONG: All instances share the same array!
    items: {
      type: Array,
      default: []
    }
  }
}
</script>
```

## Correct: Factory Function

```vue
<script>
export default {
  props: {
    // CORRECT: New object for each instance
    user: {
      type: Object,
      default: function() {
        return { name: '', email: '' }
      }
    },
    // CORRECT: ES6 shorthand for factory function
    items: {
      type: Array,
      default: () => []
    }
  }
}
</script>
```

## Why Factory Functions Are Required

```vue
<script>
// ParentComponent.vue
export default {
  data() {
    return {
      users: ['Alice', 'Bob', 'Charlie']
    }
  },
  template: '<div><UserCard v-for="user in users" :key="user" /></div>'
}

// UserCard.vue with WRONG default
export default {
  props: {
    tags: {
      type: Array,
      default: [] // Shared across all instances!
    }
  }
}

// First UserCard gets pushed to
tags.push('vip')

// Second and third UserCard also have 'vip' tag!
// Because they all share the same array reference
```

## Object Default with Data

```vue
<script>
export default {
  props: {
    // Complex object default
    config: {
      type: Object,
      default: function() {
        return {
          theme: 'light',
          language: 'en',
          notifications: true,
          itemsPerPage: 10
        }
      }
    },
    // Array with default data
    listItems: {
      type: Array,
      default: function() {
        return [
          { id: 1, label: 'Option 1' },
          { id: 2, label: 'Option 2' }
        ]
      }
    }
  }
}
</script>
```

## TypeScript with Vue.extend()

```vue
<script lang="ts">
import Vue from 'vue'

interface Config {
  theme: string
  language: string
  notifications: boolean
}

interface ListItem {
  id: number
  label: string
}

export default Vue.extend({
  name: 'MyComponent',
  props: {
    config: {
      type: Object as PropType<Config>,
      default: (): Config => ({
        theme: 'light',
        language: 'en',
        notifications: true
      })
    },
    listItems: {
      type: Array as PropType<ListItem[]>,
      default: (): ListItem[] => [
        { id: 1, label: 'Option 1' },
        { id: 2, label: 'Option 2' }
      ]
    }
  }
})
</script>
```

## Empty Defaults

```vue
<script>
export default {
  props: {
    // Empty object default
    metadata: {
      type: Object,
      default: () => ({})
    },
    // Empty array default
    tags: {
      type: Array,
      default: () => []
    }
  }
}
</script>
```

## Default with Props Validation

```vue
<script>
export default {
  props: {
    // Factory function + type checking
    user: {
      type: Object,
      // Must match the structure
      default: function() {
        return {
          id: 0,
          name: '',
          email: '',
          role: 'user'
        }
      },
      // Add validation
      validator: function(value) {
        return typeof value.id === 'number' &&
               typeof value.name === 'string' &&
               typeof value.email === 'string'
      }
    }
  }
}
</script>
```

## Nested Objects and Arrays

```vue
<script>
export default {
  props: {
    // Nested object structure
    settings: {
      type: Object,
      default: function() {
        return {
          ui: {
            theme: 'light',
            sidebar: true
          },
          api: {
            baseUrl: '',
            timeout: 5000
          }
        }
      }
    },
    // Array of objects
    rows: {
      type: Array,
      default: function() {
        return [
          { id: 1, selected: false },
          { id: 2, selected: false }
        ]
      }
    }
  }
}
</script>
```

## Common Patterns

```vue
<script>
export default {
  props: {
    // Pagination default
    pagination: {
      type: Object,
      default: () => ({
        page: 1,
        perPage: 10,
        total: 0
      })
    },
    // Filter defaults
    filters: {
      type: Object,
      default: () => ({
        search: '',
        status: 'all',
        sortBy: 'createdAt',
        sortOrder: 'desc'
      })
    },
    // Form field defaults
    formData: {
      type: Object,
      default: () => ({
        username: '',
        email: '',
        age: null
      })
    }
  }
}
</script>
```

## Arrow Function Shorthand

```vue
<script>
export default {
  props: {
    // ES6 arrow function (same as function() {})
    items: {
      type: Array,
      default: () => []
    },
    user: {
      type: Object,
      default: () => ({ name: '', email: '' })
    }
  }
}
</script>
```

## Reference

- [Vue 2 Props - Default Values](https://v2.vuejs.org/v2/guide/components-props.html#Prop-Validation)
