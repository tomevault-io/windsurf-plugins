---
trigger: always_on
description: Phantom is an event-sourced, semantic-aware version control layer for agentic AI development, built on top of Git. It enables multiple AI coding agents to work on the same codebase simultaneously with automatic symbol-level conflict detection, FUSE-based filesystem isolation, and instant propagation of finished work.
---

# CLAUDE.md — Phantom

## What is Phantom

Phantom is an event-sourced, semantic-aware version control layer for agentic AI development, built on top of Git. It enables multiple AI coding agents to work on the same codebase simultaneously with automatic symbol-level conflict detection, FUSE-based filesystem isolation, and instant propagation of finished work.

Written in Rust (edition 2024, `rust-version = "1.88"`). Linux-first (FUSE). MIT license.

## Quick Reference

```bash
# Build and test
cargo build && cargo test

# System dependency (Linux)
sudo apt install libfuse3-dev pkg-config build-essential

# Install the binary
cargo install --path crates/phantom-cli

# Usage
ph init                              # Initialize in a git repo
ph <agent-name>                      # Create/resume agent overlay (interactive)
ph <agent-name> --background         # Run agent in background
ph plan "add caching layer"          # Decompose feature into parallel agents
ph plan --from phantom-plan-<id>.md  # Dispatch a previously-saved plan (no re-planning)
ph submit <agent>                    # Submit overlay: semantic merge to trunk, ripple to agents
ph resolve <agent>                   # Auto-resolve conflicts via AI agent
ph resume                            # Select and resume an interactive agent session
ph tasks                             # List all agent task overlays
ph status                            # Show overlays, changesets, queue
ph log [filter]                      # Query event log (agent name or cs-id)
ph changes                           # Recent submits and materializations
ph background                        # Watch background agents
ph remove <agent>                    # Tear down overlay (immediate, no prompt)
ph rollback [changeset-id]           # Drop changeset, replay downstream
ph down                              # Unmount all overlays, remove .phantom/ (prompts unless -f)
```

## Workspace Structure

9 crates + 1 integration test crate:

```
crates/
├── phantom-core/           # Core types, traits, errors (zero deps on other phantom crates)
├── phantom-git/            # Git operations (git2 wrapper, tree building, text merge)
├── phantom-events/         # SQLite WAL event store (sqlx)
├── phantom-overlay/        # FUSE overlay filesystem (fuser, feature-gated)
├── phantom-semantic/       # Tree-sitter parsing + semantic merge engine
├── phantom-orchestrator/   # Materializer, ripple, live rebase, submit service (uses phantom-git)
├── phantom-session/        # PTY management, CLI adapters, context files, post-session automation
├── phantom-cli/            # Binary crate — the `phantom` command
└── phantom-testkit/        # Shared test utilities (builders, mocks, test repos)

tests/integration/          # End-to-end tests with real git repos
```

## Crate Responsibilities

### phantom-core (`crates/phantom-core/`)
Zero dependencies on other phantom crates. Defines all shared types:
- **IDs**: `ChangesetId`, `AgentId`, `EventId`, `SymbolId`, `ContentHash` (BLAKE3), `GitOid` (20-byte, no git2 dependency), `PlanId`
- **Changeset**: status lifecycle (`InProgress → Submitted/Conflicted/Resolving/Dropped`), `SemanticOperation` (AddSymbol/ModifySymbol/DeleteSymbol/AddFile/DeleteFile/RawDiff), `TestResult`
- **Event**: `EventKind` enum with 17+ variants including `TaskCreated`, `ChangesetSubmitted`, `ChangesetMaterialized`, `LiveRebased`, `PlanCreated`, `AgentLaunched/Completed`, `Unknown` (forward-compat via `serde(other)`)
- **Conflict**: `ConflictDetail` with `ConflictKind` (BothModifiedSymbol, ModifyDeleteSymbol, BothModifiedDependencyVersion, RawTextConflict, BinaryFile) and `ConflictSpan` (byte ranges + line numbers)
- **Traits**: `EventStore` (async), `SymbolIndex`, `SemanticAnalyzer` (extract_symbols, diff_symbols, three_way_merge)
- **Plan**: multi-domain task decomposition (`Plan`, `PlanDomain`, `PlanStatus`)
- **Notification**: `TrunkNotification` with per-file `TrunkFileStatus` (TrunkVisible/Shadowed/RebaseMerged/RebaseConflict)

### phantom-git (`crates/phantom-git/`)
Git operations built on `git2`. Depends only on `phantom-core` and `git2` — no event store, semantic analysis, or overlay dependencies.
- `GitOps`: thin wrapper around `git2::Repository` — `head_oid()`, `read_file_at_commit()`, `changed_files()`, `revert_commit_oid()`, `reset_to_commit()`, `text_merge()`
- `GitError`: typed error enum for git operations
- `tree`: tree building from blobs/overlay — `build_tree_from_oids()`, `create_blobs_from_overlay()`, `create_blobs_from_content()`
- `oid_to_git_oid` / `git_oid_to_oid`: lossless conversions between `GitOid` and `git2::Oid`
- `test_support`: test repo helpers (`init_repo`, `advance_trunk`, `commit_file`)

### phantom-events (`crates/phantom-events/`)
SQLite WAL-mode event store via `sqlx`. Schema versioned (currently v2 with `kind_version` column).
- `SqliteEventStore`: implements `EventStore` trait, supports `in_memory()` and `open(path)`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Maelwalser/phantom](https://github.com/Maelwalser/phantom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
