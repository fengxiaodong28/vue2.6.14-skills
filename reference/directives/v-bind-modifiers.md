---
title: v-bind Modifiers in Vue 2
impact: MEDIUM
impactDescription: v-bind modifiers (.prop, .camel, .sync) change how attribute binding works
type: capability
tags:
  - vue2.6.14
  - v-bind
  - binding-modifiers
  - attributes
  - : shorthand
  - two-way-binding
---

# v-bind Modifiers in Vue 2

Impact: MEDIUM - Vue 2's `v-bind` directive supports modifiers that change how bindings work. Use `.prop` to bind as DOM property, `.camel` to convert attribute names, and `.sync` for two-way binding syntax sugar.

## Task Checklist

- [ ] Use `.prop` to bind as DOM property instead of attribute
- [ ] Use `.camel` to convert kebab-case attributes to camelCase properties
- [ ] Use `.sync` for two-way binding with props (Vue 2.3+)
- [ ] Understand that `.sync` is syntactic sugar for `v-bind` + `v-on`
- [ ] Prefer explicit events over `.sync` for complex scenarios

## Overview of Modifiers

| Modifier | Behavior | Use Case |
|----------|----------|----------|
| `.prop` | Bind as DOM property | Custom properties, non-standard attributes |
| `.camel` | Convert kebab-case to camelCase | SVG attributes, custom element properties |
| `.sync` | Two-way binding sugar | Parent-child two-way communication |

## Basic v-bind Syntax

```vue
<template>
  <div>
    <!-- Full syntax -->
    <img v-bind:src="imageSrc">

    <!-- Shorthand -->
    <img :src="imageSrc">

    <!-- Multiple bindings with object -->
    <div v-bind="{ id: dynamicId, class: dynamicClass }">
      Content
    </div>
  </div>
</template>
```

## .prop Modifier

Binds as a DOM property instead of an HTML attribute:

```vue
<template>
  <div>
    <!-- Regular attribute binding -->
    <div :text-content="message">
      This content is replaced
    </div>
    <!-- Result: <div text-content="Hello">...</div> -->
    <!-- Note: text-content is NOT a standard attribute -->

    <!-- Using .prop to bind as DOM property -->
    <div :text-content.prop="message">
      This content is replaced
    </div>
    <!-- Result: The textContent property is set on the DOM element -->
    <!-- Visual: The element displays "Hello" -->
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: 'Hello'
    }
  }
}
</script>
```

**When to use .prop:**

```vue
<template>
  <div>
    <!-- Bind to input.value property directly -->
    <input :value.prop="inputValue">

    <!-- Bind to custom web component property -->
    <my-element :custom-property.prop="customValue"></my-element>

    <!-- Bind to scroll position -->
    <div :scroll-top.prop="scrollPosition">
      Content
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      inputValue: 'Initial value',
      customValue: 123,
      scrollPosition: 0
    }
  }
}
</script>
```

**Difference between attribute and property:**

```vue
<template>
  <div>
    <!-- Attribute: value is initial value, changes don't reflect -->
    <input :value="text" placeholder="Attribute binding">

    <!-- Property: value is live, changes reflect -->
    <input :value.prop="text" placeholder="Property binding">
  </div>
</template>
```

## .camel Modifier

Converts kebab-case attribute name to camelCase (useful for in-DOM templates):

```vue
<template>
  <!-- In .vue files with vue-loader, camelCase works normally -->
  <svg :viewBox="viewBoxString">
    <!-- viewBox is camelCased automatically -->
  </svg>

  <!-- In DOM templates (without build step), need .camel -->
  <div id="app">
    <!-- This would look for "viewbox" attribute (wrong) -->
    <svg v-bind:view-box="viewBoxString"></svg>

    <!-- With .camel, correctly binds to viewBox property -->
    <svg v-bind:view-box.camel="viewBoxString"></svg>
  </div>
</template>

<script>
export default {
  data() {
    return {
      viewBoxString: '0 0 100 100'
    }
  }
}
</script>
```

**Custom element properties:**

```vue
<template>
  <!-- For custom elements with camelCase properties -->
  <my-element
    :my-property.camel="value"
    :anotherProp.camel="anotherValue"
  >
  </my-element>
</template>

<script>
export default {
  data() {
    return {
      value: 'test',
      anotherValue: 123
    }
  }
}
</script>
```

## .sync Modifier (Vue 2.3+)

