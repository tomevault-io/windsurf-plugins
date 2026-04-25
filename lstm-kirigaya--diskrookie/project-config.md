---
trigger: always_on
description: > 本文档面向 AI 编程助手，帮助快速理解和开发本项目。
---

# DiskRookie 项目指南

> 本文档面向 AI 编程助手，帮助快速理解和开发本项目。

## 项目概述

**DiskRookie（磁盘菜鸟）** 是一款 AI 驱动的智能磁盘分析与清理工具，采用 Tauri 2.x 构建。项目目标是让小白用户也能像电脑高手一样轻松清理磁盘。

### 核心特性

- 🚀 **Rust 高性能扫描**：基于 Rust 核心构建，最高速度可达 1 秒扫描 5000 个文件
- 🤖 **AI 智能分析**：自动分析文件用途，智能决定哪些需要删除、哪些需要迁移
- 🌐 **网盘无缝对接**：支持 Google Drive、OneDrive、阿里云盘、百度网盘、Dropbox 及 WebDAV NAS

### 技术栈

| 层级 | 技术 |
|------|------|
| 后端核心 | Rust (Edition 2021) |
| 桌面框架 | Tauri 2.x |
| 前端框架 | React 19 + TypeScript 5.x |
| 构建工具 | Vite 7.x |
| UI 组件 | PrimeReact + MUI + TailwindCSS |
| 图表库 | Recharts |
| 国际化 | i18next |

## 项目结构

```
DiskRookie/
├── Cargo.toml                    # Workspace 根配置
├── clippy.toml                   # Clippy  lint 配置
├── crates/                       # Rust 核心 crate
│   ├── common/                   # 通用错误类型和配置
│   ├── domain-model/             # 领域模型（数据结构）
│   ├── disk-scanner/             # 磁盘扫描引擎（含 MFT 扫描）
│   ├── ai-engine/                # AI 分析引擎
│   ├── executor/                 # 清理计划执行器
│   └── ntfs-reader/              # NTFS MFT 读取器（Windows 专用）
├── apps/desktop/                 # Tauri 桌面应用
│   ├── src-tauri/                # Rust 后端代码
│   │   ├── src/commands/         # Tauri 命令处理器
│   │   ├── Cargo.toml
│   │   └── tauri.conf.json       # Tauri 配置
│   ├── frontend/                 # React 前端
│   │   ├── src/                  # TypeScript/React 源码
│   │   ├── package.json
│   │   ├── vite.config.ts
│   │   └── tsconfig.json
│   └── package.json              # Tauri 应用 package.json
├── docs/                         # 文档
├── figures/                      # 截图和示意图
└── .github/workflows/            # CI/CD 工作流
```

### Crate 依赖关系

```
common (基础错误类型)
  ↑
domain-model (领域模型)
  ↑
disk-scanner → common, domain-model
ai-engine → common, domain-model
executor → common, domain-model
ntfs-reader (独立 crate，Windows only)

desktop (Tauri 应用) → 依赖所有 workspace crates
```

## 开发环境配置

### 前置要求

- **Rust**: 1.86+ (`rustc --version`)
- **Node.js**: 20+ (`node --version`)
- **包管理器**: npm 或 pnpm

### 首次设置

```bash
# 1. 克隆项目
git clone <repository-url>
cd DiskRookie

# 2. 安装前端依赖
cd apps/desktop/frontend
npm install
cd ..

# 3. 安装 Tauri CLI（全局）
npm install -g @tauri-apps/cli@latest

# 4. 配置 OAuth 环境变量（可选）
cp .env.example .env
# 编辑 .env 填入你的 OAuth 客户端密钥
```

### 常用开发命令

```bash
# 启动开发模式（自动启动 Rust 后端 + React 前端）
cd apps/desktop
npm run dev

# 仅启动前端开发服务器
cd apps/desktop/frontend
npm run dev

# 构建发布版本
cd apps/desktop
npm run build

# 构建 Windows x86_64 可执行文件
npx tauri build --target x86_64-pc-windows-msvc

# 构建 macOS ARM64 版本
npx tauri build --target aarch64-apple-darwin
```

## 代码风格规范

### Rust 代码规范

项目使用 Workspace 级别的 lint 配置，定义在根目录 `Cargo.toml`：

