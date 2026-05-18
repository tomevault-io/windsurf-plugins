---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## PROJECT OVERVIEW

**Project**: 115Master - Tampermonkey 用户脚本
**Purpose**: 增强 115 网盘的浏览和播放体验

**Tech Stack**:

- Vue 3 + TypeScript + Vite
- vite-plugin-monkey (打包工具)
- Tailwind CSS + DaisyUI (样式)
- localforage (IndexedDB 存储)
- pnpm workspaces + Turbo (Monorepo)

**Core Features**:

- 画质增强
- 视频缩略图预览
- 在线字幕加载
- 播放列表管理
- 快捷键系统

---

## ENVIRONMENT REQUIREMENTS

CRITICAL: 必须使用 pnpm 作为包管理器

| Requirement  | Version                       | Notes               |
| ------------ | ----------------------------- | ------------------- |
| Node.js      | >= 20.12                      | 运行时环境          |
| pnpm         | >= 9.15.9                     | REQUIRED - 强制使用 |
| Browser      | Chrome 130+ or 115Browser 35+ | 目标浏览器          |
| Tampermonkey | >= 5.3.3                      | 用户脚本管理器      |

---

## DEVELOPMENT COMMANDS

### Essential Commands

```bash
pnpm install          # 安装依赖 (MUST use pnpm)
pnpm dev              # 开发环境 (热重载)
pnpm build            # 生产构建
pnpm build:plus       # Plus 版本构建
pnpm type-check       # TypeScript 类型检查
pnpm lint             # ESLint 检查
pnpm lint:fix         # ESLint 自动修复
```

### Testing Commands

```bash
pnpm test             # 运行测试
pnpm test:watch       # 监听模式
pnpm test:coverage    # 覆盖率报告
pnpm test:ui          # Vitest UI
```

### Analysis Commands

```bash
pnpm analyze          # 构建分析 (生成 stats.html)
pnpm lint:inspector   # ESLint 配置检查器
```

### Changesets Commands

```bash
pnpm changeset            # 创建变更记录
pnpm version-packages     # 消费 changesets，更新版本号和 CHANGELOG（由 CI 自动执行）
```

**Changesets 工作流**:

1. 开发完功能/修复后，运行 `pnpm changeset`
2. 选择受影响的包, 如:（`@115master/monkey`、`@115master/shared`）
3. 选择版本类型：`patch`（修复）/ `minor`（功能）/ `major`（破坏性变更）
4. 填写变更描述（会生成 `.changeset/xxx.md` 文件，需要一起提交）
5. `version-packages` 和发版由 Release workflow (`.github/workflows/release.yml`) 自动处理
6. 配置使用 `@changesets/changelog-github`，CHANGELOG 会自动关联 PR 和贡献者

### Monorepo Commands

```bash
pnpm clean            # 清理所有构建产物和 node_modules
turbo run dev --filter @115master/monkey     # 仅运行 monkey 应用
turbo run build --filter @115master/shared   # 仅构建 shared 包
```

---

## PROJECT ARCHITECTURE

### Monorepo Structure

```txt
├── apps/                      # 应用程序目录
│   └── monkey/                # Tampermonkey 用户脚本应用
│       ├── src/               # 源代码
│       ├── package.json
│       ├── tsconfig.json
│       └── vite.config.ts
├── packages/                  # 共享包目录
│   ├── eslint-config/         # ESLint 共享配置
│   ├── tsconfig/              # TypeScript 共享配置
│   └── shared/                # 共享工具库
├── package.json               # 根 package.json
├── pnpm-workspace.yaml        # pnpm 工作区配置
└── turbo.json                 # Turbo 任务配置
```

### Apps/Monkey Directory Structure

```txt
apps/monkey/src/
├── main.ts                    # 入口文件，路由匹配和页面初始化
├── pages/
│   ├── home/                  # 首页 (Mod 模式)
│   │   ├── BaseMod/           # 修改器基类和管理器
│   │   ├── FileListMod/       # 文件列表增强
│   │   ├── TopFilePathMod/    # 顶部路径显示
│   │   └── TopHeaderMod/      # 顶部头部增强
│   ├── video/                 # 视频播放页面
│   │   ├── components/        # 播放器组件
│   │   ├── data/              # 数据管理
│   │   └── index.vue          # 主组件
│   └── magnet/                # 磁力链接页面
├── components/
│   ├── XPlayer/               # 视频播放器核心
│   │   ├── components/        # 子组件
│   │   ├── hooks/             # 逻辑 hooks
│   │   ├── events/            # 事件系统
│   │   └── utils/             # 工具函数
│   └── ...
├── hooks/                     # 全局 hooks
├── utils/
│   ├── cache/                 # 缓存系统
│   ├── request/               # 网络请求
│   ├── drive115/              # 115 API
│   └── ...
├── constants/                 # 常量
└── types/                     # 类型定义
```

### Key Architecture Patterns

#### 1. Routing System

- 使用 `glob-to-regexp` 进行模式匹配
- 路由规则定义在 `apps/monkey/src/main.ts`
- 每个 URL 模式对应一个页面处理函数

#### 2. Page Modes

- **Home**: Mod 模式 - 继承 `BaseMod`，实现 `destroy()` 方法
- **Video/Magnet**: Vue 组件模式 - 完整组件生命周期

#### 3. Component Organization

```txt
ComponentName/
├── components/    # 子组件
├── hooks/         # Composition API 逻辑
├── utils/         # 工具函数
├── types/         # 类型定义
├── styles/        # 样式
└── index.vue      # 主组件
```

#### 4. Cache System

- `GMRequestCache`: 缓存网络请求
- `ActressesFaceCache`: 缓存演员头像
- 基于 IndexedDB 持久化

#### 5. Player Architecture (XPlayer)

- **事件系统**: mitt 组件间通信
- **数据管理**: 分层数据流
- **快捷键**: 可配置快捷键支持
- **画质**: 支持多种画质和 HDR

---

## CODING STANDARDS

### Vue Component Rules

MUST:

- 使用 `<script setup lang="ts">` 语法
- 使用 `<style module>` CSS Modules
- 遵循模板顺序: Template -> Script -> Style
- 在 template 中使用 `:class="$style['class-name']"`

### Tailwind Class Abstraction

MUST use `clsx` utility to abstract Tailwind classes:

```typescript
import { clsx } from '@/utils/clsx'

const styles = clsx({
  container: {
    main: 'bg-base-100 flex h-full flex-col rounded-xl',
    header: 'flex items-center justify-between px-4 py-2',
  }
})
```

### TypeScript Rules

MUST:

- 优先使用 `type` 而非 `interface`
- 避免使用 `any` 和 `unknown`
- 合理使用泛型
- 组件 props 使用类型定义
- **优先使用 `type-fest` 辅助类型**:
  - `RequireAtLeastOne` - 至少需要一个属性
  - `ValueOf` - 获取对象值的类型
  - `Opaque` - 创建不透明类型

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cbingb666/115master](https://github.com/cbingb666/115master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
