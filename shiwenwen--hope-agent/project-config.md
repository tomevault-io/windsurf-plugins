---
trigger: always_on
description: 基于 Tauri 2 + React 19 + Rust 的本地 AI 助手桌面应用，内置丰富的 Provider 模板与预设模型，GUI 傻瓜式配置。支持三种运行模式：桌面 GUI（Tauri）、HTTP/WS 守护进程（`hope-agent server`）、ACP stdio（`hope-agent acp`）。
---

# Hope Agent

基于 Tauri 2 + React 19 + Rust 的本地 AI 助手桌面应用，内置丰富的 Provider 模板与预设模型，GUI 傻瓜式配置。支持三种运行模式：桌面 GUI（Tauri）、HTTP/WS 守护进程（`hope-agent server`）、ACP stdio（`hope-agent acp`）。

## 开发命令

```bash
pnpm tauri dev         # 启动开发模式（前端 + Tauri 热重载）
pnpm dev               # 仅前端 Vite 开发服务器
pnpm tauri build       # 构建生产包
pnpm sync:version      # 以 package.json 为单一来源，同步 src-tauri 版本号
pnpm release:verify    # 校验 package.json / src-tauri 版本一致；可附 -- --tag vX.Y.Z
pnpm tsc --noEmit      # 前端类型检查
pnpm lint              # Lint
node scripts/sync-i18n.mjs --check   # 检查各语言翻译缺失
node scripts/sync-i18n.mjs --apply   # 从翻译文件补齐缺失翻译

# Server 模式（HTTP/WS 守护进程）
hope-agent server start              # 前台启动 HTTP/WS 服务
hope-agent server install            # 注册系统服务（macOS launchd / Linux systemd）
hope-agent server uninstall          # 卸载系统服务
hope-agent server status             # 查看服务运行状态
hope-agent server stop               # 停止服务
```

## 提交前检查（强制）

推送（`git push`）前**必须**本地跑一遍以下四条，任一失败都要先修再推——CI 会对同一组检查投红票，红在 CI 上要等 10 分钟反馈。`pnpm install` 后会自动启用 [`.husky/pre-push`](.husky/pre-push) 钩子，该钩子就按这个顺序跑这四条；裸跑也可以：

```bash
cargo fmt --all --check                                                    # 对应 CI: rust.yml fmt job
cargo clippy -p ha-core -p ha-server --all-targets --locked -- -D warnings # 对应 CI: rust.yml clippy job
cargo test  -p ha-core -p ha-server --locked                               # 对应 CI: rust.yml test job
pnpm tsc --noEmit                                                           # 对应 CI: lint.yml 前端类型
pnpm lint                                                                    # 对应 CI: lint.yml ESLint
```

- **clippy / test 只覆盖 `ha-core` + `ha-server`**（CI 也是如此）—— `src-tauri` 不在本地钩子范围内，tauri-specific 的 lint / test 问题请用 `cargo {clippy,test} --workspace` 主动自查
- **fmt 用 `--all`**，覆盖整个 workspace；钩子用的是 `cargo fmt --all --check`，裸跑时用不用 `--all` 都能对齐 CI，不过 `--all` 更稳
- **Rust 版本由仓库根目录 [`rust-toolchain.toml`](rust-toolchain.toml) 固定**，本地与 CI 共用同一版本和组件集合；升级 Rust 时优先改这个文件，再验证 pre-push 五项检查全部通过
- **应急开关**：
  - `HA_SKIP_PREPUSH=1 git push` — 整段钩子跳过（仅限文档 / 纯 `.md` 改动 / 弱网紧急场合）
  - `HA_SKIP_PREPUSH_TEST=1 git push` — 只跳过 `cargo test`（WIP 分支快速推送，test 让 CI 兜底；fmt/clippy/tsc/eslint 仍然跑）
  - 禁止用 `--no-verify`，因为它会把 GPG 签名等其它钩子也一并绕过

## 项目结构

