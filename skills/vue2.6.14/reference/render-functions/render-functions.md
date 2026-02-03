---
title: Render Functions in Vue 2
impact: HIGH
impactDescription: Render functions provide a programmatic way to create component templates using JavaScript instead of templates
type: capability
tags:
  - vue2.6.14
  - render-functions
  - h-function
  - createElement
  - JSX
  - advanced-components
---

# Render Functions in Vue 2

Impact: HIGH - Render functions provide a programmatic way to create component templates using JavaScript's full power instead of the template syntax. They offer more flexibility and control for advanced component patterns, dynamic component generation, and performance-critical scenarios.

## Task Checklist

- [ ] Use `h()` (createElement) to create VNodes programmatically
- [ ] Understand the three arguments of `h()`: tag, data, children
- [ ] Use render functions for dynamic component generation
- [ ] Use functional components with render functions for performance
- [ ] Consider JSX for better readability when using render functions

## Basic Render Function

```vue
<script>
export default {
  render(h) {
    // h arguments: tag, data, children
    return h('div', { attrs: { id: 'app' } }, [
      h('h1', 'Hello Render Functions!'),
      h('p', 'This is created with JavaScript')
    ])
  }
}
</script>

<!-- Equivalent template:
<template>
  <div id="app">
    <h1>Hello Render Functions!</h1>
    <p>This is created with JavaScript</p>
  </div>
</template>
-->
```

## The h() Function (createElement)

```vue
<script>
export default {
  render(h) {
    // Signature: h(tag, data, children)

    // 1. Simple tag
    return h('div', 'Hello')

    // 2. Tag with children array
    return h('div', [
      h('span', 'Child 1'),
      h('span', 'Child 2')
    ])

    // 3. Tag with data object
    return h('div', {
      attrs: { id: 'my-id', class: 'my-class' },
      props: { myProp: 'value' },
      on: { click: this.handleClick }
    }, 'Content')

    // 4. Component
    return h('my-component', {
      props: { title: 'Hello' }
    })
  },
  methods: {
    handleClick() {
      console.log('Clicked!')
    }
  }
}
</script>
```

## Data Object Structure

```vue
<script>
export default {
  render(h) {
    return h('div', {
      // HTML attributes
      attrs: {
        id: 'my-id',
        class: 'static-class',
        style: 'color: red;'
      },

      // Props (for components)
      props: {
        myProp: 'value',
        anotherProp: 123
      },

      // DOM properties
      domProps: {
        innerHTML: 'HTML content',
        value: 'input value'
      },

      // Event handlers
      on: {
        click: this.handleClick,
        input: this.handleInput
      },

      // Native event handlers (for components)
      nativeOn: {
        click: this.handleNativeClick
      },

      // Class and style (object or array)
      class: {
        'active': this.isActive,
        'disabled': this.isDisabled
      },
      style: {
        color: 'red',
        fontSize: '14px'
      },

      // Other special properties
      key: 'unique-key',
      ref: 'my-ref',
      refInFor: true,
      slot: 'my-slot',

      // Directives
      directives: [
        {
          name: 'show',
          value: this.isVisible
        },
        {
          name: 'custom-directive',
          value: 'directive value',
          modifiers: { modifier1: true }
        }
      ],

      // Scoped slots
      scopedSlots: {
        default: props => h('span', props.text)
      }
    }, 'Content')
  },
  data() {
    return {
      isActive: true,
      isDisabled: false,
      isVisible: true
    }
  },
  methods: {
    handleClick(event) {
      console.log('Clicked', event)
    },
    handleInput(event) {
      console.log('Input', event.target.value)
    },
    handleNativeClick(event) {
      console.log('Native click', event)
    }
  }
}
</script>
```

## Children Argument Types

