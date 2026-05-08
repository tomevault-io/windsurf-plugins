---
trigger: always_on
description: > 本文档面向 AI 编码代理。阅读者应对本项目一无所知，所有信息均基于实际代码与配置，不假设、不泛化。
---

# SiYuan MCP Sisyphus —— AI 编码代理项目指南

> 本文档面向 AI 编码代理。阅读者应对本项目一无所知，所有信息均基于实际代码与配置，不假设、不泛化。

---

## 项目概述

**SiYuan MCP Sisyphus** 是 SiYuan Note（思源笔记）的插件 + 独立 CLI 工具，让 AI Agent 能够通过两种接口安全地操作思源笔记：

1. **MCP Server 插件**：作为 SiYuan 插件运行，对外暴露 MCP（Model Context Protocol）服务。AI 客户端（Claude Desktop、Cursor、Cherry Studio 等）通过 HTTP 或 stdio 连接。
2. **独立 CLI `siyuan-sisyphus`**：发布到 npm 的包名 `siyuan-sisyphus`，安装后提供 `siyuan-sisyphus` / `siyuan` 命令。直接通过思源 HTTP API 执行单次操作后退出，无需 MCP 客户端。

两种接口共享同一套底层能力（10 个聚合工具、115+ 个 action），覆盖思源绝大部分功能：笔记本管理、文档操作、块级读写、属性视图（数据库）、搜索、标签、文件资源、闪卡、系统接口等。

- 仓库地址：`https://github.com/yangtaihong59/siyuan-plugins-mcp-sisyphus`
- 作者：Taihong Yang
- 许可证：MIT
- 当前版本：`0.3.3`（根 `package.json` 与 `plugin.json` 同步）

---

## 技术栈

| 层级 | 技术 |
|------|------|
| 语言 | TypeScript 5.1+ |
| 构建工具 | Vite 5.2+（多 target 构建） |
| UI 框架 | Svelte 4（插件设置面板、桌面悬浮宠物组件） |
| 校验 | Zod 4（action 参数校验） |
| MCP SDK | `@modelcontextprotocol/sdk` ^1.26.0 |
| 测试 | Vitest 1.6+（Node 环境） |
| 文档 | VitePress 1.6+（`docs/` 双语站点） |
| 包管理 | pnpm（推荐） |

**关键约束**：
- 所有产物为 **CommonJS (CJS)**。
- 插件在 Electron 渲染进程以 CJS 运行；MCP Server 以 Node 进程运行；CLI 为自包含 CJS bundle。
- **必须兼容远程场景**：任何读写操作都经过思源 HTTP API（`http://127.0.0.1:6806` 或用户配置地址），**禁止直接访问本地文件系统**。特许例外仅有两处：① CLI 自身配置（`~/.siyuan-sisyphus/config.json`）；② 上传/下载/导出类 action（如 `upload_asset`、`export_resources`），因 SiYuan API 不支持流式二进制传输，必须通过本地文件系统中转。

---

## 项目结构

