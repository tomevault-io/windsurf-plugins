---
trigger: always_on
description: 本文件是智能体进入 FileTerm 仓库时的入口地图，不是完整手册。详细事实以 `docs/` 为准；当代码、设计或计划变化时，优先更新对应文档，不要把所有知识继续堆进这里。
---

# FileTerm Agent Guide

本文件是智能体进入 FileTerm 仓库时的入口地图，不是完整手册。详细事实以 `docs/` 为准；当代码、设计或计划变化时，优先更新对应文档，不要把所有知识继续堆进这里。

## 1. 项目定位

FileTerm 是面向开发者与运维场景的桌面远程工作台，技术栈以 Electron + React + TypeScript 为主，围绕 `SSH / SFTP / FTP` 构建可日常使用的多标签桌面客户端。

当前阶段：**骨架 + 核心链路打通 + 分层整理**。质量门禁（ESLint/Prettier/Husky/CI 测试）已落地，四项门禁全绿；App.tsx 从 3898 行收敛至 1337 行（-66%），已拆出 7 个 hooks + ModalPortalManager + ErrorBoundary。

## 2. 先读哪里

- 架构地图：`docs/architecture.md`
- 设计规范：`docs/design.md`
- 路线图：`docs/roadmap.md`
- 进行中计划：`docs/plans/active/`
- 已完成计划：`docs/plans/completed/`
- 架构决策：`docs/decisions/`
- 质量与回归：`docs/quality/`
- 已隐藏功能：`docs/hidden-features.md`
- 功能草案：`.agents/extensions/`
- 项目技能：`.agents/skills/`

如果任务只改一个小点，先读本文件和相关源码即可；如果任务跨 `main / preload / renderer / packages` 多层，必须先看 `docs/architecture.md` 和 `docs/plans/active/`。

## 3. 硬性边界

### 架构边界

- `packages/core` 是领域模型的 single source of truth。
- Renderer 不直接访问 SSH / SFTP / FTP protocol clients。
- 所有系统能力必须走 `main -> preload -> renderer`。
- SSH/SFTP 与 FTP 在 controller/protocol 层保持分离，不做伪统一。
- Transfer 进度统一进入 transfer system（`main/services/transfers/`），不在组件里零散维护。
- 会话事件通过 `WorkspaceSessionRuntime` 的全局 Event Emitter 统一分发，不分散监听 controller。
- 新状态优先进入 `packages/core` 定义类型，再下沉到服务层和 UI。
- 新窗口能力先定义 IPC 边界，再做 renderer 交互。
- 主题样式优先走 `token -> theme vars -> component skins -> terminal colors`。

### 平台兼容边界

- **CWD 目录跟随**：终端工作目录 (CWD) 变化通过底层会话流安全捕获，经 runtime 广播同步给文件管理器，严禁 UI 层轮询或直接探测平台路径。
- **POSIX CWD 注入门控**：`supportsPosixShellSetup()` 仅对 `linux` / `busybox` 返回 true。Windows / unknown 平台**严禁注入** Linux shell CWD 脚本，采用 fail-closed 双重门控（`detectPlatformAndSetupShell` + `injectShellSetup` 各一道）。
- **CRLF 归一化**：系统指标解析入口必须对远端输出做 `replace(/\r\n?/g, '\n')` 归一化，避免 `'windows\r'` 等污染导致平台误判。
- **Sudo 与 Root 状态同步**：终端执行 `sudo` 或切换用户态需被底层 runtime 解析，双向同步到文件管理器权限模型。

### 资源与安全边界

- **离线资源就地化**：所有图标、字体与基础样式资源预置在代码库中打包输出，严禁运行时动态拉取外部 CDN 资源。
- **macOS 钥匙串规避**：禁用 safeStorage，用品牌重命名等替代机制存储凭据，避免触发 macOS 系统安全弹窗。
- 连接的 `group`（文件夹名）和 `parentId`（文件夹 ID）必须双向同步，存储层负责自愈。

## 4. 代码位置

