---
trigger: always_on
description: 基于 Claude Agent SDK 的桌面端通用 Agent 产品。开源项目（Apache-2.0），使用 Conventional Commits，不提交敏感信息。
---

# MyAgents - Desktop AI Agent

基于 Claude Agent SDK 的桌面端通用 Agent 产品。开源项目（Apache-2.0），使用 Conventional Commits，不提交敏感信息。

## 技术栈

| 层级 | 技术 |
|------|------|
| 桌面框架 | Tauri v2 (Rust) |
| 前端 | React 19 + TypeScript + Vite + TailwindCSS |
| 后端 | Bun + Claude Agent SDK (多实例 Sidecar) |
| 通信 | Rust HTTP/SSE Proxy (reqwest via `local_http` 模块) |
| 运行时 | 双运行时：Bun（Agent Runtime / Sidecar）+ Node.js（MCP Server / 社区工具），均内置于应用包 |

## 项目结构

- `src/renderer/` — React 前端（api/、context/、hooks/、components/、pages/）
- `src/server/` — Bun 后端 Sidecar
- `src/server/plugin-bridge/` — OpenClaw Plugin Bridge（独立 Bun 进程，加载社区 Channel 插件）
- `src/cli/` — 自配置 CLI（`myagents` 命令，同步到 `~/.myagents/bin/`，详见下方说明）
- `src/shared/` — 前后端共享类型
- `src-tauri/` — Tauri Rust 层
- `specs/` — 设计文档（tech_docs/、guides/、prd/、research/）
- `bundled-agents/myagents_helper/` — 内置 MA 小助理（见下方说明）

### 内置 MA 小助理（`bundled-agents/myagents_helper/`）

应用内置了一个 AI 助手（MA 小助理），运行在 `~/.myagents/` 工作区中，职能是产品首席客服 — 帮用户诊断问题、配置工具、管理 Agent。

**核心机制**：小助理通过 `/self-config` Skill 调用内置 `myagents` CLI 工具，**直接执行**用户请求的管理操作（配置 Provider、安装 MCP、管理 Agent Channel、创建定时任务等），而不是输出操作步骤让用户自己做。CLI 通过 Admin API（`/api/admin/*`）与 Rust Management API 通信，能力与 GUI 对等。

**文件结构**：
- `CLAUDE.md` — 小助理的元认知（架构速览、日志格式、错误速查表、诊断工作流）
- `.claude/skills/self-config/SKILL.md` — CLI 操作技能（MCP/Provider/Agent/Cron/Plugin CRUD）
- `.claude/skills/support/SKILL.md` — 用户支持技能（日志分析、Bug Report 生成）

**开发约束**：
- 修改 `bundled-agents/myagents_helper/` 的 CLAUDE.md 或 Skills 后，MUST bump `ADMIN_AGENT_VERSION`（`src-tauri/src/commands.rs`），否则用户端小助理不会更新。
- 修改 `src/cli/myagents.ts` 或 `src/cli/myagents.cmd` 后，MUST bump `CLI_VERSION`（`src-tauri/src/commands.rs`），否则用户端 CLI 不会更新。两个版本门控独立运作。

## 开发命令

```bash
bun install                 # 依赖安装
./start_dev.sh              # 浏览器开发模式 (快速迭代)
npm run tauri:dev           # Tauri 开发模式 (完整桌面体验)
./build_dev.sh              # Debug 构建 (含 DevTools)
./build_macos.sh            # 生产构建
./publish_release.sh        # 发布到 R2
npm run typecheck && npm run lint  # 代码质量检查
```

---

## 核心架构约束

### 第一原则：架构延续性

**每个功能都在已有架构上生长，而不是另起炉灶。** 项目已有成熟的分层设计、通信模式、安全约束和前端规范。新功能 MUST 复用现有模块和模式（如 `local_http`、`process_cmd`、`broadcast()`、`awaitSessionTermination()`），禁止为单点需求发明新的技术方案。

开发前 MUST 做的三件事（这两份文档是项目的全局核心规范，渐进式按需读取）：
1. **读架构文档** @specs/ARCHITECTURE.md — 理解系统分层、模块边界、数据流；从这里再下钻到 `specs/tech_docs/*` 的专题文档
2. **读设计规范** @specs/DESIGN.md — 前端开发 MUST 遵循 Token/组件/页面规范
3. **搜索现有实现** — 先在代码库中搜索类似功能是否已有模式，复用而非重建

如果需求确实需要架构变更（新的通信模式、新的状态管理方式、新的进程类型），MUST 先与用户讨论方案，不得自行引入。对接外部 SDK/插件时，MUST 先读源码确认接口约定（函数签名、config schema、返回值格式），再写适配层。

### Claude Agent SDK 交互规范