```vue
<script>
export default {
  render(h) {
    // 1. String (text node)
    return h('div', 'Simple text')

    // 2. Array (multiple children)
    return h('div', [
      h('p', 'First paragraph'),
      h('p', 'Second paragraph')
    ])

    // 3. Single VNode
    return h('div', h('span', 'Nested span'))

    // 4. Mixed array of VNodes and strings
    return h('div', [
      'Text before ',
      h('strong', 'bold text'),
      ' text after'
    ])

    // 5. Using slots
    return h('div', [
      this.$slots.header,
      this.$slots.default,
      this.$slots.footer
    ])

    // 6. Using scoped slots
    return h('div', [
      this.$scopedSlots.default({
        text: 'Slot prop value'
      })
    ])
  }
}
</script>
```

## Dynamic Component Rendering

```vue
<template>
  <div>
    <select v-model="selectedTag">
      <option value="h1">Heading 1</option>
      <option value="h2">Heading 2</option>
      <option value="h3">Heading 3</option>
      <option value="p">Paragraph</option>
      <option value="div">Div</option>
    </select>

    <div class="output">
      <!-- Dynamic tag with render function -->
      <dynamic-heading :tag="selectedTag" :text="message" />
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      selectedTag: 'h1',
      message: 'Dynamic heading content'
    }
  },
  components: {
    DynamicHeading: {
      name: 'DynamicHeading',
      props: ['tag', 'text'],
      render(h) {
        // Dynamically create different tags
        return h(this.tag, {
          class: 'dynamic-heading'
        }, this.text)
      }
    }
  }
}
</script>

<style>
.dynamic-heading {
  color: #42b983;
  margin: 10px 0;
}
</style>
```

## Functional Components with Render

```vue
<template>
  <div>
    <!-- Using functional component -->
    <smart-list :items="items" />
  </div>
</template>

<script>
// Functional component - more efficient
export default {
  name: 'FunctionalComponent',
  functional: true, // No state, no this context
  props: {
    items: {
      type: Array,
      required: true
    }
  },
  render(h, context) {
    // context.props, context.children, context.slots, context.data, context.parent
    const items = context.props.items

    if (items.length === 0) {
      return h('p', 'No items')
    }

    if (items.length === 1) {
      return h('div', { class: 'single' }, items[0].text)
    }

    return h('ul', { class: 'list' }, items.map(item =>
      h('li', { key: item.id }, item.text)
    ))
  }
}
</script>
```

## Advanced: Component with Dynamic Slots

```vue
<template>
  <div>
    <button @click="currentLayout = 'list'">List View</button>
    <button @click="currentLayout = 'grid'">Grid View</button>

    <dynamic-layout :type="currentLayout">
      <template #item="{ item }">
        <div class="item">{{ item.name }}</div>
      </template>
    </dynamic-layout>
  </div>
</template>

<script>
export default {
  data() {
    return {
      currentLayout: 'list',
      items: [
        { id: 1, name: 'Item 1' },
        { id: 2, name: 'Item 2' },
        { id: 3, name: 'Item 3' }
      ]
    }
  },
  components: {
    DynamicLayout: {
      name: 'DynamicLayout',
      props: ['type'],
      render(h) {
        const slot = this.$scopedSlots.item

        if (this.type === 'list') {
          return h('ul', { class: 'list-view' },
            this.items.map(item =>
              h('li', { key: item.id }, [slot({ item })])
            )
          )
        }

        return h('div', { class: 'grid-view' },
          this.items.map(item =>
            h('div', { key: item.id, class: 'grid-item' }, [slot({ item })])
          )
        )
      }
    }
  }
}
</script>
```

## Render Function with JSX

```vue
<script>
export default {
  name: 'JSXComponent',
  props: {
    title: String,
    items: Array
  },
  data() {
    return {
      isActive: true
    }
  },
  methods: {
    handleClick() {
      console.log('Clicked')
    }
  },
  render() {
    // JSX syntax - requires Babel configuration
    return (
      <div class={{ active: this.isActive }}>
        <h1>{this.title}</h1>
        <ul>
          {this.items.map(item => (
            <li key={item.id} onClick={this.handleClick}>
              {item.name}
            </li>
          ))}
        </ul>
      </div>
    )
  }
}
</script>

<!-- babel.config.js for JSX -->
/*
module.exports = {
  presets: ['@vue/cli-plugin-babel/preset'],
  plugins: ['@vue/babel-plugin-transform-vue-jsx']
}
*/
```

