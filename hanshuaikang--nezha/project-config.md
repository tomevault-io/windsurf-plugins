---
trigger: always_on
description: Nezha 是一款面向 AI 编程智能体（Claude Code、Codex）的桌面任务管理器，提供多项目工作区、实时终端输出、会话自动发现、权限感知执行、Git 集成和用量分析等核心功能。
---

# Nezha — AGENTS.md

## 项目概述

Nezha 是一款面向 AI 编程智能体（Claude Code、Codex）的桌面任务管理器，提供多项目工作区、实时终端输出、会话自动发现、权限感知执行、Git 集成和用量分析等核心功能。

**技术栈：** React 19 + TypeScript + Vite（前端） · Tauri 2 + Rust（桌面壳） · xterm.js（终端） · Shiki（语法高亮）

---

## 开发命令

```bash
pnpm dev            # 启动 Vite 开发服务器（端口 1420）
pnpm build          # tsc 类型检查 + Vite 打包
pnpm lint           # 运行 ESLint
pnpm test           # 运行 Vitest
pnpm tauri dev      # 启动完整桌面应用（自动启动开发服务器）
pnpm tauri build    # 构建生产环境桌面二进制包
```

Rust 后端位于 `src-tauri/`，修改后需重启 `tauri dev`。

---

## 架构设计

### 前端（`src/`）

| 文件 | 职责 |
|------|------|
| `App.tsx` | 根组件；持有所有状态（projects、tasks、buffers）及 Tauri 事件监听器 |
| `types.ts` | TypeScript 接口的权威定义——修改数据结构时优先编辑此文件 |
| `styles/index.ts` + `styles/*` | 模块化 CSS-in-JS 样式入口；按 layout / panels / task / terminal / dialogs / common 拆分 |
| `App.css` | 仅用于暗色/亮色主题的 CSS 自定义属性 |
| `utils.ts` | UI 工具函数：头像颜色生成、路径缩短、localStorage 辅助方法 |

**组件树（简化版）：**
```
App
├── WelcomePage              — 项目选择页
└── ProjectPage
    ├── ProjectRail          — 左侧导航栏（项目切换器）
    ├── TaskPanel            — 任务列表侧边栏
    │   ├── BranchBar        — Git 分支切换 / 创建
    │   ├── TaskList → TaskListItem
    │   └── SidebarFooterActions → AppSettingsDialog
    ├── NewTaskView          — 任务创建视图
    │   ├── PromptEditor
    │   ├── MentionPopover
    │   ├── ImageAttachments
    │   └── AgentPermSelector
    ├── TodoTaskView         — Todo 任务编辑 / 启动视图
    ├── RunningView          — 运行中任务头部（恢复、取消）
    │   └── TerminalView     — xterm.js 封装组件
    ├── SessionView          — 会话消息查看器（JSONL 回放）
    ├── AnalyticsDashboard   — 每周 token / 工具调用用量图表
    ├── ShellTerminalPanel   — 嵌入式交互 Shell 终端
    ├── SettingsDialog       — 项目级设置（config.toml 编辑器 + 智能体配置）
    ├── RightToolbar         — 右侧面板与 Shell 的开关入口
    └── 右侧面板（同时只有一个处于激活状态）：
        ├── FileExplorer → FileViewer → ImagePreviewPane
        ├── GitChanges → GitDiffViewer     — 暂存/未暂存变更、提交 UI
        └── GitHistory → GitDiffViewer     — 提交日志、提交差异查看器
```

状态从 `App.tsx` 通过 props 向下传递；异步更新通过 Tauri 事件向上传递：
- `agent-output` — `{ task_id, data }` 运行中任务的原始 PTY 字节流
- `task-status` — `{ task_id, status }` 任务生命周期状态变更
- `task-session` — `{ task_id, session_id, session_path }` 会话发现
- `shell-output` — `{ shell_id, data }` 嵌入式 Shell 的 PTY 字节流

### 后端（`src-tauri/src/`）

`lib.rs` 是精简的入口点，负责注册所有模块和 Tauri 命令处理列表。业务逻辑拆分到各专职模块：

