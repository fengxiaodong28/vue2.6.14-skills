---
title: Type-Safe Computed Properties in Vue 2 TypeScript
impact: LOW
impactDescription: Computed properties need explicit return type annotations for full TypeScript support
type: best-practice
tags:
  - vue2
  - typescript
  - computed
  - return-type
  - type-annotations
---

# Type-Safe Computed Properties in Vue 2 TypeScript

Impact: LOW - Computed properties in Vue 2 TypeScript benefit from explicit return type annotations for full type safety and autocomplete.

## Task Checklist

- [ ] Add explicit return type annotations to computed properties
- [ ] Use proper type annotations for getters and setters
- [ ] Leverage `this` context typing in computed properties

## Basic Computed Property with Type

```vue
<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'UserProfile',
  data() {
    return {
      user: {
        firstName: 'John',
        lastName: 'Doe'
      }
    }
  },
  computed: {
    // Explicit return type annotation
    fullName(): string {
      return `${this.user.firstName} ${this.user.lastName}`
    },

    // Boolean return type
    isValid(): boolean {
      return this.user.firstName !== '' && this.user.lastName !== ''
    },

    // Number return type
    age(): number {
      return this.user.age || 0
    }
  }
})
</script>
```

## Computed Property with Getter and Setter

```vue
<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'FormInput',
  props: {
    value: {
      type: String,
      required: true
    }
  },
  computed: {
    inputValue: {
      // Getter with return type
      get(): string {
        return this.value
      },
      // Setter with parameter type
      set(newValue: string): void {
        this.$emit('input', newValue)
      }
    }
  }
})
</script>
```

## Object Type Computed Property

```vue
<script lang="ts">
import Vue from 'vue'

interface User {
  id: number
  name: string
  email: string
}

interface FormState {
  isValid: boolean
  errors: string[]
}

export default Vue.extend({
  name: 'UserForm',
  data() {
    return {
      user: {} as User,
      formState: {} as FormState
    }
  },
  computed: {
    // Object return type
    userSummary(): { name: string; email: string } {
      return {
        name: this.user.name,
        email: this.user.email
      }
    },

    // Interface return type
    formValidation(): FormState {
      const errors = []
      if (!this.user.name) errors.push('Name required')
      if (!this.user.email) errors.push('Email required')

      return {
        isValid: errors.length === 0,
        errors
      }
    }
  }
})
</script>
```

## Array Type Computed Property

```vue
<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'TodoList',
  data() {
    return {
      todos: [
        { id: 1, text: 'Learn Vue 2', done: false },
        { id: 2, text: 'Build something', done: false }
      ]
    }
  },
  computed: {
    // Array return type
    activeTodos(): { id: number; text: string }[] {
      return this.todos.filter(todo => !todo.done)
    },

    // Number return type
    activeCount(): number {
      return this.activeTodos.length
    }
  }
})
</script>
```

## Async Computed Properties (Not Supported)

```vue
<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'MyComponent',
  data() {
    return {
      userId: 123
    }
  },
  computed: {
    // WRONG: Computed cannot be async!
    userData: async function(): Promise<User> {
      const response = await fetch(`/api/users/${this.userId}`)
      return await response.json()
    },

    // CORRECT: Use method instead
    fetchUserData(): Promise<void> {
      // Async logic here
    }
  },
  created() {
    this.fetchUserData()
  }
})
</script>
```

## Computed Properties with Dependencies

```vue
<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'Calculator',
  data() {
    return {
      a: 10,
      b: 20,
      c: 30
    }
  },
  computed: {
    // Computed with clear dependencies
    sum(): number {
      return this.a + this.b + this.c
    },
    // Computed that uses another computed
    doubledSum(): number {
      return this.sum * 2
    }
  }
})
</script>
```

## Complex Computed with Destructuring

```vue
<script lang="ts">
import Vue from 'vue'

interface Address {
  city: string
  state: string
  zip: string
}

export default Vue.extend({
  name: 'LocationDisplay',
  data() {
    return {
      address: {
        city: 'New York',
        state: 'NY',
        zip: '10001'
      } as Address
    }
  },
  computed: {
    // Destructuring in computed
    location(): string {
      const { city, state } = this.address
      return `${city}, ${state}`
    },

    // Computed with array destructuring
    firstItem(): string | undefined {
      const [first] = this.items
      return first
    }
  }
})
</script>
```

## Computed with Watcher Combination

```vue
<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'SearchComponent',
  data() {
    return {
      searchQuery: '',
      allItems: [] as Item[],
      filteredItems: [] as Item[]
    }
  },
  computed: {
    // Computed for filtering
    matchingItems(): Item[] {
      if (!this.searchQuery) return this.allItems
      return this.allItems.filter(item =>
        item.name.toLowerCase().includes(this.searchQuery.toLowerCase())
      )
    }
  },
  watch: {
    // Watch computed property
    matchingItems: {
      handler(newItems: Item[]) {
        this.filteredItems = newItems
      },
      immediate: true
    }
  }
})
</script>
```

## Cached Behavior Note

Remember: Computed properties are cached based on reactive dependencies:

```vue
<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'CachedExample',
  data() {
    return {
      items: [] as string[]
    }
  },
  computed: {
    // This is cached - won't re-evaluate unless items change
    itemCount(): number {
      console.log('Computing item count')
      return this.items.length
    }
  },
  methods: {
    addItems() {
      // Adding items triggers re-computation
      this.items.push('item1', 'item2')
    },
    getItems() {
      // Calling this twice - second time uses cache
      console.log(this.itemCount) // First call - computes
      console.log(this.itemCount) // Second call - cached!
    }
  }
})
</script>
```

## Reference

- [Vue 2 TypeScript - Computed Properties](https://v2.vuejs.org/v2/guide/typescript.html#Annotating-Return-Types)
