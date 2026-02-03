---
title: Vue Class Component Basics for Vue 2
impact: HIGH
impactDescription: vue-class-component@7.x enables class-based components with decorators
type: best-practice
tags:
  - vue2
  - class-component
  - typescript
  - decorators
  - vue-property-decorator
---

# Vue Class Component Basics for Vue 2

Impact: HIGH - vue-class-component@7.x enables class-based component definition with TypeScript decorators for Vue 2.6.14.

## Task Checklist

- [ ] Install vue-class-component@7.x and vue-property-decorator
- [ ] Use @Component decorator to define components
- [ ] Define data as class properties
- [ ] Use @Prop decorator for props
- [ ] Define methods as class methods

## Installation

```bash
# For Vue 2.6.14
npm install vue-class-component@^7.2.6
npm install vue-property-decorator@^9.1.2

# TypeScript types
npm install --save-dev @types/node

# ❌ WRONG: vue-facing-decorator is for Vue 3
```

## Basic Component

```vue
<template>
  <div>
    <h1>{{ title }}</h1>
    <p>Count: {{ count }}</p>
    <button @click="increment">Increment</button>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'

@Component
export default class MyComponent extends Vue {
  // Data - class properties
  title: string = 'Hello World'
  count: number = 0

  // Methods - class methods
  increment(): void {
    this.count++
  }
}
</script>
```

## Props with Decorators

```vue
<script lang="ts">
import { Component, Vue, Prop } from 'vue-property-decorator'

@Component
export default class UserCard extends Vue {
  // Basic prop
  @Prop(String) readonly name!: string

  // Prop with default value
  @Prop({ default: 'Guest' }) readonly title!: string

  // Required prop
  @Prop({ required: true }) readonly userId!: number

  // Prop with type
  @Prop({ type: Boolean, default: false }) readonly isActive!: boolean

  // Object prop with default factory
  @Prop({
    type: Object,
    default: () => ({ id: 0, name: '' })
  }) readonly user!: { id: number; name: string }

  // Array prop with default factory
  @Prop({
    type: Array,
    default: () => []
  }) readonly items!: string[]

  // Custom validator
  @Prop({
    type: Number,
    validator: (value) => value >= 0 && value <= 100
  }) readonly percentage!: number
}
</script>
```

## Computed Properties

```vue
<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'

@Component
export default class Calculator extends Vue {
  count: number = 0
  multiplier: number = 2

  // Computed getter
  get double(): number {
    return this.count * 2
  }

  // Computed with getter and setter
  get fullName(): string {
    return `${this.firstName} ${this.lastName}`
  }

  set fullName(value: string) {
    const [first, last] = value.split(' ')
    this.firstName = first
    this.lastName = last
  }

  firstName: string = 'John'
  lastName: string = 'Doe'
}
</script>
```

## Watch Decorator

```vue
<script lang="ts">
import { Component, Vue, Watch } from 'vue-property-decorator'

@Component
export default class UserProfile extends Vue {
  userId: number = 0
  user: any = null
  loading: boolean = false

  // Watch property
  @Watch('userId')
  onUserIdChanged(newVal: number, oldVal: number): void {
    console.log(`User ID changed: ${oldVal} -> ${newVal}`)
    this.fetchUser(newVal)
  }

  // Watch with immediate option
  @Watch('userId', { immediate: true })
  onUserIdImmediate(newVal: number): void {
    if (newVal) {
      this.fetchUser(newVal)
    }
  }

  // Watch with deep option
  @Watch('user', { deep: true })
  onUserChanged(newVal: any, oldVal: any): void {
    console.log('User object changed:', newVal)
  }

  async fetchUser(id: number): Promise<void> {
    this.loading = true
    try {
      const response = await fetch(`/api/users/${id}`)
      this.user = await response.json()
    } finally {
      this.loading = false
    }
  }
}
</script>
```

## Lifecycle Hooks as Class Methods

```vue
<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'

@Component
export default class LifecycleDemo extends Vue {
  data: any = null

  // Lifecycle hooks as methods
  created(): void {
    console.log('Component created')
    this.fetchData()
  }

  mounted(): void {
    console.log('Component mounted')
  }

  beforeDestroy(): void {
    console.log('Component before destroy') // Vue 2 naming
  }

  destroyed(): void {
    console.log('Component destroyed') // Vue 2 naming
  }

  beforeUpdate(): void {
    console.log('Component before update')
  }

  updated(): void {
    console.log('Component updated')
  }

  async fetchData(): Promise<void> {
    // Fetch data
  }
}
</script>
```

## Emit Decorator

```vue
<script lang="ts">
import { Component, Vue, Emit } from 'vue-property-decorator'

@Component
export default class FormInput extends Vue {
  value: string = ''

  // Simple emit
  @Emit('submit')
  handleSubmit() {
    // Return value is emitted as payload
    return {
      value: this.value,
      timestamp: Date.now()
    }
  }

  // Emit with computed event name
  @Emit()
  input() {
    // Event name is 'input' (method name)
    return this.value
  }

  // Emit without return (event only)
  @Emit()
  cancel() {
    // No return, just emits 'cancel' event
  }

  updateValue(newValue: string): void {
    this.value = newValue
    this.input() // Emits input event
  }
}
</script>
```

