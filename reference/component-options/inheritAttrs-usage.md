---
title: inheritAttrs Option for Attribute Inheritance in Vue 2
impact: MEDIUM
impactDescription: inheritAttrs controls whether parent scope attribute bindings fall through to child component root
type: capability
tags:
  - vue2.6.14
  - inheritAttrs
  - attribute-inheritance
  - fallthrough-attributes
  - $attrs
---

# inheritAttrs Option for Attribute Inheritance in Vue 2

Impact: MEDIUM - Vue 2's `inheritAttrs` option controls whether parent-scope attribute bindings that are not recognized as props "fall through" to the child component's root element. Set to `false` to manually control attribute placement with `$attrs`.

## Task Checklist

- [ ] Use `inheritAttrs: false` when creating wrapper/base components
- [ ] Use `$attrs` to manually bind fallthrough attributes
- [ ] Remember `class` and `style` are always merged (not affected)
- [ ] Use `v-bind="$attrs"` for transparent wrapper components
- [ ] Keep `inheritAttrs: true` (default) for most components

## Default Behavior (inheritAttrs: true)

By default, parent attributes that aren't declared as props are automatically added to the root element:

```vue
<!-- Parent.vue -->
<template>
  <child-component
    class="parent-class"
    id="custom-id"
    data-value="123"
    title="Custom title"
  />
</template>

<!-- ChildComponent.vue -->
<template>
  <div class="default-class">
    Child content
  </div>
</template>

<!-- Rendered result (inheritAttrs: true is default) -->
<div
  class="parent-class default-class"
  id="custom-id"
  data-value="123"
  title="Custom title"
>
  Child content
</div>
```

## Disabling Automatic Inheritance

```vue
<!-- BaseButton.vue -->
<script>
export default {
  name: 'BaseButton',
  inheritAttrs: false, // Disable automatic attribute inheritance
  props: {
    type: {
      type: String,
      default: 'button'
    },
    disabled: Boolean
  }
}
</script>

<template>
  <button
    v-bind="$attrs"
    :type="type"
    :disabled="disabled"
    class="base-button"
  >
    <slot></slot>
  </button>
</template>
```

**Usage:**

```vue
<template>
  <div>
    <!-- These attributes will be bound to the button element -->
    <base-button
      class="my-button"
      id="submit-btn"
      data-track="submit"
      title="Click to submit"
      :disabled="isSubmitting"
    >
      Submit
    </base-button>
  </div>
</template>

<!-- Rendered result -->
<button
  type="button"
  class="base-button my-button"
  id="submit-btn"
  data-track="submit"
  title="Click to submit"
  disabled="false"
>
  Submit
</button>
```

## Creating a Wrapper Component

```vue
<!-- CustomInput.vue -->
<template>
  <div class="input-wrapper">
    <label v-if="label">{{ label }}</label>

    <!-- Manually bind $attrs to the input -->
    <input
      v-bind="$attrs"
      v-model="inputValue"
      class="custom-input"
    >

    <span v-if="error" class="error">{{ error }}</span>
  </div>
</template>

<script>
export default {
  name: 'CustomInput',
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
    }
  }
}
</script>

<style scoped>
.input-wrapper {
  margin-bottom: 1rem;
}
.custom-input {
  border: 1px solid #ccc;
  padding: 0.5rem;
  width: 100%;
}
.error {
  color: red;
  font-size: 0.875rem;
}
</style>
```

**Usage:**

```vue
<template>
  <div>
    <!-- All these attributes go to the input, not the wrapper -->
    <custom-input
      v-model="email"
      label="Email"
      type="email"
      placeholder="Enter your email"
      required
      maxlength="100"
      pattern="[^@]+@[^@]+\.[^@]+"
    />
  </div>
</template>
```

## Combining with Explicit Attributes

```vue
<!-- BaseButton.vue -->
<template>
  <button
    v-bind="$attrs"
    :type="nativeType"
    :disabled="disabled || loading"
    :class="buttonClass"
    @click="handleClick"
  >
    <slot v-if="!loading"></slot>
    <span v-else class="spinner"></span>
  </button>
</template>

<script>
export default {
  name: 'BaseButton',
  inheritAttrs: false,
  props: {
    nativeType: {
      type: String,
      default: 'button'
    },
    disabled: Boolean,
    loading: Boolean,
    variant: {
      type: String,
      default: 'default'
    }
  },
  computed: {
    buttonClass() {
      return [
        'base-button',
        `base-button--${this.variant}`
      ]
    }
  },
  methods: {
    handleClick(event) {
      if (!this.disabled && !this.loading) {
        this.$emit('click', event)
      }
    }
  }
}
</script>
```

