---
title: TypeScript Type Improvements in Vue 2.6
impact: MEDIUM
impactDescription: Vue 2.6 introduced better TypeScript support with improved type inference, ThisType support, and enhanced component type definitions
type: capability
tags:
  - vue2.6.14
  - typescript
  - type-inference
  - thistype
  - vue-extend
  - component-types
---

# TypeScript Type Improvements in Vue 2.6

Impact: MEDIUM - Vue 2.6 introduced significant improvements to TypeScript support including better type inference for computed properties, `ThisType` support for better context typing in component options, and enhanced type definitions for component options.

> **Note**: TypeScript improvements were introduced in Vue 2.6.0 and require `vue-template-compiler` version 2.6+ for full type support.

## Task Checklist

- [ ] Use `ThisType` for better component options type inference
- [ ] Leverage improved computed property type inference
- [ ] Use enhanced component type definitions
- [ ] Configure tsconfig.json for Vue 2.6 TypeScript support
- [ ] Use Vue.extend() for proper component typing

## ThisType Support (Vue 2.6.0+)

Vue 2.6.0 added `ThisType` support for better type inference in component options.

```vue
<script lang="ts">
import Vue from 'vue'

// Before Vue 2.6: Limited type inference
const Component1 = {
  data() {
    return {
      count: 0
    }
  },
  methods: {
    increment() {
      // this.count had any type
      this.count++ // Type: any
    }
  }
}

// Vue 2.6+: Better type inference with ThisType
const Component2 = {
  data() {
    return {
      count: 0,
      message: 'Hello'
    }
  },
  computed: {
    doubleCount(): number {
      return this.count * 2 // this is properly typed
    },
    greeting(): string {
      return this.message // Proper type inference
    }
  },
  methods: {
    increment(): void {
      this.count++ // Type: number
      this.message = 'Updated' // Type: string
    }
  }
}

export default Component2
</script>
```

## Enhanced Computed Property Type Inference

Vue 2.6 improved type inference for computed properties.

```vue
<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'TypedComponent',
  data() {
    return {
      firstName: 'John',
      lastName: 'Doe',
      items: [1, 2, 3]
    }
  },
  computed: {
    // Vue 2.6+: Better inference for getter
    fullName(): string {
      return `${this.firstName} ${this.lastName}`
    },
    // Vue 2.6+: Proper typing for getter/setter
    fullNameComputed: {
      get(): string {
        return `${this.firstName} ${this.lastName}`
      },
      set(value: string) {
        [this.firstName, this.lastName] = value.split(' ')
      }
    },
    // Array methods properly typed
    itemCount(): number {
      return this.items.length
    },
    firstItem(): number | undefined {
      return this.items[0]
    }
  }
})
</script>
```

## Improved Component Options Typing

```vue
<script lang="ts">
import Vue from 'vue'

// Vue 2.6: Enhanced component options typing
interface UserData {
  id: number
  name: string
  email: string
}

export default Vue.extend({
  name: 'TypedOptionsComponent',
  // Props with proper typing
  props: {
    user: {
      type: Object as () => UserData,
      required: true
    },
    status: {
      type: String,
      validator: (value: string): boolean => {
        return ['active', 'inactive'].includes(value)
      }
    }
  },
  // Data with typed return
  data(): {
    localData: UserData
    isLoading: boolean
  } {
    return {
      localData: this.user,
      isLoading: false
    }
  },
  // Computed with proper typing
  computed: {
    userDisplayName(): string {
      return this.localData.name
    },
    isActive(): boolean {
      return this.status === 'active'
    }
  },
  // Methods with proper parameter typing
  methods: {
    async fetchUserData(): Promise<void> {
      this.isLoading = true
      try {
        const response = await fetch(`/api/users/${this.user.id}`)
        const data: UserData = await response.json()
        this.localData = data
      } finally {
        this.isLoading = false
      }
    },
    updateUser(data: Partial<UserData>): void {
      this.localData = { ...this.localData, ...data }
    }
  },
  // Watch with proper typing
  watch: {
    'user.id'(newId: number, oldId: number): void {
      console.log(`User changed from ${oldId} to ${newId}`)
      this.fetchUserData()
    },
    status(newStatus: string): void {
      console.log(`Status changed to ${newStatus}`)
    }
  }
})
</script>
```

## tsconfig.json Configuration for Vue 2.6

```json
{
  "compilerOptions": {
    "target": "ES2015",
    "module": "ES2015",
    "strict": true,
    "jsx": "preserve",
    "moduleResolution": "node",
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "sourceMap": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "vue/*": ["vue/types/*"]
    },
    "lib": ["ES2015", "DOM"],
    "types": ["vite/client", "vue/types/vue"]
  },
  "include": [
    "src/**/*.ts",
    "src/**/*.tsx",
    "src/**/*.vue",
    "tests/**/*.ts"
  ],
  "exclude": [
    "node_modules"
  ]
}
```

## Vue Shims for Vue 2.6

```typescript
// src/shims-vue.d.ts

// Vue 2.6+ type definitions
declare module '*.vue' {
  import Vue from 'vue'
  export default Vue
}

// Augment Vue types with custom properties
declare module 'vue/types/vue' {
  interface Vue {
    $myGlobalProperty: string
  }
}

// Component options augmentation
import { VueConstructor } from 'vue/types/vue'

interface VueConstructor {
  $myGlobalMethod(): void
}
```

## Enhanced Class Component Typing

