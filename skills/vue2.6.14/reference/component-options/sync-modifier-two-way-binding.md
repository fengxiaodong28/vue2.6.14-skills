---
title: Using .sync Modifier for Two-Way Binding (Vue 2.3+)
impact: MEDIUM
impactDescription: .sync modifier provides clean two-way binding pattern removed in Vue 3
type: best-practice
tags:
  - vue2.6.14
  - two-way-binding
  - sync-modifier
  - component-communication
  - deprecated-in-vue3
---

# Using .sync Modifier for Two-Way Binding (Vue 2.3+)

Impact: MEDIUM - The `.sync` modifier provides a clean two-way binding pattern for Vue 2. Note: This feature is removed in Vue 3, replaced by `v-model` with arguments.

## Task Checklist

- [ ] Use `.sync` modifier for two-way binding in Vue 2
- [ ] Emit `update:propName` events from child component
- [ ] Be aware `.sync` is deprecated in Vue 3 (use `v-model` instead)

## Basic Usage

```vue
<!-- Parent.vue -->
<template>
  <div>
    <p>Parent count: {{ count }}</p>
    <!-- .sync is sugar for :value + @update:value -->
    <ChildComponent :value.sync="count" />
  </div>
</template>

<script>
import ChildComponent from './ChildComponent.vue'

export default {
  components: { ChildComponent },
  data() {
    return {
      count: 0
    }
  }
}
</script>
```

```vue
<!-- ChildComponent.vue -->
<template>
  <div>
    <button @click="increment">{{ value }}</button>
  </div>
</template>

<script>
export default {
  props: {
    value: {
      type: Number,
      required: true
    }
  },
  methods: {
    increment() {
      // Emit update:value to work with .sync
      this.$emit('update:value', this.value + 1)
    }
  }
}
</script>
```

## Equivalent Without .sync

```vue
<!-- Parent.vue - WITHOUT .sync -->
<template>
  <div>
    <!-- This is what .sync compiles to -->
    <ChildComponent
      :value="count"
      @update:value="count = $event"
    />
  </div>
</template>
```

## Multiple .sync Modifiers

```vue
<!-- Parent.vue -->
<template>
  <div>
    <UserForm
      :name.sync="userName"
      :email.sync="userEmail"
      :age.sync="userAge"
    />
  </div>
</template>

<script>
export default {
  data() {
    return {
      userName: '',
      userEmail: '',
      userAge: 0
    }
  }
}
</script>
```

```vue
<!-- UserForm.vue -->
<template>
  <form>
    <input :value="name" @input="updateName($event.target.value)" />
    <input :value="email" @input="updateEmail($event.target.value)" />
    <input :value="age" @input="updateAge($event.target.value)" />
  </form>
</template>

<script>
export default {
  props: ['name', 'email', 'age'],
  methods: {
    updateName(value) {
      this.$emit('update:name', value)
    },
    updateEmail(value) {
      this.$emit('update:email', value)
    },
    updateAge(value) {
      this.$emit('update:age', value)
    }
  }
}
</script>
```

## Custom Prop Name

```vue
<!-- Parent.vue -->
<template>
  <ChildComponent :custom-value.sync="localValue" />
</template>

<!-- ChildComponent.vue -->
<script>
export default {
  props: {
    customValue: Number
  },
  methods: {
    updateValue() {
      this.$emit('update:customValue', newValue)
    }
  }
}
</script>
```

## .sync with v-bind Object Syntax

```vue
<!-- Parent.vue -->
<template>
  <div>
    <!-- Sync multiple properties at once -->
    <UserForm v-bind.sync="user" />
  </div>
</template>

<script>
export default {
  data() {
    return {
      user: {
        name: 'John',
        email: 'john@example.com',
        age: 30
      }
    }
  }
}
</script>
```

## Vue 3 Alternative

**Note**: `.sync` is removed in Vue 3. Use `v-model` instead:

```vue
<!-- Vue 3 syntax -->
<ChildComponent v-model:value="count" />

<!-- Child component emits update:modelValue -->
this.$emit('update:modelValue', newValue)
```

## Reference

- [Vue 2 .sync Modifier](https://vue2.docs.vuejs.org/v2/guide/components-custom-events.html#sync-Modifier)
- [Vue 3 Migration Guide - v-model](https://v3-migration.vuejs.org/breaking-changes/v-model.html)
