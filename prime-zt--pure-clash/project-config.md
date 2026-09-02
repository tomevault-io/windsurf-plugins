---
trigger: always_on
description: - 使用 Rust 和 Zed GPUI 构建轻量、原生的 Mihomo 桌面客户端，产品名为 `Pure Clash`，Cargo 包名为 `pure-clash`。
---

# Pure Clash

## 项目目标与边界

- 使用 Rust 和 Zed GPUI 构建轻量、原生的 Mihomo 桌面客户端，产品名为 `Pure Clash`，Cargo 包名为 `pure-clash`。
- 面向 Windows 11 x64 + MSVC 与 Linux x64，负责配置管理、代理组选择、规则与连接查看、流量监控、系统代理/TUN 控制及 Mihomo 内核生命周期。
- 不在 Rust 中重写代理内核、协议栈或规则引擎；网络转发由独立的 Mihomo 进程完成。
- 当前仓库五个基础页面（概览/代理/连接/配置/设置）与关于页均使用真实数据；连接、流量与延迟测试经 controller 实时接入。

## 核心模块

- `src/main.rs`、`src/startup.rs`：应用入口、启动参数模式、快捷键和原生窗口创建；`--autostart` 用于登录后零窗口后台启动。
- `src/app/`：界面模块。`mod.rs` 持有 Pure Clash 状态与业务逻辑（内核生命周期、系统代理/TUN、订阅管线、后台任务与共享渲染助手），`shell.rs` 的 `AppShell` 持有长期业务实体、托盘和可重建的主窗口句柄；渲染按区域拆分子模块：`frame.rs`（标题栏、状态徽标、窗口按钮、Linux 客户端装饰）、`sidebar.rs`（侧栏与内核卡片）、`header.rs`（页面路由与页头开关芯片）、`overview.rs`/`proxies.rs`/`connections.rs`/`profiles.rs`/`settings.rs`/`about.rs`（五个基础页面加关于页，含版本、源码仓库、开源组件清单与 GitHub Releases 更新检查）。子模块经 `use super::*` 访问父模块的私有状态，跨页面复用的小组件（连接行、横幅等）以 `pub(super)` 暴露。
- `src/config.rs`：`AppConfig`、程序目录初始化、`config/app.json` 读取与即时原子持久化；通过随包版本 marker 迁移仍跟随旧随包内核的配置。
- `src/platform/mod.rs`：跨平台目录、内核进程守护接口、登录自启、托盘抽象和主窗口装饰策略；各平台行为差异统一从这里解析。`src/platform/{windows,linux}/autostart.rs` 分别维护当前用户 Run 注册值与 XDG Autostart desktop entry，`src/platform/file.rs` 提供 Windows `MoveFileExW` 与 unix `rename` 的同目录原子替换。
- `src/platform/windows/job.rs`：Windows Mihomo 专用 Job Object，启用 `JOB_OBJECT_LIMIT_KILL_ON_JOB_CLOSE`。
- `src/platform/linux/child_guard.rs`：Linux 普通内核进程守护，通过 `PR_SET_PDEATHSIG` 保证主进程异常退出时内核被回收，由 `KernelProcessGuard` 调用。
- `src/platform/linux/tun_service.rs`：Linux TUN 一次授权服务的安装/卸载、按 UID 隔离的长度前缀 JSON IPC、runtime bundle 物化与服务端 root 内核生命周期；服务先在同一状态目录 staging 完整 bundle 并执行 `mihomo -t`，通过后才停止旧内核和原子切换 runtime，新内核启动失败会恢复备份并尽力重启旧内核。服务版本与协议、随包内核版本共同决定是否刷新 root 副本。
- `src/platform/linux/elevation.rs`：Linux TUN 服务客户端，把 `launch_elevated`/`ElevatedProcess` 映射到服务 IPC，并在 GPUI 与单实例初始化前分流安装器与 systemd 服务模式。
- `src/platform/{windows,linux}/process_guard.rs`：各平台同名 `KernelProcessGuard`，统一内核子进程的守护（Job Object / pdeathsig）与终止策略（TerminateProcess / SIGTERM→SIGKILL），`mihomo` 模块保持平台无关。
- `src/platform/windows/single_instance.rs`：Windows 当前会话单实例锁；后续启动只通知首实例恢复并激活主窗口。
- `src/platform/tray.rs`：平台无关托盘抽象，定义 `TrayAction` 并按平台导出同名 `SystemTray`。
- `src/platform/windows/tray.rs`：Windows 系统托盘，复用 EXE 应用图标，转发单击与右键菜单事件并动态更新运行状态提示。
- `src/platform/linux/tray.rs`：Linux 托盘，基于 ksni（SNI/DBus 纯 Rust 实现）提供图标、菜单和状态文本，与 Windows 行为对齐。
- `src/platform/linux/window_ctrl.rs`：Linux 主窗口重建后的显示与激活，衔接 Wayland/X11 差异。
- `src/platform/windows/window_ctrl.rs`：Windows 主窗口重建后的显示与激活，支撑托盘唤起。
- `src/mihomo/process.rs`：使用 `-t` 校验默认配置，按 `-d` / `-f` 启停真实内核并回收子进程；平台进程管理全部委托 `platform::KernelProcessGuard`，模块内不含平台分支；普通内核的 stdout/stderr 经泵线程逐行写入 kernel.log（打不开日志也持续排空管道）。
- `config/mihomo/default.yaml`：嵌入程序的首次启动默认配置，仅包含一个内置 `DIRECT` 节点。
- `src/logging.rs`：零框架文件日志（`jiff` 仅提供本地时区时间戳）。运行日志 app.log 与内核日志 kernel.log 分文件、各自按大小轮转为 `.1`（1MB+3MB 单文件上限，磁盘合计约 8MB）；宏 `log_error!`/`log_warn!`/`log_info!`/`log_debug!` 以标签区分模块，所有消息写入前经 `redact` 脱敏（干净行零分配借用原文）；panic hook 记录 panic；初始化失败降级为空日志绝不阻断启动。写入路径按频率区分：app.log 低频用 `LineWriter` 逐行落盘保证崩溃诊断不丢尾，kernel.log 高频用 `BufWriter` 批量落盘（停止内核时等待泵线程排空并冲刷），轮转判断用累计字节计数而非每行查询文件元数据。
- `src/profile.rs`：URL 订阅和本地配置文件的读取、大小/编码限制、结构校验、落盘与运行时配置同步管线。
- `src/mihomo/config.rs`：客户端本地基线（端口/controller/secret）、订阅合并与结构预检。
- `src/mihomo/controller.rs`：external controller REST 客户端（版本/模式/代理组）与订阅下载。
- `src/mihomo/geodata.rs`、`geodata/`：随包 GeoSite/GeoIP/MMDB 数据的清单校验、首次离线安装、完整性状态和设置页官方更新事务。
- `src/ui/`：GPUI 缺失的通用控件（自绘单行文本输入，参照官方 input 示例实现）。
- `locales/`：`rust-i18n` 编译期加载的简体中文与英文界面资源。
- `src/theme.rs`：浅色/深色调色板、字重和透明色辅助 trait。
- `src/assets.rs`、`assets/icons/`：嵌入式 SVG 资源注册与本地图标；`app.svg` 是带背景的应用图标源文件。
- `build.rs`、`src/kernel.rs`、`kernel/{版本}/`：从随包 manifest 注入默认内核版本，并按启动配置解析运行时路径。
- `packaging/windows/`：Windows 专属的 NSIS 安装器定义和 PowerShell 7 打包入口。
- `packaging/linux/`：Linux 发行包资源（desktop 条目、图标、deb 维护脚本、rpm scriptlet、AppImage 组装脚本）；deb/rpm 元数据在 `Cargo.toml` 的 `package.metadata.deb` / `generate-rpm`，安装布局为 `/opt/pure-clash` + `/usr/bin` 软链，内核版本目录升级时同步两段 assets。完整卸载会在程序文件移除前调用内部 root 服务清理入口，升级事务不会删除软链或服务。
- `.github/workflows/release.yml`：推送 `v*` 标签触发的发布流水线，构建 Windows NSIS 与 Linux deb/rpm/AppImage 并发布 GitHub Release；标签版本与 Cargo 版本不一致时直接失败。构建 job 默认只有 `contents: read`，仅发布 job 可写；外部 AppImage 工具固定不可变版本并校验官方 SHA-256。
- `docs/pure-clash-architecture.md`：Mihomo 进程、REST/WebSocket 控制、安全、配置和产品化技术基线。

