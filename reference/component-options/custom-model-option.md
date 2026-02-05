---
title: Custom v-model Behavior with model Option
impact: MEDIUM
impactDescription: The model option customizes prop and event used by v-model for two-way binding
type: capability
tags:
  - vue2.6.14
  - v-model
  - model-option
  - two-way-binding
  - custom-components
---

# Custom v-model Behavior with model Option

Impact: MEDIUM - Vue 2's `model` option allows components to customize the prop and event used by `v-model`. By default, `v-model` uses `value` as the prop and `input` as the event, but this can be changed for custom components.

## Task Checklist

- [ ] Use `model` option to customize v-model prop and event
- [ ] Default is `value` prop + `input` event
- [ ] Common alternatives: `checked` + `change` for checkboxes
- [ ] Emit update event matching the model configuration
- [ ] Document custom v-model behavior for component users

## Default v-model Behavior

By default, `v-model` on a component uses:

- **Prop**: `value`
- **Event**: `input`

```vue
<!-- ChildComponent.vue -->
<template>
  <input
    :value="value"
    @input="$emit('input', $event.target.value)"
  >
</template>

<script>
export default {
  props: ['value']
}
</script>

<!-- Usage -->
<template>
  <child-component v-model="message" />
  <!-- Equivalent to: -->
  <child-component
    :value="message"
    @input="message = $event"
  />
</template>
```

## Custom Model for Checkbox

```vue
<!-- CustomCheckbox.vue -->
<template>
  <input
    type="checkbox"
    :checked="checked"
    @change="$emit('change', $event.target.checked)"
  >
</template>

<script>
export default {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    checked: Boolean
  }
}
</script>

<!-- Usage -->
<template>
  <div>
    <custom-checkbox v-model="isActive" />
    <p>Active: {{ isActive }}</p>
  </div>
</template>

<script>
import CustomCheckbox from './CustomCheckbox.vue'

export default {
  components: { CustomCheckbox },
  data() {
    return {
      isActive: false
    }
  }
}
</script>
```

## Custom Model for Radio Buttons

```vue
<!-- CustomRadio.vue -->
<template>
  <label>
    <input
      type="radio"
      :name="name"
      :value="value"
      :checked="checked === value"
      @change="$emit('change', value)"
    >
    <slot></slot>
  </label>
</template>

<script>
export default {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    name: String,
    value: {
      required: true
    },
    checked: {
      required: true
    }
  }
}
</script>

<!-- Usage -->
<template>
  <div>
    <custom-radio v-model="selected" value="option1">
      Option 1
    </custom-radio>
    <custom-radio v-model="selected" value="option2">
      Option 2
    </custom-radio>
    <custom-radio v-model="selected" value="option3">
      Option 3
    </custom-radio>
    <p>Selected: {{ selected }}</p>
  </div>
</template>

<script>
import CustomRadio from './CustomRadio.vue'

export default {
  components: { CustomRadio },
  data() {
    return {
      selected: 'option1'
    }
  }
}
</script>
```

## Custom Model for Select Dropdown

```vue
<!-- CustomSelect.vue -->
<template>
  <select
    :value="selected"
    @change="$emit('change', $event.target.value)"
  >
    <slot></slot>
  </select>
</template>

<script>
export default {
  model: {
    prop: 'selected',
    event: 'change'
  },
  props: {
    selected: {
      required: true
    }
  }
}
</script>

<!-- Usage -->
<template>
  <div>
    <custom-select v-model="color">
      <option value="">Select a color</option>
      <option value="red">Red</option>
      <option value="green">Green</option>
      <option value="blue">Blue</option>
    </custom-select>
    <p>Color: {{ color }}</p>
  </div>
</template>
```

## Number Input with Custom Model

```vue
<!-- NumberInput.vue -->
<template>
  <input
    type="number"
    :value="numberValue"
    @input="handleInput"
    @change="handleChange"
  >
</template>

<script>
export default {
  model: {
    prop: 'numberValue',
    event: 'update'
  },
  props: {
    numberValue: {
      type: Number,
      default: 0
    },
    min: Number,
    max: Number
  },
  methods: {
    handleInput(event) {
      let value = parseFloat(event.target.value)

      // Validate
      if (isNaN(value)) {
        value = 0
      }

      if (this.min !== undefined && value < this.min) {
        value = this.min
      }

      if (this.max !== undefined && value > this.max) {
        value = this.max
      }

      this.$emit('update', value)
    },
    handleChange(event) {
      this.$emit('change', parseFloat(event.target.value))
    }
  }
}
</script>

<!-- Usage -->
<template>
  <div>
    <number-input v-model="quantity" :min="1" :max="100" />
    <p>Quantity: {{ quantity }}</p>
  </div>
</template>
```

