---
title: Using $forceUpdate to Force Re-rendering in Vue 2
impact: MEDIUM
impactDescription: $forceUpdate forces a component instance to re-render, bypassing reactivity
type: capability
tags:
  - vue2.6.14
  - $forceUpdate
  - re-rendering
  - reactivity
  - edge-case
---

# Using $forceUpdate to Force Re-rendering in Vue 2

Impact: MEDIUM - Vue 2's `$forceUpdate` method forces a component instance to re-render, bypassing the normal reactivity system. This should rarely be needed and indicates a reactivity problem when used.

## Task Checklist

- [ ] Avoid using `$forceUpdate` when possible
- [ ] Fix reactivity issues at the source instead of forcing updates
- [ ] Use `$forceUpdate` only for edge cases with third-party objects
- [ ] Understand that `$forceUpdate` doesn't affect child components
- [ ] Prefer reactive alternatives (Vue.set, object replacement)

## When $forceUpdate is Used

```vue
<template>
  <div>
    <p>{{ nonReactiveData.value }}</p>
    <button @click="updateWithoutReactivity">
      Update (Won't Work)
    </button>
    <button @click="updateWithForce">
      Update (With Force)
    </button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // Object created outside Vue (not reactive)
      nonReactiveData: Object.freeze({ value: 'Initial' })
    }
  },
  methods: {
    updateWithoutReactivity() {
      // This change won't trigger re-render
      this.nonReactiveData.value = 'Updated'
      // Template shows old value
    },
    updateWithForce() {
      // Change the data
      this.nonReactiveData.value = 'Updated'
      // Force re-render
      this.$forceUpdate()
    }
  }
}
</script>
```

## Better Alternatives to $forceUpdate

### Alternative 1: Use Vue.set/$set

```vue
<script>
export default {
  data() {
    return {
      user: {
        name: 'John'
        // Adding 'email' later won't be reactive
      }
    }
  },
  methods: {
    // ❌ WRONG: Direct property addition (not reactive)
    addEmail() {
      this.user.email = 'john@example.com'
      this.$forceUpdate() // Don't do this!
    },

    // ✅ CORRECT: Use $set for reactivity
    addEmailCorrect() {
      this.$set(this.user, 'email', 'john@example.com')
      // No need for $forceUpdate
    }
  }
}
</script>
```

### Alternative 2: Object Replacement

```vue
<script>
export default {
  data() {
    return {
      config: {
        setting1: 'value1'
      }
    }
  },
  methods: {
    // ❌ WRONG: Direct mutation of non-reactive object
    updateConfig() {
      this.config.setting2 = 'value2'
      this.$forceUpdate()
    },

    // ✅ CORRECT: Replace entire object (reactive)
    updateConfigCorrect() {
      this.config = {
        ...this.config,
        setting2: 'value2'
      }
    }
  }
}
</script>
```

### Alternative 3: Make Objects Reactive Upfront

```vue
<script>
export default {
  data() {
    return {
      // Initialize all possible properties
      user: {
        name: '',
        email: '',
        phone: '',
        address: ''
      }
    }
  },
  methods: {
    updateUser() {
      // All properties are reactive from the start
      this.user.email = 'john@example.com'
      // No need for $forceUpdate
    }
  }
}
</script>
```

## Legitimate Use Cases for $forceUpdate

### Use Case 1: Third-Party Libraries

```vue
<template>
  <div>
    <canvas ref="canvas"></canvas>
    <button @click="redraw">Redraw Canvas</button>
  </div>
</template>

<script>
import ThirdPartyLib from 'some-third-party-lib'

export default {
  data() {
    return {
      libInstance: null
    }
  },
  mounted() {
    // Initialize third-party library
    this.libInstance = new ThirdPartyLib(this.$refs.canvas, {
      data: this.initialData
    })
  },
  methods: {
    redraw() {
      // Update third-party library state
      this.libInstance.setData(newData)

      // Library doesn't trigger Vue reactivity
      this.$forceUpdate()
    }
  }
}
</script>
```

### Use Case 2: Reactivity Edge Cases

```vue
<script>
export default {
  data() {
    return {
      // Using window.location (not reactive)
      currentUrl: window.location.href
    }
  },
  mounted() {
    // Poll for URL changes (not ideal, but sometimes necessary)
    this.urlChecker = setInterval(() => {
      if (this.currentUrl !== window.location.href) {
        this.currentUrl = window.location.href
        // This change might not trigger re-render
        this.$forceUpdate()
      }
    }, 1000)
  },
  beforeDestroy() {
    clearInterval(this.urlChecker)
  }
}
</script>
```

### Use Case 3: Frozen Objects (Rare)

```vue
<script>
export default {
  data() {
    return {
      // Intentionally frozen object
      constants: Object.freeze({
        API_URL: 'https://api.example.com',
        VERSION: '1.0.0'
      })
    }
  },
  methods: {
    updateConstants() {
      // Can't make frozen objects reactive
      // But if you need to display updated frozen values:
      this.constants = Object.freeze({
        ...this.constants,
        VERSION: '2.0.0'
      })
      // This works (replacing the object)
    }
  }
}
</script>
```

## $forceUpdate Doesn't Affect Child Components

