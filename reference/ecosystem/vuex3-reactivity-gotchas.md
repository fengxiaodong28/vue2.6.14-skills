---
title: Vuex Reactivity Gotchas in Vue 2
impact: HIGH
impactDescription: Vuex state in Vue 2 has same reactivity limitations as components
type: gotcha
tags:
  - vue2.6.14
  - vuex
  - reactivity
  - state-mutation
  - gotchas
---

# Vuex Reactivity Gotchas in Vue 2

Impact: HIGH - Vuex uses Vue 2's reactivity system, so all Vue 2 reactivity limitations apply to Vuex state mutations.

## Task Checklist

- [ ] Always mutate state through mutations
- [ ] Use `Vue.set` for adding new properties
- [ ] Use array mutation methods for array changes
- [ ] Initialize all state properties upfront
- [ ] Replace objects entirely for bulk updates

## Problem: Direct Property Addition

```javascript
// ❌ WRONG: Adding property directly in mutation
const store = new Vuex.Store({
  state: {
    user: {
      name: 'John'
    }
  },
  mutations: {
    addEmail(state) {
      state.user.email = 'john@example.com'
      // NOT REACTIVE! View won't update
    }
  }
})
```

```javascript
// ✅ CORRECT: Use Vue.set
mutations: {
  addEmail(state) {
    Vue.set(state.user, 'email', 'john@example.com')
  }
}

// ✅ ALSO CORRECT: Replace entire object
mutations: {
  addEmail(state) {
    state.user = {
      ...state.user,
      email: 'john@example.com'
    }
  }
}
```

## Problem: Array Index Assignment

```javascript
// ❌ WRONG: Direct array index assignment
mutations: {
  updateItem(state, { index, value }) {
    state.items[index] = value
    // NOT REACTIVE!
  }
}
```

```javascript
// ✅ CORRECT: Use Vue.set
mutations: {
  updateItem(state, { index, value }) {
    Vue.set(state.items, index, value)
  }
}

// ✅ ALSO CORRECT: Use splice
mutations: {
  updateItem(state, { index, value }) {
    state.items.splice(index, 1, value)
  }
}
```

## Problem: Array Length Changes

```javascript
// ❌ WRONG: Direct length assignment
mutations: {
  truncate(state, newLength) {
    state.items.length = newLength
    // NOT REACTIVE!
  }
}
```

```javascript
// ✅ CORRECT: Use splice
mutations: {
  truncate(state, newLength) {
    state.items.splice(newLength)
  }
}
```

## Problem: Deleting Properties

```javascript
// ❌ WRONG: Using delete operator
mutations: {
  removeUserField(state, field) {
    delete state.user[field]
    // NOT REACTIVE!
  }
}
```

```javascript
// ✅ CORRECT: Use Vue.delete
mutations: {
  removeUserField(state, field) {
    Vue.delete(state.user, field)
  }
}

// ✅ ALSO CORRECT: Create new object without property
mutations: {
  removeUserField(state, field) {
    const { [field]: removed, ...rest } = state.user
    state.user = rest
  }
}
```

## State Initialization

```javascript
// ✅ CORRECT: Initialize all possible properties
const store = new Vuex.Store({
  state: {
    user: {
      id: null,
      name: '',
      email: '',
      phone: '',
      address: null,
      preferences: {
        theme: 'light',
        notifications: true
      }
    },
    items: [],
    pagination: {
      page: 1,
      limit: 20,
      total: 0
    }
  }
})
```

## Bulk State Updates

```javascript
mutations: {
  // ❌ WRONG: Individual property updates
  updateUser(state, userData) {
    state.user.name = userData.name
    state.user.email = userData.email
    state.user.phone = userData.phone
    // Multiple reactive updates - inefficient
  },

  // ✅ CORRECT: Replace entire object
  updateUser(state, userData) {
    state.user = {
      ...state.user,
      ...userData
    }
  },

  // ✅ ALSO CORRECT: For nested objects
  updateUserPreferences(state, preferences) {
    state.user.preferences = {
      ...state.user.preferences,
      ...preferences
    }
  }
}
```

## Array Mutations in Vuex

```javascript
mutations: {
  // ✅ CORRECT: Use array mutation methods
  addItem(state, item) {
    state.items.push(item)
  },

  addItems(state, items) {
    state.items.push(...items)
  },

  removeItem(state, index) {
    state.items.splice(index, 1)
  },

  removeItemById(state, id) {
    const index = state.items.findIndex(item => item.id === id)
    if (index >= 0) {
      state.items.splice(index, 1)
    }
  },

  updateItem(state, { index, item }) {
    Vue.set(state.items, index, item)
  },

  clearItems(state) {
    state.items.splice(0) // Clear reactively
  },

  // Or replace array
  setItems(state, items) {
    state.items = items
  }
}
```

## Object Addition Patterns

