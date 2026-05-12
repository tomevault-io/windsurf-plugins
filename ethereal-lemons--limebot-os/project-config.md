---
trigger: always_on
description: This document is the authoritative source of truth for the LimeBot codebase. It is read by developers building on LimeBot **and** by the agent itself at runtime. Everything here is accurate to the current implementation.
---

# 🍋 AGENTS.md — LimeBot Developer & Agent Reference

This document is the authoritative source of truth for the LimeBot codebase. It is read by developers building on LimeBot **and** by the agent itself at runtime. Everything here is accurate to the current implementation.

---

## 🏛️ Architecture Overview

LimeBot is an **event-driven agentic system**. Every user interaction is an `InboundMessage` that flows through an async message bus, gets processed by the agent loop, and produces `OutboundMessage` events routed back to the originating channel.

```
Channel (Discord / WhatsApp / Web)
        │  InboundMessage
        ▼
   MessageBus (asyncio.Queue)
        │
        ▼
   AgentLoop._process_message()
     ├─ Auto-RAG (vector search + grep)
     ├─ Stable prompt cache (30s TTL)
     ├─ LLM call (LiteLLM / acompletion)
     ├─ Stream consumption + tool call extraction
     ├─ Tool execution loop (up to 30 iterations)
     ├─ Tag processing (save_soul, log_memory, etc.)
     └─ OutboundMessage → MessageBus → Channel.send()
```

---

## 📁 Core Module Reference

### `core/loop.py` — AgentLoop

The heart of the system. Manages:

- **Session history** per `session_key` (`channel_chatid`) with dirty-flag persistence
- **Stable prompt cache** — the rarely-changing part of the system prompt (soul + identity + user context) is cached for 30 seconds per `(sender_id, channel)` pair. Only the volatile suffix (memory, RAG results, timestamp) is rebuilt each message.
- **Auto-RAG** — before every LLM call, runs semantic vector search (falls back to grep if embeddings are unavailable). Injects matching memories into the prompt automatically.
- **Tool execution loop** — after each LLM response, if tool calls are returned, executes them in parallel and loops back to the LLM (up to 30 iterations). Sensitive tools require user confirmation.
- **Sub-agent delegation** — `spawn_agent` creates an isolated session that runs its own tool loop, can use named specialist profiles, and reports back to the parent session.
- **Per-session dedup** — identical consecutive messages within 2 seconds are silently dropped, keyed per session (not globally).
- **History summarization** — when token count exceeds limit, older messages are summarized by the LLM and squashed; large tool outputs from old turns are truncated in-place.

**Key configuration knobs (set via `self.config`):**
- `config.llm.model` — active chat model
- `config.llm.enable_dynamic_personality` — enables mood, affinity, and proactive jobs
- `config.autonomous_mode` — bypasses confirmation gate for sensitive tools
- `config.whitelist.allowed_paths` — roots for filesystem access

---

### `core/prompt.py` — Prompt Builder

Builds the system prompt from persona files. Two-part architecture:

**`build_stable_system_prompt()`** — rarely changes, cached 30s:
- Injects `SOUL.md` + `IDENTITY.md`
- Channel-specific style override (Discord → Web → General fallback chain)
- Dynamic personality block (affinity score → behavior tier) if enabled
- User context from `persona/users/{sender_id}.md`
- Skills system prompt additions
- Filesystem access declaration

**`get_volatile_prompt_suffix()`** — rebuilt every message:
- Auto-RAG recalled context
- Today's episodic memory journal (last 5 entries + long-term essence preview)
- Current timestamp

**Persona validation functions** (all use atomic temp-file writes + timestamped backups + auto-rotation to 3 most recent):
- `validate_and_save_soul(content)` — requires 100+ chars, soul keyword presence
- `validate_and_save_identity(content)` — requires `**Name:**`, `**Style:**`, 50+ chars
- `validate_and_save_mood(content)` — atomic write, no content minimum
- `validate_and_save_relationships(content)` — atomic write for `RELATIONSHIPS.md`

**Setup mode** — on startup, LimeBot bootstraps local `SOUL.md`, `IDENTITY.md`, and `MEMORY.md` from their `.example` templates when they are missing. If `SOUL.md` or `IDENTITY.md` are still missing or invalid after that, `is_setup_complete()` returns False and the entire system prompt is replaced with a setup interview prompt. The agent must emit `<save_soul>` and `<save_identity>` tags with valid content to exit setup mode.

---

### `core/subagents.py` — Subagent Registry

Discovers, loads, and describes named subagent profiles from built-ins and writable project/user locations.

- **Built-in specialists** — ships with named profiles such as `reviewer`, `verifier`, and `explorer`
- **Turn-level recommendation** — `recommend_subagent(task)` matches the current request against built-in intent keywords first, then against token overlap in custom subagent descriptions
- **Prompt injection** — `get_prompt_additions(current_message)` adds available subagents to the prompt and can call out the strongest match for the current turn
- **Selection-aware routing** — if a global default specialist is selected, the prompt nudges the model to prefer that specialist unless the user asks otherwise
- **Project shadowing** — project-defined subagents can override built-in ones with the same name


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ethereal-Lemons/LimeBot-OS](https://github.com/Ethereal-Lemons/LimeBot-OS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
