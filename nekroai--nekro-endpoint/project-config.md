---
trigger: always_on
description: 你好，AI 助手！这份文档是你的核心工作指南。在你进行任何编码、重构或分析工作之前，**必须**完整阅读并理解本文档。你的首要任务是确保你的所有操作都与本文档中描述的架构和原则保持一致，并在你对项目架构做出任何修改后，**主动更新本文档本身 [global.mdc](mdc:.cursor/rules/global.mdc) **。
---


# 工作指导手册

你好，AI 助手！这份文档是你的核心工作指南。在你进行任何编码、重构或分析工作之前，**必须**完整阅读并理解本文档。你的首要任务是确保你的所有操作都与本文档中描述的架构和原则保持一致，并在你对项目架构做出任何修改后，**主动更新本文档本身 [global.mdc](mdc:.cursor/rules/global.mdc) **。

## 1. 项目核心定位

这是一个基于 **Cloudflare** 技术栈的、**生产级**、**类型安全**的 **Hono + React + D1** 全栈应用模板。

- **核心价值**: 提供开箱即用的开发体验，整合最佳实践，实现从数据库到前端的端到端类型安全。
- **目标用户**: 希望在 Cloudflare 生态中快速构建现代化 Web 应用的开发者。
- **你的角色**: 维护并增强这个模板的工程化能力、易用性和健壮性。始终以"最佳实践"和"最低维护成本"为原则进行开发。

**当前项目状态**（NekroEndpoint 实现）：

- **Phase 1（MVP）**：100% 完成 ✅
- **Phase 2（权限组系统）**：100% 完成 ✅
- **Phase 3.1（动态代理端点）**：100% 完成 ✅
- **Phase 3.2（脚本端点）**：计划中 ⏳
- **可投入生产使用**：适用于静态内容托管、固定代理和动态子路径代理场景

## 2. 核心技术与架构

### 2.1. 整体架构

这是一个**混合渲染模式 (Hybrid Rendering)** 的单体应用，部署在 Cloudflare Pages & Workers 上。

- **开发环境 (`pnpm dev`)**:
  - Hono 后端 (Wrangler) 运行在 `localhost:8787`，作为**主入口**。
  - Vite 前端服务器运行在 `localhost:5173`。
  - **关键认知**: `index.ts` 中的开发逻辑会将所有非 API 的前端请求**代理**到 Vite 服务器 (`5173`)。这使得前端能享受 Vite 带来的**热更新 (HMR)**。**你绝不能假设开发时后端可以直接访问 `frontend/dist` 下的任何文件**。
- **生产环境 (`pnpm deploy`)**:
  - Vite 将前端代码构建为静态资源，输出到 `frontend/dist` 目录。
  - Hono 后端 (`src/index.ts`) 会根据 `manifest.json` **服务器端渲染 (SSR)** 初始 HTML，并由 Cloudflare Pages 提供静态资源。
  - **关键认知**: `index.ts` 中通过**动态 `import()`** 来加载生产构建产物 (`manifest.json` 和 `index.html`)，这是为了避免在开发环境中因找不到这些文件而导致构建失败。**这是本项目的核心架构设计，必须理解并维护**。

### 2.2. 后端 (`src/`)

- **入口: `src/index.ts`**:
  - 这是应用的统一入口。
  - 使用 `if (c.env.NODE_ENV === "development")` 来分离开发和生产逻辑。
  - **OpenAPI 注册**: API 路由通过一个独立的 `OpenAPIHono` 实例 (`apiApp`) 进行注册，然后统一挂载到主 `app` 上。这是为了确保 Swagger UI 能正确发现所有端点。在添加新的 API 模块时，必须遵循这个模式。
- **路由: `src/routes/`**:
  - Hono 的路由模块。包含功能示例和认证路由。
  - 使用 `@hono/zod-openapi` 的 `createRoute` 来创建类型安全且自动生成文档的路由。
  - **认证路由** (`auth.ts`): 实现 GitHub OAuth 登录、用户信息获取、登出和 API Key 重新生成功能。
- **中间件: `src/middleware/`**:
  - **认证中间件** (`auth.ts`): 验证用户会话，注入用户信息到 context，保护需要认证的路由。
- **数据校验: `src/validators/` 和 `common/validators/`**:
  - 使用 Zod 定义所有 API 的请求/响应/路径参数的 Schema。这些 Schema 是类型安全和 API 文档的来源。
  - **认证 Schema** (`common/validators/auth.schema.ts`): 定义认证相关的数据结构。
- **数据库 ORM: `src/db/`**:
  - `schema.ts` 文件使用 Drizzle ORM 定义数据库表结构。这是**唯一的数据源 (Single Source of Truth)**。
  - 包含 `features`（功能开关）、`users`（用户）、`userSessions`（会话）表。
- **工具函数: `src/utils/`**:
  - **加密工具** (`encryption.ts`): 生成用户专属的 API Key。
  - **HTML 模板**: 统一的 HTML 模板生成器，避免重复代码。
- **集中化配置管理**:
  - **SEO 配置**: `src/config/seo.ts` - 所有 SEO 相关配置的单一数据源
  - **自动化脚本**: `scripts/generateHtml.ts` - 自动生成开发环境的 HTML 模板

