---
trigger: always_on
description: FlowPartner 是一款面向非专业用户的 AI Agent 桌面应用。这些没有计算机背景的用户过度信任 AI，因此该 AI Agent 桌面应用必须代替用户承担安全守门人的角色。
---

﻿# AGENTS.md

## 项目概述

FlowPartner 是一款面向非专业用户的 AI Agent 桌面应用。这些没有计算机背景的用户过度信任 AI，因此该 AI Agent 桌面应用必须代替用户承担安全守门人的角色。

**核心优先级**：防呆 > 安全 > 可恢复 > 功能 > 性能。

任何可能使用户误操作或陷入不可恢复状态的设计均视为不合格。

**所有的设计都是针对我们编写的软件来说的，要求软件实现这些功能；在代码编写中并不需要遵循这样的安全设计。在代码编写过程中，应该在不确定的时候，询问用户的意见。**

所有的方案应该采用工业级、长久性、大型项目使用的方案，不要先用简单方案替代；想想如果你要跟他打交道很长时间，你会选择的方案。

**uv.lock 依赖纪律**：不允许更改 uv.lock 的源，因为这样会导致 Github Actions 无法拉取到包。pyproject.toml 中禁止配置 `[tool.uv] index-url` 之类的镜像（镜像属个人网络环境偏好；历史上仓库内的清华源配置导致任何不带 `--frozen` 的 uv 命令在重锁时把整个 lock 改写为镜像地址）。因此：
- 一切 uv 命令必须带 `--frozen`（只运行工具不安装依赖时用 `--no-sync`）
- 提交前若发现 uv.lock 存在与本任务无关的 diff，先 `git checkout -- agent/uv.lock` 还原，再排查触发重锁的原因

---

## 架构概览（当前实际状态）

### 总体架构：WebSocket + gRPC 双向通信 + LLM 流式调用 + 多 Agent + 快照

```
Frontend (Electron + React + TypeScript)
    │
    │  WebSocket (JSON: {action: "start_chat", content: "..."} 等六种 action)
    ▼
Backend (Go)
    ├── WebSocketHandler (internal/handler/ws.go)
    │       ↓ 注册 session，生成 sessionId
    │       ├── start_chat / permission_response / cancel_task（聊天与审批）
    │       └── manual_snapshot / system_lock / restore（快照控制）
    ├── bridge.Manager (internal/bridge/manager.go)
    │       ├── sessions map: sessionId → WebSocket conn
    │       └── CmdChan: 发往 Python 的指令通道
    │       ↓ gRPC bidirectional stream（端口从 50051 起动态发现）
    ├── AgentHandler (internal/handler/agent.go)
    │       ├── SyncChannel: 接收 Python 事件 → 转发到 WebSocket
    │       ├── CallLLM: 服务端流式 RPC → 调用 LLM Client
    │       ├── ExecuteTool: 工具执行代理
    │       └── ListAgents / GetAgent: 向 Python 提供智能体定义
    │
    ├── LLM Client (internal/llm/client.go)
    │       ├── HTTP POST → OpenAI 兼容 API
    │       ├── SSE 流式解析 (internal/llm/sse.go)
    │       ├── 错误分类 (internal/llm/error.go)
    │       └── URL 规范化 (internal/llm/url.go)
    │
    ├── ModelConfig Handler (internal/handler/model_config.go)
    │       ├── CRUD: /api/model_configs
    │       ├── Activate: /api/model_configs/{id}/activate
    │       └── 加密存储 API Key (internal/crypto)
    │
    ├── AgentDef Handler (internal/handler/agent_def.go)   ← 多 Agent
    │       ├── CRUD: /api/agents（内置主智能体 id 固定为 "main"）
    │       ├── 变更后经 gRPC 广播 agents_changed 失效通知
    │       └── 持久化到 internal/storage/agents.go（agents.json 单一真相源）
    │
    ├── Snapshot 子系统 (internal/snapshot/, internal/handler/snapshot_handler.go)
    │       ├── 触发：工作区文件变更防抖 60s / 15min 周期兜底 / 锁屏 flush / 手动 / 还原前预快照
    │       ├── capture + exclude（敏感文件默认排除、单文件 100MB 上限）+ manifest
    │       ├── restore（先自动预快照保证可逆）+ retain（保留 30 天 / 5GB）
    │       ├── 状态经 WebSocket broadcast（snapshot_status / snapshot_message）
    │       └── REST: /api/snapshots
    │
    ├── Keystore (internal/keystore/keystore.go)
    │       ├── TryActivate: 解密 + 解锁（带速率限制）
    │       ├── SwitchKey: 原子切换密钥
    │       └── GetKey: 供 LLM Client 使用
    │
    ├── Tools Executor (internal/tools/)
    │       ├── executor.go: 工具调度（read/write/bash/edit/trash/purge）
    │       ├── approval.go: 权限审批管理（一次性令牌 + 会话级信任）
    │       ├── path_guard.go: 双层路径验证（词法 + 符号链接解析）
    │       ├── delete_guard.go: 拦截 shell 删除类命令（rm/del/Remove-Item 等）
    │       ├── trash.go / purge.go: 可恢复删除（回收站）/ 强制审批的永久删除
    │       └── 执行结果通过 gRPC ExecuteTool 返回
    ▼
Python Agent (agent/src/agent/)
    ├── FlowPartnerClient (grpc_client.py): gRPC 客户端
    │       ├── call_llm_via_go: 处理服务端流式响应
    │       │       ├── 逐 chunk 接收 SSE JSON
    │       │       ├── 重建 tool_calls delta（丢弃参数非法的调用）
    │       │       └── 发送 llm_chunk 事件到前端
    │       ├── execute_tool: 通过 gRPC 代理执行工具（含越权审批等待）
    │       ├── ListAgents/GetAgent: 拉取智能体定义（TTL 缓存 + agents_changed 失效）
    │       ├── connect_and_listen: 双向流事件循环
    │       └── _cancel_task: 取消指定会话任务（cancel_task 命令分发的实现）
    ├── core/react_agent.py: ReAct 循环（思考→行动→观察，带护栏）
    ├── core/subagent_runner.py: 子 agent 派发与事件转发（subagent_* 事件）
    ├── core/agent_registry.py: 智能体定义缓存
    └── tools/: registry.py + file_ops.py + agent_tool.py
            （工具 read/write/bash/edit/trash/purge 全部通过 Go 代理执行；
              agent__<name> 为子智能体调度入口）
```

