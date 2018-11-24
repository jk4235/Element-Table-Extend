<template>
  <div id="app">
    <DefaultTable
      :search="true"
      :columns="columns"
      :get-data="tableData">
      <el-table-column
        sortable
        prop="date"
        label="日期"
        width="180">
      </el-table-column>
      <el-table-column
        prop="name"
        label="姓名"
        width="180">
      </el-table-column>
      <el-table-column
        prop="address"
        label="地址">
      </el-table-column>
    </DefaultTable>
  </div>
</template>

<script>
import DefaultTable from './components/Table/Table'

export default {
  name: 'App',
  components: {
    DefaultTable
  },
  data () {
    return {
      tableData: [
        {
          date: '2016-05-02',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄'
        }, {
          date: '2016-05-04',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1517 弄'
        }, {
          date: '2016-05-01',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1519 弄'
        }, {
          date: '2016-05-03',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1516 弄'
        }
      ],
      columns: [
        {
          type: 'selection'
        },
        {
          field: 'sex',
          title: '性别',
          align: 'center'
        },
        {
          field: 'deptname',
          title: '部门',
          align: 'center'
        },
        {
          field: 'worktype',
          title: '工种',
          align: 'center'
        },
        {
          field: 'worklevel',
          title: '职位',
          align: 'center'
        },
        {
          field: 'mobile',
          title: '手机号',
          sortable: true,
          align: 'center'
        }
      ]
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
