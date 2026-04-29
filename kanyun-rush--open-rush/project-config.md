---
trigger: always_on
description: AI coding agents（Claude Code, Cursor 等）在本仓库工作时的完整指南。
---

# AGENTS.md

AI coding agents（Claude Code, Cursor 等）在本仓库工作时的完整指南。

## Project Overview

OpenRush 是企业级 AI Agent 基础设施平台——自托管、多场景、为每一位团队成员而建。核心理念：**AI Agent 成为主要交互界面**，底层有正确的基础设施支撑。

核心特性：

- Claude Code 原生执行（Anthropic API / AWS Bedrock / 自定义端点）
- 可插拔沙箱隔离（OpenSandbox 默认，可切换 E2B / Docker）
- 双层凭据管理（Platform Vault + User Vault，env 注入沙箱）
- 15 状态 Run 状态机 + 断线恢复 + 断点续跑
- Redis-backed 可恢复 SSE 流（双层：agent→control, control→browser）
- Skills & MCP 插件生态
- 跨会话 Agent 记忆（pgvector）

## 三层架构

```
浏览器
  │ ← SSE② 流式返回
  ▼
┌─ apps/web（Next.js 16）──────────────────────────────────┐
│  用户界面 + 控制面 API + SSE 端点                          │
│                                                            │
│  职责：                                                     │
│  • 用户界面渲染（React 19）                                 │
│  • Control API（创建 Run、查询状态、SSE② 端点）            │
│  • 项目/用户 CRUD（直接操作 DB）                           │
│  • NextAuth.js v5 认证                                     │
│                                                            │
│  不做：                                                     │
│  ✗ 不执行 AI 模型调用                                      │
│  ✗ 不操作沙箱文件系统                                      │
│  ✗ 不处理 Agent 运行时逻辑                                 │
└───────────┬────────────────────────────────────────────────┘
            │ 入队 pg-boss job
            ▼
┌─ apps/control-worker（Node.js + pg-boss）──────────────────┐
│  任务编排 + 状态机驱动                                       │
│                                                              │
│  职责：                                                       │
│  • 消费 pg-boss 队列（run:execute, run:finalize）            │
│  • 驱动 RunStateMachine（15 状态转换）                       │
│  • 沙箱生命周期管理（通过 SandboxProvider）                  │
│  • Agent Bridge（SSE① 消费 + 事件持久化）                   │
│  • Finalization（workspace snapshot + PR + checkpoint）      │
│  • 定时恢复卡住的 Run（run:recover 每 2 分钟）              │
│                                                              │
│  不做：                                                       │
│  ✗ 不直接处理 HTTP 请求                                      │
│  ✗ 不渲染 UI                                                 │
└───────────┬──────────────────────────────────────────────────┘
            │ HTTP + SSE①（通过 SandboxProvider）
            ▼
┌─ apps/agent-worker（Hono :8787，在沙箱容器内）──────────────┐
│  AI 执行环境                                                  │
│                                                                │
│  职责：                                                         │
│  • 接收 prompt，调用 Claude Code                               │
│  • 工具执行（Bash, Read, Write, Edit 等）                     │
│  • SSE① 流式输出 UIMessageChunk                              │
│  • 断点恢复（接收 checkpoint，重建上下文）                     │
│  • 工作区文件系统操作                                          │
│                                                                │
│  约束：                                                         │
│  • 无状态（恢复数据来自 DB/OSS，不依赖进程内存）              │
│  • 凭据通过 env 注入，不持有明文密钥                          │
└────────────────────────────────────────────────────────────────┘
```

### 改代码时怎么判断改哪个？


| 你要改的东西                     | 改 apps/web | 改 apps/control-worker | 改 apps/agent-worker |
| -------------------------- | ---------- | --------------------- | ------------------- |
| 页面 UI、组件、样式                | ✅          |                       |                     |
| 用户认证（NextAuth）             | ✅          |                       |                     |
| 项目/用户 CRUD API             | ✅          |                       |                     |
| SSE② 端点（browser ← control） | ✅          |                       |                     |
| Run 状态机转换逻辑                |            | ✅                     |                     |
| 沙箱创建/销毁                    |            | ✅                     |                     |
| Finalization（PR 创建、产物上传）   |            | ✅                     |                     |
| pg-boss 队列处理               |            | ✅                     |                     |
| AI 对话、Prompt 执行            |            |                       | ✅                   |
| 工具调用（Bash、文件操作）            |            |                       | ✅                   |
| SSE① 流式输出                  |            |                       | ✅                   |
| 断点恢复（restore）              |            | ✅ + ✅                 |                     |



| 你要改的东西                             | 改哪个 package            |
| ---------------------------------- | ---------------------- |
| Zod schema、枚举、状态机                  | packages/contracts     |
| 数据库表结构、migration                   | packages/db            |
| RunService、AgentService、EventStore | packages/control-plane |
| SandboxProvider 接口/实现              | packages/sandbox       |
| AI Provider 抽象                     | packages/agent-runtime |
| Redis SSE 流                        | packages/stream        |
| 外部服务集成（Git、OSS）                    | packages/integrations  |
| Skill 安装/管理                        | packages/skills        |
| MCP server/client                  | packages/mcp           |
| 跨会话记忆                              | packages/memory        |


## Monorepo Structure

```
open-rush/
├── apps/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kanyun-rush/open-rush](https://github.com/kanyun-rush/open-rush) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
