---
trigger: always_on
description: Vue3/TypeScript 前端编码规范和最佳实践
---


# Vue3/TypeScript 前端编码规范

## 代码风格

### 基本规范

- 使用 2 空格缩进
- 使用单引号（字符串）
- 每行代码不超过 100 字符
- 使用分号结尾
- 使用 Prettier 格式化代码

### 文件命名

```bash
# ✅ Vue 组件：使用大驼峰（PascalCase）
ContainerCard.vue
ComposeCreateView.vue
NetworkDetailView.vue

# ✅ TypeScript 文件：使用小驼峰（camelCase）
useContainer.ts
containerStats.ts
axiosConfig.ts

# ✅ 类型定义文件
types.ts
vite-env.d.ts
auto-imports.d.ts

# ❌ 避免：使用连字符或下划线
container-card.vue
compose_create_view.vue
```

## TypeScript 规范

### 类型定义

```typescript
// ✅ 使用 interface 定义对象类型
interface ContainerStatus {
  id: string
  name: string
  running: boolean
  status: 'UpdateAvailable' | 'UpToDate' | 'Error'
}

// ✅ 使用 type 定义联合类型和函数类型
type DeviceType = 'desktop' | 'tablet' | 'mobile'
type ErrorHandler = (error: Error) => void

// ✅ 明确指定函数返回类型
const fetchContainers = async (useCache = true): Promise<void> => {
  // ...
}

// ✅ 使用泛型
const formatResponse = <T>(data: T): ApiResponse<T> => {
  return { code: 0, data, msg: 'success' }
}
```

### 类型导入

```typescript
// ✅ 使用 type 关键字导入类型
import type { ContainerStatus, BatchUpdateResult } from '@/common/types'
import type { IconProps, MessageApi } from 'naive-ui'

// ✅ 混合导入
import { ref, computed, type Ref } from 'vue'
```

### 避免使用 any

```typescript
// ❌ 避免使用 any
const handleError = (error: any) => {
  console.error(error)
}

// ✅ 使用 unknown 或具体类型
const handleError = (error: unknown) => {
  if (error instanceof Error) {
    console.error(error.message)
  }
}

// ✅ 或定义具体类型
interface ApiError {
  code: number
  message: string
}

const handleError = (error: ApiError) => {
  console.error(error.message)
}
```

## Vue 3 Composition API

### Setup 语法

```vue
<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import type { ContainerStatus } from '@/common/types'

// ✅ 使用 ref 和 reactive
const loading = ref(false)
const containers = ref<ContainerStatus[]>([])

// ✅ 使用 computed
const runningCount = computed(() => 
  containers.value.filter(c => c.running).length
)

// ✅ 生命周期钩子
onMounted(async () => {
  await fetchData()
})

// ✅ 定义函数
const fetchData = async (): Promise<void> => {
  loading.value = true
  try {
    // 获取数据
  } catch (error) {
    console.error('获取数据失败:', error)
  } finally {
    loading.value = false
  }
}
</script>
```

### 组件通信

```vue
<script setup lang="ts">
// ✅ Props 定义（使用 TypeScript）
interface Props {
  containerId: string
  showActions?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  showActions: true
})

// ✅ Emits 定义
interface Emits {
  (e: 'update', id: string): void
  (e: 'delete', id: string): void
}

const emit = defineEmits<Emits>()

const handleUpdate = () => {
  emit('update', props.containerId)
}
</script>
```

## Pinia Store 规范

```typescript
// ✅ 使用 Composition API 风格的 Store
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useContainerStore = defineStore('container', () => {
  // ✅ 状态定义（使用 ref）
  const containers = ref<ContainerStatus[]>([])
  const loading = ref(false)
  
  // ✅ 计算属性
  const runningContainers = computed(() => 
    containers.value.filter(c => c.running)
  )
  
  // ✅ Actions（异步函数）
  const fetchContainers = async (): Promise<void> => {
    loading.value = true
    try {
      const data = await containerApi.getContainers()
      if (data.code === 0) {
        containers.value = data.data.containers
      } else {
        throw new Error(data.msg)
      }
    } catch (error) {
      console.error('获取容器列表失败:', error)
      throw error
    } finally {
      loading.value = false
    }
  }
  
  // ✅ 返回所有需要暴露的状态和方法
  return {
    // 状态
    containers,
    loading,
    
    // 计算属性
    runningContainers,
    
    // 方法
    fetchContainers,
  }
})
```

## 错误处理

```typescript
// ✅ 统一的错误处理模式
const handleAction = async (): Promise<boolean> => {
  try {
    const data = await api.performAction()
    if (data.code === 0) {
      message.success('操作成功')
      return true
    } else {
      throw new Error(data.msg)
    }
  } catch (error) {
    console.error('操作失败:', error)
    const errorMsg = error instanceof Error ? error.message : '未知错误'
    message.error(errorMsg)
    throw error
  }
}

// ✅ 使用自定义工具函数处理换行错误
import { showErrorWithNewlines } from '@/common/utils'

try {
  await someAction()
} catch (error: any) {
  showErrorWithNewlines(message, error.message)
}
```

## Vue 模板规范

```vue
<template>
  <!-- ✅ 使用 v-if 和 v-else-if -->
  <div v-if="loading">加载中...</div>
  <div v-else-if="error">{{ error }}</div>
  <div v-else>
    <!-- ✅ 使用 v-for 时必须提供 key -->
    <div
      v-for="container in containers"
      :key="container.id"
      class="container-item"
    >
      {{ container.name }}
    </div>
  </div>
  
  <!-- ✅ 事件处理使用箭头函数或方法引用 -->
  <n-button @click="handleClick">点击</n-button>
  <n-button @click="() => handleUpdate(id)">更新</n-button>
  
  <!-- ✅ 属性换行对齐 -->
  <n-input
    v-model:value="formData.name"
    placeholder="请输入名称"
    :disabled="loading"
    @update:value="handleChange"
  />
</template>
```

## 样式规范

```vue
<style scoped>
/* ✅ 使用 scoped 避免样式污染 */
.container-card {
  padding: 16px;
  border-radius: 8px;
  background: var(--n-color);
}

/* ✅ 使用 CSS 变量（Naive UI 主题变量） */
.title {
  color: var(--n-text-color);
  font-size: var(--n-font-size);
}

/* ✅ 使用嵌套（如果使用 SCSS） */
.container-item {
  padding: 12px;
  
  &:hover {
    background: var(--n-color-hover);
  }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jianxcao/watch-docker](https://github.com/jianxcao/watch-docker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
