<template>
  <el-breadcrumb class="app-breadcrumb" separator="/">
    <transition-group name="breadcrumb">
      <el-breadcrumb-item v-for="(item,index) in processedLevelList" :key="item.path">
        <span v-if="item.redirect==='noRedirect'||index==processedLevelList.length-1" class="no-redirect">{{ item.meta.title }}</span>
        <a v-else @click.prevent="handleLink(item)">{{ item.meta.title }}</a>
      </el-breadcrumb-item>
    </transition-group>
  </el-breadcrumb>
</template>

<script>
import pathToRegexp from 'path-to-regexp'

export default {
  data() {
    return {
      levelList: null,
      processedLevelList: []
    }
  },
  watch: {
    $route() {
      this.processBreadcrumb()
    }
  },
  created() {
    this.processBreadcrumb()
  },
  methods: {
    processBreadcrumb() {
      // 获取原始匹配的路由
      const matched = this.getMatchedRoutes()

      // 处理路由数据
      const processed = this.processRoutes(matched)

      // 对面包屑进行排序 - BUG在这里！
      this.processedLevelList = this.sortBreadcrumbItems(processed)
    },
    getMatchedRoutes() {
      // 获取匹配的路由
      let matched = this.$route.matched.filter(item => item.meta && item.meta.title)
      const first = matched[0]

      if (!this.isDashboard(first)) {
        matched = [{ path: '/dashboard', meta: { title: 'Dashboard' }}].concat(matched)
      }

      return matched.filter(item => item.meta && item.meta.title && item.meta.breadcrumb !== false)
    },
    processRoutes(routes) {
      // 添加额外信息到路由
      return routes.map((route, index) => ({
        ...route,
        originalIndex: index,
        depth: route.path.split('/').filter(s => s).length
      }))
    },
    sortBreadcrumbItems(routes) {
      // 修复：保持面包屑原有顺序，按照路由匹配顺序显示
      // 面包屑应该按照路由层级从浅到深显示
      return routes
    },
    isDashboard(route) {
      const name = route && route.name
      if (!name) {
        return false
      }
      return name.trim().toLocaleLowerCase() === 'Dashboard'.toLocaleLowerCase()
    },
    pathCompile(path) {
      const { params } = this.$route
      var toPath = pathToRegexp.compile(path)
      return toPath(params)
    },
    handleLink(item) {
      const { redirect, path } = item
      if (redirect) {
        this.$router.push(redirect)
        return
      }
      this.$router.push(this.pathCompile(path))
    }
  }
}
</script>

<style lang="scss" scoped>
.app-breadcrumb.el-breadcrumb {
  display: inline-block;
  font-size: 14px;
  line-height: 50px;
  margin-left: 8px;

  .no-redirect {
    color: #97a8be;
    cursor: text;
  }
}
</style>
