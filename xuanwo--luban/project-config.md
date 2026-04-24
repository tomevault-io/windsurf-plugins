---
trigger: always_on
description: This document constrains and guides how AI agents (and human contributors) should work in this repository. The goals are: predictable behavior, regressibility, reviewability, and maintainability. If this document conflicts with existing repository conventions, the repository conventions take precedence, and this document should be updated in the PR/change accordingly.
---

# AGENTS.md

This document constrains and guides how AI agents (and human contributors) should work in this repository. The goals are: predictable behavior, regressibility, reviewability, and maintainability. If this document conflicts with existing repository conventions, the repository conventions take precedence, and this document should be updated in the PR/change accordingly.

## 0. Quick start (read this first)
- Default branch: `main`.
- Read `postmortem/README.md` before implementing new features or large refactors.
- Use `pnpm` for all web-related commands; do not use `npm`.
- Prefer `just` recipes over ad-hoc `cargo` commands:
  - `just -l` (discover workflows)
  - `just fmt && just lint && just test` (full local verification)
  - `just test-fast` (domain-only tests)
  - `just run` / `just run release`
  - `just build` / `just build release`
- Keep changes small and reviewable. For functional changes, add or update tests.
- After finishing a task, run the relevant checks, then commit and push.

## 0.0 Mock-first UI workflow + contract-driven integration (required)

The UI source of truth is `web/` in mock mode, and the web/server boundary is governed by explicit
consumer-driven contracts (CDC) under `docs/contracts/`.

See:

- `docs/migrations/2026-01-22-web-mock-mode-contracts.md`
- `docs/contracts/README.md`
- `docs/contracts/progress.md`

For UI/UX work, the workflow is:

- **Mock first**: iterate interaction and UI behavior in `web/` mock mode without requiring the Rust server.
- **Contract first**: when UI changes require new backend capabilities, update contracts before (or together with) server changes.
- **Provider verified**: the Rust server must be validated against contracts via automated checks (contract tests in CI).

## 0.1 Contract alignment rules (hard requirement)

Any change that affects the web/server interaction surface must be aligned with contracts:

- HTTP: `/api/*`
- WebSocket: `/api/events`, `/api/pty/*`
- Message schemas: `WsClientMessage`, `WsServerMessage`, `ClientAction`, `ServerEvent`

Alignment means, at minimum:

1. Update the relevant contract documents under `docs/contracts/features/`.
2. Update `docs/contracts/progress.md` to reflect the new/changed surface and verification status.

For functional changes (non-refactor) that touch these surfaces, also:

3. Add or update provider-side contract tests so the Rust server is enforced against the contract in CI.

## 0.2 Postmortems (required)

This repository treats postmortems as a first-class engineering artifact.

- Before implementing a new feature, skim the relevant postmortems in `postmortem/`.
  - Prefer to reuse proven patterns and avoid repeating known failure modes.
- When fixing a **Sev-1** or **Sev-2** issue (or any bug that required multiple iterations):
  1. Create a new postmortem in `postmortem/` using the template in `postmortem/README.md`.
  2. Update `postmortem/README.md` to include the new incident in the index.
  3. Ensure the postmortem includes:
     - severity, impact, and detection
     - root cause analysis
     - "introduced by" commit(s) and "fixed by" commit(s)
     - reproduction and verification steps
     - prevention/action items

## 0.3 User-Facing Prompt Templates (required)

Luban ships prompt templates as a user-facing feature for tasks across *any* repository.

- Prompts must be **repository-agnostic**:
  - Do not hardcode Luban-specific commands, tools, or workflows.
  - Always instruct the code agent to discover and follow the target repository's own practices
    (README/CONTRIBUTING/CI).
- For code agents (Claude Code/Codex), prefer actionable guidance:
  - Encourage direct investigation and implementation in the worktree.
  - Avoid requiring "patch/diff" artifacts as the primary output, unless the user explicitly asks.

## 0.4 User-First Responsiveness (hard requirement)

Luban is a direct user-facing application. User interactions must not be blocked by non-essential
work.

- Only load/request data that is strictly required for the current user action.
- Prefer lazy loading, incremental loading, and background refresh for non-critical resources.
- Do not put heavyweight global snapshots, full scans, or broad synchronization on hot UI paths.
- When a new request is added to an interactive flow, document why it is necessary and what would
  break without it.
- Prefer patch-style updates (delta/event-driven) over full refetch when only partial data changes.

## 1. Project context and goals (first-screen context for agents)
- Tech stack: Rust server + web frontend (optionally wrapped by Tauri)
- All engineering commands are managed via `justfile` (prefer `just` over invoking `cargo` directly)
- Primary goals:
  1) Keep the server responsive; avoid blocking the async runtime or request handlers
  2) Make business logic unit-testable, property-testable, and regression-testable

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Xuanwo/luban](https://github.com/Xuanwo/luban) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
