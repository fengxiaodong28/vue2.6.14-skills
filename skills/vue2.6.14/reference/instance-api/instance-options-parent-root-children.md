---
title: Instance Properties $options, $parent, $root, $children, $isServer in Vue 2
impact: MEDIUM
impactDescription: Instance properties for accessing component options, parent/root instances, child components, and server-side rendering detection
type: capability
tags:
  - vue2.6.14
  - instance-properties
  - $options
  - $parent
  - $root
  - $children
  - $isServer
  - component-api
---

# Instance Properties $options, $parent, $root, $children, $isServer in Vue 2

Impact: MEDIUM - These instance properties provide access to component initialization options, parent/root component instances, child component instances, and server-side rendering context. They are useful for advanced component communication, debugging, and SSR-aware code.

## Task Checklist

- [ ] Use `$options` to access component initialization options
- [ ] Use `$parent` to access the parent component instance
- [ ] Use `$root` to access the root Vue instance
- [ ] Use `$children` to access direct child component instances
- [ ] Use `$isServer` to detect server-side rendering

## $options - Component Initialization Options

The `$options` property holds the options used to create the component instance.

```vue
<template>
  <div class="options-demo">
    <h2>{{ title }}</h2>
    <button @click="showOptions">Show Options</button>
  </div>
</template>

<script>
export default {
  name: 'OptionsDemo',
  // Custom option
  customOption: 'my custom value',
  props: {
    title: String
  },
  data() {
    return {
      message: 'Hello'
    }
  },
  methods: {
    showOptions() {
      // Access component options
      console.log('Component name:', this.$options.name)
      console.log('Props definition:', this.$options.props)
      console.log('Data function:', this.$options.data)
      console.log('Custom option:', this.$options.customOption)
      console.log('All options:', this.$options)
    }
  }
}
</script>
```

## Custom Options with $options

```vue
<template>
  <div class="custom-options">
    <button @click="logConfig">Log Config</button>
  </div>
</template>

<script>
export default {
  name: 'CustomOptionsComponent',
  // Define custom options
  metaInfo: {
    title: 'My Page',
    description: 'Page description'
  },
  permissions: {
    canEdit: true,
    canDelete: false
  },
  apiConfig: {
    endpoint: '/api/data',
    method: 'GET'
  },
  methods: {
    logConfig() {
      // Access custom options
      const meta = this.$options.metaInfo
      const perms = this.$options.permissions
      const api = this.$options.apiConfig

      console.log('Meta:', meta)
      console.log('Permissions:', perms)
      console.log('API Config:', api)

      // Use custom options for logic
      if (perms.canEdit) {
        console.log('User can edit')
      }
    }
  }
}
</script>
```

## $parent - Parent Component Instance

Access the parent component instance.

```vue
<template>
  <div class="parent-demo">
    <h2>Parent Component</h2>
    <p>Parent message: {{ message }}</p>

    <child-component />
  </div>
</template>

<script>
export default {
  name: 'ParentComponent',
  components: {
    ChildComponent: {
      name: 'ChildComponent',
      template: `
        <div class="child">
          <button @click="accessParent">Access Parent</button>
        </div>
      `,
      methods: {
        accessParent() {
          // Access parent component
          const parent = this.$parent

          if (parent) {
            console.log('Parent name:', parent.$options.name)
            console.log('Parent data:', parent.message)

            // Call parent method
            if (parent.parentMethod) {
              parent.parentMethod()
            }

            // Modify parent data (use with caution!)
            parent.message = 'Modified from child'
          }
        }
      }
    }
  },
  data() {
    return {
      message: 'Hello from parent'
    }
  },
  methods: {
    parentMethod() {
      console.log('Parent method called from child')
    }
  }
}
</script>
```

## $root - Root Vue Instance

Access the root Vue instance of the component tree.

