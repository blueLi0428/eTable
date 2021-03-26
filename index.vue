<template>
    <div class="home">
        <el-table
            class="e-tables"
            :data="tableDataList"
            style="width: 100%"
            :height="height"
            :border="border"
            :stripe="stripe"
            :fit="fit"
            :show-header="showHeader"
            :highlight-current-row="highlightCurrentRow && !showSelect"
            :row-key="rowKey"
            :empty-text="emptyText"
            :default-expand-all="defaultExpandAll"
            :expand-row-keys="expandRowKeys"
            :default-sort="defaultSort"
            :tooltip-effect="tooltipEffect ? 'dark' : 'light'"
            :show-summary="showSummary"
            :sum-text="sumText"
            :summary-method="showSummary ? (summaryMethod ? summaryMethod : null) : null"
            :span-method="spanMethod ? spanMethod : null"
            :indent="indent"
            @select="select ? select : null"
            @select-all="selectAll ? selectAll : null"
            @selection-change="selectionChange ? selectionChange : null"
            @cell-mouse-enter="cellMouseEnter ? cellMouseEnter : null"
            @cell-mouse-leave="cellMouseLeave ? cellMouseLeave : null"
            @cell-click="cellClick ? cellClick : null"
            @cell-dblclick="cellDblclick ? cellDblclick : null"
            @row-click="rowClick ? rowClick : null"
            @row-dblclick="rowDbClick ? rowDbClick : null"
            @row-contextmenu="rowContextmenu ? rowContextmenu : null"
            @header-click="headerClick ? headerClick : null"
            @header-contextmenu="headerContextmenu ? headerContextmenu : null"
            @current-change="_currentChange"
            :row-class-name="_rowClassName"
            :row-style="{ height: 28+'px' }"
            :cell-style="{ padding: 0+'px' }"
            sort-orders="['ascending', 'descending', 'default']"
            :tree-props="{children: 'children', hasChildren: 'hasChildren'}"
            :size="size"
            ref="eTable">
            <el-table-column type="selection" width="50" v-if="showSelect" :align="align" reserve-selection/>
            <el-table-column type="index" label="项次" width="50" v-if="showIndex" :align="align"/>
            <el-table-column type="expand" label="操作" width="50" v-if="showExpand" :align="align"/>
            <el-table-column v-for="(item,index) in tableColumns" :key="index" :label="item.name" :fixed="item.fixed ? item.fixed : false" :align="align"
                             :prop="item.prop" :resizable="item.lock" :width="item.width" show-overflow-tooltip v-if="item.show" :sortable="item.sort">
                <template slot-scope="{ row, column, $index }" v-if="showColumn">
<!--                    是编辑状态-->
                    <template v-if="isEdit && $refs.eTable.selection[0] === row">
<!--                        只读类别-->
                        <template v-if="item.type === 'readonly'">
                            <span v-if="item.formatter" v-html="_formatter(row, column, row[item.prop], $index, item.formatter)" />
                            <span v-else>{{ row[item.prop] }}</span>
                        </template>
<!--                        下拉框-->
                        <template v-else-if="item.type === 'select'">
<!--                            可手动输入的下拉框-->
                            <el-select v-if="item.selectIsInput" v-model="row[item.prop]" placeholder="请选择" :size="size" allow-create filterable
                                       clearable @change="_rowSelectChange(row, column, $index)">
                                <el-option v-for="(x, option) in item.selectVal" :label="x.label" :value="x.value" :key="option" />
                            </el-select>
<!--                            不可手动新增的下拉框-->
                            <el-select v-else v-model="row[item.prop]" placeholder="请选择" :size="size" allow-create filterable clearable @change="_rowSelectChange(row, column, $index)">
                                <el-option v-for="(x, option) in item.selectVal" :label="x.label" :value="x.value" :key="option" />
                            </el-select>
                        </template>
