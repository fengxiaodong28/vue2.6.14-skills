---
name: vue2.6.14
description: Vue 2.6.14 Options API best practices, reactivity system, component patterns, and ecosystem integration. Each reference shows Vue 2.6.14 specific solutions only.
version: 1.0.0
license: MIT
author: github.com/username
---

# Vue 2.6.14 Options API & Best Practices

Vue 2.6.14 development guide focusing on the Options API with Object.defineProperty-based reactivity, component patterns, and Vue 2 ecosystem integration (Vue Router 3.x, Vuex 3.x).

## Reactivity System

### Reactivity Caveats and Limitations

Vue 2 uses `Object.defineProperty` for reactivity which has specific limitations not present in Vue 3's Proxy-based system.

**Can't detect property addition on objects** → See [reactivity-property-addition](reference/reactivity-property-addition.md)

**Can't detect array index assignment** → See [reactivity-array-index-assignment](reference/reactivity-array-index-assignment.md)

**Can't detect array length changes** → See [reactivity-array-length-changes](reference/reactivity-array-length-changes.md)

**Frozen objects can't be made reactive** → See [reactivity-frozen-objects](reference/reactivity-frozen-objects.md)

### $set and $delete Usage

**Adding reactive properties to objects** → See [using-set-for-reactivity](reference/using-set-for-reactivity.md)

**Deleting reactive properties from objects** → See [using-delete-for-reactivity](reference/using-delete-for-reactivity.md)

**Vue.observable() for minimal state** → See [reactivity-observable](reference/reactivity-observable.md)

## Component Lifecycle

### Lifecycle Hooks

**Never use arrow functions in lifecycle hooks** → See [no-arrow-functions-in-lifecycle-hooks](reference/no-arrow-functions-in-lifecycle-hooks.md)

**Stateful methods shared across component instances** → See [stateful-methods-lifecycle](reference/stateful-methods-lifecycle.md)

### Vue 2 Specific Lifecycle Hooks

Vue 2 uses different lifecycle names compared to Vue 3:

- `beforeDestroy` (Vue 2) vs `beforeUnmount` (Vue 3)
- `destroyed` (Vue 2) vs `unmounted` (Vue 3)

**See**: [vue2-lifecycle-hooks](reference/vue2-lifecycle-hooks.md)

## Component Patterns

### Props and Validation

**Typing complex props with interfaces** → See [props-complex-types-interface](reference/props-complex-types-interface.md)

**Prop validation with custom validators** → See [props-custom-validators](reference/props-custom-validators.md)

**Default values for object and array props** → See [props-default-values-objects](reference/props-default-values-objects.md)

### Events and Component Communication

**Props down, events up pattern** → See [component-communication-props-events](reference/component-communication-props-events.md)

**Using .sync modifier (Vue 2.3+)** → See [sync-modifier-two-way-binding](reference/sync-modifier-two-way-binding.md)

**Event bus for sibling communication** → See [event-bus-pattern](reference/event-bus-pattern.md)

**Provide/Inject reactivity limitations** → See [provide-inject-not-reactive](reference/provide-inject-not-reactive.md)

### Slots

**Default and named slots** → See [slots-default-named](reference/slots-default-named.md)

**Scoped slots with slot props** → See [slots-scoped-slot-props](reference/slots-scoped-slot-props.md)

**v-for with v-if priority issue** → See [v-for-v-if-priority](reference/v-for-v-if-priority.md)

**Missing :key on v-for** → See [v-for-missing-key](reference/v-for-missing-key.md)

### Watchers

**Deep and immediate watch options** → See [watch-options-deep-immediate](reference/watch-options-deep-immediate.md)

## TypeScript Integration

### Vue.extend() Pattern

**Using Vue.extend() for TypeScript** → See [ts-vue-extend-pattern](reference/ts-vue-extend-pattern.md)

**Type-safe computed properties** → See [ts-computed-return-types](reference/ts-computed-return-types.md)

**Type-safe event handlers** → See [ts-options-api-type-event-handlers](reference/ts-options-api-type-event-handlers.md)

### Type Definitions

**PropType for complex prop types** → See [ts-proptype-complex-types](reference/ts-proptype-complex-types.md)

**Shims for .vue files** → See [ts-shims-vue-files](reference/ts-shims-vue-files.md)

## Build Tooling

### Vue CLI Configuration

**vue.config.js basic setup** → See [cli-vue-config-basic](reference/cli-vue-config-basic.md)

**Configuring Less preprocessor** → See [cli-less-configuration](reference/cli-less-configuration.md)

**Environment variables** → See [cli-environment-variables](reference/cli-environment-variables.md)

### Version Compatibility

**vue-template-compiler version matching** → See [version-matching-template-compiler](reference/version-matching-template-compiler.md)

**Common build errors and solutions** → See [cli-common-build-errors](reference/cli-common-build-errors.md)

## Ecosystem Integration

### Vue Router 3.x

**Router setup and configuration** → See [router3-basic-setup](reference/router3-basic-setup.md)

**Dynamic routes with params** → See [router3-dynamic-routes](reference/router3-dynamic-routes.md)

**Navigation guards** → See [router3-navigation-guards](reference/router3-navigation-guards.md)

**Lazy loading routes** → See [router3-lazy-loading](reference/router3-lazy-loading.md)

### Vuex 3.x

**Store setup and modules** → See [vuex3-store-modules](reference/vuex3-store-modules.md)

**State, getters, mutations, actions** → See [vuex3-core-concepts](reference/vuex3-core-concepts.md)

**Vuex reactivity gotchas** → See [vuex3-reactivity-gotchas](reference/vuex3-reactivity-gotchas.md)

## @vue/composition-api (Vue 2)

### Basic Setup

**Composition API plugin setup** → See [composition-api-setup](reference/composition-api-setup.md)

### Reactivity

**Ref vs Reactive for reactive state** → See [composition-api-ref-reactive](reference/composition-api-ref-reactive.md)

### Composables

**Creating reusable logic with composables** → See [composition-api-composables](reference/composition-api-composables.md)

### Vuex Integration

**Using Vuex with Composition API** → See [composition-api-vuex](reference/composition-api-vuex.md)

## vue-class-component (Vue 2)

### Basic Usage

**Class-based component definition** → See [vue-class-component-basics](reference/vue-class-component-basics.md)

### Mixins

**Using mixins with class components** → See [vue-class-component-mixins](reference/vue-class-component-mixins.md)

## Comparison

**Composition API vs Class Component** → See [composition-vs-class-component](reference/composition-vs-class-component.md)

## Vue 2 vs Vue 3 Differences

**Key syntax differences** → See [vue2-vs-vue3-syntax](reference/vue2-vs-vue3-syntax.md)

**Filters removed in Vue 3** → See [vue2-filters-deprecated](reference/vue2-filters-deprecated.md)

**$listeners and $attrs changes** → See [vue2-listeners-attrs](reference/vue2-listeners-attrs.md)
