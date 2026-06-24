---
trigger: always_on
description: **Porrima** — A feature-rich agent framework and user interface with persistent memory, project context, image generation, and agentic tool execution. npm workspaces monorepo: `server/` (Express + TypeScript) and `client/` (React + Vite + Tailwind).
---

# AGENTS.md

## Project

**Porrima** — A feature-rich agent framework and user interface with persistent memory, project context, image generation, and agentic tool execution. npm workspaces monorepo: `server/` (Express + TypeScript) and `client/` (React + Vite + Tailwind).

## Quick Reference

- **Server port**: 3001 — `cd server && npm run dev` (tsx watch mode)
- **Client port**: 5173 — `cd client && npm run dev` (Vite, proxies `/api` to server)
- **Build server**: `cd server && npm run build` (outputs to `server/dist/`)
- **Build client**: `cd client && npm run build` (outputs to `client/dist/`)
- **Type check**: `npx tsc --noEmit` from either `server/` or `client/`
- **Data dir**: `~/.porrima/` (chats, projects, settings, memories, artifacts)
- **Models dir**: `~/.local/share/llama-models/` (symlinked GGUFs for llama.cpp router)
- **systemd services**:
  - `porrima.service` — main server (auto-starts on boot)
  - `llama-server.service` — llama.cpp router (port 32100, GPU inference)
  - `extraction-model.service` — memory extraction server (port 32101, CPU-only)
  - `reranker.service` — Qwen3-Reranker-0.6B (port 32102, CPU-only, memory retrieval)
  - `embedding-model.service` — embedding server (port 32103, CPU-only)
  - `title-generation.service` — title/recap server (port 32104, CPU-only)
  - `sync-llama-models.timer` — auto-syncs HuggingFace GGUF downloads every 5 min

## Architecture

See [docs/architecture.md](docs/architecture.md) for full details.

Three chat types: **agent** (memory-augmented), **quick** (standalone), and **system** (synthesis, wake cycles, and automations). The chat route (`server/src/routes/chat.ts`) owns memory augmentation, SSE/persistence, compaction, and extraction around the shared agent loop in `agent-loop-runner.ts`. Chat storage is SQLite with FTS5 full-text search. LLM system uses OpenAI-compatible (llama.cpp) backend for all inference.

## Tool System

See [docs/tool-system.md](docs/tool-system.md) for full details.

Native pi-ai tool calling with TypeBox schemas. Registry in `agent-tools.ts` with memory, filesystem, and sandbox tools. The low-level loop lives in `agent-loop-runner.ts`; the HTTP chat route and headless automation runner provide their own callbacks for transport, persistence, compaction, and follow-up prompts. `ask_user` pauses the HTTP loop and persists state. Message reconstruction splits persisted messages back into the pi-ai multi-message format.

## Memory System

See [docs/memory-system.md](docs/memory-system.md) for full details.

Two complementary memory systems: **atomic memories** (8 categories: preference, fact, behavior, instruction, context, decision, note, reflection) and **memory blocks** (structured, editable knowledge documents). Atomic memories are extracted automatically via LLM; blocks are agent-curated documents that organize knowledge by topic/project/domain.

Hybrid retrieval: vector search + FTS5 with RRF fusion, then cross-encoder reranking via Qwen3-Reranker-0.6B with chat-type-specific instructions. Memory blocks loaded by scope (global/project) with progressive disclosure — descriptions always in context, full content via `read_memory_block` tool. Extraction pipeline sees loaded blocks to prevent redundant extraction.

Indexed compaction archives full-fidelity messages in `context_archives` table with cross-chat FTS search. KV cache optimization uses delta-based memory injection — frozen memories in system prompt, new memories appended as delta messages to preserve longest-common-prefix caching. Key files: `memory-storage.ts`, `memory-extraction.ts`, `memory-context.ts`, `memory-tools.ts`, `reranker.ts`, `system-chat.ts`, `automation-storage.ts`, `automation-scheduler.ts`, `automation-runner.ts`, `chat-turn-runner.ts`, `agent-loop-runner.ts`, `llm-stream.ts`.

## Automations

See [docs/automations.md](docs/automations.md) for full details.

Automations are configurable recurring system-chat tasks. Built-ins cover synthesis and wake cycles; custom tasks support interval or daily schedules, order, activation policy, editable prompt steps, run history, and optional push notifications. Startup calls `ensureAutomationDefaults()` once, then `automation-scheduler.ts` checks due tasks every 5 minutes. Prompt text stays in user-role trigger/follow-up messages so the stable system-chat prefix remains KV-cache friendly.

See also: [docs/memory-blocks.md](docs/memory-blocks.md) for the block system details.

## Artifacts & Image Systems

See [docs/artifacts-and-images.md](docs/artifacts-and-images.md) for full details.

- **Artifacts**: `create_artifact` tool writes HTML to `~/.porrima/artifacts/`. Blob URLs for iframe src (critical for Chrome animation performance).
- **Image Corpus**: SQLite + sqlite-vec + FTS5. Hybrid search via RRF. Density-based clustering (0.85 threshold).
- **Corpus/Clustering**: SQLite corpus storage, enrichment, FTS/vector search, density-based clustering, and D3 visualization.
- **Image Generation**: ComfyUI integration with GPU/resource coordination for agent/tool-initiated image work.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [asa-degroff/Porrima](https://github.com/asa-degroff/Porrima) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
