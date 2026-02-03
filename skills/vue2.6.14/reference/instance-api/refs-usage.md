---
title: Template Refs with $refs in Vue 2
impact: MEDIUM
impactDescription: $refs provides direct access to DOM elements and child components
type: capability
tags:
  - vue2.6.14
  - refs
  - $refs
  - template-refs
  - dom-access
  - child-components
---

# Template Refs with $refs in Vue 2

Impact: MEDIUM - Vue 2's `$refs` provides direct access to DOM elements and child component instances. Use `ref` attributes in templates to register references, then access them via `this.$refs`. Avoid overuse as it breaks component encapsulation.

## Task Checklist

- [ ] Use `ref` attributes to register DOM elements or child components
- [ ] Access refs with `this.$refs` after component is mounted
- [ ] Remember refs are not reactive
- [ ] Use refs for DOM manipulation and child component method calls
- [ ] Prefer props and events over refs for parent-child communication

## Basic Usage

```vue
<template>
  <div>
    <!-- Register a ref on a DOM element -->
    <input ref="myInput" type="text">

    <!-- Register a ref on a component -->
    <child-component ref="myChild"></child-component>

    <button @click="focusInput">Focus Input</button>
    <button @click="callChildMethod">Call Child Method</button>
  </div>
</template>

<script>
import ChildComponent from './ChildComponent.vue'

export default {
  components: { ChildComponent },
  methods: {
    focusInput() {
      // Access DOM element directly
      this.$refs.myInput.focus()
    },
    callChildMethod() {
      // Access child component instance
      this.$refs.myChild.childMethod()
    }
  }
}
</script>
```

## Refs on DOM Elements

```vue
<template>
  <div>
    <!-- Text input -->
    <input ref="usernameInput" type="text" placeholder="Username">

    <!-- Textarea -->
    <textarea ref="messageInput" placeholder="Message"></textarea>

    <!-- Button -->
    <button ref="submitButton">Submit</button>

    <!-- Custom element -->
    <div ref="customElement" class="custom"></div>

    <button @click="handleFocus">Focus Inputs</button>
    <button @click="measureElement">Measure Element</button>
  </div>
</template>

<script>
export default {
  mounted() {
    // Refs are available after mounted()
    console.log(this.$refs.usernameInput) // <input> element
    console.log(this.$refs.messageInput)  // <textarea> element
  },
  methods: {
    handleFocus() {
      // Focus the input
      this.$refs.usernameInput.focus()

      // Select all text
      this.$refs.usernameInput.select()

      // Scroll into view
      this.$refs.usernameInput.scrollIntoView({ behavior: 'smooth' })
    },
    measureElement() {
      const el = this.$refs.customElement

      // Access DOM properties
      console.log('Offset width:', el.offsetWidth)
      console.log('Scroll height:', el.scrollHeight)
      console.log('Client rect:', el.getBoundingClientRect())
    }
  }
}
</script>
```

## Refs on Child Components

```vue
<!-- Parent.vue -->
<template>
  <div>
    <child-component ref="child"></child-component>

    <button @click="callChildMethod">Call Child Method</button>
    <button @click="getChildData">Get Child Data</button>
  </div>
</template>

<script>
import ChildComponent from './ChildComponent.vue'

export default {
  components: { ChildComponent },
  methods: {
    callChildMethod() {
      // Call child's method directly
      this.$refs.child.doSomething('from parent')
    },
    getChildData() {
      // Access child's data
      console.log(this.$refs.child.childData)

      // Access child's computed property
      console.log(this.$refs.child.childComputed)
    }
  }
}
</script>

<!-- ChildComponent.vue -->
<template>
  <div>
    <p>{{ childData }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      childData: 'Hello from child'
    }
  },
  computed: {
    childComputed() {
      return this.childData.toUpperCase()
    }
  },
  methods: {
    doSomething(message) {
      console.log('Child received:', message)
      this.childData = message
    }
  }
}
</script>
```

## Dynamic Refs with v-for