- Electron main process：`apps/desktop/src/main`
- IPC 注册：`apps/desktop/src/main/ipc/`（按领域拆分）
- 业务服务层：`apps/desktop/src/main/services`
- Transfer 服务：`apps/desktop/src/main/services/transfers/`
- 会话运行时：`apps/desktop/src/main/services/workspace/workspace-session-runtime.ts`
- 系统指标采集：`apps/desktop/src/main/services/sessions/system-metrics/`（按平台拆分：linux / busybox / windows collector + parser + platform-probe）
- Preload 安全 API：`apps/desktop/src/preload/preload.cts`
- React renderer：`apps/desktop/src/renderer`
- 主入口：`apps/desktop/src/renderer/App.tsx`（1337 行，已拆出 7 个 hooks + 2 个 layout 组件）
- Hooks 目录：`apps/desktop/src/renderer/hooks/`
  - `useWorkspaceTabs.ts`、`useWorkspaceModals.ts`、`useFileOperations.ts`
  - `useSshInteractions.ts`、`useFileEditor.ts`、`useWorkspaceIpcSync.ts`
  - `useWorkspaceDataOps.ts`
- Layout 组件：`apps/desktop/src/renderer/features/layout/`（ModalPortalManager / StandaloneWindowFrame / WindowMenubar / TabBar）
- ErrorBoundary：`apps/desktop/src/renderer/features/common/ErrorBoundary.tsx`
- 主工作区：`apps/desktop/src/renderer/features/workspace/HomeWorkspace.tsx`
- 终端组件：`apps/desktop/src/renderer/components/TerminalView.tsx`
- 主题样式：`apps/desktop/src/renderer/styles/themes/`
- 领域类型：`packages/core`
- 存储抽象：`packages/storage`
- 共享常量：`packages/shared`

## 5. 当前侧边栏布局

侧边栏导航顺序：**概览 → 连接管理器 → 命令管理器 → 设置**

已从 UI 隐藏但代码保留的功能见 `docs/hidden-features.md`，包括：

- 快速连接（Quick Connect）侧边栏入口
- Docs 侧边栏入口
- 页脚 Changelog / API Reference / Status 导航
- 页脚 System Latency 文字

## 6. 当前热点

这些文件功能集中，改动前要格外注意边界：

- `apps/desktop/src/main/ipc/`
- `apps/desktop/src/main/services/workspace-service.ts`（已是 façade 薄委托）
- `apps/desktop/src/main/services/transfers/transfer-service.ts`
- `apps/desktop/src/main/services/file-profile-repository.ts`
- `apps/desktop/src/main/services/sessions/ssh-session-controller.ts`（as any 已清零）
- `apps/desktop/src/main/services/sessions/system-metrics/`（多平台采集 + CRLF 归一化）
- `apps/desktop/src/main/services/workspace/workspace-session-runtime.ts`
- `apps/desktop/src/renderer/App.tsx`（1337 行，hooks 编排 + 布局组合）
- `apps/desktop/src/renderer/hooks/`（8 个 hooks，复杂逻辑集中在 useWorkspaceTabs / useFileOperations）
- `apps/desktop/src/renderer/features/workspace/HomeWorkspace.tsx`
- `apps/desktop/src/renderer/features/layout/ModalPortalManager.tsx`（全局模态框统一挂载）
- `apps/desktop/src/renderer/styles/themes/`

## 7. 质量门禁（已落地）

所有代码改动必须通过以下门禁，pre-push 自动阻断不通过项：

| 门禁     | 命令                                                   | 状态                                          |
| -------- | ------------------------------------------------------ | --------------------------------------------- |
| 类型检查 | `npm run typecheck`                                    | 4 workspace 全过                              |
| 静态检查 | `npm run lint --max-warnings=0`                        | 零 error 零 warning                           |
| 格式检查 | `npm run format:check`                                 | All files Prettier                            |
| 单元测试 | `npm test -w @fileterm/desktop`                        | 53/53 pass（unit 47 + controllers 6）         |
| 协议测试 | `npm run test:transfers:protocol -w @fileterm/desktop` | 6/6 pass（需先 build core + main，CI 单独跑） |

提交门禁：

- **pre-commit**（`.husky/pre-commit`）：`npx lint-staged` — 对暂存 `.ts/.tsx` 文件执行 prettier + eslint --fix

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [St0ff3l/fileterm](https://github.com/St0ff3l/fileterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
