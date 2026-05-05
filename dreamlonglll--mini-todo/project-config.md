---
trigger: always_on
description: 本文档用于介绍 Mini-Todo 项目，帮助 AI 助手快速了解项目结构和开发规范。
---

# Mini-Todo 项目指南

本文档用于介绍 Mini-Todo 项目，帮助 AI 助手快速了解项目结构和开发规范。

## 项目简介

Mini-Todo 是一款基于 **Tauri 2.x + Vue 3 + TypeScript** 开发的 Windows 桌面待办事项管理应用，集成了 AI Agent 调度执行、工作流编排、定时任务等高级功能。

## 技术栈

| 层级 | 技术选型 | 说明 |
|------|----------|------|
| 前端框架 | Vue 3 + TypeScript | 组合式 API，类型安全 |
| UI 组件库 | Element Plus | 包含图标库 @element-plus/icons-vue |
| 状态管理 | Pinia | Vue 官方推荐状态管理 |
| 桌面框架 | Tauri 2.x | 轻量级跨平台桌面框架 |
| 后端语言 | Rust | 高性能，内存安全 |
| 数据库 | SQLite (rusqlite) | 轻量级本地数据库 |
| 拖拽功能 | vuedraggable | Vue 拖拽排序库 |
| 异步运行时 | Tokio | Rust 异步任务调度 |
| 定时调度 | cron (Rust crate) | Cron 表达式解析 |

## 项目结构

```
mini-todo/
├── docs/                           # 文档目录
│   └── 开发文档/                    # 开发相关文档
├── src/                            # Vue 前端源码
│   ├── assets/                     # 静态资源
│   ├── components/                 # Vue 组件
│   │   ├── AgentSettings.vue       # Agent 配置组件
│   │   ├── AgentStatusBadge.vue    # Agent 状态徽章
│   │   ├── CalendarView.vue        # 日历视图
│   │   ├── CronEditor.vue          # Cron 表达式编辑器
│   │   ├── QuadrantView.vue        # 四象限视图
│   │   ├── SchedulerPanel.vue      # 调度面板
│   │   ├── SettingsPanel.vue       # 设置面板
│   │   ├── TitleBar.vue            # 标题栏
│   │   ├── TodoItem.vue            # 待办项组件
│   │   └── TodoList.vue            # 待办列表
│   ├── router/                     # 路由配置
│   ├── stores/                     # Pinia 状态管理
│   │   ├── agentStore.ts           # Agent 相关状态
│   │   ├── appStore.ts             # 应用全局状态
│   │   ├── schedulerStore.ts       # 调度器状态
│   │   └── todoStore.ts            # 待办状态
│   ├── types/                      # TypeScript 类型定义
│   │   ├── agent.ts                # Agent 类型
│   │   ├── scheduler.ts            # 调度器类型
│   │   ├── todo.ts                 # 待办类型
│   │   └── workflow.ts             # 工作流类型
│   ├── utils/                      # 工具函数
│   │   ├── logWindow.ts            # 日志窗口管理
│   │   ├── holiday.ts              # 节假日工具
│   │   └── lunar.ts                # 农历工具
│   ├── views/                      # 页面视图
│   │   ├── AgentLogView.vue        # Agent 执行日志窗口（独立 WebView）
│   │   ├── EditorView.vue          # 待办编辑主视图
│   │   ├── MainView.vue            # 主视图（待办列表）
│   │   ├── SubtaskEditorView.vue   # 子任务编辑视图
│   │   ├── WorkflowView.vue        # 工作流配置视图（独立 WebView）
│   │   ├── NotificationView.vue    # 通知视图
│   │   └── SettingsView.vue        # 设置视图
│   ├── App.vue                     # 根组件
│   └── main.ts                     # 入口文件
├── src-tauri/                      # Tauri/Rust 后端源码
│   ├── src/
│   │   ├── commands/               # Tauri 命令（前后端桥接）
│   │   │   ├── agent_cmd.rs        # Agent 管理命令
│   │   │   ├── data.rs             # 数据导入导出
│   │   │   ├── holiday.rs          # 节假日命令
│   │   │   ├── notification_cmd.rs # 通知命令
│   │   │   ├── prompt_template_cmd.rs # 提示词模板命令
│   │   │   ├── scheduler_cmd.rs    # 调度命令
│   │   │   ├── settings_cmd.rs     # 设置命令
│   │   │   ├── sync_cmd.rs         # 同步命令
│   │   │   ├── todo.rs             # 待办 CRUD 命令
│   │   │   ├── window.rs           # 窗口管理命令
│   │   │   └── workflow_cmd.rs     # 工作流命令
│   │   ├── db/                     # 数据库层
│   │   │   ├── agent_db.rs         # Agent 配置表操作
│   │   │   ├── agent_execution_db.rs # Agent 执行记录操作
│   │   │   ├── connection.rs       # 数据库连接管理
│   │   │   ├── dependency_db.rs    # 任务依赖关系
│   │   │   ├── migrations.rs       # 数据库迁移（v1~v20）
│   │   │   ├── models.rs           # 数据模型定义
│   │   │   ├── prompt_template_db.rs # 提示词模板操作
│   │   │   ├── scheduler_db.rs     # 调度状态操作
│   │   │   └── workflow_db.rs      # 工作流步骤操作
│   │   ├── services/               # 业务服务层
│   │   │   ├── agent/              # Agent 集成
│   │   │   │   ├── runner.rs       # AgentManager + AgentRunner trait
│   │   │   │   ├── claude_code.rs  # Claude Code CLI 实现
│   │   │   │   └── codex.rs        # OpenAI Codex CLI 实现
│   │   │   ├── scheduler/          # 任务调度引擎
│   │   │   │   ├── engine.rs       # 调度主引擎（tick 循环）
│   │   │   │   ├── workflow.rs     # 工作流执行逻辑
│   │   │   │   ├── state_machine.rs # 调度状态机
│   │   │   │   ├── priority_queue.rs # 优先级队列
│   │   │   │   ├── concurrency.rs  # 并发控制
│   │   │   │   └── cron_manager.rs # Cron 定时管理
│   │   │   ├── notification.rs     # 通知服务
│   │   │   └── webdav.rs           # WebDAV 同步
│   │   ├── lib.rs                  # 库入口
│   │   └── main.rs                 # 主入口
│   ├── Cargo.toml                  # Rust 依赖配置
│   └── tauri.conf.json             # Tauri 配置
├── public/                         # 公共静态资源
├── package.json                    # Node 依赖配置
└── vite.config.ts                  # Vite 构建配置
```

