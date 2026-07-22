---
trigger: always_on
description: 欢迎来到 SpringBlade 开源前端版 Saber（master 分支）。本文档汇总了工程结构、Avue CRUD 约定、动态路由与权限、Axios 拦截链、Vuex 状态与多租户/国际化等常见约定，帮助 Claude Code 快速上手，产出与项目风格一致的代码。
---

# CLAUDE.md

欢迎来到 SpringBlade 开源前端版 Saber（master 分支）。本文档汇总了工程结构、Avue CRUD 约定、动态路由与权限、Axios 拦截链、Vuex 状态与多租户/国际化等常见约定，帮助 Claude Code 快速上手，产出与项目风格一致的代码。

作为 AI 助手参与本项目开发时，你必须：

- 动手前先看一看目标模块（`src/views/{module}/` + `src/api/{module}/`）的现有实现，模仿它的结构与风格通常是最快的路径
- 需要外部知识时，优先查阅 Vue 3、Element Plus、Avue、Vite、Vuex、vue-i18n 的官方文档，而不是凭记忆作答
- 需求不够清晰时，先复述已知信息、抛出关键澄清问题，再动手
- 面对跨模块改动（动态路由、权限按钮、状态持久化等），拆成可验证的小步骤逐步推进比一气呵成更稳妥

> 所有开发内容必须建立在深度思考过的基础之上，禁止机械生成与错误填充。
> 如果你已了解所有规范，请在用户第一次对话时说明："我已充分了解 SpringBlade 开源前端版开发规范。"

---

## 1. 项目概览

**Saber** 是 SpringBlade 开源微服务平台的官方前端工程，基于 Vue 3 生态构建的企业级后台管理系统，与后端 SpringBlade（Boot/Cloud）配套使用。

### 1.1 技术栈

| 技术               | 用途               |
| ------------------ | ------------------ |
| Vue 3              | 核心框架           |
| Element Plus       | UI 组件库          |
| @smallwei/avue     | 增强型 CRUD 组件库 |
| Vue Router         | 路由管理           |
| Vuex               | 状态管理           |
| Axios              | HTTP 客户端        |
| Vite               | 构建工具           |
| vue-i18n           | 国际化（中/英/日） |
| Sass               | CSS 预处理器       |
| crypto-js / sm-crypto | 加密（AES / SM2） |
| NProgress          | 顶部进度条         |
| js-cookie          | Token Cookie 存储  |

---

## 2. 项目架构

