---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**重要提示：**
- 当功能发生变化时，请保持此文件和 `README.md` 同步更新。请更新文档以反映当前状态，但是需要经过我的允许后再修改。
- 所有的注释和日志优先采用中文，保留必要的专业术语部分。
- 所有的依赖包的安装都要先进行搜索，综合判断依赖采用的版本，而不是默认采用某个版本。
- 状态管理上我们全部采用 Jotai 来实现。
- 这是个开源项目，本地存储优先，善用配置文件优于大部分默认采用 localstorage，不采用本地数据库方案。
- 保证充分的组件化以及人类的可读性，每次完成改动后都要思考这一点，运行@code-simplifier 来简化优化代码，保持简单直接不过渡设计的风格。
- 在 UI 设计上采用更现代的方案，UI 组件推荐采用 ShadcnUI，在合适的情况下，用卡片和阴影取代边框，用符合主题的饱满色彩，设置界面要设置背景，为未来做不同主题留下空间。
- 采用 BDD 行为驱动开发的方案。

## 项目概述

Proma 是一个集成通用 AI Agent 的下一代人工智能软件，采用 Electron 桌面应用架构。

## Monorepo 结构

Bun workspace monorepo：

```
proma-v2/
├── packages/
│   ├── shared/     # 共享类型、IPC 通道常量、配置、工具函数 (v0.1.15)
│   ├── core/       # AI Provider 适配器、代码高亮服务 (v0.2.2)
│   └── ui/         # 共享 UI 组件 (CodeBlock, MermaidBlock) (v0.1.3)
└── apps/
    └── electron/   # Electron 桌面应用 (v0.9.5)
        └── src/
            ├── main/       # 主进程 + 服务层 (main/lib/)
            ├── preload/    # IPC 上下文桥接
            └── renderer/   # React UI (Vite + Tailwind + Radix UI)
```

**包命名规范**：`@proma/*` 作用域（`@proma/core`、`@proma/shared`、`@proma/ui`、`@proma/electron`）

**依赖管理**：package.json 中使用 `workspace:*` 引用内部包

### 包职责详解

#### @proma/shared (v0.1.15)
- **导出模块**：`./types`、`./config`、`./utils`、`./constants/permission-rules`
- **关键类型**：`AgentMessage`、`ChatMessage`、`Channel`、`PermissionRequest`、`FeishuConfig`
- **依赖**：无运行时依赖（仅 TypeScript）

#### @proma/core (v0.2.2)
- **导出模块**：`./providers`、`./highlight`、`./types`、`./utils`
- **关键功能**：Provider 适配器注册表、代码高亮（Shiki）
- **依赖**：`@proma/shared`、`shiki`
- **Peer 依赖**：`@anthropic-ai/claude-agent-sdk`、`@anthropic-ai/sdk`、`@modelcontextprotocol/sdk`

#### @proma/ui (v0.1.3)
- **关键组件**：共享 React UI 组件库
- **依赖**：`@proma/core`、`beautiful-mermaid`、`shiki`、Radix UI
- **Peer 依赖**：`react@^18.3.0`、`react-dom@^18.3.0`

#### @proma/electron (v0.9.5)
- **职责**：Electron 桌面应用主体，集成所有包
- **关键依赖**：
  - `@anthropic-ai/claude-agent-sdk@0.3.143` - Agent SDK
  - `@larksuiteoapi/node-sdk` - 飞书集成
  - Radix UI、TipTap、Tailwind CSS
  - 文件解析：`pdf-parse`、`officeparser`、`word-extractor`

## 常用命令

```bash
# 开发模式（推荐 - 自动启动 Vite + Electron + 热重载）
bun run dev

# 手动开发模式（调试时更稳定）
# 终端 1: cd apps/electron && bun run dev:vite
# 终端 2: cd apps/electron && bun run dev:electron

# 构建并运行
bun run electron:start

# 仅构建
bun run electron:build

# 类型检查（所有包）
bun run typecheck

# 单包类型检查
cd packages/core && bun run typecheck

# 测试
bun test

# 打包分发
cd apps/electron
bun run dist:mac      # macOS
bun run dist:win      # Windows
bun run dist:linux    # Linux
bun run dist:fast     # 当前架构快速打包
```

### Electron 构建脚本（`apps/electron/` 目录下）

```bash
bun run build:main        # esbuild → dist/main.cjs
bun run build:preload     # esbuild → dist/preload.cjs
bun run build:renderer    # Vite → dist/renderer/
bun run build:resources   # 复制 resources/ 到 dist/
bun run generate:icons    # 生成应用图标
```

