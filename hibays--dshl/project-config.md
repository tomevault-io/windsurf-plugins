---
trigger: always_on
description: DSHL（DeepSeek Harness Launcher）——一个 webui.me 包装启动器：检查运行环境 → 安装/解析 `@deepseek-ai/dsh` → 启动 `dsh web` → 路由浏览器到它的 URL。根 crate 名是 **`dshl-core`**（源码在仓库根 `src/`，纯 lib）。同一内核两条交付轨道：
---

# Repository Guidelines

## Project Overview

DSHL（DeepSeek Harness Launcher）——一个 webui.me 包装启动器：检查运行环境 → 安装/解析 `@deepseek-ai/dsh` → 启动 `dsh web` → 路由浏览器到它的 URL。根 crate 名是 **`dshl-core`**（源码在仓库根 `src/`，纯 lib）。同一内核两条交付轨道：

- **Track A 安装器**：`crates/dshl`（exe；打包产物 NSIS/deb/dmg，由 `release.yml` 发布）。
- **Track B 插件轨**：`crates/dshl-native`（napi-rs cdylib `.node`）+ `plugins/*` JS 包（`release-native.yml` 发 6 个 `@dshl/native-<platform>-<arch>` 子包；`release-plugins.yml` 发聚合包 `@dshl/native`、`@dshl/pipe`、`@dshl/control`；发布步骤缺 `NPM_TOKEN` secret 时自跳过——no-op 不报错）。

助手铁律：

- **用中文回复**。
- **不要动 git**：不 commit/add/stage、不改 .git 状态。用户手动管理暂存区以便自行审查。
- **改 `t!()` 消息必须同步 `locales/en.yml` 与 `locales/zh-CN.yml`**（rust-i18n，fallback = zh-CN；缺失键退化为中文而非空串）。
- **Windows 系统 API 一律走 `windows-rs 0.62`**（按模块开 feature），不写手写 `#[link] extern "system"` FFI。唯一例外：`crates/dshl-native` 的 napi-rs。
- dsh 的安装/更新**绝不 `-g`**（不污染用户环境）：装进 dshl 缓存，运行时把 bin 目录注入 dsh 进程 PATH。
- 镜像（mirror）永远临时生效（env / CLI 参数），从不写入全局配置。

## Architecture & Data Flow

**入口链**（A/B 两轨仅入口不同，内核同一份）：

- Track A：`crates/dshl/src/main.rs` → `dshl_cli::run_cli()`。壳只做三件事：转发 windows_subsystem 属性、调 run_cli（其内部从不 `process::exit`）、把 `RunOutcome`（HelpPrinted/VersionPrinted/ArgsError/AlreadyRunning）映射为退出码。
- Track B：`crates/dshl-native` 的 `#[napi] launch(...)` → `dshl_cli::run_with_options(...)`。
- 共用参数层 `crates/dshl-cli`：run / control / signal / handle 子命令。

**启动管线**（`src/flow/`，顺序执行）：`system`（OS/arch 检测）→ `runtime_env`（node/bun/pnpm/fnm 探测与就绪）→ `mirror_check`（镜像策略解析）→ `prepare`（安装/解析 dsh，`prepare::run()` 返回 `Command`）→ `launch`（捕获 stdout 里的 URL 并转入监督）。

**dsh 来源策略**（`src/config.rs` `DshMode`）：`global`（严格全局，缺失报错）/ `hybrid`（默认：全局优先，缺失或版本不符落缓存）/ `private`（恒装缓存、不碰全局）。缓存位置 `flow/prepare.rs::dsh_dir()` = `<cache>/dshl`，dsh 是其中的 node module（`node_modules/@deepseek-ai/dsh`），以 `node <bin 入口>` 直接运行（从 package.json 解析 bin）。包管理器可配 npm / bun / pnpm。

**控制面**（`src/control.rs`）：`DSHL_CONTROL_URL=dshl://<token>@127.0.0.1:<port>` 注入 dsh 进程环境；方法 `ping | shutdown | switch-profile | open-terminal | restart`。插件轨 `open-terminal` 优先本地 addon、回落管道；restart/shutdown 对空 client 有防护。

**进程监督**（`src/process/`）：dsh 是受监督子进程，stdout/stderr 逐行进 `<cache>/dshl/dsh.log`；优雅停止（Windows 隐藏控制台 + `GenerateConsoleCtrlEvent` 发 Ctrl+C，Unix SIGTERM），**从不自动强杀**；强杀由 Job Object / PDEATHSIG 兜底。崩溃恢复：5s 倒计时自动重启。

