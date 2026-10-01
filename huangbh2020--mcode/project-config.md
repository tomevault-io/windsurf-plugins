---
trigger: always_on
description: 本文件指导 AI agent(含本项目自身用 Claude Code 开发时)如何理解并参与 Mcode 的开发。先读本文,再动手。
---

# AGENTS.md

本文件指导 AI agent(含本项目自身用 Claude Code 开发时)如何理解并参与 Mcode 的开发。先读本文,再动手。

---

## 项目是什么

**Mcode**(*my* Code)- 基于 Claude Agent SDK 构建的**桌面端 GUI**(Electron 三栏 IDE)。

核心理念:**不重新实现 agent,只做 Claude 的交互界面**。通过 Agent SDK 驱动 claude agent loop;本应用负责会话管理、实时渲染、工具审批、IDE 能力(文件/git/终端)。

- 使用 `@anthropic-ai/claude-agent-sdk`,内部管理 claude 二进制(项目不直接 spawn)
- 项目 MIT,可独立开源
- 架构受 [Synara](https://github.com/Emanuele-web04/synara) 启发,但用主流 TS 重写(无 effect-ts、无 bun)
- 内置 `AgentProvider` 抽象层,后续可扩展其他 agent 平台(OpenAI Codex、Gemini CLI 等)

---

## 权威文档(动手前必读)

| 主题 | 文档 |
|------|------|
| 技术栈、架构、踩坑记录 | [`docs/tech-stack.md`](docs/tech-stack.md) |
| claude stream-json 数据格式(旧 CLI 方式的 dump 记录,SDK 的 SDKMessage 与此对应) | [`docs/claude-stream-json.md`](docs/claude-stream-json.md) |
| Pi SDK 接入记录 | [`docs/pi-sdk-integration.md`](docs/pi-sdk-integration.md) |
| Claude Agent SDK 参考 | https://code.claude.com/docs/en/agent-sdk |

改 `SdkMessageAdapter` 或涉及 SDK 输出解析时,**必须**先读 stream-json 文档——SDK 的 `SDKMessage` 类型本质上是对 CLI stream-json 的类型化封装,字段语义一一对应。

---

## 进程架构(三进程)

```
Renderer (React 19, contextIsolation:true, nodeIntegration:false)
        ↕  Electron IPC(preload contextBridge + zod 校验)
Main (Node.js)
  ├── RuntimeManager      持 ProviderRegistry,构造 ProviderContext
  │     └── AgentProvider  ClaudeAgentSdkProvider(→ query() → SDKMessage → RuntimeEvent)
  ├── SessionManager      会话生命周期(SQLite via better-sqlite3)
  └── IDE Services        terminal / git / checkpoint(P4)
        ↕  @anthropic-ai/claude-agent-sdk (query)
     claude 二进制(SDK 内打包,项目不直接 spawn)
```

**安全边界**:renderer 不能 `require()` 任何 Node 模块。通往 Node 的唯一桥梁是 preload 暴露的 `window.api`,所有消息经 zod 校验后才放行。新增 IPC 通道时,必须在 `packages/contracts/src/ipc.ts` 定义 schema + 通道常量,并在 preload 白名单注册。

---

## 目录地图

```
packages/contracts/src/        # 跨进程共享(无运行时逻辑)
  runtime.ts                   # RuntimeEvent 联合 — provider 中立的归一化事件
  session.ts                   # Project / Session / Message 领域类型
  ipc.ts                       # zod schema + IPC 通道常量 + RPC 类型表
  provider.ts                  # AgentProvider 接口 / ProviderContext / TurnHandle

apps/desktop/src/
  main/                        # 主进程
    claude/
      RuntimeManager.ts        # ★ 会话↔provider 映射,构造 ProviderContext
      ApprovalBridge.ts        # 工具审批/AskUserQuestion 的 IPC 异步桥
    providers/
      registry.ts              # ProviderRegistry 单例(启动时注册所有 provider)
      claude-sdk/
        ClaudeAgentSdkProvider.ts  # AgentProvider 实现(query() 包装 + canUseTool 桥)
        SdkMessageAdapter.ts       # ★ SDKMessage → RuntimeEvent 归一化(改前读 SDK 文档)
      pi-sdk/
        PiAgentSdkProvider.ts      # AgentProvider 实现(createAgentSession 包装 + 内联 Extension 注入)
        mcodeExtension.ts          # ★ 内联 Pi Extension:tool_call 权限/路径守卫 + AskUserQuestion 工具 + system prompt
        PiMessageAdapter.ts        # Pi SDK 事件 → RuntimeEvent 归一化
    ipc/{claude,projects}.ts   # IPC handler
    lib/
      logger.ts                # 文件+stderr 日志(userData/logs/main.log)
      askQuestion.ts           # ★ 共享:parseQuestions / formatAnswersForModel / ASK_SYSTEM_PROMPT(Claude + Pi 共用)
    store/{db,repositories}.ts # SQLite 持久化(better-sqlite3,2026-09-14 从 sql.js 迁移)
  preload/index.ts             # contextBridge 白名单 API
  renderer/                    # 前端(React)
    stores/sessionStore.ts     # ★ Zustand store,ingest RuntimeEvent → ChatMessage(locale 状态也在此)
    lib/i18n/                  # ★ 中英双语文案:core.ts(translate) + index.ts(useI18n) + zh|en/{common,layout,lib,chat-stream,chat-composer,ide,browser,settings,store}.ts
    hooks/useClaudeEvents.ts   # 订阅 IPC 事件流
    components/{layout,chat}/  # UI(chat/ 下 ActivityCluster + ActivityConsole + activityShared = 活动区)
```

---

## 开发命令

```bash
# 启动开发(electron-vite,HMR)
cd D:\00-huangbh-project\my-claude-gui
pnpm dev

# 类型检查(改完代码先跑这个,最快定位问题)
cd apps/desktop && npx tsc --noEmit -p tsconfig.json

# 构建
pnpm build
```

### ⚠️ 启动前注意
异常退出后,5173 端口可能残留(TIME_WAIT)。若窗口没弹出,先在任务管理器结束所有 `electron.exe`,或等约 30 秒端口释放。

---

## 环境

- Node.js ≥ 22.13(pnpm 11 要求)
- **Electron 37.10.3(2026-09-24 从 33 升级;内置 Node 22.21.1)**:主进程 Node 跟随 Electron 大版本,不能单独升——pi 0.87.x 需要的 `node:fs.globSync`/`markAsUncloneable`(Node ≥ 22.14)自此原生可用,`polyfillWorkerThreads` 自动变 no-op。升级要点:better-sqlite3 的 postinstall 钩子动态按新 Electron 版本拉 ABI 预构建(npmmirror 连 prebuild 都镜像,零手工);node-pty 1.1.0 自带 NAPI prebuilds,跨 ABI 免重建;electron-vite 2.3/electron-builder 25 无需动。注意 `^37.0.0` 曾被解析到 37.0.0 基线版,写 `"^37.10.3"` 显式下限
- pnpm ≥ 9(经 `corepack enable` 启用,本机 11.16.0)
- Claude Code CLI(本机装在 `D:\soft\nodejs\node_global`,非默认路径——`ClaudePathResolver` 已处理)
- `.npmrc` 配了国内 electron 镜像(直连 GitHub 会超时),任何人重装不会踩
- **`@anthropic-ai/claude-agent-sdk` 钉死精确版本 `0.3.238`(不带 `^`;2026-08-27 从 0.3.218 显式升级)**:防止 `^0.3.x` 在普通 `pnpm install` 时静默漂移(2026-08-23 曾意外漂到本版)。注意:本版捆绑 CLI 2.1.238 的 `sdkCompat.testedWrapperVersions` 名单止于 0.3.227、不含 wrapper 自身(该字段仅宿主元数据,`sdk.mjs` 不消费它);本次升级已过 checksum 比对 + 对话框 kind 存在性 + 冒烟(system/init 报 2.1.238)三项验证。升级要显式改版本号,升级前必查:changelog + issues 搜 "Stream closed"/permission;新包 manifest 的 testedWrapperVersions 要含 wrapper 自身;`grep -ac "permission_exit_plan_mode_v2" claude.exe` 确认对话框 kind 没改名;升级后回归计划审批/AskUserQuestion/工具审批/子代理收尾四条链路

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huangbh2020/mcode](https://github.com/huangbh2020/mcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
