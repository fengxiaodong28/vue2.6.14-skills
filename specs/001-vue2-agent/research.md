# Research: Vue 2.6.14 Agent Skills

**Feature**: 001-vue2-agent
**Date**: 2026-02-03
**Phase**: Phase 0 - Research & Analysis

## Overview

This document consolidates research findings for creating Vue 2.6.14 Agent Skills following the Vercel Labs Agent Skills specification. Research covers Vue 2 vs Vue 3 differences, TypeScript integration patterns, Less preprocessor usage, and reference skill analysis.

---

## Research Topic 1: Vue 2.6.14 vs Vue 3 Syntax Differences

### Decision: Focus Exclusively on Vue 2.6.14 Options API

**Rationale**:
- Vue 2.6.14 is the final Vue 2 release (NEVER version - "Nb" - released December 2020)
- Options API is the primary paradigm for Vue 2, while Vue 3 defaults to Composition API
- Agents must clearly distinguish between Vue 2 Options API and Vue 3 Composition API to avoid providing incorrect guidance

### Key Differences Agents Must Know

| Aspect | Vue 2.6.14 (Options API) | Vue 3 (Composition API) |
|--------|--------------------------|-------------------------|
| Component Definition | `Vue.component()`, `Vue.extend()` | `defineComponent()`, `<script setup>` |
| Reactive Data | `data()` function returning object | `ref()`, `reactive()` in `setup()` |
| Methods | `methods: { ... }` object | Functions in `setup()` |
| Lifecycle Hooks | `mounted()`, `created()`, etc. | `onMounted()`, `onCreated()`, etc. |
| Template Refs | `ref="name"` + `this.$refs.name` | `ref="name"` + `const name = ref(null)` |
| Props Definition | `props: { ... }` object | `defineProps<{ ... }>()` |
| Events | `this.$emit()` | `defineEmits<{ ... }>()` |
| Reactivity System | `Object.defineProperty` | ES6 Proxy |
| Async Components | `Vue.component('async', () => import(...))` | `defineAsyncComponent()` |
| Teleport | N/A (use portal libraries) | Built-in `<Teleport>` |
| Fragments | N/A (must have single root) | Built-in support |

### Critical Vue 2.6.14 Reactivity Caveats

Agents must warn users about these Vue 2 limitations:

1. **Property Addition**: Cannot detect property addition/deletion on plain objects
   - Solution: Use `Vue.set()` / `this.$set()` or `Vue.delete()` / `this.$delete()`

2. **Array Index Changes**: Cannot detect direct index assignment
   - Solution: Use `Vue.set()` / `this.$set()` or `splice()`

3. **Array Length Modification**: Cannot detect length assignment
   - Solution: Use `splice()` instead of `arr.length = newLength`

4. **Object.freeze()**: Frozen objects cannot be made reactive
   - Note: Document this limitation for optimization scenarios

---

## Research Topic 2: TypeScript Integration for Vue 2.6.14

### Decision: Use Vue.extend() for Class-Style Components

**Rationale**:
- `Vue.extend()` provides the best TypeScript inference for Vue 2 Options API
- `vue-class-component` + `vue-property-decorator` is popular but deprecated
- Alternative: `Vue.component()` with manual type annotations

### TypeScript Patterns for Vue 2.6.14

#### Pattern 1: Vue.extend() (Recommended)

```typescript
import Vue from 'vue'

// Component with TypeScript
export default Vue.extend({
  name: 'MyComponent',
  props: {
    title: {
      type: String,
      required: true
    },
    count: {
      type: Number,
      default: 0
    }
  },
  data() {
    return {
      localMessage: 'Hello'
    }
  },
  computed: {
    displayTitle(): string {
      return `${this.title}: ${this.count}`
    }
  },
  methods: {
    handleClick(): void {
      this.count++
    }
  }
})
```

#### Pattern 2: Vue.component() with Type Annotations

```typescript
import Vue from 'vue'

export default Vue.component({
  name: 'MyComponent',
  props: {
    // Props need explicit typing
    title: String as PropType<string>
  },
  // ... rest of component
})
```

#### Pattern 3: Class-Style Components (Legacy)

```typescript
import { Component, Vue, Prop } from 'vue-property-decorator'
import { Options } from 'vue-class-component'

@Options({
  components: { ChildComponent }
})
@Component
export default class MyComponent extends Vue {
  @Prop({ type: String, required: true })
  readonly title!: string

  @Prop({ type: Number, default: 0 })
  readonly count!: number

  localMessage: string = 'Hello'

  get displayTitle(): string {
    return `${this.title}: ${this.count}`
  }

  handleClick(): void {
    this.count++
  }
}
```

**Decision**: Use Pattern 1 (Vue.extend) as primary, document Pattern 2 as alternative, avoid Pattern 3 (deprecated)

