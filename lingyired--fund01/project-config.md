---
trigger: always_on
description: 你是一位精通 Chrome Extension (MV3)、TypeScript、React、pnpm workspaces、Tauri 2 的工程师。你写可维护、高性能的代码。本文档指导你在 `fund01` monorepo 中进行开发与扩展。
---

# CLAUDE.md — fund01 开发指南

你是一位精通 Chrome Extension (MV3)、TypeScript、React、pnpm workspaces、Tauri 2 的工程师。你写可维护、高性能的代码。本文档指导你在 `fund01` monorepo 中进行开发与扩展。

## 项目目标

`fund01` 是一个基金/组合盯盘工具的 monorepo，目标支持多个运行时：

- **apps/chrome**：Chrome 扩展（popup + badge 模式），MV3 Service Worker 后端定时刷新
- **apps/tauri**（未来）：Tauri 2 桌面应用（macOS menubar app），Rust 后端常驻

共享代码在 `packages/`：

- `@fund01/core` — 纯业务逻辑 + 接口契约（DataPort / ConfigPort / EventPort）
- `@fund01/services` — 数据请求层（原生 fetch）
- `@fund01/ui` — React 组件（无运行时耦合，通过 PortsContext 接受 Port 实现）

## 目录保护规则

- **严禁修改** `node_modules/`、`dist/`、`pnpm-lock.yaml`（除非依赖变更）
- 所有新增、修改、重构工作在 `packages/`、`apps/`、根配置文件、`docs/` 中进行
- 如果需要参考原始实现，`wzk-fund` 仓库（`/Users/lingsmbp/Documents/github/wzk-fund`）的 `server/`、`web/`、`chrome/` **只读查阅**，不要使用 Edit/Write 修改它们
- 迁移代码时，从原仓库读取后，在 `fund01/` 对应位置重新实现

## 命令

```bash
pnpm install          # 安装依赖
pnpm dev:chrome       # 启动 Chrome 扩展开发模式（rsbuild --watch）
pnpm build:chrome     # 构建生产版本到 apps/chrome/dist/
pnpm zip:chrome       # 打包 Chrome 扩展为可上传 Web Store 的 zip，并同步重命名副本到 release-chrome/（Fund01_{version}.zip）
pnpm typecheck        # 全仓库递归 TypeScript 类型检查
node scripts/build-tauri-all.mjs            # 双架构 Tauri 打包（arm64 + x86_64，见「双架构发布产物」）
node scripts/build-tauri-all.mjs --arch arm64   # 仅 Apple Silicon 版
node scripts/build-tauri-all.mjs --arch x86_64  # 仅 Intel 版
pnpm --filter @fund01/tauri tauri:build:release:all   # ⭐ 发 GitHub Release 专用：双架构签名构建 + create-dmg 打 DMG（见「双架构发布产物」）
pnpm --filter @fund01/tauri tauri:build:windows:cross:all        # macOS 本机交叉编译 Windows NSIS 包（x64+arm64，见「Windows 发布产物」）
pnpm --filter @fund01/tauri tauri:build:windows:cross -- --arch x64      # 仅 x64；--arch arm64 仅 arm64
```

**Windows 安装包构建**（见「Windows 发布产物」）：`scripts/build-release-windows.mjs` 是 Windows 专属（非 win32 会被平台校验挡下）；macOS 本机可用 `pnpm --filter @fund01/tauri tauri:build:windows:cross` 交叉编译打 NSIS 包（cargo-xwin）。正式发版仍建议走 GitHub Actions `.github/workflows/build-release.yml`（windows-latest runner 原生构建）。macOS 的 release 包也可选择交给该 workflow（macos-15 runner + Fund01 证书签名，见「发布 workflow」）。

加载扩展：Chrome 打开 `chrome://extensions` → 开启「开发者模式」→「加载已解压的扩展程序」→ 选择 `apps/chrome/dist/`。

## 版本号与构建戳规则

Fund01 是 pnpm monorepo，含两个被分发的产物与若干内部包。**发布版本号**与**构建戳**是两个不同职责，必须分开对待。

### 产物与版本归属
- **Chrome 扩展**（`apps/chrome`）：版本来源 `apps/chrome/package.json` 的 `version`；构建脚本 `scripts/copy-manifest.mjs` 会把它覆盖到 `dist/manifest.json`（**不要只改 manifest.json**）。UI 经 `chromeWindowPort.getVersion()` 读 `chrome.runtime.getManifest().version`。
- **Tauri 桌面端**（`apps/tauri`）：版本来源 `tauri.conf.json` 与 `Cargo.toml` 必须一致（含 `Cargo.lock` 的 `fund01-tauri` 条目，只改该条目）。UI 经 `tauriWindowPort.preloadVersion()` → Rust `get_version` 命令。
- **内部包**（`packages/core`、`packages/ui`、`packages/services`）：纯 workspace 内部包，不单独发布，`dependencies` 均为 `workspace:*`。其 `package.json` 的 `version` 仅为 pnpm 占位，**发布流程不依赖其值，无需主动 bump**；版本真相是 git commit。

