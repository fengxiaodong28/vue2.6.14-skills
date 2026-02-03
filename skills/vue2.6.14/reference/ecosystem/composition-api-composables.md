---
title: Composables in Composition API for Vue 2
impact: MEDIUM
impactDescription: Creating reusable logic with composables in Vue 2.6.14
type: best-practice
tags:
  - vue2.6.14
  - composition-api
  - composables
  - reusable-logic
  - code-organization
---

# Composables in Composition API for Vue 2

Impact: MEDIUM - Composables enable reusable logic extraction and better code organization in Vue 2.6.14 Composition API.

## Task Checklist

- [ ] Create composable functions that return reactive state
- [ ] Use ref/reactive for internal state
- [ ] Return methods and computed properties
- [ ] Accept parameters for configuration
- [ ] Clean up side effects in onUnmounted

## Basic Composable Pattern

```javascript
// composables/useCounter.js
import { ref, computed } from '@vue/composition-api'

export function useCounter(initialValue = 0) {
  // Internal state
  const count = ref(initialValue)

  // Computed
  const double = computed(() => count.value * 2)
  const triple = computed(() => count.value * 3)

  // Methods
  const increment = () => { count.value++ }
  const decrement = () => { count.value-- }
  const reset = () => { count.value = initialValue }
  const setValue = (value) => { count.value = value }

  // Return public API
  return {
    count,
    double,
    triple,
    increment,
    decrement,
    reset,
    setValue
  }
}
```

```vue
<!-- Usage in component -->
<template>
  <div>
    <p>Count: {{ count }}</p>
    <p>Double: {{ double }}</p>
    <button @click="increment">+</button>
    <button @click="decrement">-</button>
    <button @click="reset">Reset</button>
  </div>
</template>

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

## Async Data Fetching Composable

```javascript
// composables/useFetch.js
import { ref, onMounted } from '@vue/composition-api'

