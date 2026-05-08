---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test

```bash
cargo build                    # Build
cargo test                     # All 179 tests
cargo test config              # Tests in a specific module
cargo test test_parse_minimal  # Single test by name
cargo clippy -- -D warnings    # Lint (CI-style)
cargo fmt --check              # Check formatting
```

Run with debug logging:
```bash
RUST_LOG=yoclaw=debug cargo run
```

## Architecture

yoclaw is a single-binary AI agent orchestrator built on **yoagent v0.5.1** (git dep: `github.com/yologdev/yoagent.git`). It connects LLMs to messaging platforms with SQLite persistence.

### Core flow

```
Channel (Telegram/Discord/Slack) → MessageCoalescer (debounce) → Queue (SQLite) → Conductor → Agent (yoagent) → Response → Channel
```

### Key constraint

`Agent::prompt()` takes `&mut self` — only one session processes at a time. The Conductor switches sessions by saving/loading conversation state to the tape table (`save_messages` → `clear_messages` → `restore_messages`). This means concurrent messages are queued and processed sequentially. Fine for personal/small-team use, but does not scale horizontally. Scaling would require running multiple yoclaw instances, each with its own agent.

### Module responsibilities

- **conductor/** — Owns the yoagent `Agent`. Handles session switching, streams `AgentEvent` via `stream_response()`, persists to tape. `resolve_provider()` returns `DynProvider(Box<dyn StreamProvider>)` to support multiple LLM providers (anthropic, openai, google, vertex, azure, bedrock, openai_responses). `delegate.rs` builds `SubAgentTool` workers from config. `tools.rs` implements `MemorySearchTool`/`MemoryStoreTool`, `SpawnWorkerTool`/`ListWorkersTool`/`RemoveWorkerTool` for dynamic workers. `direct_workers` HashMap enables direct worker delegation bypassing the main agent.
- **channels/** — `ChannelAdapter` trait (`Send + Sync`, stored as `Arc<dyn ChannelAdapter>`) for messaging platforms. `telegram.rs` (teloxide), `discord.rs` (serenity), `slack.rs` (Socket Mode). `coalesce.rs` debounces rapid messages per session with per-channel configurable debounce. Trait includes `send_placeholder()`/`edit_message()` for streaming support.
- **db/** — `Db` wraps `Arc<Mutex<Connection>>`. All methods use `spawn_blocking` for async safety. Tables: tape, queue, memory (+ FTS5), audit, state, cron_jobs, cron_runs, saved_workers. `vector.rs` (behind `semantic` feature flag) provides `EmbeddingEngine` (embedding-gemma-300m) and sqlite-vec KNN search; `memory.rs` uses RRF (Reciprocal Rank Fusion) to merge FTS5 and vector results, then applies temporal decay weighted by RRF scores.
- **scheduler/** — Unified scheduler for cortex maintenance and cron jobs. `cortex.rs` handles memory dedup, stale cleanup, consolidation, session indexing. `cron.rs` runs due jobs via ephemeral or persistent agents based on session mode. `tools.rs` provides `CronScheduleTool` for conversational cron management.
- **security/** — `SecureToolWrapper` wraps every `AgentTool`, checks `SecurityPolicy` before delegating. `BudgetTracker` uses `AtomicU64` for sync compatibility with yoagent's `on_before_turn` callback. `injection.rs` provides 3-layer detection: L1 pattern matching (35 patterns), L2 `HeuristicScorer` (6 signals, 0.0–1.0 score), L3 optional async `LlmJudge`. `heuristics.rs` uses `OnceLock` for regex compilation.
- **skills/** — Loads `SKILL.md` files, parses `tools` from YAML frontmatter, filters out skills requiring disabled tools.
- **web/** — Embedded web UI via rust-embed (`web/dist/`). Axum server with REST API (`/api/sessions`, `/api/queue`, `/api/budget`, `/api/audit`) and SSE (`/api/events`). SSE events include `StreamChunk` and `StreamEnd` for real-time streaming to web clients.
- **config.rs** — TOML parsing with `${ENV_VAR}` expansion and `~` tilde expansion.
- **migrate.rs** — Migration from OpenClaw installations (persona, skills, memories).

### yoagent integration

- `AgentEvent` is NOT `Serialize` — tape stores `Vec<AgentMessage>` snapshots (which IS Serialize)
- `on_before_turn` / `on_after_turn` callbacks are **sync** — hence `AtomicU64` for budget tracking
- `AgentTool::execute()` uses v0.5.1 signature: `execute(params, ctx: ToolContext)` where `ToolContext` bundles `tool_call_id`, `tool_name`, `cancel`, `on_update`, `on_progress`
- Workers use `SubAgentTool` (ephemeral: fresh `agent_loop` per invocation)
- Direct worker delegation: `delegate_to_worker` calls `SubAgentTool::execute` directly, persists exchange to tape, invalidates session
- Ephemeral agents: `run_ephemeral_prompt()` in `scheduler/mod.rs` uses `agent_loop` directly for cron/cortex tasks; `AgentLoopConfig` requires `input_filters` field
- Persistent agents: `run_persistent_prompt()` loads prior conversation from tape, runs `agent_loop` (max 5 turns), saves back — used by cron jobs with `session_mode = "persistent"`
- Default tools from `yoagent::tools::default_tools()` are wrapped with `SecureToolWrapper`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yologdev/yoclaw](https://github.com/yologdev/yoclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
