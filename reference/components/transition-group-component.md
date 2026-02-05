---
title: transition-group Built-in Component in Vue 2
impact: MEDIUM
impactDescription: transition-group provides enter/leave transitions for multiple elements or components in a list with proper positioning animations
type: capability
tags:
  - vue2.6.14
  - transition-group
  - built-in-components
  - list-transitions
  - animations
  - v-for-transitions
---

# transition-group Built-in Component in Vue 2

Impact: MEDIUM - The `<transition-group>` component provides transitions for multiple items in a list. Unlike `<transition>` which only handles single elements, `<transition-group>` renders a real DOM element (default `<span>`) and properly animates items being added, removed, or reordered in a list with FLIP animations.

## Task Checklist

- [ ] Use `<transition-group>` for list item transitions
- [ ] Always provide a `:key` for each item
- [ ] Use CSS classes or JavaScript hooks for animations
- [ ] Specify `tag` prop to change wrapper element type
- [ ] Understand v-move class for position changes during reordering

## Basic Usage

```vue
<template>
  <div>
    <button @click="add">Add</button>
    <button @click="remove">Remove</button>

    <!-- Basic list transition -->
    <transition-group name="list">
      <span
        v-for="item in items"
        :key="item.id"
        class="list-item"
      >
        {{ item.text }}
      </span>
    </transition-group>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: [
        { id: 1, text: 'Item 1' },
        { id: 2, text: 'Item 2' },
        { id: 3, text: 'Item 3' }
      ],
      idCounter: 4
    }
  },
  methods: {
    add() {
      this.items.push({
        id: this.idCounter++,
        text: `Item ${this.idCounter - 1}`
      })
    },
    remove() {
      this.items.pop()
    }
  }
}
</script>

<style>
/* Enter/leave transitions */
.list-enter-active, .list-leave-active {
  transition: all 0.5s;
}
.list-enter, .list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

/* Move transition for reordering */
.list-move {
  transition: transform 0.5s;
}

/* Ensure leaving items are taken out of layout */
.list-leave-active {
  position: absolute;
}
</style>
```

## Tag Prop - Custom Wrapper Element

```vue
<template>
  <div>
    <!-- Default renders <span> -->
    <transition-group name="list">
      <div v-for="item in items" :key="item.id">
        {{ item.name }}
      </div>
    </transition-group>

    <!-- Render as <ul> -->
    <transition-group name="list" tag="ul">
      <li v-for="item in items" :key="item.id">
        {{ item.name }}
      </li>
    </transition-group>

    <!-- Render as <div> -->
    <transition-group name="list" tag="div" class="list-container">
      <div v-for="item in items" :key="item.id" class="item">
        {{ item.name }}
      </div>
    </transition-group>
  </div>
</template>
```

## CSS Transition Classes

The `<transition-group>` component adds a special `v-move` class for position changes:

```vue
<template>
  <transition-group name="flip-list">
    <div
      v-for="item in items"
      :key="item.id"
      class="flip-list-item"
    >
      {{ item.number }}
    </div>
  </transition-group>
</template>

<style>
/* Enter/leave transitions */
.flip-list-enter-active,
.flip-list-leave-active {
  transition: all 0.5s;
}
.flip-list-enter,
.flip-list-leave-to {
  opacity: 0;
  transform: translateY(30px);
}

/* Move transition for position changes (FLIP animation) */
.flip-list-move {
  transition: transform 0.5s;
}

/* Important: Ensure leaving items are removed from layout */
.flip-list-leave-active {
  position: absolute;
  width: 100%;
}
</style>
```

## List Reordering Animation

