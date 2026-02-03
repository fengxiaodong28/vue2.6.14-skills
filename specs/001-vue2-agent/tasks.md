# Tasks: Vue 2.6.14 Agent Skills

**Input**: Design documents from `/specs/001-vue2-agent/`
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: This is a documentation/skills project. Validation is performed through `npx skills` discoverability and YAML frontmatter parsing.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3, US4)
- Include exact file paths in descriptions

## Path Conventions

- **Skills repository**: `skills/` at repository root
- Each skill directory: `skills/<skill-name>/SKILL.md`
- Paths shown below reflect the project structure from plan.md

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [x] T001 Create skills directory structure at skills/
- [x] T002 Create README.md in repository root with project description and installation instructions
- [x] T003 [P] Create .gitignore excluding node_modules/, .DS_Store, *.log
- [x] T004 [P] Create package.json with name, description, and repository metadata
- [x] T005 [P] Create LICENSE file (MIT recommended for open source skills)

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [x] T006 Validate Vercel Labs skills specification compliance by reviewing contracts/skill-schema.md
- [x] T007 Validate skill content template structure from contracts/skill-content-template.md
- [x] T008 Review Vue 2.6.14 vs Vue 3 differences from research.md to avoid syntax conflation
- [x] T009 Review TypeScript integration patterns (Vue.extend) from research.md
- [x] T010 Review Less preprocessor configuration from research.md

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - Vue 2 Core Concepts Skill (Priority: P1) 🎯 MVP

**Goal**: Create the foundational Vue 2.6.14 core concepts skill that enables all other Vue 2 assistance.

**Independent Test**: Ask the agent questions about Vue 2 reactivity, lifecycle, and directives. Verify responses are accurate against Vue 2.6.14 documentation and do NOT include Vue 3 Composition API syntax.

### Implementation for User Story 1

- [x] T011 [P] [US1] Create skills/vue2-core-concepts/ directory
- [x] T012 [US1] Create skills/vue2-core-concepts/SKILL.md with YAML frontmatter (name: vue2-core-concepts, description: Vue 2.6.14 Options API, reactivity system, lifecycle hooks, and directives)
- [x] T013 [US1] Add "When to Use" section with scenarios: writing Vue 2 components, explaining reactivity, understanding lifecycle, using directives, distinguishing Vue 2 from Vue 3
- [x] T014 [US1] Add "Reactivity System" section covering Object.defineProperty-based reactivity, change detection caveats, $set/$delete usage
- [x] T015 [US1] Add "Component Lifecycle" section with all 8 hooks in execution order (beforeCreate, created, beforeMount, mounted, beforeUpdate, updated, beforeDestroy, destroyed)
- [x] T016 [US1] Add "Template Syntax & Directives" section covering v-if, v-for, v-bind, v-on, v-model, filters
- [x] T017 [US1] Add "Computed vs Methods" section explaining caching differences and when to use each
- [x] T018 [US1] Add "Common Gotchas" section with v-for + v-if issue, missing :key, reactivity caveats with solutions
- [x] T019 [US1] Add code examples using Vue 2.6.14 Options API with Vue.extend(), TypeScript, and Less
- [x] T020 [US1] Add "Related Skills" section referencing [vue2-component-patterns], [vue2-cli-build-tooling], [vue2-ecosystem-integration]
- [x] T021 [US1] Validate YAML frontmatter with name and description fields

**Checkpoint**: At this point, User Story 1 (vue2-core-concepts skill) should be fully functional and testable independently

---

## Phase 4: User Story 2 - Vue 2 Component Patterns Skill (Priority: P2)

**Goal**: Create the component patterns skill for props, events, slots, and TypeScript integration.

**Independent Test**: Request the agent to create components with props, events, and slots. Verify code follows Vue 2 Options API conventions and uses TypeScript with Vue.extend().

### Implementation for User Story 2

