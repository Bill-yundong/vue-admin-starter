<template>
  <component :is="type" v-bind="linkProps(to)">
    <slot />
  </component>
</template>

<script>
import { isExternal } from '@/utils/validate'

export default {
  props: {
    to: {
      type: String,
      required: true
    }
  },
  computed: {
    isExternal() {
      return isExternal(this.to)
    },
    type() {
      if (this.isExternal) {
        return 'a'
      }
      return 'router-link'
    }
  },
  methods: {
    linkProps(to) {
      if (this.isExternal) {
        return {
          href: to,
          target: '_blank',
          rel: 'noopener'
        }
      }
      // BUG: 路径处理逻辑错误 - 重复拼接路径
      return {
        to: this.processRoutePath(to)
      }
    },
    processRoutePath(path) {
      const basePath = this.extractBasePath(path)
      const paramProcessed = this.injectPathParams(basePath)
      const queryString = this.buildQueryString(path)
      return paramProcessed + queryString
    },
    extractBasePath(path) {
      const queryIndex = path.indexOf('?')
      if (queryIndex === -1) {
        return path
      }
      return path.substring(0, queryIndex)
    },
    injectPathParams(path) {
      return path.replace(/:([^/]+)/g, (match, param) => {
        return this.$route.params[param] || match
      })
    },
    buildQueryString(path) {
      const queryIndex = path.indexOf('?')
      if (queryIndex === -1) {
        return ''
      }
      return path.substring(queryIndex)
    }
  }
}
</script>