```vue
<template>
  <div class="reorder-demo">
    <button @click="shuffle">Shuffle</button>
    <button @click="add">Add</button>
    <button @click="remove">Remove</button>

    <transition-group name="list" tag="ul" class="reorder-list">
      <li
        v-for="item in items"
        :key="item.id"
        class="reorder-item"
      >
        {{ item.text }}
      </li>
    </transition-group>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: [
        { id: 1, text: 'Item 1' },
        { id: 2, text: 'Item 2' },
        { id: 3, text: 'Item 3' },
        { id: 4, text: 'Item 4' },
        { id: 5, text: 'Item 5' }
      ],
      nextId: 6
    }
  },
  methods: {
    shuffle() {
      // Fisher-Yates shuffle
      for (let i = this.items.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1))
        ;[this.items[i], this.items[j]] = [this.items[j], this.items[i]]
      }
    },
    add() {
      const newItem = {
        id: this.nextId++,
        text: `Item ${this.nextId - 1}`
      }
      // Insert at random position
      const index = Math.floor(Math.random() * (this.items.length + 1))
      this.items.splice(index, 0, newItem)
    },
    remove() {
      if (this.items.length > 0) {
        const index = Math.floor(Math.random() * this.items.length)
        this.items.splice(index, 1)
      }
    }
  }
}
</script>

<style>
.reorder-list {
  list-style: none;
  padding: 0;
  position: relative;
}

.reorder-item {
  width: 100%;
  padding: 15px;
  margin: 5px 0;
  background: #42b983;
  color: white;
  border-radius: 4px;
}

/* Enter/leave transitions */
.list-enter-active,
.list-leave-active {
  transition: all 0.5s;
}

.list-enter {
  opacity: 0;
  transform: translateX(-100%);
}

.list-leave-to {
  opacity: 0;
  transform: translateX(100%);
}

/* Move transition for reordering */
.list-move {
  transition: transform 0.5s;
}

/* Ensure smooth layout during leave transition */
.list-leave-active {
  position: absolute;
  width: 100%;
}
</style>
```

## Staggered List Animations

```vue
<template>
  <div>
    <button @click="addItem">Add Item</button>

    <transition-group
      name="stagger"
      tag="ul"
      class="stagger-list"
    >
      <li
        v-for="(item, index) in items"
        :key="item.id"
        :style="{ transitionDelay: `${index * 50}ms` }"
        class="stagger-item"
      >
        {{ item.text }}
      </li>
    </transition-group>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: [
        { id: 1, text: 'Item 1' },
        { id: 2, text: 'Item 2' },
        { id: 3, text: 'Item 3' }
      ],
      nextId: 4
    }
  },
  methods: {
    addItem() {
      this.items.push({
        id: this.nextId++,
        text: `Item ${this.nextId - 1}`
      })
    }
  }
}
</script>

<style>
.stagger-list {
  list-style: none;
  padding: 0;
}

.stagger-item {
  padding: 10px;
  margin: 5px 0;
  background: #42b983;
  color: white;
  border-radius: 4px;
}

.stagger-enter-active {
  transition: all 0.5s;
}

.stagger-enter {
  opacity: 0;
  transform: translateX(-30px);
}

/* Leave doesn't need stagger */
.stagger-leave-active {
  transition: all 0.3s;
}

.stagger-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

.stagger-move {
  transition: transform 0.5s;
}

.stagger-leave-active {
  position: absolute;
  width: 100%;
}
</style>
```

## JavaScript Hooks

```vue
<template>
  <div>
    <button @click="shuffle">Shuffle</button>

    <transition-group
      name="list"
      @before-enter="beforeEnter"
      @enter="enter"
      @after-enter="afterEnter"
      @enter-cancelled="enterCancelled"
      @before-leave="beforeLeave"
      @leave="leave"
      @after-leave="afterLeave"
      @leave-cancelled="leaveCancelled"
    >
      <div
        v-for="item in items"
        :key="item.id"
        class="js-item"
      >
        {{ item.text }}
      </div>
    </transition-group>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: [
        { id: 1, text: 'Item 1' },
        { id: 2, text: 'Item 2' },
        { id: 3, text: 'Item 3' }
      ]
    }
  },
  methods: {
    shuffle() {
      this.items.sort(() => Math.random() - 0.5)
    },
    beforeEnter(el) {
      el.style.opacity = 0
      el.style.transform = 'translateY(20px)'
    },
    enter(el, done) {
      const delay = el.dataset.index * 100
      setTimeout(() => {
        el.style.transition = 'all 0.3s'
        el.style.opacity = 1
        el.style.transform = 'translateY(0)'
        setTimeout(done, 300)
      }, delay)
    },
    afterEnter(el) {
      el.style.transition = ''
    },
    enterCancelled(el) {
      // Handle cancellation
    },
    beforeLeave(el) {
      // Before leave
    },
    leave(el, done) {
      el.style.transition = 'all 0.3s'
      el.style.opacity = 0
      el.style.transform = 'translateY(-20px)'
      setTimeout(done, 300)
    },
    afterLeave(el) {
      el.style.transition = ''
    },
    leaveCancelled(el) {
      // Handle cancellation
    }
  }
}
</script>

<style>
.js-item {
  padding: 10px;
  margin: 5px 0;
  background: #42b983;
  color: white;
  border-radius: 4px;
}
</style>
```