- [x] T022 [P] [US2] Create skills/vue2-component-patterns/ directory
- [x] T023 [US2] Create skills/vue2-component-patterns/SKILL.md with YAML frontmatter (name: vue2-component-patterns, description: Vue 2.6.14 component patterns with props, events, slots, and TypeScript)
- [x] T024 [US2] Add "When to Use" section with scenarios: creating components with props, component communication, using slots, TypeScript integration
- [x] T025 [US2] Add "Props & Validation" section covering prop types, required props, default values, custom validators
- [x] T026 [US2] Add "Events & $emit" section covering custom events, event validation, props down / events up pattern
- [x] T027 [US2] Add "Slots" section covering default slots, named slots, scoped slots with slot props
- [x] T028 [US2] Add "Component Communication Patterns" section covering parent-child, sibling (via event bus or parent), provide/inject with reactivity warnings
- [x] T029 [US2] Add "TypeScript Integration" section covering Vue.extend() pattern, interface definitions, PropType usage
- [x] T030 [US2] Add "Common Gotchas" section covering prop mutation, .sync modifier, provide/inject reactivity limitations
- [x] T031 [US2] Add code examples showing complete Vue 2 SFCs with props, events, slots, TypeScript (Vue.extend), and Less
- [x] T032 [US2] Add "Related Skills" section referencing [vue2-core-concepts], [vue2-cli-build-tooling]
- [x] T033 [US2] Validate YAML frontmatter and ensure no Vue 3 syntax (defineProps, defineEmits, script setup)

**Checkpoint**: At this point, User Stories 1 AND 2 should both work independently

---

## Phase 5: User Story 3 - Vue 2 CLI and Build Tooling Skill (Priority: P3)

**Goal**: Create the CLI and build tooling skill for project setup and configuration.

**Independent Test**: Ask the agent to create a Vue 2 project structure, explain vue.config.js, or configure Less. Verify guidance is specific to Vue 2 and Vue CLI 4.x/5.x.

### Implementation for User Story 3

- [x] T034 [P] [US3] Create skills/vue2-cli-build-tooling/ directory
- [x] T035 [US3] Create skills/vue2-cli-build-tooling/SKILL.md with YAML frontmatter (name: vue2-cli-build-tooling, description: Vue CLI 4.x/5.x, webpack configuration, Less preprocessor, and TypeScript setup for Vue 2.6.14)
- [x] T036 [US3] Add "When to Use" section with scenarios: creating Vue 2 projects, configuring webpack, setting up Less, configuring TypeScript, troubleshooting build errors
- [x] T037 [US3] Add "Project Setup" section covering Vue CLI 4.x/5.x installation, project creation commands for Vue 2
- [x] T038 [US3] Add "vue.config.js Configuration" section covering publicPath, configureWebpack, chainWebpack, devServer, css loader options
- [x] T039 [US3] Add "Less Preprocessor Setup" section covering less/less-loader installation, global variables, mixins, vue.config.js css.loaderOptions
- [x] T040 [US3] Add "TypeScript Configuration" section covering tsconfig.json, Vue.extend() usage, @types/vue declarations
- [x] T041 [US3] Add "Environment Variables" section covering .env files, process.env usage, vue.config.js env variables
- [x] T042 [US3] Add "Common Build Issues" section covering webpack errors, Less compilation issues, TypeScript errors, vue-template-compiler version matching
- [x] T043 [US3] Add code examples showing vue.config.js configurations and project structure
- [x] T044 [US3] Add "Related Skills" section referencing [vue2-core-concepts], [vue2-component-patterns], [vue2-ecosystem-integration]
- [x] T045 [US3] Validate YAML frontmatter and ensure guidance is specific to Vue 2 (not Vue 3 + Vite)

**Checkpoint**: All user stories should now be independently functional

---

## Phase 6: User Story 4 - Vue 2 Ecosystem Integration Skill (Priority: P4)

**Goal**: Create the ecosystem integration skill for Vue Router, Vuex, and common libraries.

**Independent Test**: Request the agent to set up Vue Router 3.x, Vuex 3.x, or axios. Verify correct usage patterns for Vue 2 ecosystem.

### Implementation for User Story 4

