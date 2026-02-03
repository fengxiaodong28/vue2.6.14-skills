---
title: PropType for Complex Prop Types in Vue 2 TypeScript
impact: MEDIUM
impactDescription: PropType enables proper TypeScript type checking for complex prop types
type: best-practice
tags:
  - vue2
  - typescript
  - props
  - proptype
  - complex-types
---

# PropType for Complex Prop Types in Vue 2 TypeScript

Impact: MEDIUM - Using `PropType` enables full TypeScript type checking for complex prop types like objects, arrays, unions, and more.

## Task Checklist

- [ ] Import `PropType` from `vue`
- [ ] Use `as PropType<T>()` for complex type annotations
- [ ] Define interfaces for object props
- [ ] Use union types for multiple allowed types
- [ ] Use array literal types for typed arrays

## Basic PropType Usage

```vue
<script lang="ts">
import Vue from 'vue'
import { PropType } from 'vue'

interface User {
  id: number
  name: string
  email: string
}

export default Vue.extend({
  name: 'UserProfile',
  props: {
    // ✅ CORRECT: Use PropType for object types
    user: {
      type: Object as PropType<User>,
      required: true
    },

    // ✅ CORRECT: Array of specific type
    items: {
      type: Array as PropType<number[]>,
      default: () => []
    },

    // ✅ CORRECT: Array of objects
    users: {
      type: Array as PropType<User[]>,
      default: () => []
    }
  }
})
</script>
```

## Object Props with Interfaces

```vue
<script lang="ts">
import Vue from 'vue'
import { PropType } from 'vue'

interface Todo {
  id: number
  text: string
  completed: boolean
  priority: 'low' | 'medium' | 'high'
}

interface TodoFilter {
  status?: 'all' | 'active' | 'completed'
  priority?: string
}

export default Vue.extend({
  name: 'TodoList',
  props: {
    todo: {
      type: Object as PropType<Todo>,
      required: true
    },

    filter: {
      type: Object as PropType<TodoFilter>,
      default: () => ({})
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
type Alignment = 'left' | 'center' | 'right'

export default Vue.extend({
  name: 'ThemedComponent',
  props: {
    // String literal union
    theme: {
      type: String as PropType<Theme>,
      default: 'auto'
    },

    // Union of multiple types
    value: {
      type: [String, Number] as PropType<string | number>,
      default: ''
    },

    // Array of union types
    sizes: {
      type: Array as PropType<Size[]>,
      default: () => ['medium']
    }
  }
})
</script>
```

## Function Props

```vue
<script lang="ts">
import Vue from 'vue'
import { PropType } from 'vue'

type ValidatorFunction = (value: any) => boolean
type AsyncHandler<T> = (data: T) => Promise<void>
type EventCallback = (event: Event) => void

export default Vue.extend({
  name: 'HandlerComponent',
  props: {
    // Function type
    validator: {
      type: Function as PropType<ValidatorFunction>,
      required: true
    },

    // Async function
    onSubmit: {
      type: Function as PropType<AsyncHandler<{ name: string }>>,
      required: true
    },

    // Event handler
    onClick: {
      type: Function as PropType<EventCallback>,
      default: () => {}
    }
  }
})
</script>
```

## Record and Dictionary Types

```vue
<script lang="ts">
import Vue from 'vue'
import { PropType } from 'vue'

export default Vue.extend({
  name: 'DictComponent',
  props: {
    // Record type (string keys, string values)
    headers: {
      type: Object as PropType<Record<string, string>>,
      default: () => ({})
    },

    // Dictionary with specific value type
    counts: {
      type: Object as PropType<Record<string, number>>,
      default: () => ({})
    },

    // Dynamic key-value pairs
    metadata: {
      type: Object as PropType<{ [key: string]: any }>,
      default: () => ({})
    }
  }
})
</script>
```

## Array of Complex Types

```vue
<script lang="ts">
import Vue from 'vue'
import { PropType } from 'vue'

interface SelectOption {
  label: string
  value: string | number
  disabled?: boolean
  group?: string
}

interface TableColumn {
  key: string
  label: string
  width?: string | number
  sortable?: boolean
}

export default Vue.extend({
  name: 'DataTable',
  props: {
    // Array of complex objects
    options: {
      type: Array as PropType<SelectOption[]>,
      default: () => []
    },

    // Array with required items
    columns: {
      type: Array as PropType<TableColumn[]>,
      required: true
    },

    // Readonly array
    fixedItems: {
      type: Array as PropType<ReadonlyArray<string>>,
      default: () => []
    }
  }
})
</script>
```

## Nested Object Types

