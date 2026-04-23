---
trigger: always_on
description: > This file provides project-level context for AI coding assistants and new contributors:
---

# AGENTS.md — DuckyClaw Project Guide

> This file provides project-level context for AI coding assistants and new contributors:
> business logic, technical architecture, directory layout, and coding guidelines.

---

## 1. Business Logic

DuckyClaw is a **hardware-oriented AI agent running on edge devices**. Its core value is enabling users to interact with IoT devices through natural language via IM channels (Telegram / Discord / Feishu). The on-device agent receives instructions, invokes local MCP tools to perform real actions, and replies through the same channel.

**Typical user scenarios:**

| Scenario | Flow |
|----------|------|
| Set a reminder | User sends "remind me in 5 minutes" on Telegram → Agent calls `get_current_time` + `cron_add` → cron fires and pushes a message when due |
| Manage files | User sends "read /memory/MEMORY.md" → Agent calls `read_file` → returns file content |
| Remote execution | User sends "run `ls -la`" (Raspberry Pi) → Agent calls `tool_exec` → returns command output |
| Memory & diary | Agent autonomously writes important info to MEMORY.md or daily notes, preserving context across sessions |
| OpenClaw control | User sends claw-machine command → Agent sends control via ACP/WebSocket gateway |

**Key business characteristics:**

- **Unified multi-channel**: Telegram, Discord, Feishu, WebSocket, serial CLI all share a single agent loop
- **Tool loop**: A single user message can trigger up to 10 LLM ↔ tool iterations until the agent produces a final reply
- **Persistent memory**: Long-term memory (MEMORY.md), daily notes (YYYY-MM-DD.md), personality (SOUL.md), user profile (USER.md) persisted on flash/SD
- **Cross-platform deployment**: Same codebase runs on Tuya T5AI, ESP32-S3, Raspberry Pi, and Linux desktop via conditional compilation

---

## 2. Technical Architecture

### 2.1 Layered Overview

```
┌──────────────────────────────────────────────────────┐
│                     IM Channel Layer                  │
│  Telegram  │  Discord  │  Feishu  │  WebSocket │ CLI │
└─────────────────────┬────────────────────────────────┘
                      │ im_msg_t (inbound / outbound)
              ┌───────▼───────┐
              │  Message Bus  │  Thread-safe dual queue
              └───────┬───────┘
                      │
              ┌───────▼───────┐
              │  Agent Loop   │  Outer: wait for msg
              │               │  Inner: ≤10 tool iterations
              └───┬───────┬───┘
                  │       │
        ┌─────────▼─┐ ┌──▼──────────┐
        │ Context   │ │ Cloud AI    │
        │ Builder   │ │ (ai_agent)  │
        │ sys prompt│ │ stream cb   │
        └─────┬─────┘ └──────┬──────┘
              │              │
    ┌─────────▼──────────────▼──────────┐
    │           MCP Tools Layer          │
    │ files │ cron │ exec │ openclaw    │
    └─────────┬──────────────┬──────────┘
              │              │
    ┌─────────▼─────┐  ┌────▼─────────┐
    │   Memory /    │  │   Gateway    │
    │   Session     │  │  WS + ACP    │
    └───────────────┘  └──────────────┘
```

### 2.2 Core Data Flow

1. **Inbound**: IM channel (or WS/cron/ACP) builds an `im_msg_t` → `message_bus_push_inbound()`
2. **Agent consumes**: `agent_loop_task` blocks on `message_bus_pop_inbound()`
3. **Inner loop**: `context_build_system_prompt()` assembles system prompt + history → `ai_agent_send_text()` sends to cloud
4. **AI callback**: `ducky_claw_chat.c`'s `__ai_chat_handle_event` receives stream events:
   - `STREAM_START/DATA/STOP`: accumulate text → record in history
   - `END`: calls `agent_loop_set_last_response()` + `agent_loop_notify_turn_done()` (posts semaphore)
5. **Tool execution**: Cloud AI triggers MCP tool call → `__on_tool_executed` hook records result → sets `s_turn.tool_called = true`
6. **Loop decision**: After semaphore post, agent checks `tool_called` — if true, feeds tool result as next input; otherwise forwards final reply to outbound queue
7. **Outbound**: `outbound_dispatch_task` dequeues message → dispatches to the corresponding IM SDK by `channel` field

### 2.3 Thread Model

| Thread | Responsibility | Entry |
|--------|---------------|-------|
| `tuya_app_main` / `main` | SDK init + `tuya_iot_yield` main loop | `user_main()` |
| `agent_loop` | Outer loop + inner tool iteration | `agent_loop_task()` |
| `outbound_loop` | Outbound message dispatch | `outbound_dispatch_task()` |
| `ws_server` | WebSocket server | `ws_server.c` |
| `acp_client` | OpenClaw gateway WS client | `acp_client.c` |
| `cron_service` | Scheduled task scheduler | `cron_service.c` |
| IM channel threads | Telegram polling / Discord gateway / Feishu WS | `*_bot.c` |

### 2.4 Synchronization

- **Agent loop ↔ AI callback**: Binary semaphore `s_turn.sem` (`agent_loop_task` waits, `ducky_claw_chat` posts)
- **Shared history**: `s_history_mutex` protects `s_history_json` (cJSON array, sliding window ≤ 10 entries)
- **Tool state**: `s_turn.lock` protects `tool_called` / `tool_result`
- **Message bus**: `tal_queue` is inherently thread-safe

---

## 3. Directory Layout

```
DuckyClaw/
├── agent/                  # Core agent loop and context builder
│   ├── agent_loop.c/h      #   Outer + inner tool-iteration loop (semaphore sync)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tuya/DuckyClaw](https://github.com/tuya/DuckyClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
