---
title: Ref vs Reactive in Composition API for Vue 2
impact: HIGH
impactDescription: Understanding when to use ref vs reactive for reactive state
type: best-practice
tags:
  - vue2.6.14
  - composition-api
  - ref
  - reactive
  - reactivity
---

# Ref vs Reactive in Composition API for Vue 2

Impact: HIGH - Understanding the difference between `ref()` and `reactive()` is crucial for writing correct Composition API code in Vue 2.6.14.

## Task Checklist

- [ ] Use `ref()` for primitive values (string, number, boolean)
- [ ] Use `reactive()` for objects and arrays
- [ ] Access `ref` values with `.value` in JavaScript
- [ ] Use `toRefs()` to destructure reactive objects
- [ ] Use `readonly()` to prevent modifications

## Ref for Primitive Values

```javascript
import { ref } from '@vue/composition-api'

export default {
  setup() {
    // ✅ CORRECT: Use ref for primitives
    const count = ref(0)
    const message = ref('Hello')
    const isVisible = ref(true)

    // Access with .value in JavaScript
    const increment = () => {
      count.value++
    }

    const updateMessage = (newMessage) => {
      message.value = newMessage
    }

    const toggle = () => {
      isVisible.value = !isVisible.value
    }

    return {
      count,
      message,
      isVisible,
      increment,
      updateMessage,
      toggle
    }
  }
}
```

## Reactive for Objects

```javascript
import { reactive } from '@vue/composition-api'

export default {
  setup() {
    // ✅ CORRECT: Use reactive for objects
    const user = reactive({
      name: 'John',
      age: 30,
      email: 'john@example.com',
      address: {
        city: 'New York',
        country: 'USA'
      }
    })

    // Direct access, no .value needed
    const updateName = (name) => {
      user.name = name
    }

    const updateCity = (city) => {
      user.address.city = city
    }

    return {
      user,
      updateName,
      updateCity
    }
  }
}
```

## Ref vs Reactive - Key Differences

```javascript
import { ref, reactive } from '@vue/composition-api'

export default {
  setup() {
    // Ref - needs .value
    const count = ref(0)
    console.log(count.value) // 0
    count.value++

    // Reactive - direct access
    const state = reactive({ count: 0 })
    console.log(state.count) // 0
    state.count++

    // Ref for single object
    const userRef = ref({ name: 'John' })
    console.log(userRef.value.name) // Need .value
    userRef.value.name = 'Jane'

    // Reactive for object
    const userReactive = reactive({ name: 'John' })
    console.log(userReactive.name) // No .value
    userReactive.name = 'Jane'

    return {
      count,
      state,
      userRef,
      userReactive
    }
  }
}
```

## Ref with Objects

```javascript
import { ref } from '@vue/composition-api'

export default {
  setup() {
    // You CAN use ref for objects (replaces entire object)
    const user = ref({
      name: 'John',
      age: 30
    })

    // Must use .value to access
    const updateName = (name) => {
      user.value.name = name
    }

    // Replace entire object
    const resetUser = () => {
      user.value = {
        name: 'Guest',
        age: 0
      }
    }

    return {
      user,
      updateName,
      resetUser
    }
  }
}
```

## Reactive Loses Reactivity When Destructured

```javascript
import { reactive } from '@vue/composition-api'

export default {
  setup() {
    const state = reactive({
      count: 0,
      message: 'Hello'
    })

    // ❌ WRONG: Loses reactivity
    // const { count, message } = state

    // ✅ CORRECT: Use toRefs
    const count = state.count
    const message = state.message

    // count and message are NOT reactive here!

    const increment = () => {
      count++ // Won't trigger reactivity!
    }

    return {
      count,
      message,
      increment
    }
  }
}
```

## Using toRefs to Maintain Reactivity

```javascript
import { reactive, toRefs } from '@vue/composition-api'

export default {
  setup() {
    const state = reactive({
      count: 0,
      message: 'Hello',
      user: {
        name: 'John'
      }
    })

    // ✅ CORRECT: toRefs maintains reactivity
    const { count, message, user } = toRefs(state)

    // These are refs now, use .value
    const increment = () => {
      count.value++
    }

    const updateMessage = (msg) => {
      message.value = msg
    }

    // Nested object is still reactive
    const updateUserName = (name) => {
      user.value.name = name
    }

    return {
      count,
      message,
      user,
      increment,
      updateMessage,
      updateUserName
    }
  }
}
```

## Computed with Ref and Reactive

```javascript
import { ref, reactive, computed } from '@vue/composition-api'

export default {
  setup() {
    // Ref with computed
    const count = ref(0)
    const doubleCount = computed(() => count.value * 2)

    // Reactive with computed
    const state = reactive({
      firstName: 'John',
      lastName: 'Doe'
    })

    const fullName = computed(() => {
      return `${state.firstName} ${state.lastName}`
    })

    // Writable computed
    const fullNameWritable = computed({
      get: () => `${state.firstName} ${state.lastName}`,
      set: (value) => {
        [state.firstName, state.lastName] = value.split(' ')
      }
    })

    return {
      count,
      doubleCount,
      state,
      fullName,
      fullNameWritable
    }
  }
}
```

