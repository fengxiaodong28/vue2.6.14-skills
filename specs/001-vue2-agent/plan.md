# Implementation Plan: Vue 2.6.14 Agent Skills

**Branch**: `001-vue2-agent` | **Date**: 2026-02-03 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-vue2-agent/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

Create a collection of Agent Skills for Vue 2.6.14 development following the Vercel Labs Agent Skills specification. The skills will enable AI coding agents (Claude Code, Cursor, etc.) to provide accurate Vue 2.6.14 guidance with the specific technology stack: Vue 2.6.14 + vue-template-compiler@2.6.14 + Less + TypeScript.

The deliverable is Agent Skills (SKILL.md files with YAML frontmatter), NOT a runnable Vue application. Reference implementations from antfu/skills and vuejs-ai/skills repositories inform the structure and best practices.

## Technical Context

**Language/Version**: Markdown (SKILL.md files) with YAML frontmatter
**Primary Dependencies**:
- Vercel Labs skills CLI (`npx skills`)
- Vue 2.6.14 documentation as knowledge source
- Reference skills: antfu/skills (vue), vuejs-ai/skills (vue-options-api-best-practices)

**Target Platform**: Agent Skills ecosystem (Claude Code, Cursor, and 35+ compatible agents)
**Project Type**: Documentation/skills repository (not a runnable application)
**Technology Stack Focus**: Vue 2.6.14 Options API + TypeScript + Less
**Constraints**:
- Skills MUST follow Vercel Labs specification format
- Each skill MUST have valid YAML frontmatter (name, description)
- Code examples MUST be Vue 2.6.14 Options API syntax (NOT Composition API)
- TypeScript integration patterns for Vue 2 Options API
- Less preprocessor for styles

**Scope/Scope**: 4 main skills covering:
1. Vue 2.6.14 Core Concepts (P1)
2. Vue 2 Component Patterns with TypeScript (P2)
3. Vue 2 CLI and Build Tooling (P3)
4. Vue 2 Ecosystem Integration (P4)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Verify compliance with the following principles from `.specify/memory/constitution.md`:

### I. Code Quality
- [x] Functions are single-purpose with cyclomatic complexity <= 10 (N/A - this is documentation, not code)
- [x] Naming conventions are descriptive and clear (skill names follow kebab-case convention)
- [x] Code organization reflects architectural boundaries (skills organized by topic)
- [x] Public APIs and complex logic include documentation (all skills include explanatory content)
- [x] Linting and formatting configured (will use markdownlint for SKILL.md files)

### II. Testing Standards
- [x] Test coverage plan meets 80% minimum (N/A - documentation skills, validation through usage)
- [x] Unit, integration, and E2E tests planned appropriately (validation: `npx skills` discoverability)
- [x] Test-first approach defined for critical business logic (N/A - not applicable for documentation)
- [x] Test independence and mocking strategy documented (validation: YAML frontmatter parsing)

### III. User Experience Consistency
- [x] Design system or component library identified (N/A - markdown documentation)
- [x] Interaction patterns follow established conventions (Vercel Labs skills specification)
- [x] Responsive design approach defined (N/A - markdown renders responsively)
- [x] Accessibility (WCAG 2.1 AA) requirements addressed (markdown follows accessibility best practices)
- [x] Loading states and error messaging planned (N/A - static documentation)

### IV. Performance Requirements
- [x] Response time targets defined (N/A - static markdown files)
- [x] Resource budgets established (SKILL.md files are text-based, minimal size)
- [x] Runtime performance considerations addressed (N/A - no runtime code)
- [x] Core Web Vitals monitoring planned (N/A - static documentation)

### Development Standards
- [x] Security requirements addressed (no sensitive data in skill files)
- [x] Version control strategy defined (conventional commits for skill updates)

**Status**: All applicable checks PASSED. This is a documentation/skills project, not a software application, so many constitution principles are N/A.

## Project Structure

### Documentation (this feature)

