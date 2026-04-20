---
trigger: always_on
description: handleSearch() {
---

# SCM前端项目输入控件库规则

## 1. 基础输入控件

### 1.1 普通文本输入框
```vue
<!-- 无需import，Element UI原生组件 -->
<el-input
  v-model.trim="dataJson.tempJson.name"
  clearable
  placeholder="请输入"
  :maxlength="dataJson.inputSettings.maxLength.name"
  @keyup.enter.native="handleSearch"
/>
```
- **Import**: 无需import (Element UI)
- **绑定字段**: `dataJson.searchForm.*` 或 `dataJson.tempJson.*`
- **校验**: 通过 `el-form-item` 的 `prop` 属性
- **常用属性**: `clearable`, `placeholder`, `maxlength`, `@keyup.enter.native`

### 1.2 多行文本输入框
```vue
<!-- 无需import，Element UI原生组件 -->
<el-input
  v-model.trim="dataJson.tempJson.remark"
  type="textarea"
  clearable
  show-word-limit
  placeholder="请输入"
  :maxlength="500"
/>
```
- **Import**: 无需import (Element UI)
- **特殊属性**: `type="textarea"`, `show-word-limit`

## 2. 数字/金额输入控件

### 2.1 数字输入组件
```vue
<!-- Import -->
import numeric from '@/components/40_input/numeric'

<!-- 使用 -->
<numeric
  v-model="dataJson.tempJson.amount"
  :positive-percentage="true"
  :decimal-places="4"
  :minimum-value="0"
  :maximum-value="10000000000"
  style="width: 100%"
  @change.native="handleAmountChange"
/>
```
- **Import**: `import numeric from '@/components/40_input/numeric'`
- **Components注册**: `components: { numeric }`
- **主要属性**:
  - `positive-percentage`: 正百分比模式
  - `percentage`: 百分比模式
  - `decimal-places`: 小数位数
  - `currency-symbol`: 货币符号 (默认¥)
  - `minimum-value`: 最小值
  - `maximum-value`: 最大值

## 3. 字典选择控件

### 3.1 字典下拉选择框
```vue
<!-- Import -->
import SelectDict from '@/components/00_dict/select/SelectDict.vue'

<!-- 使用 -->
<select-dict
  v-model="dataJson.tempJson.type"
  :para="CONSTANTS.DICT_B_PROJECT_TYPE"
  init-placeholder="请选择"
  style="width:100%"
  @change="handleTypeChange"
/>
```
- **Import**: `import SelectDict from '@/components/00_dict/select/SelectDict.vue'`
- **Components注册**: `components: { SelectDict }`
- **必需属性**: `:para` (字典常量)
- **可选属性**: `init-placeholder`, `disabled`, `filter-para`

### 3.2 字典多选下拉框
```vue
<!-- Import -->
import SelectDicts from '@/components/00_dict/select/SelectDicts.vue'

<!-- 使用 -->
<select-dicts
  v-model="dataJson.searchForm.status_list"
  :para="CONSTANTS.DICT_B_PROJECT_STATUS"
  init-placeholder="请选择单据状态"
  :disabled="false"
/>
```
- **Import**: `import SelectDicts from '@/components/00_dict/select/SelectDicts.vue'`
- **Components注册**: `components: { SelectDicts }`
- **返回值**: 数组类型

### 3.3 字典单选按钮组
```vue
<!-- Import -->
import RadioDict from '@/components/00_dict/redio'

<!-- 使用 -->
<radio-dict
  v-model="dataJson.tempJson.type"
  :value="dataJson.tempJson.type"
  :para="CONSTANTS.DICT_B_PROJECT_TYPE"
  :disabled="false"
  @change="handleTypeChange"
/>
```
- **Import**: `import RadioDict from '@/components/00_dict/redio'`
- **Components注册**: `components: { RadioDict }`
- **注意**: 路径是 `redio` 不是 `radio`

## 4. 业务搜索选择控件

### 4.1 通用搜索输入框
```vue
<!-- Import -->
import InputSearch from '@/components/40_input/inputSearch'

<!-- 使用 -->
<input-search
  v-model.trim="dataJson.tempJson.supplier_name"
  :disabled="false"
  @onInputSearch="handleSupplierDialog"
/>
```
- **Import**: `import InputSearch from '@/components/40_input/inputSearch'`
- **Components注册**: `components: { InputSearch }`
- **事件**: `@onInputSearch` (点击选择按钮触发)

### 4.2 供应商选择控件
```vue
<!-- Import -->
import SelectCpSupplier from '@/components/xx/supplier/SelectCpSupplier.vue'

<!-- 使用 -->
<select-cp-supplier
  v-model.trim="dataJson.searchForm.supplier_name"
  :placeholder="isPlaceholderShow('请选择供应商')"
  placement="left"
  @keyup.enter.native="handleSearch"
  @onReturnData="handleSupplierReturnDataName"
/>
```
- **Import**: 根据实际路径 (业务专用组件)
- **事件**: `@onReturnData` (选择完成后触发)

### 4.3 客户选择控件
```vue
<!-- Import -->
import SelectSeCustomer from '@/components/xx/customer/SelectSeCustomer.vue'

<!-- 使用 -->
<select-se-customer
  v-model.trim="dataJson.searchForm.purchaser_name"
  :placeholder="isPlaceholderShow('请选择主体企业')"
  placement="left"
  @keyup.enter.native="handleSearch"
  @onReturnData="handleCustomerReturnDataName"
/>
```
- **Import**: 根据实际路径 (业务专用组件)

### 4.4 仓库选择控件
```vue
<!-- Import -->
import SelectWarehouse from '@/components/xx/warehouse/SelectWarehouse.vue'

<!-- 使用 -->
<select-warehouse
  v-model.trim="dataJson.searchForm.warehouse_name"
  placeholder="请选择仓库"
  placement="left"
  @keyup.enter.native="handleSearch"
  @onReturnData="handleWarehouseReturnDataName"
/>
```
- **Import**: 根据实际路径 (业务专用组件)

## 5. 日期时间控件

### 5.1 日期选择器
```vue
<!-- 无需import，Element UI原生组件 -->
<el-date-picker
  v-model="dataJson.tempJson.plan_time"
  value-format="yyyy-MM-dd"
  type="date"
  clearable
  placeholder="选择日期"
  style="width: 100%"
/>
```
- **Import**: 无需import (Element UI)
- **格式**: `value-format="yyyy-MM-dd"`

### 5.2 日期时间选择器
```vue
<!-- 无需import，Element UI原生组件 -->
<el-date-picker
  v-model="dataJson.tempJson.sign_date"
  value-format="yyyy-MM-dd HH:mm:ss"
  type="datetime"
  clearable
  placeholder="选择日期"
  style="width: 100%"
/>
```
- **Import**: 无需import (Element UI)
- **格式**: `value-format="yyyy-MM-dd HH:mm:ss"`
- **类型**: `type="datetime"`

### 5.3 日期范围选择器
```vue
<!-- 无需import，Element UI原生组件 -->
<el-date-picker
  v-model="dataJson.searchForm.plan_times"
  type="datetimerange"
  range-separator="至"
  start-placeholder="计划时间开始"
  end-placeholder="计划时间结束"
  format="yyyy-MM-dd HH:mm:ss"
  value-format="yyyy-MM-dd HH:mm:ss"
/>
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zxhbloomer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
