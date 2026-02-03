# Contract: Skill Content Template

**Feature**: 001-vue2-agent
**Date**: 2026-02-03
**Contract Type**: Content Standard / Template

## Overview

This contract defines the standard content structure for all Vue 2.6.14 Agent Skills. Following this template ensures consistency across skills and optimal agent comprehension.

---

## Standard Content Structure

```markdown
---
name: <skill-name>
description: <brief description>
---

# <Skill Title>

<Brief overview paragraph (2-3 sentences)>

## When to Use

Use this skill when:
- <Scenario 1 with specific condition>
- <Scenario 2 with specific condition>
- <Scenario 3 with specific condition>

## <Section 1: Core Concepts / Patterns / Features>

### <Subsection 1>

<Explanation of concept/pattern/feature>

**Key Points**:
- Point 1
- Point 2
- Point 3

### <Subsection 2>

<Explanation of concept/pattern/feature>

## Code Examples

### Example 1: <Title>

<Brief description of what this example demonstrates>

```vue
<template>
  <div class="component-name">
    <!-- Vue 2.6.14 template -->
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'ComponentName',
  // Vue 2 Options API
})
</script>

<style lang="less" scoped>
/* Less preprocessor styles */
</style>
```

### Example 2: <Title>

<Brief description>

```vue
<!-- Full SFC example -->
</template>
```

## Common Gotchas

### Gotcha 1: <Title>

<Description of the problem>

**Solution**: <How to fix it>

### Gotcha 2: <Title>

<Description of the problem>

**Solution**: <How to fix it>

## Related Skills

- [<skill-name-1>]: <Brief description>
- [<skill-name-2>]: <Brief description>
```

---

## Section Requirements

### Required Sections

Every skill MUST include:

1. **When to Use**: Clear scenarios for skill activation
2. **Core Content**: At least one main section with substantive content
3. **Code Examples**: If applicable to the skill topic

### Optional Sections

These sections MAY be included when relevant:

1. **Common Gotchas**: Troubleshooting for known issues
2. **Related Skills**: Cross-references to other skills
3. **Best Practices**: Recommendations and conventions
4. **Migration Notes**: Vue 2 to Vue 3 guidance (if applicable)

---

## Content Style Guidelines

### Writing Style

1. **Concise**: Agents process brief instructions more efficiently
2. **Action-Oriented**: Use imperative mood ("Do this", not "You should do this")
3. **Specific**: Avoid vague statements; provide concrete examples
4. **Scannable**: Use headings, bullet points, and code blocks for structure

### Code Examples

1. **Complete SFC**: Always show template, script, and style sections
2. **Vue 2 Syntax**: Use Options API, NOT Composition API
3. **TypeScript**: Use `<script lang="ts">` and `Vue.extend()`
4. **Less**: Use `<style lang="less" scoped>`
5. **Comments**: Add comments for non-obvious patterns

```vue
<template>
  <div class="user-card">
    <h2>{{ user.name }}</h2>
    <p>{{ user.email }}</p>
    <button @click="handleEdit">Edit</button>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

interface User {
  id: number
  name: string
  email: string
}

export default Vue.extend({
  name: 'UserCard',
  props: {
    user: {
      type: Object as PropType<User>,
      required: true
    }
  },
  methods: {
    handleEdit(): void {
      this.$emit('edit', this.user.id)
    }
  }
})
</script>

<style lang="less" scoped>
@card-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);

.user-card {
  box-shadow: @card-shadow;
  padding: 16px;

  button {
    background: #42b983;
    color: white;

    &:hover {
      opacity: 0.9;
    }
  }
}
</style>
```

### Cross-References

Use the `[skill-name]` syntax to reference related skills:

```markdown
See [vue2-component-patterns] for prop validation patterns.

For TypeScript setup, see [vue2-cli-build-tooling].
```

---

## Skill-Specific Adaptations

### Skill 1: vue2-core-concepts

**Focus**: Reactivity, lifecycle, directives, template syntax

**Required Sections**:
- When to Use
- Reactivity System
- Component Lifecycle
- Template Syntax & Directives
- Common Gotchas (reactivity caveats)

### Skill 2: vue2-component-patterns

**Focus**: Props, events, slots, component communication

**Required Sections**:
- When to Use
- Props & Validation
- Events & $emit
- Slots (named, scoped)
- Component Communication Patterns
- TypeScript Integration

### Skill 3: vue2-cli-build-tooling

**Focus**: Vue CLI, webpack, vue.config.js

**Required Sections**:
- When to Use
- Project Setup
- vue.config.js Configuration
- Less Preprocessor Setup
- TypeScript Configuration
- Common Build Issues

### Skill 4: vue2-ecosystem-integration

**Focus**: Vue Router, Vuex, axios, UI libraries

**Required Sections**:
- When to Use
- Vue Router 3.x
- Vuex 3.x
- HTTP Client (axios)
- UI Component Libraries
- Integration Patterns

---

## Quality Checklist

Before finalizing a skill, verify:

