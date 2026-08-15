---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Tydora（代码内称为 "zmd"）是一个基于 Tauri v2 + React 19 的桌面 Markdown 编辑器。支持 WYSIWYG 和源码两种编辑模式，以及基于"仓库（Vault）"概念的文件管理。

## 技术栈

- **前端**: React 19 + TypeScript + Vite 6
- **后端**: Rust (Tauri v2)，Windows 为主要目标平台
- **编辑器**: TipTap 3.x（WYSIWYG 模式）+ CodeMirror 6（源码模式）+ tiptap-markdown（Markdown 序列化）
- **Tauri 插件**: `plugin-fs`（文件系统）、`plugin-dialog`（系统对话框）

## 常用命令

```bash
# 安装依赖
npm install

# 开发模式（Vite dev server，端口 1420）
npm run dev

# 构建前端（TypeScript 编译 + Vite 打包）
npm run build

# 预览生产构建（vite preview）
npm run preview

# 启动 Tauri（桌面应用）— 通过 scripts/run-tauri.mjs 加载 .env 签名密钥
npm run tauri

# 同步版本号（从 VERSION 文件同步到 tauri.conf.json、Cargo.toml、package.json）
npm run sync-version

# 文档站点
npm run docs:build    # 构建 MkDocs 文档站点
npm run docs:serve    # 本地预览文档站点
npm run docs:clean    # 清理构建的文档
```

**注意**：项目目前没有配置 test/lint/format 脚本。`npm run build` 内置了 `tsc` 类型检查。`scripts/run-tauri.mjs` 用于加载 `.env` 文件中的签名密钥环境变量，然后转发参数给 `npx tauri`。

## 版本管理

项目使用统一的版本管理方案，所有版本号从根目录的 `VERSION` 文件同步：

```bash
# 版本号源文件
VERSION                    # 纯文本文件，只包含版本号（如 0.0.7）

# 同步脚本
npm run sync-version       # 将 VERSION 中的版本号同步到以下文件：
                          # - src-tauri/tauri.conf.json
                          # - src-tauri/Cargo.toml
                          # - package.json
```

**发布流程**：

1. 修改 `VERSION` 文件中的版本号
2. 运行 `npm run sync-version` 同步所有配置
3. 或直接构建（`npm run tauri`），构建前会自动同步版本

## 架构概览

### 前端 (src/)

**入口**: `main.tsx` → 用 `ThemeProvider` 包裹整个应用，挂载到 `#root`。

**主题系统**:

- `ThemeManager.ts` — 8 种内置主题定义（`white`、`mint`、`mint-dark`、`liquid-glass`、`claude-code`、`purple`、`hermes`、`next`）。默认主题为 `mint`。
- `CustomThemeManager.ts` — 自定义主题的导入/导出/管理，存储在 Tauri `appDataDir` 中。
- `codeThemes.ts` — 11 种代码语法高亮配色方案。
- `themes.tsx` 提供 `ThemeContext`，主题通过 `document.documentElement.dataset.theme` 设置，并在切换时同步更新 highlight.js 代码高亮样式。

**App 组件** (`App.tsx`) — 应用状态中心：

- 管理编辑器内容 (`content`)、当前文件路径 (`fileName`)、修改状态 (`modified`)、编辑模式 (`viewMode: "ir" | "sv"`)
- 管理仓库列表 (`vaults`)、当前激活仓库索引 (`activeVaultIndex`)、侧栏状态 (`sidebarOpen`/`sidebarWidth`)
- 所有状态持久化到 `localStorage`：`zmd-vaults`、`zmd-active-vault`、`zmd-sidebar-width`、`zmd-window-state`、`zmd-theme`
- 窗口位置/大小通过 Tauri Window API 恢复和防抖保存
- 自定义窗口控件（最小化/最大化/关闭），替代系统原生标题栏
- Ctrl+S 全局快捷键保存
- `EditorErrorBoundary` 类组件防止编辑器崩溃导致白屏

