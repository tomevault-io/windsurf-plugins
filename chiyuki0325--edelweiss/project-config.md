---
trigger: always_on
description: Reference for contributors working on the Cahciua codebase. Improve code when you touch it; avoid one-off patterns.
---

# Cahciua Agent Guide

Reference for contributors working on the Cahciua codebase. Improve code when you touch it; avoid one-off patterns.

**Maintenance rule**: When you add, rename, or remove a file, change a key pattern, or complete a milestone — update this file in the same commit. Outdated docs are worse than no docs.

## What Is Cahciua

Cahciua is a Telegram / QQ group chat bot built on the **Deterministic Context Pipeline (DCP)** architecture. DCP constructs LLM context through a three-layer pure-function pipeline:

1. **Adaptation**: Platform Event → CanonicalIMEvent (anti-corruption layer).
2. **Projection**: `IC' = Reducers(IC, CanonicalIMEvent)` — pure-function state machine producing an Intermediate Context (IC).
3. **Rendering**: `RC = Render(IC, RenderParams)` — serialization with viewport filtering, producing Rendered Context (RC).

The Driver layer sits after Rendering: it merges RC (chat context) with its own TRs (bot responses, tool results) by timestamp to assemble the final LLM API request. Driver owns tool call loops, reactive scheduling, and context compaction. Supports three API formats: OpenAI Chat Completions (`openai-chat`, via xsai with SSE streaming), OpenAI Responses API (`responses`, via direct fetch with SSE streaming), and Anthropic Messages API (`anthropic-messages`, via direct fetch with SSE streaming). TRs are stored as provider-agnostic `ConversationEntry[]` via the unified API layer; format conversion happens at API boundaries when composing context or sending requests.

Key design goals: KV Cache friendly (append-only history, static system prompt, epoch-based compaction), group chat native (message batching, multi-user identity tracking, anti-injection via XML fencing), autonomous reply (bot decides whether to respond via Tool Call, not synchronous response).

## Current Progress

| Layer | Status | Notes |
|-------|--------|-------|
| Telegram integration | Done | Bot + userbot, dedup, fileId merge, credential redaction, per-session ingress queue, blocking image-to-text, blocking animation-to-text, blocking custom-emoji-to-text, send message reactions via bot, receive message reactions via Bot API updates, fetch reaction actors via userbot for count-only updates |
| OneBot integration | Done | OneBot 11 reverse WebSocket server, access-token check, message/notice adaptation, QQ face descriptions, image-to-text hydration, send/download PlatformAdapter |
| Adaptation | Done | Types, conversion, dual timestamps, rich text parsing, string IDs, phantom edit filtering |
| DB / Persistence | Done | events, messages, turn_responses, turn_responses_v2, compactions, probe_responses, probe_responses_v2, image_alt_texts, subagents, subagent_messages, background_tasks, message_reaction_snapshots tables; 29 migrations |
| Projection | Done | Reducer (message/blocked-message/edit/delete/reaction), MetaReducer (user rename detection), Immer-based immutability |
| Rendering | Done | `render(IC, RenderParams) → RC`, XML serialization, viewport filtering, thumbnail content pieces, passive reaction event rendering, blocked-message placeholders as deleted messages, inline `<image>` / `<animation>` / `<sticker>` / `<custom-emoji>` alt text rendering |
| Driver | Done | Triple-provider SSE streaming (OpenAI Chat Completions via xsai + Responses API via fetch + Anthropic Messages API via fetch), unified API codec layer (provider-agnostic IR with format conversion at boundaries), manual tool execution, Telegram-only `react_message`, per-step TR persistence (v2 schema), mid-turn interruption, reasoning sanitization (per-provider format), reactive orchestration (alien-signals), context compaction (LLM-based summarization with append-only history), probe/activate gate (small model decides silence vs activation), subagent delegation with isolated helper context and mailbox communication, skills system (user-facing tool definitions loaded from markdown files), background tasks (long-running shell tasks with lifecycle management), typing-aware debounce scheduling (debounce-scoped Telegram typing presence with online heartbeat / markAsRead / supergroup channel-difference fallback), offline/online reply gating via /offline /online commands, rtk output compaction (optional argv0 rewriting + pipe fallback for bash tool) |
| Eval harness | Initial | Offline LLM eval suites for comparing prompt variants against fixed IC fixtures, repeated runs, custom TypeScript evaluators, side-effect-free tool traces, and probability summaries |

## Tech Stack

- **Runtime**: Node.js (>=22), TypeScript, tsx (dev), tsdown (build).
- **Telegram Bot API**: grammY — primary message handling, sending replies, commands.
- **Telegram User API**: gramjs (`telegram` on npm) — MTProto client for history fetching, reply-to context resolution, seeing other bots' messages.
- **OneBot 11**: reverse WebSocket over `ws` — QQ ingress/egress via array message segments, with optional bearer access token. Outbound fenced code blocks can be rendered to images through the optional system `silicon` binary; if unavailable or failing, OneBot send falls back to plain text and logs a warning.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chiyuki0325/Edelweiss](https://github.com/chiyuki0325/Edelweiss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
