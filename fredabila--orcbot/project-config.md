---
trigger: always_on
description: OrcBot is a TypeScript/Node.js autonomous AI agent that can chat over Telegram, WhatsApp, Discord, and a web gateway. It plans multi-step tasks, executes tools (skills), browses the web, and learns from its interactions. All state is file-backed under `~/.orcbot/` by default.
---

# OrcBot AI Coding Instructions

## Project Overview
OrcBot is a TypeScript/Node.js autonomous AI agent that can chat over Telegram, WhatsApp, Discord, and a web gateway. It plans multi-step tasks, executes tools (skills), browses the web, and learns from its interactions. All state is file-backed under `~/.orcbot/` by default.
---

## Architecture Map

| Layer | File(s) | Role |
|-------|---------|------|
| **CLI entrypoint** | [src/cli/index.ts](src/cli/index.ts) | Wires all subsystems, TUI menus, user commands |
| **Core agent** | [src/core/Agent.ts](src/core/Agent.ts) | Orchestrates memory, LLM, skills, channels, scheduling; runs the main action loop |
| **Decision engine** | [src/core/DecisionEngine.ts](src/core/DecisionEngine.ts) | Assembles context + prompt, calls LLM, validates response, runs termination review |
| **Modular prompts** | [src/core/prompts/](src/core/prompts/) | 8 task-specific helpers + PromptRouter; only relevant helpers are injected per task |
| **Simulation/planning** | [src/core/SimulationEngine.ts](src/core/SimulationEngine.ts) | Produces a pre-plan before execution starts |
| **Parser** | [src/core/ParserLayer.ts](src/core/ParserLayer.ts) | Normalizes raw LLM text into structured JSON (3-tier fallback) |
| **Context compactor** | [src/core/ContextCompactor.ts](src/core/ContextCompactor.ts) | Truncation and LLM-based summarization for oversized context |
| **Decision pipeline** | [src/core/DecisionPipeline.ts](src/core/DecisionPipeline.ts) | Post-parse guardrails on tool calls |
| **Memory** | [src/memory/MemoryManager.ts](src/memory/MemoryManager.ts) | Short/episodic/long memory in JSON, consolidation, daily markdown logs |
| **Action queue** | [src/memory/ActionQueue.ts](src/memory/ActionQueue.ts) | Priority-sorted durable queue with retry, chaining, TTL, stale recovery |
| **Storage** | [src/storage/JSONAdapter.ts](src/storage/JSONAdapter.ts) | Atomic JSON persistence with backup/recovery |
| **LLM routing** | [src/core/MultiLLM.ts](src/core/MultiLLM.ts) | Routes by model name (gemini→Google, nvidia:→NVIDIA, else OpenAI), auto-fallback |
| **Skills manager** | [src/core/SkillsManager.ts](src/core/SkillsManager.ts) | Registry, dynamic plugin loading, skill matching |
| **Channels** | [src/channels/](src/channels/) | Telegram (Telegraf), WhatsApp (Baileys), Discord (discord.js), Gateway (Express+WS) |
| **Browser** | [src/tools/WebBrowser.ts](src/tools/WebBrowser.ts) | Playwright with semantic snapshots, Serper search, 2Captcha |
| **Scheduler** | [src/core/Scheduler.ts](src/core/Scheduler.ts) | Cron-based via croner, emits `scheduler:tick` on EventBus |
| **Config** | [src/config/ConfigManager.ts](src/config/ConfigManager.ts) | YAML hot-reload, feature toggles, API keys |
| **Token tracking** | [src/core/TokenTracker.ts](src/core/TokenTracker.ts) | Per-model token/cost tracking and budgets |
| **Runtime tuner** | [src/core/RuntimeTuner.ts](src/core/RuntimeTuner.ts) | Auto-adjusts limits based on runtime signals |

---

## Memory System (Critical — Read This First)

Memory is the most complex subsystem and the #1 source of agent behavior bugs. Understand these layers:

### Storage Layer
- **JSONAdapter** ([src/storage/JSONAdapter.ts](src/storage/JSONAdapter.ts)): Atomic writes (temp→rename), `.bak` crash recovery, in-memory cache. Every `saveMemory` serializes the full array — keep memory count reasonable.
- **DailyMemory** ([src/memory/DailyMemory.ts](src/memory/DailyMemory.ts)): Append-only markdown files (`YYYY-MM-DD.md`). Good for auditing but **not read by the decision engine** — only accessible via `memory_search`/`memory_get` skills.
- **VectorMemory** ([src/memory/VectorMemory.ts](src/memory/VectorMemory.ts)): Embedding-based semantic index for similarity search. File-backed JSON (`vector_memory.json`) with atomic writes. Uses OpenAI `text-embedding-3-small` (256 dims) or Google `text-embedding-004` as fallback. Gracefully disabled when no embedding API key is configured.

### Memory Types
| Type | Lifespan | Purpose |
|------|----------|---------|
| `short` | Until consolidation (~30 entries) | Step observations, tool results, system injections, inbound messages |
| `episodic` | Permanent (last 5 shown in context) | LLM-generated summaries of consolidated short memories; action conclusions |
| `long` | Permanent | Rarely used directly; DailyMemory's MEMORY.md is the long-term store |

### How Memory Flows into LLM Prompts
1. `getRecentContext()` returns the 20 most-recent short memories + last 5 episodic summaries
2. DecisionEngine splits these into:
   - **Step History** — entries with `{actionId}-step-*` prefix → ground truth for current task
   - **Other Context** — up to 5 recent memories from other sources (filtered to exclude cross-action `[SYSTEM:]` injections)
   - **Thread Context** — from `searchMemory('short')`, filtered by source+chatId, ranked by semantic similarity (vector memory) or keyword overlap (fallback) → the conversation thread
3. Joins with: core instructions, channel context, user profile, journal tail, learning tail


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fredabila/orcbot](https://github.com/fredabila/orcbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