## Conditional Rendering in Render Functions

```vue
<script>
export default {
  name: 'ConditionalRender',
  props: {
    type: String,
    isLoading: Boolean,
    error: String,
    data: Object
  },
  render(h) {
    // Loading state
    if (this.isLoading) {
      return h('div', { class: 'loading' }, 'Loading...')
    }

    // Error state
    if (this.error) {
      return h('div', { class: 'error' }, [
        h('p', this.error),
        h('button', {
          on: { click: () => this.$emit('retry') }
        }, 'Retry')
      ])
    }

    // Different content types
    switch (this.type) {
      case 'header':
        return h('h1', { class: 'header' }, this.data.text)

      case 'paragraph':
        return h('p', { class: 'paragraph' }, this.data.text)

      case 'button':
        return h('button', {
          class: 'btn',
          on: { click: () => this.$emit('click') }
        }, this.data.text)

      default:
        return h('div', { class: 'default' }, this.data.text)
    }
  }
}
</script>
```

## List Rendering with Render Functions

```vue
<script>
export default {
  name: 'TodoList',
  props: {
    todos: {
      type: Array,
      default: () => []
    }
  },
  render(h) {
    if (this.todos.length === 0) {
      return h('p', { class: 'empty' }, 'No todos yet')
    }

    return h('ul', { class: 'todo-list' },
      this.todos.map(todo =>
        h('li', {
          key: todo.id,
          class: { completed: todo.completed }
        }, [
          h('input', {
            attrs: { type: 'checkbox' },
            domProps: { checked: todo.completed },
            on: {
              change: () => this.$emit('toggle', todo.id)
            }
          }),
          h('span', { class: 'text' }, todo.text),
          h('button', {
            on: {
              click: () => this.$emit('delete', todo.id)
            }
          }, '×')
        ])
      )
    )
  }
}
</script>

<style scoped>
.todo-list {
  list-style: none;
  padding: 0;
}

.todo-list li {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px;
  margin: 5px 0;
  background: white;
  border: 1px solid #ddd;
}

.todo-list li.completed .text {
  text-decoration: line-through;
  opacity: 0.6;
}
</style>
```

## VNode Must Be Unique

```vue
<script>
export default {
  name: 'UniqueVNodes',
  props: {
    items: Array
  },
  render(h) {
    // ❌ WRONG: Reusing the same VNode
    const wrongVNode = h('span', 'Same VNode')
    return h('div', [
      wrongVNode,
      wrongVNode // This will cause an error!
    ])

    // ✅ CORRECT: Create new VNodes
    return h('div', [
      h('span', 'First VNode'),
      h('span', 'Second VNode')
    ])

    // ✅ CORRECT: Use function to create VNodes
    return h('div', this.items.map(item =>
      h('span', { key: item.id }, item.text)
    ))
  }
}
</script>
```

## Accessing Slots in Render Functions

```vue
<script>
export default {
  name: 'SlotAccess',
  render(h) {
    // Access default slot
    const defaultSlot = this.$slots.default

    // Access named slots
    const headerSlot = this.$slots.header
    const footerSlot = this.$slots.footer

    // Access scoped slots
    const scopedSlot = this.$scopedSlots.default

    return h('div', { class: 'wrapper' }, [
      // Render header slot
      headerSlot ? h('header', headerSlot) : null,

      // Render default slot
      h('main', defaultSlot),

      // Render scoped slot
      scopedSlot ? scopedSlot({ message: 'Hello' }) : null,

      // Render footer slot
      footerSlot ? h('footer', footerSlot) : null
    ])
  }
}
</script>

<!-- Usage -->
<!--
<slot-access>
  <template #header>
    <h1>Header</h1>
  </template>

  <p>Default content</p>

  <template #default="{ message }">
    <p>{{ message }}</p>
  </template>

  <template #footer>
    <p>Footer</p>
  </template>
</slot-access>
-->
```