<!--                        日期控件-->
                        <template v-else-if="item.type === 'dateTime'">
                            <i><el-date-picker type="date" v-model="row[item.prop]" :size="size" @change="_rowInputChange(row, column, $index)"
                                               value-format="yyyyMMdd" format="yyyyMMdd" style="width: 100%" /></i>
                        </template>
<!--                        时间日期控件-->
                        <template v-else-if="item.type === 'dateAndTime'">
                            <i><el-date-picker type="datetime" v-model="row[item.prop]" :size="size" @change="_rowInputChange(row, column, $index)"
                                               value-format="yyyy-MM-dd HH:mm:ss" format="yyyy-MM-dd HH:mm:ss" style="width: 100%" /></i>
                        </template>
<!--                        一般输入框-->
                        <template v-else>
                            <el-input :size="size" v-model="row[item.prop]" :autofocus="true" @input="_rowInputInput(row, column, $index)"
                                      :type="item.type" @change="_rowInputChange(row, column, $index)">
                                <!-- 是否含有搜索按钮-->
                                <i v-if="item.search" slot="suffix" class="el-icon-search el-input__icon"
                                   @click="_searchIcon(row, column, $index)"></i>
                            </el-input>
                        </template>
<!--                        编辑状态的显示内容-->
                        <template v-else>
                            编辑状态下暂不支持显示当前按钮
                        </template>
                    </template>
<!--                    非编辑状态-->
                    <template v-else>
                        <template v-if="item.type === 'button'">
                            <el-button :size="size" @click="_rowButton(row,item.prop, $index)" :type="item.btnType" :plain="item.plain">
                                <i :class="item.icon" />
                                {{ item.name}}
                            </el-button>
                        </template>
                        <span v-else>
                            {{ row[item.prop] }}
                        </span>
                    </template>
                </template>
            </el-table-column>
        </el-table>
        <div style="height: 40px" v-if="isPage">
            <div class="block" style="float:right;">
                <el-pagination
                    @size-change="_pageSizeChange"
                    @current-change="_pageCurrentChange"
                    :current-page="pageParam.eCurrentPage"
                    :page-sizes="pageRange"
                    :page-size="pageParam.ePageSize"
                    layout="total, sizes, prev, pager, next, jumper"
                    :total="pageParam.eTotal"
                ></el-pagination>
            </div>
        </div>
    </div>
