---
trigger: always_on
description: 本项目使用 `skills/rust-design-conventions` 技能作为 Rust 全栈设计与性能权威指南。
---

# Rust Design Conventions / Rust 设计规范

本项目使用 `skills/rust-design-conventions` 技能作为 Rust 全栈设计与性能权威指南。
当涉及编写 Rust 代码、项目结构、模块/Crate 划分、API 设计、Cargo.toml 配置、命名
规范、性能优化、内存与布局、并发（Send/Sync/原子）、异步（Future/Pin/Tokio）、
零成本抽象、unsafe/FFI、零拷贝、生命周期、宏系统、构建/Features/交叉编译、测试、
文档注释、Lint/clippy、SemVer、依赖管理与供应链等任务时，**优先读取该技能**。

技能位置：

- `skills/rust-design-conventions/SKILL.md` — 主文件：默认行为规则 + 场景路由索引。
- `skills/rust-design-conventions/references/*.md` — 按主题拆分的深度参考模块
  （api-design、async-programming、cargo-build-features、code-robustness、
  concurrency、dependency-management、documentation、file-layout、lifetimes、
  lint-and-clippy、macros、memory-and-layout、naming-conventions、
  performance-optimization、performance-pitfalls、testing-standards、unsafe-rust、
  zero-copy、zero-cost-abstractions）。

> 用法：不要一次性读取所有参考文件。根据当前任务场景，按主文件中的「场景 → 模块对照表」
> 按需读取最相关的 1–2 个模块。

## Default Project Settings

When creating Rust projects or Cargo.toml files, ALWAYS use:

```toml
[package]
edition = "2024"

[lints.rust]
unsafe_code = "warn"

[lints.clippy]
all = "warn"
pedantic = "warn"
```


# Rust coding guidelines

* Prioritize code correctness and clarity. Speed and efficiency are secondary priorities unless otherwise specified.
* Do not write organizational or comments that summarize the code. Comments should only be written in order to explain "why" the code is written in some way in the case there is a reason that is tricky / non-obvious.
* Prefer implementing functionality in existing files unless it is a new logical component. Avoid creating many small files.
* Avoid using functions that panic like `unwrap()`, instead use mechanisms like `?` to propagate errors.
* Be careful with operations like indexing which may panic if the indexes are out of bounds.
* Never silently discard errors with `let _ =` on fallible operations. Always handle errors appropriately:
  - Propagate errors with `?` when the calling function should handle them
  - Use `.log_err()` or similar when you need to ignore errors but want visibility
  - Use explicit error handling with `match` or `if let Err(...)` when you need custom logic
  - Example: avoid `let _ = client.request(...).await?;` - use `client.request(...).await?;` instead
* When implementing async operations that may fail, ensure errors propagate to the UI layer so users get meaningful feedback.
* Never create files with `mod.rs` paths - prefer `src/some_module.rs` instead of `src/some_module/mod.rs`.
* When creating new crates, prefer specifying the library root path in `Cargo.toml` using `[lib] path = "...rs"` instead of the default `lib.rs`, to maintain consistent and descriptive naming (e.g., `gpui.rs` or `main.rs`).
* Avoid creative additions unless explicitly requested
* Use full words for variable names (no abbreviations like "q" for "queue")
* Use variable shadowing to scope clones in async contexts for clarity, minimizing the lifetime of borrowed references.
  Example:
  ```rust
  executor.spawn({
      let task_ran = task_ran.clone();
      async move {
          *task_ran.borrow_mut() = true;
      }
  });
  ```

# BMCBL Project Structure / 项目结构

BMCBL 是一个基于 GPUI 的原生 Rust 桌面启动器（Windows 优先）。下面给出仓库的文件路径树，并说明每个目录与关键文件的功能。图标资源（`crates/lucide-gpui/icons/`）与 `vendor/` 第三方依赖在树中省略，避免噪声。

## Workspace Layout / 顶层布局

```
BMCBL/
├── Cargo.toml              # Workspace 根清单，声明成员 crate 与共享依赖
├── Cargo.lock              # 依赖锁定
├── build.rs                # 应用级构建脚本：嵌入 Windows 清单、图标、payload 元数据
├── src/                    # BMCBL 应用主 crate（二进制 + 库）
├── crates/                 # 本地工作空间成员 crate
│   ├── gpui-hooks/         # GPUI React 风格 hooks 适配层（use_state 等）
│   ├── gpui-hooks-macros/  # hooks 的过程宏
│   ├── lucide-gpui/        # Lucide 图标资源 crate（基于 GPUI）
│   ├── nova-gfx/           # 跨后端图形抽象（Vulkan/DX12/Metal）与示例
│   ├── bmcbl-plugin-api/   # 插件宿主/插件间公共 API 类型
│   ├── bmcbl-plugin-macros/# 插件开发派生宏
│   └── bmcbl-plugin-tools/ # 插件打包/校验工具
├── vendor/gpui/            # 内嵌的 GPUI 框架源码（独立子清单，勿直接耦合业务）
├── assets/                 # 嵌入资源（编译期通过 AssetSource 打包）
├── docs/                   # 架构与设计文档
├── examples/plugins/       # 插件示例（bedrock-notes、hello-wasm）
└── scripts/                # 构建/校验/性能脚本（PowerShell）
```

## Application Source / 应用主 crate (`src/`)

```
src/
├── main.rs                 # 二进制入口：解析参数并启动 app
├── lib.rs                  # 库根：重导出模块，供测试与集成
├── app.rs                  # 应用启动：globals、字体注册、窗口、启动策略
├── startup.rs              # 启动流程编排（初始化顺序、单实例检查等）
├── launch.rs               # Minecraft 进程拉起逻辑
├── result.rs               # 统一错误/结果类型别名
├── config/                 # 配置模型与持久化
│   ├── config.rs           # Config 结构与字段
│   ├── defaults.rs         # 默认值
│   ├── storage.rs          # 读写配置文件
│   └── test.rs             # 配置测试辅助
├── core/                   # 非 UI 业务逻辑（核心领域）
│   ├── mod.rs
│   ├── minecraft/          # MC 版本管理、mod 管理、地图、截图、服务器、
│   │                       #   level.dat、资源包、UWP/AppX/GDK 集成、
│   │                       #   key patcher、mouse lock、远程版本源等
│   ├── curseforge/         # CurseForge API 客户端与数据模型
│   ├── easytier/           # EasyTier 联网（虚拟局域网）集成
│   ├── inject/             # 注入/补丁相关底层工具
│   ├── online/             # 在线房间/对等连接业务
│   ├── version/            # 版本号解析与比较
│   ├── sponsors.rs         # 赞助者数据
│   └── ui_prefs.rs         # UI 偏好（桥接 config 与 UI）
├── downloads/              # 下载引擎

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Chlna6666/Better-Minecraft-Bedrock-Launcher](https://github.com/Chlna6666/Better-Minecraft-Bedrock-Launcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
