---
trigger: always_on
description: > 版本: 2.0.0-beta.12 | 许可: MIT | 仓库: https://github.com/PapyrusOR/Papyrus_Desktop
---

﻿# Papyrus 项目开发信息

> 版本: 2.0.0-beta.12 | 许可: MIT | 仓库: https://github.com/PapyrusOR/Papyrus_Desktop

## 项目简介

Papyrus（莎草纸）是一款专注于高强度记忆训练的极简、全键盘驱动、AI Agent 加持的**间隔重复（SRS）复习引擎**桌面应用。核心理念为"大道至简"——通过极简交互帮助用户进入深度复习的"心流"状态。

---

## 技术栈

| 层级 | 技术 | 版本 |
|------|------|------|
| 后端运行时 | Node.js | 24+ |
| 后端语言 | TypeScript | 5 |
| 后端框架 | Fastify | 5 |
| 前端框架 | React | 19.2.4 |
| 前端语言 | TypeScript | 5 |
| 前端构建 | Vite | 8 |
| UI 组件库 | Arco Design (web-react) | 2.66.14 |
| CSS 框架 | Tailwind CSS | 3.4（类名 `tw-` 前缀） |
| 桌面壳 | Electron | 41.1.0 |
| 打包 | electron-builder | 26.8 |
| 算法 | SM-2 间隔重复 | — |
| AI SDK | OpenAI SDK | 4.96 |
| 校验 | Zod | 3.25 |
| 国际化 | i18next / react-i18next | 26 / 17 |
| 后端测试 | Jest + ts-jest | 29 |
| E2E 测试 | Playwright | 1.59 |
| CI/CD | GitHub Actions | — |

---

## 目录结构

```
Papyrus-beta12/
├── backend/                    # Node.js + TypeScript 后端
│   ├── src/
│   │   ├── ai/                 # AI 功能模块
│   │   │   └── tools/          # AI 工具定义 (cards, notes, files, data, relations, settings, extensions)
│   │   ├── api/                # Fastify 路由 & 服务器
│   │   │   ├── server.ts       # 服务入口，注册所有路由
│   │   │   └── routes/         # 20+ 路由模块
│   │   ├── core/               # 核心业务逻辑
│   │   │   ├── cards.ts        # 卡片 CRUD
│   │   │   ├── notes.ts        # 笔记管理
│   │   │   ├── sm2.ts          # SM-2 算法
│   │   │   ├── versioning.ts   # 版本历史
│   │   │   ├── crypto.ts       # AES-GCM 加密
│   │   │   ├── relations.ts    # 关系管理
│   │   │   └── files.ts        # 文件操作
│   │   ├── db/                 # SQLite 持久化 (node:sqlite, WAL)
│   │   ├── cli/                # Desktop CLI 管理辅助
│   │   ├── integrations/       # 外部集成 (file-watcher/Obsidian)
│   │   ├── mcp/                # MCP 服务端点
│   │   └── utils/              # 工具 (auth, logger, paths, proxy, client-id)
│   └── tests/                  # 测试 (unit/ + integration/)
├── frontend/                   # React 19 前端
│   └── src/
│       ├── StartPage/          # 首页 (今日概览、复习队列、节气主题)
│       ├── ScrollPage/         # 卷轴复习页 (闪卡学习)
│       ├── NotesPage/          # 笔记管理 (关系图、文件夹树)
│       ├── ChartsPage/         # 统计图表
│       ├── FilesPage/          # 文件库
│       ├── ExtensionsPage/     # 扩展管理
│       ├── SettingsPage/       # 设置 (AI配置、无障碍、外观、快捷键)
│       ├── ChatPanel/          # AI 聊天面板
│       ├── components/         # 公共组件 (MarkdownView, ReasoningChain, ToolCallCard...)
│       ├── hooks/              # 自定义 Hooks
│       ├── i18n/               # 国际化配置
│       ├── icons/              # 图标系统 (30+ AI 模型/提供商 Logo)
│       ├── locales/            # 语言包 (zh-CN, en-US, zh-TW, ja-JP)
│       ├── contexts/           # React Context (AccessibilityContext)
│       └── utils/              # 工具函数
├── electron/                   # Electron 主进程
│   ├── main.js                 # 主进程入口
│   ├── preload.js              # 预加载脚本
│   ├── diagnostic-window.js    # 诊断窗口
│   └── diagnostic-preload.js   # 诊断预加载
├── e2e/                        # Playwright E2E 测试
├── scripts/                    # 构建/发布脚本
├── build/                      # Electron 构建资源 (证书、NSIS、macOS 权限)
├── assets/                     # 应用图标 (.ico, .icns, .png, .svg)
├── docs/                       # 项目文档
├── examples/                   # 扩展开发模板
└── tools/                      # 开发工具 (图标生成)
```

