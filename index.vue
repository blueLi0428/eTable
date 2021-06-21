<template>
    <div class="home" ref="home">
        <pl-table
            class="e-tables"
            :data="tableDataList"
            style="width: 100%"
            :height="tableHeight"
            :border="border"
            :stripe="stripe"
            :fit="fit"
            :show-header="showHeader"
            :highlight-current-row="highlightCurrentRow"
            :row-key="rowKey"
            :empty-text="emptyText"
            :default-expand-all="defaultExpandAll"
            :expand-row-keys="expandRowKeys"
            :default-sort="defaultSort"
            :tooltip-effect="tooltipEffect ? 'dark' : 'light'"
            :show-summary="showSummary"
            :sum-text="sumText"
            :summary-method="_summaryMethod"
            :span-method="_spanMethod"
            :indent="indent"
            @select="_select"
            @select-all="_selectAll"
            @selection-change="_selectionChange"
            @cell-mouse-enter="_cellMouseEnter"
            @cell-mouse-leave="_cellMouseLeave"
            @cell-click="_cellClick"
            @cell-dblclick="_cellDblclick"
            @row-click="_rowClick"
            @row-dblclick="_rowDbClick"
            @row-contextmenu="_rowContextmenu"
            @header-click="_headerClick"
            @header-contextmenu="_headerContextmenu"
            @current-change="_currentChange"
            :row-class-name="_rowClassName"
            @header-dragend="_headerDragent"
            :row-style="{ height: 28+'px' }"
            :cell-style="_cellStyle"
            sort-orders="['ascending', 'descending', 'default']"
            :tree-props="{children: 'children', hasChildren: 'hasChildren'}"
            :size="size"
            :use-virtual="useVirtual"
            :row-height="rowHeight"
            :big-data-checkbox="bigDataCheckbox"
            ref="eTable">
            <pl-table-column type="selection" width="50" v-if="showSelect" :align="align" reserve-selection/>
            <pl-table-column type="index" label="项次" width="50" v-if="showIndex" :align="align" :fixed="indexFixed" />
            <pl-table-column type="expand" label="操作" width="50" v-if="showExpand" :align="align"/>
            <pl-table-column v-if="showAppendColumn && appendLeft" :label="appendText" :min-width="appendWidth" :align="align">
                <template slot-scope="{ row, column, $index }">
                    <slot name="append-column" :row="row" :column="column" :$index="$index" />
                </template>
            </pl-table-column>
            <pl-table-column v-for="(item,index) in tableColumns" :key="index" :label="item.name" :fixed="item.fixed ? item.fixed : false" :align="align"
                             :prop="item.prop" :resizable="item.lock" :min-width="item.width" show-overflow-tooltip v-if="item.show" :sortable="item.sort">
                <template slot-scope="{ row, column, $index }" v-if="showColumn">
                    <!--                    是编辑状态-->
                    <template v-if="isEdit && $refs.eTable.$refs.singleTable.selection[0] === row || isSearchEdit">
                        <!--                        只读类别-->
                        <template v-if="item.type === 'readonly' && !isSearchEdit">
                            <span v-if="item.formatter" v-html="_formatter(row, column, row[item.prop], $index, item.formatter)" />
                            <span v-else>{{ row[item.prop] }}</span>
                        </template>
                        <!--                        下拉框-->
                        <template v-else-if="item.type === 'select'">
                            <!--                            可手动输入的下拉框-->
                            <el-select v-if="item.selectIsInput" v-model="row[item.prop]" placeholder="请选择" :size="size" allow-create filterable
                                       clearable @change="_rowSelectChange(row, column, $index)">
                                <el-option v-for="(x, option) in JSON.parse(item.selectVal)" :label="x.label" :value="x.value" :key="option" />
                            </el-select>
                            <!--                            不可手动新增的下拉框-->
                            <el-select v-else v-model="row[item.prop]" placeholder="请选择" :size="size" clearable @change="_rowSelectChange(row, column, $index)">
                                <el-option v-for="(x, option) in JSON.parse(item.selectVal)" :label="x.label" :value="x.value" :key="option" />
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
                                      :type="item.type" @change="_rowInputChange(row, column, $index)" :ref="'eTableInput' + index + '_' + $index"
                                      @keyup.enter.native="_inputEnter('eTableInput' + index + '_', $index)">
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
                        <span v-else v-html="_formatter(row, column, row[item.prop], $index, item.formatter)" />
                    </template>
                </template>
            </pl-table-column>
            <pl-table-column v-if="showAppendColumn && !appendLeft" :label="appendText" :min-width="appendWidth" :align="align">
                <template slot-scope="{ row, column, $index }">
                    <slot name="append-column" :row="row" :column="column" :$index="$index" />
                </template>
            </pl-table-column>
        </pl-table>
        <div v-if="isPage">
            <div class="block" style="float:right;">
                <el-pagination
                    @size-change="_pageSizeChange"
                    @current-change="_pageCurrentChange"
                    :current-page="pageParam.currentPage"
                    :page-sizes="pageRange"
                    :page-size="pageParam.pageSize"
                    layout="total, sizes, prev, pager, next, jumper"
                    :total="pageParam.total"
                ></el-pagination>
            </div>
        </div>
        <el-button
            v-if="isColumnEdit"
            class="user_column"
            type="text"
            icon="el-icon-menu"
            @click="showColumnEdit"
        />
        <el-dialog :visible.sync="isColumnDialog" width="65%" top="50px" title="列属性面板" center append-to-body>
            <div>
                <el-table :data="columnRowData" highlight-current-row @current-change="columnCurrentChange" ref="columnTable" stripe max-height="600">
                    <el-table-column type="index" fixed="left" />
                    <!--                    <el-table-column prop="prop" label="属性名" fixed="left" width="120px"/>-->
                    <!--                    <el-table-column prop="tableIndex" label="顺序" width="50px" />-->
                    <el-table-column prop="name" label="显示名称" width="100px" />
                    <el-table-column label="宽度" width="120px" align="center">
                        <template slot-scope="{ row }">
                            <el-input-number
                                v-model="row.width"
                                style="width: 110px"
                                :controls="false"
                            />
                        </template>
                    </el-table-column>
                    <el-table-column label="是否固定显示" width="200px">
                        <template slot-scope="{ row }">
                            <el-radio-group v-model="row.fixed" :fill="row.fixed === 'left' ? '#13ce66' : row.fixed === 'right' ? '#ff4949' : '#dddd00'">
                                <el-radio-button label="left" fill="#13ce66">靠左</el-radio-button>
                                <el-radio-button label="right" fill="#ff4949">靠右</el-radio-button>
                                <el-radio-button label="" fill="#ff4949">不固定</el-radio-button>
                            </el-radio-group>
                        </template>
                    </el-table-column>
                    <el-table-column label="是否显示" width="135" align="center">
                        <template slot-scope="{ row }">
                            <el-radio-group v-model="row.show" :fill="row.show ? '#13ce66' : '#ff4949'">
                                <el-radio-button :label="true" fill="#13ce66">显示</el-radio-button>
                                <el-radio-button :label="false" fill="#ff4949">隐藏</el-radio-button>
                            </el-radio-group>
                        </template>
                    </el-table-column>
                    <el-table-column label="锁定" width="170" align="center">
                        <template slot-scope="{ row }">
                            <el-radio-group v-model="row.lock" :fill="row.lock ? '#13ce66' : '#ff4949'">
                                <el-radio-button :label="true" fill="#13ce66">可拖动</el-radio-button>
                                <el-radio-button :label="false" fill="#ff4949">禁止拖动</el-radio-button>
                            </el-radio-group>
                        </template>
                    </el-table-column>
                    <el-table-column label="是否支持排序" width="170" align="center">
                        <template slot-scope="{ row }">
                            <el-radio-group v-model="row.sort" :fill="row.sort ? '#13ce66' : '#ff4949'">
                                <el-radio-button :label="true" fill="#13ce66">可排序</el-radio-button>
                                <el-radio-button :label="false" fill="#ff4949">不可排序</el-radio-button>
                            </el-radio-group>
                        </template>
                    </el-table-column>
                    <el-table-column label="编辑" width="200px" align="center">
                        <template slot-scope="{ row, column, index }">
                            <el-button :disabled="row.tableIndex === 1" @click="columnRowUp(row, column, index)">上移</el-button>
                            <el-button :disabled="row.tableIndex === columnRowData[columnRowData.length - 1].tableIndex" @click="columnRowDown(row, column, index)">下移</el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </div>
            <div slot="footer" class="dialog-footer">
                <el-button type="success" plain @click="columnRowSave">保存</el-button>
                <el-button type="warning" plain @click="columnRowReset" style="margin-left: 30px;">恢复默认</el-button>
                <el-button type="primary" plain @click="isColumnDialog = false" style="margin-left: 30px;">取消设置</el-button>
            </div>
        </el-dialog>
    </div>
