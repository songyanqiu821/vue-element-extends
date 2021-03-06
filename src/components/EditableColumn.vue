<template>
  <el-table-column v-if="type === 'selection' || group" v-bind="attrs">
    <slot></slot>
  </el-table-column>
  <el-table-column v-else-if="type === 'index'" v-bind="attrs">
    <template slot="header" slot-scope="scope">
      <slot name="head" v-bind="getHeadScope(scope)">#</slot>
    </template>
    <slot></slot>
  </el-table-column>
  <el-table-column v-else-if="type === 'expand'" v-bind="attrs">
    <template slot="header" slot-scope="scope">
      <slot name="head" v-bind="getHeadScope(scope)"></slot>
    </template>
    <template slot-scope="scope">
      <slot v-bind="getRowScope(scope)"></slot>
    </template>
  </el-table-column>
  <el-table-column v-else-if="editRender" v-bind="attrs">
    <template slot="header" slot-scope="scope">
      <slot name="head" v-bind="getHeadScope(scope)">
        <i v-if="checkRequired(scope)" class="editable-required-icon"></i>
        <i v-if="checkIcon(scope)" class="el-icon-edit-outline editable-header-icon"></i>
        {{ scope.column.label }}
      </slot>
    </template>
    <template slot-scope="scope">
      <template v-if="renderOpts.type === 'visible'">
        <slot name="edit" v-bind="getRowScope(scope)">
          <template v-if="compName === 'ElSelect'">
            <el-select v-model="scope.row.data[scope.column.property]" v-bind="getRendAttrs(scope)" v-on="getRendEvents(scope)">
              <el-option v-for="(item, index) in renderOpts.options" :key="index" :value="item[renderOpts.optionProps.value]" :label="item[renderOpts.optionProps.label]" v-bind="item.attrs"></el-option>
            </el-select>
          </template>
          <template v-else-if="comps.includes(compName)">
            <component :is="compName" v-model="scope.row.data[scope.column.property]" v-bind="getRendAttrs(scope)" v-on="getRendEvents(scope)"></component>
          </template>
          <template v-else>
            <el-input v-model="scope.row.data[scope.column.property]" v-bind="getRendAttrs(scope)" v-on="getRendEvents(scope)"></el-input>
          </template>
        </slot>
      </template>
      <template v-else>
        <template v-if="scope.row.editActive && (scope.row.config.mode === 'row' ? scope.row.editActive : scope.row.editActive === scope.column.property)">
          <slot name="edit" v-bind="getRowScope(scope)">
            <template v-if="compName === 'ElSelect'">
              <el-select v-model="scope.row.data[scope.column.property]" v-bind="getRendAttrs(scope)" v-on="getRendEvents(scope)">
                <el-option v-for="(item, index) in renderOpts.options" :key="index" :value="item[renderOpts.optionProps.value]" :label="item[renderOpts.optionProps.label]" v-bind="item.attrs"></el-option>
              </el-select>
            </template>
            <template v-else-if="comps.includes(compName)">
              <component :is="compName" v-model="scope.row.data[scope.column.property]" v-bind="getRendAttrs(scope)" v-on="getRendEvents(scope)"></component>
            </template>
            <template v-else>
              <el-input v-model="scope.row.data[scope.column.property]" v-bind="getRendAttrs(scope)" v-on="getRendEvents(scope)"></el-input>
            </template>
          </slot>
        </template>
        <template v-else>
          <slot v-bind="getRowScope(scope)">{{ formatColumnLabel(scope) }}</slot>
        </template>
      </template>
      <template v-if="scope.row.validActive && scope.row.validActive === scope.column.property">
        <template v-if="scope.row.config.useDefaultValidTip">
          <slot name="valid" v-bind="{rule: scope.row.validRule || {}, $index: scope.$index, row: scope.row.data, column: scope.column, store: scope.store, editRender: renderOpts, _self: scope._self, _row: scope.row}">
            <div class="editable-valid_error">
              <span class="valid-message">{{ scope.row.validRule ? scope.row.validRule.message : '' }}</span>
            </div>
          </slot>
        </template>
        <template v-else-if="!scope.row.config.validTooltip.disabled">
          <el-tooltip :value="scope.row.showValidMsg" v-bind="scope.row.config.validTooltip">
            <div class="editable-valid_wrapper"></div>
            <div slot="content">
              <slot name="valid" v-bind="{rule: scope.row.validRule || {}, $index: scope.$index, row: scope.row.data, column: scope.column, store: scope.store, editRender: renderOpts, _self: scope._self, _row: scope.row}">{{ scope.row.validRule ? scope.row.validRule.message : '' }}</slot>
            </div>
          </el-tooltip>
        </template>
      </template>
    </template>
  </el-table-column>
  <el-table-column v-else v-bind="attrs">
    <template slot-scope="scope">
      <slot v-bind="getRowScope(scope)">{{ formatColumnLabel(scope) }}</slot>
    </template>
  </el-table-column>