```
siyuan-plugins-mcp-sisyphus/
├── src/
│   ├── index.ts                 # 插件入口（SiYuan Plugin 基类）
│   ├── server-launcher.ts       # HttpServerLauncher：在插件进程内 spawn mcp-server.cjs 子进程
│   ├── index.scss               # 插件全局样式（几乎为空）
│   │
│   ├── api/                     # 思源 HTTP API 封装层
│   │   ├── client.ts            # SiYuanClient：fetch 封装、token 注入、超时控制
│   │   ├── notebook.ts          # /api/notebook/*
│   │   ├── document.ts          # /api/filetree/* 与文档相关
│   │   ├── block.ts             # /api/block/*
│   │   ├── av.ts                # /api/av/*（attribute view / 数据库）
│   │   ├── search.ts            # /api/search/*、/api/query/sql
│   │   ├── file.ts              # /api/file/*、/api/export/*、/api/asset/*
│   │   ├── system.ts            # /api/system/*
│   │   ├── tag.ts               # /api/tag/*
│   │   ├── flashcard.ts         # /api/riff/*
│   │   └── transaction.ts       # /api/transaction 批量操作
│   │
│   ├── mcp/                     # MCP 服务器核心
│   │   ├── server.ts            # MCP Server 入口：createSiYuanServer()、startMcpServer()
│   │   ├── http-transport.ts    # HTTP 模式 MCP 传输层（SSE / postMessage）
│   │   ├── tool-registry.ts     # TOOL_REGISTRY：10 个聚合工具的注册表
│   │   ├── tool-lifecycle.ts    # 工具调用生命周期：analytics、telemetry、token 计数、错误包装
│   │   ├── config.ts            # ToolConfig 类型、默认值、配置迁移（扁平 → 嵌套）、危险动作定义
│   │   ├── permissions.ts       # PermissionManager：笔记本级权限（rwd/rw/r/none）
│   │   ├── types.ts             # 所有 action 的 Zod schema 定义
│   │   ├── resources.ts         # MCP Resources：help 文档资源路由
│   │   ├── help.ts              # 各工具的帮助文案与提示
│   │   ├── normalize.ts         # 请求参数归一化（类型短码展开、sortBy 别名等）
│   │   ├── analytics.ts         # 调用统计与洞察数据聚合
│   │   ├── telemetry.ts         # 调用事件遥测上报
│   │   ├── token-usage.ts       # 请求/响应近似 token 计算
│   │   ├── puppy-state.ts       # 桌面悬浮宠物（ToolPuppy）状态管理
│   │   ├── server-instructions.ts # 服务端 instructions 文本构建
│   │   ├── runtime.ts           # 运行时环境检测（isPluginMode 等）
│   │   ├── noop-*.ts            # 用于替换 MCP SDK 重模块的 no-op shim
│   │   └── tools/               # 10 个聚合工具的实现
│   │       ├── define-tool.ts   # defineTool() 工厂：统一工具定义模式
│   │       ├── shared.ts        # 聚合工具公共函数：schema 构建、结果包装、分页、错误翻译
│   │       ├── context.ts       # 工具上下文辅助（权限校验、子文档查询等）
│   │       ├── errorTranslation.ts # 思源错误码 → 用户友好文案
│   │       ├── ui-refresh.ts    # 触发思源 UI 刷新（所有模式默认开启）
│   │       ├── help-render.ts   # help action 输出渲染
│   │       ├── schema-analyzer.ts # schema 分析与描述裁剪
│   │       ├── notebook.ts      # notebook 工具
│   │       ├── document.ts      # document 工具
│   │       ├── block.ts         # block 工具
│   │       ├── av.ts            # av（数据库）工具
│   │       ├── search.ts        # search 工具
│   │       ├── file.ts          # file 工具
│   │       ├── tag.ts           # tag 工具
│   │       ├── system.ts        # system 工具
│   │       ├── flashcard.ts     # flashcard 工具
│   │       └── mascot.ts        # mascot（吉祥物余额）工具
│   │
│   ├── cli/                     # 独立 CLI 源码（被 cli Vite target 打包）
│   │   ├── index.ts             # CLI 入口：命令分发（dispatch/list/help/init/config/version）
│   │   ├── dispatch.ts          # 核心转发：调 TOOL_REGISTRY[tool].callTool()
│   │   ├── args.ts              # 顶层参数解析：支持 kebab/camel/snake flag 混用
│   │   ├── flag-mapper.ts       # 基于 inputSchema 做 flag → 参数映射与类型强转
│   │   ├── render.ts            # 人类可读渲染 / --json 紧凑输出 / ANSI 颜色 / 交互式分页
│   │   ├── list-help.ts         # list 与 help 子命令实现
│   │   ├── config.ts            # 配置读写（~/.siyuan-sisyphus/config.json）
│   │   ├── config-command.ts    # config 子命令：多 profile 管理（list/get/set/use）
│   │   ├── init.ts              # 交互式初始化
│   │   ├── plugin-check.ts      # 插件在线检测
│   │   └── runtime.ts           # CLI 运行时环境检测
│   │
│   ├── components/              # Svelte UI 组件
│   │   ├── ToolPuppy.svelte     # 桌面悬浮宠物主组件
│   │   ├── Puppy*.svelte        # 宠物子组件（气泡、SVG、结果遮罩等）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yangtaihong59/siyuan-plugins-mcp-sisyphus](https://github.com/yangtaihong59/siyuan-plugins-mcp-sisyphus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