Two-way binding syntax sugar for props. The `.sync` modifier expands into a `v-bind` and a `v-on`:

```vue
<template>
  <div>
    <!-- Using .sync (two-way binding) -->
    <child-component :title.sync="documentTitle"></child-component>

    <!-- This is syntactic sugar for: -->
    <child-component
      :title="documentTitle"
      @update:title="documentTitle = $event"
    ></child-component>
  </div>
</template>

<script>
import ChildComponent from './ChildComponent.vue'

export default {
  components: { ChildComponent },
  data() {
    return {
      documentTitle: 'Default Title'
    }
  }
}
</script>
```

**Child component emits update event:**

```vue
<!-- ChildComponent.vue -->
<template>
  <div>
    <input
      :value="title"
      @input="$emit('update:title', $event.target.value)"
    >
  </div>
</template>

<script>
export default {
  props: {
    title: {
      type: String,
      required: true
    }
  }
}
</script>
```

## .sync with Multiple Props

```vue
<template>
  <div>
    <!-- Multiple .sync bindings -->
    <user-editor
      :name.sync="user.name"
      :email.sync="user.email"
      :age.sync="user.age"
    ></user-editor>

    <!-- Expands to: -->
    <user-editor
      :name="user.name"
      @update:name="user.name = $event"
      :email="user.email"
      @update:email="user.email = $event"
      :age="user.age"
      @update:age="user.age = $event"
    ></user-editor>
  </div>
</template>
```

## .sync with Object Syntax (Not Recommended)

```vue
<template>
  <!-- This updates entire object (less explicit) -->
  <user-form v-bind.sync="user"></user-form>

  <!-- Not recommended: hard to track what changes -->
  <!-- Better to be explicit about which properties sync -->
</template>

<script>
export default {
  data() {
    return {
      user: {
        name: 'John',
        email: 'john@example.com'
      }
    }
  }
}
</script>
```

## Real-World Examples

### 1. Modal with .sync

```vue
<!-- Parent.vue -->
<template>
  <div>
    <button @click="showModal = true">Open Modal</button>

    <!-- Using .sync for two-way visibility binding -->
    <modal
      :visible.sync="showModal"
      :title.sync="modalTitle"
      @confirm="handleConfirm"
    >
      <p>Modal content goes here</p>
    </modal>
  </div>
</template>

<script>
import Modal from './Modal.vue'

export default {
  components: { Modal },
  data() {
    return {
      showModal: false,
      modalTitle: 'My Modal'
    }
  },
  methods: {
    handleConfirm() {
      console.log('Confirmed')
      this.showModal = false
    }
  }
}
</script>
```

```vue
<!-- Modal.vue -->
<template>
  <div v-if="visible" class="modal-overlay" @click.self="close">
    <div class="modal-content">
      <div class="modal-header">
        <input :value="title" @input="updateTitle">
        <button @click="close">×</button>
      </div>
      <div class="modal-body">
        <slot></slot>
      </div>
      <div class="modal-footer">
        <button @click="$emit('confirm')">Confirm</button>
        <button @click="close">Cancel</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    visible: Boolean,
    title: String
  },
  methods: {
    close() {
      this.$emit('update:visible', false)
    },
    updateTitle(event) {
      this.$emit('update:title', event.target.value)
    }
  }
}
</script>
```

### 2. Accordion with .sync

```vue
<!-- Parent.vue -->
<template>
  <div>
    <accordion-item
      v-for="(item, index) in items"
      :key="index"
      :title="item.title"
      :expanded.sync="item.expanded"
    >
      {{ item.content }}
    </accordion-item>
  </div>
</template>

<script>
import AccordionItem from './AccordionItem.vue'

export default {
  components: { AccordionItem },
  data() {
    return {
      items: [
        { title: 'Item 1', content: 'Content 1', expanded: false },
        { title: 'Item 2', content: 'Content 2', expanded: false },
        { title: 'Item 3', content: 'Content 3', expanded: false }
      ]
    }
  }
}
</script>
```

```vue
<!-- AccordionItem.vue -->
<template>
  <div class="accordion-item">
    <div class="accordion-header" @click="toggle">
      <h3>{{ title }}</h3>
      <span>{{ expanded ? '−' : '+' }}</span>
    </div>
    <div v-if="expanded" class="accordion-body">
      <slot></slot>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    title: String,
    expanded: Boolean
  },
  methods: {
    toggle() {
      this.$emit('update:expanded', !this.expanded)
    }
  }
}
</script>
```