</template>

<script>
import XEUtils from 'xe-utils'
import {
  TableColumn,
  Autocomplete,
  Input,
  Select,
  Option,
  Cascader,
  TimeSelect,
  TimePicker,
  DatePicker,
  InputNumber,
  Switch,
  Rate,
  ColorPicker,
  Slider,
  Tooltip
} from 'element-ui'

export default {
  name: 'ElEditableColumn',
  props: {
    group: Boolean,
    editRender: Object,

    /**
     * 还原 ElTableColumn 所有属性
     */
    index: [Number, Function],
    type: String,
    label: String,
    columnKey: String,
    prop: String,
    width: String,
    minWidth: String,
    fixed: [Boolean, String],
    sortable: [Boolean, String],
    sortMethod: Function,
    sortBy: [String, Array, Function],
    sortOrders: Array,
    resizable: { type: Boolean, default: true },
    formatter: Function,
    showOverflowTooltip: Boolean,
    align: { type: String, default: 'left' },
    headerAlign: String,
    className: { type: String, default: '' },
    labelClassName: String,
    selectable: Function,
    reserveSelection: Boolean,
    filters: Array,
    filterPlacement: String,
    filterMultiple: { type: Boolean, default: true },
    filterMethod: Function,
    filteredValue: Array
  },
  components: {
    ElTableColumn: TableColumn,
    ElAutocomplete: Autocomplete,
    ElInput: Input,
    ElSelect: Select,
    ElOption: Option,
    ElCascader: Cascader,
    ElTimeSelect: TimeSelect,
    ElTimePicker: TimePicker,
    ElDatePicker: DatePicker,
    ElInputNumber: InputNumber,
    ElSwitch: Switch,
    ElRate: Rate,
    ElColorPicker: ColorPicker,
    ElSlider: Slider,
    ElTooltip: Tooltip
  },
  inject: [
    '$editable'
  ],
  data () {
    return {
      // 支持内置组件
      comps: [
        'ElAutocomplete',
        'ElInput',
        'ElSelect',
        'ElCascader',
        'ElTimeSelect',
        'ElTimePicker',
        'ElDatePicker',
        'ElInputNumber',
        'ElSwitch',
        'ElRate',
        'ElColorPicker',
        'ElSlider'
      ],
      isDefaultInput: ['ElInput', 'ElInputNumber'].includes(this.editRender ? this.editRender.name : null)
    }
  },
  computed: {
    renderOpts () {
      let editRender = this.editRender
      return Object.assign({
        name: editRender ? 'ElInput' : null,
        type: 'default',
        autofocus: editRender && this.isDefaultInput,
        optionProps: {
          value: 'value',
          label: 'label'
        }
      }, editRender)
    },
    isReadonly () {
      return !this.editRender
    },
    compName () {
      return this.renderOpts.name
    },
    attrs () {
      let sortBy
      let className = this.className ? ` ${this.className}` : ''
      let editTypeClass = this.isReadonly ? ' editable-col_readonly' : ' editable-col_edit'
      let autofocusClass = this.renderOpts.autofocus ? ' autofocus' : ''
      if (XEUtils.isFunction(this.sortBy)) {
        sortBy = this.sortBy
      } else if (XEUtils.isString(this.sortBy)) {
        sortBy = `data.${this.sortBy}`
      } else if (XEUtils.isArray(this.sortBy)) {
        sortBy = this.sortBy.map(name => `data.${name}`)
      } else if (this.prop) {
        sortBy = `data.${this.prop}`
      }
      return {
        index: this.index,
        type: this.type,
        label: this.label,
        columnKey: this.columnKey,
        prop: this.prop,
        width: this.width,
        minWidth: this.minWidth,
        fixed: this.fixed,
        sortable: this.sortable,
        sortMethod: this.sortMethod ? this.sortMethodEvent : this.sortMethod,
        sortBy,
        sortOrders: this.sortOrders,
        resizable: this.resizable,
        showOverflowTooltip: this.showOverflowTooltip,
        align: this.align,
        headerAlign: this.headerAlign,
        className: `editable-column${editTypeClass}${autofocusClass}${className}`,
        labelClassName: this.labelClassName,
        selectable: this.selectable ? this.selectableEvent : this.selectable,
        reserveSelection: this.reserveSelection,
        filters: this.filters,
        filterPlacement: this.filterPlacement,
        filterMultiple: this.filterMultiple,
        filterMethod: this.filterMethod ? this.filterMethodEvent : this.filterMethod,
        filteredValue: this.filteredValue
      }
    }
  },
  methods: {
    getHeadScope (scope) {
      return {
        $index: scope.$index,
        column: scope.column,
        store: scope.store,
        editRender: this.renderOpts,
        _self: scope._self
      }
    },
    getRowScope (scope) {
      return {
        $index: scope.$index,
        row: scope.row.data,
        column: scope.column,
        store: scope.store,
        editRender: this.renderOpts,
        _self: scope._self,
        _row: scope.row
      }
    },
    getRowIdentity (row, column) {
      return XEUtils.get(row.data, column.property)
    },
    getRendAttrs (scope) {
      let size = scope.row.config.size
      return Object.assign({ size }, this.renderOpts.attrs)
    },
    getRendEvents ({ $index, row, column, store }) {
      let scope = { $index, row: row.data, column, store, editRender: this.renderOpts, _row: row }
      let defEvent = { [this.isDefaultInput ? 'input' : 'change']: val => this.$editable.updateStatus(scope) }
      if (this.renderOpts.events) {
        return Object.assign(defEvent, XEUtils.objectMap(this.renderOpts.events, cb => function () {
          cb.apply(null, [scope].concat(Array.from(arguments)))
        }))
      }
      return defEvent
    },
    formatColumnLabel (scope) {
      if (this.formatter) {
        return this.formatter(scope.row.data, scope.column, this.getRowIdentity(scope.row, scope.column), scope.$index)
      }
      switch (this.compName) {
        case 'ElSelect':
          return this.getSelectLabel(scope)
        case 'ElCascader':
          return this.getCascaderLabel(scope)
        case 'ElTimePicker':
          return this.getTimePickerLabel(scope)
        case 'ElDatePicker':
          return this.getDatePickerLabel(scope)
      }
      return this.getRowIdentity(scope.row, scope.column)
    },
    getSelectLabel ({ row, column }) {
      let labelProp = this.renderOpts.optionProps.label
      let valueProp = this.renderOpts.optionProps.value
      let value = this.getRowIdentity(row, column)
      let selectItem = this.renderOpts.options.find(item => item[valueProp] === value)
      return selectItem ? selectItem[labelProp] : null
    },
    matchCascaderData (values, index, list, labels) {
      let val = values[index]
      if (list && values.length > index) {
        list.forEach(item => {
          if (item.value === val) {
            labels.push(item.label)
            this.matchCascaderData(values, ++index, item.children, labels)
          }
        })
      }
    },
    getCascaderLabel ({ row, column }) {
      let values = this.getRowIdentity(row, column) || []
      let labels = []
      let attrs = this.renderOpts.attrs || {}
      this.matchCascaderData(values, 0, attrs.options || [], labels)
      return labels.join(` ${attrs.separator || '/'} `)
    },
    getTimePickerLabel ({ row, column }) {
      let value = this.getRowIdentity(row, column)
      let attrs = this.renderOpts.attrs || {}
      return XEUtils.toDateString(value, attrs.format || 'hh:mm:ss')
    },
    getDatePickerLabel ({ row, column }) {
      let value = this.getRowIdentity(row, column)
      let attrs = this.renderOpts.attrs || {}
      switch (attrs.type) {
        case 'week':
          return this.getFormatDate(value, attrs, 'yyyywWW')
        case 'month':
          return this.getFormatDate(value, attrs, 'yyyy-MM')
        case 'year':
          return this.getFormatDate(value, attrs, 'yyyy')
        case 'dates':
          return this.getFormatDates(value, attrs, ', ', 'yyyy-MM-dd')
        case 'daterange':
          return this.getFormatDates(value, attrs, ` ${attrs.rangeSeparator || '-'} `, 'yyyy-MM-dd')
        case 'datetimerange':
          return this.getFormatDates(value, attrs, ` ${attrs.rangeSeparator || '-'} `, 'yyyy-MM-dd HH:ss:mm')
      }
      return this.getFormatDate(value, attrs, 'yyyy-MM-dd')
    },
    getFormatDate (value, attrs, defaultFormat) {
      return XEUtils.toDateString(value, attrs.format || defaultFormat)
    },
    getFormatDates (values, attrs, separator, defaultFormat) {
      return XEUtils.toArray(values).map(date => this.getFormatDate(date, attrs, defaultFormat)).join(separator)
    },
    checkRequired ({ column, store }) {
      if (column.property && this.$editable && this.$editable.editRules) {
        let rules = this.$editable.editRules[column.property]
        if (rules) {
          return rules.some(rule => rule.required === true)
        }
      }
      return false
    },
    checkIcon ({ column, store }) {
      return column.property && this.$editable && this.$editable.editConfig ? !(this.$editable.editConfig.showIcon === false) : true
    },
    sortByEvent (row, index) {
      return this.sortBy(row.data, index)
    },
    sortMethodEvent (v1, v2) {
      return this.sortMethod(v1.data, v2.data)
    },
    selectableEvent (row, index) {
      return this.selectable(row.data, index)
    },
    filterMethodEvent (value, row, column) {
      return this.filterMethod(value, row.data, column)
    }
  }
}
</script>

