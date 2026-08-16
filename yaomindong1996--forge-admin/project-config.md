---
trigger: always_on
description: 本文件为 Claude Code（claude.ai/code）在本仓库中开发时提供指导。
---

# CLAUDE.md

本文件为 Claude Code（claude.ai/code）在本仓库中开发时提供指导。

## 仓库概述

Forge Admin 是一套企业级中后台框架：前端 **Vue 3 + Naive UI**，后端 **Java 17 + Spring Boot 3** 微内核插件化架构。内置 RBAC + 多租户 + 数据权限、AI 代码生成、Flowable 工作流、AI 大屏报表、H5/uni-app 移动端，以及较新的 **统一能力开放平台（capability）** 与 **企业微信（WeCom）协作** 能力。

后端位于 **`forge-server/`**（不是 `forge/` —— README.md 与 AGENTS.md 仍用过时的 `forge/` 路径；其中 AGENTS.md 第 16 行写了 `forge-server/`，但下方完整模块树、启动命令、配置路径仍全是 `forge/`，前后矛盾）。前端共**三个端**：`forge-admin-ui/`（管理端，dev :3000）、`forge-report-ui/`（AI 大屏设计器，dev :3021）、`forge-h5-ui/`（uni-app 移动端 H5，dev :3001）。文档：`forge-docs/`（VitePress）。各端位置、端口、代理细节见下方"常用命令 → 前端"。

## 常用命令

### 后端（`forge-server/`）
```bash
cd forge-server && mvn clean install -DskipTests   # 全量构建（快速）
cd forge-server && mvn test -P enable-tests        # 运行测试（默认跳过测试！）
cd forge-server/forge-admin-server && mvn spring-boot:run        # 主后台服务，端口 8580
cd forge-server/forge-report-server && mvn spring-boot:run       # 大屏服务，端口 8581
cd forge-server/forge-app-server && mvn spring-boot:run          # App 接口服务，端口 8583
cd forge-server/forge-flow/forge-flow-server && mvn spring-boot:run  # 独立流程服务，端口 8081
```
Maven Profile（默认 `dev`）：`local`、`dev`、`prod`。测试执行受 `enable-tests` Profile 门控 —— 直接 `mvn test` 会跳过测试。构建需要 Java 17、MySQL 8+、Redis 6+。登录账号：`admin` / `123456`。

**后端各服务端口与前端代理的对应关系**：`forge-admin-server`(8580) ↔ 管理端 `/dev-api`、`forge-app-server`(8583) ↔ H5 `/dev-api`、`forge-report-server`(8581) ↔ 大屏 `/forge-report-api`、`forge-flow-server`(8081) ↔ 管理端与 H5 的 `/api/flow`。`forge-business`、`forge-flow-client` 为业务/客户端模块，非独立服务。

本地配置：把 `forge-server/forge-admin-server/src/main/resources/application-dev.example.yml` 复制为 `application-dev.yml`（已 gitignore），填入 MySQL、Redis、AI 供应商等配置。

### 前端（三个端）

| 端 | 目录 | 技术栈 | 配置文件 | dev 端口 | dev 访问地址 | dev 命令 |
|----|------|--------|----------|----------|--------------|----------|
| **管理端** | `forge-admin-ui/` | Vue 3 + Naive UI + Vite | `vite.config.js` + `.env.development` | `3000`（`VITE_HTTP_PORT`） | `http://localhost:3000/` | `pnpm dev` |
| **大屏设计器** | `forge-report-ui/` | Vue 3 + Vite（TS） | `vite.config.ts` | `3021`（硬编码） | `http://localhost:3021/forge-report` | `npm run dev` |
| **H5 移动端** | `forge-h5-ui/` | uni-app 3 + Vue 3 | `vite.config.js` + `.env.development` | `3001`（`VITE_HTTP_PORT`） | `http://localhost:3001/` | `pnpm dev:h5` |

**管理端 `forge-admin-ui`**：接口前缀 `/dev-api`。代理规则（`vite.config.js`）：
- `/dev-api/api/flow`、`/dev-api/api/workspace` → 流程服务 `8081`
- 其余 `/dev-api` → 主后台服务 `http://127.0.0.1:8580/`
- `/ws` → WebSocket 转发到主后台服务