export function useFetch(url, options = {}) {
  const data = ref(null)
  const error = ref(null)
  const loading = ref(false)

  const fetch = async () => {
    loading.value = true
    error.value = null

    try {
      const response = await fetch(url, options)
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`)
      }
      data.value = await response.json()
    } catch (err) {
      error.value = err.message
    } finally {
      loading.value = false
    }
  }

  // Auto-fetch on mount
  onMounted(() => {
    fetch()
  })

  return {
    data,
    error,
    loading,
    fetch,
    // Helper to refetch
    refetch: fetch
  }
}
```

```vue
<script>
import { useFetch } from '@/composables/useFetch'

export default {
  setup() {
    const { data, error, loading, refetch } = useFetch('/api/users/1')

    return {
      user: data,
      error,
      loading,
      refetch
    }
  }
}
</script>
```

## Local Storage Composable

```javascript
// composables/useStorage.js
import { ref, watch, onMounted } from '@vue/composition-api'

export function useStorage(key, defaultValue = null) {
  // Load from storage
  const storedValue = localStorage.getItem(key)
  const value = ref(storedValue ? JSON.parse(storedValue) : defaultValue)

  // Save to storage whenever value changes
  watch(
    value,
    (newValue) => {
      if (newValue === null || newValue === undefined) {
        localStorage.removeItem(key)
      } else {
        localStorage.setItem(key, JSON.stringify(newValue))
      }
    },
    { deep: true }
  )

  return value
}
```

```vue
<script>
import { useStorage } from '@/composables/useStorage'

export default {
  setup() {
    const theme = useStorage('theme', 'light')
    const userPreferences = useStorage('preferences', {
      notifications: true,
      language: 'en'
    })

    return {
      theme,
      userPreferences
    }
  }
}
</script>
```

## Mouse Tracking Composable

```javascript
// composables/useMouse.js
import { ref, onMounted, onUnmounted } from '@vue/composition-api'

export function useMouse() {
  const x = ref(0)
  const y = ref(0)

  const update = (event) => {
    x.value = event.pageX
    y.value = event.pageY
  }

  onMounted(() => {
    window.addEventListener('mousemove', update)
  })

  onUnmounted(() => {
    window.removeEventListener('mousemove', update)
  })

  return { x, y }
}
```

## Window Size Composable

```javascript
// composables/useWindowSize.js
import { ref, onMounted, onUnmounted } from '@vue/composition-api'

export function useWindowSize() {
  const width = ref(window.innerWidth)
  const height = ref(window.innerHeight)

  const update = () => {
    width.value = window.innerWidth
    height.value = window.innerHeight
  }

  onMounted(() => {
    window.addEventListener('resize', update)
  })

  onUnmounted(() => {
    window.removeEventListener('resize', update)
  })

  // Computed breakpoints
  const isMobile = computed(() => width.value < 768)
  const isTablet = computed(() => width.value >= 768 && width.value < 1024)
  const isDesktop = computed(() => width.value >= 1024)

  return {
    width,
    height,
    isMobile,
    isTablet,
    isDesktop
  }
}
```

## Debounce/Throttle Composable

```javascript
// composables/useDebounce.js
import { ref, watch } from '@vue/composition-api'

export function useDebounce(value, delay = 300) {
  const debouncedValue = ref(value.value)
  let timeout = null

  watch(
    value,
    (newValue) => {
      clearTimeout(timeout)
      timeout = setTimeout(() => {
        debouncedValue.value = newValue
      }, delay)
    },
    { immediate: true }
  )

  return debouncedValue
}

// composables/useThrottle.js
export function useThrottle(value, limit = 300) {
  const throttledValue = ref(value.value)
  let inThrottle = false
  let lastValue = value.value

  watch(
    value,
    (newValue) => {
      lastValue = newValue
      if (!inThrottle) {
        throttledValue.value = newValue
        inThrottle = true
        setTimeout(() => {
          inThrottle = false
          throttledValue.value = lastValue
        }, limit)
      }
    },
    { immediate: true }
  )

  return throttledValue
}
```

## Form Validation Composable

```javascript
// composables/useForm.js
import { ref, computed } from '@vue/composition-api'

export function useForm(initialValues, validationRules = {}) {
  const values = ref({ ...initialValues })
  const errors = ref({})
  const touched = ref({})

  const validate = (field) => {
    if (!validationRules[field]) {
      return true
    }

    const rules = validationRules[field]
    const value = values.value[field]

    for (const rule of rules) {
      if (rule.required && !value) {
        errors.value[field] = rule.message || 'This field is required'
        return false
      }
      if (rule.pattern && !rule.pattern.test(value)) {
        errors.value[field] = rule.message || 'Invalid format'
        return false
      }
      if (rule.validator && !rule.validator(value)) {
        errors.value[field] = rule.message || 'Validation failed'
        return false
      }
    }

    delete errors.value[field]
    return true
  }

  const validateAll = () => {
    let isValid = true
    for (const field of Object.keys(validationRules)) {
      if (!validate(field)) {
        isValid = false
      }
    }
    return isValid
  }

  const setFieldValue = (field, value) => {
    values.value[field] = value
    touched.value[field] = true
    validate(field)
  }

  const reset = () => {
    values.value = { ...initialValues }
    errors.value = {}
    touched.value = {}
  }

  const isValid = computed(() => {
    return Object.keys(errors.value).length === 0
  })

  return {
    values,
    errors,
    touched,
    isValid,
    setFieldValue,
    validate,
    validateAll,
    reset
  }
}
```

```vue
<script>
import { useForm } from '@/composables/useForm'

export default {
  setup() {
    const { values, errors, isValid, setFieldValue, validateAll, reset } = useForm(
      {
        email: '',
        password: ''
      },
      {
        email: [
          { required: true, message: 'Email is required' },
          { pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/, message: 'Invalid email' }
        ],
        password: [
          { required: true, message: 'Password is required' },
          { validator: (v) => v.length >= 6, message: 'Password must be at least 6 characters' }
        ]
      }
    )

    const submit = () => {
      if (validateAll()) {
        console.log('Form submitted:', values.value)
      }
    }

    return {
      values,
      errors,
      isValid,
      setFieldValue,
      submit,
      reset
    }
  }
}
</script>
```

## Pagination Composable

```javascript
// composables/usePagination.js
import { ref, computed } from '@vue/composition-api'

export function usePagination(initialPage = 1, pageSize = 10) {
  const currentPage = ref(initialPage)
  const itemsPerPage = ref(pageSize)
  const totalItems = ref(0)

  const totalPages = computed(() => {
    return Math.ceil(totalItems.value / itemsPerPage.value)
  })

  const offset = computed(() => {
    return (currentPage.value - 1) * itemsPerPage.value
  })

  const hasNextPage = computed(() => currentPage.value < totalPages.value)
  const hasPrevPage = computed(() => currentPage.value > 1)

  const nextPage = () => {
    if (hasNextPage.value) {
      currentPage.value++
    }
  }

  const prevPage = () => {
    if (hasPrevPage.value) {
      currentPage.value--
    }
  }

  const goToPage = (page) => {
    if (page >= 1 && page <= totalPages.value) {
      currentPage.value = page
    }
  }

  const setTotal = (total) => {
    totalItems.value = total
  }

  const reset = () => {
    currentPage.value = 1
  }

  return {
    currentPage,
    itemsPerPage,
    totalItems,
    totalPages,
    offset,
    hasNextPage,
    hasPrevPage,
    nextPage,
    prevPage,
    goToPage,
    setTotal,
    reset
  }
}
```

## WebSocket Composable

```javascript
// composables/useWebSocket.js
import { ref, onUnmounted } from '@vue/composition-api'

export function useWebSocket(url) {
  const data = ref(null)
  const error = ref(null)
  const isConnected = ref(false)
  let socket = null

  const connect = () => {
    socket = new WebSocket(url)

    socket.onopen = () => {
      isConnected.value = true
      error.value = null
    }

    socket.onmessage = (event) => {
      data.value = JSON.parse(event.data)
    }

    socket.onclose = () => {
      isConnected.value = false
    }

    socket.onerror = (event) => {
      error.value = event.message
    }
  }

  const send = (message) => {
    if (socket && socket.readyState === WebSocket.OPEN) {
      socket.send(JSON.stringify(message))
    }
  }

  const disconnect = () => {
    if (socket) {
      socket.close()
    }
  }

  onUnmounted(() => {
    disconnect()
  })

  return {
    data,
    error,
    isConnected,
    connect,
    send,
    disconnect
  }
}
```

## Composable Composition

```javascript
// composables/useUserManagement.js
import { ref } from '@vue/composition-api'
import { useFetch } from './useFetch'
import { useStorage } from './useStorage'

export function useUserManagement() {
  // Compose multiple composables
  const token = useStorage('authToken', '')
  const currentUser = ref(null)

  const { data: userData, loading, fetch: fetchUser, error } = useFetch()

  const login = async (credentials) => {
    const response = await fetch('/api/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(credentials)
    })

    if (response.ok) {
      const { token: newToken, user } = await response.json()
      token.value = newToken
      currentUser.value = user
    }
  }

  const logout = () => {
    token.value = ''
    currentUser.value = null
  }

  const fetchCurrentUser = async () => {
    await fetchUser('/api/user', {
      headers: {
        'Authorization': `Bearer ${token.value}`
      }
    })
    currentUser.value = userData.value
  }

  return {
    token,
    currentUser,
    loading,
    error,
    login,
    logout,
    fetchCurrentUser
  }
}
```

## Best Practices

```javascript
// ✅ DO: Return plain reactive values
export function useGood() {
  const count = ref(0)
  const increment = () => count.value++
  return { count, increment }
}

// ❌ DON'T: Return the entire component setup
export function useBad() {
  // Don't create full components here
  // Composables should be reusable functions
}

// ✅ DO: Accept parameters for configuration
export function useCounter(initialValue = 0, step = 1) {
  const count = ref(initialValue)
  const increment = () => count.value += step
  return { count, increment }
}

// ✅ DO: Clean up side effects
export function useInterval(callback, delay) {
  const interval = ref(null)

  onMounted(() => {
    interval.value = setInterval(callback, delay)
  })

  onUnmounted(() => {
    clearInterval(interval.value)
  })
}
```

## Reference

- [Composition API FAQ - Composables](https://vue-composition-api-rfc.netlify.app/#faq-composables)
