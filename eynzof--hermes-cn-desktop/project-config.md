---
trigger: always_on
description: Hermes Agent CN 桌面端 — 用 Tauri v2 + React 构建的独立桌面应用，替代原 Electron 壳。
---

# Codex 工作指引

## 项目概述

Hermes Agent CN 桌面端 — 用 Tauri v2 + React 构建的独立桌面应用，替代原 Electron 壳。
对接后端是 [Hermes-CN-Core](https://github.com/Eynzof/Hermes-CN-Core)（CN 核心 runtime，原名 hermes-agent-cn）内置 Dashboard；桌面端 managed runtime 默认使用端口 9120，避开用户全局 Hermes Agent 常用的 9119。当前版本 **0.5.4**，bundle identifier `cn.org.hermesagent.desktop`（升级承重标识，勿改）。

## 项目结构

```
Hermes-CN-Desktop/
├── src/                    Rust Tauri 后端（~24,000 行，crate lib 名 hermes_agent_cn）
│   ├── main.rs               入口：解析 HERMES_HOME、启动 dashboard、注册 60 个命令（generate_handler!）、系统托盘
│   ├── lib.rs / state.rs     库入口（声明 18 个 module）+ AppState（Mutex<AppStateInner>）
│   ├── tray.rs               系统托盘菜单
│   ├── error.rs              AppError 统一错误类型
│   ├── environment.rs / bootstrap.rs / connection.rs / path_resolver.rs / env_file.rs
│   ├── supervisor.rs / prevent_sleep.rs / cron_runs.rs / update_stage.rs / util.rs / ui_store.rs
│   ├── session_archive.rs / session_log.rs   会话归档与日志读取
│   ├── commands/             60 个 #[tauri::command]（22 个模块，列表见 main.rs 的 generate_handler!）
│   │   ├── api_proxy.rs         HTTP 代理（api_request / external_request / upload_file）
│   │   ├── ws_proxy.rs          /api/ws WebSocket 中继（webview 原生 WS 被拦时的兜底）
│   │   ├── gateway.rs           runtime config + gateway URL 刷新
│   │   ├── runtime_manager.rs   managed runtime 下载/更新/回滚
│   │   ├── desktop_update.rs    桌面端自更新
│   │   ├── profiles.rs          profile 切换（含故障恢复）
│   │   ├── config_migration.rs  配置迁移
│   │   ├── im_onboarding.rs     飞书/钉钉/企微/微信 接入引导
│   │   └── connection/memory/skills/terminal/backup/log_export/debug_bundle/notify/
│   │       preview/environment/file_dialogs/restart/ui_store/yolo/mod.rs
│   └── process/
│       ├── dashboard.rs         dashboard 子进程管理（probe/spawn/port fallback）
│       ├── gateway.rs           gateway 子进程 / 冲突检测
│       └── runtime.rs           managed runtime 安装/签名验证
├── web/                    React 前端（Vite + TanStack Query + Jotai）
│   ├── src/
│   │   ├── lib/tauri-bridge.ts    Tauri invoke 包装 + hermesDesktop shim
│   │   ├── lib/runtime.ts         平台检测（web / electron / tauri）
│   │   ├── lib/transport.ts       HTTP 路由（native IPC vs fetch）+ auth header 注入
│   │   ├── lib/gateway-client.ts  网关 WS 客户端（JSON-RPC over /api/ws，退避/唤醒重连/session.resume）
│   │   └── lib/gateway-socket-path.ts  原生 WS vs Rust 中继的 socket 路径选择与自动回退
│   └── vite.config.ts
├── packages/
│   ├── protocol/              Zod schemas（hermes-api.ts）、IPC 类型、会话日志解析
│   └── shared-ui/             设计 token（tokens/*.css）、components/composites/hooks
├── e2e/                       Playwright E2E（真实 web → 真实 Core 后端 → 本地 fake model）
├── tests/                     Rust 集成测试（crate 名 hermes_agent_cn）
├── static/                    打包 stage 目标（bundled-runtime / -skills / -plugins / dashboard）
├── Cargo.toml                 Rust 依赖
├── tauri.conf.json            Tauri 窗口/打包/CSP 配置
├── pnpm-workspace.yaml        pnpm monorepo（web + packages/* + e2e）
└── package.json               workspace root + 构建脚本
```

## 后端事实来源

UI 对接的是 hermes-agent Dashboard。**不要凭参数名猜后端行为**。

后端源码在同级的 `../Hermes-CN-Core`（`pnpm tauri:dev` 默认从这里把 backend 装进桌面 dev-runtime，可用 `--source` 覆盖）。查：
- REST 路由：`hermes_cli/web_server.py`
- Gateway 事件：`tui_gateway/server.py`
- 上游 Web 实现：`web/src/lib/api.ts`、`gatewayClient.ts`

## 开发流程

### 开发前预检（双仓同步 + Worktree 隔离）

Hermes CN 的需求与 bug 修复通常**同时横跨 Desktop 与 Core 两个仓库**。正式动手写代码前，两个仓库都必须先过这道预检，**不要直接在 `main` 上改**：

1. **确认主分支已与远端同步**。对 Desktop 与 Core 分别 `git fetch origin`，确认本地 `main` 与 `origin/main` 一致（`git rev-list --left-right --count main...origin/main` 应为 `0  0`）；落后就先快进，工作区脏就先收拾干净。
2. **为每个仓库开独立的功能分支 + git worktree**，让 Desktop 与 Core 的改动互不干扰、可并行：
   ```bash
   git -C <repo> fetch origin
   git -C <repo> worktree add ../wt/<repo>-<topic> -b <branch> origin/main
   ```
   分支命名沿用 Conventional 风格（`feat/` `fix/` `docs/` `chore/` …）。同一任务在两仓用同名分支，方便对应。
3. 不要在同一个工作目录里来回 `git checkout` 切分支——双仓并行时极易串味；每条线一个 worktree。

**收尾流程（每个仓库都要走完，缺一不可）**：改完 → `pnpm typecheck && pnpm test:unit && cargo check` → commit → push → 开 PR → **盯 PR 上 GitHub Actions 的构建与测试全绿**（`rust-test.yml` / `web-test.yml`），没过就回去修，别把任务当完成。

### 仓库技能

开始任何工作前，必须先阅读项目结构技能：`.codex/skills/project-structure/SKILL.md`。

双仓库（Desktop + Core）最新分支启动、dev 冒烟或打包态补验，必须使用：
`.codex/skills/desktop-dual-repo-test/SKILL.md`。

发版、版本号更新、安装包发布或 GitHub Release 相关任务必须按顺序使用仓库内技能：**先过** `.codex/skills/desktop-release-preflight/SKILL.md`（发版前安全闸门：防内核静默降级 / 防 schema 重置 / identifier 不变 / 公证签名 / 国内镜像先有 artifactUrl 再发清单 / 先发 canary），**再做** `.codex/skills/desktop-release-sync-landing/SKILL.md`（版本同步与官网清单）。
只要桌面端公开版本发生变化，就必须同步处理 `Eynzof/hermes-agent-cn-desktop-landing`，
更新官网版本与 `https://desktop.hermesagent.org.cn/latest.json` 清单；如果 release 资产尚未生成，
需要明确说明 Landing 同步被阻塞，不能把桌面端发版任务当作已经完整结束。

### 启动顺序

一步起 Tauri dev（推荐）。`pnpm tauri:dev` 会先 `version:sync`，再跑 `scripts/tauri-dev-managed.mjs`：把后端装进桌面 dev-runtime、禁用 PATH 上的全局 hermes，再启动 Tauri dev（自动加载 Vite devUrl 9545）：

```bash
pnpm tauri:dev                                 # 托管 runtime（默认 source = ../Hermes-CN-Core）
pnpm tauri:dev -- --source ../Hermes-CN-Core   # 指定本地后端源码安装进 runtime
# pnpm tauri:dev:external 现已是 deprecated 别名：桌面端锁 managed runtime，跑的就是和 tauri:dev 相同的 managed 路径
```

手动分步（调试 Rust 时用）：
```bash
hermes dashboard --no-open   # 终端 1：先起后端 Dashboard

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eynzof/Hermes-CN-Desktop](https://github.com/Eynzof/Hermes-CN-Desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
