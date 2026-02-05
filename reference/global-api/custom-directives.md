---
title: Creating Custom Directives in Vue 2
impact: MEDIUM
impactDescription: Custom directives allow encapsulating DOM manipulation logic for reuse across components
type: capability
tags:
  - vue2.6.14
  - custom-directives
  - dom-manipulation
  - directives
  - Vue.directive
---

# Creating Custom Directives in Vue 2

Impact: MEDIUM - Custom directives in Vue 2 allow you to encapsulate direct DOM manipulation logic for reuse. They're useful for integrating with third-party libraries, implementing focus management, and creating reusable behaviors.

## Task Checklist

- [ ] Register directives globally with `Vue.directive()` or locally in component
- [ ] Use appropriate hook functions: bind, inserted, update, componentUpdated, unbind
- [ ] Pass directive values, modifiers, and arguments for flexible behavior
- [ ] Clean up side effects in the unbind hook

## Directive Lifecycle Hooks

```javascript
// Hook functions execute in order:
Vue.directive('my-directive', {
  bind: function (el, binding, vnode) {
    // Called once when directive is first bound to element
    // One-time setup, can't access parent element yet
  },

  inserted: function (el, binding, vnode) {
    // Called when bound element is inserted into DOM
    // Safe to access parent node here
  },

  update: function (el, binding, vnode, oldVnode) {
    // Called after component's VNode has updated
    // May be called before children have updated
  },

  componentUpdated: function (el, binding, vnode, oldVnode) {
    // Called after component's VNode AND children VNodes have updated
  },

  unbind: function (el, binding, vnode) {
    // Called when directive is unbound from element
    // Clean up side effects
  }
})
```

## Binding Object Properties

```javascript
{
  name: 'focus',           // Directive name (without v- prefix)
  rawName: 'v-focus',      // Full directive name
  value: true,             // Value passed to directive (v-focus="true")
  expression: 'isFocused',  // Expression as string (v-focus="isFocused")
  arg: 'text',             // Argument passed (v-focus:text)
  modifiers: {             // Modifiers used (v-focus.lazy.trim)
    lazy: true,
    trim: true
  },
  oldValue: false          // Previous value (only in update/componentUpdated)
}
```

## Example 1: Auto-focus Directive

```javascript
// Register globally
import Vue from 'vue'

Vue.directive('focus', {
  // When element is inserted into DOM
  inserted: function (el) {
    el.focus()
  }
})

// Or register locally
export default {
  directives: {
    focus: {
      inserted: function (el) {
        el.focus()
      }
    }
  }
}
```

**Usage:**

```vue
<template>
  <input v-focus placeholder="Auto-focused input">
</template>
```

## Example 2: Click-Outside Directive

```javascript
// directives/click-outside.js
export default {
  bind: function (el, binding, vnode) {
    el.clickOutsideEvent = function (event) {
      // Check if click was outside the element and its children
      if (!(el == event.target || el.contains(event.target))) {
        // Call the method passed to the directive
        binding.value(event)
      }
    }
    document.body.addEventListener('click', el.clickOutsideEvent)
  },

  unbind: function (el) {
    document.body.removeEventListener('click', el.clickOutsideEvent)
  }
}

// main.js - Register globally
import Vue from 'vue'
import ClickOutside from './directives/click-outside'

Vue.directive('click-outside', ClickOutside)
```

**Usage:**

```vue
<template>
  <div v-click-outside="closeDropdown" class="dropdown">
    <button @click="isOpen = !isOpen">Toggle</button>
    <div v-if="isOpen" class="dropdown-menu">
      Dropdown content
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isOpen: false
    }
  },
  methods: {
    closeDropdown() {
      this.isOpen = false
    }
  }
}
</script>
```

## Example 3: Infinite Scroll Directive

```javascript
Vue.directive('infinite-scroll', {
  bind: function (el, binding) {
    const callback = binding.value
    const options = {
      rootMargin: '100px' // Start loading 100px before bottom
    }

    el.observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          callback()
        }
      })
    }, options)

    el.observer.observe(el)
  },

  unbind: function (el) {
    if (el.observer) {
      el.observer.disconnect()
    }
  }
})
```

**Usage:**

