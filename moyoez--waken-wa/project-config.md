---
trigger: always_on
description: 面向在本仓库中协作的人类开发者与 AI Agent：说明项目目的、目录结构、数据与环境约定，以及修改代码时的注意点。
---

# AGENTS.md — 开发指引

面向在本仓库中协作的人类开发者与 AI Agent：说明项目目的、目录结构、数据与环境约定，以及修改代码时的注意点。

---

## 1. 项目简介

**waken-wa**（见 [package.json](package.json)）是基于 **Next.js App Router** 的个人站点：

- **访客**：首页展示个人状态、日程、动态时间线、灵感内容等；可配置整站访问锁（hCaptcha 等）。
- **管理员**：后台管理用户、设备、API Token、站点设置、灵感素材与活动等。

数据层使用 **Drizzle ORM**。运行时根据环境在 **SQLite**（本地 `file:`，默认开发）与 **PostgreSQL**（部署常见）之间切换。

### 技术栈一览

| 层级 | 说明 |
|------|------|
| 框架 | Next.js 16.x（[package.json](package.json)），`output: 'standalone'`（[next.config.mjs](next.config.mjs)） |
| UI | React 19、Tailwind 4、Radix UI、[components/ui/](components/ui/)（类 shadcn 结构） |
| 数据 | Drizzle、[better-sqlite3](package.json) / [pg](package.json)；双 schema：[drizzle/schema.sqlite.ts](drizzle/schema.sqlite.ts)、[drizzle/schema.pg.ts](drizzle/schema.pg.ts) |
| 统一表导出 | [lib/drizzle-schema.ts](lib/drizzle-schema.ts) 按 `DATABASE_URL` 是否为 Postgres URL 选择 |
| DB 入口 | [lib/db.ts](lib/db.ts)（`server-only`），开发环境会缓存连接实例 |
| 认证 | JWT（[jose](package.json)）+ Cookie；密码 [bcryptjs](package.json)；核心 [lib/auth.ts](lib/auth.ts) |
| 校验 | Zod、react-hook-form |
| 活动实时更新 | [app/api/activity/stream/route.ts](app/api/activity/stream/route.ts)：SSE，服务端定时聚合数据（非 WebSocket） |

---

## 2. 仓库地图

```mermaid
flowchart TB
  subgraph app_layer [app]
    pages[pages admin inspiration]
    api[api routes]
  end
  subgraph shared [shared]
    lib[lib]
    hooks[hooks]
    comp[components]
    types[types]
  end
  subgraph data [data]
    schema[drizzle schemas]
    scripts[scripts]
  end
  pages --> lib
  api --> lib
  lib --> schema
  scripts --> schema
```

### 顶级目录职责

- **[app/](app/)**
  - **页面**：[app/page.tsx](app/page.tsx) 首页（站点锁、资料、日程横幅、灵感、活动流等，`dynamic = 'force-dynamic'`）；[app/admin/](app/admin/)（登录、仪表盘、[setup](app/admin/setup/page.tsx)）；[app/inspiration/](app/inspiration/) 灵感列表与详情。
  - **API**
    - 认证：[app/api/auth/](app/api/auth/)（login / logout / session）
    - 管理：[app/api/admin/](app/api/admin/)（users、settings、devices、tokens、activity、settings/export、setup/admin、change-password 等）
    - 活动：[app/api/activity/route.ts](app/api/activity/route.ts)、[app/api/activity/stream/route.ts](app/api/activity/stream/route.ts)
    - 灵感：[app/api/inspiration/](app/api/inspiration/)（entries、assets、img）
    - 站点解锁：[app/api/site/unlock/route.ts](app/api/site/unlock/route.ts)
- **[lib/](lib/)**：数据库、认证、活动聚合（含 Steam：[lib/steam.ts](lib/steam.ts)、[lib/activity-feed.ts](lib/activity-feed.ts)）、主题、站点配置、[lib/rate-limit.ts](lib/rate-limit.ts) 等。
- **[components/](components/)**：业务组件（如 [current-status.tsx](components/current-status.tsx)、[activity-feed-provider.tsx](components/activity-feed-provider.tsx)）、[components/admin/](components/admin/)、[components/ui/](components/ui/)。
- **[hooks/](hooks/)**：如 [use-activity-feed.ts](hooks/use-activity-feed.ts)、[use-is-client.ts](hooks/use-is-client.ts)。
- **[types/](types/)**：领域与 API 相关 TypeScript 类型。
- **[drizzle/](drizzle/)**：双 schema；本地开发数据库默认放在 [data/dev.db](data/dev.db)。
- **[scripts/](scripts/)**：环境解析与数据库脚本（见下文）。
- **[proxy.ts](proxy.ts)**：导出 `proxy` 与 `matcher`，对敏感路径限流、对 `/api/admin/*`（除 setup）要求 `session` Cookie。若你使用的 Next.js 版本对边界层文件名或导出约定不同，以**当前仓库能实际生效的配置**为准，并查阅对应版本官方文档。

