---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 开发工作流

**IMPORTANT**：执行任何开发任务（编写代码、修改配置、添加依赖）前，必须先调用 `/dev-workflow` skill。它会加载项目知识库（`dev-notes/knowledge/`）中的最佳实践和踩坑记录，并在开发完成后引导更新知识库。

知识库主题：

- `dev-notes/knowledge/theme-and-styling.md` — shadcn/ui、主题变量、窗口装饰
- `dev-notes/knowledge/editor.md` — CM6、Y.Doc、y-codemirror.next、@swarmnote/editor submodule
- `dev-notes/knowledge/rust-backend.md` — Tauri command、SeaORM、YDocManager、P2P
- `dev-notes/knowledge/toolchain.md` — Biome、Lefthook、Lingui、Cargo workspace、Vite

## Project Overview

SwarmNote is a decentralized, local-first, peer-to-peer note-taking app built with Tauri v2 + React 19 + Rust. Notes sync between devices via P2P networking (libp2p) without a central server. Targets desktop (Windows/macOS/Linux) and Android.

## Development Commands

```bash
# First-time setup: init git submodule (libs/core) + install deps
git submodule update --init --recursive
pnpm install

# Launch full Tauri desktop app (starts frontend + Rust backend)
pnpm tauri dev

# Frontend dev server only (Vite on port 1420)
pnpm dev

# Build frontend (TypeScript compile + Vite build)
pnpm build

# Build Tauri app for distribution
pnpm tauri build

# Frontend lint (Biome check)
pnpm lint

# Frontend lint CI mode (no auto-fix, exits non-zero on errors)
pnpm lint:ci

# Frontend format (Biome auto-fix)
pnpm format

# Rust format + lint
cd src-tauri && cargo fmt
cd src-tauri && cargo clippy -- -D warnings

# Rust backend tests
cd src-tauri && cargo test

# Run a single Rust test
cd src-tauri && cargo test <test_name>

# yrs-blocknote crate tests (independent of Tauri)
cd crates/yrs-blocknote && cargo test

# i18n: extract messages from source
pnpm lingui extract

# Generate CHANGELOG from conventional commits
pnpm changelog

# Show unreleased changes
pnpm changelog:latest
```

No test framework is configured for the frontend yet.

**Linux 构建系统依赖**（Ubuntu/Debian）：`libwebkit2gtk-4.1-dev libappindicator3-dev librsvg2-dev patchelf libdbus-1-dev`

**工具版本**：Node 22+、pnpm 10+、Rust stable

## Code Quality Toolchain

- **Biome** (`biome.json`): 前端 lint + format（替代 ESLint + Prettier），`recommended` 规则集，自动 organize imports，2空格缩进，行宽 100。排除 `routeTree.gen.ts`、`src/locales/**`、`src/components/ui/**`（部分 a11y 规则关闭）
- **rustfmt** (`src-tauri/rustfmt.toml`): Rust 代码格式化，4空格缩进（`.editorconfig` 控制）
- **Clippy**: Rust 静态分析，CI 中以 `-D warnings` 运行
- **Lefthook** (`lefthook.yml`): Git hooks 管理
  - `pre-commit`: Biome check（前端）+ cargo fmt --check（Rust），并行执行
  - `commit-msg`: commitlint 校验 Conventional Commits 格式
- **commitlint** (`commitlint.config.js`): 提交信息必须遵循 [Conventional Commits](https://www.conventionalcommits.org/)（`feat:`, `fix:`, `docs:`, `chore:` 等）
- **git-cliff** (`cliff.toml`): 基于 Conventional Commits 自动生成 CHANGELOG
- **GitHub Actions** (`.github/workflows/ci.yml`): PR/push 到 main/develop 时自动跑 lint + build + test

## Architecture

### High-Level Structure

```text
swarmnote/
├── src/                  # React + TypeScript frontend
├── src-tauri/            # Rust backend (Tauri v2, Cargo workspace)
│   ├── entity/           # SeaORM entity 定义（独立 crate）
│   ├── migration/        # SeaORM 数据库迁移（独立 crate）
│   └── src/              # Tauri commands + 业务逻辑
├── crates/
│   └── yrs-blocknote/    # 独立通用 crate：BlockNote Y.Doc ↔ Markdown 双向转换
├── libs/core/            # swarm-p2p-core (git submodule, libp2p 封装)
├── docs/                 # Astro + Starlight 文档站
├── openspec/             # Spec-driven change management (OpenSpec)
├── dev-notes/            # Planning docs, PRD, tech selection
└── milestones/           # Version planning: requirements + design per version
```

### yrs-blocknote crate

独立通用库（`crates/yrs-blocknote/`），实现 BlockNote JSON、Markdown、Y.Doc 三种格式的双向转换。不依赖 Tauri/SwarmNote，可发布到 crates.io。

依赖：`yrs 0.25`（yjs Rust 实现）+ `comrak 0.51`（GFM Markdown 解析/渲染）。Block 数据模型为中心枢纽，serde 序列化与 BlockNote JSON 格式一致。

公开 API：`markdown_to_blocks` / `blocks_to_markdown` / `doc_to_blocks` / `blocks_to_doc` / `markdown_to_doc` / `doc_to_markdown` / `replace_doc_content`（在现有 Doc 上清空并重写内容，CRDT 历史连续）。

**重要约定**：所有 `Doc` 必须以 `OffsetKind::Utf16` 创建（与前端 JS yjs 一致）。yrs 默认的 `OffsetKind::Bytes` 会导致 CJK 字符 `block_offset` 溢出 panic。

SwarmNote 通过 path 依赖引用：`yrs-blocknote = { path = "../crates/yrs-blocknote", features = ["uuid"] }`。

### Backend Modules (src-tauri/src/)

| 模块 | 职责 |
|------|------|
| `identity/` | 设备身份（PeerId）、OS keychain 持久化、设备名管理（device_name 通过 agent_version 传播到 P2P 网络） |
| `workspace/` | 多窗口工作区管理、per-window DB 绑定（RwLock<HashMap>）、最近工作区 |
| `document/` | 文档 & 文件夹 CRUD（通过 SeaORM 操作 workspace DB） |
| `fs/` | 文件系统 I/O、workspace 目录扫描、文件监听（notify debounce）、媒体保存 |
| `network/` | P2P 节点生命周期（NetManager）、事件循环分发、DHT 在线宣告 |
| `pairing/` | 设备配对码生成/验证、配对请求/响应流程（PairingManager） |
| `protocol/` | 自定义 P2P 协议定义（AppRequest/AppResponse）、OsInfo（设备信息通过 agent_version 编解码，含 name/hostname） |
| `device/` | DeviceManager——追踪在线设备信息 |
| `yjs/` | YDocManager——per-doc Y.Doc 生命周期、yrs ↔ DB 持久化、debounce 自动保存、外部 .md 变更检测与重载 |
| `config/` | 全局配置持久化（最近工作区列表等） |
| `tray.rs` | 系统托盘（仅桌面端，最后一个窗口隐藏到托盘而非退出） |
| `error.rs` | 统一错误类型 AppError，序列化为 `{ kind, message }` 供前端消费 |

### Database Architecture

**双数据库设计**：

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swarm-apps/SwarmNote](https://github.com/swarm-apps/SwarmNote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
