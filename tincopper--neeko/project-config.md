---
trigger: always_on
description: > AI 编程助手项目上下文与开发规范。此文件是所有 AI Agent 的单一事实源（Single Source of Truth）。
---

# Neeko — Repository Guidelines

> AI 编程助手项目上下文与开发规范。此文件是所有 AI Agent 的单一事实源（Single Source of Truth）。

## 项目概览

**Neeko** 是一个基于 Tauri 2.0 + React 18 的桌面应用，统一管理多项目 AI Agent 会话。支持三种项目类型。

1. 本地项目
2. WSL 项目
3. SSH 远程项目

核心目标是将终端会话、Git 操作、文件变更、IDE 启动与 Skill 管理聚合到同一窗口，并保持会话可恢复。

- **版本**: 1.0.4
- **标识符**: `com.neeko.app`
- **许可证**: Apache 2.0
- **包管理器**: pnpm `9.12.2`
- **Node**: 18+
- **Rust edition**: 2021
- **前端端口**: 1420（与 `tauri.conf.json` 中 `devUrl` 对齐）

## 目录结构

### 前端（Feature-Based 架构）

```
src/
├── app/                              # 应用入口
│   ├── App.tsx                       # 组合层：hooks + JSX 编排
│   ├── main.tsx                      # 入口点
│   ├── AppModals.tsx, AppProviders.tsx
│   ├── components/                   # 应用级组件（SplashScreen 等）
│   ├── hooks/
│   │   ├── useAppShell.ts            # 主协调 hook
│   │   └── index.ts
│   └── dock/                         # Dock 布局组件
├── features/                         # 21 个功能域模块
│   ├── action-menu/ agent/ browser/ connection/ conversation/
│   ├── debug/ editor/ file/ git/ lsp/ notification/
│   ├── project/ quick-open/ session/ settings/ skill/
│   ├── status-bar/ symbol-nav/ task/ terminal/ theme/
│   └── (每个域有自己的 components/ hooks/ store/)
├── shared/                           # 跨域共享
│   ├── components/                   # 共享 UI 组件
│   ├── contexts/                     # React contexts
│   ├── dock/                         # Dock 系统
│   ├── hooks/                        # 共享 hooks
│   │   ├── useToast.ts, useKeyboardShortcuts.ts
│   │   ├── useProjectActions.ts, useSplitLayout.ts
│   │   └── ...
│   ├── store/                        # zustand 状态管理
│   │   ├── projectStore.ts, gitStore.ts, connectionStore.ts
│   │   ├── editorStore.ts, browserStore.ts, dockStore.ts
│   │   ├── lspStore.ts, notificationStore.ts, taskStore.ts
│   │   └── ...
│   ├── types/                        # 全局 TypeScript 类型（按域分文件）
│   │   ├── project.ts, git.ts, connection.ts
│   │   ├── terminal.ts, agent.ts, session.ts
│   │   ├── settings.ts, task.ts, skill.ts, editorGroup.ts
│   │   └── ...
│   └── utils/                        # 共享工具函数（27 个文件）
│       ├── terminal.ts, agents.ts, distros.ts
│       ├── fileIcons.ts, idePresets.ts, platform.ts
│       └── ...
├── layout/                           # 窗口布局框架
├── lib/                              # 工具库
├── ui/                               # 通用 UI 组件
├── styles/                           # 全局样式
└── testing/                          # 测试 setup
    └── setup.ts, factories.ts
```

### 后端（Domain-Driven 模块化架构）

```
src-tauri/src/
├── main.rs                           # Tauri 应用入口
├── lib.rs                            # 模块聚合 + neeko_invoke_handler!
├── app.rs                            # Tauri Builder 组装
├── app_state.rs                      # AppStateWrapper 组装中心
├── common/                           # 共享基础设施
│   ├── error.rs                      # AppError（thiserror 枚举）
│   ├── logger.rs                     # 文件日志，写入 ~/.neeko/neeko.log
│   ├── runtime.rs                    # 异步运行时工具
│   └── ...
├── agent/                            # Agent 管理（opencode, claude-code, gemini, codex, qoder, codebuddy）
│   ├── commands.rs                   # Tauri 命令
│   ├── commands_commit.rs            # Commit 相关命令
│   ├── manager.rs                    # AgentManager
│   └── mod.rs
├── project/                          # 项目管理
│   ├── commands.rs                   # 项目 CRUD
│   ├── commands_ide.rs              # IDE 启动命令
│   └── mod.rs
├── session/                          # 会话持久化
│   ├── commands.rs
│   ├── manager.rs
│   └── mod.rs
├── terminal/                         # 终端管理（local/WSL/remote PTY）
│   ├── commands.rs
│   ├── services.rs
│   └── mod.rs
├── connection/                       # WSL + SSH 连接
│   ├── commands.rs
│   ├── services.rs
│   └── mod.rs
├── conversation/                     # 对话扫描/搜索/导出
│   ├── commands.rs
│   └── mod.rs
├── git/                              # Git 操作（git2-rs）
│   ├── commands.rs
│   ├── services/
│   └── mod.rs
├── skill/                            # Skill 管理（install, configure, tag, sync）
├── settings/                         # 应用设置管理
├── task/                             # 任务配置与执行
├── file/                             # 文件系统操作
├── browser/                          # 内置浏览器 webview
├── dap/                              # Debug Adapter Protocol
├── lsp/                              # Language Server Protocol
├── core/                             # 核心运行时与进程工具
└── theme/                            # 主题同步
```

## Architecture and Data Flow

### Backend 主链路

`src-tauri/src/main.rs` 调用 `neeko_lib::run`。

`src-tauri/src/app.rs` 负责 Tauri Builder 组装。

1. 初始化日志与 PATH
2. 注入 `SkillStore` 与 `AppStateWrapper`
3. 在 setup 阶段恢复 session、启动 watcher、加载自定义 agent
4. 注册命令处理器

命令注册入口：

```rust
.invoke_handler(crate::neeko_invoke_handler!())
```

`src-tauri/src/lib.rs` 中 `neeko_invoke_handler!` 维护完整命令清单，是命令注册单一事实源。

### Frontend 主链路

`src/app/main.tsx` 挂载应用。

`src/app/App.tsx` 组合层。

1. 调用 `useAppShell`
2. 初始化阶段显示 `SplashScreen`
3. 正常阶段挂载 `TitleBar`、`AppLayout`、`AppModals`、`AppToast`

状态协同由 hooks 与 store 完成。

1. 组合入口 `src/app/hooks/useAppShell.ts`
2. 全局状态 `src/shared/store/`（zustand）
3. 类型定义 `src/shared/types/`

### 关键数据流

1. UI 交互触发 hooks
2. hooks 通过 `@tauri-apps/api/core` 的 `invoke` 调用 Rust 命令

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tincopper/neeko](https://github.com/tincopper/neeko) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
