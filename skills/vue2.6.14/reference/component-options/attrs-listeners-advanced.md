---
title: Transparent Wrapper Components with $attrs and $listeners
impact: MEDIUM
impactDescription: $attrs and $listeners enable creating transparent wrapper components that pass through attributes and events
type: capability
tags:
  - vue2.6.14
  - $attrs
  - $listeners
  - transparent-wrappers
  - attribute-delegation
  - event-delegation
---

# Transparent Wrapper Components with $attrs and $listeners

Impact: MEDIUM - Vue 2's `$attrs` and `$listeners` enable creating transparent wrapper components that pass through parent attributes and events to child elements. This is essential for building wrapper components, base UI components, and higher-order components.

## Task Checklist

- [ ] Use `$attrs` to pass through non-prop attributes
- [ ] Use `$listeners` to pass through event listeners
- [ ] Set `inheritAttrs: false` for manual attribute control
- [ ] Remember `class` and `style` are handled specially
- [ ] Use for wrapper components and base UI components

## What Are $attrs and $listeners?

```javascript
export default {
  mounted() {
    // $attrs: Attributes passed to component that aren't declared as props
    console.log(this.$attrs)
    // Does NOT include: class, style (handled separately)

    // $listeners: Event listeners passed to component
    console.log(this.$listeners)
    // Includes: @click, @input, @focus, etc. (except .native modifiers)
  }
}
```

## Basic Transparent Wrapper

```vue
<!-- BaseInput.vue -->
<template>
  <div class="input-wrapper">
    <label v-if="label">{{ label }}</label>
    <!-- Pass through all attributes and listeners -->
    <input
      v-bind="$attrs"
      v-on="$listeners"
      v-model="inputValue"
    >
  </div>
</template>

<script>
export default {
  name: 'BaseInput',
  inheritAttrs: false, // Disable automatic inheritance
  props: {
    label: String,
    value: [String, Number]
  },
  computed: {
    inputValue: {
      get() {
        return this.value
      },
      set(value) {
        this.$emit('input', value)
      }
    }
  }
}
</script>

<!-- Usage -->
<template>
  <base-input
    v-model="email"
    label="Email"
    type="email"
    placeholder="Enter email"
    required
    @focus="handleFocus"
    @blur="handleBlur"
  />
</template>
```

## Complete Wrapper Component

```vue
<!-- CustomButton.vue -->
<template>
  <button
    v-bind="$attrs"
    v-on="$listeners"
    :class="buttonClass"
    :disabled="disabled || loading"
  >
    <span v-if="loading" class="spinner"></span>
    <slot v-else></slot>
  </button>
</template>

<script>
export default {
  name: 'CustomButton',
  inheritAttrs: false,
  props: {
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
  computed: {
    buttonClass() {
      return [
        'custom-button',
        `custom-button--${this.variant}`,
        `custom-button--${this.size}`
      ]
    }
  }
}
</script>

<style scoped>
.custom-button {
  /* Base styles */
}
.custom-button--primary {
  background: #007bff;
}
.custom-button--small {
  padding: 4px 12px;
}
</style>

<!-- Usage -->
<template>
  <custom-button
    variant="primary"
    size="large"
    loading
    class="my-custom-class"
    id="submit-btn"
    data-track="submit"
    @click="handleSubmit"
    @mouseover="handleHover"
  >
    Submit
  </custom-button>
</template>
```

## Wrapping Third-Party Components

```vue
<!-- Select2Wrapper.vue -->
<template>
  <select2
    v-bind="$attrs"
    v-on="$listeners"
    :options="localOptions"
    @change="handleChange"
  >
    <slot></slot>
  </select2>
</template>

<script>
import Select2 from 'vue-select2'

export default {
  name: 'Select2Wrapper',
  components: { Select2 },
  inheritAttrs: false,
  props: {
    options: Array
  },
  computed: {
    localOptions() {
      // Transform options if needed
      return this.options.map(opt => ({
        id: opt.id,
        text: opt.label
      }))
    }
  },
  methods: {
    handleChange(value) {
      // Emit in different format if needed
      this.$emit('input', value)
      this.$emit('change', value)
    }
  }
}
</script>
```

## Multiple Child Elements

