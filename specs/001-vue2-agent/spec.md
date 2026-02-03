# Feature Specification: Vue 2.6.14 Agent Skills

**Feature Branch**: `001-vue2-agent`
**Created**: 2026-02-03
**Status**: Draft
**Input**: User description: "构建一个关于 vue 2.6.14版本的Agent skills，符合 https://github.com/vercel-labs/skills 相关规范"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Vue 2 Core Concepts Skill (Priority: P1)

As a developer working with an AI coding agent, I want the agent to have comprehensive knowledge of Vue 2.6.14 core concepts so that it can provide accurate guidance when writing Vue 2 code.

**Why this priority**: This is the foundational skill that enables all other Vue 2-related assistance. Without core concept knowledge, the agent cannot effectively help with Vue 2 development.

**Independent Test**: Can be tested by asking the agent questions about Vue 2 reactivity system, component lifecycle, template syntax, and directives, then verifying the accuracy of responses against Vue 2.6.14 documentation.

**Acceptance Scenarios**:

1. **Given** a user asks about Vue 2 reactivity, **When** the agent invokes this skill, **Then** the agent provides accurate information about Object.defineProperty-based reactivity, change detection caveats, and $set/$delete usage
2. **Given** a user asks about component lifecycle, **When** the agent invokes this skill, **Then** the agent correctly lists all lifecycle hooks (beforeCreate, created, beforeMount, mounted, beforeUpdate, updated, beforeDestroy, destroyed) in execution order
3. **Given** a user asks about template syntax, **When** the agent invokes this skill, **Then** the agent explains directives (v-if, v-for, v-bind, v-on, v-model), filters, and template limitations
4. **Given** a user asks about computed vs methods, **When** the agent invokes this skill, **Then** the agent explains the caching difference and when to use each

---

### User Story 2 - Vue 2 Component Patterns Skill (Priority: P2)

As a developer building Vue 2 applications, I want the agent to know Vue 2 component patterns and best practices so that it can help me structure components correctly.

**Why this priority**: Component architecture is central to Vue applications. This skill enables the agent to provide guidance on props, events, slots, and component communication patterns specific to Vue 2.

**Independent Test**: Can be tested by requesting the agent to create component examples with props, custom events, slots, and parent-child communication, then verifying the code follows Vue 2 conventions.

**Acceptance Scenarios**:

1. **Given** a user requests a component with props, **When** the agent invokes this skill, **Then** the agent creates a component with proper prop validation (type, required, default)
2. **Given** a user asks about component communication, **When** the agent invokes this skill, **Then** the agent explains props down, events up pattern, and provides $emit examples
3. **Given** a user needs slot usage, **When** the agent invokes this skill, **Then** the agent demonstrates named slots, scoped slots, and slot props syntax
4. **Given** a user asks about provide/inject, **When** the agent invokes this skill, **Then** the agent explains the pattern and warns about reactivity limitations

---

### User Story 3 - Vue 2 CLI and Build Tooling Skill (Priority: P3)

As a developer setting up Vue 2 projects, I want the agent to understand Vue CLI and webpack configuration so that it can help with project setup and build issues.

**Why this priority**: Project setup and build configuration are common pain points. This skill helps developers get started and troubleshoot build-related issues.

**Independent Test**: Can be tested by asking the agent to create a new Vue 2 project structure, explain vue.config.js options, or diagnose common build errors.

**Acceptance Scenarios**:

1. **Given** a user wants to create a Vue 2 project, **When** the agent invokes this skill, **Then** the agent provides Vue CLI 4.x or 5.x commands appropriate for Vue 2
2. **Given** a user asks about webpack configuration, **When** the agent invokes this skill, **Then** the agent explains vue.config.js structure and common configuration options
3. **Given** a user encounters a build error, **When** the agent invokes this skill, **Then** the agent suggests solutions for common Vue 2 build issues
4. **Given** a user asks about environment variables, **When** the agent invokes this skill, **Then** the agent explains .env files and process.env usage in Vue 2

---

### User Story 4 - Vue 2 Ecosystem Integration Skill (Priority: P4)

As a developer building production Vue 2 applications, I want the agent to know popular Vue 2 libraries and integration patterns so that it can recommend and implement common features.

**Why this priority**: Real-world applications require routing, state management, and other common functionality. This skill enables the agent to provide guidance on the Vue 2 ecosystem.

**Independent Test**: Can be tested by requesting the agent to set up Vue Router 3.x, Vuex 3.x, or other common Vue 2 libraries, then verifying correct usage patterns.

**Acceptance Scenarios**:

1. **Given** a user needs routing, **When** the agent invokes this skill, **Then** the agent implements Vue Router 3.x with dynamic routes, navigation guards, and lazy loading
2. **Given** a user needs state management, **When** the agent invokes this skill, **Then** the agent implements Vuex 3.x with modules, getters, mutations, and actions
3. **Given** a user asks about HTTP requests, **When** the agent invokes this skill, **Then** the agent demonstrates axios integration with interceptors
4. **Given** a user needs UI components, **When** the agent invokes this skill, **Then** the agent recommends Vue 2-compatible component libraries (Element UI, Vuetify, Ant Design Vue)

---

### Edge Cases

- What happens when the user is working with Vue 3 instead of Vue 2?
- How does the agent handle questions about deprecated Vue 1.x syntax?
- What if the user asks about TypeScript integration with Vue 2?
- How does the agent handle migration questions from Vue 2 to Vue 3?
- What if the user asks about Vue 2 SSR (Nuxt.js) vs client-side only?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The agent skills MUST follow the Vercel Labs skills specification format (SKILL.md files with YAML frontmatter)
- **FR-002**: Each skill MUST include a `name` field (lowercase, hyphens allowed) and `description` field
- **FR-003**: Vue 2 Core Concepts skill MUST cover reactivity system, component lifecycle, directives, and template syntax
- **FR-004**: Vue 2 Component Patterns skill MUST cover props, events, slots, and component communication
- **FR-005**: Vue 2 CLI and Build Tooling skill MUST cover Vue CLI, webpack, and vue.config.js
- **FR-006**: Vue 2 Ecosystem Integration skill MUST cover Vue Router 3.x, Vuex 3.x, and common libraries
- **FR-007**: All code examples in skills MUST be valid Vue 2.6.14 syntax
- **FR-008**: Skills MUST include version information specifying compatibility with Vue 2.6.14
- **FR-009**: Skills MUST distinguish Vue 2 syntax from Vue 3 where differences exist
- **FR-010**: Each skill MUST include "When to Use" section explaining appropriate scenarios
- **FR-011**: Skills MUST be discoverable via the standard skills CLI (`npx skills`)
- **FR-012**: Skills MUST support installation to Claude Code (.claude/skills/)
- **FR-013**: Each skill MUST include troubleshooting guidance for common issues
- **FR-014**: Skills MUST reference official Vue 2 documentation for further learning

### Key Entities

- **SKILL.md File**: The primary artifact containing skill instructions with YAML frontmatter (name, description, metadata)
- **Skill Directory**: Container for each skill with optional supporting files (examples, tests)
- **Skill Frontmatter**: YAML metadata block with required fields (name, description) and optional fields (metadata.internal)
- **Skill Repository**: The collection of all Vue 2 agent skills following Vercel Labs specification

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Developers can install skills using `npx skills add` command and skills are discoverable by agents
- **SC-002**: Agent provides accurate Vue 2.6.14 guidance for at least 90% of core concept questions
- **SC-003**: Code examples generated by agent using skills are syntactically correct Vue 2 code
- **SC-004**: All skills include comprehensive "When to Use" guidance enabling appropriate skill activation
- **SC-005**: Skills cover all major Vue 2.6.14 features: reactivity, components, lifecycle, directives, router, and state management
- **SC-006**: Each skill file passes YAML frontmatter validation (name and description fields present)
- **SC-007**: Skills can be installed to at least Claude Code agent without manual configuration

### Non-Functional Requirements

- **NFR-001**: All skill content MUST be accessible and follow clear documentation standards
- **NFR-002**: Skill files MUST use valid YAML frontmatter (parseable by standard YAML parsers)
- **NFR-003**: Each skill MUST include clear, concise instructions that agents can follow without ambiguity
- **NFR-004**: Skills MUST be version-controlled and follow the project's constitution principles for code quality
- **NFR-005**: Skill descriptions MUST clearly indicate Vue 2.6.14 compatibility to prevent confusion with Vue 3

## Assumptions

1. Users have basic familiarity with Vue.js and are seeking Vue 2-specific guidance
2. The target agent is Claude Code, though skills should be compatible with other agents per Vercel Labs specification
3. Users have Node.js and npm/npx available for skill installation
4. The skills repository will be hosted on GitHub for easy installation via `npx skills add`
5. Vue 2.6.14 is the target version as it's the final Vue 2 release with long-term support considerations

## Dependencies

- Vercel Labs skills CLI (`npx skills`)
- Agent that supports the Agent Skills specification (Claude Code, Cursor, etc.)
- Git repository for hosting skills (GitHub or compatible service)