## Draggable List with Transitions

```vue
<template>
  <div class="drag-list">
    <transition-group name="drag" tag="ul" class="drag-list-ul">
      <li
        v-for="(item, index) in items"
        :key="item.id"
        draggable="true"
        @dragstart="dragStart(index, $event)"
        @dragover.prevent
        @dragenter.prevent
        @drop="drop(index)"
        class="drag-item"
      >
        {{ item.text }}
      </li>
    </transition-group>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: [
        { id: 1, text: 'Drag me 1' },
        { id: 2, text: 'Drag me 2' },
        { id: 3, text: 'Drag me 3' },
        { id: 4, text: 'Drag me 4' }
      ],
      dragIndex: null
    }
  },
  methods: {
    dragStart(index, event) {
      this.dragIndex = index
      event.dataTransfer.effectAllowed = 'move'
      event.dataTransfer.setData('text/html', event.target.innerHTML)
    },
    drop(index) {
      const draggedItem = this.items[this.dragIndex]
      this.items.splice(this.dragIndex, 1)
      this.items.splice(index, 0, draggedItem)
      this.dragIndex = null
    }
  }
}
</script>

<style>
.drag-list-ul {
  list-style: none;
  padding: 0;
}

.drag-item {
  padding: 15px;
  margin: 5px 0;
  background: #42b983;
  color: white;
  border-radius: 4px;
  cursor: move;
}

.drag-enter-active,
.drag-leave-active {
  transition: all 0.3s;
}

.drag-enter,
.drag-leave-to {
  opacity: 0;
  transform: scale(0.9);
}

.drag-move {
  transition: transform 0.3s;
}

.drag-leave-active {
  position: absolute;
  width: 100%;
}
</style>
```

## Grid Layout Transitions

```vue
<template>
  <div class="grid-demo">
    <button @click="add">Add</button>
    <button @click="remove">Remove</button>
    <button @click="shuffle">Shuffle</button>

    <transition-group name="grid" tag="div" class="grid-container">
      <div
        v-for="item in items"
        :key="item.id"
        class="grid-item"
      >
        {{ item.text }}
      </div>
    </transition-group>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: Array.from({ length: 9 }, (_, i) => ({
        id: i + 1,
        text: `${i + 1}`
      })),
      nextId: 10
    }
  },
  methods: {
    add() {
      this.items.push({
        id: this.nextId++,
        text: `${this.nextId - 1}`
      })
    },
    remove() {
      if (this.items.length > 0) {
        this.items.pop()
      }
    },
    shuffle() {
      for (let i = this.items.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1))
        ;[this.items[i], this.items[j]] = [this.items[j], this.items[i]]
      }
    }
  }
}
</script>

<style>
.grid-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
  gap: 10px;
  position: relative;
}

.grid-item {
  aspect-ratio: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #42b983;
  color: white;
  font-size: 24px;
  font-weight: bold;
  border-radius: 8px;
}

.grid-enter-active,
.grid-leave-active {
  transition: all 0.5s;
}

.grid-enter {
  opacity: 0;
  transform: scale(0);
}

.grid-leave-to {
  opacity: 0;
  transform: scale(0);
}

.grid-move {
  transition: transform 0.5s;
}

.grid-leave-active {
  position: absolute;
  width: calc(100% / 3 - 10px);
}
</style>
```

