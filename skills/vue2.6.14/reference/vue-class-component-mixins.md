---
title: Vue Class Component Mixins for Vue 2
impact: MEDIUM
impactDescription: Using mixins with vue-class-component@7.x in Vue 2.6.14
type: best-practice
tags:
  - vue2
  - class-component
  - mixins
  - code-reuse
  - typescript
---

# Vue Class Component Mixins for Vue 2

Impact: MEDIUM - Mixins with vue-class-component enable code reuse across class-based components in Vue 2.6.14.

## Task Checklist

- [ ] Create mixin classes extending Vue
- [ ] Use Mixins() function to combine mixins
- [ ] Define shared logic in mixin classes
- [ ] Handle naming conflicts properly
- [ ] Use mixins for cross-cutting concerns

## Basic Mixin Pattern

```typescript
// mixins/ValidationMixin.ts
import { Vue, Component } from 'vue-property-decorator'

@Component
export default class ValidationMixin extends Vue {
  errors: string[] = []
  touched: { [key: string]: boolean } = {}

  validateRequired(value: any): boolean {
    if (!value) {
      this.errors.push('This field is required')
      return false
    }
    return true
  }

  validateEmail(email: string): boolean {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
    if (!emailRegex.test(email)) {
      this.errors.push('Invalid email format')
      return false
    }
    return true
  }

  validateMinLength(value: string, min: number): boolean {
    if (value.length < min) {
      this.errors.push(`Must be at least ${min} characters`)
      return false
    }
    return true
  }

  clearErrors(): void {
    this.errors = []
    this.touched = {}
  }

  hasErrors(): boolean {
    return this.errors.length > 0
  }
}
```

## Using Mixins in Components

```vue
<script lang="ts">
import { Component, Mixins } from 'vue-property-decorator'
import ValidationMixin from '@/mixins/ValidationMixin'

@Component
export default class MyForm extends Mixins(ValidationMixin) {
  email: string = ''
  password: string = ''

  submitForm(): void {
    this.clearErrors()

    if (!this.validateRequired(this.email)) {
      return
    }

    if (!this.validateEmail(this.email)) {
      return
    }

    if (!this.validateRequired(this.password)) {
      return
    }

    if (!this.validateMinLength(this.password, 6)) {
      return
    }

    // Submit form
    console.log('Form submitted')
  }
}
</script>
```

## Multiple Mixins

```typescript
// mixins/ApiMixin.ts
import { Vue, Component } from 'vue-property-decorator'

@Component
export default class ApiMixin extends Vue {
  loading: boolean = false
  error: string | null = null

  async fetchWithLoading(url: string): Promise<any> {
    this.loading = true
    this.error = null

    try {
      const response = await fetch(url)
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`)
      }
      return await response.json()
    } catch (err) {
      this.error = err.message
      throw err
    } finally {
      this.loading = false
    }
  }

  async postWithLoading(url: string, data: any): Promise<any> {
    this.loading = true
    this.error = null

    try {
      const response = await fetch(url, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data)
      })
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`)
      }
      return await response.json()
    } catch (err) {
      this.error = err.message
      throw err
    } finally {
      this.loading = false
    }
  }
}
```

```typescript
// mixins/AuthMixin.ts
import { Vue, Component } from 'vue-property-decorator'

@Component
export default class AuthMixin extends Vue {
  token: string | null = localStorage.getItem('token')
  user: any = null

  get isAuthenticated(): boolean {
    return !!this.token
  }

  setToken(token: string): void {
    this.token = token
    localStorage.setItem('token', token)
  }

  clearToken(): void {
    this.token = null
    localStorage.removeItem('token')
  }

  logout(): void {
    this.clearToken()
    this.user = null
    this.$router.push('/login')
  }
}
```

```vue
<script lang="ts">
import { Component, Mixins } from 'vue-property-decorator'
import ValidationMixin from '@/mixins/ValidationMixin'
import ApiMixin from '@/mixins/ApiMixin'
import AuthMixin from '@/mixins/AuthMixin'

// Combine multiple mixins
@Component
export default class LoginForm extends Mixins(
  ValidationMixin,
  ApiMixin,
  AuthMixin
) {
  email: string = ''
  password: string = ''

  async submitLogin(): Promise<void> {
    this.clearErrors()

    if (!this.validateRequired(this.email) || !this.validateEmail(this.email)) {
      return
    }

    if (!this.validateRequired(this.password)) {
      return
    }

    try {
      const response = await this.postWithLoading('/api/login', {
        email: this.email,
        password: this.password
      })

      this.setToken(response.token)
      this.user = response.user
      this.$router.push('/dashboard')
    } catch (err) {
      console.error('Login failed:', err)
    }
  }
}
</script>
```