<style>
.editable .editable-required-icon:before {
  content: "*";
  color: #f56c6c;
}
.editable.el-table--mini .editable-column {
  height: 42px;
}
.editable.el-table--small .editable-column {
  height: 48px;
}
.editable .editable-column,
.editable.el-table--medium .editable-column {
  height: 62px;
  padding: 0;
}
.editable .editable-column.editable-col_dirty {
  position: relative;
}
.editable .editable-column.editable-col_dirty:before {
  content: '';
  top: -5px;
  left: -5px;
  position: absolute;
  border: 5px solid;
  border-color: transparent #f56c6c transparent transparent;
  -webkit-transform: rotate(45deg);
  transform: rotate(45deg);
}
.editable .editable-column .cell > .edit-input,
.editable .editable-column .cell > .el-cascader,
.editable .editable-column .cell > .el-autocomplete,
.editable .editable-column .cell > .el-input-number,
.editable .editable-column .cell > .el-date-editor,
.editable .editable-column .cell > .el-select {
  width: 100%;
}
.editable .editable-column.editable-col_edit.editable-col_active .cell {
  line-height: inherit;
  overflow: inherit;
  position: relative;
}
.editable.editable_click .editable-column.editable-col_edit,
.editable.editable_dblclick .editable-column.editable-col_edit {
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}
.editable .editable-column.valid-error .el-input__inner,
.editable .editable-column.valid-error .el-input__inner:focus,
.editable .editable-column.valid-error .el-textarea__inner,
.editable .editable-column.valid-error .el-textarea__inner:focus {
  border-color: #f56c6c;
}
/* custom tip */
.editable .editable-column .editable-valid_error {
  display: none;
}
.editable .editable-column.valid-error .el-input__inner,
.editable .editable-column.valid-error .el-input__inner:focus,
.editable .editable-column.valid-error .el-textarea__inner,
.editable .editable-column.valid-error .el-textarea__inner:focus {
  border-color: #f56c6c;
}
.editable .el-table__row:last-child .editable-column.valid-error .editable-valid_error {
  top: auto;
  bottom: -moz-calc(100% + 10px);
  bottom: -webkit-calc(100% + 10px);
  bottom: calc(100% + 10px);
}
.editable .el-table__row:last-child .editable-column.valid-error .editable-valid_error:before {
  top: auto;
  bottom: -12px;
  border-color: #f56c6c transparent transparent transparent;
}
.editable .editable-column.valid-error .editable-valid_error,
.editable .el-table__row:first-child .editable-column.valid-error .editable-valid_error {
  display: block;
  position: absolute;
  top: -moz-calc(100% + 10px);
  top: -webkit-calc(100% + 10px);
  top: calc(100% + 10px);
  left: 10px;
  bottom: auto;
  z-index: 9;
  min-width: 300px;
}
.editable .editable-column.valid-error .editable-valid_error>.valid-message,
.editable .el-table__row:first-child .editable-column.valid-error .editable-valid_error>.valid-message {
  display: inline-block;
  padding: 8px 10px;
  line-height: 1.5;
  color: #fff;
  background-color: #f56c6c;
  border-radius: 4px;
  font-size: 12px;
  word-wrap: break-word;
}
.editable .editable-column .editable-valid_error:before,
.editable .el-table__row:first-child .editable-column.valid-error .editable-valid_error:before {
  content: "";
  position: absolute;
  top: -12px;
  left: 20px;
  bottom: auto;
  border: 6px solid;
  border-color: transparent transparent #f56c6c transparent;
}
/* el tip */
.editable .editable-column .editable-valid_wrapper {
  display: block;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  position: absolute;
  z-index: -1;
}
.el-tooltip__popper.editable-valid_tooltip {
  background: #f56c6c;
}
.el-tooltip__popper.editable-valid_tooltip[x-placement^=top] .popper__arrow,
.el-tooltip__popper.editable-valid_tooltip[x-placement^=top] .popper__arrow::after {
  border-top-color: #f56c6c;
}
.el-tooltip__popper.editable-valid_tooltip[x-placement^=bottom] .popper__arrow,
.el-tooltip__popper.editable-valid_tooltip[x-placement^=bottom] .popper__arrow::after {
  border-bottom-color: #f56c6c;
}
.el-tooltip__popper.editable-valid_tooltip[x-placement^=left] .popper__arrow,
.el-tooltip__popper.editable-valid_tooltip[x-placement^=left] .popper__arrow::after {
  border-left-color: #f56c6c;
}
.el-tooltip__popper.editable-valid_tooltip[x-placement^=right] .popper__arrow,
.el-tooltip__popper.editable-valid_tooltip[x-placement^=right] .popper__arrow::after {
  border-right-color: #f56c6c;
}
</style>