**Usage:**

```vue
<template>
  <div>
    <!-- Explicit props + inheritAttrs -->
    <base-button
      variant="primary"
      loading
      class="my-custom-class"
      data-track="signup"
      title="Sign up for our service"
    >
      Sign Up
    </base-button>
  </div>
</template>
```

## Multiple Elements with $attrs

```vue
<!-- IconicButton.vue -->
<template>
  <div class="iconic-button-wrapper" v-bind="$attrs">
    <icon :name="icon" />
    <button class="iconic-button" @click="$emit('click')">
      <slot></slot>
    </button>
  </div>
</template>

<script>
export default {
  name: 'IconicButton',
  inheritAttrs: false,
  props: {
    icon: String
  }
}
</script>
```

## Non-Inherited Attributes

`class` and `style` are special and will always be merged:

```vue
<!-- ChildComponent.vue -->
<template>
  <div class="child-class" style="color: red;">
    Content
  </div>
</template>

<script>
export default {
  inheritAttrs: false // Still affects class and style
}
</script>

<!-- Usage -->
<child-component class="parent-class" style="font-size: 16px;" />

<!-- Rendered result -->
<div
  class="child-class parent-class"  <!-- Merged! -->
  style="color: red; font-size: 16px;"  <!-- Merged! -->
>
  Content
</div>
```

## Complete Example: Card Component

```vue
<!-- BaseCard.vue -->
<template>
  <div class="card">
    <header v-if="$slots.header" class="card-header" v-bind="headerAttrs">
      <slot name="header"></slot>
    </header>

    <div class="card-body">
      <slot></slot>
    </div>

    <footer v-if="$slots.footer" class="card-footer" v-bind="footerAttrs">
      <slot name="footer"></slot>
    </footer>
  </div>
</template>

<script>
export default {
  name: 'BaseCard',
  inheritAttrs: false,
  props: {
    // Card-specific props
    elevation: {
      type: Number,
      default: 1,
      validator: value => value >= 0 && value <= 24
    },
    rounded: {
      type: Boolean,
      default: true
    }
  },
  computed: {
    cardClasses() {
      return [
        `elevation-${this.elevation}`,
        { 'rounded': this.rounded }
      ]
    },
    // Split $attrs for different parts
    headerAttrs() {
      return {
        ...this.$attrs,
        class: 'card-header-attrs'
      }
    },
    footerAttrs() {
      return {
        ...this.$attrs,
        class: 'card-footer-attrs'
      }
    }
  }
}
</script>

<style scoped>
.card {
  background: white;
  transition: box-shadow 0.3s;
}
.rounded {
  border-radius: 8px;
}
.elevation-1 {
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}
.elevation-2 {
  box-shadow: 0 4px 8px rgba(0,0,0,0.15);
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
```

**Usage:**

```vue
<template>
  <base-card
    elevation="2"
    class="my-card"
    id="user-card"
  >
    <template #header>
      <h3>User Profile</h3>
    </template>

    <p>User information goes here</p>

    <template #footer>
      <button>Edit</button>
    </template>
  </base-card>
</template>
```

## Form Field Component

