---
trigger: always_on
description: // ✅ 组件文件: PascalCase
---

# 代码质量标准和开发规范

## 代码风格规范

### 命名规范
```typescript
// 文件命名
// ✅ 组件文件: PascalCase
ComponentName.vue
PropertyEditor.vue

// ✅ 工具函数: camelCase  
utilityFunction.ts
apiService.ts

// ✅ 配置文件: kebab-case
api-config.ts
build-config.ts

// ✅ 常量: SCREAMING_SNAKE_CASE
const API_BASE_URL = 'https://api.example.com'
const MAX_RETRY_COUNT = 3

// ✅ 变量和函数: camelCase
const userName = 'admin'
const isLoading = false
function handleClick() {}

// ✅ 类型和接口: PascalCase
interface UserInfo {}
type ComponentProps = {}
```

### 导入导出规范
```typescript
// ✅ 推荐的导入顺序
// 1. Node.js 内置模块
import path from 'path'

// 2. 第三方库
import { ref, computed, watch } from 'vue'
import { defineStore } from 'pinia'
import { ElMessage } from 'element-plus'

// 3. 内部模块 - 按路径深度排序
import type { ApiResponse } from '@/types'
import { apiClient } from '@/services/base'
import { useAppStore } from '@/stores/app'
import ComponentEditor from '@/components/editors/ComponentEditor.vue'

// ✅ 导出规范
// 优先使用命名导出
export const utilFunction = () => {}
export { ApiService } from './api'

// 默认导出仅用于Vue组件和主要模块
export default defineComponent({
  name: 'ComponentName'
})
```

## 代码注释规范

### JSDoc 注释标准
```typescript
/**
 * 用户信息接口
 * @interface UserInfo
 */
interface UserInfo {
  /** 用户ID */
  id: string
  /** 用户名 */
  username: string
  /** 用户邮箱 */
  email: string
}

/**
 * 获取用户信息
 * @param {string} userId - 用户ID
 * @param {Object} options - 可选参数
 * @param {boolean} options.includeProfile - 是否包含详细信息
 * @returns {Promise<ApiResponse<UserInfo>>} 返回用户信息
 * @throws {Error} 当用户不存在时抛出错误
 * @example
 * ```typescript
 * const user = await getUserInfo('123', { includeProfile: true })
 * console.log(user.data.username)
 * ```
 */
async function getUserInfo(
  userId: string, 
  options: { includeProfile?: boolean } = {}
): Promise<ApiResponse<UserInfo>> {
  // 实现逻辑
}
```

### Vue组件注释
```vue
<template>
  <!-- 主容器 - 负责布局和样式控制 -->
  <div class="component-container">
    <!-- 标题区域 -->
    <header class="header">
      <h1>{{ title }}</h1>
    </header>
    
    <!-- 内容区域 - 支持插槽自定义 -->
    <main class="content">
      <slot name="content">
        <!-- 默认内容 -->
      </slot>
    </main>
  </div>
</template>

<script setup lang="ts">
/**
 * 通用容器组件
 * 提供标准的页面布局结构，支持标题和内容自定义
 * 
 * @component Container
 * @example
 * <Container title="页面标题">
 *   <template #content>
 *     自定义内容
 *   </template>
 * </Container>
 */

interface Props {
  /** 页面标题 */
  title: string
  /** 是否显示边框 */
  bordered?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  bordered: true
})

// 组件逻辑...
</script>
```

## 错误处理规范

### 统一错误处理
```typescript
// ✅ 错误类型定义
enum ErrorCode {
  NETWORK_ERROR = 'NETWORK_ERROR',
  VALIDATION_ERROR = 'VALIDATION_ERROR',
  PERMISSION_DENIED = 'PERMISSION_DENIED',
  RESOURCE_NOT_FOUND = 'RESOURCE_NOT_FOUND'
}

interface AppError {
  code: ErrorCode
  message: string
  details?: any
  timestamp: number
}

// ✅ 错误处理工具
class ErrorHandler {
  /**
   * 创建标准化错误对象
   */
  static createError(code: ErrorCode, message: string, details?: any): AppError {
    return {
      code,
      message,
      details,
      timestamp: Date.now()
    }
  }

  /**
   * 处理API错误
   */
  static handleApiError(error: any): AppError {
    if (error.response) {
      const { status, data } = error.response
      switch (status) {
        case 404:
          return this.createError(
            ErrorCode.RESOURCE_NOT_FOUND,
            '请求的资源不存在',
            { originalError: error }
          )
        case 403:
          return this.createError(
            ErrorCode.PERMISSION_DENIED,
            '没有权限访问该资源',
            { originalError: error }
          )
        default:
          return this.createError(
            ErrorCode.NETWORK_ERROR,
            data?.message || '网络请求失败',
            { originalError: error }
          )
      }
    }
    
    return this.createError(
      ErrorCode.NETWORK_ERROR,
      '网络连接失败',
      { originalError: error }
    )
  }

  /**
   * 显示用户友好的错误消息
   */
  static showUserError(error: AppError) {
    ElMessage.error(error.message)
    
    // 开发环境下输出详细错误信息
    if (import.meta.env.DEV) {
      console.error('错误详情:', error)
    }
  }
}

// ✅ 使用示例
async function loadUserData(userId: string) {
  try {
    const response = await userApi.getUser(userId)
    return response.data
  } catch (error) {
    const appError = ErrorHandler.handleApiError(error)
    ErrorHandler.showUserError(appError)
    throw appError
  }
}
```

### 组件级错误边界
```vue
<template>
  <div class="error-boundary">
    <template v-if="!hasError">
      <slot />
    </template>
    <template v-else>
      <div class="error-display">
        <el-alert
          title="组件加载失败"
          :description="errorMessage"
          type="error"
          show-icon
        >
          <template #default>
            <el-button @click="retry">重试</el-button>
          </template>
        </el-alert>
      </div>
    </template>
  </div>
</template>

<script setup lang="ts">
/**
 * 错误边界组件
 * 捕获子组件的错误并提供恢复机制
 */
import { ref, onErrorCaptured } from 'vue'

const hasError = ref(false)
const errorMessage = ref('')

// 捕获子组件错误
onErrorCaptured((error: Error) => {
  hasError.value = true
  errorMessage.value = error.message
  
  // 记录错误到日志系统
  console.error('组件错误:', error)
  
  // 阻止错误向上传播
  return false
})

const retry = () => {
  hasError.value = false

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [suyancc/easy_window](https://github.com/suyancc/easy_window) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
