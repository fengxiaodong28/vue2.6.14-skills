---
title: Instance Properties $data, $props, $el in Vue 2
impact: MEDIUM
impactDescription: $data, $props, and $el provide direct access to component data, props, and root DOM element
type: capability
tags:
  - vue2.6.14
  - instance-properties
  - $data
  - $props
  - $el
  - component-api
---

# Instance Properties $data, $props, $el in Vue 2

Impact: MEDIUM - `$data`, `$props`, and `$el` are instance properties that provide direct access to a component's data object, props object, and root DOM element respectively. These are useful for debugging, testing, and advanced component manipulation.

## Task Checklist

- [ ] Use `$data` to access the component's reactive data object
- [ ] Use `$props` to access the component's received props
- [ ] Use `$el` to access the component's root DOM element
- [ ] Remember these are read-only for safety (mostly)
- [ ] Prefer template refs over `$el` when possible

## $data - Component Data Object

The `$data` property returns the component's reactive data object.

```vue
<template>
  <div class="user-profile">
    <h2>{{ user.name }}</h2>
    <p>Email: {{ user.email }}</p>
    <button @click="logData">Log $data</button>
  </div>
</template>

<script>
export default {
  name: 'UserProfile',
  data() {
    return {
      user: {
        name: 'John Doe',
        email: 'john@example.com'
      },
      loading: false,
      error: null
    }
  },
  methods: {
    logData() {
      // Access the entire data object
      console.log('Component data:', this.$data)
      // Output:
      // {
      //   user: { name: 'John Doe', email: 'john@example.com' },
      //   loading: false,
      //   error: null
      // }

      // Direct property access (same as this.user)
      console.log('User:', this.$data.user)

      // Modifying $data works the same as modifying properties directly
      this.$data.user.name = 'Jane Doe'
      // This is equivalent to: this.user.name = 'Jane Doe'
    }
  }
}
</script>
```

## $props - Component Props Object

The `$props` property returns the object of props received by the component.

```vue
<template>
  <div class="product-card">
    <h2>{{ title }}</h2>
    <p>Price: {{ price }}</p>
    <p>In Stock: {{ inStock }}</p>
    <button @click="logProps">Log Props</button>
  </div>
</template>

<script>
export default {
  name: 'ProductCard',
  props: {
    title: {
      type: String,
      required: true
    },
    price: {
      type: Number,
      default: 0
    },
    inStock: {
      type: Boolean,
      default: false
    }
  },
  methods: {
    logProps() {
      // Access all received props
      console.log('Props:', this.$props)
      // Output (when called as <product-card title="Widget" price="9.99" />):
      // {
      //   title: 'Widget',
      //   price: 9.99,
      //   inStock: false
      // }

      // Check if prop was passed
      console.log('Has price:', this.$props.price !== undefined)

      // Props are read-only - Vue will warn about mutation
      // this.$props.title = 'New Title' // ❌ WARNING: Avoid mutating props!
    }
  }
}
</script>
```

## $el - Root DOM Element

The `$el` property returns the root DOM element that the Vue instance is mounted to.

```vue
<template>
  <div class="my-component">
    <h2>Component Title</h2>
    <p>Content here</p>
    <button @click="focusElement">Focus Me</button>
  </div>
</template>

<script>
export default {
  name: 'MyComponent',
  mounted() {
    // Access the root DOM element
    console.log('Root element:', this.$el)
    console.log('Element class:', this.$el.className)
    // Output: "my-component"

    // Manipulate the root element
    this.$el.style.border = '1px solid red'

    // Add event listener to root element
    this.$el.addEventListener('click', this.handleClick)
  },
  beforeDestroy() {
    // Clean up event listeners
    this.$el.removeEventListener('click', this.handleClick)
  },
  methods: {
    handleClick(event) {
      console.log('Root element clicked', event)
    },
    focusElement() {
      // Focus the component's root element
      this.$el.focus()
    }
  }
}
</script>
```

## Comparing Direct Access vs Instance Properties

```vue
<template>
  <div class="comparison">
    <p>{{ message }}</p>
    <button @click="compareAccess">Compare Access Methods</button>
  </div>
</template>

<script>
export default {
  name: 'ComparisonComponent',
  data() {
    return {
      message: 'Hello Vue 2!'
    }
  },
  props: {
    title: String
  },
  methods: {
    compareAccess() {
      // Data access - all equivalent
      console.log(this.message)         // Recommended
      console.log(this.$data.message)  // Access via $data

      // Props access
      console.log(this.title)          // Recommended
      console.log(this.$props.title)   // Access via $props

      // DOM element
      console.log(this.$el)            // Root DOM element
    }
  }
}
</script>
```

