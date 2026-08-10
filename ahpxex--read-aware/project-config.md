---
trigger: always_on
description: - Product: `ReadAware`
---

## Project Context

- Product: `ReadAware`
- Type: AI-native reading application
- Core capability: context-rich reading and AI-assisted understanding
- Repo: `monorepo` managed by `Turborepo` over `bun` workspaces (`apps/*`, `packages/*`, `plugins/*`)
- Runtime shell: a `Tauri` desktop app **only** (the shipping target) that wraps the React web frontend. The web build is just Tauri's bundled frontend plus local dev / Storybook — not a standalone browser or PWA product
- Frontend: `React 19` as a client-rendered SPA (no SSR)
- Routing: `TanStack Router` (file-based, via the Vite router plugin)
- Styling: `Tailwind CSS v4` (tokens via `@theme` in `apps/web/src/index.css`)
- State management: `Jotai`
- Bundler: `Vite`
- Package manager: `bun`

## AI Architecture Decisions

> **Implementation status** (v0.3.0 — keep this block honest; the sections
> below describe **decided direction**, this one describes what actually runs):
>
> - Frontend-only monorepo (`apps/web` + `apps/desktop`); the Python backend is
>   gone.
> - **Persistence is SQLite**, not the old IndexedDB/localStorage interim layer.
>   IndexedDB survives only in one-time migration code and a font cache.
> - **Event sourcing is live for writes.** Every state change goes through
>   `commit_events`, which appends to `domain_events` and applies it to the
>   projections in ONE transaction (`storage/apply.rs`).
>   `rebuild_projections` replays the log into the tables;
>   `verify_projections` replays into scratch and diffs, so drift is
>   detectable rather than assumed absent.
> - **Known gap:** rows written before that landed still carry mutations the
>   log never recorded (a recolor, a memory reinforcement). `verify_projections`
>   reports them; they cannot be recovered, only outgrown.
> - Not built yet: the sync engine, and the consolidation pipeline behind
>   profile/entity events (they are logged but project to nothing).
> - Target on-device schema: `docs/data-model.md`. Current audit:
>   `docs/review-0.3.0.html`.

- Product architecture: single-agent system (one orchestrator over deterministic pipelines, not one LLM loop doing everything)
- User experience: the in-book chat is one persistent surface per book (prompt assembly is stateless per turn — continuity lives in the memory layer, not the transcript); the global (Context page) chat supports multiple user-created threads. Memory never splits per thread
- System model: memory-first, not transcript-first
- Deployment model: **local-first** — data and retrieval live on-device; the remote backend is a sync/relay layer, not where business logic lives
- Two independent axes — keep them separate:
  - **Data + retrieval: local** (on-device store + SQLite FTS; no vector store — see Storage Responsibilities)
  - **LLM inference: remote** (BYO API key or a thin proxy; no local model required)
- Frontend: a `React + TypeScript` SPA, shipped **only** inside the `Tauri` desktop app (desktop-only)
- On-device storage: `SQLite` only (source of truth + FTS retrieval). **No embeddings / vector store in the default architecture** (decided 2026-07-02, see `docs/agent-architecture.md` §4)
- Remote backend: sync + relay only (see Storage Responsibilities)

### Agent Model

- ReadAware uses one core agent that orchestrates the product's intelligence
- This agent is responsible for:
  - building and updating the user's profile
  - updating user memory over time
  - retrieving relevant book notes, highlights, and prior conversations
  - assembling the right context for the current reading moment
- Do not model the product as multiple user-visible agents unless the product direction explicitly changes

### Memory and Context

- The core system problem is memory management, not chat history management
- User-visible chat should feel continuous, but the system should not rely on dumping all prior messages into the prompt
- Treat chat transcripts as raw source material, not as the memory layer itself
- Memory is **event-sourced**. Model it in layers:
  - `raw events` — append-only, immutable; **this is the unit of sync** (see Storage Responsibilities)
  - working memory — local projection
  - long-term user memory — local projection
  - book / highlight / note memory — local projection
  - exportable context bundles — local projection
- Everything above `raw events` is a **local projection rebuilt from the event log** — projections are recomputed on-device, never synced directly. This is enforced, not aspirational: `storage/apply.rs` is the only writer of a projection row, and `rebuild_projections` can reproduce every one of them from the log. Two things are deliberately NOT derived, and both are excluded from the check: book cover artwork (extracted from object-storage content) and chat presentation state (`parts_json`, `error`)
- Design the **write / consolidation pipeline** as explicitly as retrieval; it is the harder half:
  - promotion from raw events into long-term memory (summarization / consolidation)
  - conflict resolution when new information contradicts old memory
  - decay / forgetting so memory does not grow into noise
  - dedup / entity resolution behind "repeated appearance across books or conversations"
- Memory retrieval should consider more than text-match relevance, including:
  - relevance to the current reading goal

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ahpxex/read-aware](https://github.com/ahpxex/read-aware) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
