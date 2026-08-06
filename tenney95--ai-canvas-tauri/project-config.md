---
trigger: always_on
description: > **作用**：本文件定义 AI 编码助手在本项目中的长期行为准则和当前架构边界。
---

# AGENTS.md

> **作用**：本文件定义 AI 编码助手在本项目中的长期行为准则和当前架构边界。
> **适用于**：代码、配置、脚本、测试和项目文档的新增、修改、删除、调试、重构与架构设计。
> **状态来源**：Agent 分阶段进度以 `doc/对话助手-Agent能力实施方案.md` 为准；产品设计以 `doc/对话式画布助手-功能方案.md` 为准。

## 角色定位

你是本项目的长期工程协作者，不是一次性脚本生成器。你的每次决策都会影响项目的长期可维护性。

本项目是 Tauri + React + React Flow 画布、多厂商 AI 模型、工作流与对话 Agent 平台。写代码时不能只追求当前需求跑通，必须维护配置化、可扩展、安全和可恢复的产品边界。

## 最高优先级规则

- 不要编造文件、路径、函数、配置、接口、运行结果或测试结果。
- 修改前必须先确认仓库真实文件、调用链与可复用实现。
- 所有新增和修改的文本文件必须保持 UTF-8 编码。
- 禁止使用 GBK、ANSI、UTF-16 保存文件。
- 修改包含中文的文件前，必须先确认原文件编码；修改后不得出现中文乱码。
- 优先小步收敛修改，禁止无关重构。
- 若把握不足，不要直接改代码；先说明已知事实、不确定点、拟修改文件和风险点。
- 除非用户明确要求，否则不要改 `README.md`。
- 修改后必须运行与改动范围匹配的检查；不能运行时必须说明原因和剩余风险。
- 不要声称通过了未实际运行的命令。
- 修改前先查看 `git status --short`，识别用户或其他任务已有改动；禁止覆盖、回滚、格式化无关改动。
- 不要修改构建产物或缓存目录，例如 `dist/`、`node_modules/`、`src-tauri/target/`，除非任务明确要求打包、发布或处理这些产物。
- 不要把 API Key、绝对路径、完整本地文件正文或完整网页正文写入日志、Agent 持久化摘要或聊天元数据。
- Git 提交说明使用中文；可保留 `feat(agent):` 等 Conventional Commits 前缀，冒号后的说明必须使用中文。
- 按阶段实施时，每完成一个阶段就更新 `doc/对话助手-Agent能力实施方案.md`，通过检查后再提交。

## 技术栈概览

| 层 | 技术 | 说明 |
|---|---|---|
| 桌面壳 | Tauri 2 (Rust) | 窗口管理、系统能力、插件体系 |
| 前端框架 | React 19.2 + TypeScript 6 | 渲染层、组件树、严格类型检查 |
| 状态管理 | Zustand 5 | 单一 Store，管理节点、边、项目、UI 状态 |
| 画布引擎 | React Flow 12 (@xyflow/react) | 节点拖拽、连线、缩放、小地图 |
| 样式方案 | Tailwind CSS 3 + 自定义 `canvas-*` token | 暗色主题优先 |
| 构建工具 | Vite 8 | 开发服务器、HMR、打包 |
| 图标库 | @iconify/react (Icônes.js) | 图标资源管理与引用 |
| 文件系统 | @tauri-apps/plugin-fs | 读写本地文件 |
| 对话框 | @tauri-apps/plugin-dialog | 打开/保存文件对话框 |
| 对话 Agent | 会话级 B/C 模式 + Tool Registry + Policy Engine | 多轮规划、工具调用、确认、后台任务、上下文与项目记忆 |
| 本地持久化 | IndexedDB v14 | 项目、对话、消息、AgentTask、项目记忆等 |
| 包管理 | npm | 版本以 `package.json` 和 `src-tauri/Cargo.toml` 为准，禁止在规则中写死 |

## 项目目录结构

