---
title: v-model Modifiers in Vue 2
impact: MEDIUM
impactDescription: v-model modifiers (.lazy, .number, .trim) change how form input binding behaves
type: capability
tags:
  - vue2.6.14
  - v-model
  - form-input
  - modifiers
  - two-way-binding
---

# v-model Modifiers in Vue 2

Impact: MEDIUM - Vue 2's `v-model` directive supports modifiers that change how form input bindings work. Use `.lazy`, `.number`, and `.trim` to control when and how data is updated.

## Task Checklist

- [ ] Use `.lazy` to update on change event instead of input
- [ ] Use `.number` to automatically convert input to number
- [ ] Use `.trim` to automatically trim whitespace
- [ ] Understand that modifiers can be chained
- [ ] Create custom modifiers for components

## Overview of Built-in Modifiers

| Modifier | Behavior | Event |
|----------|----------|-------|
| `.lazy` | Syncs after `change` event (blur) | `change` |
| `.number` | Converts value to number | `input` |
| `.trim` | Trims whitespace | `input` |
| Chained | Apply multiple modifiers | varies |

## Default v-model Behavior

By default, `v-model` syncs input with each `input` event:

```vue
<template>
  <div>
    <input v-model="message" placeholder="Type something">
    <p>You typed: {{ message }}</p>
    <!-- Updates on every keystroke -->
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: ''
    }
  }
}
</script>
```

## .lazy Modifier

Syncs on `change` event instead of `input` (when input loses focus):

```vue
<template>
  <div>
    <!-- Without .lazy: Updates on every keystroke -->
    <input v-model="message" placeholder="Regular v-model">
    <p>Regular: {{ message }}</p>

    <!-- With .lazy: Updates when input loses focus -->
    <input v-model.lazy="lazyMessage" placeholder="Lazy v-model">
    <p>Lazy: {{ lazyMessage }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: '',
      lazyMessage: ''
    }
  }
}
</script>
```

**Use cases for .lazy:**
- Reducing re-renders for long text inputs
- Form fields where you don't need instant validation
- Search inputs where user types full query before searching

```vue
<!-- Search with lazy loading -->
<template>
  <div>
    <input v-model.lazy="searchQuery" @change="performSearch" placeholder="Search...">
    <div v-for="result in searchResults" :key="result.id">
      {{ result.name }}
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      searchQuery: '',
      searchResults: []
    }
  },
  methods: {
    async performSearch() {
      if (this.searchQuery.length < 2) {
        this.searchResults = []
        return
      }
      this.searchResults = await api.search(this.searchQuery)
    }
  }
}
</script>
```

## .number Modifier

Automatically converts input value to a number:

```vue
<template>
  <div>
    <label>Age:</label>
    <input v-model.number="age" type="number">

    <label>Price:</label>
    <input v-model.number="price" type="number" step="0.01">

    <p>Age type: {{ typeof age }} (value: {{ age }})</p>
    <p>Price type: {{ typeof price }} (value: {{ price }})</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      age: '',
      price: ''
    }
  }
}
</script>
```

**Important notes about .number:**
- Returns `NaN` if input can't be converted to a number
- Returns empty string if input is empty
- Works with type="number" and type="text"

```vue
<template>
  <div>
    <input v-model.number="quantity" type="text" placeholder="Enter number">

    <!-- Handle NaN and empty values -->
    <p>Valid: {{ isValidNumber }}</p>
    <p>Doubled: {{ doubledQuantity }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      quantity: ''
    }
  },
  computed: {
    isValidNumber() {
      return typeof this.quantity === 'number' && !isNaN(this.quantity)
    },
    doubledQuantity() {
      return this.isValidNumber ? this.quantity * 2 : 'N/A'
    }
  }
}
</script>
```

## .trim Modifier

Automatically trims whitespace from input:

```vue
<template>
  <div>
    <label>Username (no spaces):</label>
    <input v-model.trim="username" placeholder="Enter username">

    <p>Value: "{{ username }}"</p>
    <p>Length: {{ username.length }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      username: ''
    }
  }
}
</script>
```

**Use cases for .trim:**
- Username/email fields
- Tags or keywords
- Preventing accidental spaces

## Chaining Modifiers

Multiple modifiers can be combined:

```vue
<template>
  <div>
    <!-- Lazy + Number -->
    <input v-model.lazy.number="age" type="text" placeholder="Age">
    <p>Age (lazy number): {{ age }}, type: {{ typeof age }}</p>

    <!-- Lazy + Trim -->
    <input v-model.lazy.trim="username" placeholder="Username">
    <p>Username (lazy trim): "{{ username }}"</p>

    <!-- All three -->
    <input v-model.lazy.number.trim="price" placeholder="Price">
    <p>Price (lazy number trim): {{ price }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      age: null,
      username: '',
      price: null
    }
  }
}
</script>
```

## Custom v-model Modifiers

Components can define custom modifiers with the `model` property:

