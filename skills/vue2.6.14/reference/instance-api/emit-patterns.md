---
title: Custom Events with $emit in Vue 2
impact: HIGH
impactDescription: $emit enables child-to-parent communication through custom events
type: capability
tags:
  - vue2.6.14
  - events
  - $emit
  - child-to-parent
  - component-communication
---

# Custom Events with $emit in Vue 2

Impact: HIGH - Vue 2's `$emit` method enables child-to-parent communication through custom events. This is the primary way for child components to communicate with their parents, maintaining proper unidirectional data flow.

## Task Checklist

- [ ] Use `$emit` to send events from child to parent
- [ ] Define emitted events in component `emits` option (Vue 2.7+) or document them
- [ ] Pass data to parent as event payload
- [ ] Use kebab-case for event names in templates
- [ ] Consider event validation and documentation

## Basic Event Emission

```vue
<!-- ChildComponent.vue -->
<template>
  <button @click="handleClick">
    Click Me
  </button>
</template>

<script>
export default {
  methods: {
    handleClick() {
      // Emit a custom event
      this.$emit('button-clicked')
    }
  }
}
</script>

<!-- Parent.vue -->
<template>
  <div>
    <child-component @button-clicked="handleButtonClick" />
  </div>
</template>

<script>
import ChildComponent from './ChildComponent.vue'

export default {
  components: { ChildComponent },
  methods: {
    handleButtonClick() {
      console.log('Button was clicked!')
    }
  }
}
</script>
```

## Emitting with Data

```vue
<!-- ChildComponent.vue -->
<template>
  <div>
    <input v-model="message" type="text">
    <button @click="sendToParent">Send to Parent</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: ''
    }
  },
  methods: {
    sendToParent() {
      // Emit event with data payload
      this.$emit('message-sent', this.message)
    }
  }
}
</script>

<!-- Parent.vue -->
<template>
  <div>
    <child-component @message-sent="handleMessage" />
    <p>Received: {{ receivedMessage }}</p>
  </div>
</template>

<script>
import ChildComponent from './ChildComponent.vue'

export default {
  components: { ChildComponent },
  data() {
    return {
      receivedMessage: ''
    }
  },
  methods: {
    handleMessage(message) {
      this.receivedMessage = message
    }
  }
}
</script>
```

## Emitting Multiple Values

```vue
<!-- ChildComponent.vue -->
<template>
  <button @click="submitData">
    Submit
  </button>
</template>

<script>
export default {
  data() {
    return {
      username: 'john',
      email: 'john@example.com',
      age: 30
    }
  },
  methods: {
    submitData() {
      // Emit multiple values as separate arguments
      this.$emit('submit', this.username, this.email, this.age)

      // Or emit as an object (often cleaner)
      this.$emit('submit', {
        username: this.username,
        email: this.email,
        age: this.age
      })
    }
  }
}
</script>

<!-- Parent.vue -->
<template>
  <child-component @submit="handleSubmit" />
</template>

<script>
import ChildComponent from './ChildComponent.vue'

export default {
  components: { ChildComponent },
  methods: {
    // Separate arguments
    handleSubmit(username, email, age) {
      console.log('Username:', username)
      console.log('Email:', email)
      console.log('Age:', age)
    }

    // Or with object destructuring
    // handleSubmit({ username, email, age }) {
    //   console.log(username, email, age)
    // }
  }
}
</script>
```

## Named Events with Parameters

```vue
<!-- UserCard.vue -->
<template>
  <div class="user-card">
    <h3>{{ user.name }}</h3>
    <p>{{ user.email }}</p>
    <button @click="edit">Edit</button>
    <button @click="delete">Delete</button>
  </div>
</template>

<script>
export default {
  props: {
    user: {
      type: Object,
      required: true
    }
  },
  methods: {
    edit() {
      this.$emit('edit', this.user.id)
    },
    delete() {
      this.$emit('delete', this.user.id, this.user.name)
    }
  }
}
</script>

<!-- Parent.vue -->
<template>
  <div>
    <user-card
      v-for="user in users"
      :key="user.id"
      :user="user"
      @edit="handleEdit"
      @delete="handleDelete"
    />
  </div>
</template>

<script>
import UserCard from './UserCard.vue'

export default {
  components: { UserCard },
  data() {
    return {
      users: [
        { id: 1, name: 'John', email: 'john@example.com' },
        { id: 2, name: 'Jane', email: 'jane@example.com' }
      ]
    }
  },
  methods: {
    handleEdit(userId) {
      console.log('Edit user:', userId)
    },
    handleDelete(userId, userName) {
      if (confirm(`Delete user ${userName}?`)) {
        this.users = this.users.filter(u => u.id !== userId)
      }
    }
  }
}
</script>
```

