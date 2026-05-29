---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Remote Control Server (RCS) 是一个基于 Elysia + Bun 的 AI Agent 控制面板后端（package name: `fenix`），配合 React 19 + Vite 前端，使用 PostgreSQL + Drizzle ORM 持久化。核心功能包括：

- **ACP 协议支持**：通过 WebSocket 与 acp-link Agent通信，实现远程 Agent 控制和事件流转发
- **配置管理**：Providers/Models/Agents/Skills/MCP 的动态配置，存储于 PostgreSQL（`src/services/config/` 子模块）
- **多租户**：better-auth organization 插件实现多组织隔离，所有配置和资源以 `organizationId` 为范围，通过 `AuthContext` 传递
- **会话管理**：会话事件推送，支持 ACP session/list 按 cwd 过滤
- **认证授权**：better-auth + `@better-auth/api-key` 插件，支持用户会话和 acp-link 的 Bearer token
- **定时 HTTP 任务**：cron 调度、执行历史记录、失败重试
- **知识库/工作流引擎/Meta Agent**：知识库管理、DAG 工作流引擎（`@fenix/workflow-engine`）、元智能体编排
- **IM 通道**（开发中）：多平台消息通道接入与路由绑定
- **S3 文件存储**（可选）、**Redis 缓存**（可选）、**Hermes 消息推送**（可选）
- **Workspace Packages**：`packages/` 下有 acp-link、core、plugin-sdk、plugin-opencode、sdk、workflow-engine 六个内部包

**依赖结构**：`web/` 没有独立的 `package.json`，所有前后端依赖统一在根 `package.json` 管理。前端代码在 `web/` 但依赖安装/升级都在根目录执行。

## 常用命令

```bash
bun run dev              # 后端开发（热重载）
bun run dev:web          # 前端开发（Vite dev server，独立进程）
bun run build:web        # 生产构建前端（修改前端代码后必须执行！）
bun run docs:dev         # 文档开发（VitePress）
bun run docs:build       # 构建文档
bun run precheck         # ⚠️ 提交前必须通过（格式化 + import 排序 + tsc + biome check）
bun run check:deps       # 依赖健康检查
bun run db:push          # 数据库 schema 同步（开发环境）
bun run db:generate --name <名称>  # 生成迁移文件（修改 schema.ts 后执行）
bun run db:migrate       # 应用迁移文件（生产环境）
```

### 测试

```bash
bun test src/__tests__/                       # 后端全部测试
bun test src/__tests__/store.test.ts          # 后端单个文件
bun test web/src/__tests__/                   # 前端全部测试
bun test web/src/__tests__/config-mcp-page.test.ts  # 前端单个文件
```

测试账号：`admin@test.com` / `admin123456`

### 关键注意事项

- **`bun run precheck` 是代码质量的第一标准**。流程：`biome format --write` → `biome check --write --linter-enabled=false`（import 排序）→ `tsc` → `biome check`。格式和 import 排序自动修复
- 后端挂载 `web/dist/` 提供前端静态文件，修改前端后**必须** `bun run build:web`
- **前端开发代理**：`web/vite.config.ts` 配置了 `/web`、`/api`、`/acp` 代理到 `http://localhost:3000`，`dev:web` 时前端请求自动转发到后端
- **严禁手写 SQL 迁移**：所有 schema 变更通过 `src/db/schema.ts` → `drizzle-kit generate` → `push/migrate`
- **环境变量**：新增必须先在 `src/env.ts` 的 `envSchema` 中声明（Zod `zod/v4`）
- **代码质量工具**：Biome v2.4.15 统一 lint + format（`biome.json`），不使用 ESLint/Prettier
- **Swagger API 文档**：`/docs/swagger`，新增路由时添加 `.tags()` 分组
- **工作目录漂移**：Bash `cd web` 后相对路径会出错，使用绝对路径或每次回 cd

## 架构关键点

### 后端架构 (Elysia + Bun)

