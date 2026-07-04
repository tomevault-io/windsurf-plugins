---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

**sdkmate**（`sdkm`）是一款面向全栈工程师的跨平台 SDK 版本管理器，用 Rust 编写。通过符号链接切换 + 操作系统环境变量操纵，管理 Java、Node.js、Python、Maven、Rust 等开发环境的版本。

## 构建与开发命令

```bash
cargo build                          # 调试构建
cargo build --release                # 发布构建（产出 sdkm 二进制）
cargo test                           # 运行所有测试（workspace 级别）
cargo test -p <crate>                # 运行指定 crate 的测试（cli / sdkcore / util）
cargo fmt                            # 格式化代码
cargo clippy --all-targets --all-features  # 代码检查
```

## 工作区架构

三成员 Cargo workspace + 根二进制包：

```
sdkmate (root) → 产出 sdkm 二进制，入口在 main.rs
  crates/cli    → CLI 层：clap 命令定义 + handler 实现
  crates/sdkcore → 核心业务逻辑：config、init、install、list、switch、version、env 操作、符号链接
  crates/util   → 共享工具：宏、SDK 类型、终端输出、配置辅助、路径
```

**sdkcore 内部模块**：`version` 是版本解析公共模块(install/switch/list 共用),含 `cache`(缓存+fetch)、`fuzzy`(模糊匹配+相近版本建议)、`discovery`(各 SDK 版本发现+resolve 编排);下载 URL 构建留在 `install/download_url`(install 专属)。

**依赖关系**：`cli → sdkcore, util`；`sdkcore → util`；`util` 无内部依赖。所有 crate 继承 workspace 元数据（版本、edition、license），均为 `publish = false`。

## 核心架构模式

### 版本切换机制
- 创建符号链接 `<symlink_dir>/<sdk_name>` → `<store>/<sdk>/<version>` 目录
- 将符号链接的 bin 目录加入操作系统 PATH
- 通过平台特定的环境变量操作设置额外变量（如 JAVA_HOME）
- 切换后更新 config.toml 中的 `current_version`

### 配置项 (`config.toml`)
```toml
[network]
proxy = ""                # HTTP proxy URL, e.g. "http://127.0.0.1:7890"
ssl_verify = true         # SSL certificate verification
connect_timeout = 30      # Connection timeout in seconds
cache_ttl_secs = 3600     # Version API cache TTL (seconds), 0 = always fetch
github_token = ""         # GitHub PAT for higher API rate limit

[[sdk]]
name = "java"
download_url = "..."
bin_dir = "bin"
```

### 平台抽象
- `EnvOperation` trait：`WindowsEnvOperation`（注册表 + WM_SETTINGCHANGE 广播）和 `UnixEnvOperation`（shell profile 修改）
- `cfg(windows)` / `cfg(unix)` 条件编译用于环境操作、符号链接、默认路径
- 类型别名 `OsEnvOperation` 在编译时选择平台实现

### Home 目录发现
- sdkm 的 "home" = 运行中可执行文件的父目录（`current_exe()`），使工具可移植
- Store 目录：`<exe_parent>/store/`，用户在此放置 SDK 安装（如 `store/java/21/`）
- 配置文件：`<exe_parent>/config.toml`，TOML 格式，serde 序列化/反序列化 + 模板渲染

### 模板渲染系统
- `TemplateRenderer` 解析 URL 和环境变量配置中的占位符：`{sdk_dir}`、`{sdkm_home}`、`{sdks_install_dir}`、`{os}`、`{arch}`、`{ext}`
- OS/arch 检测使用 `OnceLock` 静态变量；映射 macOS→darwin、x86_64→x64、aarch64→arm64

### 错误处理与输出
- 全项目使用 `anyhow::Result`（启用 backtrace 特性）；`anyhow::bail!` 和 `.context()` 处理错误
- 无自定义错误类型，不使用 thiserror（`BugReport` 标记类型除外——用于 CLI 层检测不可由用户解决的错误）
- 终端输出通过自定义宏：`info!`、`success!`、`warning!`、`error!`、`detail!`（crossterm 彩色，不是 `log` crate）
- CLI 错误：用 `error!` 宏打印；debug 构建额外显示 backtrace；检测 `BugReport` 标记时提示 GitHub issue URL
- CLI 错误退出码：失败时 `process::exit(1)`

