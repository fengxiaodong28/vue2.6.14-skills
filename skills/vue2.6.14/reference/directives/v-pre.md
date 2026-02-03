---
title: v-pre Directive in Vue 2
impact: LOW
impactDescription: v-pre skips compilation for elements and children, useful for displaying raw mustache tags
type: capability
tags:
  - vue2.6.14
  - v-pre
  - directives
  - compilation
  - performance
---

# v-pre Directive in Vue 2

Impact: LOW - The `v-pre` directive skips compilation for elements and all their children. This is useful for displaying raw mustache tags and can speed up compilation for large numbers of nodes without directives.

## Task Checklist

- [ ] Use `v-pre` to display raw mustache tags without compilation
- [ ] Use `v-pre` to speed up compilation on static content
- [ ] Remember that v-pre skips all Vue directives on the element
- [ ] Do not use `v-pre` with child components or dynamic content

## Basic Usage

```vue
<template>
  <div>
    <!-- Normal interpolation - will be compiled -->
    <p>{{ message }}</p>
    <!-- Renders: Hello World -->

    <!-- With v-pre - skips compilation -->
    <p v-pre>{{ message }}</p>
    <!-- Renders literally: {{ message }} -->
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: 'Hello World'
    }
  }
}
</script>
```

## Displaying Raw Mustache Syntax

```vue
<template>
  <div>
    <h3>Vue Template Syntax Example:</h3>

    <!-- v-pre displays the raw template syntax -->
    <pre v-pre><code>{{ message }}</code></pre>
    <!-- Renders: {{ message }} -->

    <!-- Code block showing template syntax -->
    <div v-pre>
      <p>{{ showMessage }}</p>
      <button @click="handleClick">Click</button>
    </div>
    <!-- All Vue syntax is displayed as-is -->
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: 'Hello World'
    }
  }
}
</script>
```

## Performance Optimization

```vue
<template>
  <div>
    <!-- Large static content can use v-pre for faster compilation -->
    <div v-pre>
      <h1>Static Heading</h1>
      <p>Static paragraph with lots of content.</p>
      <p>Another static paragraph with {{ raw }} mustache syntax.</p>
      <p>No directives, just static content.</p>
    </div>

    <!-- Dynamic content without v-pre -->
    <div>
      <h1>{{ dynamicTitle }}</h1>
      <p>{{ dynamicContent }}</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      dynamicTitle: 'Dynamic Heading',
      dynamicContent: 'Dynamic content'
    }
  }
}
</script>
```

## Documentation and Code Examples

```vue
<template>
  <div class="documentation">
    <h2>Template Syntax Guide</h2>

    <!-- Show actual template syntax without compiling -->
    <div v-pre class="code-example">
<template>
  <div>{{ message }}</div>
</template>
    </div>

    <!-- Another example -->
    <pre v-pre><code>v-for="item in items" :key="item.id">{{ item.name }}</v-for></code></pre>
  </div>
</template>
```

## Comparison: With and Without v-pre

```vue
<template>
  <div>
    <!-- Without v-pre: normal compilation -->
    <div>
      <p>{{ computedMessage }}</p>
      <button @click="handleClick">Click Me</button>
    </div>

    <!-- With v-pre: raw template display -->
    <div v-pre>
      <p>{{ computedMessage }}</p>
      <button @click="handleClick">Click Me</button>
    </div>
  </div>
</template>

<script>
export default {
  computed: {
    computedMessage() {
      return 'This is computed'
    }
  },
  methods: {
    handleClick() {
      console.log('Clicked')
    }
  }
}
</script>

<!-- Renders: -->
<div>
  <!-- First div: compiled and reactive -->
  <div>
    <p>This is computed</p>
    <button>Click Me</button>
  </div>

  <!-- Second div: raw display -->
  <div>
    <p>{{ computedMessage }}</p>
    <button @click="handleClick">Click Me</button>
  </div>
</div>
```

## Important Limitations

**What v-pre does:**
- Skips Vue template compilation
- Displays content exactly as written
- Ignores all Vue directives (v-if, v-for, v-on, etc.)
- Ignores mustache interpolation