```vue
<!-- FormField.vue -->
<template>
  <div class="form-field" v-bind="wrapperAttrs">
    <label
      v-if="label"
      :for="inputId"
      class="form-field-label"
      v-bind="labelAttrs"
    >
      {{ label }}
      <span v-if="required" class="required">*</span>
    </label>

    <div class="form-field-input-wrapper">
      <!-- Pass remaining attrs to input -->
      <input
        :id="inputId"
        v-bind="inputAttrs"
        v-model="internalValue"
        class="form-field-input"
      >
    </div>

    <span v-if="error" class="form-field-error">{{ error }}</span>
    <span v-else-if="hint" class="form-field-hint">{{ hint }}</span>
  </div>
</template>

<script>
export default {
  name: 'FormField',
  inheritAttrs: false,
  props: {
    label: String,
    hint: String,
    error: String,
    required: Boolean,
    value: [String, Number]
  },
  data() {
    return {
      inputId: `input-${Math.random().toString(36).substr(2, 9)}`
    }
  },
  computed: {
    internalValue: {
      get() {
        return this.value
      },
      set(value) {
        this.$emit('input', value)
      }
    },
    // Split attrs for different elements
    wrapperAttrs() {
      const { class: wrapperClass, ...rest } = this.$attrs
      return {
        class: ['form-field-wrapper', wrapperClass]
      }
    },
    labelAttrs() {
      const { class, id, ...rest } = this.$attrs
      return rest
    },
    inputAttrs() {
      const { class: wrapperClass, ...rest } = this.$attrs
      return rest
    }
  }
}
</script>

<style scoped>
.form-field {
  margin-bottom: 1rem;
}
.form-field-label {
  display: block;
  margin-bottom: 0.25rem;
  font-weight: 500;
}
.required {
  color: red;
}
.form-field-input {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}
.form-field-error {
  color: red;
  font-size: 0.875rem;
  margin-top: 0.25rem;
}
.form-field-hint {
  color: #666;
  font-size: 0.875rem;
  margin-top: 0.25rem;
}
</style>
```

## Transparent Wrapper Pattern

```vue
<!-- TransparentWrapper.vue -->
<script>
export default {
  name: 'TransparentWrapper',
  inheritAttrs: false,
  functional: true,
  render(h, { children, data, props }) {
    return h('div', {
      class: 'transparent-wrapper',
      attrs: data.attrs  // Pass through all attributes
    }, children)
  }
}
</script>
```

## When to Use inheritAttrs: false

### ✅ Use Cases

1. **Wrapper components** - Wrapping inputs, buttons with extra elements
2. **Base UI components** - Design system components
3. **Complex components** - Multiple root elements or conditional roots
4. **Attribute redirection** - Attributes should go to non-root element

### ❌ Avoid When

1. **Simple components** - Single element, straightforward props
2. **No wrapper needed** - Direct element rendering
3. **Default behavior works** - No special attribute handling needed

## Comparison Table

| Feature | inheritAttrs: true (default) | inheritAttrs: false |
|---------|----------------------------|---------------------|
| Auto-attr binding | ✅ Yes | ❌ No |
| Manual control needed | ❌ No | ✅ Yes |
| Best for | Simple components | Wrapper/base components |
| Access to $attrs | ✅ Yes | ✅ Yes |
| class/style merging | ✅ Always | ✅ Always |

## Common Patterns

### Pattern 1: Input Wrapper

```vue
<template>
  <div class="input-group">
    <span class="prefix" v-if="$slots.prefix">
      <slot name="prefix"></slot>
    </span>
    <input v-bind="$attrs" v-model="value">
    <span class="suffix" v-if="$slots.suffix">
      <slot name="suffix"></slot>
    </span>
  </div>
</template>

<script>
export default {
  inheritAttrs: false,
  props: ['value']
}
</script>
```

### Pattern 2: Conditional Root Element

```vue
<template>
  <!-- Need inheritAttrs: false because root changes conditionally -->
  <component
    :is="tag"
    v-bind="$attrs"
    class="conditional-root"
  >
    <slot></slot>
  </component>
</template>

<script>
export default {
  inheritAttrs: false,
  props: {
    tag: {
      type: String,
      default: 'div'
    }
  }
}
</script>
```

### Pattern 3: Multiple Roots with Different Attrs

```vue
<template>
  <div class="outer">
    <div class="inner" v-bind="innerAttrs">
      <slot></slot>
    </div>
    <div class="footer" v-bind="footerAttrs">
      <slot name="footer"></slot>
    </div>
  </div>
</template>

<script>
export default {
  inheritAttrs: false,
  computed: {
    innerAttrs() {
      const { footer, ...inner } = this.$attrs
      return inner
    },
    footerAttrs() {
      return this.$attrs.footer || {}
    }
  }
}
</script>
```

## Reference

- [Vue 2 Guide - disable-attribute-inheritance](https://vue2.docs.vuejs.org/v2/guide/components-props.html#Disabling-Attribute-Inheritance)
- [Vue 2 API - inheritAttrs](https://vue2.docs.vuejs.org/v2/api/#inheritAttrs)
