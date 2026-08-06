---
trigger: always_on
description: > 本文档旨在帮助大语言模型快速理解本项目的架构、代码组织和核心逻辑。
---

# AGENTS.md - AI 助手项目理解指南

> 本文档旨在帮助大语言模型快速理解本项目的架构、代码组织和核心逻辑。

## 项目概述

**Buy A Coffee** 是一个基于 Google ADK (Agent Development Kit) 的多 Agent 系统，实现智能咖啡订购和配送服务。项目展示了 A2A (Agent-to-Agent) 协作、工具调用 (Tool Calling)、多系统协同等 AI Agent 核心能力。

### 技术栈

| 层级 | 技术 | 用途 |
|------|------|------|
| AI 框架 | Google ADK | Agent 定义、工具绑定、多 Agent 协作 |
| A2A 协议 | A2A SDK | Agent 间通信协议 |
| 后端框架 | FastAPI | REST API、SSE 流式响应 |
| 数据库 | SQLite + aiosqlite | 异步数据持久化 |
| 前端框架 | React + TypeScript | 用户界面 |
| UI 组件 | CopilotKit | AI 聊天组件 |
| 样式 | TailwindCSS | 响应式设计 |

## 目录结构

```
buy-a-coffee/
├── backend/
│   ├── main.py              # 主入口（统一部署）
│   ├── run_all.py           # 分布式部署启动脚本
│   ├── config.py            # 全局配置
│   │
│   ├── shared/              # 共享模块
│   │   ├── __init__.py
│   │   ├── database.py      # 共享数据库基础设施
│   │   ├── http_client.py   # HTTP 客户端（工具调用后端 API）
│   │   └── utils.py         # 共享工具
│   │
│   ├── assistant/           # 助手服务
│   │   ├── __init__.py
│   │   ├── agent.py         # Assistant Agent（天气、时间、提醒）
│   │   └── tools.py         # 助手工具
│   │
│   ├── coffee/              # 希希咖啡服务 ☕
│   │   ├── __init__.py
│   │   ├── main.py          # 独立后端 API 入口 (端口 8001)
│   │   ├── a2a.py           # 独立 A2A Agent 入口 (端口 8003)
│   │   ├── agent.py         # Coffee Agent（含 order/query 子 Agent）
│   │   ├── api.py           # REST API 路由
│   │   ├── database.py      # 咖啡店数据库
│   │   └── tools.py         # 咖啡相关工具（通过 HTTP 调用后端）
│   │
│   ├── delivery/            # 送了么配送服务 🛵
│   │   ├── __init__.py
│   │   ├── main.py          # 独立后端 API 入口 (端口 8002)
│   │   ├── a2a.py           # 独立 A2A Agent 入口 (端口 8004)
│   │   ├── agent.py         # Delivery Agent
│   │   ├── api.py           # REST API 路由
│   │   ├── database.py      # 配送数据库
│   │   └── tools.py         # 配送相关工具（通过 HTTP 调用后端）
│   │
│   └── gateway/             # 网关服务 🌐
│       ├── __init__.py
│       ├── main.py          # 主网关入口 (端口 8000)
│       └── agent.py         # Root Agent（通过 A2A 调用子 Agent）
│
├── frontend/
│   └── src/
│       ├── components/      # React 组件
│       ├── App.tsx          # 主应用组件
│       └── main.tsx         # 入口文件
│
└── data/                    # SQLite 数据库文件（运行时生成）
```

## 部署模式

### 1. 统一部署（默认）

所有服务在同一个进程中运行，适合开发和演示：

```bash
cd backend
python main.py
```

服务启动后：
- API 文档: http://localhost:8000/docs
- 聊天接口: http://localhost:8000/api/chat/stream
- 咖啡 API: http://localhost:8000/api/coffee
- 配送 API: http://localhost:8000/api/delivery
- 咖啡 Agent Card: http://localhost:8000/agents/coffee/.well-known/agent.json
- 配送 Agent Card: http://localhost:8000/agents/delivery/.well-known/agent.json

### 2. 分布式部署

各服务独立运行，通过 A2A 协议通信，适合生产环境：

```bash
cd backend
python run_all.py
```

或手动启动各服务：

```bash
# 1. 启动咖啡店后端 API (端口 8001)
python -m coffee.main

# 2. 启动配送后端 API (端口 8002)
python -m delivery.main

# 3. 启动咖啡店 Agent A2A (端口 8003)
python -m coffee.a2a

# 4. 启动配送 Agent A2A (端口 8004)
python -m delivery.a2a

# 5. 启动主网关 (端口 8000)
DEPLOY_MODE=distributed python -m gateway.main
```

服务架构：

```
┌─────────────────────────────────────────────────────────────┐
│                     前端 (localhost:5173)                    │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                 主网关 (localhost:8000)                      │
│                     Root Agent                               │
│                         │                                    │
│    ┌────────────────────┼────────────────────┐              │
│    │                    │                    │              │
│    ▼                    ▼                    ▼              │
│ Assistant           A2A 调用            A2A 调用           │
│  Agent                  │                    │              │
└─────────────────────────┼────────────────────┼──────────────┘
                          │                    │
          ┌───────────────┘                    └───────────────┐
          ▼                                                    ▼
┌─────────────────────┐                          ┌─────────────────────┐
│ 咖啡 Agent A2A      │                          │ 配送 Agent A2A      │
│ (localhost:8003)    │                          │ (localhost:8004)    │
│                     │                          │                     │
│ coffee_agent        │                          │ delivery_agent      │
│ ├── order_agent     │                          │                     │
│ └── query_agent     │                          │                     │
└─────────┬───────────┘                          └─────────┬───────────┘
          │ HTTP                                           │ HTTP
          ▼                                                ▼
┌─────────────────────┐                          ┌─────────────────────┐
│ 咖啡后端 API        │                          │ 配送后端 API        │
│ (localhost:8001)    │                          │ (localhost:8002)    │
│                     │                          │                     │
│ /api/coffee/*       │                          │ /api/delivery/*     │
└─────────────────────┘                          └─────────────────────┘
```

## A2A 架构

### 核心设计

本项目的核心特性是使用 **A2A (Agent-to-Agent) 协议** 实现 Agent 间通信：

1. **咖啡 Agent** 和 **配送 Agent** 作为 A2A 服务暴露

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentrun-ai/buy-me-a-coffee](https://github.com/agentrun-ai/buy-me-a-coffee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