## v-on with Inline Statements

```vue
<template>
  <div>
    <!-- Call method -->
    <child-component @event="handlerMethod" />

    <!-- Inline statement with $event -->
    <child-component @event="count += 1" />

    <!-- Inline statement with data -->
    <child-component @event="message = $event" />

    <!-- Inline statement with method -->
    <child-component @event="handleEvent($event, 'extra')" />
  </div>
</template>

<script>
export default {
  data() {
    return {
      count: 0,
      message: ''
    }
  },
  methods: {
    handlerMethod(payload) {
      console.log('Received:', payload)
    },
    handleEvent(payload, extra) {
      console.log('Payload:', payload, 'Extra:', extra)
    }
  }
}
</script>
```

## Event Validation (Vue 2.7+)

```vue
<script>
export default {
  // Define emitted events for validation
  emits: {
    // No validation
    click: null,

    // With validation
    submit: payload => {
      if (!payload.username) {
        console.warn('Submit event requires username')
        return false
      }
      if (payload.username.length < 3) {
        console.warn('Username must be at least 3 characters')
        return false
      }
      return true
    }
  },
  methods: {
    handleSubmit() {
      // This will be validated against the emits definition
      this.$emit('submit', { username: this.username })
    }
  }
}
</script>
```

## Common Event Patterns

### 1. Form Submission

```vue
<!-- FormComponent.vue -->
<template>
  <form @submit.prevent="handleSubmit">
    <input v-model="formData.name" type="text" placeholder="Name">
    <input v-model="formData.email" type="email" placeholder="Email">
    <button type="submit">Submit</button>
  </form>
</template>

<script>
export default {
  data() {
    return {
      formData: {
        name: '',
        email: ''
      }
    }
  },
  methods: {
    handleSubmit() {
      // Validate
      if (!this.formData.name || !this.formData.email) {
        this.$emit('validation-error', 'All fields are required')
        return
      }

      // Emit success with form data
      this.$emit('submit', { ...this.formData })
    }
  }
}
</script>
```

### 2. Toggle/Change Events

```vue
<!-- ToggleSwitch.vue -->
<template>
  <button
    @click="toggle"
    :class="{ active: isActive }"
    class="toggle"
  >
    <span class="slider"></span>
  </button>
</template>

<script>
export default {
  props: {
    value: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      isActive: this.value
    }
  },
  watch: {
    value(newVal) {
      this.isActive = newVal
    }
  },
  methods: {
    toggle() {
      this.isActive = !this.isActive
      // Emit change event with new value
      this.$emit('input', this.isActive)
      // Also emit a named change event
      this.$emit('change', this.isActive)
    }
  }
}
</script>
```

### 3. Confirmation Dialog

```vue
<!-- ConfirmDialog.vue -->
<template>
  <div v-if="visible" class="modal-overlay" @click.self="cancel">
    <div class="modal">
      <h3>{{ title }}</h3>
      <p>{{ message }}</p>
      <div class="actions">
        <button @click="confirm" class="btn-primary">
          {{ confirmText }}
        </button>
        <button @click="cancel" class="btn-secondary">
          {{ cancelText }}
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    visible: Boolean,
    title: {
      type: String,
      default: 'Confirm'
    },
    message: String,
    confirmText: {
      type: String,
      default: 'Confirm'
    },
    cancelText: {
      type: String,
      default: 'Cancel'
    }
  },
  methods: {
    confirm() {
      this.$emit('confirm')
      this.$emit('update:visible', false)
    },
    cancel() {
      this.$emit('cancel')
      this.$emit('update:visible', false)
    }
  }
}
</script>

<!-- Usage -->
<template>
  <div>
    <button @click="showDeleteDialog = true">Delete Item</button>

    <confirm-dialog
      :visible.sync="showDeleteDialog"
      title="Delete Item"
      message="Are you sure you want to delete this item?"
      confirm-text="Delete"
      cancel-text="Cancel"
      @confirm="deleteItem"
      @cancel="onCancel"
    />
  </div>
</template>
```

### 4. Loading States

