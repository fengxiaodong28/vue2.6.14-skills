---
title: Dynamic Routes with Params in Vue Router 3.x
impact: MEDIUM
impactDescription: Dynamic routes enable parameterized URLs and data-driven navigation
type: capability
tags:
  - vue2.6.14
  - vue-router
  - dynamic-routes
  - route-params
  - url-parameters
---

# Dynamic Routes with Params in Vue Router 3.x

Impact: MEDIUM - Dynamic routes enable parameterized URLs for user profiles, product details, and other data-driven pages.

## Task Checklist

- [ ] Use colon syntax for dynamic segments (`:id`)
- [ ] Access params via `$route.params` in components
- [ ] Use `props: true` to pass params as component props
- [ ] Handle optional params with question mark (`:id?`)
- [ ] Validate params before using in data fetching

## Basic Dynamic Routes

```javascript
// router/index.js
import Vue from 'vue'
import Router from 'vue-router'
import User from '@/views/User.vue'
import Post from '@/views/Post.vue'

Vue.use(Router)

export default new Router({
  routes: [
    // Dynamic segment with :param
    {
      path: '/user/:id',
      component: User,
      name: 'user'
    },

    // Multiple dynamic segments
    {
      path: '/post/:category/:id',
      component: Post,
      name: 'post'
    }
  ]
})
```

## Accessing Route Params

```vue
<!-- User.vue -->
<template>
  <div>
    <h1>User Profile</h1>
    <p>User ID: {{ userId }}</p>
  </div>
</template>

<script>
export default {
  name: 'User',
  computed: {
    userId() {
      return this.$route.params.id
    }
  },
  watch: {
    '$route.params.id': {
      handler(newId) {
        this.fetchUserData(newId)
      },
      immediate: true
    }
  },
  methods: {
    fetchUserData(id) {
      // Fetch user data with ID
      fetch(`/api/users/${id}`)
        .then(res => res.json())
        .then(data => {
          this.user = data
        })
    }
  }
}
</script>
```

## Pass Route Params as Props

```javascript
// router/index.js
export default new Router({
  routes: [
    {
      path: '/user/:id',
      component: User,
      // Pass route params as props
      props: true
    }
  ]
})
```

```vue
<!-- User.vue - params now available as props -->
<script>
export default {
  name: 'User',
  props: {
    id: {
      type: [String, Number],
      required: true
    }
  },
  created() {
    // Use prop instead of $route.params
    this.fetchUserData(this.id)
  },
  methods: {
    fetchUserData(id) {
      fetch(`/api/users/${id}`)
        .then(res => res.json())
        .then(data => {
          this.user = data
        })
    }
  }
}
</script>
```

## Custom Props Function

```javascript
// router/index.js
export default new Router({
  routes: [
    {
      path: '/user/:id',
      component: User,
      // Custom props function
      props: (route) => ({
        id: Number(route.params.id),
        // Add query params as props too
        tab: route.query.tab || 'profile'
      })
    }
  ]
})
```

```vue
<script>
export default {
  props: {
    id: Number,
    tab: {
      type: String,
      default: 'profile'
    }
  }
}
</script>
```

## Optional Params

```javascript
// router/index.js
export default new Router({
  routes: [
    // Optional param with ?
    {
      path: '/product/:category/:id?',
      component: Product,
      name: 'product'
    },

    // Or with default route
    {
      path: '/search/:query?',
      component: Search,
      name: 'search'
    }
  ]
})
```

```vue
<script>
export default {
  computed: {
    productId() {
      // Handle undefined optional param
      return this.$route.params.id || 'default'
    }
  }
}
</script>
```

## Programmatic Navigation with Params

```javascript
// Using path string
this.$router.push(`/user/${userId}`)

// Using named route with params
this.$router.push({
  name: 'user',
  params: { id: userId }
})

// With query params
this.$router.push({
  name: 'user',
  params: { id: userId },
  query: { tab: 'settings' }
})

// Replace instead of push (no history entry)
this.$router.replace({
  name: 'user',
  params: { id: newUserId }
})
```

## Advanced Matching Patterns

```javascript
// router/index.js
export default new Router({
  routes: [
    // One or more segments (*)
    {
      path: '/user/:id/posts/*',
      component: Posts,
      name: 'user-posts'
    },

    // Repeat param
    {
      path: '/tags/:tags+',
      component: TagSearch,
      name: 'tag-search'
    },

    // Zero or more
    {
      path: '/files/:path*',
      component: FileBrowser,
      name: 'files'
    }
  ]
})
```