**入口**：`src/index.ts` — 挂载路由和插件，启动时执行：DB 初始化 → 重置所有 `agent_session.status` 为 idle → `validateEnv()` → `applyEnv()` → `getCoreRuntime()`（注册 opencode 插件 + local node）→ `startScheduler()` → 清理残留 acp-link 进程 → auto-start 带 `autoStart=true` 的 Environment 实例 → `app.listen()`。全局请求体限制 10MB，路径双斜杠自动 302 重定向。优雅关闭：Hermes → ACP 连接 → Relay 连接 → 所有 Instance → Scheduler → Cache → PG 连接

- `/v1/*`：旧版环境/会话 API（`src/routes/v1/`）
- `/v2/*`：**主流**，Worker/CodeSession 相关 API（`src/routes/v2/`）
- `/web/*`：控制面板业务 API（`src/routes/web/`，18 个子模块：auth/config/environments/instances/sessions/files/s3-files/user-file/skills/tasks/knowledge-bases/channels/organizations/control/meta-agent/workflow-defs/workflow-engine/workflow-sse）
- `/acp/*`：ACP WebSocket 端点（`src/routes/acp/`）
- `/mcp/*`：MCP 知识库查询（`src/routes/mcp/`）
- `/workflow-ui/*`：Workflow 可视化编辑器代理到 acpx-g 服务（`src/routes/web/workflow-proxy.ts`）
- `/hooks/*`：Webhook 触发路由（无认证）

**认证层**（`src/auth/`）：better-auth + organization + apiKey 插件。`src/plugins/auth.ts` 提供 `authGuardPlugin`（`sessionAuth` macro），`src/services/org-context.ts` 的 `loadOrgContext` 从请求解析 `AuthContext`（`organizationId`/`userId`/`role`）。认证优先级：better-auth session cookie → API Key（`rcs_xxx`）→ Environment Secret → 全局 `RCS_API_KEYS`。组织 ID 从 `x-active-org-id` header > `activeOrganizationId` query param > `active_org_id` cookie 提取，结果缓存 60 秒。测试通过 `setTestAuth()` + `setTestOrgContext()` 绕过

**验证层**（`src/schemas/`）：所有路由请求体通过 Zod v4 schema 校验。新增路由必须创建对应 schema 文件（如 `session.schema.ts`），通过 `index.ts` 统一导出。v1/v2 路由各有独立 schema 文件。

**配置服务**（`src/services/config/`）：6 张配置表 CRUD，多租户隔离（`AuthContext` 首参 + `organization_id` WHERE）。返回值约定：delete → boolean，get → 对象 | null，list → 数组

**传输层**（`src/transport/`）：三层架构

- **ACP WS Handler**（`acp-ws-handler.ts`）：管理 acp-link 的 WebSocket 连接注册（`connections` Map），NDJSON 格式通信，server 端 keep_alive + 客户端活跃超时检测（3 倍 keepalive 间隔）
- **Relay 子模块**（`relay/`）：前端到 Agent 的中继桥接，两种模式：
  - **Instance 模式**（优先）：通过 `CoreRuntimeFacade.connectInstanceRelay()` 获取 handle，消息直接转发；handle 未就绪时 buffer 消息，连接后 flush
  - **EventBus 模式**（fallback）：直连 acp-link WS 时，通过 EventBus subscribe 转发 inbound 消息
  - `RelayConnectionManager`（`connection-manager.ts`）管理所有前端 relay 连接，最后一个 relay 断开时关闭 core relay handle
  - `message-router.ts`：过滤 keep_alive/connect、发布 inbound 到 EventBus（供 SSE 订阅者）、flush outbound buffer
- **EventBus**（`event-bus.ts`）：pub/sub 事件总线，per-session/per-agent 隔离。每个 bus 缓存最近 5000 条事件（超限时裁剪到一半），支持 `getEventsSince(seqNum)` 做 SSE 断线重连。`event-service.ts` 是 service 层的统一入口封装

**插件层**（`src/plugins/`）：auth、cors、error-handler（`AppError` → HTTP 状态码）、logger（requestId）、rate-limit、static、require-team-scope（`requireOrgScope` 组织资源校验）

**核心服务模块**（`src/services/`）按业务域划分：


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HuangPuStar/FenixAgent](https://github.com/HuangPuStar/FenixAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
