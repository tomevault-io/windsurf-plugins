---
trigger: always_on
description: > 面向 Cursor / AI Agent 的项目速查。详细设计见 `docs/INDEX.md`、`docs/ARCHITECTURE.md`、`docs/MODULES.md`。
---

# Hermes Desktop — Agent 编码指南

> 面向 Cursor / AI Agent 的项目速查。详细设计见 `docs/INDEX.md`、`docs/ARCHITECTURE.md`、`docs/MODULES.md`。

## 项目是什么

**hermes-desktop** 是基于 Electron + React + TypeScript 的 AI 助手桌面应用：安装/配置 Hermes Agent、管理 Gateway、聊天、多 Profile 运行时（Portal Desktop）。

| 项 | 值 |
|---|---|
| 版本 | 0.3.6（… + **v9.0.0 Serve-First Runtime Phase 0/1/2/3** + **v8.2.0 Persistent Chat Workspace + Session Catalog** + **v8.1.1 Durable Runtime Closure** + **v8.1.0 Durable Chat Runtime** + **v8.0.5 Chat Interaction Loop** + **v8.0.4 Chat Turn Lifecycle** + **v8.0.3 Chat Workspace Layout** + **v8.0.2 Chat Full Integration** + **v8.0.1 Chat 迁移闭环** + **v7.6 Hermes Agent MCP Host Mode** + **v7.5.1 Runtime Skill Fixed Route** + **v7.4.2 Chat-first Work Controls** + …） |
| appId | `com.smc.smc-ai-copilot`（productName: **SMC-Copilot**；主程序 **desktop.exe**） |
| 后端 | Hermes Python Gateway，`http://127.0.0.1:8642`（default Profile） |

## 架构：三层进程（必须遵守）

```
Renderer (React)  →  window.hermesAPI / smcShell / desktopAuth / desktopUserConfig / profileRuntime / aiosBrowser / shellView
       ↓ ipcRenderer.invoke
Preload (contextBridge)  →  src/preload/index.ts + auth-api / shell-api / user-config-api
       ↓ IPC
Main (Node.js)  →  src/main/index.ts + 各 *-ipc.ts / *.ts 模块
       ↓ HTTP / child_process
Portal Auth Backend (:8000)  +  Hermes Python Gateway (:8642)
```

**硬性规则：**

- 渲染进程**禁止** `import` Node 模块；只通过 `window.*` API 访问主进程
- **禁止**猜测新增 IPC channel；新通道必须：Main 注册 handler → Preload 封装 → `index.d.ts` 类型 → `docs/API_CONTRACTS.md`
- **禁止**绕过 Preload 使用 `ipcRenderer`（除 Preload 自身）
- 改 Gateway 启停/进程管理前，先读 `src/main/hermes.ts` 与 `profile-runtime-manager.ts`
- 不要改 `electron-builder.yml` 的 appId / publish、i18n 语言代码与回退链
- **禁止**修改含 `#command by loudon` 的注释行与整块注释（软约束，见 `.cursor/rules/008-loudon-command-comments.mdc`）

## 目录地图