## Animated Todo List

```vue
<template>
  <div class="todo-app">
    <form @submit.prevent="addTodo">
      <input
        v-model="newTodoText"
        placeholder="Add a todo"
        required
      >
      <button type="submit">Add</button>
    </form>

    <transition-group name="todo" tag="ul" class="todo-list">
      <li
        v-for="todo in todos"
        :key="todo.id"
        class="todo-item"
        :class="{ completed: todo.completed }"
      >
        <input
          type="checkbox"
          v-model="todo.completed"
        >
        <span>{{ todo.text }}</span>
        <button @click="removeTodo(todo.id)">×</button>
      </li>
    </transition-group>
  </div>
</template>

<script>
export default {
  data() {
    return {
      newTodoText: '',
      todos: [
        { id: 1, text: 'Learn Vue.js', completed: false },
        { id: 2, text: 'Build an app', completed: false },
        { id: 3, text: 'Deploy to production', completed: false }
      ],
      nextId: 4
    }
  },
  methods: {
    addTodo() {
      if (this.newTodoText.trim()) {
        this.todos.unshift({
          id: this.nextId++,
          text: this.newTodoText,
          completed: false
        })
        this.newTodoText = ''
      }
    },
    removeTodo(id) {
      const index = this.todos.findIndex(t => t.id === id)
      this.todos.splice(index, 1)
    }
  }
}
</script>

<style>
.todo-app {
  max-width: 400px;
  margin: 0 auto;
}

.todo-app form {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.todo-app input[type="text"] {
  flex: 1;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.todo-list {
  list-style: none;
  padding: 0;
  position: relative;
}

.todo-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px;
  margin: 5px 0;
  background: white;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.todo-item.completed span {
  text-decoration: line-through;
  opacity: 0.6;
}

.todo-item button {
  margin-left: auto;
  background: #ff4444;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

/* Transitions */
.todo-enter-active {
  transition: all 0.3s ease-out;
}

.todo-leave-active {
  transition: all 0.3s ease-in;
}

.todo-enter {
  opacity: 0;
  transform: translateX(-30px);
}

.todo-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

.todo-move {
  transition: transform 0.3s;
}

.todo-leave-active {
  position: absolute;
  width: 100%;
}
</style>
```

## transition-group Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `tag` | String | `'span'` | The wrapper element tag to render |
| `name` | String | `'v'` | Prefix for transition CSS classes |
| `appear` | Boolean | `false` | Apply transition on initial render |
| `mode` | String | - | Not used in transition-group (items can overlap) |
| `duration` | Number \| Object | - | Manual duration specification |
| `enter-class` | String | - | Deprecated in Vue 2.6.0+ |
| `leave-class` | String | - | Deprecated in Vue 2.6.0+ |
| `enter-to-class` | String | - | Deprecated in Vue 2.6.0+ |
| `leave-to-class` | String | - | Deprecated in Vue 2.6.0+ |
| `enter-active-class` | String | - | Custom class for enter active state |
| `leave-active-class` | String | - | Custom class for leave active state |
| `move-class` | String | - | Custom class for move state |
| `css` | Boolean | `true` | Whether to apply CSS transition classes |

## Important: Keys Are Required

```vue
<!-- ❌ WRONG: No keys -->
<transition-group name="list">
  <li v-for="item in items">{{ item }}</li>
</transition-group>

<!-- ✅ CORRECT: Always use :key -->
<transition-group name="list">
  <li v-for="item in items" :key="item.id">{{ item.text }}</li>
</transition-group>
```

## Common Patterns

### 1. Animated Data Table

