---
title: Programmatic Slot Access with $slots and $scopedSlots
impact: LOW
impactDescription: $slots and $scopedSlots provide programmatic access to slot content for render functions
type: capability
tags:
  - vue2.6.14
  - slots
  - $slots
  - scoped-slots
  - $scopedSlots
  - render-functions
---

# Programmatic Slot Access with $slots and $scopedSlots

Impact: LOW - Vue 2's `$slots` and `$scopedSlots` provide programmatic access to slot content, primarily useful when writing render functions. For most template-based components, use `<slot>` elements instead.

## Task Checklist

- [ ] Use `$slots` for accessing static slots in render functions
- [ ] Use `$scopedSlots` for accessing scoped slots in render functions
- [ ] Understand that scoped slots return functions in Vue 2.6+
- [ ] Prefer `<slot>` template syntax for regular components
- [ ] Access slots via `this.$slots` and `this.$scopedSlots`

## Basic Slot Access

```vue
<template>
  <div class="my-component">
    <!-- Template syntax (recommended) -->
    <slot name="header"></slot>
    <slot></slot>
    <slot name="footer"></slot>
  </div>
</template>

<script>
export default {
  mounted() {
    // Access slots programmatically
    console.log(this.$slots.header)   // VNode array for header slot
    console.log(this.$slots.default)   // VNode array for default slot
    console.log(this.$slots.footer)    // VNode array for footer slot
  }
}
</script>
```

## $slots vs $scopedSlots

```javascript
export default {
  mounted() {
    // $slots: Static slots (no props)
    // Array of VNodes or undefined
    this.$slots.default
    this.$slots.header
    this.$slots.footer

    // $scopedSlots: Scoped slots (functions returning VNodes)
    // Functions or undefined
    this.$scopedSlots.default
    this.$scopedSlots.item
    this.$scopedSlots.row
  }
}
```

## Checking if Slot Exists

```vue
<template>
  <div class="card">
    <div v-if="$slots.header" class="card-header">
      <slot name="header"></slot>
    </div>

    <div class="card-body">
      <slot></slot>
    </div>

    <div v-if="$slots.footer" class="card-footer">
      <slot name="footer"></slot>
    </div>
  </div>
</template>

<script>
export default {
  mounted() {
    // Check if slot has content
    const hasHeader = !!this.$slots.header
    const hasFooter = !!this.$slots.footer
    const hasDefault = !!this.$slots.default

    console.log('Has header:', hasHeader)
    console.log('Has footer:', hasFooter)
    console.log('Has default content:', hasDefault)
  }
}
</script>
```

## Using Slots in Render Functions

### Static Slots with $slots

```javascript
export default {
  render(h) {
    // Access default slot
    const defaultSlot = this.$slots.default || []

    // Access named slots
    const headerSlot = this.$slots.header || []
    const footerSlot = this.$slots.footer || []

    return h('div', { class: 'my-component' }, [
      // Render header slot if exists
      headerSlot.length ? h('header', {}, headerSlot) : null,

      // Render default slot
      h('main', {}, defaultSlot),

      // Render footer slot if exists
      footerSlot.length ? h('footer', {}, footerSlot) : null
    ])
  }
}
```

### Scoped Slots with $scopedSlots

```javascript
export default {
  props: {
    items: Array
  },
  render(h) {
    // Access scoped slot function
    const itemSlot = this.$scopedSlots.item

    if (!itemSlot) {
      return h('p', 'No item slot provided')
    }

    // Map items through scoped slot
    const itemNodes = this.items.map((item, index) => {
      return itemSlot({
        item: item,
        index: index
      })
    })

    return h('ul', {}, itemNodes)
  }
}
```

**Usage:**

```vue
<template>
  <list-renderer :items="items">
    <template #item="{ item, index }">
      <li>{{ index + 1 }}: {{ item.name }}</li>
    </template>
  </list-renderer>
</template>
```

## Complete Example: Table Component