```
Saber/
├── src/
│   ├── main.js                  # 应用入口（全局组件注册、插件挂载）
│   ├── App.vue                  # 根组件
│   ├── permission.js            # 路由守卫（鉴权、标签页、锁屏）
│   ├── axios.js                 # HTTP 拦截器（Token 注入、错误统一处理、401 登出）
│   ├── error.js                 # 全局错误处理
│   ├── mockProdServer.js        # Mock 服务（生产构建可选）
│   ├── api/                     # API 接口层（按业务模块组织）
│   │   ├── system/              # 系统管理（菜单/字典/部门/角色/租户等）
│   │   ├── desk/                # 工作台（通知等）
│   │   ├── tool/                # 开发工具（代码生成/数据源）
│   │   ├── base/                # 基础数据（区域等）
│   │   ├── report/              # 报表
│   │   ├── user.js              # 认证与用户（登录/刷新/登出/验证码）
│   │   └── logs.js              # 日志上报
│   ├── views/                   # 业务页面（按模块目录组织）
│   │   ├── system/              # 用户/角色/菜单/字典/部门/租户/岗位/应用/参数/顶部菜单
│   │   ├── authority/           # 角色权限/数据权限/接口权限
│   │   ├── monitor/log/         # 通用日志/接口日志/错误日志
│   │   ├── desk/                # 通知公告
│   │   ├── tool/                # 代码生成/数据源
│   │   ├── base/                # 区域维护
│   │   ├── report/              # 报表列表
│   │   ├── user/                # 个人中心
│   │   ├── util/                # 工具 Demo（表格/表单/权限/store/tags 等）
│   │   └── wel/                 # 首页/控制台
│   ├── option/                  # Avue 表格/表单配置（可选抽离）
│   │   └── crud/                # 配合 mixins/crud.js 动态加载
│   ├── config/                  # 全局配置
│   │   ├── website.js           # 核心配置（认证、菜单、OAuth2、租户、授权地址）
│   │   ├── env.js               # 接口基础地址（来自 Vite 环境变量）
│   │   └── iconList.js          # 图标元数据
│   ├── store/                   # Vuex 状态管理
│   │   ├── index.js             # Store 入口
│   │   ├── getters.js           # 全局 getter
│   │   └── modules/             # user / common / tags / logs
│   ├── router/                  # 路由系统
│   │   ├── index.js             # Router 入口
│   │   ├── avue-router.js       # 动态路由核心（菜单→路由、iframe 转换、Token 透传）
│   │   ├── page/                # 页面级路由（登录、锁屏、错误页）
│   │   └── views/               # 视图路由（首页、控制台、util Demo）
│   ├── components/              # 全局公共组件（basic-container/basic-block/iframe 等）
│   ├── mixins/                  # 混入（crud.js 按路径自动装配 option 与 api）
│   ├── utils/                   # 工具函数（auth/crypto/sm2/validate/store/util/func）
│   ├── lang/                    # 国际化（zh / en / ja）
│   ├── styles/                  # 全局样式 & 多主题（theme/ 下多套配色）
│   ├── mac/                     # macOS 风格主题外壳（index/login/lock）
│   └── page/                    # 页面布局框架（主布局、登录、锁屏）
├── vite/                        # Vite 插件配置（auto-import / compression / setup-extend）
├── vite.config.mjs              # Vite 主配置（端口 2888、别名、代理）
├── .env.development             # 开发环境变量
├── .env.production              # 生产环境变量
├── .prettierrc.json             # Prettier 配置
└── package.json
```

### 2.1 路径别名

`@` → `./src`、`~` → `./`、`components` → `./src/components`、`styles` → `./src/styles`、`utils` → `./src/utils`

---

## 3. 核心机制

### 3.1 Avue 组件代码生成（Skill 调用）

当涉及 `avue-crud`、`avue-form`、`avue-tree` 等 Avue 组件的代码生成或配置时，**优先调用 `avue-design` Skill**。该 Skill 覆盖 Avue 全部组件类型。

**触发场景**：用户要求创建 CRUD 表格、Avue 表单、树组件、数据展示页面，或提到 avue、crud 表格、动态表单、JSON 配置表单等关键词。

### 3.2 Avue Option 配置

本工程中 Avue 配置有两种常见做法，按需选择：

- **内联 option**（主流）：直接写在 `.vue` 文件的 `data()` 里，`src/views/system/` 下大多如此，便于单文件阅读与独立维护
- **抽离到 `src/option/`**：用于配合 `mixins/crud.js` 的自动装配（按路径映射同名 `src/option/*.js` 与 `src/api/*.js`），适合重复性高、模板化的模块

两种做法可共存，同一模块选定一种后保持一致。

### 3.3 API 接口规范

- 所有 API 通过 `src/axios.js` 封装的 Axios 实例发起，按业务模块组织于 `src/api/`
- 命名约定：列表 `getList(current, size, params)`、详情 `getDetail(id)` / `getXxx(id)`、新增 `add(row)`、更新 `update(row)`、删除 `remove(ids)`、树形 `getXxxTree()`
- `add` 与 `update` 通常指向同一后端接口（如 `/blade-system/{module}/submit`），由后端根据 `id` 是否存在区分，这是 BladeX 的固定写法，保持一致即可
- 后端微服务前缀：`/blade-auth/`、`/blade-system/`、`/blade-desk/`、`/blade-develop/`、`/blade-log/`、`/blade-report/`
- `/user/` 开头的接口走前端 Mock 或聚合层，用于少量登录态辅助接口（如 `/user/getUserInfo`、`/user/send-logs`）

### 3.4 认证机制


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chillzhuang/Saber](https://github.com/chillzhuang/Saber) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