```javascript
// Access wildcard params
this.$route.params.pathMatch // Everything after /files/
```

## Params Validation

```vue
<script>
export default {
  props: {
    id: {
      type: [String, Number],
      required: true,
      validator: value => {
        // Validate ID format
        return !isNaN(value) && value > 0
      }
    }
  },
  watch: {
    id: {
      handler(newId) {
        if (!this.isValidId(newId)) {
          this.$router.push({ name: 'not-found' })
          return
        }
        this.loadData(newId)
      },
      immediate: true
    }
  },
  methods: {
    isValidId(id) {
      return id && !isNaN(id) && id > 0
    },
    loadData(id) {
      // Fetch data
    }
  }
}
</script>
```

## Nested Routes with Params

```javascript
// router/index.js
export default new Router({
  routes: [
    {
      path: '/user/:id',
      component: User,
      props: true,
      children: [
        {
          path: 'posts',
          component: UserPosts,
          props: true
        },
        {
          path: 'posts/:postId',
          component: PostDetail,
          props: true
        }
      ]
    }
  ]
})
```

```vue
<!-- UserPosts.vue - can access parent param -->
<script>
export default {
  props: {
    id: String // From parent route
  },
  computed: {
    // Access both parent and child params
    params() {
      return this.$route.params
    }
  }
}
</script>
```

## Router Guard with Params

```javascript
// router/index.js
export default new Router({
  routes: [
    {
      path: '/admin/:section',
      component: AdminPanel,
      beforeEnter: (to, from, next) => {
        // Validate params
        const validSections = ['users', 'settings', 'content']
        if (!validSections.includes(to.params.section)) {
          next({ name: 'not-found' })
        } else {
          next()
        }
      }
    }
  ]
})
```

## Redirect with Params

```javascript
export default new Router({
  routes: [
    // Redirect old URL pattern to new pattern
    {
      path: '/profile/:userId',
      redirect: to => {
        return {
          name: 'user',
          params: { id: to.params.userId }
        }
      }
    },

    // Dynamic redirect
    {
      path: '/old/:category/:id',
      redirect: to => {
        const { category, id } = to.params
        return { name: 'product', params: { type: category, id } }
      }
    }
  ]
})
```

## Alias with Params

```javascript
export default new Router({
  routes: [
    {
      path: '/user/:id',
      component: User,
      props: true,
      // Alias with different param name
      alias: '/profile/:userId',
      props: route => ({
        id: route.params.userId || route.params.id
      })
    }
  ]
})
```

## SEO-Friendly URLs

```javascript
// Use slug instead of ID
export default new Router({
  routes: [
    {
      path: '/blog/:slug',
      component: BlogPost,
      name: 'blog-post'
    }
  ]
})
```

```javascript
// Generate slug from title
function slugify(title) {
  return title
    .toLowerCase()
    .replace(/[^a-z0-9]+/g, '-')
    .replace(/(^-|-$)/g, '')
}

// Navigate
this.$router.push({
  name: 'blog-post',
  params: { slug: slugify(post.title) }
})
```

## Common Patterns

```javascript
// 1. Pagination
{
  path: '/products/page/:page',
  component: ProductList,
  props: route => ({
    page: Number(route.params.page) || 1
  })
}

// 2. Date-based routes
{
  path: '/archive/:year/:month?/:day?',
  component: Archive,
  props: true
}

// 3. Multi-level hierarchy
{
  path: '/shop/:category/:subcategory?',
  component: Shop,
  props: true
}

// 4. Locale routing
{
  path: '/:locale/about',
  component: About,
  beforeEnter: (to, from, next) => {
    const locales = ['en', 'zh', 'ja']
    if (locales.includes(to.params.locale)) {
      next()
    } else {
      next('/en/about')
    }
  }
}
```

## Params in Navigation

```vue
<template>
  <div>
    <!-- Router link with params -->
    <router-link :to="{ name: 'user', params: { id: user.id } }">
      View Profile
    </router-link>

    <!-- With query params -->
    <router-link
      :to="{
        name: 'user',
        params: { id: user.id },
        query: { tab: 'activity' }
      }"
    >
      Activity Feed
    </router-link>

    <!-- Or direct path -->
    <router-link :to="`/user/${user.id}`">
      Simple Link
    </router-link>
  </div>
</template>
```

## Reference

- [Vue Router 3 - Dynamic Route Matching](https://v3.router.vuejs.org/guide/essentials/dynamic-matching.html)
