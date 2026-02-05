---
title: Default and Named Slots in Vue 2
impact: LOW
impactDescription: Slots enable content distribution with default and named slot outlets
type: capability
tags:
  - vue2.6.14
  - slots
  - content-distribution
  - default-slot
  - named-slots
---

# Default and Named Slots in Vue 2

Impact: LOW - Slots enable flexible content distribution in components. Default slots provide fallback content, named slots allow multiple content outlets.

## Task Checklist

- [ ] Use `<slot>` element to create slot outlets
- [ ] Use `slot` attribute to target named slots
- [ ] Provide default content between `<slot>` tags
- [ ] Fallback to default slot when no content provided

## Default Slot (Fallback Content)

```vue
<!-- SubmitButton.vue -->
<template>
  <button class="btn">
    <!-- Default slot with fallback content -->
    <slot>
      Submit
    </slot>
  </button>
</template>

<!-- Usage: with default content -->
<SubmitButton />
<!-- Renders: <button class="btn">Submit</button> -->

<!-- Usage: with custom content -->
<SubmitButton>
  Save Changes
</SubmitButton>
<!-- Renders: <button class="btn">Save Changes</button> -->
```

## Named Slots

```vue
<!-- BaseLayout.vue -->
<template>
  <div class="layout">
    <!-- Named slot for header -->
    <header class="header">
      <slot name="header">
        <h1>Default Header</h1>
      </slot>
    </header>

    <!-- Default slot for main content -->
    <main class="content">
      <slot></slot>
    </main>

    <!-- Named slot for footer -->
    <footer class="footer">
      <slot name="footer">
        <p>&copy; 2024</p>
      </slot>
    </footer>
  </div>
</template>

<!-- Usage -->
<BaseLayout>
  <template #header>
    <h1>My Custom Header</h1>
  </template>

  <template #default>
    <p>My main content goes here</p>
  </template>

  <template #footer>
    <p>Custom footer text</p>
  </template>
</BaseLayout>
```

## Multiple Named Slots

```vue
<!-- Card.vue -->
<template>
  <div class="card">
    <!-- Named slots for different sections -->
    <div class="card-header">
      <slot name="header">
        <h3>Card Title</h3>
      </slot>
    </div>

    <div class="card-body">
      <!-- Default slot for body content -->
      <slot></slot>
    </div>

    <div class="card-footer">
      <slot name="footer">
        <button>Action</button>
      </slot>
    </div>
  </div>
</template>

<!-- Usage -->
<Card>
  <template #header>
    <h2>Custom Title</h2>
  </template>

  <template #default>
    <p>Card body content</p>
  </template>

  <template #footer>
    <button>Custom Action</button>
  </template>
</Card>
```

## Old Syntax (Still Works in Vue 2)

```vue
<!-- Parent Component - Old slot syntax -->
<MyComponent>
  <h2 slot="header">Title</h2>
  <p>Default content</p>
  <p slot="footer">Footer content</p>
</MyComponent>

<!-- Child Component -->
<template>
  <div>
    <slot name="header">
      <h1>Default Header</h1>
    </slot>
    <slot></slot>
    <slot name="footer">
      <p>Default Footer</p>
    </slot>
  </div>
</template>
```

## v-slot Shorthand (Vue 2.6.0+)

> **Note**: The `#` shorthand for `v-slot` was introduced in Vue 2.6.0. For Vue 2.5.x and earlier, use `slot-scope` for scoped slots.

```vue
<!-- Using # shorthand for v-slot (Vue 2.6.0+) -->
<BaseLayout>
  <template #header>
    <h1>Header</h1>
  </template>

  <!-- #default is optional for default slot -->
  <template #default>
    <p>Main content</p>
  </template>

  <template #footer>
    <p>Footer</p>
  </template>
</BaseLayout>
```

## Dynamic Slot Names

```vue
<script>
export default {
  data() {
    return {
      currentSlot: 'header'
    }
  }
}
</script>

<template>
  <div>
    <!-- Dynamic slot name -->
    <template #[currentSlot]>
      <h1>Dynamic Content</h1>
    </template>
  </div>
</template>
```

## Slots Without Fallback

```vue
<!-- Child.vue -->
<template>
  <div>
    <!-- Slot without fallback - renders nothing if no content -->
    <slot name="optional"></slot>

    <!-- Check if slot has content -->
    <slot name="optional">
      <!-- This only renders if parent provides content -->
      <div class="optional-content">
        <slot name="optional"></slot>
      </div>
    </slot>
  </div>
</template>

<!-- Usage without optional slot -->
<Child>
  <!-- optional slot remains empty -->
</Child>
```

## Compiling Fallback (v-if with $slots)

In Vue 2, you can check if slot has content:

```vue
<script>
export default {
  computed: {
    hasHeaderSlot() {
      return !!this.$slots.header
    },
    hasDefaultSlot() {
      return !!this.$slots.default
    }
  }
}
</script>

<template>
  <div>
    <div v-if="hasHeaderSlot" class="header">
      <slot name="header"></slot>
    </div>

    <div v-else class="header">
      <h1>Default Header</h1>
    </div>
  </div>
</template>
```

## Reference

- [Vue 2 Guide - Slots](https://v2.vuejs.org/v2/guide/components-slots.html)
