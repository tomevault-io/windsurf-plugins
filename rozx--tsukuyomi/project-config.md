---
trigger: always_on
description: > **项目**: AI 驱动的日本小说翻译器 (Vue 3 + Quasar + TypeScript + Electron)
---

# Tsukuyomi Translator - AI Coding Agent Guide

> **项目**: AI 驱动的日本小说翻译器 (Vue 3 + Quasar + TypeScript + Electron)

---

## 开发命令

```bash
# 安装依赖
bun install
bun run setup:git-hooks  # 首次 clone 后必跑：注册 .githooks/pre-commit（自动 bump build 号）

# 开发
bun run dev              # 前端(9000) + 后端(8080) 同时启动
bun run dev:electron     # Electron 桌面应用开发模式

# 构建
bun run build:spa        # 构建 Web SPA
bun run build:electron   # 构建 Electron 桌面应用

# 代码质量 (修改后必须运行)
bun run lint             # ESLint 检查
bun run type-check       # TypeScript 类型检查 (vue-tsc --noEmit)
bun run quality-check    # Fallow 代码质量检查 (bunx fallow)
bun run format           # Prettier 格式化

# 测试
bun test                           # 运行所有测试
bun test book-service              # 按文件名模式匹配
bun test -t "应该保存书籍"         # 按测试名匹配
bun test --watch                   # 监听模式
```

**修改代码后必须运行**: `bun run lint && bun run type-check && bun run quality-check`

**首次 clone 必跑**: `bun run setup:git-hooks` — 把 `core.hooksPath` 指向 [`.githooks/`](.githooks/) 启用 pre-commit。hook 文件或目录缺失时 git **静默跳过**，build 号不会自增，记得跑。

---

## 代码风格

### 导入规范

```typescript
// 类型导入必须使用 type 关键字 (ESLint @typescript-eslint/consistent-type-imports 强制)
import type { Novel, Chapter } from 'src/models/novel';
import { BookService } from 'src/services/book-service';
```

### 格式化

- 单引号、行宽 100、分号结尾、2 空格缩进、UTF-8、LF 换行
- 运行 `bun run format` 自动格式化

### 命名规范

| 类型      | 规范                 | 示例                   |
| --------- | -------------------- | ---------------------- |
| Service   | PascalCase + Service | `BookService`          |
| 文件名    | kebab-case           | `book-service.ts`      |
| 测试文件  | `.test.ts` 后缀      | `book-service.test.ts` |
| 变量/函数 | camelCase            | `getAllBooks`          |
| 常量      | UPPER_SNAKE_CASE     | `MAX_RETRY_COUNT`      |

### Vue 组件

- `<script setup lang="ts">` 置于 template 之后
- Props 使用 TypeScript 接口：`defineProps<Props>()`
- Emits 类型安全：`defineEmits<{ save: [id: string] }>()`

### ESLint 关键规则

- `@typescript-eslint/consistent-type-imports`: error — 必须用 `import type`
- `@typescript-eslint/no-explicit-any`: warn — 避免使用 any
- `@typescript-eslint/no-unused-vars`: warn — 未使用变量以 `_` 前缀忽略
- `@typescript-eslint/no-floating-promises`: off
- `@typescript-eslint/no-misused-promises`: warn
- TypeScript strict 模式已启用 (quasar.config.ts)

### Fallow 误报抑制

`bun run quality-check` 跑的 fallow 无法识别 Vue `<template>` 消费者、动态 import、抽象基类多态调用等路径。遇到 `unused-export` / `unused-class-member` 告警：

1. **优先删真死代码** — 别急着抑制，先用 Grep 确认是否真的没人用
2. **确认是误报** 就用**行内注释**抑制，**不要**往 `.fallowrc.json` 加 `ignoreExports` / `usedClassMembers`（用户明确反对根配置里的符号白名单）
3. 规则名是**单数**：`unused-export` / `unused-class-member`（不是复数）

```ts
// fallow-ignore-next-line unused-export
export const MODEL_ID = '...';

/**
 * 抽象方法，子类实现通过 NovelScraperFactory 多态分派
 */
// fallow-ignore-next-line unused-class-member
abstract fetchNovel(url: string): Promise<Novel>;
```

注释放在声明**正上方一行**；有 JSDoc 时夹在 JSDoc 的 `*/` 与声明之间。

---

## 架构分层

```
数据流向: pages/components (UI) → composables (逻辑复用) → stores (Pinia 状态) → services (业务逻辑) → IndexedDB/API

src/
├── models/        # 数据结构定义 (纯 TypeScript，无依赖)
├── services/      # 业务逻辑 (不依赖 Vue/Pinia)
│   ├── ai/        # AI 子系统
│   │   ├── core/        # 基础 AI 服务
│   │   ├── providers/   # AI 提供商 (OpenAI/Gemini)
│   │   ├── tasks/       # AI 任务 (translate/polish/proofread/explain/assistant)
│   │   └── tools/       # 30+ AI 工具定义 (function calling)
│   └── scraper/   # 小说网站爬虫 (ncode/kakuyomu/syosetu 等)
├── composables/   # Vue Composition API 封装
├── stores/        # Pinia 状态管理 (12 个 store)
├── components/    # UI 组件
├── pages/         # 页面组件
├── router/        # Vue Router 路由配置
├── i18n/          # 国际化 (zh-CN/zh-TW/en-US)
├── utils/         # 工具函数
├── constants/     # 常量定义
├── types/         # 全局类型定义
└── __tests__/     # 测试文件 (70+ 测试文件)
```

**核心 Services**: `book-service`, `chapter-service`, `chapter-content-service`, `memory-service`, `memory-scoring`, `embedding-service`, `embedding-queue`, `terminology-service`, `sync-data-service`

---

## 设备变体规则 (Dispatcher + Desktop / Tablet / Mobile)

**强制规则**: 所有 `src/layouts/*.vue`、`src/pages/*.vue`，以及任何在桌面 / 平板 / 手机上呈现差异明显的组件，都必须使用 dispatcher + 三变体模式。**严禁**在页面或布局里直接写 `v-if="isPhone"` / `v-if="isElectron"` 分支。

### 分派规则

唯一实现位置: [`src/composables/useDeviceVariant.ts`](src/composables/useDeviceVariant.ts)

- Electron 永远强制 `'desktop'`（不看窗口尺寸）
- Web 端按 `useResponsiveLayout()` 断点: `'mobile'` / `'tablet'` / `'desktop'`
- 禁止在别处手写 `isElectron ? ... : isPhone ? ...`。豁免：叶子对话框（`BookDialog`、`NovelScraperDialog` 等）

### 标准文件结构

```
src/pages/<name>.vue                          # dispatcher (30 行内，路由指向它)
src/pages/<name-kebab>/
  <Name>Desktop.vue
  <Name>Tablet.vue                            # 通常是 <Desktop /> 的 3 行 wrapper
  <Name>Mobile.vue
src/composables/<name-kebab>/use<Name>.ts     # 业务逻辑，provide/inject 跨变体
```

同构适用于 `src/layouts/` 和需要差异渲染的 `src/components/`（如 `AppRightPanel`、`TranslationProgress`）。

### Dispatcher 模板

```vue
<script setup lang="ts">
import { computed } from 'vue';
import { useDeviceVariant } from 'src/composables/useDeviceVariant';
import { provide<Name>Page } from 'src/composables/<name-kebab>/use<Name>Page';
import <Name>Desktop from './<name-kebab>/<Name>Desktop.vue';
import <Name>Tablet from './<name-kebab>/<Name>Tablet.vue';

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rozx/Tsukuyomi](https://github.com/rozx/Tsukuyomi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
