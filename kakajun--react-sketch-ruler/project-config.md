---
trigger: always_on
description: > 本文件面向 AI 编程助手。如果你正在阅读此文件，说明你需要在不了解项目背景的情况下快速上手 `react-sketch-ruler`。以下内容全部基于项目实际代码与配置，请勿凭假设推断。
---

# AGENTS.md

> 本文件面向 AI 编程助手。如果你正在阅读此文件，说明你需要在不了解项目背景的情况下快速上手 `react-sketch-ruler`。以下内容全部基于项目实际代码与配置，请勿凭假设推断。

---

## 项目概述

`react-sketch-ruler` 是一个基于 **React + TypeScript** 的标尺组件库，由 `vue3-sketch-ruler` 迁移而来，与 Vue 版本共用底层包 `@sketch-ruler/core` 和 `@sketch-ruler/canvas`。适用于低代码平台、大屏可视化、做图工具等场景，提供类似 Photoshop 的缩放、平移与标尺辅助线体验。

- **版本**：`3.0.0-beta.0`
- **协议**：MIT
- **仓库地址**：https://github.com/kakajun/react-sketch-ruler
- **线上 Demo**：https://kakajun.github.io/react-sketch-ruler

项目采用 **pnpm workspace** 管理的单体仓库（monorepo），包含两个子包：

| 包路径                  | 说明                                                |
| ----------------------- | --------------------------------------------------- |
| `packages/sketch-ruler` | 组件库本体，发布到 npm（包名 `react-sketch-ruler`） |
| `packages/sketch-ruler/AGENTS.md` | 插件系统详细指南（面向 AI 助手）             |
| `packages/docs`         | 文档与示例站点，部署到 GitHub Pages                 |

---

## 技术栈与运行时架构

### 核心依赖

- **React**：Peer dependency，支持 `>=18 || >=19`
- **React-DOM**：同上
- **底层引擎**：`@sketch-ruler/core`（TransformEngine、PluginManager、CanvasManager 等）、`@sketch-ruler/canvas`（InputManager）
- **构建工具**：Vite（库模式 + 文档站点）
- **类型生成**：`vite-plugin-dts`（Rollup 合并类型声明）
- **样式**：Less（`*.less`）
- **测试**：Vitest + `@testing-library/react` + `jsdom`
- **格式化**：`oxfmt`
- **静态检查**：`oxlint`
- **包管理**：pnpm（`pnpm-workspace.yaml` 定义 `packages/*`）

### 构建产物（`packages/sketch-ruler`）

- `lib/index.js` — ESM 入口
- `lib/index.umd.cjs` — UMD 入口
- `lib/index.css` — 组件样式
- `lib/index.d.ts` — 类型声明

### 运行时架构要点

- **TransformEngine**：内置变换引擎，负责缩放/平移状态管理，零外部 panzoom 依赖。
- **InputManager**：来自 `@sketch-ruler/canvas`，封装鼠标/触摸/滚轮事件。
- **PluginManager**：来自 `@sketch-ruler/core`，支持生命周期钩子（`beforeZoom`、`afterZoom`、`onLineCreate` 等）。
- **缩放原点计算**：`zoomMode`（`pointer` / `viewport-center` / `content-center`）统一由 `@sketch-ruler/core` 的 `getZoomOrigin()` 计算，React 层不再手写原点逻辑。
- **插槽机制**：`SketchRule` 的 `children` 中，若元素带有 `slot="toolbar"`，会被提取到顶部工具栏区域渲染；其余内容作为画布主体。
- **多实例**：每个 `SketchRule` 实例拥有独立的 `TransformEngine`。

---

## 仓库结构与模块划分

