---
title: Typing Complex Props with Interfaces in Vue 2 TypeScript
impact: MEDIUM
impactDescription: PropType enables TypeScript interfaces for complex prop type validation
type: best-practice
tags:
  - vue2.6.14
  - typescript
  - props
  - interfaces
  - proptype
  - vue-extend
---

# Typing Complex Props with Interfaces in Vue 2 TypeScript

Impact: MEDIUM - Using TypeScript interfaces with `PropType` enables proper type checking for complex prop types like objects and arrays in Vue 2 Options API.

## Task Checklist

- [ ] Import `PropType` from 'vue' for complex prop types
- [ ] Define interfaces for complex prop shapes
- [ ] Use `as PropType<Type>` for prop type validation

## Basic Object Prop with Interface

```vue
<script lang="ts">
import Vue from 'vue'
import { PropType } from 'vue'

// Define interface
interface User {
  id: number
  name: string
  email: string
  role: 'admin' | 'user' | 'guest'
}

export default Vue.extend({
  name: 'UserProfile',
  props: {
    // Use PropType for interface types
    user: {
      type: Object as PropType<User>,
      required: true
    }
  }
})
</script>
```

## Array Prop with Interface

```vue
<script lang="ts">
import Vue from 'vue'
import { PropType } from 'vue'

interface MenuItem {
  id: number
  label: string
  icon?: string
  disabled?: boolean
}

export default Vue.extend({
  name: 'MenuList',
  props: {
    items: {
      type: Array as PropType<MenuItem[]>,
      required: true,
      default: () => []
    }
  }
})
</script>
```

## Union Types

```vue
<script lang="ts">
import Vue from 'vue'
import { PropType } from 'vue'

type Theme = 'light' | 'dark' | 'auto'
type Size = 'small' | 'medium' | 'large'

export default Vue.extend({
  name: 'ThemedButton',
  props: {
    theme: {
      type: String as PropType<Theme>,
      default: 'auto'
    },
    size: {
      type: String as PropType<Size>,
      default: 'medium'
    }
  }
})
</script>
```

## Function Prop Type

```vue
<script lang="ts">
import Vue from 'vue'
import { PropType } from 'vue'

type ValidatorFunction = (value: string) => boolean

export default Vue.extend({
  name: 'FormInput',
  props: {
    validator: {
      type: Function as PropType<ValidatorFunction>,
      required: false
    }
  },
  methods: {
    validateInput(value: string): boolean {
      if (this.validator) {
        return this.validator(value)
      }
      return true
    }
  }
})
</script>
```

## Complex Nested Types

```vue
<script lang="ts">
import Vue from 'vue'
import { PropType } from 'vue'

interface Address {
  street: string
  city: string
  state: string
  zipCode: string
}

interface Company {
  name: string
  address: Address
}

interface Employee {
  id: number
  name: string
  email: string
  company?: Company
  roles: string[]
}

export default Vue.extend({
  name: 'EmployeeCard',
  props: {
    employee: {
      type: Object as PropType<Employee>,
      required: true
    }
  },
  computed: {
    companyName(): string {
      return this.employee.company?.name || 'No company'
    }
  }
})
</script>
```

## Array of Objects with Default Factory

```vue
<script lang="ts">
import Vue from 'vue'
import { PropType } from 'vue'

interface Tab {
  id: string
  label: string
  content: string
}

export default Vue.extend({
  name: 'TabPanel',
  props: {
    tabs: {
      type: Array as PropType<Tab[]>,
      // Default must be a function for arrays/objects
      default: (): Tab[] => [
        { id: '1', label: 'Tab 1', content: 'Content 1' }
      ]
    },
    activeTab: {
      type: String as PropType<string>,
      default: '1'
    }
  }
})
</script>
```

## Record/Dictionary Type

```vue
<script lang="ts">
import Vue from 'vue'
import { PropType } from 'vue'

export default Vue.extend({
  name: 'DataGrid',
  props: {
    // Record type for dictionary-like objects
    data: {
      type: Object as PropType<Record<string, string | number>>,
      required: true
    }
  },
  methods: {
    getValue(key: string): string | number {
      return this.data[key] || ''
    }
  }
})
</script>
```

## Generic Component Pattern (with Class Component)

```vue
<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator'

interface SelectOption<T> {
  value: T
  label: string
}

@Component
export default class Select<T> extends Vue {
  @Prop({ type: Array, required: true })
  options!: SelectOption<T>[]

  @Prop()
  value!: T
}
</script>
```

## Reference

- [Vue 2 TypeScript Support](https://vue2.docs.vuejs.org/v2/guide/typescript.html)
- [TypeScript - PropType](https://github.com/vuejs/vue-class-component/blob/master/README.md)
