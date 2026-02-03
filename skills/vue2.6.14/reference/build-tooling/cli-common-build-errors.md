---
title: Common Build Errors and Solutions in Vue CLI
impact: MEDIUM
impactDescription: Understanding and resolving common Vue CLI build errors
type: troubleshooting
tags:
  - vue2.6.14
  - vue-cli
  - build-errors
  - troubleshooting
  - webpack
---

# Common Build Errors and Solutions in Vue CLI

Impact: MEDIUM - Common build errors can be resolved by understanding their causes and applying the correct fixes.

## Task Checklist

- [ ] Check vue-template-compiler version matches vue version
- [ ] Verify webpack configuration for asset imports
- [ ] Check for circular dependencies
- [ ] Ensure Node.js version compatibility
- [ ] Verify environment variables are properly prefixed

## Version Mismatch Error

```
Error:
[Vue warn]: You are using the runtime-only build of Vue where the template compiler is not available.
```

**Cause**: Using runtime-only build with template compiler.

**Solution**: Ensure Vue alias points to full build:

```javascript
// vue.config.js
module.exports = {
  runtimeCompiler: true
}

// Or in webpack config
resolve: {
  alias: {
    vue$: 'vue/dist/vue.esm.js'
  }
}
```

## Template Compiler Version Mismatch

```
Error:
[Vue warn]: It seems you are using the standalone build of Vue
or your template compiler version doesn't match your runtime version.
```

**Cause**: `vue` and `vue-template-compiler` versions don't match.

**Solution**: Match versions exactly:

```bash
npm uninstall vue-template-compiler
npm install vue-template-compiler@2.6.14
```

```json
// package.json
{
  "dependencies": {
    "vue": "^2.6.14"
  },
  "devDependencies": {
    "vue-template-compiler": "^2.6.14"
  }
}
```

## Module Not Found

```
Error:
Module not found: Error: Can't resolve '@/components/MyComponent.vue'
```

**Cause**: Path alias not configured correctly.

**Solution**:

```javascript
// vue.config.js
const path = require('path')

module.exports = {
  configureWebpack: {
    resolve: {
      alias: {
        '@': path.resolve(__dirname, 'src')
      }
    }
  }
}
```

```json
// tsconfig.json or jsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

## Failed to Compile with Loader Errors

```
Error:
Module build failed: TypeError: this.getOptions is not a function
```

**Cause**: Incompatible loader version or webpack version mismatch.

**Solution**: Reinstall dependencies:

```bash
rm -rf node_modules package-lock.json
npm install
```

Or update specific loader:

```bash
npm install --save-dev sass-loader@^10
```

## Asset Import Errors

```
Error:
Module not found: Error: Can't resolve './logo.png'
```

**Cause**: Incorrect asset import path or wrong loader configuration.

**Solution**: Use correct import syntax:

```vue
<script>
// ✅ CORRECT
import logo from '@/assets/logo.png'

export default {
  data() {
    return {
      logoUrl: logo
    }
  }
}
</script>

<template>
  <img :src="logoUrl" alt="Logo" />
</template>
```

## Circular Dependencies

```
Error:
Circular dependency detected:
src/components/A.vue -> src/components/B.vue -> src/components/A.vue
```

**Cause**: Components import each other.

**Solution**: Refactor to remove circular dependency:

```javascript
// ❌ WRONG: A.vue imports B.vue, B.vue imports A.vue

// ✅ CORRECT: Create shared component or use events/bus

// Option 1: Create shared component C
// Option 2: Use event bus for communication
// Option 3: Lift state to parent component
```

## Environment Variable Not Available

```
Error:
process.env.VUE_APP_API_URL is undefined
```

**Cause**: Environment variable doesn't start with `VUE_APP_`.

**Solution**: Ensure correct prefix:

```bash
# .env
VUE_APP_API_URL=https://api.example.com  # ✅ Correct
API_URL=https://api.example.com          # ❌ Wrong - not available in client code
```

```javascript
// Access in code
const apiUrl = process.env.VUE_APP_API_URL
```

## Cannot Find Module 'vue'

```
Error:
Cannot find module 'vue' or its corresponding type declarations
```

**Cause**: TypeScript configuration issue or missing type definitions.

**Solution**:

```bash
# Install types
npm install --save-dev @types/node

