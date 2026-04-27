---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Koda is a personal AI assistant built in Rust (edition 2024) — not a product,
not a platform. See [DESIGN.md](DESIGN.md) for principles (P1: Personal,
P2: Simple enough to own alone, P3: Build for the world six months from now).

**Development philosophy:** Make it work, make it right, make it fast — in that
order. Ship working code first, refactor to clean design second, optimize for
performance only when measured.

Four-crate workspace:
- `koda-core` (library) — pure engine with zero terminal deps
- `koda-cli` (binary `koda`) — CLI frontend with ratatui TUI
- `koda-ast` (binary `koda-ast`) — tree-sitter AST analysis (library + standalone MCP server)
- `koda-email` (binary `koda-email`) — email via IMAP/SMTP (library + standalone MCP server)

See [DESIGN.md](DESIGN.md) for architectural decisions. See [#70](https://github.com/lijunzh/koda/issues/70) for the TUI design.

### Current Architecture

Simple inference loop: stream LLM response → execute tool calls → repeat.
No phases, no tiers — the model drives execution directly.

- **Context from API** (`providers/`): queries actual context window from provider
  - Fallback: `model_context.rs` lookup table
- **Rate limit retry**: exponential backoff (2/4/8/16/32s) for 429 errors
- **Built-in agents**: default (others via user-created agent configs)
- **Git checkpointing** (`git.rs`): auto-snapshot before each turn

Approval is per-tool. Two modes (Auto/Confirm) control
how mutations are gated:

- **ToolEffect** (`approval.rs`): ReadOnly / LocalMutation / Destructive / RemoteAction
  - Auto: local mutations auto-approved, destructive need confirmation
  - Confirm: every non-read action needs confirmation
- **Hardcoded floors**: destructive ops and outside-project writes always need
  confirmation regardless of mode
- **Folder scoping** (`approval.rs`, `bash_safety.rs`):
  - `is_outside_project()`: checks file tool paths against project_root
  - `lint_bash_paths()`: heuristic bash command analysis for cd/path escapes
  - Startup warning when project_root == $HOME

## Documentation Rules

**When to update docs with a PR:**
- User-facing feature added/changed → update root README + relevant crate README
- Tool added/changed in koda-ast/koda-email → update the crate README's tool/protocol section
- Architecture or design decision → add to the appropriate section in `DESIGN.md` with rationale
- New crate → must ship with a README.md (required for crates.io)
- Keep feature coverage symmetric — if AST and email have equivalent capabilities, they get equivalent documentation
- Internal refactors don't require doc updates unless they change crate boundaries or public APIs

**On release:**
- Move CHANGELOG.md `[Unreleased]` to versioned section
- Bump version in all 4 crate Cargo.toml files (koda-core, koda-cli, koda-ast, koda-email)
- Verify README quick-start examples still work
- Check that CHANGELOG entries match what's documented in README/DESIGN.md

## Build & Development Commands

```bash
cargo build                              # Debug build (all crates)
cargo build --release -p koda-cli        # Release build (CLI only)
cargo test --workspace --features koda-core/test-support  # Run all tests (incl. E2E)
cargo test -p koda-core --features test-support          # Engine tests only
cargo test -p koda-cli                                   # CLI tests only
cargo test -p koda-ast                                   # AST library + server tests
cargo test -p koda-email                                 # Email library + server tests
cargo test -p koda-core --test perf_test                 # Run a specific test file
cargo fmt --all                          # Format all crates
cargo fmt --all --check                                                    # Check formatting (CI enforced)
cargo clippy --workspace --all-targets --features koda-core/test-support -- -D warnings  # Lint (CI enforced)
cargo check --workspace --all-targets                                       # Compile-check all targets, no features (CI enforced, ubuntu + windows)
cargo doc --workspace --no-deps                                             # Build docs
```

## Architecture

### Workspace

```
koda/
├── Cargo.toml              # Workspace root
├── koda-core/              # Engine library (zero terminal deps)
│   ├── src/
│   │   ├── lib.rs          # Crate root
│   │   ├── agent.rs        # KodaAgent (shared config: tools, prompt)
│   │   ├── approval.rs     # Approval modes + tool confirmation gates
│   │   ├── bash_path_lint.rs # Heuristic path-escape detection for bash commands
│   │   ├── bash_safety.rs  # Bash command safety classification
│   │   ├── bg_agent.rs     # Background sub-agent registry
│   │   ├── compact.rs      # Session compaction (summarize old messages)
│   │   ├── config.rs       # Agent/provider config + provider metadata
│   │   ├── context.rs      # Context window token tracking
│   │   ├── db/             # SQLite persistence (split into submodules)
│   │   │   ├── mod.rs      # Database struct, migrations, schema (WAL mode)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lijunzh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