## Watch Ref vs Reactive

```javascript
import { ref, reactive, watch } from '@vue/composition-api'

export default {
  setup() {
    // Watch ref
    const count = ref(0)

    watch(count, (newVal, oldVal) => {
      console.log(`Count: ${oldVal} -> ${newVal}`)
    })

    // Watch reactive property (needs function)
    const state = reactive({
      count: 0,
      message: 'Hello'
    })

    watch(
      () => state.count, // Must return the property
      (newVal, oldVal) => {
        console.log(`State.count: ${oldVal} -> ${newVal}`)
      }
    )

    // Watch multiple refs
    const firstName = ref('John')
    const lastName = ref('Doe')

    watch(
      [firstName, lastName],
      ([newFirst, newLast], [oldFirst, oldLast]) => {
        console.log('Name changed:', newFirst, newLast)
      }
    )

    return {
      count,
      state,
      firstName,
      lastName
    }
  }
}
```

## Readonly

```javascript
import { ref, reactive, readonly } from '@vue/composition-api'

export default {
  setup() {
    // Readonly ref
    const count = ref(0)
    const readonlyCount = readonly(count)

    // Can't modify readonly
    // readonlyCount.value++ // Warning: Set operation on key "value" failed

    // But original can be modified
    const increment = () => {
      count.value++
      // readonlyCount.value also updates
    }

    // Readonly reactive
    const config = readonly(reactive({
      apiUrl: 'https://api.example.com',
      timeout: 5000
    }))

    // Can't modify
    // config.apiUrl = '...' // Warning

    return {
      readonlyCount,
      config,
      increment
    }
  }
}
```

## Array Reactivity

```javascript
import { ref, reactive } from '@vue/composition-api'

export default {
  setup() {
    // Ref for array
    const itemsRef = ref([1, 2, 3])

    const addItemRef = (item) => {
      itemsRef.value.push(item) // Need .value
    }

    // Reactive for array
    const itemsReactive = reactive([1, 2, 3])

    const addItemReactive = (item) => {
      itemsReactive.push(item) // Direct access
    }

    // Array operations with ref
    const filterItems = () => {
      itemsRef.value = itemsRef.value.filter(x => x > 1)
    }

    return {
      itemsRef,
      itemsReactive,
      addItemRef,
      addItemReactive,
      filterItems
    }
  }
}
```

## Best Practices

```javascript
import { ref, reactive, toRefs, computed } from '@vue/composition-api'

export default {
  setup() {
    // ✅ DO: Use ref for primitives
    const count = ref(0)
    const message = ref('Hello')

    // ✅ DO: Use reactive for objects
    const user = reactive({
      name: 'John',
      age: 30
    })

    // ✅ DO: Use ref when you need to replace entire object
    const config = ref({
      theme: 'light',
      lang: 'en'
    })

    // ✅ DO: Use toRefs when returning reactive object
    const state = reactive({
      count: 0,
      loading: false
    })
    const { count: stateCount, loading } = toRefs(state)

    // ✅ DO: Use reactive for form data
    const form = reactive({
      username: '',
      password: '',
      email: ''
    })

    return {
      count,
      message,
      user,
      config,
      stateCount,
      loading,
      form
    }
  }
}
```

## When to Use What

| Scenario | Use | Why |
|----------|-----|-----|
| Primitive value | `ref()` | Primitives need wrapper |
| Single object to replace | `ref()` | Easy replacement |
| Object with nested props | `reactive()` | Direct access no .value |
| Form data | `reactive()` | Clean syntax |
| Array | `ref()` or `reactive()` | Both work |
| Computed source | Either | Depends on use case |
| Destructuring needed | `ref()` + `toRefs()` | Maintains reactivity |
| Readonly needed | `readonly()` | Prevents modification |

## Common Pitfalls

```javascript
import { ref, reactive, toRefs } from '@vue/composition-api'

export default {
  setup() {
    // ❌ WRONG: Forgetting .value on ref
    const count = ref(0)
    // console.log(count) // Wrong: prints RefImpl
    console.log(count.value) // Correct: prints 0

    // ❌ WRONG: Destructuring reactive loses reactivity
    const state = reactive({ count: 0 })
    // const { count } = state // Not reactive

    // ✅ CORRECT: Use toRefs
    const { count: stateCount } = toRefs(state)

    // ❌ WRONG: Reassigning reactive object loses reactivity
    let user = reactive({ name: 'John' })
    // user = reactive({ name: 'Jane' }) // Loses original reactivity

    // ✅ CORRECT: Use ref for replacement
    const userRef = reactive({ name: 'John' })
    userRef = reactive({ name: 'Jane' })

    return {}
  }
}
```

## Reference

- [@vue/composition-api Reactivity](https://github.com/vuejs/composition-api#reactivity-api)