## Range Slider Component

```vue
<!-- RangeSlider.vue -->
<template>
  <div class="range-slider">
    <input
      type="range"
      :min="min"
      :max="max"
      :step="step"
      :value="currentValue"
      @input="handleInput"
      @change="handleChange"
    >
    <span class="value">{{ currentValue }}</span>
  </div>
</template>

<script>
export default {
  model: {
    prop: 'currentValue',
    event: 'update'
  },
  props: {
    currentValue: {
      type: Number,
      default: 0
    },
    min: {
      type: Number,
      default: 0
    },
    max: {
      type: Number,
      default: 100
    },
    step: {
      type: Number,
      default: 1
    }
  },
  methods: {
    handleInput(event) {
      this.$emit('update', parseFloat(event.target.value))
    },
    handleChange(event) {
      this.$emit('change', parseFloat(event.target.value))
    }
  }
}
</script>

<style scoped>
.range-slider {
  display: flex;
  align-items: center;
  gap: 12px;
}
.range-slider input[type="range"] {
  flex: 1;
}
.value {
  min-width: 40px;
  text-align: right;
}
</style>

<!-- Usage -->
<template>
  <div>
    <range-slider
      v-model="volume"
      :min="0"
      :max="100"
      :step="5"
      @change="onVolumeChange"
    />
    <p>Volume: {{ volume }}%</p>
  </div>
</template>

<script>
import RangeSlider from './RangeSlider.vue'

export default {
  components: { RangeSlider },
  data() {
    return {
      volume: 50
    }
  },
  methods: {
    onVolumeChange(newValue) {
      console.log('Volume changed to:', newValue)
    }
  }
}
</script>
```

## Toggle Switch Component

```vue
<!-- ToggleSwitch.vue -->
<template>
  <button
    class="toggle-switch"
    :class="{ 'toggle-switch--on': toggled }"
    @click="toggle"
    type="button"
  >
    <span class="toggle-slider"></span>
  </button>
</template>

<script>
export default {
  model: {
    prop: 'toggled',
    event: 'toggle'
  },
  props: {
    toggled: {
      type: Boolean,
      default: false
    }
  },
  methods: {
    toggle() {
      this.$emit('toggle', !this.toggled)
    }
  }
}
</script>

<style scoped>
.toggle-switch {
  position: relative;
  width: 50px;
  height: 26px;
  background: #ccc;
  border: none;
  border-radius: 13px;
  padding: 0;
  cursor: pointer;
  transition: background 0.3s;
}
.toggle-switch--on {
  background: #4caf50;
}
.toggle-slider {
  position: absolute;
  top: 3px;
  left: 3px;
  width: 20px;
  height: 20px;
  background: white;
  border-radius: 50%;
  transition: transform 0.3s;
}
.toggle-switch--on .toggle-slider {
  transform: translateX(24px);
}
</style>

<!-- Usage -->
<template>
  <div>
    <toggle-switch v-model="notificationsEnabled" />
    <p>Notifications: {{ notificationsEnabled ? 'On' : 'Off' }}</p>
  </div>
</template>
```

## Complete Form Component with Custom Model

