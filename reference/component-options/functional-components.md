---
title: Functional Components in Vue 2
impact: LOW
impactDescription: Functional components are stateless, instanceless components with better performance for simple presentational components
type: capability
tags:
  - vue2.6.14
  - functional-components
  - performance
  - stateless-components
  - render-functions
---

# Functional Components in Vue 2

Impact: LOW - Functional components in Vue 2 are stateless and instanceless, offering better performance for simple presentational components. They have no `this` context, no lifecycle hooks, and only receive `context` as an argument.

## Task Checklist

- [ ] Use functional components for simple presentational components
- [ ] Pass `functional: true` in component options
- [ ] Use `context` instead of `this` for accessing props, children, slots
- [ ] Remember there's no `this`, no data, no computed properties
- [ ] Consider performance benefits vs regular components

## Basic Functional Component

```javascript
// FunctionalComponent.vue
export default {
  functional: true,
  props: {
    message: String
  },
  render(createElement, context) {
    // No 'this' - use context.props instead
    return createElement('div', context.props.message)
  }
}
```

## Context Object Properties

```javascript
export default {
  functional: true,
  render(createElement, context) {
    // context.props: Props passed to component
    console.log(context.props)

    // context.children: Array of child VNodes
    console.log(context.children)

    // context.slots: Functions for named slots
    console.log(context.slots())

    // context.data: Object passed to component (attrs, props, class, style)
    console.log(context.data)

    // context.parent: Parent component instance
    console.log(context.parent)

    // context.listeners: Event listeners
    console.log(context.listeners)

    // context.injections: Injected properties
    console.log(context.injections)

    // context.scopedSlots: Scoped slot functions
    console.log(context.scopedSlots)
  }
}
```

## Functional vs Regular Component

```javascript
// Regular component
const RegularComponent = {
  data() {
    return {
      count: 0
    }
  },
  methods: {
    increment() {
      this.count++
    }
  },
  render(createElement) {
    return createElement('button', {
      on: {
        click: this.increment
      }
    }, `Count: ${this.count}`)
  }
}

// Functional component (no state, no this)
const FunctionalComponent = {
  functional: true,
  props: {
    count: Number,
    onIncrement: Function
  },
  render(createElement, { props, listeners }) {
    return createElement('button', {
      on: {
        click: listeners.increment
      }
    }, `Count: ${props.count}`)
  }
}
```

## Practical Examples

### 1. Heading Component

```vue
<!-- Heading.vue -->
<script>
export default {
  functional: true,
  props: {
    level: {
      type: Number,
      default: 1,
      validator: value => value >= 1 && value <= 6
    },
    text: String
  },
  render(createElement, { props, data, children }) {
    return createElement(
      `h${props.level}`,
      data,
      children || props.text
    )
  }
}
</script>

<!-- Usage -->
<template>
  <div>
    <heading :level="1" text="Main Title" />
    <heading :level="2">Subtitle</heading>
    <heading :level="3">Custom Text</heading>
  </div>
</template>
```

### 2. Truncate Text Component

```vue
<!-- TruncateText.vue -->
<script>
export default {
  functional: true,
  props: {
    text: String,
    length: {
      type: Number,
      default: 50
    }
  },
  render(createElement, { props }) {
    const truncated = props.text.length > props.length
      ? props.text.substring(0, props.length) + '...'
      : props.text

    return createElement('span', {
      attrs: {
        title: props.text
      }
    }, truncated)
  }
}
</script>

<!-- Usage -->
<template>
  <truncate-text
    :text="longDescription"
    :length="100"
  />
</template>
```

### 3. Button Component with Variants

```vue
<!-- BaseButton.vue -->
<script>
export default {
  functional: true,
  props: {
    type: {
      type: String,
      default: 'button',
      validator: value => ['button', 'submit', 'reset'].includes(value)
    },
    variant: {
      type: String,
      default: 'default',
      validator: value => ['default', 'primary', 'danger', 'success'].includes(value)
    },
    size: {
      type: String,
      default: 'medium',
      validator: value => ['small', 'medium', 'large'].includes(value)
    },
    disabled: Boolean,
    loading: Boolean
  },
  render(createElement, { props, listeners, children, data }) {
    // Build class array
    const classes = [
      'base-button',
      `base-button--${props.variant}`,
      `base-button--${props.size}`,
      {
        'base-button--disabled': props.disabled,
        'base-button--loading': props.loading
      }
    ]

    // Merge with any classes passed to component
    if (data.class) {
      classes.push(data.class)
    }

    // Merge static class
    if (data.staticClass) {
      classes.push(data.staticClass)
    }

    return createElement('button', {
      attrs: {
        type: props.type,
        disabled: props.disabled || props.loading,
        ...data.attrs
      },
      class: classes,
      on: listeners
    }, [
      props.loading ? createElement('span', { class: 'spinner' }) : null,
      children
    ])
  }
}
</script>

<style scoped>
.base-button {
  /* Base styles */
}
.base-button--primary {
  background: #007bff;
}
.base-button--small {
  padding: 4px 8px;
}
.base-button--medium {
  padding: 8px 16px;
}
.base-button--large {
  padding: 12px 24px;
}
.base-button--loading {
  position: relative;
  pointer-events: none;
}
.spinner {
  /* Spinner styles */
}
</style>
```

### 4. List Item Component

