---
title: Watch Options - deep and immediate
impact: MEDIUM
impactDescription: Watch options provide powerful data observation with deep watching and immediate callback execution
type: best-practice
tags:
  - vue2
  - watch
  - deep
  - immediate
  - data-observation
---

# Watch Options - deep and immediate

Impact: MEDIUM - The `watch` option allows observing data changes with options for deep object/array watching and immediate callback execution on component creation.

## Task Checklist

- [ ] Use `deep: true` to watch nested object/array changes
- [ ] Use `immediate: true` to trigger callback immediately on creation
- [ ] Clean up watchers using `$watch()` return value
- [ ] Avoid arrow functions in watchers (wrong `this` binding)

## Basic Watch

```javascript
export default {
  data() {
    return {
      question: '',
      answer: 'Questions usually contain a question mark. ;-)'
    }
  },
  watch: {
    // Simple watch - triggered when question changes
    question: function(newValue, oldValue) {
      if (newValue.indexOf('?') > -1) {
        this.answer = 'Answers usually contain a question mark.'
      } else {
        this.answer = ''
      }
    }
  }
}
```

## Deep Watch for Objects

```javascript
export default {
  data() {
    return {
      user: {
        profile: {
          name: '',
          address: {
            city: ''
          }
        }
      }
    }
  },
  watch: {
    // WRONG: Won't detect nested changes
    'user.profile.name': function(newValue, oldValue) {
      console.log('Name changed')
    },
    // CORRECT: Use deep for nested watching
    user: {
      handler: function(newValue, oldValue) {
        console.log('User object changed')
        // But note: newValue === oldValue for object mutations!
      },
      deep: true
    }
  }
}
```

## Important: Object Mutation Limitation

```javascript
export default {
  data() {
    return {
      user: {
        name: 'John'
      }
    }
  },
  watch: {
    user: {
      handler: function(newValue, oldValue) {
        // When mutating objects, newValue === oldValue!
        console.log('New:', newValue, 'Old:', oldValue) // Same reference
      },
      deep: true
    }
  },
  methods: {
    changeName() {
      // This triggers the watcher
      this.user = { ...this.user, name: 'Jane' }

      // This also triggers watcher (due to deep: true)
      this.user.name = 'Bob'
      // But newValue === oldValue in handler!
    }
  }
}
```

## Deep Watch for Arrays

```javascript
export default {
  data() {
    return {
      items: [
        { id: 1, tags: ['vue', 'javascript'] }
      ]
    }
  },
  watch: {
    items: {
      handler: function(newValue, oldValue) {
        console.log('Items array changed')
        // Array mutations trigger this
      },
      deep: true
    }
  }
}
```

## Immediate Watch

```javascript
export default {
  data() {
    return {
      query: '',
      results: []
    }
  },
  watch: {
    query: {
      handler: function(newValue) {
        // Fetch results when query changes
        this.fetchResults(newValue)
      },
      // Trigger immediately on component creation
      immediate: true
    }
  },
  methods: {
    fetchResults(query) {
      // Fetch logic
    }
  }
}
```

## Both deep and immediate

```javascript
export default {
  data() {
    return {
      config: {
        api: {
          url: '',
          timeout: 5000
        }
      }
    }
  },
  watch: {
    config: {
      handler: function(newConfig) {
        // Save to localStorage whenever config changes
        localStorage.setItem('config', JSON.stringify(newConfig))
      },
      deep: true,
      immediate: true
    }
  }
}
```

## Watching with String Method Name

```javascript
export default {
  data() {
    return {
      value: 10
    }
  },
  methods: {
    handleValueChange(newValue, oldValue) {
      console.log('Value changed from', oldValue, 'to', newValue)
    }
  },
  watch: {
    // Reference method by string name
    value: 'handleValueChange'
  }
}
```

## Array of Watchers

```javascript
export default {
  data() {
    return {
      e: { f: { g: 5 } }
    }
  },
  watch: {
    // You can pass an array of callbacks
    e: [
      'handleFirst',
      function handleSecond(val, oldVal) {
        console.log('Second handler', val)
      },
      {
        handler: 'handleThird',
        deep: true
      }
    ]
  },
  methods: {
    handleFirst(val, oldVal) {
      console.log('First handler', val)
    },
    handleSecond(val, oldVal) {
      console.log('Second handler', val)
    },
    handleThird(val, oldVal) {
      console.log('Third handler', val)
    }
  }
}
```

## Watching Nested Properties

```javascript
export default {
  data() {
    return {
      e: {
        f: {
          g: 5
        }
      }
    }
  },
  watch: {
    // Watch specific nested property
    'e.f.g': function(val, oldVal) {
      console.log('e.f.g changed from', oldVal, 'to', val)
    }
  }
}
```

## Programmatic Watching with $watch

```javascript
export default {
  data() {
    return {
      someObject: {}
    }
  },
  created() {
    // Unwatch function returned by $watch
      this.unwatch = this.$watch('someObject', (newValue, oldValue) => {
        console.log('Changed!', newValue, oldValue)
      }, {
        deep: true
      })
  },
  beforeDestroy() {
    // Clean up watcher
    if (this.unwatch) {
      this.unwatch()
    }
  }
}
```

## Cannot Watch Properties on Component Instance

```javascript
export default {
  data() {
    return {
      firstName: 'John',
      lastName: 'Doe'
    }
  },
  computed: {
    fullName() {
      return this.firstName + ' ' + this.lastName
    }
  },
  watch: {
    // WRONG: Can't watch computed properties directly
    fullName: function(newValue) {
      // This won't work as expected
    },
    // CORRECT: Watch the underlying data
    firstName: function(newValue) {
      console.log('First name changed')
    },
    lastName: function(newValue) {
      console.log('Last name changed')
    }
  }
}
```

## Watch vs Methods

```javascript
export default {
  data() {
    return {
      count: 0
    }
  },
  methods: {
    increment() {
      this.count++
      // Doesn't automatically watch count
    }
  },
  watch: {
    count: function(newValue, oldValue) {
      // Automatically triggered whenever count changes
      console.log('Count changed from', oldValue, 'to', newValue)
    }
  }
}
```

## Reference

- [Vue 2 API - watch](https://v2.vuejs.org/v2/api/#watch)
- [Vue 2 Guide - Computed Properties vs Watchers](https://v2.vuejs.org/v2/guide/computed.html#Computed-vs-Watchers)
