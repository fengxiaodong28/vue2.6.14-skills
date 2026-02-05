---
title: Special Attributes in Vue 2 (slot, is, key, ref)
impact: MEDIUM
impactDescription: Special attributes provide key functionality for component identity, template refs, conditional rendering, and dynamic components
type: capability
tags:
  - vue2.6.14
  - special-attributes
  - key
  - ref
  - slot
  - is
  - v-bind-sync
  - component-api
---

# Special Attributes in Vue 2

Impact: MEDIUM - Special attributes like `key`, `ref`, `slot`, and `is` provide essential functionality for component identity management, template references, slot assignment, and dynamic component rendering. These attributes enable advanced component patterns and proper reactivity in list rendering.

## Task Checklist

- [ ] Use `key` for efficient list rendering and component identity
- [ ] Use `ref` for direct DOM element and component instance access
- [ ] Use `slot` for static slot content (deprecated in favor of v-slot)
- [ ] Use `is` for dynamic components and resolving DOM element limitations
- [ ] Understand deprecated vs recommended slot syntax

## key Attribute

The `key` attribute provides a unique identity for elements/components in lists, enabling Vue to optimize rendering and track element state.

```vue
<template>
  <div>
    <!-- ✅ CORRECT: Using key with v-for -->
    <ul>
      <li v-for="item in items" :key="item.id">
        {{ item.name }}
      </li>
    </ul>

    <!-- ❌ WRONG: Using index as key (causes issues with reordering) -->
    <ul>
      <li v-for="(item, index) in items" :key="index">
        {{ item.name }}
      </li>
    </ul>

    <!-- ✅ CORRECT: Using key for conditional components -->
    <transition>
      <component :is="currentComponent" :key="componentKey" />
    </transition>

    <!-- Force component recreation -->
    <user-profile :key="userId" />
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
      ],
      currentComponent: 'ComponentA',
      componentKey: 0,
      userId: 123
    }
  },
  methods: {
    refreshComponent() {
      // Changing key forces component to destroy and recreate
      this.componentKey++
    }
  }
}
</script>
```

### Key Best Practices

```vue
<template>
  <div>
    <!-- ✅ Use unique, stable IDs -->
    <li v-for="user in users" :key="user.id">
      {{ user.name }}
    </li>

    <!-- ✅ Use compound keys for uniqueness -->
    <li v-for="item in items" :key="`${item.category}-${item.id}`">
      {{ item.name }}
    </li>

    <!-- ❌ Avoid non-unique values -->
    <li v-for="item in items" :key="Math.random()">
      <!-- Random keys cause unnecessary re-renders -->
    </li>

    <!-- ❌ Avoid using index when list can be reordered -->
    <li v-for="(item, index) in items" :key="index">
      <!-- Causes issues with item state and reordering -->
    </li>
  </div>
</template>
```

### Key with v-if

```vue
<template>
  <div>
    <!-- When toggling between components with same tag -->
    <transition>
      <button
        v-if="isEditing"
        key="save-button"
        @click="save"
      >
        Save
      </button>
      <button
        v-else
        key="edit-button"
        @click="edit"
      >
        Edit
      </button>
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
```

## ref Attribute

The `ref` attribute provides direct access to DOM elements or child component instances.

```vue
<template>
  <div>
    <!-- Element ref -->
    <input ref="inputField" placeholder="Focus me" />

    <!-- Component ref -->
    <user-profile ref="userProfile" :user-id="123" />

    <!-- Dynamic ref with v-for -->
    <div v-for="item in items" :key="item.id" :ref="`item-${item.id}`">
      {{ item.name }}
    </div>

    <button @click="focusInput">Focus Input</button>
    <button @click="callChildMethod">Call Child Method</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: [
        { id: 1, name: 'Item 1' },
        { id: 2, name: 'Item 2' }
      ]
    }
  },
  mounted() {
    // Access refs after mounted
    this.$refs.inputField.focus()
  },
  methods: {
    focusInput() {
      // Access DOM element
      const input = this.$refs.inputField
      if (input) {
        input.focus()
        input.value = 'Hello'
      }
    },
    callChildMethod() {
      // Access child component instance
      const profile = this.$refs.userProfile
      if (profile) {
        // Call child component method
        profile.refreshData()

        // Access child component data
        console.log(profile.userData)
      }
    },
    accessDynamicRefs() {
      // Access refs created with v-for
      const item1 = this.$refs['item-1']
      if (item1) {
        // item1 is an array when created with v-for
        const element = Array.isArray(item1) ? item1[0] : item1
        console.log(element.textContent)
      }
    }
  }
}
</script>
```

### Ref Timing

```vue
<script>
export default {
  props: {
    show: Boolean
  },
  mounted() {
    // ✅ Refs are available in mounted
    console.log(this.$refs.myElement)
  },
  watch: {
    show(newVal) {
      if (newVal) {
        // ❌ Ref not available immediately - use nextTick
        console.log(this.$refs.myElement) // undefined

        // ✅ Use nextTick for v-if refs
        this.$nextTick(() => {
          console.log(this.$refs.myElement) // available
        })
      }
    }
  }
}
</script>
```