## 核心功能

### 待办管理
- 创建、编辑、删除待办事项
- 支持一级子任务（含内容详情、Agent 配置）
- 优先级设置（高/中/低）
- 完成状态标记
- 拖拽排序

### Agent 集成
- **支持的 Agent 类型**：Claude Code CLI、OpenAI Codex CLI
- **Agent 配置管理**：CLI 路径、启用/禁用、健康检查
- **手动执行**：子任务编辑页直接触发 Agent 执行
- **定时调度**：通过 Cron 表达式自动调度子任务
- **执行日志**：独立 WebView 窗口，支持实时流式日志和历史记录查看
- **任务终止与重执行**：运行中的任务可终止，已完成/失败/取消的任务可重新执行

### 工作流系统
- **步骤类型**：执行子任务、执行提示词
- **步骤控制**：启动、停止、继续、重置
- **上下文传递**：步骤可配置"带入上一步结果"，自动继承前一步 Agent 会话
  - Claude Code：使用 `-r session_id` 恢复会话
  - Codex CLI：使用 `resume --last` 模式
- **提示词库**：可创建和管理提示词模板，快速添加到工作流

### 任务调度
- **调度策略**：手动执行、定时执行（Cron）
- **调度状态机**：none → pending → queued → running → completed/failed/cancelled
- **优先级队列**：基于优先级和入队时间排序
- **并发控制**：可配置最大并发执行数
- **任务依赖**：子任务间可设置依赖关系
- **失败重试**：可配置重试次数和间隔

### 通知提醒
- Windows 系统通知

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dreamlonglll/mini-todo](https://github.com/dreamlonglll/mini-todo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
