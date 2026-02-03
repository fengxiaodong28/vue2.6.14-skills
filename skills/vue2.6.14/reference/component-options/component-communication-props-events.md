---
title: Props Down, Events Up Pattern in Vue 2
impact: HIGH
impactDescription: Proper component communication follows unidirectional data flow
type: best-practice
tags:
  - vue2.6.14
  - component-communication
  - props
  - events
  - data-flow
---

# Props Down, Events Up Pattern in Vue 2

Impact: HIGH - Vue 2 follows unidirectional data flow: props flow down from parent to child, events flow up from child to parent. Never mutate props directly.

## Task Checklist

- [ ] Pass data down using props
- [ ] Communicate up using `$emit` events
- [ ] Never mutate props directly in child components
- [ ] Use `v-model` for two-way binding syntax sugar
- [ ] Use `.sync` modifier for explicit two-way binding (Vue 2.3+)

## The Pattern

```
Parent Component
     │
     ├─ props (data down) ────→ Child Component
     │
     └─ events (actions up) ←─── Child Component (via $emit)
```

## Basic Props Down

```vue
<!-- Parent.vue -->
<template>
  <div>
    <!-- Pass data down via props -->
    <ChildComponent
      :title="postTitle"
      :author="postAuthor"
      :content="postContent"
    />
  </div>
</template>

<script>
export default {
  data() {
    return {
      postTitle: 'Vue 2 Communication',
      postAuthor: 'John Doe',
      postContent: 'Props down, events up...'
    }
  }
}
</script>
```

```vue
<!-- ChildComponent.vue -->
<template>
  <div class="post">
    <h2>{{ title }}</h2>
    <p class="author">By {{ author }}</p>
    <p>{{ content }}</p>
  </div>
</template>

<script>
export default {
  props: {
    title: String,
    author: String,
    content: String
  }
}
</script>
```

## Basic Events Up

```vue
<!-- Parent.vue -->
<template>
  <div>
    <!-- Listen to events from child -->
    <ChildComponent @like="handleLike" @share="handleShare" />
  </div>
</template>

<script>
export default {
  methods: {
    handleLike(count) {
      console.log('Post liked!', count)
      // Update parent state
      this.likeCount = count
    },
    handleShare(platform) {
      console.log('Shared to', platform)
    }
  }
}
</script>
```

```vue
<!-- ChildComponent.vue -->
<template>
  <div class="post">
    <button @click="incrementLikes">Like ({{ likes }})</button>
    <button @click="shareToTwitter">Share</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      likes: 0
    }
  },
  methods: {
    incrementLikes() {
      this.likes++
      // Emit event to parent
      this.$emit('like', this.likes)
    },
    shareToTwitter() {
      this.$emit('share', 'twitter')
    }
  }
}
</script>
```

## Never Mutate Props Directly

```vue
<!-- ❌ WRONG: Direct prop mutation -->
<template>
  <div>
    <input v-model="value" />
  </div>
</template>

<script>
export default {
  props: {
    value: String
  }
}
</script>
```

```vue
<!-- ✅ CORRECT: Use local data or computed -->
<template>
  <div>
    <input v-model="localValue" />
  </div>
</template>

<script>
export default {
  props: {
    value: String
  },
  data() {
    return {
      localValue: this.value
    }
  },
  watch: {
    value(newValue) {
      this.localValue = newValue
    }
  }
}
</script>
```

## Two-Way Binding with v-model

```vue
<!-- Parent.vue -->
<template>
  <div>
    <!-- v-model is syntax sugar for :value + @input -->
    <TextInput v-model="username" />
    <!-- Same as: -->
    <TextInput :value="username" @input="username = $event" />
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

```vue
<!-- TextInput.vue -->
<template>
  <input
    :value="value"
    @input="$emit('input', $event.target.value)"
  />
</template>

<script>
export default {
  props: {
    value: String
  }
}
</script>
```

## Custom v-model

```vue
<!-- Parent.vue -->
<template>
  <div>
    <CheckBox v-model="isChecked" />
  </div>
</template>

<script>
export default {
  data() {
    return {
      isChecked: false
    }
  }
}
</script>
```

```vue
<!-- CheckBox.vue -->
<template>
  <input
    type="checkbox"
    :checked="checked"
    @change="$emit('change', $event.target.checked)"
  />
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
```

## .sync Modifier (Vue 2.3+)

```vue
<!-- Parent.vue -->
<template>
  <div>
    <!-- .sync is syntax sugar for :prop + @update:prop -->
    <UserForm :name.sync="userName" />
    <!-- Same as: -->
    <UserForm
      :name="userName"
      @update:name="userName = $event"
    />
  </div>
