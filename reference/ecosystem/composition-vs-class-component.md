---
title: Composition API vs Class Component in Vue 2
impact: LOW
impactDescription: Comparing Composition API and Class Component approaches for Vue 2
type: comparison
tags:
  - vue2.6.14
  - composition-api
  - class-component
  - comparison
  - choosing-approach
---

# Composition API vs Class Component in Vue 2

Impact: LOW - Understanding the differences between @vue/composition-api and vue-class-component helps choose the right approach for Vue 2.6.14 projects.

## Task Checklist

- [ ] Understand Composition API pros and cons
- [ ] Understand Class Component pros and cons
- [ ] Choose based on project requirements
- [ ] Consider TypeScript integration needs
- [ ] Evaluate team expertise

## Quick Comparison

| Aspect | Composition API | Class Component |
|--------|----------------|-----------------|
| Syntax | Function-based | Class-based with decorators |
| TypeScript Support | Good | Excellent (native classes) |
| Learning Curve | Medium for Vue 2 devs | Medium for JS devs |
| Code Organization | Composables | Mixins/Inheritance |
| IDE Support | Improving | Excellent (classes) |
| Reusability | Composable functions | Mixins/Classes |
| Vue 3 Migration | Easier | Harder |

## Same Logic - Both Approaches

### Composition API Version

```vue
<script lang="ts">
import { ref, computed, watch, onMounted } from '@vue/composition-api'

interface User {
  id: number
  name: string
  email: string
}

export default {
  props: {
    userId: {
      type: Number,
      required: true
    }
  },
  setup(props, { emit }) {
    const user = ref<User | null>(null)
    const loading = ref(false)
    const error = ref<string | null>(null)

    const displayName = computed(() => {
      return user.value?.name || 'Unknown User'
    })

    const fetchUser = async (id: number) => {
      loading.value = true
      error.value = null

      try {
        const response = await fetch(`/api/users/${id}`)
        if (!response.ok) throw new Error('User not found')
        user.value = await response.json()
      } catch (err) {
        error.value = err.message
      } finally {
        loading.value = false
      }
    }

    watch(
      () => props.userId,
      (newId) => {
        fetchUser(newId)
      },
      { immediate: true }
    )

    const handleSubmit = () => {
      emit('submit', user.value)
    }

    return {
      user,
      loading,
      error,
      displayName,
      handleSubmit
    }
  }
}
</script>
```

### Class Component Version

```vue
<script lang="ts">
import { Component, Vue, Prop, Watch, Emit } from 'vue-property-decorator'

interface User {
  id: number
  name: string
  email: string
}

@Component
export default class UserProfile extends Vue {
  @Prop({ type: Number, required: true }) readonly userId!: number

  user: User | null = null
  loading: boolean = false
  error: string | null = null

  get displayName(): string {
    return this.user?.name || 'Unknown User'
  }

  async fetchUser(id: number): Promise<void> {
    this.loading = true
    this.error = null

    try {
      const response = await fetch(`/api/users/${id}`)
      if (!response.ok) throw new Error('User not found')
      this.user = await response.json()
    } catch (err) {
      this.error = err.message
    } finally {
      this.loading = false
    }
  }

  @Watch('userId', { immediate: true })
  onUserIdChanged(newId: number): void {
    this.fetchUser(newId)
  }

  @Emit('submit')
  handleSubmit(): User | null {
    return this.user
  }
}
</script>
```

## Reusable Logic Comparison

### Composition API: Composable

```javascript
// composables/useCounter.js
import { ref, computed } from '@vue/composition-api'

export function useCounter(initial = 0) {
  const count = ref(initial)
  const double = computed(() => count.value * 2)

  const increment = () => count.value++
  const decrement = () => count.value--
  const reset = () => count.value = initial

  return {
    count,
    double,
    increment,
    decrement,
    reset
  }
}
```

### Class Component: Mixin

```typescript
// mixins/CounterMixin.ts
import { Vue, Component } from 'vue-property-decorator'

@Component
export default class CounterMixin extends Vue {
  count: number = 0

  get double(): number {
    return this.count * 2
  }

  increment(): void {
    this.count++
  }

  decrement(): void {
    this.count--
  }

  reset(): void {
    this.count = 0
  }
}
```

## Props and Events Comparison

### Composition API

```vue
<script>
export default {
  props: {
    value: String,
    items: Array,
    config: {
      type: Object,
      default: () => ({})
    }
  },
  setup(props, { emit }) {
    // Access props
    console.log(props.value)

    // Emit events
    const update = (newValue) => {
      emit('input', newValue)
      emit('change', newValue)
    }

    return { update }
  }
}
</script>
```

### Class Component

```vue
<script lang="ts">
import { Component, Vue, Prop, Emit } from 'vue-property-decorator'

@Component
export default class MyComponent extends Vue {
  @Prop(String) readonly value!: string
  @Prop(Array) readonly items!: any[]
  @Prop({ default: () => ({}) }) readonly config!: any

  @Emit('input')
  @Emit('change')
  update(newValue: string) {
    return newValue
  }
}
</script>
```