## Using $data for Deep Copy

```vue
<template>
  <div class="form-state">
    <form @submit.prevent="submit">
      <input v-model="form.name" placeholder="Name">
      <input v-model="form.email" placeholder="Email">
      <button type="submit">Submit</button>
      <button type="button" @click="reset">Reset</button>
    </form>
  </div>
</template>

<script>
export default {
  name: 'FormState',
  data() {
    return {
      form: {
        name: '',
        email: ''
      },
      // Save initial state for reset
      initialState: null
    }
  },
  created() {
    // Store initial state using JSON.parse/stringify for deep copy
    this.initialState = JSON.parse(JSON.stringify(this.$data.form))
  },
  methods: {
    submit() {
      // Handle form submission
      console.log('Submitted:', this.form)
    },
    reset() {
      // Reset to initial state
      this.form = JSON.parse(JSON.stringify(this.initialState))
    },
    saveCheckpoint() {
      // Save current state as checkpoint
      this.checkpoint = JSON.parse(JSON.stringify(this.$data))
    },
    restoreCheckpoint() {
      // Restore from checkpoint
      if (this.checkpoint) {
        Object.assign(this.$data, JSON.parse(JSON.stringify(this.checkpoint)))
      }
    }
  }
}
</script>
```

## Using $props for Dynamic Component Behavior

```vue
<template>
  <div class="dynamic-card" :class="$props.variant">
    <h2>{{ $props.title }}</h2>
    <p>{{ $props.content }}</p>

    <!-- Check if certain props exist -->
    <button v-if="$props.actionText" @click="$emit('action')">
      {{ $props.actionText }}
    </button>
  </div>
</template>

<script>
export default {
  name: 'DynamicCard',
  props: {
    title: String,
    content: String,
    variant: {
      type: String,
      default: 'default'
    },
    actionText: String
  }
}
</script>

<style scoped>
.default {
  border: 1px solid #ccc;
}
.primary {
  border: 2px solid #42b983;
  background: #f0fdf4;
}
.warning {
  border: 2px solid #f59e0b;
  background: #fffbeb;
}
</style>
```

## Using $el for Third-Party Library Integration

```vue
<template>
  <div class="chart-container">
    <canvas ref="chartCanvas"></canvas>
    <button @click="destroyChart">Destroy Chart</button>
  </div>
</template>

<script>
import Chart from 'chart.js'

export default {
  name: 'ChartComponent',
  data() {
    return {
      chartInstance: null
    }
  },
  mounted() {
    this.initChart()
  },
  beforeDestroy() {
    this.destroyChart()
  },
  methods: {
    initChart() {
      const canvas = this.$el.querySelector('canvas')

      this.chartInstance = new Chart(canvas, {
        type: 'bar',
        data: {
          labels: ['Red', 'Blue', 'Yellow'],
          datasets: [{
            label: '# of Votes',
            data: [12, 19, 3],
            backgroundColor: ['red', 'blue', 'yellow']
          }]
        }
      })
    },
    destroyChart() {
      if (this.chartInstance) {
        this.chartInstance.destroy()
        this.chartInstance = null
      }
    }
  }
}
</script>
```

## $el with Multi-Root Components

```vue
<template>
  <!-- Vue 2 only supports single root element -->
  <div class="single-root">
    <header>Header</header>
    <main>Main content</main>
    <footer>Footer</footer>
  </div>
</template>

<script>
export default {
  name: 'SingleRootComponent',
  mounted() {
    // $el refers to the single root div
    console.log(this.$el) // <div class="single-root">...</div>

    // To access child elements, use querySelector
    const header = this.$el.querySelector('header')
    const main = this.$el.querySelector('main')
    const footer = this.$el.querySelector('footer')
  }
}
</script>
```

## Debugging with Instance Properties