**TipTapEditor** (`src/Editor/TipTapEditor.tsx`) — TipTap 封装：

- 使用 `forwardRef` + `useImperativeHandle` 暴露 `getValue`/`setValue`/`resize`/`executeCommand` 等方法
- 通过内部标志 `isInternalRef` 区分程序化变更和用户输入，避免 `setValue` 后触发 `onChange` 造成循环
- 使用 `tiptap-markdown` 进行 Markdown ↔ HTML 双向转换
- 通过 `useEditor` hook 创建编辑器实例，配置 20+ 扩展
- 支持两种模式：IR（即时渲染/WYSIWYG）和 SV（源码模式，使用 CodeMirror）

**SourceEditor** (`src/Editor/SourceEditor.tsx`) — CodeMirror 6 源码编辑器：

- 使用 `@codemirror/lang-markdown` 提供 Markdown 语法支持
- 自定义行号显示、搜索高亮、主题适配

**Sidebar** (`Sidebar.tsx`) — 包含三个子模块：

1. **FileTree**: 递归树形组件，渲染仓库目录文件结构。支持展开/折叠目录、右键菜单（新建文件/文件夹、重命名、删除、复制路径）、内联重命名、拖拽移动文件到其他目录。
2. **Outline**: 解析 Markdown 标题（`# ~ ######`），渲染可点击的大纲列表，点击滚动到编辑器对应位置。
3. **VaultSwitcher**: 底部仓库选择器，含仓库列表下拉菜单和主题选择设置弹出面板。

**Vault 概念**：用户选择一个本地文件夹作为"仓库"，应用展示该文件夹内的文件树。支持多仓库切换，仓库列表持久化到 localStorage。

### 后端 (src-tauri/)

**入口**: `main.rs` → 调用 `tydora_lib::run()`

**源码结构**:

```
src-tauri/src/
├── main.rs
├── lib.rs
└── commands/
    ├── mod.rs
    └── watcher_commands.rs    # 文件监听命令（notify crate）
```

**Tauri 插件**（5 个）:

| 插件                          | 用途      |
| --------------------------- | ------- |
| `tauri-plugin-fs`           | 文件系统访问  |
| `tauri-plugin-dialog`       | 系统对话框   |
| `tauri-plugin-window-state` | 窗口状态持久化 |
| `tauri-plugin-updater`      | 应用自动更新  |
| `tauri-plugin-process`      | 进程管理    |

**自定义 Tauri 命令**（15 个）:

| 命令                        | 用途                                              |
| ------------------------- | ----------------------------------------------- |
| `get_default_content`     | 获取默认编辑器内容                                       |
| `get_app_version`         | 获取应用版本号                                         |
| `get_cwd`                 | 获取当前工作目录                                        |
| `open_settings_window`    | 打开设置窗口（800×600，无装饰）                             |
| `open_file_in_new_window` | 在新窗口打开文件（1200×800）                              |
| `open_file_location`      | 在系统文件管理器中定位文件                                   |
| `open_file`               | 用系统默认应用打开文件                                     |
| `open_directory`          | 在系统文件管理器中打开目录                                   |
| `open_mindmap_window`     | 打开思维导图窗口（900×600）                               |
| `open_graph_window`       | 打开知识图谱窗口（1000×700）                              |
| `watch_vault`             | 启动仓库文件系统监听                                      |
| `unwatch_vault`           | 停止仓库文件系统监听                                      |
| `run_markdown_publish`    | 调用 `@abstractwebunit/markdown-publish` CLI 发布网站 |
| `preview_site`            | 启动 HTTP 服务器预览已发布站点                              |
| `stop_preview`            | 停止预览 HTTP 服务器                                   |

**状态管理**（lib.rs）:

- `WatcherState` (`Mutex<Option<RecommendedWatcher>>`) — 仓库文件监听状态

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zuorn/Tydora](https://github.com/zuorn/Tydora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
