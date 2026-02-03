---
title: Array Length Changes Not Detected in Vue 2
impact: HIGH
impactDescription: Vue 2's Object.defineProperty cannot detect direct array length modifications
type: gotcha
tags:
  - vue2
  - reactivity
  - arrays
  - array-length
  - limitations
---

# Array Length Changes Not Detected in Vue 2

Impact: HIGH - Direct array length modifications are not reactive in Vue 2 due to Object.defineProperty limitations. Always use array methods or $set for length changes.

## Task Checklist

- [ ] Use `splice()` to truncate arrays
- [ ] Use `push()` with spread for appending multiple items
- [ ] Never assign directly to `array.length`
- [ ] Create new array reference when needed

## Problem: Direct Length Assignment

Vue 2 cannot detect when you directly modify an array's length property:

```javascript
// ❌ WRONG: Not reactive
export default {
  data() {
    return {
      items: ['a', 'b', 'c', 'd', 'e']
    }
  },
  methods: {
    truncateItems() {
      // Direct length assignment - NOT REACTIVE
      this.items.length = 2
      console.log(this.items.length) // 2, but view won't update
    }
  }
}
```

## Solution: Use Array Methods

```javascript
// ✅ CORRECT: Use splice() to truncate
export default {
  data() {
    return {
      items: ['a', 'b', 'c', 'd', 'e']
    }
  },
  methods: {
    truncateItems() {
      // splice() IS reactive
      this.items.splice(2)
      // items is now ['a', 'b']
    }
  }
}
```

## Clearing Arrays

```javascript
// ❌ WRONG
this.items.length = 0

// ✅ CORRECT: Method 1 - splice
this.items.splice(0)

// ✅ CORRECT: Method 2 - empty array
this.items = []

// ✅ CORRECT: Method 3 - splice with length
this.items.splice(0, this.items.length)
```

## Applying Multiple Items

```javascript
// When adding multiple items to array

// ❌ WRONG: Not reactive
this.items[this.items.length] = newItem

// ✅ CORRECT: Use push
this.items.push(newItem)

// ✅ CORRECT: Use spread with push
this.items.push(...newItems)
```

## Truncating to Specific Length

```javascript
export default {
  data() {
    return {
      items: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    }
  },
  methods: {
    // Keep first N items
    keepFirst(n) {
      this.items.splice(n)
    },

    // Keep last N items
    keepLast(n) {
      this.items.splice(0, this.items.length - n)
    },

    // Pagination
    setPage(page, pageSize) {
      const start = page * pageSize
      this.items.splice(0, start)
      this.items.splice(pageSize)
    }
  }
}
```

## Array Reference Replacement

```javascript
// Sometimes creating a new array reference is clearest

export default {
  methods: {
    // Using slice to create new array
    truncate(length) {
      this.items = this.items.slice(0, length)
    },

    // Using filter
    removeItems(condition) {
      this.items = this.items.filter(condition)
    },

    // Using map
    transformItems(mapper) {
      this.items = this.items.map(mapper)
    }
  }
}
```

## Vue.set with Array Index

While Vue.set works for adding properties to objects, for arrays it's better to use native array methods:

```javascript
// This works but is not idiomatic
this.$set(this.items, this.items.length, newItem)

// Better: use push
this.items.push(newItem)

// For specific index assignment (replacing)
this.$set(this.items, index, newValue)
```

## Reactive Array Utility Methods

```javascript
export default {
  methods: {
    // Safe truncate with reactive update
    safeTruncate(newLength) {
      if (newLength < this.items.length) {
        this.items.splice(newLength)
      }
    },

    // Ensure array has at most N items
    maxItems(max) {
      if (this.items.length > max) {
        this.items.splice(max)
      }
    },

    // Pad array to minimum length
    padTo(min, fillValue = null) {
      while (this.items.length < min) {
        this.items.push(fillValue)
      }
    }
  }
}
```

## Watch Array Length

```javascript
// Watch for length changes
export default {
  data() {
    return {
      items: []
    }
  },
  watch: {
    // Watch the array
    items: {
      handler(newItems) {
        console.log('Array length:', newItems.length)
      },
      deep: true
    },

    // Or watch just the length
    'items.length': {
      handler(newLength, oldLength) {
        console.log(`Length changed from ${oldLength} to ${newLength}`)
      }
    }
  }
}
```

## Common Patterns

```javascript
export default {
  methods: {
    // Pagination
    getPage(page, pageSize) {
      const start = page * pageSize
      const end = start + pageSize
      return this.allItems.slice(start, end)
    },

    // Infinite scroll - add more items
    loadMore() {
      const newItems = fetchMoreItems()
      this.items.push(...newItems)
    },

    // Search result limiting
    onSearch(query) {
      const results = search(query)
      this.items = results.slice(0, 100) // Limit to 100 results
    }
  }
}
```

## Summary Table

| Operation | Wrong (Not Reactive) | Correct (Reactive) |
|-----------|---------------------|-------------------|
| Truncate | `arr.length = n` | `arr.splice(n)` |
| Clear | `arr.length = 0` | `arr.splice(0)` or `arr = []` |
| Remove first N | `arr.length = arr.length - n` | `arr.splice(0, n)` |
| Add to end | `arr[arr.length] = x` | `arr.push(x)` |
| Set specific index | `arr[i] = x` | `this.$set(arr, i, x)` |

## Reference

- [Vue 2 Reactivity - Change Detection Caveats](https://v2.vuejs.org/v2/guide/reactivity.html#Change-Detection-Caveats)
- [Vue 2 List Rendering - Array Mutation Methods](https://v2.vuejs.org/v2/guide/list.html#Mutation-Methods)