</template>
```

```vue
<!-- UserForm.vue -->
<template>
  <input
    :value="name"
    @input="$emit('update:name', $event.target.value)"
  />
</template>

<script>
export default {
  props: {
    name: String
  }
}
</script>
```

## Multiple .sync Modifiers

```vue
<!-- Parent.vue -->
<template>
  <div>
    <UserForm
      :first-name.sync="user.firstName"
      :last-name.sync="user.lastName"
      :age.sync="user.age"
    />
  </div>
</template>

<script>
export default {
  data() {
    return {
      user: {
        firstName: '',
        lastName: '',
        age: 0
      }
    }
  }
}
</script>
```

## Validating Props

```vue
<script>
export default {
  props: {
    // Basic type check
    title: String,

    // Multiple types
    value: [String, Number],

    // Required with type
    userId: {
      type: Number,
      required: true
    },

    // Default value
    count: {
      type: Number,
      default: 0
    },

    // Object/array default (must use factory function)
    item: {
      type: Object,
      default: () => ({ id: 0, name: '' })
    },

    // Custom validator
    age: {
      type: Number,
      validator: value => value >= 0 && value <= 120
    }
  }
}
</script>
```

## Event Payloads

```vue
<!-- Child.vue -->
<script>
export default {
  methods: {
    // Emit with single value
    emitValue() {
      this.$emit('change', this.value)
    },

    // Emit with multiple values (as object)
    emitMultiple() {
      this.$emit('submit', {
        id: this.id,
        data: this.formData,
        timestamp: Date.now()
      })
    },

    // Emit with multiple arguments
    emitArguments() {
      this.$emit('action', this.type, this.payload)
    }
  }
}
</script>

<!-- Parent.vue -->
<template>
  <div>
    <!-- Receive object payload -->
    <Child @submit="handleSubmit" />

    <!-- Receive multiple arguments -->
    <Child @action="handleAction" />
  </div>
</template>

<script>
export default {
  methods: {
    handleSubmit(payload) {
      console.log(payload.id, payload.data, payload.timestamp)
    },
    handleAction(type, payload) {
      console.log(type, payload)
    }
  }
}
</script>
```

## Provide/Inject (Deep Nesting)

```vue
<!-- Root.vue -->
<script>
export default {
  provide() {
    return {
      theme: this.theme,  // Note: not reactive by default!
      updateTheme: this.updateTheme
    }
  },
  data() {
    return {
      theme: 'light'
    }
  },
  methods: {
    updateTheme(newTheme) {
      this.theme = newTheme
    }
  }
}
</script>
```

```vue
<!-- DeepChild.vue -->
<script>
export default {
  inject: ['theme', 'updateTheme'],
  methods: {
    changeTheme() {
      // Call injected method to update
      this.updateTheme('dark')
    }
  }
}
</script>
```

## Event Bus (Sibling Communication)

```javascript
// event-bus.js
import Vue from 'vue'
export const EventBus = new Vue()
```

```vue
<!-- SiblingA.vue -->
<script>
import { EventBus } from './event-bus'

export default {
  methods: {
    notify() {
      EventBus.$emit('message', 'Hello from SiblingA')
    }
  }
}
</script>
```

```vue
<!-- SiblingB.vue -->
<script>
import { EventBus } from './event-bus'

export default {
  mounted() {
    EventBus.$on('message', payload => {
      console.log('Received:', payload)
    })
  },
  beforeDestroy() {
    // Clean up event listener
    EventBus.$off('message')
  }
}
</script>
```

## Summary Table

| Direction | Method | Example |
|-----------|--------|---------|
| Down (Parent→Child) | Props | `:prop="value"` |
| Up (Child→Parent) | Events | `@event="handler"` |
| Two-way binding | v-model | `v-model="value"` |
| Explicit two-way | .sync | `:prop.sync="value"` |
| Deep nesting | provide/inject | `provide() { ... }` / `inject: ['key']` |
| Sibling | Event Bus | `EventBus.$emit/$on` |

## Reference

- [Vue 2 Guide - Props](https://v2.vuejs.org/v2/guide/components-props.html)
- [Vue 2 Guide - Custom Events](https://v2.vuejs.org/v2/guide/components-custom-events.html)
- [Vue 2 Guide - .sync Modifier](https://v2.vuejs.org/v2/guide/components-custom-events.html#sync-Modifier)
