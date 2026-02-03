---
title: Vue Router 3.x Basic Setup for Vue 2
impact: HIGH
impactDescription: Vue Router 3.x is the compatible version for Vue 2, NOT Vue Router 4.x
type: best-practice
tags:
  - vue2.6.14
  - vue-router
  - vue-router-3
  - router
  - routing
---

# Vue Router 3.x Basic Setup for Vue 2

Impact: HIGH - Vue 2 requires Vue Router 3.x. Vue Router 4.x is ONLY compatible with Vue 3. Installing the wrong version will cause errors.

## Task Checklist

- [ ] Install `vue-router@3` for Vue 2 projects
- [ ] Create router configuration file in `src/router/index.js`
- [ ] Register router with Vue instance in `main.js`

## Installation

```bash
# CORRECT: Vue Router 3.x for Vue 2
npm install vue-router@3
# OR
yarn add vue-router@3

# WRONG: Vue Router 4.x is for Vue 3 only
npm install vue-router@4
```

## Basic Router Setup

```javascript
// src/router/index.js
import Vue from 'vue'
import VueRouter from 'vue-router'
import Home from '@/views/Home.vue'
import About from '@/views/About.vue'
import User from '@/views/User.vue'

Vue.use(VueRouter)

const routes = [
  {
    path: '/',
    name: 'home',
    component: Home
  },
  {
    path: '/about',
    name: 'about',
    component: About
  },
  {
    path: '/user/:id',
    name: 'user',
    component: User,
    props: true // Pass route params as props
  },
  {
    // Catch-all 404
    path: '*',
    redirect: '/'
  }
]

const router = new VueRouter({
  mode: 'history', // or 'hash'
  base: process.env.BASE_URL,
  routes,
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      return savedPosition
    } else {
      return { x: 0, y: 0 }
    }
  }
})

export default router
```

## Register with Vue Instance

```javascript
// src/main.js
import Vue from 'vue'
import App from './App.vue'
import router from './router'

new Vue({
  router,
  render: h => h(App)
}).$mount('#app')
```

## Router View and Links

```vue
<template>
  <div id="app">
    <!-- Navigation links -->
    <nav>
      <router-link to="/">Home</router-link>
      <router-link to="/about">About</router-link>
      <router-link :to="{ name: 'user', params: { id: 123 } }">User</router-link>
    </nav>

    <!-- Route outlet -->
    <router-view></router-view>
  </div>
</template>
```

## Dynamic Routes

```javascript
const routes = [
  {
    path: '/user/:id',
    name: 'user',
    component: User,
    props: true // Pass route params as props
  }
]

// In component
export default {
  props: {
    id: { // :id from route
      type: [String, Number],
      required: true
    }
  }
}
```

## Navigation Guards (Vue 2)

```javascript
// Global guards
router.beforeEach((to, from, next) => {
  if (to.matched.some(record => record.meta.requiresAuth)) {
    if (!isAuthenticated()) {
      next({ name: 'login' })
    } else {
      next()
    }
  } else {
    next()
  }
})

// Per-route guard
{
  path: '/admin',
  meta: { requiresAuth: true },
  beforeEnter: (to, from, next) => {
    // ...
  }
}

// Component guards
export default {
  beforeRouteEnter(to, from, next) {
    // Called before component is created
    next(vm => {
      // Access component instance via callback
    })
  },
  beforeRouteUpdate(to, from, next) {
    // Called when route changes but same component reused
  },
  beforeRouteLeave(to, from, next) {
    // Called when navigating away
  }
}
```

## Reference

- [Vue Router 3.x Documentation](https://v3.router.vuejs.org/)
- [Vue Router Installation](https://v3.router.vuejs.org/installation.html)
