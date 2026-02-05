---
title: Stateful Methods Shared Across Component Instances
impact: MEDIUM
impactDescription: Methods defined on component prototype are shared across all instances
type: gotcha
tags:
  - vue2.6.14
  - component-patterns
  - methods
  - stateful
  - prototype
---

# Stateful Methods Shared Across Component Instances

Impact: MEDIUM - Methods are shared across all component instances on the prototype. State stored on methods or closures will be shared unexpectedly.

## Task Checklist

- [ ] Store component state in `data()` not on methods
- [ ] Use `data()` function (not object) for unique instance state
- [ ] Avoid closure variables in method definitions
- [ ] Use lifecycle hooks for instance-specific initialization

## Problem: Shared Method State

```javascript
// ❌ WRONG: Counter state is shared across all instances
export default {
  name: 'Counter',
  methods: {
    // This count is shared across ALL component instances!
    count: 0,
    increment() {
      this.count++
      console.log(this.count) // All instances share same count!
    }
  }
}
```

## Solution: Use Data Function

```javascript
// ✅ CORRECT: Each instance has its own state
export default {
  name: 'Counter',
  data() {
    return {
      count: 0 // Unique for each instance
    }
  },
  methods: {
    increment() {
      this.count++
    }
  }
}
```

## Data Must Be a Function

```javascript
// ❌ WRONG: Data as object - shared across all instances
export default {
  data: {
    count: 0
  }
}

// ✅ CORRECT: Data as function - unique per instance
export default {
  data() {
    return {
      count: 0
    }
  }
}
```

## Closure Variables in Methods

```javascript
// ❌ WRONG: Closure variables are shared
export default {
  methods: {
    // This closure variable is shared across all instances
    cache: new Map(),
    fetchData(key) {
      if (this.cache.has(key)) {
        return this.cache.get(key)
      }
      const data = fetchFromAPI(key)
      this.cache.set(key, data)
      return data
    }
  }
}

// ✅ CORRECT: Put cache in data
export default {
  data() {
    return {
      cache: new Map()
    }
  },
  methods: {
    fetchData(key) {
      if (this.cache.has(key)) {
        return this.cache.get(key)
      }
      const data = fetchFromAPI(key)
      this.cache.set(key, data)
      return data
    }
  }
}
```

## Method Properties vs Data Properties

```javascript
export default {
  name: 'MyComponent',
  data() {
    return {
      // ✅ CORRECT: Reactive state
      localState: 'value1'
    }
  },
  methods: {
    // ❌ WRONG: Method properties are shared
    sharedState: 'value2',

    // ✅ CORRECT: Functions work correctly
    getState() {
      return this.localState
    }
  }
}
```

## Instance-Specific Initialization

```javascript
// ✅ CORRECT: Use lifecycle hooks for instance-specific setup
export default {
  data() {
    return {
      timer: null,
      items: []
    }
  },
  mounted() {
    // Each instance gets its own timer
    this.timer = setInterval(() => {
      this.fetchData()
    }, 5000)
  },
  beforeDestroy() {
    // Clean up instance-specific resources
    if (this.timer) {
      clearInterval(this.timer)
    }
  }
}
```

## Singleton Pattern Gotcha

```javascript
// ❌ WRONG: Singleton module imported in component
// apiClient.js
export const apiClient = {
  cache: new Map(),
  get(key) {
    return this.cache.get(key)
  }
}

// component.vue
import { apiClient } from './apiClient'

export default {
  methods: {
    fetchData() {
      // Cache is shared across ALL components using apiClient
      return apiClient.get(this.url)
    }
  }
}

// ✅ CORRECT: Create instance in component or data
export default {
  data() {
    return {
      apiClient: createAPIClient()
    }
  }
}
```

## Computed Properties Are Instance-Specific

```javascript
// ✅ CORRECT: Computed properties are instance-specific
export default {
  props: {
    items: Array
  },
  computed: {
    // Each instance has its own computed value
    filteredItems() {
      return this.items.filter(item => item.active)
    },
    itemCount() {
      return this.items.length
    }
  }
}
```

## Event Handlers and State

```javascript
// ❌ WRONG: Handler state stored in closure
export default {
  methods: {
    handleClicks: {
      count: 0,
      handler(event) {
        this.count++
        console.log(`Clicked ${this.count} times`)
      }
    }
  }
}

// ✅ CORRECT: Store in data
export default {
  data() {
    return {
      clickCount: 0
    }
  },
  methods: {
    handleClick(event) {
      this.clickCount++
      console.log(`Clicked ${this.clickCount} times`)
    }
  }
}
```

## Managing Resources Per Instance

```javascript
export default {
  data() {
    return {
      // Instance-specific resources
      websocket: null,
      eventHandlers: new Map()
    }
  },
  mounted() {
    // Create instance-specific websocket connection
    this.websocket = new WebSocket(this.url)
    this.websocket.onmessage = (event) => {
      this.handleMessage(event)
    }
  },
  beforeDestroy() {
    // Clean up instance-specific resources
    if (this.websocket) {
      this.websocket.close()
      this.websocket = null
    }
    this.eventHandlers.clear()
  }
}
```

## Static Methods on Components

```javascript
// If you need truly shared utility functions, use static methods
export default {
  name: 'UtilityComponent',
  methods: {
    // Instance method
    instanceMethod() {
      return UtilityComponent.sharedMethod(this.value)
    }
  }
}

// Static/shared method
UtilityComponent.sharedMethod = function(value) {
  return value * 2
}

// Or use a separate utility module
// utils.js
export function sharedMethod(value) {
  return value * 2
}
```

## Mixins and Shared State

```javascript
// ❌ WRONG: Mixin with shared state
const myMixin = {
  data: {
    // Data as object - shared!
    shared: 'value'
  }
}

// ✅ CORRECT: Mixin with data function
const myMixin = {
  data() {
    return {
      shared: 'value' // Unique per instance
    }
  }
}
```

## Summary Table

| Pattern | Shared? | Correct Usage |
|---------|---------|---------------|
| `data: {}` object | Yes (wrong) | Never use |
| `data()` function | No (correct) | Always use |
| `methods: {}` functions | Shared functions | OK for functions |
| Method properties | Shared (wrong) | Put in data |
| Closure variables | Shared (wrong) | Put in data |
| `computed` | No (correct) | Safe to use |

## Reference

- [Vue 2 Guide - Data Details](https://v2.vuejs.org/v2/guide/instance.html#Data-and-Methods)
- [Vue 2 Guide - Mixins](https://v2.vuejs.org/v2/guide/mixins.html)