```javascript
mutations: {
  // Adding to object/map
  addEntry(state, { key, value }) {
    Vue.set(state.entries, key, value)
  },

  // Merging objects
  mergeEntries(state, newEntries) {
    state.entries = {
      ...state.entries,
      ...newEntries
    }
  },

  // Deep update
  updateNested(state, { path, value }) {
    const keys = path.split('.')
    let target = state
    for (let i = 0; i < keys.length - 1; i++) {
      if (!target[keys[i]]) {
        Vue.set(target, keys[i], {})
      }
      target = target[keys[i]]
    }
    Vue.set(target, keys[keys.length - 1], value)
  }
}
```

## Getters and Reactivity

```javascript
getters: {
  // ✅ CORRECT: Getters are reactive and cached
  filteredItems(state) {
    return state.items.filter(item => item.active)
  },

  // ✅ CORRECT: Getter with argument (returns function)
  itemById: state => id => {
    return state.items.find(item => item.id === id)
  },

  // ✅ CORRECT: Using other getters
  activeItemsCount(state, getters) {
    return getters.filteredItems.length
  }
}
```

## Watch Gotchas

```javascript
// In component
watch: {
  // ❌ WRONG: Watching nested property directly
  'user.email': {
    handler(newEmail) {
      // May not trigger if email added reactively
    }
  },

  // ✅ CORRECT: Watch entire object
  user: {
    handler(newUser) {
      console.log('User changed:', newUser.email)
    },
    deep: true
  },

  // ✅ CORRECT: Use getter for specific property
  userEmail() {
    return this.$store.state.user.email
  },
  userEmail(newEmail) {
    console.log('Email changed:', newEmail)
  }
}
```

## Actions vs Mutations

```javascript
actions: {
  // ❌ WRONG: Don't mutate state in actions
  badAction(state) {
    state.user.name = 'New Name'
  },

  // ✅ CORRECT: Commit mutation
  goodAction({ commit }) {
    commit('SET_USER_NAME', 'New Name')
  },

  // ✅ CORRECT: Async action then commit
  async fetchUser({ commit }, userId) {
    const user = await api.getUser(userId)
    commit('SET_USER', user)
  }
}
```

## Module State Reactivity

```javascript
const moduleA = {
  state: {
    details: {}
  },
  mutations: {
    // ✅ Use Vue.set in modules too
    setDetail(state, { key, value }) {
      Vue.set(state.details, key, value)
    },

    // ✅ Or replace entire object
    setDetails(state, details) {
      state.details = {
        ...state.details,
        ...details
      }
    }
  }
}
```

## MapState Helper Gotchas

```vue
<script>
export default {
  computed: {
    // ✅ This works correctly
    ...mapState({
      user: state => state.user,
      itemCount: state => state.items.length
    }),

    // ❌ This creates non-reactive computed if not careful
    ...mapState({
      userEmail: state => {
        // Computed each time, but caching works
        return state.user.email
      }
    })
  }
}
</script>
```

## Vuex with TypeScript

```typescript
import { VuexModule, Module, Mutation, Action } from 'vuex-module-decorators'

@Module({ namespaced: true })
class UserModule extends VuexModule {
  user: User | null = null

  @Mutation
  setUser(user: User) {
    this.user = user
  }

  @Mutation
  updateUserField<K extends keyof User>(field: K, value: User[K]) {
    if (this.user) {
      Vue.set(this.user, field, value)
    }
  }

  get userName(): string {
    return this.user?.name || ''
  }
}
```

## Reactive State Updates Pattern

```javascript
// Utility for safe state updates
const updateState = (state, path, value) => {
  const keys = path.split('.')
  let target = state
  for (let i = 0; i < keys.length - 1; i++) {
    if (!target[keys[i]]) {
      Vue.set(target, keys[i], {})
    }
    target = target[keys[i]]
  }
  Vue.set(target, keys[keys.length - 1], value)
}

// Use in mutations
mutations: {
  updateNestedField(state, { path, value }) {
    updateState(state, path, value)
  }
}
```

## Common Gotchas Summary

| Gotcha | Wrong | Correct |
|--------|-------|---------|
| Add property | `obj.new = val` | `Vue.set(obj, 'new', val)` |
| Update array index | `arr[i] = val` | `Vue.set(arr, i, val)` |
| Delete property | `delete obj.key` | `Vue.delete(obj, 'key')` |
| Change array length | `arr.length = n` | `arr.splice(n)` |
| Nested updates | `obj.a.b = val` | `Vue.set(obj.a, 'b', val)` |
| Bulk updates | Multiple assignments | `obj = { ...obj, ...updates }` |

## Reference

- [Vuex 3 Guide - Reactivity](https://vuex.vuejs.org/guide/reactivity.html)
- [Vue 2 Reactivity - Change Detection](https://v2.vuejs.org/v2/guide/reactivity.html)