```vue
<script lang="ts">
import { Component, Vue, Prop, Watch } from 'vue-property-decorator'

interface User {
  id: number
  name: string
}

@Component
export default class TypedClassComponent extends Vue {
  // Props with typing
  @Prop({ type: Object, required: true })
  readonly user!: User

  @Prop({ type: Boolean, default: false })
  readonly isActive!: boolean

  // Data with typing
  localData: User = this.user
  loading: boolean = false

  // Computed with typing
  get displayName(): string {
    return this.user.name
  }

  // Methods with typing
  async loadData(): Promise<void> {
    this.loading = true
    try {
      const response = await fetch(`/api/users/${this.user.id}`)
      this.localData = await response.json()
    } finally {
      this.loading = false
    }
  }

  // Watch with typing
  @Watch('user.id')
  onUserIdChanged(newId: number, oldId: number): void {
    console.log(`User ID changed from ${oldId} to ${newId}`)
    this.loadData()
  }
}
</script>
```

## Functional Component Typing

```vue
<script lang="ts">
import { CreateElement, VNode } from 'vue/types/vue'
import { PropOptions } from 'vue/types/options'

// Functional component with proper typing
interface FunctionalComponentProps {
  title: string
  count: number
  items: string[]
}

export default {
  name: 'FunctionalTypedComponent',
  functional: true,
  props: {
    title: {
      type: String,
      required: true
    },
    count: {
      type: Number,
      default: 0
    },
    items: {
      type: Array as PropOptions<string[]>,
      default: () => []
    }
  },
  render(createElement: CreateElement, context: {
    props: FunctionalComponentProps
  }): VNode {
    const { title, count, items } = context.props

    return createElement('div', [
      createElement('h2', title),
      createElement('p', `Count: ${count}`),
      createElement('ul', items.map(item =>
        createElement('li', item)
      ))
    ])
  }
}
</script>
```

## Plugin Development Typing

```typescript
// src/plugins/my-plugin.ts
import { PluginFunction } from 'vue/types/plugin'
import { VueConstructor } from 'vue/types/vue'

interface MyPluginOptions {
  apiKey: string
  debug?: boolean
}

// Typed plugin function
const install: PluginFunction<MyPluginOptions> = function(
  Vue: VueConstructor,
  options?: MyPluginOptions
): void {
  // Typed Vue property
  Vue.prototype.$myPlugin = {
    apiKey: options?.apiKey || '',
    debug: options?.debug || false,
    fetchData(): Promise<any> {
      return fetch('/api/data').then(r => r.json())
    }
  }

  // Typed global method
  Vue.$myGlobalMethod = function(): void {
    console.log('Global method called')
  }

  // Typed component
  Vue.component('MyPluginComponent', {
    props: {
      data: Object as () => any
    },
    methods: {
      logData(): void {
        if (options?.debug) {
          console.log(this.data)
        }
      }
    }
  })
}

export default install

// Augment Vue types
declare module 'vue/types/vue' {
  interface Vue {
    $myPlugin: {
      apiKey: string
      debug: boolean
      fetchData(): Promise<any>
    }
  }
}

interface VueConstructor {
  $myGlobalMethod(): void
}
```

## Store Typing with Vuex

```typescript
// src/store/types.ts
export interface RootState {
  version: string
  isLoading: boolean
}

export interface UserState {
  currentUser: User | null
  users: User[]
}

export interface User {
  id: number
  name: string
  email: string
}

// Typing store getters
export interface Getters {
  userById: (state: UserState) => (id: number) => User | undefined
  activeUsers: (state: UserState) => User[]
}

// Typing store mutations
export interface Mutations {
  SET_CURRENT_USER(state: UserState, user: User): void
  SET_USERS(state: UserState, users: User[]): void
  ADD_USER(state: UserState, user: User): void
}

// Typing store actions
export interface Actions {
  fetchUsers(context: ActionContext<UserState, RootState>): Promise<void>
  addUser(context: ActionContext<UserState, RootState>, user: User): Promise<void>
}

import { ActionContext } from 'vuex'

// Usage in component
<script lang="ts">
import { mapState, mapGetters, mapActions } from 'vuex'
import { namespace } from 'vuex-class'
import Vue from 'vue'
import Component from 'vue-class-component'

@Component
export default class TypedVuexComponent extends Vue {
  // Typed computed from store
  @State('users')
  users!: User[]

  @State('currentUser')
  currentUser!: User | null

  // Typed getter
  @Getter('userById')
  getUserById!: (id: number) => User | undefined

  // Typed action
  @Action('fetchUsers')
  fetchUsers!: () => Promise<void>

  // Usage
  mounted() {
    this.fetchUsers()
    const user = this.getUserById(1)
    console.log(user?.name)
  }
}
</script>
```

## Best Practices

1. **Always use `Vue.extend()`** - Provides proper component typing
2. **Enable strict mode in tsconfig.json** - Catches more type errors
3. **Use interfaces for data shapes** - Better type safety
4. **Type computed properties explicitly** - Better inference
5. **Use decorators with vue-class-component** - Cleaner syntax
6. **Maintain Vue shims** - Keep type definitions up to date

## Reference

- [Vue 2 TypeScript Support](https://vue2.docs.vuejs.org/v2/guide/typescript.html)
- [Vue 2.6 Release Notes - TypeScript Improvements](https://github.com/vuejs/vue/releases/tag/v2.6.0)
- [vue-property-decorator Documentation](https://github.com/kaorun343/vue-property-decorator)