## Ref Decorator

```vue
<template>
  <div>
    <input ref="inputRef" />
    <button @click="focusInput">Focus</button>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Ref } from 'vue-property-decorator'

@Component
export default class RefDemo extends Vue {
  // Template ref with type
  @Ref('inputRef') readonly inputRef!: HTMLInputElement

  focusInput(): void {
    this.inputRef.focus()
    this.inputRef.value = 'Hello'
  }
}
</script>
```

## Provide/Inject

```vue
<!-- Parent.vue -->
<script lang="ts">
import { Component, Vue, Provide } from 'vue-property-decorator'

@Component
export default class ParentComponent extends Vue {
  theme: string = 'light'
  config: any = {
    apiUrl: 'https://api.example.com',
    timeout: 5000
  }

  // Provide value
  @Provide('theme')
  provideTheme(): string {
    return this.theme
  }

  // Provide reactive object
  @Provide('config')
  provideConfig(): any {
    return this.config
  }

  // Provide with symbol
  @Provide(Symbol('userInfo'))
  provideUserInfo() {
    return { name: 'John', id: 123 }
  }
}
</script>
```

```vue
<!-- Child.vue -->
<script lang="ts">
import { Component, Vue, Inject } from 'vue-property-decorator'

@Component
export default class ChildComponent extends Vue {
  // Inject value
  @Inject('theme') readonly theme!: string

  // Inject with default value
  @Inject({ from: 'theme', default: 'light' }) readonly themeWithDefault!: string

  // Inject reactive object
  @Inject('config') readonly config!: any

  // Inject with symbol
  @Inject(Symbol('userInfo')) readonly userInfo!: { name: string; id: number }

  mounted(): void {
    console.log(this.theme)
    console.log(this.config)
  }
}
</script>
```

## Model Decorator (Custom v-model)

```vue
<script lang="ts">
import { Component, Vue, Model, Emit } from 'vue-property-decorator'

@Component
export default class CustomInput extends Vue {
  // Define v-model prop and event
  @Model('change', { type: String, default: '' }) readonly value!: string

  // Emit when value changes
  @Emit('change')
  updateValue(newValue: string): string {
    return newValue
  }

  // Or use method
  onInput(event: Event): void {
    const target = event.target as HTMLInputElement
    this.updateValue(target.value)
  }
}
</script>

<template>
  <input :value="value" @input="onInput" />
</template>
```

## Mixins

```typescript
// mixins/ValidationMixin.ts
import { Vue, Component } from 'vue-property-decorator'

@Component
export default class ValidationMixin extends Vue {
  errors: string[] = []

  validateRequired(value: any): boolean {
    if (!value) {
      this.errors.push('This field is required')
      return false
    }
    return true
  }

  validateEmail(email: string): boolean {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
    return emailRegex.test(email)
  }

  clearErrors(): void {
    this.errors = []
  }
}
```

```vue
<script lang="ts">
import { Component, Vue, Mixins } from 'vue-property-decorator'
import ValidationMixin from '@/mixins/ValidationMixin'

// Extend with mixins
@Component
export default class MyForm extends Mixins(ValidationMixin) {
  email: string = ''
  password: string = ''

  submitForm(): void {
    this.clearErrors()

    if (!this.validateRequired(this.email)) {
      return
    }

    if (!this.validateEmail(this.email)) {
      this.errors.push('Invalid email')
      return
    }
  }
}
</script>
```

## Component Registration

```vue
<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
import ChildComponent from '@/components/ChildComponent.vue'

@Component({
  // Register components
  components: {
    ChildComponent
  }
})
export default class ParentComponent extends Vue {
  // Component logic
}
</script>
```

## Full Example with TypeScript

```vue
<template>
  <div class="user-card">
    <h2>{{ user.name }}</h2>
    <p>Email: {{ user.email }}</p>
    <p>Status: {{ isActive ? 'Active' : 'Inactive' }}</p>
    <button @click="toggleStatus">Toggle Status</button>
    <button @click="save">Save</button>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Prop, Emit } from 'vue-property-decorator'

interface User {
  id: number
  name: string
  email: string
}

@Component
export default class UserCard extends Vue {
  // Props
  @Prop({ type: Object, required: true }) readonly user!: User
  @Prop({ type: Boolean, default: false }) readonly isActive!: boolean

  // Local state
  loading: boolean = false

  // Computed
  get displayName(): string {
    return this.user.name || 'Unknown User'
  }

  // Watch
  @Watch('user', { immediate: true, deep: true })
  onUserChanged(user: User): void {
    console.log('User changed:', user)
  }

  // Methods
  toggleStatus(): void {
    this.$emit('toggle', this.user.id)
  }

  @Emit('save')
  save(): User {
    return this.user
  }

  // Lifecycle
  mounted(): void {
    console.log('UserCard mounted')
  }
}
</script>

<style scoped>
.user-card {
  padding: 16px;
  border: 1px solid #ddd;
  border-radius: 8px;
}
</style>
```

## Reference

- [vue-class-component Documentation](https://class-component.vuejs.org/)
- [vue-property-decorator Documentation](https://github.com/kaorun343/vue-property-decorator)