```vue
<!-- FormField.vue -->
<template>
  <div class="form-field">
    <!-- Pass some attrs to label -->
    <label v-if="label" v-bind="labelAttrs">
      {{ label }}
    </label>

    <!-- Pass remaining attrs to input -->
    <input
      v-bind="inputAttrs"
      v-on="$listeners"
      v-model="inputValue"
    >

    <span v-if="error" class="error">{{ error }}</span>
  </div>
</template>

<script>
export default {
  name: 'FormField',
  inheritAttrs: false,
  props: {
    label: String,
    value: [String, Number],
    error: String
  },
  computed: {
    inputValue: {
      get() {
        return this.value
      },
      set(value) {
        this.$emit('input', value)
      }
    },
    // Split attrs between label and input
    labelAttrs() {
      const { id, ariaLabel, ariaDescribedby, ...rest } = this.$attrs
      return {
        for: id,
        'aria-label': ariaLabel
      }
    },
    inputAttrs() {
      const { id, ariaLabel, ariaDescribedby } = this.$attrs
      return {
        ...this.$attrs,
        id: id,
        'aria-label': ariaLabel,
        'aria-describedby': ariaDescribedby
      }
    }
  }
}
</script>
```

## Conditional Root Element

```vue
<!-- SmartLink.vue -->
<template>
  <component
    :is="tag"
    v-bind="$attrs"
    v-on="$listeners"
  >
    <slot></slot>
  </component>
</template>

<script>
export default {
  name: 'SmartLink',
  inheritAttrs: false,
  props: {
    to: [String, Object],
    href: String
  },
  computed: {
    tag() {
      if (this.to) {
        return 'router-link'
      } else if (this.href) {
        return 'a'
      } else {
        return 'button'
      }
    }
  }
}
</script>

<!-- Usage -->
<template>
  <div>
    <!-- Renders as router-link -->
    <smart-link to="/about" class="nav-link">
      About
    </smart-link>

    <!-- Renders as anchor -->
    <smart-link href="https://example.com" class="external-link">
      External
    </smart-link>

    <!-- Renders as button -->
    <smart-link @click="handleClick" class="action-btn">
      Click Me
    </smart-link>
  </div>
</template>
```

## Card Wrapper Component

```vue
<!-- BaseCard.vue -->
<template>
  <div class="card" :class="cardClasses">
    <div v-if="$slots.header" class="card-header">
      <slot name="header"></slot>
    </div>

    <div class="card-body">
      <slot></slot>
    </div>

    <div v-if="$slots.footer" class="card-footer">
      <slot name="footer"></slot>
    </div>
  </div>
</template>

<script>
export default {
  name: 'BaseCard',
  inheritAttrs: false,
  props: {
    variant: {
      type: String,
      default: 'default'
    },
    elevation: {
      type: Number,
      default: 1
    },
    rounded: {
      type: Boolean,
      default: true
    }
  },
  computed: {
    cardClasses() {
      return [
        `card--${this.variant}`,
        `elevation-${this.elevation}`,
        { 'rounded': this.rounded }
      ]
    }
  },
  mounted() {
    // Log $attrs for debugging
    console.log('Non-prop attributes:', this.$attrs)
    console.log('Event listeners:', this.$listeners)
  }
}
</script>

<style scoped>
.card {
  background: white;
}
.rounded {
  border-radius: 8px;
}
.elevation-1 {
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}
.card-header {
  padding: 16px;
  border-bottom: 1px solid #eee;
}
.card-body {
  padding: 16px;
}
.card-footer {
  padding: 16px;
  border-top: 1px solid #eee;
}
</style>

<!-- Usage -->
<template>
  <base-card
    variant="primary"
    :elevation="2"
    class="user-card"
    id="user-123"
    data-user-id="123"
    @click="handleCardClick"
  >
    <template #header>
      <h3>User Profile</h3>
    </template>

    <p>User content goes here</p>

    <template #footer>
      <button>Edit</button>
    </template>
  </base-card>
</template>
```

## Creating a Design System Base Component

```vue
<!-- BaseButton.vue -->
<script>
export default {
  name: 'BaseButton',
  inheritAttrs: false,
  props: {
    tag: {
      type: String,
      default: 'button'
    },
    type: {
      type: String,
      default: 'button'
    },
    variant: {
      type: String,
      default: 'default'
    },
    size: {
      type: String,
      default: 'medium'
    },
    block: {
      type: Boolean,
      default: false
    },
    disabled: Boolean
  },
  computed: {
    componentProps() {
      // Props to pass to the root element
      return {
        type: this.type,
        disabled: this.disabled || this.loading
      }
    },
    componentAttrs() {
      // Non-prop attrs to pass (excluding class and style which are merged)
      const attrs = { ...this.$attrs }
      delete attrs.class
      delete attrs.style
      return attrs
    },
    componentListeners() {
      // All listeners pass through
      return this.$listeners
    },
    componentClasses() {
      return [
        'base-button',
        `base-button--${this.variant}`,
        `base-button--${this.size}`,
        {
          'base-button--block': this.block,
          'base-button--disabled': this.disabled
        }
      ]
    }
  },
  render(h) {
    return h(this.tag, {
      attrs: {
        ...this.componentAttrs,
        ...this.componentProps
      },
      class: this.componentClasses,
      on: this.componentListeners
    }, this.$slots.default)
  }
}
</script>

<style scoped>
.base-button {
  /* Base styles */
}
.base-button--primary {
  background: #007bff;
  color: white;
}
.base-button--small {
  padding: 4px 12px;
  font-size: 0.875rem;
}
.base-button--block {
  width: 100%;
}
</style>
```

