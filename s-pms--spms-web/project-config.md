---
trigger: always_on
description: > 本文档供 AI 编程助手（如 Qwen Code、Cursor、GitHub Copilot 等）参考，遵循本文档规范可以保证代码风格一致。使用 VibeCoding 开发时请严格遵循本文档约定。
---

# SPMS-Web 开发规范

> 本文档供 AI 编程助手（如 Qwen Code、Cursor、GitHub Copilot 等）参考，遵循本文档规范可以保证代码风格一致。使用 VibeCoding 开发时请严格遵循本文档约定。

## 项目概述

SPMS-Web 是 SPMS 智能生产管理系统的 Web 前端项目，基于 **Vue 3 + TypeScript + Vite** 开发，使用 **Element Plus** 作为 UI 组件库，依赖 **@airpower/web** 基础框架。

## 技术栈

| 技术          | 版本  | 用途                               |
| ------------- | ----- | ---------------------------------- |
| Vue           | 3.5+  | 前端框架 (Composition API + setup) |
| TypeScript    | 5.9+  | 类型系统                           |
| Vite          | 7.3+  | 构建工具                           |
| Element Plus  | 2.12+ | UI 组件库                          |
| Pinia         | 3.0+  | 状态管理                           |
| Vue Router    | 4.6+  | 路由                               |
| @airpower/web | 2.9+  | 基础框架 (CRUD组件、Hooks、工具)   |
| Axios         | 1.13  | HTTP 客户端                        |
| ECharts       | 6.0   | 图表                               |
| SCSS          | 1.96+ | 样式预处理                         |
| ESLint        | 9.39+ | 代码检查 (@antfu/eslint-config)    |

## 项目结构

```
src/
├── assets/          # 静态资源 (css, images等)
├── base/            # 抽象基类
│   ├── AbstractBaseService.ts    # 服务基类
│   └── BaseEntity.ts             # 实体基类
├── component/       # 全局公共组件
├── config/          # 应用配置
│   ├── AppConfig.ts    # 应用配置
│   ├── Constant.ts     # 常量
│   └── routes.ts       # 路由配置
├── hook/            # 自定义 Hooks
│   └── useMyTable.ts  # 表格Hook封装
├── model/           # 数据模型 (Entity, Service, Enum)
├── view/            # 页面视图
├── App.vue
├── env.d.ts
└── main.ts
```

## 目录分层规范

### 1. model/ - 数据模型层

**结构**:
```
model/
├── business-module/        # 业务模块 (如 asset, channel, factory, mes 等)
│   ├── function-module/    # 功能模块 (如 material, device, storage 等)
│   │   ├── XxxEntity.ts       # 实体类
│   │   ├── XxxService.ts      # 服务类
│   │   ├── XxxTypeEnum.ts     # 枚举类 (可选)
│   │   └── ...
```

**规范**:
- 按业务模块 → 功能模块二级分组
- 命名：
  - 实体: `XxxEntity.ts` (PascalCase)
  - 服务: `XxxService.ts` (PascalCase)
  - 枚举: `XxxEnum.ts` (PascalCase)

### 2. view/ - 视图层

**结构**:
```
view/console/business-module/function-module/
├── component/              # 局部组件
│   ├── editor.vue          # 编辑器弹窗
│   ├── selector.vue        # 选择器弹窗 (可选)
│   └── detail.vue          # 详情查看 (可选)
└── list.vue               # 列表页面
```

**规范**:
- 列表页: `list.vue` (kebab-case)
- 编辑器弹窗: `editor.vue`
- 选择器弹窗: `selector.vue`
- 详情组件: `detail.vue`
- 局部组件放入 `component/` 子目录

## 编码规范

### 格式规范

| 规则          | 取值     |
| ------------- | -------- |
| 缩进          | 2 空格   |
| 分号          | 不使用   |
| 引号          | 单引号   |
| Trailing 逗号 | 自动保留 |

### 命名规范

| 类型      | 规范                | 示例                                |
| --------- | ------------------- | ----------------------------------- |
| 类        | PascalCase          | `MaterialEntity`, `MaterialService` |
| 枚举类    | PascalCase          | `MaterialTypeEnum`                  |
| 枚举常量  | UPPER_SNAKE_CASE    | `PRODUCT`, `PURCHASE`               |
| 接口/类型 | PascalCase (前缀 I) | `IBaseTableResult`                  |
| 函数/变量 | camelCase           | `useMyTable`, `onPublish`           |
| TS 文件   | PascalCase          | `MaterialEntity.ts`                 |
| Vue 文件  | kebab-case          | `editor.vue`, `list.vue`            |

### 类型规范

- **必须**声明类型，不使用 `any`
- 确定有值时使用非空断言 `!`
- 泛型正确传递类型参数

### 导入路径规范

- 使用 `@/` 别名开头指向 `src/`
- 同级导入使用相对路径 `./`
- 第三方包直接导入

**正确示例**:
```typescript
import { AbstractBaseService } from '@/base/AbstractBaseService'
import { MaterialEntity } from './MaterialEntity'
import { WebEnum } from '@airpower/web'
```

## Vue 组件规范

### 块顺序 (强制校验)

强制顺序：`script` → `template` → `style`

```vue
<script lang="ts" setup>
// ...
</script>

<template>
  <!-- ... -->
</template>

<style lang="scss" scoped></style>
```

### 属性换行 (强制校验)

- 单行最多 1 个属性
- 多行每个属性占一行

**正确示例**:
```vue
<ATable
  :use-hook="hook"
  show-export
/>
```

### Script Setup 规范

- **必须**使用 `<script lang="ts" setup>`
- 弹窗 Props 使用 `DialogProps.withParam()`
- 充分利用 TypeScript 类型推断

```typescript
const props = defineProps(DialogProps.withParam(new MaterialEntity()))

const {
  title,
  formData,
  rules,
  formRef,
  isLoading,
  onSubmit,
} = useEditor(props, MaterialService)
```

## AirPower 装饰器规范

### 装饰器顺序 (必须遵循)

```typescript
@Model({ label: '物料' })
export class MaterialEntity extends BaseEntity {
  @Table({ ... })  // 1. 表格配置
  @Form({ ... })    // 2. 表单配置
  @Search({ ... })  // 3. 搜索配置
  @Field({ ... })   // 4. 字段标记
  name!: string
}
```

### 常用装饰器配置说明

| 装饰器    | 用途       | 常用配置项                                                                                                  |
| --------- | ---------- | ----------------------------------------------------------------------------------------------------------- |
| `@Model`  | 类标记     | `label` - 类标签                                                                                            |
| `@Field`  | 字段标记   | `label` - 字段标签，`dictionary` - 枚举字典                                                                 |
| `@Table`  | 表格列配置 | `width` - 宽度，`force` - 强制显示，`copy` - 可复制，`datetime` - 日期格式化，`color` - 染色                |
| `@Form`   | 表单配置   | `requiredString`/`requiredNumber`/`requiredPayload` - 必填，`defaultValue` - 默认值，`placeholder` - 占位符 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [s-pms/SPMS-Web](https://github.com/s-pms/SPMS-Web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
