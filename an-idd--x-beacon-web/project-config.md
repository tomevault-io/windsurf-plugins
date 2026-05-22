---
trigger: always_on
description: > 项目启动后请把本文件复制到 `web/CLAUDE.md`，这是该子项目的工作指南。
---

# CLAUDE.md（WebUI 子项目）

> 项目启动后请把本文件复制到 `web/CLAUDE.md`，这是该子项目的工作指南。
>
> 配套需求文档：[webui-requirements.md](webui-requirements.md)。开工前必须先读完那份。

---

## 项目定位

**X-BEACON WebUI** —— X-BEACON 网关的运维管理界面。这是 X-BEACON 主项目的子工程，部署独立，但与主网关（Go）紧耦合（共用 auth / scope 体系）。

**对应主项目**：`/Users/kk/AI_Project/X-Beacon/`（CLAUDE.md 在仓库根目录），主语言 Go，本子工程语言 TypeScript。

---

## 技术栈

### 核心选型（已在需求文档定调，不再讨论）

- **构建**：Vite 5+
- **框架**：Vue 3 + TypeScript（strict 模式）
- **API 风格**：**`<script setup>` + Composition API 唯一**，禁止 Options API 混用
- **路由**：Vue Router 4
- **数据**：`@tanstack/vue-query` v5
- **状态**：Pinia（仅跨页 UI 状态；服务端状态全归 Vue Query）
- **UI**：**Arco Design Vue**（图表见下：v0.1 不绑定 `@arco-design/charts`，hand-rolled SVG 也接受）
- **工具**：`@vueuse/core`（按需）
- **HTTP**：原生 `fetch` + `src/api/client.ts` 薄封装
- **校验**：Arco Form 内置规则；跨字段 / API type 共享时用 zod
- **测试**：Vitest + Vue Test Utils

### 禁用清单（引入前必须讨论）

- ❌ Nuxt / SSR / 任何 server-side rendering 框架
- ❌ Tailwind CSS（与 Arco 样式系统冲突）
- ❌ Element Plus / Naive UI / Ant Design Vue（**UI kit 只能有一套**）
- ❌ Vuex（用 Pinia）
- ❌ axios / ky / superagent
- ❌ moment / dayjs（`Intl.DateTimeFormat` + 几行 utility）
- ❌ lodash 全量（按需复制单函数即可）
- ❌ Options API（一律 `<script setup>`）
- ❌ Storybook（v0.1 用例不多，不值这个工程量）
- ❌ 手撸全局事件总线 / mitt（用 Pinia store 或 props/emit）

---

## 目录结构

```
web/
├── src/
│   ├── main.ts                # 入口：挂 Pinia + Router + VueQuery + Arco
│   ├── App.vue                # 顶层 layout（Arco <a-layout>）
│   ├── api/
│   │   ├── client.ts          # fetch 封装：注入 Authorization、统一错误
│   │   ├── keys.ts            # /admin/keys 系列
│   │   ├── logs.ts            # /admin/logs
│   │   ├── pricing.ts         # /admin/pricing
│   │   └── stats.ts           # /admin/stats/*
│   ├── pages/
│   │   ├── Login.vue
│   │   ├── Dashboard.vue
│   │   ├── Keys.vue
│   │   ├── Logs.vue
│   │   └── Pricing.vue        # v0.2
│   ├── components/
│   │   ├── Layout/
│   │   │   ├── AppShell.vue   # 侧边栏 + 顶栏
│   │   │   └── NavMenu.vue
│   │   ├── Charts/
│   │   │   └── QPSChart.vue
│   │   ├── Keys/
│   │   │   ├── KeysTable.vue
│   │   │   └── CreateKeyModal.vue
│   │   └── ...
│   ├── composables/
│   │   ├── useAuth.ts         # localStorage key 读写 + 401 redirect
│   │   └── usePaginated.ts
│   ├── stores/
│   │   └── auth.ts            # 当前 key + scopes（仅 UI 状态）
│   ├── lib/
│   │   ├── format.ts          # 时间 / 货币 / 字节
│   │   └── errors.ts          # API error 解析
│   └── router/
│       └── index.ts           # 路由表 + 守卫
├── public/
├── index.html
├── vite.config.ts
├── tsconfig.json
├── package.json
└── CLAUDE.md
```

### 目录职责约定

