---
title: v-once Directive for Static Content Optimization
impact: LOW
impactDescription: v-once renders element/component once then treats it as static content for better performance
type: capability
tags:
  - vue2.6.14
  - v-once
  - directives
  - performance
  - static-content
  - optimization
---

# v-once Directive for Static Content Optimization

Impact: LOW - The `v-once` directive renders an element or component only once, then treats it as static content. On subsequent re-renders, the element/component and all its children will be skipped. This is useful for optimizing update performance for content that doesn't change.

## Task Checklist

- [ ] Use `v-once` for content that renders once and never changes
- [ ] Use `v-once` for expensive components that don't need reactivity
- [ ] Remember `v-once` affects all children of the element
- [ ] Combine with `v-for` for static lists
- [ ] Avoid using `v-once` with dynamic or reactive content

## Basic Usage

```vue
<template>
  <div>
    <!-- Renders once, then becomes static -->
    <span v-once>This renders once: {{ count }}</span>

    <!-- Regular rendering (always reactive) -->
    <span>This renders reactively: {{ count }}</span>

    <button @click="count++">Increment</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      count: 0
    }
  }
}
</script>
```

**Result:**
- First span renders "This renders once: 0" and never updates
- Second span updates normally with count

## Single Element Optimization

```vue
<template>
  <div>
    <!-- Static header - renders once -->
    <header v-once>
      <h1>{{ pageTitle }}</h1>
      <p>{{ pageDescription }}</p>
    </header>

    <!-- Dynamic content -->
    <main>
      <p>{{ dynamicContent }}</p>
    </main>
  </div>
</template>

<script>
export default {
  data() {
    return {
      pageTitle: 'Static Page Title',
      pageDescription: 'This never changes',
      dynamicContent: 'This changes frequently'
    }
  }
}
</script>
```

## Component with v-once

```vue
<template>
  <div>
    <!-- Static footer component - renders once -->
    <app-footer v-once />

    <!-- Dynamic list -->
    <todo-list :todos="todos" />
  </div>
</template>

<script>
import AppFooter from './AppFooter.vue'
import TodoList from './TodoList.vue'

export default {
  components: {
    AppFooter,
    TodoList
  },
  data() {
    return {
      todos: []
    }
  }
}
</script>

<!-- AppFooter.vue -->
<template>
  <footer class="app-footer">
    <p>© 2024 Company Inc. All rights reserved.</p>
    <p>Privacy Policy | Terms of Service</p>
  </footer>
</template>
```

## v-once with v-for

```vue
<template>
  <div>
    <!-- Static list items with v-for -->
    <ul v-once>
      <li v-for="item in staticItems" :key="item.id">
        {{ item.name }}
      </li>
    </ul>

    <!-- Dynamic list -->
    <ul>
      <li v-for="item in dynamicItems" :key="item.id">
        {{ item.name }}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      staticItems: [
        { id: 1, name: 'Static Item 1' },
        { id: 2, name: 'Static Item 2' }
      ],
      dynamicItems: []
    }
  }
}
</script>
```

## Expensive Components

```vue
<template>
  <div>
    <!-- Heavy chart component - render once -->
    <heavy-chart
      v-once
      :data="staticChartData"
      :options="chartOptions"
    />

    <!-- Dynamic chart -->
    <light-chart
      :data="dynamicChartData"
    />
  </div>
</template>

<script>
import HeavyChart from './HeavyChart.vue'
import LightChart from './LightChart.vue'

export default {
  components: {
    HeavyChart,
    LightChart
  },
  data() {
    return {
      staticChartData: { /* static data */ },
      dynamicChartData: {},
      chartOptions: {}
    }
  }
}
</script>
```

## Content With Children

```vue
<template>
  <div>
    <!-- v-once affects entire subtree -->
    <div v-once class="static-section">
      <h3>Static Section</h3>
      <p>{{ staticText }}</p>
      <child-component />
      <grand-child-component />
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      staticText: 'This section is static after initial render'
    }
  }
}
</script>
```

## v-once Lifecycle

```vue
<script>
export default {
  data() {
    return {
      content: 'Initial content'
    }
  },
  mounted() {
    // First render: v-once element renders
    console.log('First render')

    setTimeout(() => {
      // This won't trigger re-render for v-once element
      this.content = 'Updated content'
      console.log('Content updated, but v-once won\'t re-render')
    }, 2000)
  }
}
</script>

<template>
  <div v-once>
    <p>{{ content }}</p>
  </div>
</template>
```

## Nested v-once Behavior

```vue
<template>
  <div>
    <!-- Parent with v-once -->
    <div v-once class="parent">
      <p>Parent: {{ parentData }}</p>

      <!-- Child without v-once - still won't update! -->
      <child-component :data="childData" />
    </div>
  </div>
</template>

<script>
import ChildComponent from './ChildComponent.vue'

export default {
  components: { ChildComponent },
  data() {
    return {
      parentData: 'Parent data',
      childData: 'Child data'
    }
  }
}
</script>
```

## Performance Comparison

