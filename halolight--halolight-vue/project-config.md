---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Halolight Vue 是一个基于 Vue 3 + Vite 的现代化中文后台管理系统，使用 TypeScript、Tailwind CSS 4、shadcn-vue 和 @vueuse/motion 构建。

- **在线预览**: https://halolight-vue.h7ml.cn
- **GitHub**: https://github.com/halolight/halolight-vue

## 技术栈速览

- **核心框架**: Vue 3.5 (Script Setup) + Vite 7 (Rolldown) + TypeScript 5.9
- **路由**: Vue Router 4
- **状态管理**: Pinia 2 + pinia-plugin-persistedstate v4
- **数据请求**: TanStack Vue Query v5 + Axios
- **样式**: Tailwind CSS 4、shadcn-vue (Radix Vue / Reka UI)、lucide-vue-next
- **动画/交互**: @vueuse/motion、grid-layout-plus (可拖拽仪表盘)
- **图表**: ECharts 6 (配合 vue-echarts)
- **Mock**: Mock.js
- **构建/规范**: pnpm 10、ESLint 9 + TypeScript
- **测试**: Vitest + Vue Test Utils + Testing Library

## 常用命令

```bash
pnpm dev          # 启动开发服务器 (http://localhost:5173)
pnpm build        # 生产构建
pnpm preview      # 预览构建产物
pnpm lint         # ESLint 检查
pnpm lint:fix     # ESLint 自动修复
pnpm type-check   # TypeScript 类型检查 (vue-tsc)
pnpm test         # 运行单元测试 (watch 模式)
pnpm test:run     # 运行单元测试 (单次)
pnpm test:ui      # 运行测试并打开 UI 界面
pnpm test:coverage # 生成覆盖率报告
```

## 架构

### 应用入口 (src/main.ts)

Vue 应用通过插件链式调用初始化：

```ts
app.use(createPinia())
   .use(router)
   .use(VueQueryPlugin)
   .mount('#app')
```

### 核心目录结构

```
src/
├── api/                    # API 服务定义 (Axios 封装)
│   ├── types.ts            # API 类型定义
│   ├── mock-data.ts        # Mock 数据生成
│   ├── analytics.ts        # 分析 API
│   ├── calendar.ts         # 日历 API
│   ├── files.ts            # 文件 API
│   ├── messages.ts         # 消息 API
│   ├── security.ts         # 安全 API
│   ├── settings.ts         # 设置 API
│   └── users.ts            # 用户 API
├── assets/                 # 静态资源
├── components/
│   ├── ui/                 # shadcn-vue 基础组件
│   │   ├── avatar/
│   │   ├── badge/
│   │   ├── button/
│   │   ├── card/
│   │   ├── checkbox/
│   │   ├── dropdown-menu/
│   │   ├── input/
│   │   ├── label/
│   │   ├── scroll-area/
│   │   ├── separator/
│   │   ├── sheet/
│   │   ├── switch/
│   │   ├── tabs/
│   │   ├── textarea/
│   │   └── tooltip/
│   ├── common/             # 通用业务组件
│   │   ├── AppHeader.vue   # 顶部导航栏
│   │   ├── AppSidebar.vue  # 侧边栏
│   │   ├── AppFooter.vue   # 页脚
│   │   └── AppTabs.vue     # 多页签
│   ├── auth/               # 认证相关组件
│   │   └── AuthShell.vue   # 认证页面外壳
│   ├── layout/             # 布局辅助组件
│   └── dashboard/          # 仪表盘专用组件
│       └── widgets/        # 仪表盘部件
│           ├── StatsWidget.vue
│           ├── ChartLineWidget.vue
│           ├── ChartBarWidget.vue
│           ├── ChartPieWidget.vue
│           ├── TasksWidget.vue
│           ├── UsersWidget.vue
│           ├── NotificationsWidget.vue
│           ├── CalendarWidget.vue
│           └── WidgetCard.vue
├── composables/            # 组合式函数 (Hooks)
│   ├── useDashboardData.ts # 仪表盘数据查询
│   ├── useChartTheme.ts    # 图表主题自适应
│   └── queries/            # TanStack Query 封装
├── config/                 # 全局配置
│   ├── index.ts            # 配置导出
│   ├── env.ts              # 环境变量
│   └── menu.ts             # 菜单配置
├── layouts/                # 布局组件
│   ├── AdminLayout.vue     # 管理后台布局
│   └── AuthLayout.vue      # 认证页布局
├── lib/                    # 工具库
│   └── utils.ts            # 通用工具函数 (cn, etc.)
├── mock/                   # Mock 数据定义
│   ├── index.ts            # Mock 入口
│   └── modules/            # 按模块拆分
│       ├── analytics.ts
│       ├── calendar.ts
│       ├── dashboard.ts
│       ├── files.ts
│       ├── messages.ts
│       ├── notifications.ts
│       ├── security.ts
│       ├── settings.ts
│       └── users.ts
├── plugins/                # 插件配置
│   ├── query-client.ts     # TanStack Query 配置
│   └── mock.ts             # Mock 插件
├── router/                 # 路由配置
│   └── index.ts            # 路由定义
├── stores/                 # Pinia Stores
│   ├── auth.ts             # 认证状态
│   ├── layout.ts           # 布局状态
│   ├── navigation.ts       # 导航状态
│   ├── tabs.ts             # 多页签状态
│   └── ui-settings.ts      # UI 设置 (主题/皮肤)
├── types/                  # TypeScript 类型定义
│   ├── dashboard.ts
│   ├── analytics.ts
│   ├── calendar.ts
│   ├── files.ts
│   ├── security.ts
│   ├── settings.ts
│   └── mock.d.ts
├── views/                  # 页面视图
│   ├── auth/               # 登录/注册/找回密码
│   │   ├── LoginView.vue
│   │   ├── RegisterView.vue
│   │   ├── ForgotPasswordView.vue
│   │   └── ResetPasswordView.vue
│   ├── dashboard/          # 仪表盘
│   │   ├── DashboardView.vue
│   │   ├── ProfileView.vue
│   │   └── NotificationsView.vue
│   ├── analytics/          # 数据分析
│   ├── calendar/           # 日历
│   ├── files/              # 文件管理
│   ├── messages/           # 消息
│   ├── security/           # 安全设置
│   ├── settings/           # 系统设置
│   ├── users/              # 用户管理
│   └── legal/              # 法律条款
│       ├── PrivacyView.vue
│       └── TermsView.vue
└── __tests__/              # 测试文件
    └── setup.ts
```

### 数据流模式

1. **API 请求**: `src/api/*.ts` 封装 Axios 请求 → `src/composables/` 使用 `useQuery`/`useMutation` 封装 → 组件 `<script setup>` 中调用
2. **状态管理**:
   - **全局状态** (用户信息、菜单折叠、主题色) 使用 **Pinia** (`src/stores/`)
   - **局部状态** 使用 `ref`/`reactive`
3. **Mock 数据**: 开发环境下通过 Mock.js 拦截 Axios 请求返回数据


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [halolight/halolight-vue](https://github.com/halolight/halolight-vue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