## Pagination Mixin

```typescript
// mixins/PaginationMixin.ts
import { Vue, Component } from 'vue-property-decorator'

@Component
export default class PaginationMixin extends Vue {
  currentPage: number = 1
  pageSize: number = 20
  totalItems: number = 0

  get totalPages(): number {
    return Math.ceil(this.totalItems / this.pageSize)
  }

  get hasNextPage(): boolean {
    return this.currentPage < this.totalPages
  }

  get hasPrevPage(): boolean {
    return this.currentPage > 1
  }

  get offset(): number {
    return (this.currentPage - 1) * this.pageSize
  }

  nextPage(): void {
    if (this.hasNextPage) {
      this.currentPage++
      this.onPageChange()
    }
  }

  prevPage(): void {
    if (this.hasPrevPage) {
      this.currentPage--
      this.onPageChange()
    }
  }

  goToPage(page: number): void {
    if (page >= 1 && page <= this.totalPages) {
      this.currentPage = page
      this.onPageChange()
    }
  }

  resetPagination(): void {
    this.currentPage = 1
    this.onPageChange()
  }

  onPageChange(): void {
    // Override in component
    this.$emit('page-changed', {
      page: this.currentPage,
      pageSize: this.pageSize,
      offset: this.offset
    })
  }
}
```

## Resize Handler Mixin

```typescript
// mixins/ResizeMixin.ts
import { Vue, Component } from 'vue-property-decorator'

@Component
export default class ResizeMixin extends Vue {
  windowWidth: number = window.innerWidth
  windowHeight: number = window.innerHeight
  resizeHandler: (() => void) | null = null

  get isMobile(): boolean {
    return this.windowWidth < 768
  }

  get isTablet(): boolean {
    return this.windowWidth >= 768 && this.windowWidth < 1024
  }

  get isDesktop(): boolean {
    return this.windowWidth >= 1024
  }

  mounted(): void {
    this.resizeHandler = () => {
      this.windowWidth = window.innerWidth
      this.windowHeight = window.innerHeight
      this.onResize()
    }
    window.addEventListener('resize', this.resizeHandler)
  }

  beforeDestroy(): void {
    if (this.resizeHandler) {
      window.removeEventListener('resize', this.resizeHandler)
    }
  }

  onResize(): void {
    // Override in component
    this.$emit('resize', {
      width: this.windowWidth,
      height: this.windowHeight
    })
  }
}
```

## Scroll Handler Mixin

```typescript
// mixins/ScrollMixin.ts
import { Vue, Component } from 'vue-property-decorator'

@Component
export default class ScrollMixin extends Vue {
  scrollY: number = 0
  scrollX: number = 0
  isScrollingDown: boolean = false
  lastScrollY: number = 0
  scrollHandler: (() => void) | null = null

  get isScrolledTop(): boolean {
    return this.scrollY === 0
  }

  get isScrolledBottom(): boolean {
    return (
      window.innerHeight + this.scrollY >=
      document.body.offsetHeight
    )
  }

  mounted(): void {
    this.scrollHandler = () => {
      this.scrollX = window.scrollX
      this.scrollY = window.scrollY
      this.isScrollingDown = this.scrollY > this.lastScrollY
      this.lastScrollY = this.scrollY
      this.onScroll()
    }
    window.addEventListener('scroll', this.scrollHandler, { passive: true })
  }

  beforeDestroy(): void {
    if (this.scrollHandler) {
      window.removeEventListener('scroll', this.scrollHandler)
    }
  }

  onScroll(): void {
    // Override in component
    this.$emit('scroll', {
      x: this.scrollX,
      y: this.scrollY,
      isDown: this.isScrollingDown
    })
  }

  scrollToTop(smooth = true): void {
    window.scrollTo({
      top: 0,
      behavior: smooth ? 'smooth' : 'auto'
    })
  }
}
```

## Form State Mixin

