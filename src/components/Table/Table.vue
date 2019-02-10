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
      v-bind="originProps"
      class="el-table"
      element-loading-text="Loading"
      @filter-change="handleFilterChange"
      @current-change="handleCurrentChange"
      @select-all="handleSelectAll"
      @select="handleSelect">
      <el-table-column
        v-for="tableColumn in columns"
        :filters="tableColumn.filters"
        :filter-method="tableColumn.filterMethod"
        :label="tableColumn.title"
        :key="tableColumn.field"
        :prop="tableColumn.field"
        :sortable="tableColumn.sortable"
        :formatter="tableColumn.formatter"
        :type="tableColumn.type"
        :align="tableColumn.align || 'center'"
        :min-width="tableColumn.width">
        <transparent
          slot-scope="{ row, $index, column }"
          :slot="tableColumn.template ? 'default' : 'undefined'"
          :v-node="tableColumn.template(row, $index, column)"/>
      </el-table-column>
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
  export default {
    name: 'DefaultTable',
    components: {
      pagination,
      transparent
    },
    props: {
      columns: {
        type: Array,
        default() {
          return []
        }
      },
      originProps: {
        type: Object,
        default() {
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
        default() {
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
      searchMode: { type: String, default: 'server' },
      initSelected: {
        type: Array,
        default() {
          return []
        }
      },
      initSelectedField: { type: String, default: 'id' },
      paginationLayout: {
        type: String,
        default: 'total, sizes, prev, pager, next, jumper'
      }
    },
    data() {
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
        searchFields: []
      }
    },
    computed: {
      viewData() {
        if (this.isPagination) {
          const { pageSize } = this.queryParams
          return this.tableData.slice((this.currentPage - 1) * pageSize, this.currentPage * pageSize)
        }
        return this.tableData
      },
      hasSelection() {
        return this.columns.filter(cv => cv.type === 'selection').length > 0
      }
    },
    watch: {
      refresh() {
        if (this.refresh) {
          this.fetchData()
          this.$emit('refreshed')
        }
      },
      initSelected() {
        this.initSelect()
      },
      getData() {
        this.currentPage = 1
        this.clearSearchData()
        this.fetchData()
      }
    },
    created() {
      this.fetchData()
    },
    mounted() {
      this.initTable()
    },
    methods: {
      initTable() {
        this.initSelect()
        this.searchFields = this.getSearchFields()
      },
      fetchData() {
        this.listLoading = true
        if (this.clientSearch) {
          this.clientDataHandle()
        } else {
          if (typeof this.getData !== 'function') {
            this.clientDataHandle() // 仅当getData为获取数据的api时才支持远程搜索及分页，否则全部转为本地模式
          } else {
            this.getData(this.queryParams).then((response) => {
              this.tableData = response.rows ? response.rows : response
              this.clientSearch = !response.rows// 如果后端接口返回的数据结构符合分页的要求，则自动转为本地模式
              if (response.rows && this.isPagination === false) {
                this.isPagination = true
              }
              this.total = response.total ? response.total : this.tableData.length
              this.listLoading = false
              this.rawData = this.tableData
              this.initSelect()
            })
          }
        }
      },
      clientDataHandle() {
        this.clientSearch = true
        this.tableData = this.searchDataBak.length > 0 ? this.searchDataBak : this.getData
        this.total = this.tableData.length
        this.listLoading = false
        this.rawData = this.searchDataBak.length > 0 ? this.rawData : this.tableData
        this.initSelect()
      },
      setSearchText(text) {
        this.queryParams.search = text
        this.isSearchTextChange = true
      },
      /* 设置进行本地搜索的列，目前是设置了sortable的列 */
      getSearchFields() {
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
        searchFieldByColumns.forEach(function(v) {
          searchFieldBySlot.push(v)
        })
        return searchFieldBySlot
      },
      searchData() {
        if (this.isSearchTextChange) {
          this.tableData = this.rawData
          this.currentPage = 1
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
      clearSearchData() {
        this.searchDataBak = []
        this.queryParams.search = ''
      },
      handleSelect(selection, row) {
        this.$emit('select', selection, row)
      },
      handleSelectAll(selection) {
        this.$emit('select-all', { selection, viewData: this.viewData })
      },
      handleFilterChange(filters) {
        this.$emit('filter-change', filters)
      },
      setSelect(rows) {
        rows.forEach((cv) => {
          this.$nextTick(() => { this.$refs.table.toggleRowSelection(cv, true) })
        })
      },
      initSelect() {
        if (!this.hasSelection) return false
        if (this.$refs.table) { this.$refs.table.clearSelection() }
        const rows = this.viewData.filter(cv =>
          this.initSelected.filter(item =>
            item[this.initSelectedField] === cv[this.initSelectedField]).length > 0)
        this.setSelect(rows)
        return true
      },
      handleCurrentChange(currentRow, oldCurrentRow) {
        this.$emit('current-change', currentRow, oldCurrentRow)
      }
    }
  }
</script>

<style scoped>
  .toolbar, .el-table {
    margin-bottom: 10px;
  }
</style>
