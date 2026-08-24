---
trigger: always_on
description: > 面向在本仓库工作的 AI agent 与人类协作者。**完整架构/测试矩阵/限制清单见 [DEVELOPERS.md](DEVELOPERS.md)；用户文档见 [README.md](README.md)；UI / 视觉设计 System 的唯一权威见 [DESIGN.md](DESIGN.md)。**
---

# AGENTS.md — DSH Config Manager 仓库协作指南

> 面向在本仓库工作的 AI agent 与人类协作者。**完整架构/测试矩阵/限制清单见 [DEVELOPERS.md](DEVELOPERS.md)；用户文档见 [README.md](README.md)；UI / 视觉设计 System 的唯一权威见 [DESIGN.md](DESIGN.md)。**
> 本文写「做改动前必须知道」的注意点——隐性约定与坑，避免重复造轮子或踩雷。

## 🗣️ 语言与交流

- 与用户交流一律用**中文**。
- 代码注释、commit message、文档以中文为主（与技术术语混排，如 `SyncEngine`、`adapter`）。

## 📦 项目概览

- **用途**：DSH（DeepSeek Harness）配置的备份 / 导出 / 导入 / 迁移 / 远程同步 / 配置市场插件，双面 Cordis 插件。
- **技术栈**：TypeScript 5.9（strict + `verbatimModuleSyntax` + `noUncheckedIndexedAccess`）、Node.js ≥ 22（host 半）、React 18 + CSS Modules（web 半）、`node:test` 零依赖测试、tsdown + lightningcss 打包 client。
- **样式方案**：**CSS Modules（唯一样式表 `src/client/config-manager.module.css`）**，颜色/字体/阴影全部消费 DSH Design System 的 `--dsw-*` CSS 变量，**不引入 Tailwind / CSS-in-JS / Sass / UI 组件库 / 图标库**。
- **不进 node_modules 也能预判结构**：`src/{core,schema,security,adapters,ui,client,sync,market,profiles,migrations,utils}` 分层清晰，见下。

## 🗂️ 仓库结构与职责

```
src/
├── index.ts          host 半 Cordis 入口（name='config-manager'；/api/dsh-config-manager/* 路由，约 2800 行）
├── core/             核心引擎：exporter / importer / restore / rollback / run-registry / plugin-cli，与 DSH 运行时解耦（ConfigAdapter/HostContext 接口 + 内存 mock）
├── schema/           领域类型 / Manifest / 版本判定（CURRENT_SCHEMA_VERSION=1）
├── security/         secret-scanner / redaction / zip-security / integrity / encryption（scrypt + AES-256-GCM）
├── adapters/         13 个真实配置适配器（settings/ui/providers/plugins/mcp/prompts/skills/agentPresets/workspaces/credentials/pluginFiles/sessions/self）
├── sync/             同步体系：SyncEngine + GitTransport/WebDavTransport + AutoSyncScheduler + 配置/状态/历史/sync-selection
├── market/           配置市场：GitMarketReader + index-parser + security 校验 + builtin 内置市场；「一键上传/我的配置」：GitHub REST 薄客户端（github-repos.ts）+ 上传编排（my-repo.ts）+ 通用 git 写文件器（git-file-writer.ts）
├── migrations/       schema 迁移链（registry + v1→v2 占位）
├── profiles/         ProfileManager（保存/切换带 Preview+快照+回滚）
├── ui/               框架无关 UI 逻辑层（纯函数 + 控制器，无 React，node 可测）—— 见下「UI 分层」
├── utils/            通用工具（paths/zip/hashing/json/logger）
└── client/           React 界面（浏览器半）—— 见下「页面与组件落位」
tests/                集成式测试（node --test）
docs/                 设计文档（docs/design/，现有 UI 与市场发布设计）
```

### UI 分层（改动前必读，避免放错层）

**任何 UI 改动有两条铁律：**
1. **框架无关逻辑放 `src/ui/`**（纯函数/控制器：`export-flow.ts`、`import-wizard.ts`、`conflict-view.ts`、`path-mapping.ts`、`report.ts`、`errors.ts`、`progress.ts`、`flow.ts`、`types.ts`、`i18n.ts`）——**禁止在 React 组件里写可测试的业务逻辑**；
2. **React 壳只做装配**（`src/client/` 的组件只负责渲染 + 交互状态，把渲染模型/控制器来自 `src/ui/` 的纯函数）。

