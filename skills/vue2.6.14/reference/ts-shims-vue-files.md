---
title: TypeScript Shims for .vue Files in Vue 2
impact: MEDIUM
impactDescription: Type declaration files enable TypeScript to recognize .vue files
type: best-practice
tags:
  - vue2
  - typescript
  - shims
  - type-declarations
  - vue-files
---

# TypeScript Shims for .vue Files in Vue 2

Impact: MEDIUM - TypeScript declaration files (shims) are required for TypeScript to properly recognize and type-check `.vue` Single File Components.

## Task Checklist

- [ ] Create `shims-vue.d.ts` or `vue-shim.d.ts` file
- [ ] Include declaration file in `tsconfig.json`
- [ ] Extend `Vue` interface for global properties
- [ ] Define module declarations for `.vue` files

## Basic Vue Shim File

Create `src/shims-vue.d.ts`:

```typescript
// shims-vue.d.ts
declare module '*.vue' {
  import Vue from 'vue'
  export default Vue
}
```

This file tells TypeScript that any `.vue` file should be treated as a Vue component.

## Complete Shim File with Global Properties

```typescript
// shims-vue.d.ts
declare module '*.vue' {
  import Vue from 'vue'
  export default Vue
}

// Extend Vue interface for global properties
declare module 'vue/types/vue' {
  interface Vue {
    $http: AxiosInstance
    $store: Vuex.Store<any>
    $router: VueRouter
    $route: Route
  }
}

// Extend VueConstructor interface for global methods
declare module 'vue/types/vue' {
  interface VueConstructor {
    $globalMethod(): void
  }
}
```

## Extended Shim File with Custom Types

```typescript
// shims-vue.d.ts
import Vue from 'vue'
import VueRouter, { Route } from 'vue-router'
import { Store } from 'vuex'
import Axios from 'axios'

// Declare .vue file module
declare module '*.vue' {
  export default Vue
}

// Declare global properties
declare module 'vue/types/vue' {
  interface Vue {
    // Axios instance
    $api: Axios.AxiosInstance

    // Router
    $router: VueRouter
    $route: Route

    // Store
    $store: Store<any>

    // Custom properties
    $auth: {
      token: string
      user: User | null
      login(credentials: Credentials): Promise<void>
      logout(): void
    }

    // Theme
    $theme: {
      current: 'light' | 'dark'
      setTheme(theme: string): void
    }
  }
}

// Type definitions for custom types
interface User {
  id: number
  name: string
  email: string
}

interface Credentials {
  username: string
  password: string
}
```

## Shim File for Asset Files

```typescript
// shims-assets.d.ts
declare module '*.svg' {
  const content: string
  export default content
}

declare module '*.png' {
  const content: string
  export default content
}

declare module '*.jpg' {
  const content: string
  export default content
}

declare module '*.jpeg' {
  const content: string
  export default content
}

declare module '*.gif' {
  const content: string
  export default content
}

declare module '*.webp' {
  const content: string
  export default content
}

declare module '*.ico' {
  const content: string
  export default content
}

declare module '*.bmp' {
  const content: string
  export default content
}
```

## Shim File for SCSS/Less Modules

```typescript
// shims-css-modules.d.ts
declare module '*.scss' {
  const classes: { [key: string]: string }
  export default classes
}

declare module '*.sass' {
  const classes: { [key: string]: string }
  export default classes
}

declare module '*.less' {
  const classes: { [key: string]: string }
  export default classes
}

declare module '*.styl' {
  const classes: { [key: string]: string }
  export default classes
}

declare module '*.css' {
  const classes: { [key: string]: string }
  export default classes
}
```

## Complete TypeScript Configuration

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["es2015", "dom"],
    "module": "esnext",
    "moduleResolution": "node",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "allowJs": true,
    "jsx": "preserve",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },
    "typeRoots": [
      "node_modules/@types",
      "src/types"
    ]
  },
  "include": [
    "src/**/*.ts",
    "src/**/*.tsx",
    "src/**/*.vue",
    "src/types/**/*.d.ts"
  ],
  "exclude": [
    "node_modules"
  ]
}
```

## Using Shims in Components

```vue
<!-- TypeScript now understands .vue imports -->
<script lang="ts">
import Vue from 'vue'
import { Component } from 'vue-property-decorator'
import MyComponent from '@/components/MyComponent.vue' // ✅ Now recognized!

