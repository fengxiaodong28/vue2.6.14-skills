---
title: Event Bus Pattern for Sibling Component Communication
impact: LOW
impactDescription: Event bus enables communication between non-parent-child components
type: pattern
tags:
  - vue2.6.14
  - event-bus
  - component-communication
  - sibling-components
  - cross-component
---

# Event Bus Pattern for Sibling Component Communication

Impact: LOW - Event bus enables communication between components that don't have a direct parent-child relationship. Consider Vuex for complex state management instead.

## Task Checklist

- [ ] Create event bus as separate Vue instance
- [ ] Emit events with `$emit` on the bus
- [ ] Listen to events with `$on`
- [ ] Clean up listeners with `$off` in `beforeDestroy`

## Creating Event Bus

```javascript
// src/event-bus.js
import Vue from 'vue'

export const EventBus = new Vue()
```

## Emitting Events

```vue
<!-- SiblingA.vue -->
<template>
  <button @click="sendMessage">Send Message</button>
</template>

<script>
import { EventBus } from '@/event-bus'

export default {
  methods: {
    sendMessage() {
      // Emit event with optional payload
      EventBus.$emit('message-sent', {
        text: 'Hello from SiblingA',
        timestamp: Date.now()
      })
    }
  }
}
</script>
```

## Listening to Events

```vue
<!-- SiblingB.vue -->
<template>
  <div>
    <p>Last message: {{ lastMessage }}</p>
  </div>
</template>

<script>
import { EventBus } from '@/event-bus'

export default {
  data() {
    return {
      lastMessage: null
    }
  },
  created() {
    // Listen to events
    EventBus.$on('message-sent', this.handleMessage)
  },
  beforeDestroy() {
    // IMPORTANT: Clean up event listeners
    EventBus.$off('message-sent', this.handleMessage)
  },
  methods: {
    handleMessage(payload) {
      this.lastMessage = payload.text
      console.log('Received:', payload)
    }
  }
}
</script>
```

## Multiple Event Types

```javascript
// Event bus can handle multiple event types
EventBus.$emit('user-updated', user)
EventBus.$emit('notification', { type: 'success', message: 'Saved!' })
EventBus.$emit('data-refresh')

// Listen to multiple events
EventBus.$on('user-updated', this.onUserUpdated)
EventBus.$on('notification', this.onNotification)
EventBus.$on('data-refresh', this.refreshData)

// Clean up
beforeDestroy() {
  EventBus.$off('user-updated', this.onUserUpdated)
  EventBus.$off('notification', this.onNotification)
  EventBus.$off('data-refresh', this.refreshData)
}
```

## One-Time Event Listener

```javascript
// Listen to event only once
EventBus.$once('initial-data', this.handleInitialData)
```

## Global Event Bus (Not Recommended)

```javascript
// Attach to Vue prototype (not recommended)
// main.js
Vue.prototype.$bus = new Vue()

// Use anywhere
this.$bus.$emit('event', data)
this.$bus.$on('event', handler)
this.$bus.$off('event', handler)
```

## Cleaning Up (Important!)

```vue
<script>
export default {
  created() {
    // Store reference to handler for cleanup
    this.messageHandler = this.handleMessage.bind(this)

    EventBus.$on('message', this.messageHandler)
  },
  beforeDestroy() {
    // Always clean up to prevent memory leaks
    EventBus.$off('message', this.messageHandler)
  },
  methods: {
    handleMessage(payload) {
      // Handle message
    }
  }
}
</script>
```

## When to Use Event Bus vs Alternatives

| Pattern | Use Case |
|---------|----------|
| **Event Bus** | Simple cross-component communication, prototype projects |
| **Props Down / Events Up** | Parent-child communication (preferred) |
| **Provide/Inject** | Ancestor to descendant communication |
| **Vuex** | Complex state management, shared application state |
| **Vue Router** | Cross-route communication via route params/query |

## Reference

- [Vue 2 Event Bus Pattern](https://vue2.docs.vuejs.org/v2/guide/migration.html#Event-Bus)
