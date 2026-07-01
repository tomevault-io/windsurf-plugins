---
trigger: always_on
description: - [2. 项目结构与命名](#2-项目结构与命名)
---

# AGENTS.md

## 索引

- [1. 技术栈](#1-技术栈)
- [2. 项目结构与命名](#2-项目结构与命名)
- [3. 核心禁忌](#3-核心禁忌)
- [4. 开发工作流](#4-开发工作流)
  - [4.1 开发环境设置（Lix）](#41-开发环境设置lix)
- [5. 构建与 CI 调试](#5-构建与-ci-调试)
- [附录 框架约定与扩展机制](#附录-框架约定与扩展机制)

## 1. 技术栈

### Rust (apps/server)

- **Edition 2024** - 使用 RPIT 生命周期捕获规则等新语法
- **Web**: Axum 0.8 + tower-http + utoipa-axum (Scalar OpenAPI)
- **ORM**: Sea-ORM (sqlx-postgres, sqlx-sqlite, with-chrono, with-rust_decimal)
- **Auth**: jsonwebtoken (HS256, access + refresh token) + bcrypt
- **Config**: config crate (YAML)
- **ID**: xid (XID format)
- **DB**: PostgreSQL (sea-orm-migration)
- **Error**: 自定义 `#[error]` proc-macro (framework-macros)
- **Testing**: cucumber (BDD) + testcontainers
- **Other**: chrono, strum, utoipa-scalar
- **开发环境**: Nix (Lix) + flake + direnv（可复现开发环境）

### TypeScript

**管理后台 (apps/server-ui):**

- React 19 + Mantine v9 + @mantine/core + @mantine/hooks
- TanStack Router (file-based routing, auto code-splitting)
- TanStack Query (React Query)
- Axios + `postJson<T>()` / `getJson<T>()` / `putJson<T>()` / `deleteJson<T>()`
- i18next + react-i18next (命名空间翻译)
- UnoCSS + CSS Modules (.module.css)
- zod (runtime validation)
- 环境变量: `import.meta.env.VITE_*`
- Prettier: `{ singleQuote: true }`

**浏览器扩展 (apps/extension):**

- WXT framework (Chrome MV3)
- React 19 + Ant Design v5 + @ant-design/icons
- react-router 7 (页面路由)
- zustand (状态管理)
- fetch (API 调用)
- @electric-sql/pglite (嵌入式 WASM PostgreSQL)
- @mlc-ai/web-llm (浏览器内 LLM 推理)
- UnoCSS
- 测试: Vitest + Playwright BDD (playwright-bdd)

## 2. 项目结构与命名

### 目录结构

```
├── docs/                      mdBook 项目文档
├── flake.nix                  Nix flake 配置
├── flake.lock                 Nix flake lock
├── .envrc                     direnv 自动激活（use flake）
├── rust-toolchain.toml        Rust 版本（由 flake 自动读取）
├── .node-version              Node.js 版本（由 flake 自动读取）
├── packages/                  workspace 占位
├── libs/
│   └── analysis/              Lit Web Components 分析组件库
├── apps/
│   ├── server/                Rust 后端
│   │   ├── src/               应用入口 (main.rs, clap, logging, runtime 等)
│   │   ├── api/src/           API 路由层 (每个模块一个 .rs 文件)
│   │   ├── service/src/       业务逻辑层
│   │   ├── entity/src/        Sea-ORM Entity (自动生成 + 手动补充)
│   │   ├── migration/src/     数据库迁移
│   │   ├── web/src/           Web 层 (静态文件服务)
│   │   ├── framework/         框架层 (error, auth, config, data, middleware 等)
│   │   ├── macros/            proc-macro crate
│   │   └── build-metadata/    构建元数据 crate
│   ├── server-ui/src/         React 管理后台
│   ├── server-ui-e2e/         管理后台 E2E 测试 (Playwright)
│   │   ├── components/        UI 组件
│   │   ├── hooks/             自定义 Hooks
│   │   ├── routes/            TanStack Router 路由文件
│   │   ├── api/               API 调用层
│   │   ├── config/            配置
│   │   ├── data/              数据类型定义
│   │   ├── i18n/              国际化
│   │   ├── store/             状态管理
│   │   ├── widget/            微件组件
│   │   ├── assets/            静态资源
│   │   └── utils/             工具函数
│   └── extension/             浏览器扩展
│       ├── entrypoints/       入口 (background, content, offscreen, admin 等)
│       ├── common/            共享代码 (api, data/domain/bo/dto, extension)
│       └── lib/               第三方库
```

新建文件请严格遵循对应的目录位置。

### 命名风格

| 语言/层         | 命名风格                  | 示例                                        |
| --------------- | ------------------------- | ------------------------------------------- |
| Rust 变量/函数  | snake_case                | `create_routes`, `get_job_by_id`            |
| Rust 类型/Trait | PascalCase                | `ApiResult<T>`, `Principal`                 |
| Rust 错误码枚举 | `*ErrorCode` (PascalCase) | `UserErrorCode`, `JobErrorCode`             |
| Rust 模块名     | snake_case                | `job.rs`, `user.rs`                         |
| TS 变量/函数    | camelCase                 | `loadJobs`, `handleSubmit`                  |
| TS 组件/类      | PascalCase                | `RouteComponent`, `JobFormData`             |
| TS 组件文件     | PascalCase + .tsx         | `ExcelPreview.tsx`, `LocationMap.tsx`        |
| TS 非组件文件   | camelCase + .ts           | `taskDataPlan.ts`, `http.ts`                 |
| DB 表/字段      | snake_case                | `company_tag`, `first_scan_datetime`        |
| URI             | snake_case                | `/api/job/search`、`/api/auth/access_token` |
| 扩展 API 方法   | className + methodName    | `dataSourceMetadataSearch`                  |

## 3. 核心禁忌

### 绝对不能做的

- **不要**手动编辑 `routeTree.gen.ts` (TanStack Router 自动生成和覆盖)
- **不要**修改 Moon workspace 结构 (`.moon/workspace.yml`, `.moon/toolchains.yml`) 及各项目 `moon.yml` 配置
- **不要**引入新依赖前未检查现有依赖是否已满足需求
- **不要**使用非 Edition 2024 的 Rust 语法（如 `'_` 生命周期 elision 规则、`impl<T>` 旧式 trait bound 等）
- **不要**手动编辑 `flake.lock`（使用 `nix flake update` 更新）

### 必须遵守的

- **提交信息**: 必须使用 Conventional Commits 格式（`feat:` / `fix:` / `perf:` / `remove:` / `deprecate:` / `security:`）。Lefthook commit-msg hook 自动校验格式，不满足会被拒绝。如需跳过用 `git commit --no-verify`。如需在 changelog 中展示详细说明，在 footer 中写入 `CHANGELOG: <description>`。破坏性变更使用 `feat!:` 或 `BREAKING CHANGE:` footer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lastsunday/job-hunting](https://github.com/lastsunday/job-hunting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