- [x] T046 [P] [US4] Create skills/vue2-ecosystem-integration/ directory
- [x] T047 [US4] Create skills/vue2-ecosystem-integration/SKILL.md with YAML frontmatter (name: vue2-ecosystem-integration, description: Vue Router 3.x, Vuex 3.x, axios, and Vue 2-compatible UI libraries)
- [x] T048 [US4] Add "When to Use" section with scenarios: adding routing, state management, HTTP requests, UI components
- [x] T049 [US4] Add "Vue Router 3.x" section covering router setup, dynamic routes, navigation guards, lazy loading, router-link and router-view
- [x] T050 [US4] Add "Vuex 3.x" section covering store setup, state, getters, mutations, actions, modules
- [x] T051 [US4] Add "HTTP Client (axios)" section covering installation, interceptors, request/response handling, integration with Vuex
- [x] T052 [US4] Add "UI Component Libraries" section covering Element UI, Vuetify, Ant Design Vue (all Vue 2 compatible)
- [x] T053 [US4] Add "Integration Patterns" section covering router + state management, API integration patterns
- [x] T054 [US4] Add "Common Gotchas" section covering router vs component guards, Vuex reactivity in Vue 2, axios cancellation
- [x] T055 [US4] Add code examples showing Vue Router 3.x, Vuex 3.x, and axios integration with Vue 2 components
- [x] T056 [US4] Add "Related Skills" section referencing [vue2-core-concepts], [vue2-component-patterns], [vue2-cli-build-tooling]
- [x] T057 [US4] Validate YAML frontmatter and ensure all ecosystem libraries are Vue 2 compatible (Router 3.x, Vuex 3.x, NOT Vue 3 versions)

**Checkpoint**: All user stories should now be independently functional

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [x] T058 [P] Run `npx skills list` to verify all 4 skills are discoverable
- [x] T059 [P] Validate YAML frontmatter for all skills using Python YAML parser
- [x] T060 [P] Cross-reference all skills: ensure [skill-name] references are correct
- [x] T061 [P] Verify all code examples use Vue 2.6.14 Options API (NO Composition API, NO script setup)
- [x] T062 [P] Verify all code examples use TypeScript with Vue.extend() pattern
- [x] T063 [P] Verify all code examples use Less with `<style lang="less" scoped>`
- [x] T064 [P] Ensure no Vue 3 syntax leaks into any skill (defineProps, defineEmits, setup, ref, reactive, etc.)
- [x] T065 Test each skill with AI agent by asking relevant Vue 2 questions
- [x] T066 Update README.md with installation instructions and skill descriptions
- [x] T067 Add skill usage examples to README.md

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3-6)**: All depend on Foundational phase completion
  - User stories can then proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 → P2 → P3 → P4)
- **Polish (Phase 7)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P2)**: Can start after Foundational (Phase 2) - References US1 but independently testable
- **User Story 3 (P3)**: Can start after Foundational (Phase 2) - References US1/US2 but independently testable
- **User Story 4 (P4)**: Can start after Foundational (Phase 2) - References US1/US2/US3 but independently testable

### Within Each User Story

- Directory creation can be parallel ([P] marked)
- SKILL.md creation must happen before content sections
- Content sections can be added in any order (but sequential for logical flow)
- Validation happens after content is complete

### Parallel Opportunities

- All Setup tasks marked [P] can run in parallel (T003, T004, T005)
- All user story directory creation tasks are [P] (T011, T022, T034, T046)
- Different user stories can be worked on in parallel by different team members
- All Polish validation tasks marked [P] can run in parallel

---

## Parallel Example: User Story 1

```bash
# Launch directory creation for all user stories together:
Task: "Create skills/vue2-core-concepts/ directory"
Task: "Create skills/vue2-component-patterns/ directory"
Task: "Create skills/vue2-cli-build-tooling/ directory"
Task: "Create skills/vue2-ecosystem-integration/ directory"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL - blocks all stories)
3. Complete Phase 3: User Story 1 (vue2-core-concepts)
4. **STOP and VALIDATE**: Test skill with agent, verify Vue 2 guidance accuracy
5. Demo/publish if ready (MVP delivers core Vue 2 knowledge)

### Incremental Delivery

1. Complete Setup + Foundational → Foundation ready
2. Add User Story 1 → Test independently → Deploy/Demo (MVP!)
3. Add User Story 2 → Test independently → Deploy/Demo
4. Add User Story 3 → Test independently → Deploy/Demo
5. Add User Story 4 → Test independently → Deploy/Demo
6. Each story adds value without breaking previous stories

### Parallel Team Strategy

With multiple developers (or AI agents):

1. Team completes Setup + Foundational together
2. Once Foundational is done:
   - Developer A: User Story 1 (vue2-core-concepts)
   - Developer B: User Story 2 (vue2-component-patterns)
   - Developer C: User Story 3 (vue2-cli-build-tooling)
   - Developer D: User Story 4 (vue2-ecosystem-integration)
3. Stories complete and integrate independently

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Stop at any checkpoint to validate story independently
- Avoid: Vue 3 syntax, Composition API, script setup, Vite (use Vue CLI)
- All code examples MUST use: Vue 2.6.14 + Options API + TypeScript (Vue.extend) + Less
