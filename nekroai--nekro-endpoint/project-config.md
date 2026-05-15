---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

**NekroEndpoint** 是一个基于 Cloudflare Workers 构建的**端点编排平台**，允许用户在全球边缘节点上创建和管理 API 端点。

### 核心功能

- **三种端点类型**：
  - **静态端点**（Static）：托管文本内容、配置文件
  - **代理端点**（Proxy）：转发请求到目标 URL（如加速 GitHub raw 内容）
  - **脚本端点**（Script）：运行自定义 JavaScript 脚本（Phase 3）

- **双层密钥机制**：
  - **Platform API Key**：用户管理平台功能的凭证（创建/编辑端点、管理权限组）
  - **Endpoint Access Key**：访问已发布端点的凭证（通过权限组管理，可对外分发）

- **权限组系统**：创建权限组 → 生成访问密钥 → 端点关联权限组 → 细粒度访问控制

- **树形端点管理**：层级化组织端点，支持拖拽排序，每个节点都是完整的端点

- **用户激活制**：管理员手动激活后才能发布端点（未激活用户可创建和编辑，但不能发布）

- **管理员审查**：查看所有用户的端点树和内容，强制下线任何端点

### 技术栈

- **后端**：Hono (OpenAPI)、Cloudflare Workers、Cloudflare D1 (SQLite)
- **前端**：React 18、Material-UI、Monaco Editor、React Router、Vite
- **数据库 ORM**：Drizzle ORM
- **类型安全**：Zod（验证）、TypeScript
- **认证**：GitHub OAuth + 会话管理
- **部署**：Cloudflare Pages & Workers

## 常用开发命令

### 开发环境

```bash
# 启动全栈开发服务器（后端 :8787 + 前端 :5173）
pnpm dev

# 仅启动后端（Wrangler）
pnpm dev:backend

# 仅启动前端（Vite）
pnpm dev:frontend

# 类型检查（前后端）
pnpm typecheck
```

### 数据库操作

```bash
# 修改 schema 后生成迁移文件
pnpm db:generate

# 应用迁移（本地开发）
pnpm db:migrate

# 应用迁移（生产环境）
pnpm db:migrate:prod

# 打开 Drizzle Studio（数据库可视化工具）
pnpm db:studio

# 检查 schema 一致性
pnpm db:check

# 数据库填充（seed）
pnpm db:seed         # 本地
pnpm db:seed:prod    # 生产
```

### 构建与部署

```bash
# 构建前端（客户端 + 服务端 SSR 包）
pnpm build

# 生产构建（包含数据库迁移）
pnpm build:prod

# 部署到 Cloudflare
pnpm deploy

# 本地测试生产构建
pnpm serve:prod
```

### 测试

```bash
# 运行测试
pnpm test

# 运行特定测试文件
pnpm test <test-file-name>
```

## 核心架构原理

### 1. 混合渲染模式（开发 vs 生产）

**这是本项目最关键的架构设计，必须深刻理解。**

#### 开发模式 (`pnpm dev`)

- Hono 后端运行在 `localhost:8787`（通过 Wrangler）
- Vite 前端运行在 `localhost:5173`
- **关键机制**：所有非 API 请求到 `:8787` 都会被**代理**到 `:5173`，实现 HMR（热模块替换）
- **禁止假设**：`frontend/dist` 目录在开发环境下**不存在**

```typescript
// src/index.ts:84-97
if (c.env.NODE_ENV === "development") {
  // 代理所有前端请求到 Vite 开发服务器
  const url = new URL(c.req.url);
  url.hostname = "localhost";
  url.port = "5173";
  return fetch(url.toString());
}
```

#### 生产模式 (`pnpm deploy`)

- Vite 将前端构建到：
  - `frontend/dist/client`（静态资源）
  - `dist/server`（SSR 渲染包）
- Hono 通过 `ASSETS` binding 服务静态文件
- **动态导入**：使用 `import()` 加载构建产物，避免开发环境找不到文件导致构建失败

```typescript
// src/index.ts:116-127
const manifestModule = await import("../dist/client/.vite/manifest.json");
const { render } = await import("../dist/server/entry-server.mjs");
```