```vue
<template>
  <div class="scroll-container" v-infinite-scroll="loadMore">
    <div v-for="item in items" :key="item.id">
      {{ item.name }}
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: [],
      page: 1
    }
  },
  methods: {
    async loadMore() {
      const newItems = await fetchItems(this.page)
      this.items.push(...newItems)
      this.page++
    }
  }
}
</script>
```

## Example 4: Tooltip Directive

```javascript
Vue.directive('tooltip', {
  bind: function (el, binding) {
    const tooltipText = binding.value

    el.addEventListener('mouseenter', function () {
      const tooltip = document.createElement('div')
      tooltip.className = 'v-tooltip'
      tooltip.textContent = tooltipText
      tooltip.style.position = 'absolute'
      tooltip.style.background = '#333'
      tooltip.style.color = '#fff'
      tooltip.style.padding = '5px 10px'
      tooltip.style.borderRadius = '4px'
      tooltip.style.fontSize = '12px'
      tooltip.style.zIndex = '9999'
      tooltip.id = 'v-tooltip-' + Date.now()

      document.body.appendChild(tooltip)

      // Position tooltip above element
      const rect = el.getBoundingClientRect()
      tooltip.style.top = (rect.top + window.scrollY - tooltip.offsetHeight - 5) + 'px'
      tooltip.style.left = (rect.left + (rect.width - tooltip.offsetWidth) / 2) + 'px'

      el._tooltip = tooltip
    })

    el.addEventListener('mouseleave', function () {
      if (el._tooltip) {
        document.body.removeChild(el._tooltip)
        el._tooltip = null
      }
    })
  },

  unbind: function (el) {
    if (el._tooltip) {
      document.body.removeChild(el._tooltip)
    }
  }
})
```

**Usage:**

```vue
<template>
  <button v-tooltip="'Click here to submit'">Submit</button>
</template>
```

## Example 5: Lazy Load Images Directive

```javascript
Vue.directive('lazy', {
  bind: function (el, binding) {
    el.src = 'data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7' // 1x1 transparent

    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          const img = new Image()
          img.src = binding.value
          img.onload = () => {
            el.src = binding.value
            el.classList.add('loaded')
          }
          observer.unobserve(el)
        }
      })
    })

    observer.observe(el)
    el._lazyObserver = observer
  },

  unbind: function (el) {
    if (el._lazyObserver) {
      el._lazyObserver.disconnect()
    }
  }
})
```

**Usage:**

```vue
<template>
  <img v-lazy="imageUrl" alt="Lazy loaded image">
</template>

<script>
export default {
  data() {
    return {
      imageUrl: 'https://example.com/large-image.jpg'
    }
  }
}
</script>
```

## Example 6: Copy to Clipboard Directive

```javascript
Vue.directive('copy', {
  bind: function (el, binding) {
    el.addEventListener('click', async function () {
      try {
        await navigator.clipboard.writeText(binding.value)

        // Visual feedback
        const originalText = el.textContent
        el.textContent = 'Copied!'
        el.classList.add('copied')

        setTimeout(() => {
          el.textContent = originalText
          el.classList.remove('copied')
        }, 2000)
      } catch (err) {
        console.error('Failed to copy:', err)
      }
    })
  }
})
```

**Usage:**

```vue
<template>
  <button v-copy="textToCopy">Copy Text</button>
</template>

<script>
export default {
  data() {
    return {
      textToCopy: 'Hello, World!'
    }
  }
}
</script>
```

## Example 7: Using Modifiers and Arguments

```javascript
Vue.directive('color', {
  bind: function (el, binding) {
    const color = binding.value || 'black'
    const modifiers = binding.modifiers

    // Apply styles based on modifiers
    el.style.color = color

    if (modifiers.bold) {
      el.style.fontWeight = 'bold'
    }

    if (modifiers.underline) {
      el.style.textDecoration = 'underline'
    }

    // Use argument for property name
    if (binding.arg === 'bg') {
      el.style.backgroundColor = color
      el.style.color = '' // Reset color
    }
  },

  update: function (el, binding) {
    // React to value changes
    const color = binding.value || 'black'
    if (binding.arg === 'bg') {
      el.style.backgroundColor = color
    } else {
      el.style.color = color
    }
  }
})
```

**Usage:**

