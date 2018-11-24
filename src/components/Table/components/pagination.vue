<template>
  <el-pagination
    :current-page="currentPage"
    :page-sizes="[10, 20, 50]"
    :page-size="queryParams.pageSize"
    :total="total"
    :layout="layout"
    @size-change="handleSizeChange"
    @current-change="handleCurrentChange"></el-pagination>
</template>

<script>
export default {
  name: 'Index',
  props: {
    fetchData: {
      type: Function,
      default: null,
      required: true
    },
    total: {
      type: Number,
      default: 0
    },
    queryParams: {
      type: Object,
      default () {
        return {
          offset: 0,
          pageSize: 10,
          sort: 'CreatedTime',
          sortOrder: 'asc',
          search: ''
        }
      }
    },
    currentPage: {
      type: Number,
      default: 1
    },
    onSizeChange: {
      type: Function,
      default () {
        return this.handleSizeChange
      }
    },
    onCurrentChange: {
      type: Function,
      default () {
        return this.handleCurrentChange
      }
    },
    layout: {
      type: String,
      default: 'total, sizes, prev, pager, next, jumper'
    }
  },
  methods: {
    handleSizeChange (size) {
      this.queryParams.pageSize = size
      this.fetchData()
    },
    handleCurrentChange (page) {
      this.$emit('current-change', page)
      this.queryParams.offset = this.queryParams.pageSize * (page - 1)
      this.fetchData()
    }
  }
}
</script>

<style scoped>

</style>
