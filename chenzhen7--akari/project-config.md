---
trigger: always_on
description: **Akari** 是一个 AI Agent 并行开发管理平台。完整产品架构见 [docs/设计文档.md](docs/设计文档.md)，开发计划见 [docs/开发计划.md](docs/开发计划.md)。
---

# CLAUDE.md

---

## 项目概述

**Akari** 是一个 AI Agent 并行开发管理平台。完整产品架构见 [docs/设计文档.md](docs/设计文档.md)，开发计划见 [docs/开发计划.md](docs/开发计划.md)。

---

## 技术栈

```
前端 (apps/web): React 19 + TypeScript + Vite + Tailwind CSS + shadcn/ui
画布: @xyflow/react
看板: @dnd-kit/core
终端: @xterm/xterm + FitAddon + WebLinksAddon
状态: Zustand
Diff: @monaco-editor/react（懒加载）

后端 (apps/server): Node.js + Fastify 5 + @fastify/websocket
终端复用: node-pty（PTY，Shell: PowerShell 7 / pwsh.exe）
Git 操作: simple-git
文件监听: chokidar
通信: WebSocket（ws://localhost:3001/ws）
数据库: SQLite - better-sqlite3

共享类型: packages/shared-types（workspace:*）
```

---

## 项目结构（当前实际）

```
akari/
├── apps/
│   ├── server/                        # 后端 Fastify 服务
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   └── src/
│   │       ├── index.ts               # 入口：依赖组装、插件注册、服务启动
│   │       ├── session-manager.ts     # SessionManager Facade（组合 Service / 事件连线）
│   │       ├── core/                  # 领域核心（状态机、工厂）
│   │       │   ├── session-state-machine.ts   # 会话状态机 + validateTransition
│   │       │   └── session-factory.ts         # AgentSession / MainSession 工厂
│   │       ├── services/              # 业务编排层（可单测、依赖接口）
│   │       │   ├── session-lifecycle.service.ts
│   │       │   ├── tab.service.ts
│   │       │   ├── terminal.service.ts
│   │       │   ├── worktree.service.ts
│   │       │   ├── git-query.service.ts
│   │       │   ├── workspace.service.ts
│   │       │   ├── workspace-session-registry.service.ts
│   │       │   └── hook-dispatcher.service.ts
│   │       ├── infrastructure/        # 基础设施（实现细节）
│   │       │   ├── db/
│   │       │   │   ├── canvas-edge-store.ts
│   │       │   │   ├── settings-store.ts
│   │       │   │   └── repositories/
│   │       │   │       ├── session.repository.ts
│   │       │   │       ├── workspace.repository.ts
│   │       │   │       ├── settings.repository.ts
│   │       │   │       └── canvas-edge.repository.ts
│   │       │   ├── git/
│   │       │   │   ├── git-command-runner.ts
│   │       │   │   ├── git-repository.ts
│   │       │   │   ├── git-repository-registry.ts
│   │       │   │   ├── git-repository-detector.ts
│   │       │   │   └── git-utils.ts
│   │       │   ├── pty/
│   │       │   │   └── terminal-multiplexer.ts
│   │       │   └── fs/
│   │       │       └── file-system.service.ts
│   │       ├── types/
│   │       │   └── fastify.d.ts       # Fastify 装饰器类型声明
│   │       ├── plugins/
│   │       │   ├── websocket.ts       # WebSocket 注册与客户端消息处理
│   │       │   └── static.ts          # SPA static fallback
│   │       ├── routes/                # HTTP/WebSocket 入口（通过 SessionManager Facade 调用）
│   │       │   ├── health.ts
│   │       │   ├── settings.ts
│   │       │   ├── repo.ts
│   │       │   ├── sessions.ts
│   │       │   ├── git.ts
│   │       │   ├── files.ts
│   │       │   ├── diff.ts
│   │       │   ├── terminal.ts
│   │       │   ├── tabs.ts
│   │       │   ├── workspace.ts
│   │       │   ├── canvas.ts
│   │       │   └── hooks.ts
│   │       └── agent-adapters/        # AgentAdapter 接口 + 各品牌实现
│   │           ├── base.ts
│   │           ├── claude.ts
│   │           ├── claude-orchestrator.ts
│   │           ├── kimi.ts
│   │           ├── aider.ts
│   │           ├── shell.ts
│   │           └── index.ts
│   ├── desktop/                       # Electron 桌面端
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   ├── electron-builder.yml       # NSIS / portable 打包配置
│   │   ├── src/
│   │   │   ├── main.ts                # Electron 主进程
│   │   │   └── preload.ts             # 预加载脚本
│   │   └── dist/                      # tsc 输出
│   └── web/                           # 前端（Feature/Module 架构）
│       ├── package.json
│       ├── vite.config.ts             # 含 /api 和 /ws 反向代理 + alias
│       ├── tsconfig.json / app / node
│       ├── index.html
│       └── src/
│           ├── app/
│           │   ├── App.tsx            # 应用根组件
│           │   └── main.tsx           # 渲染入口
│           ├── features/              # 按功能模块组织
│           │   ├── session/           # 会话相关组件、store、lib
│           │   ├── terminal/          # 终端连接 store、terminalBus
│           │   ├── git/               # Git 图组件 + git-graph-utils
│           │   ├── diff/              # Diff 查看器
│           │   ├── explorer/          # 文件树 + 编辑器 + file-tree-store
│           │   ├── kanban/            # 看板视图
│           │   ├── canvas/            # 画布视图
│           │   ├── workspace/         # 工作区选择器 + store
│           │   ├── command-center/    # 广播命令中心
│           │   ├── settings/          # 设置对话框
│           │   └── layout/            # AppShell、TopNav、Sidebar 等布局组件
│           └── shared/                # 跨 feature 的通用能力
│               ├── components/
│               │   ├── ui/            # shadcn/ui 组件
│               │   ├── icons/         # ClaudeIcon、KimiIcon
│               │   └── theme-provider.tsx
│               ├── lib/               # 通用工具（api、utils、toast、agent-config 等）
│               ├── hooks/             # 通用 hooks
│               ├── stores/            # 跨 feature 的 store（ui-store、window-store 等）
│               └── types/             # 通用类型
├── packages/
│   └── shared-types/                  # 前后端共享类型包
│       ├── package.json
│       └── src/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chenzhen7/akari](https://github.com/chenzhen7/akari) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