```vue
<template>
  <div>
    <!-- Simple usage -->
    <p v-color="'red'">Red text</p>

    <!-- With modifier -->
    <p v-color.bold="'blue'">Bold blue text</p>

    <!-- With multiple modifiers -->
    <p v-color.bold.underline="'green'">Bold, underlined green text</p>

    <!-- With argument -->
    <p v-color:bg="'yellow'">Yellow background</p>

    <!-- Dynamic value -->
    <p v-color="textColor">Dynamic color</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      textColor: 'purple'
    }
  }
}
</script>
```

## Example 8: Long Press Directive

```javascript
Vue.directive('longpress', {
  bind: function (el, binding) {
    if (typeof binding.value !== 'function') {
      console.warn('v-longpress requires a function value')
      return
    }

    let timer = null
    const duration = 500 // ms

    const start = (e) => {
      if (e.type === 'click' && e.button !== 0) return
      timer = setTimeout(() => {
        binding.value(e)
      }, duration)
    }

    const cancel = () => {
      if (timer) {
        clearTimeout(timer)
        timer = null
      }
    }

    el.addEventListener('mousedown', start)
    el.addEventListener('touchstart', start)
    el.addEventListener('mouseup', cancel)
    el.addEventListener('mouseleave', cancel)
    el.addEventListener('touchend', cancel)
    el.addEventListener('touchcancel', cancel)

    el._longpressHandlers = { start, cancel }
  },

  unbind: function (el) {
    if (el._longpressHandlers) {
      const { start, cancel } = el._longpressHandlers
      el.removeEventListener('mousedown', start)
      el.removeEventListener('touchstart', start)
      el.removeEventListener('mouseup', cancel)
      el.removeEventListener('mouseleave', cancel)
      el.removeEventListener('touchend', cancel)
      el.removeEventListener('touchcancel', cancel)
    }
  }
})
```

**Usage:**

```vue
<template>
  <button v-longpress="handleLongPress">Hold for 500ms</button>
</template>

<script>
export default {
  methods: {
    handleLongPress(event) {
      console.log('Long pressed!', event)
      // Show context menu, etc.
    }
  }
}
</script>
```

## Shorthand Directive Syntax

For simple directives, you can use function shorthand (bind + update):

```javascript
// Function shorthand equivalent to bind + update with same logic
Vue.directive('color', function (el, binding) {
  el.style.color = binding.value
})
```

## Local Directive Registration

```vue
<template>
  <input v-model="text" v-focus>
</template>

<script>
export default {
  directives: {
    focus: {
      inserted: function (el) {
        el.focus()
      }
    }
  },
  data() {
    return {
      text: ''
    }
  }
}
</script>
```

## Dynamic Directive Arguments

> **Note**: Dynamic directive arguments were introduced in Vue 2.6.0. This feature allows directive arguments to be dynamic JavaScript expressions.

```vue
<template>
  <!-- Dynamic attribute name (Vue 2.6.0+) -->
  <button v-bind:[key]="value">Dynamic</button>

  <!-- Dynamic event name (Vue 2.6.0+) -->
  <button @[event]="handler">Click</button>

  <!-- Dynamic directive argument (Vue 2.6.0+) -->
  <div v-color:[attribute]="value"></div>
</template>

<script>
export default {
  data() {
    return {
      attribute: 'border',
      key: 'id',
      value: 'red'
    }
  }
}
</script>
```

## Best Practices

1. **Always clean up** in the unbind hook
2. **Use unique IDs** for created DOM elements to avoid conflicts
3. **Store references** on the element for cleanup (el._property)
4. **Check value type** in bind for better error messages
5. **Prefer composition** over complex directives (consider components first)

## When to Use Directives vs Components

| Use Directives For | Use Components For |
|--------------------|-------------------|
| Low-level DOM manipulation | Complex UI with template |
| Behavior that applies to any element | Reusable visual components |
| Integrating 3rd party libraries | Self-contained functionality |
| Micro-features (focus, scroll, etc.) | Features with state and lifecycle |

## Reference

- [Vue 2 Guide - Custom Directives](https://vue2.docs.vuejs.org/v2/guide/custom-directive.html)
- [Vue 2 API - Vue.directive](https://vue2.docs.vuejs.org/v2/api/#Vue-directive)