# Or for Vue 2 with TypeScript
npm install --save-dev vue-class-component vue-property-decorator
```

```typescript
// Create shims-vue.d.ts
declare module '*.vue' {
  import Vue from 'vue'
  export default Vue
}
```

## Prod Build Memory Limit

```
Error:
FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed
```

**Cause**: Node.js memory limit for production builds.

**Solution**: Increase Node.js memory limit:

```bash
# On Unix/Linux
NODE_OPTIONS="--max-old-space-size=4096" npm run build

# On Windows
set NODE_OPTIONS=--max-old-space-size=4096
npm run build

# Or in package.json
{
  "scripts": {
    "build": "node --max-old-space-size=4096 node_modules/@vue/cli-service/bin/vue-cli-service.js build"
  }
}
```

## PostCSS Loader Error

```
Error:
Module build failed: ModuleBuildError: Module build failed: Error:
No PostCSS Config found
```

**Cause**: Missing PostCSS configuration file.

**Solution**: Create PostCSS config:

```javascript
// postcss.config.js
module.exports = {
  plugins: {
    autoprefixer: {},
    ...(process.env.NODE_ENV === 'production'
      ? {
          cssnano: {
            preset: 'default'
          }
        }
      : {})
  }
}
```

## Less/Sass Loader Errors

```
Error:
Module build failed: TypeError: this.getOptions is not a function
```

**Cause**: Loader version incompatibility with Webpack.

**Solution**: Use compatible loader versions:

```bash
# For Vue CLI 4/5
npm install --save-dev less less-loader@^7
npm install --save-dev sass sass-loader@^10
```

## ESlint Errors During Build

```
Error:
error: 'foo' is assigned a value but never used
```

**Cause**: ESLint preventing build.

**Solution**: Fix lint errors or disable lint-on-save:

```javascript
// vue.config.js
module.exports = {
  lintOnSave: false // or 'default' or 'warning'
}
```

## Public Path Issues

```
Error:
404 Not Found: /js/app.js
```

**Cause**: Incorrect publicPath configuration.

**Solution**: Set correct public path:

```javascript
// vue.config.js
module.exports = {
  publicPath: process.env.NODE_ENV === 'production'
    ? '/my-sub-directory/'
    : '/'
}
```

## Hot Module Replacement Not Working

**Symptoms**: Changes not reflecting without full page reload.

**Solutions**:

```javascript
// vue.config.js
module.exports = {
  devServer: {
    hot: true,
    liveReload: true
  }
}

// Or disable HMR for specific issues
module.exports = {
  devServer: {
    hot: false
  }
}
```

## WebSocket Connection Failed

```
Error:
[WDS] Disconnected!
get http://localhost:8080/sockjs-node/info?t=... net::ERR_CONNECTION_REFUSED
```

**Cause**: Port conflict or firewall blocking WebSocket.

**Solution**:

```javascript
// vue.config.js
module.exports = {
  devServer: {
    port: 8080,
    host: 'localhost',
    https: false,
    disableHostCheck: true
  }
}
```

## Source Map Error

```
Error:
Invalid source map: Failed to parse source map
```

**Solution**: Adjust source map settings:

```javascript
// vue.config.js
module.exports = {
  productionSourceMap: false, // Disable for production
  configureWebpack: {
    devtool: process.env.NODE_ENV === 'production'
      ? 'source-map'
      : 'cheap-module-eval-source-map'
  }
}
```

## Duplicate Build Artifacts

```
Warning:
Conflict: Multiple assets emit different content to the same filename
```

**Solution**: Configure filenames to include hash:

```javascript
// vue.config.js
module.exports = {
  configureWebpack: {
    output: {
      filename: 'js/[name].[contenthash:8].js',
      chunkFilename: 'js/[name].[contenthash:8].js'
    }
  },
  css: {
    extract: {
      filename: 'css/[name].[contenthash:8].css',
      chunkFilename: 'css/[name].[contenthash:8].css'
    }
  }
}
```

## Webpack Bundle Analyzer

```bash
# Install
npm install --save-dev @vue/cli-plugin-bundle-analyzer

# Run
npx vue-cli-service build --report
```

```javascript
// vue.config.js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin

module.exports = {
  configureWebpack: {
    plugins: [
      new BundleAnalyzerPlugin({
        analyzerMode: 'static',
        openAnalyzer: false
      })
    ]
  }
}
```

## Reference

- [Vue CLI Configuration Docs](https://cli.vuejs.org/config/)
- [Webpack Configuration Guide](https://webpack.js.org/configuration/)
