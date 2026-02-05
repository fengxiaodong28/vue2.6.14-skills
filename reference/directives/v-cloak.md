---
title: v-cloak Directive for Hiding Uncompiled Templates
impact: LOW
impactDescription: v-cloak hides uncompiled mustache bindings until Vue instance finishes compilation
type: capability
tags:
  - - vue2.6.14
  - v-cloak
  - directives
  - template-compilation
  - flash-prevention
---

# v-cloak Directive for Hiding Uncompiled Templates

Impact: LOW - The `v-cloak` directive is used to hide uncompiled mustache bindings until the Vue instance finishes compilation. Combined with CSS rules like `[v-cloak] { display: none }`, it prevents the "flash of uncompiled templates" that users see before Vue takes over.

## Task Checklist

- [ ] Add `[v-cloak] { display: none }` CSS rule
- [ ] Apply `v-cloak` to the root element of your app
- [ ] Remember v-cloak is removed after compilation
- [ ] Use `v-cloak` on the root mount element, not child elements
- [ ] Can be used on multiple root elements in multi-root setups

## Basic Usage

```vue
<template>
  <!-- Apply v-cloak to the root element -->
  <div v-cloak id="app">
    <p>{{ message }}</p>
    <button @click="handleClick">Click Me</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: 'Hello Vue!'
    }
  },
  methods: {
    handleClick() {
      console.log('Button clicked!')
    }
  }
}
</script>

<style>
/* Required CSS */
[v-cloak] {
  display: none;
}
</style>
```

## How v-cloak Works

1. **Before Vue compiles**: Element with `v-cloak` remains visible, hiding content
2. **During compilation**: Vue processes the template
3. **After compilation**: `v-cloak` attribute is removed from the element

```vue
<!-- Before Vue compiles (visible in DOM): -->
<div v-cloak id="app">
  <p>{{ message }}</p>
</div>
<!-- User sees blank (due to CSS display: none) -->

<!-- After Vue compiles (v-cloak removed): -->
<div id="app">
  <p>Hello Vue!</p>
</div>
<!-- User sees compiled content -->
```

## Preventing Template Flash

**Without v-cloak (problematic):**

```vue
<!-- User might briefly see: -->
<!-- {{ message }} -->
<template>
  <div id="app">
    <p>{{ message }}</p>
  </div>
</template>
```

**With v-cloak (smooth):**

```vue
<!-- User sees nothing until Vue is ready -->
<template>
  <div v-cloak id="app">
    <p>{{ message }}</p>
  </div>
</template>

<style>
[v-cloak] {
  display: none;
}
</style>
```

## Complete Setup Example

```vue
<!-- App.vue -->
<template>
  <div v-cloak id="app">
    <app-header />
    <router-view />
    <app-footer />
  </div>
</template>

<script>
import AppHeader from './AppHeader.vue'
import AppFooter from './AppFooter.vue'

export default {
  name: 'App',
  components: {
    AppHeader,
    AppFooter
  }
}
</script>

<style>
[v-cloak] {
  display: none;
}
</style>
```

## Multiple Root Elements

```vue
<template>
  <!-- v-cloak can be on multiple root elements -->
  <div v-cloak class="sidebar">
    <p>{{ sidebarContent }}</p>
  </div>

  <div v-cloak class="main">
    <p>{{ mainContent }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      sidebarContent: 'Sidebar',
      mainContent: 'Main content'
    }
  }
}
</script>

<style>
[v-cloak] {
  display: none;
}
</style>
```

## v-cloak with Different CSS Approaches

### Inline Styles

```vue
<template>
  <div v-cloak style="display: none;">
    <p>{{ message }}</p>
  </div>
</template>
```

### Scoped CSS

```vue
<template>
  <div v-cloak id="app">
    <p>{{ message }}</p>
  </div>
</template>

<style scoped>
[v-cloak] {
  display: none;
}
</style>
```

### Global CSS

```css
/* styles/global.css */
[v-cloak] {
  display: none;
}
```

## CSS-in-JS Solutions

```javascript
// main.js
import Vue from 'Vue'
import App from './App.vue'

// Add v-cloak styles programmatically
const vCloakStyle = document.createElement('style')
vCloakStyle.textContent = `
  [v-cloak] {
    display: none !important;
  }
`
document.head.appendChild(vCloakStyle)

new Vue({
  render: h => h(App)
}).$mount('#app')
```

## v-cloak with Preloaders

**When using webpack with vue-loader:** v-cloak is often handled automatically. However, for custom setups:

```javascript
// vue.config.js
module.exports = {
  chainWebpack: config => {
    config.module
      .rule('vue')
      .use('vue-loader')
      .loader('vue-loader/lib/plugin')
      .tap(options => {
        // Add v-cloak handling
        options.preloaders = {
          vCloak: 'v-cloak'
        }
      })
  }
}
```

