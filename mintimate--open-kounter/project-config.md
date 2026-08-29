---
trigger: always_on
description: > 本文件是 `open-kounter` 仓库唯一代码规范入口。目标：可执行、可验证、低歧义。
---

# AGENTS.md — Open Kounter Agent Operating Guide

> 本文件是 `open-kounter` 仓库唯一代码规范入口。目标：可执行、可验证、低歧义。
> 适用对象：在本仓库工作的 AI 代理与人类协作者。结合本仓库实际技术栈（Vue 3 + `<script setup>` + JavaScript + Tailwind v3 + EdgeOne Pages Functions + Blob）裁剪。

---

## 1) 指令优先级

冲突时：用户明确需求 → 安全稳定性 → 本规范 → 现有代码风格
无法消解时：不泄露敏感信息（Token / Passkey / OIDC Secret）、不引入破坏性变更（数据迁移、Blob Schema 变更）、不修改需求范围外逻辑。

---

## 2) 工作流

- **先理解再改动**：定位组件、数据流、Blob Key 与边界，不基于猜测修改
- **小步提交**：最小可行改动，每步可解释“为什么改、改了什么、如何验证”
- **变更后验证**：`npm run build` 必须通过、无新增警告
- **同步更新文档**：修改路由 / API 命名约定 / Blob Schema / 环境变量 / UI 公共类 / 规范本身时，必须同步更新本文件与 [README.md](README.md)
- **登录态在 `App.vue` 中通过 `isLoggedIn` + `<router-view>` props 下发**，不要绕过 `App.vue` 自行从 `localStorage` 读取 token 来判断登录态

---

## 3) 项目架构

### 技术栈

