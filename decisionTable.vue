<template>
  <div>
    <!-- <el-button type="primary"
                     plain
                     round
                     @click="addNewTable">新增决策表</el-button> -->
    <el-table :data="decisionTableData"
              border
              v-loading="decisionLoading"
              :row-class-name="tableRowClassName"
              :span-method="objectSpanMethod"
              :row-style="{fontSize:'1.25rem'}"
              :header-cell-style="{fontSize:'1.25rem'}"
              style="width:100%;"
              @header-contextmenu="headerContextmenu"
              @row-contextmenu="rowContextmenu"
              @cell-click="cellClick">
      <el-table-column v-for="(headerItem, headerIndex) in paramsHeader"
                       :key="headerIndex"
                       :index="headerIndex"
                       type="params">

        <template slot="header">
          <span class="action-class"
                @click='headerClick(headerItem,headerIndex)'>
            {{headerItem.header}}
          </span>
          <vue-context-menu :contextMenuData="paramsContextMenuData"
                            :transferIndex="paramsTransferIndex"
                            @addColumn="addParamsColumn(headerItem, headerIndex)"
                            @deleteColumn="deleteParamsColumn(headerItem, headerIndex)"></vue-context-menu>

        </template>
        <template slot-scope="scope">
          <span class="params-class">
            {{scope.row.params[headerIndex].name}}
          </span>
          <vue-context-menu :contextMenuData="paramsValueContextMenuData"
                            :transferIndex="paramsValueTransferIndex"
                            @clear="clearParamsRow(headerItem, headerIndex)"
                            @addRow="addParamsRow(headerItem, headerIndex)"
                            @deleteRow="deleteParamsRow(headerItem, headerIndex)"></vue-context-menu>
          <!-- 动作类型展示名称 -->
          <!-- <span slot="reference"
                    class="action-class">{{isAcItem.actionTypeName}}:</span> -->

        </template>
      </el-table-column>
      <el-table-column v-for="(valuation, vIndex) in valuationHeader"
                       align="center"
                       :key="vIndex + 100"
                       :index="vIndex"
                       type="value">
        <template slot="header">
          <span class="value-header"
                @click='valuationHeaderClick(valuation,vIndex)'>
            {{valuation.header}}
          </span>
          <vue-context-menu :contextMenuData="valuationContextMenuData"
                            :transferIndex="valuationTransferIndex"
                            @addColumn="addValuationColumn(valuation, vIndex)"
                            @deleteColumn="deleteValuationColumn(valuation, vIndex)"></vue-context-menu>
        </template>
        <template slot-scope="scope">
          <el-popover placement="bottom"
                      width="100"
                      trigger="hover">
            <div class="param-type"
                 v-for="(dropdown, droIndex) in valueList"
                 @click="valueAcTyClick(scope.row,dropdown,droIndex, vIndex, scope.$index)"
                 :key="dropdown.value">
              {{dropdown.label}}
            </div>
            <span class="params-class"
                  slot="reference">
              <span>
                {{scope.row.valuation[vIndex].value}}
              </span>
            </span>

            <!-- 动作类型展示名称 -->
            <!-- <span slot="reference"
                    class="action-class">{{isAcItem.actionTypeName}}:</span> -->
          </el-popover>

        </template>
      </el-table-column>
    </el-table>
    <el-dialog title="配置条件"
               :visible.sync="conDialogVisible"
               width="65%"
               @close="handleCloseCon"
               :append-to-body="true"
               center>
      <el-row :gutter="5">
        <el-col :span="24">
          <el-divider content-position="left">
          </el-divider>
        </el-col>
        <el-col :span="24"
                :offset="1">
          <el-row :gutter="0">
            <el-col :span="3">
              <el-select v-model="configurationList.dispute"
                         style="width:100%"
                         placeholder="请选择">
                <el-option label="或者"
                           value="||"></el-option>
                <el-option label="并且"
                           value="&&"></el-option>
              </el-select>
            </el-col>
            <el-col :span="21">
              <div v-for="(set, index) in configurationList.ruelSet "
                   :key="index">
                <!-- 操作符选择 -->
                <el-select v-model="set.symbol"
                           style="width:25%"
                           class="left-form"
                           placeholder="选择比较操作符,可搜索"
                           filterable
                           clearable>
                  <el-option v-for="(symbol,index) in symbolList"
                             :key="index"
                             :label="symbol.label"
                             :value="symbol.value">
                  </el-option>
                </el-select>
                <!-- 值类型选择 -->
                <el-select v-model="set.valueType"
                           class="left-form"
                           style="width:15%"
                           filterable
                           clearable
                           placeholder="选择值类型">
                  <el-option v-for="(valueType, index) in valueTypeList"
                             :key="index"
                             :label="valueType.label"
                             :value="valueType.value">
                  </el-option>
                </el-select>
                <!-- 最终值选择 -->
                <span>
                  <el-input v-model="set.value"
                            style="width:25%"
                            v-if="set.valueType === 'Input'"></el-input>
                  <el-select v-model="set.value"
                             style="width:25%"
                             class="left-form"
                             filterable
                             clearable
                             placeholder="选择变量"
                             v-if="set.valueType === 'Parameter'">
                    <el-option v-for="(variable, index) in variableList"
                               :key="index"
                               :label="variable.label"
                               :value="variable.value">
                    </el-option>
                  </el-select>
                </span>
                <!-- 操作 -->
                <span class="left-form">
                  <el-button type="primary"
                             icon="el-icon-plus"
                             circle
                             size="small"
                             @click="addNewRuleSet(index)"></el-button>
                  <el-button type="danger"
                             icon="el-icon-delete"
                             circle
                             size="small"
                             @click="deleteRuleSet(set,index)"></el-button>
                </span>
              </div>
            </el-col>
          </el-row>
        </el-col>
        <el-col :span="24">
          <el-divider></el-divider>
        </el-col>
      </el-row>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="conDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="configEnsure">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog :title="valuationTitle"
               :visible.sync="vaDialogVisible"
               width="40%"
               @close="handleCloseVa"
               :append-to-body="true"
               center>
      <el-row :gutter="5"
              type="flex"
              justify="center">
        <el-col :span="10">
          <el-input v-model="valuationValue"
                    v-if="valuationType === 'Input'"></el-input>
          <el-select v-model="valuationValue"
                     class="left-form"
                     filterable
                     clearable
                     placeholder="选择变量"
                     v-if="valuationType === 'Parameter'">
            <el-option v-for="(variable, index) in variableList"
                       :key="index"
                       :label="variable.label"
                       :value="variable.value">
            </el-option>
          </el-select>
        </el-col>
      </el-row>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="vaDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="valuationEnsure">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog title="选择参数或变量"
               :visible.sync="paDialogVisible"
               width="40%"
               @close="handleClosePa"
               :append-to-body="true"
               center>
      <el-row :gutter="5"
              type="flex"
              justify="center">
        <el-col :span="10">
          <el-cascader v-model="paramsValue"
                       :options="headerOption"
                       :props="{ expandTrigger: 'hover' }"
                       placeholder="选择参数或变量"
                       :show-all-levels="false"></el-cascader>
        </el-col>
      </el-row>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="paDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="paramsValueEnsure">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog title="选择参数或变量"
               :visible.sync="vaDialogVisible1"
               width="40%"
               @close="handleCloseVa1"
               :append-to-body="true"
               center>
      <el-row :gutter="5"
              type="flex"
              justify="center">
        <el-col :span="10">
          <el-cascader v-model="valuationValue1"
                       :options="headerOption"
                       :props="{ expandTrigger: 'hover' }"
                       placeholder="选择参数或变量"
                       :show-all-levels="false"></el-cascader>
        </el-col>
      </el-row>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="vaDialogVisible1 = false">取 消</el-button>
        <el-button type="primary"
                   @click="valuationValueEnsure">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      conDialogVisible: false,
      decisionLoading: false,
      vaDialogVisible: false,
      paDialogVisible: false,
      vaDialogVisible1: false,
      // 决策表数据
      decisionTableData: [
        {
          params: [
            {
              id: Math.random(),
              name: '无'
            }, {
              id: Math.random(),
              name: '无'
            }
          ],
          valuation: [
            {
              value: '无'
            }
          ]
        }, {
          params: [
            {
              id: Math.random(),
              name: '无'
            }, {
              id: Math.random(),
              name: '无'
            }
          ],
          valuation: [
            {
              value: '无'
            }
          ]
        }
      ],
      // 决策表变量、参数表头
      paramsHeader: [
        {
          header: '请选择参数或变量'
        }, {
          header: '请选择参数或变量'
        }
      ],
      // 决策表值表头
      valuationHeader: [
        {
          header: '是否通过'
        }
      ],
      valuaHeader: [
        {
          value: '1',
          type: 'operation',
          label: '添加赋值列'
        }, {
          value: '2',
          type: 'operation',
          label: '删除赋值列'
        }, {
          value: '3',
          type: 'data',
          label: '选择变量',
          children: [
            {
              value: '3.1',
              label: '年龄'
            }, {
              value: '3.2',
              label: '信用评分'
            }
          ]
        }
      ],
      // 表格数据点击选择
      paramTypeList: [
        {
          value: 'condition',
          icon: 'el-icon-s-tools',
          label: '配置条件'
        }, {
          value: 'clear',
          icon: 'el-icon-brush',
          label: '清空'
        }, {
          value: 'addRow',
          icon: 'el-icon-circle-plus-outline',
          label: '添加条件行'
        }, {
          value: 'deleteRow',
          icon: 'el-icon-remove-outline',
          label: '删除条件行'
        }
      ],
      valueList: [
        {
          value: 'pass',
          label: '通过'
        }, {
          value: 'refuse',
          label: '拒绝'
        }
        // {
        //   value: 'Input',
        //   label: '输入值'
        // }, {
        //   value: 'Parameter',
        //   label: '变量'
        // }, {
        //   value: 'clear',
        //   label: '清空'
        // }
      ],
      // 符号list
      symbolList: [
        {
          value: '>',
          label: '大于'
        }, {
          value: '>=',
          label: '大于等于'
        }, {
          value: '<',
          label: '小于'
        }, {
          value: '<=',
          label: '小于等于'
        }, {
          value: '==',
          label: '等于'
        }, {
          value: '===',
          label: '恒等于(不分大小写)'
        }, {
          value: '!==',
          label: '不等于'
        }, {
          value: '!===',
          label: '不恒等于(不分大小写)'
        }
      ],

      // 值类型list
      valueTypeList: [
        {
          value: 'Input',
          label: '输入值'
        }, {
          value: 'Parameter',
          label: '变量'
        }
      ],
      headerOption: [
        {
          value: '1',
          label: '选择变量',
          children: [
            {
              value: 'age',
              label: '年龄'
            }, {
              value: 'education',
              label: '学历'
            }, {
              value: 'score',
              label: '信用评分'
            }
          ]
        }
      ],
      // 变量参数list
      variableList: [
        {
          value: 'age',
          label: '年龄'
        }, {
          value: 'education',
          label: '学历'
        }, {
          value: 'score',
          label: '信用评分'
        }
      ],
      // 配置条件list
      configurationList: {
        dispute: '&&',
        ruelSet: [
          {
            symbol: [],
            valueType: '',
            value: ''
          }
        ]
      },
      // 点击配置数据的行列参数
      columnIndex: null,
      rowIndex: null,
      // 合并行参数
      spanArr: [],
      position: 0,
      valuationTitle: null,
      valuationType: null,
      valuationValue: null,
      // 表头右键菜单配置
      paramsContextMenuData: {
        // the contextmenu name(@1.4.1 updated)
        menuName: 'paramsHeader',
        // The coordinates of the display(菜单显示的位置)
        axis: {
          x: null,
          y: null
        },
        // Menu options (菜单选项)
        menulists: [{
          fnHandler: 'addColumn', // Binding events(绑定事件)
          icoName: 'el-icon-plus',
          btnName: '添加条件列' // The name of the menu option (菜单名称)
        }, {
          fnHandler: 'deleteColumn',
          icoName: 'el-icon-delete',
          btnName: '删除条件列'
        }]
      },
      valuationContextMenuData: {
        // the contextmenu name(@1.4.1 updated)
        menuName: 'valuationHeader',
        // The coordinates of the display(菜单显示的位置)
        axis: {
          x: null,
          y: null
        },
        // Menu options (菜单选项)
        menulists: [{
          fnHandler: 'addColumn', // Binding events(绑定事件)
          icoName: 'el-icon-plus',
          btnName: '添加赋值列' // The name of the menu option (菜单名称)
        }, {
          fnHandler: 'deleteColumn',
          icoName: 'el-icon-delete',
          btnName: '删除赋值列'
        }]
      },
      // 表头右键菜单索引
      paramsTransferIndex: null,
      valuationTransferIndex: null,
      // 表格右键菜单配置
      paramsValueContextMenuData: {
        // the contextmenu name(@1.4.1 updated)
        menuName: 'paramsValue',
        // The coordinates of the display(菜单显示的位置)
        axis: {
          x: null,
          y: null
        },
        // Menu options (菜单选项)
        menulists: [
          // {
          //   fnHandler: 'condition',
          //   icoName: 'el-icon-s-tools',
          //   btnName: '配置条件'
          // },
          {
            fnHandler: 'clear',
            icoName: 'el-icon-brush',
            btnName: '清空'
          }, {
            fnHandler: 'addRow', // Binding events(绑定事件)
            icoName: 'el-icon-plus',
            btnName: '添加条件行' // The name of the menu option (菜单名称)
          }, {
            fnHandler: 'deleteRow',
            icoName: 'el-icon-delete',
            btnName: '删除条件行'
          }
        ]
      },
      // 表格右键菜单索引
      paramsValueTransferIndex: null,
      // 表头参数选择
      paramsValue: [],
      paramsIndex: null,
      valuationValue1: [],
      valuationIndex: null
    }
  },
  methods: {
    // 表格背景色设置
    tableRowClassName ({ row, rowIndex }) {
      row.index = rowIndex
      if (rowIndex % 2 !== 0) {
        return 'warning-row'
      } else {
        return ''
      }
    },
    // 新增决策表
    addNewTable () {
      this.rowspan(this.decisionTableData)
    },
    // 配置条件框关闭回调事件
    handleCloseCon () {
      this.decisionLoading = false
      this.configurationList = {
        dispute: '&&',
        ruelSet: [
          {
            symbol: [],
            valueType: '',
            value: ''
          }
        ]
      }
    },
    // 决策表配置关闭回调框事件
    handleCloseCenter () {
      this.decisionLoading = false
      this.decisionTableData = [
        {
          params: [
            {
              id: Math.random(),
              name: '无'
            }, {
              id: Math.random(),
              name: '无'
            }
          ],
          valuation: [
            {
              value: '无'
            }
          ]
        }, {
          params: [
            {
              id: Math.random(),
              name: '无'
            }, {
              id: Math.random(),
              name: '无'
            }
          ],
          valuation: [
            {
              value: '无'
            }
          ]
        }
      ]
      this.paramsHeader = [
        {
          header: '请选择参数或变量'
        }, {
          header: '请选择参数或变量'

        }
      ]
      this.valuationHeader = [
        {
          header: '请选择赋值'
        }
      ]
    },
    // 赋值框关闭回调
    handleCloseVa () {
      this.decisionLoading = false
      this.valuationTitle = null
      this.valuationValue = null
      this.valuationType = null
    },
    // 条件表头关闭回调
    handleClosePa () {
      this.paramsValue = []
    },
    handleCloseVa1 () {
      this.valuationValue1 = []
    },
    // 决策表表头右击事件
    headerContextmenu (column, event) {
      console.log(column)
      this.paramsTransferIndex = column.index
      this.valuationTransferIndex = column.index
      event.preventDefault()
      var x = event.clientX
      var y = event.clientY
      if (column.type === 'params') {
        this.paramsContextMenuData.axis = {
          x, y
        }
      } else {
        this.valuationContextMenuData.axis = {
          x, y
        }
      }
    },
    // 决策表行右击事件
    rowContextmenu (row, column, event) {
      this.paramsValueTransferIndex = column.index
      this.rowIndex = row.index
      this.columnIndex = column.index
      event.preventDefault()
      var x = event.clientX
      var y = event.clientY
      if (column.type === 'params') {
        this.paramsValueContextMenuData.axis = {
          x, y
        }
      } else {
      }
    },
    // 表头点击事件
    headerClick (headerItem, headerIndex) {
      this.paDialogVisible = true
      this.paramsIndex = headerIndex
    },
    valuationHeaderClick (headerItem, headerIndex) {
      this.vaDialogVisible1 = true
      this.valuationIndex = headerIndex
      // this.paramsIndex = headerIndex
    },
    // 添加列
    addParamsColumn (headerItem, headerIndex) {
      // 添加列
      this.paramsHeader.push(
        {
          header: '请选择参数或变量'
        }
      )
      // 同时添加行
      this.paramsHeader.forEach((header, headerIdnex) => {
        this.decisionTableData.forEach((deci, deciIndex) => {
          deci.params.push({
            id: Math.random(),
            name: '无'
          })
        })
      })
    },
    addValuationColumn (headerItem, headerIndex) {
      // 添加列
      this.valuationHeader.push(
        {
          header: '请选择赋值'
        }
      )
      // 同时添加行
      this.valuationHeader.forEach((header, headerIdnex) => {
        this.decisionTableData.forEach((deci, deciIndex) => {
          deci.valuation.push({
            value: '无'
          })
        })
      })
    },
    // 删除列
    deleteParamsColumn (headerItem, headerIndex) {
      console.log(headerItem)
      console.log(headerIndex)
      // 删除列
      var indexHeader = this.paramsHeader.indexOf(headerItem)
      if (indexHeader !== -1) {
        this.paramsHeader.splice(indexHeader, 1)
      }
      // 同时删除行
      var deleteIndex = this.decisionTableData.indexOf(headerItem)
      if (deleteIndex !== -1) {
        this.decisionTableData.splice(deleteIndex, 1)
      }
    },
    deleteValuationColumn (headerItem, headerIndex) {
      // 删除列
      var indexHeader = this.valuationHeader.indexOf(headerItem)
      if (indexHeader !== -1) {
        this.valuationHeader.splice(indexHeader, 1)
      }
      // 同时删除行
      var deleteIndex = this.decisionTableData.indexOf(headerItem)
      if (deleteIndex !== -1) {
        this.decisionTableData.splice(deleteIndex, 1)
      }
    },
    // 添加行
    addParamsRow (headerItem, headerIndex) {
      this.decisionLoading = true
      var paramsData = []
      var valuation = []
      if (headerIndex === 0) {
        this.paramsHeader.forEach(el => {
          paramsData.push({
            id: Math.random(),
            name: '无'
          })
        })
        this.valuationHeader.forEach(va => {
          valuation.push({
            value: '无'
          })
        })
        this.decisionTableData.push({
          params: paramsData,
          valuation: valuation
        })
      } else {
        var id = null
        var numIn = headerIndex - 1
        if (numIn > 0) {
          for (var i = 0; i < this.paramsHeader.length; i++) {
            if (i <= numIn) {
              for (var n = 0; n <= numIn; n++) {
                id = this.decisionTableData[this.rowIndex].params[n].id
                if (i === n) {
                  paramsData.push({
                    id: id,
                    name: '无'
                  })
                }
              }
            } else {
              paramsData.push({
                id: Math.random(),
                name: '无',
                value: '无'
              })
            }
          }
          this.valuationHeader.forEach(va => {
            valuation.push({
              value: '无'
            })
          })
          this.decisionTableData.splice(this.rowIndex, 0, {
            params: paramsData,
            valuation: valuation
          })
        } else {
          id = this.decisionTableData[this.rowIndex].params[numIn].id
          this.paramsHeader.forEach((el, index) => {
            if (numIn === index) {
              paramsData.push({
                id: id,
                name: '无',
                value: '无'
              })
            } else {
              paramsData.push({
                id: Math.random(),
                name: '无',
                value: '无'
              })
            }
          })
          this.valuationHeader.forEach(va => {
            valuation.push({
              value: '无'
            })
          })
          this.decisionTableData.splice(this.rowIndex, 0, {
            params: paramsData,
            valuation: valuation
          })
        }
      }
      this.rowspan(this.decisionTableData)
      this.decisionLoading = false
    },
    // 删除条件行
    deleteParamsRow (headerItem, headerIndex) {
      this.decisionLoading = true
      // 删除行
      this.decisionTableData.splice(this.rowIndex, 1)
      this.rowspan(this.decisionTableData)
      this.decisionLoading = false
    },
    // 清空行数据
    clearParamsRow (headerItem, headerIndex) {
      this.decisionLoading = true
      // 清控单元格数据
      this.decisionTableData[this.rowIndex].params[headerIndex].name = '无'
      this.rowspan(this.decisionTableData)
      this.decisionLoading = false
    },
    // 表格行点击事件
    cellClick (row, column, cell, event) {
      this.rowIndex = row.index
      this.columnIndex = column.index
      if (column.type === 'params') {
        this.conDialogVisible = true
      }
    },
    // 表格参数选择事件
    paramAcTyClick (item, dropdown, droIndex, headerIndex, scopeIndex) {
      this.decisionLoading = true
      if (dropdown.value === 'clear') {
        // 清控单元格数据
        var clearIndex = this.decisionTableData.indexOf(item)
        this.decisionTableData[clearIndex].params[headerIndex].name = '无'
        this.decisionLoading = false
      } else if (dropdown.value === 'addRow') {
        /* 添加单元格数据 */
        // 判断多少列添加行数数据
        var paramsData = []
        if (headerIndex === 0) {
          this.paramsHeader.forEach(el => {
            paramsData.push({
              id: Math.random(),
              name: '无',
              value: '无'
            })
          })
          this.decisionTableData.push({
            params: paramsData
          })
        } else {
          var id = null
          var numIn = headerIndex - 1
          console.log(scopeIndex)
          console.log(this.decisionTableData[scopeIndex].params[numIn].id)
          id = this.decisionTableData[scopeIndex].params[numIn].id
          this.paramsHeader.forEach((el, index) => {
            if (numIn === index) {
              paramsData.push({
                id: id,
                name: '无',
                value: '无'
              })
            } else {
              paramsData.push({
                id: Math.random(),
                name: '无',
                value: '无'
              })
            }
          })
          this.decisionTableData.splice(scopeIndex, 0, {
            params: paramsData
          })
        }

        // console.log(this.decisionTableData)
        this.rowspan(this.decisionTableData)
        this.decisionLoading = false
      } else if (dropdown.value === 'deleteRow') {
        // 删除行
        var deleteIndex = this.decisionTableData.indexOf(item)
        if (deleteIndex !== -1) {
          this.decisionTableData.splice(deleteIndex, 1)
        }
        // 同时删除列
        var indexHeader = this.paramsHeader.indexOf(item)
        if (indexHeader !== -1) {
          this.paramsHeader.splice(indexHeader, 1)
        }
        this.decisionLoading = false
      } else {
        this.columnIndex = headerIndex
        this.rowIndex = scopeIndex
        this.conDialogVisible = true
      }
    },
    // 获得数据相同的行数
    rowspan (data) {
      this.spanArr = []
      var header = this.paramsHeader
      var array = []
      for (var h = 0; h < header.length; h++) {
        data.forEach(el1 => {
          el1.params.forEach((el2, index) => {
            if (h === index) {
              array.push(el2.id)
            }
          })
        })
      }

      // 对类型去重
      var array2 = []
      for (var j = 0; j < array.length; j++) {
        if (array2.indexOf(array[j]) === -1) {
          array2.push(array[j])
        }
      }
      var tmpRowColum = []
      for (var a = 0; a < header.length; a++) {
        var rowNum = 0
        var columNum = 0
        columNum = a
        for (var d = 0; d < data.length; d++) {
          rowNum = d
          tmpRowColum.push({
            rowNum: rowNum,
            columNum: columNum
          })
        }
      }
      var tmpMerge = []
      for (var ar = 0; ar < array2.length; ar++) {
        var mergeNum = 0
        for (var da = 0; da < data.length; da++) {
          data[da].params.forEach(el => {
            if (array2[ar] === el.id) {
              mergeNum = mergeNum + 1
            }
          })
        }
        if (mergeNum > 1) {
          tmpMerge.push(mergeNum)
          for (var n = 0; n < mergeNum - 1; n++) {
            tmpMerge.push(0)
          }
        } else {
          tmpMerge.push(mergeNum)
        }
      }
      tmpRowColum.forEach((r, rin) => {
        tmpMerge.forEach((m, min) => {
          if (rin === min) {
            tmpRowColum[rin].mergeNum = m
          }
        })
      })
      console.log(tmpRowColum)
      this.spanArr = tmpRowColum
    },
    objectSpanMethod ({ row, column, rowIndex, columnIndex }) {
      // 表格合并行
      for (var i = 0; i < this.spanArr.length; i++) {
        var rowNum = this.spanArr[i].rowNum
        var columNum = this.spanArr[i].columNum
        var mergeNum = this.spanArr[i].mergeNum
        if (columnIndex === columNum) {
          if (rowIndex === rowNum) {
            return {
              rowspan: mergeNum,
              colspan: 1
            }
          } else {
            if (rowIndex !== 0 && rowIndex <= (rowNum + mergeNum - 1)) {
              return {
                rowspan: 0,
                colspan: 0
              }
            }
          }
        }
      }

      // })
    },
    // 配置条件确定
    configEnsure () {
      this.conDialogVisible = false
      console.log(this.configurationList)
      // var valueText = '测试'
      var connect = null
      if (this.configurationList.dispute === '&&') {
        connect = '并且'
      } else {
        connect = '或者'
      }
      var ruleText = ''
      if (this.configurationList.ruelSet.length > 0) {
        this.configurationList.ruelSet.forEach((el, index) => {
          this.symbolList.forEach(symbol => {
            if (symbol.value === el.symbol) {
              if (this.configurationList.ruelSet.length - 1 === index) {
                ruleText += symbol.label + el.value
              } else {
                ruleText += symbol.label + el.value + connect
              }
            }
          })
        })
      }

      this.decisionTableData[this.rowIndex].params[this.columnIndex].name = ruleText
    },
    // 配置条件规则处理
    transforRule () {

    },
    // 添加新规则区间
    addNewRuleSet (index) {
      this.configurationList.ruelSet.push(
        {
          symbol: [],
          valueType: '',
          value: ''
        }
      )
    },
    // 删除规则区间
    deleteRuleSet (item, num) {
      if (this.configurationList.ruelSet.length < 2) {
        this.$message.error('最后一条禁止删除')
        return
      }
      var index = this.configurationList.ruelSet.indexOf(item)
      if (index !== -1) {
        this.configurationList.ruelSet.splice(index, 1)
      }
    },
    // 表头赋值选择事件
    valueChange (index, item) {
      var header = null
      this.valuaHeader.forEach(el => {
        if (this.valuationHeader[index].headerChange.length > 0) {
          if (this.valuationHeader[index].headerChange[0] === el.value) {
            if (el.type === 'operation') {
              // 添加列
              if (el.value === '1') {
                this.valuationHeader.push(
                  {
                    header: '请选择赋值'
                  }
                )
                this.valuationHeader.forEach((header, headerIdnex) => {
                  this.decisionTableData.forEach((deci, deciIndex) => {
                    deci.params.push({
                      id: Math.random(),
                      name: '无',
                      value: '无'
                    })
                  })
                })
                // 清空选择，关闭选择框
                this.valuationHeader.forEach(chen => {
                  chen.headerChange = []
                })
              } else {
                // 删除列
                var indexHeader = this.valuationHeader.indexOf(item)
                if (indexHeader !== -1) {
                  this.valuationHeader.splice(indexHeader, 1)
                }
                // 同时删除行
                // var deleteIndex = this.decisionTableData.indexOf(item)
                // if (deleteIndex !== -1) {
                //   this.decisionTableData.splice(deleteIndex, 1)
                // }
                // 清空选择，关闭选择框
                this.valuationHeader.forEach(chen => {
                  chen.headerChange = []
                })
              }
            }
            // 表头数据选择
            if (this.valuationHeader[index].headerChange.length > 1) {
              if (el.children !== null && el.children !== undefined) {
                el.children.forEach(ch => {
                  if (this.valuationHeader[index].headerChange[1] === ch.value) {
                    header = ch.label
                  }
                })
                this.valuationHeader[index].header = header
              }
            }
          }
        }
      })
    },
    // 表格赋值选择事件
    valueAcTyClick (item, dropdown, droIndex, headerIndex, scopeIndex) {
      console.log(dropdown)
      this.decisionLoading = true
      this.columnIndex = headerIndex
      this.rowIndex = scopeIndex
      // if (dropdown.value === 'Input') {
      //   this.vaDialogVisible = true
      //   this.valuationType = dropdown.value
      //   this.valuationTitle = '输入值'
      // } else if (dropdown.value === 'Parameter') {
      //   this.vaDialogVisible = true
      //   this.valuationType = dropdown.value
      //   this.valuationTitle = '选择变量'
      // }
      if (dropdown.value === 'pass') {
        this.decisionTableData[this.rowIndex].valuation[this.columnIndex].value = '通过'
        this.decisionLoading = false
      } else if (dropdown.value === 'refuse') {
        this.decisionTableData[this.rowIndex].valuation[this.columnIndex].value = '拒绝'
        this.decisionLoading = false
      } else {
        // 清控单元格数据
        var clearIndex = this.decisionTableData.indexOf(item)
        this.decisionTableData[clearIndex].valuation[headerIndex].value = '无'
        this.decisionLoading = false
      }
    },
    // 赋值确定
    valuationEnsure () {
      // valuationValue
      var ruleValue = null
      if (this.valuationType === 'Input') {
        ruleValue = this.valuationValue
      } else if (this.valuationType === 'Parameter') {
        this.variableList.forEach(el => {
          if (el.value === this.valuationValue) {
            ruleValue = el.label
          }
        })
      }
      this.decisionTableData[this.rowIndex].valuation[this.columnIndex].value = ruleValue
      this.vaDialogVisible = false
    },
    // 条件列赋值
    paramsValueEnsure () {
      var header = null
      this.headerOption.forEach((el, index) => {
        if (el.children !== null && el.children !== undefined) {
          el.children.forEach((ch, chIndex) => {
            if (ch.value === this.paramsValue[1]) {
              header = ch.label
            }
          })
        }
      })
      this.paramsHeader[this.paramsIndex].header = header
      this.paDialogVisible = false
    },
    valuationValueEnsure () {
      var header = null
      this.headerOption.forEach((el, index) => {
        if (el.children !== null && el.children !== undefined) {
          el.children.forEach((ch, chIndex) => {
            if (ch.value === this.valuationValue1[1]) {
              header = ch.label
            }
          })
        }
      })
      this.valuationHeader[this.valuationIndex].header = header
      this.vaDialogVisible1 = false
    }

  },
  mounted () {

  }
}
</script>

<style lang="scss">
.param-type {
  cursor: pointer;
  text-align: center;
  margin-top: 10px;
  font-size: 16px;
  &:hover {
    color: #409eff;
  }
}
.params-class {
  cursor: pointer;
  font-size: 18px;
  color: #000;
}
.value-header {
  cursor: pointer;
  color: #ff9800;
}
.context-menu-list[data-v-1931f86e] {
  position: relative;
  background: #fff;
  text-decoration: none;
  list-style: none;
  margin: 3px 0;
  padding: 3px;
  font-size: 16px;
  color: #000;
  letter-spacing: 1.5px;
  font-weight: bold;
}
</style>