```vue
<!-- FormInput.vue -->
<template>
  <div class="form-input" :class="{ 'has-error': error }">
    <label v-if="label">{{ label }}</label>
    <input
      :type="type"
      :value="inputValue"
      :placeholder="placeholder"
      :disabled="disabled"
      @input="handleInput"
      @blur="handleBlur"
      @focus="handleFocus"
    >
    <span v-if="error" class="error-message">{{ error }}</span>
  </div>
</template>

<script>
export default {
  model: {
    prop: 'inputValue',
    event: 'update'
  },
  props: {
    inputValue: {
      type: [String, Number],
      default: ''
    },
    label: String,
    type: {
      type: String,
      default: 'text'
    },
    placeholder: String,
    disabled: Boolean,
    error: String,
    validate: Function
  },
  methods: {
    handleInput(event) {
      const value = event.target.value

      // Run validation if provided
      if (this.validate) {
        const error = this.validate(value)
        this.$emit('validate', error)
      }

      this.$emit('update', value)
      this.$emit('input', value)
    },
    handleBlur(event) {
      this.$emit('blur', event.target.value)
    },
    handleFocus(event) {
      this.$emit('focus', event.target.value)
    }
  }
}
</script>

<style scoped>
.form-input {
  margin-bottom: 1rem;
}
.form-input label {
  display: block;
  margin-bottom: 0.25rem;
  font-weight: 500;
}
.form-input input {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}
.form-input.has-error input {
  border-color: red;
}
.error-message {
  color: red;
  font-size: 0.875rem;
  margin-top: 0.25rem;
}
</style>

<!-- Usage -->
<template>
  <form @submit.prevent="handleSubmit">
    <form-input
      v-model="formData.email"
      label="Email"
      type="email"
      placeholder="Enter your email"
      :validate="validateEmail"
      @validate="emailError = $event"
      :error="emailError"
    />

    <form-input
      v-model="formData.password"
      label="Password"
      type="password"
      placeholder="Enter your password"
    />

    <button type="submit">Submit</button>
  </form>
</template>

<script>
import FormInput from './FormInput.vue'

export default {
  components: { FormInput },
  data() {
    return {
      formData: {
        email: '',
        password: ''
      },
      emailError: ''
    }
  },
  methods: {
    validateEmail(value) {
      if (!value) return 'Email is required'
      if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) {
        return 'Invalid email format'
      }
      return ''
    },
    handleSubmit() {
      if (!this.emailError) {
        console.log('Form valid:', this.formData)
      }
    }
  }
}
</script>
```

## Multiple v-model on Same Component

```vue
<!-- CurrencyInput.vue -->
<template>
  <div class="currency-input">
    <span class="symbol">{{ symbol }}</span>
    <input
      type="text"
      :value="formattedValue"
      @input="handleInput"
    >
  </div>
</template>

<script>
export default {
  model: {
    prop: 'value',
    event: 'input'
  },
  props: {
    value: {
      type: Number,
      default: 0
    },
    symbol: {
      type: String,
      default: '$'
    }
  },
  computed: {
    formattedValue() {
      return this.value.toFixed(2)
    }
  },
  methods: {
    handleInput(event) {
      const parsed = parseFloat(event.target.value)
      if (!isNaN(parsed)) {
        this.$emit('input', parsed)
      }
    }
  }
}
</script>

<!-- Usage: Can still use multiple bindings with .sync -->
<template>
  <div>
    <currency-input
      :value.sync="amount"
      :symbol.sync="currencySymbol"
    />
  </div>
</template>
```

## Custom Model with Validation

```vue
<!-- ValidatedInput.vue -->
<script>
export default {
  model: {
    prop: 'validatedValue',
    event: 'validated'
  },
  props: {
    validatedValue: {
      required: true
    },
    validationRules: {
      type: Array,
      default: () => []
    }
  },
  computed: {
    validationResult() {
      for (const rule of this.validationRules) {
        const result = rule(this.validatedValue)
        if (result !== true) {
          return result
        }
      }
      return null
    }
  },
  methods: {
    updateValue(value) {
      this.$emit('validated', value)
      this.$emit('validation', this.validationResult)
    }
  }
}
</script>
```

## Documentation Pattern

```vue
<script>
/**
 * CustomSelect Component
 *
 * Custom v-model:
 * - Prop: `selected` (the selected value)
 * - Event: `change` (emitted when selection changes)
 *
 * @example
 * <custom-select v-model="selectedValue">
 *   <option value="1">Option 1</option>
 *   <option value="2">Option 2</option>
 * </custom-select>
 */
export default {
  model: {
    prop: 'selected',
    event: 'change'
  },
  // ...
}
</script>
```

## Summary Table

| Component Type | Prop | Event | Use Case |
|---------------|------|-------|----------|
| Default (text input) | value | input | Text inputs, general use |
| Checkbox | checked | change | Checkboxes, toggles |
| Radio | checked | change | Radio button groups |
| Select | selected | change | Dropdown selects |
| Custom | *any* | *any* | Specialized components |

## Best Practices

1. **Choose meaningful prop/event names** for your component's purpose
2. **Follow conventions** (checked/change for checkboxes, etc.)
3. **Document custom model** behavior for component users
4. **Emit validation events** alongside model updates
5. **Consider using `.sync`** for complex two-way bindings

## Reference

- [Vue 2 Guide - Custom Events](https://vue2.docs.vuejs.org/v2/guide/components-custom-events.html#customizing-component-v-model)
- [Vue 2 API - model](https://vue2.docs.vuejs.org/v2/api/#model)
