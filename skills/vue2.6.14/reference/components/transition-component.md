---
title: transition Built-in Component in Vue 2
impact: MEDIUM
impactDescription: transition component enables enter/leave transitions for single elements/components with CSS classes or JavaScript hooks
type: capability
tags:
  - vue2.6.14
  - transition
  - built-in-components
  - animations
  - css-transitions
---

# transition Built-in Component in Vue 2

Impact: MEDIUM - The `<transition>` component allows you to apply enter/leave transitions for single elements or components. It works with CSS transitions/animations or JavaScript hooks, providing smooth UI state changes.

## Task Checklist

- [ ] Use `<transition>` for single element/component transitions
- [ ] Apply CSS classes for transition states (v-enter-active, v-leave-active, etc.)
- [ ] Use JavaScript hooks for complex transitions
- [ ] Specify transition name with `name` prop for custom CSS classes
- [ ] Use `mode` prop for transition timing control between elements

## Basic Usage

```vue
<template>
  <div>
    <button @click="show = !show">Toggle</button>

    <!-- Basic transition -->
    <transition>
      <p v-if="show">Hello Vue 2!</p>
    </transition>
  </div>
</template>

<script>
export default {
  data() {
    return {
      show: true
    }
  }
}
</script>

<style>
/* Default transition classes */
.v-enter-active, .v-leave-active {
  transition: opacity 0.3s;
}
.v-enter, .v-leave-to {
  opacity: 0;
}
</style>
```

## Transition Name Prop

```vue
<template>
  <div>
    <button @click="show = !show">Toggle</button>

    <!-- Named transition -->
    <transition name="fade">
      <p v-if="show">Fade transition</p>
    </transition>
  </div>
</template>

<style>
/* Named transition classes */
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
</style>
```

## CSS Transition Classes

Vue 2 applies six CSS classes during a transition:

```vue
<template>
  <transition name="slide">
    <div v-if="show" class="box">
      Sliding Box
    </div>
  </transition>
</template>

<style>
/* 1. v-enter: Starting state for enter (added before element insert) */
.slide-enter {
  transform: translateX(-100%);
}

/* 2. v-enter-active: Active state for enter */
.slide-enter-active {
  transition: transform 0.3s ease-out;
}

/* 3. v-enter-to: Ending state for enter (removed after transition completes) */
.slide-enter-to {
  transform: translateX(0);
}

/* 4. v-leave: Starting state for leave */
.slide-leave {
  transform: translateX(0);
}

/* 5. v-leave-active: Active state for leave */
.slide-leave-active {
  transition: transition 0.3s ease-in;
}

/* 6. v-leave-to: Ending state for leave */
.slide-leave-to {
  transform: translateX(100%);
}

.box {
  width: 100px;
  height: 100px;
  background: #42b983;
}
</style>
```

## CSS Animations

```vue
<template>
  <div>
    <button @click="show = !show">Toggle</button>

    <transition name="bounce">
      <p v-if="show" class="bounce-element">Bouncing!</p>
    </transition>
  </div>
</template>

<style>
@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.5);
  }
  100% {
    transform: scale(1);
  }
}

.bounce-enter-active {
  animation: bounce-in 0.5s;
}

.bounce-leave-active {
  animation: bounce-in 0.5s reverse;
}

.bounce-element {
  display: inline-block;
  padding: 20px;
  background: #42b983;
  color: white;
}
</style>
```

## Custom Transition Classes

```vue
<template>
  <transition
    name="fade"
    enter-active-class="animate__animated animate__fadeIn"
    leave-active-class="animate__animated animate__fadeOut"
  >
    <div v-if="show" class="box">
      Animate.css Transition
    </div>
  </transition>
</template>

<style>
/* Requires animate.css library */
.box {
  width: 200px;
  height: 100px;
  background: #42b983;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
```

## JavaScript Hooks

```vue
<template>
  <div>
    <button @click="show = !show">
      Toggle with JavaScript Hooks
    </button>

    <transition
      @before-enter="beforeEnter"
      @enter="enter"
      @after-enter="afterEnter"
      @enter-cancelled="enterCancelled"
      @before-leave="beforeLeave"
      @leave="leave"
      @after-leave="afterLeave"
      @leave-cancelled="leaveCancelled"
    >
      <div v-if="show" class="box">
        JavaScript Transition
      </div>
    </transition>
  </div>
</template>

<script>
export default {
  data() {
    return {
      show: true
    }
  },
  methods: {
    // Called before the enter transition starts
    beforeEnter(el) {
      console.log('beforeEnter', el)
      el.style.opacity = 0
      el.style.transform = 'translateX(-100%)'
    },

    // Called when the enter transition starts
    // Use done to indicate transition completion when using JavaScript
    enter(el, done) {
      console.log('enter', el)

      // Force reflow
      const reflow = el.offsetHeight

      // Start the animation
      el.style.transition = 'all 0.5s ease'
      el.style.opacity = 1
      el.style.transform = 'translateX(0)'

      // Call done when animation finishes
      setTimeout(done, 500)
    },

    // Called when the enter transition has finished
    afterEnter(el) {
      console.log('afterEnter', el)
    },

    // Called when the enter transition is cancelled
    enterCancelled(el) {
      console.log('enterCancelled', el)
    },

    // Called before the leave transition starts
    beforeLeave(el) {
      console.log('beforeLeave', el)
    },

    // Called when the leave transition starts
    leave(el, done) {
      console.log('leave', el)
      el.style.transition = 'all 0.5s ease'
      el.style.opacity = 0
      el.style.transform = 'translateX(100%)'

      setTimeout(done, 500)
    },

    // Called when the leave transition has finished
    afterLeave(el) {
      console.log('afterLeave', el)
    },

    // Called when the leave transition is cancelled
    leaveCancelled(el) {
      console.log('leaveCancelled', el)
    }
  }
}
</script>

<style>
.box {
  width: 200px;
  height: 100px;
  background: #42b983;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
```

