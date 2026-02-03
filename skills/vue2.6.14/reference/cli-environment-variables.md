---
title: Environment Variables in Vue CLI Projects
impact: LOW
impactDescription: Environment variables provide build-time configuration accessible in client code
type: best-practice
tags:
  - vue2
  - vue-cli
  - environment-variables
  - build-configuration
---

# Environment Variables in Vue CLI Projects

Impact: LOW - Environment variables in Vue CLI provide build-time configuration that can be accessed in client code. All env variables must start with `VUE_APP_` to be available in client code.

## Task Checklist

- [ ] Prefix all client-side environment variables with `VUE_APP_`
- [] Use `.env` files for environment-specific configuration
- [] Access env vars in code via `process.env.VUE_APP_*`
- [ ] Never include secrets or keys in environment files
- [** Rebuild required after changing .env files **

## .env Files

```bash
# .env (default, committed to git)
VUE_APP_TITLE=My App
VUE_APP_API_URL=https://api.example.com
VUE_APP_PUBLIC_PATH=/my-app/

# .env.local (local overrides, gitignored)
VUE_APP_API_URL=http://localhost:3000

# .env.production (production only)
VUE_APP_API_URL=https://api.production.com
VUE_APP_PUBLIC_PATH=/production/

# .env.development (development only)
VUE_APP_API_URL=http://localhost:3000
VUE_APP_DEBUG=true
```

## Accessing in Code

```javascript
// In any component or JavaScript file
export default {
  data() {
    return {
      apiUrl: process.env.VUE_APP_API_URL,
      title: process.env.VUE_APP_TITLE,
      debug: process.env.VUE_APP_DEBUG === 'true'
    }
  },
  created() {
    console.log('API URL:', this.apiUrl)
    console.log('App Title:', this.title)
  }
}
```

## In vue.config.js

```javascript
// vue.config.js
module.exports = {
  // Environment variables are available here
  publicPath: process.env.VUE_APP_PUBLIC_PATH || '/',
  outputDir: process.env.VUE_APP_OUTPUT_DIR || 'dist',

  // Use in build configuration
  configureWebpack: config => {
    if (process.env.VUE_APP_DEBUG === 'true') {
      // Development-specific config
      config.devtool = 'cheap-module-eval-source-map'
    }
  }
}
```

## Only VUE_APP_* Variables Available in Client Code

**Important**: Only environment variables that start with `VUE_APP_` are embedded in the client bundle:

```javascript
// ✅ Available in client code
console.log(process.env.VUE_APP_API_URL)
console.log(process.env.VUE_APP_TITLE)

// ❌ NOT available in client code (undefined)
console.log(process.env.NODE_ENV) // Actually replaced by build tool
console.log(process.env.BASE_URL) // Available but only if injected by build tool
```

## NODE_ENV and BASE_URL

These are special variables injected by Vue CLI:

| Variable | Description |
|----------|-------------|
| `NODE_ENV` | Set to 'development', 'production', or 'test' |
| `BASE_URL` | Matches publicPath option in vue.config.js |

```javascript
// In client code
console.log(process.env.NODE_ENV) // 'development' or 'production'
console.log(process.env.BASE_URL) // Current base URL path
```

## Build-Time vs Runtime Variables

Environment variables are resolved at **build time**:

```javascript
// This value is baked into the bundle at build time
const apiUrl = process.env.VUE_APP_API_URL

// If VUE_APP_API_URL = 'https://api.example.com'
// After build, this becomes:
const apiUrl = 'https://api.example.com'
```

To change values at **runtime**, use window object:

```javascript
// main.ts
declare global {
  interface Window {
    runtimeConfig?: {
      apiUrl: string
    }
  }
}

// Fetch runtime config
fetch('/config.json').then(r => r.json()).then(config => {
  window.runtimeConfig = config
})

// Use in components
this.$http.get(window.runtimeConfig.apiUrl)
```

## Environment-Specific Configuration

```javascript
// vue.config.js
module.exports = {
  // Different config based on environment
  configureWebpack: config => {
    if (process.env.NODE_ENV === 'production') {
      // Production webpack config
      config.optimization.splitChunks({
        chunks: 'all'
      })
    }
  },

  // Chain webpack based on custom env var
  chainWebpack: config => {
    if (process.env.VUE_APP_ANALYTICS === 'true') {
      // Add analytics plugin
    }
  }
}
```

## Type Definitions for Environment Variables

```typescript
// src/env.d.ts (or similar)
/// <reference types="vite/client" />

declare const NODE_ENV: 'development' | 'production' | 'test'
declare const VUE_APP_API_URL: string
declare const VUE_APP_TITLE: string
declare const VUE_APP_PUBLIC_PATH: string

interface ImportMetaEnv {
  baseUrl: string
  VUE_APP_API_URL: string
  VUE_APP_TITLE: string
  VUE_APP_PUBLIC_PATH: string
  // ...other env vars
}

declare const import.meta: {
  env: ImportMetaEnv
}
```

## Using in Templates

```vue
<template>
  <div>
    <h1>{{ title }}</h1>
    <p>API: {{ apiUrl }}</p>
    <div v-if="isDebug">Debug mode enabled</div>
  </div>
</template>

<script>
export default {
  computed: {
    title() {
      return process.env.VUE_APP_TITLE
    },
    apiUrl() {
      return process.env.VUE_APP_API_URL
    },
    isDebug() {
      return process.env.VUE_APP_DEBUG === 'true'
    }
  }
}
</script>
```

## Creating .env Files

```bash
# .env (default)
VUE_APP_TITLE=My Vue 2 App
VUE_APP_API_URL=https://api.example.com
VUE_APP_VERSION=1.0.0

# .env.development
VUE_APP_API_URL=http://localhost:3000
VUE_APP_DEBUG=true

# .env.production
VUE_APP_API_URL=https://api.production.com
VUE_APP_DEBUG=false
```

## Security Considerations

```bash
# ❌ NEVER commit secrets to .env files
DATABASE_PASSWORD=secret123
API_KEY=sk_live_xxx

# ✅ Use placeholder values instead
VUE_APP_DATABASE_URL=postgresql://localhost:5432/mydb
VUE_APP_API_KEY=your-api-key-here
```

## Reference

- [Vue CLI - Environment Variables and Modes](https://cli.vuejs.org/guide/mode-and-env.html)