```
Cargo.toml              Workspace 根（members: crates/ha-core, crates/ha-server, src-tauri）
crates/
  ha-core/              核心业务逻辑（零 Tauri 依赖，纯 Rust 库）
  ha-server/            HTTP/WS 服务器（axum，REST API + WebSocket 流式推送）
src-tauri/              Tauri 桌面 Shell（薄壳，调用 ha-core）
src/                    前端（React + TypeScript）
  components/           chat/ settings/ dashboard/ cron/ common/ ui/ 等
  lib/                  Transport 抽象层：transport.ts + transport-tauri.ts + transport-http.ts
  i18n/locales/         12 种语言翻译文件
skills/                 内置技能（bundled skills，随应用发行；当前 ha-settings / ha-skill-creator / ha-find-skills）
```

ha-core 按功能域拆分模块，具体用 `ls crates/ha-core/src/` / `Glob` 查看，无需在此维护清单。主要领域：`agent/`（AssistantAgent + Provider + Tool Loop）、`chat_engine/`、`context_compact/`、`memory/`、`skills/`、`tools/`、`channel/`（IM 渠道）、`subagent/`、`team/`、`cron/`、`acp/`、`dashboard/`、`recap/`、`awareness/`、`config/`、`session/`、`project/`、`plan/`、`ask_user/`、`async_jobs/`、`failover/`、`platform/`、`security/`、`logging/`。

## 技术栈

| 层     | 技术                                                                 |
| ------ | -------------------------------------------------------------------- |
| 前端   | React 19 + TypeScript, Vite 8, Tailwind CSS v4, shadcn/ui (Radix UI) |
| 桌面   | Tauri 2                                                              |
| 服务器 | axum (HTTP/WS), clap (CLI)                                           |
| 后端   | Rust, tokio, reqwest（ha-core 库，零 Tauri 依赖）                    |
| 渲染   | Streamdown + Shiki + KaTeX + Mermaid                                 |
| 多语言 | i18next (12 种语言)                                                  |

## 架构约定

### 分层 & 运行模式

- **三 Crate 架构**：`ha-core`（核心业务逻辑，**零 Tauri 依赖**）/ `ha-server`（axum HTTP/WS 薄壳）/ `src-tauri`（Tauri 桌面薄壳）。业务逻辑全进 ha-core，其它两个只做适配
- **三种运行模式**：`hope-agent`（桌面 GUI）/ `hope-agent server`（HTTP/WS 守护进程，含 install/uninstall/status/stop 子命令）/ `hope-agent acp`（stdio ACP 协议）
- **前后端通信**：前端走 Transport 抽象层（`src/lib/transport.ts`），桌面走 Tauri IPC，server 走 HTTP + WS。**新 invoke 必须同时补齐两套适配**
- **EventBus 事件总线**：`ha-core::EventBus` 替代原 Tauri `APP_HANDLE`，让核心逻辑脱离 Tauri 依赖；Tauri shell / axum server 各自订阅并转发到各自前端通道
- **状态管理**：`ha-core::CoreState`（`tokio::sync::Mutex`），Tauri 走 `State<AppState>`，server 走 axum `Extension`；前端保持轻量 React state
- **Guardian 心跳**：桌面 + server 共用 keepalive，保证后台任务（Channel 轮询、Cron 调度等）持续运行
- **桌面自动更新 / Release 版本单一来源**：`package.json` 是桌面版本的单一真相源；`pnpm version` 生命周期钩子会运行 `scripts/sync-version.mjs`，同步 `src-tauri/Cargo.toml` 与 `src-tauri/tauri.conf.json`。GitHub Release workflow 在 tag 构建前执行 `pnpm release:verify -- --tag vX.Y.Z`，并上传 Tauri updater `latest.json` 工件。Updater 公钥跟仓库 `src-tauri/updater.pub.pem`，私钥仅存本机 `~/.tauri/hope-agent-updater.key` 和 GitHub Secrets，严禁入仓
- **系统服务**：`server install` 在 macOS 注册 launchd plist（`~/Library/LaunchAgents/`），Linux 注册 systemd unit（`~/.config/systemd/user/`）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shiwenwen/hope-agent](https://github.com/shiwenwen/hope-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