## Event Transformation

```vue
<!-- NumberInput.vue -->
<template>
  <input
    v-bind="$attrs"
    :value="inputValue"
    @input="handleInput"
    @blur="handleBlur"
  >
</template>

<script>
export default {
  name: 'NumberInput',
  inheritAttrs: false,
  props: {
    value: [String, Number],
    min: Number,
    max: Number
  },
  data() {
    return {
      inputValue: this.value
    }
  },
  methods: {
    handleInput(event) {
      let value = event.target.value

      // Validate and transform
      const numValue = parseFloat(value)

      if (!isNaN(numValue)) {
        // Emit transformed event
        this.$emit('input', numValue)
      }
    },
    handleBlur(event) {
      const numValue = parseFloat(event.target.value)

      // Validate on blur
      if (isNaN(numValue)) {
        this.$emit('input', this.min || 0)
      } else if (this.min !== undefined && numValue < this.min) {
        this.$emit('input', this.min)
      } else if (this.max !== undefined && numValue > this.max) {
        this.$emit('input', this.max)
      }

      // Forward blur event with transformed value
      this.$emit('blur', parseFloat(event.target.value))
    }
  }
}
</script>
```

## Combining with $slots

```vue
<!-- BaseModal.vue -->
<template>
  <div
    v-if="show"
    v-bind="$attrs"
    class="modal-overlay"
    @click.self="handleOverlayClick"
  >
    <div class="modal-content" :class="sizeClass">
      <div v-if="$slots.header" class="modal-header">
        <slot name="header"></slot>
        <button @click="close" class="close-btn">&times;</button>
      </div>

      <div class="modal-body">
        <slot></slot>
      </div>

      <div v-if="$slots.footer" class="modal-footer">
        <slot name="footer"></slot>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'BaseModal',
  inheritAttrs: false,
  props: {
    show: Boolean,
    size: {
      type: String,
      default: 'medium',
      validator: value => ['small', 'medium', 'large'].includes(value)
    },
    closeOnOverlayClick: {
      type: Boolean,
      default: true
    }
  },
  computed: {
    sizeClass() {
      return `modal-content--${this.size}`
    }
  },
  methods: {
    close() {
      this.$emit('update:show', false)
      this.$emit('close')
    },
    handleOverlayClick() {
      if (this.closeOnOverlayClick) {
        this.close()
      }
    }
  }
}
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0,0,0,0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}
.modal-content {
  background: white;
  border-radius: 8px;
  max-height: 90vh;
  overflow: auto;
}
.modal-content--small {
  width: 300px;
}
.modal-content--medium {
  width: 500px;
}
.modal-content--large {
  width: 800px;
}
</style>
```

## Delegation Patterns

### Pattern 1: Complete Delegation

```vue
<script>
export default {
  // Pass EVERYTHING through
  inheritAttrs: false,
  render(h) {
    return h('input', {
      attrs: this.$attrs,
      on: this.$listeners
    })
  }
}
</script>
```

### Pattern 2: Selective Delegation

```vue
<script>
export default {
  inheritAttrs: false,
  props: ['value'],
  render(h) {
    return h('input', {
      attrs: {
        type: 'text',
        ...this.$attrs
      },
      on: {
        ...this.$listeners,
        // Override specific event
        input: this.handleInput
      },
      domProps: {
        value: this.value
      }
    })
  },
  methods: {
    handleInput(event) {
      // Transform value
      const transformed = event.target.value.toUpperCase()
      this.$emit('input', transformed)
    }
  }
}
</script>
```

### Pattern 3: Split Delegation

```vue
<script>
export default {
  inheritAttrs: false,
  render(h) {
    return h('div', {
      attrs: {
        class: 'wrapper',
        id: this.$attrs.id
      }
    }, [
      // Pass remaining attrs to child
      h('input', {
        attrs: this.$attrs,
        on: this.$listeners
      })
    ])
  }
}
</script>
```

