---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PackyCode Cost Monitor is a Chrome browser extension built with Plasmo framework that monitors PackyCode API usage and budgets. It features dual token authentication (JWT/API Key), real-time purchase status monitoring, and budget tracking with notifications.

**📚 Complete Documentation**: See [docs/README.md](docs/README.md) for comprehensive technical documentation including architecture design, usage guides, and developer resources organized by Linus & Dan's design philosophies.

## Data Task System

**类型安全的数据获取任务管理系统**

系统使用统一的数据获取任务管理，确保所有执行路径的行为完全一致。

### 核心设计原则

**单一数据源**: 所有数据获取任务配置统一在 `utils/taskRegistry.ts` 的 `TASK_REGISTRY` 中。

### 添加新的数据获取任务

1. **添加任务类型枚举** (在 `utils/taskRegistry.ts`):

```typescript
export enum DataTaskType {
  FETCH_USER_INFO = "fetchUserInfo",
  CHECK_PURCHASE_STATUS = "checkPurchaseStatus",
  YOUR_NEW_TASK = "yourNewTask" // 添加新任务类型
}
```

2. **添加任务配置** (TypeScript 会强制要求):

```typescript
export const TASK_REGISTRY: Record<DataTaskType, TaskDefinition> = {
  // 现有任务...
  [DataTaskType.YOUR_NEW_TASK]: {
    type: DataTaskType.YOUR_NEW_TASK,
    description: "你的任务描述",
    handler: yourDataFetchFunction,
    priority: 10
  }
  // TypeScript 编译时会检查完整性！
}
```

### 系统架构优势

1. **编译时安全**: TypeScript 强制所有任务类型都有对应配置
2. **行为一致**: alarm 轮询、手动触发、background 消息使用完全相同的配置
3. **语义准确**: 命名反映实际功能（数据获取），而非使用方式（刷新）
4. **无字符串字面量**: 枚举约束防止拼写错误和类型逃逸

### 执行流程

```
用户点击刷新 ──┐
Alarm 定时器 ──┤
Background 消息 ──┘
               │
               ▼
        taskExecutor.fetchAllDataAsync()
               │
               ▼
        taskRegistry.executeAllTasks()
               │
               ▼
        按优先级执行 TASK_REGISTRY 中的所有任务
               │
               ▼
        fetchUserInfo() + checkAndNotifyPurchaseStatus()
```

### 强制约束机制

- **TypeScript 编译失败** 如果任何枚举值缺少配置
- **运行时类型守卫** 验证所有 action 字符串
- **枚举设计** 防止字符串字面量绕过类型检查
- **单一配置源** 确保不同执行路径的行为完全一致

### 文件职责

- **utils/taskRegistry.ts**: 数据获取任务的注册表和执行逻辑
- **utils/taskExecutor.ts**: 简化的任务批量执行接口
- **background.ts**: 通过定时器和消息处理调用任务系统
- **popup.tsx**: 通过任务执行器触发数据获取

就是这样！无需复杂工具、注册系统或合规检查 - 只需简单的 TypeScript 类型约束。

## Development Commands

```bash
# Development
pnpm dev                 # Start development server, loads build/chrome-mv3-dev
pnpm build              # Build for production
pnpm package            # Package extension for distribution

# Code Quality
pnpm lint               # Run ESLint
pnpm lint:fix           # Auto-fix ESLint issues
pnpm format             # Format code with Prettier
pnpm format:check       # Check code formatting
pnpm type-check         # TypeScript type checking
```

## Code Quality Requirements

### 🔍 Mandatory Code Diagnostics

**CRITICAL**: After ANY code modification, you MUST run diagnostics using `mcp__ide__getDiagnostics` to ensure:

1. **Zero TypeScript Errors**: All type errors must be resolved before considering the task complete
2. **Zero TypeScript Warnings**: Address all warnings to maintain code quality
3. **Clean Compilation**: The code must compile without any issues
4. **Type Safety**: Ensure all new code maintains the project's type safety standards

**Workflow**: 
```
Code Modification → Run Diagnostics → Fix Issues → Run Diagnostics Again → Confirm Clean
```

If diagnostics reveal any issues:
- Fix them immediately
- Re-run diagnostics to confirm resolution
- Never leave code with unresolved diagnostics

This is a **non-negotiable requirement** for maintaining the project's high code quality standards.

## Architecture Overview

### Core Components

- **popup.tsx**: Main UI entry point with budget monitoring interface
- **background.ts**: Service worker handling alarms, token management, and data task execution
- **CombinedStatus.tsx**: Unified authentication and purchase status display

### Data Task Architecture

统一的数据获取任务管理系统，确保所有执行路径的行为完全一致：

#### 核心文件

- **utils/taskRegistry.ts**: 数据获取任务的注册表，定义所有可执行的数据任务
- **utils/taskExecutor.ts**: 任务执行器，提供统一的批量执行接口
- **background.ts**: 通过 Chrome alarms 和消息处理调用任务系统
- **popup.tsx**: 通过任务执行器触发手动数据获取

#### 设计约束

- **类型安全**: 所有任务都通过 `DataTaskType` 枚举定义，防止字符串字面量逃逸
- **编译时检查**: `TASK_REGISTRY` 必须包含所有枚举值的配置，否则编译失败
- **单一配置源**: alarm 轮询、手动触发、background 消息使用完全相同的任务配置
- **优先级执行**: 任务按 priority 排序执行，确保数据依赖关系

#### 执行流程统一性

```typescript
// 所有执行路径都汇聚到同一个函数
executeAllTasks() // 从 TASK_REGISTRY 按优先级执行所有任务
  ├── alarm 轮询调用
  ├── 手动刷新调用
  └── background 消息调用
```

### Authentication System

Dual token system supporting both JWT (from web cookies) and API Keys:

- **JWT Tokens**: Auto-extracted from PackyCode website cookies, expire-aware
- **API Keys**: Long-lived tokens copied from dashboard, detected via webRequest API
- Token switching handled automatically when API keys are detected

### Data Task Monitoring

- **任务轮询**: 每30秒通过 Chrome alarms 执行 `executeAllTasks()`
- **状态检测**: 监控 `purchaseDisabled` 字段变化 (true→false) 和用户预算使用情况
- **通知推送**: 当购买可用时发送 Chrome notifications
- **数据流**: background.ts → taskRegistry.executeAllTasks() → Chrome Storage → UI components

### Storage Architecture

Uses Plasmo Storage API with these key data:

- `packy_token`: Current authentication token

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [94mashiro/packycode-cost](https://github.com/94mashiro/packycode-cost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
