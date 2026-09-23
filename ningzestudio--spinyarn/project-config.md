---
trigger: always_on
description: Rust 编写的 Minecraft 日志反混淆引擎，以**两个独立构建产物**交付：
---

# SpinYarn

Rust 编写的 Minecraft 日志反混淆引擎，以**两个独立构建产物**交付：

1. **`spinyarn`**——Axum Web API 服务（`src/`，Axum + Tokio），HTTP 端点反混淆
2. **`libspinyarn_capi`**——C ABI 共享库（`crates/capi/`，cdylib），供 PHP 等宿主语言嵌入（另有 `crates/php/` PHP 8 扩展）

两者共享同一核心：**`spinyarn-core`**（`crates/core/`，纯 Rust 同步库，无 axum/tokio 依赖）。利用 Fabric Yarn 映射表将混淆堆栈追踪（`class_XXX`/`method_XXX`/`field_XXX`）转换为可读名称。

## 架构要点

### Workspace 结构（Cargo workspace，resolver=2）
```
spinyarn (根 package：Axum Web API binary)
├── src/                 # main.rs + api/（Axum handler）+ error.rs + lib.rs（re-export core）
├── crates/core/         # spinyarn-core：同步核心库（config/cache/mapping/deobfuscator + Spinyarn 门面）
├── crates/capi/         # spinyarn-capi：C ABI（cdylib + staticlib），include/spinyarn.h
└── crates/php/          # PHP 8 扩展（C 源码，config.m4 + spinyarn.c，非 cargo 成员）
```
- **`spinyarn-core`**（无 tokio/axum/utoipa 依赖，纯同步）：`config`/`cache`/`mapping`/`deobfuscator` 四大模块 + **`Spinyarn` 门面类型**（`crates/core/src/lib.rs`）——持有 `mappings_dir`/可选 LRU cache。**完整流水线** `deobfuscate()`（透传→缓存→加载→反混淆，C ABI 用）+ **分步方法**（Web API 用，精确控制信号量门控）：`ensure_available()`/`get_cached()`/`load()`/`insert_cached()`/`deobfuscate_loaded()`，另有 `has_mapping()`/`cache_stats()`/`unload()`/`mappings_dir()`。构造器：`new(&Config)`、`from_settings(dir)`、`from_full_settings(dir, cache_max, cache_high, cache_low)`（0 = 禁用/自动，MySQLi 风格全参数）。这是 C ABI 与任何嵌入宿主的统一入口
- **`spinyarn-capi`**（`crates/capi/src/lib.rs`）：`#[no_mangle] extern "C"` 函数，全部 `catch_unwind` 防 panic 跨 FFI 边界（UB）。句柄/结果用 `Box::into_raw` 指针 + 显式 free。**无配置文件**——`spinyarn_init(mappings_dir)` 简版（默认缓存）与 `spinyarn_init_full(mappings_dir, cache_max, cache_high, cache_low)` 全参数版（MySQLi 风格，0 = 禁用/自动）。`spinyarn_deobfuscate` 对 `content_len` 设 64MB 上限防恶意越界读/OOM
- **`spinyarn`（binary）**：`src/lib.rs` 用 `pub use spinyarn_core::{...}` re-export 核心，`AppState` 持 `Arc<Spinyarn>` + `Semaphore`；`src/api/deobfuscate.rs` 复用门面分步方法（缓存命中与 ensure 步骤不占信号量，仅 load 步骤占），与 core 门面逻辑不再重复

