---
title: Event Modifiers in Vue 2 (v-on)
impact: MEDIUM
impactDescription: Event modifiers change how event handlers behave - preventing defaults, stopping propagation, and more
type: capability
tags:
  - vue2.6.14
  - v-on
  - event-modifiers
  - events
  - @ shorthand
---

# Event Modifiers in Vue 2 (v-on)

Impact: MEDIUM - Vue 2's event modifiers allow you to handle common event patterns directly in the template. Use `.stop`, `.prevent`, `.capture`, `.self`, `.once`, `.passive`, and key modifiers to control event behavior without writing extra JavaScript.

## Task Checklist

- [ ] Use `.stop` to call `event.stopPropagation()`
- [ ] Use `.prevent` to call `event.preventDefault()`
- [ ] Use `.capture` to add event listener in capture mode
- [ ] Use `.self` to only trigger if event.target is the element itself
- [ ] Use `.once` to trigger handler at most once
- [ ] Use `.passive` for better scroll performance
- [ ] Use key modifiers for keyboard events

## All Event Modifiers

| Modifier | Behavior | DOM Equivalent |
|----------|----------|----------------|
| `.stop` | Stop event propagation | `event.stopPropagation()` |
| `.prevent` | Prevent default behavior | `event.preventDefault()` |
| `.capture` | Use capture mode | `addEventListener(..., true)` |
| `.self` | Only trigger on self | Check `event.target === event.currentTarget` |
| `.once` | Trigger only once | Remove listener after first call |
| `.passive` | Passive listener | `{ passive: true }` option |
| `.native` | Listen for native event on component | Direct DOM event |
| `.left/.right/.middle` | Mouse button filter | Check `event.button` |

## .stop Modifier

Stops event from bubbling up to parent elements:

```vue
<template>
  <div @click="handleParentClick">
    <div @click="handleChildClick">
      <button @click.stop="handleButtonClick">
        Click Me
      </button>
    </div>
  </div>
</template>

<script>
export default {
  methods: {
    handleButtonClick() {
      console.log('Button clicked')
      // Parent click handlers won't be triggered
    },
    handleChildClick() {
      console.log('Child clicked')
    },
    handleParentClick() {
      console.log('Parent clicked')
    }
  }
}
</script>
```

## .prevent Modifier

Prevents the browser's default behavior:

```vue
<template>
  <div>
    <!-- Prevent form submission -->
    <form @submit.prevent="handleSubmit">
      <input type="text" v-model="name">
      <button type="submit">Submit</button>
    </form>

    <!-- Prevent link navigation -->
    <a href="/page" @click.prevent="handleClick">
      Custom Navigation
    </a>

    <!-- Prevent default for any event -->
    <div @mousemove.prevent>
      Can't select text here
    </div>
  </div>
</template>

<script>
export default {
  methods: {
    handleSubmit() {
      // Form won't actually submit
      console.log('Form submitted:', this.name)
    },
    handleClick() {
      // Won't navigate to /page
      console.log('Link clicked')
    }
  },
  data() {
    return {
      name: ''
    }
  }
}
</script>
```

## Chaining Modifiers

Modifiers can be chained:

```vue
<template>
  <div>
    <!-- Stop propagation AND prevent default -->
    <a href="/link" @click.stop.prevent="doThat">
      Click Me
    </a>

    <!-- Just prevent default without stopping propagation -->
    <form @submit.prevent="onSubmit">
      <!-- form content -->
    </form>
  </div>
</template>
```

## .capture Modifier

Adds event listener in capture mode (fires before descendant events):

```vue
<template>
  <div @click.capture="handleParentClick">
    <!-- Parent's handler fires BEFORE child's -->
    <div @click="handleChildClick">
      Click Child
    </div>
  </div>
</template>

<script>
export default {
  methods: {
    handleParentClick() {
      console.log('1. Parent click (capture)')
      // This fires first!
    },
    handleChildClick() {
      console.log('2. Child click')
      // This fires second
    }
  }
}
</script>
```

## .self Modifier

Only triggers if `event.target` is the element itself (not a child):

```vue
<template>
  <div @click.self="handleDivClick" class="container">
    <!-- Only clicking the div (not the button) triggers this -->
    <button @click="handleButtonClick">
      Click Me
    </button>
  </div>
</template>

<script>
export default {
  methods: {
    handleDivClick() {
      console.log('Div clicked directly')
      // Won't fire when clicking the button
    },
    handleButtonClick() {
      console.log('Button clicked')
    }
  }
}
</script>
```