```vue
<template>
  <div id="app">
    <header-component />
    <main-component />
    <footer-component />
  </div>
</template>

<script>
// Main App Component
export default {
  name: 'App',
  data() {
    return {
      globalState: {
        user: null,
        theme: 'light',
        language: 'en'
      }
    }
  },
  methods: {
    setGlobalTheme(theme) {
      this.globalState.theme = theme
    },
    setGlobalUser(user) {
      this.globalState.user = user
    }
  },
  components: {
    'header-component': {
      name: 'HeaderComponent',
      template: `
        <header>
          <button @click="toggleTheme">Toggle Theme</button>
          <span>Current theme: {{ currentTheme }}</span>
        </header>
      `,
      computed: {
        currentTheme() {
          // Access root state
          return this.$root.globalState.theme
        }
      },
      methods: {
        toggleTheme() {
          const root = this.$root
          const newTheme = root.globalState.theme === 'light' ? 'dark' : 'light'
          root.setGlobalTheme(newTheme)
        }
      }
    },
    'main-component': {
      name: 'MainComponent',
      template: `
        <main>
          <p>Current user: {{ currentUser }}</p>
        </main>
      `,
      computed: {
        currentUser() {
          return this.$root.globalState.user || 'Not logged in'
        }
      }
    },
    'footer-component': {
      name: 'FooterComponent',
      template: `
        <footer>
          <p>Language: {{ language }}</p>
        </footer>
      `,
      computed: {
        language() {
          return this.$root.globalState.language
        }
      }
    }
  }
}
</script>
```

## $children - Child Component Instances

Access direct child component instances (Vue 2 only, deprecated in Vue 3).

```vue
<template>
  <div class="children-demo">
    <child-item v-for="item in items" :key="item.id" :data="item" />
    <button @click="validateChildren">Validate All Children</button>
    <button @click="collectChildData">Collect Child Data</button>
  </div>
</template>

<script>
export default {
  name: 'ChildrenDemo',
  data() {
    return {
      items: [
        { id: 1, name: 'Item 1' },
        { id: 2, name: 'Item 2' },
        { id: 3, name: 'Item 3' }
      ]
    }
  },
  methods: {
    validateChildren() {
      // Access all direct children
      this.$children.forEach((child, index) => {
        console.log(`Child ${index}:`, child.$options.name)

        // Call child method if it exists
        if (child.validate) {
          const isValid = child.validate()
          console.log(`Child ${index} valid:`, isValid)
        }
      })
    },
    collectChildData() {
      // Collect data from all children
      const allData = this.$children.map(child => {
        return {
          name: child.$options.name,
          data: child.$data
        }
      })

      console.log('All children data:', allData)
      return allData
    }
  },
  components: {
    ChildItem: {
      name: 'ChildItem',
      props: ['data'],
      data() {
        return {
          value: '',
          valid: false
        }
      },
      template: `
        <div class="child-item">
          <span>{{ data.name }}</span>
          <input v-model="value" @input="validate">
        </div>
      `,
      methods: {
        validate() {
          this.valid = this.value.length > 0
          return this.valid
        }
      }
    }
  }
}
</script>
```

## $isServer - Server-Side Rendering Detection

Detect if the component is running on the server.

```vue
<template>
  <div class="ssr-aware">
    <p>Running on: {{ environment }}</p>
    <p v-if="!isServer">Client-only content</p>
  </div>
</template>

<script>
export default {
  name: 'SSRAware',
  computed: {
    environment() {
      return this.$isServer ? 'Server (SSR)' : 'Client'
    },
    isServer() {
      return this.$isServer
    }
  },
  mounted() {
    // This only runs on client
    if (!this.$isServer) {
      console.log('Component mounted on client')
      this.initClientOnlyFeatures()
    }
  },
  methods: {
    initClientOnlyFeatures() {
      // Initialize browser-only APIs
      if (typeof window !== 'undefined') {
        window.addEventListener('resize', this.handleResize)
      }
    },
    handleResize() {
      console.log('Window resized')
    }
  },
  beforeDestroy() {
    if (!this.$isServer) {
      window.removeEventListener('resize', this.handleResize)
    }
  }
}
</script>
```

## Practical: Global Event Bus with $root