```vue
<template>
  <div v-pre>
    <!-- These will NOT work -->
    <div v-if="showContent">This won't be shown conditionally</div>
    <div v-for="item in items">{{ item.name }}</div>
    <button @click="handleClick">Click</button>
    <p>{{ interpolatedText }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      showContent: true,
      items: [],
      interpolatedText: 'Text'
    }
  },
  methods: {
    handleClick() {
      console.log('This won\'t execute')
    }
  }
}
</script>
```

**What happens:**
- `v-if` is not evaluated (always shown)
- `v-for` is not processed
- `@click` event handler is not attached
- `{{ interpolatedText }}` is displayed literally

## Use Cases

### 1. Documentation Sites

```vue
<template>
  <div class="code-block">
    <pre v-pre><code>{{ templateCode }}</code></pre>
  </div>
</template>

<script>
export default {
  data() {
    return {
      templateCode: `<div>{{ message }}</div>`
    }
  }
}
</script>
```

### 2. Code Showcase

```vue
<template>
  <div class="tutorial">
    <h3>Example Code:</h3>
    <div v-pre class="example">
<template>
  <div class="user-card">
    <h2>{{ user.name }}</h2>
    <p>{{ user.email }}</p>
  </div>
</template>
    </div>
  </div>
</template>
```

### 3. Template Documentation

```vue
<template>
  <article>
    <h2>Vue Template Syntax</h2>
    <p>Use double mustaches for interpolation:</p>
    <div v-pre class="syntax-example">
{{ variable }}
{{ object.property }}
{{ methodCall() }}
    </div>
  </article>
</template>
```

### 4. Performance for Static Content

```vue
<template>
  <div>
    <!-- Static content that won't change -->
    <footer v-pre class="static-footer">
      <p>© 2024 Company Inc. All rights reserved.</p>
      <p>{{ companyName }}</p>
      <a href="/privacy">Privacy</a>
      <a href="/terms">Terms</a>
    </footer>

    <!-- Dynamic content -->
    <nav>
      <router-link to="/">Home</router-link>
      <router-link to="/about">About</router-link>
    </nav>
  </div>
</template>

<script>
export default {
  data() {
    return {
      companyName: 'Company Inc.'
    }
  }
}
</script>
```

## v-pre vs v-once

```vue
<template>
  <div>
    <!-- v-pre: Skips compilation, but content can change -->
    <div v-if="isExampleShown" v-pre>
      {{ staticContent }}
    </div>

    <!-- v-once: Renders once, then becomes static -->
    <div v-once>
      {{ initialContent }}
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isExampleShown: true,
      staticContent: 'This is a template example',
      initialContent: 'Initial content'
    }
  }
}
</script>
```

## Common Pitfalls

### Pitfall 1: Using v-pre with Dynamic Content

```vue
<!-- ❌ WRONG: v-pre prevents reactivity -->
<template>
  <div v-pre>
    <p>{{ message }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: 'Hello'
    }
  }
}
</script>
<!-- The message won't update! -->
```

### Pitfall 2: Using v-pre with Child Components

```vue
<!-- ❌ WRONG: v-pre breaks child components -->
<template>
  <div v-pre>
    <child-component></child-component>
  </div>
</template>

<!-- ✅ CORRECT: Remove v-pre for child components -->
<template>
  <div>
    <child-component></child-component>
  </div>
</template>
```

## Best Practices

1. **Use v-pre** for:
   - Displaying raw template syntax
   - Documentation and tutorials
   - Static content optimization

2. **Avoid v-pre** for:
   - Dynamic or reactive content
   - Components with children
   - Event handling
   - Any v-* directives

3. **Consider alternatives**:
   - For code display: use `&lt;code&gt;` HTML entities
   - For static optimization: use `v-once` once rendered

## Reference

- [Vue 2 API - v-pre](https://vue2.docs.vuejs.org/v2/api/#v-pre)
- [Vue 2 Guide - Syntax - Raw HTML](https://vue2.docs.vuejs.org/v2/guide/syntax.html#Raw-HTML)
