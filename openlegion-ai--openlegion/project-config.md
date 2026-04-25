---
trigger: always_on
description: OpenLegion Engine is a container-isolated multi-agent runtime. LLM-powered agents run in Docker containers, coordinated through a central mesh host. Users interact via CLI REPL, messaging channels (Telegram/Discord/Slack/WhatsApp/Webhook), or a web dashboard.
---

# CLAUDE.md — engine

## Overview

OpenLegion Engine is a container-isolated multi-agent runtime. LLM-powered agents run in Docker containers, coordinated through a central mesh host. Users interact via CLI REPL, messaging channels (Telegram/Discord/Slack/WhatsApp/Webhook), or a web dashboard.

## Architecture

### System Layout

```
User (CLI REPL / Telegram / Discord / Slack / WhatsApp / Webhook)
  → Mesh Host (FastAPI :8420) — routes messages, enforces permissions, proxies APIs, VNC proxy
    → Agent Containers (FastAPI :8400 each) — isolated execution with private memory
    → Browser Service Container (FastAPI :8500 + KasmVNC :6080) — shared Camoufox browser
```

Three trust zones: **User** (full trust), **Mesh** (trusted coordinator), **Agents** (untrusted, sandboxed). All cross-zone communication is HTTP + JSON with Pydantic contracts from `src/shared/types.py`.

### Entry Points

| Entry Point | File | What It Starts |
|---|---|---|
| `openlegion` CLI | `src/cli/__main__.py` → `src/cli/main.py` | CLI commands: start, stop, status, chat, wallet, version |
| Agent container | `src/agent/__main__.py` → `src/agent/server.py` | Agent FastAPI server on :8400 |
| Browser service | `src/browser/__main__.py` → `src/browser/server.py` | KasmVNC + Openbox + FastAPI on :8500 |
| Mesh host | `src/host/server.py` | FastAPI app on :8420 (started by CLI) |
| Dashboard | `src/dashboard/server.py` | Mounted as router on mesh host |

### Module Responsibilities

| Path | Responsibility |
|---|---|
| **`src/shared/`** | |
| `types.py` | Pydantic models — THE cross-component contract (335 lines, 24 models). `_generate_id()` helper. `AGENT_ID_RE_PATTERN` unified regex. |
| `utils.py` | `sanitize_for_prompt()`, `setup_logging()`, misc helpers |
| `trace.py` | Distributed trace-ID generation and propagation |
| `models.py` | Model cost/context window registry backed by LiteLLM, `estimate_cost()` |
| **`src/agent/`** | |
| `loop.py` | Agent execution loop (task + chat mode). `MAX_ITERATIONS=20`, `CHAT_MAX_TOOL_ROUNDS=30`, `CHAT_MAX_TOTAL_ROUNDS=200`, `_MAX_SESSION_CONTINUES=5`, `HEARTBEAT_MAX_ITERATIONS=10`. Env-var bounds clamped via `_clamp_env()`. |
| `server.py` | Agent FastAPI server (27 endpoints). `_FILE_CAPS` (7 entries) enforced on workspace writes (HTTP 413). `_WORKSPACE_ALLOWLIST` frozenset gates reads/writes. |
| `llm.py` | LLM client — routes through mesh proxy, never holds keys |
| `context.py` | Context window management (write-then-compact, `_SUMMARIZATION_INPUT_LIMIT=20_000`). Empty summary falls back to hard prune. |
| `skills.py` | Skill registry and tool discovery |
| `memory.py` | Per-agent SQLite + sqlite-vec + FTS5 memory store |
| `workspace.py` | Persistent markdown workspace. Bootstrap files: PROJECT.md, SYSTEM.md, INSTRUCTIONS.md, SOUL.md, USER.md, MEMORY.md, INTERFACE.md. Also manages HEARTBEAT.md (loaded separately), daily logs, learnings. |
| `mesh_client.py` | Agent-side HTTP client for mesh communication |
| `loop_detector.py` | Tool loop detection with escalating responses (warn/block/terminate) |
| `mcp_client.py` | MCP tool server client and lifecycle |
| `attachments.py` | Multimodal attachment enrichment (images → base64 vision blocks, PDFs → text extraction) |
| **`src/agent/builtins/`** | |
| `browser_tool.py` | Browser automation via shared Camoufox service. CAPTCHA detection/solving, screenshot capture with multimodal image blocks. |
| `exec_tool.py` | Shell execution scoped to `/data` (`_MAX_TIMEOUT=300`) |
| `file_tool.py` | File I/O with two-stage path traversal protection (`lstat()` for symlink safety) |
| `http_tool.py` | HTTP requests with CRED handles, SSRF protection (DNS pinning, IP blocking incl. CGNAT, 6to4, Teredo), cross-origin auth header stripping |
| `memory_tool.py` | Memory search with hierarchical fallback, memory save |
| `mesh_tool.py` | Blackboard (with `sanitize_for_prompt()`), pub/sub, notify_user, list_agents, artifacts, cron, spawn |
| `coordination_tool.py` | Structured multi-agent coordination protocol — hand_off, check_inbox, update_status. Higher-level wrappers over blackboard for inter-agent work handoffs. |
| `vault_tool.py` | Credential generation without returning actual values |
| `web_search_tool.py` | DuckDuckGo search (no API key needed) |
| `image_gen_tool.py` | Image generation via Gemini or OpenAI DALL-E 3, saves output as artifacts |
| `skill_tool.py` | Self-authoring with AST validation. `_FORBIDDEN_IMPORTS` (23 modules), `_FORBIDDEN_CALLS` (16 functions), `_FORBIDDEN_ATTRS` (11 attributes). |
| `subagent_tool.py` | In-process subagents (MAX_DEPTH=2, MAX_CONCURRENT=3, MAX_TTL=600s, DEFAULT_MAX_ITERATIONS=10) |
| `introspect_tool.py` | Runtime state query (permissions, budget, fleet, cron, health) |
| `wallet_tool.py` | Wallet operations — get address, get balance, read contract, transfer, execute (Ethereum + Solana) |
| `fleet_tool.py` | Operator-only fleet management tools (`list_templates`, `apply_template`) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openlegion-ai/openlegion](https://github.com/openlegion-ai/openlegion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
