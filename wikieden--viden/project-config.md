---
trigger: always_on
description: This is the canonical repository-wide instruction file for coding agents and
---

# Viden Agent Guide

## How To Use This Guide

This is the canonical repository-wide instruction file for coding agents and
automation. Tool-specific entry files such as root `CLAUDE.md` must point here
instead of maintaining a competing copy of project policy.

Instruction precedence is:

1. the user's current request;
2. this repository-root `AGENTS.md`;
3. the nearest nested `AGENTS.md` for files being changed;
4. durable architecture, design, and development documents linked below.

Read every applicable instruction file before editing. When a task crosses
Core, TUI, GUI, or the design package, split ownership or explicitly reconcile
the nested rules before writing. Never assume a historical branch description
or chat summary is current; verify Git and the referenced source-of-truth files.

Key sources:

- roadmap and release sequencing: `PLAN.md` and `docs/staged-roadmap.md`;
- V3 branch topology: `docs/parallel-development-plan.md`;
- architecture and module boundaries: `docs/architecture.md` and
  `docs/modules.md`;
- coding, documentation, and comment standard:
  `docs/development-standards.md`;
- frontend contract: `docs/frontend-integration-contract.md`;
- visual adoption rules: `docs/viden-design-adoption.md` and the nested
  instructions under `docs/viden-design/Viden/`.

## Mission

Viden is a Rust-first, local-first agentic developer workspace inspired by
`.ref/claude-code-main`. Treat the reference project as a behavioral guide, not
as a file-by-file port. Preserve user-facing runtime patterns where valuable,
but keep the implementation Rust-native and simpler than the reference when the
extra platform machinery is not yet needed.

## Current Architecture

Workspace code is split by product surface and reusable core:

- `apps/cli`: binary entrypoint, flags, and bootstrap.
- `apps/tui`: terminal rendering, input orchestration, previews, and app-specific TUI state.
- `apps/gui`: planned desktop client boundary; currently governed by its nested
  `AGENTS.md` and must remain framework-neutral until the GUI gate passes.
- `crates/core`: stable runtime facade and shared contract re-exports.
- `crates/context`: native context selection, immutable content references,
  retrieval, compaction, quality, and cost accounting.
- `crates/runtime`: session engine, slash commands, provider/tool loop, workflow command routing.
- `crates/provider`: provider abstraction, registry, and protocol adapters.
- `crates/plugin-api`: shared plugin manifest, capability, permission, and provider descriptor contracts.
- `crates/plugin-host`: static plugin registry boundary for provider/tool/agent/workflow plugins.
- `crates/tools`: local shell, file, search, web, and Git tool implementations.
- `crates/permissions`: permission modes, path scope checks, and allow/ask/deny decisions.
- `crates/session`: JSONL transcript storage and rebuildable SQLite session index.
- `crates/types`: shared domain types for messages, tools, permissions, sessions, runtime snapshots, tasks, and memory.
- `crates/config`: layered config resolution.
- `crates/workflows`: project tasks, project/session memory, resume context, and workflow event storage.
- `crates/lsp`: read-only semantic diagnostics, symbols, references, and
  document synchronization.
- `plugins/providers/deepseek`: DeepSeek provider plugin.

## Non-Negotiable Invariants

- All model tool calls and local command effects must flow through the shared runtime path.
- Permission checks happen before mutation, not after.
- Transcript history remains auditable and append-only for session facts.
- JSONL stays canonical for durable logs; SQLite is a derived, rebuildable index.
- Session state and workflow state are related but separate:
  - `viden-session` records what happened in a session.
  - `viden-workflows` records durable project task and memory state.
- Project memory suggested by an assistant must not become active without explicit confirmation.
- Plan mode must block mutating workflow, file, shell, Git, and memory/task changes.
- Core is the only authority for runtime facts and side effects. Frontends may
  own presentation state but must not create parallel business reducers.
- Frontends must recover missing or out-of-order state through the versioned
  snapshot/replay contract, never by guessing from display text.

## Standard Change Workflow

Before editing:

1. Read the applicable root and nested `AGENTS.md` files.
2. Inspect `git status`, active worktrees, and the actual branch base. Fetch the
   remote when branch freshness affects the task.
3. Identify the owning product track and write scope. Do not start if the same
   files are owned by another active task without a serialization decision.
4. Locate the current contract, design, test, and documentation sources before
   adding a new abstraction or surface.

While editing:

1. Keep the change focused and reversible.
2. For behavior changes, use TDD and verify the initial failure is relevant.
3. Preserve runtime, permission, persistence, and frontend dependency
   boundaries.
4. Update affected English/Chinese docs and concise invariant comments in the
   same change set.
5. Run the smallest useful check after each meaningful increment.

Before handoff:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wikieden/viden](https://github.com/wikieden/viden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
