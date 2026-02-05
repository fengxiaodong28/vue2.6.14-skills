---
title: Composition API Setup in Vue 2.6.14
impact: HIGH
impactDescription: @vue/composition-api@1.7.x enables Composition API in Vue 2.6.14
type: best-practice
tags:
  - vue2.6.14
  - composition-api
  - vue-composition-api
  - setup-function
  - reactive-state
---

# Composition API Setup in Vue 2.6.14

Impact: HIGH - @vue/composition-api@1.7.x plugin enables Vue 3 Composition API in Vue 2.6.14 applications.

## Task Checklist

- [ ] Install @vue/composition-api@1.7.x for Vue 2
- [ ] Register plugin before creating root instance
- [ ] Use `setup()` function for component logic
- [ ] Import reactive functions from @vue/composition-api
- [ ] Note differences from Vue 3 Composition API

## Installation

```bash
# For Vue 2.6.14
npm install @vue/composition-api@^1.7.0

# ❌ WRONG: @vueuse/core is a different library
# ❌ WRONG: Vue 3 has Composition API built-in
```

## Plugin Registration

```javascript
// main.js
import Vue from 'vue'
import App from './App.vue'
import VueCompositionAPI from '@vue/composition-api'

// Register before creating root instance
Vue.use(VueCompositionAPI)

new Vue({
  render: h => h(App)
}).$mount('#app')
```

## Basic Setup Function

```vue
<template>
  <div>
    <p>Count: {{ count }}</p>
    <p>Double: {{ double }}</p>
    <button @click="increment">Increment</button>
  </div>
</template>

<script>
import { ref, computed } from '@vue/composition-api'

export default {
  setup() {
    // Reactive state
    const count = ref(0)

    // Computed properties
    const double = computed(() => count.value * 2)

    // Methods
    const increment = () => {
      count.value++
    }

    // Return everything template needs
    return {
      count,
      double,
      increment
    }
  }
}
</script>
```

## Reactive State

```vue
<script>
import { ref, reactive, readonly } from '@vue/composition-api'

export default {
  setup() {
    // ref for primitive values
    const count = ref(0)
    const message = ref('Hello')

    // reactive for objects
    const user = reactive({
      name: 'John',
      age: 30,
      email: 'john@example.com'
    })

    // readonly objects
    const config = readonly({
      apiUrl: 'https://api.example.com',
      timeout: 5000
    })

    // Access ref values with .value
    const increment = () => {
      count.value++ // Need .value in JS
    }

    // Reactive properties don't need .value
    const updateName = (name) => {
      user.name = name // Direct access
    }

    return {
      count,
      message,
      user,
      config,
      increment,
      updateName
    }
  }
}
</script>
```

## Computed Properties

```vue
<script>
import { ref, computed } from '@vue/composition-api'

export default {
  setup() {
    const firstName = ref('John')
    const lastName = ref('Doe')

    // Read-only computed
    const fullName = computed(() => {
      return `${firstName.value} ${lastName.value}`
    })

    // Writable computed
    const fullNameWritable = computed({
      get: () => `${firstName.value} ${lastName.value}`,
      set: (value) => {
        [firstName.value, lastName.value] = value.split(' ')
      }
    })

    return {
      firstName,
      lastName,
      fullName,
      fullNameWritable
    }
  }
}
</script>
```

## Watch and WatchEffect

```vue
<script>
import { ref, watch, watchEffect } from '@vue/composition-api'

export default {
  setup() {
    const count = ref(0)
    const message = ref('Hello')

    // watchEffect - auto tracks dependencies
    watchEffect(() => {
      console.log(`Count is: ${count.value}`)
      console.log(`Message is: ${message.value}`)
    })

    // watch - specific source
    watch(count, (newValue, oldValue) => {
      console.log(`Count changed from ${oldValue} to ${newValue}`)
    })

    // Watch multiple sources
    watch(
      [count, message],
      ([newCount, newMessage], [oldCount, oldMessage]) => {
        console.log('Values changed:', newCount, newMessage)
      }
    )

    // Watch with options
    watch(
      count,
      (value) => {
        console.log('Count:', value)
      },
      {
        immediate: true,
        deep: true
      }
    )

    return {
      count,
      message
    }
  }
}
</script>
```

## Props and Emits

```vue
<script>
import { watch } from '@vue/composition-api'

export default {
  props: {
    title: String,
    count: {
      type: Number,
      default: 0
    }
  },
  setup(props, { emit }) {
    // Access props (reactive)
    console.log(props.title)
    console.log(props.count)

    // Watch props
    watch(
      () => props.count,
      (newVal, oldVal) => {
        console.log(`Count changed: ${oldVal} -> ${newVal}`)
      }
    )

    // Emit events
    const increment = () => {
      emit('update:count', props.count + 1)
      emit('change', props.count + 1)
    }

    return {
      increment
    }
  }
}
</script>
```

## Context Arguments

```vue
<script>
import { ref, onMounted, onUnmounted } from '@vue/composition-api'

export default {
  setup(props, context) {
    // Destructure context for cleaner syntax
    const { attrs, slots, emit, expose } = context

    const count = ref(0)

    // Access attrs
    console.log(attrs.id)
    console.log(attrs.class)

    // Access slots
    const hasDefaultSlot = () => !!slots.default

    // Emit events
    const handleClick = () => {
      emit('click', count.value)
    }

    // Expose public properties (for template refs)
    expose({
      count,
      reset: () => { count.value = 0 }
    })

    // Lifecycle hooks
    onMounted(() => {
      console.log('Mounted')
    })

    onUnmounted(() => {
      console.log('Unmounted')
    })

    return {
      count,
      handleClick,
      hasDefaultSlot
    }
  }
}
</script>
```

