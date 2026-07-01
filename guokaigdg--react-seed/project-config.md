---
trigger: always_on
description: > 本文档面向 AI 编码助手（Claude / Cursor / Comate / Copilot 等），用于在最短时间内建立对本仓库的准确认知，并在生成 / 修改代码时遵守一致的工程约定。人类读者请优先阅读 [`README.md`](./README.md) 与 [`docs/README.en.md`](./docs/README.en.md)。
---

# AGENTS.md

> 本文档面向 AI 编码助手（Claude / Cursor / Comate / Copilot 等），用于在最短时间内建立对本仓库的准确认知，并在生成 / 修改代码时遵守一致的工程约定。人类读者请优先阅读 [`README.md`](./README.md) 与 [`docs/README.en.md`](./docs/README.en.md)。

---

## 目录

1. [项目概览](#1-项目概览)
2. [技术栈](#2-技术栈)
3. [目录结构](#3-目录结构)
4. [路径别名](#4-路径别名)
5. [脚本与环境变量](#5-脚本与环境变量)
6. [运行与构建](#6-运行与构建)
7. [开发约定（必读）](#7-开发约定必读)
8. [代码规范与提交规范](#8-代码规范与提交规范)
9. [AI 修改代码须知](#9-ai-修改代码须知)
10. [常见任务剧本（Cookbook）](#10-常见任务剧本cookbook)
11. [外部参考](#11-外部参考)

---

## 1. 项目概览

`react-seed` 是一套**企业级 React SPA 前端模板**，目标是为新业务直接复用其工程化基建（Vite / TS / 路由 / 状态 / 请求 / 规范）。

- 类型：单页应用（SPA），构建产物为静态文件，可通过 `gh-pages` 一键发布
- 入口：`src/index.tsx` → `src/App.tsx` → `src/router/index.tsx`
- 在线 Demo：<https://guokaigdg.github.io/react-seed/>
- 姊妹分支：[移动端版](https://github.com/guokaigdg/react-template-mobile)

> 本仓库**不是**业务项目；新增内容时优先思考是否对模板"通用"。

## 2. 技术栈

| 分类 | 选型 | 版本 / 备注 |
| --- | --- | --- |
| 框架 | React + ReactDOM | **19.x**（启用 `react-jsx` 自动 JSX 运行时，无需 `import React`） |
| 语言 | TypeScript | **5.x**，`strict: true`，并启用 `noUnusedLocals` / `noUnusedParameters` |
| 构建 | **Vite 8** + `@vitejs/plugin-react` | 配置位于 `vite.config.ts`（根目录，Vite 8 默认 Rolldown 打包） |
| 开发服务 | `vite` 内置 dev server | 端口 3000；`server.host: true` |
| 路由 | `react-router` | **v7**，从 `react-router` 导入（**不是** `react-router-dom`） |
| 状态管理 | MobX + `mobx-react-lite` | mobx 6 / lite 4，`enforceActions: 'always'` |
| 网络请求 | axios + `axios-retry` | 全局封装见 `src/api/request.ts` |
| 样式 | Less + CSS Modules | Vite 内置，`*.module.less` 自动开启 CSS Modules |
| 图标 | `@phosphor-icons/react` + 本地 svg as React 组件（`vite-plugin-svgr`） | svg 资源：`src/assets/icons/svg/` |
| 代码质量 | ESLint 9（flat config）+ Prettier + Stylelint + husky + lint-staged + commitlint | 配置：`eslint.config.mjs` |
| 环境变量 | Vite `.env.[mode]` | 三套环境：`development` / `qa` / `production`，通过 `import.meta.env` 访问 |
| Node | ≥ 22.22.1；npm ≥ 7；volta 固定 22.22.3 | 见 `package.json#engines` / `volta` |

## 3. 目录结构

```
react-seed/
├── vite.config.ts              # Vite 配置（alias / plugins / build / server）
├── index.html                  # Vite 入口 HTML（根目录，引用 /src/index.tsx）
├── public/
│   └── favicon.ico             # 静态资源（不参与构建处理）
├── src/
│   ├── index.tsx               # ReactDOM.createRoot 挂载
│   ├── App.tsx                 # 根组件，BrowserRouter + useRoutes
│   ├── vite-env.d.ts           # Vite 类型与 import.meta.env 声明
│   ├── router/
│   │   ├── index.tsx           # 集中式路由表（React.lazy + SuspenseLazy）
│   │   └── README.md
│   ├── api/
│   │   ├── request.ts          # axios 实例 + 拦截器 + 重试封装
│   │   ├── home-two/           # 按页面拆分接口目录
│   │   │   ├── index.ts
│   │   │   └── types/home-two.ts
│   │   ├── home-order/
│   │   └── README.md
│   ├── store/                  # MobX
│   │   ├── index.ts            # configure + stores 聚合 + Context + useStores
│   │   ├── global/index.ts
│   │   └── about/index.ts
│   ├── components/             # 通用组件（每个组件一个目录 + barrel 导出）
│   │   ├── Button/             # 含 buttonHelpers.tsx
│   │   ├── Card/
│   │   ├── SuspenseLazy/       # 懒加载高阶
│   │   └── index.ts            # 统一 export
│   ├── view/                   # 页面级组件
│   │   ├── Home/               # 含 7 个子页面（One/Two/Three/Four/Mobx/Icon/Order）
│   │   ├── Dashboard/
│   │   ├── About/              # 演示 index.module.less（CSS Modules）
│   │   ├── Tab/                # 顶部导航
│   │   └── NotFound/
│   ├── constants/              # 常量与枚举（enum.ts）
│   ├── interface/              # 业务 TS 类型说明（含 index.md）
│   ├── types/                  # 全局 .d.ts：api / file / style
│   ├── utils/
│   │   ├── useHook/useRequest.ts   # 自研 hook：请求 + loading + 错误
│   │   ├── validate.ts
│   │   └── variable.ts
│   ├── assets/                 # 图片、svg（icons/svg/* 通过 ?react 后缀作为 React 组件引入）
│   └── styles/index.less       # 全局样式入口
├── docs/                       # 设计文档（含 ui.md / data.md / README.en.md）
├── .env.development            # 开发环境变量
├── .env.qa                     # QA 环境变量
├── .env.production             # 生产环境变量
├── eslint.config.mjs           # ESLint 9 flat config
├── tsconfig.json               # path alias + 严格模式
├── package.json
└── README.md
```

## 4. 路径别名

`tsconfig.json#paths` 与 `vite.config.ts#resolve.alias` **必须保持同步**。优先使用别名，避免 `../../../` 长相对路径：

| 别名           | 实际路径           |
| -------------- | ------------------ |
| `@/*`          | `src/*`            |
| `Components/*` | `src/components/*` |
| `Utils/*`      | `src/utils/*`      |

```ts
import {Button, Card} from '@/components';
import {useStores} from '@/store';
import routes from '@/router';
import request from '@/api/request';
```

## 5. 脚本与环境变量

### 5.1 npm scripts（`package.json`）

| 命令                    | 行为                                                    |
| ----------------------- | ------------------------------------------------------- |
| `npm run dev` / `start` | `vite` 启动 dev server（默认 `development` mode）       |
| `npm run build:qa`      | `vite build --mode qa`                                  |
| `npm run build:prod`    | `vite build --mode production`                          |
| `npm run preview`       | `vite preview`，本地预览生产产物                        |
| `npm run deploy`        | `build:prod` + `gh-pages -d build`，发布到 GitHub Pages |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [guokaigdg/react-seed](https://github.com/guokaigdg/react-seed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