## .once Modifier

Event handler will only be triggered once:

```vue
<template>
  <div>
    <!-- Button can only be clicked once -->
    <button @click.once="handleSubmit">
      Submit (One Time)
    </button>

    <!-- Load more only works once -->
    <button @click.once="loadMore">
      Load More
    </button>
  </div>
</template>

<script>
export default {
  methods: {
    handleSubmit() {
      console.log('Submitted!')
      // Handler removed after first click
    },
    loadMore() {
      console.log('Loading more...')
      // This will only work once
    }
  }
}
</script>
```

## .passive Modifier

Improves scroll performance by telling the browser you won't call `preventDefault()`:

```vue
<template>
  <!-- Better scroll performance for touch devices -->
  <div
    @scroll.passive="onScroll"
    @touchmove.passive="onTouchMove"
    class="scroll-container"
  >
    <!-- Scrollable content -->
  </div>
</template>

<script>
export default {
  methods: {
    onScroll() {
      // Can still read event properties
      console.log('Scroll position:', event.target.scrollTop)
    },
    onTouchMove() {
      // But can't call event.preventDefault()
      console.log('Touch move detected')
    }
  }
}
</script>

<style>
.scroll-container {
  height: 200px;
  overflow-y: auto;
}
</style>
```

## Key Modifiers

Listen for specific keyboard keys:

```vue
<template>
  <div>
    <!-- Only trigger on Enter key -->
    <input @keyup.enter="submitForm" placeholder="Press Enter to submit">

    <!-- Only trigger on specific keys -->
    <input @keyup.enter="submit">
    <input @keyup.esc="cancel">
    <input @keyup.space="toggle">
    <input @keyup.tab="handleTab">
    <input @keyup.delete="deleteItem">
    <input @keyup.up="moveUp">
    <input @keyup.down="moveDown">
    <input @keyup.left="moveLeft">
    <input @keyup.right="moveRight">

    <!-- Key aliases are case-insensitive -->
    <input @keyup.page-down="onPageDown">
  </div>
</template>

<script>
export default {
  methods: {
    submitForm() {
      console.log('Form submitted')
    },
    cancel() {
      console.log('Cancelled')
    },
    toggle() {
      console.log('Toggled')
    },
    deleteItem() {
      console.log('Item deleted')
    },
    moveUp() {
      console.log('Moving up')
    }
  }
}
</script>
```

## Custom Key Modifiers

You can also use any valid key name exposed by `KeyboardEvent.key`:

```vue
<template>
  <div>
    <!-- Using exact key names (kebab-case or quoted) -->
    <input @keyup.page-down="onPageDown">
    <input @keyup.arrow-up="onArrowUp">
    <input @keyup.ctrl.enter="submit">

    <!-- Custom key alias (defined globally) -->
    <input @keyup.f1="showHelp">
    <input @keyup.media-play-pause="togglePlay">
  </div>
</template>

<script>
// Define custom key aliases globally (in main.js)
// Vue.config.keyCodes.f1 = 112
// Vue.config.keyCodes['media-play-pause'] = 179

export default {
  methods: {
    onPageDown() {
      console.log('Page Down pressed')
    },
    onArrowUp() {
      console.log('Arrow Up pressed')
    },
    submit() {
      console.log('Ctrl + Enter pressed')
    }
  }
}
</script>
```

## Modifier Combinations

```vue
<template>
  <div>
    <!-- Multiple key modifiers -->
    <input @keyup.ctrl.enter="submit">

    <!-- Key + modifier combinations -->
    <input @keyup.alt.delete="removeItem">
    <input @keyup.shift.tab="reverseTab">

    <!-- Event + key modifiers -->
    <input @keydown.enter.prevent="preventEnterSubmit">
    <input @keyup.enter.exact="submitOnlyWithEnter">
  </div>
</template>
```

## .exact Modifier

Requires exact combination of modifiers (no extra keys pressed):

```vue
<template>
  <div>
    <!-- Fires even if Alt or Shift is also pressed -->
    <button @click.ctrl="onClick">A</button>

    <!-- Fires only when Ctrl is pressed (with no other modifiers) -->
    <button @click.ctrl.exact="onCtrlClick">A</button>

    <!-- Fires only when no system modifiers are pressed -->
    <button @click.exact="onClick">A</button>
  </div>
</template>

<script>
export default {
  methods: {
    onClick() {
      console.log('Clicked with no modifiers')
    },
    onCtrlClick() {
      console.log('Clicked with ONLY Ctrl')
    }
  }
}
</script>
```

