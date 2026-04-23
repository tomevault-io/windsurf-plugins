---
trigger: always_on
description: 本文件为 `SublinkPro` 的人类与 AI 贡献者提供仓库级工作说明。
---

# AGENTS.md

本文件为 `SublinkPro` 的人类与 AI 贡献者提供仓库级工作说明。  
This file gives human and AI contributors a repo-specific working guide for `SublinkPro`.

## 1. 项目形态 / Project shape

- 这是一个**单仓库全栈应用**，不是 monorepo。  
  This repository is a **single full-stack application**, not a monorepo.
- **后端**位于仓库根目录，使用 Go。  
  The **backend** lives at the repo root and is written in Go.
- **前端**位于 `webs/`，使用 React + Vite。  
  The **frontend** lives in `webs/` and uses React + Vite.
- **生产环境集成方式**是先构建前端资源，再由 Go 产物在生产构建中提供并嵌入这些静态资源。  
  **Production integration** works by building the frontend first, then serving and embedding those built assets from the Go production build.

主要入口文件：  
Primary entry points:

- 后端：`main.go`  
  Backend: `main.go`
- 前端：`webs/src/index.jsx`  
  Frontend: `webs/src/index.jsx`

关键架构边界：  
Important architectural boundaries:

- `routers/`：注册路由。  
  `routers/`: route registration.
- `api/`：HTTP handler / controller 逻辑。  
  `api/`: HTTP handlers / controller logic.
- `services/`：业务逻辑与长生命周期后台流程。  
  `services/`: business logic and long-running/background workflows.
- `models/`：持久化、模型与迁移相关逻辑。  
  `models/`: persistence, data models, and migration-related logic.
- `middlewares/`：认证与请求链路中间件。  
  `middlewares/`: auth and request middleware.
- `node/`：订阅与协议解析/转换逻辑。  
  `node/`: subscription and protocol parsing/conversion logic.
- `webs/src/api/`：前端请求边界。  
  `webs/src/api/`: the frontend API boundary.
- `webs/src/views/`：前端页面级功能。  
  `webs/src/views/`: page-level frontend features.

## 2. 事实来源 / Source of truth

当多份说明不一致时，按下面顺序信任这些文件：  
When instructions conflict, trust these files first:

1. `webs/package.json`：前端命令与依赖事实来源。  
   `webs/package.json`: source of truth for frontend scripts and dependencies.
2. `docs/development.md`：本地开发流程与目录说明。  
   `docs/development.md`: local development workflow and project structure.
3. `docs/configuration.md`：配置优先级、环境变量与运行语义。  
   `docs/configuration.md`: config precedence, environment variables, and runtime behavior.
4. `.github/workflows/build-release.yml`：CI / release 构建预期。  
   `.github/workflows/build-release.yml`: CI and release build expectations.
5. `Dockerfile`：生产构建路径的权威实现。  
   `Dockerfile`: canonical production build sequence.

对于前端命令、输出目录和工具链，请优先依赖上面的仓库文件，不要套用通用模板或旧脚手架默认值。  
For frontend commands, output paths, and toolchain behavior, rely on the repository files above rather than generic template defaults or framework assumptions.

## 3. 技术栈 / Tech stack

- 后端：Go + Gin + GORM  
  Backend: Go + Gin + GORM
- 核心网络/代理内核：mihomo（MetaCubeX）  
  Core network/proxy engine: mihomo (MetaCubeX)
- 默认数据库：SQLite  
  Default database: SQLite
- 可选数据库：MySQL / PostgreSQL（通过 DSN）  
  Optional databases: MySQL / PostgreSQL via DSN
- 前端：React 19 + Vite  
  Frontend: React 19 + Vite
- UI：Material UI  
  UI: Material UI
- 前端包管理器：Yarn 4（`packageManager: yarn@4.10.3`）  
  Frontend package manager: Yarn 4 (`packageManager: yarn@4.10.3`)
- 定时调度：`robfig/cron`  
  Scheduler: `robfig/cron`

### mihomo 说明 / About mihomo

- `mihomo` 在这个仓库里不是普通第三方依赖，而是多个核心网络能力的底层支撑。  
  In this repository, `mihomo` is not a minor third-party dependency; it is the underlying engine for several core networking capabilities.
- 从上游语义上看，mihomo 更接近**代理 / 路由 / DNS / 网络出站控制内核**，而不是单纯的 GUI、单一协议实现，或“只是一个客户端”。  
  Upstream, mihomo is better described as a **proxy / routing / DNS / outbound networking core**, not merely a GUI, a single protocol implementation, or “just a client.”
- 在本仓库里，contributors 应把 mihomo 理解为“代理适配器创建、代理拨号、延迟/速度测试、DNS 解析、Host 注入与部分代理外呼能力”的关键集成点。  
  In this repo, contributors should treat mihomo as the key integration point for proxy adapter creation, proxied dialing, delay/speed testing, DNS resolution, Host injection, and some outbound proxy-backed requests.

## 4. 目录指南 / Directory guide

### 后端 / Backend

- `main.go`：应用入口、启动编排、配置/数据库初始化、服务启动与静态资源服务。  
  `main.go`: application entry, startup orchestration, bootstrap, and serving behavior.
- `api/`：后端 HTTP handler。  
  `api/`: backend HTTP handlers.
- `routers/`：路由注册。  
  `routers/`: route registration.
- `services/`：业务服务与后台子系统。  
  `services/`: business logic and long-running subsystems.
- `services/mihomo/`：mihomo 集成核心层；处理代理适配器、测速、DNS 解析和 Host 同步。  
  `services/mihomo/`: core mihomo integration layer for proxy adapters, testing, DNS resolution, and Host synchronization.
- `services/scheduler/`：定时任务与调度管理；涉及 cron / 任务中心时优先从这里入手。  
  `services/scheduler/`: scheduled jobs and scheduler management; start here for cron/task work.
- `models/`：数据库模型、迁移和持久化逻辑。  
  `models/`: database models, migrations, and persistence logic.
- `middlewares/`：认证和请求链路中间件。  
  `middlewares/`: auth and request pipeline logic.
- `utils/`：共享工具函数。  
  `utils/`: shared helpers.
- `template/`：运行时模板资源。  
  `template/`: runtime template assets.
- `config/`、`settings/`、`database/`、`constants/`、`dto/`：后端支持层。  
  `config/`, `settings/`, `database/`, `constants/`, and `dto/`: backend support layers.

### 前端 / Frontend

- `webs/src/views/`：主要页面与功能入口。  
  `webs/src/views/`: page-level features.
- `webs/src/api/`：前端请求层。  
  `webs/src/api/`: request layer.
- `webs/src/routes/`：路由与导航结构。  
  `webs/src/routes/`: route setup and navigation structure.
- `webs/src/layout/`、`webs/src/themes/`、`webs/src/components/`：共享 UI 外壳与组件。  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZeroDeng01/sublinkPro](https://github.com/ZeroDeng01/sublinkPro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
