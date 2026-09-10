---
trigger: always_on
description: 本文件为在本代码库中工作的 AI 编程智能体（DeepSeek Harness、Claude Code、Codex、Cursor 等）提供指引。
---

# AGENTS.md

本文件为在本代码库中工作的 AI 编程智能体（DeepSeek Harness、Claude Code、Codex、Cursor 等）提供指引。

## AI 贡献准则

- **不得以 AI 身份在 Issue 或 PR 上发表评论**。讨论区只属于人类。
- **先讨论再实现**：非平凡改动（如新功能、重构）开工前，先在 Issue 评论中与维护者就实现方向达成一致。
- **新增依赖须先行讨论**：引入任何第三方依赖前，必须在 Issue 中说明用途、体积与维护活跃度，与维护者达成一致后再动手；运行时依赖优先使用已有依赖覆盖，禁止为单一小功能引入重量级库。
- **版本只改 package.json**：所有依赖版本统一在根目录 `package.json` 中管理，禁止使用任何形式的补丁或 postinstall 脚本修改第三方包。
- **披露 AI 使用**：当提交中较大部分由 AI 生成时，请在 commit message 末尾追加 trailer，注明实际使用的智能体，例如：

  ```
  Assisted-by: DeepSeek Harness
  ```
- 贡献流程遵循下方[「PR 约定」](#pr-约定)章节。

## 项目概述

ContiNew Admin UI 是 ContiNew Admin 的前端部分，基于 Gi Demo 前端模板适配开发的高质量多租户中后台管理系统。后端对应项目为 [continew-admin](https://github.com/continew-org/continew-admin)（Spring Boot 3 + Sa-Token），本前端通过 REST API 与之交互。

**技术栈**：Vue 3.5 + Arco Design Vue 2.57 + TypeScript 5 + Vite 5 + Pinia

**当前版本**：4.2.0-SNAPSHOT | **主分支**：`dev` | **Node**：22 | **包管理**：pnpm 9（勿用 npm/yarn，勿混用 `package-lock.json` 与 `pnpm-lock.yaml`）

## 核心架构

```
config/          Vite 插件配置（config/plugins/）
public/          静态资源（原样拷贝到 dist 根目录）
src/
  apis/          API 模块：<module>/index.ts（聚合）+ 资源文件 + type.ts
  assets/        静态资源（icons/svg/ 等）
  components/    Gi* 系列业务组件（unplugin-vue-components 自动导入）
  config/        全局配置（setting.ts）
  constant/      常量（common.ts、file.ts）
  directives/    自定义指令（permission/：v-permission、v-role）
  hooks/         组合式函数（hooks/modules/，由 hooks/index.ts 导出）
  layout/        布局根组件 + components/ + hooks/（四套布局异步切换）
  mock/          Mock 文件（vite-plugin-mock 加载）
  router/        路由：route.ts、guard.ts、asyncModules.ts
  stores/        Pinia stores（stores/modules/）
  styles/        样式：var.scss、index.scss、arco-ui/
  types/         全局类型：api.d.ts、router.d.ts、auto-imports.d.ts
  utils/         工具：http.ts、auth.ts、has.ts 等
  views/         页面（文件路径即后端动态路由 component 字段值）
```

关键机制：

- **自动导入**：`unplugin-auto-import` 提供 Vue/Router API 自动导入（生成 `src/types/auto-imports.d.ts`）；`unplugin-vue-components` 将 `src/components` 下组件按需导入（生成 `src/types/components.d.ts`）。因此 `GiTable`、`GiForm`、`GiPageLayout`、`GiCell*` 等业务组件在模板中直接使用即可，**不要重复 import**（业务子组件如 `AddDrawer` 仍需显式导入）。
- **动态路由**：登录后由 `useRouteStore.generateRoutes()` 拉取后端菜单树，经 `formatAsyncRoutes` 转换后 `router.addRoute` 注入；`src/router/asyncModules.ts` 用 `import.meta.glob('@/views/**/*.vue')` 收集页面模块——新增页面须放在 `src/views/` 下，文件路径即后端返回的 `component` 字段值（如 `system/user/index`）。≥3 层的多级路由会被 `flatMultiLevelRoutes` 降级为二级以兼容 keep-alive。
- **状态管理**：Pinia store 位于 `src/stores/modules/`，经 `src/stores/index.ts` 统一导出，`pinia-plugin-persistedstate` 持久化（`app`/`route` 全量，`user` 仅 token/roles/permissions，`tenant` 持久化租户开关与 tenantId，`dict` 仅内存）。
- **HTTP 请求层**：`src/utils/http.ts` 统一导出 `{ get, post, put, patch, del, request, requestNative, download }`；约定后端返回 `{ code, data, msg, success, timestamp }`（`ApiRes<T>`），`success=false` 时按 msg 长度自动选择提示组件，`code === '401'` 弹 Modal 引导重新登录；分页类型 `PageRes<T>`、查询类型 `PageQuery` 定义于 `src/types/api.d.ts`。API 函数典型签名：`http.get<PageRes<T.UserResp[]>>(BASE_URL, query)`。
- **业务组件（Gi* 系列）**：`GiTable`（封装 Arco Table + 工具栏 + 分页，`#top` 放搜索表单、`#toolbar-left/right` 放按钮、`#<dataIndex>` 自定义列）、`GiForm`（columns 配置驱动，`ColumnItem[]` 描述字段）、`GiPageLayout`（`#left` 侧栏插槽）、`GiCell*`（单元格渲染）、`GiSvgIcon`（SVG 雪碧图，图标放 `src/assets/icons/svg/`）等。
- **Hooks**：`src/hooks/modules/` 共 12 个组合式函数，经 `src/hooks/index.ts` 导出；核心是 `useTable`（分页、加载、多选、查询/刷新/删除），接收一个 `(page) => Promise<ApiRes<PageRes<T[]>>>` 的 API 函数。
- **权限控制**：指令 `v-permission="['system:user:create']"`、`v-role="['admin']"`；编程式 `src/utils/has.ts`（`hasPerm`/`hasRole` 等）；超级管理员权限标识 `*:*:*`、超级角色 `role_admin`；权限标识格式 `模块:资源:操作`，与后端菜单表 `permission` 字段一致。路由级权限由后端按角色过滤后返回，前端不二次拦截。
- **认证与多租户**：Token 存 localStorage（`src/utils/auth.ts`）；登录流程为 `useUserStore.accountLogin` → `setToken` + `setTenantId` → 路由守卫触发 `getInfo` + `generateRoutes`；请求头 `Authorization: Bearer <token>` 与 `X-Tenant-Id` 在 http 拦截器全局注入；客户端 ID 来自 `VITE_CLIENT_ID`。
- **样式体系**：`src/styles/var.scss` 全局 SCSS 变量由 Vite `additionalData` 注入（`$color-theme` 等映射 Arco CSS 变量）；主题色经 `@arco-design/color` 动态生成 `--primary-1~10`；暗黑模式通过 `document.body` 设置 `arco-theme="dark"`；Arco 覆盖样式位于 `src/styles/arco-ui/`（less）。
- **布局系统**：`src/layout/index.vue` 依据 `appStore.layout` 异步切换四种布局：`LayoutMix`（混合，默认）、`LayoutTop`、`LayoutDefault`、`LayoutColumns`。
- **Vite 插件**：插件配置位于 `config/plugins/`（app-info、vue、auto-import、components、svg-icon、mock 等），根配置见 `vite.config.ts`。

## 构建与测试命令

```bash
# 启动开发服务器（端口 5173，Vite proxy 代理 VITE_API_PREFIX 到 VITE_API_BASE_URL）
pnpm dev

# 生产构建（类型检查 + 构建到 dist/，drop console/debugger，静态资源按类型分目录）
pnpm build

# 以 test 模式构建（使用 .env.test，启用 Mock 打包）
pnpm build:test

# 仅类型检查（vue-tsc --noEmit），不产出文件
pnpm typecheck

# ESLint 检查 / 自动修复
pnpm lint
pnpm lint:fix

# 预览构建产物（端口 5050）
pnpm preview

# 使用 npmmirror 源安装依赖（首次拉取项目用）
pnpm bootstrap
```

本项目暂无单元测试框架，代码改动的验证方式是完整执行下方三道门禁。

### 提交前门禁（必须通过）

提交代码前，AI 智能体**必须**让门禁通过：

1. `pnpm lint`——ESLint 代码规范检查；
2. `pnpm typecheck`——vue-tsc 全量类型检查；
3. `pnpm build`——生产构建（内含一次类型检查，验证可构建性）。

被 lint 拦截时执行 `pnpm lint:fix` 自动修复，再重跑 `pnpm lint` 确认。三道门禁全部通过后才能提交。

开发环境通过 Vite proxy 代理后端接口（`VITE_API_PREFIX` → `VITE_API_BASE_URL`），Mock 默认启用（`vite-plugin-mock`，`src/mock/`）；生产构建是否打包 Mock 由 `VITE_BUILD_MOCK` 决定。

## 代码风格

遵循 **[@antfu/eslint-config](https://github.com/antfu/eslint-config)**，ESLint 即代码格式的唯一事实源（**不使用 Prettier**，请勿在项目中引入）。

### AI 智能体关键规则


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [continew-org/continew-admin-ui](https://github.com/continew-org/continew-admin-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
