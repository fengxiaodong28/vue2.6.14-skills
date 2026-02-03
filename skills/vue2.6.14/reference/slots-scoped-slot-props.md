---
title: Scoped Slots with Slot Props in Vue 2
impact: MEDIUM
impactDescription: Scoped slots pass data from child to parent for custom rendering
type: capability
tags:
  - vue2
  - slots
  - scoped-slots
  - slot-props
  - component-patterns
---

# Scoped Slots with Slot Props in Vue 2

Impact: MEDIUM - Scoped slots allow child components to pass data to parent for custom rendering. Essential for reusable components.

## Task Checklist

- [ ] Use `v-slot` directive with slot prop name for scoped slots
- [ ] Destructure slot props for cleaner code
- [ ] Provide meaningful data through slot props

## Basic Scoped Slot

```vue
<!-- UserList.vue (Child) -->
<template>
  <div class="user-list">
    <!-- Scoped slot: passes user and index to parent -->
    <slot
      v-for="(user, index) in users"
      :user="user"
      :index="index"
    ></slot>
  </div>
</template>

<script>
export default {
  props: {
    users: {
      type: Array,
      required: true
    }
  }
}
</script>
```

```vue
<!-- Parent.vue -->
<template>
  <div>
    <UserList :users="users">
      <!-- Default scoped slot receives slot props -->
      <template #default="{ user, index }">
        <div class="user-item">
          {{ index + 1 }}. {{ user.name }}
        </div>
      </template>
    </UserList>
  </div>
</template>

<script>
export default {
  data() {
    return {
      users: [
        { id: 1, name: 'Alice' },
        { id: 2, name: 'Bob' }
      ]
    }
  }
}
</script>
```

## Named Scoped Slots

```vue
<!-- DataTable.vue (Child) -->
<template>
  <table>
    <thead>
      <tr>
        <!-- Named scoped slot for header -->
        <slot name="header" :columns="columns"></slot>
      </tr>
    </thead>
    <tbody>
      <tr v-for="row in data" :key="row.id">
        <!-- Named scoped slot for rows -->
        <slot name="row" :row="row"></slot>
      </tr>
    </tbody>
  </table>
</template>

<script>
export default {
  props: {
    columns: Array,
    data: Array
  }
}
</script>
```

```vue
<!-- Parent.vue -->
<template>
  <DataTable :columns="columns" :data="tableData">
    <!-- Header slot -->
    <template #header="{ columns }">
      <th v-for="col in columns" :key="col.key">
        {{ col.label }}
      </th>
    </template>

    <!-- Row slot -->
    <template #row="{ row }">
      <td>{{ row.name }}</td>
      <td>{{ row.email }}</td>
      <td>{{ row.age }}</td>
    </template>
  </DataTable>
</template>
```

## Slot Props Destructuring

```vue
<template>
  <!-- Destructure slot props directly -->
  <UserList :users="users">
    <template #default="{ user, index }">
      <span>{{ index }}: {{ user.name }}</span>
    </template>
  </UserList>
</template>
```

## Old Syntax (Still Works)

```vue
<!-- Old slot-scope syntax (Vue 2.5+) -->
<template>
  <UserList :users="users">
    <template slot-scope="{ user, index }">
      <span>{{ index }}: {{ user.name }}</span>
    </template>
  </UserList>
</template>
```

## Scope Attribute (Oldest Syntax)

```vue
<!-- Even older scope attribute (still supported) -->
<template>
  <UserList :users="users">
    <template scope="props">
      <span>{{ props.index }}: {{ props.user.name }}</span>
    </template>
  </UserList>
</template>
```

## Real-World Example: Reusable List

```vue
<!-- SelectList.vue -->
<template>
  <div class="select-list">
    <slot
      v-for="item in items"
      :item="item"
      :selected="isSelected(item)"
      :select="() => select(item)"
      :deselect="() => deselect(item)"
      name="item"
    >
      <!-- Default slot content -->
      {{ item.label }}
    </slot>
  </div>
</template>

<script>
export default {
  props: {
    items: Array,
    value: [String, Number, Array]
  },
  methods: {
    isSelected(item) {
      if (Array.isArray(this.value)) {
        return this.value.includes(item.value)
      }
      return this.value === item.value
    },
    select(item) {
      this.$emit('input', item.value)
    },
    deselect(item) {
      this.$emit('input', null)
    }
  }
}
</script>
```

```vue
<!-- Usage -->
<template>
  <SelectList :items="options" v-model="selected">
    <template #item="{ item, selected, select, deselect }">
      <div
        class="option"
        :class="{ active: selected }"
        @click="selected ? deselect() : select()"
      >
        <span>{{ item.label }}</span>
        <span v-if="selected">✓</span>
      </div>
    </template>
  </SelectList>
</template>
```

## Reference

- [Vue 2 Scoped Slots](https://vue2.docs.vuejs.org/v2/guide/components-slots.html#Scoped-Slots)
