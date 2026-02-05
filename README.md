# Vue 2.6.14 Agent Skills

> Agent Skills for Vue 2.6.14 development following the [Vercel Labs Agent Skills](https://github.com/vercel-labs/skills) specification.

## Overview

This repository contains an Agent Skill that enables AI coding agents (Claude Code, Cursor, and 35+ compatible agents) to provide accurate guidance for Vue 2.6.14 development using the Options API.

**Technology Stack**: Vue 2.6.14 Options API + TypeScript + Less

Inspired by [vuejs-ai/skills](https://github.com/vuejs-ai/skills) - the official Vue 3 skills repository, adapted for Vue 2.6.14.

## Features

### ✅ Complete Vue 2.6.14 Coverage

- **80 Reference Files** organized in 12 categories
- **Vue 2.6.0+ Specific Features**: Dynamic arguments, v-slot syntax, async error handling, TypeScript improvements
- **100% Version Accurate**: All content validated against Vue 2.6.14 API
- **Vercel Labs Skills Compliant**: Follows official skills specification
- **Production Ready**: Used in real-world Vue 2.6.14 projects

## Installation

### Install from Local Path

```bash
# From project root
cd /path/to/vue2.6.14-skills

# Install the skill
npx skills add .
```

### Install from GitHub (after publishing)

```bash
# Install to Claude Code
npx skills add https://github.com/fengxiaodong28/vue2.6.14-skills
```

### Verify Installation

```bash
# List installed skills
npx skills list

# Should show: vue2.6.14
```

## Available Skill

| Skill | Description | Reference Files | Categories |
|-------|-------------|----------------|------------|
| **vue2.6.14** | Vue 2.6.14 Options API best practices, reactivity system, component patterns, and ecosystem integration | 80 reference files | 12 categories |

## Usage

Once installed, your AI agent will automatically invoke the vue2.6.14 skill when you ask questions like:

```
"Why can't I add new properties to my Vue 2 object?"

"How do I use Vue.extend() with TypeScript?"

"Why doesn't v-for with v-if work in Vue 2?"

"Set up Vue Router 3.x with Vue 2."

"What's the difference between beforeDestroy and beforeUnmount?"

"How do I use dynamic directive arguments in Vue 2.6?"

"Handle async component errors in Vue 2.6+"
```

## Skill Categories

| Category | Files | Coverage |
|----------|-------|----------|
| **Reactivity System** | 7 | Object.defineProperty limitations, $set/$delete, Vue.observable |
| **Global API** | 6 | Vue.config, Vue.nextTick, custom directives, plugins |
| **Lifecycle** | 4 | beforeDestroy/destroyed, errorCaptured, async errors |
| **Component Options** | 15 | Props, slots, events, watchers, functional components |
| **Directives** | 11 | v-model/v-on/v-bind modifiers, v-if/v-for, v-html security |
| **Instance API** | 5 | $refs, $emit, $forceUpdate, $data/$props/$el |
| **Built-in Components** | 3 | transition, keep-alive, transition-group |
| **Render Functions** | 1 | createElement, JSX, functional components |
| **TypeScript** | 6 | Vue.extend(), PropType, ThisType, type improvements |
| **Build Tooling** | 5 | Vue CLI, Less, environment variables |
| **Ecosystem** | 14 | Vue Router 3.x, Vuex 3.x, Composition API |
| **Comparison** | 3 | Vue 2 vs Vue 3 differences |

## Vue 2.6.14 Specific Features

This skill covers Vue 2.6.0+ features that differentiate it from earlier Vue 2 versions:

### ✅ Dynamic Directive Arguments (Vue 2.6.0+)

```vue
<template>
  <!-- Dynamic attribute name -->
  <button v-bind:[key]="value">Dynamic</button>

  <!-- Dynamic event name -->
  <button @[event]="handler">Click</button>
</template>
```

### ✅ v-slot New Syntax (Vue 2.6.0+)

```vue
<template>
  <my-component>
    <!-- v-slot shorthand (Vue 2.6.0+) -->
    <template #header>
      <h1>Header</h1>
    </template>
  </my-component>
</template>
```

### ✅ Async Component Error Handling (Vue 2.6.0+)

```javascript
export default {
  errorCaptured(err, vm, info) {
    // Catch errors from descendant components
    console.error('Error:', err)
    return false // Stop propagation
  }
}
```

### ✅ TypeScript Improvements (Vue 2.6.0+)

- `ThisType` support for better component options typing
- Enhanced computed property type inference
- Better component type definitions

## Project Structure

```text
vue2.6.14-skills/
├── reference/                       # 80 reference files in 12 categories
│   ├── reactivity/                   # Reactivity system (7 files)
│   ├── global-api/                   # Global API (6 files)
│   ├── lifecycle/                    # Lifecycle (4 files)
│   ├── component-options/            # Component options (15 files)
│   ├── directives/                   # Directives (11 files)
│   ├── instance-api/                 # Instance API (5 files)
│   ├── components/                   # Built-in components (3 files)
│   ├── render-functions/             # Render functions (1 file)
│   ├── types/                        # TypeScript (6 files)
│   ├── build-tooling/                # Build tooling (5 files)
│   ├── ecosystem/                    # Ecosystem (14 files)
│   └── comparison/                   # Vue 2 vs 3 comparison (3 files)
├── SKILL.md                         # Main skill file with categorized links
├── README.md                        # This file
├── CLAUDE.md                        # Developer guidelines
└── LICENSE                          # MIT license
```

## Quick Start

### 1. Install the Skill

```bash
npx skills add ./skills/vue2.6.14
```

### 2. Start Coding

Open your Vue 2.6.14 project in Claude Code and start asking questions!

### 3. Example Interactions

**You:** "How do I fix array reactivity issues in Vue 2?"

**Claude:** [Automatically references `reactivity-array-index-assignment.md` and provides solutions using `Vue.set()` or `splice()`]

**You:** "Create a component with TypeScript"

**Claude:** [References `ts-vue-extend-pattern.md` and generates proper Vue.extend() code with type safety]

**You:** "Why is my v-if inside v-for not working?"

**Claude:** [Explains v-for has higher priority than v-if, references `v-for-v-if-priority.md`]

## Skill Examples

### Example 1: Vue 2.6.14 Component with TypeScript

```vue
<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'TypedComponent',
  props: {
    title: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      localData: 'value'
    }
  },
  computed: {
    // Vue 2.6: Better type inference
    doubled(): string {
      return this.localData.repeat(2)
    }
  }
})
</script>

<style lang="less" scoped>
.typed-component {
  color: #42b983;
}
</style>
```

### Example 2: Reactivity Workarounds

```javascript
// Add new reactive property
this.$set(this.user, 'name', 'John')

// Handle array index assignment
this.$set(this.items, 0, 'newValue')
this.items.splice(0, 1, 'newValue')

// Create reactive state
const state = Vue.observable({
  count: 0
})
```

### Example 3: Vue Router 3.x Integration

```javascript
import VueRouter from 'vue-router'

const router = new VueRouter({
  mode: 'history',
  routes: [
    { path: '/', component: Home },
    { path: '/user/:id', component: User, props: true }
  ]
})

export default new Vue({
  router
})
```

## Development

### Validating Skills

```bash
# List all skills
npx skills list

# Validate YAML frontmatter (Python)
python3 -c "import yaml; yaml.safe_load(open('skills/vue2.6.14/SKILL.md'))"
```

### Adding New Reference Files

1. Create file in appropriate category under `reference/`
2. Follow the reference file format:
   - YAML frontmatter (title, impact, impactDescription, type, tags)
   - Clear description with Impact statement
   - Task Checklist
   - Code examples (Vue 2.6.14, Options API, TypeScript, Less)
   - Common gotchas
   - Reference links

3. Update `SKILL.md` to add link in appropriate category

### File Format Template

```markdown
---
title: Descriptive Title
impact: HIGH|MEDIUM|LOW
impactDescription: Brief description of the issue
type: capability|best-practice|anti-pattern
tags:
  - vue2.6.14
  - category
  - relevant-tags
---

# Title

Impact: LEVEL - Description

## Task Checklist

- [ ] Action item 1
- [ ] Action item 2

## Code Example

```vue
<template>
  <!-- Vue 2.6.14 template -->
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  // Options API
})
</script>

