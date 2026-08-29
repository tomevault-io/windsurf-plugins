---
trigger: always_on
description: 本文件约束自动化代理在本工作区中的默认行为。
---

# AGENTS.md — ainovel-gui 项目约束

本文件约束自动化代理在本工作区中的默认行为。

---

## 指令优先级

1. 当前会话用户的明确要求
2. 仓库自身规则、文档与约定
3. 本文件
4. 相关 Superpowers / skill 流程定义

- 默认以 Superpowers 作为主工作流体系，但不默认启用 full Superpowers。
- 本文件保留个人硬门禁、环境约束、交付偏好与沟通方式。
- 只读分析任务可不进入完整实现流程，但结论必须清晰、可追溯。
- 若用户明确要求 `continue nonstop`，默认持续推进，直到满足验收标准或出现真实阻塞。

---

## 仓库专有信息

### Repository purpose

ainovel-gui 是 **独立桌面创作管理平台**，基于 [ainovel-cli](https://github.com/crazytreeChen/ainovel-cli) 的 AI 长篇小说创作引擎提供可视化 GUI。
**不是 CLI 的包装外壳**——GUI 直接管理全部数据，`ainovel-cli` 仅作为 AI 推理引擎被调度。

技术栈：Electron（主进程）+ Vite + React 18 + TypeScript + Zustand（状态管理）+ React Router v7。
本地持久化：[better-sqlite3](https://github.com/WiseLibs/better-sqlite3)（SQLite，主存储）+ JSON 文件（CLI 兼容导出）。
目标平台：macOS / Windows / Linux 桌面端。

**编辑平台**：macOS
**构建/运行平台**：macOS / Windows / Linux
**CI**：GitHub Actions（push to master 触发 lint/typecheck/build；push `v*` tag 触发多平台构建 + Release）

### Build

```bash
# 安装依赖
npm install

# 开发模式（Vite dev server + Electron）
npm run electron:dev

# 仅前端开发
npm run dev

# 生产构建
npm run build

# macOS DMG
npm run dist:mac

# Windows NSIS
npm run dist:win
```

依赖项：Node.js ≥ 18，运行 AI 创作需要安装 `ainovel-cli`。

### Architecture

```
┌─────────────────────────────────────────────────────┐
│                AINovel GUI (Electron)                │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────────┐ │
│  │ 书籍 │ │ 大纲 │ │ 章节 │ │ 角色 │ │ 时间线...│ │
│  │ 管理 │ │ 管理 │ │ 管理 │ │ 管理 │ │ 14模块   │ │
│  └──┬───┘ └──┬───┘ └──┬───┘ └──┬───┘ └────┬─────┘ │
│     └────────┴────────┴────────┴──────────┘        │
│                       ↕ IPC                         │
│  ┌──────────────────────────────────────────────┐   │
│  │         Electron 主进程                       │   │
│  │  SQLite / 子进程管理 / IPC 桥接                │   │
│  └─────┬───────────────┬───────────────────────┘   │
│        ↓               ↕ spawn                     │
│   SQLite (主存储)     ainovel-cli --headless        │
│   ~/.ainovel-gui/     (仅 AI 推理引擎)              │
│      ainovel.db                                   │
│        ↓                                           │
│   JSON 文件导出 (books/{uuid}/)                    │
│   [CLI 兼容 / 备份 / 交换]                         │
└─────────────────────────────────────────────────────┘
```

### 14 个功能模块

| # | 模块 | 数据源（主存储 / 导出） | 说明 |
|---|------|------------------------|------|
| M1 | **书籍管理** | `books` + `progress` / `book.json`, `progress.json` | 多本书列表 + CRUD |
| M2 | **创作工作台** | 全部 | 四面板布局 + 命令面板，对标 TUI 全部面板 |
| M3 | **创作控制** | host lifecycle | 开始/暂停/停止/干预/恢复 |
| M4 | **大纲管理** | `outline_entries` + `volumes` + `arcs` + `arc_chapters` + `compass` / `outline.json`, `compass.json` | 卷-弧-章树 + 指南针 |
| M5 | **章节管理** | `chapters` + `drafts` + `chapter_plans` / `chapters/*.md`, `drafts/*.plan.json` | Markdown 编辑器 + 元数据 |
| M6 | **角色管理** | `characters_t` + `cast_entries` + `character_snapshots` / `characters.json`, `cast_ledger.json` | 角色卡片 + 配角名册 |
| M7 | **时间线管理** | `timeline_events` + `foreshadow_entries` + `relationship_entries` + `state_changes` / `timeline.json`, `foreshadow_ledger.json`, `relationship_state.json`, `state_changes.json` | 事件线 + 伏笔 + 关系 |
| M8 | **评审管理** | `reviews` / `reviews/*.json` | 七维雷达图 + 问题清单 |
| M9 | **模型管理** | `config` (providers/roles) | Provider + 角色模型分配 |
| M10 | **系统设置** | `config` (全局) | 工作目录 / 主题 / 语言 |
| M11 | **用户规则** | `user_rules` / `user_rules.json` | 规则查看/编辑/检查 |
| M12 | **诊断报告** | diag engine | 四维度发现 + 建议导出 |
| M13 | **仿写画像** | `simulation_profiles` / `simulation_profile.json` | 风格分析可视化 |
| M14 | **导出管理** | exp engine | TXT/EPUB 导出面板 |

### 数据存储

**双层模型**：SQLite 为主存储，JSON 文件为 CLI 兼容导出。

- **主存储（SQLite）**：所有元数据写入 `~/.ainovel-gui/ainovel.db`（`better-sqlite3`，WAL 模式 + 外键约束）。
  Schema 定义见 `electron/database.ts`，表包括：`books`, `progress`, `outline_entries`, `volumes`, `arcs`, `arc_chapters`, `compass`, `chapters`, `drafts`, `chapter_plans`, `summaries`, `characters_t`, `character_snapshots`, `cast_entries`, `timeline_events`, `foreshadow_entries`, `relationship_entries`, `state_changes`, `world_rules`, `style_rules`, `reviews`, `run_meta`, `usage_stats`, `user_rules`, `simulation_profiles`, `user_directives`, `config`, `_meta`。
  GUI 写入时**先写 SQLite**，必要时再同步导出到 JSON。

- **导出 / 备份（JSON + Markdown）**：每本书可选导出到 `~/.ainovel-gui/books/{book-uuid}/`，
  数据格式与 [ainovel-cli](https://github.com/crazytreeChen/ainovel-cli) 完全兼容，可直接互换使用。
  此目录主要用途：CLI 互通、人工备份、跨工具迁移。

```
~/.ainovel-gui/
├── ainovel.db            # SQLite 主存储（GUI 写入）
├── config.json           # 兼容 ainovel-cli 风格的导出（与 config 表互相同步）
└── books/                # 导出 / 备份目录（CLI 兼容格式）
    └── {book-uuid}/
        ├── book.json             # 书籍元信息
        ├── premise.md            # 故事前提
        ├── outline.json          # 扁平大纲
        ├── layered_outline.json  # 分层大纲
        ├── compass.json          # 指南针
        ├── progress.json         # 进度状态
        ├── characters.json       # 角色档案
        ├── cast_ledger.json      # 配角名册
        ├── timeline.json         # 时间线
        ├── foreshadow_ledger.json # 伏笔台账
        ├── relationship_state.json # 关系
        ├── state_changes.json    # 状态变化
        ├── world_rules.json      # 世界观
        ├── style_rules.json      # 风格规则
        ├── run.json              # 运行元信息
        ├── usage.json            # 用量统计
        ├── user_rules.json       # 用户规则快照
        ├── simulation_profile.json # 仿写画像

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crazytreeChen/ainovel-gui](https://github.com/crazytreeChen/ainovel-gui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
