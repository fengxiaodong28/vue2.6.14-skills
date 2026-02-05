---
title: v-html Security Considerations in Vue 2
impact: HIGH
impactDescription: Using v-html with untrusted content can lead to XSS attacks
type: anti-pattern
tags:
  - vue2.6.14
  - v-html
  - security
  - xss
  - content-sanitization
---

# v-html Security Considerations in Vue 2

Impact: HIGH - Using `v-html` with untrusted user content can lead to Cross-Site Scripting (XSS) attacks. Never use `v-html` with user-provided content unless it has been properly sanitized.

## Problem

The `v-html` directive renders raw HTML as the element's innerHTML. If this content comes from user input or an untrusted source, it can execute malicious JavaScript code, leading to XSS attacks.

## Task Checklist

- [ ] Never use `v-html` with untrusted user input
- [ ] Always sanitize HTML content before using `v-html`
- [ ] Use a library like DOMPurify for HTML sanitization
- [ ] Consider safer alternatives like text interpolation or components
- [ ] Be aware that scoped styles don't apply to `v-html` content

## Incorrect: Using v-html with Untrusted Content

```vue
<template>
  <div>
    <!-- ❌ DANGEROUS: User input rendered directly as HTML -->
    <div v-html="userComment"></div>

    <!-- ❌ DANGEROUS: API response without sanitization -->
    <div v-html="apiResponse.content"></div>

    <!-- ❌ DANGEROUS: URL parameter content -->
    <div v-html="urlContent"></div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // User could submit malicious HTML/JS
      userComment: '<img src=x onerror=alert("XSS")>',
      apiResponse: { content: '' },
      urlContent: ''
    }
  },
  async created() {
    // API might return malicious content
    this.apiResponse = await fetchContentFromAPI()
    this.urlContent = this.$route.params.content
  }
}
</script>
```

## Correct: Sanitizing HTML Before Use

```vue
<template>
  <div>
    <!-- ✅ SAFE: Sanitized HTML -->
    <div v-html="sanitizedComment"></div>

    <!-- ✅ SAFE: Sanitized API content -->
    <div v-html="sanitizedContent"></div>
  </div>
</template>

<script>
import DOMPurify from 'dompurify'

export default {
  data() {
    return {
      userComment: '',
      apiResponse: { content: '' }
    }
  },
  computed: {
    sanitizedComment() {
      return DOMPurify.sanitize(this.userComment)
    },
    sanitizedContent() {
      return DOMPurify.sanitize(this.apiResponse.content, {
        // Specify allowed tags
        ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'u', 'a', 'ul', 'ol', 'li'],
        // Specify allowed attributes
        ALLOWED_ATTR: ['href']
      })
    }
  }
}
</script>
```

## Installing DOMPurify

```bash
npm install dompurify
# or
yarn add dompurify
```

## DOMPurify Basic Usage

```javascript
import DOMPurify from 'dompurify'

const dirty = '<img src=x onerror=alert(1)//>'
const clean = DOMPurify.sanitize(dirty)
// Result: '<img src="x">'

// With config
const cleanConfigured = DOMPurify.sanitize(dirty, {
  ALLOWED_TAGS: ['b', 'em', 'p'],
  ALLOWED_ATTR: []
})
```

## Vue Filter for HTML Sanitization

```javascript
// filters/sanitize.js
import DOMPurify from 'dompurify'

export default function sanitizeHtml(html) {
  return DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'u', 'a', 'ul', 'ol', 'li', 'h1', 'h2', 'h3', 'h4', 'h5', 'h6'],
    ALLOWED_ATTR: ['href', 'title', 'target'],
    ALLOW_DATA_ATTR: false
  })
}

// main.js - Register globally
import Vue from 'vue'
import sanitizeHtml from './filters/sanitize'

Vue.filter('sanitize', sanitizeHtml)
```

**Usage:**

```vue
<template>
  <div>
    <!-- Using filter -->
    <div v-html="userComment | sanitize"></div>

    <!-- Or in computed -->
    <div v-html="safeComment"></div>
  </div>
</template>

<script>
export default {
  computed: {
    safeComment() {
      return this.$options.filters.sanitize(this.userComment)
    }
  }
}
</script>
```

## Rich Text Editor with Sanitization

