---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

芝士学爆（Gitee 仓库 review-system；package.json 与数据目录名仍为 forgetting-curve-reminder）— 基于 Electron + React + TypeScript 的 Windows 桌面应用。基于 FSRS 算法管理知识点复习提醒，集成番茄钟、每日计划、学习统计等功能。

## 常用命令

```bash
# 开发模式（Vite + Electron 热重载）
npm run dev

# 类型检查
npx tsc --noEmit

# ESLint 静态检查（0 错误，警告可存在）
npm run lint

# 单元测试（Vitest + Testing Library）
npm run test

# 生产构建
npm run build

# 一键构建+打包（需设 ELECTRON_MIRROR 镜像加速）
npm run package

# 同步 main+tags 到 GitHub（构建发布仓库）
npm run sync-github
```

**工作流程（必须遵守）**：用户提出需求 → 修改代码 → **自检** → 用户检查确认满足要求 → **此时才打包** Windows 安装包（输出到 `release/芝士学爆-Setup-<版本号>.exe`，版本号见 package.json）。未经用户确认，不执行 `npm run package`。

**自检分级**：

- **简单任务**（单文件小改、纯样式/文案调整、类型修复）：`npx tsc --noEmit` 零错误 + `npm run build` 通过即可
- **复杂任务**（涉及 3+ 文件、跨模块交互链路、弹窗/表单/导航等 UI 行为变更、数据层或迁移改动）：在上述基础上，**必须通过 Playwright（浏览器连接 dev server，可用 mock 注入 electronAPI）验证交互行为**

## CI/CD 与发布（GitHub Actions）

- **双仓库**：Gitee `review-system` 为主开发仓库；GitHub `Strelizia-PJ/review-system`（public）为构建发布仓库（remote 名 `github`）。日常推 Gitee，发布前 `npm run sync-github`
- **ci.yml**（push/PR main）：lint → tsc → vitest → vite build
- **release.yml**（tag `v*` 触发）：windows-latest 产出 NSIS 安装包（含 latest.yml/blockmap），macos-latest 产出 DMG（x64+arm64，ad-hoc 签名），全部自动上传 GitHub Release（内置 GITHUB_TOKEN，无需个人凭据）
- **发布 SOP**：`npm version <版本>` → `npm run sync-github`（tag 推送触发构建）→ 浏览器或匿名 API 确认 Actions 成功 → Gitee 网页手动建对应 Release
- **自动更新**：electron-updater，仅 Windows 打包环境启用（启动 30s 后静默检查 + 设置页手动检查/进度/重启安装）；更新源为 GitHub Releases。macOS 未签名不支持在线更新（设置页显示跳转下载页）。CI 状态可用匿名 API `https://api.github.com/repos/Strelizia-PJ/review-system/actions/runs` 查询
- **已知限制**：国内访问 GitHub 更新源可能慢（后续可加 OSS 镜像）；macOS 首次打开需右键→打开

## 工程化规范

- **ESLint**（flat config `eslint.config.mjs`）：typescript-eslint recommended + react-hooks；`no-explicit-any` 降为警告（迁移代码的历史字段访问）；空 catch 允许；提交前 husky+lint-staged 自动 `eslint --fix` + `prettier`
- **Prettier**：无分号、单引号、行宽 110
- **测试**（`tests/`）：Vitest + @testing-library/react + happy-dom；覆盖 constants 纯函数（FSRS 预览/保留率/封顶）与共享组件（Button/Badge/Bars/ConfirmDialog）；数据层单测待 IStorage 抽象后补充

## 技术栈

| 层          | 技术                                                         |
| ----------- | ------------------------------------------------------------ |
| 桌面框架    | Electron 42                                                  |
| 前端        | React 18 + TypeScript                                        |
| 构建        | Vite 8 + vite-plugin-electron                                |
| 样式        | Tailwind CSS 3（`darkMode: 'class'`）                        |
| 状态管理    | Zustand 5                                                    |
| 数据存储    | JSON 文件（`%APPDATA%/forgetting-curve-reminder/data.json`） |
| Markdown    | @uiw/react-md-editor + remark-math + rehype-katex            |
| 非标准 JSON | json5                                                        |
| 打包        | electron-builder (NSIS)                                      |

## 架构分层

### 数据流

```
React Component → Zustand Hook → preload.ts (contextBridge)
    → IPC (ipcMain.handle) → queries.ts → connection.ts → data.json
```

### 目录结构

```
electron/                  # Electron 主进程（Node.js 环境）
├── main.ts                # 窗口管理 + IPC 处理器注册 + 生命周期
├── preload.ts             # contextBridge，暴露 electronAPI 给渲染进程
├── tray.ts                # 系统托盘
├── notifications.ts       # Windows 原生通知
├── scheduler.ts           # 定时复习检查（每小时）
├── auto-start.ts          # 开机自启动
└── database/
    ├── connection.ts      # JSON 文件读写 + AppData 接口 + ID 计数器
    ├── migrations.ts      # schema_version 递增迁移
    ├── queries.ts         # 知识复习 + 每日计划 + 学习会话 + 统计查询
    ├── game-import.ts     # Chill with You 游戏存档解析导入
    └── images.ts          # 本地图片文件存储 + 孤儿图片清理

src/                       # React 渲染进程
├── main.tsx               # React 入口
├── App.tsx                # 根组件
├── vite-env.d.ts          # window.electronAPI 全局类型声明
├── types/index.ts         # 共享 TypeScript 类型
├── hooks/                 # Zustand stores
│   ├── useKnowledge.ts    # 知识点 CRUD + 选中状态
│   ├── useReview.ts       # 复习数据查询
│   ├── useTheme.ts        # 暗色模式（light/dark/system）
│   ├── useDailyPlans.ts   # 每日计划
│   ├── usePomodoro.ts     # 番茄钟计时核心逻辑
│   └── useStudyStats.ts   # 学习统计 + 月份导航
├── components/
│   ├── layout/            # AppLayout（导航框架）+ Sidebar
│   ├── knowledge/         # 知识点列表、卡片、添加表单、详情页(Markdown)
│   ├── review/            # 今日复习（含逾期）、统计面板
│   ├── mistakes/          # 易错点（计数器 + 排序）
│   ├── manage/            # 调度管理（全局/单点间隔上限、改期、提前复习）
│   ├── plans/             # 每日计划页面
│   ├── pomodoro/          # 番茄钟计时器页面
│   ├── stats/             # 学习统计（日历热力图 + 补登）
│   └── import/            # 游戏数据导入
├── constants.ts           # 共享常量（QUALITY_LABELS, DAY_LABELS_SUNDAY_FIRST, DAY_LABELS_MONDAY_FIRST）
└── styles/
    └── index.css           # 全局样式 + @uiw/react-md-editor 适配
```

### 导航系统

应用使用基于状态的页面切换（非路由）。`AppLayout` 维护 `currentPage: NavPage` 状态，`Sidebar` 渲染导航按钮。`NavPage` 类型定义在 `src/types/index.ts`：

```ts
;'knowledge' |
  'today' |
  'mistakes' |
  'manage' |
  'plans' |
  'pomodoro' |
  'study-stats' |
  'import' |
  'stats' |
  'settings'
```

逾期复习不再单独成页：今日复习面板包含全部待复习条目（含逾期，红色高亮标注）。

知识点详情页通过 `useKnowledge.selectedId` 单独处理，不占用 NavPage。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Strelizia-PJ/review-system](https://github.com/Strelizia-PJ/review-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
