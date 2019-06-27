<template>
  <el-table-column
    :selectable="tableColumn.selectable"
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
    <template v-if="tableColumn.children">
      <my-column v-for="(column, index) in tableColumn.children" :table-column="column" :key="index"></my-column>
    </template>
    <transparent
      slot-scope="{ row, $index, column }"
      :slot="tableColumn.template ? 'default' : 'undefined'"
      :v-node="tableColumn.template(row, $index, column)"/>
  </el-table-column>
</template>

<script>
import transparent from './transparent'
export default {
  name: 'MyColumn',
  components: {
    transparent
  },
  props: {
    tableColumn: {
      type: Object,
      default () {
        return {}
      }
    }
  }
}
</script>

<style scoped>

</style>