### Ref with v-for

```vue
<template>
  <div>
    <div
      v-for="(item, index) in items"
      :key="item.id"
      :ref="'items'"
    >
      {{ item.name }}
    </div>

    <button @click="logAllRefs">Log All Refs</button>
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
  methods: {
    logAllRefs() {
      // With v-for, ref becomes an array
      const refs = this.$refs.items
      console.log(refs) // [div, div, div]

      // Iterate through all refs
      refs.forEach((ref, index) => {
        console.log(`Ref ${index}:`, ref.textContent)
      })

      // Check if refs exist
      if (refs && refs.length) {
        const firstRef = refs[0]
        firstRef.style.color = 'red'
      }
    }
  }
}
</script>
```

## slot Attribute (Deprecated)

> **Note**: The `slot` attribute was used in Vue 2.5 and earlier. In Vue 2.6.0+, `v-slot` is the recommended syntax. The `slot` attribute is deprecated but still supported for backward compatibility.

```vue
<!-- Parent Component -->
<template>
  <div>
    <!-- ❌ DEPRECATED: Using slot attribute (Vue 2.5.x syntax) -->
    <base-layout>
      <h1 slot="header">Page Title</h1>
      <p slot="footer">Page footer</p>

      <p>Main content goes here.</p>
    </base-layout>

    <!-- ✅ RECOMMENDED: Using v-slot directive (Vue 2.6.0+) -->
    <base-layout>
      <template v-slot:header>
        <h1>Page Title</h1>
      </template>

      <template v-slot:footer>
        <p>Page footer</p>
      </template>

      <template v-slot:default>
        <p>Main content goes here.</p>
      </template>
    </base-layout>
  </div>
</template>

<!-- Child Component -->
<template>
  <div class="layout">
    <header>
      <slot name="header"></slot>
    </header>

    <main>
      <slot></slot>
    </main>

    <footer>
      <slot name="footer"></slot>
    </footer>
  </div>
</template>
```

## is Attribute

The `is` attribute is used for dynamic components and working around HTML element restrictions.

### Dynamic Components

```vue
<template>
  <div>
    <button @click="currentTab = 'home'">Home</button>
    <button @click="currentTab = 'profile'">Profile</button>
    <button @click="currentTab = 'settings'">Settings</button>

    <!-- Dynamic component -->
    <component :is="currentTab" :user="currentUser" />
  </div>
</template>

<script>
export default {
  data() {
    return {
      currentTab: 'home',
      currentUser: { name: 'John' }
    }
  },
  components: {
    home: {
      props: ['user'],
      template: '<div>Home - Welcome {{ user.name }}</div>'
    },
    profile: {
      props: ['user'],
      template: '<div>Profile - {{ user.name }}</div>'
    },
    settings: {
      props: ['user'],
      template: '<div>Settings - User preferences</div>'
    }
  }
}
</script>
```

### DOM Template Parsing Limitations

```vue
<template>
  <div>
    <!-- ❌ WRONG: <ul> only allows <li> as direct children -->
    <ul>
      <custom-component></custom-component>
    </ul>

    <!-- ✅ CORRECT: Use is attribute -->
    <ul>
      <li is="custom-component"></li>
    </ul>

    <!-- Table example -->
    <table>
      <!-- ❌ WRONG -->
      <my-row></my-row>

      <!-- ✅ CORRECT -->
      <tr is="my-row"></tr>
    </table>

    <!-- Select example -->
    <select>
      <!-- ❌ WRONG -->
      <option-wrapper></option-wrapper>

      <!-- ✅ CORRECT -->
      <option is="option-wrapper"></option>
    </select>
  </div>
</template>

<script>
export default {
  components: {
    CustomComponent: {
      template: '<li>Custom component content</li>'
    },
    MyRow: {
      template: '<td>Row content</td>'
    },
    OptionWrapper: {
      template: '<option>Option content</option>'
    }
  }
}
</script>
```

### Dynamic Component with keep-alive

```vue
<template>
  <div>
    <keep-alive>
      <component :is="currentView" :key="viewKey" />
    </keep-alive>
  </div>
</template>

<script>
export default {
  data() {
    return {
      currentView: 'view-a',
      viewKey: 0
    }
  },
  methods: {
    switchView(viewName) {
      // Force component refresh by changing key
      if (this.currentView === viewName) {
        this.viewKey++
      } else {
        this.currentView = viewName
      }
    }
  }
}
</script>
```

## Special Attribute Combinations

### key + ref Together

```vue
<template>
  <div>
    <div
      v-for="item in items"
      :key="item.id"
      :ref="`item-${item.id}`"
    >
      {{ item.name }}
    </div>

    <button @click="highlightFirst">Highlight First</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: [
        { id: 1, name: 'First' },
        { id: 2, name: 'Second' },
        { id: 3, name: 'Third' }
      ]
    }
  },
  methods: {
    highlightFirst() {
      // Access specific ref by key
      const firstItem = this.$refs['item-1']
      if (firstItem && firstItem[0]) {
        firstItem[0].style.background = 'yellow'
      }
    }
  }
}
</script>
```

