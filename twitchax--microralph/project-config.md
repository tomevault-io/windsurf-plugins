---
trigger: always_on
description: This document provides detailed workflows and troubleshooting for AI coding agents working in this repository.
---

# microralph — Agents Guide

This document provides detailed workflows and troubleshooting for AI coding agents working in this repository.

## Workspace Overview

This project is a Cargo workspace with the root binary crate and optional sub-crates:

- `src/`: Main Rust source code (root `microralph` crate — the `mr` binary)
  - `commands/`: CLI command implementations (bootstrap, devcontainer, graph, init, refactor, reindex, run, status, suggest, validate, worktree)
  - `config/`: Configuration loading and constitution editing
  - `prd/`: PRD types, parsing, indexing, and operations (edit, new, finalize)
  - `prompt/`: Prompt loading and expansion
  - `runner/`: Runner implementations (copilot, claude, codex, mock)
  - `util/`: Shared utilities (colors, spinner, qa_workflow)
  - `worktree/`: Worktree orchestration (types, state, git helpers, IPC, daemon)
  - `main.rs`: CLI entry point
  - `changelog.rs`: Changelog generation
- `crates/mr-ui/`: UI dashboard crate (feature-gated behind `ui` feature in root)
  - Built with Leptos 0.8, Axum 0.8, and Thaw UI 0.5
  - Feature-gated: only compiled when `--features ui` is passed
  - Has `ssr` and `hydrate` features for server and client compilation
- `.mr/`: microralph state directory
  - `prds/`: PRD files
  - `templates/`: PRD templates
  - `prompts/`: Static prompt files for each stage
  - `skills/`: Agent-managed persistent skills (learned techniques reused across runs)
  - `PRDS.md`: Auto-generated PRD index
  - `worktrees/`: Worktree orchestration state (state.yaml, daemon.sock, daemon.pid)

## Quick Start

```bash
# Build
cargo build

# Test
cargo make test

# Full CI (fmt, clippy, test)
cargo make ci

# UAT (the one true gate)
cargo make uat

# Dev container (start and exec into container)
cargo make devcontainer
```

## PRD Creation Workflow (`mr new`)

The `mr new` command creates a new PRD through a single-phase interactive flow:

1. **Interactive Session**: The user is dropped directly into an interactive chat session with the underlying agent (Copilot or Claude). The agent has full project context (existing PRDs, constitution, codebase scan) injected via the prompt — along with the next PRD ID and target file path. The agent gathers info from the user, writes the PRD file directly to disk in `.mr/prds/`, and tells the user to exit.
2. **Validation**: On clean exit, the Rust side scans `.mr/prds/` for the newly created PRD file, validates it, and regenerates the index. If no file was created, a placeholder PRD is generated.
3. **Abort on Ctrl+C**: If the user force-quits (Ctrl+C / SIGINT) during the interactive session, PRD creation is aborted entirely — no partial PRD is created.

### Runner Interactive Mode

The `Runner` trait provides `execute_interactive(prompt, working_dir)` which spawns the CLI with `Stdio::inherit()` for direct user interaction. The agent writes the PRD file directly to disk during this session.

### Error Handling

- **`RunnerError::Interrupted`**: Returned when the interactive process is killed by a signal (e.g., SIGINT from Ctrl+C). Detected on Unix via `ExitStatusExt::signal()`.
- **`RunnerError::ProcessFailed`**: Returned for non-zero exit codes without signal interruption.
- Both error types abort PRD creation entirely with no file written.

### Prompts

- **Interactive prompt** (`prd_new_interactive.md`): Single prompt that instructs the agent to gather information from the user interactively and then write the PRD file directly to disk. Includes `{{next_id}}`, `{{prd_path}}`, and `{{slug}}` placeholders so the agent knows where to write. Defined in `src/commands/init.rs` as `PROMPT_PRD_NEW_INTERACTIVE`.

### Important Notes

- **Single-phase architecture**: The agent both gathers info and writes the PRD during the interactive session. There is no separate synthesis phase.
- **No Q/A workflow**: The old multi-round Q/A loop has been fully removed. There is no `--legacy` or `--non-interactive` fallback.
- **Prompt management**: All prompts are defined in `src/commands/init.rs` and materialized to `.mr/prompts/` per constitution rule 7.
- **Mock testing**: `MockRunner` supports `set_interactive_error()` for testing error paths without requiring actual CLI tools.

## PRD Edit Workflow (`mr prd edit`)

The `mr prd edit` command modifies an existing PRD through a single-phase interactive flow, matching the pattern established by `mr new`:

1. **Interactive Session**: The user is dropped into an interactive chat session with the underlying agent. The agent receives full context — existing PRD content, constitution, existing PRDs list, the PRD file path, and optional user-provided `--context`. The agent discusses changes with the user and writes the updated PRD directly to disk (overwriting the original file).
2. **Validation**: On clean exit, the Rust side re-reads the PRD file, validates it, and regenerates the index.
3. **Abort on Ctrl+C**: If the user force-quits (Ctrl+C / SIGINT) during the interactive session, the edit is aborted — the original PRD is preserved unchanged.

### Usage

```bash
# Edit a PRD interactively
mr prd edit PRD-0001

# Edit with upfront context to guide the session
mr prd edit PRD-0001 --context "add a new task for logging"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [twitchax/microralph](https://github.com/twitchax/microralph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