```vue
<!-- DataLoader.vue -->
<template>
  <div class="data-loader">
    <slot v-if="!loading && !error" :data="data" />

    <div v-if="loading" class="loading">
      Loading...
    </div>

    <div v-if="error" class="error">
      {{ error }}
      <button @click="retry">Retry</button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    url: String
  },
  data() {
    return {
      loading: false,
      error: null,
      data: null
    }
  },
  mounted() {
    this.loadData()
  },
  methods: {
    async loadData() {
      this.loading = true
      this.error = null

      try {
        const response = await fetch(this.url)
        this.data = await response.json()
        this.$emit('loaded', this.data)
      } catch (err) {
        this.error = err.message
        this.$emit('error', err)
      } finally {
        this.loading = false
      }
    },
    retry() {
      this.loadData()
      this.$emit('retry')
    }
  }
}
</script>
```

### 5. Pagination

```vue
<!-- Pagination.vue -->
<template>
  <div class="pagination">
    <button
      @click="goToPage(currentPage - 1)"
      :disabled="currentPage === 1"
    >
      Previous
    </button>

    <span class="page-info">
      Page {{ currentPage }} of {{ totalPages }}
    </span>

    <button
      @click="goToPage(currentPage + 1)"
      :disabled="currentPage === totalPages"
    >
      Next
    </button>
  </div>
</template>

<script>
export default {
  props: {
    totalItems: {
      type: Number,
      required: true
    },
    itemsPerPage: {
      type: Number,
      default: 10
    },
    currentPage: {
      type: Number,
      default: 1
    }
  },
  computed: {
    totalPages() {
      return Math.ceil(this.totalItems / this.itemsPerPage)
    }
  },
  methods: {
    goToPage(page) {
      if (page < 1 || page > this.totalPages) return

      this.$emit('page-changed', page)

      // Also emit offset/limit for API requests
      const offset = (page - 1) * this.itemsPerPage
      this.$emit('fetch-data', { offset, limit: this.itemsPerPage })
    }
  }
}
</script>
```

## Event Naming Conventions

```vue
<script>
export default {
  methods: {
    // ✅ GOOD: Use kebab-case for event names
    sendUpdate() {
      this.$emit('update-value')
      this.$emit('item-selected')
      this.$emit('form-submit')
    },

    // ❌ AVOID: camelCase (though it works)
    sendUpdate() {
      this.$emit('updateValue')
      this.$emit('itemSelected')
      this.$emit('formSubmit')
    }
  }
}
</script>

<!-- In template, use kebab-case -->
<template>
  <child-component
    @update-value="handleUpdate"
    @item-selected="handleSelect"
    @form-submit="handleSubmit"
  />
</template>
```

## Documenting Events

```vue
<script>
/**
 * UserForm Component
 *
 * @emits {string} submit - Emitted when form is submitted with user data
 * @emits {string} cancel - Emitted when user clicks cancel
 * @emits {Error} error - Emitted when validation fails
 *
 * @example
 * <user-form
 *   @submit="handleSubmit"
 *   @cancel="handleCancel"
 *   @error="handleError"
 * />
 */
export default {
  // ...
}
</script>
```

## Best Practices

1. **Use descriptive event names** that indicate what happened
2. **Pass relevant data** as event payload
3. **Use kebab-case** for event names
4. **Document emitted events** for better DX
5. **Validate event payloads** (Vue 2.7+)
6. **Consider using `.sync`** for simple two-way binding

## Common Pitfalls

### Pitfall 1: Mutating Props

```vue
<!-- ❌ WRONG: Child trying to mutate prop directly -->
<script>
export default {
  props: ['value'],
  methods: {
    update() {
      this.value = 'new value' // Don't do this!
    }
  }
}
</script>

<!-- ✅ CORRECT: Emit event instead -->
<script>
export default {
  props: ['value'],
  methods: {
    update() {
      this.$emit('input', 'new value')
    }
  }
}
</script>
```

### Pitfall 2: Forgetting $event in Inline Handlers

```vue
<template>
  <!-- ❌ WRONG: Can't access event payload -->
  <child-component @event="doSomething(payload)" />

  <!-- ✅ CORRECT: Use $event for payload -->
  <child-component @event="doSomething($event)" />

  <!-- Or use a method -->
  <child-component @event="doSomething" />
</template>
```

## Reference

- [Vue 2 Guide - Custom Events](https://vue2.docs.vuejs.org/v2/guide/components-custom-events.html)
- [Vue 2 API - vm.$emit](https://vue2.docs.vuejs.org/v2/api/#vm-emit)
