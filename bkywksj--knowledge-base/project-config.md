---
trigger: always_on
description: - **所有源码与配置文件统一强制使用 UTF-8（无 BOM）**
---

# CLAUDE.md - Tauri Desktop App Framework

## 语言设置
**必须使用中文**与用户对话。

## 🔴 文件编码与注释规范（必须遵守）

### 1. 统一编码
- **所有源码与配置文件统一强制使用 UTF-8（无 BOM）**
- 包括但不限于：`.rs`、`.tsx`、`.ts`、`.js`、`.json`、`.toml`、`.html`、`.css`、`.md`
- **绝对禁止**：UTF-8 with BOM、GBK、GB2312、ANSI、ISO-8859-1 等混用
- 创建文件时必须强制使用 UTF-8（无 BOM）

### 2. 中文内容规范
- 中文注释、中文日志、中文文档必须可读，不允许乱码（如"鍥藉"）
- 发现乱码优先检查文件实际编码与 IDE 显示编码是否一致

### 3. 注释规范
- 保留已有业务注释，不随意删除历史说明和关键注释块
- 新增代码必须补充必要注释，说明"为什么这样做"，避免空泛注释

### 4. 变更原则
- 功能改动与注释改动尽量分开，便于回溯
- 修编码问题时不改业务逻辑，只做最小必要修改

---

## 术语约定
| 术语 | 含义 | 对应目录 |
|------|------|---------|
| **后端** | Rust Core（Tauri 后端进程） | `src-tauri/src/` |
| **前端** | React UI（WebView 进程） | `src/` |
| **配置** | Tauri 核心配置 | `src-tauri/tauri.conf.json` |
| **权限** | Capabilities 安全声明 | `src-tauri/capabilities/` |
| **Command** | Rust 侧可被前端调用的函数 | `#[tauri::command]` |
| **IPC** | 进程间通信（前端 ↔ Rust） | `invoke()` / `listen()` |

---

## 核心架构（必须牢记）

| 项目 | 规范 |
|------|------|
| **应用类型** | Tauri 2.x 桌面应用（双进程架构） |
| **后端语言** | Rust 2021 edition |
| **前端框架** | React 19 + TypeScript 5.8 |
| **UI 组件库** | Ant Design (v5+) + Lucide React 图标库 |
| **样式方案** | TailwindCSS 4 + CSS Variables 设计令牌 |
| **状态管理** | Zustand (v5+)（全局状态）+ React Hooks（局部状态） |
| **路由方案** | React Router 7（HashRouter） |
| **构建工具** | Vite 7 (前端) + Cargo (后端) |
| **通信机制** | Tauri IPC（`invoke` 调用 Rust Commands） |
| **序列化** | serde + serde_json（Rust ↔ JSON ↔ TypeScript） |
| **数据库** | SQLite（rusqlite，Rust 直接操作） |
| **错误处理** | thiserror（Rust）+ ErrorBoundary（React） |
| **安全模型** | Capabilities 细粒度权限声明 |
| **应用标识** | `com.agilefr.kb` |

### 双进程架构

```
┌───────────────────────────────────────────────────────┐
│                     Tauri 应用                         │
│                                                       │
│  ┌──────────────────┐  IPC (invoke)  ┌──────────────────┐
│  │   WebView 进程    │ ◄════════════► │   Rust Core 进程  │
│  │                  │                │                  │
│  │  React 19        │  Commands      │  commands/       │
│  │  Ant Design 5    │  Events        │  services/       │
│  │  TailwindCSS 4   │  ────────►     │  database/       │
│  │  Zustand         │                │  models/         │
│  │  React Router    │  ◄────────     │  error.rs        │
│  │                  │  返回值         │  state.rs        │
│  │  UI 渲染         │                │                  │
│  │  用户交互        │                │  系统 API        │
│  │  前端状态        │                │  文件操作        │
│  │                  │                │  SQLite 数据库   │
│  └──────────────────┘                └──────────────────┘
└───────────────────────────────────────────────────────┘
```

### 后端三层架构

```
Commands 层（IPC 入口）→ Services 层（业务逻辑）→ Database 层（数据访问）
```

### 分层职责

| 层级 | 职责 | 关键技术 |
|------|------|---------|
| **WebView 层** | UI 渲染、用户交互 | React 19 + Ant Design + TailwindCSS |
| **状态管理层** | 全局状态、设置管理 | Zustand（`src/store/`） |
| **API 封装层** | 统一 invoke 调用 | `src/lib/api/index.ts` |
| **IPC 桥接层** | 前后端通信 | `invoke()` / `listen()` |
| **Command 层** | IPC 接口定义 | `#[tauri::command]`（`src-tauri/src/commands/`） |
| **Service 层** | 业务逻辑 | `src-tauri/src/services/` |
| **Database 层** | 数据访问 DAO | `src-tauri/src/database/`（rusqlite） |
| **Plugin 层** | 功能扩展 | `tauri::Builder.plugin()` 注册 |
| **Capabilities 层** | 安全权限控制 | JSON 声明式权限 |

---

## 目录结构

```
tauri/
├── index.html                    # HTML 入口（SPA 挂载点）
├── package.json                  # Node.js 依赖和脚本
├── tsconfig.json                 # TypeScript 配置（含 @/ 路径别名）
├── vite.config.ts                # Vite 构建配置（TailwindCSS + 路径别名）
│
├── src/                          # ★ 前端源码（React + TypeScript）
│   ├── main.tsx                  # 前端入口（ReactDOM.createRoot）
│   ├── App.tsx                   # 主组件（ConfigProvider + 主题 + ErrorBoundary）
│   ├── Router.tsx                # 路由配置（React Router）
│   ├── vite-env.d.ts             # Vite 类型声明
│   ├── theme/                    # 主题配置
│   │   └── antdTheme.ts          # Ant Design 暗色/亮色主题
│   ├── styles/                   # 样式系统
│   │   ├── variables.css         # CSS 设计令牌（颜色/间距/圆角）
│   │   └── global.css            # TailwindCSS + 全局样式
│   ├── store/
│   │   └── index.ts              # Zustand 全局状态（主题/侧边栏）
│   ├── types/
│   │   └── index.ts              # TypeScript 类型定义（与 Rust 对齐）
│   ├── hooks/
│   │   └── useCommand.ts         # useCommand Hook + safeInvoke 工具
│   ├── lib/
│   │   └── api/
│   │       └── index.ts          # API 调用封装（systemApi / configApi）
│   ├── components/
│   │   ├── ui/
│   │   │   └── ErrorBoundary.tsx  # 错误边界组件
│   │   └── layout/
│   │       ├── AppLayout.tsx      # 应用布局（Sider + Header + Content）
│   │       └── Sidebar.tsx        # 侧边栏导航
│   └── pages/
│       ├── home/
│       │   └── index.tsx          # 首页
│       ├── settings/
│       │   └── index.tsx          # 设置页（配置管理）
│       └── about/
│           └── index.tsx          # 关于页（系统信息）
│
├── src-tauri/                    # ★ Rust 后端（Tauri Core）
│   ├── Cargo.toml                # Rust 依赖配置
│   ├── build.rs                  # Tauri 构建脚本
│   ├── tauri.conf.json           # ★ Tauri 核心配置
│   ├── capabilities/             # ★ 权限声明
│   │   └── default.json
│   ├── icons/                    # 应用图标
│   └── src/
│       ├── main.rs               # Rust 进程入口
│       ├── lib.rs                # ★ 核心入口（Builder + 插件 + Command 注册）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bkywksj/knowledge-base](https://github.com/bkywksj/knowledge-base) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