```
react-sketch-ruler
├── packages/
│   ├── sketch-ruler/          # 组件库
│   │   ├── src/
│   │   │   ├── index.tsx      # 入口：导出组件、Hooks、类型、插件工具
│   │   │   ├── index-types.ts # React 层 Props / Methods / Palette 类型定义
│   │   │   ├── sketch-ruler/
│   │   │   │   ├── index.tsx       # SketchRule 主组件（forwardRef + useImperativeHandle）
│   │   │   │   ├── RulerWrapper.tsx# 标尺容器（横/纵）
│   │   │   │   ├── RulerLine.tsx   # 单条参考线渲染与交互
│   │   │   │   ├── index.less      # 组件样式
│   │   │   │   └── cornerImg64.ts  # 左上角眼睛图标 Base64
│   │   │   ├── minimap/
│   │   │   │   └── index.tsx       # Minimap 缩略图组件
│   │   │   ├── hooks/
│   │   │   │   ├── useCanvasTransform.ts # 封装 TransformEngine
│   │   │   │   ├── useGuideLines.ts      # 参考线状态管理
│   │   │   │   ├── useInputManager.ts    # 绑定 InputManager
│   │   │   │   ├── useSketchRuler.ts     # 组合 Hook（底层控制）
│   │   │   │   ├── useRulerScale.ts      # 刻度计算
│   │   │   │   └── useSnapDetection.ts   # 吸附逻辑
│   │   │   ├── plugins/
│   │   │   │   └── index.ts       # definePlugin 辅助函数 + 核心类型重导出
│   │   │   └── global.d.ts        # *.less 模块声明
│   │   ├── test/
│   │   │   ├── sketch-ruler.spec.tsx      # 主组件测试
│   │   │   ├── use-canvas-transform.spec.ts
│   │   │   ├── use-guide-lines.spec.ts
│   │   │   ├── use-input-manager.spec.ts
│   │   │   ├── use-ruler-scale.spec.ts
│   │   │   ├── use-sketch-ruler.spec.ts
│   │   │   └── use-snap-detection.spec.ts # Hooks 测试
│   │   ├── vite.config.ts         # 库构建配置
│   │   ├── tsconfig.json          # 继承 ../../tsconfig.common.json
│   │   └── package.json           # 包名 react-sketch-ruler
│   └── docs/                      # 文档示例站点
│       ├── src/
│       │   ├── main.tsx           # 应用入口（React 18 createRoot）
│       │   ├── App.tsx            # 根组件
│       │   ├── router/            # 路由配置
│       │   ├── i18n/              # react-i18next 国际化配置与语言包
│       │   ├── components/layout/ # 布局组件（Aside、Header）
│       │   ├── examples/          # 各种示例页面（Basic、Comprehensive、Bigscreen…）
│       │   └── assets/            # 图片、样式、图标
│       ├── vite.config.ts         # 站点构建配置（base: './'）
│       ├── tsconfig.json
│       └── package.json           # 包名 root-doc
├── scripts/
│   └── release.js               # 交互式发版脚本（semver + enquirer）
├── .github/workflows/
│   └── gh-pages.yml             # CI：测试 → 构建 Demo → 部署 GitHub Pages
├── tsconfig.common.json         # 公共 TS 配置（strict、composite、declarationMap）
├── tsconfig.json                # 根配置，仅用于 IDE 自动补全（references 指向两个子包）
├── .oxfmtrc.json                # oxfmt 格式化配置
├── .oxlintrc.json               # oxlint 静态检查配置
├── .node-version                # Node 版本：v22.17.0
├── pnpm-workspace.yaml
└── package.json                 # root（private）
```

---

## 常用命令

所有命令均在仓库根目录执行：

| 命令              | 说明                                                     |
| ----------------- | -------------------------------------------------------- |
| `pnpm install`    | 安装依赖                                                 |
| `pnpm dev`        | 先构建组件库，再启动文档站点开发服务器（`0.0.0.0:5274`） |
| `pnpm build`      | 构建组件库（输出到 `packages/sketch-ruler/lib`）         |
| `pnpm build:demo` | 构建组件库 + 构建文档站点（输出到 `packages/docs/dist`） |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kakajun/react-sketch-ruler](https://github.com/kakajun/react-sketch-ruler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
