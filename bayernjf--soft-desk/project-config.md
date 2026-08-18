---
trigger: always_on
description: 本文档供 AI coding agents（Trae / Claude Code / Cursor / Codex / Copilot 等）在本项目工作时自动读取。
---

# AGENTS.md — SoftDesk 项目指令

本文档供 AI coding agents（Trae / Claude Code / Cursor / Codex / Copilot 等）在本项目工作时自动读取。
请严格遵循以下约定。

---

## 项目概览

SoftDesk 是一款跨平台本地软件管理与智能启动工具，支持 macOS + Windows 双端。

- **桌面框架**：Electron 31 + React 18 + TypeScript
- **构建工具**：Vite 6 + electron-builder 24
- **包管理器**：npm（**不要用 pnpm/yarn**）
- **测试**：Vitest
- **状态管理**：Zustand
- **云同步/认证**：Supabase
- **本地数据库**：better-sqlite3
- **全局快捷键**：Electron globalShortcut
- **鼠标中键监听**：uiohook-napi（需 macOS 辅助功能权限）

### 项目结构

```
soft-desk/
├── electron/               # Electron 主进程
│   ├── main.ts             # 主入口（窗口/托盘/IPC）
│   ├── preload.ts          # preload 脚本（IPC 桥接）
│   ├── scanner.ts          # macOS 软件扫描
│   ├── scanner-win.ts      # Windows 软件扫描
│   ├── database.ts         # 本地数据库（better-sqlite3）
│   ├── auth.ts             # Supabase 认证
│   ├── ai.ts               # AI 软件分类
│   ├── updater.ts          # 自动更新（electron-updater）
│   ├── monitor.ts          # 使用时长监控
│   ├── window-locator.ts   # 已运行窗口聚焦
│   └── lib/logger.ts       # 主进程日志
├── src/                    # React 渲染进程
│   ├── pages/              # 页面组件（Dashboard/Library/Favorites 等）
│   ├── components/         # UI 组件
│   │   ├── features/       # 功能组件（SoftwareCard/RadialMenu 等）
│   │   └── layout/         # 布局组件（Layout/Sidebar）
│   ├── stores/             # Zustand 状态管理
│   ├── services/           # 业务服务层
│   ├── hooks/              # 自定义 hooks
│   ├── lib/                # 工具库（supabase/searchMatch 等）
│   ├── types/              # TypeScript 类型定义
│   └── data/               # 静态数据（分类/AI Provider 等）
├── build/                  # 图标等构建资源
├── scripts/                # 构建脚本（after-pack/tray-icon）
├── docs/                   # 项目文档
├── .github/workflows/      # CI 配置
└── package.json            # 单包配置（非 monorepo）
```

### 平台注意事项

- **主进程 vs 渲染进程**：`electron/` 下是主进程代码，`src/` 下是渲染进程代码。主进程通过 `preload.ts` 暴露的 `window.softdesk` 与渲染进程通信。
- **跨平台扫描**：`scanner.ts`（macOS）和 `scanner-win.ts`（Windows）是两套独立实现，通过 `CROSS_PLATFORM_RULES` 对齐 bundleId。
- **原生模块**：`better-sqlite3` 和 `uiohook-napi` 需要针对 Electron 版本重编译（`npm run rebuild`）。
- **preload CJS**：preload 用 esbuild 单独打包成纯净 CJS，不要改 vite.config.ts 里的 `preload-force-cjs` 插件逻辑。
- **多入口**：Vite 配置了 3 个入口（main/radial/animation），radial 是径向菜单独立窗口。

---

## 常用命令

所有命令在仓库根目录执行。

### 本地开发

```bash
cp .env.example .env         # 首次：填入 Supabase URL 和 anon key
npm install                  # 安装依赖
npm run dev                  # 启动开发模式（Vite + Electron，热更新）
npm run restart              # 杀掉残留 Electron 进程后重启 dev
```

### 验证（提交前必须通过）

```bash
npm run check                # TypeScript 类型检查（tsc --noEmit）
npm run lint                 # ESLint
npm run test                 # Vitest 单测
npm run build                # 生产构建（tsc + vite build）
```

### 打包