- [ ] YAML frontmatter is valid (name, description present)
- [ ] "When to Use" section has 3+ specific scenarios
- [ ] Code examples use Vue 2.6.14 Options API
- [ ] Code examples include `<script lang="ts">` and `Vue.extend()`
- [ ] Code examples include `<style lang="less" scoped>`
- [ ] Common gotchas section includes solutions
- [ ] Related skills are referenced using `[skill-name]` syntax
- [ ] Content is concise and scannable
- [ ] No Vue 3 Composition API syntax
- [ ] No `<script setup>` syntax

---

## Example: Complete Skill

```yaml
---
name: vue2-core-concepts
description: Vue 2.6.14 Options API, reactivity system, lifecycle hooks, and directives
metadata:
  version: "1.0.0"
  source: https://vue2.docs.vuejs.org
---

# Vue 2.6.14 Core Concepts

Vue 2.6.14 core concepts including the Options API, Object.defineProperty-based reactivity system, component lifecycle, and template directives. This skill focuses exclusively on Vue 2 syntax and patterns.

## When to Use

Use this skill when:
- Writing Vue 2 components using Options API
- Explaining Vue 2's reactivity system and its limitations
- Understanding component lifecycle hooks
- Using Vue 2 directives (v-if, v-for, v-model, etc.)
- Distinguishing Vue 2 from Vue 3 syntax

## Reactivity System

Vue 2 uses `Object.defineProperty` for reactivity, which has limitations compared to Vue 3's Proxy-based system.

### Reactivity Caveats

1. **Property Addition**: Cannot detect new properties on plain objects
   - Solution: Use `this.$set(obj, 'key', value)`

2. **Array Index Changes**: Cannot detect direct index assignment
   - Solution: Use `this.$set(arr, index, value)` or `arr.splice()`

3. **Array Length**: Cannot detect length assignment
   - Solution: Use `arr.splice(newLength)` instead

```vue
<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  data() {
    return {
      user: {} as Record<string, any>
    }
  },
  methods: {
    addProperty(): void {
      // WRONG: this.user.name = 'John' // Not reactive
      // RIGHT: this.$set(this.user, 'name', 'John')
    }
  }
})
</script>
```

## Component Lifecycle

Vue 2 lifecycle hooks in execution order:

1. `beforeCreate` - Instance initialized, data/observers not ready
2. `created` - data/observers ready, DOM not mounted
3. `beforeMount` - DOM compile complete, not mounted
4. `mounted` - Component mounted to DOM
5. `beforeUpdate` - Data changed, DOM not yet updated
6. `updated` - DOM updated after data change
7. `beforeDestroy` - Before instance destruction
8. `destroyed` - Instance destroyed

```vue
<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  created() {
    console.log('Data available, DOM not ready')
    // Good for: API calls, initial data setup
  },
  mounted() {
    console.log('DOM ready')
    // Good for: DOM manipulation, third-party init
  },
  beforeDestroy() {
    console.log('Cleanup')
    // Good for: event listeners, timers cleanup
  }
})
</script>
```

## Template Syntax & Directives

### Common Directives

- `v-if` / `v-else-if` / `v-else` - Conditional rendering
- `v-show` - Toggle visibility (keeps DOM)
- `v-for` - List rendering
- `v-bind` (or `:`) - Attribute binding
- `v-on` (or `@`) - Event handling
- `v-model` - Two-way data binding

```vue
<template>
  <div>
    <!-- Conditional rendering -->
    <div v-if="isLoading">Loading...</div>
    <div v-else-if="hasError">Error: {{ error }}</div>
    <div v-else>{{ content }}</div>

    <!-- List rendering -->
    <li v-for="item in items" :key="item.id">
      {{ item.name }}
    </li>

    <!-- Two-way binding -->
    <input v-model="formData.name" />
    <textarea v-model="formData.message"></textarea>
  </div>
</template>
```

## Common Gotchas

### Gotcha 1: v-for with v-if

**Problem**: `v-if` has higher priority than `v-for` in Vue 2

```vue
<!-- WRONG: v-if doesn't have access to item -->
<li v-for="item in items" v-if="item.active">
  {{ item.name }}
</li>

<!-- RIGHT: Use template or computed -->
<template v-for="item in items">
  <li v-if="item.active" :key="item.id">
    {{ item.name }}
  </li>
</template>
```

### Gotcha 2: :key on v-for

**Problem**: Missing `:key` causes Vue to reuse DOM elements inefficiently

```vue
<!-- WRONG -->
<li v-for="item in items">{{ item.name }}</li>

<!-- RIGHT: Use unique identifier -->
<li v-for="item in items" :key="item.id">{{ item.name }}</li>
```

## Related Skills

- [vue2-component-patterns]: Props, events, slots, and component communication
- [vue2-cli-build-tooling]: Vue CLI and build configuration
- [vue2-ecosystem-integration]: Vue Router, Vuex, and ecosystem libraries
```

---

## Compliance

This content template complies with:
- Vercel Labs Agent Skills specification
- Vue 2.6.14 documentation standards
- TypeScript best practices for Vue 2
- Less preprocessor conventions
