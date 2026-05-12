---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Hermes Context Manager (HMC) is a plugin for the Hermes agent gateway that automatically optimizes conversation context. It uses a **silent-first** architecture: most of the pipeline runs with no main-model involvement (pattern matching, code-aware compression, deduplication, truncation, error purging). Background compression via an auxiliary model handles the rest. The main model never sees compression tools or nudges.

HMC runs **in tandem** with Hermes's built-in `ContextCompressor` — it's a layered per-tool-call / per-turn silent compressor, not a replacement. Both run. The `ContextEngine` abstract base class in `agent.context_engine` is the wrong interface for this plugin: that's a mutually-exclusive single-select slot for engines that do one big summarization pass on token pressure. HMC's compression happens continuously at four hook points and cannot be expressed as a single `compress(messages)` callback. If a future contributor is tempted to migrate HMC to `ContextEngine`, don't.

## Commands

```bash
# Run all tests
python -m unittest discover -s tests -v

# Run a single test file
python -m unittest tests/test_engine.py -v

# Run a single test case
python -m unittest tests.test_engine.TestMaterialize.test_dedup_identical_reads -v

# Install locally for development
hermes plugins install file:///absolute/path/to/hermes-context-manager
hermes gateway restart
```

No linter or formatter is configured. No Makefile.

## Architecture

### Two-tier pipeline

The plugin hooks into Hermes's lifecycle via four hooks (`pre_tool_call`, `post_tool_call`, `pre_llm_call`, `on_session_end`). Compression runs at three of the four hook points:

- **`post_tool_call`** — single-message strategies (short_circuit, code_filter, truncation) run in place on the new tool output as it lands, so agent loops between user turns stay compressed continuously. Publishes a `tool` SSE event.
- **`pre_llm_call`** — full `materialize_view` pipeline: all single-message strategies plus dedup and error_purge (which need the whole conversation). Triggers background compression if the context threshold is hit. Publishes a `turn` SSE event.
- **`on_session_end`** — runs a final materialize pass to catch tail-end tool outputs that never saw a `pre_llm_call` sweep, then restores the raw conversation so Hermes saves the unmutated session file. Writes the analytics row and publishes a `session_end` SSE event.

**Layer 0 — Silent strategies** (engine.py, zero model involvement). The pipeline in `materialize_view`:

1. **Short-circuit pattern matching** (`short_circuits.py`) — compresses known output shapes (JSON success, test results, git output) to one-liners. Errors are never short-circuited.
2. **Code-aware compression** (`code_filter.py`) — strips function/class bodies from tool outputs containing source code, preserving signatures, imports, and docstrings. Supports Python (indentation-aware), Rust, Go, JS, TS. String-aware brace counting handles `"hello {world}"` literals. JSX bailout for React safety.
3. **Head/tail truncation** (`truncation.py`) — keeps first/last lines of long outputs with a gap marker.
4. **Deduplication** — fingerprints tool inputs (`tool_name::sorted_args`), suppresses repeated reads of same content. `write_file` and `patch` are always protected. Second pass hashes normalized content.
5. **Error purging** — removes tool error outputs older than N turns.
6. **Tool-output pruning** — mutates pruned content to placeholders and credits savings to the `dedup` or `error_purge` bucket.
7. **Message ID snapshot** — builds an in-memory `{m001: timestamp, ...}` mapping for internal addressing. Does NOT mutate message content.

`apply_strategies_to_tool_output` wraps strategies 1-3 for the `post_tool_call` path; `materialize_view` runs all seven for `pre_llm_call` and `on_session_end`.

**Layer 1 — Background Compression** (`background_compressor.py`):
- Triggers when context usage > 80% (`max_context_percent`)
- Uses Hermes auxiliary model (not main model) for summarization
- Three-phase lock pattern: locked prepare → unlocked LLM call → locked commit
- Builds searchable index entries at `~/.hermes/hmc_state/{session}_index.jsonl`
- Deletes compressed messages from context

**Layer 2 — Passive Context** (`prompts.py`):
- One-liner system context injected via `pre_llm_call` return value
- Lists indexed phase topics when the session has any
- Does NOT advertise any tool the plugin doesn't register

### Persistent Analytics (`analytics.py`)

Cross-session, cross-project SQLite store at `~/.hermes/hmc_state/analytics.db`. One row per `(session, strategy)` is written at `on_session_end`. WAL mode + `synchronous=NORMAL` + 5s busy_timeout + on-write 90-day TTL. Queryable via the `hmc_control analytics` action with `scope`, `period`, and `limit` args.

### Opt-In Web Dashboard (`dashboard.py`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [entrepeneur4lyf/hermes-context-manager](https://github.com/entrepeneur4lyf/hermes-context-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