<style lang="less" scoped>
// Less styles
</style>
```

## Reference

- [Vue 2 Documentation](https://vue2.docs.vuejs.org/)
```

## Contributing

Contributions are welcome! Please ensure:

1. **Vue 2.6.14 Only**: Content must be specific to Vue 2.6.14
2. **Options API**: Use Options API, not Composition API (unless documenting @vue/composition-api)
3. **TypeScript**: Use `Vue.extend()` pattern
4. **Less Styles**: Use `<style lang="less">` in examples
5. **Version Notes**: Add notes for Vue 2.6.0+ features

### Guidelines

- Use kebab-case for file names (e.g., `reactivity-property-addition.md`)
- Mark critical issues with `impact: CRITICAL`
- Include complete, copy-pasteable code examples
- Add version notes for Vue 2.6.0+ features
- Update tags to use `vue2.6.14` (not just `vue2`)

## Version Compatibility

| Skill Version | Vue Version | Notes |
|---------------|-------------|-------|
| 1.0.0 | 2.6.14 | Initial release with 80 reference files |

## Comparison with Other Vue Versions

| Feature | Vue 2.6.14 | Vue 2.7.x | Vue 3.x |
|---------|------------|-----------|---------|
| Options API | ✅ | ✅ | ⚠️ Deprecated |
| Object.defineProperty Reactivity | ✅ | ✅ | ❌ (Proxy) |
| beforeDestroy/destroyed | ✅ | ✅ | ❌ (beforeUnmount/unmounted) |
| $listeners, $attrs | ✅ | ✅ | ⚠️ Different behavior |
| Filters | ✅ | ✅ | ❌ Removed |
| Dynamic Arguments | ✅ (2.6.0+) | ✅ | ✅ |
| v-slot Syntax | ✅ (2.6.0+) | ✅ | ✅ |
| Composition API | ⚠️ Plugin only | ⚠️ Plugin only | ✅ Built-in |

