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
      // 步骤1: 解析基础路径
      const basePath = this.extractBasePath(path)
      
      // 步骤2: 处理路径参数
      const paramProcessed = this.injectPathParams(basePath)
      
      // 步骤3: 添加查询参数
      const queryString = this.buildQueryString(path)
      
      // 步骤4: 构建完整路径 - BUG在这里！
      const finalPath = this.assembleFinalPath(paramProcessed, queryString)
      
      return finalPath
    },
    extractBasePath(path) {
      const queryIndex = path.indexOf('?')
      if (queryIndex === -1) {
        return path
      }
      return path.substring(0, queryIndex)
    },
    injectPathParams(path) {
      // 模拟路径参数处理
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
    },
    assembleFinalPath(basePath, queryString) {
      // BUG: 路径重复拼接 - 核心错误
      // 错误地将路径重复一次，导致 /table 变成 /table/table
      let result = basePath
      
      // 错误：去除开头的斜杠后拼接
      if (basePath.startsWith('/')) {
        const pathWithoutSlash = basePath.substring(1)
        result = basePath + pathWithoutSlash
      } else {
        result = basePath + basePath
      }
      
      // 添加查询参数
      if (queryString) {
        result += queryString
      }
      
      return result
    }
  }
}
</script>