**大屏设计器 `forge-report-ui`**：接口前缀 `/forge-report-api` → `http://localhost:8581`（`vite.config.ts`）；另有 `/llm` 直连阿里百炼 OpenAI 兼容接口的 SSE 代理（禁用上游 gzip 避免缓冲流式响应）。

**H5 `forge-h5-ui`**：接口前缀 `/dev-api`。代理规则（`vite.config.js` + `.env.development`）：
- `VITE_HTTP_PROXY_TARGET=http://127.0.0.1:8583/`（**app-server**，不是 8580）
- `VITE_FLOW_PROXY_TARGET=http://127.0.0.1:8081/`
- `/dev-api/api/flow`、`/dev-api/ai/business/flow` → flow-server（8081），其余 `/dev-api` → app-server（8583）

改各端网络行为前，先读对应项目的 `vite.config.*` 与 `.env*`。要求 `Node >= 20.19`、`pnpm >= 8`。构建：管理端 `pnpm build`（`NODE_OPTIONS=--max-old-space-size=40961`）、大屏 `npm run build`、H5 `pnpm build:h5`；测试仅管理端有（`pnpm test`，Vitest）。

### 数据库 / Flyway
Flyway 迁移只在 **`forge-admin-server` 启动时**执行（`forge-report-server` 单独启动不会执行）。扫描位置默认 `filesystem:./db/migration,filesystem:../db/migration,filesystem:forge-server/db/migration`；可用 `FORGE_FLYWAY_LOCATIONS` / `FORGE_FLYWAY_ENABLED` 覆盖。

- `forge-server/db/migration/` — 版本化迁移脚本 `V<版本>__<小写蛇形描述>.sql`（版本必须大于 `1.0.0`、单调递增；已执行脚本禁止修改 —— 修正时新增下一个版本脚本）。
- `forge-server/db/seed/required/R__*.sql` — 系统必需初始化数据；`forge-server/db/seed/demo/D__*.sql` — 演示数据（默认不导入）；`forge-server/db/seed/optional/O__*.sql` — 可选模块数据。
- SQL 必须幂等或有防重复保护（`CREATE TABLE IF NOT EXISTS`、`INSERT ... SELECT ... WHERE NOT EXISTS`、新增列/索引前查 `information_schema`）。业务内置数据 `tenant_id` 必须为 `1`（禁止 `0`）。`sys_resource` / `sys_role_resource` 等权限资源脚本必须做 `NOT EXISTS` 防重复。

## 架构

### 后端分层
`forge-server/` 是 Maven 多模块工程，分三层：

1. **`forge-framework/forge-starter-parent/`** — 23 个技术 Starter（纯技术能力，无业务逻辑）：`forge-starter-core`（统一响应 `RespInfo` 位于 `.../starter/core/domain/RespInfo.java`，全局异常 `GlobalExceptionHandler` 位于 `.../starter/core/exception/`，以及 `@ApiEncrypt`/`@ApiDecrypt`/`@OperationLog` 注解）、`forge-starter-web`（Undertow 嵌入式服务器）、`forge-starter-auth`（Sa-Token）、`forge-starter-orm`（MyBatis-Plus + 动态数据源）、`forge-starter-tenant`（租户拦截，追加 `WHERE tenant_id = ?`）、`forge-starter-datascope`（`DataScopeInterceptor` 位于 `.../datascope/handler/DataScopeInterceptor.java`，配置前缀 `forge.datascope`，`enabled` 默认 `true`）、`forge-starter-crypto`（API 加解密）、`forge-starter-excel`、`forge-starter-file`、`forge-starter-log`（操作日志）、`forge-starter-idempotent`（`@Idempotent`）、`forge-starter-cache`、`forge-starter-config`、`forge-starter-trans`、`forge-starter-social`、`forge-starter-websocket`、`forge-starter-message`、`forge-starter-job`、`forge-starter-api-config`、`forge-starter-openapi-security`、`forge-starter-outbound`、`forge-starter-collaboration`、`forge-starter-id`。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yaomindong1996/forge-admin](https://github.com/yaomindong1996/forge-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
