---
title: Type-Safe Event Handlers in Vue 2 Options API
impact: LOW
impactDescription: Event handlers in Vue 2 need proper type annotations for parameters and event objects
type: best-practice
tags:
  - vue2
  - typescript
  - events
  - event-handlers
  - dom-events
---

# Type-Safe Event Handlers in Vue 2 Options API

Impact: LOW - Event handlers in Vue 2 Options API require proper typing for event parameters to achieve full type safety and autocomplete.

## Task Checklist

- [ ] Type event target parameter as `Event` or specific element type
- [ ] Type custom event parameters
- [ ] Use `$event` for inline event handlers
- [ ] Avoid arrow functions that lose `this` context

## Basic Event Handler Types

```vue
<template>
  <div>
    <button @click="handleClick">Click me</button>
    <input @input="handleInput" />
    <form @submit="handleSubmit">
      <button type="submit">Submit</button>
    </form>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'EventHandlers',
  methods: {
    // No parameters
    handleClick(): void {
      console.log('Button clicked')
    },

    // Event parameter (native event)
    handleInput(event: Event): void {
      const target = event.target as HTMLInputElement
      console.log('Input value:', target.value)
    },

    // Form submit event
    handleSubmit(event: Event): void {
      event.preventDefault()
      // Form submission logic
    }
  }
})
</script>
```

## Keyboard Event Types

```vue
<template>
  <div>
    <input @keyup.enter="handleEnter" />
    <input @keydown.esc="handleEscape" />
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  methods: {
    handleEnter(event: KeyboardEvent): void {
      const target = event.target as HTMLInputElement
      console.log('Enter key pressed, value:', target.value)
    },

    handleEscape(event: KeyboardEvent): void {
      console.log('Escape key pressed')
    }
  }
})
</script>
```

## Mouse Event Types

```vue
<template>
  <div>
    <button @click="handleClick">Click</button>
    <div @mousemove="handleMouseMove">Move here</div>
    <button @mousedown="handleMouseDown">Down</button>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  methods: {
    handleClick(event: MouseEvent): void {
      console.log('Click at:', event.clientX, event.clientY)
    },

    handleMouseMove(event: MouseEvent): void {
      console.log('Mouse at:', event.clientX, event.clientY)
    },

    handleMouseDown(event: MouseEvent): void {
      console.log('Mouse button:', event.button)
    }
  }
})
</script>
```

## Custom Event Emissions

```vue
<script lang="ts">
import Vue from 'vue'

interface Payload {
  id: number
  action: string
  timestamp: number
}

export default Vue.extend({
  methods: {
    // Emit with typed payload
    emitEvent(): void {
      const payload: Payload = {
        id: 123,
        action: 'update',
        timestamp: Date.now()
      }
      this.$emit('custom-event', payload)
    },

    // Event listener with typed payload
    handleCustomEvent(payload: Payload): void {
      console.log('Received:', payload.action)
      console.log('ID:', payload.id)
    }
  }
})
</script>
```

## Component Custom Events

```vue
<script lang="ts">
import Vue from 'vue'
import { PropType } from 'vue'

interface User {
  id: number
  name: string
}

export default Vue.extend({
  props: {
    user: {
      type: Object as PropType<User>,
      required: true
    }
  },
  methods: {
    // Emit custom event with proper type
    editUser(): void {
      this.$emit('edit', {
        user: this.user,
        timestamp: Date.now()
      })
    },

    // Handler with typed parameter
    onEdit(payload: { user: User; timestamp: number }): void {
      console.log('Editing:', payload.user.name)
    }
  }
})
</script>
```

## Inline Event Handlers

```vue
<template>
  <div>
    <!-- Inline handler with $event -->
    <button @click="handleInline($event)">Inline</button>

    <!-- Inline handler with value -->
    <input @input="value = $event.target.value" />
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  methods: {
    handleInline(event: MouseEvent): void {
      console.log('Clicked at:', event.clientX, event.clientY)
    }
  }
})
</script>
```

## Native Event Modifier

```vue
<template>
  <div>
    <!-- Native event listener on component -->
    <button @click.native="handleNativeClick">Click</button>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  methods: {
    handleNativeClick(event: MouseEvent): void {
      console.log('Native click event')
    }
  }
})
</script>
```

## Event Handler with Component $emit

```vue
<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  methods: {
    // Emit with type-safe payload
    saveData<T>(data: T): void {
      this.$emit('save', data)
    },

    // Receive with type
    handleSave<T>(data: T): void {
      console.log('Saved:', data)
    }
  }
})
</script>
```

## Function Type for Methods

```vue
<script lang="ts">
import Vue from 'vue'

type AsyncHandler = () => Promise<void>

export default Vue.extend({
  methods: {
    // Function type for method
    fetchData: async (): Promise<void> => {
      const response = await fetch('/api/data')
      const data = await response.json()
      this.data = data
    },

    // Type for callback function
    executeCallback(callback: (result: string) => void): void {
      const result = callback('operation result')
      console.log('Callback result:', result)
    }
  }
})
</script>
```

## Reference

- [Vue 2 Guide - Event Handling](https://v2.vuejs.org/v2/guide/events.html)
