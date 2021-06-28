# eTable
基于[pl-table](https://github.com/livelyPeng/pl-table)组件封装的表格，用于解决用户自定义配置列的需求。
使用[element-resize-detector](https://github.com/wnr/element-resize-detector)组件实现表格根据外层标签高度变化

## 使用场景
单选表格、多选表格、可编辑表格、表格列自定义显示
## 使用方式

- ### 全局注册

```javascript
// 此处index为组件地址
import eTabLe from "index";
Vue.component('e-table', eTabLe);
```



- ### 页面内注册

```javascript
// 此处index为组件地址
import eTabLe from "index";
export default {
	components: {
            eTabLe
        }
}
```

- ### 页面内使用

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
| tableIndex | Integer | - | false | 当前列在当前表格中的显示顺序 |
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
| selectVal | String | [] | false | 如果type为select，该属性为下拉框中的选项。注：需提供JSON字符串选项 |
| selectIsInput | boolean | false | false | 如果type为select，该属性为下拉框是够可手动新增，新增的label与value值均为输入值|
| btnType | String | - | false | 如果type为button，该属性为按钮样式风格，参见elementui按钮风格 |
| isPlain | boolean | false | false | 如果type为button，该属性为按钮样式plain风格 |
| icon | String | - | false | 如果type为button，该属性为按钮前缀增加icon |

## 参数使用见[wiki](/wiki)

## 其他
- 获取当前表格选中行（非多选表格）              this.$refs.tableTest.selection[0]
- 获取当前表格选中行（多选表格）                this.$refs.tableTest.selection


## 现有BUG
~~如果页面数据量较大，会造成页面卡顿，此BUG为elementui现存的BUG，解决方案有pl-table等~~（已更新使用为pl-table组件）
大数据状态下，使用表格列编辑后无效，修改的数据不能绑定到数据源中。