- `unsafe_code = "warn"` - 警告使用 unsafe 代码
- `unused_imports = "warn"` - 警告未使用的导入
- `dead_code = "warn"` - 警告死代码
- `correctness` lints - deny 级别（必须修复）
- `suspicious`, `complexity`, `perf`, `style` - warn 级别

**Clippy 配置**（`clippy.toml`）：
- 认知复杂度上限：30
- 函数行数警告阈值：200 行
- 类型复杂度上限：300

**提交前检查**：
```bash
# 格式化检查
cargo fmt --all -- --check

# Clippy 检查
cargo clippy --workspace --all-targets -- -D warnings

# 安全检查
cargo audit
```

### 前端代码规范

- **ESLint**: 使用 `@eslint/js` 和 `typescript-eslint`
- **TypeScript**: 严格模式，使用项目引用（project references）
- **TailwindCSS**: 用于样式

**前端检查命令**：
```bash
cd apps/desktop/frontend
npx tsc -b --noEmit      # 类型检查
npm run lint             # ESLint 检查
npx vite build           # 构建测试
```

## 模块说明

### 1. `crates/common` - 通用基础

- `error.rs` - 定义 `DiskAnalyzerError` 错误类型
- `config.rs` - 配置管理
- `telemetry.rs` - 遥测/日志

### 2. `crates/domain-model` - 领域模型

核心数据结构：
- `scan_result.rs` - 扫描结果
- `file_tree.rs` - 文件树结构
- `cleanup_plan.rs` - 清理计划
- `action.rs` - 操作动作定义
- `risk.rs` - 风险等级
- `top_file_entry.rs` - 大文件条目

### 3. `crates/disk-scanner` - 磁盘扫描

高性能文件扫描实现：
- `scanner.rs` - 主扫描逻辑
- `mft_scan.rs` - NTFS MFT 快速扫描（Windows）
- `node.rs` - 文件树节点
- `filters.rs` - 文件过滤逻辑

依赖：`rayon`（并行扫描）、`ntfs-reader`（Windows MFT）

### 4. `crates/ai-engine` - AI 分析

- `planner.rs` - 清理计划生成
- `prompt.rs` - AI 提示词模板
- `validator.rs` - 结果验证

### 5. `crates/executor` - 执行器

- `delete.rs` - 删除操作
- `move.rs` - 移动操作
- `dry_run.rs` - 试运行模式
- `permission.rs` - 权限处理

### 6. `apps/desktop/src-tauri/src/commands` - Tauri 命令

| 模块 | 功能 |
|------|------|
| `scan.rs` | 扫描路径命令 |
| `analyze.rs` | AI 分析命令 |
| `plan.rs` | 获取清理计划 |
| `execute.rs` | 执行清理计划 |
| `delete.rs` | 删除文件/目录 |
| `storage.rs` | 文件系统存储操作 |
| `oauth.rs` | OAuth 认证（Google/Baidu/Aliyun/Dropbox）|
| `cloud_upload.rs` | 云存储上传 |
| `permission.rs` | 管理员权限检查 |

## 数据存储

应用数据存储在用户主目录的 `.disk-rookie` 文件夹中：

```
~/.disk-rookie/
├── settings.json              # AI 设置
├── system-prompt.txt          # AI 系统提示词
├── prompt-instruction.txt     # 用户自定义提示词
├── theme.txt                  # 主题设置
└── snapshots/                 # 快照目录
    ├── index.json             # 快照索引
    └── <snapshot-id>.json     # 快照数据
```

**Windows**: `C:\Users\<用户名>\.disk-rookie\`
**macOS/Linux**: `~/.disk-rookie/`

## CI/CD 流程

### 持续集成 (`.github/workflows/ci.yml`)

每次 push/PR 到 main 分支时触发：

1. **Clippy** - Rust 代码 lint 检查
2. **Rustfmt** - 代码格式化检查
3. **Security Audit** - `cargo audit` 安全检查
4. **Frontend** - TypeScript 类型检查、ESLint、构建测试

**注意**: Linux CI 会将 `ntfs-reader` crate 替换为 stub，因为该 crate 仅支持 Windows。

### 发布流程 (`.github/workflows/release.yml`)

支持三种触发方式：

1. **Commit Message 触发**（推荐）: 包含 `[release]` 标记
   ```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LSTM-Kirigaya/DiskRookie](https://github.com/LSTM-Kirigaya/DiskRookie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