```vue
<!-- CustomCurrencyInput.vue -->
<template>
  <input
    :value="value"
    @input="onInput"
    @blur="onBlur"
    @change="onChange"
  >
</template>

<script>
export default {
  name: 'CustomCurrencyInput',
  props: {
    value: [String, Number]
  },
  methods: {
    onInput(event) {
      let value = event.target.value

      // Apply .trim modifier
      if (this.$vnode.data.modifiers?.trim) {
        value = value.trim()
      }

      // Apply .uppercase modifier (custom)
      if (this.$vnode.data.modifiers?.uppercase) {
        value = value.toUpperCase()
      }

      // Apply .lowercase modifier (custom)
      if (this.$vnode.data.modifiers?.lowercase) {
        value = value.toLowerCase()
      }

      this.$emit('input', value)
    },

    onBlur(event) {
      // For .lazy modifier - emit on blur
      let value = event.target.value
      this.$emit('change', value)
    },

    onChange(event) {
      // For .lazy modifier
      let value = event.target.value
      this.$emit('change', value)
    }
  }
}
</script>
```

**Usage with custom modifiers:**

```vue
<template>
  <div>
    <!-- With .trim -->
    <custom-currency-input v-model.trim="field1">

    <!-- With .uppercase (custom) -->
    <custom-currency-input v-model.uppercase="field2">

    <!-- With .lazy + .trim -->
    <custom-currency-input v-model.lazy.trim="field3">
  </div>
</template>

<script>
import CustomCurrencyInput from './CustomCurrencyInput.vue'

export default {
  components: { CustomCurrencyInput },
  data() {
    return {
      field1: '',
      field2: '',
      field3: ''
    }
  }
}
</script>
```

## Using Modifiers with Select Elements

```vue
<template>
  <div>
    <!-- Select with .number modifier -->
    <select v-model.number="selectedId">
      <option :value="null">Select an option</option>
      <option v-for="item in items" :key="item.id" :value="item.id">
        {{ item.name }}
      </option>
    </select>

    <p>Selected ID: {{ selectedId }} (type: {{ typeof selectedId }})</p>

    <!-- Multi-select with .number -->
    <select v-model.number="selectedIds" multiple>
      <option v-for="item in items" :key="item.id" :value="item.id">
        {{ item.name }}
      </option>
    </select>

    <p>Selected IDs: {{ selectedIds }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      selectedId: null,
      selectedIds: [],
      items: [
        { id: 1, name: 'Option 1' },
        { id: 2, name: 'Option 2' },
        { id: 3, name: 'Option 3' }
      ]
    }
  }
}
</script>
```

## Form Validation with Modifiers

```vue
<template>
  <form @submit.prevent="submitForm">
    <div>
      <label>Email:</label>
      <input v-model.trim="email" type="email">
      <span v-if="errors.email" class="error">{{ errors.email }}</span>
    </div>

    <div>
      <label>Age:</label>
      <input v-model.lazy.number="age" type="text" min="0" max="120">
      <span v-if="errors.age" class="error">{{ errors.age }}</span>
    </div>

    <div>
      <label>Username:</label>
      <input v-model.trim="username">
      <span v-if="errors.username" class="error">{{ errors.username }}</span>
    </div>

    <button type="submit">Submit</button>
  </form>
</template>

<script>
export default {
  data() {
    return {
      email: '',
      age: null,
      username: '',
      errors: {}
    }
  },
  methods: {
    submitForm() {
      this.errors = {}

      // Validate email
      if (!this.email) {
        this.errors.email = 'Email is required'
      } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(this.email)) {
        this.errors.email = 'Invalid email format'
      }

      // Validate age (number modifier ensures it's a number)
      if (this.age === null || this.age === '') {
        this.errors.age = 'Age is required'
      } else if (this.age < 0 || this.age > 120) {
        this.errors.age = 'Age must be between 0 and 120'
      } else if (isNaN(this.age)) {
        this.errors.age = 'Age must be a valid number'
      }

      // Validate username (trim modifier removes spaces)
      if (!this.username) {
        this.errors.username = 'Username is required'
      } else if (this.username.length < 3) {
        this.errors.username = 'Username must be at least 3 characters'
      }

      if (Object.keys(this.errors).length === 0) {
        console.log('Form valid:', { email: this.email, age: this.age, username: this.username })
      }
    }
  }
}
</script>
```

## Performance Considerations

```vue
<template>
  <!-- Without .lazy: Updates on every keystroke (can cause many re-renders) -->
  <textarea v-model="longText" @input="updateSuggestions"></textarea>

  <!-- With .lazy: Updates only on blur (better performance) -->
  <textarea v-model.lazy="longText" @change="updateSuggestions"></textarea>
</template>
```

## Common Patterns

### 1. Search Input with Debounce and Lazy

```vue
<template>
  <div>
    <input
      v-model.lazy="searchQuery"
      @change="handleSearch"
      placeholder="Search..."
    >
    <p>Results for: {{ searchQuery }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      searchQuery: ''
    }
  },
  methods: {
    handleSearch() {
      // Search only when user is done typing (on blur/enter)
      this.$emit('search', this.searchQuery)
    }
  }
}
</script>
```

### 2. Numeric Input with Validation

```vue
<template>
  <div>
    <input
      v-model.lazy.number="quantity"
      type="text"
      min="1"
      placeholder="Quantity"
    >
    <button @click="increment">+</button>
    <button @click="decrement">-</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      quantity: 1
    }
  },
  methods: {
    increment() {
      if (typeof this.quantity === 'number') {
        this.quantity++
      }
    },
    decrement() {
      if (typeof this.quantity === 'number' && this.quantity > 1) {
        this.quantity--
      }
    }
  }
}
</script>
```

## Reference

- [Vue 2 Guide - Form Input Bindings](https://vue2.docs.vuejs.org/v2/guide/forms.html)
- [Vue 2 API - v-model](https://vue2.docs.vuejs.org/v2/api/#v-model)
