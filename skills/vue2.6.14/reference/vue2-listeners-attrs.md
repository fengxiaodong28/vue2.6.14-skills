---
title: $listeners and $attrs in Vue 2
impact: MEDIUM
impactDescription: Understanding $listeners and $attrs for component communication
type: capability
tags:
  - vue2
  - attrs
  - listeners
  - component-communication
  - inherit-attrs
---

# $listeners and $attrs in Vue 2

Impact: MEDIUM - `$listeners` and `$attrs` provide flexible component composition and transparent wrapper components in Vue 2.

## Task Checklist

- [ ] Understand `$attrs` contains non-prop attributes
- [ ] Understand `$listeners` contains event listeners
- [ ] Use `inheritAttrs: false` to manual binding
- [ ] Create transparent wrapper components
- [ ] Note differences from Vue 3

## Understanding $attrs

`$attrs` contains attribute bindings (class, style, non-prop attributes) that are not recognized as props:

```vue
<!-- Parent Component -->
<template>
  <ChildComponent
    id="my-id"
    class="parent-class"
    data-custom="custom-value"
    title="Tooltip"
    v-model="value"
  />
</template>
```

```vue
<!-- ChildComponent.vue -->
<script>
export default {
  props: {
    value: String // This is a prop, won't be in $attrs
  },
  created() {
    console.log(this.$attrs)
    // Output:
    // {
    //   id: "my-id",
    //   class: "parent-class",
    //   "data-custom": "custom-value",
    //   title: "Tooltip"
    // }
  }
}
</script>
```

## Understanding $listeners

`$listeners` contains event listeners from parent:

```vue
<!-- Parent Component -->
<template>
  <ChildComponent
    @click="handleClick"
    @input="handleInput"
    @custom-event="handleCustom"
  />
</template>
```

```vue
<!-- ChildComponent.vue -->
<script>
export default {
  created() {
    console.log(this.$listeners)
    // Output:
    // {
    //   click: function() {...},
    //   input: function() {...},
    //   custom-event: function() {...}
    // }
  },
  methods: {
    emitClick() {
      this.$listeners.click(event)
      // Or use: this.$emit('click', event)
    }
  }
}
</script>
```

## Transparent Wrapper Component

Create a component that passes all attrs and listeners to inner element:

```vue
<!-- BaseInput.vue -->
<script>
export default {
  name: 'BaseInput',
  inheritAttrs: false, // Disable automatic attribute inheritance
  props: {
    value: [String, Number]
  },
  computed: {
    internalValue: {
      get() {
        return this.value
      },
      set(val) {
        this.$emit('input', val)
      }
    }
  }
}
</script>

<template>
  <div class="input-wrapper">
    <label v-if="label">{{ label }}</label>
    <!-- Pass $attrs to input element -->
    <input
      v-bind="$attrs"
      v-model="internalValue"
      class="form-input"
    />
  </div>
</template>
```

Usage:
```vue
<template>
  <!-- All non-prop attributes go to the input element -->
  <BaseInput
    v-model="username"
    type="text"
    placeholder="Enter username"
    maxlength="50"
    autofocus
    @focus="handleFocus"
    @blur="handleBlur"
  />
</template>
```

## Button Wrapper with $listeners

```vue
<!-- BaseButton.vue -->
<script>
export default {
  name: 'BaseButton',
  inheritAttrs: false,
  props: {
    variant: {
      type: String,
      default: 'default',
      validator: (value) => ['default', 'primary', 'danger'].includes(value)
    },
    loading: Boolean
  },
  computed: {
    buttonClasses() {
      return [`btn-${this.variant}`, { 'btn-loading': this.loading }]
    }
  }
}
</script>

<template>
  <!-- Pass $attrs and $listeners to button -->
  <button
    v-bind="$attrs"
    v-on="$listeners"
    :class="buttonClasses"
    :disabled="loading"
  >
    <span v-if="loading">Loading...</span>
    <slot v-else />
  </button>
</template>
```

Usage:
```vue
<template>
  <BaseButton
    variant="primary"
    loading
    @click="submit"
    @mouseenter="showTooltip"
    data-track="submit-button"
  >
    Submit
  </BaseButton>
</template>
```

## Form Component Wrapper

```vue
<!-- BaseForm.vue -->
<script>
export default {
  name: 'BaseForm',
  inheritAttrs: false,
  props: {
    model: Object,
    rules: Object
  },
  methods: {
    handleSubmit(event) {
      event.preventDefault()
      this.$emit('submit', this.model)
    },
    handleReset() {
      this.$emit('reset')
    }
  }
}
</script>

<template>
  <form v-bind="$attrs" @submit="handleSubmit" @reset="handleReset">
    <slot :model="model" :rules="rules" />
  </form>
</template>
```

## Select Wrapper Component

```vue
<!-- BaseSelect.vue -->
<script>
export default {
  name: 'BaseSelect',
  inheritAttrs: false,
  props: {
    value: [String, Number, Array],
    options: {
      type: Array,
      default: () => []
    },
    labelKey: {
      type: String,
      default: 'label'
    },
    valueKey: {
      type: String,
      default: 'value'
    }
  },
  computed: {
    internalValue: {
      get() {
        return this.value
      },
      set(val) {
        this.$emit('input', val)
      }
    }
  }
}
</script>

<template>
  <select
    v-bind="$attrs"
    v-model="internalValue"
    v-on="$listeners"
    class="form-select"
  >
    <option v-for="option in options" :key="option[valueKey]" :value="option[valueKey]">
      {{ option[labelKey] }}
    </option>
  </select>
</template>
```

