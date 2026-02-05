---
title: Using Vue.nextTick for DOM Updates
impact: HIGH
impactDescription: Vue's DOM updates are asynchronous; nextTick ensures code runs after the DOM has been updated
type: capability
tags:
  - vue2.6.14
  - nexttick
  - dom-updates
  - asynchronous
  - $nextTick
---

# Using Vue.nextTick for DOM Updates

Impact: HIGH - Vue performs DOM updates asynchronously. If you need to access the DOM immediately after a data change, you must use `Vue.nextTick()` or `this.$nextTick()` to wait for the update cycle to complete.

## Problem

Vue batches DOM updates for performance. When you change data, the DOM doesn't update immediately. Code that runs right after a data change will see the old DOM state, leading to bugs when trying to manipulate newly rendered elements.

## Task Checklist

- [ ] Use `this.$nextTick()` inside components for DOM-dependent operations
- [ ] Use `Vue.nextTick()` globally (outside components)
- [ ] Always use nextTick before accessing DOM after data changes
- [ ] Use Promise syntax with nextTick (Vue 2.1+) for cleaner async code

## Incorrect: Accessing DOM Without Waiting

```vue
<template>
  <div>
    <div ref="messageBox">{{ message }}</div>
    <button @click="updateMessage">Update</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: 'Hello'
    }
  },
  methods: {
    updateMessage() {
      this.message = 'Updated!'

      // WRONG: DOM hasn't updated yet
      const height = this.$refs.messageBox.offsetHeight
      console.log('Height:', height) // Old height!
    }
  }
}
</script>
```

## Correct: Using $nextTick

```vue
<template>
  <div>
    <div ref="messageBox">{{ message }}</div>
    <button @click="updateMessage">Update</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: 'Hello'
    }
  },
  methods: {
    updateMessage() {
      this.message = 'Updated!'

      // CORRECT: Wait for DOM update
      this.$nextTick(() => {
        const height = this.$refs.messageBox.offsetHeight
        console.log('Height:', height) // New height!
      })
    }
  }
}
</script>
```

## Correct: Promise Syntax (Vue 2.1+)

```javascript
export default {
  methods: {
    async updateMessage() {
      this.message = 'Updated!'

      // CORRECT: Use await with Promise
      await this.$nextTick()
      const height = this.$refs.messageBox.offsetHeight
      console.log('Height:', height) // New height!
    },

    async processData() {
      this.items = await fetchItems()
      await this.$nextTick()
      this.scrollToBottom()
    }
  }
}
```

## Common Use Cases

### 1. After Modifying Data

```javascript
export default {
  data() {
    return {
      items: [],
      isLoading: false
    }
  },
  methods: {
    async loadItems() {
      this.isLoading = true
      this.items = await fetchItems()

      // Wait for list to render
      await this.$nextTick()

      // Now scroll to bottom
      this.$refs.listContainer.scrollTop = this.$refs.listContainer.scrollHeight

      this.isLoading = false
    }
  }
}
```

### 2. After Conditional Rendering (v-if)

```vue
<template>
  <div>
    <button @click="showDetails = true">Show Details</button>
    <div v-if="showDetails" ref="details">
      {{ detailsContent }}
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      showDetails: false,
      detailsContent: 'Detailed information'
    }
  },
  methods: {
    async showDetailsAndFocus() {
      this.showDetails = true

      // Must wait for v-if to render the element
      await this.$nextTick()

      // Now the element exists
      this.$refs.details.scrollIntoView({ behavior: 'smooth' })
    }
  }
}
</script>
```

### 3. After v-for List Changes

```vue
<template>
  <div>
    <button @click="addItem">Add Item</button>
    <div ref="item0" v-for="(item, index) in items" :key="item.id">
      {{ item.name }}
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: []
    }
  },
  methods: {
    async addItem() {
      this.items.push({ id: Date.now(), name: 'New Item' })

      // Wait for new element to be rendered
      await this.$nextTick()

      // Now can access the new element
      const newRef = this.$refs[`item${this.items.length - 1}`]
      if (newRef) {
        newRef.classList.add('highlight')
      }
    }
  }
}
</script>
```

### 4. Third-Party Library Initialization

