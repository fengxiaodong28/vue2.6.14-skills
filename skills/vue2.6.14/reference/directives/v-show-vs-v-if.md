---
title: v-show vs v-if Conditional Rendering in Vue 2
impact: MEDIUM
impactDescription: v-show toggles display CSS, v-if conditionally renders elements; choose based on use case
type: capability
tags:
  - vue2.6.14
  - v-show
  - v-if
  - conditional-rendering
  - performance
---

# v-show vs v-if Conditional Rendering in Vue 2

Impact: MEDIUM - Vue 2 provides two ways to conditionally render content: `v-show` and `v-if`. Understanding the difference is crucial for performance and component behavior. `v-show` uses CSS display, `v-if` performs actual DOM addition/removal.

## Task Checklist

- [ ] Use `v-show` for frequent toggle with low initial render cost
- [ ] Use `v-if` for infrequent toggles or when initial render cost is high
- [ ] Use `v-if` when conditional block requires different lifecycle hooks
- [ ] Remember that `v-if` has higher priority than `v-show` when used together
- [ ] Consider `v-show` for simple toggle visibility

## Key Differences

| Feature | v-show | v-if |
|---------|--------|------|
| CSS | Toggles `display: none` | Adds/removes from DOM |
| Initial Cost | Always renders (higher initial cost) | Only renders if true (lower initial cost if false) |
| Toggle Cost | Low (just CSS) | Higher (DOM manipulation) |
| Lifecycle | Always runs lifecycle hooks | Runs hooks only when rendered |
| Event Listeners | Always present | Added/removed with element |
| Best For | Frequent toggles | Infrequent toggles |

## v-show: CSS-Based Visibility

```vue
<template>
  <div>
    <button @click="isVisible = !isVisible">
      Toggle (v-show)
    </button>

    <!-- Element always in DOM, just hidden with CSS -->
    <div v-show="isVisible" class="panel">
      This panel uses v-show.
      It's always in the DOM, just hidden with display: none.
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isVisible: true
    }
  }
}
</script>

<style scoped>
.panel {
  padding: 20px;
  background: #f0f0f0;
}
/* When v-show="false", Vue adds: style="display: none;" */
</style>
```

## v-if: Conditional DOM Rendering

```vue
<template>
  <div>
    <button @click="isVisible = !isVisible">
      Toggle (v-if)
    </button>

    <!-- Element added to/removed from DOM -->
    <div v-if="isVisible" class="panel">
      This panel uses v-if.
      It's only in the DOM when isVisible is true.
      Created and destroyed each time it toggles.
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isVisible: true
    }
  },
  watch: {
    isVisible(newVal) {
      if (newVal) {
        console.log('Panel created and mounted')
      } else {
        console.log('Panel destroyed')
      }
    }
  }
}
</script>
```

## Performance Comparison

### Frequent Toggles (Use v-show)

```vue
<template>
  <div>
    <!-- GOOD: v-show for frequent toggles -->
    <div v-show="showDetails" class="details-panel">
      <!-- Heavy content that toggles often -->
      <p>Lots of content here...</p>
      <data-table :data="largeDataSet"></data-table>
    </div>

    <button @click="showDetails = !showDetails">
      Toggle Details
    </button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      showDetails: false,
      largeDataSet: [] // Thousands of items
    }
  }
}
</script>
```

### Infrequent Toggles (Use v-if)

```vue
<template>
  <div>
    <!-- GOOD: v-if for infrequent toggles -->
    <modal v-if="showModal" @close="showModal = false">
      <!-- Modal content -->
    </modal>

    <button @click="showModal = true">
      Open Modal (rarely used)
    </button>
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
```

## Lifecycle Hook Behavior

```vue
<template>
  <div>
    <button @click="toggleA">Toggle v-if</button>
    <button @click="toggleB">Toggle v-show</button>

    <!-- v-if: Hooks fire on each toggle -->
    <div v-if="showA" class="box-a">
      Element with v-if
    </div>

    <!-- v-show: Hooks fire once, never again -->
    <div v-show="showB" class="box-b">
      Element with v-show
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      showA: true,
      showB: true
    }
  },
  methods: {
    toggleA() {
      this.showA = !this.showA
      // Every toggle: created → mounted → beforeDestroy → destroyed
    },
    toggleB() {
      this.showB = !this.showB
      // Hooks only fire once on initial render
    }
  },
  components: {
    BoxA: {
      template: '<div class="box-a">v-if element</div>',
      created() {
        console.log('v-if: created')
      },
      mounted() {
        console.log('v-if: mounted')
      },
      beforeDestroy() {
        console.log('v-if: beforeDestroy')
      },
      destroyed() {
        console.log('v-if: destroyed')
      }
    },
    BoxB: {
      template: '<div class="box-b">v-show element</div>',
      created() {
        console.log('v-show: created')
      },
      mounted() {
        console.log('v-show: mounted')
      },
      beforeDestroy() {
        console.log('v-show: beforeDestroy - never called!')
      },
      destroyed() {
        console.log('v-show: destroyed - never called!')
      }
    }
  }
}
</script>
```

