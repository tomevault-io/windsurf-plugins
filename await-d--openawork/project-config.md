---
trigger: always_on
description: **提交：** 7c73b44 | **分支：** main
---

# OpenAWork — 项目知识库

**生成时间：** 2026-03-21
**提交：** 7c73b44 | **分支：** main
**文档版本：** v2（含构建命令、测试命令、代码风格完整指引）

## 交互语言

- **所有对话必须使用中文**——包括回复、解释、提问、确认等一切交互内容，不得使用英文回复用户。

## 概述

跨平台 AI Agent 工作台：Fastify 网关 + React Web + Tauri 桌面端 + Expo 移动端。
技术栈：TypeScript（严格模式，NodeNext 模块），pnpm monorepo，Zod 校验，SQLite + Postgres + Redis。

## 目录结构

```
OpenAWork/
├── apps/
│   ├── web/          # React SPA（Vite），主要 UI
│   ├── desktop/      # Tauri v2 封装——通过相对导入直接复用 Web 页面
│   └── mobile/       # Expo Router（React Native），聊天 + 会话 + 设置
├── packages/
│   ├── agent-core/   # 核心 Agent 状态机、工具、会话、Provider 管理
│   ├── shared/       # 仅含消息/流类型——零业务逻辑
│   ├── shared-ui/    # 60+ React 组件，被所有应用使用
│   ├── multi-agent/  # 多 Agent 工作流 DAG 编排器
│   ├── skill-registry/ # 技能安装/生命周期/安全沙箱
│   ├── web-client/   # 浏览器端 WS + SSE 客户端及认证辅助
│   ├── platform-adapter/ # 平台路径解析（桌面 vs Web vs 移动端）
│   ├── mcp-client/   # MCP 协议客户端
│   ├── lsp-client/   # 网关 LSP 客户端
│   ├── pairing/      # 设备配对（二维码流程）
│   ├── browser-automation/ # 基于 Playwright 的浏览器工具
│   ├── logger/       # 结构化日志
│   ├── telemetry/    # Sentry + 分析
│   ├── artifacts/    # 产物存储与检索
│   └── skill-types/  # 共享技能类型定义
├── services/
│   └── agent-gateway/ # Fastify 5 HTTP/WS 服务器——后端
├── docs/             # 运行手册、技能开发指南、故障复盘模板
├── scripts/          # version.mjs、vite-plugin-version.mjs
└── .evidence/        # 参考实现（fastify、ioredis、postgres）——只读
```

## 查找指引

| 任务                | 位置                                                  |
| ------------------- | ----------------------------------------------------- |
| Agent 状态机        | `packages/agent-core/src/state-machine.ts`            |
| LLM Provider 配置   | `packages/agent-core/src/provider/`                   |
| 工具定义            | `packages/agent-core/src/tools/` + `tool-contract.ts` |
| 会话持久化          | `packages/agent-core/src/sqlite-session-store.ts`     |
| 多 Agent DAG        | `packages/multi-agent/src/dag.ts` + `orchestrator.ts` |
| 网关 HTTP 路由      | `services/agent-gateway/src/routes/`                  |
| 网关 WS 流式        | `services/agent-gateway/src/routes/stream.ts`         |
| 消息渠道            | `services/agent-gateway/src/channels/`                |
| 技能安装/安全       | `packages/skill-registry/src/`                        |
| 共享类型（消息/流） | `packages/shared/src/index.ts`                        |
| UI 组件             | `packages/shared-ui/src/`                             |
| Web 应用路由        | `apps/web/src/App.tsx`                                |
| 桌面 Tauri 命令     | `apps/desktop/src-tauri/src/`                         |
| 认证状态（Zustand） | `apps/web/src/stores/auth.ts`                         |
| 浏览器端网关客户端  | `packages/web-client/src/`                            |
| Docker 基础设施     | `docker-compose.yml`                                  |
| CI 流水线           | `.github/workflows/ci.yml`                            |
| Agent 路由逻辑      | `packages/agent-core/src/routing.ts`                  |

## 架构说明