```typescript
// mixins/FormStateMixin.ts
import { Vue, Component } from 'vue-property-decorator'

@Component
export default class FormStateMixin extends Vue {
  isSubmitting: boolean = false
  isDirty: boolean = false
  isValid: boolean = false
  serverErrors: { [key: string]: string[] } = {}

  setSubmitting(submitting: boolean): void {
    this.isSubmitting = submitting
  }

  setDirty(dirty: boolean): void {
    this.isDirty = dirty
  }

  setValid(valid: boolean): void {
    this.isValid = valid
  }

  setServerErrors(errors: { [key: string]: string[] }): void {
    this.serverErrors = errors
  }

  clearServerErrors(): void {
    this.serverErrors = {}
  }

  hasServerError(field: string): boolean {
    return !!this.serverErrors[field]?.length
  }

  getServerErrors(field: string): string[] {
    return this.serverErrors[field] || []
  }

  getFirstServerError(field: string): string | null {
    const errors = this.getServerErrors(field)
    return errors.length > 0 ? errors[0] : null
  }

  resetFormState(): void {
    this.isSubmitting = false
    this.isDirty = false
    this.isValid = false
    this.serverErrors = {}
  }
}
```

## Modal/Dialog Mixin

```typescript
// mixins/ModalMixin.ts
import { Vue, Component } from 'vue-property-decorator'

@Component
export default class ModalMixin extends Vue {
  showModal: boolean = false
  modalLoading: boolean = false
  modalError: string | null = null

  openModal(): void {
    this.showModal = true
    this.modalError = null
  }

  closeModal(): void {
    this.showModal = false
    this.modalError = null
  }

  setModalLoading(loading: boolean): void {
    this.modalLoading = loading
  }

  setModalError(error: string): void {
    this.modalError = error
  }

  clearModalError(): void {
    this.modalError = null
  }

  handleModalEscape(event: KeyboardEvent): void {
    if (event.key === 'Escape' && this.showModal) {
      this.closeModal()
    }
  }
}
```

## Watcher Mixin

```typescript
// mixins/WatcherMixin.ts
import { Vue, Component, Watch } from 'vue-property-decorator'

@Component
export default class WatcherMixin extends Vue {
  // Override these in your component
  watchedValues: { [key: string]: any } = {}

  @Watch('watchedValues', { deep: true, immediate: true })
  onWatchedValuesChanged(
    newVal: any,
    oldVal: any
  ): void {
    // Emit event for component to handle
    this.$emit('watched-change', newVal, oldVal)
  }
}
```

## Global Mixin Registration

```typescript
// main.ts
import Vue from 'vue'
import App from './App.vue'
import GlobalMixin from './mixins/GlobalMixin'

// Register mixin globally
Vue.mixin(GlobalMixin)

new Vue({
  render: (h) => h(App)
}).$mount('#app')
```

```typescript
// mixins/GlobalMixin.ts
import { Vue, Component } from 'vue-property-decorator'

@Component
export default class GlobalMixin extends Vue {
  // Global properties available in all components
  get apiBaseUrl(): string {
    return process.env.VUE_APP_API_URL || ''
  }

  get isDevelopment(): boolean {
    return process.env.NODE_ENV === 'development'
  }

  // Global methods
  formatDate(date: Date | string): string {
    return new Date(date).toLocaleDateString()
  }

  formatCurrency(amount: number): string {
    return new Intl.NumberFormat('en-US', {
      style: 'currency',
      currency: 'USD'
    }).format(amount)
  }

  // Global error handler
  handleError(error: Error): void {
    console.error('Error:', error)
    this.$emit('global-error', error)
  }
}
```

## Mixin Best Practices

```typescript
// ✅ DO: Keep mixins focused on single concern
@Component
export default class ValidationMixin extends Vue {
  // Only validation logic
}

// ✅ DO: Use descriptive names
@Component
export default class UserPaginationMixin extends Vue {
  // Clear purpose
}

// ❌ DON'T: Mix unrelated concerns
@Component
export default class KitchenSinkMixin extends Vue {
  // Validation + API + Auth + Pagination (too much!)
}

// ✅ DO: Document mixin requirements
/**
 * Requires component to have:
 * - email: string property
 * - password: string property
 * - submitForm() method
 */
@Component
export default class LoginValidationMixin extends Vue {
  validateLoginForm(): boolean {
    // Implementation
  }
}
```

## Reference

- [vue-class-component Mixins](https://class-component.vuejs.org/guide/extend-and-mixins.html)
- [Vue 2 Mixins Documentation](https://v2.vuejs.org/v2/guide/mixins.html)
