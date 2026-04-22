---
trigger: always_on
description: This document provides essential information for AI coding agents working on this project.
---

# AGENTS.md - AI Toolbox Development Guide

This document provides essential information for AI coding agents working on this project.

## Communication Language

与用户的所有对话必须使用**中文**，包括问题澄清、方案说明、进度反馈和结果总结。代码注释和 commit message 仍使用英文。

## Module Documentation

主要功能模块可以在自身目录下放置 `AGENTS.md`，用于记录该模块的 `Source of Truth`、设计决策、关键流程、易错点和最小验证。修改模块代码前，如果目标目录或更近的父目录存在模块级 `AGENTS.md`，必须先阅读。

### Hard Rules

1. 修改任何模块目录下的文件前，若该模块目录或更近的父目录存在 `AGENTS.md`，必须先读，再设计或实现。
2. 当一次改动引入新的非显而易见设计决策、修复未来高概率复发的问题、或重构跨文件关键流程时，应在同一任务内同步更新对应模块的 `AGENTS.md`。
3. 根 `AGENTS.md` 只保留全局规则。模块专属内容应下沉到对应模块目录下的 `AGENTS.md`。
4. 模块级 `AGENTS.md` 只写高价值信息：`Source of Truth`、`Why`、`关键流程`、`Gotchas`、`最小验证`。不要写文件清单、类型定义、API 清单、数据库字段表、或可被代码直接证明的事实。
5. 当根与模块 `AGENTS.md` 冲突时，以作用域更近、语义更具体的文档为准。
6. 新增模块级 `AGENTS.md` 时，需在本节索引中同一任务内补上入口。
7. 模块级 `AGENTS.md` 建设处于迁移期时，根 `AGENTS.md` 中已有的高价值信息只能补充重组，不能因为下沉而删弱；确认模块文档已完整覆盖前，不要移除根中的对应规则。
8. 后续开发中，只要遇到值得长期沉淀的知识、用户反复强调的约束、或已经复发/高风险复发的坑点，必须在同一任务内及时写回对应模块的 `AGENTS.md`，不能等“下次再整理”。
9. 如果某条经验已经上升为跨模块通用规则、全局架构约束、或多个模块都会反复踩到的铁律，应同步写回根 `AGENTS.md`，而不是只埋在单个模块文档里。
10. 数据库相关经验默认按全局规则处理，优先写回根 `AGENTS.md`；只有某条数据库约束严格局限于单一模块时，才在对应模块 `AGENTS.md` 补充模块特有语义。
11. 仓库内用于给 agent 阅读的模块文档 `AGENTS.md` 不是应用运行时资源；本地开发 watcher 和类似热重载链路应尽量忽略它们，避免把文档编辑误当成代码改动。
12. 不要把上一条误用到产品运行时 prompt 文件上。当前仓库里的 OpenCode / Codex 运行时 prompt 文件名就是 `AGENTS.md`，Claude Code 运行时 prompt 文件名是 `CLAUDE.md`；它们属于真实业务数据，备份、恢复、WSL/SSH 同步和页面交互都依赖这些文件，不能按“仅 agent 文档”排除。

### Template

- `docs/module-agents-template.md`：模块级 `AGENTS.md` 模板。新增模块文档时先复制，再删除不适用小节。

### Index