@Component({
  components: {
    MyComponent
  }
})
export default class App extends Vue {
  // Global properties are now typed
  created() {
    this.$api.get('/users') // ✅ Autocomplete available!
    this.$router.push('/home') // ✅ Type-safe!
    this.$store.dispatch('fetchData') // ✅ Works!
  }
}
</script>
```

## Vue Prototype Extensions

```typescript
// main.ts - also needs types
import Vue from 'vue'
import Axios from 'axios'
import router from './router'
import store from './store'

const api = Axios.create({
  baseURL: process.env.VUE_APP_API_URL
})

// Extend Vue prototype
Vue.prototype.$api = api
Vue.prototype.$http = api

// In your shims file
declare module 'vue/types/vue' {
  interface Vue {
    $api: Axios.AxiosInstance
    $http: Axios.AxiosInstance
  }
}
```

## Vue Plugin Typings

```typescript
// shims-plugins.d.ts
import { PluginObject } from 'vue'

declare module 'vue/types/vue' {
  interface Vue {
    $myPlugin: MyPlugin
  }
}

declare module 'vue/types/vue' {
  interface VueConstructor {
    $myPluginGlobal: string
  }
}

interface MyPlugin {
  doSomething(): void
  getValue(): string
}

const MyPluginPlugin: PluginObject<any> = {
  install(Vue) {
    Vue.prototype.$myPlugin = {
      doSomething() {},
      getValue() { return 'value' }
    }
  }
}

export default MyPluginPlugin
```

## JSX Declaration

```typescript
// shims-jsx.d.ts
import { VNode } from 'vue'

declare global {
  namespace JSX {
    interface Element extends VNode {}
    interface ElementClass extends Vue {}
    interface ElementAttributesProperty {
      $props: {}
    }
    interface IntrinsicElements {
      [elem: string]: any
    }
  }
}
```

## Environment Variable Typings

```typescript
// shims-env.d.ts
declare namespace NodeJS {
  interface ProcessEnv {
    NODE_ENV: 'development' | 'production' | 'test'
    VUE_APP_API_URL: string
    VUE_APP_TITLE: string
    VUE_APP_VERSION: string
  }
}
```

## Component Props Typing with Shims

```vue
<script lang="ts">
import { Vue, Component, Prop } from 'vue-property-decorator'

interface User {
  id: number
  name: string
}

@Component
export default class UserCard extends Vue {
  @Prop({ type: Object, required: true })
  readonly user!: User

  @Prop({ type: String, default: 'medium' })
  readonly size!: 'small' | 'medium' | 'large'

  @Prop({ type: Boolean, default: false })
  readonly showEmail!: boolean
}
</script>
```

## Vuex Store Typings

```typescript
// shims-vuex.d.ts
import { Store } from 'vuex'

declare module 'vue/types/vue' {
  interface Vue {
    $store: Store<MyStoreState>
  }
}

interface MyStoreState {
  users: User[]
  loading: boolean
  error: string | null
}

interface User {
  id: number
  name: string
  email: string
}
```

## Directory Structure

```
src/
├── types/
│   ├── shims-vue.d.ts
│   ├── shims-assets.d.ts
│   ├── shims-css-modules.d.ts
│   ├── shims-env.d.ts
│   └── index.d.ts
├── main.ts
└── shims-tsx.d.ts
```

## Common Errors and Solutions

```typescript
// ❌ ERROR: Cannot find module '@/components/MyComponent.vue'
// Solution: Check tsconfig.json paths and ensure shims-vue.d.ts exists

// ❌ ERROR: Property '$api' does not exist on type 'Vue'
// Solution: Extend Vue interface in shims-vue.d.ts

// ❌ ERROR: Could not find a declaration file for module '*.svg'
// Solution: Create shims-assets.d.ts

// ❌ ERROR: TS2307: Cannot find module 'vue'
// Solution: npm install --save-dev @types/vue
```

## Reference

- [Vue 2 TypeScript Guide](https://v2.vuejs.org/v2/guide/typescript.html)
- [TypeScript Declaration Files](https://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html)