## Complex Wrapper with Multiple Children

```vue
<!-- DateRangePicker.vue -->
<template>
  <div class="date-range-picker" v-bind="wrapperAttrs">
    <!-- Start date input -->
    <input
      v-bind="startInputAttrs"
      v-on="startInputListeners"
      :value="startValue"
      @input="updateStart"
    >

    <span class="separator">to</span>

    <!-- End date input -->
    <input
      v-bind="endInputAttrs"
      v-on="endInputListeners"
      :value="endValue"
      @input="updateEnd"
    >
  </div>
</template>

<script>
export default {
  name: 'DateRangePicker',
  inheritAttrs: false,
  props: {
    start: String,
    end: String
  },
  computed: {
    startValue() {
      return this.start
    },
    endValue() {
      return this.end
    },
    wrapperAttrs() {
      const { 'start-id': startId, 'end-id': endId, ...attrs } = this.$attrs
      return attrs
    },
    startInputAttrs() {
      const { 'start-id': startId, ...attrs } = this.$attrs
      return {
        id: startId,
        placeholder: 'Start date',
        type: 'date'
      }
    },
    endInputAttrs() {
      const { 'end-id': endId, ...attrs } = this.$attrs
      return {
        id: endId,
        placeholder: 'End date',
        type: 'date'
      }
    },
    startInputListeners() {
      const { 'start-change': startChange, ...listeners } = this.$listeners
      return listeners
    },
    endInputListeners() {
      const { 'end-change': endChange, ...listeners } = this.$listeners
      return listeners
    }
  },
  methods: {
    updateStart(value) {
      this.$emit('update:start', value)
      this.$emit('start-change', value)
    },
    updateEnd(value) {
      this.$emit('update:end', value)
      this.$emit('end-change', value)
    }
  }
}
</script>
```

## Debugging $attrs and $listeners

```vue
<script>
export default {
  mounted() {
    // Debug helper
    console.group('Component Props and Attrs')
    console.log('Props:', this.$options.props)
    console.log('$attrs:', this.$attrs)
    console.log('$listeners:', this.$listeners)
    console.groupEnd()

    // See which attrs are NOT recognized as props
    const nonProps = Object.keys(this.$attrs)
    console.log('Non-prop attributes:', nonProps)

    // See which listeners are registered
    const listeners = Object.keys(this.$listeners)
    console.log('Event listeners:', listeners)
  }
}
</script>
```

## Best Practices

1. **Always set `inheritAttrs: false`** when manually handling `$attrs`
2. **Document delegation behavior** for component consumers
3. **Use `v-bind="$attrs"`** for transparent attribute passing
4. **Use `v-on="$listeners"`** for transparent event passing
5. **Transform events** when needed between wrapper and child

## Common Pitfalls

### Pitfall 1: Forgetting v-bind and v-on

```vue
<!-- ❌ WRONG: attrs and listeners not passed -->
<template>
  <input>
  </input>
</template>

<!-- ✅ CORRECT: Use v-bind and v-on -->
<template>
  <input
    v-bind="$attrs"
    v-on="$listeners"
  >
  </input>
</template>
```

### Pitfall 2: Double Delegation

```vue
<!-- ❌ WRONG: Both automatic and manual inheritance -->
<script>
export default {
  // inheritAttrs defaults to true!
  // This causes double application of attrs
  render(h) {
    return h('input', {
      attrs: this.$attrs
    })
  }
}
</script>

<!-- ✅ CORRECT: Disable automatic inheritance -->
<script>
export default {
  inheritAttrs: false,
  render(h) {
    return h('input', {
      attrs: this.$attrs
    })
  }
}
</script>
```

## Summary Table

| Property | Contains | Common Use |
|----------|----------|------------|
| `$attrs` | Non-prop attributes (except class/style) | `v-bind="$attrs"` |
| `$listeners` | Event listeners (except .native) | `v-on="$listeners"` |
| `$props` | Declared props | Validation, internal use |
| `class`/`style` | Merged separately | Always merged |

## Reference

- [Vue 2 Guide - disable-attribute-inheritance](https://vue2.docs.vuejs.org/v2/guide/components-props.html#Disabling-Attribute-Inheritance)
- [Vue 2 API - vm.$attrs](https://vue2.docs.vuejs.org/v2/api/#vm-attrs)
- [Vue 2 API - vm.$listeners](https://vue2.docs.vuejs.org/v2/api/#vm-listeners)
