# eTable
基于elementui官方表格的封装
## 使用方式
```vue
<e-table
    height="610"
    data-url="/pudgaController/getList"
    :params="tableParams"
    name="tempTable"
    :current-change="_currentChange"
    align="center"
    highlight-current-row
    @loadData="loadData"
    ref="tableTest"
></e-table>

computed: {
    tableParams() {
        return {
            param1: this.param1,
        };
    },
},
methods: {
    /**
    * 表格选中行切换事件
    * @param currentRow 当前选中行
    * @param oldCurrentRow 旧的选中行
    */
    _currentChange(currentRow, oldCurrentRow) {
        
    },
    // 表格加载成功回调
    loadData(tableData) {
    }
}
```
## 后台返回表格列数据源格式

| 属性名 | 属性类别 | 默认值 | 是否必须 | 备注 | 
|  ----  | ----  | ----  | ----  | ---- |
| id | Integer | - | true | 数据库字段主键 | 
| tableId | String | - | true | 绑定的上述主键的name属性中的值，用于数据库查询时查询对应表格的所有列属性 |
| width | int | 0 | false | 当前列宽 |
| prop | String | - | true | 当前列需要从数据中获取的属性名称(eg：{"age":"18", "sex": "男"}中的age和sex) |
| show | boolean | false | false | 当前列是否需要显示 |
| lock | boolean | false | false | 当前列是否可支持拖动宽度 |
| name | String | - | true | 当前列表头显示内容 |
| sort | boolean | false | false | 当前列是否支持前端排序 |
| formatter | String | - | false | 当前列是否需要格式化显示内容，支持返回HTML代码段 |
| fixed | String | - | false | 当前列是否固定，left为左边，right为右边 |
| type | String | readonly | true | 编辑状态下当前单元格类别：参照input标签的type类型，其他： dateTime为时间控件，select为下拉框，selectVal中存入选择框的JSON字符串，格式参照selectVal， 如果为readonly则表示不可编辑, 如果为button，则表示当前列不可编辑，且增加按钮 |
| search | boolean | false | false | 如果type为text，是否在输入框内最后追加搜索图标，且提供回调函数 |
| selectVal | String | [] | false | 如果type为select,该属性为下拉 |

## 其他
- 获取当前表格选中行（非多选表格）              this.$refs.tableTest.$refs.eTable.selection[0]
- 获取当前表格选中行（多选表格）                this.$refs.tableTest.$refs.eTable.selection
