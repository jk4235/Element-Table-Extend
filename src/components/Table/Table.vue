<template>
  <div class="app-container">
    <el-row class="toolbar">
      <slot name="toolbar"/>
      <el-input
        v-if="search"
        v-model="queryParams.search"
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
      v-bind="originProps"
      element-loading-text="Loading"
      @row-click="$emit('row-click', $event)"
      @filter-change="handleFilterChange"
      @current-change="handleCurrentChange"
      @select-all="handleSelectAll"
      @sort-change="handleSortChange"
      @select="handleSelect">
      <my-column
        v-for="(tableColumn, index) in columns" :table-column="tableColumn" :key="index">
      </my-column>
      <slot name="default"/>
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
import myColumn from './components/MyColumn'

export default {
  name: 'DefaultTable',
  components: {
    pagination,
    transparent,
    myColumn
  },
  props: {
    columns: {
      type: Array,
      default () {
        return []
      }
    },
    originProps: {
      type: Object,
      default () {
        return {
          highlightCurrentRow: true,
          fit: true,
          border: true
        }
      }
    },
    getData: {
      type: [Function, Array],
      required: true
    },
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
    searchMode: { type: String, default: 'client' },
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
    },
    draggable: {
      type: Boolean,
      default: false
    },
    onDragEnd: {
      type: Function
    }
  },
  data () {
    return {
      tableData: [],
      listLoading: true,
      total: 0,
      isSearchTextChange: false,
      clientSearch: this.searchMode === 'client',
      rawData: [],
      currentPage: 1,
      isPagination: this.pagination,
      searchDataBak: [],
      searchFields: [],
      dragSort: null
    }
  },
  computed: {
    viewData () {
      if (this.isPagination && this.clientSearch) {
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
    refresh: {
      handler: function (newVal) {
        if (newVal) {
          this.fetchData()
          this.$emit('update:refresh', false)
        }
      },
      immediate: true
    },
    initSelected () {
      this.initSelect()
    },
    getData () {
      this.currentPage = 1
      this.queryParams.offset = 0
      this.clearSearchData()
      this.fetchData()
    }
  },
  created () {
    this.fetchData()
  },
  mounted () {
    this.initTable()
  },
  methods: {
    initTable () {
      this.initSelect()
      this.searchFields = this.getSearchFields()
      this.setDragSort()
    },
    fetchData () {
      this.listLoading = true
      if (this.clientSearch) {
        this.clientDataHandle()
      } else {
        if (typeof this.getData !== 'function') {
          this.clientDataHandle() // 仅当getData为获取数据的api时才支持远程搜索及分页，否则全部转为本地模式
        } else {
          this.getData(this.queryParams).then((response) => {
            this.tableData = response.rows ? response.rows : response
            this.clientSearch = !response.rows// 如果后端接口返回的数据结构不符合分页的要求，则自动转为本地模式
            if (response.rows && this.isPagination === false) {
              this.isPagination = true
            }
            this.total = response.total ? response.total : this.tableData.length
            this.listLoading = false
            this.rawData = this.tableData
            this.initSelect()
            this.setDragSort()
          }).catch(e => {
            this.listLoading = false
            console.log(e) // for debug
          })
        }
      }
    },
    clientDataHandle () {
      this.clientSearch = true
      this.tableData = this.searchDataBak.length > 0 ? this.searchDataBak : this.getData
      this.total = this.tableData.length
      this.listLoading = false
      this.rawData = this.searchDataBak.length > 0 ? this.rawData : this.tableData
      this.initSelect()
      this.setDragSort()
    },
    setSearchText (text) {
      this.queryParams.search = text
      this.isSearchTextChange = true
    },
    /* 设置进行本地搜索的列，目前是设置了sortable的列 */
    getSearchFields () {
      const searchFieldByColumns = this.columns
        .filter(
          cv => cv.sortable
        ).map(
          cv => cv.field
        )
      const searchFieldBySlot = this.$slots.default ? this.$slots.default
        .filter(
          cv => {
            return cv.componentInstance && cv.componentInstance.sortable !== false
          }
        ).map(
          cv => cv.componentInstance.$options.propsData.prop
        ) : []
      searchFieldByColumns.forEach(function (v) {
        searchFieldBySlot.push(v)
      })
      return searchFieldBySlot
    },
    searchData () {
      if (this.isSearchTextChange) {
        this.tableData = this.rawData
        this.currentPage = 1
        this.queryParams.offset = 0 // 远程搜索时offset清0,防止搜索时index不对
        if (this.clientSearch) {
          if (this.queryParams.search === '') {
            this.total = this.tableData.length
            this.initSelect()
          } else {
            this.searchDataBak = this.rawData
              .filter(item => {
                return this.searchFields
                  .map(
                    field => (item[field] ? item[field].toString() : '')
                  ).filter(
                    cv => cv.match(this.queryParams.search)
                  ).length > 0
              })
            this.tableData = this.searchDataBak
            this.total = this.tableData.length
          }
          this.initSelect()
        } else {
          this.fetchData()
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
    handleFilterChange (filters) {
      this.$emit('filter-change', filters)
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
    },
    handleCurrentChange (currentRow, oldCurrentRow) {
      this.$emit('current-change', currentRow, oldCurrentRow)
    },
    handleSortChange ({ column, prop, order }) {
      if (this.clientSearch) {
        this.tableData.sort((a, b) => {
          if (order === 'ascending') {
            if (a[prop] < b[prop]) {
              return -1
            }
            if (a[prop] > b[prop]) {
              return 1
            }
            return 0
          } else {
            if (a[prop] < b[prop]) {
              return 1
            }
            if (a[prop] > b[prop]) {
              return -1
            }
            return 0
          }
        })
      } else {
        this.queryParams.sort = prop
        this.queryParams.sortOrder = order === 'ascending' ? 'asc' : 'desc'
        this.fetchData()
      }
    },
    setDragSort () {
      this.$nextTick(() => {
        const el = this.$refs.table && this.$refs.table.$el.querySelector('.el-table__body-wrapper > table > tbody')
        if (this.draggable && this.onDragEnd && el) {
            import('sortablejs').then(Sortable => {
              if (this.dragSort) this.dragSort.destroy() // 防止在Dom上重复绑定事件
              this.dragSort = Sortable.create(el, {
                onEnd: evt => this.onDragEnd(evt)
              })
            })
        }
      })
    }
  }
}
</script>

<style scoped>
  .toolbar, .el-table {
    margin-bottom: 10px;
  }
</style>
<style>
  .sortable-ghost{
    opacity: .8;
    color: #fff!important;
    background: #42b983!important;
  }
</style>