```bash
npm run dist:mac             # 打包 macOS（dmg）
npm run dist:win             # 打包 Windows（exe/nsis）— 需在 Windows 上执行
npm run rebuild              # 重编译原生模块（better-sqlite3/uiohook-napi）
```

---

## 环境变量

- **`.env` 被 .gitignore**，仅 `.env.example` 入库
- 只有一份 `.env`，放在仓库根目录
- 变量前缀：`VITE_`（同时注入到渲染进程和主进程）
- `VITE_SUPABASE_URL` / `VITE_SUPABASE_ANON_KEY`：Supabase 连接信息
- **不要在前端代码里硬编码 Supabase URL 或 key**

---

## 代码规范

### Commit Message 格式

遵循 `.trae/rules/git-commit-message.md`：

```
<type>(<scope>): <短命令式描述, <=50 字符>

<可选 body：说明改动目的>
```

**type**：`feat` / `fix` / `refactor` / `chore` / `docs` / `test` / `style` / `perf` / `ci`
**scope**：`scanner` / `updater` / `settings` / `ci` / `build` / 具体模块名

**原子提交规则**：
- 按业务模块/文件类型拆分，每个 commit 只做一件事
- docs / config / test / feature / fix 分别独立 commit
- Commit message 用**英文**
- **Never merge unrelated file edits into one commit**

### 通用约定

- 主进程代码放 `electron/`，渲染进程代码放 `src/`，不要混放
- 跨平台逻辑通过 `src/services/software-matching.ts` 的 `matchSoftware` 统一匹配（id → bundleId fallback）
- 新增 IPC 通道时，在 `src/types/electron.d.ts` 里补类型声明
- 前端样式用 Tailwind CSS，不要引入其他 CSS 框架
- 数据库/API 的枚举字段必须保存稳定的英文机器值，禁止保存中文展示文案
- 枚举合法值、TypeScript 类型、国际化 key 和状态样式应集中在对应领域文件，禁止在页面重复定义映射
- 页面展示文案通过 `i18next` 翻译 key 获取；用户输入的标题、正文、名称等自由文本保持原文
- SQL `CHECK` 约束必须与 TypeScript `as const` 枚举保持一致，外部数据在 Service 边界进行运行时校验
- Node 版本 24（CI 环境）

---

## Git 工作流

### 分支

| 分支 | 用途 | 触发 CI | 触发 Release |
|------|------|---------|-------------|
| `main` | 生产，**必须 PR review 合并** | ✅ CI + 打包 | ✅ 自动 patch 发版 |
| `dev` | 开发集成 | ✅ CI + 打包 | ✅ Dev Snapshot |
| `feature/*` | 功能/修复分支 | ❌ 不触发 | ❌ |

### 发版机制

| 触发方式 | 行为 | 版本号 | tag | prerelease |
|---------|------|--------|-----|:----------:|
| push `v*` tag（手动） | 正式发布 | 指定版本号 | vX.Y.Z | false |
| push/合并到 `main` | 正式发布，自动 patch 递增 | 最新 tag +0.0.1 | vX.Y.Z | false |
| push 到 `dev` | Dev Snapshot，覆盖式更新 | `0.0.0-dev`（固定） | `snapshot`（固定） | true |

**dev snapshot 覆盖机制**：每次构建前先删除旧的 snapshot release，再重建，保证 assets 始终只有最新的一份。

### Snapshot 版本管理规则（必须遵守）

- snapshot 版本号固定为 `0.0.0-dev`，**不包含日期、commit hash 等变化后缀**
- artifact 文件名通过 electron-builder `artifactName` 配置，必须固定（不含版本号或使用固定版本号），确保新构建能覆盖旧文件
- publish 前必须先删除 snapshot release 上的旧 assets，防止文件堆积
- 正式 release（vX.Y.Z）保留历史文件，不删除旧 assets
- `scripts/set-version.cjs`（如有）是版本注入的唯一入口，snapshot 模式输出 `0.0.0-dev`

### 三条核心指令（开发者说以下话时执行）

#### 1. 「提交代码」——仅 commit + push 当前分支，不合并

1. `git status` 检查改动
2. 分析改动，按原子规则拆分 commit（参考 `.trae/rules/git-commit-message.md`）
3. `git pull --rebase`（push 前必须先 pull，有冲突则报告后停止）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bayernjf/soft-desk](https://github.com/bayernjf/soft-desk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