```javascript
// DataTable.js
export default {
  name: 'DataTable',
  props: {
    columns: Array,
    data: Array
  },
  render(h) {
    // Get scoped slots
    const cellSlot = this.$scopedSlots.cell
    const rowSlot = this.$scopedSlots.row
    const emptySlot = this.$scopedSlots.empty

    // No data
    if (!this.data || this.data.length === 0) {
      return emptySlot ? emptySlot() : h('p', 'No data available')
    }

    // Header row
    const headerRow = this.columns.map(column => {
      return h('th', column.label || column.key)
    })

    const header = h('thead', [
      h('tr', headerRow)
    ])

    // Body rows
    const bodyRows = this.data.map((rowData, rowIndex) => {
      // Check for custom row slot
      if (rowSlot) {
        return rowSlot({ rowData, rowIndex })
      }

      // Default row rendering
      const cells = this.columns.map(column => {
        const cellData = rowData[column.key]

        // Check for custom cell slot
        if (cellSlot) {
          return cellSlot({
            rowData,
            column,
            value: cellData,
            rowIndex
          })
        }

        // Default cell
        return h('td', cellData)
      })

      return h('tr', cells)
    })

    const body = h('tbody', bodyRows)

    return h('table', { class: 'data-table' }, [header, body])
  }
}
```

**Usage:**

```vue
<template>
  <data-table :columns="columns" :data="users">
    <!-- Custom cell slot -->
    <template #cell="{ rowData, value }">
      <span v-if="column.key === 'name'" class="user-name">
        {{ value }}
      </span>
      <span v-else-if="column.key === 'email'">
        <a :href="`mailto:${value}`">{{ value }}</a>
      </span>
      <span v-else>{{ value }}</span>
    </template>

    <!-- Custom row slot -->
    <template #row="{ rowData, rowIndex }">
      <tr :class="{ even: rowIndex % 2 === 0 }">
        <td>{{ rowData.name }}</td>
        <td>{{ rowData.email }}</td>
        <td>{{ rowData.role }}</td>
      </tr>
    </template>

    <!-- Empty state slot -->
    <template #empty>
      <div class="empty-state">
        <p>No users found</p>
        <button @click="createUser">Create User</button>
      </div>
    </template>
  </data-table>
</template>

<script>
import DataTable from './DataTable.js'

export default {
  components: { DataTable },
  data() {
    return {
      columns: [
        { key: 'name', label: 'Name' },
        { key: 'email', label: 'Email' },
        { key: 'role', label: 'Role' }
      ],
      users: []
    }
  }
}
</script>
```

## Functional Component with Slots

```javascript
// FunctionalLayout.js
export default {
  name: 'FunctionalLayout',
  functional: true,
  render(h, { props, slots, scopedSlots }) {
    // Access slots differently in functional components
    const defaultSlot = slots().default || []
    const headerSlot = slots().header || []
    const footerSlot = slots().footer || []

    // Scoped slots
    const itemSlot = scopedSlots.item

    return h('div', { class: 'layout' }, [
      headerSlot.length ? h('header', {}, headerSlot) : null,
      h('main', {}, defaultSlot),
      footerSlot.length ? h('footer', {}, footerSlot) : null
    ])
  }
}
```

## Conditional Slot Rendering

```vue
<script>
export default {
  render(h) {
    const hasHeader = this.$slots.header && this.$slots.header.length > 0
    const hasFooter = this.$slots.footer && this.$slots.footer.length > 0

    return h('div', { class: 'container' }, [
      // Conditionally render header
      hasHeader ? h('header', { class: 'header' }, this.$slots.header) : null,

      // Main content
      h('main', { class: 'content' }, this.$slots.default || []),

      // Conditionally render footer
      hasFooter ? h('footer', { class: 'footer' }, this.$slots.footer) : null
    ])
  }
}
</script>
```

## Wrapping Slot Content

```vue
<script>
export default {
  render(h) {
    const defaultSlot = this.$slots.default || []

    // Wrap each slot item
    const wrappedItems = defaultSlot.map(node => {
      return h('div', { class: 'wrapper' }, [node])
    })

    return h('div', { class: 'container' }, wrappedItems)
  }
}
</script>
```

## Multiple Fallbacks for Slots

```vue
<script>
export default {
  render(h) {
    // Try custom slot, fallback to default content
    const headerContent = this.$slots.header || [
      h('h2', 'Default Header')
    ]

    const footerContent = this.$slots.footer || [
      h('p', '© 2024 All rights reserved')
    ]

    return h('div', { class: 'page' }, [
      h('header', {}, headerContent),
      h('main', {}, this.$slots.default || []),
      h('footer', {}, footerContent)
    ])
  }
}
</script>
```

