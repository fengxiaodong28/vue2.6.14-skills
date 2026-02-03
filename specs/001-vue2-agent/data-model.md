# Data Model: Vue 2.6.14 Agent Skills

**Feature**: 001-vue2-agent
**Date**: 2026-02-03
**Phase**: Phase 1 - Design & Contracts

## Overview

This document defines the data model for Vue 2.6.14 Agent Skills. Since this is a documentation/skills project (not a software application), the "data model" describes the structure of skill files and their relationships.

---

## Entity 1: SKILL.md File

**Description**: The primary artifact containing agent instructions with YAML frontmatter

### Attributes

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string (kebab-case) | Yes | Unique identifier for the skill (e.g., `vue2-core-concepts`) |
| `description` | string | Yes | Brief explanation of what the skill does and when to use it |
| `metadata` | object | No | Optional metadata (author, version, source, internal flag) |
| `content` | markdown | Yes | The actual instructions for the AI agent |

### Structure

```text
SKILL.md
├── YAML Frontmatter (---)
│   ├── name: string (required)
│   ├── description: string (required)
│   └── metadata: object (optional)
│       ├── internal: boolean (default: false)
│       ├── author: string
│       ├── version: string
│       └── source: URL
└── Markdown Content
    ├── When to Use section
    ├── Core Concepts / Patterns
    ├── Code Examples (Vue 2.6.14 syntax)
    ├── Common Gotchas / Troubleshooting
    └── Cross-References to related skills
```

### Validation Rules

1. **Name Format**: Must be lowercase kebab-case (a-z, 0-9, hyphens)
2. **Description Length**: Should be 10-200 characters for optimal display
3. **YAML Validity**: Frontmatter must parse as valid YAML
4. **Content Sections**: Each skill should include at minimum:
   - "When to Use" section
   - Core content (concepts, patterns, or instructions)
   - Code examples (if applicable)

---

## Entity 2: Skill Frontmatter

**Description**: YAML metadata block at the top of each SKILL.md file

### Schema

```yaml
---
# Required Fields
name: <kebab-case-skill-name>
description: <brief description>

# Optional Fields
metadata:
  # Hide from normal discovery (only visible with INSTALL_INTERNAL_SKILLS=1)
  internal: <boolean>  # default: false

  # Skill author attribution
  author: <username or name>

  # Semantic version
  version: "<semver>"  # e.g., "1.0.0"

  # Source documentation URL
  source: <URL>
---
```

### Field Values Reference

| Field | Valid Values | Example |
|-------|--------------|---------|
| `name` | kebab-case | `vue2-core-concepts`, `vue2-component-patterns` |
| `description` | plain text | `Vue 2.6.14 Options API, reactivity system, lifecycle hooks, and directives` |
| `metadata.internal` | boolean | `true` (work-in-progress), `false` (default) |
| `metadata.author` | string | `github.com/username` or `Author Name` |
| `metadata.version` | semver | `"1.0.0"`, `"2.1.3"` |
| `metadata.source` | URL | `https://vue2.docs...` |

---

## Entity 3: Skill Content

**Description**: Markdown instructions within each SKILL.md file

### Standard Content Template

Each skill follows this structure:

```markdown
---
name: skill-name
description: Skill description
---

# Skill Title

Brief overview paragraph.

## When to Use

Use this skill when:
- [Scenario 1]
- [Scenario 2]
- [Scenario 3]

## Core Concepts

### Concept 1
[Explanation]

### Concept 2
[Explanation]

## Code Examples

### Example 1: [Title]

```vue
<template>
  <!-- Vue 2.6.14 template syntax -->
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

## Common Gotchas

### Issue 1
[Description + Solution]

### Issue 2
[Description + Solution]

## Related Skills