| 模块目录 | 说明 |
|---------|------|
| `tauri/src/coding/` | Coding 域共享规则：runtime location、事件驱动托盘、WSL Direct、跨工具 CLI/路径语义 |
| `tauri/src/coding/claude_code/` | Claude Code 后端配置、prompt、plugin、MCP 与 WSL 同步约束 |
| `tauri/src/coding/codex/` | Codex 后端配置、auth/config.toml、prompt、plugin 与 WSL 同步约束 |
| `tauri/src/coding/mcp/` | MCP Server 后端存储、工具配置同步、导入导出与 WSL 联动 |
| `tauri/src/coding/open_code/` | OpenCode 后端配置文件、provider、prompt、tray 与 WSL 同步约束 |
| `tauri/src/coding/open_claw/` | OpenClaw 后端配置文件与 WSL 同步约束 |
| `tauri/src/coding/oh_my_openagent/` | Oh My OpenAgent 后端配置、临时本地态、应用链路与 OpenCode WSL 联动 |
| `tauri/src/coding/oh_my_opencode_slim/` | Oh My OpenCode Slim 后端配置、临时本地态、应用链路与 OpenCode WSL 联动 |
| `tauri/src/coding/session_manager/` | 会话浏览、详情、重命名、导入导出与运行时路径解析 |
| `tauri/src/coding/skills/` | Skills 中央仓库、导入发现、同步、托盘和 WSL/SSH 相关链路 |
| `tauri/src/coding/tools/` | Skills/MCP 共用工具适配、检测与自定义工具存储 |
| `tauri/src/coding/wsl/` | WSL 同步配置、自动同步监听、WSL Direct 状态消费 |
| `tauri/src/coding/ssh/` | SSH 连接、文件映射、手动同步、MCP/Skills 远端同步 |
| `web/features/coding/claudecode/` | Claude Code 前端页面、根目录配置、provider 与 prompt 交互 |
| `web/features/coding/codex/` | Codex 前端页面、根目录配置、provider 与 prompt 交互 |
| `web/features/coding/mcp/` | MCP 前端页面、服务器管理、导入流程与工具同步交互 |
| `web/features/coding/opencode/` | OpenCode 前端页面、配置路径、provider、prompt 与模型刷新交互 |
| `web/features/coding/openclaw/` | OpenClaw 前端页面、配置路径、provider 与配置文件交互 |
| `web/features/coding/shared/` | coding 共享前端语义：根目录弹窗、全局 prompt、favorite provider、会话面板、连通性测试 |
| `web/features/coding/skills/` | Skills 前端页面、中央仓库视角、分组展示与批量同步交互 |
| `web/features/settings/` | WSL/SSH 设置页、同步入口、moduleStatuses 消费和 UI 边界 |
| `tauri/src/settings/backup/` | 备份恢复、WebDAV、自动备份与恢复后续链路 |

后续新增模块级 `AGENTS.md` 时，继续在此表追加，不在根文档其他位置零散登记。

## Project Overview

AI Toolbox is a cross-platform desktop application built with:
- **Frontend**: React 19 + TypeScript 5 + Ant Design 5 + Vite 7
- **Backend**: Tauri 2.x + Rust
- **Database**: SurrealDB 2.x (embedded SurrealKV)
- **Package Manager**: pnpm

## Directory Structure

```
ai-toolbox/
├── web/                    # Frontend source code
│   ├── app/                # App entry, routes, providers
│   ├── components/         # Shared components
│   ├── features/           # Feature modules
│   │   ├── coding/         # Coding tools (claudecode, codex, opencode, skills)
│   │   ├── daily/          # Daily notes
│   │   └── settings/       # App settings
│   ├── stores/             # Zustand state stores
│   ├── i18n/               # i18next localization
│   ├── constants/          # Module configurations
│   ├── hooks/              # Global hooks
│   ├── services/           # API services
│   └── types/              # Global type definitions
├── tauri/                  # Rust backend
│   ├── src/                # Rust source
│   │   ├── coding/         # Coding modules (claude_code, codex, open_code, skills)
│   │   └── settings/       # Settings modules
│   └── Cargo.toml          # Rust dependencies
└── package.json            # Frontend dependencies
```

## Build & Development Commands

### Frontend (pnpm)

```bash
# Install dependencies
pnpm install

# Start development server (frontend only)
pnpm dev

# Build frontend for production
pnpm build

# Type check
pnpm tsc --noEmit
```

### Tauri (Full App)

```bash
# Start full app in development mode
pnpm tauri dev

# Build production app
pnpm tauri build
```

### Rust (Backend)

```bash
# Check Rust code
cd tauri && cargo check

# Build Rust in release mode
cd tauri && cargo build --release

# Format Rust code
cd tauri && cargo fmt

# Lint Rust code
cd tauri && cargo clippy
```

### Testing

```bash
# Frontend tests
pnpm test

# Run single test file
node --test web/test/path/to/test.test.ts

# Rust tests
cd tauri && cargo test


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coulsontl/ai-toolbox](https://github.com/coulsontl/ai-toolbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
