# Vue 2.6.14 Agent Skills

> Agent Skills for Vue 2.6.14 development following the [Vercel Labs Agent Skills](https://github.com/vercel-labs/skills) specification.

## Overview

This repository contains an Agent Skill that enables AI coding agents (Claude Code, Cursor, and 35+ compatible agents) to provide accurate guidance for Vue 2.6.14 development using the Options API.

**Technology Stack**: Vue 2.6.14 Options API + TypeScript + Less

Inspired by [vuejs-ai/skills](https://github.com/vuejs-ai/skills) - the official Vue 3 skills repository, adapted for Vue 2.6.14.

## Installation

### Install from Local Path

```bash
# For development / testing
npx skills add ./vue2.6.14-skills
```

### Install from GitHub (after publishing)

```bash
# Install to Claude Code
npx skills add username/vue2.6.14-skills
```

## Available Skill

| Skill | Description | Reference Files |
|-------|-------------|----------------|
| **vue2.6.14** | Vue 2.6.14 Options API best practices, reactivity system, component patterns, and ecosystem integration | 19+ granular reference files |

## Usage

Once installed, your AI agent will automatically invoke the vue2.6.14 skill when you ask questions like:

```
"Why can't I add new properties to my Vue 2 object?"

"How do I use Vue.extend() with TypeScript?"

"Why doesn't v-for with v-if work in Vue 2?"

"Set up Vue Router 3.x with Vue 2."

"What's the difference between beforeDestroy and beforeUnmount?"
```

## Skill Content

The main SKILL.md provides categorized links to granular reference files covering:

### Reactivity System
- Property addition caveats (`reactivity-property-addition.md`)
- Array index assignment (`reactivity-array-index-assignment.md`)
- Array length changes (`reactivity-array-length-changes.md`)
- Frozen objects (`reactivity-frozen-objects.md`)
- $set and $delete usage

### Component Lifecycle
- Arrow functions in lifecycle hooks (`no-arrow-functions-in-lifecycle-hooks.md`)
- Vue 2 specific hooks (`vue2-lifecycle-hooks.md`)
- Stateful methods (`stateful-methods-lifecycle.md`)

### Component Patterns
- Props with interfaces (`props-complex-types-interface.md`)
- .sync modifier (`sync-modifier-two-way-binding.md`)
- Provide/Inject (`provide-inject-not-reactive.md`)
- Event bus pattern (`event-bus-pattern.md`)
- Slots (`slots-default-named.md`, `slots-scoped-slot-props.md`)
- v-for + v-if priority (`v-for-v-if-priority.md`)
- Missing :key (`v-for-missing-key.md`)

### TypeScript Integration
- Vue.extend() pattern (`ts-vue-extend-pattern.md`)
- PropType for complex types
- Type-safe computed properties
- Shims for .vue files

### Build Tooling
- vue.config.js setup (`cli-vue-config-basic.md`)
- Less configuration (`cli-less-configuration.md`)
- Version matching (`version-matching-template-compiler.md`)

### Ecosystem Integration
- Vue Router 3.x setup (`router3-basic-setup.md`)
- Vuex 3.x modules (`vuex3-store-modules.md`)
- Axios integration
- UI component libraries

### Vue 2 vs Vue 3
- Key syntax differences (`vue2-vs-vue3-syntax.md`)
- Filters deprecated (`vue2-filters-deprecated.md`)
- Lifecycle naming differences

### Skill Examples

#### Example 1: Creating a Vue 2 Component with TypeScript

```vue
<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'MyComponent',
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
  }
})
</script>

<style lang="less" scoped>
.my-component {
  color: #42b983;
}
</style>
```

#### Example 2: Vue 2 Reactivity Caveats

```javascript
// Property addition - use $set
this.$set(this.user, 'name', 'John')

// Array index changes - use $set or splice
this.$set(this.items, 0, 'newValue')
this.items.splice(0, 1, 'newValue')
```

#### Example 3: Vue Router 3.x with Vue 2

```javascript
import VueRouter from 'vue-router'

const router = new VueRouter({
  mode: 'history',
  routes: [
    { path: '/', component: Home },
    { path: '/user/:id', component: User, props: true }
  ]
})
```

#### Example 4: Less Configuration in vue.config.js

```javascript
module.exports = {
  css: {
    loaderOptions: {
      less: {
        additionalData: `
          @import "@/styles/variables.less";
        `
      }
    }
  }
}
```

## Development

### Project Structure

```text
skills/
└── vue2.6.14/                   # Vue 2.6.14 skill
    ├── SKILL.md                 # Main skill file with categorized links
    └── reference/               # 19+ granular reference files
        ├── reactivity-property-addition.md
        ├── reactivity-array-index-assignment.md
        ├── no-arrow-functions-in-lifecycle-hooks.md
        ├── vue2-lifecycle-hooks.md
        ├── v-for-v-if-priority.md
        ├── v-for-missing-key.md
        ├── sync-modifier-two-way-binding.md
        ├── provide-inject-not-reactive.md
        ├── event-bus-pattern.md
        ├── slots-scoped-slot-props.md
        ├── props-complex-types-interface.md
        ├── ts-vue-extend-pattern.md
        ├── cli-vue-config-basic.md
        ├── cli-less-configuration.md
        ├── version-matching-template-compiler.md
        ├── router3-basic-setup.md
        ├── vuex3-store-modules.md
        ├── vue2-vs-vue3-syntax.md
        └── vue2-filters-deprecated.md
```

**Note**: Following the [vuejs-ai/skills](https://github.com/vuejs-ai/skills) repository structure:
- `SKILL.md`: Main skill overview with YAML frontmatter and categorized links to reference files
- `reference/`: Granular reference files with frontmatter tables, task checklists, and code examples

### Validating Skills

```bash
# List installed skills
npx skills list

# Validate YAML frontmatter
python3 -c "import yaml; yaml.safe_load(open('skills/vue2.6.14/SKILL.md'))"

# Check reference files
ls skills/vue2.6.14/reference/
```

## Contributing

Contributions are welcome! Please follow the vuejs-ai/skills format:

### Reference File Format

Each reference file should follow this structure:

1. **Frontmatter table**: title, impact, impactDescription, type, tags
2. **Problem description**: Clear explanation of the issue
3. **Task Checklist**: Actionable checklist items
4. **Incorrect code example**: What NOT to do
5. **Correct code example**: Best practice solution
6. **Reference links**: Official documentation

### Guidelines

1. **Vue 2.6.14 Only**: Use Options API, NOT Composition API
2. **TypeScript Pattern**: Use `Vue.extend()` for components
3. **Style Preprocessor**: Use Less in examples (`<style lang="less">`)
4. **Naming**: Use kebab-case for reference file names (e.g., `reactivity-property-addition.md`)
5. **Impact Level**: Mark issues as CRITICAL, HIGH, MEDIUM, or LOW
6. **Type**: Mark as capability, best-practice, anti-pattern, or reference

## License

[MIT](LICENSE)

## Related Resources

- [vuejs-ai/skills](https://github.com/vuejs-ai/skills) - Vue 3 official skills (inspiration for this format)
- [Vercel Labs Skills](https://github.com/vercel-labs/skills) - Agent skills specification
- [Vue 2.6.14 Documentation](https://vue2.docs.vuejs.org/)
- [Vue Router 3.x](https://v3.router.vuejs.org/)
- [Vuex 3.x](https://vuex.vuejs.org/)
- [antfu/skills](https://github.com/antfu/skills) - Reference skills implementation
