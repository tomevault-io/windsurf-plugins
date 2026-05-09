---
trigger: always_on
description: Naive UI 组件库使用规范
---


# Naive UI 组件库规范

本项目使用 [Naive UI](https://www.naiveui.com/) 作为主要组件库

## 组件导入

### 自动导入配置
```typescript
// ✅ 使用自动导入，无需手动 import
<template>
  <n-button type="primary">按钮</n-button>
  <n-input v-model:value="text" />
  <n-select v-model:value="value" :options="options" />
</template>

// ❌ 不需要手动导入
import { NButton, NInput } from 'naive-ui'
```

### Message、Dialog、Notification 等组合式 API
```typescript
// ✅ 使用 composable API
<script setup lang="ts">
import { useMessage, useDialog, useNotification } from 'naive-ui'

const message = useMessage()
const dialog = useDialog()
const notification = useNotification()

function handleClick() {
  message.success('操作成功')
  dialog.warning({
    title: '警告',
    content: '确定要执行此操作吗？'
  })
}
</script>
```

## 常用组件使用

### Button 按钮
```vue
<template>
  <!-- ✅ 基础按钮 -->
  <n-button type="primary">主要按钮</n-button>
  <n-button type="info">信息按钮</n-button>
  <n-button type="success">成功按钮</n-button>
  <n-button type="warning">警告按钮</n-button>
  <n-button type="error">错误按钮</n-button>
  
  <!-- ✅ 按钮尺寸 -->
  <n-button size="tiny">极小</n-button>
  <n-button size="small">小</n-button>
  <n-button size="medium">中（默认）</n-button>
  <n-button size="large">大</n-button>
  
  <!-- ✅ 按钮状态 -->
  <n-button :loading="loading">加载中</n-button>
  <n-button :disabled="disabled">禁用</n-button>
  
  <!-- ✅ 图标按钮 -->
  <n-button circle quaternary>
    <template #icon>
      <n-icon><PauseIcon /></n-icon>
    </template>
  </n-button>
  
  <!-- ✅ 文字按钮 -->
  <n-button text tag="a" href="/settings">设置</n-button>
</template>
```

### Input 输入框
```vue
<template>
  <!-- ✅ 基础输入 -->
  <n-input 
    v-model:value="text"
    placeholder="请输入内容"
    clearable
  />
  
  <!-- ✅ 带图标 -->
  <n-input v-model:value="search" placeholder="搜索">
    <template #prefix>
      <n-icon><SearchIcon /></n-icon>
    </template>
  </n-input>
  
  <!-- ✅ 密码输入 -->
  <n-input 
    v-model:value="password"
    type="password"
    show-password-on="click"
  />
  
  <!-- ✅ 文本域 -->
  <n-input
    v-model:value="description"
    type="textarea"
    :rows="4"
    placeholder="多行文本"
  />
  
  <!-- ✅ 输入组 -->
  <n-input-group>
    <n-input v-model:value="username" placeholder="用户名" />
    <n-button type="primary">搜索</n-button>
  </n-input-group>
</template>
```

### Select 选择器
```vue
<script setup lang="ts">
const value = ref<string>()
const options = [
  { label: '选项1', value: '1' },
  { label: '选项2', value: '2' },
  { label: '选项3', value: '3' }
]

// ✅ 带分组的选项
const groupedOptions = [
  {
    type: 'group',
    label: '分组1',
    key: 'group1',
    children: [
      { label: '选项1', value: '1' }
    ]
  }
]
</script>

<template>
  <!-- ✅ 基础选择器 -->
  <n-select v-model:value="value" :options="options" />
  
  <!-- ✅ 多选 -->
  <n-select 
    v-model:value="values"
    :options="options"
    multiple
    filterable
  />
  
  <!-- ✅ 可搜索 -->
  <n-select
    v-model:value="value"
    :options="options"
    filterable
    placeholder="搜索选择"
  />
  
  <!-- ✅ 自定义渲染 -->
  <n-select
    v-model:value="value"
    :options="options"
    :render-label="renderLabel"
  />
</template>
```

### Dialog 对话框
```vue
<script setup lang="ts">
const showModal = ref(false)
const dialog = useDialog()

// ✅ 使用 useDialog API
function showConfirm() {
  dialog.warning({
    title: '确认删除',
    content: '确定要删除这个种子吗？',
    positiveText: '确定',
    negativeText: '取消',
    onPositiveClick: () => {
      message.success('已删除')
    }
  })
}

// ✅ 使用 n-modal 组件
</script>

<template>
  <!-- ✅ Modal 组件 -->
  <n-modal 
    v-model:show="showModal"
    preset="dialog"
    title="对话框标题"
    positive-text="确定"
    negative-text="取消"
    @positive-click="handleConfirm"
  >
    对话框内容
  </n-modal>
  
  <!-- ✅ 自定义内容的 Modal -->
  <n-modal v-model:show="showModal">
    <n-card
      style="width: 600px"
      title="自定义对话框"
      :bordered="false"
      size="huge"
      role="dialog"
      aria-modal="true"
    >
      <template #header-extra>
        <n-button @click="showModal = false">关闭</n-button>
      </template>
      自定义内容
    </n-card>
  </n-modal>
</template>
```

### Message 消息提示
```typescript
// ✅ 使用 message API
const message = useMessage()

// 成功消息
message.success('操作成功')

// 错误消息
message.error('操作失败')

// 警告消息
message.warning('请注意')

// 信息消息
message.info('提示信息')

// 加载消息
const loading = message.loading('加载中...', {
  duration: 0 // 持续显示
})
// 手动关闭
loading.destroy()

// ✅ 配置消息
message.success('操作成功', {
  duration: 3000,
  closable: true,
  onClose: () => {
    console.log('消息已关闭')
  }
})
```

### Table 表格
```vue
<script setup lang="ts">
import type { DataTableColumns } from 'naive-ui'

interface TorrentRow {
  id: number
  name: string
  size: number
  progress: number
}

const columns: DataTableColumns<TorrentRow> = [
  {
    key: 'name',
    title: '名称',
    width: 300,
    ellipsis: {
      tooltip: true
    }
  },
  {
    key: 'size',
    title: '大小',
    render: (row) => formatBytes(row.size)
  },
  {
    key: 'progress',
    title: '进度',
    render: (row) => h(NProgress, {
      percentage: row.progress,
      type: 'line'
    })
  }
]

const data = ref<TorrentRow[]>([])
const loading = ref(false)
const pagination = reactive({
  page: 1,
  pageSize: 20,
  showSizePicker: true,
  pageSizes: [10, 20, 50, 100]
})
</script>

<template>
  <!-- ✅ 基础表格 -->
  <n-data-table
    :columns="columns"
    :data="data"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jianxcao/transmission-web](https://github.com/jianxcao/transmission-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