---

## 开发命令

### 根目录（Monorepo 协调）

| 命令 | 说明 |
|------|------|
| `.\start-dev.ps1` | 自动检查端口与依赖后启动前后端 |
| `npm run electron:dev` | Electron 开发模式 |
| `npm run build:frontend` | 构建前端生产版本 |
| `npm run build:backend` | 构建后端生产版本 |
| `npm run build:installer` | 完整构建安装包 |
| `npm run electron:build` | 全平台构建 |
| `npm run electron:build:win` | 仅构建 Windows |
| `npm run electron:build:mac` | 仅构建 macOS |
| `npm run electron:build:linux` | 仅构建 Linux |
| `npm run bump:patch/minor/major/beta/release` | 版本号管理 |
| `npm run sync-version` | 同步版本号到子包 |
| `npm run generate-icons` | 生成图标 |
| `npm run generate-cert` | 生成代码签名证书（PowerShell） |

### 后端 (`backend/`)

| 命令 | 说明 |
|------|------|
| `npm run dev` | tsx watch 热重载 Fastify |
| `npm run build` | tsc 编译到 dist/ |
| `npm run start` | 运行编译后的 dist/api/server.js |
| `npm run typecheck` | tsc --noEmit 类型检查 |
| `npm test` | Jest 单元 + 集成测试 |
| `npm run test:watch` | Jest 监听模式 |

### 前端 (`frontend/`)

| 命令 | 说明 |
|------|------|
| `npm run dev` | Vite 开发服务器 (localhost:5173) |
| `npm run build` | 生产构建到 dist/ |
| `npm run typecheck` | TypeScript 类型检查 |

### E2E 测试

| 命令 | 说明 |
|------|------|
| `npx playwright test` | Playwright E2E 测试 |

---

## 架构概览

### 前后端通信

- 后端默认监听 `127.0.0.1:8000`，可通过 `PAPYRUS_PORT` 环境变量覆盖
- 前端开发时通过 Vite proxy 将 `/api` 请求代理到后端
- Electron 模式下通过 `PAPYRUS_AUTH_TOKEN` 进行本地 API 保护
- 用户数据默认存储在 `$HOME/PapyrusData`，可通过 `PAPYRUS_DATA_DIR` 覆盖

### 后端架构

```
backend/src/
├── api/server.ts           # Fastify 应用入口，注册路由、CORS、限流、认证
├── api/routes/             # 20+ 路由模块
├── core/                   # 核心业务逻辑（UI 无关）
│   ├── cards.ts            # 卡片 CRUD
│   ├── notes.ts            # 笔记管理
│   ├── sm2.ts              # SM-2 间隔重复算法
│   ├── versioning.ts       # 版本历史
│   ├── crypto.ts           # AES-GCM 加密
│   ├── relations.ts        # 关系管理
│   └── files.ts            # 文件操作
├── ai/                     # AI Agent 系统
│   ├── config.ts           # AI 配置管理
│   ├── provider.ts         # AI 提供商接口
│   ├── tool-manager.ts     # 工具调用管理
│   ├── llm-cache.ts        # LLM 响应缓存

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LiYuanStudio/Papyrus_Desktop](https://github.com/LiYuanStudio/Papyrus_Desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
