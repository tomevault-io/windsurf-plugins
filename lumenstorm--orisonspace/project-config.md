---
trigger: always_on
description: AI 写作助手的 Agent 编排库。基于 agentic loop + tool calling 架构，作为 `@orison/desktop-agent` 包内嵌于桌面主进程，为创作工作区提供智能写作、工具调用、原生 skill 按需加载能力。
---

# Orison Agent

AI 写作助手的 Agent 编排库。基于 agentic loop + tool calling 架构，作为 `@orison/desktop-agent` 包内嵌于桌面主进程，为创作工作区提供智能写作、工具调用、原生 skill 按需加载能力。

## 架构概览

```
┌─────────────────────────────────────────────────────────┐
│  Desktop Shell (Electron main process)                  │
│                                                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │  Model Gateway (IPC)                            │    │
│  │  model:generate-text / -image                    │    │
│  └─────────────────────────────────────────────────┘    │
│         ▲                                               │
│         │ injected via setGenerateTextFn()               │
│         ▼                                               │
│  ┌─────────────────────────────────────────────────┐    │
│  │  @orison/desktop-agent (library)                │    │
│  │                                                 │    │
│  │  ┌──────────┐   ┌──────────────────┐           │    │
│  │  │  Workflow │──▶│  LLM Provider    │           │    │
│  │  │  Runtime  │   │  (IPC injection) │           │    │
│  │  └──────────┘   └──────────────────┘           │    │
│  │       │                                         │    │
│  │       ▼                                         │    │
│  │  ┌──────────┐                                   │    │
│  │  │  Tools   │                                   │    │
│  │  │  Registry│                                   │    │
│  │  └──────────┘                                   │    │
│  │       │                                         │    │
│  │       ▼                                         │    │
│  │  ┌──────────────────────────────────┐           │    │
│  │  │  Built-in Tools                  │           │    │
│  │  │  • Skills (metadata + loader)    │           │    │
│  │  │  • spawn_agent (subagent)        │           │    │
│  │  └──────────────────────────────────┘           │    │
│  │                                                 │    │
│  │  ┌──────────────────────────────────────────┐   │    │
│  │  │  Shell-provided Tools (via setExecuteToolFn) │    │
│  │  │  • File I/O (read/write/list)    │           │    │
│  │  │  • Search (regex across files)   │           │    │
│  │  │  • Story Memory (YAML)           │           │    │
│  │  │  • Chapters (list/read/write)    │           │    │
│  │  │  • Image Gen (generate/edit)     │           │    │
│  │  │  • Git (status/commit/log/diff)  │           │    │
│  │  └──────────────────────────────────┘           │    │
│  └─────────────────────────────────────────────────┘    │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │  Agent IPC Handlers (agentIpc.ts)                │   │
│  │  agent:create-session / stream-message / ...     │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
         ▲
         │ IPC (Electron)
         ▼
┌─────────────────────────────────────────────────────────┐
│  Frontend (renderer process)                            │
│  • Chat messages (text + tool calls + images)           │
│  • Tool execution progress                             │
│  • Skill settings & session history                    │
└─────────────────────────────────────────────────────────┘
```

## 集成方式

Agent 不再作为独立 HTTP 服务运行。它是一个 TypeScript 库，通过依赖注入集成到 Electron shell：

```typescript
import { createWorkflowRuntime, setGenerateTextFn, setExecuteToolFn } from '@orison/desktop-agent';

// 注入 LLM 调用能力
setGenerateTextFn(async (body, abort) => { /* shell model gateway */ });
setExecuteToolFn(async (toolId, params, ctx) => { /* shell tool handlers */ });

// 创建 runtime 实例
const runtime = createWorkflowRuntime();
```

通信通过 Electron IPC（`agent:*` 通道），不再需要 HTTP/SSE。

## IPC 通道

Agent 通过 Electron IPC 与渲染层通信（`agent:*` 通道）：

| 通道 | 说明 |
|------|------|
| `agent:create-session` | 创建会话 |
| `agent:get-session` | 获取会话状态 |
| `agent:set-session-model` | 设置会话使用的模型（modelRef，下一轮生效） |
| `agent:list-sessions` | 列出项目会话 |
| `agent:delete-session` | 删除会话 |
| `agent:stream-message` | 发送消息并启动流式执行 |
| `agent:resolve-confirmation` | 用户确认/拒绝工具调用 |
| `agent:list-skills` | 列出项目 skills |
| `agent:execute-skill` | 兼容入口：按名称加载 skill 内容 |
| `agent:list-continuations` | 列出会话 continuations |
| `agent:restore-continuation` | 恢复 continuation |
| `agent:abort-run` | 中止当前执行 |
| `agent:list-skill-packages` | 列出 skill 包及启用状态 |
| `agent:set-package-enabled` | 启用/禁用 skill 包 |
| `agent:set-skill-enabled` | 启用/禁用单个 skill |

### Stream 事件格式

通过 `agent:stream-event` IPC 事件推送到渲染层：

```json
{ "type": "assistant", "data": { "id": "...", "content": "...", "toolCalls": [...] } }
{ "type": "tool", "data": { "id": "...", "results": [...] } }
{ "type": "child", "data": { "source": "subagent", "role": "...", "depth": 1, "event": {...} } }
{ "type": "confirm_required", "data": { "sessionId": "...", "callId": "...", "name": "...", "input": {...} } }
{ "type": "compaction", "data": { "compactedCount": 5 } }
{ "type": "done", "data": { "status": "completed" } }
{ "type": "error", "data": { "message": "..." } }
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LumenStorm/OrisonSpace](https://github.com/LumenStorm/OrisonSpace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