| 路径 | 职责 | 常见改动 |
|---|---|---|
| `src/main/` | 主进程：IPC、Gateway、配置、SQLite、Enterprise Install | 新 IPC、后端逻辑 |
| `src/main/copilot-serve/` | **V1.3** Serve 进程探测 / deploy / logs；**v9.0** production 禁止 spawn/stop | Runtime 进程策略 |
| `src/main/copilot-runtime-client/` | **v9.0** Main-only Serve SDK：pairing / keytar Device Token / handshake / proxy-fetch / SSE scaffold | Serve-First 连接层 |
| `src/main/runtime-adapters/` | **v9.0** Phase 2 Instance/Config/MCP/Diagnostics + **Phase 3 ServeChatRuntimeAdapter**（Session/Task/Files 仍 stub） | cutover |
| `src/main/workspace-chat/` | **team_v1.8** Workspaces Chat：resolve / 模型 / 附件 / SSE 代理到 `copilot-serve` | Chat 面板 IPC |
| `src/shared/workspace-chat/` | `workspace-chat-contract.ts` — Renderer/Main/Preload 共享类型 | 改 chat 契约时 |
| `src/main/browser/` | Web Operator（BrowserView、安全、审计、Tool Server） | 浏览器自动化 |
| `src/main/enterprise/` | 企业安装、本地 zip/git 源、**agent-deps-installer**、**pip-mirror-config** | 安装/预检/Doctor/依赖 |
| `src/main/runtime/` | **V5.3** `runtime-paths.ts` + **`portal-root-resolver.ts`** — hermes/serve/portal 统一路径契约 + `buildCopilotRuntimeEnv` + `resolveEffectivePortalMonorepoRoot` | 改安装目录结构 / 运行时路径 / Portal 根解析 |
| `src/main/window/` | V1.4.1 窗口 IPC（`registerWindowIpc`） | 标题栏按钮 |
| `src/main/shell/` | **main-window-controller.ts**（默认 1280×800）、ShellView、菜单、`portal-view-coordinator` | 主窗口尺寸、壳层 IPC |
| `src/main/auth/` | **V3.3** Auth Client、Token Vault、Endpoint Config、Header 注入 | Portal 登录 + Bearer 注入 |
| `src/main/mcp-skill-gateway-runtime/` | **V6.4** nodeskclaw MCP Skill Gateway 本地 Proxy + Hermes `config.yaml` 注册 | MCP Gateway 远程桥接 |
| `src/main/genehub/` | **V6.5** nodeskclaw GeneHub 连接、设备/Profile 注册、Bundle 安装执行器 | 企业 Skill 同步 |
| `src/main/user-config/` | **V3.3** 本地 bootstrap apply / diff / applier | 登录后桌面配置落盘 |
| `src/main/startup/` | **V3.3.1** `startup-decision.ts`、`startup-ipc.ts` | 启动门控（auth + bootstrap 前置） |
| `src/preload/` | contextBridge：`hermesAPI`、`smcShell`、`desktopAuth`、`desktopUserConfig`、`aiosBrowser`、`profileRuntime`、`profileEntry`、`shellView` | API 桥接（影响面大） |
| `src/renderer/src/screens/MainPage/` | **V2.0** 主界面壳：`MainPage`、`MainTopBar`、`MainViewTabs` 等 | 顶栏 / 工作区 Tabs |
| `src/renderer/src/components/overlay/` | **V5.7.8** `OverlayProvider` / `DialogLayer` / `DrawerLayer` / `NativeShellLayerGate` | 全局弹层 + native layer gate |
| `src/renderer/src/` | React UI：`screens/`、`components/`、`modules/auth/` | **主要 UI 开发区** |
| `src/shared/shell/` | **`main-page-constants.ts`**、**`browser-partitions.ts`**、shell-view-contract、view-contract | 布局尺寸、Session 分区常量、ShellView 契约 |
| `src/shared/workspace/` | **workspace-contract**、**workspace-secondary-nav** | Workspace 模块元数据、二级 panel 导航 |
| `src/renderer/src/workspace/` | **workspace-registry**、workspace-tabs、resolve-workspace | 顶栏 Tab 与 `WorkspaceRenderer` 路由 |
| `resources/profiles/` | Profile 模板 + SOUL.md | Profile 种子配置 |
| `resources/skills/` | 内置技能包 | 新技能 |
| `tests/` | Vitest | 单测/集成测 |
| `docs/` | 架构、契约、Renderer 专项文档（见「文档体系」） | 随功能更新；见 `.cursor/rules/007-sync-project-docs.mdc` |
| `prd/` | 版本 PRD（V2.x MainPage、V5.x Hermes/WebOperator 等） | 对应里程碑任务 |

## Preload 暴露的全局 API

| 全局对象 | 文件 | 用途 |
|---|---|---|
| `window.hermesAPI` | `src/preload/index.ts` | 安装、配置、聊天、会话、模型、技能等；**含 `windowControls`、`getInstallerPrecheck`、`.mcp`（V6.1 MCP Registry）** |
| `window.aiosBrowser` | `src/preload/browser-api.ts` | Web Operator（legacy 13 方法 + **V5.7** frame/snapshot/结构化动作 API + 事件） |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [loudon84/ai-os-desktop](https://github.com/loudon84/ai-os-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
