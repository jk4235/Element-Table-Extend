<template>
  <div id="app">
    <DefaultTable
      :search="true"
      search-mode="server"
      :columns="columns"
      :get-data="getData">
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
import axios from 'axios'

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
          address: '上海市普陀区金沙江路 1518 弄',
          index: 1
        },
        {
          date: '2016-05-04',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1517 弄',
          index: 2
        },
        {
          date: '2016-05-01',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1519 弄',
          index: 3
        },
        {
          date: '2016-05-03',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1516 弄',
          index: 4
        },
        {
          date: '2016-05-02',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄',
          index: 5
        },
        {
          date: '2016-05-04',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1517 弄',
          index: 6
        },
        {
          date: '2016-05-01',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1519 弄',
          index: 7
        },
        {
          date: '2016-05-03',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1516 弄',
          index: 8
        },
        {
          date: '2016-05-02',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄',
          index: 9
        },
        {
          date: '2016-05-04',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1517 弄',
          index: 10
        },
        {
          date: '2016-05-01',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1519 弄',
          index: 11
        },
        {
          date: '2016-05-03',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1516 弄',
          index: 12
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
          field: 'index',
          title: '手机号',
          sortable: true,
          align: 'center'
        }
      ]
    }
  },
  methods: {
    getData (queryParam) {
      const index = queryParam.offset / queryParam.pageSize + 1
      return new Promise((resolve, reject) => {
        axios.get(`https://www.easy-mock.com/mock/5b5550199c2bd4581c709e00/mock/tableData?pageIndex=${index}`).then((response) => {
          const data = {
            total: response.data.total,
            rows: response.data.data
          }
          resolve(data)
        }).catch(e => {
          reject(e)
        })
      })
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