```vue
<template>
  <div>
    <child-component :data="parentData" />
    <button @click="forceParentOnly">
      Force Parent Only
    </button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      parentData: 'Initial'
    }
  },
  methods: {
    forceParentOnly() {
      this.parentData = 'Updated'
      // Only forces this component to re-render
      // Child component won't be affected
      this.$forceUpdate()
    }
  }
}
</script>
```

## Complete Example: Reactivity Problem

```vue
<template>
  <div class="todo-app">
    <h2>Todo List</h2>

    <!-- Problem: Adding items to array index doesn't work -->
    <div>
      <input v-model="newTodo" @keyup.enter="addTodoWrong">
      <button @click="addTodoWrong">Add (Wrong Way)</button>
    </div>

    <!-- Solution: Using array methods or $set -->
    <div>
      <input v-model="newTodo" @keyup.enter="addTodoRight">
      <button @click="addTodoRight">Add (Right Way)</button>
    </div>

    <ul>
      <li v-for="(todo, index) in todos" :key="index">
        {{ todo }}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      newTodo: '',
      todos: ['Learn Vue', 'Build something']
    }
  },
  methods: {
    // ❌ WRONG: Direct array index assignment
    addTodoWrong() {
      const newIndex = this.todos.length
      this.todos[newIndex] = this.newTodo // Not reactive!
      this.$forceUpdate() // Don't rely on this!
      this.newTodo = ''
    },

    // ✅ CORRECT: Use array push method
    addTodoRight() {
      this.todos.push(this.newTodo) // Reactive!
      this.newTodo = ''
    },

    // ✅ ALSO CORRECT: Use $set for index
    addTodoAlternative() {
      const newIndex = this.todos.length
      this.$set(this.todos, newIndex, this.newTodo) // Reactive!
      this.newTodo = ''
    }
  }
}
</script>
```

## Detecting When $forceUpdate is Needed

```vue
<script>
export default {
  data() {
    return {
      externalData: null
    }
  },
  mounted() {
    // Working with external data source
    const externalObject = getExternalObject()

    // Check if object is reactive
    if (!Vue.util.isReactive(externalObject)) {
      console.warn('External object is not reactive')
      // Make it reactive or use workarounds
    }

    this.externalData = Vue.observable(externalObject)
  }
}
</script>
```

## Performance Impact

```vue
<script>
export default {
  data() {
    return {
      items: [] // Large array
    }
  },
  methods: {
    // ❌ AVOID: Unnecessary force updates
    updateWithForce() {
      this.items.push(newItem)
      this.$forceUpdate() // Unnecessary, push is already reactive
    },

    // ✅ BETTER: Trust Vue's reactivity
    updateCorrectly() {
      this.items.push(newItem)
      // Vue will automatically detect the change
    }
  }
}
</script>
```

## Common Mistakes

### Mistake 1: Using $forceUpdate for Normal Reactivity

```vue
<script>
export default {
  data() {
    return {
      count: 0
    }
  },
  methods: {
    // ❌ WRONG: Using $forceUpdate unnecessarily
    increment() {
      this.count++
      this.$forceUpdate() // Not needed!
    },

    // ✅ CORRECT: Just update the data
    incrementCorrect() {
      this.count++
      // Vue handles reactivity automatically
    }
  }
}
</script>
```

### Mistake 2: Relying on $forceUpdate as a Fix

```vue
<script>
export default {
  methods: {
    // ❌ WRONG: Using $forceUpdate to bypass reactivity problems
    updateComplexState() {
      this.state.nested.property = 'new value'
      this.$forceUpdate() // Hiding the real problem
    },

    // ✅ CORRECT: Fix the reactivity issue
    updateComplexStateCorrect() {
      this.$set(this.state.nested, 'property', 'new value')
      // Proper reactivity
    }
  }
}
</script>
```

## Best Practices

1. **Avoid $forceUpdate** for normal Vue reactivity
2. **Fix reactivity at the source** using Vue.set, object replacement, or proper initialization
3. **Use $forceUpdate** only for:
   - Third-party library integration
   - Truly non-reactive external data
   - Extreme edge cases
4. **Document why** $forceUpdate is used when necessary
5. **Consider alternatives** like Vue.observable for external state

## Decision Tree

```
Need to update view after data change?
│
├─ Is the data in Vue's data()?
│  ├─ YES → Normal reactivity should work
│  │        Check for reactivity caveats (use $set)
│  └─ NO → Consider making it reactive
│
├─ Is the data from a third-party library?
│  ├─ YES → $forceUpdate might be appropriate
│  └─ NO → Fix reactivity properly
│
└─ Have you exhausted all reactivity options?
   ├─ YES → $forceUpdate as last resort
   └─ NO → Try $set, object replacement, Vue.observable
```

## Summary Table

| Scenario | Solution |
|----------|----------|
| Adding new object property | `this.$set(obj, 'key', value)` |
| Adding array item | `array.push(item)` |
| Array index assignment | `this.$set(array, index, value)` |
| Third-party library | `$forceUpdate` (acceptable) |
| Window/document properties | Use `$forceUpdate` or watch with polling |
| Frozen objects | Replace entire object |

## Reference

- [Vue 2 API - vm.$forceUpdate](https://vue2.docs.vuejs.org/v2/api/#vm-forceUpdate)
- [Vue 2 Reactivity - Change Detection Caveats](https://vue2.docs.vuejs.org/v2/guide/reactivity.html#Change-Detection-Caveats)