### 3. Custom Input Component with .sync

```vue
<!-- CustomInput.vue -->
<template>
  <div class="custom-input">
    <label :for="inputId">{{ label }}</label>
    <input
      :id="inputId"
      :value="value"
      @input="updateValue"
      :type="type"
      :placeholder="placeholder"
    >
    <span v-if="error" class="error">{{ error }}</span>
  </div>
</template>

<script>
export default {
  props: {
    value: {
      type: [String, Number],
      default: ''
    },
    label: String,
    type: {
      type: String,
      default: 'text'
    },
    placeholder: String,
    error: String
  },
  data() {
    return {
      inputId: `input-${Math.random().toString(36).substr(2, 9)}`
    }
  },
  methods: {
    updateValue(event) {
      this.$emit('update:value', event.target.value)
    }
  }
}
</script>

<style scoped>
.custom-input {
  margin-bottom: 1rem;
}
.error {
  color: red;
  font-size: 0.875rem;
}
</style>
```

**Usage:**

```vue
<template>
  <div>
    <custom-input
      :value.sync="formData.name"
      label="Name"
      placeholder="Enter your name"
    ></custom-input>

    <custom-input
      :value.sync="formData.email"
      label="Email"
      type="email"
      placeholder="Enter your email"
    ></custom-input>
  </div>
</template>
```

### 4. .prop for Direct DOM Manipulation

```vue
<template>
  <div>
    <!-- Scroll to specific position -->
    <div
      ref="scrollContainer"
      :scroll-top.prop="scrollTop"
      class="scroll-container"
    >
      <!-- Long content -->
    </div>

    <button @click="scrollTop = 0">Scroll to Top</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      scrollTop: 0
    }
  },
  methods: {
    scrollToPosition(position) {
      this.scrollTop = position
    }
  }
}
</script>
```

## v-bind Shorthand with Modifiers

```vue
<template>
  <div>
    <!-- Full syntax -->
    <div v-bind:text-content.prop="text"></div>

    <!-- Shorthand -->
    <div :text-content.prop="text"></div>

    <!-- Dynamic attribute with .prop -->
    <div :[attrName].prop="attrValue"></div>
  </div>
</template>
```

## Comparison: .sync vs Explicit Events

```vue
<template>
  <div>
    <!-- With .sync (concise but less explicit) -->
    <my-dialog :visible.sync="dialogVisible">
      Content
    </my-dialog>

    <!-- With explicit events (more control) -->
    <my-dialog
      :visible="dialogVisible"
      @update:visible="dialogVisible = $event"
      @close="handleDialogClose"
      @confirm="handleDialogConfirm"
    >
      Content
    </my-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      dialogVisible: false
    }
  },
  methods: {
    handleDialogClose() {
      this.dialogVisible = false
      // Additional cleanup
    },
    handleDialogConfirm(result) {
      console.log('Confirmed with:', result)
      this.dialogVisible = false
    }
  }
}
</script>
```

## Common Pitfalls

### Pitfall 1: Overusing .sync

```vue
<!-- ❌ AVOID: Too many .sync props make data flow unclear -->
<complex-form
  :field1.sync="form.field1"
  :field2.sync="form.field2"
  :field3.sync="form.field3"
  :field4.sync="form.field4"
  :field5.sync="form.field5"
></complex-form>

<!-- ✅ BETTER: Use explicit events for complex forms -->
<complex-form
  :value="form"
  @change="handleFormChange"
  @submit="handleSubmit"
></complex-form>
```

### Pitfall 2: Using .sync for Everything

```vue
<!-- ❌ AVOID: .sync not needed for read-only props -->
<user-card :name.sync="user.name"></user-card>

<!-- ✅ BETTER: Just use one-way binding for display -->
<user-card :name="user.name"></user-card>
```

## Best Practices

1. **Use .sync** for simple two-way bindings (modals, accordions, toggles)
2. **Prefer explicit events** for complex forms and data flow
3. **Use .prop** when binding to DOM properties directly
4. **Use .camel** only for DOM templates (not in .vue files)
5. **Document props** that use `.sync` in component comments

## Reference

- [Vue 2 Guide - Two-way Binding with .sync](https://vue2.docs.vuejs.org/v2/guide/components-custom-events.html#sync-Modifier)
- [Vue 2 API - v-bind](https://vue2.docs.vuejs.org/v2/api/#v-bind)