## Complex Slot Composition

```javascript
export default {
  render(h) {
    // Get all available slots
    const slots = this.$slots

    // Build header with fallback
    const header = h('header', {}, [
      slots['header-start'] || [],
      slots.title || [h('h1', 'Default Title')],
      slots['header-end'] || []
    ])

    // Build main content with sidebar
    const main = h('div', { class: 'main-layout' }, [
      h('aside', {}, slots.sidebar || []),
      h('main', {}, slots.default || [])
    ])

    // Build footer
    const footer = h('footer', {}, slots.footer || [])

    return h('div', { class: 'page' }, [header, main, footer])
  }
}
```

## Accessing Slot Context

```javascript
export default {
  props: {
    users: Array
  },
  methods: {
    getUserSlot(user, index) {
      const slot = this.$scopedSlots.user

      if (!slot) {
        // Fallback rendering
        return this.$createElement('div', `${user.name} (${user.email})`)
      }

      // Call slot with context
      return slot({
        user: user,
        index: index,
        isAdmin: user.role === 'admin'
      })
    }
  },
  render(h) {
    const userList = this.users.map((user, index) => {
      return h('li', { key: user.id }, [
        this.getUserSlot(user, index)
      ])
    })

    return h('ul', {}, userList)
  }
}
```

## Slots with JSX

```vue
<script>
export default {
  functional: true,
  render(h, { slots, scopedSlots }) {
    const { header, footer, default: defaultSlot } = slots()

    return (
      <div className="container">
        {header && <header>{header}</header>}
        <main>{defaultSlot}</main>
        {footer && <footer>{footer}</footer>}
      </div>
    )
  }
}
</script>
```

## Checking Slot Content Type

```javascript
export default {
  mounted() {
    // Check what's in the default slot
    if (this.$slots.default) {
      this.$slots.default.forEach(vnode => {
        console.log('VNode tag:', vnode.tag)
        console.log('VNode text:', vnode.text)
        console.log('VNode children:', vnode.children)
        console.log('VNode data:', vnode.data)
      })
    }
  }
}
```

## Manipulating Slot VNodes

```vue
<script>
export default {
  render(h) {
    // Clone and modify slot VNodes
    const modifiedSlots = (this.$slots.default || []).map(vnode => {
      return h(vnode.componentOptions || vnode.tag, {
        ...vnode.data,
        // Add custom class
        class: { 'slot-content': true }
      }, vnode.children || vnode.text)
    })

    return h('div', { class: 'wrapper' }, modifiedSlots)
  }
}
</script>
```

## Best Practices

1. **Prefer `<slot>` syntax** for template-based components
2. **Use `$slots` and `$scopedSlots`** in render functions
3. **Always check for existence** before accessing slots (`this.$slots.header || []`)
4. **Document slot expectations** for component consumers
5. **Provide fallback content** when slot is optional

## Common Pitfalls

### Pitfall 1: Forgetting Slots Can Be Empty

```javascript
// ❌ WRONG: Assuming slot exists
render(h) {
  return h('div', this.$slots.header) // Error if empty!
}

// ✅ CORRECT: Provide fallback
render(h) {
  return h('div', this.$slots.header || [])
}
```

### Pitfall 2: Not Calling Scoped Slot Function

```javascript
// ❌ WRONG: Using scoped slot like static slot
render(h) {
  return this.$scopedSlots.item // Returns function, not VNodes!
}

// ✅ CORRECT: Call the scoped slot function
render(h) {
  return this.$scopedSlots.item({ item: myItem })
}
```

## Comparison Table

| Feature | Template Syntax | Programmatic |
|---------|----------------|--------------|
| Static slots | `<slot name="header">` | `this.$slots.header` |
| Scoped slots | `<slot name="item" :item="item">` | `this.$scopedSlots.item(props)` |
| Check exists | v-if on slot element | `!!this.$slots.header` |
| Use case | Most components | Render functions |

## Reference

- [Vue 2 Guide - Slots](https://vue2.docs.vuejs.org/v2/guide/components-slots.html)
- [Vue 2 API - vm.$slots](https://vue2.docs.vuejs.org/v2/api/#vm-slots)
- [Vue 2 API - vm.$scopedSlots](https://vue2.docs.vuejs.org/v2/api/#vm-scopedSlots)
