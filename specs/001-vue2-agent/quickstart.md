# Quickstart: Vue 2.6.14 Agent Skills

**Feature**: 001-vue2-agent
**Date**: 2026-02-03

## Overview

This guide helps you install, test, and contribute to the Vue 2.6.14 Agent Skills collection.

---

## Installation

### Install from GitHub Repository

```bash
# Install to Claude Code (project-level)
npx skills add <your-username>/vue2.6.14-skills

# Install to Claude Code (global)
npx skills add <your-username>/vue2.6.14-skills -g

# Install to specific agents
npx skills add <your-username>/vue2.6.14-skills -a claude-code -a cursor

# Install all skills from the repo
npx skills add <your-username>/vue2.6.14-skills --all
```

### Install from Local Path

```bash
# For development / testing
npx skills add ./path/to/vue2.6.14-skills

# Local installation to specific agent
npx skills add ./path/to/vue2.6.14-skills -a claude-code
```

### Verify Installation

```bash
# List installed skills
npx skills list

# List skills for specific agent
npx skills list -a claude-code

# Check for updates
npx skills check
```

---

## Skill Discovery

The skills CLI automatically searches these directories:

```
skills/                  # Root skills directory
skills/.curated/         # Curated/featured skills
skills/.experimental/    # Experimental skills
.claude/skills/          # Claude Code project skills
```

### Available Skills

| Skill | Description | When to Use |
|-------|-------------|-------------|
| `vue2-core-concepts` | Vue 2.6.14 Options API, reactivity, lifecycle | Writing Vue 2 components |
| `vue2-component-patterns` | Props, events, slots with TypeScript | Component architecture |
| `vue2-cli-build-tooling` | Vue CLI, webpack, Less configuration | Project setup |
| `vue2-ecosystem-integration` | Vue Router, Vuex, axios | Full-stack features |

---

## Testing Skills

### Test Skill Discovery

```bash
# Verify skills are discoverable
npx skills list | grep vue2

# Expected output:
# vue2-core-concepts
# vue2-component-patterns
# vue2-cli-build-tooling
# vue2-ecosystem-integration
```

### Test YAML Validation

```bash
# Using Python
python3 << 'EOF'
import yaml
import glob

for skill_file in glob.glob('skills/*/SKILL.md'):
    with open(skill_file) as f:
        content = f.read()
        # Extract frontmatter between --- markers
        frontmatter = content.split('---')[1]
        data = yaml.safe_load(frontmatter)
        print(f"✓ {skill_file}: name={data.get('name')}")
        assert 'name' in data, f"Missing 'name' in {skill_file}"
        assert 'description' in data, f"Missing 'description' in {skill_file}"
EOF
```

### Test with AI Agent

In your AI agent (Claude Code, Cursor, etc.), ask:

```
"How do I create a Vue 2.6.14 component with TypeScript props?"

"Explain Vue 2's reactivity system and its limitations."

"How do I configure Less in Vue CLI?"
```

The agent should invoke the appropriate skill and provide accurate Vue 2.6.14 guidance.

---

## Creating New Skills

### 1. Create Skill Directory

```bash
mkdir -p skills/my-new-skill
cd skills/my-new-skill
```

### 2. Create SKILL.md

```bash
# Initialize with template
npx skills init
```

Or manually create `SKILL.md`:

```yaml
---
name: my-new-skill
description: Brief description of what this skill does
---

# My New Skill

## When to Use

Use this skill when:
- Scenario 1
- Scenario 2

## Content

Your skill content here...
```

### 3. Validate Skill

```bash
# Test discovery
npx skills list | grep my-new-skill

# Validate YAML
python3 -c "import yaml; print(yaml.safe_load(open('SKILL.md')))"
```

### 4. Test with Agent

Ask your AI agent a question that should trigger this skill.

---

## Contributing Guidelines

### Code Style

1. **Vue 2.6.14 Only**: Use Options API, NOT Composition API
2. **TypeScript**: Use `Vue.extend()` pattern
3. **Less**: Use `<style lang="less">` in examples
4. **Complete SFCs**: Always show template, script, and style

### Skill Content

1. **When to Use**: Include 3+ specific scenarios
2. **Code Examples**: Full, copy-pasteable Vue 2 SFCs
3. **Gotchas**: Document common issues and solutions
4. **Cross-References**: Link related skills with `[skill-name]`

### Commit Messages

Follow conventional commits:

```bash
feat: add vue2-typescript-integration skill
docs: update vue2-core-concepts with reactivity examples
fix: correct vue2-component-patterns prop validation syntax
```

---

## Troubleshooting

### "No skills found"

**Cause**: Missing `name` or `description` in frontmatter

**Solution**: Verify YAML frontmatter has required fields

```bash
# Check frontmatter
head -5 skills/*/SKILL.md
```

### "Skill not loading in agent"

**Cause**: Skill installed to wrong directory

**Solution**: Verify installation path

```bash
# Claude Code project skills
ls .claude/skills/

# Claude Code global skills
ls ~/.claude/skills/
```

### "Invalid YAML"

**Cause**: Malformed frontmatter

**Solution**: Validate YAML syntax

```bash
# Python validation
python3 -c "import yaml; yaml.safe_load(open('SKILL.md'))"
```

### "Agent provides Vue 3 syntax"

**Cause**: Agent conflating Vue 2 and Vue 3

**Solution**: Ensure skill content explicitly states "Vue 2.6.14" and uses Options API exclusively

---

## Resources

- **Vercel Labs Skills**: https://github.com/vercel-labs/skills
- **Vue 2.6.14 Docs**: https://vue2.docs.vuejs.org/
- **Vue Router 3.x**: https://v3.router.vuejs.org/
- **Vuex 3.x**: https://vuex.vuejs.org/
- **Reference Skills**:
  - antfu/skills: https://github.com/antfu/skills
  - vuejs-ai/skills: https://github.com/vuejs-ai/skills

---

## FAQ

**Q: Can I use these skills with Vue 3?**

A: No. These skills are specifically for Vue 2.6.14. For Vue 3, see antfu/skills.

**Q: Do I need to use TypeScript?**

A: The skills provide TypeScript examples, but JavaScript is also supported. The patterns apply to both.

**Q: Can I use SCSS instead of Less?**

A: These skills focus on Less as specified. SCSS patterns are similar but require different configuration.

**Q: Will these skills work with Nuxt.js?**

A: Nuxt.js uses Vue 2 syntax but has additional conventions. These skills cover core Vue 2, which applies to Nuxt, but Nuxt-specific patterns are not covered.

**Q: How do I migrate from Vue 2 to Vue 3?**

A: See the official Vue 3 migration guide. These skills do not cover migration.
