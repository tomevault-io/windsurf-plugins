---
trigger: always_on
description: > 本文件为 AI 编程助手提供项目导航与开发约定，帮助 Agent 在本仓库中高效工作。
---

# AGENTS.md

> 本文件为 AI 编程助手提供项目导航与开发约定，帮助 Agent 在本仓库中高效工作。

## 项目概述

CF Manager 是一个 Cloudflare 多账户管理平台，提供 Workers / Pages / DNS / KV / D1 / R2 / AI 推理 / 浏览器渲染的统一管理界面，同时暴露 OpenAI 兼容 API 供外部调用。

项目采用**双后端架构**——同一套业务逻辑分别用 Express（Docker 自建部署）和 Hono（Cloudflare Pages 部署）实现，共享同一套前端。

## 仓库结构

```
cf-manager/
├── backend/          # Express 后端（Docker 部署版）
│   └── src/
│       ├── index.ts        # Express 入口，路由挂载
│       ├── config.ts       # 环境变量配置
│       ├── db.ts           # SQLite 数据库初始化
│       ├── routes/         # API 路由（accounts, dns, workers, ai, openai, storage, ...）
│       ├── services/       # 业务逻辑层（Cloudflare SDK 封装、加密、配额追踪、代理等）
│       ├── models/         # 数据模型（account, auditLog, quotaUsage, catalogSource）
│       ├── middleware/     # 认证、响应包装、错误处理、日志、请求ID
│       └── data/           # 运行时数据（model-pricing.json 等自动同步文件）
├── frontend/         # Vue 3 前端
│   └── src/
│       ├── api/            # Axios 封装的 API 调用
│       ├── views/          # 页面组件（Dashboard, Accounts, DNS, Workers, AI, Storage, ...）
│       ├── components/     # 可复用组件
│       ├── stores/         # Pinia 状态管理
│       ├── router/         # Vue Router 路由配置
│       └── utils/          # 工具函数
├── worker/           # Hono 后端（Cloudflare Pages 部署版）
│   ├── src/
│   │   ├── index.ts        # Hono 入口 + Pages Functions handler
│   │   ├── types.ts        # Env 接口（D1, KV, ASSETS 等 binding）
│   │   ├── routes/         # API 路由（与 backend 对称）
│   │   ├── services/       # 业务逻辑（与 backend 对称，用 fetch 替代 SDK）
│   │   ├── db/             # D1 数据模型 + schema.sql
│   │   ├── middleware/     # 认证、响应包装、错误处理
│   │   └── pages/          # 伪装 nginx 页面
│   ├── build.js            # 一键构建脚本（前端 + worker + ZIP）
│   └── wrangler.toml       # Wrangler 配置
├── shared/           # 前后端共享源（唯一真实来源）
│   ├── model-pricing.json  # AI 模型定价
│   ├── catalog.schema.json # Catalog JSON Schema
│   └── catalogValidator.ts # Catalog 校验器源码
├── scripts/          # 构建辅助脚本
│   ├── sync-shared.js      # 将 shared/ 同步到 backend 和 worker
│   ├── gen-version.js      # 从 CHANGELOG.md 生成 version.ts
│   └── gen-catalog-validator.js  # 预编译 AJV 校验器（兼容 Workers）
├── docker/           # Docker 构建配置（all-in-one 单容器）
├── docs/             # 文档
├── docker-compose.yml
├── deploy.sh         # Docker 一键部署脚本
└── CHANGELOG.md      # 更新日志（版本号来源）
```

## 技术栈

| 层级 | Docker 版 (backend/) | Worker 版 (worker/) |
|---|---|---|
| 框架 | Express 5 + TypeScript | Hono 4 + TypeScript |
| 数据库 | SQLite (better-sqlite3) | Cloudflare D1 |
| CF 交互 | `cloudflare` SDK (Node.js) | 原生 `fetch` 调用 CF REST API |
| 部署 | Docker Compose | Cloudflare Pages |
| 模块系统 | CommonJS | ESM (esbuild bundle) |
| TS target | ES2022 | ESNext |
| 前端 | Vue 3 + Naive UI + Pinia + Vite + TypeScript | 同左 |

## 关键架构约定

### 双后端对称性