</template>
<script>
import Cookies from 'js-cookie'
import elementResizeDetectorMaker from 'element-resize-detector'
export default {
    name: "eTable",
    props: {
        name: {
            type: String,
            default: ''
        },
        tableData: [ Object, Array ],
        pageSize: {
            type: Number,
            default: 10,
        },
        currentPage: {
            type: Number,
            default: () => 1
        },
        dataUrl: {
            type: String,
            default: ''
        },
        isGetData: {
            type: Boolean,
            default: () => true
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
        // 当前表格的cell-style事件
        cellStyle: {
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
        // index项是否默认固定展示
        indexFixed: {
            type: Boolean,
            default: () => false
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
        // 是否让用户自定义列属性
        isColumnEdit: {
            type: Boolean,
            default: () => true
        },
        // 设置pl表格默认行高
        rowHeight: {
            type: Number,
            default: () => 23
        },
        // 是否开启表格虚拟化，用于解决表格数据过多卡顿
        useVirtual: {
            type: Boolean,
            default: () => false
        },
        // 当存在复选表格时，如果数据量超过3000建议开启此功能，用于解决复选操作时的卡顿
        bigDataCheckbox: {
            type: Boolean,
            default: () => false
        },
        // 搜索状态下是否所有字段编辑
        isSearchEdit: {
            type: Boolean,
            default: () => false
        },
        // 是否显示追加列
        showAppendColumn: {
            type: Boolean,
            default: () => false
        },
        // 追加列的显示名
        appendText: {
            type: String,
            default: () => '操作'
        },
        // 追加列的宽度
        appendWidth: {
            type: Number,
            default: () => 80
        },
        // 追加列是否显示在索引列后
        appendLeft: {
            type: Boolean,
            default: () => false
        }
    },
    computed:{

    },
    data() {
        return {
            tableColumns: [],// 当前表格加载的列数据
            tableDataList: [],// 当前表格的数据源
            tableHeight: 0,// 表格高度
            pageParam: {
                currentPage: 1,// 当前页
                pageSize: 10,// 每页大小
                total: 0,// 总条数
            },
            showColumn: true, //是否显示列
            isColumnDialog: false,// 是否显示当前属性字段弹出层
            columnSelectRow: {},// 选中行
            columnRowData: [],// 另外设置数据源，防止在未保存的时候修改样式，造成页面混乱
            data: [],
            selection: [],
            defaultData: [],// 初始化表格值，为初始化赋值时或者通过组件发请求获取到表格时的值
        };
    },
    created(){
    },
    updated(){
        this.$refs.eTable.doLayout()
    },
    mounted() {
        let _this = this
        if (_this.height && _this.height.indexOf('%') !== -1) {
            const erd = elementResizeDetectorMaker()
            erd.listenTo(_this.$refs.home, function(element) {
                _this.tableHeight = element.offsetHeight
            })
        } else {
            this.tableHeight = this.height
        }

        this.pageParam.pageSize = this.pageSize ? this.pageSize : 10
        this._getTableColumn()
        this._getTableData()
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
            if (_this.dataUrl && _this.isGetData) {
                if (_this.isPage) {
                    _this.params.pageSize = _this.pageParam.pageSize
                    _this.params.currentPage = _this.pageParam.currentPage
                }
                let loading = _this.openLoading();
                try {
                    const res = await _this.$store.dispatch('templateapi/getTableColumn', {
                        url: _this.dataUrl,
                        data: _this.params
                    })
                    if (res.code !== 200) {
                        _this.$message.error(res.msg)
                    } else {
                        if (this.isPage) {
                            if (res.data.records) {
                                _this.tableDataList = res.data.records
                            } else {
                                _this.tableDataList = res.data
                            }
                        } else {
                            _this.tableDataList = res.data
                        }

                        _this._addIndex(_this.tableDataList)
                        _this.showColumn = false
                        _this.$nextTick(() => {
                            _this.showColumn = true
                        })
                        if (this.isPage) {
                            _this.pageParam = {
                                total: res.data.total,
                                currentPage: res.data.current,
                                pageSize: res.data.size
                            }
                        } else {
                            _this.pageParam = {
                                total: res.data.length,
                                currentPage: 1,
                                pageSize: res.data.length
                            }
                        }
                    }
                } catch (e) {
                    console.error('error', e)
                    _this.tableDataList = []
                } finally {
                    loading.close()
                }
            } else {
                _this._addIndex(_this.tableData)
                _this.tableDataList = _this.tableData
                _this.pageParam.total = _this.pageTotal
                _this.pageParam.pageSize = _this.pageSize
            }
            this.$nextTick(() => {
                _this.setCurrentRow(_this.tableDataList[0])
            })
            _this.defaultData = JSON.parse(JSON.stringify(_this.tableDataList))
            _this.data = _this.tableDataList
        },
        /**
         * 增加_index属性
         * @param data 数据源
         * @private
         */
        _addIndex(data) {
            if (data instanceof Array){
                data.forEach((x, index) => {
                    this.$set(x, '_index', index)
                })
            }
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
            this.pageParam.pageSize = val
            this.$emit("page-changed", this.pageParam)
            this._getTableData()
        },
        /**
         * 当前页面change事件
         * @param val 当前页
         * @private
         */
        _pageCurrentChange(val) {
            this.pageParam.currentPage = val
            this.$emit("page-changed", this.pageParam)
            this._getTableData()
        },
        /**
         * 某一行切换选中事件（非checkbox选中）
         * @param currentRow 当前选中行
         * @param oldCurrentRow 旧的选中行
         * @private
         */
        _currentChange(currentRow, oldCurrentRow) {
            if (currentRow === null) {
                return
            }
            // 是否为复选
            if (this.showSelect) {
                // 保留选中状态
                if (this.keepSelected) {
                    let isSelect = false
                    for (let x of this.$refs.eTable.$refs.singleTable.selection) {
                        if (currentRow === x) {
                            // 说明已选中，再次点击需要取消选中
                            this.$refs.eTable.toggleRowSelection([{ row: currentRow, selected:false }])
                            isSelect = true
                            break
                        }
                    }
                    if (!isSelect) {
                        // 说明没有选中，需要设置为选中
                        this.$refs.eTable.toggleRowSelection([{ row: currentRow, selected:true }])
                    }
                } else {
                    // 不需要判断直接清理以往选中项，设置当前行为选中
                    this.$refs.eTable.clearSelection();
                    this.$refs.eTable.toggleRowSelection([{ row: currentRow, selected:true }]);
                }
            } else {
                this.$refs.eTable.$refs.singleTable.selection[0] = currentRow;
            }
            if (this.currentChange) {
                this.currentChange(currentRow, oldCurrentRow)
            }
            this.selection = this.$refs.eTable.$refs.singleTable.selection
        },
        /**
         * 设置索引
         * @param row 当前行
         * @param rowIndex 当前行所在的索引，从0开始
         * @private
         */
        _rowClassName({row, rowIndex}) {
            row._index = rowIndex
        },

        /**
         *输入框回车切换下一行事件事件
         **/
        _inputEnter(refName, rowIndex) {
            if (this.tableData.length <= rowIndex + 1)
                // 设置选中行
                this.setCurrentRow(this.tableData[rowIndex + 1])
            this.$nextTick(() => {
                this.$refs[refName + (rowIndex + 1)][0].focus()
            })
        },

        /**
         * 当拖动表头改变了列的宽度的时候会触发该事件
         * @param newWidth
         * @param oldWidth
         * @param column
         * @param event
         */
        _headerDragent(newWidth, oldWidth, column, event) {
            for (let x of this.tableColumns) {
                if (x.prop === column.property && oldWidth === x.width) {
                    x.width = newWidth
                    this.columnRowData = JSON.parse(JSON.stringify(this.tableColumns))
                    this.columnRowSave()
                }
            }
        },

        /**
         * 单元格style事件
         * @param row 行数据
         * @param column 列数据
         * @param rowIndex 行索引
         * @param columnIndex 列索引
         **/
        _cellStyle({row, column, rowIndex, columnIndex}) {
            if (this.cellStyle) {
                let style = this.cellStyle(row, column, rowIndex, columnIndex)
                if (!style) {
                    return {padding: 0 + 'px'}

                }
                if (!style.padding) {
                    style.padding = 0 + 'px'
                    return style
                }
            }
        },
        /**
         * 合并行方法
         * @param row 当前行
         * @param column 当前列
         * @param rowIndex 当前行索引
         * @param columnIndex 当前列索引
         */
        _spanMethod(row, column, rowIndex, columnIndex) {
            if (this.spanMethod) {
                this.spanMethod(row, column, rowIndex, columnIndex)
            }
        },
        /**
         * 选中某一行（复选框）
         * @param selection 选中的所有行
         * @param row 当前选中行
         */
        _select(selection, row) {
            if (this.select) {
                this.select(selection, row)
            }
        },
        /**
         * 选中所有行数据（复选框）
         * @param selection 选中的所有行
         */
        _selectAll(selection) {
            if (this.selectAll) {
                this.selectAll(selection)
            }
        },
        /**
         * 选中行发生变化触发（复选）
         * @param selection 选中的所有行
         */
        _selectionChange(selection) {
            if (this.selectionChange) {
                this.selectionChange(selection)
            }
        },
        /**
         * 单元格hover进入后触发
         * @param row 选中的所有行
         * @param column 当前列
         * @param cell 当前单元格
         * @param event 事件
         */
        _cellMouseEnter(row, column, cell, event) {
            if (this.cellMouseEnter) {
                this.cellMouseEnter(row, column, cell, event)
            }
        },
        /**
         * 单元格hover移出后触发
         * @param row 选中的所有行
         * @param column 当前列
         * @param cell 当前单元格
         * @param event 事件
         */
        _cellMouseLeave(row, column, cell, event) {
            if (this.cellMouseLeave) {
                this.cellMouseLeave(row, column, cell, event)
            }
        },
        /**
         * 单元格单击事件
         * @param row 选中的所有行
         * @param column 当前列
         * @param cell 当前单元格
         * @param event 事件
         */
        _cellClick(row, column, cell, event) {
            if (this.cellClick) {
                this.cellClick(row, column, cell, event)
            }
        },
        /**
         * 单元格双击事件
         * @param row 选中的所有行
         * @param column 当前列
         * @param cell 当前单元格
         * @param event 事件
         */
        _cellDblclick(row, column, cell, event) {
            if (this.cellDblclick) {
                this.cellDblclick(row, column, cell, event)
            }
        },
        /**
         * 行双击事件
         * @param row 选中的所有行
         * @param column 当前列
         * @param event 事件
         */
        _rowClick(row, column, event) {
            if (this.rowClick) {
                this.rowClick(row, column, event)
            }
        },
        /**
         * 行双击事件
         * @param row 选中的所有行
         * @param column 当前列
         * @param event 事件
         */
        _rowDbClick(row, column, event){
            if (this.rowDbClick) {
                this.rowDbClick(row, column, event)
            }
        },
        /**
         * 某一行被鼠标右键时触发
         * @param row 选中的所有行
         * @param column 当前列
         * @param event 事件
         */
        _rowContextmenu(row, column, event) {
            if (this.rowContextmenu){
                this.rowContextmenu(row, column, event)
            }
        },
        /**
         * 某一列表头被点击时
         * @param column 当前列
         * @param event 事件
         */
        _headerClick(column, event) {
            if (this.headerClick) {
                this.headerClick(column, event)
            }
        },
        /**
         * 某一列表头被鼠标右键点击时
         * @param column 当前列
         * @param event 事件
         */
        _headerContextmenu(column, event) {
            if (this.headerContextmenu) {
                this.headerContextmenu(column, event)
            }
        },
        /**
         * 自定义合计计算方法
         * @param columns 未行
         * @param data 数据源
         */
        _summaryMethod(columns, data) {
            if (this.showSummary && this.summaryMethod) {
                this.summaryMethod(columns, data)
            }
        },

        /**
         * 恢复数据显示为默认查询结果
         */
        _resetTableData() {
            this.tableDataList = JSON.parse(JSON.stringify(this.defaultData))
            this.data = this.tableDataList
        },

        /**
         * 显示表格列用户自定义属性界面
         */
        showColumnEdit(){
            this.columnRowData = JSON.parse(JSON.stringify(this.tableColumns))
            this.isColumnDialog = true
        },
        /**
         * 表格列属性设置界面当前用户选中行
         * @param row
         */
        columnCurrentChange(row){
            this.columnSelectRow = row
        },
        /**
         * 当前选中行上移
         * @param row
         * @param column
         * @param index
         */
        columnRowUp(row, column, index){
            let up = JSON.parse(JSON.stringify(row))
            for(let i = 0; i < this.columnRowData.length - 1; i++){
                if(this.columnRowData[i + 1].tableIndex === up.tableIndex){
                    this.columnRowData[i + 1] = JSON.parse(JSON.stringify(this.columnRowData[i]))
                    this.columnRowData[i + 1].tableIndex = i + 2
                    this.columnRowData[i] = up
                    this.columnRowData[i].tableIndex = i + 1
                    this.$refs.columnTable.setCurrentRow(this.columnRowData[i], true)
                    return
                }
            }
        },
        /**
         * 当前选中行下移
         * @param row
         * @param column
         * @param index
         */
        columnRowDown(row, column, index){
            let down = JSON.parse(JSON.stringify(row))
            for(let i = 0; i < this.columnRowData.length - 1; i++){
                if(this.columnRowData[i].tableIndex === down.tableIndex){
                    this.columnRowData[i] = JSON.parse(JSON.stringify(this.columnRowData[i + 1]))
                    this.columnRowData[i].tableIndex = i + 1
                    this.columnRowData[i + 1] = down
                    this.columnRowData[i + 1].tableIndex = i + 2
                    this.$refs.columnTable.setCurrentRow(this.columnRowData[i + 1], true)
                    return
                }
            }
        },
        /**
         * 保存当前设置
         */
        columnRowSave(){
            let param = {
                userTableColumnStr: JSON.stringify(this.columnRowData),
                tableName: this.name
            }
            let loading = this.openLoading("保存中...")
            this.$store.dispatch('templateapi/updateByUserColumn', param).then(res =>{
                if(res.code === 200){
                    this.$message.success("修改成功！")
                    this.tableColumns = JSON.parse(JSON.stringify(this.columnRowData))
                    this.isColumnDialog = false
                }else {
                    this.$message.error("修改失败！")
                }
            }).finally(() =>{
                loading.close()
            })
        },
        /**
         * 重置为初始化设置
         */
        columnRowReset(){
            let param = {
                tableName: this.name
            }
            let loading = this.openLoading("保存中...")
            this.$store.dispatch('templateapi/resetUserColumn', param).then(res =>{
                if(res.code === 200){
                    this.$message.success("重置成功！")
                    this.tableColumns = res.data
                    this.isColumnDialog = false
                }else {
                    this.$message.error("重置失败！")
                }
            }).finally(() =>{
                loading.close()
            })
        },

        /**
         * el-table原生方法移植
         */
        /**
         * 设置某一行选中
         * @param row
         */
        setCurrentRow(row) {
            this.$refs.eTable.setCurrentRow(row)
            this.selection = this.$refs.eTable.$refs.singleTable.selection
        },
        /**
         * 复选框设置选中状态
         * @param row 选中行
         * @param bool 是否选中，默认true
         */
        toggleRowSelection(row, bool = true) {
            this.$refs.eTable.toggleRowSelection([{ row: row, selected:bool }])
            this.selection = this.$refs.eTable.$refs.singleTable.selection
        },
        /**
         * 用于多选表格，切换所有行的选中状态
         */
        toggleAllSelection() {
            this.$refs.eTable.toggleAllSelection()
            this.selection = this.$refs.eTable.$refs.singleTable.selection
        },
        /**
         * 多选表格清除所有选中行
         */
        clearSelection() {
            this.$refs.eTable.clearSelection()
            this.selection = this.$refs.eTable.$refs.singleTable.selection
        },
        /**
         * 表格组件导出
         * @param data 数据源，不传则默认为当前表格展示数据
         * @param fileName 导出文件名
         * @param isAllColumn 是否展示隐藏设置字段
         */
        exportTableData(data, fileName, isAllColumn = false) {
            let _this = this
            let tHeader = []
            let filterVal = []
            let list
            let titStyle = {font: {name: '宋体', sz: 12, color: {rgb: "000000"}, bold: true, italic: false, underline: false}, alignment: {horizontal: "center", vertical: "center"},}
            let cellStyle = {font: {name: '宋体', sz: 12, color: {rgb: "000000"}, italic: false, underline: false}, alignment: {horizontal: "left", vertical: "center"}}
            _this.tableColumns.forEach(x => {
                if (isAllColumn) {
                    tHeader.push(x.name)
                    filterVal.push(x.prop)
                } else {
                    if (x.show) {
                        tHeader.push(x.name)
                        filterVal.push(x.prop)
                    }
                }
            })
            if (data && data.length > 0) {// 传入数据导出
                list = JSON.parse(JSON.stringify(data))
            } else {
                list = JSON.parse(JSON.stringify(_this.tableDataList))
            }
            const excelData = _this.formatJson(filterVal, list)
            import('@/vendor/Export2Excel').then(excel => {
                excel.export_json_to_excel({
                    header: tHeader,
                    data: excelData,
                    filename: fileName,
                    autoWidth: false,
                    bookType: 'xlsx',
                    cellWidth: 15.29,
                    titStyle: titStyle,
                    cellStyle: cellStyle,
                    rowHeight: 25
                })
            })
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

        },
        tableData() {
            this.tableDataList = this.tableData
            this.data = this.tableData
            this.selection = this.$refs.eTable.$refs.singleTable.selection
        },
        pageTotal(val) {
            this.pageParam.total = val
        },
        pageSize(val) {
            this.pageParam.pageSize = val
        },
        currentPage(val) {
            this.pageParam.currentPage = val
        }
    }
};
</script>

<style scoped>
.el-table tr td{
    padding: 6px 0;
}
.e-tables .el-table__fixed-left{
    height: 100% !important;
}
.cell span{
    color: #000000;
    font-weight: 700;
}
.home{
    height: calc(100% - 12px);
    position:relative;
}
.user_column{
    position: absolute;
    top: 0;
    right: 5px;
    z-index: 3;
}
</style>
<style>
.el-table__body tr.current-row>td{
    background-color: #92ADC6 !important;
}
</style>