---

## Research Topic 3: Less Preprocessor in Vue 2.6.14

### Decision: Use Less with vue-loader Configuration

**Rationale**:
- Less is compatible with Vue 2's vue-loader
- Provides variables, mixins, nesting for styles
- User specified Less as the style preprocessor

### Less Configuration in Vue CLI

#### Installation

```bash
npm install less less-loader --save-dev
# or
yarn add less less-loader --dev
```

#### Component Style Usage

```vue
<style lang="less" scoped>
@primary-color: #42b983;
@border-radius: 4px;

.button {
  background-color: @primary-color;
  border-radius: @border-radius;

  &--large {
    padding: 12px 24px;
  }
}
</style>
```

#### Global Less Variables (vue.config.js)

```javascript
module.exports = {
  css: {
    loaderOptions: {
      less: {
        additionalData: `
          @import "@/styles/variables.less";
          @import "@/styles/mixins.less";
        `
      }
    }
  }
}
```

#### Common Less Patterns for Vue 2

1. **Scoped Styles with Deep Selector**:
```less
// Use /deep/ or >>> (Vue 2) or ::v-deep (Vue 2.7+)
.parent {
  /deep/ .child {
    color: @primary-color;
  }
}
```

2. **Mixins for Reusable Styles**:
```less
// styles/mixins.less
.flex-center() {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

---

## Research Topic 4: Vercel Labs Skills Specification

### Decision: Full Compliance with Vercel Labs Specification

**Rationale**: Ensures compatibility with 35+ AI coding agents

### Required Frontmatter Fields

```yaml
---
name: skill-name
description: Brief description of what this skill does
---
```

### Optional Frontmatter Fields

```yaml
---
name: skill-name
description: Brief description
metadata:
  internal: true        # Hide from normal discovery
  author: username       # Skill author
  version: "1.0.0"       # Skill version
  source: URL            # Source URL
---
```

### Skill Discovery Locations

The skills CLI searches for skills in these directories:

```
skills/
skills/.curated/
skills/.experimental/
skills/.system/
.claude/skills/
.cursors/skills/
```

### Skill Content Best Practices

From analyzing antfu/skills and vuejs-ai/skills:

1. **"When to Use" Section**: Clearly state when agents should invoke this skill
2. **Cross-References**: Link to related skills using `[skill-name]` syntax
3. **Code Examples**: Provide concrete, copy-pasteable examples
4. **Troubleshooting**: Include common gotchas and solutions
5. **Concise Instructions**: Agents process instructions more efficiently when brief

---

## Research Topic 5: Reference Skills Analysis

### antfu/skills: Vue Skill Structure

**Key Patterns to Emulate**:

1. **Clear Scope Statement**: "Use when writing Vue SFCs, defineProps/defineEmits..."
2. **Technology-Specific Guidance**: Composition API, script setup macros
3. **Reference Attribution**: Links to official documentation sources

**Adaptations Needed for Vue 2**:
- Change from Composition API to Options API
- Update version references from Vue 3 to Vue 2.6.14
- Replace script setup patterns with Options API patterns
- Update TypeScript patterns (defineComponent → Vue.extend)

### vuejs-ai/skills: Options API Best Practices

**Key Patterns to Emulate**:

1. **Categorized Gotchas**: TypeScript, Methods, Lifecycle sections
2. **Cross-Reference Syntax**: Use `[ts-options-api-use-definecomponent]` for related skills
3. **Specific Problem Statements**: "Need to enable TypeScript type inference..."

**Adaptations for Our Skills**:
- Maintain categorized structure (TypeScript, Patterns, Ecosystem)
- Use cross-references between our 4 skills
- Include troubleshooting sections for common issues

---

## Summary: Technology Stack Decisions

| Technology | Version/Choice | Rationale |
|------------|----------------|-----------|
| Vue | 2.6.14 | Final Vue 2 release, long-term support consideration |
| Component Style | Options API | Native Vue 2 paradigm, not Composition API |
| TypeScript | Via Vue.extend() | Best type inference for Vue 2 Options API |
| Style Preprocessor | Less | User-specified, compatible with vue-loader |
| Build Tool | Vue CLI 4.x/5.x | Standard for Vue 2 projects |
| Template Compiler | vue-template-compiler@2.6.14 | User-specified, matches Vue version |

---

## Next Steps

1. **Phase 1A**: Create `data-model.md` documenting skill structure
2. **Phase 1B**: Create `contracts/skill-schema.md` and `contracts/skill-content-template.md`
3. **Phase 1C**: Create `quickstart.md` for skill installation and testing
4. **Agent Context Update**: Run update script to inform AI about Vue 2.6.14 + Less + TypeScript stack
5. **Proceed to Tasks**: Use `/speckit.tasks` to generate implementation tasks
