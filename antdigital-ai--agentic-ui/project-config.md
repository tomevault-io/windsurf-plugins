---
trigger: always_on
description: > Agentic UI 项目开发指南 - 为 AI 编程助手提供项目上下文和开发规范
---

# AGENTS.md

> Agentic UI 项目开发指南 - 为 AI 编程助手提供项目上下文和开发规范

## 📑 目录

- [项目背景](#项目背景)
- [快速开始](#快速开始)
- [代码规范](#代码规范)
  - [基本编码规范](#基本编码规范)
  - [命名规范](#命名规范)
  - [TypeScript 规范](#typescript-规范)
  - [样式规范](#样式规范)
  - [代码格式化](#代码格式化)
- [开发指南](#开发指南)
  - [测试指南](#测试指南)
  - [演示代码规范](#演示代码规范)
  - [国际化规范](#国际化规范)
  - [组件开发模板](#组件开发模板)
- [文档和 Changelog](#文档和-changelog-规范)
- [Git 和 Pull Request](#git-和-pull-request-规范)
- [质量保证](#质量保证)
- [工具链和环境](#工具链和环境)
- [常见问题和故障排查](#常见问题和故障排查)

---

## 项目背景

这是 [ant-design/agentic-ui](https://github.com/ant-design/agentic-ui) 的源代码仓库，发布为 npm 包 `@ant-design/agentic-ui`，是一个面向智能体的 UI 组件库。

### 核心特性

- 使用 TypeScript 和 React 开发
- 兼容 React 16.9+ 版本（peerDependencies: `>=16.9.0`）
- 基于 Ant Design 和 `@ant-design/cssinjs` 构建
- 提供多步推理可视化、工具调用展示、任务执行协同等 Agentic UI 能力
- 支持国际化（i18n）
- 使用 Dumi 构建文档站点
- 使用 Father 构建产物
- 使用 Vitest + React Testing Library 进行单元测试
- 使用 Playwright 进行 E2E 测试

### 设计理念

- **过程透明化**：可见思考与工具调用，让用户理解智能体的推理过程
- **主动协作**：智能体主动发起交互，与用户协同完成任务
- **端到端任务协同**：从"回答一句话"到"完成一件事"，让智能体成为用户的协作伙伴

---

## 快速开始

### 开发环境要求

- **Node.js**: >= 18.17.0（推荐 22 LTS）
- **包管理器**: 仓库锁定 **pnpm 9.15.9**（与 `pnpm-lock.yaml` lockfile v9 一致；**勿用全局 pnpm 10+**，其在 Node 18 上会报 `requires at least Node.js v22.13`）
- **操作系统**: Windows 10+, macOS 10.15+, Linux
- **浏览器兼容性**: 现代浏览器（Chrome 80+、Edge、Firefox、Safari）

### 安装依赖

```bash
# 克隆项目
git clone git@github.com:ant-design/agentic-ui.git
cd agentic-ui

# 使用 Corepack 启用 package.json 中锁定的 pnpm 版本（Node 16.9+ 自带 Corepack）
corepack enable
pnpm install
```

若未使用 Corepack，可显式激活同一版本：

```bash
corepack enable
corepack prepare pnpm@9.15.9 --activate
pnpm install
```

### 常用开发命令

```bash
pnpm start              # 启动文档站点（http://localhost:8000）
pnpm run build          # 构建项目
pnpm test               # 运行单元测试（默认跳过 e2e、大体积 chart/Workspace 目录、branches/coverage 等补洞套件，见 vitest.config.ts）
pnpm run test:full      # 全量 Vitest（`--mode full`；与 `test:coverage:full` 同一套 exclude，不含覆盖率）
pnpm run test:coverage  # 生成覆盖率（默认同上精简集）
pnpm run test:coverage:full  # 全量测试 + 覆盖率（`--mode full`；Codecov workflow / 发布前）
pnpm run test:e2e       # 运行 E2E 测试
pnpm run lint           # 代码检查（ESLint + Stylelint）
pnpm run prettier       # 代码格式化
pnpm tsc                # TypeScript 类型检查
```

### 项目结构

```
agentic-ui/
├── src/                    # 组件源代码
│   ├── ComponentName/      # 单个组件目录（如 Bubble、MarkdownEditor、ThoughtChainList 等）
│   │   ├── ComponentName.tsx   # 主组件实现（部分组件）
│   │   ├── SubComponent.tsx    # 子组件（如 AIBubble.tsx、UserBubble.tsx）
│   │   ├── style.ts            # 样式文件（CSS-in-JS）
│   │   ├── types.ts            # 类型定义
│   │   ├── hooks/              # 组件专属 hooks
│   │   ├── utils/              # 工具函数
│   │   ├── plugins/            # 插件（如 MarkdownEditor）
│   │   ├── components/         # 子组件目录（如 Bubble/List、Bubble/MessagesContent）
│   │   ├── __tests__/          # 测试文件
│   │   └── index.tsx           # 导出入口
│   ├── Components/           # 通用组件
│   ├── Hooks/                # 共享 Hooks
│   ├── Utils/                # 工具函数库
│   ├── I18n/                 # 国际化
│   ├── Types/                # 共享类型定义
│   ├── Constants/            # 常量定义
│   └── index.ts              # 组件总入口
├── docs/                    # 文档
│   ├── components/           # 组件 API 文档
│   │   ├── bubble.md
│   │   ├── markdown-editor.md
│   │   └── ...
│   └── development/          # 开发相关文档
│       ├── changelog.zh-CN.md
│       └── changelog.en-US.md
├── tests/                   # 测试文件
├── e2e/                     # E2E 测试
├── scripts/                 # 构建和工具脚本
├── .dumirc.ts               # Dumi 配置
├── .fatherrc.ts             # Father 构建配置
├── vitest.config.ts         # Vitest 配置
├── playwright.config.ts     # Playwright 配置
├── package.json
└── tsconfig.json
```

---

## 代码规范

### 基本编码规范

- ✅ 使用 TypeScript 和 React 书写
- ✅ 使用函数式组件和 Hooks，**避免类组件**
- ✅ 使用 `forwardRef` 实现组件 ref 传递
- ✅ 使用提前返回（early returns）提高代码可读性
- ✅ 避免引入新依赖，严控打包体积
- ✅ 兼容现代浏览器
- ✅ 保持向下兼容，避免 breaking change
- ✅ 组件名使用大驼峰（PascalCase），如 `MarkdownEditor`、`Bubble`
- ✅ 属性名使用小驼峰（camelCase），如 `onClick`、`defaultValue`
- ✅ 合理使用 `React.memo`、`useMemo` 和 `useCallback` 优化性能
- ✅ 使用 `clsx` 处理类名拼接
- ✅ 支持 Semantic 样式系统（`classNames` 和 `styles` 属性）

#### Props 命名

| 用途       | 命名规则                        | 示例                          |
| ---------- | ------------------------------- | ----------------------------- |
| 初始化属性 | `default` + `PropName`          | `defaultValue`、`defaultOpen` |
| 强制渲染   | `forceRender`                   | `forceRender`                 |
| 子组件渲染 | `SubComponentName` + `Render`   | `titleRender`、`footerRender` |
| 数据源     | `dataSource`                    | `dataSource`                  |
| 面板开启   | 使用 `open`，避免使用 `visible` | `open`、`defaultOpen`         |
| 显示相关   | `show` + `PropName`             | `showSearch`、`showHeader`    |
| 功能性     | `PropName` + `able`             | `disabled`、`readable`        |
| 禁用       | `disabled`                      | `disabled`                    |
| 额外内容   | `extra`                         | `extra`                       |
| 图标       | `icon`                          | `icon`、`prefixIcon`          |
| 触发器     | `trigger`                       | `trigger`                     |
| 配置属性   | `Config` 后缀                   | `toolbarConfig`               |

#### 事件命名

| 类型       | 命名规则                                | 示例                  |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antdigital-ai/agentic-ui](https://github.com/antdigital-ai/agentic-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