## Transition Modes

```vue
<template>
  <div>
    <button @click="toggle">Toggle with Mode</button>

    <!-- mode="out-in": Current element transitions out first -->
    <transition name="fade" mode="out-in">
      <button v-if="isEditing" key="save" @click="save">
        Save
      </button>
      <button v-else key="edit" @click="edit">
        Edit
      </button>
    </transition>

    <!-- mode="in-out": New element transitions in first -->
    <transition name="slide" mode="in-out">
      <component :is="currentComponent" />
    </transition>

    <!-- Default: Both transitions happen simultaneously -->
    <transition name="fade">
      <component :is="currentComponent" />
    </transition>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isEditing: false
    }
  },
  methods: {
    toggle() {
      this.isEditing = !this.isEditing
    },
    save() {
      // Handle save
      this.isEditing = false
    },
    edit() {
      // Handle edit
      this.isEditing = true
    }
  }
}
</script>

<style>
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}

.slide-enter-active, .slide-leave-active {
  transition: transform 0.3s;
}
.slide-enter, .slide-leave-to {
  transform: translateX(100%);
}
</style>
```

## Transition with Dynamic Components

```vue
<template>
  <div>
    <button @click="currentView = 'home'">Home</button>
    <button @click="currentView = 'about'">About</button>
    <button @click="currentView = 'contact'">Contact</button>

    <transition name="fade" mode="out-in">
      <component :is="currentView" :key="currentView" />
    </transition>
  </div>
</template>

<script>
export default {
  data() {
    return {
      currentView: 'home'
    }
  },
  components: {
    home: {
      template: '<div class="view"><h2>Home</h2><p>Welcome home!</p></div>'
    },
    about: {
      template: '<div class="view"><h2>About</h2><p>About us page</p></div>'
    },
    contact: {
      template: '<div class="view"><h2>Contact</h2><p>Contact us page</p></div>'
    }
  }
}
</script>

<style>
.view {
  padding: 20px;
  background: #f5f5f5;
  margin-top: 20px;
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
</style>
```

## Transition Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `name` | String | `'v'` | Prefix for transition CSS classes |
| `appear` | Boolean | `false` | Apply transition on initial render |
| `mode` | String | - | Controls timing between enter/leave (`'in-out'` or `'out-in'`) |
| `duration` | Number \| Object | - | Manual duration specification |
| `enter-class` | String | - | Deprecated in Vue 2.6.0+ |
| `leave-class` | String | - | Deprecated in Vue 2.6.0+ |
| `enter-to-class` | String | - | Deprecated in Vue 2.6.0+ |
| `leave-to-class` | String | - | Deprecated in Vue 2.6.0+ |
| `enter-active-class` | String | - | Custom class for enter active state |
| `leave-active-class` | String | - | Custom class for leave active state |
| `appear-class` | String | - | Deprecated in Vue 2.6.0+ |
| `appear-to-class` | String | - | Deprecated in Vue 2.6.0+ |
| `appear-active-class` | String | - | Custom class for appear active state |
| `css` | Boolean | `true` | Whether to apply CSS transition classes |
| `type` | String | - | Specify transition type (`'transition'` or `'animation'`) |
| `persisted` | Boolean | `false` | For fine-grained control (used with v-on) |

## Duration Prop

```vue
<template>
  <div>
    <!-- Single duration for both enter and leave -->
    <transition :duration="1000">
      <p v-if="show">1 second transition</p>
    </transition>

    <!-- Different durations for enter and leave -->
    <transition :duration="{ enter: 500, leave: 1000 }">
      <p v-if="show">Custom durations</p>
    </transition>
  </div>
</template>

<script>
export default {
  data() {
    return {
      show: true
    }
  }
}
</script>
```

## Appear on Initial Render

```vue
<template>
  <transition appear appear-active-class="fade-appear-active">
    <div class="box">
      This element fades in on mount
    </div>
  </transition>
</template>

<style>
.fade-appear-active {
  animation: fadeIn 1s;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.box {
  width: 200px;
  height: 100px;
  background: #42b983;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
```