## 技术栈、目录与约定

- Rust 2024 edition；GPUI 使用 Zed `v1.17.2` 对应提交 `c8e44cfa7bda9b2e22c8d6934d78969352e7f61a`，平台后端使用同提交的 `gpui_platform`；`rust-i18n = 4.2.1`；Windows 托盘使用 `tray-icon = 0.24.2`；unix 目标使用 `libc` 发送 SIGTERM 与设置父进程死亡信号；非 Windows 目标使用 `directories = 6.0` 解析标准用户目录。
- 当前 Cargo 包版本为 `0.2.3`；正式发布标签必须使用匹配的 `v0.2.3`，否则发布流水线会拒绝构建。
- UI、业务说明和代码注释使用中文；协议字段、类型名和函数名保留英文。
- Cargo/可执行文件/安装包前缀统一为 `pure-clash`，界面和 Windows 发行名统一为 `Pure Clash`。
- 保持单包、小依赖；平台无关路径留在 `src/platform/mod.rs`，只有实际接入系统代理、凭据、进程监管等能力时才新增 `src/platform/windows/`、`linux/` 或 `macos/` 子模块。
- 所有 Mihomo API、CLI 和配置字段均以官方文档及当前锁定内核版本为准，不凭记忆补字段。

## 常用命令

- 运行：`cargo run`
- 检查：`cargo check`
- 测试：`cargo test`
- 格式化：`cargo fmt --check`
- 发布构建：`cargo build --release`
- NSIS 打包：`pwsh -NoLogo -NoProfile -File .\packaging\windows\build-installer.ps1`

## 架构决策与限制

- Mihomo 作为独立 sidecar 运行；Pure Clash 通过仅监听 loopback 的 REST/WebSocket external controller 通信，并为每次安装生成高强度随机 secret。
- 客户端只接受本机 controller，不开放局域网控制；日志与诊断信息必须脱敏，不记录订阅 URL、认证头或 controller secret。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prime-zt/pure-clash](https://github.com/prime-zt/pure-clash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
