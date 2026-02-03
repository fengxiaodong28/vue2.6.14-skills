---
title: Prop Validation with Custom Validators
impact: MEDIUM
impactDescription: Custom validator functions enable runtime prop type checking beyond native constructors
type: best-practice
tags:
  - vue2.6.14
  - props
  - validation
  - type-checking
  - runtime-validation
---

# Prop Validation with Custom Validators

Impact: MEDIUM - Custom validators allow runtime prop validation beyond native type checking, ensuring data integrity and providing helpful error messages.

## Task Checklist

- [ ] Use validator function for complex validation logic
- [ ] Return `true` for valid, `false` for invalid
- [ ] Provide helpful warning messages
- [ ] Keep validation synchronous and fast

## Basic Custom Validator

```vue
<script>
export default {
  props: {
    age: {
      type: Number,
      validator: function(value) {
        // Must return true for valid, false for invalid
        return value >= 0 && value <= 120
      }
    }
  }
}
</script>
```

## String Validation

```vue
<script>
export default {
  props: {
    email: {
      type: String,
      validator: function(value) {
        // Simple email validation
        return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)
      }
    },
    username: {
      type: String,
      validator: function(value) {
        // Must be 3-20 characters, alphanumeric and underscore only
        return /^[a-zA-Z0-9_]{3,20}$/.test(value)
      }
    }
  }
}
</script>
```

## Array Validation

```vue
<script>
export default {
  props: {
    tags: {
      type: Array,
      validator: function(value) {
        // Must have at least one tag
        return value.length > 0
      }
    },
    scores: {
      type: Array,
      validator: function(value) {
        // All scores must be between 0 and 100
        return value.every(score => score >= 0 && score <= 100)
      }
    }
  }
}
</script>
```

## Object Validation

```vue
<script>
export default {
  props: {
    user: {
      type: Object,
      validator: function(value) {
        // Must have required fields
        return value.id && value.name && value.email
      }
    },
    config: {
      type: Object,
      validator: function(value) {
        // Must have specific keys with valid values
        const validModes = ['development', 'production', 'test']
        return validModes.includes(value.mode)
      }
    }
  }
}
</script>
```

## Multiple Validations Combined

```vue
<script>
export default {
  props: {
    password: {
      type: String,
      required: true,
      // Combine type with validator for comprehensive checks
      validator: function(value) {
        // Must be at least 8 characters
        if (value.length < 8) return false
        // Must contain at least one number
        if (!/\d/.test(value)) return false
        // Must contain at least one letter
        if (!/[a-zA-Z]/.test(value)) return false
        // Must contain at least one special character
        if (!/[!@#$%^&*]/.test(value)) return false
        return true
      }
    }
  }
}
</script>
```

## Accessing Other Props in Validator

```vue
<script>
export default {
  props: {
    minAge: {
      type: Number,
      default: 0
    },
    maxAge: {
      type: Number,
      default: 120
    },
    userAge: {
      type: Number,
      validator: function(value) {
        // Can access other props via `this`
        return value >= this.minAge && value <= this.maxAge
      }
    }
  }
}
</script>
```

## Type Checking + Custom Validator

```vue
<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  props: {
    // Native type check first, then custom validator
    priority: {
      type: Number,
      validator: function(value) {
        return value >= 1 && value <= 10
      }
    },
    status: {
      type: String,
      validator: function(value) {
        // Enum-like validation
        return ['pending', 'active', 'completed', 'failed'].includes(value)
      }
    }
  }
})
</script>
```

## Providing Default Error Messages

The built-in validation warnings don't support custom error messages, but you can add warnings manually:

```vue
<script>
export default {
  props: {
    percentage: {
      type: Number,
      validator: function(value) {
        const isValid = value >= 0 && value <= 100
        if (!isValid) {
          console.warn(
            '[Invalid Prop] percentage must be between 0 and 100'
          )
        }
        return isValid
      }
    }
  }
}
</script>
```

## Async Validation (Not Supported)

```vue
<script>
export default {
  props: {
    // WRONG: Async validators don't work!
    username: {
      type: String,
      validator: async function(value) {
        // This won't work - validators must be synchronous
        const result = await checkUsername(value)
        return result
      }
    }
  }
}
</script>
```

For async validation, use `watch` or `created()` hook:

```vue
<script>
export default {
  props: {
    username: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      validationError: null
    }
  },
  watch: {
    username: {
      immediate: true,
      handler: async function(value) {
        this.validationError = null
        try {
          const isValid = await this.validateUsername(value)
          if (!isValid) {
            this.validationError = 'Username already taken'
            this.$emit('validation-error', 'username')
          }
        } catch (error) {
          console.error('Validation error:', error)
        }
      }
    }
  },
  methods: {
    async validateUsername(username) {
      // Async validation logic
      return true
    }
  }
}
</script>
```

## Reference

- [Vue 2 Props - Custom Validation](https://v2.vuejs.org/v2/guide/components-props.html#Prop-Validation)