```vue
<script lang="ts">
import Vue from 'vue'
import { PropType } from 'vue'

interface Address {
  street: string
  city: string
  state: string
  zip: string
  country: string
}

interface ContactInfo {
  email: string
  phone?: string
  address?: Address
}

interface UserProfile {
  id: number
  name: string
  contact: ContactInfo
  preferences: {
    theme: string
    notifications: boolean
    language: string
  }
}

export default Vue.extend({
  name: 'UserCard',
  props: {
    profile: {
      type: Object as PropType<UserProfile>,
      required: true
    }
  }
})
</script>
```

## Enum Types

```vue
<script lang="ts">
import Vue from 'vue'
import { PropType } from 'vue'

enum Status {
  Pending = 'pending',
  Active = 'active',
  Inactive = 'inactive',
  Suspended = 'suspended'
}

enum Priority {
  Low = 1,
  Medium = 2,
  High = 3,
  Urgent = 4
}

export default Vue.extend({
  name: 'StatusBadge',
  props: {
    status: {
      type: String as PropType<Status>,
      default: Status.Pending
    },

    priority: {
      type: Number as PropType<Priority>,
      default: Priority.Medium
    }
  }
})
</script>
```

## Generic Component Pattern

```vue
<script lang="ts">
import Vue, { VNode } from 'vue'
import { PropType } from 'vue'

// Generic type for list items
interface ListProps<T> {
  items: T[]
  keyField: keyof T
  renderItem: (item: T, index: number) => VNode
}

export default Vue.extend({
  name: 'GenericList',
  props: {
    items: {
      type: Array as PropType<any[]>,
      default: () => []
    },
    itemKey: {
      type: String as PropType<string>,
      required: true
    }
  }
})
</script>
```

## Custom Validator with PropType

```vue
<script lang="ts">
import Vue from 'vue'
import { PropType } from 'vue'

interface Config {
  min: number
  max: number
  step: number
}

export default Vue.extend({
  name: 'RangeSlider',
  props: {
    config: {
      type: Object as PropType<Config>,
      required: true,
      validator: (value: Config) => {
        return value.min < value.max && value.step > 0
      }
    }
  }
})
</script>
```

## Optional and Readonly Props

```vue
<script lang="ts">
import Vue from 'vue'
import { PropType } from 'vue'

export default Vue.extend({
  name: 'OptionalProps',
  props: {
    // Optional prop with default
    optionalUser: {
      type: Object as PropType<{ name: string }>,
      default: () => ({ name: 'Guest' })
    },

    // Nullable prop
    nullableValue: {
      type: [String, null] as PropType<string | null>,
      default: null
    },

    // Readonly prop (convention, not enforced)
    readOnlyConfig: {
      type: Object as PropType<Readonly<{ id: string }>>,
      required: true
    }
  }
})
</script>
```

## Complex Default Values

```vue
<script lang="ts">
import Vue from 'vue'
import { PropType } from 'vue'

interface FormConfig {
  fields: Array<{
    name: string
    type: string
    required: boolean
  }>
  validation: {
    enabled: boolean
    rules: Record<string, any>
  }
}

export default Vue.extend({
  name: 'DynamicForm',
  props: {
    config: {
      type: Object as PropType<FormConfig>,
      default: () => ({
        fields: [],
        validation: {
          enabled: false,
          rules: {}
        }
      })
    }
  }
})
</script>
```

## Type Guards with Props

```vue
<script lang="ts">
import Vue from 'vue'
import { PropType } from 'vue'

interface ImageData {
  type: 'image'
  url: string
  alt: string
}

interface TextData {
  type: 'text'
  content: string
}

type ContentData = ImageData | TextData

export default Vue.extend({
  name: 'ContentRenderer',
  props: {
    data: {
      type: Object as PropType<ContentData>,
      required: true
    }
  },
  computed: {
    isImage(): boolean {
      return this.data.type === 'image'
    },
    isText(): boolean {
      return this.data.type === 'text'
    }
  }
})
</script>
```

## Common PropType Patterns

```vue
<script lang="ts">
import Vue from 'vue'
import { PropType } from 'vue'

export default Vue.extend({
  props: {
    // Tuple
    coordinates: {
      type: Array as PropType<[number, number]>,
      required: true
    },

    // Promise
    dataLoader: {
      type: Function as PropType<() => Promise<any>>,
      required: true
    },

    // Map/Set
    tagMap: {
      type: Object as PropType<Map<string, number>>,
      default: () => new Map()
    },

    // Complex union
    variant: {
      type: [String, Object] as PropType<'primary' | 'secondary' | { custom: string }>,
      default: 'primary'
    }
  }
})
</script>
```

## Reference

- [Vue 2 TypeScript - Annotating Props](https://v2.vuejs.org/v2/guide/typescript.html#Annotating-Props)
- [Vue 2 API - Prop Options](https://v2.vuejs.org/v2/guide/components-props.html#Prop-Validation)