## CLI 命令结构

| 命令 | 别名 | Handler | 状态 |
|------|------|---------|------|
| `sdkm init` | — | cli/InitHandler | 已实现（目录部署检测 + 项目目录识别 + 平台建议路径） |
| `sdkm install <SDK> <VERSION>` | `i` | cli/InstallHandler | 已实现（模块拆分为 download_url/downloader/extractor/progress，版本解析提取到 `version/` 公共模块，12阶段异步流程） |
| `sdkm list [SDK] [--remote] [--limit N]` | `ls`, `l` | cli/ListHandler | 已实现（交互式 TUI 选择器 + 远程版本 + 安装/切换动作触发） |
| `sdkm switch <SDK> <VERSION>` | `s` | cli/SwitchHandler | 已实现（PATH 冲突检测 + extra_paths 支持 + **备份回滚机制** + **版本模糊匹配（与 install 共用核心）**） |
| `sdkm current [SDK]` | `c` | cli/CurrentHandler | 已实现 |
| `sdkm config` | — | cli/ConfigHandler | 已实现（set/get/list/delete/edit/add-sdk/remove-sdk 子命令，按类型校验 + 原子写入 + 回滚） |

每个命令在 `crates/cli/src/impls/` 中有 `CommandHandler` trait 实现，委托给 `crates/sdkcore/src/manager/` 中的 `SdkManager` 方法。

### install 子模块架构（重构后）

原 `install.rs` 单文件已拆为 `install/` 模块目录；版本解析逻辑已进一步提取到公共模块 `version/`(见下)：

- `mod.rs` — 安装流程入口，12 阶段同步/异步编排（resolve → check local → build URL → download → extract → verify → normalize → verify install → cleanup → auto-switch）
- `download_url.rs` — 下载 URL 构建：按 SDK 分发的自由函数 `build_download_url`（各 SDK os/arch 风格集中在此；install 专属，不属于版本解析）
- `downloader.rs` — 下载：reqwest 客户端构建 + 主/备源切换 + 重试机制
- `extractor.rs` — 解压：tar.gz/zip 解压 + 目录标准化 + 安装验证
- `progress.rs` — 进度显示：各阶段 indicaotr 风格的进度条

### version 公共模块（2026-06-30 提取）

版本解析逻辑从 `install/resolver.rs` 提取为公共模块 `crates/sdkcore/src/version/`（install/switch/list 三方共用），按职责拆为多子文件：

- `mod.rs` — 模块声明 + 公共 re-export + `truncate` 辅助
- `cache.rs` — 版本数据缓存 + 网络获取：`VersionSource`、`fetch_version_data`（主备切换 + 重试 + 缓存兜底）
- `fuzzy.rs` — 纯版本字符串模糊匹配：`FuzzyMatch`、`fuzzy_match_version_core`、`suggest_similar_version`（最长公共前缀 + 数值距离）
- `discovery.rs` — 各 SDK 版本发现 + 解析编排：`VersionEntry`/`ResolvedVersion`/`VersionDiscovery` trait（仅 `parse_version_data`）/ `get_version_discovery` / `fuzzy_match_version`（薄封装）/ `resolve_sdk_version` / `resolve_java_version` + 各 SDK 发现结构体 + serde 解析

**拆分要点**：原 `SdkInstallStrategy` trait 拆为 `VersionDiscovery`（只含 parse，公共）+ install 侧自由函数 `build_download_url`（按 SDK 分发，各 SDK os/arch 风格集中）；`get_install_strategy` → `get_version_discovery`；`ConfigBasedStrategy`（带 os_style/arch_style 字段）→ `ConfigBasedDiscovery`（单元结构体，custom SDK 下载 URL 风格改由 `build_download_url` 的 Custom 分支用 `OsStyle::Default`/`ArchStyle::Default` 表达，与原 `ConfigBasedStrategy::default()` 行为一致）。

## 当前开发进度（2026-07-02）

### 本次改动（2026-07-02）
1. **发布二进制体积优化**（6.34 MB → 2.74 MB，降 56%）——三管齐下，零功能影响：

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [borenchan/sdkmate](https://github.com/borenchan/sdkmate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
