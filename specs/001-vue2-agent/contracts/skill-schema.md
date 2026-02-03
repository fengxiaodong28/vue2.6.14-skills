# Contract: SKILL.md Frontmatter Schema

**Feature**: 001-vue2-agent
**Date**: 2026-02-03
**Contract Type**: Data Schema / Validation

## Overview

This contract defines the schema for SKILL.md frontmatter following the Vercel Labs Agent Skills specification.

---

## Schema Definition

### Required Fields

```yaml
---
name: <required: string, kebab-case, lowercase, a-z, 0-9, hyphens>
description: <required: string, 10-200 characters>
---
```

### Optional Fields

```yaml
---
metadata:
  internal: <optional: boolean, default: false>
  author: <optional: string>
  version: <optional: string, semver format>
  source: <optional: URL>
---
```

---

## Field Specifications

### name

| Property | Value |
|----------|-------|
| Type | `string` |
| Required | Yes |
| Format | kebab-case (lowercase, a-z, 0-9, hyphens) |
| Pattern | `^[a-z0-9]+(-[a-z0-9]+)*$` |
| Max Length | 100 characters |
| Examples | `vue2-core-concepts`, `vue2-component-patterns`, `vue2-cli-build-tooling` |

### description

| Property | Value |
|----------|-------|
| Type | `string` |
| Required | Yes |
| Format | Plain text (no markdown) |
| Min Length | 10 characters |
| Max Length | 200 characters |
| Purpose | Briefly explain what the skill does and when to use it |

### metadata.internal

| Property | Value |
|----------|-------|
| Type | `boolean` |
| Required | No |
| Default | `false` |
| Purpose | Hide from normal discovery (only visible with `INSTALL_INTERNAL_SKILLS=1`) |
| Use Case | Work-in-progress skills or internal tooling |

### metadata.author

| Property | Value |
|----------|-------|
| Type | `string` |
| Required | No |
| Format | Free text |
| Examples | `github.com/username`, `Author Name` |

### metadata.version

| Property | Value |
|----------|-------|
| Type | `string` |
| Required | No |
| Format | Semantic versioning (semver) |
| Pattern | `^\d+\.\d+\.\d+(-[a-z0-9.]+)?$` |
| Examples | `1.0.0`, `2.1.3`, `1.0.0-beta.1` |

### metadata.source

| Property | Value |
|----------|-------|
| Type | `string` (URL) |
| Required | No |
| Format | Valid HTTP/HTTPS URL |
| Purpose | Link to source documentation or reference material |

---

## Validation Rules

### Rule 1: YAML Parsing

The frontmatter MUST be valid YAML. Invalid YAML will cause skill discovery to fail.

```yaml
# Valid
---
name: vue2-core-concepts
description: Vue 2.6.14 core concepts and patterns
---

# Invalid (missing closing ---)
---
name: vue2-core-concepts
description: Vue 2.6.14 core concepts and patterns
```

### Rule 2: Required Fields Presence

Both `name` and `description` MUST be present in every skill file.

### Rule 3: Name Format

Skill names MUST follow kebab-case format:
- All lowercase
- Hyphens as separators
- No spaces or underscores

```yaml
# Valid
name: vue2-core-concepts
name: typescript-integration

# Invalid
name: Vue2_Core_Concepts
name: vue2CoreConcepts
name: Vue 2 Core Concepts
```

### Rule 4: Description Length

Descriptions SHOULD be between 10-200 characters for optimal display in CLI tools.

---

## Examples

### Minimal Valid Skill

```yaml
---
name: vue2-core-concepts
description: Vue 2.6.14 Options API, reactivity system, and lifecycle hooks
---
```

### Complete Skill with Metadata

```yaml
---
name: vue2-component-patterns
description: Vue 2.6.14 component patterns with TypeScript and Less
metadata:
  author: github.com/username
  version: "1.0.0"
  source: https://vue2.docs.vuejs.org
---
```

### Internal Skill (WIP)

```yaml
---
name: vue2-experimental-feature
description: Experimental Vue 2 patterns (work in progress)
metadata:
  internal: true
  version: "0.1.0"
---
```

---

## Error Conditions

| Error | Cause | Resolution |
|-------|--------|------------|
| `No skills found` | Missing `name` or `description` | Add required fields |
| `Invalid YAML` | Malformed frontmatter | Fix YAML syntax |
| `Skill not loading` | Invalid `name` format | Use kebab-case |

---

## Testing

Validate a skill file using:

```bash
# Check if skills are discoverable
npx skills list

# Parse YAML to validate
python -c "import yaml; print(yaml.safe_load(open('SKILL.md')))"
```

---

## Compliance

This contract complies with the Vercel Labs Agent Skills specification as documented at:
https://github.com/vercel-labs/skills