## Multiple Event Handlers

```vue
<!-- ChildComponent.vue -->
<script>
export default {
  methods: {
    handleClick(event) {
      // Do something locally
      this.localHandler(event)

      // Emit to parent via $listeners
      if (this.$listeners.click) {
        this.$emit('click', event)
      }
    },
    localHandler(event) {
      console.log('Local click handler', event)
    }
  }
}
</script>

<template>
  <!-- Click is handled both locally and in parent -->
  <button
    v-bind="$attrs"
    v-on="{
      ...$listeners,
      click: handleClick
    }"
  >
    <slot />
  </button>
</template>
```

## Class and Style Merging

```vue
<!-- Parent passes class and style -->
<template>
  <ChildComponent class="parent-class" style="color: red" />
</template>
```

```vue
<!-- ChildComponent.vue -->
<script>
export default {
  inheritAttrs: false,
  props: {
    variant: String
  },
  computed: {
    wrapperClasses() {
      return [
        'child-wrapper',
        `child-${this.variant}`,
        // Merge parent class from $attrs
        this.$attrs.class
      ]
    },
    wrapperStyles() {
      return {
        // Merge parent styles
        ...this.$attrs.style,
        fontSize: '14px'
      }
    }
  }
}
</script>

<template>
  <div :class="wrapperClasses" :style="wrapperStyles">
    Content
  </div>
</template>
```

## Dynamic Component Wrapper

```vue
<!-- DynamicWrapper.vue -->
<script>
export default {
  name: 'DynamicWrapper',
  inheritAttrs: false,
  props: {
    tag: {
      type: String,
      default: 'div'
    }
  }
}
</script>

<template>
  <!-- Dynamic tag with attrs and listeners -->
  <component
    :is="tag"
    v-bind="$attrs"
    v-on="$listeners"
  >
    <slot />
  </component>
</template>
```

Usage:
```vue
<!-- Renders as <button> with all attrs/listeners -->
<DynamicWrapper tag="button" @click="handler">
  Click me
</DynamicWrapper>

<!-- Renders as <a> with all attrs/listeners -->
<DynamicWrapper tag="a" href="/home" @click.prevent="navigate">
  Go home
</DynamicWrapper>
```

## Vue 2 vs Vue 3 Difference

```javascript
// Vue 2
this.$attrs  // Non-prop attributes
this.$listeners  // Event listeners

// Vue 3 (merged into $attrs)
this.$attrs  // Includes both non-prop attributes AND event listeners
// $listeners no longer exists in Vue 3
```

For Vue 3 compatibility:

```vue
<!-- Vue 2 component written with Vue 3 compatibility in mind -->
<script>
export default {
  inheritAttrs: false,
  created() {
    // For Vue 3 compatibility
    const attrs = { ...this.$attrs }
    const listeners = this.$listeners || {}

    // In Vue 3, listeners would be in attrs with 'on' prefix
    // This prepares for potential migration
  }
}
</script>
```

## Patterns for Component Libraries

```vue
<!-- BaseButton.vue - Library component -->
<script>
export default {
  name: 'BaseButton',
  inheritAttrs: false,
  props: {
    variant: String,
    size: {
      type: String,
      default: 'medium',
      validator: (value) => ['small', 'medium', 'large'].includes(value)
    }
  },
  computed: {
    componentClasses() {
      return [
        'btn',
        `btn-${this.variant || 'default'}`,
        `btn-${this.size}`
      ]
    },
    // Filter out attributes we handle explicitly
    filteredAttrs() {
      const { class, style, ...rest } = this.$attrs
      return rest
    }
  }
}
</script>

<template>
  <button
    v-bind="filteredAttrs"
    v-on="$listeners"
    :class="componentClasses"
  >
    <slot />
  </button>
</template>
```

## Debugging $attrs and $listeners

```vue
<script>
export default {
  name: 'DebugComponent',
  inheritAttrs: false,
  created() {
    console.log('Attrs:', this.$attrs)
    console.log('Listeners:', Object.keys(this.$listeners))
  }
}
</script>

<template>
  <div v-bind="$attrs" v-on="$listeners">
    <div style="background: #f0f0f0; padding: 10px;">
      <h4>Debug Info</h4>
      <p><strong>$attrs:</strong> {{ $attrs }}</p>
      <p><strong>$listeners keys:</strong> {{ Object.keys($listeners) }}</p>
    </div>
    <slot />
  </div>
</template>
```

## Common Patterns

| Pattern | $attrs Usage | $listeners Usage |
|---------|--------------|------------------|
| Transparent wrapper | Pass to inner element | Pass to inner element |
| Filtered attrs | Remove handled props | Remove handled events |
| Enhanced component | Merge with local attrs | Merge with local handlers |
| Dynamic tag | Pass to component | Pass to component |

## Reference

- [Vue 2 Guide - $attrs](https://v2.vuejs.org/v2/api/#vm-attrs)
- [Vue 2 Guide - $listeners](https://v2.vuejs.org/v2/api/#vm-listeners)
- [Vue 2 Guide - inheritAttrs](https://v2.vuejs.org/v2/api/#inheritAttrs)