`backend/src/` 和 `worker/src/` 的路由（routes/）、服务（services/）、中间件（middleware/）需要**保持功能对称**。新增功能时通常需要同时修改两端：

- `backend/src/routes/*.ts` ↔ `worker/src/routes/*.ts`
- `backend/src/services/*.ts` ↔ `worker/src/services/*.ts`（名称可能不同，如 `cfFactory.ts` ↔ `cfApi.ts`、`encryptionService.ts` ↔ `encryption.ts`）
- `backend/src/models/*.ts` ↔ `worker/src/db/models.ts`（worker 端集中在一个文件）

### 共享文件同步机制

`shared/` 是唯一真实来源。构建/开发前 `scripts/sync-shared.js` 会自动将其复制到 backend 和 worker。**不要直接编辑 `backend/src/data/model-pricing.json` 或 `worker/src/data/model-pricing.json`**，应编辑 `shared/model-pricing.json`。新增共享文件时在 `sync-shared.js` 的 `jobs` 数组中追加条目。

### 版本号管理

版本号从 `CHANGELOG.md` 的首个 `## [x.y.z]` 提取，由 `scripts/gen-version.js` 生成 `version.ts`。**不要手动编辑 `version.ts`**。发版时更新 `CHANGELOG.md`。

### API 响应格式

- **内部 API**（`/api/*`）：经过 `responseWrapper` 中间件，自动包装为 `{ success: true, data }` 或 `{ success: false, error }`
- **外部 API**（`/v1/*` 和 `/api/v1/*`）：OpenAI 兼容格式，**不经过** responseWrapper
- 前端 Axios 拦截器自动解包 `success`/`data`，错误时提取 `error.message`

### 认证

- 通过 `Authorization: Bearer <API_SECRET>` 头部认证
- API Token 加密存储（AES），密钥为 `ENCRYPTION_KEY` 环境变量
- Worker 端的 `Env` 接口定义在 `worker/src/types.ts`

### 安全特性

- 根路径伪装为 nginx 默认页，管理界面在 `/admin/`
- 演示账户受 `DEMO_ACCOUNT_IDS` 保护，不可删除/修改
- API Token 在返回时脱敏为 `***encrypted***`

## 开发命令

### 本地开发

```bash
# 后端（http://localhost:3001）
cd backend && npm install && ENCRYPTION_KEY="dev-key" npm run dev

# 前端（http://localhost:5173，自动代理 /api 到后端）
cd frontend && npm install && npm run dev

# Worker 本地（需先构建前端到 worker/public/）
cd worker && npm run dev
```

### 构建

```bash
# Worker 一键构建（前端 + worker + ZIP）
cd worker && npm run build

# 后端编译
cd backend && npm run build

# 前端构建
cd frontend && npm run build
```

### Docker 部署

```bash
cp .env.example .env  # 至少设置 ENCRYPTION_KEY
chmod +x deploy.sh && ./deploy.sh
```

## 代码风格约定

- **TypeScript strict 模式**：所有项目均启用 `strict: true`
- **命名**：文件使用 camelCase（如 `accountRouter.ts`），类型/接口使用 PascalCase（如 `AccountInput`），路由变量使用 camelCase
- **路由定义**：backend 使用 Express `Router`，worker 使用 `new Hono<{ Bindings: Env }>()`
- **错误处理**：路由中使用 `try/catch + next(err)`（backend）或直接 `return c.json({ error: { code, message } }, status)`（worker）
- **数据库查询**：backend 使用同步的 `better-sqlite3`，worker 使用异步的 `D1Database.prepare().all()`
- **注释**：代码中中文注释和英文注释混用，新增代码可使用中文注释
- **环境变量**：通过 `config.ts`（backend）或 `c.env`（worker）访问，不直接引用 `process.env`（backend 的 config.ts 除外）

## 反馈与交互约定

若环境中存在 MCP 反馈工具（如 `feedback-agent` 等），在以下场景中**优先使用 MCP 工具**向用户反馈或交互，而非直接结束对话：

- **任务完成时**：通过 MCP 工具通知用户任务已完成，等待确认
- **即将结束对话时**：通过 MCP 工具确认用户是否还有后续需求

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hefy2027/cf-manager](https://github.com/hefy2027/cf-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