```vue
<template>
  <div class="rich-text-editor">
    <!-- Toolbar -->
    <div class="toolbar">
      <button @click="format('bold')">Bold</button>
      <button @click="format('italic')">Italic</button>
      <button @click="format('underline')">Underline</button>
      <button @click="insertLink">Link</button>
    </div>

    <!-- Content editable -->
    <div
      ref="editor"
      contenteditable="true"
      @input="handleInput"
      class="editor-content"
    ></div>

    <!-- Preview (sanitized) -->
    <div class="preview">
      <h4>Preview (Safe):</h4>
      <div v-html="sanitizedContent" class="preview-content"></div>
    </div>
  </div>
</template>

<script>
import DOMPurify from 'dompurify'

export default {
  data() {
    return {
      rawContent: ''
    }
  },
  computed: {
    sanitizedContent() {
      return DOMPurify.sanitize(this.rawContent, {
        ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'u', 'a', 'ul', 'ol', 'li'],
        ALLOWED_ATTR: ['href', 'target'],
        ALLOW_DATA_ATTR: false
      })
    }
  },
  mounted() {
    this.$refs.editor.addEventListener('paste', this.handlePaste)
  },
  beforeDestroy() {
    if (this.$refs.editor) {
      this.$refs.editor.removeEventListener('paste', this.handlePaste)
    }
  },
  methods: {
    handleInput(event) {
      this.rawContent = event.target.innerHTML
    },
    handlePaste(event) {
      // Prevent default paste
      event.preventDefault()

      // Get plain text from clipboard
      const text = event.clipboardData.getData('text/plain')

      // Insert as plain text (safe)
      document.execCommand('insertText', false, text)
    },
    format(command) {
      document.execCommand(command, false, null)
    },
    insertLink() {
      const url = prompt('Enter URL:')
      if (url) {
        document.execCommand('createLink', false, url)
      }
    }
  }
}
</script>

<style scoped>
.rich-text-editor {
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 16px;
}
.toolbar {
  display: flex;
  gap: 8px;
  margin-bottom: 12px;
}
.toolbar button {
  padding: 4px 12px;
  cursor: pointer;
}
.editor-content {
  min-height: 150px;
  border: 1px solid #ddd;
  padding: 12px;
}
.preview-content {
  border: 1px solid #ddd;
  padding: 12px;
  background: #f9f9f9;
  min-height: 50px;
}
</style>
```

## Alternative: Use Trusted HTML Components

```vue
<template>
  <div>
    <!-- Instead of v-html, parse and render trusted content -->
    <content-renderer :content="structuredContent" />
  </div>
</template>

<script>
import ContentRenderer from './ContentRenderer.vue'

export default {
  components: { ContentRenderer },
  data() {
    return {
      // Use structured data instead of raw HTML
      structuredContent: [
        { type: 'paragraph', content: 'Hello world' },
        { type: 'bold', content: 'Bold text' },
        { type: 'link', url: '/path', content: 'Click here' }
      ]
    }
  }
}
</script>
```

```vue
<!-- ContentRenderer.vue -->
<template>
  <div class="content-renderer">
    <component
      :is="getComponentForBlock(block)"
      v-for="(block, index) in content"
      :key="index"
      v-bind="getBlockProps(block)"
    >
      {{ block.content }}
    </component>
  </div>
</template>

<script>
export default {
  props: {
    content: Array
  },
  methods: {
    getComponentForBlock(block) {
      const components = {
        paragraph: 'p',
        bold: 'strong',
        italic: 'em',
        link: 'a',
        list: 'ul',
        listItem: 'li'
      }
      return components[block.type] || 'span'
    },
    getBlockProps(block) {
      if (block.type === 'link') {
        return { href: block.url, target: '_blank' }
      }
      return {}
    }
  }
}
</script>
```

## Common XSS Attack Examples

### 1. Script Injection

```javascript
// User input:
'<script>alert("XSS")</script>'

// With v-html - UNSAFE
<div v-html="userInput"></div>
// Result: Script executes!

// Sanitized - SAFE
<div v-html="DOMPurify.sanitize(userInput)"></div>
// Result: Script tag removed
```

### 2. Event Handler Injection

```javascript
// User input:
'<img src=x onerror="alert(\'XSS\')">'

// With v-html - UNSAFE
<div v-html="userInput"></div>
// Result: Alert executes!

// Sanitized - SAFE
<div v-html="DOMPurify.sanitize(userInput)"></div>
// Result: onerror removed
```

### 3. JavaScript Protocol

