---
title: Missing :key on v-for Causes Rendering Issues
impact: HIGH
impactDescription: Missing unique key attribute in v-for causes incorrect DOM updates and performance issues
type: best-practice
tags:
  - vue2.6.14
  - v-for
  - key-attribute
  - rendering
  - performance
---

# Missing :key on v-for Causes Rendering Issues

Impact: HIGH - Not using a unique `:key` attribute with `v-for` causes Vue to reuse DOM elements inefficiently, leading to incorrect rendering, loss of component state, and poor performance.

## Task Checklist

- [ ] Always use `:key` with `v-for`
- [ ] Use unique identifier (like ID) as key value
- [ ] Avoid using array index as key when list can change order
- [ ] Use unique values for sibling v-for directives

## Incorrect: Missing Key

```vue
<template>
  <div>
    <!-- WRONG: No key attribute -->
    <li v-for="item in items">
      {{ item.name }}
    </li>

    <!-- WRONG: Using index as key (when list can change) -->
    <li v-for="(item, index) in items" :key="index">
      {{ item.name }}
    </li>
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
  }
}
</script>
```

## Correct: Using Unique Identifier

```vue
<template>
  <div>
    <!-- CORRECT: Using unique ID -->
    <li v-for="item in items" :key="item.id">
      {{ item.name }}
    </li>

    <!-- CORRECT: Using unique identifier for objects -->
    <div v-for="user in users" :key="user.email">
      {{ user.name }}
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
      users: [
        { email: 'user1@example.com', name: 'User 1' },
        { email: 'user2@example.com', name: 'User 2' }
      ]
    }
  }
}
</script>
```

## When Index as Key IS Acceptable

```vue
<template>
  <!-- OK: Index as key when list is static and won't change -->
  <li v-for="(item, index) in staticList" :key="index">
    {{ item }}
  </li>
</template>

<script>
export default {
  data() {
    return {
      // Static list that won't be reordered
      staticList: ['First', 'Second', 'Third']
    }
  }
}
</script>
```

## Nested v-for with Keys

```vue
<template>
  <div v-for="category in categories" :key="category.id">
    <h3>{{ category.name }}</h3>
    <!-- Inner loop also needs keys -->
    <div
      v-for="item in category.items"
      :key="`${category.id}-${item.id}`"
    >
      {{ item.name }}
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      categories: [
        {
          id: 'c1',
          name: 'Category 1',
          items: [
            { id: 'i1', name: 'Item 1' },
            { id: 'i2', name: 'Item 2' }
          ]
        }
      ]
    }
  }
}
</script>
```

## Key with Components

```vue
<template>
  <!-- Components also need :key -->
  <TodoItem
    v-for="todo in todos"
    :key="todo.id"
    :todo="todo"
    @toggle="toggleTodo(todo.id)"
  />
</template>

<script>
import TodoItem from './TodoItem.vue'

export default {
  components: { TodoItem },
  data() {
    return {
      todos: [
        { id: 1, text: 'Learn Vue', done: false },
        { id: 2, text: 'Build something', done: false }
      ]
    }
  },
  methods: {
    toggleTodo(id) {
      const todo = this.todos.find(t => t.id === id)
      if (todo) todo.done = !todo.done
    }
  }
}
</script>
```

## Common Issues Without Keys

1. **Lost Input Focus**: Reordering list without keys causes input focus to jump
2. **Incorrect State**: Component state gets mixed up
3. **Animation Issues**: Transitions don't work correctly
4. **Performance**: Vue can't efficiently update DOM

## Key Best Practices

```vue
<template>
  <!-- Use unique ID (best) -->
  <div v-for="item in items" :key="item.id">

  <!-- Use unique combination (if no ID) -->
  <div v-for="item in items" :key="`${item.type}-${item.name}`">

  <!-- Avoid: index when list changes -->
  <div v-for="(item, index) in items" :key="index">

  <!-- Avoid: random values (new key on each render) -->
  <div v-for="item in items" :key="Math.random()">
</template>
```

## Reference

- [Vue 2 List Rendering - :key](https://vue2.docs.vuejs.org/v2/guide/list.html#Maintaining-State)
