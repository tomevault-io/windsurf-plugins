---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

**claude-code-java** 是一个可嵌入任何 Java 应用的 AI Agent 引擎，支持 CLI、REST API、Web Playground 三种交互方式。

**模块结构**：
- `claude-code-java-service` — 纯 Java 17 库，包含所有 Agent 能力，无框架依赖
- `claude-code-java-start` — Spring Boot 3.2 应用层，提供 CLI REPL、REST API 和 Web Playground

## 构建与运行

**配置**（首次运行前必须完成）：
```bash
# 编辑 claude-code-java-start/src/main/resources/claude.properties
# 将 OPENAI_API_KEY=your_api_key_here 替换为真实的 API Key
# 或设置环境变量：export OPENAI_API_KEY=<your-key>
```

**编译**：
```bash
mvn compile
```

**Web Playground + REST API 模式**（推荐，端口 8080）：
```bash
mvn exec:java -pl claude-code-java-start \
  -Dexec.mainClass="ai.claude.code.Application"
# 访问 http://localhost:8080
```

**CLI 交互模式**（REPL）：
```bash
mvn exec:java -pl claude-code-java-start \
  -Dexec.mainClass="ai.claude.code.Application" \
  -Dspring.profiles.active=cli
```

**调试**：在 `claude.properties` 中设置 `DEBUG_PRINT_PAYLOAD=true` 可打印完整的 API 请求/响应报文。

## 架构

### 模块职责

| 模块 | 定位 | 特征 |
|------|------|------|
| `claude-code-java-service` | 纯 Java 17 库 | 无框架依赖，可被任何 Java 项目引用 |
| `claude-code-java-start` | Spring Boot 3.2 应用 | CLI + REST API + Web Playground（单页应用） |

### service 模块包结构

```
src/main/java/ai/claude/code/
├── core/        — OpenAiClient, AnthropicClient, ClientFactory, BaseTools,
│                  SecurityUtils, ShellUtils, ToolHandler, AgentEventListener
├── capability/  — TodoManager, ContextCompactor, BackgroundRunner,
│                  TaskStore, WorktreeManager, SkillLoader, MessageBus,
│                  TeammateRunner, SessionStore, TeamProtocol, TaskPoller
├── tool/        — ToolProvider, ToolUtils,
│                  FileTools, TodoTool, SkillTool, CompactTool,
│                  TaskTools, BackgroundTools, TeamTools, WorktreeTools
└── agent/
    ├── AgentLoop.java       — 核心 while 循环（构造器注入）
    ├── TeammateLoop.java    — Teammate 独立 LLM 循环（Runnable，含空闲轮询）
    ├── AgentAssembler.java  — 组装工厂（初始化 capability，返回 AgentLoop）
    └── SlashRouter.java     — /help /tasks /compact /team /worktree 路由
```

### start 模块包结构

```
src/main/java/ai/claude/code/
├── Application.java         — @SpringBootApplication 统一入口
├── cli/
│   └── CliRunner.java       — @Profile("cli") CLI REPL
├── web/
│   ├── controller/
│   │   ├── ChatController.java   — POST /api/chat（同步）
│   │   │                           GET  /api/sessions（列表 + teammate 数量）
│   │   │                           GET  /api/sessions/{id}/messages
│   │   │                           GET  /api/sessions/{id}/teammates
│   │   │                           GET  /api/transcripts/{filename}
│   │   │                           DELETE /api/sessions/{id}
│   │   └── StreamController.java — POST /api/chat/stream（SSE 流式）
│   ├── dto/
│   │   ├── ChatRequest.java      — record { sessionId, message }
│   │   ├── ChatResponse.java     — record { sessionId, reply }
│   │   └── SessionMeta.java      — record { sessionId, preview, updatedAt, teammateCount }
│   └── service/
│       ├── ChatService.java      — 同步会话管理 + AgentLoop 调用
│       └── StreamService.java    — SSE 流式：注册 AgentEventListener → SseEmitter 推送
└── config/
    └── AgentBeans.java      — Spring @Bean 配置，调用 AgentAssembler.build()
src/main/resources/
├── static/index.html        — Web Playground 单页应用（无构建步骤，直接修改生效）
└── claude.properties        — API Key、模型、调试开关
```

### 核心设计模式

**AgentLoop**：核心 while 循环。`stop_reason == "tool_calls"` 时继续，`"end_turn"` 时退出。`messages` 数组跨轮次共享，实现多轮对话。

**AgentEventListener**：事件监听接口（`core/` 包），定义 Agent 执行周期内所有关键节点的回调：
- `onThinkingStart/End`、`onTextDelta`（流式文字）
- `onToolStart/End/Error`（工具执行）
- `onTeamToolStart/End/Text/Done`（Teammate 工具事件转发）
- `onCompactDone(summary, transcriptFile)`（压缩完成，含 transcript 文件名）

`StreamService.SseAgentEventListener` 实现此接口，将每个事件序列化为 SSE 帧推送给浏览器。CLI 模式通过另一个实现打印到 stdout。同一个 AgentLoop 通过切换 listener 实现不同输出模式。

**AgentAssembler**：工厂方法，初始化所有 capability，组装 ToolProvider，返回配置好的 AgentLoop。提供两个 `build()` 重载：一个自动创建所有依赖，另一个接受外部预创建的 capability（供 Spring 注入）。

**ToolProvider 模式**：每个工具类实现 `ToolProvider` 接口：
- `handlers()` — 工具名 → 处理器函数的 Map
- `definitions()` — LLM 可调用的 JSON schema 列表

新增工具：新建实现 `ToolProvider` 的类，在 `AgentAssembler.buildProviders()` 中加一行。

**TeammateRunner + TeammateLoop**：Agent Teams 核心。`TeammateRunner` 管理 Teammate 线程生命周期（spawn/list/shutdownAll），同时持有 `mainListener` 引用（通过 `setMainListener()` 注入），将 Teammate 工具事件转发给 Lead 的 SSE 流。`TeammateLoop` 是每个 Teammate 独立的 LLM 对话循环（实现 `Runnable`），拥有 8 个工具（bash/read/write/edit/msg_send/msg_read/idle/claim_task）、空闲轮询（每 5s 检查收件箱和 TaskStore，最多 60s）、身份再注入（消息数 < 6 时自动补充角色信息）、**每次工具调用完成后立即调用 `saveSession()` 持久化**（保证 Web Playground 实时可读）。

**SessionStore**：REST 会话持久化（`.sessions/{sessionId}.json`）。命名约定：
- Lead 会话：`{sessionId}.json`
- Teammate 子会话：`{leadSessionId}-tm-{name}.json`
- `listAll()` 过滤 `-tm-` 文件，侧边栏只展示 Lead 会话
- `listTeammates(leadSessionId)` 返回该 Lead 的所有 Teammate 信息

**ContextCompactor**：三层压缩管道：
1. **Micro Compact**（每轮自动）：将 `MICRO_COMPACT_KEEP_COUNT=3` 轮以前的 `tool_result` 替换为简短摘要
2. **Auto Compact**（`messages.size() >= 40`）：调用 LLM 生成对话摘要，压缩前历史存档为 `.transcripts/{timestamp}.json`，compacted user 消息携带 `_transcript_file` 字段（私有字段，API 调用前剥离）
3. **Manual Compact**（用户执行 `/compact`）：随时可触发


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fluentlc/claude-code-java](https://github.com/fluentlc/claude-code-java) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
