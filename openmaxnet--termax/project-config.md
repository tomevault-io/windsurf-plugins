---
trigger: always_on
description: > 适用范围：Termax SSH 终端客户端（Tauri v2 + Rust + React 19 + TypeScript + Zustand + Tailwind CSS v4）
---

# Termax 项目编码规范

> 适用范围：Termax SSH 终端客户端（Tauri v2 + Rust + React 19 + TypeScript + Zustand + Tailwind CSS v4）
> 核心原则：不影响现有功能、遵循最佳实践、良好的扩展性、中文注释、单一职责

---

## 目录

1. [通用规范](#1-通用规范)
   - 1.1 [统一命名体系](#11-统一命名体系)
   - 1.2 [注释规范](#12-注释规范)
   - 1.3 [重复代码抽取原则](#13-重复代码抽取原则)
   - 1.4 [单一职责与扩展性](#14-单一职责与扩展性)
2. [Rust 后端规范](#2-rust-后端规范)
3. [TypeScript / React 前端规范](#3-typescript--react-前端规范)
4. [可复用组件库 (ui/)](#4-可复用组件库-ui)
5. [可复用 Hooks 库 (hooks/)](#5-可复用-hooks-库-hooks)
6. [Tauri IPC 通信规范](#6-tauri-ipc-通信规范)
7. [状态管理规范](#7-状态管理规范)
8. [国际化 (i18n) 规范](#8-国际化-i18n-规范)
9. [样式规范](#9-样式规范)
10. [测试规范](#10-测试规范)
11. [Git 提交规范](#11-git-提交规范)
12. [新功能开发 Checklist](#12-新功能开发-checklist)

---

## 1. 通用规范

### 1.1 统一命名体系

所有命名遵循**前缀统一**原则，确保项目内命名一致、避免外部库冲突。

#### 1.1.1 CSS 变量前缀：`--tx-`

所有 CSS 设计令牌（Design Token）使用 `--tx-` 作为命名空间前缀，格式为 `--tx-{类别}-{层级}`：

```css
/* ═══ 背景层级 ═══ */
--tx-bg-base          /* 最底层页面背景 */
--tx-bg-surface       /* 面板/侧边栏背景 */
--tx-bg-elevated      /* 弹出层背景（弹窗、下拉菜单、工具提示） */
--tx-bg-hover         /* 悬停态背景 */
--tx-bg-active        /* 激活态背景 */
--tx-bg-overlay       /* 遮罩层背景（rgba） */

/* ═══ 文本层级 ═══ */
--tx-text-primary     /* 主要文本（最高对比度） */
--tx-text-secondary   /* 次要文本（中等对比度） */
--tx-text-tertiary    /* 辅助文本（低对比度） */
--tx-text-inverse     /* 反转文本（深色背景上的浅色文字） */
--tx-text-link        /* 链接文本 */

/* ═══ 强调色 ═══ */
--tx-accent-default   /* 默认强调色 */
--tx-accent-hover     /* 悬停强调色 */
--tx-accent-muted     /* 淡化强调色（选中背景等） */

/* ═══ 边框 ═══ */
--tx-border-light     /* 浅边框（分隔线、面板边框） */
--tx-border-default   /* 默认边框（输入框、按钮边框） */
--tx-border-focus     /* 焦点边框（focus ring） */

/* ═══ 语义色 ═══ */
--tx-green            /* 成功/已连接 */
--tx-green-bg         /* 成功背景 */
--tx-red              /* 错误/断开 */
--tx-red-bg           /* 错误背景 */
--tx-yellow           /* 警告/连接中 */

/* ═══ 阴影 ═══ */
--tx-shadow-sm        /* 小阴影（按钮、卡片） */
--tx-shadow-md        /* 中阴影（弹窗、下拉菜单） */
--tx-shadow-lg        /* 大阴影（模态框） */

/* ═══ 排版 ═══ */
--tx-font-sans        /* UI 无衬线字体栈 */
--tx-font-mono        /* 终端等宽字体栈 */
--tx-font-size-xs     /* 11px */
--tx-font-size-sm     /* 12px */
--tx-font-size-base   /* 13px */
--tx-font-size-lg     /* 14px */

/* ═══ 间距 ═══ */
--tx-space-1          /* 4px */
--tx-space-2          /* 8px */
--tx-space-3          /* 12px */
--tx-space-4          /* 16px */

/* ═══ 圆角 ═══ */
--tx-radius-sm        /* 4px */
--tx-radius-md        /* 6px */
--tx-radius-lg        /* 8px */
```

**规则：**
- CSS 变量**必须**使用 `--tx-` 前缀，禁止裸名（如 `--bg-base`）
- 新增变量在 `src/index.css` 的 `:root` 和 `.dark` 中同步定义
- 引用时使用 `var(--tx-xxx)` 语法，不允许硬编码颜色值
- 动画命名使用 `@keyframes tx-xxx` 格式（如 `tx-fade-in`、`tx-scale-in`）

#### 1.1.2 前端组件前缀：`T`

所有可复用的通用 UI 组件使用 `T` 前缀：

| 组件            | 用途                     |
|----------------|-------------------------|
| `TButton`      | 通用按钮（含 variant）     |
| `TIconButton`  | 图标按钮（含 tooltip 延迟）  |
| `TConfirm`     | 确认对话框                 |
| `TContextMenu` | 右键上下文菜单              |
| `TDropdown`    | 下拉菜单弹出层              |
| `TEmpty`       | 空状态占位                 |
| `TSelect`      | 自定义下拉选择器            |
| `TSplitPane`   | 通用分屏容器                |
| `TTip`         | 工具提示气泡                |
| `TSplash`      | 启动画面                   |

**规则：**
- 通用 UI 组件**必须**放在 `src/ui/` 目录
- 通用 UI 组件**必须**使用 `T` 前缀命名文件和导出名
- 应用外壳组件（`AppLayout`、`TitleBar` 等）放在 `src/app/` 目录，**不使用** `T` 前缀
- 功能模块组件（`TerminalTab`、`FileBrowser` 等）放在 `src/features/{domain}/` 目录，**不使用** `T` 前缀
- Hook 文件使用 `use` 前缀，Store 文件使用 `Store` 后缀，**不使用** `T` 前缀

#### 1.1.3 IPC 方法前缀：领域_动作

| 领域       | Rust 命令前缀 | 前端 ipc 方法前缀 | 示例                                  |
|-----------|-------------|-----------------|--------------------------------------|
| SSH 连接   | `ssh_`      | `ssh` + 动词     | `connect_ssh` → `ipc.sshConnect()`   |
| SFTP      | `sftp_`     | `sftp` + 动词    | `sftp_list_files` → `ipc.sftpListFiles()` |
| 本地终端   | `local_`    | `local` + 动词   | `connect_local` → `ipc.localConnect()` |
| 连接配置   | `config_`   | `config` + 动词  | `save_config` → `ipc.configSave()`   |
| 系统监控   | `monitor_`  | `monitor` + 动词 | `monitor_fetch` → `ipc.monitorFetch()` |
| SFTP 编辑  | `sftp_edit_`| `sftpEdit` + 动词| `sftp_start_edit` → `ipc.sftpEditStart()` |

**规则：**
- Rust 命令名：`{领域}_{动作}`，全 snake_case（如 `sftp_upload_file`）
- 前端 `ipc` 对象按**功能域分组**，方法名使用 camelCase（如 `ipc.sftp.uploadFile()`）
- 事件名：`{领域}-{动作}`，全 kebab-case（如 `term-output`、`session-ready`）

#### 1.1.4 事件名规范

```
领域-动作
 └──┘ └──┘
  │     └─ 具体动作（output, ready, error, uploaded）
  └─────── 功能领域（term, session, sftp-edit）

完整事件列表：
  term-output         ← 终端数据输出
  session-ready       ← 会话认证完成、shell 就绪
  session-error       ← 会话错误（断开、认证失败等）
  sftp-edit-uploaded  ← SFTP 编辑自动上传完成
```

#### 1.1.5 通用命名速查表

| 语言   | 类型               | 约定              | 示例                              |
|--------|-------------------|-------------------|-----------------------------------|
| Rust   | 模块/文件          | snake_case        | `ssh_cmd.rs`, `connection_config` |
| Rust   | 类型/结构体/枚举    | PascalCase        | `AppError`, `SessionHandle`       |
| Rust   | 函数/方法          | snake_case        | `spawn_session`, `connect_ssh`    |
| Rust   | 常量              | UPPER_SNAKE       | `CONFIG_FILE`                     |
| Rust   | Cargo crate       | snake_case        | `termax-core`, `termax-ssh`       |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openmaxnet/Termax](https://github.com/openmaxnet/Termax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
