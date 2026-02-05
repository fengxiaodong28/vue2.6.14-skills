---
title: Vue Plugin Development with Vue.use
impact: MEDIUM
impactDescription: Plugins allow adding global-level functionality to Vue with Vue.use() for reusable features
type: capability
tags:
  - vue2.6.14
  - plugins
  - Vue.use
  - plugin-development
  - global-functionality
---

# Vue Plugin Development with Vue.use

Impact: MEDIUM - Vue plugins allow you to add global-level functionality to Vue. Use `Vue.use()` to install plugins that add global methods, directives, mixins, or inject component options.

## Task Checklist

- [ ] Create an `install` method on the plugin object/function
- [ ] Accept `Vue` constructor as first argument of install method
- [ ] Optionally accept options as second argument
- [ ] Call `Vue.use()` before creating the root Vue instance
- [ ] Use plugins for reusable features across the application

## Basic Plugin Structure

```javascript
// MyPlugin.js
const MyPlugin = {
  install(Vue, options) {
    // 1. Add global method or property
    Vue.myGlobalMethod = function () {
      // logic...
    }

    // 2. Add a global asset
    Vue.directive('my-directive', {
      bind(el, binding, vnode, oldVnode) {
        // logic...
      }
    })

    // 3. Inject some component options
    Vue.mixin({
      created: function () {
        // logic...
      }
    })

    // 4. Add an instance method
    Vue.prototype.$myMethod = function (methodOptions) {
      // logic...
    }
  }
}

export default MyPlugin
```

## Example 1: HTTP Client Plugin

```javascript
// plugins/http.js
import axios from 'axios'

const HttpPlugin = {
  install(Vue, options = {}) {
    const http = axios.create({
      baseURL: options.baseURL || '/api',
      timeout: options.timeout || 5000,
      headers: options.headers || {}
    })

    // Request interceptor
    http.interceptors.request.use(
      config => {
        // Add auth token if available
        const token = Vue.prototype.$auth?.token
        if (token) {
          config.headers.Authorization = `Bearer ${token}`
        }
        return config
      },
      error => Promise.reject(error)
    )

    // Response interceptor
    http.interceptors.response.use(
      response => response.data,
      error => {
        // Global error handling
        if (error.response?.status === 401) {
          Vue.prototype.$router?.push('/login')
        }
        return Promise.reject(error)
      }
    )

    // Add to Vue prototype
    Vue.prototype.$http = http
  }
}

export default HttpPlugin
```

**Usage:**

```javascript
// main.js
import Vue from 'vue'
import HttpPlugin from './plugins/http'

Vue.use(HttpPlugin, {
  baseURL: process.env.VUE_APP_API_URL,
  timeout: 10000
})
```

**In components:**

```vue
<script>
export default {
  async created() {
    try {
      const users = await this.$http.get('/users')
      this.users = users
    } catch (error) {
      console.error('Failed to fetch users:', error)
    }
  }
}
</script>
```

## Example 2: Notification/Toast Plugin

```javascript
// plugins/toast.js
import ToastComponent from './ToastComponent.vue'

const ToastPlugin = {
  install(Vue, options = {}) {
    const ToastConstructor = Vue.extend(ToastComponent)

    const toast = (message, toastOptions = {}) => {
      const instance = new ToastConstructor({
        propsData: {
          message,
          type: toastOptions.type || 'info',
          duration: toastOptions.duration || 3000,
          position: toastOptions.position || 'top-right'
        }
      })

      instance.$mount()

      document.body.appendChild(instance.$el)

      // Auto remove after duration
      setTimeout(() => {
        instance.visible = false
        setTimeout(() => {
          if (instance.$el.parentNode) {
            document.body.removeChild(instance.$el)
          }
        }, 300) // Wait for transition
      }, instance.duration)

      return instance
    }

    // Add convenience methods
    toast.success = (message, options) => toast(message, { ...options, type: 'success' })
    toast.error = (message, options) => toast(message, { ...options, type: 'error' })
    toast.warning = (message, options) => toast(message, { ...options, type: 'warning' })
    toast.info = (message, options) => toast(message, { ...options, type: 'info' })

    Vue.prototype.$toast = toast

    // Global component method
    Vue.$toast = toast
  }
}

export default ToastPlugin
```

**ToastComponent.vue:**