```javascript
// User input:
'<a href="javascript:alert(\'XSS\')">Click me</a>'

// Sanitized with default config - SAFE (href kept but dangerous)
<div v-html="DOMPurify.sanitize(userInput)"></div>

// Better sanitization - Remove dangerous href
DOMPurify.sanitize(userInput, {
  ALLOWED_URI_REGEXP: /^(?:(?:(?:f|ht)tps?|mailto|tel|callto|cid|xmpp):|[^a-z]|[a-z+.\-]+(?:[^a-z+.\-:]|$))/i
})
```

## DOMPurify Configuration Examples

```javascript
// Strict config - only basic formatting
const strictConfig = {
  ALLOWED_TAGS: ['p', 'br', 'strong', 'em'],
  ALLOWED_ATTR: []
}

// Medium config - links allowed
const mediumConfig = {
  ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'u', 'a'],
  ALLOWED_ATTR: ['href'],
  ALLOW_DATA_ATTR: false
}

// Loose config - more HTML elements
const looseConfig = {
  ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'u', 'a', 'ul', 'ol', 'li', 'h1', 'h2', 'h3', 'h4', 'h5', 'h6', 'blockquote', 'code', 'pre'],
  ALLOWED_ATTR: ['href', 'title'],
  ALLOW_DATA_ATTR: false
}

// For user-generated content, use strict or medium config
```

## v-html and Scoped Styles

```vue
<template>
  <div>
    <!-- Scoped styles don't apply to v-html content -->
    <div class="container">
      <div v-html="htmlContent" class="html-content"></div>
    </div>
  </div>
</template>

<style scoped>
/* This works on the container */
.container {
  padding: 16px;
}

/* This does NOT work on v-html content because of scoped styles */
.html-content p {
  color: red; /* Won't apply! */
}

/* Solution: Use :deep() or global styles */
</style>
```

**Solution 1: Use :deep() (Vue 2.7+) or >>>/::v-deep (older Vue 2)**

```vue
<style scoped>
.container >>> .html-content p {
  color: red;
}

/* or */
.container ::v-deep .html-content p {
  color: red;
}
</style>
```

**Solution 2: Use non-scoped style block**

```vue
<style>
/* Add a unique class to prevent conflicts */
.my-component-unique-class .html-content p {
  color: red;
}
</style>
```

## Alternative: Use Text Interpolation

```vue
<template>
  <div>
    <!-- For plain text, use mustache syntax (always safe) -->
    <p>{{ userComment }}</p>

    <!-- Only use v-html when HTML is actually needed -->
    <div v-if="needsHtml" v-html="sanitizedContent"></div>
  </div>
</template>
```

## Whitelist Approach for HTML

```javascript
// utils/html-whitelist.js
export const HTML_WHITELIST = {
  tags: ['p', 'br', 'strong', 'em', 'u', 'a', 'ul', 'ol', 'li', 'h1', 'h2', 'h3'],
  attributes: {
    a: ['href', 'title']
  }
}

export function sanitizeHtml(html, whitelist = HTML_WHITELIST) {
  return DOMPurify.sanitize(html, {
    ALLOWED_TAGS: whitelist.tags,
    ALLOWED_ATTR: Object.values(whitelist.attributes).flat()
  })
}
```

## Server-Side Sanitization

```javascript
// Express middleware example
const DOMPurify = require('dompurify')
const express = require('express')

const app = express()

app.use(express.json())

app.post('/api/comments', (req, res) => {
  const { content } = req.body

  // Sanitize on server (defense in depth)
  const sanitizedContent = DOMPurify.sanitize(content, {
    ALLOWED_TAGS: ['p', 'br', 'strong', 'em'],
    ALLOWED_ATTR: []
  })

  // Store sanitized content
  db.saveComment({ content: sanitizedContent })

  res.json({ success: true })
})
```

## Summary Table

| Approach | Safe? | When to Use |
|----------|-------|-------------|
| `{{ text }}` | ✅ Always safe | Plain text content |
| `v-html="rawHTML"` | ❌ Unsafe | Never with untrusted content |
| `v-html="DOMPurify.sanitize(html)"` | ✅ Safe | When HTML display is needed |
| Component rendering | ✅ Safe | Structured content |

## Best Practices

1. **Never** use `v-html` with untrusted user input
2. **Always** sanitize HTML content before using `v-html`
3. **Use** DOMPurify or similar sanitization library
4. **Consider** safer alternatives (text, components)
5. **Remember** scoped styles don't apply to `v-html` content
6. **Validate** content on server-side too (defense in depth)

## Reference

- [DOMPurify Documentation](https://github.com/cure53/DOMPurify)
- [Vue 2 Guide - v-html](https://vue2.docs.vuejs.org/v2/guide/syntax.html#Raw-HTML)
- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