## Using v-cloak with Different App Structures

### Single Root Element

```vue
<!-- Most common pattern -->
<template>
  <div v-cloak id="app">
    <router-view />
  </div>
</template>

<style>
[v-cloak] {
  display: none;
}
</style>
```

### Template Root Element

```html
<!-- index.html -->
<div id="app" v-cloak>
  <p>Loading...</p>
</div>
```

```vue
<!-- App.vue -->
<template>
  <div id="app">
    <!-- v-cloak will be removed here -->
    <router-view />
  </div>
</template>

<style>
[v-cloak] {
  display: none;
}
</style>
```

## v-cloak with Loading States

```vue
<template>
  <div v-cloak id="app">
    <!-- Initial content shown during loading -->
    <div class="loading-screen">
      <p>Loading application...</p>
    </div>
  </div>
</template>

<style>
[v-cloak] {
  display: none;
}
.loading-screen {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f0f0f0;
  z-index: 9999;
}
</style>
```

## Debugging v-cloak

```vue
<template>
  <div>
    <!-- Check if v-cloak is applied -->
    <p v-cloak>Vue is loading...</p>
    <p v-else>Vue is ready!</p>
  </div>
</template>
```

## Common Issues

### Issue 1: v-cloak Not Working

**Problem:** Content still flashes before compilation

**Solutions:**

1. **Check CSS selector specificity:**

```css
/* Make sure v-cloak selector is specific enough */
[v-cloak] {
  display: none !important;
}
```

2. **Ensure v-cloak is on the mount point:**

```javascript
// main.js
new Vue({
  el: '#app', // Make sure this matches the v-cloak element
  render: h => h(App)
})
```

3. **Check for CSS conflicts:**

```vue
<style>
/* Avoid more specific selectors that override v-cloak */
#app[v-cloak] {
  display: none !important;
}
</style>
```

### Issue 2: v-cloak Staying After Compile

**Problem:** Element stays hidden after Vue compiles

**Possible causes:**
- CSS rule has `!important` and Vue can't override
- Multiple `v-cloak` attributes conflicting
- Selector specificity issues

**Solution:**

```css
/* Use proper v-cloak selector */
[v-cloak] {
  display: none;
}

/* NOT this (too specific) */
div[v-cloak] {
  display: none !important;
}
```

## v-cloak with SSR (Server-Side Rendering)

```vue
<!-- In SSR, v-cloak is handled differently -->
<template>
  <div v-cloak>
    <!-- Content -->
  </div>
</template>

<script>
export default {
  // In SSR, v-cloak should be managed server-side
  serverPrefetch() {
    return Promise.all([
      this.fetchData()
    ])
  },
  methods: {
    fetchData() {
      // Fetch initial data
    }
  }
}
</script>
```

## Best Practices

1. **Always add the CSS rule**: `[v-cloak] { display: none; }`
2. **Apply to the root element**: Usually the mount point element
3. **Use with v-show for loading states**: Create smooth loading experience
4. **Test without JavaScript disabled**: Ensure graceful degradation
5. **Keep it simple**: Don't overcomplicate the selector

## Complete Component Example

```vue
<!-- App.vue -->
<template>
  <div v-cloak id="app">
    <!-- Loading state -->
    <div v-if="loading" class="app-loading">
      <div class="spinner"></div>
      <p>Loading...</p>
    </div>

    <!-- App content -->
    <div v-else>
      <app-nav />
      <router-view />
      <app-footer />
    </div>
  </div>
</template>

<script>
import AppNav from './components/AppNav.vue'
import AppFooter from './components/AppFooter.vue'

export default {
  name: 'App',
  components: {
    AppNav,
    AppFooter
  },
  data() {
    return {
      loading: true
    }
  },
  mounted() {
    // Simulate initial data fetch
    setTimeout(() => {
      this.loading = false
    }, 1000)
  }
}
</script>

<style>
[v-cloak] {
  display: none;
}

#app {
  min-height: 100vh;
}

.app-loading {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: #f5f5f5;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid #f3f3f3;
  border-top: 4px solid #42b983;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>
```

## v-cloak vs CSS display: none

```vue
<template>
  <div>
    <!-- v-cloak: Hides only until compilation -->
    <div v-cloak class="compiling">
      {{ message }}
    </div>

    <!-- CSS display: none: Always hidden -->
    <div class="hidden">
      {{ message }}
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: 'Hello'
    }
  }
}
</script>

<style>
[v-cloak] {
  display: none;
}

.hidden {
  display: none;
}
</style>
```

## Reference

- [Vue 2 API - v-cloak](https://vue2.docs.vuejs.org/v2/api/#v-cloak)
- [Vue 2 Guide - Lifecycle Diagram - Mount](https://vue2.docs.vuejs.org/v2/guide/instance.html#Lifecycle-Diagram)
