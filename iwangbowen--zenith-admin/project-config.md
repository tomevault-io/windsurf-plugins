---
trigger: always_on
description: Zenith Admin 是一个基于 **Hono + React + Drizzle ORM** 的全栈后台管理系统，采用 npm monorepo 结构。
---

# Zenith Admin — AI 协作指南

Zenith Admin 是一个基于 **Hono + React + Drizzle ORM** 的全栈后台管理系统，采用 npm monorepo 结构。

---

## 项目结构

```text
packages/
├── server/   Hono HTTP 服务，Drizzle ORM，PostgreSQL，JWT 认证
├── web/      React 19 + Vite + Semi Design 前端
└── shared/   前后端共享的 TypeScript 类型 + Zod 验证 schema
```

---

## 常用命令

```bash
npm run dev            # 同时启动 server + web 开发服务器
npm run dev:server     # 仅启动后端（端口 3300）
npm run dev:web        # 仅启动前端（端口 5373）
npm run build          # 顺序构建：shared → server → web
npm run db:generate    # 生成 Drizzle 迁移文件
npm run db:migrate     # 执行数据库迁移
npm run db:seed        # 填充初始种子数据
```

---

## 架构约定

### 后端（`packages/server`）

- **框架**：Hono v4，通过 `@hono/node-server` 运行在 Node.js
- **路由**：所有路由挂载在 `/api` 前缀，文件位于 `packages/server/src/routes/`
- **认证**：Access Token（2h）+ Refresh Token（30d）双 token 机制；`packages/server/src/middleware/auth.ts` 中 `authMiddleware` 注入 `c.set('user', payload)`；签发/校验统一走 `packages/server/src/lib/jwt.ts` 的 `signToken` / `verifyToken`（基于 `hono/jwt`）
- **请求上下文**：全局挂载 `hono/context-storage`，辅助函数可用 `packages/server/src/lib/context.ts` 的 `currentUser()` / `getCtx()` 零参取值，无需再层层透传 `c` 或 `user`
- **路由定义模式**：所有路由文件统一使用 `defineOpenAPIRoute` + `router.openapiRoutes()` 模式（参考 `packages/server/src/routes/api-tokens.ts`）。不使用 `<AuthEnv>` 泛型，不添加全局 `router.use('*', authMiddleware)`；每个受保护路由在 `createRoute` 的 `middleware: [authMiddleware, guard(...)] as const` 中显式声明。收集所有路由常量后调用 `router.openapiRoutes([route1, route2, ...] as const)` 统一注册
- **Service 层**：业务逻辑、数据映射、前置校验从路由中提取到 `packages/server/src/services/xxx.service.ts`（已覆盖所有路由）。命名约定：`mapXxx`（数据映射，纯函数）、`ensureXxx`（前置校验，抛 `HTTPException`）。**禁止**在 service 中调用 `c.json()`、直接访问 Hono `Context`、使用 `console.*`；需要当前登录用户时统一通过 `packages/server/src/lib/context.ts` 的 `currentUser()` 获取（依赖全局 `contextStorage()`）。路由 handler 只负责取参数、调 service、返回 HTTP 响应。DB 唯一约束异常（PG 错误码 `23505`）统一在 service 中通过 `packages/server/src/lib/db-errors.ts` 的 `rethrowPgUniqueViolation(err, msg)` 映射为 `HTTPException(400, { message })`。错误处理使用 Hono 原生 `HTTPException`（`hono/http-exception`），由 `packages/server/src/index.ts` 的全局 `onError` 统一转为标准 JSON 错误响应。
- **验证**：所有入参通过 `@hono/zod-openapi` 的 `createRoute` 声明 Zod schema 后自动校验，路由内用 `c.req.valid()` 取已验证数据；`defaultHook: validationHook` 自动将校验失败转为 `{ code: 400, message: '...', data: null }`
- **响应辅助函数**：`packages/server/src/lib/openapi-schemas.ts` 提供三个路由 `responses:` 块快捷函数（必须使用展开语法）：`...ok(DTO, desc)`（单对象 200 响应）、`...okPaginated(DTO, desc)`（分页 200 响应）、`...okMsg(desc)`（仅消息的 200 响应）；**禁止**直接写 `200: { content: jsonContent(apiResponse(DTO)) }`。同文件还提供两个响应 **body** 构造函数：`okBody(data, msg?)` 构造成功响应体（`{ code: 0 as const, message, data }`），`errBody(msg, code?)` 构造错误响应体（`{ code, message, data: null }`）；路由 handler 中**必须**使用 `c.json(okBody(data), 200)` / `c.json(errBody(msg, 404), 404)` 模式，**禁止**内联写 `{ code: 0 as const, message, data }` 字面量对象。另外提供 `IdParam`（数值 id path 参数）、`PaginationQuery`（分页 query）、`BatchIdsBody`（批量操作 body）等公共 schema
- **DTO 中心化**：所有响应实体 DTO 按业务域拆分至 `packages/server/src/lib/dtos/`（`iam.ts` / `auth.ts` / `dict.ts` / `files.ts` / `logs.ts` / `notices.ts` / `system.ts` / `workflow.ts` / `dashboard.ts` / `region.ts` / `messages.ts`），通过 `packages/server/src/lib/openapi-dtos.ts`（re-export barrel）对外统一暴露。各路由文件通过 `import { XxxDTO } from '../lib/openapi-dtos'` 导入，**新增实体请直接在对应子文件中维护**。**禁止在路由文件内本地声明带 `.openapi('EntityName')` 的实体 DTO**，避免 Swagger Components 重复/冲突
- **统一响应**：`{ code: 0, message: 'success', data: T }`，失败时 `code` 为非零值；使用 `okBody(data, msg?)` / `errBody(msg, code?)` 构造响应体，禁止内联写字面量对象
- **数据库**：Drizzle ORM + PostgreSQL，schema 定义在 `packages/server/src/db/schema.ts`，迁移文件在 `packages/server/drizzle/`，统一数据库类型别名位于 `packages/server/src/db/types.ts`；计数查询统一用 `db.$count(table, where)`；分页列表的 `total` 与 `list` 必须用 `Promise.all` **并行执行**；**SQL-builder 分页**统一使用 `withPagination(query.$dynamic(), page, pageSize)`（来自 `packages/server/src/lib/where-helpers.ts`）；**RQB 分页**统一使用 `offset: pageOffset(page, pageSize)`（来自 `packages/server/src/lib/pagination.ts`）；禁止手写 `(page - 1) * pageSize`；关联数据查询优先使用 Drizzle RQB（`db.query.tableName.findMany/findFirst({ with: { relation: true } })`），尤其是通过联结表读取角色/岗位/菜单权限等场景，避免先查主表再手工二次聚合；`schema.ts` 已声明所有 `xxxRelations`，`db` 实例已传入 `schema`，可直接使用；若 helper 需要同时接受 `db` 与 `tx`，统一从 `packages/server/src/db/types.ts` 导入 `DbExecutor` / `DbTransaction`，禁止使用 `Parameters<Parameters<typeof db.transaction>[0]>[0]` 之类的手工推导
- **枚举同步**：数据库 pg enum、TypeScript union type、Zod enum **三者必须保持一致**