- **UI 分层**（`src/ui/`）：`state.rs` 集中共享状态；`window` / `launch` / `supervisor` / `bindings` / `crash` / `exit` / `vfs` / `geometry`（窗口几何持久化：`<cache>/dshl/window-state.json` + webui 硬限 clamp）各司其职；webui.me 保活 WebSocket 在 `src/wskeep.rs`。

**内嵌终端**（`src/pty/`）：portable-pty + tokio-tungstenite 内嵌独立 WS 服务，页面端 xterm.js（token 经 URL query 校验）。

## Key Directories

| 目录 | 用途 |
|---|---|
| `src/` | dshl-core 内核（见 Architecture） |
| `src/platform/` | OS 原语：detect / paths / process / dpi / theme / window / single_instance / actions |
| `src/tray/` | 托盘图标，每 OS 一个实现（windows / macos / linux），统一 7 函数契约（start / hide_to_tray / quit_requested / restore_requested / open_url_requested / set_icon / shutdown）+ `is_started` 查询 |
| `src/install/` | node/bun/pnpm/**nub** 安装管线（fnm 兜底链在 `node.rs` / `download.rs` 内；镜像感知分支在 `flow/runtime_env.rs`）+ download + stream 输出泵 |
| `src/ui/geometry.rs` | 窗口几何持久化（WebView 与外置浏览器共用一份 `window-state.json`，物理像素 + webui 硬限 clamp） |
| `src/testutil.rs` | 测试专用助手：按 OS 选 shell 的 `shell()`（Windows `%COMSPEC%`/cmd，Unix `sh`），供子进程类测试共用 |
| `src/version.rs`, `src/probe.rs`, `src/mirror.rs` | 版本解析与预发布比较、工具探测（`Tool`）、镜像决策 |
| `crates/dshl/` | Track A exe 壳；`build.rs` 用 winresource 嵌 Windows 图标 |
| `crates/dshl-cli/` | CLI 参数层（两轨共用） |
| `crates/dshl-native/` | napi-rs cdylib（crate-type 仅 cdylib） |
| `plugins/dshl-native` | 加载 `.node` addon → Cordis 服务 `dshlNativeBackend` |
| `plugins/dshl-pipe` | 连接运行中 dshl 的控制管道（DSHL_CONTROL_URL）→ `dshlPipeBackend` |
| `plugins/dshl-control` | 顶层聚合：native/pipe 折叠成统一 `nativeCapabilities` 服务 + HTTP 路由 + plugin-guard |
| `plugins/dshl-control/src/backend-contract.js` | native / pipe 两档 backend 的能力契约（`TIERS` 清单；消费方折叠进 `nativeCapabilities` 前校验） |
| `assets/` | 前端三件套 `index.html` / `app.js` / `styles.css`（+ 字体、svg） |
| `packing/` | `windows/dshl.nsi`、`macos/build-dmg.sh`、`linux/build-deb.sh` |
| `.github/workflows/` | `ci.yml`、`release.yml`、`release-native.yml`、`release-plugins.yml` |

各插件包带 `cordis.patch.yml`，供 dsh bundle 时打补丁接线。

## Development Commands

```sh
cargo build --workspace                        # 产出 target/debug/dshl.exe（沙箱用；改完源码必须先重 build）
cargo clippy --workspace --all-targets -- -D warnings   # CI 同款门禁
cargo fmt --all -- --check
cargo test --workspace --locked
cargo test -p dshl-core --lib install::stream  # 跑单个测试模块
node plugins/dshl-native/scripts/build-native.mjs [--release]  # 本地 .node → plugins/dshl-native/native/（gitignored）
npm run check                                  # node --check 全部插件 JS
npm pack --workspaces --dry-run                # JS 包打包校验

# 上述门禁的脚本化封装（CI ci.yml 直接调用，本地与 CI 单源）：
scripts/gate.ps1 [-Rust|-Js]                   # Windows（gate.bat 转发壳）
scripts/gate.sh [--rust|--js]                  # Linux/macOS 等价实现
# 打包与发布（镜像 release*.yml 的单机子集）：
scripts/package.ps1 [--NoInstaller]              # Track A 本机全流程（Windows）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hibays/DSHL](https://github.com/hibays/DSHL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