## Transition Events

```vue
<template>
  <transition
    @before-enter="handleBeforeEnter"
    @enter="handleEnter"
    @after-enter="handleAfterEnter"
    @enter-cancelled="handleEnterCancelled"
    @before-leave="handleBeforeLeave"
    @leave="handleLeave"
    @after-leave="handleAfterLeave"
    @leave-cancelled="handleLeaveCancelled"
  >
    <div v-if="show" class="box">
      Transition with Events
    </div>
  </transition>
</template>

<script>
export default {
  data() {
    return {
      show: true
    }
  },
  methods: {
    handleBeforeEnter(el) {
      console.log('Before enter')
    },
    handleEnter(el, done) {
      console.log('Enter')
      done()
    },
    handleAfterEnter(el) {
      console.log('After enter')
    },
    handleEnterCancelled(el) {
      console.log('Enter cancelled')
    },
    handleBeforeLeave(el) {
      console.log('Before leave')
    },
    handleLeave(el, done) {
      console.log('Leave')
      done()
    },
    handleAfterLeave(el) {
      console.log('After leave')
    },
    handleLeaveCancelled(el) {
      console.log('Leave cancelled')
    }
  }
}
</script>
```

## Transition with Velocity.js

```vue
<template>
  <div>
    <button @click="show = !show">Toggle with Velocity</button>

    <transition
      @enter="enter"
      @leave="leave"
    >
      <div v-if="show" class="box">
        Velocity.js Transition
      </div>
    </transition>
  </div>
</template>

<script>
import Velocity from 'velocity-animate'

export default {
  data() {
    return {
      show: true
    }
  },
  methods: {
    enter(el, done) {
      Velocity(
        el,
        { opacity: 1, translateX: 0 },
        { duration: 500, complete: done }
      )
    },
    leave(el, done) {
      Velocity(
        el,
        { opacity: 0, translateX: '100%' },
        { duration: 500, complete: done }
      )
    }
  }
}
</script>

<style>
.box {
  width: 200px;
  height: 100px;
  background: #42b983;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
```

## Common Patterns

### 1. Modal Fade In/Out

```vue
<template>
  <div>
    <button @click="showModal = true">Show Modal</button>

    <transition name="modal">
      <div v-if="showModal" class="modal-mask">
        <div class="modal-container">
          <h3>Modal Title</h3>
          <p>Modal content goes here...</p>
          <button @click="showModal = false">Close</button>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
export default {
  data() {
    return {
      showModal: false
    }
  }
}
</script>

<style>
.modal-enter-active, .modal-leave-active {
  transition: opacity 0.3s;
}

.modal-enter, .modal-leave-to {
  opacity: 0;
}

.modal-mask {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-container {
  background: white;
  padding: 20px;
  border-radius: 8px;
  max-width: 400px;
}
</style>
```

### 2. Slide Sidebar

```vue
<template>
  <div>
    <button @click="showSidebar = !showSidebar">Toggle Sidebar</button>

    <transition name="slide">
      <div v-if="showSidebar" class="sidebar">
        <h3>Sidebar</h3>
        <nav>
          <a href="#">Link 1</a>
          <a href="#">Link 2</a>
          <a href="#">Link 3</a>
        </nav>
      </div>
    </transition>
  </div>
</template>

<script>
export default {
  data() {
    return {
      showSidebar: false
    }
  }
}
</script>

<style>
.slide-enter-active, .slide-leave-active {
  transition: transform 0.3s ease;
}

.slide-enter, .slide-leave-to {
  transform: translateX(-100%);
}

.sidebar {
  position: fixed;
  top: 0;
  left: 0;
  width: 250px;
  height: 100%;
  background: #2c3e50;
  color: white;
  padding: 20px;
}

.sidebar nav a {
  display: block;
  color: white;
  text-decoration: none;
  padding: 10px 0;
}
</style>
```

## Best Practices

1. **Always use `key` with dynamic components** - Ensures proper transitions
2. **Use `mode="out-in"` for route transitions** - Prevents overlap
3. **Prefer CSS transitions** - Better performance than JavaScript
4. **Use `appear` for initial mount animations** - Smooth first render
5. **Test accessibility** - Respect `prefers-reduced-motion` media query
6. **Avoid nested transitions** - Can cause unexpected behavior

## Accessibility Consideration

```vue
<template>
  <transition name="fade">
    <div v-if="show" class="content">
      Content
    </div>
  </transition>
</template>

<style>
@media (prefers-reduced-motion: reduce) {
  .fade-enter-active, .fade-leave-active {
    transition: none;
  }
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
</style>
```

## Reference

- [Vue 2 API - transition](https://v2.vuejs.org/v2/api/#transition)
- [Vue 2 Guide - Transitions Enter/Leave](https://v2.vuejs.org/v2/guide/transitions.html)