- [skill-name-1]: [Brief description]
- [skill-name-2]: [Brief description]
```

### Content Style Guidelines

1. **When to Use Section**: Clear, actionable scenarios for skill activation
2. **Code Examples**: Always include full SFC structure (template, script, style)
3. **Vue 2 Syntax**: Use Options API, NOT Composition API
4. **TypeScript**: Use `Vue.extend()` pattern for components
5. **Less**: Use `<style lang="less">` with scoped styles
6. **Cross-References**: Use `[skill-name]` syntax for related skills

---

## Entity 4: Code Example

**Description**: Vue 2.6.14 code snippets within skill content

### Standard Vue 2.6.14 SFC Format

```vue
<template>
  <div class="component-name">
    <!-- Template syntax: v-if, v-for, v-bind, v-on, v-model -->
    <h1>{{ title }}</h1>
    <button @click="handleClick">{{ buttonText }}</button>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'ComponentName',
  props: {
    title: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      buttonText: 'Click me'
    }
  },
  computed: {
    displayTitle(): string {
      return this.title.toUpperCase()
    }
  },
  methods: {
    handleClick(): void {
      // Event handling
    }
  },
  mounted() {
    // Lifecycle hook
  }
})
</script>

<style lang="less" scoped>
@primary-color: #42b983;

.component-name {
  color: @primary-color;

  button {
    background: @primary-color;

    &:hover {
      opacity: 0.8;
    }
  }
}
</style>
```

### Code Example Guidelines

1. **Complete SFC**: Always show template, script, and style sections
2. **TypeScript**: Use `<script lang="ts">` and `Vue.extend()`
3. **Less**: Use `<style lang="less" scoped>`
4. **Vue 2 Only**: NO Composition API, NO `<script setup>`
5. **Clarity**: Include comments for non-obvious patterns

---

## Entity 5: Skill Repository

**Description**: The collection of all Vue 2.6.14 Agent Skills

### Repository Structure

```text
vue2.6.14-skills/
├── skills/                          # Root skills directory (auto-discovered)
│   ├── vue2-core-concepts/          # Skill 1 (P1)
│   │   └── SKILL.md
│   ├── vue2-component-patterns/     # Skill 2 (P2)
│   │   └── SKILL.md
│   ├── vue2-cli-build-tooling/      # Skill 3 (P3)
│   │   └── SKILL.md
│   └── vue2-ecosystem-integration/  # Skill 4 (P4)
│       └── SKILL.md
├── examples/                        # Optional supporting code
│   └── vue2-typescript-components/
│       ├── MyComponent.vue
│       └── tsconfig.json
├── .specify/                        # SpecKit templates
├── specs/                           # Feature specifications
│   └── 001-vue2-agent/
├── .gitignore
├── package.json
└── README.md
```

### Skill Relationships

```
vue2-core-concepts (P1)
    ├── References: Component patterns (for usage examples)
    └── Referenced by: All other skills

vue2-component-patterns (P2)
    ├── Depends on: Core concepts (reactivity, lifecycle)
    └── References: CLI build tooling (for TypeScript setup)

vue2-cli-build-tooling (P3)
    ├── References: Component patterns (for TypeScript examples)
    └── Referenced by: Ecosystem integration (for build config)

vue2-ecosystem-integration (P4)
    ├── Depends on: Core concepts, Component patterns
    └── References: CLI build tooling (for webpack aliases)
```

---

## State Transitions

### Skill Lifecycle

```text
[Draft] → [Review] → [Published]
    ↓         ↓           ↓
 (local)  (internal)   (discoverable)
```

| State | Description | `metadata.internal` |
|-------|-------------|---------------------|
| Draft | Work in progress | `true` |
| Review | Ready for team review | `true` |
| Published | Ready for use | `false` |

---

## Summary

The data model consists of 5 primary entities:

1. **SKILL.md File**: The primary artifact with YAML frontmatter and markdown content
2. **Skill Frontmatter**: YAML metadata (name, description, optional fields)
3. **Skill Content**: Markdown instructions with standard sections
4. **Code Example**: Vue 2.6.14 SFC snippets following Options API + TypeScript + Less
5. **Skill Repository**: The collection of skills with defined relationships

All entities follow the Vercel Labs Agent Skills specification and focus exclusively on Vue 2.6.14 with Options API, TypeScript via Vue.extend(), and Less preprocessor.