## Computed Properties Comparison

### Composition API

```vue
<script>
import { ref, computed } from '@vue/composition-api'

export default {
  setup() {
    const firstName = ref('John')
    const lastName = ref('Doe')

    const fullName = computed(() => `${firstName.value} ${lastName.value}`)

    const fullNameWritable = computed({
      get: () => `${firstName.value} ${lastName.value}`,
      set: (value) => {
        [firstName.value, lastName.value] = value.split(' ')
      }
    })

    return {
      firstName,
      lastName,
      fullName,
      fullNameWritable
    }
  }
}
</script>
```

### Class Component

```vue
<script lang="ts">
import { Vue, Component } from 'vue-property-decorator'

@Component
export default class NameDisplay extends Vue {
  firstName: string = 'John'
  lastName: string = 'Doe'

  get fullName(): string {
    return `${this.firstName} ${this.lastName}`
  }

  set fullName(value: string) {
    [this.firstName, this.lastName] = value.split(' ')
  }
}
</script>
```

## Watchers Comparison

### Composition API

```vue
<script>
import { ref, watch } from '@vue/composition-api'

export default {
  setup() {
    const count = ref(0)
    const user = ref({ name: 'John' })

    watch(count, (newVal, oldVal) => {
      console.log(`Count: ${oldVal} -> ${newVal}`)
    })

    watch(
      () => user.value.name,
      (newName, oldName) => {
        console.log(`Name: ${oldName} -> ${newName}`)
      }
    )

    return { count, user }
  }
}
</script>
```

### Class Component

```vue
<script lang="ts">
import { Component, Vue, Watch } from 'vue-property-decorator'

@Component
export default class WatchDemo extends Vue {
  count: number = 0
  user: any = { name: 'John' }

  @Watch('count')
  onCountChanged(newVal: number, oldVal: number) {
    console.log(`Count: ${oldVal} -> ${newVal}`)
  }

  @Watch('user.name')
  onNameChanged(newName: string, oldName: string) {
    console.log(`Name: ${oldName} -> ${newName}`)
  }
}
</script>
```

## Lifecycle Hooks Comparison

### Composition API

```vue
<script>
import {
  onMounted,
  onBeforeUnmount,
  onUpdated
} from '@vue/composition-api'

export default {
  setup() {
    onMounted(() => {
      console.log('Mounted')
    })

    onUpdated(() => {
      console.log('Updated')
    })

    onBeforeUnmount(() => {
      console.log('Before unmount')
    })

    return {}
  }
}
</script>
```

### Class Component

```vue
<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'

@Component
export default class LifecycleDemo extends Vue {
  mounted() {
    console.log('Mounted')
  }

  updated() {
    console.log('Updated')
  }

  beforeDestroy() {
    console.log('Before destroy')
  }
}
</script>
```

## When to Use Which

### Use Composition API When:

- Planning to migrate to Vue 3 in the future
- Prefer functional programming style
- Need flexible code organization with composables
- Team is comfortable with hooks-based patterns
- Want Vue 3 ecosystem compatibility

```javascript
// Better for composable logic
export function useForm() {
  // Reusable form logic
}

export function usePagination() {
  // Reusable pagination logic
}
```

### Use Class Component When:

- Heavy TypeScript usage (native class types)
- Team has OOP/class-based background
- Using decorators extensively
- Need strong IDE autocomplete
- Prefer traditional class inheritance

```typescript
// Better for typed class definitions
@Component
export class UserService extends Vue {
  @Prop() readonly userId!: number

  // Strong typing throughout
  user: User | null = null
}
```

## Mixing Both Approaches

You can use both in the same project:

```vue
<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
import { useCounter } from '@/composables/useCounter'

@Component
export default class MixedComponent extends Vue {
  // Class-based for component structure
  title: string = 'Counter'

  // Composition API for reusable logic
  setup() {
    const { count, increment, decrement } = useCounter()
    return { count, increment, decrement }
  }

  // Class-based for component-specific logic
  handleReset(): void {
    this.count = 0
  }
}
</script>
```

## Migration Considerations

### Vue 2 → Vue 3 Migration Path

```javascript
// Vue 2 with Composition API
import { ref } from '@vue/composition-api'

// Vue 3 (mostly compatible)
import { ref } from 'vue'
```

Class components require more migration work to Vue 3:

```typescript
// Vue 2
@Component
class MyComponent extends Vue {
  // Need to convert to Composition API or script setup
}
```

## Reference

- [@vue/composition-api GitHub](https://github.com/vuejs/composition-api)
- [vue-class-component Documentation](https://class-component.vuejs.org/)
- [Vue 3 Composition API RFC](https://vue-composition-api-rfc.netlify.app/)
