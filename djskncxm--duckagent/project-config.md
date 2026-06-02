---
trigger: always_on
description: 多 Agent 协作系统，用于 Android 逆向工程。Agent 平权——通过 @mention 互相点名，不设中心路由。
---

# Android 逆向 Multi-Agent 系统

## 项目概述

多 Agent 协作系统，用于 Android 逆向工程。Agent 平权——通过 @mention 互相点名，不设中心路由。

当前阶段：Phase 4 — MCP 工具协议 + 配置驱动 + 惰性连接。

## 技术栈

- 语言：Python 3.12+
- 模型调用：litellm（统一接口，切模型只改配置）
- 消息总线：MessageBus ABC → LocalMessageBus（SQLite + asyncio.Queue）+ HttpMessageBus（HTTP + WebSocket）
- 服务端：FastAPI + uvicorn（独立进程）
- 工具协议：MCP（Model Context Protocol）— Agent 是纯 MCP 客户端
- Trace 工具：MCP stdio server → ak_search（C daemon，mmap + 行索引）
- IDA 工具：MCP Streamable HTTP → IDA Pro MCP server（127.0.0.1:13337/mcp）
- JADX 工具：MCP stdio → jadx-mcp-server（连接 JADX-GUI 8650 端口）
- 包管理：uv
- CLI：typer + Textual TUI
- 下一阶段：Unidbg Agent、更完善的 prompt 和分析方法

## 架构

### 多进程模式（`duck launch`）

```
进程: bus-server (FastAPI + SQLite + WebSocket)     :8720
进程: main-agent  ──┐
进程: trace-agent ──┼── HTTP POST /api/v1/publish + WS /ws?agent_id=<id>
进程: ida-agent   ──┘
进程: tui         ──── WS /ws?role=observer (看全部) + HTTP POST (发消息)
```

### MCP 工具连接（Agent 是纯客户端）

```
ida_jadx_agent (MCP client)
    ├─ ida-pro-mcp (Streamable HTTP)  → IDA Pro（用户手动管理）
    └─ jadx-mcp (stdio subprocess)    → JADX-GUI（用户手动管理）

trace_agent (MCP client)
    └─ trace (stdio subprocess)       → 内置 trace MCP server (ak_search)
```

Agent 不管理 MCP server 生命周期。连接是惰性的——第一次调工具时才连。
连不上不崩溃，返回 error 给 LLM，LLM 自行适应。

### 单进程模式（`duck run`，向后兼容）

```
┌──────────────────────────────────────────────────────────────────┐
│  单进程 (asyncio)                                                │
│                                                                  │
│                     ┌─────────────┐                              │
│  Textual TUI ←───→  │   人类      │                              │
│                     └─────────────┘                              │
│                           │                                      │
│         @trace_agent      │      @ida_jadx_agent                 │
│              ┌────────────┼────────────┐                         │
│              ▼            │            ▼                         │
│  ┌──────────────┐  ┌──────────┐  ┌───────────────┐              │
│  │ TraceAgent   │  │MainAgent │  │ IdaJadxAgent  │              │
│  │ (trace MCP)  │  │ (纯推理) │  │ (IDA+JADX MCP)│              │
│  └──────┬───────┘  └────┬─────┘  └───────┬───────┘              │
│         │               │                │                       │
│         └───────────────┼────────────────┘                       │
│                         ▼                                        │
│              ┌─────────────────────┐                             │
│              │ MessageBus (SQLite) │                             │
│              └─────────────────────┘                             │
└──────────────────────────────────────────────────────────────────┘
```

### 角色

| 角色 | agent_id | 职责 | MCP Servers |
|------|----------|------|-------------|
| MainAgent | main_agent | 协调、拆解任务、综合结论 | 无（纯推理） |
| TraceAgent | trace_agent | 执行流分析、算法还原 | trace（内置 stdio） |
| IdaJadxAgent | ida_jadx_agent | 静态分析、反汇编、反编译 | ida-pro-mcp + jadx-mcp |
| 人（Leader） | human | 终审、路径决策 | Textual TUI |

### 未来角色

| 角色 | 职责 | 工具 |
|------|------|------|
| Unidbg | 补环境、模拟执行、验证 | unidbg Java API (MCP) |

## MCP 工具系统

### 设计原则

- Agent 是**纯 MCP 客户端**——不启动/停止 server 进程
- 连接**惰性**——第一次 `think()` 调工具时才触发连接
- 连不上**优雅降级**——返回 error JSON 给 LLM，不阻塞 agent
- 工具 description 自动加 `[server名]` 前缀——LLM 一眼看出来源
- 工具失败**隔离**——IDA 挂了不影响 JADX，反之亦然

### MCP Server 配置

读取 `~/.claude/.mcp.json`（Claude Code 兼容格式）：

```json
{
    "mcpServers": {
        "ida-pro-mcp": {
            "type": "http",
            "url": "http://127.0.0.1:13337/mcp"
        },
        "jadx-mcp": {
            "command": "uv",
            "args": ["run", "/path/to/jadx_mcp_server.py"]
        }
    }
}
```

支持两种传输：
- **stdio**：subprocess 子进程（stderr 输出被吞到 /dev/null）
- **http**：Streamable HTTP（连接外部 MCP server）

### Agent ↔ MCP Server 映射

通过环境变量配置哪个 agent 连哪些 server：

```bash
DUCKAGENT_TRACE_AGENT_MCP_SERVERS=trace              # 内置
DUCKAGENT_JADX_AGENT_MCP_SERVERS=ida-pro-mcp,jadx-mcp  # 从 .mcp.json 读
DUCKAGENT_MAIN_AGENT_MCP_SERVERS=                    # 纯推理，无工具
```

### 添加新 MCP server

零代码——在 `~/.claude/.mcp.json` 加一条 + 配环境变量：

```bash
# 例：给 ida_jadx_agent 加一个 frida MCP server
# 1. 在 .mcp.json 的 mcpServers 里加 "frida": {...}
# 2. 设置 DUCKAGENT_JADX_AGENT_MCP_SERVERS=ida-pro-mcp,jadx-mcp,frida
```

## TUI

基于 Textual 框架，面板式布局。`duck run` 启动 TUI 模式，`duck log`/`duck send` 保留为纯命令行模式。

```
┌─────────────────────────────────────────┬──────────────┐
│  Messages                               │ Agents       │
│                                         ├──────────────│
│  📤 [14:25] you → main_agent @trace ▎🔵│ main_agent   │
│  分析这个 trace 里的签名算法         ▎  │ ● idle       │
│                                         │ 处理: -      │
│  📤 [14:25] main_agent → all       ▎⚪│ 结论: -      │
│  [@trace_agent]                     ▎  ├──────────────│
│  @trace_agent 分析 AES 加密...      ▎  │ trace_agent  │
│                                         │ ● thinking   │
│  📋 [14:26] trace_agent → main     ▎⚪│ 处理: ...    │
│  发现 HMAC-SHA256 签名...          ▎  │ 结论: -      │
│                                         ├──────────────│
│  📋 [14:27] main_agent → you       ▎🟢│ ida_jadx_ag  │
│  分析结果：签名算法是 HMAC-SHA256  ▎  │ ● idle       │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [djskncxm/DuckAgent](https://github.com/djskncxm/DuckAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