- **无缓存模型（基础）+ LRU 缓存（默认开启）**：无缓存时每次调用独立加载映射 → 反混淆 → 释放（单版本解析表实测 ~10MB）。`[cache]` 段启用**有界 LRU**（`crates/core/src/cache.rs`，默认 `max_entries=44`/高水位 40/低水位 30，**共享缓存池**——Yarn/Vanilla 映射同池缓存，key 为 `version+mapping_type`）：解析后表以 `Arc<LoadedMappings>` 缓存，高水位触发批量淘汰至低水位，缓存大小在低~高水位间波动（避免长期满载）；命中请求跳过加载（热请求 ~6ms）且不占并发信号量。实测：缓存水位 30~40 条目 ≈ 300~400MB（服务器内存可承受）；命中/驱逐/条目数经 `/health` 暴露
- **并发限流**（仅 Web API）：`src/api/mod.rs::AppState` 的 `Semaphore`，默认 32（`config.toml` 的 `server.max_concurrency` 可调，未配置时 `SPINYARN_MAX_CONCURRENCY` 环境变量兜底）。无缓存模型下每并发请求持有一整套版本表（~10MB），限流把峰值内存钉在 N×10MB，突发流量 OOM 换成短暂排队（稳态并发约 16，平时不触发）。C ABI 路径无信号量（宿主自行并发控制）
- **映射表外置（无下载）**：映射文件（Yarn `*.tiny.gz` 与 Vanilla `vanilla/*.txt`）由宿主预先放置到 `mappings_dir`，本引擎**不做任何网络下载**；请求的版本本地缺失时反混淆直接透传原样（不报错）
- **双映射类型**：请求 `mapping_type` 参数（`yarn` 默认 / `vanilla`）；`crates/core/src/mapping/dispatcher.rs` 调度机负责按类型加载与分派引擎（`LineEngine` / `VanillaEngine`）。Vanilla 用 TSRG 解析器（`crates/core/src/mapping/vanilla.rs`），短混淆名只能走结构化堆栈解析（类确认 + TSRG 行号区间定位重载），不适用 residual 正则
- CPU 密集操作（gzip 解压 + 解析 + 反混淆）：Web API 路径放入 `tokio::task::spawn_blocking` 不阻塞 runtime；core/C ABI 路径本就是同步阻塞调用
- **访问日志中间件**：`tower_http::TraceLayer` 记录每个请求的 method/uri/status/耗时。deobfuscate 走 INFO，health 探针走 DEBUG（避免噪音）
- 端点：`POST /api/v1/deobfuscate`（64MB 上限）、`POST /api/v1/deobfuscate/plain`（成功返回 `text/plain`，失败 JSON）、`GET /api/v1/health`；映射管理：`POST /api/v1/mappings/load`（Maven 拉取/刷新）、`POST /api/v1/mappings/load/local`（本地文件加载，路径限 `mappings/` 内防穿越）、`GET /api/v1/mappings`（列出版本）、`GET /api/v1/mappings/{type}/{version}`（统计）、`DELETE /api/v1/mappings/{version}`（卸载文件+缓存）；`GET /api/v1/openapi.json`（utoipa 生成的 OpenAPI 3.0 规范）
- 管理端点（v0.2 曾移除，v0.4 以映射管理形态回归）

## 关键约定

### 内置版本 + 透传
**无硬编码版本清单**：`crates/core/src/mapping/local.rs::is_version_supported` 运行时判断——外部映射目录存在 `<version>.tiny.gz` 即可反混淆，否则 → **原样透传**（`success: true`，计数为 0），不报错。往映射目录新增版本文件（含 pre-release）无需改代码即自动生效。本引擎**不做任何下载**，映射由宿主预先放置。

### 映射外置（与二进制同级部署）
- **映射不嵌入二进制**：`build.rs`/`embedded.rs` 已移除，binary ~6MB；C ABI 库 ~6MB
- 默认映射目录 = **二进制同级 `./mappings/`**（`std::env::current_exe()` 定位，不依赖工作目录）；`config.toml` 的 `maven.mappings_dir` 或 `SPINYARN_MAPPINGS_DIR` 可覆盖
- **映射由宿主预先下载放置**：本引擎不下载；宿主用 `bash scripts/download_mappings.sh [版本...]` + `python3 scripts/download_vanilla_mappings.py` 预生成映射文件放入映射目录。Release 制品不打包 `mappings/`
- 加载：外部映射目录存在即用，否则透传
- **C ABI 注意**：`exe_dir()` 在 cdylib 下是宿主进程（如 php-fpm），默认 `mappings/` 会落到宿主可执行文件旁，通常不是期望位置——PHP 侧应通过 `spinyarn_init($mappings_dir)` 显式传映射目录
- **C ABI 部署约定**：C ABI **不读配置文件**，由 PHP 调用方直接传 `spinyarn_init($mappings_dir)`——映射目录通常取 PHP 项目根下的相对路径（如 `__DIR__ . '/mappings'`）

### 配置加载
`Config::load()` 按顺序查找：二进制同级 `config.toml` → 当前目录 `config.toml` → `SpinYarn.toml` → `/etc/spinyarn/config.toml`，都没找到则**在二进制同级自动生成默认 `config.toml`**（`toml::to_string_pretty` 序列化默认值，写失败仅 warn 不 panic）。配置项：`server.host`/`server.port`/`server.max_body_size`（默认 64MB，无环境变量兜底）/`server.max_concurrency`（默认 32，`SPINYARN_MAX_CONCURRENCY` 兜底）/`maven.mappings_dir`（默认二进制同级 `./mappings`，`SPINYARN_MAPPINGS_DIR` 兜底）/`cache.enabled`（默认 true）/`cache.max_entries`（44）/`cache.high_watermark`（40）/`cache.low_watermark`（30）。启动时若端口已被占用，`main.rs` 自动 `port + 1` 递增重试直至找到空闲端口（`u16` 溢出保护）。

### 版本格式兼容
`crates/core/src/mapping/tiny_v2.rs` 自动检测 v1（平铺 `CLASS`/`FIELD`/`METHOD`）和 v2（缩进 `c`/`\tf`/`\tm`）格式，列位置按头部命名空间名定位（兼容 1.14 特殊列序）。

### 反混淆引擎
- 堆栈行：手写 memchr 解析（支持 `knot/`、`java.base/` 前缀、嵌套类回退、源文件名替换）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NingZeStudio/SpinYarn](https://github.com/NingZeStudio/SpinYarn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
