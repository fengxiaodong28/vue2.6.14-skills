---
title: Vuex 3.x Store Setup and Modules for Vue 2
impact: HIGH
impactDescription: Vue 2 requires Vuex 3.x, NOT Vuex 4.x which is for Vue 3 only
type: best-practice
tags:
  - vue2
  - vuex
  - vuex-3
  - state-management
  - store
  - modules
---

# Vuex 3.x Store Setup and Modules for Vue 2

Impact: HIGH - Vue 2 requires Vuex 3.x. Vuex 4.x is ONLY compatible with Vue 3. Installing the wrong version will cause errors.

## Task Checklist

- [ ] Install `vuex@3` for Vue 2 projects
- [ ] Create store with modules for better organization
- [ ] Use namespaced modules to avoid naming conflicts

## Installation

```bash
# CORRECT: Vuex 3.x for Vue 2
npm install vuex@3
# OR
yarn add vuex@3

# WRONG: Vuex 4.x is for Vue 3 only
npm install vuex@4
```

## Basic Store Setup

```javascript
// src/store/index.js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    count: 0,
    user: null
  },
  getters: {
    doubleCount: state => state.count * 2,
    isAuthenticated: state => !!state.user
  },
  mutations: {
    INCREMENT(state) {
      state.count++
    },
    SET_USER(state, user) {
      state.user = user
    }
  },
  actions: {
    increment({ commit }) {
      commit('INCREMENT')
    },
    async login({ commit }, credentials) {
      const user = await api.login(credentials)
      commit('SET_USER', user)
    }
  }
})
```

## Store with Modules (Recommended)

```javascript
// src/store/modules/user.js
export default {
  namespaced: true, // Important for avoiding naming conflicts

  state: {
    currentUser: null,
    isAuthenticated: false
  },

  getters: {
    userId: state => state.currentUser?.id || null,
    userName: state => state.currentUser?.name || ''
  },

  mutations: {
    SET_USER(state, user) {
      state.currentUser = user
      state.isAuthenticated = !!user
    },
    CLEAR_USER(state) {
      state.currentUser = null
      state.isAuthenticated = false
    }
  },

  actions: {
    async login({ commit }, credentials) {
      try {
        const user = await api.login(credentials)
        commit('SET_USER', user)
      } catch (error) {
        commit('CLEAR_USER')
        throw error
      }
    },

    async logout({ commit }) {
      await api.logout()
      commit('CLEAR_USER')
    }
  }
}
```

```javascript
// src/store/modules/posts.js
export default {
  namespaced: true,

  state: {
    posts: [],
    currentPost: null,
    loading: false,
    error: null
  },

  getters: {
    postById: state => id => {
      return state.posts.find(post => post.id === id)
    }
  },

  mutations: {
    SET_POSTS(state, posts) {
      state.posts = posts
    },
    SET_LOADING(state, loading) {
      state.loading = loading
    },
    SET_ERROR(state, error) {
      state.error = error
    }
  },

  actions: {
    async fetchPosts({ commit }) {
      commit('SET_LOADING', true)
      try {
        const posts = await api.getPosts()
        commit('SET_POSTS', posts)
      } catch (error) {
        commit('SET_ERROR', error)
      } finally {
        commit('SET_LOADING', false)
      }
    }
  }
}
```

```javascript
// src/store/index.js
import Vue from 'vue'
import Vuex from 'vuex'
import user from './modules/user'
import posts from './modules/posts'

Vue.use(Vuex)

export default new Vuex.Store({
  modules: {
    user,
    posts
  },
  strict: process.env.NODE_ENV !== 'production' // Enable strict mode in development
})
```

## Register with Vue Instance

```javascript
// src/main.js
import Vue from 'vue'
import App from './App.vue'
import store from './store'
import router from './router'

new Vue({
  store,
  router,
  render: h => h(App)
}).$mount('#app')
```

## Using Vuex in Components

```vue
<script>
import { mapState, mapGetters, mapMutations, mapActions } from 'vuex'

export default {
  computed: {
    // Using mapState with namespace
    ...mapState('user', ['currentUser', 'isAuthenticated']),
    ...mapState('posts', ['posts', 'loading']),

    // Using mapGetters with namespace
    ...mapGetters('user', ['userId', 'userName']),

    // Local computed
    localComputed() {
      return this.userName.toUpperCase()
    }
  },

  methods: {
    // Using mapActions with namespace
    ...mapActions('user', ['login', 'logout']),
    ...mapActions('posts', ['fetchPosts']),

    // Local method
    async handleLogin() {
      try {
        await this.login({ username: 'user', password: 'pass' })
      } catch (error) {
        console.error('Login failed', error)
      }
    }
  },

  created() {
    this.fetchPosts()
  }
}
</script>
```

## Direct Store Access (Without Helpers)

```javascript
// Access state
this.$store.state.user.currentUser
this.$store.state.posts.loading

// Access getters
this.$store.getters['user/userName']
this.$store.getters['posts/postById'](123)

// Commit mutations
this.$store.commit('user/SET_USER', userData)
this.$store.commit('posts/SET_LOADING', true)

// Dispatch actions
this.$store.dispatch('user/login', credentials)
this.$store.dispatch('posts/fetchPosts')
```

## Reference

- [Vuex 3.x Documentation](https://vuex.vuejs.org/)
- [Vuex Modules](https://vuex.vuejs.org/guide/modules.html)