- `pages/` 只放路由级页面，不放业务组件
- `components/` 内按业务模块二级分类（`Keys/` / `Logs/`），不要按"原子-分子-organism"分（admin 场景过度设计）
- `api/` 一份请求一个文件，函数名对应 endpoint（`listKeys` / `revokeKey` / ...）
- `composables/` 放可被两个以上页面复用的 composable，单页面专用 composable 内联在页面 `<script setup>` 中
- `stores/` 只放 UI 跨页状态（auth / theme 等），**不**放接口数据缓存
- 任何路径 ≥ 3 层嵌套都是过度组织，警告

---

## 编码规范

### TypeScript

- `tsconfig.json` 开 `strict: true`，**不**用 `any`（实在跑不通用 `unknown`）
- API 响应类型在 `src/api/<file>.ts` 顶部用 `interface` 显式定义，**不**从后端反向生成
- 组件 props 用 `defineProps<{ ... }>()` 泛型形式，不用 runtime declaration
- 事件用 `defineEmits<{ (e: 'submit', payload: Foo): void }>()` 类型化

### Vue

- **统一 `<script setup>` + Composition API**，禁止 Options API
- 单文件组件（SFC）顺序：`<script setup lang="ts">` → `<template>` → `<style scoped>`
- 一个文件一个组件（少于 30 行的纯展示组件可内联）
- 服务端数据**永远**走 `useQuery` / `useMutation`，不要 `onMounted + ref + fetch`
- 表单**永远**走 `<a-form>` + `:model` + `:rules`，不要散落的受控 input
- v-for 不要用 array index 当 key（除非列表确实是不可变顺序）
- 模板里少写复杂表达式（> 1 个三元 / > 2 个属性访问），抽到 `computed`

### Arco 使用约定

- 按需引入：用 `unplugin-vue-components` + `@arco-design/web-vue/resolver`，避免 import 噪音
- 表格优先用 `<a-table :data :columns>` + `:pagination` + `:loading`，三件套搞定 90% 列表场景
- 弹层确认用 `<a-popconfirm>`（删除 / revoke），不要自己写 dialog
- 复杂 Modal 用 `<a-modal v-model:visible>`，不要在 setup 里 `Modal.open()` 命令式调用（命令式难以 v-if 控制 + 难测）
- 主题：**保持默认**。颜色 / 间距走 Arco token（`--color-primary` / `--color-text-1` 等），不在 `<style>` 里硬编码 hex
- 国际化：v0.1 中文为主，`#{{ $t() }}` 不引入；prop 文案直接中文写死

### 样式

- 默认 `<style scoped>`，不写全局样式（除非真的需要，放 `src/styles/global.css`）
- 颜色 / 间距用 Arco CSS 变量，**不**硬编码
- 不用 BEM / SMACSS 等命名规范（Arco token 已经够强约束）
- dark mode：v0.1 跟随系统（Arco 自带 `body[arco-theme=dark]`），不做切换器

### 错误处理

- API client 统一识别 401 / 403 / 4xx / 5xx，抛 `ApiError { status, code, message }`
- 401 → 清 store + 跳 `/login`
- 403 → `Message.warning('权限不足')`，不跳走
- 5xx → `Message.error(...)` + `console.error`，**不**展示 stack trace
- Vue Query 的 `onError` 不在每个 useQuery 都写一遍，用 QueryClient 的 default

### 安全

- API key **绝不**进 URL（query / path），只放 Authorization header
- localStorage 里只存 key，不存 user 信息
- **禁止**在 UI 里渲染 prompt / message 内容（即便后端返回了——但后端按需求文档不会返回）
- `v-html` 慎用，任何动态内容用 `{{ }}` 或 `<a-typography-text>`

---

## 与后端的契约

### Auth

- 所有 `/admin/*` 请求带 `Authorization: Bearer <key>`
- key 在 localStorage `xb-webui-key` 下，由 `useAuth` composable 读写
- 401 = key 无效 / 撤销 / 缺 scope；403 通常是 scope 缺失（admin:webui vs admin:pricing）

### Endpoint 契约源头

[webui-requirements.md §5](webui-requirements.md) 是 single source of truth。前端 type 定义和那份文档对齐；如果实际返回值有出入，**先和后端对齐再改前端 type**，不要本地 patch。

### 错误响应格式

后端走 OpenAI 风格 envelope：

```json
{ "error": { "message": "...", "type": "...", "code": "..." } }
```

`api/client.ts` 解析这个 shape 抛 `ApiError`，UI 不直接看 raw HTTP body。

---

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [An-idd/X-Beacon-Web](https://github.com/An-idd/X-Beacon-Web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
