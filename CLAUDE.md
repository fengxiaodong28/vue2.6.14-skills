# vue2.6.14-skills Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-02-03

## Active Technologies

- Markdown (SKILL.md files) with YAML frontmatter (001-vue2-agent)
- Vue 2.6.14 Options API (skill content focus)
- TypeScript with Vue.extend() (skill content focus)
- Less CSS preprocessor (skill content focus)
- Vue CLI 4.x/5.x (skill content focus)
- Vue Router 3.x (skill content focus)
- Vuex 3.x (skill content focus)

## Project Type

This is an **Agent Skills repository** following the Vercel Labs skills specification. The deliverables are SKILL.md files with YAML frontmatter, NOT runnable Vue applications.

## Project Structure

```text
skills/                              # Agent skills directory
├── vue2-core-concepts/              # Vue 2.6.14 core concepts
│   └── SKILL.md
├── vue2-component-patterns/         # Component patterns with TypeScript
│   └── SKILL.md
├── vue2-cli-build-tooling/          # Vue CLI and build configuration
│   └── SKILL.md
└── vue2-ecosystem-integration/      # Vue Router, Vuex, ecosystem
    └── SKILL.md

examples/                            # Optional code examples
specs/                               # Feature specifications
└── 001-vue2-agent/
    ├── spec.md
    ├── plan.md
    ├── research.md
    ├── data-model.md
    ├── quickstart.md
    └── contracts/
```

## Commands

```bash
# List installed skills
npx skills list

# Add a skill (for testing)
npx skills add ./skills/vue2-core-concepts

# Check for skill updates
npx skills check

# Validate YAML frontmatter
python3 -c "import yaml; yaml.safe_load(open('skills/vue2-core-concepts/SKILL.md'))"
```

## Code Style

### SKILL.md Files

- **Format**: YAML frontmatter + Markdown content
- **Required fields**: `name` (kebab-case), `description` (10-200 chars)
- **Optional fields**: `metadata.internal`, `metadata.author`, `metadata.version`, `metadata.source`

### Skill Content Guidelines

1. **Vue 2.6.14 Only**: Use Options API, NOT Composition API
2. **TypeScript Pattern**: Use `Vue.extend()` for components
3. **Style Preprocessor**: Use Less in examples (`<style lang="less">`)
4. **Complete SFCs**: Always show template, script, and style sections
5. **When to Use**: Include 3+ specific scenarios
6. **Code Examples**: Full, copy-pasteable Vue 2.6.14 code
7. **Common Gotchas**: Document issues with solutions
8. **Cross-References**: Use `[skill-name]` for related skills

### Vue 2.6.14 Code Example Template

```vue
<template>
  <div class="component-name">
    <!-- Vue 2 template syntax -->
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'ComponentName',
  props: {
    // Props with validation
  },
  data() {
    return {
      // Reactive data
    }
  },
  computed: {
    // Computed properties
  },
  methods: {
    // Methods
  },
  mounted() {
    // Lifecycle hook
  }
})
</script>

<style lang="less" scoped>
// Less preprocessor styles
</style>
```

## Recent Changes

- 001-vue2-agent: Added Markdown (SKILL.md files) with YAML frontmatter
- 001-vue2-agent: Added Vue 2.6.14 + TypeScript + Less technology stack
- 001-vue2-agent: Created skill structure templates and contracts

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->
