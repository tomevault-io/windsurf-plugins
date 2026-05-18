---
trigger: always_on
description: <!-- AI 必读：在本项目中，你的所有推理过程、代码注释和回答都必须使用中文进行交互。 -->
---

<!-- AI 必读：在本项目中，你的所有推理过程、代码注释和回答都必须使用中文进行交互。 -->

# 🤖 Robot Admin — AI 编码指南

> **本文件面向 AI 编程助手**（Copilot / Cursor / Claude 等）。
> 在对本项目生态进行任何代码生成、修改或建议之前，**必须完整阅读本指南**。
> 最后更新：2026-04-13

## 目录

一、项目全景 · 二、技术栈与工具链 · 三、包管理器与运行命令 · 四、项目结构与目录约定 · 五、编码规范 · 六、Vue SFC 编写规范 · 七、组件库编写规范 · 八、演示页面编写规范 · 九、Store 编写规范 · 十、API 与请求规范 · 十一、路由与权限 · 十二、样式与主题规范 · 十三、TypeScript 规范 · 十四、Git 提交规范 · 十五、ESLint 规则摘要 · 十六、构建与部署 · 十七、生态包速查表 · 十八、常见坑与注意事项 · 十九、新功能开发 Checklist

---

## 一、项目全景

Robot Admin 是一个**企业级后台管理系统**生态，由 4 个关联仓库组成：

| 仓库                     | 简称   | 用途                                     | npm 包名                           |
| ------------------------ | ------ | ---------------------------------------- | ---------------------------------- |
| **Robot_Admin**          | 主项目 | Vue 3 SPA 后台应用                       | —                                  |
| **naive-ui-components**  | 组件库 | 51 个业务组件（基于 Naive UI）           | `@robot-admin/naive-ui-components` |
| **robot-admin-packages** | 包集合 | 7 个独立 npm 包（指令/请求/布局/主题等） | `@robot-admin/*`                   |
| **AgileTeam_Doc**        | 文档库 | VitePress 2.0 团队文档站                 | —                                  |

### 核心信息

- **作者**: ChenYu (`ycyplus@gmail.com`)
- **许可证**: MIT
- **演示站**: https://robotadmin.cn
- **Node 版本要求**: `>=22.x`
- **包管理器**: **Bun** `>=1.x`（**不使用 npm / yarn / pnpm**）

---

## 二、技术栈与工具链

### 核心框架

| 技术       | 版本   | 用途                                            |
| ---------- | ------ | ----------------------------------------------- |
| Vue        | 3.5.30 | 渐进式框架                                      |
| TypeScript | ~5.8.3 | 类型安全                                        |
| Vite       | 8.0.3  | 构建工具                                        |
| Naive UI   | 2.44.1 | UI 组件库                                       |
| Pinia      | 3.0.4  | 状态管理                                        |
| Vue Router | 4.6.4  | 路由系统                                        |
| UnoCSS     | 66.6.6 | 原子化 CSS（presetWind3 + attributify + icons） |
| Sass       | 1.97.3 | 样式预处理                                      |

### 自有包生态（@robot-admin/\*）

| 包名                               | 版本  | 功能                           |
| ---------------------------------- | ----- | ------------------------------ |
| `@robot-admin/naive-ui-components` | 0.8.2 | 51+ 个业务组件                 |
| `@robot-admin/layout`              | 2.2.0 | 6 种布局模式 + 设置管理        |
| `@robot-admin/request-core`        | 0.1.3 | Axios + 7 插件 + useTableCrud  |
| `@robot-admin/theme`               | 0.1.1 | 主题切换（Light/Dark/System）  |
| `@robot-admin/directives`          | 1.1.0 | 11 个 Vue 指令                 |
| `@robot-admin/form-validate`       | 2.0.0 | 48+ 验证规则                   |
| `@robot-admin/file-utils`          | 1.0.0 | 文件处理（Excel/ZIP/分片上传） |
| `@robot-admin/git-standards`       | 1.0.3 | Git 工程化标准                 |

### 开发工具链

| 工具                    | 用途                                                   |
| ----------------------- | ------------------------------------------------------ |
| ESLint 10 + Oxlint      | 双重 Lint（Oxlint 用 Rust 编写，极速）                 |
| Prettier 3.8            | 代码格式化                                             |
| Commitizen + Commitlint | Git 提交规范                                           |
| Husky 9 + lint-staged   | Pre-commit 钩子                                        |
| unplugin-auto-import    | Vue/Router/Pinia/VueUse 自动导入                       |
| unplugin-vue-components | 组件自动导入（NaiveUiResolver + RobotNaiveUiResolver） |

---

## 三、包管理器与运行命令

### ⚠️ 强制使用 Bun

```bash
# ✅ 正确
bun install
bun run dev
bun run build
bun run lint

# ❌ 错误 — 不要使用
npm install
yarn install
pnpm install
```

### 主要脚本

| 命令                     | 用途         | 说明                        |
| ------------------------ | ------------ | --------------------------- |
| `bun run dev`            | 标准开发     | 默认端口 1988               |
| `bun run dev:local`      | 本地包调试   | `USE_LOCAL_PACKAGES=true`   |
| `bun run dev:components` | 组件库联调   | `USE_LOCAL_COMPONENTS=true` |
| `bun run dev:devtools`   | Vue DevTools | `VITE_DEVTOOLS=true`        |
| `bun run build`          | 生产构建     | env-manager prod 模式       |
| `bun run build:test`     | 测试构建     | `--mode test`               |
| `bun run build:staging`  | 预发构建     | `--mode staging --profile`  |
| `bun run lint`           | 代码检查     | Oxlint → ESLint 双重检查    |
| `bun run format`         | 代码格式化   | Prettier                    |
| `bun run type-watch`     | 实时 TS 检查 | `vue-tsc --watch`           |
| `bun run analyze`        | 构建分析     | rollup-plugin-visualizer    |
| `bun run cz`             | 规范化提交   | Commitizen 交互式           |

### 组件库脚本（naive-ui-components）

```bash
bun run dev          # watch 模式开发
bun run build        # tsdown + scss + merge-css + gen-exports 全流程
bun run lint         # oxlint + eslint
bun run check:exports # 检测导出命名冲突
```

### 包集合脚本（robot-admin-packages）

```bash
# 在具体包目录下
bun run build        # 构建单个包
bun run changeset    # 创建变更集
bun run version      # 版本号递增
bun run release      # 发布到 npm
```

---

## 四、项目结构与目录约定

### Robot_Admin 主结构

```
Robot_Admin/
├── src/
│   ├── main.ts                    # 应用入口（启动引导流程）
│   ├── App.vue                    # 根组件（NConfigProvider 包裹）
│   │
│   ├── api/                       # API 接口定义

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChenyCHENYU/Robot_Admin](https://github.com/ChenyCHENYU/Robot_Admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