```vue
<template>
  <transition :name="transition">
    <div v-if="visible" :class="classes" :style="positionStyle">
      <span class="toast-icon">{{ icon }}</span>
      <span class="toast-message">{{ message }}</span>
    </div>
  </transition>
</template>

<script>
export default {
  name: 'ToastComponent',
  props: {
    message: String,
    type: {
      type: String,
      default: 'info',
      validator: (value) => ['success', 'error', 'warning', 'info'].includes(value)
    },
    duration: {
      type: Number,
      default: 3000
    },
    position: {
      type: String,
      default: 'top-right'
    }
  },
  data() {
    return {
      visible: true
    }
  },
  computed: {
    classes() {
      return ['toast', `toast--${this.type}`]
    },
    positionStyle() {
      const positions = {
        'top-right': { top: '20px', right: '20px' },
        'top-left': { top: '20px', left: '20px' },
        'bottom-right': { bottom: '20px', right: '20px' },
        'bottom-left': { bottom: '20px', left: '20px' },
        'top-center': { top: '20px', left: '50%', transform: 'translateX(-50%)' }
      }
      return positions[this.position] || positions['top-right']
    },
    transition() {
      const transitions = {
        'top-right': 'slide-left',
        'top-left': 'slide-right',
        'bottom-right': 'slide-left',
        'bottom-left': 'slide-right',
        'top-center': 'slide-down'
      }
      return transitions[this.position] || 'slide-left'
    },
    icon() {
      const icons = {
        success: '✓',
        error: '✕',
        warning: '⚠',
        info: 'ℹ'
      }
      return icons[this.type]
    }
  }
}
</script>

<style scoped>
.toast {
  display: flex;
  align-items: center;
  padding: 12px 16px;
  border-radius: 4px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.15);
  z-index: 9999;
}
.toast--success { background: #f0f9eb; color: #67c23a; }
.toast--error { background: #fef0f0; color: #f56c6c; }
.toast--warning { background: #fdf6ec; color: #e6a23c; }
.toast--info { background: #f4f4f5; color: #909399; }
.toast-icon { margin-right: 8px; font-weight: bold; }
.slide-left-enter-active, .slide-left-leave-active { transition: all 0.3s; }
.slide-left-enter { transform: translateX(100%); opacity: 0; }
.slide-left-leave-to { transform: translateX(100%); opacity: 0; }
</style>
```

**Usage:**

```javascript
// main.js
import Vue from 'vue'
import ToastPlugin from './plugins/toast'

Vue.use(ToastPlugin, {
  defaultDuration: 5000,
  defaultPosition: 'top-center'
})
```

**In components:**

```vue
<script>
export default {
  methods: {
    submitForm() {
      this.$toast.success('Form submitted successfully!')
    },
    handleError() {
      this.$toast.error('Something went wrong!')
    }
  }
}
</script>
```

## Example 3: Event Bus Plugin

```javascript
// plugins/event-bus.js
class EventBus {
  constructor() {
    this.events = {}
  }

  on(event, callback) {
    if (!this.events[event]) {
      this.events[event] = []
    }
    this.events[event].push(callback)
  }

  off(event, callback) {
    if (!this.events[event]) return
    if (!callback) {
      delete this.events[event]
      return
    }
    this.events[event] = this.events[event].filter(cb => cb !== callback)
  }

  emit(event, ...args) {
    if (!this.events[event]) return
    this.events[event].forEach(callback => callback(...args))
  }

  once(event, callback) {
    const onceCallback = (...args) => {
      callback(...args)
      this.off(event, onceCallback)
    }
    this.on(event, onceCallback)
  }
}

const EventBusPlugin = {
  install(Vue) {
    const bus = new EventBus()

    Vue.prototype.$bus = bus

    // Also add to Vue constructor for access outside components
    Vue.$bus = bus
  }
}

export default EventBusPlugin
```

**Usage:**

```javascript
// Component A - Emit event
export default {
  methods: {
    notify() {
      this.$bus.emit('user-logged-in', { userId: 123 })
    }
  }
}

// Component B - Listen to event
export default {
  created() {
    this.$bus.on('user-logged-in', this.handleUserLogin)
  },
  beforeDestroy() {
    this.$bus.off('user-logged-in', this.handleUserLogin)
  },
  methods: {
    handleUserLogin(data) {
      console.log('User logged in:', data)
    }
  }
}
```

## Example 4: Validation Plugin

```javascript
// plugins/validation.js

const ValidationPlugin = {
  install(Vue, options = {}) {
    const rules = options.rules || {}

    // Add validation mixin
    Vue.mixin({
      data() {
        return {
          validationErrors: {}
        }
      },
      methods: {
        $validate(data, schema) {
          this.validationErrors = {}
          let isValid = true

          for (const field in schema) {
            const fieldRules = schema[field]
            const value = data[field]

            for (const rule of fieldRules) {
              const result = this.$validateField(value, rule)
              if (result !== true) {
                this.validationErrors[field] = result
                isValid = false
                break
              }
            }
          }

          return isValid
        },

        $validateField(value, rule) {
          // Built-in rules
          if (rule.required && (value === undefined || value === null || value === '')) {
            return rule.message || `${rule.field || 'This field'} is required`
          }

          if (rule.email && value && !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) {
            return rule.message || 'Invalid email format'
          }

          if (rule.min && value && value.length < rule.min) {
            return rule.message || `Minimum length is ${rule.min}`
          }

          if (rule.max && value && value.length > rule.max) {
            return rule.message || `Maximum length is ${rule.max}`
          }

          // Custom rule
          if (rule.validate && typeof rule.validate === 'function') {
            return rule.validate(value) || true
          }

          return true
        },

        $hasError(field) {
          return !!this.validationErrors[field]
        },

        $getError(field) {
          return this.validationErrors[field] || ''
        }
      }
    })
  }
}

export default ValidationPlugin
```