---

## 3. 本地开发

- **依赖安装与构建**：由开发者在本地执行（例如 `pnpm install`、`pnpm dev`、`pnpm build`）；请勿假设 CI/Agent 环境已安装依赖。
- **Node / 包管理器版本**：[package.json](package.json) 未声明 `engines` 时，以团队约定或部署环境为准。

常用脚本（定义见 [package.json](package.json)）：

| 命令 | 用途 |
|------|------|
| `pnpm dev` | 开发服务器 |
| `pnpm build` / `pnpm start` | 生产构建与启动 |
| `pnpm lint` | ESLint |
| `pnpm db:push` | 按当前环境选择配置执行 `drizzle-kit push`（[scripts/drizzle-push-by-env.mjs](scripts/drizzle-push-by-env.mjs)） |
| `pnpm db:push:postgres` | 强制 PostgreSQL 配置推送 |
| `pnpm db:init` | 数据库初始化脚本 [scripts/init-db.mjs](scripts/init-db.mjs) |

`postinstall` 会运行 [scripts/init-db.mjs](scripts/init-db.mjs)，用于安装后的数据库准备（与脚本实现保持一致）。

---

## 4. 数据库与环境变量

### 选择 SQLite 还是 PostgreSQL

- **`DATABASE_URL`** 为主开关：值为 `postgres(ql)://...` 时使用 PostgreSQL（见 [lib/db-env.ts](lib/db-env.ts)、[lib/db.ts](lib/db.ts)）。
- 未配置或非 Postgres URL 时，应用侧默认使用 SQLite 文件路径（例如 [lib/db.ts](lib/db.ts) 中的 `file:./data/dev.db` 逻辑）。

### 别名与脚本

- [lib/db-env.ts](lib/db-env.ts)：`applyDatabaseUrlAliases()`、`pickPostgresUrlFromEnv()`，在 `DATABASE_URL` 缺失时从 `POSTGRES_URL` 等补齐。
- [scripts/resolve-database-env.mjs](scripts/resolve-database-env.mjs)：加载 `.env` / `.env.local`、选择 Drizzle 配置、`POSTGRES_URL_NON_POOLING` 在初始化时优先（直连）。

### 其他

- **`JWT_SECRET`**：可选；未设置时从数据库 `system_secrets` 读取或生成（[lib/auth.ts](lib/auth.ts)）。
- hCaptcha、反向代理下的公开 URL 等：见各模块注释，例如 [lib/public-request-url.ts](lib/public-request-url.ts) 中的 `PUBLIC_APP_URL`。

### 双 schema 约定

新增或变更表结构时须同时维护：

1. [drizzle/schema.pg.ts](drizzle/schema.pg.ts)
2. [drizzle/schema.sqlite.ts](drizzle/schema.sqlite.ts)
3. 在 [lib/drizzle-schema.ts](lib/drizzle-schema.ts) 中导出并在应用中使用统一符号

推送 schema 使用 `pnpm db:push` 或 `pnpm db:push:postgres`（具体 config 文件见 [drizzle.config.pg.ts](drizzle.config.pg.ts)、[drizzle.config.sqlite.ts](drizzle.config.sqlite.ts)）。

### SQLite JSON 绑定注意事项

- **SQLite（better-sqlite3）参数绑定不接受对象/数组**：写入时只能 bind number/string/bigint/buffer/null。
- 若你在 PG 用 `jsonb(...)`、在 SQLite 用 `text(..., { mode: 'json' })`：
  - **写入 SQLite 时不要直接传 JS object**；请传 `JSON.stringify(value)`（或确保 Drizzle/driver 层会做 stringify）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MoYoez/waken-wa](https://github.com/MoYoez/waken-wa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