### ref + v-if

```vue
<template>
  <div>
    <div v-if="showElement" ref="conditionalElement">
      Conditional content
    </div>

    <button @click="toggle">Toggle</button>
    <button @click="focusOnShow">Focus on Show</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      showElement: false
    }
  },
  methods: {
    toggle() {
      this.showElement = !this.showElement

      // If showing, focus after DOM update
      if (this.showElement) {
        this.$nextTick(() => {
          this.$refs.conditionalElement.focus()
        })
      }
    },
    focusOnShow() {
      this.showElement = true
      this.$nextTick(() => {
        const el = this.$refs.conditionalElement
        if (el) {
          el.focus()
        }
      })
    }
  }
}
</script>
```

## Common Patterns

### 1. List with Active Item

```vue
<template>
  <div class="list-with-active">
    <div
      v-for="item in items"
      :key="item.id"
      :ref="`item-${item.id}`"
      :class="{ active: activeId === item.id }"
      @click="setActive(item.id)"
    >
      {{ item.name }}
    </div>

    <button @click="scrollToActive">Scroll to Active</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      activeId: 1,
      items: [
        { id: 1, name: 'Item 1' },
        { id: 2, name: 'Item 2' },
        { id: 3, name: 'Item 3' }
      ]
    }
  },
  methods: {
    setActive(id) {
      this.activeId = id
    },
    scrollToActive() {
      const activeRef = this.$refs[`item-${this.activeId}`]
      if (activeRef && activeRef[0]) {
        activeRef[0].scrollIntoView({ behavior: 'smooth' })
      }
    }
  }
}
</script>

<style scoped>
.active {
  background: #42b983;
  color: white;
}
</style>
```

### 2. Dynamic Form Fields

```vue
<template>
  <div class="dynamic-form">
    <div v-for="(field, index) in fields" :key="field.id">
      <label>{{ field.label }}</label>

      <!-- Dynamic component based on field type -->
      <component
        :is="getFieldComponent(field.type)"
        :ref="`field-${field.id}`"
        v-model="field.value"
        v-bind="field.props"
      />

      <button @click="removeField(index)">Remove</button>
    </div>

    <button @click="addField">Add Field</button>
    <button @click="validateAll">Validate All</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      fields: [],
      fieldIdCounter: 0
    }
  },
  methods: {
    getFieldComponent(type) {
      const components = {
        text: 'text-input',
        number: 'number-input',
        select: 'select-input'
      }
      return components[type] || 'text-input'
    },
    addField() {
      this.fields.push({
        id: this.fieldIdCounter++,
        type: 'text',
        label: 'New Field',
        value: '',
        props: {}
      })
    },
    removeField(index) {
      this.fields.splice(index, 1)
    },
    validateAll() {
      this.fields.forEach(field => {
        const ref = this.$refs[`field-${field.id}`]
        if (ref && ref[0] && ref[0].validate) {
          ref[0].validate()
        }
      })
    }
  },
  components: {
    'text-input': {
      props: ['value'],
      template: '<input type="text" :value="value" @input="$emit(\'input\', $event.target.value)">'
    },
    'number-input': {
      props: ['value'],
      template: '<input type="number" :value="value" @input="$emit(\'input\', $event.target.value)">'
    },
    'select-input': {
      props: ['value'],
      template: '<select :value="value" @change="$emit(\'input\', $event.target.value)"><option>Option 1</option></select>'
    }
  }
}
</script>
```

### 3. Force Component Remount

```vue
<template>
  <div>
    <button @click="refresh">Refresh Data</button>

    <!-- Changing key forces component to remount -->
    <data-view
      :key="refreshKey"
      :data-source="apiUrl"
    />
  </div>
</template>

<script>
export default {
  data() {
    return {
      refreshKey: 0,
      apiUrl: '/api/data'
    }
  },
  methods: {
    refresh() {
      // Increment key to force DataView to remount
      this.refreshKey++
    }
  }
}
</script>
```

## Best Practices

1. **Always use `key` with `v-for`** - Essential for efficient updates
2. **Use unique IDs for keys** - Avoid using array indices
3. **Use `$nextTick` for conditional refs** - DOM may not be updated immediately
4. **Prefer v-slot over slot attribute** - New syntax is more powerful
5. **Use `is` for dynamic components** - Enables component switching
6. **Use `is` for custom elements in restricted contexts** - Tables, lists, selects
7. **Access refs as arrays for v-for** - Refs become arrays when used with v-for

## Reference

- [Vue 2 API - Special Attributes](https://v2.vuejs.org/v2/api/#Special-Attributes)
- [Vue 2 Guide - Syntax - key](https://v2.vuejs.org/v2/api/#key)
- [Vue 2 Guide - Syntax - ref](https://v2.vuejs.org/v2/api/#ref)
- [Vue 2 Guide - Syntax - is](https://v2.vuejs.org/v2/api/#is)
- [Vue 2 Guide - Slots - Deprecated Syntax](https://v2.vuejs.org/v2/guide/components-slots.html#Deprecated-Syntax)
