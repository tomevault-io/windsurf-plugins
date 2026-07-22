---
trigger: always_on
description: > 本文档为前端工程架构指导手册，采用多层渐进式结构，帮助开发者快速理解项目结构、定位功能模块。
---

# Web Frontend Architecture Guide

> 本文档为前端工程架构指导手册，采用多层渐进式结构，帮助开发者快速理解项目结构、定位功能模块。

## 📑 Quick Navigation

- [工程概述](#工程概述)
- [技术栈](#技术栈)
- [目录结构](#目录结构)
- [功能模块详解](#功能模块详解)
- [核心配置](#核心配置)
- [开发指南](#开发指南)

---

## 工程概述

**项目名称**: derisk-web  
**项目类型**: Next.js 15 应用 (静态导出)  
**构建方式**: 静态导出 (output: 'export')  
**UI框架**: Ant Design 5 + Tailwind CSS  
**状态管理**: React Context + Hooks  
**国际化**: i18next  

---

## 技术栈

### 核心框架
- **Next.js**: 15.4.2 (App Router)
- **React**: 18.2.0
- **TypeScript**: 5.x

### UI & 样式
- **Ant Design**: 5.26.6 (主UI组件库)
- **Tailwind CSS**: 4.1.18 (样式工具)
- **@ant-design/icons**: 6.0.0
- **@ant-design/x**: 1.5.0 (AI对话组件)

### 数据可视化
- **@antv/g6**: 5.0.49 (图可视化)
- **@antv/gpt-vis**: 0.5.2 (GPT可视化组件)
- **@antv/ava**: 3.4.1 (自动图表)

### 状态与通信
- **Axios**: 1.10.0 (HTTP请求)
- **@microsoft/fetch-event-source**: 2.0.1 (SSE流式请求)
- **ahooks**: 3.9.0 (React Hooks工具集)

### 其他
- **reactflow**: 11.11.4 (流程图编辑器)
- **CodeMirror**: 6.x (代码编辑器)
- **markdown-it**: 14.1.0 (Markdown渲染)

---

## 目录结构

### 一级目录结构

```
web/
├── public/              # 静态资源
├── src/                 # 源代码
│   ├── app/             # Next.js App Router 页面
│   ├── client/          # API客户端层
│   ├── components/      # 可复用组件
│   ├── contexts/        # React Context 状态管理
│   ├── hooks/           # 自定义Hooks
│   ├── locales/         # 国际化资源
│   ├── services/        # 业务服务层
│   ├── styles/          # 全局样式
│   ├── types/           # TypeScript类型定义
│   └── utils/           # 工具函数
├── next.config.mjs      # Next.js配置
├── tsconfig.json        # TypeScript配置
├── tailwind.config.js   # Tailwind配置
└── package.json         # 项目依赖
```

### 详细目录树

<details>
<summary><b>📁 src/app - 页面路由结构</b></summary>

```
src/app/
├── layout.tsx              # 根布局 (全局Provider)
├── page.tsx                # 首页 (Chat主页)
├── not-found.tsx           # 404页面
├── i18n.ts                 # 国际化初始化
│
├── application/            # 【应用管理模块】
│   ├── page.tsx           # 应用列表
│   ├── layout.tsx         # 应用布局
│   ├── app/               # 应用详情页
│   │   ├── page.tsx
│   │   └── components/    # 应用详情组件
│   │       ├── chat-content.tsx
│   │       ├── agent-header.tsx
│   │       ├── tab-agents.tsx
│   │       ├── tab-knowledge.tsx
│   │       ├── tab-prompts.tsx
│   │       ├── tab-skills.tsx
│   │       └── tab-tools.tsx
│   └── explore/           # 应用探索页
│
├── chat/                   # 【独立对话页面】
│   └── page.tsx
│
├── knowledge/              # 【知识库管理】
│   ├── page.tsx           # 知识库列表
│   └── chunk/             # 知识块管理
│
├── prompt/                 # 【提示词管理】
│   ├── page.tsx
│   ├── layout.tsx
│   └── [type]/            # 动态路由 - 提示词详情
│
├── agent-skills/           # 【Agent技能管理】
│   ├── page.tsx           # 技能列表
│   └── detail/            # 技能详情
│
├── v2-agent/               # 【V2 Agent页面】
│   └── page.tsx
│
├── mcp/                    # 【MCP工具管理】
│   ├── page.tsx
│   ├── detail/
│   ├── CreatMcpModel.tsx
│   └── CustomUpload.tsx
│
├── models/                 # 【模型管理】
│   └── page.tsx
│
├── channel/                # 【渠道管理】
│   ├── page.tsx           # 渠道列表
│   ├── create/            # 创建渠道
│   ├── [id]/              # 编辑渠道 (动态路由)
│   └── components/        # 渠道组件
│
├── cron/                   # 【定时任务】
│   ├── page.tsx           # 任务列表
│   ├── create/            # 创建任务
│   ├── edit/              # 编辑任务
│   └── components/
│
├── settings/               # 【系统设置】
│   └── config/
│
└── vis-merge-test/         # 【可视化测试页】
    └── page.tsx
```
</details>

<details>
<summary><b>📁 src/components - 组件库</b></summary>

```
src/components/
├── layout/                 # 布局组件
│   ├── side-bar.tsx       # 侧边栏导航
│   ├── float-helper.tsx   # 浮动帮助
│   ├── user-bar.tsx       # 用户栏
│   └── menlist.tsx        # 菜单列表
│
├── chat/                   # 【对话组件模块】 ⭐核心模块
│   ├── content/           # 对话内容区
│   │   └── home-chat.tsx  # 首页对话容器
│   ├── input/             # 输入组件
│   ├── auto-chart/        # 自动图表生成
│   │   ├── advisor/       # 图表推荐算法
│   │   ├── charts/        # 图表类型实现
│   │   └── helpers/       # 辅助工具
│   └── chat-content-components/  # 对话内容子组件
│       ├── VisComponents/ # 可视化组件集合
│       │   ├── VisStepCard/
│       │   ├── VisMsgCard/
│       │   ├── VisLLM/
│       │   ├── VisCodeIde/
│       │   ├── VisMonitor/
│       │   ├── VisRunningWindow/
│       │   └── ... (20+组件)
│       └── ...
│
├── vis-merge/              # 可视化合并组件
├── blurred-card/           # 模糊卡片
└── agent-version-selector/ # Agent版本选择器
```
</details>

<details>
<summary><b>📁 src/client/api - API客户端</b></summary>

```
src/client/api/
├── index.ts                # API基础封装 (GET/POST/PUT/DELETE)
├── request.ts              # 请求工具
│
├── app/                    # 应用相关API
├── chat/                   # 对话相关API
│   └── index.ts           # 推荐问题、反馈、停止对话
├── flow/                   # 流程编排API
├── knowledge/              # 知识库API
├── prompt/                 # 提示词API
├── tools/                  # 工具API
│   ├── index.ts
│   ├── v2.ts
│   └── interceptors.ts
├── skill/                  # 技能API
├── cron/                   # 定时任务API
├── channel/                # 渠道API
├── evaluate/               # 评估API
└── v2/                     # V2版本API
```
</details>

<details>
<summary><b>📁 src/types - TypeScript类型定义</b></summary>

```
src/types/
├── global.d.ts            # 全局类型声明
├── app.ts                 # 应用类型 (IApp, AgentParams等)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [derisk-ai/OpenDerisk](https://github.com/derisk-ai/OpenDerisk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