**Usage:**

```javascript
// main.js
import Vue from 'vue'
import ValidationPlugin from './plugins/validation'

Vue.use(ValidationPlugin, {
  rules: {
    email: {
      email: true,
      message: 'Please enter a valid email'
    }
  }
})
```

**In component:**

```vue
<template>
  <form @submit.prevent="submit">
    <div>
      <label>Name:</label>
      <input v-model="form.name">
      <span v-if="$hasError('name')" class="error">{{ $getError('name') }}</span>
    </div>
    <div>
      <label>Email:</label>
      <input v-model="form.email">
      <span v-if="$hasError('email')" class="error">{{ $getError('email') }}</span>
    </div>
    <button type="submit">Submit</button>
  </form>
</template>

<script>
export default {
  data() {
    return {
      form: {
        name: '',
        email: ''
      }
    }
  },
  methods: {
    submit() {
      const schema = {
        name: [{ required: true, field: 'Name' }],
        email: [
          { required: true, field: 'Email' },
          { email: true }
        ]
      }

      if (this.$validate(this.form, schema)) {
        // Form is valid
        console.log('Submitting:', this.form)
      }
    }
  }
}
</script>
```

## Example 5: Analytics Plugin

```javascript
// plugins/analytics.js
const AnalyticsPlugin = {
  install(Vue, options = {}) {
    const tracker = options.tracker || window.gtag

    const analytics = {
      track(event, params = {}) {
        if (typeof tracker === 'function') {
          tracker('event', event, params)
        }
        console.log('[Analytics]', event, params)
      },

      pageView(path, title) {
        if (typeof tracker === 'function') {
          tracker('config', options.trackingId, {
            page_path: path,
            page_title: title
          })
        }
        console.log('[Analytics] Page View:', path, title)
      },

      setUserId(userId) {
        if (typeof tracker === 'function') {
          tracker('set', 'user_id', userId)
        }
      }
    }

    Vue.prototype.$analytics = analytics

    // Auto track page views
    if (options.autoTrack !== false) {
      Vue.mixin({
        watch: {
          '$route'(to) {
            this.$analytics.pageView(to.path, to.meta.title || document.title)
          }
        }
      })
    }
  }
}

export default AnalyticsPlugin
```

## Multiple Plugins Installation

```javascript
// main.js
import Vue from 'vue'
import Router from 'vue-router'
import Store from './store'
import HttpPlugin from './plugins/http'
import ToastPlugin from './plugins/toast'
import AnalyticsPlugin from './plugins/analytics'

Vue.use(Router)
Vue.use(Store)

Vue.use(HttpPlugin, {
  baseURL: process.env.VUE_APP_API_URL
})

Vue.use(ToastPlugin)

Vue.use(AnalyticsPlugin, {
  trackingId: process.env.VUE_APP_GA_ID,
  autoTrack: true
})
```

## Plugin with Component Registration

```javascript
// plugins/ui-components.js
import Button from './components/ui/Button.vue'
import Input from './components/ui/Input.vue'
import Modal from './components/ui/Modal.vue'

const UIPlugin = {
  install(Vue, options = {}) {
    const prefix = options.prefix || ''

    Vue.component(`${prefix}Button`, Button)
    Vue.component(`${prefix}Input`, Input)
    Vue.component(`${prefix}Modal`, Modal)
  }
}

export default UIPlugin
```

## Plugin Best Practices

1. **Check if already installed** to prevent double installation:

```javascript
const MyPlugin = {
  install(Vue, options) {
    if (Vue.prototype.$myPluginInstalled) return
    Vue.prototype.$myPluginInstalled = true

    // Plugin logic...
  }
}
```

2. **Merge options with defaults**:

```javascript
const MyPlugin = {
  install(Vue, options = {}) {
    const defaults = {
      timeout: 5000,
      position: 'top-right'
    }
    const settings = { ...defaults, ...options }

    // Use settings...
  }
}
```

3. **Provide clean uninstall** (if needed):

```javascript
const MyPlugin = {
  install(Vue, options) {
    Vue.prototype.$myMethod = function () {}

    // Store for potential cleanup
    Vue._myPluginInstalled = true
  },

  uninstall(Vue) {
    delete Vue.prototype.$myMethod
    delete Vue._myPluginInstalled
  }
}
```

## Reference

- [Vue 2 Guide - Writing a Plugin](https://vue2.docs.vuejs.org/v2/guide/plugins.html)
- [Vue 2 API - Vue.use](https://vue2.docs.vuejs.org/v2/api/#Vue-use)