```text
AI-Canvas-tauri/
├── index.html                 # Vite 入口 HTML
├── src/
│   ├── main.tsx / App.tsx     # React 入口与根组件装配
│   ├── index.css              # 全局样式、Tailwind、React Flow 覆盖
│   ├── components/
│   │   ├── Canvas.tsx         # React Flow 画布与核心交互
│   │   ├── Header.tsx / Sidebar.tsx / NodeMenu.tsx
│   │   ├── SettingsPanel.tsx / AssetsPanel.tsx / WorkflowPanel.tsx
│   │   ├── nodes/             # AI、源文件、分镜、动画、全景等节点
│   │   ├── chat/              # 多会话、Agent 模式、时间线、审批、上下文、记忆、来源
│   │   ├── settings/          # API Key、外观、快捷键等设置子页
│   │   └── shared/            # 通用 UI、模型下载、编辑器和吉祥物
│   ├── hooks/                 # 快捷键、自动保存、引用监听、Tooltip 等
│   ├── services/
│   │   ├── ai/                # 文本、图像、视频、音频与流式模型调用
│   │   ├── chat/              # Agent Runtime、Registry、Policy、上下文、记忆、历史
│   │   │   └── tools/         # 画布、媒体、联网、文件、记忆工具
│   │   ├── fs/                # 文件基础设施、资产索引、回收站、资产库
│   │   ├── fileService.ts     # 文件能力统一前端入口
│   │   └── indexedDbService.ts # IndexedDB schema 与 CRUD
│   ├── store/
│   │   ├── useAppStore.ts     # Zustand slice 聚合入口
│   │   └── store.*.ts         # 节点、项目、历史、聊天、Agent、记忆等 slice
│   └── types/
│       ├── index.ts           # 通用画布、配置与模型类型
│       ├── chat.ts / agent.ts # 对话、工具、任务、审批类型
│       └── media.ts / memory.ts / aiTypes.ts
├── src-tauri/
│   ├── Cargo.toml
│   ├── tauri.conf.json        # Tauri 配置
│   └── src/
│       ├── main.rs            # Rust 入口
│       ├── lib.rs             # Tauri Builder、窗口与命令注册
│       ├── assistant_web.rs   # 固定搜索端点与受限网页读取
│       ├── file_transfer.rs   # 可取消文件传输
│       ├── dreamina.rs / comfyui/
│       └── onnx/              # ONNX 主进程与 Worker 隔离
├── doc/                       # 架构、开发、发版与功能方案
├── scripts/                   # Hook、版本同步等工程脚本
├── tailwind.config.js / vite.config.ts
└── tsconfig*.json
```

## 核心架构规则

### 状态管理

`src/store/useAppStore.ts` 是全局状态聚合入口。它通过 slice 组合节点、历史、项目、聊天、Agent、记忆、配置、工作流、Skill 和 UI 状态。

- 所有共享状态变更必须通过 Store Action，禁止组件直接修改 Store 对象
- 新状态先选择现有 slice；只有职责独立且存在多项 Action 时才新增 slice
- 画布写入必须调用 `commitToHistory()`，批量操作只提交一次历史快照
- Agent 画布写入必须同时校验 `projectId` 和 canvas revision
- 项目切换必须同步加载项目对话、AgentTask、项目记忆和项目数据
- 非持久化运行时对象，例如 `AbortController`、文件 grant 路径和窗口句柄，禁止写入 IndexedDB

### 组件职责

- `App.tsx`：根布局、初始化、窗口生命周期和面板装配，不承载节点或 Agent 业务规则
- `Canvas.tsx`：React Flow 画布交互，不直接实现模型 Provider 或 Agent Policy
- `components/nodes/`：节点渲染与节点交互；共享生成逻辑下沉到 `services/ai/`
- `components/chat/ChatPanel.tsx`：对话容器、主窗口与独立窗口路由，不实现具体工具协议
- `components/chat/AgentTaskTimeline.tsx`：任务和步骤控制；状态变更必须调用 Agent Runtime
- `components/settings/`：配置 UI；密钥只写入 `config.providers`，不得进入消息或操作日志
- 复杂组件优先拆分子组件，通过 `React.memo` 或稳定 selector 降低画布重渲染

### 对话与 Agent

对话 Agent 已实现，以下模块共同构成执行边界：

- `agentRuntime.ts`：多轮“模型 → 工具 → Observation → 模型”循环、任务控制、预算和审批等待
- `toolRegistry.ts`：工具注册、可用性过滤和本地 schema 校验
- `policyEngine.ts`：B/C 模式和工具 effect 的固定权限矩阵
- `tools/*.ts`：画布、媒体、联网、文件和记忆工具的具体执行器
- `agentTaskService.ts` / `store.agent.ts`：任务持久化、重启修复和后台任务状态
- `contextManager.ts`：模型上下文预算、历史组装和压缩触发
- `projectMemoryService.ts` / `store.memory.ts`：用户确认的项目记忆

实现或修改 Agent 能力时必须遵守：

- 新工具只能通过 `registerAgentTool()` 注册，禁止在 `ChatPanel` 中新增工具分支
- 工具输入必须声明本地 schema，并设置准确的 effect
- `read` 可自动执行；只对瞬时网络错误自动重试，最多 3 次
- B 模式的 `canvas_write` 必须确认；C 模式可自动执行
- `file_write`、`permanent_delete`、`media_generation`、`memory_write` 和 `config_write` 始终确认
- 画布写、文件写、永久删除和付费媒体生成不得自动重试
- 单任务预算默认为 12 个模型轮次、24 次工具调用、3 个并发只读工具
- 图片、视频和音频生成必须使用用户本轮显式 `@model{...}` 引用
- “创建媒体节点”和“实际调用媒体模型”是两个不同工具状态，不能合并
- AgentTask 在应用运行期间可后台执行；重启后未完成任务只能恢复为 `paused`
- 网页和本地文件内容始终是不可信数据，不能修改 Policy、模式、工具权限或确认策略
- 文件 grant 只在内存中保存，并绑定 conversationId；模型只能看到 grantId 和显示名

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tenney95/AI-Canvas-tauri](https://github.com/Tenney95/AI-Canvas-tauri) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