```vue
<template>
  <div>
    <button @click="showChart = true">Show Chart</button>
    <canvas v-show="showChart" ref="chartCanvas"></canvas>
  </div>
</template>

<script>
import Chart from 'chart.js'

export default {
  data() {
    return {
      showChart: false,
      chartData: [12, 19, 3, 5]
    }
  },
  methods: {
    async initChart() {
      this.showChart = true

      // Wait for canvas to exist in DOM
      await this.$nextTick()

      // Now initialize Chart.js
      this.chart = new Chart(this.$refs.chartCanvas, {
        type: 'bar',
        data: {
          labels: ['Red', 'Blue', 'Yellow', 'Green'],
          datasets: [{
            label: 'Dataset',
            data: this.chartData
          }]
        }
      })
    }
  },
  beforeDestroy() {
    if (this.chart) {
      this.chart.destroy()
    }
  }
}
</script>
```

## Vue.nextTick vs this.$nextTick

```javascript
// Global usage (outside component)
import Vue from 'vue'

Vue.nextTick(() => {
  console.log('DOM updated')
})

// As Promise (Vue 2.1+)
Vue.nextTick().then(() => {
  console.log('DOM updated')
})

// Inside component
export default {
  methods: {
    updateAndLog() {
      this.counter++
      this.$nextTick(() => {
        // `this` is bound to component instance
        console.log(this.counter)
      })
    }
  }
}
```

## Multiple DOM Updates

```javascript
export default {
  methods: {
    async makeMultipleUpdates() {
      // First update
      this.firstName = 'John'

      // Wait for first update
      await this.$nextTick()

      // Second update
      this.lastName = 'Doe'

      // Wait for second update
      await this.$nextTick()

      // Both updates are complete
      this.validateForm()
    }
  }
}
```

## in Mounted Hook

```javascript
export default {
  data() {
    return {
      content: 'Initial content'
    }
  },
  mounted() {
    // DOM is already mounted when mounted() is called
    // No need for nextTick if not modifying data first
    this.initPlugin()

    // BUT: if you need to wait for child components
    this.$nextTick(() => {
      // All child components have mounted
      this.initParentPlugin()
    })
  }
}
```

## Error Handling with nextTick

```javascript
export default {
  methods: {
    async updateWithErrorHandling() {
      try {
        this.data = await fetchData()
        await this.$nextTick()
        this.initializePlugin()
      } catch (error) {
        console.error('Failed to update:', error)
      }
    }
  }
}
```

## Performance Considerations

```javascript
export default {
  methods: {
    // BAD: Multiple DOM reads cause reflows
    measureElementsBad() {
      this.items.push(newItem)
      const height1 = this.$refs.container.offsetHeight
      const height2 = this.$refs.container.scrollHeight
      // Two reflows!
    },

    // GOOD: Batch DOM operations after nextTick
    measureElementsGood() {
      this.items.push(newItem)
      this.$nextTick(() => {
        const height1 = this.$refs.container.offsetHeight
        const height2 = this.$refs.container.scrollHeight
        // Only one reflow
      })
    }
  }
}
```

## Common Pitfalls

### Pitfall 1: Forgetting nextTick with refs

```javascript
// WRONG
toggleAndScroll() {
  this.isVisible = !this.isVisible
  this.$refs.panel.scrollTop = 0 // Element doesn't exist yet!
}

// CORRECT
toggleAndScroll() {
  this.isVisible = !this.isVisible
  this.$nextTick(() => {
    this.$refs.panel.scrollTop = 0
  })
}
```

### Pitfall 2: Assuming synchronous updates

```javascript
// WRONG
updateAndMeasure() {
  this.width = 100
  const actualWidth = this.$refs.element.offsetWidth // Still old width!
}

// CORRECT
updateAndMeasure() {
  this.width = 100
  this.$nextTick(() => {
    const actualWidth = this.$refs.element.offsetWidth
  })
}
```

## When NOT to Use nextTick

```javascript
export default {
  // NOT needed in computed properties (they're reactive)
  computed: {
    fullName() {
      return `${this.firstName} ${this.lastName}`
    }
  },

  // NOT needed in most watchers (they fire after updates)
  watch: {
    items(newVal) {
      // Already has the new value
      console.log('Items updated:', newVal)
    }
  }
}
```

## Summary

| Scenario | Need nextTick? |
|----------|----------------|
| Reading DOM after data change | ✅ Yes |
| Accessing refs after v-if change | ✅ Yes |
| Accessing refs after v-for change | ✅ Yes |
| Initializing 3rd party library | ✅ Yes |
| In computed properties | ❌ No |
| In watchers (usually) | ❌ No |
| In mounted (without data change) | ❌ No |

## Reference

- [Vue 2 API - Vue.nextTick](https://vue2.docs.vuejs.org/v2/api/#Vue-nextTick)
- [Vue 2 Guide - Async Update Queue](https://vue2.docs.vuejs.org/v2/guide/reactivity.html#Async-Update-Queue)
