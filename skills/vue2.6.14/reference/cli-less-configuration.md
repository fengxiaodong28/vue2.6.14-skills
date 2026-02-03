---
title: Configuring Less Preprocessor in Vue CLI for Vue 2
impact: MEDIUM
impactDescription: Less requires proper loader configuration in vue.config.js for Vue 2 projects
type: best-practice
tags:
  - vue2
  - vue-cli
  - less
  - css-preprocessor
  - vue-config
---

# Configuring Less Preprocessor in Vue CLI for Vue 2

Impact: MEDIUM - Less preprocessor requires proper configuration in `vue.config.js` to work correctly with Vue 2 projects using Vue CLI.

## Task Checklist

- [ ] Install `less` and `less-loader` as dev dependencies
- [ ] Configure `css.loaderOptions.less` in `vue.config.js`
- [ ] Enable `javascriptEnabled: true` for Less functions

## Installation

```bash
# Install Less and loader
npm install less less-loader --save-dev
# OR
yarn add less less-loader --dev
```

## Basic vue.config.js Configuration

```javascript
// vue.config.js
module.exports = {
  css: {
    loaderOptions: {
      less: {
        lessOptions: {
          // Enable inline JavaScript in Less (needed for some Less functions)
          javascriptEnabled: true,

          // Modify global variables
          modifyVars: {
            '@primary-color': '#42b983',
            '@border-radius': '4px',
            '@text-color': '#333333'
          }
        },

        // Alternative: Import global Less files
        additionalData: `
          @import "@/styles/variables.less";
          @import "@/styles/mixins.less";
        `
      }
    }
  }
}
```

## Global Variables File

```less
/* src/styles/variables.less */
@primary-color: #42b983;
@secondary-color: #35495e;
@border-color: #e0e0e0;
@border-radius: 4px;
@spacing-unit: 8px;
```

```less
/* src/styles/mixins.less */
.flex-center() {
  display: flex;
  justify-content: center;
  align-items: center;
}

.card-style() {
  border: 1px solid @border-color;
  border-radius: @border-radius;
  padding: @spacing-unit * 2;
}
```

## Component Usage

```vue
<template>
  <div class="user-card">
    <h2 class="user-card__title">{{ user.name }}</h2>
    <p class="user-card__email">{{ user.email }}</p>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  props: {
    user: {
      type: Object,
      required: true
    }
  }
})
</script>

<style lang="less" scoped>
// Using global variables
@border-color: @border-color;

.user-card {
  // Using global variables
  border: 1px solid @border-color;
  border-radius: @border-radius;
  padding: @spacing-unit * 2;

  // Nesting
  &__title {
    color: @primary-color;
    font-size: 18px;
  }

  &__email {
    color: @secondary-color;

    // Pseudo-class
    &:hover {
      text-decoration: underline;
    }
  }
}

// Using mixins
.call-to-action {
  .flex-center();
  .card-style();

  background: @primary-color;

  &:hover {
    opacity: 0.9;
  }
}
</style>
```

## Deep Selectors (Scoped Styles)

```vue
<style lang="less" scoped>
.parent {
  // Vue 2 deep selector syntaxes
  /deep/ .child {
    color: @primary-color;
  }

  >>> .sibling {
    color: @secondary-color;
  }

  // Vue 2.7+ syntax
  ::v-deep .another-child {
    color: @primary-color;
  }
}
</style>
```

## Common Less Functions

```less
// Color functions
.button {
  background: lighten(@primary-color, 10%);
  border: 1px solid darken(@primary-color, 5%);
  color: contrast(@primary-color);
}

// Math operations
.container {
  width: 100% - @spacing-unit * 2;
  margin: @spacing-unit * 2;
}

// Mixins with parameters
.button-size(@padding) {
  padding: @padding;
  border-radius: @padding * 2;
}

.primary-button {
  .button-size(12px);
  background: @primary-color;
}
```

## Reference

- [Vue CLI - CSS Loading](https://cli.vuejs.org/guide/css.html#passing-options-to-pre-processor-loaders)
- [Less Documentation](https://lesscss.org/)
