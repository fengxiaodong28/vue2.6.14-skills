<!--
  Sync Impact Report
  ==================
  Version Change: Initial (0.0.0) -> 1.0.0
  Modified Principles: N/A (initial creation)
  Added Sections: All principles (Code Quality, Testing Standards, UX Consistency, Performance Requirements)
  Removed Sections: N/A
  Templates Status:
    - plan-template.md: Will require update to reference new principles
    - spec-template.md: Aligned with new principles
    - tasks-template.md: Testing discipline aligned
    - agent-file-template.md: No changes needed
    - Command files: No agent-specific references found
  Follow-up TODOs: None
-->

# Vue 2.6.14 Skills Constitution

## Core Principles

### I. Code Quality

Code quality is non-negotiable. All code MUST meet these standards before being considered complete:

- **Clean Code Practices**: Functions MUST be small and single-purpose. Each function should do one thing well. Cyclomatic complexity MUST NOT exceed 10 per function.
- **Naming Conventions**: Variables, functions, and classes MUST use descriptive names that clearly express intent. No abbreviations unless widely understood (e.g., `id`, `url`, `ctx`).
- **Code Organization**: Related functionality MUST be co-located. Directory structure MUST reflect architectural boundaries, not file types.
- **Documentation**: Public APIs, complex algorithms, and non-obvious business logic MUST include inline comments explaining "why", not "what".
- **Linting and Formatting**: All code MUST pass configured linters with zero warnings. Automated formatting MUST be applied before commit.
- **Error Handling**: Errors MUST be handled explicitly at appropriate boundaries. Silent failures and catch-all error handlers are prohibited.
- **Code Review**: All changes MUST undergo peer review focusing on correctness, maintainability, and adherence to these principles.

**Rationale**: High-quality code reduces bugs, improves maintainability, and enables the team to move faster. Technical debt from poor code quality compounds exponentially over time.

### II. Testing Standards

Testing is a first-class concern. These standards MUST be followed:

- **Test Coverage**: All code MUST have corresponding tests. Critical paths MUST have 100% coverage. Overall project coverage MUST NOT fall below 80%.
- **Test Types**: Three levels of testing are required:
  - **Unit Tests**: Test individual functions and components in isolation. MUST be fast and deterministic.
  - **Integration Tests**: Test interactions between components. MUST cover all API contracts and data flow.
  - **End-to-End Tests**: Test critical user journeys. MUST be limited to prevent brittleness.
- **Test-First for Critical Paths**: For core business logic and user-facing features, tests MUST be written before implementation (TDD). Tests MUST fail initially, then pass after implementation.
- **Test Independence**: Tests MUST NOT depend on execution order. Each test MUST set up its own state and clean up afterwards.
- **Mocking Strategy**: External dependencies (databases, APIs, file systems) MUST be mocked in unit tests. Integration tests MAY use real dependencies.
- **Assertion Quality**: Tests MUST assert on observable behavior, not implementation details. One assertion per test is preferred for clarity.
- **Test Maintenance**: Tests MUST be refactored alongside production code. Failing tests block deployment.

**Rationale**: A comprehensive test suite catches regressions early, documents expected behavior, and enables confident refactoring. Test-first development clarifies requirements before implementation.

### III. User Experience Consistency

User experience MUST be consistent across all features and interactions:

- **Design System**: All UI components MUST use a shared design system. Custom styles MUST be exceptions, not the norm.
- **Interaction Patterns**: Similar actions MUST behave consistently throughout the application. Common patterns (navigation, forms, feedback) MUST follow established conventions.
- **Responsive Design**: All features MUST work across supported viewport sizes. Mobile-first approach is preferred.
- **Accessibility**: All features MUST meet WCAG 2.1 AA standards. Keyboard navigation, screen reader support, and color contrast are mandatory.
- **Loading States**: Users MUST receive clear feedback for all async operations. Loading spinners, progress bars, or skeleton screens MUST be used appropriately.
- **Error Messages**: Error messages MUST be actionable and user-friendly. Technical jargon and stack traces MUST NOT be shown to end users.
- **Performance Feedback**: Perceived performance is as important as actual performance. Optimistic UI updates and progressive rendering MUST be used where appropriate.

**Rationale**: Consistent UX reduces cognitive load, builds user trust, and reduces support burden. Accessible design ensures all users can effectively use the application.

### IV. Performance Requirements

Performance is a feature. These requirements MUST be met:

- **Response Time Targets**:
  - Initial page load MUST complete within 3 seconds on 4G networks
  - Time to Interactive (TTI) MUST be under 5 seconds
  - API responses MUST return within 200ms for 95th percentile
  - User interactions (clicks, form submissions) MUST respond within 100ms
- **Resource Budgets**:
  - Initial JavaScript bundle MUST NOT exceed 200KB gzipped
  - Total page weight MUST NOT exceed 2MB for typical pages
  - Image assets MUST be optimized and served in modern formats (WebP, AVIF)
- **Runtime Performance**:
  - Frames per second MUST stay at or above 60fps during animations and scrolling
  - Memory leaks MUST be prevented. Long-running operations MUST NOT accumulate memory
  - Expensive computations MUST be debounced, throttled, or moved to web workers
- **Optimization Discipline**:
  - Performance MUST be measured before optimization
  - Profiling MUST be used to identify actual bottlenecks, not assumed ones
  - Critical rendering path MUST be optimized for first paint
- **Monitoring**: Core Web Vitals (LCP, FID, CLS) MUST be monitored and kept in "good" ranges

**Rationale**: Performance directly impacts user engagement, conversion rates, and accessibility. Slow applications frustrate users and increase abandonment.

## Development Standards

### Version Control

- **Commit Messages**: MUST follow conventional commit format (type: description). Types: feat, fix, docs, style, refactor, test, chore.
- **Branch Strategy**: Feature branches MUST be short-lived. Main branch MUST always be deployable.
- **Code Review**: All changes MUST be reviewed before merging. Reviewers MUST verify compliance with constitution principles.

### Security

- **Input Validation**: All user input MUST be validated and sanitized. Never trust client-side validation alone.
- **Dependency Management**: Dependencies MUST be kept up to date. Vulnerable packages MUST be patched within 7 days of disclosure.
- **Secrets Management**: Secrets MUST NOT be committed to repositories. Environment-specific configuration MUST be used.

## Governance

### Amendment Process

This constitution governs all development practices. Amendments require:

1. Proposal documenting the change and rationale
2. Team review and consensus approval
3. Migration plan for existing code and documentation
4. Template updates to reflect new principles

### Compliance

- All pull requests MUST demonstrate compliance with applicable principles
- Violations MUST be justified with documented rationale
- Complexity MUST be minimized and explicitly justified when necessary
- Templates and task generation MUST reflect current constitution principles

### Versioning

Constitution follows semantic versioning:
- **MAJOR**: Principle removal or backward-incompatible changes
- **MINOR**: New principles or material expansion of guidance
- **PATCH**: Clarifications, wording improvements, non-semantic changes

**Version**: 1.0.0 | **Ratified**: 2026-02-03 | **Last Amended**: 2026-02-03