```vue
<!-- ListItem.vue -->
<script>
export default {
  functional: true,
  props: {
    item: Object,
    index: Number,
    active: Boolean
  },
  render(createElement, { props, listeners, slots }) {
    const classes = {
      'list-item': true,
      'list-item--active': props.active
    }

    return createElement('li', {
      class: classes,
      on: {
        click: () => listeners.click?.(props.item, props.index)
      }
    }, [
      // Default slot
      slots().default || createElement('span', props.item.name),

      // Named slots
      slots().icon ? createElement('span', { class: 'list-item__icon' }, slots().icon) : null,

      slots().action ? createElement('span', { class: 'list-item__action' }, slots().action) : null
    ])
  }
}
</script>

<!-- Usage -->
<template>
  <ul>
    <list-item
      v-for="(item, index) in items"
      :key="item.id"
      :item="item"
      :index="index"
      :active="selectedItem === item"
      @click="selectItem"
    >
      {{ item.name }}
      <template #icon>
        <icon :name="item.icon" />
      </template>
      <template #action>
        <button @click.stop="edit(item)">Edit</button>
      </template>
    </list-item>
  </ul>
</template>
```

## Using Template Syntax (Functional Option)

```vue
<!-- FunctionalTemplate.vue -->
<template functional>
  <div class="card" :class="props.variant">
    <h3 v-if="props.title">{{ props.title }}</h3>
    <slot></slot>
    <slot name="footer"></slot>
  </div>
</template>

<script>
export default {
  functional: true,
  props: {
    title: String,
    variant: {
      type: String,
      default: 'default'
    }
  }
}
</script>

<!-- Usage -->
<template>
  <functional-card variant="primary" title="Card Title">
    <p>Card content goes here</p>
    <template #footer>
      <button>Action</button>
    </template>
  </functional-template>
</template>
```

## Passing Through Attributes

```vue
<!-- TransparentWrapper.vue -->
<script>
export default {
  functional: true,
  render(createElement, { children, data }) {
    // Pass through all attributes, listeners, etc.
    return createElement('div', data, children)
  }
}
</script>

<!-- Usage -->
<template>
  <transparent-wrapper
    class="my-class"
    @click="handleClick"
    data-id="123"
  >
    Content
  </transparent-wrapper>

  <!-- Renders: -->
  <div class="my-class" data-id="123">Content</div>
</template>
```

## Functional Component with JSX

```vue
<!-- FunctionalJsx.vue -->
<script>
export default {
  functional: true,
  props: {
    level: Number,
    text: String
  },
  render(h, { props, children }) {
    const Tag = `h${props.level || 1}`
    return (
      <Tag class="heading">
        {children || props.text}
      </Tag>
    )
  }
}
</script>
```

## Working with Slots

```vue
<!-- SlotContainer.vue -->
<script>
export default {
  functional: true,
  render(createElement, { slots }) {
    // Access default slot
    const defaultSlot = slots().default || []

    // Access named slots
    const header = slots().header || []
    const footer = slots().footer || []

    return createElement('div', { class: 'container' }, [
      createElement('header', {}, header),
      createElement('main', {}, defaultSlot),
      createElement('footer', {}, footer)
    ])
  }
}
</script>

<!-- Usage -->
<template>
  <slot-container>
    <template #header>
      <h1>Page Title</h1>
    </template>

    <p>Main content goes here</p>

    <template #footer>
      <p>© 2024</p>
    </template>
  </slot-container>
</template>
```

## Performance Considerations

```javascript
// Functional components are faster because:
// 1. No component instance creation
// 2. No lifecycle hooks
// 3. Less memory overhead

// But consider:
// - Only useful for simple presentational components
// - Regular components are fine for most cases
// - Vue 3 handles regular components more efficiently

// When to use functional components:
// - Wrapper components
// - Simple presentational components
// - High-volume rendering (hundreds/thousands of items)
```

## When NOT to Use Functional Components

```vue
<script>
// ❌ AVOID: Components with state
export default {
  functional: true,
  // No data(), no computed, no methods
  // These won't work!
}

// ❌ AVOID: Components that need lifecycle hooks
export default {
  functional: true,
  // No mounted, beforeDestroy, etc.
  // Can't use third-party libraries that need hooks
}

// ❌ AVOID: Complex components with logic
export default {
  functional: true,
  // Complex logic makes code harder to read
  // Better to use regular component
}
</script>
```

## Comparison Table

| Feature | Regular Component | Functional Component |
|---------|------------------|---------------------|
| `this` context | ✅ Yes | ❌ No |
| data() | ✅ Yes | ❌ No |
| computed | ✅ Yes | ❌ No |
| methods | ✅ Yes | ❌ No |
| lifecycle hooks | ✅ Yes | ❌ No |
| props | ✅ Yes | ✅ Yes (via context.props) |
| slots | ✅ Yes | ✅ Yes (via context.slots) |
| performance | Good | Better |
| memory usage | Higher | Lower |
| use case | Most components | Simple presentational |

## Best Practices

1. **Use functional components** for simple presentational components
2. **Keep them simple** - complex logic defeats the purpose
3. **Prefer template syntax** when possible (easier to read)
4. **Pass through attributes** transparently
5. **Document that it's functional** for other developers

## Reference

- [Vue 2 Guide - Functional Components](https://vue2.docs.vuejs.org/v2/guide/render-function.html#Functional-Components)
- [Vue 2 API - functional](https://vue2.docs.vuejs.org/v2/api/#functional)