- **运行平台**：[EdgeOne Pages](https://pages.edgeone.ai/zh/document/product-introduction)（Cloud Functions + Edge Functions + Blob）
- **前端**
  - **Vue 3.3+**，统一使用 Composition API `<script setup>`，**禁止 Options API**
  - **JavaScript**（本仓库不使用 TypeScript；需要类型提示用 JSDoc）
  - **Vue Router 4**（仅 `Home` + `NotFound`，登录态由 `App.vue` 控制）
  - **Vue I18n 11**（简体中文 / 英文；默认 `zh-CN`，语言资源集中在 `src/locales/`）
  - **Tailwind v3**（保留 `tailwind.config.js`；同时使用 `@theme {}` 块在 `style.css` 中声明设计 Token）
  - **Vite 7**
  - **原生 `fetch`**（不引入 axios，统一调用 `/api/*` 与 `/legacy-api/*`）
- **后端**
  - **EdgeOne Pages Cloud Functions**（`cloud-functions/api/**.js`）：处理认证、计数器、Passkey、OIDC、初始化与迁移
  - **EdgeOne Pages Edge Functions**（`edge-functions/legacy-api/**.js`）：仅用于读取旧版 `OPEN_KOUNTER` KV 命名空间，导出供 Blob 导入
  - **Blob Store**：通过 `@edgeone/pages-blob` 创建，默认 store 名 `open-kounter`，可经 `OPEN_KOUNTER_BLOB_STORE` 覆盖
- **状态管理**：本仓库**不引入 Pinia / Vuex**。组件间状态优先使用 props/emit；登录态保存在 `App.vue` 的 `ref` 中并下发；持久化仅写 `localStorage` 的 `open_kounter_token`、非敏感主题偏好 `open_kounter_theme` 与语言偏好 `open_kounter_locale`

### 目录分层

```tree
.
├── client/
│   └── adapter.js              # 客户端适配器（兼容 LeanCloud AV.Counter API）
├── cloud-functions/            # 主后端逻辑（运行于 EdgeOne Cloud Functions）
│   └── api/
│       ├── _api.js             # 通用响应、CORS、鉴权工具
│       ├── _blobStore.js       # Blob Store 工厂 + Key 命名规则 + 读写工具
│       ├── _legacyMigration.js # 旧 KV → Blob 迁移逻辑
│       ├── auth.js             # Token / OIDC session 校验
│       ├── counter.js          # 计数器核心（inc / batch_inc / set / delete / list / summary / export / import / set_config）
│       ├── init.js             # 首次初始化与迁移触发
│       ├── passkey.js          # Passkey 注册 / 登录 / 管理
│       └── oidc/
│           ├── login.js        # OIDC 登录发起（state / nonce / code_verifier 写入 Blob）
│           ├── callback.js     # OIDC 回调换码 + ID Token 校验 + 绑定/登录
│           └── status.js       # OIDC 配置与绑定状态查询
├── edge-functions/             # 兼容旧版 KV 的迁移出口（运行于 EdgeOne Edge Functions）
│   └── legacy-api/
│       └── migrate.js          # 导出旧 KV 数据供 Blob 导入
├── src/                        # 前端管理后台（Vue 3 + Vite）
│   ├── components/
│   │   ├── common/
│   │   │   ├── ConfirmModal.vue        # 通用确认弹窗
│   │   │   ├── LanguageSwitcher.vue     # 简体中文 / 英文切换
│   │   │   └── ThemeSwitcher.vue       # 亮色 / 系统 / 暗色三段式切换
│   │   ├── dashboard/
│   │   │   ├── AnalyticsOverview.vue   # 累计指标、热门页面与最近活跃
│   │   │   ├── CounterList.vue         # 计数器列表
│   │   │   ├── DataBackup.vue          # 数据备份与恢复（含旧 KV 迁移入口）
│   │   │   ├── DomainConfig.vue        # 域名白名单配置
│   │   │   ├── OidcManager.vue         # OIDC 绑定管理
│   │   │   ├── PasskeyManager.vue      # Passkey 管理
│   │   │   └── SingleCounterManager.vue# 单个计数器管理
│   │   ├── Dashboard.vue       # 仪表盘主组件
│   │   ├── Login.vue           # 登录组件（Token / Passkey / OIDC 渐进式）
│   │   └── NotFound.vue        # 404
│   ├── views/
│   │   └── Home.vue            # 首页（根据 isLoggedIn 切换 Login / Dashboard）
│   ├── router/index.js         # 路由表
│   ├── App.vue                 # 根组件（登录态 + OIDC 回调处理 + 全局 layout）
│   ├── i18n.js                 # Vue I18n 初始化 + 语言偏好读取 / 应用 / 持久化
│   ├── locales/
│   │   ├── en-US.js            # 英文资源
│   │   └── zh-CN.js            # 简体中文资源（默认）
│   ├── main.js                 # 入口
│   ├── style.css               # ⭐ 唯一全局 CSS（@theme + 主题映射 + 基础样式）
│   └── theme.js                # 主题偏好读取、解析、应用与持久化
├── other/                      # 文档资源（演示图等）
├── edgeone.json                # EdgeOne 配置（构建命令 / 输出目录 / 函数路由）
├── index.html
├── package.json
├── tailwind.config.js          # Tailwind v3 配置（颜色 token 与 @theme 保持同步）
├── vite.config.js
├── README.md
└── AGENTS.md                   # 本文件
```

### 分层依赖方向

`cloud-functions/api/_blobStore.js  ←  cloud-functions/api/_api.js  ←  cloud-functions/api/{auth,counter,init,passkey,oidc/*}.js`
`src/components/dashboard/*  ←  src/components/Dashboard.vue  ←  src/views/Home.vue  ←  src/App.vue`
前端**不直接 import 后端代码**；后端不持有任何前端引用。

### 禁止

- 在 `cloud-functions/api/{auth,counter,init,passkey,oidc/*}.js` 中**直接 new BlobStore**；必须走 `_blobStore.js` 暴露的工厂与读写工具
- 在 `cloud-functions/api/**` 中**手写 CORS 头与 401/200 JSON 响应**；统一走 `_api.js` 的 `successResponse` / `failResponse` / `optionsResponse`
- 在前端组件中直接读 / 写 `localStorage.open_kounter_token` 之外的认证字段；登录态以 `App.vue` 为单一信源
- 引入 Pinia / Vuex / axios / Element Plus / 任何 UI 组件库（保持零业务依赖膨胀）
- 重新引入旧版 KV 写入路径；所有写入只走 Blob

### 内聚与耦合

- 同业务域聚合：`cloud-functions/api/counter.js` ↔ `src/components/dashboard/CounterList.vue` / `SingleCounterManager.vue`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mintimate/open-kounter](https://github.com/Mintimate/open-kounter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