项目的核心 AI 运行时是 Claude Agent SDK（`@anthropic-ai/claude-agent-sdk`），所有 Agent 会话、工具调用、子 Agent 派发都通过它驱动。SDK 持续迭代，API 行为、环境变量、消息类型可能随版本变更。

**禁止凭假设编写 SDK 交互代码。** 涉及 SDK 的任何开发（`query()` 参数、`SDKMessage` 类型处理、环境变量设置、Hook 注册、MCP 集成等），MUST 先查阅官方文档确认实际行为：
- **SDK 文档**：https://platform.claude.com/docs/zh-CN/agent-sdk/overview
- **SDK 类型定义**：`node_modules/@anthropic-ai/claude-agent-sdk/sdk.d.ts`（当前版本 0.2.111）
- **SDK 工具类型**：`node_modules/@anthropic-ai/claude-agent-sdk/sdk-tools.d.ts`

典型错误案例：臆测 `seedReadState` 的调用时机导致"先读后改"语义被绕过、臆测环境变量名导致模型别名不生效。这类问题的根因都是没有查文档就动手写代码。

### Tab-scoped 隔离

每个 Chat Tab 拥有独立的 Bun Sidecar 进程（Tab/CronTask/BackgroundCompletion/Agent 四种 Owner 共享 SidecarManager）。MUST 在 Tab 内使用 `useTabState()` 返回的 `apiGet`/`apiPost`，**禁止**在 Tab 内使用全局 `apiPostJson`/`apiGetJson`（会发到 Global Sidecar）。

### Rust 代理层

所有前端 HTTP/SSE 流量 MUST 通过 Rust 代理层（`invoke` → Rust → reqwest → Bun Sidecar），**禁止**从 WebView 直接发起 HTTP 请求。

### local_http 模块（致命陷阱）

所有连接本地 Sidecar（`127.0.0.1`）的 reqwest 客户端 MUST 通过 `crate::local_http::builder()` / `blocking_builder()` / `json_client()` / `sse_client()` 创建。内置 `.no_proxy()` 防止系统代理拦截 localhost。**禁止**裸 `reqwest::Client::builder()` 或 `reqwest::Client::new()` 连接 localhost，否则系统代理（Clash/V2Ray）会导致 502。

### process_cmd 模块（Windows 控制台窗口陷阱）

所有 Rust 层子进程 MUST 通过 `crate::process_cmd::new()` 创建，**禁止**裸 `std::process::Command::new()`。内置 Windows `CREATE_NO_WINDOW` 标志，防止 GUI 应用启动子进程（bun.exe Sidecar / Plugin Bridge / bun init 等）时弹出黑色控制台窗口。遵循与 `local_http` 相同的 "pit of success" 模式。例外：`#[cfg(windows)]` 守卫内的系统工具命令（taskkill/powershell/wmic）已内联处理；`commands.rs` 的 OS opener（open/explorer/xdg-open）和 Unix pgrep 是用户可见的系统命令，无需隐藏；`terminal.rs` 的 PTY 进程由 `portable-pty` 的 `CommandBuilder` + `slave.spawn_command()` 管理，不走 `std::process::Command`。

### proxy_config 子进程代理策略（Bun fetch 陷阱）

所有可能发起 HTTP 请求的 Rust 层子进程（Bun Sidecar、Plugin Bridge、npm install 等）MUST 在 spawn 前调用 `crate::proxy_config::apply_to_subprocess(&mut cmd)`。该函数确保：用户配置代理时注入 `HTTP_PROXY` + `NO_PROXY`；未配置时继承系统网络行为但**始终注入 `NO_PROXY`** 保护 localhost。**禁止**手动 `cmd.env("HTTP_PROXY", ...)` 或 `cmd.env_remove("HTTP_PROXY")`。Bun 的 `fetch()` 会读取 `HTTP_PROXY` 环境变量，没有 `NO_PROXY` 的话，Sidecar 内部的 localhost 通信（admin-api、cron-tool、bridge-tools 等）会被系统代理拦截 → 502。

### 零外部依赖与双运行时

应用内置三个运行时依赖，用户无需自行安装任何东西：

| 依赖 | 用途 | 打包位置 |
|------|------|---------|
| **Bun** | Agent Runtime — 运行 Claude Agent SDK（`executable: 'bun'`），Sidecar 主进程、Plugin Bridge | `src-tauri/binaries/bun-*` |
| **Node.js** | 功能层 — MCP Server 执行（`npx`）、社区 npm 包、AI Bash 环境中的 `node`/`npx`/`npm` | `src-tauri/resources/nodejs/`（v0.1.44+，含 node + npm/npx） |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hAcKlyc/MyAgents](https://github.com/hAcKlyc/MyAgents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