```vue
<template>
  <div>
    <div
      v-for="(item, index) in items"
      :key="item.id"
      :ref="'item-' + index"
      class="item"
    >
      {{ item.name }}
    </div>

    <button @click="scrollToItem(2)">Scroll to Item 2</button>
    <button @click="highlightAll">Highlight All</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: [
        { id: 1, name: 'Item 1' },
        { id: 2, name: 'Item 2' },
        { id: 3, name: 'Item 3' }
      ]
    }
  },
  mounted() {
    // With v-for, refs are stored in an array
    console.log(this.$refs['item-0']) // [<div>, <div>, ...]
  },
  methods: {
    scrollToItem(index) {
      const refName = 'item-' + index
      const element = this.$refs[refName][0]

      if (element) {
        element.scrollIntoView({ behavior: 'smooth' })
      }
    },
    highlightAll() {
      // Access all items with v-for refs
      Object.keys(this.$refs).forEach(refName => {
        if (refName.startsWith('item-')) {
          const elements = this.$refs[refName]
          elements.forEach(el => {
            el.style.backgroundColor = 'yellow'
          })
        }
      })
    }
  }
}
</script>
```

## Refs Timing Considerations

```vue
<template>
  <div>
    <div v-if="showElement" ref="conditionalElement">
      This element is conditionally rendered
    </div>

    <button @click="showElement = true">Show Element</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      showElement: false
    }
  },
  watch: {
    showElement(newVal) {
      if (newVal) {
        // Ref not available immediately after data change
        console.log(this.$refs.conditionalElement) // undefined

        // Use $nextTick to wait for DOM update
        this.$nextTick(() => {
          console.log(this.$refs.conditionalElement) // <div>
          this.$refs.conditionalElement.focus()
        })
      }
    }
  }
}
</script>
```

## Common Use Cases

### 1. Auto-focus Input

```vue
<template>
  <div>
    <input ref="searchInput" type="text" placeholder="Search...">
    <button @click="focusSearch">Focus Search</button>
  </div>
</template>

<script>
export default {
  mounted() {
    // Auto-focus on mount
    this.$refs.searchInput.focus()
  },
  methods: {
    focusSearch() {
      this.$refs.searchInput.focus()
    }
  }
}
</script>
```

### 2. Access Third-Party Library

```vue
<template>
  <div>
    <canvas ref="chartCanvas"></canvas>
  </div>
</template>

<script>
import Chart from 'chart.js'

export default {
  mounted() {
    // Initialize Chart.js with canvas ref
    this.chart = new Chart(this.$refs.chartCanvas, {
      type: 'bar',
      data: {
        labels: ['Red', 'Blue', 'Yellow'],
        datasets: [{
          label: '# of Votes',
          data: [12, 19, 3]
        }]
      }
    })
  },
  beforeDestroy() {
    // Clean up third-party library
    if (this.chart) {
      this.chart.destroy()
    }
  }
}
</script>
```

### 3. Form Validation

```vue
<template>
  <form @submit.prevent="handleSubmit">
    <input ref="emailInput" type="email" v-model="email">
    <span v-if="errors.email" class="error">{{ errors.email }}</span>

    <input ref="passwordInput" type="password" v-model="password">
    <span v-if="errors.password" class="error">{{ errors.password }}</span>

    <button type="submit">Submit</button>
  </form>
</template>

<script>
export default {
  data() {
    return {
      email: '',
      password: '',
      errors: {}
    }
  },
  methods: {
    handleSubmit() {
      this.errors = {}

      // Validate email
      if (!this.email) {
        this.errors.email = 'Email is required'
        this.$refs.emailInput.focus()
        return
      }

      // Validate password
      if (!this.password) {
        this.errors.password = 'Password is required'
        this.$refs.passwordInput.focus()
        return
      }

      // Submit form
      console.log('Form valid:', { email: this.email, password: this.password })
    }
  }
}
</script>
```

### 4. Child Component Method Call

```vue
<!-- Parent.vue -->
<template>
  <div>
    <video-player ref="videoPlayer" :src="videoUrl" />

    <div class="controls">
      <button @click="play">Play</button>
      <button @click="pause">Pause</button>
      <button @click="restart">Restart</button>
    </div>
  </div>
</template>

<script>
import VideoPlayer from './VideoPlayer.vue'

export default {
  components: { VideoPlayer },
  data() {
    return {
      videoUrl: '/video.mp4'
    }
  },
  methods: {
    play() {
      this.$refs.videoPlayer.play()
    },
    pause() {
      this.$refs.videoPlayer.pause()
    },
    restart() {
      this.$refs.videoPlayer.restart()
    }
  }
}
</script>

<!-- VideoPlayer.vue -->
<template>
  <div class="video-player">
    <video ref="video" :src="src" @ended="onEnded"></video>
    <div class="status">{{ status }}</div>
  </div>
</template>

<script>
export default {
  props: {
    src: String
  },
  data() {
    return {
      status: 'paused'
    }
  },
  methods: {
    play() {
      this.$refs.video.play()
      this.status = 'playing'
    },
    pause() {
      this.$refs.video.pause()
      this.status = 'paused'
    },
    restart() {
      this.$refs.video.currentTime = 0
      this.play()
    },
    onEnded() {
      this.status = 'ended'
    }
  }
}
</script>
```

