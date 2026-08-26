---
trigger: always_on
description: > 本文件面向 AI 编程助手，帮助其快速理解项目结构和约定。详细架构设计见 [docs/architecture/](docs/architecture/README.md)。
---

# AGENTS.md

> 本文件面向 AI 编程助手，帮助其快速理解项目结构和约定。详细架构设计见 [docs/architecture/](docs/architecture/README.md)。

## 项目概述

DataZen 是一个跨平台桌面数据库管理工具，基于 **Tauri v2**（Rust 后端 + React 前端）构建，集成 AI 辅助功能。

- **框架**：Tauri v2 + React 18 + TypeScript + Tailwind CSS 4
- **包管理**：pnpm（前端）、Cargo workspace（Rust）
- **状态管理**：Zustand
- **测试**：Vitest（Host 单元）、驱动 crate 内单测/E2E、WebdriverIO（Host E2E）、手工黑盒（`test/`）
- **AI**：多 Provider（OpenAI / Anthropic / DeepSeek / Custom）、MCP Server/Client
- **运行模式**：GUI 桌面应用 或 无头 MCP stdio 服务器（`--mcp-stdio`）

## 目录结构

```text
datazen/
├── src/                         # React 前端源码
│   ├── components/              # UI 组件（ai/, chart/, connection/, DataTable/, ui/）
│   ├── windows/                 # 主工作区 *Page + 子窗口 *Window（见 architecture/windows.md）
│   │   └── connection/er/       # ER 图模块（React Flow）
│   ├── stores/                  # Zustand stores（connection / panel / schema / settings / ai / dashboard 等）
│   ├── commands/                # Tauri IPC 封装
│   ├── lib/                     # 工具库
│   ├── hooks/                   # React hooks
│   ├── locales/                 # i18n
│   ├── plugins/generated.ts     # 自动生成（gitignore；pnpm install / resolve-drivers）
│   └── plugin-sdk/              # 插件前端 SDK
├── src-tauri/                   # Rust 后端
│   ├── src/
│   │   ├── ai/                  # AI Provider / protocol / context
│   │   ├── commands/            # Tauri IPC 命令
│   │   ├── db/                  # DriverRegistry
│   │   ├── theme/               # 运行时主题包
│   │   ├── mcp/                 # MCP Server/Client
│   │   ├── workflow/             # YAML Workflow 引擎
│   │   ├── services/            # ConnectionManager, QueryExecutor, DbTools
│   │   ├── cache/               # SchemaCache
│   │   ├── store/               # AES-256-GCM 加密持久化
│   │   ├── data_sync/           # 同族 Data Synchronization（门闸 / 比较 / ChangeSet / 执行）
│   │   └── transfer/            # 异构 IR 适配器 / DDL 生成（非 data_sync 执行引擎）
│   └── resources/               # 菜单翻译、Prompt 模板
├── packages/
│   ├── driver-api/              # DatabaseDriver + Command API + inventory + ReuseDriver
│   ├── ai-api/                  # AiProvider trait + factory
│   ├── drivers/                 # path 驱动 crate（测试也写在各 crate 内）
│   │   └── <id>/                # Rust `src/` + `tests/`；UI `ui/__tests__/`；E2E `e2e/`
│   ├── extensions/              # 运行时插件源码包（UI 页 + 主题；安装测试见其 README）
│   ├── extension-sdk/           # 插件侧 SDK：类型化 RPC 客户端 / useTheme / theme.css
│   └── themes/                  # （已迁移）旧 v1 ThemePack 存档，见 extensions/
├── e2e/                         # Host WebdriverIO E2E（通用 UI / IPC；非驱动方言）
├── test/                        # 手工黑盒测试
└── docs/                        # 文档：features/（功能）、architecture/（架构）、development/（开发发布）
```

## 核心架构模式

### 驱动选型（编译时，类似 Caddy 2）

1. `drivers-registry.json` 定义 path 驱动 + git 驱动；Git 可钉 `ref`
2. `scripts/resolve-drivers.mjs` 构建前执行选型、克隆 Git driver，并生成 `generated.ts`、`plugin_init.rs`、`.plugin-features.json`（前两者 gitignore；`pnpm install` / `pnpm build` 会 `--codegen-only` 补齐）
3. 通过 `inventory` crate 实现链接时自动注册；宿主 `DriverRegistry` 仅走 factories

```bash
pnpm tauri:dev                         # 默认 basic
pnpm tauri:dev --drivers=all
pnpm tauri:dev --drivers=basic,kiwi,superset
pnpm tauri:dev --drivers=postgres,mongodb,kiwi
DATAZEN_DRIVERS=all pnpm tauri:dev
DATAZEN_DRIVERS=all pnpm tauri:build
```

### 数据库驱动

- Path 驱动：`packages/drivers/*`（crate 名 `datazen-driver-<id>`），经 optional Cargo feature 注入
- Git 驱动：克隆到 `packages/drivers/<id>/`（gitignore，非 Cargo workspace member），同样 inventory 注册
- 前端 `DB_REGISTRY` 合并 `generated.ts` 的 `DRIVER_DB_ENTRIES`
- 默认 DB 图标来自 `packages/drivers/*/ui/icons/{dbType}.svg`
- 关键 trait 方法包括 `supports_offset()`、`supports_explain()`、`prompt_overrides()`
- **驱动相关测试必须写在该驱动 crate 内**，不要加到 Host（`src-tauri/`、`src/`、`e2e/specs/`）。详见下方「驱动测试落点」

### Driver Command API

`packages/driver-api` 提供统一 Command 抽象（`command_definitions()` + `execute_command()`）。`query` / `execute` 是内置 Command 有默认实现。Workflow、IPC、前端 Command Editor 都依赖 Command Definition，不按 Driver 类型硬编码。`ReuseDriver` 必须转发 Command discovery 与 execution。`metadata.requiresConnection = false` 的 Command 可通过 `driverType` 执行。

### Redis 驱动

深度能力集中在 `packages/drivers/redis`（UI + Driver Command API），宿主仅为薄 Tab 壳。**禁止** Host 按 `pluginId === 'redis'` 写设置分支。操作一律走 `execute_command` / `execute_driver_command`。

### AI 模块

多 Provider（OpenAI / Anthropic / DeepSeek / Custom）；`PromptResolver` 优先级：用户覆盖 → 驱动覆盖 → 资源文件 → 编译时英文嵌入。详见 [docs/architecture/backend/ai.md](docs/architecture/backend/ai.md)。

### MCP

Server 暴露 Tools/Resources/Prompts（DB tools 使用持久化 `connection_id`）；Client 连接外部 MCP Server；`--mcp-stdio` 启动无头模式。详见 [docs/architecture/backend/mcp.md](docs/architecture/backend/mcp.md)。

## Workflows

YAML 驱动的通用执行引擎，GUI、Tauri IPC 和 MCP 共用同一 runtime。Step 通过 Driver Command API 执行；Workflow 默认 connection 可被 Step 继承或覆盖；旧 `type: query` 自动规范化为 `Command("query")`。Workflow UI 通过 `command_definitions()` 动态发现可用 Command，不硬编码。

详细设计：[Workflow 架构文档](docs/architecture/backend/workflow.md)；用户手册：[docs/features/workflow-guide.zh-CN.md](docs/features/workflow-guide.zh-CN.md)。

### 运行时插件系统（Extensions：UI 页面 + 主题）


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flyxl/datazen](https://github.com/flyxl/datazen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