- **桌面端复用 Web 页面**：`apps/desktop/src/App.tsx` 直接从 `../../web/src/pages/` 和 `../../web/src/stores/` 相对导入——非构建产物依赖，是直接 TS 导入。
- **网关作为桌面端 Sidecar**：通过 `bun build --compile` 编译为二进制，嵌入 Tauri 应用，路径为 `apps/desktop/src-tauri/sidecars/agent-gateway/`。
- **流式输出**：网关通过 SSE（`/stream` 路由）+ WebSocket 实现实时 Agent 输出。
- **消息渠道**：Telegram、Discord、飞书、钉钉、Slack 各自实现 `MessagingChannelService` 接口，位于 `services/agent-gateway/src/channels/`。
- **哈希锚定编辑**：自定义文件编辑工具（`packages/agent-core/src/tools/hash-edit.ts`）使用 SHA-256 行哈希替代行号，防止编辑漂移。
- **路由分级**：`packages/agent-core/src/routing.ts` 定义 R0–R3 路由分级（复杂度层级），用于 Agent 任务调度。
- **.evidence/**：fastify、ioredis、postgres 的只读参考源码，禁止编辑。

## 约定

### TypeScript

- `strict: true`、`noUncheckedIndexedAccess: true`、`noImplicitOverride: true`
- 模块系统：`NodeNext`（所有导入使用 `.js` 扩展名，即使源文件为 `.ts`）
- 强制使用 `consistent-type-imports`：纯类型导入必须使用 `import type { ... }`
- **禁止** `as any`、`@ts-ignore`、`@ts-expect-error`、空 catch 块、空函数——均视为错误

### 提交规范（husky + commitlint 强制执行）

- 格式：`type(scope): <中文描述>` — **scope 必填，描述必须以中文字符开头**
- 类型：feat | fix | docs | style | refactor | perf | test | build | chore | ci | revert | release
- 标题最大长度：100 字符
- 示例：`feat(gateway): 新增GitHub路由支持`
- scope 统一使用**小写**，优先采用模块 / 包 / 应用名（如 `gateway`、`web`、`shared-ui`、`agentdocs`）
- 正文 / 尾注可选；**禁止**出现任何 Sisyphus 协作尾注或相关协作痕迹
- 详细说明与示例见：`docs/commit-convention.md`

### 自动构建触发规则

- **默认普通提交不触发自动构建**：`feat / fix / docs / style / refactor / perf / test / build / chore / ci / revert` 提交只跑 CI（lint + typecheck + test），不会自动 bump 版本、不会自动打 tag、不会触发桌面端发布。
- **只有 `release(<scope>):` 提交会触发自动构建**：自上次 `desktop-v*` tag 以来，存在至少一条形如 `release(<scope>): <中文描述>` 的提交时，`auto-release.yml` 才会执行版本提升、打 tag 与触发 `release-desktop.yml`。
- **推荐写法**：
  - `release(all): 准备发布预览版`
  - `release(preview): 触发自动构建并发布桌面预览版`
  - `release(auto): 收口本轮迭代并触发自动发布`
- **优先级**：自动 release notes 摘要会优先取最近一条 `release(<scope>):` 提交的中文描述。
- **手动通道仍可用**：`prepare-release.yml`、`release-desktop.yml`、`release-mobile.yml` 的 `workflow_dispatch` 入口不受影响，可在 GitHub Actions 页面手动触发。
- **bot 自身的 `build(release): 自动提升版本到 vX` 不算触发提交**，不会引发递归构建。

### 代码风格（Prettier）

- 单引号（`singleQuote: true`）、分号（`semi: true`）、尾随逗号（`trailingComma: "all"`）
- 行宽 100（`printWidth: 100`）、2 空格缩进、空格括号（`bracketSpacing: true`）
- 箭头函数参数始终加括号（`arrowParens: "always"`）

### 包名规范

- 所有 workspace 包使用 `@openAwork/` scope
- 包内所有导出必须经过 `src/index.ts`——禁止消费者直接导入内部模块路径

### ESLint 范围与规则


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Await-d/OpenAWork](https://github.com/Await-d/OpenAWork) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
