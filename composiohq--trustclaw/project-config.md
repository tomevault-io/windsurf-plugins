---
trigger: always_on
description: This is the AI agent runtime for TrustClaw. It orchestrates Anthropic Claude calls with Composio tools (external service integrations) and custom tools (memory, scheduling), while managing the context window through a 3-layer system adapted from [pi-mono](https://github.com/nicholasgasior/pi-mono) and [OpenClaw](https://github.com/nicholasgasior/openclaw).
---

# TrustClaw Agent

## Overview

This is the AI agent runtime for TrustClaw. It orchestrates Anthropic Claude calls with Composio tools (external service integrations) and custom tools (memory, scheduling), while managing the context window through a 3-layer system adapted from [pi-mono](https://github.com/nicholasgasior/pi-mono) and [OpenClaw](https://github.com/nicholasgasior/openclaw).

Entry point: `prepareAgentRun()` in `setup.ts`, consumed by `app/api/chat/route.ts` (web), `app/api/telegram-webhook/route.ts`, and `app/api/cron/trustclaw/execute/route.ts`.

## Architecture

```
User message (web / telegram / cron)
    │
    ▼
┌─────────────────────────────────────────────────┐
│  setup.ts - prepareAgentRun()                   │
│                                                 │
│  1. Load instance from DB                       │
│  2. Build system prompt                         │
│  3. Load messages (compaction-aware)      ◄─── context/build-context.ts
│  4. Prune context (trim/clear old tools)  ◄─── context/context-pruning.ts
│  5. Save user message to DB                     │
│  6. Init Composio session + tools               │
│  7. Create ToolLoopAgent with Anthropic   ◄─── Vercel AI SDK
│     (memory_save / memory_search tools           │
│      backed by pgvector + OpenAI embeddings)     │
│  8. Return agent + messages to caller           │
│     (caller runs agent.stream() or .generate()) │
│  9. Update assistant message in DB (onFinish)   │
│ 10. Fire-and-forget post-response tasks:        │
│     a. Memory flush (if approaching limit) ◄── compaction/memory-flush.ts
│     b. Compaction (if over limit)         ◄─── compaction/run-compaction.ts
└─────────────────────────────────────────────────┘
```

## Directory Structure

```
agent/
├── setup.ts                    # prepareAgentRun() - builds agent, tools, context
├── index.ts                    # Re-exports prepareAgentRun, types, cron-utils
├── types.ts                    # ReconstructedMessage, JsonValue, ToolResultOutput
├── strip-tool-echoes.ts        # Strips echoed tool results from assistant text
├── system-prompt.ts            # Builds system prompt from identity + soul + user + memory
├── error-parser.ts             # Parses Composio/Anthropic errors into user-friendly messages
│
├── context/                    # Context window management
│   ├── build-context.ts        # DB loading, message reconstruction, post-response orchestration
│   ├── context-window.ts       # Maps model IDs → context window size (200K for all Claude 4.x)
│   ├── token-estimation.ts     # chars/4 heuristic, shouldCompact()
│   └── context-pruning.ts      # 2-phase pruning: soft trim then hard clear of tool results
│
├── compaction/                 # Context compaction (summarization when context overflows)
│   ├── run-compaction.ts       # Cut point algorithm, LLM summarization, DB persistence
│   └── prompts.ts              # Summarization prompts, message serialization, tool failure tracking
│
└── tools/                      # Agent tool definitions (one tool per file + sibling .schema.ts)
    ├── index.ts                # createCustomTools()
    ├── memory-save.ts / .schema.ts    # Save a memory (pgvector + OpenAI embeddings)
    ├── memory-search.ts / .schema.ts  # Cosine-similarity search over memories
    ├── schedule.ts / .schema.ts       # Create/list/delete cron jobs
    └── cron-utils.ts           # computeNextRunAt(), validateCronExpression()
```

## 3-Layer Context Management

The agent can run indefinitely without losing context. Three layers work together, all running **after** the response is sent to the user (fire-and-forget via `void runPostResponseTasks()`):

### Layer 1: Context Pruning (`context/context-pruning.ts`)

Runs **before** every LLM call. Trims large tool results to save tokens.

| Phase | Trigger | Action |
|-------|---------|--------|
| Soft trim | Context chars > 30% of window | Tool results > 4KB: keep first 1500 + `...[trimmed]...` + last 1500 chars |
| Hard clear | Context chars > 50% of window | Replace oldest tool results (> 50KB total) with `[Old tool result content cleared]` |

Protected zone: last 3 assistant turns are never pruned.

### Layer 2: Memory Flush (`compaction/memory-flush.ts`)

Runs **before** compaction when context is approaching the compaction threshold (`contextWindow - reserveTokens - FLUSH_SOFT_TOKENS`) and the flush hasn't yet run for the current compaction cycle. Performs a single non-streaming LLM call with only `memory_save` / `memory_search` tools and the recent conversation, prompting the model to persist any durable facts (user preferences, key decisions, ongoing task state) to the pgvector memory store before the conversation is summarized away.

Memories are stored in the `composio_claw_memory` table with 1024-dim vectors from OpenAI's `text-embedding-3-large` model. Flush failure is non-fatal - the next compaction cycle will retry.

### Layer 3: Compaction (`compaction/run-compaction.ts`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ComposioHQ/trustclaw](https://github.com/ComposioHQ/trustclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