```text
specs/001-vue2-agent/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
skills/
├── vue2-core-concepts/           # P1 - Vue 2.6.14 Core Concepts
│   └── SKILL.md
├── vue2-component-patterns/      # P2 - Vue 2 Component Patterns with TypeScript
│   └── SKILL.md
├── vue2-cli-build-tooling/       # P3 - Vue 2 CLI and Build Configuration
│   └── SKILL.md
└── vue2-ecosystem-integration/   # P4 - Vue Router, Vuex, and ecosystem libraries
    └── SKILL.md

# Optional: examples directory for code snippets (not part of skills)
examples/                          # Optional supporting code examples
└── vue2-typescript-components/    # TypeScript component examples
```

**Structure Decision**: Skills repository structure following Vercel Labs specification. Each skill is a self-contained directory with a SKILL.md file. The repository root contains the skills/ directory for automatic discovery by `npx skills`.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

N/A - No constitution violations. This is a straightforward documentation/skills repository following established patterns from reference implementations.

## Phase 0: Research

### Research Questions

1. **Vue 2.6.14 vs Vue 3 Differences**: What are the key syntax differences between Vue 2 Options API and Vue 3 Composition API that agents must avoid conflating?

2. **TypeScript Integration for Vue 2**: What are the recommended patterns for TypeScript + Vue 2.6.14 Options API? (Vue.extend vs Vue.component + defineComponent)

3. **Less Preprocessor in Vue 2**: What are the specific Less syntax patterns and Vue 2 loader configurations for `.less` files?

4. **Vercel Labs Skills Specification**: What are the full requirements for SKILL.md frontmatter and discoverability?

5. **Reference Skills Analysis**: What patterns from antfu/skills and vuejs-ai/skills should be emulated vs adapted for Vue 2?

### Research Tasks

Launch research agents to investigate:

```bash
# Research Vue 2 vs Vue 3 syntax differences
Task: "Research Vue 2.6.14 Options API vs Vue 3 Composition API syntax differences"

# Research TypeScript patterns for Vue 2
Task: "Research TypeScript integration patterns for Vue 2.6.14 Options API"

# Research Vercel Labs skills specification
Task: "Analyze Vercel Labs Agent Skills specification requirements"

# Analyze reference skill implementations
Task: "Analyze antfu/skills and vuejs-ai/skills structure and best practices"
```

**Output**: `research.md` with findings, decisions, and rationale

## Phase 1: Design & Contracts

### Data Model

From the feature specification, the key entities are:

1. **SKILL.md File**: The primary artifact with YAML frontmatter
2. **Skill Frontmatter**: YAML metadata (name, description, optional metadata fields)
3. **Skill Content**: Markdown instructions for agents
4. **Code Examples**: Vue 2.6.14 code snippets within skill content

**Output**: `data-model.md` documenting skill structure

### Contracts

While this is a documentation project, we define "contracts" as:

1. **SKILL.md Frontmatter Schema**: Required and optional fields per Vercel Labs spec
2. **Skill Content Template**: Standard sections each skill must include
3. **Code Example Standards**: Formatting and conventions for Vue 2 code snippets

**Output**: `contracts/skill-schema.md`, `contracts/skill-content-template.md`

### Quickstart Guide

Developer guide for:
- Installing skills locally via `npx skills`
- Testing skill discovery
- Validating YAML frontmatter
- Contributing new skills

**Output**: `quickstart.md`

## Phase 2: Task Generation

**NOT PART OF THIS COMMAND** - Use `/speckit.tasks` to generate `tasks.md` after Phase 1 is complete.

## Next Steps

1. Complete Phase 0: Run research tasks and consolidate findings into `research.md`
2. Complete Phase 1: Generate `data-model.md`, `contracts/`, and `quickstart.md`
3. Run agent context update script to inform the AI agent about new technologies
4. Re-validate Constitution Check
5. Proceed to `/speckit.tasks` for task breakdown