## When to Use v-show

**Good use cases:**

```vue
<template>
  <div>
    <!-- 1. Tabs that switch frequently -->
    <tab-panel v-show="activeTab === 'home'">Home</tab-panel>
    <tab-panel v-show="activeTab === 'profile'">Profile</tab-panel>
    <tab-panel v-show="activeTab === 'settings'">Settings</tab-panel>

    <!-- 2. Toggle switches -->
    <label class="toggle">
      <input type="checkbox" v-model="enabled">
      <span v-show="enabled">Enabled</span>
      <span v-show="!enabled">Disabled</span>
    </label>

    <!-- 3. Dropdown menus -->
    <div class="dropdown">
      <button @click="isOpen = !isOpen">Menu</button>
      <div v-show="isOpen" class="dropdown-menu">
        <!-- Menu items -->
      </div>
    </div>

    <!-- 4. Expandable sections (FAQ, accordions) -->
    <div v-for="item in faqItems" :key="item.id">
      <h3 @click="item.expanded = !item.expanded">
        {{ item.question }}
      </h3>
      <div v-show="item.expanded" class="answer">
        {{ item.answer }}
      </div>
    </div>
  </div>
</template>
```

## When to Use v-if

**Good use cases:**

```vue
<template>
  <div>
    <!-- 1. Permission-based rendering -->
    <button v-if="user.isAdmin" @click="deleteUser">
      Delete User
    </button>

    <!-- 2. Resource-dependent content -->
    <error-message v-if="error" :message="error" />
    <loading-spinner v-else-if="loading" />
    <data-view v-else :data="result" />

    <!-- 3. Expensive components (modals, heavy charts) -->
    <heavy-chart v-if="showChart" :data="chartData" />

    <!-- 4. Route-based content -->
    <admin-panel v-if="$route.path.startsWith('/admin')" />
    <user-panel v-else />

    <!-- 5. Feature flags -->
    <new-feature v-if="features.newFeatureEnabled" />
  </div>
</template>
```

## v-else-if and v-else

```vue
<template>
  <div>
    <div v-if="status === 'loading'">
      Loading...
    </div>
    <div v-else-if="status === 'error'">
      Error: {{ errorMessage }}
    </div>
    <div v-else-if="status === 'empty'">
      No data available
    </div>
    <div v-else>
      <!-- Display actual data -->
      <data-list :items="items" />
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      status: 'loading', // loading, error, empty, success
      errorMessage: '',
      items: []
    }
  }
}
</script>
```

## Combining v-show and v-if

```vue
<template>
  <div>
    <!-- v-if has higher priority than v-show -->
    <!-- This won't work as expected: -->
    <div v-if="condition" v-show="visible">
      Content
    </div>

    <!-- Equivalent to: -->
    <div v-if="condition">
      <div v-show="visible">
        Content
      </div>
    </div>

    <!-- Better approach - use computed: -->
    <div v-show="shouldBeVisible">
      Content
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      condition: true,
      visible: true
    }
  },
  computed: {
    shouldBeVisible() {
      return this.condition && this.visible
    }
  }
}
</script>
```

## Using v-if on Template Element

```vue
<template>
  <div>
    <!-- v-if on <template> doesn't create a wrapper element -->
    <template v-if="isLoggedIn">
      <header>Welcome, {{ username }}</header>
      <nav>
        <router-link to="/dashboard">Dashboard</router-link>
        <router-link to="/profile">Profile</router-link>
      </nav>
    </template>

    <!-- v-show on <template> does NOT work -->
    <!-- Use a wrapper element instead -->
    <div v-show="isVisible">
      <p>Content 1</p>
      <p>Content 2</p>
    </div>
  </div>
</template>
```

## Key Consideration with v-if and v-show

```vue
<template>
  <div>
    <!-- When using v-if with v-for, v-for has higher priority -->
    <!-- This is an anti-pattern - see v-for-v-if-priority.md -->

    <!-- Better: Use computed property -->
    <li v-for="item in activeItems" :key="item.id">
      {{ item.name }}
    </li>

    <!-- Or template wrapper -->
    <template v-for="item in items">
      <li v-if="item.isActive" :key="item.id">
        {{ item.name }}
      </li>
    </template>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: [
        { id: 1, name: 'Item 1', isActive: true },
        { id: 2, name: 'Item 2', isActive: false }
      ]
    }
  },
  computed: {
    activeItems() {
      return this.items.filter(item => item.isActive)
    }
  }
}
</script>
```

## Component with v-if (Cleanup Example)

