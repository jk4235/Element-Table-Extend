<template>
  <div class="app-container">
    <el-row class="toolbar">
      <slot name="toolbar"/>
      <el-input
        v-if="search"
        :value="queryParams.search"
        placeholder="搜索"
        class="el-input"
        style="float: right; min-width: 20%; max-width: 15vh"
        suffix-icon="el-icon-search"
        @change="setSearchText"
        @blur="searchData"></el-input>
    </el-row>
    <el-table
      v-loading="listLoading"
      ref="table"
      :data="viewData"
      :border="border"
      class="el-table"
      fit
      :height="height ? height : null"
      element-loading-text="Loading"
      highlight-current-row
      @select-all="handleSelectAll"
      @select="handleSelect">
      <el-table-column
        v-for="tableColumn in columns"
        :label="tableColumn.title"
        :key="tableColumn.field"
        :prop="tableColumn.field"
        :sortable="tableColumn.sortable"
        :formatter="tableColumn.formatter"
        :type="tableColumn.type"
        :align="tableColumn.align">
        <transparent
          slot-scope="{ row, $index, column }"
          :slot="tableColumn.template ? 'default' : 'undefined'"
          :v-node="tableColumn.template(row, $index, column)"/>
      </el-table-column>
    </el-table>
    <pagination
      v-if="isPagination"
      :query-params="queryParams"
      :current-page="currentPage"
      :fetch-data="fetchData"
      :total="total"
      :layout="paginationLayout"
      class="pagination"
      @current-change="currentPage = $event"></pagination>
  </div>
</template>

<script>
import pagination from './components/pagination'
import transparent from './components/transparent'

export default {
  name: 'DefaultTable',
  components: {
    pagination,
    transparent
  },
  props: {
    columns: {
      type: Array,
      default () {
        return []
      }
    },
    getData: {
      type: [Function, Array],
      required: true
    },
    height: { type: String },
    refresh: {
      type: Boolean,
      default: false
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
    pagination: { type: Boolean, default: true },
    search: { type: Boolean, default: false },
    border: { type: Boolean, default: true },
    initSelected: {
      type: Array,
      default () {
        return []
      }
    },
    initSelectedField: { type: String, default: 'id' },
    paginationLayout: {
      type: String,
      default: 'total, sizes, prev, pager, next, jumper'
    }
  },
  data () {
    return {
      tableData: [],
      listLoading: true,
      total: 0,
      isSearchTextChange: false,
      clientSearch: false,
      rawData: [],
      currentPage: 1,
      isPagination: this.pagination,
      searchDataBak: []
    }
  },
  computed: {
    viewData () {
      if (this.isPagination) {
        const { pageSize } = this.queryParams
        return this.tableData.slice((this.currentPage - 1) * pageSize, this.currentPage * pageSize)
      }
      return this.tableData
    },
    hasSelection () {
      return this.columns.filter(cv => cv.type === 'selection').length > 0
    }
  },
  watch: {
    refresh () {
      if (this.refresh) {
        this.fetchData()
        this.$emit('refreshed')
      }
    },
    initSelected () {
      this.initSelect()
    },
    getData () {
      this.clearSearchData()
      this.fetchData()
    }
  },
  created () {
    this.fetchData()
  },
  mounted () {
    this.initSelect()
  },
  methods: {
    fetchData () {
      this.listLoading = true
      if (!this.clientSearch) {
        if (typeof this.getData === 'function') {
          this.getData(this.queryParams).then((response) => {
            this.tableData = response.rows ? response.rows : response
            this.clientSearch = !response.rows
            if (response.rows && this.isPagination === false) {
              this.isPagination = true
            }
            this.total = response.total ? response.total : this.tableData.length
            this.listLoading = false
            this.rawData = this.tableData
            this.initSelect()
          })
        } else {
          this.clientDataHandle()
        }
      } else {
        this.clientDataHandle()
      }
    },
    clientDataHandle () {
      this.clientSearch = true
      this.tableData = this.searchDataBak.length > 0 ? this.searchDataBak : this.getData
      this.total = this.tableData.length
      this.listLoading = false
      this.rawData = this.searchDataBak.length > 0 ? this.rawData : this.tableData
      this.initSelect()
    },
    setSearchText (text) {
      this.queryParams.search = text
      this.isSearchTextChange = true
    },
    searchData () {
      if (this.isSearchTextChange) {
        this.tableData = this.rawData
        this.currentPage = 1
        if (!this.clientSearch) {
          this.fetchData()
        } else {
          if (this.queryParams.search === '') {
            this.total = this.tableData.length
            this.initSelect()
          } else {
            this.searchDataBak = this.rawData.filter((item) => {
              const searchFields = this.columns.filter(cv => cv.sortable).map(cv => cv.field)
              return searchFields.map(field => (item[field] ? item[field].toString() : ''))
                .filter(cv => cv.match(this.queryParams.search)).length > 0
            })
            this.tableData = this.searchDataBak
            this.total = this.tableData.length
          }
          this.initSelect()
        }
      }
      this.isSearchTextChange = false
    },
    clearSearchData () {
      this.searchDataBak = []
      this.queryParams.search = ''
    },
    handleSelect (selection, row) {
      this.$emit('select', selection, row)
    },
    handleSelectAll (selection) {
      this.$emit('select-all', { selection, viewData: this.viewData })
    },
    setSelect (rows) {
      rows.forEach((cv) => {
        this.$nextTick(() => { this.$refs.table.toggleRowSelection(cv, true) })
      })
    },
    initSelect () {
      if (!this.hasSelection) return false
      if (this.$refs.table) { this.$refs.table.clearSelection() }
      const rows = this.viewData.filter(cv =>
        this.initSelected.filter(item =>
          item[this.initSelectedField] === cv[this.initSelectedField]).length > 0)
      this.setSelect(rows)
      return true
    }
  }
}
</script>

<style scoped>
.toolbar, .el-table {
  margin-bottom: 10px;
}
</style>
