<template>
  <div class="app-container">
    <el-input v-model="filterText" placeholder="Filter keyword" style="margin-bottom:30px;" />

    <el-tree
      ref="tree2"
      :data="processedData"
      :props="treeProps"
      :filter-node-method="filterNode"
      class="filter-tree"
      default-expand-all
    />

  </div>
</template>

<script>
export default {
  data() {
    return {
      filterText: '',
      // 原始树形数据
      sourceData: [{
        id: 1,
        label: 'Level one 1',
        children: [{
          id: 4,
          label: 'Level two 1-1',
          children: [{
            id: 9,
            label: 'Level three 1-1-1'
          }, {
            id: 10,
            label: 'Level three 1-1-2'
          }]
        }]
      }, {
        id: 2,
        label: 'Level one 2',
        children: [{
          id: 5,
          label: 'Level two 2-1'
        }, {
          id: 6,
          label: 'Level two 2-2'
        }]
      }, {
        id: 3,
        label: 'Level one 3',
        children: [{
          id: 7,
          label: 'Level two 3-1'
        }, {
          id: 8,
          label: 'Level two 3-2'
        }]
      }],
      // BUG: 处理后的数据
      processedData: [],
      // BUG: 使用了错误的属性名
      treeProps: {
        children: 'children',
        label: 'label'
      }
    }
  },
  watch: {
    filterText(val) {
      this.$refs.tree2.filter(val)
    }
  },
  created() {
    // BUG: 处理数据
    this.processTreeData()
  },
  methods: {
    processTreeData() {
      const cleaned = this.cleanData(this.sourceData)
      this.processedData = this.transformData(cleaned)
    },
    cleanData(data) {
      return data.filter(item => item && item.id !== undefined)
    },
    transformData(data) {
      return data.map(item => {
        return {
          id: item.id,
          label: item.label,
          children: item.children ? this.transformData(item.children) : undefined
        }
      })
    },
    filterNode(value, data) {
      if (!value) return true
      return data.label.indexOf(value) !== -1
    }
  }
}
</script>