```vue
<template>
  <div>
    <!-- Without v-once: Re-renders on every data change -->
    <div class="expensive-list">
      <item
        v-for="item in items"
        :key="item.id"
        :item="item"
      />
    </div>

    <!-- With v-once: Renders once, then skipped -->
    <div class="static-list">
      <item
        v-for="item in items"
        :key="item.id"
        :item="item"
        v-once
      />
    </div>
  </div>
</template>
```

## Using v-once Conditionally

```vue
<template>
  <div>
    <!-- Only use v-once for truly static content -->
    <section v-once="isContentStatic">
      <static-content />
    </section>

    <!-- Regular reactivity for dynamic content -->
    <section v-else>
      <dynamic-content />
    </section>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isContentStatic: true
    }
  }
}
</script>
```

## v-once with CSS Transitions

```vue
<template>
  <div>
    <!-- v-once prevents re-rendering but not transitions -->
    <transition name="fade">
      <div v-once>
        <h3>{{ title }}</h3>
        <p>{{ description }}</p>
      </div>
    </transition>
  </div>
</template>

<style scoped>
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
</style>
```

## Debugging v-once

```vue
<script>
export default {
  mounted() {
    // Check if an element has v-once
    const element = this.$refs.testElement
    console.log('Has v-once:', element && element.hasAttribute('v-once'))
  }
}
</script>

<template>
  <div>
    <div ref="testElement" v-once>
      Content
    </div>
  </div>
</template>
```

## Common Patterns

### 1. Documentation Content

```vue
<template>
  <article class="documentation">
    <!-- Static headers -->
    <header v-once>
      <h1>{{ pageTitle }}</h1>
      <p>{{ pageDescription }}</p>
    </header>

    <!-- Table of contents -->
    <toc :sections="tableOfContents" />

    <!-- Main content (dynamic) -->
    <section v-for="section in activeSections" :key="section.id">
      <h2>{{ section.title }}</h2>
      <div v-html="section.content"></div>
    </section>
  </article>
</template>
```

### 2. Server-Side Rendered Content

```vue
<template>
  <div>
    <!-- Content from server - use v-once -->
    <div v-once>
      <div v-html="serverContent"></div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    serverContent: String
  }
}
</script>
```

### 3. Static Navigation

```vue
<template>
  <nav class="main-nav">
    <!-- Static nav items -->
    <ul v-once>
      <li><router-link to="/">Home</router-link></li>
      <li><router-link to="/about">About</router-link></li>
      <li><router-link to="/contact">Contact</router-link></li>
    </ul>

    <!-- Dynamic user menu -->
    <user-menu :user="currentUser" />
  </nav>
</template>
```

## v-once with Keep-Alive

```vue
<template>
  <div>
    <!-- Static content in keep-alive -->
    <keep-alive>
      <component
        :is="currentView"
        v-once="isViewStatic(currentView)"
      />
    </keep-alive>
  </div>
</template>

<script>
export default {
  computed: {
    currentView() {
      return this.$route.meta.view || 'home'
    },
    isViewStatic() {
      // Determine if view content is static
      const staticViews = ['home', 'about']
      return staticViews.includes(this.currentView)
    }
  }
}
</script>
```

## Best Practices

1. **Use for truly static content** - content that never changes after initial render
2. **Avoid with dynamic data** - v-once prevents reactivity for the entire subtree
3. **Use with expensive components** - heavy components that don't need updates
4. **Consider using `v-if` instead** - for conditional rendering (destroy and recreate)
5. **Combine with `v-for`** - for lists that don't change after initial render
6. **Remember it affects children** - all children become static too

## When NOT to Use v-once

```vue
<!-- ❌ AVOID: v-once with dynamic content -->
<template>
  <div v-once>
    <!-- These won't update! -->
    <p>{{ message }}</p>
    <button @click="handleClick">Click</button>
  </div>
</template>

<!-- ✅ CORRECT: Use v-if instead -->
<template>
  <div v-if="showSection">
    <p>{{ message }}</p>
    <button @click="handleClick">Click</button>
  </div>
</template>
```

## v-once vs Static Component

```javascript
// v-once approach
export default {
  template: `
    <div v-once>{{ message }}</div>
  `
}

// Static component alternative
const StaticComponent = {
  functional: true,
  props: ['message'],
  render(h, { props }) {
    return h('div', props.message)
  }
}
```

## Performance Tips

```vue
<script>
export default {
  // Using v-once can improve render performance
  render(h) {
    // Static header
    const header = h('header', { attrs: { 'v-once': true } }, [
      h('h1', this.pageTitle),
      h('p', this.pageDescription)
    ])

    // Dynamic content
    const main = h('main', this.mainContent)

    return h('div', [header, main])
  },
  computed: {
    mainContent() {
      return this.$slots.default || []
    }
  }
}
</script>
```

## Reference

- [Vue 2 API - v-once](https://vue2.docs.vuejs.org/v2/api/#v-once)
- [Vue 2 Guide - Cheap Static Components](https://vue2.docs.vuejs.org/v2/guide/components.html#Cheap-Static-Components-with-v-once)
