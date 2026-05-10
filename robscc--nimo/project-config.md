---
trigger: always_on
description: > 每次 Claude Code 会话启动时自动读取本文件。
---

# CLAUDE.md — AgentPal (nimo)

> 每次 Claude Code 会话启动时自动读取本文件。

## 项目概览

**AgentPal**（代号 nimo）是基于 [AgentScope 1.x](https://github.com/modelscope/agentscope) 构建的开源个人智能助手平台。

- GitHub：https://github.com/robscc/nimo
- 技术栈：FastAPI（后端）+ React（前端）+ SQLite（存储）
- 目标：多渠道接入（Web / DingTalk / 飞书 / iMessage）、工具调用、SubAgent 多角色协作、定时任务

---

## 仓库结构

```
agentpal/                       ← 项目根目录（本地路径）
├── backend/                    ← FastAPI 后端
│   ├── agentpal/
│   │   ├── agents/             ← PersonalAssistant、SubAgent、CronAgent、BaseAgent
│   │   │   ├── personal_assistant.py  ← 主 Agent（流式对话 + 工具调用）
│   │   │   ├── sub_agent.py           ← SubAgent（独立上下文 + 多轮工具 + 执行日志）
│   │   │   ├── cron_agent.py          ← 定时任务 Agent（轻量，只加载 SOUL.md + AGENTS.md）
│   │   │   ├── base.py               ← Agent 基类，共享功能
│   │   │   ├── registry.py            ← SubAgent 角色注册（CRUD + 任务路由）
│   │   │   └── message_bus.py         ← Agent 间异步消息总线（DB + ZMQ 混合）
│   │   ├── scheduler/           ← 多进程 Scheduler 架构（新）
│   │   │   ├── broker.py              ← SchedulerBroker — 中央调度器（进程管理 + 消息路由）
│   │   │   ├── client.py             ← SchedulerClient — FastAPI 进程内薄客户端
│   │   │   ├── worker.py             ← worker_main — Worker 子进程入口
│   │   │   ├── process.py            ← scheduler_process_main — Scheduler 进程入口
│   │   │   ├── config.py             ← SchedulerConfig — 地址/超时/策略配置
│   │   │   ├── state.py              ← AgentState / AgentProcessInfo — 进程状态机
│   │   │   └── scheduler.py          ← AgentScheduler — 过渡兼容层
│   │   ├── api/v1/endpoints/   ← agent, tools, session, channel, sub_agents, cron, skills, workspace, config, dashboard, memory, notifications, providers, tasks
│   │   ├── channels/           ← dingtalk.py, feishu.py, imessage.py
│   │   ├── memory/             ← base / buffer / sqlite / hybrid / factory / reme_light_adapter
│   │   ├── models/             ← ORM: memory, session, tool, skill, agent, cron, message, llm_usage
│   │   ├── providers/          ← Provider 管理（manager.py, provider.py, openai_provider.py, retry_model.py）
│   │   ├── runtimes/           ← SubAgent 运行时抽象（base.py, internal.py, http.py, registry.py）
│   │   ├── zmq_bus/            ← ZMQ 消息总线（manager.py, protocol.py, daemon.py, pa_daemon.py, sub_daemon.py, cron_daemon.py, event_subscriber.py）
│   │   ├── cli/                ← 命令行工具（app.py, commands/start|stop|restart|status）
│   │   ├── workspace/          ← 工作空间管理（manager.py, context_builder.py, defaults.py, memory_writer.py）
│   │   ├── services/           ← config_file.py, cron_scheduler.py, notification_bus.py, task_event_bus.py, session_event_bus.py, skill_event_bus.py
│   │   ├── tools/              ← builtin.py (12个工具), registry.py
│   │   ├── config.py           ← pydantic-settings（优先级: env > ~/.nimo/config.yaml > .env > defaults）
│   │   ├── database.py         ← async SQLAlchemy + init_db()
│   │   └── main.py             ← FastAPI app factory + lifespan
│   ├── tests/
│   │   ├── unit/               ← 单元测试（agents, memory, channels, cron, config, skills, browser_use, providers, cli, zmq, tool_guard, workspace, notifications）
│   │   ├── integration/        ← 集成测试（API + 内存 SQLite）
│   │   └── e2e/                ← Playwright E2E 测试（需前后端运行）
│   └── pyproject.toml          ← 包元数据、依赖、ruff/pytest 配置
├── frontend/                   ← React + Vite + TypeScript + Tailwind
│   └── src/
│       ├── pages/              ← ChatPage, ToolsPage, SkillsPage, TasksPage, SessionsPage, WorkspacePage, CronPage, DashboardPage
│       ├── components/         ← Layout (侧边栏导航), SessionPanel, NimoIcon, NimoLogo, MentionPopup, TaskArtifactViewer
│       ├── hooks/              ← useTools, useSessions, useSessionMeta, useSkills, useSubAgents, useCron, useTasks, useTaskArtifacts, useNotifications, useSessionEvents
│       └── api/index.ts        ← axios base client + 全部 API 类型定义
├── .github/workflows/          ← ci.yml (lint + test matrix), release.yml
├── Makefile                    ← make dev / test / lint / format / docker-*
└── docker-compose.yml
```

---

## 开发环境启动

```bash
# 后端 (http://localhost:8099，--reload 热重载)
cd backend
.venv/bin/python -m uvicorn agentpal.main:app --port 8099 --reload

# 前端 (http://localhost:3000)
cd frontend
npm run dev
```

> **注意**：`Makefile` 里的端口是 8088，实际本地跑在 **8099**（避免与 CoPaw Console 冲突）。
> 前端 `vite.config.ts` 代理目标为 `http://localhost:8099`。

---

## 关键配置

配置优先级（从高到低）：环境变量 > `~/.nimo/config.yaml` > `.env` > 代码默认值。

当前本地配置在 `~/.nimo/config.yaml`：

```yaml
llm:
  provider: compatible
  model: qwen3.5-plus
  api_key: sk-...
  base_url: https://coding.dashscope.aliyuncs.com/v1
```

> **注意**：创建 Session 时 `model_name` 会持久化到 DB。改全局配置只影响新 Session，旧 Session 需手动更新或新建。

---

## 核心设计决策

### 1. LLM 调用（agentscope 1.x）
- 使用 `agentscope.model.OpenAIChatModel` 直接实例化，**不** 使用 `agentscope.init()`
- 工具调用需 **OpenAI 格式**（非 Anthropic format）：
  - 助手消息：`{"role": "assistant", "content": null, "tool_calls": [...]}`
  - 工具结果：`{"role": "tool", "tool_call_id": "...", "content": "..."}`
- `toolkit.call_tool_function(tool_call)` 是 coroutine，返回 AsyncGenerator：
  ```python
  async for chunk in await toolkit.call_tool_function(tool_call):
      tool_response = chunk
  ```

### 2. 流式输出（SSE）
- `POST /api/v1/agent/chat` 返回 `text/event-stream`
- 事件格式：
  ```
  data: {"type": "thinking_delta", "delta": "..."}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robscc/nimo](https://github.com/robscc/nimo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
