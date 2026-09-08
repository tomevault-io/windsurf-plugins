---
trigger: always_on
description: This document helps AI agents and automated systems work in Local Brain safely and
---

# Agent Notes

## Purpose

This document helps AI agents and automated systems work in Local Brain safely and
effectively. It summarizes product principles, development workflow, repo boundaries,
verification, and the Reflect Open patterns agents should reuse.

## Required Context

Before starting work, read `docs/README.md`. For implementation work, also read:

- `docs/plans/architecture-conventions.md`
- `docs/plans/libraries.md`
- the relevant product, schema, or numbered plan docs for the surface you are touching

For comparable desktop, CLI, database, search, AI, or UI behavior, inspect
`/Users/alex/repos/reflect-open` and reuse its proven patterns unless Local Brain has
a product-specific reason to diverge. Do not port Reflect Open's
markdown-as-source-of-truth assumptions.

## Product Principles

Local Brain is an agent-operated local-first personal CRM and memory app. The repo is
a Tauri monorepo on Reflect Open's desktop technology base: a `brain` CLI and skills
for agent writes/reads, a desktop UI for browsing and correction, and SQLite as the
durable source of truth.

- **Agent-first operation.** Most writes and reads should come from AI agents through
  the CLI/skill contract, for example daily automations, todo planning, briefings,
  and memory updates.
- **SQLite is durable truth.** Durable data lives in SQLite plus app-managed assets.
  Derived indexes can be rebuilt; markdown export is a portability feature, not the
  canonical store.
- **No hosted Local Brain APIs for MVP.** AI provider calls are BYOK and direct to
  user-approved providers. Do not add a hosted Local Brain model proxy.
- **Secrets stay in the OS keychain.** Model keys, credentials, and integration
  secrets never belong in SQLite, markdown, Git, logs, or local config files.
- **Chat writes require approval.** Chat uses the Vercel AI SDK from the desktop
  webview. Provider keys are fetched into webview memory only for a user-approved Chat
  request, and Chat write tools must require explicit user approval before mutating
  SQLite.
- **Provider-neutral CLI.** The CLI exposes typed Local Brain operations and generic
  source/external identity fields. It must not know about `gws`, Gmail, Granola,
  Google Contacts, Apple Contacts, or other upstream connector APIs.
- **Keyboard-native sparse UI.** The UI is for browsing, correction, inspection, Chat,
  and demonstration. Keep it keyboard-friendly and avoid surfaces that compete with
  the agent-operated workflow.
- **Evidence and provenance first.** Provenance lives directly on documents,
  interactions, memories, tasks, and evidence links. Factual answers should cite
  evidence references.
- **Use product nouns precisely.** A brain is the top-level local workspace. The Graph
  is the Network visualization inside a brain. Do not use these words
  interchangeably.

Current user surfaces are Today, Tasks, Network, Projects, Graph, Chat, and Settings.
Network contains People and Organizations. Documents and Interactions are first-class
records, but they are browsed inside person, organization, project, and task detail
pages, and through search or Chat.

## Development Workflow

Development happens on `master` unless the user or repository state indicates another
integration branch. When a requested change is complete and verified, proactively
create a pull request unless the user has asked you not to. Branch from the current
integration branch, use the `codex/` branch prefix by default, and target the
integration branch with the PR.

1. Check `git status --short` before editing. You may be in a dirty worktree; never
   overwrite or revert unrelated user changes.
2. Create a short plan and get signoff before proceeding.
3. Make your changes. Keep product docs, schema docs, generated DB types, and numbered
   plans aligned when you touch durable product or schema behavior.
4. Run focused checks for what changed:
   - TypeScript: `pnpm typecheck`, `pnpm lint` or `pnpm lint:fix`
   - Vitest: `pnpm test --run path/to/test` or the relevant package test
   - Database/schema: `pnpm --filter @local-brain/db db:codegen` when schema types
     need regeneration, and `pnpm --filter @local-brain/db test` for drift checks
   - Rust: `cargo test -p brain-cli`, `cargo test -p brain-schema`, or
     `cargo test -p local-brain-desktop` for relevant crate targets
5. For native, CLI, migration, or database changes, also run the relevant `cargo fmt`,
   `cargo clippy`, and `cargo test` targets.
6. Before declaring work done, run `pnpm check` (typecheck + lint + test). For native,
   CLI, migration, or database changes, also run the relevant cargo checks.
7. If the change is complete and verified, open a PR unless the user asked you not to
   or the work was only exploratory/proposal-only.

Before any `cargo` build/check/test that compiles the desktop crate, stage the CLI
sidecar once per checkout:

```bash
pnpm --filter @local-brain/desktop sidecar
```

`pnpm tauri dev` and `pnpm tauri build` stage it automatically.

Do not perform smoke tests by default. Do not start the desktop app, run
`pnpm tauri dev`, run `pnpm tauri build`, or do manual click-through verification
unless the user explicitly asks.

Common commands (repo root):

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [team-reflect/local-brain](https://github.com/team-reflect/local-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