```vue
<template>
  <div id="app">
    <notification-component />
    <button-component />
    <form-component />
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      // Event bus on root
      bus: {
        events: {}
      }
    }
  },
  created() {
    // Setup event bus methods
    this.$on = (event, callback) => {
      if (!this.bus.events[event]) {
        this.bus.events[event] = []
      }
      this.bus.events[event].push(callback)
    }

    this.$emit = (event, data) => {
      if (this.bus.events[event]) {
        this.bus.events[event].forEach(callback => callback(data))
      }
    }

    this.$off = (event, callback) => {
      if (this.bus.events[event]) {
        if (callback) {
          this.bus.events[event] = this.bus.events[event].filter(cb => cb !== callback)
        } else {
          delete this.bus.events[event]
        }
      }
    }
  },
  components: {
    NotificationComponent: {
      name: 'NotificationComponent',
      data() {
        return { message: '' }
      },
      template: `
        <div v-if="message" class="notification">
          {{ message }}
        </div>
      `,
      created() {
        // Listen to root events
        this.$root.$on('notify', this.showNotification)
      },
      beforeDestroy() {
        this.$root.$off('notify', this.showNotification)
      },
      methods: {
        showNotification(msg) {
          this.message = msg
          setTimeout(() => {
            this.message = ''
          }, 3000)
        }
      }
    },
    ButtonComponent: {
      name: 'ButtonComponent',
      template: `
        <button @click="handleClick">Show Notification</button>
      `,
      methods: {
        handleClick() {
          // Emit to root
          this.$root.$emit('notify', 'Button clicked!')
        }
      }
    },
    FormComponent: {
      name: 'FormComponent',
      template: `
        <form @submit.prevent="handleSubmit">
          <input placeholder="Enter message" v-model="message">
          <button type="submit">Submit</button>
        </form>
      `,
      data() {
        return { message: '' }
      },
      methods: {
        handleSubmit() {
          this.$root.$emit('notify', this.message)
          this.message = ''
        }
      }
    }
  }
}
</script>
```

## Component Tree Traversal

```vue
<template>
  <div class="tree-traversal">
    <middle-component />
    <button @click="traverseTree">Traverse Tree</button>
  </div>
</template>

<script>
export default {
  name: 'TreeTraversal',
  methods: {
    traverseTree() {
      console.log('=== Root ===')
      console.log('Name:', this.$options.name)
      console.log('Root:', this.$root === this)

      // Access children
      console.log('\n=== Children ===')
      this.$children.forEach((child, i) => {
        console.log(`Child ${i}:`, child.$options.name)
      })
    }
  },
  components: {
    MiddleComponent: {
      name: 'MiddleComponent',
      template: `
        <div class="middle">
          <leaf-component />
        </div>
      `,
      components: {
        LeafComponent: {
          name: 'LeafComponent',
          template: `
            <div class="leaf">
              <button @click="showAncestry">Show Ancestry</button>
            </div>
          `,
          methods: {
            showAncestry() {
              console.log('=== Component Ancestry ===')

              // Current component
              console.log('Current:', this.$options.name)

              // Parent
              if (this.$parent) {
                console.log('Parent:', this.$parent.$options.name)
              }

              // Root
              console.log('Root:', this.$root.$options.name)
              console.log('Is root?', this.$root === this)

              // Path to root
              let component = this
              let path = []
              while (component) {
                path.push(component.$options.name)
                component = component.$parent
              }
              console.log('Path:', path.join(' -> '))
            }
          }
        }
      }
    }
  }
}
</script>
```

## Common Patterns

### 1. Shared Application Configuration

```vue
<template>
  <div id="app">
    <router-view />
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      config: {
        apiUrl: 'https://api.example.com',
        version: '1.0.0',
        features: {
          darkMode: true,
          betaFeatures: false
        }
      }
    }
  }
}
</script>

<!-- Any component can access config -->
<script>
export default {
  name: 'AnyComponent',
  computed: {
    apiUrl() {
      return this.$root.config.apiUrl
    }
  }
}
</script>
```

### 2. Dynamic Form Validation