## 运行时环境

使用 Bun 代替 Node.js/npm/pnpm：

- `bun install` 安装依赖，`bun run <script>` 运行脚本
- `bun test` 运行测试（内置测试运行器，`import { test, expect } from "bun:test"`）
- Bun 自动加载 .env 文件（无需 dotenv）
- 优先使用 Bun 原生 API：`Bun.file` > `node:fs`，`Bun.$\`command\`` > `execa`

## 技术栈

| 层级 | 技术 | 版本 |
|------|------|------|
| **运行时** | Bun | 1.2.5+ |
| **语言** | TypeScript | 5.0.0+ |
| **桌面框架** | Electron | 39.5.1 |
| **前端框架** | React | 18.3.1 |
| **状态管理** | Jotai | 2.17.1 |
| **UI 组件** | Radix UI | 最新 |
| **样式** | Tailwind CSS | 3.4.17 |
| **富文本编辑器** | TipTap | 3.19.0 |
| **代码高亮** | Shiki | 3.22.0 |
| **Markdown** | React Markdown | 10.1.0 |
| **图表** | Beautiful Mermaid | 最新 |
| **数学公式** | KaTeX | 0.16+ |
| **构建工具** | Vite | 6.0.3 |
| **打包工具** | esbuild | 0.24.0+ |
| **分发工具** | Electron Builder | 25.1.8 |
| **Agent SDK** | @anthropic-ai/claude-agent-sdk | 0.3.143 |
| **飞书 SDK** | @larksuiteoapi/node-sdk | 最新 |

## 核心架构

### IPC 通信模式（最重要的架构模式）

类型定义 → 主进程处理 → Preload 桥接 → 渲染进程调用：

1. **类型 & 常量**：`@proma/shared` 定义 IPC 通道名称常量和请求/响应类型
2. **主进程处理**：`main/ipc.ts`（57KB）注册 `ipcMain.handle()` 处理器，调用 `main/lib/` 服务
3. **Preload 桥接**：`preload/index.ts` 通过 `contextBridge.exposeInMainWorld` 暴露类型安全的 API
4. **渲染进程**：通过 `window.electronAPI.*` 调用，Jotai atoms 中封装调用逻辑

添加新 IPC 通道时，需要同步修改这四个位置。

#### 主要 IPC 通道组

- `IPC_CHANNELS` - 基础通道（运行时、Git、环境）
- `CHANNEL_IPC_CHANNELS` - 渠道管理
- `CHAT_IPC_CHANNELS` - Chat 功能
- `AGENT_IPC_CHANNELS` - Agent 功能
- `ENVIRONMENT_IPC_CHANNELS` - 环境检查
- `PROXY_IPC_CHANNELS` - 代理设置
- `SYSTEM_PROMPT_IPC_CHANNELS` - 系统提示词
- `MEMORY_IPC_CHANNELS` - 记忆功能
- `CHAT_TOOL_IPC_CHANNELS` - Chat 工具
- `FEISHU_IPC_CHANNELS` - 飞书集成
- `GITHUB_RELEASE_IPC_CHANNELS` - GitHub 发布

### 主进程服务层（`main/lib/`）

#### 核心服务

| 服务 | 职责 |
|------|------|
| `agent-orchestrator.ts` | Agent 核心编排层（71KB）：并发守卫、渠道查找、环境变量构建、SDK 路径解析、消息持久化、事件流处理、错误处理、自动标题生成 |
| `agent-session-manager.ts` | Agent 会话管理：SDK 消息持久化、会话元数据 CRUD、JSONL 存储 |
| `agent-prompt-builder.ts` | Agent 系统提示词构建（18KB）：动态上下文构建、内置 Agent 构建、工作区上下文注入 |
| `agent-permission-service.ts` | Agent 权限管理：工具权限检查、权限模式管理 |
| `agent-ask-user-service.ts` | Agent 用户交互：AskUser 请求处理 |
| `agent-exit-plan-service.ts` | Agent 退出计划服务 |
| `agent-workspace-manager.ts` | 工作区管理（16KB）：MCP Server 配置、Skills 配置、工作区 CRUD |
| `chat-service.ts` | Chat 流式调用编排（20KB）：Provider 适配器集成、消息持久化、AbortController |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [proma-ai/Proma](https://github.com/proma-ai/Proma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
