# element-table-extend

> 基于element-table的扩展，组合了element-table和element-pagination
> 仿照bootstrap-table的用法，使用column数组来定义列 也支持原生写法 也可以两者混合写
> 支持排序和分页
# Usage

``````javascript
import DefaultTable from './components/Table/Table'
``````

```html
<DefaultTable
  :search="true"
  :columns="columns"
  :get-data="tableData">
</DefaultTable>
```

```javascript
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
          field: 'index',
          title: '序号',
          sortable: true
        },
        {
          field: 'date',
          title: '日期'
        },
        {
          field: 'name',
          title: '姓名'
        },
        {
          field: 'address',
          title: '地址'
        },
        {
          title: '操作',
            template: (row) => {
                const h = this.$createElement
                return h(
                  'el-button',
                  {
                    on: {
                          click: () => {}
                        }
                  },
                  ['移除']
                )
            }
        }
      ]
    }
  }
```

# props

| prop              | type                          | default                                                      | description                                                  |
| ----------------- | ----------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| columns           | Array<object>                 | ```[]```                                                     | 用于渲染表格列                                               |
| originProps       | Object                        | ```{highlightCurrentRow: true,fit: true,border: true }```    | 所有element-table支持的属性                                  |
| getData           | Function<Promise>  \| Array   | ```[]```                                                     | 必填,表格的数据来源,可以是一个数组或者是一个Promise方法,当为Function时会注入queryParams作为第一个参数,同时需将search-mode设为'server',且promise要返回结构为```{rows<Array>, total<number>}```的对象 |
| refresh           | Boolean                       | false                                                        | 支持.sync操作符, 用于主动让表格刷新下数据                    |
| queryParams       | Object                        | ```{offset: 0, pageSize: 10, sort: 'CreatedTime', sortOrder: 'asc', search: '' }``` | 用于分页,或者是向远端请求数据时的参数offset pageSize search 属性需保留 |
| pagination        | Boolean                       | true                                                         | 是否开启分页                                                 |
| search            | Boolean                       | false                                                        | 开启后表格右上方会多一个搜索框,用于筛选表格数据              |
| searchMode        | String ('client '\| 'server') | 'client'                                                     | 决定是本地过滤数据,还是远程搜索数据                          |
| initSelected      | Array                         | ```[]```                                                     | 初始选中的数据                                               |
| initSelectedField | String                        | 'id'                                                         | 用于区分数据的唯一字段                                       |
| paginationLayout  | String                        | 'total, sizes, prev, pager, next, jumper'                    | 控制分页样式                                                 |
| draggable         | Boolean                       | false                                                        | 控制能否拖动数据                                             |
| onDragEnd         | Function                      | \                                                            | 拖动完成后的回调函数,会传入evt作为参数                       |

# columns

包含一些选项的数组,用于渲染表格的每一列

| field         | type                         | description                              |
| ------------- | ---------------------------- | ---------------------------------------- |
| field         | String                       | 该列对应数据字段                         |
| title         | String                       | 该列对应表头                             |
| formatter     | Function                     | 格式化表格数据的方法                     |
| type          | String                       | 表格类型,当设置为'selection'时开启多选框 |
| selectable    | Boolean                      | 控制checkbox disabled状态                |
| filters       | Array[{ text, value }]       | 与el-table-column相同                    |
| filter-method | Function(value, row, column) | 与el-table-column相同                    |
| align         | String                       | 与el-table-column相同,默认'center'       |
| width         | String                       | 对应el-table-column的min-width           |
| template      | VNode                        | 自定义表格展示内容,参见vue 渲染函数      |
| sortable      | Boolean                      | 与el-table-column相同                    |
| children      | Array<columns>               | 由columns组成的数组,用于渲染多级表头     |



# event

| event      | param                       | description                                  |
| ---------- | --------------------------- | -------------------------------------------- |
| select     | selection, row              | 当用户手动勾选数据行的 Checkbox 时触发的事件 |
| select-all | ```{selection, viewData}``` | 当用户手动勾选全选 Checkbox 时触发的事件     |




## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report

# run unit tests
npm run unit

# run e2e tests
npm run e2e

# run all tests
npm test
```

For a detailed explanation on how things work, check out the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).