```vue
<template>
  <div>
    <button @click="showComponent = !showComponent">
      Toggle Component
    </button>

    <!-- With v-if, component is destroyed when hidden -->
    <heavy-component v-if="showComponent" @data="handleData" />
  </div>
</template>

<script>
import HeavyComponent from './HeavyComponent.vue'

export default {
  components: { HeavyComponent },
  data() {
    return {
      showComponent: false,
      componentData: null
    }
  },
  methods: {
    handleData(data) {
      this.componentData = data
    }
  }
}
</script>
```

## Decision Tree

```
Should the element be in the DOM when hidden?
│
├─ YES → Use v-show
│  ├─ Frequent toggle needed?
│  │  ├─ YES → v-show (better performance)
│  │  └─ NO → Still v-show (simpler)
│  └─ Example: tabs, dropdowns, accordions
│
└─ NO → Use v-if
   ├─ Expensive to render?
   │  ├─ YES → v-if (render only when needed)
   │  └─ NO → v-if still (cleaner DOM)
   ├─ Needs lifecycle hooks?
   │  └─ v-if (hooks fire on toggle)
   └─ Example: modals, permission-based, feature flags
```

## Real-World Examples

### 1. Tabbed Interface (v-show)

```vue
<template>
  <div class="tabs">
    <div class="tab-buttons">
      <button
        v-for="tab in tabs"
        :key="tab.id"
        @click="activeTab = tab.id"
        :class="{ active: activeTab === tab.id }"
      >
        {{ tab.label }}
      </button>
    </div>

    <div class="tab-content">
      <div v-show="activeTab === 'home'">
        <home-content />
      </div>
      <div v-show="activeTab === 'profile'">
        <profile-content />
      </div>
      <div v-show="activeTab === 'settings'">
        <settings-content />
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      activeTab: 'home',
      tabs: [
        { id: 'home', label: 'Home' },
        { id: 'profile', label: 'Profile' },
        { id: 'settings', label: 'Settings' }
      ]
    }
  }
}
</script>
```

### 2. Modal Dialog (v-if)

```vue
<template>
  <div>
    <button @click="showModal = true">Open Modal</button>

    <!-- Modal only rendered when needed -->
    <modal v-if="showModal" @close="showModal = false">
      <h2>Confirm Action</h2>
      <p>Are you sure you want to proceed?</p>
      <button @click="confirm">Confirm</button>
      <button @click="showModal = false">Cancel</button>
    </modal>
  </div>
</template>

<script>
import Modal from './Modal.vue'

export default {
  components: { Modal },
  data() {
    return {
      showModal: false
    }
  },
  methods: {
    confirm() {
      // Do something
      this.showModal = false
    }
  }
}
</script>
```

### 3: Permission-Based UI (v-if)

```vue
<template>
  <div>
    <!-- Admin-only controls -->
    <div v-if="user.isAdmin" class="admin-controls">
      <button @click="deleteUser">Delete User</button>
      <button @click="banUser">Ban User</button>
    </div>

    <!-- Moderator controls -->
    <div v-else-if="user.isModerator" class="moderator-controls">
      <button @click="flagContent">Flag Content</button>
    </div>

    <!-- Regular user view -->
    <div v-else class="user-view">
      <p>You are viewing as a regular user</p>
    </div>
  </div>
</template>

<script>
export default {
  computed: {
    user() {
      return this.$store.state.auth.user
    }
  }
}
</script>
```

## Common Pitfalls

### Pitfall 1: Using v-show for heavy components

```vue
<!-- ❌ AVOID: v-show for expensive components -->
<heavy-chart v-show="visible" :data="hugeDataset" />
<!-- Chart always rendered, even when hidden -->

<!-- ✅ BETTER: v-if for expensive components -->
<heavy-chart v-if="visible" :data="hugeDataset" />
<!-- Chart only rendered when visible -->
```

### Pitfall 2: Using v-if for frequent toggles

```vue
<!-- ❌ AVOID: v-if for very frequent toggles -->
<div v-if="menuOpen" class="menu">
  <!-- Menu content -->
</div>
<!-- Component destroyed/created on every toggle -->

<!-- ✅ BETTER: v-show for frequent toggles -->
<div v-show="menuOpen" class="menu">
  <!-- Menu content -->
</div>
<!-- Just CSS toggle, much faster -->
```

## Summary

| Use v-show when | Use v-if when |
|-----------------|---------------|
| Toggling frequently | Toggling infrequently |
| Initial render cost is low | Initial render cost is high |
| Element has no expensive initialization | Element needs cleanup on hide |
| Simple visibility toggle | Conditional logic based on permissions/data |

## Reference

- [Vue 2 Guide - Conditional Rendering](https://vue2.docs.vuejs.org/v2/guide/conditional.html)
- [Vue 2 API - v-show](https://vue2.docs.vuejs.org/v2/api/#v-show)
- [Vue 2 API - v-if](https://vue2.docs.vuejs.org/v2/api/#v-if)