## Render Function for Wrapper Component

```vue
<template>
  <div>
    <button-wrapper tag="a" href="#" class="btn">
      Click me
    </button-wrapper>

    <button-wrapper tag="button" type="submit" class="btn">
      Submit
    </button-wrapper>
  </div>
</template>

<script>
export default {
  name: 'ButtonWrapper',
  props: {
    tag: {
      type: String,
      default: 'button'
    },
    href: String,
    type: String
  },
  render(h) {
    // Build data object dynamically
    const data = {
      class: this.$attrs.class,
      on: this.$listeners
    }

    // Add tag-specific attributes
    if (this.tag === 'a' && this.href) {
      data.attrs = { href: this.href }
    } else if (this.tag === 'button' && this.type) {
      data.attrs = { type: this.type }
    }

    // Transparent wrapper - passes all attrs and listeners
    return h(this.tag, data, this.$slots.default)
  }
}
</script>
```

## Common Patterns

### 1. Higher-Order Component (HOC)

```vue
<script>
// HOC that adds loading state to any component
function withLoading(WrappedComponent) {
  return {
    name: `WithLoading${WrappedComponent.name}`,
    props: {
      isLoading: Boolean
    },
    render(h) {
      if (this.isLoading) {
        return h('div', { class: 'loading-overlay' }, [
          h('div', { class: 'spinner' }, 'Loading...')
        ])
      }

      return h(WrappedComponent, {
        props: this.$attrs,
        on: this.$listeners
      }, this.$slots.default)
    }
  }
}

// Usage
const DataListWithLoading = withLoading(DataList)
export default DataListWithLoading
</script>
```

### 2. Dynamic Level Heading

```vue
<script>
export default {
  name: 'DynamicHeading',
  props: {
    level: {
      type: Number,
      default: 1,
      validator: value => value >= 1 && value <= 6
    },
    title: String
  },
  render(h) {
    return h(`h${this.level}`, {
      attrs: { class: `heading heading-${this.level}` }
    }, this.title)
  }
}
</script>
```

### 3. Renderless Component

```vue
<script>
export default {
  name: 'Renderless',
  props: {
    items: Array
  },
  render(h) {
    // No DOM output - just provides data via scoped slot
    return this.$scopedSlots.default({
      items: this.items,
      filtered: this.filteredItems,
      count: this.items.length
    })
  },
  computed: {
    filteredItems() {
      return this.items.filter(item => item.active)
    }
  }
}
</script>

<!-- Usage -->
<!--
<renderless :items="items">
  <template #default="{ filtered, count }">
    <div>
      <p>Total: {{ count }}</p>
      <ul>
        <li v-for="item in filtered" :key="item.id">
          {{ item.name }}
        </li>
      </ul>
    </div>
  </template>
</renderless>
-->
```

## Best Practices

1. **Use JSX for complex render functions** - More readable
2. **Prefer templates when possible** - Easier to understand
3. **Always use unique keys for lists** - Required for efficient updates
4. **Cache computed VNodes** - Avoid recreating on every render
5. **Use functional components for simple display** - Better performance
6. **Access slots via $slots and $scopedSlots** - Render slot content
7. **Don't reuse VNodes** - Each VNode must be unique

## Reference

- [Vue 2 Guide - Render Functions](https://v2.vuejs.org/v2/guide/render-function.html)
- [Vue 2 API - Render Functions](https://v2.vuejs.org/v2/guide/render-function.html#The-Data-Object-In-Depth)
- [Vue 2 JSX Support](https://v2.vuejs.org/v2/guide/render-function.html#JSX)