```vue
<template>
  <div class="data-table">
    <button @click="sort('name')">Sort by Name</button>
    <button @click="sort('value')">Sort by Value</button>

    <table>
      <transition-group name="table" tag="tbody">
        <tr v-for="row in sortedRows" :key="row.id">
          <td>{{ row.name }}</td>
          <td>{{ row.value }}</td>
          <td>{{ row.date }}</td>
        </tr>
      </transition-group>
    </table>
  </div>
</template>

<script>
export default {
  data() {
    return {
      rows: [
        { id: 1, name: 'Alice', value: 100, date: '2024-01-01' },
        { id: 2, name: 'Bob', value: 200, date: '2024-01-02' },
        { id: 3, name: 'Charlie', value: 150, date: '2024-01-03' }
      ],
      sortBy: 'name'
    }
  },
  computed: {
    sortedRows() {
      return [...this.rows].sort((a, b) => {
        if (a[this.sortBy] < b[this.sortBy]) return -1
        if (a[this.sortBy] > b[this.sortBy]) return 1
        return 0
      })
    }
  },
  methods: {
    sort(column) {
      this.sortBy = column
    }
  }
}
</script>

<style>
table {
  width: 100%;
  border-collapse: collapse;
}

td {
  padding: 10px;
  border-bottom: 1px solid #ddd;
}

.table-move {
  transition: transform 0.5s;
}
</style>
```

### 2. Animated Image Gallery

```vue
<template>
  <div class="gallery">
    <transition-group name="gallery" tag="div" class="gallery-grid">
      <div
        v-for="image in images"
        :key="image.id"
        class="gallery-item"
        @click="removeImage(image.id)"
      >
        <img :src="image.url" :alt="image.title">
        <div class="overlay">Click to remove</div>
      </div>
    </transition-group>
  </div>
</template>

<script>
export default {
  data() {
    return {
      images: [
        { id: 1, url: '/image1.jpg', title: 'Image 1' },
        { id: 2, url: '/image2.jpg', title: 'Image 2' },
        { id: 3, url: '/image3.jpg', title: 'Image 3' }
      ]
    }
  },
  methods: {
    removeImage(id) {
      const index = this.images.findIndex(img => img.id === id)
      this.images.splice(index, 1)
    }
  }
}
</script>

<style>
.gallery-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 15px;
  position: relative;
}

.gallery-item {
  position: relative;
  aspect-ratio: 1;
  overflow: hidden;
  border-radius: 8px;
  cursor: pointer;
}

.gallery-item img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.gallery-enter-active {
  transition: all 0.5s;
}

.gallery-leave-active {
  transition: all 0.5s;
}

.gallery-enter {
  opacity: 0;
  transform: scale(0.5);
}

.gallery-leave-to {
  opacity: 0;
  transform: scale(0.5);
}

.gallery-move {
  transition: transform 0.5s;
}

.gallery-leave-active {
  position: absolute;
  width: calc(100% / 3 - 15px);
}
</style>
```

## Best Practices

1. **Always use unique `:key` values** - Required for Vue to track elements
2. **Use `position: absolute` for leaving items** - Ensures smooth layout changes
3. **Define `v-move` class** - Animates position changes during reordering
4. **Consider using `<ul>` with `tag` prop** - Semantic HTML for lists
5. **Set `transition-delay` for stagger effects** - Creates cascading animations
6. **Test performance with large lists** - Consider pagination for 100+ items
7. **Avoid deeply nested structures** - Can cause layout thrashing

## Performance Considerations

```vue
<!-- For large lists, consider virtual scrolling -->
<template>
  <transition-group name="list" tag="div">
    <div
      v-for="item in visibleItems"
      :key="item.id"
    >
      {{ item.text }}
    </div>
  </transition-group>
</template>

<script>
export default {
  data() {
    return {
      allItems: [], // All 1000+ items
      visibleItems: [] // Only visible items
    }
  },
  methods: {
    updateVisible() {
      // Only show items currently in viewport
      const scrollTop = this.$refs.container.scrollTop
      const viewportHeight = this.$refs.container.clientHeight

      this.visibleItems = this.allItems.filter(item => {
        return item.offset >= scrollTop &&
               item.offset <= scrollTop + viewportHeight
      })
    }
  }
}
</script>
```

## Reference

- [Vue 2 API - transition-group](https://v2.vuejs.org/v2/api/#transition-group)
- [Vue 2 Guide - List Transitions](https://v2.vuejs.org/v2/guide/transitions.html#List-Transitions)