### 2.3. 前端 (`frontend/`)

#### 路由与入口

- **统一路由配置**:
  - **核心改进**: `frontend/src/routes.tsx` - 唯一的路由定义文件，被客户端和服务端入口共享使用。
  - **开发友好**: 添加新页面时，只需要在此文件中修改一次，避免了在多个入口文件中重复定义。
- **入口文件**:
  - `entry-client.tsx`: **客户端入口**。负责在浏览器中"激活"(hydrate) 由服务器渲染的 HTML。使用统一的路由配置。
  - `entry-server.tsx`: **服务器端渲染入口**。负责在后端生成初始的 HTML 字符串。使用统一的路由配置。
  - **关键优化**: 两个入口文件都使用 `AppRoutes` 组件，确保路由定义的一致性。

#### 页面结构（完整实现）

- **pages/HomePage.tsx**: 首页，展示项目介绍
- **pages/AuthCallbackPage.tsx**: GitHub OAuth 回调处理页面
- **pages/DashboardPage.tsx**: 用户仪表盘
  - 用户信息展示
  - API Key 查看/复制/重新生成
  - 账户统计信息
- **pages/EndpointsPage.tsx**: 端点管理页面（核心功能）
  - 左侧：路径树视图（TreeView）
  - 右侧：端点编辑器（Monaco Editor）
  - 功能：创建、编辑、删除、发布、移动端点
  - 支持：静态端点、代理端点配置
- **pages/PermissionGroupsPage.tsx**: 权限组管理页面
  - 权限组列表
  - 访问密钥管理（生成、编辑、撤销、删除）
  - 密钥显示/隐藏切换
  - 快捷到期时间设置
- **pages/InitPage.tsx**: 系统初始化页面
  - 检查系统状态
  - 选择首个管理员
  - 自动激活用户
- **pages/admin/AdminUsersPage.tsx**: 管理员用户管理页面
  - 用户列表（分页、搜索、筛选）
  - 激活/停用用户
  - 删除用户
- **pages/DocsPage.tsx**: 文档页面
- **pages/Features.tsx**: 功能开关页面（模板遗留）

#### 主题系统

- **核心**:
  - `frontend/src/context/ThemeContextProvider.tsx` 提供了一个全局的 `AppThemeProvider` 和 `useAppTheme` hook。
  - `frontend/src/theme/` 目录是我们中心化的主题定义模块。
- **架构与规则**:
  - **自定义主题**: 我们通过对 Material-UI 主题进行模块扩展 (module augmentation) 来添加自定义、类型安全的主题属性。类型定义位于 `frontend/src/theme/types.ts`。
  - **中心化管理**: 所有与主题相关的样式（如特定页面的背景、自定义组件颜色等）都**必须**在 `frontend/src/theme/index.ts` 中的 `lightTheme` 和 `darkTheme` 对象里进行定义。
  - **组件内使用**: 组件**禁止**通过 `theme.palette.mode === 'dark'` 这样的条件判断来硬编码样式。**必须**直接从主题对象中获取预先定义好的自定义属性 (例如 `theme.pageBackground`)。
  - **状态切换**: 所有主题状态的读取和切换都**必须**通过 `useAppTheme` hook 进行。

#### 认证系统

- **useAuth Hook** (`hooks/useAuth.ts`): 统一的认证状态管理，提供 login/logout/refetch 方法。
- **Storage 工具** (`utils/storage.ts`): SSR 安全的 localStorage 封装，支持跨标签页状态同步。
- **认证流程**:
  1. 用户点击登录按钮
  2. 调用 `/api/auth/github` 获取 OAuth URL
  3. 跳转到 GitHub 授权页面
  4. GitHub 回调到 `/auth/callback`
  5. `AuthCallbackPage` 处理回调，获取 sessionToken
  6. 存储 sessionToken 到 localStorage
  7. 跳转到 Dashboard

#### 状态管理

- **React Query**: 用于服务器状态管理
  - `hooks/useEndpoints.ts`: 端点数据管理
  - `hooks/usePermissionGroups.ts`: 权限组数据管理
  - `hooks/useAccessKeys.ts`: 访问密钥数据管理
  - `hooks/useAuth.ts`: 认证状态管理
- **React Context**: 用于全局客户端状态
  - `ThemeContext`: 主题状态管理

#### 核心组件

- **components/WorkspaceTopBar.tsx**: 工作区顶栏组件
  - macOS 风格紧凑设计（40px 高度）
  - Logo + 导航按钮 + 更多菜单 + 用户头像
  - 主题切换按钮
- **components/endpoints/EndpointEditor.tsx**: 端点编辑器组件
  - Monaco Editor 集成
  - 静态端点内容编辑
  - 代理端点配置表单
  - 动态代理端点配置表单（Phase 3.1）
- **App.tsx**: 标准网页布局
  - 导航栏
  - 用户头像菜单
  - 登录按钮
  - 页脚
  - 主题切换按钮
  - 用于首页、文档、控制台等内容页面

#### 布局系统

项目现有**两套布局系统**，根据页面类型选择：

**1. 标准网页布局（App.tsx）**

- 用途：首页、文档等内容展示页面

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NekroAI/nekro-endpoint](https://github.com/NekroAI/nekro-endpoint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
