---
trigger: always_on
description: A multi-platform desktop automation agent framework. Users send messages through channels (Telegram, Discord, Slack, Twitch, MQTT); the agent controls the desktop, browses the web, and runs tools using an LLM. Think of it as a personal AI assistant with eyes and hands on the machine.
---

# Operator

A multi-platform desktop automation agent framework. Users send messages through channels (Telegram, Discord, Slack, Twitch, MQTT); the agent controls the desktop, browses the web, and runs tools using an LLM. Think of it as a personal AI assistant with eyes and hands on the machine.

**Package:** `operator_use/` — main source. Entry: `main.py` → `operator_use/cli/commands.py`.

---

## Message Flow

```
Channel
  └─► Bus (incoming queue)
        └─► Orchestrator (STT, message building, routing)
              └─► Agent (LLM loop + tool execution)
                    └─► Bus (outgoing queue)
                          └─► Gateway → Channel (sends response)
```

1. A channel receives a user message and pushes an `IncomingMessage` onto the bus.
2. The **Orchestrator** consumes it: runs STT if it's a voice message, converts parts to a `HumanMessage`/`ImageMessage`, then routes to the correct Agent.
3. The **Agent** runs the LLM agentic loop up to `max_iterations`: if the LLM returns a tool call, it executes the tool and loops again; if it returns text, an `AIMessage` is returned.
4. The Orchestrator converts the response to an `OutgoingMessage` (running TTS if the user spoke), then publishes it to the outgoing bus queue.
5. The **Gateway** dispatch loop picks it up and sends it back through the originating channel.

---

## Module Map

| Module | What it does |
|---|---|
| `bus/` | Two async queues: `incoming` (channel → agent) and `outgoing` (agent → channel). Decouples everything. |
| `gateway/` | Manages all channel instances. Routes inbound messages to bus; dispatches outbound messages to the right channel. Channels live in `gateway/channels/`. |
| `orchestrator/` | Pipeline layer between bus and agents. Owns STT/TTS, message construction, agent routing, and the main `ainvoke()` consume loop. Also handles session control commands (`/start`, `/stop`, `/restart`) and pending-reply coordination. |
| `agent/` | LLM agentic loop. Receives a pre-built message, builds system prompt + history, calls LLM, handles tool calls iteratively. Has no knowledge of channels, bus, or STT/TTS. |
| `context/` | Builds the full system prompt from profile files (SOUL.md, RULES.md, MEMORY.md, skills index, knowledge index, CODE.md). |
| `providers/` | Adapters for 14+ LLMs (OpenAI, Anthropic, Google, Mistral, Groq, Ollama, Cerebras…) and STT/TTS providers. All implement `BaseChatLLM` / `BaseSTT` / `BaseTTS`. |
| `computer/` | Desktop control via native accessibility APIs. **Windows:** Windows UI Automation (UIA) via `comtypes`/`pywin32`. **macOS:** Accessibility Framework via `pyobjc` (`ax/` module). **Linux:** WIP. Exposed as tools via plugins. |
| `tools/` | Built-in tool implementations (filesystem, terminal, web search/browse). Auto-registered at startup. |
| `session/` | Per-user conversation history (`SessionManager` → `Session`). Persisted as `.jsonl` files at `.operator_use/sessions/{channel}_{chat_id}.jsonl`. |
| `config/` | Pydantic settings loaded from `.operator_use/config.json`, merged with `OPERATOR_` env vars. One config class per channel and provider. |
| `messages/` | Core message models: `HumanMessage`, `AIMessage`, `ImageMessage`, `ToolMessage`. |
| `plugins/` | Optional capability bundles (e.g. `computer_use`, `browser_use`). Each registers tools and hooks on an Agent at init time, and can be enabled/disabled at runtime. |
| `skills/` | Markdown procedural guides. Loaded from `profile/skills/{name}/SKILL.md`. Available immediately without restart. |
| `acp/` | ACP (Agent Communication Protocol) — multi-agent server/client. Routes messages to named agents with per-agent token isolation. |
| `subagent/` | Spawns child agents for parallel or delegated tasks within a single agentic loop. |
| `crons/` | Cron-scheduled tasks. Persisted to `.operator_use/crons.json`. When `deliver=True`, publishes `OutgoingMessage` directly to bus at fire time. |
| `heartbeat/` | Periodic maintenance loop (~30 min). Calls `orchestrator.process_direct()` with `HEARTBEAT.md` as the prompt so the agent can do background upkeep. |
| `interceptor/` | Pre/post message hooks. `RestartInterceptor` handles self-improvement restart recovery by reading `restart.json` on startup. |
| `process/` | Manages long-running background subprocesses launched by tools. |
| `web/` | Web browsing and search tools (DuckDuckGo, HTTP fetch, page scraping). |

---

## Agent Loop Detail

```python
# Non-streaming (agent/service.py _loop)
for iteration in range(max_iterations):
    messages = await context.build_messages(history, ...)  # system prompt + history
    llm_event = await llm.ainvoke(messages, tools)
    if llm_event.type == TOOL_CALL:
        result = await tool_registry.aexecute(tool_call.name, tool_call.params)
        session.add(ToolMessage)      # loop continues
    elif llm_event.type == TEXT:
        session.add(AIMessage)
        return AIMessage              # done
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CursorTouch/Operator-Use](https://github.com/CursorTouch/Operator-Use) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