## Mouse Button Modifiers

```vue
<template>
  <div>
    <!-- Only left mouse button -->
    <div @click.left="handleLeftClick">Left click only</div>

    <!-- Only right mouse button -->
    <div @click.right.prevent="handleRightClick">
      Right click only (context menu prevented)
    </div>

    <!-- Only middle mouse button -->
    <div @click.middle="handleMiddleClick">Middle click only</div>
  </div>
</template>

<script>
export default {
  methods: {
    handleLeftClick() {
      console.log('Left clicked')
    },
    handleRightClick() {
      console.log('Right clicked')
    },
    handleMiddleClick() {
      console.log('Middle clicked')
    }
  }
}
</script>
```

## .native Modifier (Components)

Listen for native events on component root element:

```vue
<template>
  <div>
    <!-- Listen for native click on component's root element -->
    <my-custom-component @click.native="handleNativeClick">
      Content
    </my-custom-component>
  </div>
</template>

<script>
import MyCustomComponent from './MyCustomComponent.vue'

export default {
  components: { MyCustomComponent },
  methods: {
    handleNativeClick() {
      console.log('Native click on component')
    }
  }
}
</script>
```

## Common Patterns

### 1. Form with Submit Prevention

```vue
<template>
  <form @submit.prevent="handleSubmit">
    <input v-model="email" type="email" placeholder="Email">
    <input v-model="password" type="password" placeholder="Password">
    <button type="submit">Login</button>
  </form>
</template>

<script>
export default {
  data() {
    return {
      email: '',
      password: ''
    }
  },
  methods: {
    async handleSubmit() {
      // Form won't actually submit/refresh page
      const result = await this.$http.post('/login', {
        email: this.email,
        password: this.password
      })
      console.log('Login result:', result)
    }
  }
}
</script>
```

### 2. Click Outside to Close

```vue
<template>
  <div>
    <div
      v-if="isOpen"
      ref="dropdown"
      @click.stop
      class="dropdown"
    >
      <p>Dropdown content</p>
    </div>

    <!-- Click outside (on document) to close -->
    <button @click="toggleDropdown">Toggle</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isOpen: false
    }
  },
  mounted() {
    document.addEventListener('click', this.handleClickOutside)
  },
  beforeDestroy() {
    document.removeEventListener('click', this.handleClickOutside)
  },
  methods: {
    toggleDropdown() {
      this.isOpen = !this.isOpen
    },
    handleClickOutside(event) {
      const dropdown = this.$refs.dropdown
      if (dropdown && !dropdown.contains(event.target)) {
        this.isOpen = false
      }
    }
  }
}
</script>
```

### 3. Keyboard Navigation

```vue
<template>
  <div
    tabindex="0"
    @keydown.enter="selectItem"
    @keydown.arrow-up="previousItem"
    @keydown.arrow-down="nextItem"
    @keydown.esc="close"
  >
    <div v-for="(item, index) in items" :key="index" :class="{ active: index === activeIndex }">
      {{ item }}
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: ['Item 1', 'Item 2', 'Item 3'],
      activeIndex: 0
    }
  },
  methods: {
    selectItem() {
      console.log('Selected:', this.items[this.activeIndex])
    },
    previousItem() {
      this.activeIndex = Math.max(0, this.activeIndex - 1)
    },
    nextItem() {
      this.activeIndex = Math.min(this.items.length - 1, this.activeIndex + 1)
    },
    close() {
      console.log('Closed')
    }
  }
}
</script>
```

## Performance Tips

```vue
<template>
  <!-- GOOD: Use .passive for scroll/touch events -->
  <div @scroll.passive="handleScroll">
    <!-- Content -->
  </div>

  <!-- GOOD: Use .prevent to avoid default -->
  <form @submit.prevent="handleSubmit">
    <!-- Form content -->
  </form>

  <!-- AVOID: Don't use .passive if you need preventDefault() -->
  <div @touchmove="handleTouchMove">
    <!-- This might cause warning -->
  </div>

  <!-- GOOD: Use .once for one-time handlers -->
  <button @click.once="loadInitialData">
    Load Data
  </button>
</template>
```

## Reference

- [Vue 2 Guide - Event Modifiers](https://vue2.docs.vuejs.org/v2/guide/events.html#Event-Modifiers)
- [Vue 2 API - v-on](https://vue2.docs.vuejs.org/v2/api/#v-on)