```vue
<template>
  <div class="debug-component">
    <h2>{{ title }}</h2>
    <button @click="debug">Debug Info</button>
  </div>
</template>

<script>
export default {
  name: 'DebugComponent',
  props: {
    title: String
  },
  data() {
    return {
      counter: 0,
      user: { name: 'John' }
    }
  },
  computed: {
    doubleCounter() {
      return this.counter * 2
    }
  },
  methods: {
    debug() {
      // Comprehensive debug output
      console.group('Component Debug Info')
      console.log('Name:', this.$options.name)
      console.log('Props:', this.$props)
      console.log('Data:', this.$data)
      console.log('Computed:', {
        doubleCounter: this.doubleCounter
      })
      console.log('Element:', this.$el)
      console.groupEnd()

      // Or use window for quick console access
      window.debugComponent = this
      console.log('Access via: window.debugComponent')
    }
  }
}
</script>
```

## Warning: Avoid Direct $data Mutation in Most Cases

```vue
<script>
export default {
  data() {
    return {
      items: [1, 2, 3]
    }
  },
  methods: {
    // ❌ AVOID: Replacing entire $data
    badReset() {
      this.$data = {
        items: [1, 2, 3]
      }
      // This breaks reactivity and watcher connections!
    },

    // ✅ CORRECT: Reset individual properties
    goodReset() {
      this.items = [1, 2, 3]
    },

    // ✅ CORRECT: Use Object.assign to preserve reactivity
    betterReset() {
      Object.assign(this.$data, {
        items: [1, 2, 3]
      })
    }
  }
}
</script>
```

## $data for Form Reset Pattern

```vue
<template>
  <form @submit.prevent="submit" @reset="handleReset">
    <input v-model="form.name" placeholder="Name">
    <input v-model="form.email" placeholder="Email">
    <button type="submit">Submit</button>
    <button type="reset">Reset</button>
  </form>
</template>

<script>
export default {
  data() {
    return {
      form: {
        name: '',
        email: ''
      },
      defaultData: null
    }
  },
  created() {
    // Capture default state
    this.defaultData = JSON.parse(JSON.stringify(this.$data))
  },
  methods: {
    submit() {
      // Handle submit
    },
    handleReset(event) {
      event.preventDefault()
      // Reset to defaults
      Object.assign(this.$data, JSON.parse(JSON.stringify(this.defaultData)))
    }
  }
}
</script>
```

## Common Patterns

### 1. Component Self-Registration

```vue
<template>
  <div class="self-registering">
    <button @click="register">Register with Parent</button>
  </div>
</template>

<script>
export default {
  name: 'SelfRegisteringChild',
  methods: {
    register() {
      // Emit event passing component reference
      this.$emit('register', this)
      // Parent can do: child.componentMethod()
    },
    componentMethod() {
      console.log('Called from parent')
    }
  }
}
</script>
```

### 2. Accessing Child Component Data

```vue
<template>
  <div>
    <child-component ref="child" />
    <button @click="inspectChild">Inspect Child</button>
  </div>
</template>

<script>
export default {
  methods: {
    inspectChild() {
      const child = this.$refs.child

      if (child) {
        console.log('Child $data:', child.$data)
        console.log('Child $props:', child.$props)
        console.log('Child $el:', child.$el)
      }
    }
  }
}
</script>
```

### 3. DOM Measurement

```vue
<template>
  <div class="measurable" ref="container">
    <p>Content</p>
  </div>
</template>

<script>
export default {
  mounted() {
    this.measureElement()
  },
  methods: {
    measureElement() {
      // Get element dimensions
      const rect = this.$el.getBoundingClientRect()
      console.log('Width:', rect.width)
      console.log('Height:', rect.height)
      console.log('Top:', rect.top)
      console.log('Left:', rect.left)

      // Get scroll position
      console.log('Scroll top:', this.$el.scrollTop)
      console.log('Scroll height:', this.$el.scrollHeight)
    }
  }
}
</script>
```

## Best Practices

1. **Prefer direct property access** - Use `this.propName` over `this.$data.propName`
2. **Use template refs** - Prefer `ref` attribute over `$el` for child elements
3. **Don't mutate $props** - Props should remain read-only
4. **Be careful with $data replacement** - Can break reactivity
5. **Clean up DOM listeners** - Remove event listeners in `beforeDestroy`
6. **Use for debugging only** - These are primarily debugging/testing tools

## Reference

- [Vue 2 API - Instance Properties](https://v2.vuejs.org/v2/api/#Instance-Properties)
- [Vue 2 Guide - Instance Properties](https://v2.vuejs.org/v2/guide/instance.html)