## Refs Are Not Reactive

```vue
<template>
  <div>
    <input ref="input" @input="updateCount">
    <p>Input count: {{ inputCount }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      inputCount: 0
    }
  },
  methods: {
    updateCount() {
      // ❌ This won't work - refs don't trigger reactivity
      this.inputCount = this.$refs.input.value.length

      // ✅ Use v-model instead
    }
  }
}
</script>

<!-- Correct approach -->
<template>
  <div>
    <input v-model="inputValue">
    <p>Input count: {{ inputValue.length }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      inputValue: ''
    }
  }
}
</script>
```

## String Ref Function (Advanced)

```vue
<template>
  <div>
    <!-- Dynamic ref binding -->
    <div v-for="item in items" :key="item.id" :ref="setItemRef">
      {{ item.name }}
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: [
        { id: 1, name: 'Item 1' },
        { id: 2, name: 'Item 2' }
      ],
      itemRefs: []
    }
  },
  methods: {
    setItemRef(el) {
      if (el) {
        this.itemRefs.push(el)
      }
    }
  },
  mounted() {
    console.log(this.itemRefs) // [<div>, <div>]
  },
  beforeUpdate() {
    // Clear refs before update
    this.itemRefs = []
  }
}
</script>
```

## Accessing Multiple Refs with Same Name

```vue
<template>
  <div>
    <!-- Multiple elements with same ref -->
    <input ref="checkbox" type="checkbox" value="1">
    <input ref="checkbox" type="checkbox" value="2">
    <input ref="checkbox" type="checkbox" value="3">

    <button @click="getChecked">Get Checked</button>
  </div>
</template>

<script>
export default {
  methods: {
    getChecked() {
      // this.$refs.checkbox is an array
      const checked = this.$refs.checkbox
        .filter(input => input.checked)
        .map(input => input.value)

      console.log('Checked values:', checked) // ['1', '3']
    }
  }
}
</script>
```

## Refs vs Props and Events

```vue
<!-- ❌ AVOID: Using refs for data flow -->
<template>
  <div>
    <child-component ref="child" />
    <button @click="getChildData">Get Data</button>
  </div>
</template>

<script>
export default {
  methods: {
    getChildData() {
      // Breaking encapsulation
      const data = this.$refs.child.privateData
    }
  }
}
</script>

<!-- ✅ BETTER: Use props and events -->
<template>
  <div>
    <child-component @data="handleChildData" />
  </div>
</template>

<script>
export default {
  methods: {
    handleChildData(data) {
      // Child voluntarily shares data
      console.log('Received data:', data)
    }
  }
}
</script>
```

## Best Practices

1. **Use refs sparingly** - Prefer props and events for parent-child communication
2. **Use refs for**:
   - DOM manipulation (focus, scroll, measure)
   - Third-party library initialization
   - Imperative child component methods
3. **Don't use refs for**:
   - Regular data flow (use props)
   - Reactive data updates (use v-model)
   - Breaking component encapsulation
4. **Check ref existence** before using (especially with v-if)
5. **Use $nextTick** when accessing refs after data changes

## Common Pitfalls

### Pitfall 1: Accessing Ref Too Early

```javascript
// ❌ WRONG: Ref accessed in created hook
created() {
  console.log(this.$refs.myInput) // undefined
}

// ✅ CORRECT: Ref accessed in mounted hook
mounted() {
  console.log(this.$refs.myInput) // <input>
}
```

### Pitfall 2: Ref with v-if

```javascript
// ❌ WRONG: Ref might not exist
watch: {
  showPanel(newVal) {
    if (newVal) {
      this.$refs.panel.scrollIntoView() // Error!
    }
  }
}

// ✅ CORRECT: Wait for DOM update
watch: {
  showPanel(newVal) {
    if (newVal) {
      this.$nextTick(() => {
        this.$refs.panel.scrollIntoView()
      })
    }
  }
}
```

## Reference

- [Vue 2 Guide - Template Refs](https://vue2.docs.vuejs.org/v2/guide/components.html#Child-Component-Refs)
- [Vue 2 API - vm.$refs](https://vue2.docs.vuejs.org/v2/api/#vm-refs)