## Lifecycle Hooks

```vue
<script>
import {
  onMounted,
  onBeforeMount,
  onUpdated,
  onBeforeUpdate,
  onBeforeUnmount,
  onUnmounted,
  onActivated,
  onDeactivated,
  onErrorCaptured
} from '@vue/composition-api'

export default {
  setup() {
    // Mount
    onBeforeMount(() => {
      console.log('Before mount')
    })

    onMounted(() => {
      console.log('Mounted')
    })

    // Update
    onBeforeUpdate(() => {
      console.log('Before update')
    })

    onUpdated(() => {
      console.log('Updated')
    })

    // Unmount (Vue 2 naming: beforeDestroy/destroyed)
    onBeforeUnmount(() => {
      console.log('Before unmount') // beforeDestroy
    })

    onUnmounted(() => {
      console.log('Unmounted') // destroyed
    })

    // Keep-alive
    onActivated(() => {
      console.log('Activated')
    })

    onDeactivated(() => {
      console.log('Deactivated')
    })

    // Error capture
    onErrorCaptured((err, vm, info) => {
      console.error('Error:', err)
      return false // Prevent error propagation
    })

    return {}
  }
}
</script>
```

## Provide/Inject

```vue
<!-- Parent.vue -->
<script>
import { provide, ref, readonly } from '@vue/composition-api'

export default {
  setup() {
    const theme = ref('light')

    // Provide value
    provide('theme', readonly(theme))

    // Provide with ref for reactivity
    const updateTheme = (newTheme) => {
      theme.value = newTheme
    }

    provide('updateTheme', updateTheme)

    return {
      theme
    }
  }
}
</script>
```

```vue
<!-- Child.vue -->
<script>
import { inject } from '@vue/composition-api'

export default {
  setup() {
    // Inject value
    const theme = inject('theme', 'light') // Default value

    // Inject function
    const updateTheme = inject('updateTheme')

    const toggleTheme = () => {
      updateTheme(theme.value === 'light' ? 'dark' : 'light')
    }

    return {
      theme,
      toggleTheme
    }
  }
}
</script>
```

## TemplateRefs

```vue
<template>
  <div>
    <input ref="inputRef" />
    <button @click="focusInput">Focus</button>
  </div>
</template>

<script>
import { ref, onMounted } from '@vue/composition-api'

export default {
  setup() {
    const inputRef = ref(null)

    onMounted(() => {
      inputRef.value.focus()
    })

    const focusInput = () => {
      inputRef.value.focus()
    }

    return {
      inputRef,
      focusInput
    }
  }
}
</script>
```

## Using with Vuex

```vue
<script>
import { computed, createComponent } from '@vue/composition-api'
import { useStore } from './composables/useStore'

export default {
  setup() {
    const store = useStore()

    const count = computed(() => store.state.count)
    const double = computed(() => store.getters.double)

    const increment = () => {
      store.commit('increment')
    }

    const asyncIncrement = async () => {
      await store.dispatch('asyncIncrement')
    }

    return {
      count,
      double,
      increment,
      asyncIncrement
    }
  }
}
</script>
```

## Using with Vue Router

```vue
<script>
import { computed } from '@vue/composition-api'
import { useRoute, useRouter } from './composables/useRouter'

export default {
  setup() {
    const route = useRoute()
    const router = useRouter()

    const userId = computed(() => route.params.id)
    const query = computed(() => route.query)

    const navigate = (path) => {
      router.push(path)
    }

    return {
      userId,
      query,
      navigate
    }
  }
}
</script>
```

## Composables (Reusable Logic)

```javascript
// composables/useCounter.js
import { ref, computed } from '@vue/composition-api'

export function useCounter(initialValue = 0) {
  const count = ref(initialValue)

  const double = computed(() => count.value * 2)

  const increment = () => { count.value++ }
  const decrement = () => { count.value-- }
  const reset = () => { count.value = initialValue }

  return {
    count,
    double,
    increment,
    decrement,
    reset
  }
}
```

```vue
<!-- Usage in component -->
<script>
import { useCounter } from '@/composables/useCounter'

export default {
  setup() {
    const { count, double, increment, decrement, reset } = useCounter(10)

    return {
      count,
      double,
      increment,
      decrement,
      reset
    }
  }
}
</script>
```

## Limitations vs Vue 3

| Feature | @vue/composition-api | Vue 3 |
|---------|---------------------|-------|
| `setup()` script syntax | ❌ Not supported | ✅ `<script setup>` |
| Top-level await | ❌ Not supported | ✅ Supported |
| `defineProps()`/`defineEmits()` | ❌ Use setup params | ✅ Available |
| CSS Modules | Partial support | Full support |
| Teleport/Suspense | ❌ Not available | ✅ Built-in |

## TypeScript Support

```vue
<script lang="ts">
import { ref, computed, Ref } from '@vue/composition-api'

interface User {
  id: number
  name: string
  email: string
}

export default {
  setup() {
    const user: Ref<User | null> = ref(null)

    const userName = computed<string>(() => {
      return user.value?.name || 'Guest'
    })

    const fetchUser = async (id: number): Promise<void> => {
      const response = await fetch(`/api/users/${id}`)
      user.value = await response.json()
    }

    return {
      user,
      userName,
      fetchUser
    }
  }
}
</script>
```

## Reference

- [@vue/composition-api Documentation](https://github.com/vuejs/composition-api)
- [Vue 2 Composition API RFC](https://vue-composition-api-rfc.netlify.app/)
