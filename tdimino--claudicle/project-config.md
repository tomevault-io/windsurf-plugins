---
trigger: always_on
description: Open-source soul agent for Claude Code. Turns any Claude Code session into a persistent personality with three-tier memory, a cognitive pipeline, and channel adapters for Slack, SMS, and terminal. Pairs with any skill repo.
---

# Claudicle — Soul Agent Framework

Open-source soul agent for Claude Code. Turns any Claude Code session into a persistent personality with three-tier memory, a cognitive pipeline, and channel adapters for Slack, SMS, and terminal. Pairs with any skill repo.

## Stack
- Python 3.10+ (daemon, hooks, scripts, adapters)
- SQLite (three-tier memory: working, user models, soul state)
- Slack Bolt (Socket Mode for Slack integration)
- Claude Agent SDK (unified launcher mode)

## Structure
- `/subdaimones` — Sub-daimon definitions: 12 across 3 tiers (2 meta + 5 cognitive + 5 craft) with YAML frontmatter and structured protocols
- `/daimones` — Privy council: example daimon and Kothar (9 mental processes, Open Souls paradigm). User daimones live externally (e.g. `~/daimones/`)
- `/daemon` — Core: context assembly, soul engine, cognitive pipeline, memory, monitoring, monitor TUI, daemon lifecycle
- `/daemon/lifecycle.py` — Daemon lifecycle primitives: PID file (atomic write via tempfile+rename), liveness detection (`os.kill` + start_time), version handshake, `fcntl.flock` startup lock, `ensure_daemon()` fire-and-forget auto-start, `stop_daemon()` with SIGTERM→SIGKILL escalation
- `/daemon/VERSION` — Semantic version string (e.g. `0.15.0`), read by lifecycle.py, written by setup.sh
- `/daemon/cognitive_steps` — Cognitive step definitions (CognitiveStep dataclass, STEP_INSTRUCTIONS registry)
- `/daemon/daimonic/summoning.py` — Daimon summoning: awaken any entity (user model, dossier) as an ephemeral speaking daimon via Groq. `summon_entity()`/`dismiss_entity()`/`list_summoned()` API, cache trick (soul.md in memory, no filesystem writes)
- `/daemon/daimonic/whispers.py` — Daimonic intercession (external soul whispers into cognitive pipeline)
- `/daemon/processes` — Emotional state process handlers: `main_process.py` (default), `focused_process.py`, `frustrated_process.py`, `_shared.py` (shared prompt fragments)
- `/daemon/providers` — LLM provider implementations: `claude_cli.py`, `claude_sdk.py`, `anthropic_api.py`, `groq_provider.py`, `ollama_provider.py`, `openai_compat.py`
- `/daemon/monitoring` — Soul Monitor TUI (`monitor.py`), SQLite watcher (`watcher.py`), soul stream JSONL (`soul_log.py`), working memory stream (`wm_stream.py`)
- `/daemon/engine/onboarding.py` — First ensoulment mental process (4-stage interview state machine)
- `/daemon/engine/reflect.py` — Retrospective cognitive pipeline for terminal sessions (channel-agnostic reflection)
- `/daemon/engine/helpers.py` — Shared helpers: `extract_tag`, `strip_all_tags`, `store_and_emit` (extracted from soul_engine)
- `/daemon/engine/llm_client.py` — Shared LLM caller for reflection/compression (provider routing, API key resolution)
- `/daemon/engine/soul_engine.py` — Core soul engine: prompt building, cognitive cycle execution
- `/daemon/engine/pipeline.py` — Cognitive pipeline runner (multi-step XML-tagged processing)
- `/daemon/engine/context.py` — Context assembly shared across soul engine, pipeline, and reflection
- `/daemon/engine/perception.py` — Perception intake and routing for unified launcher
- `/daemon/engine/process_base.py` — Base class for mental processes (Open Souls paradigm)
- `/daemon/engine/process_router.py` — Intent classification and mental process routing
- `/daemon/engine/soul_path.py` — Soul personality file resolution (env var → symlink → fallback)
- `/daemon/engine/mycelium_bridge.py` — Mycelium bridge: file-level git notes read/write via mycelium.sh, best-effort subprocess pattern
- `/daemon/engine/compaction.py` — Context compaction for long-running sessions
- `/daemon/memory/compression.py` — Hypermnesia memory compression (heuristic/LLM, delegates to working_memory public APIs)
- `/daemon/memory/soul_state.py` — Unified soul state: topic stack (1 primary + 7 subtopics, FIFO cascade), emotional state transitions, timestamped audit log, narrative `soulStateShift` entries to working memory, `format_for_prompt()` with relative times and artifact references
- `/daemon/memory/snapshot.py` — Immutable data types (`MemoryEntry`, `WorkingMemorySnapshot`, `CognitiveOutput`), copy-on-write `with_*` methods, `load_snapshot()`/`apply_output()` boundary (routes soul state updates through `soul_state.set_state_key()`)
- `/daemon/memory/checkpoint.py` — Point-in-time bookmarks for working memory rollback (frozen `Checkpoint` dataclass, `wm_checkpoints` table)
- `/daemon/memory/daimon_memory.py` — Subdaimon persistent memory (context creation, load/store, lessons, communication logging, boot formatting)
- `/daemon/memory/daimon_output_parser.py` — Parse `## Memory Updates` markdown from subdaimon output into `CognitiveOutput` (pure `parse_output()` + deprecated `parse_and_store()` wrapper)
- `/daemon/memory/frontmatter.py` — Pure parsing for YAML frontmatter, `[[wiki links]]`, and `RAG:` tags. Single source of truth replacing duplicate parsers in user_models.py and usermodel_resolver.py

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tdimino/claudicle](https://github.com/tdimino/claudicle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