### 统一版本号（共享，2026-08-24 起）
Chrome 与 Tauri **共用同一个版本号**（基线 1.3.0）。任何一次发布——无论只改 Chrome、只改 Tauri、还是改了共享 `packages/*`——版本都两端同步 +1；即使本次改动只落在某一端，下次另一端需要更新时版本号也已对齐。用户看到的 Chrome 与桌面端始终是同一个版本，不存在「chrome-only / tauri-only」的独立版本号。

### 语义化版本（SemVer）
`MAJOR.MINOR.PATCH`：
- `PATCH`：修复 / 小幅改动，准备 commit/push 时 +1
- `MINOR`：一个功能或一批相关改动
- `MAJOR`：保留（预发布阶段暂不使用）
- 预发布基线已重置为 **1.0.0**（2026-08-18 落地：Chrome 1.2.80→1.0.0、Tauri 1.0.50→1.0.0，无历史包袱，重新计数）
- **2026-08-24 起双端统一版本号，基线 **1.3.0**：最后一次分叉为 Chrome 1.0.8 / Tauri 1.2.3，此后 Chrome 与 Tauri 不再各自计数，所有 bump 两端同步 +1。

### MINOR / PATCH 判定（怎么决定）
Fund01 是预发布、自用型 app（使用者即你自己），没有外部 API 消费者，因此**不按「是否向后兼容」分，而按「用户可感知的能力是否新增」分**：

- **PATCH**：改正 / 优化**已有**行为，没有新增用户可感知的能力。
  - bug 修复（popup 加载态、计算/缓存错误）
  - 视觉 / 文案微调（涨跌色值、间距、说明文字）
  - 性能 / 兜底逻辑改进（用户看不见机制变化，行为不变）
  - 内部重构（无用户可见变化）
- **MINOR**：新增用户可感知的能力，或一批相关改动收口成一个可命名的功能里程碑。
  - 新功能 / 新界面（指数 / 市场面板、持仓分组排序、新数据源选项）
  - 新设置项 / 新用户可控行为
- **MAJOR**：保留不用。未来若用，仅限破坏性变更（配置格式不兼容且无法自动迁移、数据存储结构重大变更、产品定位大改）。

**决策口诀**：打开后「能不能做一件之前做不到的事？」能 → MINOR；不能（只是之前能做的更对 / 更好 / 不崩）→ PATCH。**拿不准默认 PATCH**（保守），等一个功能分支整体做完、想给它一个里程碑时再 MINOR。

**MINOR / PATCH 由 AI agent 在 commit/push 时自行判定并 bump**（用户已授权 agent 拍板，无需用户逐次确认）。Agent 按本节的「用户可感知能力是否新增」标准判断：纯修复 / 优化 / 重构 → PATCH；新增用户可控能力 / 可命名功能里程碑 → MINOR；**版本号两端必须一起 +1（统一版本号）**。关键：bump 在「改动完成、准备 commit/push」时一次定，不中途纠结。

**本项目实例参照（分类，具体号随基线重置后重新计数）**：popup 加载态修复 / 涨跌色值微调 / 缓存 bug = PATCH；持仓分组排序、指数 / 市场面板、QDII 夜盘刷新 = MINOR。

### Bump 纪律（统一版本号，2026-08-24 起）
- **发布版本只在「改动完成、准备 commit/push」时 bump 一次，不在每次中间尝试时 bump。**
- **任何改动（chrome-only / tauri-only / 共享包）都两端同步 +1**：`apps/chrome/package.json` 与 Tauri 三处（`tauri.conf.json` / `Cargo.toml` / `Cargo.lock` 的 fund01-tauri 条目）必须全部改为同一新版本。
- 严禁「只 bump 一端」；提交前必须跑 `pnpm check:versions`（已实现两端相等校验）。

### 构建戳（build stamp，已实现）
- **发布版本不负责「我测的是不是刚编的最新版」——那由构建戳承担。**
- **内容字段**：git short SHA（7 位）+ 构建时间（本地 `YYYY-MM-DD HH:mm`）+ 分支名 + 工作区状态（干净 / 有未提交改动）。
- **实现（无需改 Rust / WindowPort）**：SHA 等由**前端构建脚本**在构建期捕获并注入 bundle——Chrome 与 Tauri 共用同一套：
  1. `scripts/build-info.mjs`：`getBuildDefines()` 用 git 捕获四个值，返回已 `JSON.stringify` 的 `source.define` 键值对（`__BUILD_SHA__` / `__BUILD_TIME__` / `__BUILD_BRANCH__` / `__BUILD_DIRTY__`）。
  2. `apps/chrome/rsbuild.config.ts` 与 `apps/tauri/rsbuild.config.ts`：在 `source.define` 接入 `getBuildDefines()`（注意：rsbuild define 直接文本替换 token，字符串值必须先 `JSON.stringify`，否则运行时 ReferenceError）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lingyired/fund01](https://github.com/lingyired/fund01) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