### 页面与组件落位（`src/client/`）

| 内容 | 位置 |
|---|---|
| 设置页入口 / 五 tab 容器 | `src/client/index.ts` + `src/client/ConfigManagerSection.tsx`（Export / Import / Snapshots / Sync / Market 五 tab） |
| 导出视图 | `src/client/export/ExportView.tsx` |
| 导入九步向导 | `src/client/import/ImportWizardView.tsx`（阶段子页：`ConflictList.tsx`、`PathMappingForm.tsx`、`import-file-select.ts`） |
| 快照恢复 | `src/client/snapshots/SnapshotsPanel.tsx` |
| 远程同步 | `src/client/sync/SyncSettingsView.tsx`（+ `SyncConfirmView.tsx`、`SyncHistoryView.tsx`、`sync-view.ts` 纯函数模型、`history-model.ts`） |
| 配置市场 | `src/client/market/MarketPanel.tsx`（+ `market-view.ts` 纯函数模型）；「我的配置」子视图 `src/client/market/MyConfigsView.tsx`（+ `my-configs-view.ts` 纯渲染模型、`my-configs-api.ts`，登录复用 sync device flow） |
| **共享 UI 原语** | **`src/client/common/ui.tsx`（Button/Badge/Banner/Card/Spinner/Field/SectionTitle/Empty/Checkbox）+ `common/ErrorBanner.tsx`、`common/ProgressBar.tsx`、`common/ReportView.tsx`** |
| 状态中枢 | `src/client/run-store.ts`（模块级单例 + sessionStorage，敏感字段白名单剔除） |
| 数据访问 | `src/client/api.ts`、`sync/sync-api.ts`、`market/market-api.ts`（类型化 fetch 封装，实现 `src/ui/types.ts` 的 port 契约） |
| 文案字典 | `src/client/locales.ts`（ns `config-manager`）、`sync/sync-locales.ts`（ns `config-manager-sync`）、`market/market-locales.ts`（ns `config-manager-market`），zh 源 / en 镜像 |
| **全部样式** | **`src/client/config-manager.module.css`（唯一样式表，类名经 CSS Modules 哈希）** |

- **Hook 放哪里**：本仓库不使用自定义 hooks 目录——React 组件直接内联 state + `useSyncExternalStore` 消费 `runStore`；复用逻辑一律下沉到 `src/ui/` 纯函数。**不要新造 hooks 层**。
- **Type 放哪里**：领域类型在 `src/core/types.ts` / `src/schema/types.ts` / `src/sync/*.ts` / `src/market/types.ts`；UI 层类型在 `src/ui/types.ts`；client 专属类型在 `src/client/client-types.ts`。
- **Utility 放哪里**：`src/utils/`（跨层通用）或模块内私有函数（单文件用）。带业务语义的纯函数优先 `src/ui/`。

## 🔢 版本号：三处必须同步（最容易漏）

发版时 `version` 同时存在于三处，**漏改任何一处都会产生不一致**（历史上出现过 package.json=0.1.30 而 PLUGIN_VERSION=0.1.28、lockfile=0.1.26 的漂移，0.1.31 起已修复同步）：

1. `package.json` → `"version"`
2. `src/index.ts` → `const PLUGIN_VERSION`（约 L124，注释声明 "kept in sync with package.json"）
3. `package-lock.json` → 根对象 `"version"`（约 L3 **和** `packages[""]` 的 `"version"` 约 L9，两处都要改）

bump 后跑 `npm run typecheck` 即可确认无引用遗漏。

## 🚀 发布流程（打 tag 即全自动）

CI：`.github/workflows/publish.yml`，tag `v*` push 触发全自动流水线：

```
typecheck → npm test → build → npm pack → npm publish（Trusted Publishing/OIDC）→ 创建 GitHub Release
```

发版步骤（按序）：

1. **bump 三处版本号**（见上）
2. **在 `CHANGELOG.md` 顶部新增当前版本的双语亮点段**：release 描述由 CI 从 `CHANGELOG.md` 抽取当前版本段（`.github/scripts/extract-release-notes.py`）拼接到 GitHub Release，**漏写会 CI fail fast**（不会静默发出亮点缺失的 release）
3. 提交 + `git push origin main`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xiajiajun516/dsh-config-manager](https://github.com/xiajiajun516/dsh-config-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