### 前端（`packages/web`）

- **UI 库**：Semi Design v2（`@douyinfe/semi-ui`）— 使用 Semi Design 组件时先查阅 `.claude/skills/semi-ui-skills/`
- **图标**：统一使用 `lucide-react`，禁止引入 `@douyinfe/semi-icons`
- **路由**：`react-router-dom` v7，页面组件位于 `packages/web/src/pages/`
- **认证状态**：`useAuth` hook，token 存储在 `localStorage`，key 为 `zenith_token`（来自 `@zenith/shared` constants）
- **HTTP 请求**：封装在 `src/utils/request.ts`，自动附加 Bearer token 和处理 401 跳转
- **环境变量**：`VITE_API_BASE_URL`（API 地址）、`VITE_APP_TITLE`（应用名）

### 共享层（`packages/shared`）

- 直接引用 `.ts` 源文件，**无需编译步骤**
- `types.ts`：所有实体类型（`User`, `Menu`, `Role`, `Dict` 等）及 `ApiResponse<T>`, `PaginatedResponse<T>`
- `validation.ts`：Zod schema，前后端共用，**禁止在 server/web 中重复定义**
- `constants.ts`：枚举常量（角色、状态、存储提供方等）

### 分页规范

所有列表接口返回 `PaginatedResponse<T>`：`{ list, total, page, pageSize }`

---

## 文件存储


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iwangbowen/zenith-admin](https://github.com/iwangbowen/zenith-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