**为什么使用动态 import？** 因为这些文件只在生产构建后才存在。如果使用静态 `import`，开发环境下会立即报错导致无法启动。

### 2. 端到端类型安全（单一数据源）

**所有类型定义只在 `common/` 目录下维护一次，前后端共享。**

#### 目录结构

```
common/
├── types/index.ts              # TypeScript 类型（从 Zod 推导）
├── validators/                 # Zod Schema（验证 + 类型生成）
│   ├── endpoint.schema.ts      # 端点 Schema
│   ├── permission.schema.ts    # 权限组 Schema
│   ├── auth.schema.ts          # 认证 Schema
│   └── admin.schema.ts         # 管理员 Schema
└── config/api.ts               # 共享 API 配置
```

#### 工作流

1. **定义 Schema**（`common/validators/`）：

```typescript
export const EndpointSchema = z.object({
  id: z.string(),
  path: z.string(),
  type: z.enum(["static", "proxy", "script"]),
  // ...
});
```

2. **推导类型**（`common/types/index.ts`）：

```typescript
export type Endpoint = z.infer<typeof EndpointSchema>;
```

3. **后端使用**（`src/routes/`）：

```typescript
import { EndpointSchema } from "../../common/validators/endpoint.schema";

const route = createRoute({
  responses: {
    200: {
      content: { "application/json": { schema: EndpointSchema } },
    },
  },
});
```

4. **前端使用**（`frontend/src/hooks/`）：

```typescript
import type { Endpoint } from "../../../common/types";

export function useEndpoints() {
  return useQuery({
    queryFn: async () => {
      const res = await fetch(`${getApiBase()}/endpoints`);
      return res.json() as Promise<ApiResponse<{ tree: Endpoint[] }>>;
    },
  });
}
```

### 3. 统一路由系统

**唯一路由定义文件**：`frontend/src/routes.tsx`

客户端入口（`entry-client.tsx`）和服务端入口（`entry-server.tsx`）都使用这个文件，避免重复定义。

#### 添加新页面的步骤

1. 创建页面组件：`frontend/src/pages/NewPage.tsx`
2. 在 `frontend/src/routes.tsx` 中导入并添加路由：

```typescript
import NewPage from './pages/NewPage';

<Route path="new-page" element={<NewPage />} />
```

3. **完成** —— 无需修改任何入口文件

### 4. 数据库 Schema 与迁移

**单一数据源**：`src/db/schema.ts`

#### 核心数据表

根据设计文档（`design.md`），项目包含以下数据表：

- **users**：用户表（GitHub ID、用户名、邮箱、角色、激活状态、Platform API Key）
- **userSessions**：会话表（token、过期时间）
- **endpoints**：端点表（支持树形结构，`parent_id` 字段）
  - `path`：相对路径（如 `/clash-config`）
  - `type`：`static` | `proxy` | `script`
  - `config`：JSON 配置（根据类型不同结构不同）
  - `access_control`：`public` | `authenticated`
  - `required_permission_groups`：JSON array of group IDs
  - `is_published`：是否已发布
  - `sort_order`：排序字段（支持手动拖拽排序）
- **permissionGroups**：权限组表
- **accessKeys**：访问密钥表（格式：`ep-<32位随机字符串>`）
- **envVars**：环境变量表（加密存储）
- **accessLogs**：访问日志表（统计端点访问情况）

#### 迁移工作流

```bash
# 1. 修改 src/db/schema.ts
# 2. 生成迁移文件
pnpm db:generate

# 3. 查看生成的 SQL（drizzle/ 目录）
# 4. 应用迁移
pnpm db:migrate         # 本地
pnpm db:migrate:prod    # 生产
```

### 5. 主题系统（Material-UI）

**集中化主题管理**：所有主题定义在 `frontend/src/theme/index.ts`

#### 规则

1. **定义自定义主题属性**：在 `theme/types.ts` 中扩展 MUI 主题类型
2. **集中配置**：所有主题相关样式在 `lightTheme` 和 `darkTheme` 中定义

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NekroAI/nekro-endpoint](https://github.com/NekroAI/nekro-endpoint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