| 模块 | 职责 |
|--------|---------------|
| `pty.rs` | 任务和 Shell 的 PTY 创建/读写（`run_task`、`resume_task`、`cancel_task`、`send_input`、`resize_pty`、`open_shell`、`kill_shell`） |
| `session.rs` | Claude & Codex 的会话文件监听；终端输出兜底提取；`read_session_messages` |
| `storage.rs` | 基于文件的持久化（`load_projects`、`save_projects`、`load_project_tasks`、`save_project_tasks`） |
| `fs.rs` | 文件系统命令（`read_dir_entries`、`read_file_content`、`read_image_preview`、`write_file_content`、`list_project_files`） |
| `git.rs` | 完整 Git 集成：状态、分支、日志、差异、暂存/取消暂存、提交、推送、拉取、`generate_commit_message` |
| `analytics.rs` | 解析会话 JSONL 获取 token / 工具调用指标（`read_session_metrics`、`get_weekly_analytics`） |
| `config.rs` | 项目级 `.nezha/config.toml` 管理（`init_project_config`、`read_project_config`、`write_project_config`、`read_agent_config_file`、`write_agent_config_file`） |
| `app_settings.rs` | 应用级智能体路径与版本管理（`load_app_settings`、`save_app_settings`、`detect_agent_paths`、`detect_agent_versions`） |

核心结构体：`TaskManager` 由 Tauri 托管状态持有，内部使用 `parking_lot::Mutex` 管理 PTY 主端、写入器、子进程句柄、会话映射及已认领的会话路径。

**权限模式 → CLI 标志映射：**
| 模式 | 标志 |
|------|------|
| `ask` | `--permission-mode default` |
| `auto_edit` | `--permission-mode acceptEdits` |
| `full_access` | `--dangerously-skip-permissions` |

---

## 数据模型

```typescript
// 任务状态生命周期：
// todo → pending → running ↔ input_required → done | failed | cancelled

interface Task {
  id: string;
  projectId: string;
  name?: string;
  prompt: string;
  agent: "claude" | "codex";
  permissionMode: "ask" | "auto_edit" | "full_access";
  status: TaskStatus;
  createdAt: number;
  attentionRequestedAt?: number;
  starred?: boolean;
  failureReason?: string;
  claudeSessionId?: string;
  claudeSessionPath?: string;
  codexSessionId?: string;
  codexSessionPath?: string;
}
```

**持久化存储（基于文件，非 localStorage）：**
- `~/.nezha/projects.json` — Project[]
- `~/.nezha/projects/<projectId>/tasks.json` — Task[]（每个项目独立一个文件）
- 主题及 UI 偏好存储于 localStorage

**Tauri 存储命令（`src-tauri/src/storage.rs`）：**
- `load_projects` / `save_projects`
- `load_project_tasks` / `save_project_tasks`

> 修改 Task 数据结构时，**必须同步更新 `types.ts`（TypeScript）和 `storage.rs` 中的 `Task` 结构体（Rust）**——否则新字段在序列化时会被静默丢弃。

---

## 项目配置

每个项目首次打开时会自动创建 `.nezha/config.toml`（由 `init_project_config` 触发）：

```toml
[agent]
default = "claude"        # 新任务的默认智能体
prompt_prefix = ""        # 拼接到每个任务提示词前面的文本
claude_version = ""       # 自动探测并回写的 Claude Code 版本
codex_version = ""        # 自动探测并回写的 Codex 版本

[git]
commit_prompt = "..."     # generate_commit_message 使用的提示词
```

附加到任务的图片会保存至 `.nezha/attachments/<taskId>/`，其路径会被追加到提示词末尾，以便智能体通过文件工具读取。任务完成后附件会被自动清理。

---

## 开发规范

### 样式

- 所有样式统一放在 `src/styles/` 目录下，并通过 `src/styles/index.ts` 聚合导出。新样式优先按已有模块（`layout`、`panels`、`task`、`terminal`、`dialogs`、`common`）归档，不要创建独立的业务 `.css` 文件，也不要使用行内 `style={{}}` 属性（真正一次性的除外）。
- 主题变量（颜色、间距）是 `App.css` 中的 CSS 自定义属性，在 `styles/*` 中通过 `var(--name)` 引用。

### 状态管理


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hanshuaikang/nezha](https://github.com/hanshuaikang/nezha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
