---
title: vue.config.js Basic Setup for Vue 2 Projects
impact: MEDIUM
impactDescription: Proper vue.config.js configuration ensures correct build behavior for Vue 2
type: best-practice
tags:
  - vue2
  - vue-cli
  - vue-config
  - build-configuration
  - webpack
---

# vue.config.js Basic Setup for Vue 2 Projects

Impact: MEDIUM - The `vue.config.js` file configures Vue CLI's internal webpack configuration. Proper setup ensures correct build output, development server behavior, and asset handling.

## Task Checklist

- [ ] Create `vue.config.js` in project root
- [ ] Set `publicPath` for deployment to subdirectories
- [ ] Configure `outputDir` and `assetsDir`
- [ ] Set up dev server proxy for API calls

## Basic Configuration

```javascript
// vue.config.js
module.exports = {
  // Base path for assets (useful for GitHub Pages)
  publicPath: process.env.NODE_ENV === 'production' ? '/my-project/' : '/',

  // Output directory
  outputDir: 'dist',

  // Static assets directory (inside outputDir)
  assetsDir: 'static',

  // Production source map
  productionSourceMap: false,

  // Configure webpack
  configureWebpack: {
    resolve: {
      alias: {
        '@': require('path').resolve(__dirname, 'src')
      }
    }
  },

  // Dev server configuration
  devServer: {
    port: 8080,
    hot: true,
    open: true,
    proxy: {
      '/api': {
        target: 'http://localhost:3000',
        changeOrigin: true,
        pathRewrite: {
          '^/api': ''
        }
      }
    }
  }
}
```

## Advanced Configuration

```javascript
const path = require('path')

module.exports = {
  publicPath: process.env.BASE_URL || '/',

  // Tweak internal webpack configuration
  chainWebpack: config => {
    // Remove prefetch plugins
    config.plugins.delete('prefetch')
    config.plugins.delete('preload')

    // Add alias
    config.resolve.alias
      .set('@', path.resolve(__dirname, 'src'))
      .set('~', path.resolve(__dirname))
      .set('vue$', path.resolve(__dirname, 'node_modules/vue/dist/vue.esm.js'))

    // Bundle analyzer for production
    if (process.env.NODE_ENV === 'production') {
      config
        .plugin('webpack-bundle-analyzer')
        .use(require('webpack-bundle-analyzer').BundleAnalyzerPlugin, [{
          analyzerMode: 'static'
        }])
    }
  },

  // Alternative: configureWebpack (object or function)
  configureWebpack: config => {
    if (process.env.NODE_ENV === 'production') {
      // Production-only config
      config.optimization = {
        ...config.optimization,
        splitChunks: {
          chunks: 'all',
          cacheGroups: {
            vendor: {
              test: /[\\/]node_modules[\\/]/,
              name: 'chunk-vendors',
              priority: 10
            },
            vue: {
              test: /[\\/]node_modules[\\/](vue|vue-router|vuex)[\\/]/,
              name: 'chunk-vue',
              priority: 20
            }
          }
        }
      }
    }
  },

  // CSS configuration
  css: {
    // Extract CSS in production
    extract: process.env.NODE_ENV === 'production',

    // Source map for CSS
    sourceMap: false,

    // Loader options
    loaderOptions: {
      // Less configuration
      less: {
        lessOptions: {
          modifyVars: {
            '@primary-color': '#42b983'
          },
          javascriptEnabled: true
        },
        additionalData: `
          @import "@/styles/variables.less";
        `
      },

      // CSS modules
      css: {
        modules: {
          localIdentName: '[name]__[local]--[hash:base64:5]'
        }
      }
    }
  },

  // Dev server
  devServer: {
    host: '0.0.0.0',
    port: 8080,
    https: false,
    hot: true,
    open: true,
    proxy: {
      '/api': {
        target: 'http://localhost:3000',
        changeOrigin: true,
        pathRewrite: {
          '^/api': ''
        }
      }
    },
    // History API fallback
    historyApiFallback: true
  },

  // Plugin options
  pluginOptions: {
    // Third-party plugins
    i18n: {
      locale: 'en',
      fallbackLocale: 'en',
      localeDir: 'locales',
      enableInSFC: true
    }
  },

  // Lint on save (for development)
  lintOnSave: process.env.NODE_ENV !== 'production',

  // Production source map
  productionSourceMap: false,

  // Parallel compilation
  parallel: require('os').cpus().length > 1
}
```

## Environment-Specific Configuration

```javascript
module.exports = {
  // Different publicPath for environments
  publicPath: process.env.NODE_ENV === 'production'
    ? process.env.VUE_APP_PUBLIC_PATH || '/'
    : '/',

  // Different output directories
  outputDir: process.env.VUE_APP_OUTPUT_DIR || 'dist'
}
```

## Pages Multi-Page Build

```javascript
module.exports = {
  pages: {
    index: {
      entry: 'src/main/index.js',
      template: 'public/index.html',
      filename: 'index.html',
      title: 'Main Page',
      chunks: ['chunk-vendors', 'chunk-common', 'index']
    },
    admin: {
      entry: 'src/main/admin.js',
      template: 'public/admin.html',
      filename: 'admin.html',
      title: 'Admin Panel'
    }
  }
}
```

## Reference

- [Vue CLI Configuration](https://cli.vuejs.org/config/)