### 前端启动流程

```
Electron main.cjs
    │ 启动后端子进程，读取就绪信号 __FP_BACKEND_READY__ HTTP=:%d gRPC=:%d
    │ 保存 backendPort
    ↓
preload.cjs
    │ 暴露 window.flowPartner.fetchBackendPort() → backendPort
    │ 暴露 window.flowPartner.onSystemLock(cb) / onSystemFocus(cb) → 系统锁屏/焦点监听
    │ 暴露 onCloseAction / sendCloseAction / updateCloseBehavior（关闭行为）
    │ 暴露 openExternal(url)、selectFolder()、getVersion()
    ↓
main.tsx bootstrap
    │ await window.flowPartner.fetchBackendPort()
    │ initApi(port) → 设置 BASE = http://localhost:{port}/api
    │ 渲染 React 应用
    ↓
useWebSocket hook
    │ 连接 ws://localhost:{port}/ws
    │ 支持自动重连（最多 5 次，间隔 3s）、处理超时（300s）、安全端口校验（1024-65535）
```

### 当前代码状态

**main.go 已正常工作**：
- `backend/cmd/server/main.go` 注入 `bridge.Manager` + `ApprovalManager` + 快照管理器，同时启动 HTTP server 和 gRPC server
- HTTP server 注册了 REST 路由（settings、settings/clear_api_key、history、unlock/lock/lock_status、model_configs、snapshots、agents）和 WebSocket 端点（`/ws`）
- gRPC server 注册了 `AgentHandler`，与 Python Agent 通过双向流通信
- 端口通过 `server.FindAvailablePort` 动态发现（绑定 127.0.0.1，HTTP 默认端口来自 FP_HTTP_PORT，gRPC 从 :50051 起探索并排除 HTTP 端口），就绪信号格式：`__FP_BACKEND_READY__ HTTP=:%d gRPC=:%d`
- 启动时按已保存设置应用快照配置（含过期清理，后台执行）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SongHuang1/FlowPartner](https://github.com/SongHuang1/FlowPartner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