## License

[MIT](LICENSE)

## Related Resources

- [Vue 2.6.14 Documentation](https://vue2.docs.vuejs.org/)
- [Vue 2.6.14 GitHub Release](https://github.com/vuejs/vue/tree/v2.6.14)
- [Vercel Labs Skills](https://github.com/vercel-labs/skills) - Official specification
- [vuejs-ai/skills](https://github.com/vuejs-ai/skills) - Vue 3 skills (inspiration)
- [Vue Router 3.x](https://v3.router.vuejs.org/)
- [Vuex 3.x](https://vuex.vuejs.org/)

## Changelog

### v1.0.0 (Current)

- ✅ 80 reference files across 12 categories
- ✅ Complete Vue 2.6.14 API coverage
- ✅ All tags updated to `vue2.6.14`
- ✅ Version notes added to key documents
- ✅ Vue 2.6.0+ specific features documented:
  - Dynamic directive arguments
  - v-slot syntax
  - Async component error handling
  - TypeScript improvements
- ✅ Categorized reference structure
- ✅ 100% Vercel Labs Skills compliant

## Support

For issues, questions, or contributions:
- Review [Vue 2.6.14 documentation](https://vue2.docs.vuejs.org/)
- Open an issue on GitHub repository

---

Made with ❤️ for the Vue 2.6.14 community
