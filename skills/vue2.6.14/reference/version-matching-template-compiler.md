---
title: vue-template-compiler Version Must Match vue Version
impact: CRITICAL
impactDescription: Version mismatch between vue and vue-template-compiler causes build failures
type: best-practice
tags:
  - vue2
  - vue-cli
  - vue-template-compiler
  - version-matching
  - build-errors
---

# vue-template-compiler Version Must Match vue Version

Impact: CRITICAL - The `vue-template-compiler` package version MUST exactly match the `vue` package version. A version mismatch will cause compilation errors, build failures, or runtime issues.

## Task Checklist

- [ ] Ensure `vue` and `vue-template-compiler` have exact same major and minor version
- [ ] Check package.json for version alignment
- [ ] Lock versions with exact version numbers if needed

## The Problem

`vue-template-compiler` is tightly coupled to specific `vue` versions. Using mismatched versions causes:

- Compilation errors
- Incorrect template rendering
- Build failures
- Runtime warnings

## Correct Version Matching

```json
{
  "dependencies": {
    "vue": "^2.6.14"
  },
  "devDependencies": {
    "vue-template-compiler": "^2.6.14"
  }
}
```

**Important**: The major and minor versions MUST match:
- `vue@2.6.14` → `vue-template-compiler@2.6.14` ✅
- `vue@2.6.14` → `vue-template-compiler@2.7.16` ❌ (Won't work)
- `vue@2.6.14` → `vue-template-compiler@2.5.x` ❌ (Won't work)

## Checking Current Versions

```bash
# Check installed versions
npm list vue vue-template-compiler

# Output should show matching versions:
# ├── vue@2.6.14
# └── vue-template-compiler@2.6.14
```

## Fixing Version Mismatch

```bash
# Find matching versions
npm view vue versions --json | grep "2.6."

# Install matching versions
npm install vue@2.6.14 vue-template-compiler@2.6.14 --save-exact

# OR update both to latest Vue 2
npm install vue@2.7.16 vue-template-compiler@2.7.16 --save-exact
```

## package-lock.json Issues

If you still get version mismatches after fixing package.json:

```bash
# Remove node_modules and lock file
rm -rf node_modules package-lock.json

# Reinstall dependencies
npm install
```

## Vue 2.7 Note

Vue 2.7 (the final Vue 2 release) includes composition API backported from Vue 3. If using Vue 2.7:

```json
{
  "dependencies": {
    "vue": "^2.7.16"
  },
  "devDependencies": {
    "vue-template-compiler": "^2.7.16"
  }
}
```

## Common Error Messages

```
Error: [Vue warn]: Failed to mount component: template or render function not defined.

TypeError: Cannot read property 'parseComponent' of undefined

Module build failed: Error: Cannot find module 'vue-template-compiler'
```

## Reference

- [Vue 2 Installation](https://vue2.docs.vuejs.org/v2/guide/installation.html#Release-Notes)
- [vue-template-compiler npm](https://www.npmjs.com/package/vue-template-compiler)
