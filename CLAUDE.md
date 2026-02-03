# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Type

This is an **Agent Skills repository** following the [Vercel Labs Skills](https://github.com/vercel-labs/skills) specification. The deliverables are SKILL.md files with YAML frontmatter and categorized reference documentation—NOT runnable Vue applications.

The repository contains a single skill (`vue2.6.14`) with 80 reference files covering Vue 2.6.14 Options API development.

## Repository Structure

```text
skills/
└── vue2.6.14/
    ├── SKILL.md                 # Main skill file (YAML frontmatter + categorized links)
    └── reference/               # 80 reference files in 12 categories
        ├── reactivity/           # Object.defineProperty limitations, $set/$delete
        ├── global-api/           # Vue.config, Vue.nextTick, custom directives
        ├── lifecycle/            # beforeDestroy/destroyed, errorCaptured
        ├── component-options/    # Props, slots, events, watchers, provide/inject
        ├── directives/           # v-model, v-on, v-bind modifiers, v-for/v-if
        ├── instance-api/         # $refs, $emit, $forceUpdate, $data/$props
        ├── components/           # transition, keep-alive, transition-group
        ├── render-functions/     # createElement, JSX, functional components
        ├── types/                # Vue.extend(), PropType, ThisType, shims
        ├── build-tooling/        # Vue CLI, Less, vue-template-compiler
        ├── ecosystem/            # Vue Router 3.x, Vuex 3.x, @vue/composition-api
        └── comparison/           # Vue 2 vs Vue 3 differences
```

## Commands

```bash
# List installed skills
npx skills list

# Add skill locally (for testing)
npx skills add ./skills/vue2.6.14

# Count reference files
find skills/vue2.6.14/reference -name '*.md' | wc -l
```

## Version Constraints

**Critical**: All content must be specific to **Vue 2.6.14** with the following constraints:

1. **Options API only** - Do NOT use Composition API syntax (unless documenting `@vue/composition-api` plugin)
2. **Object.defineProperty reactivity** - Vue 2 does not use Proxy-based reactivity
3. **Lifecycle hooks**: `beforeDestroy`/`destroyed` (NOT `beforeUnmount`/`unmounted`)
4. **Vue 2.6.0+ features** must be documented with version notes:
   - Dynamic directive arguments: `v-bind:[key]`, `@[event]`
   - v-slot shorthand: `#header` (NOT `slot-scope`)
   - `errorCaptured` hook for async component errors
   - `ThisType` support for better TypeScript inference

## Reference File Format

All reference files in `reference/` directories follow this structure:

```markdown
---
title: Descriptive Title
impact: HIGH|MEDIUM|LOW
impactDescription: Brief description
type: capability|best-practice|anti-pattern
tags:
  - vue2.6.14
  - category-name
  - relevant-tags
---

# Title

Impact: LEVEL - Description

## Task Checklist

- [ ] Action item

## Code Example

```vue
<template>
  <!-- Vue 2.6.14 template syntax -->
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'ComponentName',
  props: { },
  data() {
    return { }
  },
  computed: { },
  methods: { },
  mounted() { }
})
</script>

<style lang="less" scoped>
// Less preprocessor
</style>
```

## Common Gotchas

- List of issues

## Reference

- [Vue 2 Documentation](https://vue2.docs.vuejs.org/)
```

## Technology Stack (for code examples)

- **Vue**: 2.6.14 Options API
- **TypeScript**: `Vue.extend()` pattern (NOT `<script setup>`)
- **Styles**: Less preprocessor (`<style lang="less">`)
- **Router**: Vue Router 3.x
- **State**: Vuex 3.x
- **Build**: Vue CLI 4.x/5.x

## SKILL.md Format

The main `SKILL.md` file requires:

```yaml
---
name: vue2.6.14
description: Brief description (10-200 chars)
version: 1.0.0
license: MIT
author: github.com/username
---
```

Content sections use the format: `**Topic** → See [category/](reference/category/file.md)`

## Tag Convention

All reference files MUST use `vue2.6.14` tag (NOT `vue2` or generic tags).

## When Adding Reference Files

1. Place file in appropriate `reference/` subdirectory
2. Follow the reference file format above
3. Use `vue2.6.14` tag in frontmatter
4. Add link to `SKILL.md` in appropriate category
5. Update count in `README.md`

## Key Vue 2 vs Vue 3 Differences

| Concept | Vue 2.6.14 | Vue 3 |
|---------|------------|-------|
| Component definition | `Vue.extend()` | `defineComponent()` |
| Lifecycle | `beforeDestroy`/`destroyed` | `beforeUnmount`/`unmounted` |
| Event emission | `$emit` | `emit()` (setup) |
| Slot syntax | `slot-scope` | `v-slot` |
| Filters | ✅ Supported | ❌ Removed |
| Attributes | `$attrs`, `$listeners` | `$attrs` (merged) |
| Reactivity | `Vue.set()`/`Vue.delete()` | Native Proxy |

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->