</template>
<script>
import Cookies from 'js-cookie'
export default {
    name: "eTable",
    props: {
        name: {
            type: String,
            default: ''
        },
        tableData: [ Object, Array ],
        pageSize: {
            type: Number
        },
        dataUrl: {
            type: String,
            default: ''
        },
        params: {
            type: Object,
            default: null
        },
        tableColumn: {
            type: Array
        },
        pageTotal: {
            type: Number
        },
        height: {
            type: String
        },
        // 是否带有边框，默认带有
        border: {
            type: Boolean,
            default: () => true
        },
        // 是否为斑马纹 table，默认否
        stripe: {
            type: Boolean,
            default: () => false
        },
        // 列的宽度是否自撑开，默认是
        fit: {
            type: Boolean,
            default: () => true
        },
        // 是否显示表头，默认是
        showHeader: {
            type: Boolean,
            default: () => true
        },
        // 是否要高亮当前行，默认否
        highlightCurrentRow: {
            type: Boolean,
            default: () => false
        },
        // 行数据的 Key，用来优化 Table 的渲染；在使用 reserve-selection 功能与显示树形数据时，该属性是必填的。类型为 String 时，支持多层访问：
        // user.info.id，但不支持 user.info[0].id，此种情况请使用 Function。
        rowKey: {
            type: String,
            default: () => 'index'
        },
        // 空数据时显示的文本内容，也可以通过 slot="empty" 设置
        emptyText: {
            type: String,
            default: () => '暂无数据'
        },
        // 是否默认展开所有行，当 Table 包含展开行存在或者为树形表格时有效
        defaultExpandAll: {
            type: Boolean,
            default: () => false
        },
        // 可以通过该属性设置 Table 目前的展开行，需要设置 row-key 属性才能使用，该属性为展开行的 keys 数组。
        expandRowKeys: {
            type: Array,
            default: () => []
        },
        // 默认的排序列的 prop 和顺序。它的prop属性指定默认的排序的列，order指定默认排序的顺序 注：如果只指定了prop, 没有指定order, 则默认顺序是ascending
        defaultSort: {
            type: Object,
            default: () => {}
        },
        // tooltip effect 属性 提示的背景色 默认设置为dark 黑色
        tooltipEffect: {
            type: Boolean,
            default: () => true
        },
        // 是否显示合计行
        //将show-summary设置为true就会在表格尾部展示合计行。
        // 默认情况下，对于合计行，第一列不进行数据求合操作，而是显示「合计」二字（可通过sum-text配置），其余列会将本列所有数值进行求合操作，并显示出来。
        // 当然，你也可以定义自己的合计逻辑。使用summary-method并传入一个方法，返回一个数组，这个数组中的各项就会显示在合计行的各列中，具体可以参考本例中的第二个表格。
        showSummary: {
            type: Boolean,
            default: () => false
        },
        // 合计行第一列的文本
        sumText: {
            type: String,
            default: () => '合计'
        },
        // 自定义的合计计算方法
        summaryMethod: {
            type: Function
        },
        // 	合并行或列的计算方法
        spanMethod: {
            type: Function
        },
        // 展示树形数据时，树节点的缩进
        indent: {
            type: Number,
            default: () => 16
        },
        // 当用户手动勾选数据行的 Checkbox 时触发的事件
        select: {
            type: Function,
        },
        // 当用户手动勾选全选 Checkbox 时触发的事件
        selectAll: {
            type: Function
        },
        // 当选择项发生变化时会触发该事件
        selectionChange: {
            type: Function
        },
        // 当单元格 hover 进入时会触发该事件
        cellMouseEnter: {
            type: Function
        },
        // 当单元格 hover 退出时会触发该事件
        cellMouseLeave: {
            type: Function
        },
        // 当某个单元格被点击时会触发该事件
        cellClick: {
            type: Function
        },
        // 当某个单元格被双击击时会触发该事件
        cellDblclick: {
            type: Function
        },
        // 	当某一行被点击时会触发该事件
        rowClick: {
            type: Function
        },
        // 	当某一行被双击时会触发该事件
        rowDbClick: {
            type: Function
        },
        // 当某一行被鼠标右键点击时会触发该事件
        rowContextmenu: {
            type: Function
        },
        // 当某一列的表头被点击时会触发该事件
        headerClick: {
            type: Function
        },
        // 当某一列的表头被鼠标右键点击时触发该事件
        headerContextmenu: {
            type: Function
        },
        // 当表格的当前行发生变化的时候会触发该事件，如果要高亮当前行，请打开表格的 highlight-current-row 属性
        currentChange: {
            type: Function
        },
        // 复选状态下，是否保留选中项，默认不保留
        keepSelected: {
            type: Boolean,
            default: () => false
        },
        // 对齐方式 left/center/right 默认left
        align: {
            type: String,
            default: () => 'left'
        },
        // 是否显示复选框
        showSelect: {
            type: Boolean,
            default: () => false
        },
        // 是否显示项次
        showIndex: {
            type: Boolean,
            default: () => false
        },
        // 树形表格是否显示可展开的按钮
        showExpand: {
            type: Boolean,
            default: () => false
        },
        // 表格大小
        size:{
            type: String,
            default: Cookies.get('size') || 'mini'
        },
        // 是否分页
        isPage:{
            type: Boolean,
            default: true
        },
        // 分页范围
        pageRange: {
            type: Array,
            default: ()=> [10, 20, 30, 50, 100]
        },
        // 当前表格是否为编辑状态
        isEdit: {
            type: Boolean,
            default: () => false
        },
        // 当前编辑表格中下拉框change事件
        rowSelectChange: {
            type: Function,
        },
        //当前编辑表格中输入框、时间控件change事件
        rowInputChange: {
            type: Function,
        },
        //当前编辑表格中输入框、时间控件input事件
        rowInputInput: {
            type: Function,
        },
        // 当前编辑表格中输入框后search图标单击事件
        searchIcon: {
            type: Function,
        },
        // 非编辑状态下显示按钮的点击事件
        rowButton: {
            type: Function,
        },

    },
    computed:{

    },
    data() {
        return {
            tableColumns: [],// 当前表格加载的列数据
            tableDataList: [],// 当前表格的数据源
            pageParam: {
                eCurrentPage: 1,// 当前页
                ePageSize: 10,// 每页大小
                eTotal: 0,// 总条数
            },
            showColumn: true, //是否显示列
        };
    },
    created(){
        this._getTableColumn()
        this._getTableData()
    },
    updated(){
        this.$refs.eTable.doLayout()
    },
    mounted() {
        this.pageParam.ePageSize = this.pageSize ? this.pageSize : 10
    },
    methods: {
        _getTableColumn() {
            let _this = this
            if (_this.name) {
                let param = {
                    'tableName': _this.name
                }
                _this.$store.dispatch('templateapi/getTableColumn', {
                    url: '/base/tableColumn/getTableColumnByTableName',
                    data: param
                }).then(function(res) {
                    if (res.code !== 200) {
                        _this.$message.warning("当前表格暂无列设置记录！")
                        _this.tableColumns = _this.tableColumn
                    } else {
                        _this.tableColumns = res.data
                    }
                }).catch(() => {
                    _this.tableColumns = _this.tableColumn
                }).finally(() =>{
                    // this._getTableData();
                })
            }else{
                _this.tableColumns = _this.tableColumn
                // this._getTableData();
            }

        },
        /**
         * 如果传递过来的dataUrl有值，则根据地址查询数据
         */
        async _getTableData() {
            let _this = this
            if (_this.dataUrl) {
                _this.params.pageSize = _this.pageParam.ePageSize
                _this.params.currentPage = _this.pageParam.eCurrentPage
                let loading = _this.openLoading();
                try {
                    const res = await _this.$store.dispatch('templateapi/getTableColumn', {
                        url: _this.dataUrl,
                        data: _this.params
                    })
                    if (res.code !== 200) {
                        _this.$message.error(res.msg)
                    } else {
                        if (res.data.records) {
                            _this.tableDataList = res.data.records
                        } else {
                            _this.tableDataList = res.data
                        }
                        _this._addIndex(_this.tableDataList)
                        _this.showColumn = false
                        _this.$nextTick(() => {
                            _this.showColumn = true
                        })
                        _this.pageParam = {
                            eTotal: res.data.total,
                            eCurrentPage: res.data.current,
                            ePageSize: res.data.size
                        }
                    }
                } catch (e) {
                    console.log('error', e)
                    _this.tableDataList = []
                } finally {
                    loading.close()
                }
            } else {
                _this._addIndex(_this.tableData)
                _this.tableDataList = _this.tableData
                _this.pageParam.eTotal = _this.pageTotal
                _this.pageParam.ePageSize = _this.pageSize
            }
        },
        /**
         * 增加_index属性
         * @param data 数据源
         * @private
         */
        _addIndex(data) {
            data.forEach((x, index) => {
                this.$set(x, '_index', index)
            })
        },
        /**
         * 下拉框change事件
         * @param row 选中行
         * @param column 选中列数据
         * @param data 当前单元格数据
         * @param index 当前行索引
         * @param formatKey format方法名称
         * @private
         */
        _formatter(row, column, data, index, formatKey){
            let parent = this.$parent;
            while (parent) {
                if (parent[formatKey]) {
                    break;
                }
                parent = parent.$parent;
            }
            if (parent && parent[formatKey]) {
                return parent[formatKey](row, column, data, index);
            }
            return data;
        },
        /**
         * 下拉框change事件
         * @param row 选中行
         * @param column 选中列数据
         * @param index 当前行索引
         * @private
         */
        _rowSelectChange(row, column, index) {
            if (this.rowSelectChange) {
                this.rowSelectChange(row, column, index)
            }
        },
        /**
         * 输入框、事件控件的change事件
         * @param row 选中行
         * @param column 选中列数据
         * @param index 当前行索引
         * @private
         */
        _rowInputChange(row, column, index) {
            if (this.rowInputChange) {
                this.rowInputChange(row, column, index)
            }
        },
        /**
         * 输入框input事件
         * @param row 选中行
         * @param column 选中列数据
         * @param index 当前行索引
         * @private
         */
        _rowInputInput(row, column, index) {
            if (this.rowInputInput) {
                this.rowInputInput(row, column, index)
            }
        },
        /**
         * 搜索图标点击事件
         * @param row 选中行
         * @param column 选中列数据
         * @param index 当前行索引
         * @private
         */
        _searchIcon(row, column, index) {
            if (this.searchIcon) {
                this.searchIcon(row, column, index)
            }
        },
        /**
         * 按钮点击事件
         * @param row 选中行
         * @param column 选中列数据
         * @param index 当前行索引
         * @private
         */
        _rowButton(row, column, index) {
            if (this.rowButton) {
                this.rowButton(row, column, index)
            }
        },
        /**
         * 当前页大小change事件
         * @param val 切换大小
         * @private
         */
        _pageSizeChange(val) {
            this.pageParam.ePageSize = val
            this._getTableData()
            this.$emit("page-changed", this.pageParam)
        },
        /**
         * 当前页面change事件
         * @param val 当前页
         * @private
         */
        _pageCurrentChange(val) {
            this.pageParam.eCurrentPage = val
            this._getTableData()
            this.$emit("page-changed", this.pageParam)
        },
        /**
         * 某一行切换选中事件（非checkbox选中）
         * @param currentRow 当前选中行
         * @param oldCurrentRow 旧的选中行
         * @private
         */
        _currentChange(currentRow, oldCurrentRow) {
            // 是否为复选
            if (this.showSelect) {
                // 保留选中状态
                if (this.keepSelected) {
                    let isSelect = false
                    for (let x of this.$refs.eTable.selection) {
                        if (currentRow === x) {
                            // 说明已选中，再次点击需要取消选中
                            this.$refs.eTable.toggleRowSelection(currentRow, false)
                            isSelect = true
                            break
                        }
                    }
                    if (!isSelect) {
                        // 说明没有选中，需要设置为选中
                        this.$refs.eTable.toggleRowSelection(currentRow, true)
                    }
                } else {
                    // 不需要判断直接清理以往选中项，设置当前行为选中
                    this.$refs.eTable.clearSelection();
                    this.$refs.eTable.toggleRowSelection(currentRow, true);
                }
            } else {
                this.$refs.eTable.selection[0] = currentRow;
            }
            if (this.currentChange) {
                this.currentChange(currentRow, oldCurrentRow)
            }
        },
        /**
         * 设置索引
         * @param row 当前行
         * @param rowIndex 当前行所在的索引，从0开始
         * @private
         */
        _rowClassName({row, rowIndex}) {
            row._index = rowIndex
        }
    },
    watch:{
        params(){
            this._getTableData();
        },
        tableDataList(tableDataList) {
            this.$nextTick(() => {
                this.$emit('loadData', tableDataList)
            })

        }
    }
};
</script>

<style scoped>
    /* 修改表格样式 */
    .el-table tr td{
        padding: 6px 0;
    }
    .e-tables .el-table__fixed-left{
        height: 100% !important;
    }
</style>

