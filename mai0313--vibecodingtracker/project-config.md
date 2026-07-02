---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Rust CLI (`vibe_coding_tracker`, short alias `vct`) that scans on-disk session logs written by five AI coding assistants — Claude Code, OpenAI Codex, GitHub Copilot CLI, and Gemini CLI (JSONL files), plus OpenCode (a SQLite database) — and aggregates them into two views:

- **`usage`** — per-model token counts and LiteLLM-priced cost
- **`analysis`** — per-model file-operation and tool-call metrics (read/write/edit lines, Bash/Edit/Read/Write/TodoWrite call counts)

Both subcommands support four output modes (interactive TUI / static table / plain text / JSON) and four time-range filters (`--daily` / `--weekly` / `--monthly` / `--all`). The interactive TUI is the default when no output flag is given.

Two auxiliary subcommands round out the CLI: `vct version` prints build/toolchain info (table default, plus `--json` / `--text`), and `vct update` self-replaces the binary from the matching GitHub release asset (`--check` to inspect availability only, `--force` to skip the confirmation prompt).

## Common commands

The toolchain is pinned to `rust-toolchain.toml` (1.95.0, edition 2024). On this machine `cargo` is **not** on the default `PATH`; export it before invoking any cargo command:

```bash
export PATH="$HOME/.cargo/bin:$PATH"
```

Standard `cargo build` / `cargo test` / `cargo bench` work as usual. Project-specific entries:

| Command                                     | What it does                                                                          |
| ------------------------------------------- | ------------------------------------------------------------------------------------- |
| `cargo build --profile dist --locked`       | Distribution build (fat LTO, single codegen unit) — used by release artifacts         |
| `cargo build --release --features mimalloc` | Opt-in mimalloc allocator (faster one-shot, ~10× higher RSS in TUI loops)             |
| `make fmt`                                  | `cargo fmt --all` + `cargo clippy --fix` + clippy with `-D warnings`                  |
| `uvx pre-commit run --all-files`            | Run all pre-commit hooks (whitespace, JSON/YAML/TOML, mdformat, gitleaks, shellcheck) |
| `uvx pre-commit install --install-hooks`    | Install the git hooks once after cloning                                              |

Criterion benchmarks live at `benches/benchmarks.rs`; reports land in `target/criterion`.

**Before every commit / PR**, always run `make fmt` and `uvx pre-commit run -a`. CI runs both with `-D warnings`, and the pre-commit hooks gate-keep the repo (gitleaks, mdformat, etc.).

## Architecture

### Two-stage pipeline

```
JSONL session file
        │
        ▼
src/session/        ← provider detection + per-provider parsers → CodeAnalysis
        │
        ▼
src/analysis/       ← roll up CodeAnalysis records → AggregatedAnalysisRow
src/usage/          ← roll up CodeAnalysis records → UsageResult / PerProviderUsage
        │
        ▼
src/display/        ← TUI / table / text / JSON renderers
```

`src/session/` owns the "raw bytes → typed `CodeAnalysis`" boundary so both `analysis` and `usage` consume the same parsed shape. Do **not** add direct file parsing to `src/usage/` or `src/analysis/`; route everything through `src/session/parser.rs`.

### Provider classification

`src/session/detector.rs` distinguishes the four providers by JSONL markers:

- **Gemini** — first line is a session-meta record with `sessionId` + `projectHash` and *no* `messages` array
- **Copilot CLI** — first line is `type == "session.start"` with `data.producer` starting with `"copilot"`
- **Claude Code** — any record carrying a `parentUuid` field
- **Codex** — any record whose `type` is one of `session_meta` / `turn_context` / `event_msg` / `response_item`, or default fallback when no other marker is found

Four parser entry points live in `src/session/parser.rs`:

- `parse_session_file(path) -> serde_json::Value` — untyped JSON wrapper used by `vct analysis --path` in `main.rs`. Returns the same shape as the golden fixtures under `examples/`.
- `parse_session_file_typed(path) -> CodeAnalysis` — typed, content-based auto-detection. **Only** for the CLI single-file path when no provider is known up front.
- `parse_session_file_typed_with_mode(path, mode)` — same as above but lets the caller choose `ParseMode::Full` vs `UsageOnly`.
- `parse_session_file_as(path, provider, mode)` — caller already knows the provider from the source directory. Use this from every directory walker — it eliminates the "metadata sentinel mis-classifies a Claude session as Codex" bug class. `provider` is the `Provider` enum defined in `src/models/provider.rs`.

`classify_records()` is the streaming-friendly variant that returns `None` on indeterminate records and lets callers keep peeking until a marker arrives — there is **no fixed peek window**, which is critical because a Claude metadata preamble (`permission-mode`, `file-history-snapshot`, …) can be arbitrarily long.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mai0313/VibeCodingTracker](https://github.com/Mai0313/VibeCodingTracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