```vue
<template>
  <form class="dynamic-form" @submit.prevent="submit">
    <form-field
      v-for="field in fields"
      :key="field.name"
      :field="field"
    />
    <button type="submit">Submit</button>
    <button type="button" @click="validateAll">Validate All</button>
  </form>
</template>

<script>
export default {
  name: 'DynamicForm',
  data() {
    return {
      fields: [
        { name: 'email', label: 'Email', type: 'email', required: true },
        { name: 'password', label: 'Password', type: 'password', required: true }
      ]
    }
  },
  methods: {
    validateAll() {
      // Validate all child form fields
      const children = this.$children

      let allValid = true
      children.forEach(child => {
        if (child.validate) {
          const isValid = child.validate()
          if (!isValid) {
            allValid = false
          }
        }
      })

      return allValid
    },
    submit() {
      if (this.validateAll()) {
        // Collect form data
        const formData = {}
        this.$children.forEach(child => {
          if (child.field && child.value) {
            formData[child.field.name] = child.value
          }
        })
        console.log('Form data:', formData)
      }
    }
  },
  components: {
    FormField: {
      name: 'FormField',
      props: ['field'],
      data() {
        return {
          value: '',
          error: ''
        }
      },
      template: `
        <div class="form-field">
          <label>{{ field.label }}</label>
          <input
            :type="field.type"
            v-model="value"
            @blur="validate"
          >
          <span v-if="error" class="error">{{ error }}</span>
        </div>
      `,
      methods: {
        validate() {
          this.error = ''

          if (this.field.required && !this.value) {
            this.error = `${this.field.label} is required`
            return false
          }

          if (this.field.type === 'email' && this.value) {
            const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
            if (!emailRegex.test(this.value)) {
              this.error = 'Invalid email format'
              return false
            }
          }

          return true
        }
      }
    }
  }
}
</script>
```

### 3. SSR-Aware Component

```vue
<template>
  <div class="ssr-component">
    <div ref="chartContainer"></div>
  </div>
</template>

<script>
import ChartLibrary from 'chart-library'

export default {
  name: 'SSRComponent',
  data() {
    return {
      chart: null
    }
  },
  mounted() {
    // Only initialize chart on client
    if (!this.$isServer) {
      this.initChart()
    }
  },
  methods: {
    initChart() {
      // Browser-only code
      const container = this.$refs.chartContainer
      this.chart = new ChartLibrary(container, {
        // Chart options
      })
    }
  },
  beforeDestroy() {
    // Cleanup only on client
    if (!this.$isServer && this.chart) {
      this.chart.destroy()
    }
  }
}
</script>
```

## Important Warnings

```vue
<script>
export default {
  methods: {
    // ❌ AVOID: Tight coupling to parent
    badMethod() {
      // Assumes specific parent structure
      this.$parent.doSomething()
    },

    // ❌ AVOID: Modifying parent data directly
    anotherBadMethod() {
      this.$parent.someData = 'modified'
    },

    // ✅ CORRECT: Use props and events
    goodMethod() {
      this.$emit('something-happened', { data: 'value' })
    },

    // ❌ AVOID: Relying on $children order
    badChildrenAccess() {
      const firstChild = this.$children[0]
      // Order is not guaranteed!
    },

    // ✅ CORRECT: Use refs for specific children
    goodChildrenAccess() {
      const specificChild = this.$refs.specificChild
      if (specificChild) {
        specificChild.doSomething()
      }
    }
  }
}
</script>
```

## Best Practices

1. **Prefer props/events over $parent** - Better component decoupling
2. **Use refs instead of $children** - More predictable access
3. **Limit $root usage** - Consider Vuex or provide/inject for shared state
4. **Use $isServer for browser-only APIs** - Prevent SSR errors
5. **Document custom $options** - Make them discoverable for other developers
6. **Consider component hierarchy changes** - $parent/$children relationships can break

## Reference

- [Vue 2 API - Instance Properties](https://v2.vuejs.org/v2/api/#Instance-Properties)
- [Vue 2 Guide - Component Registration](https://v2.vuejs.org/v2/guide/components-registration.html)
