---
trigger: always_on
description: This repository is intended to be completed autonomously by a coding agent such as Codex.
---

# AGENTS.md

This repository is intended to be completed autonomously by a coding agent such as Codex.
Treat this file as the persistent execution charter and product/architecture specification.
Update it whenever architecture decisions, constraints, or risks materially change.

## Agent operating contract

1. Read this file fully before changing code.
2. Keep this file current. When an architectural or design decision changes, revise the affected section — do not maintain a dated completion ledger here.
3. Work autonomously in small, verifiable increments until acceptance criteria are met.
4. Do not stop at scaffolding. Leave the repository with a working vertical slice.
5. Follow TDD where practical:
   - write a failing test
   - implement the minimum code
   - make the test pass
   - refactor safely
6. Never disable tests, typechecks, lint rules, or security checks to move faster.
7. The runtime architecture is browser-first. Do not restore Electron IPC or renderer assumptions as the primary path.
8. Prefer structured schemas, explicit persistence models, and explicit failure states.
9. Preserve a minimal, modern, internal-scrollable UI.
10. If a better implementation detail is discovered, update this file before or alongside the code.
11. Before declaring a phase complete, run the relevant verification command.
12. When making changes, validate the local command path used by GitHub Actions before handoff; today that means running `pnpm ci:full` (lint, typecheck, unit tests, security). End-to-end Playwright runs are not part of CI today; run them separately with `pnpm exec playwright test` when a change affects the e2e surface.

## Product vision

Build a local-first Hermes frontend with:

- a browser-based chat interface with streaming markdown
- real Hermes profile and session browsing
- persisted local Spaces attached to sessions, with synchronized markdown/table/card content representations, extensible content-first tabs, and combined session+space workspaces
- jobs / cron visibility
- tool listing and reviewed tool execution history
- explicit disconnected, degraded, error, and empty states
- clean restart-safe persistence for hundreds of chats and thousands of messages

## Core product principles

1. Chat-first: chat is the default landing page.
2. Bridge-first: the browser talks only to a thin local bridge over HTTP and SSE.
3. Real data only: no synthetic `Local Profile`, fake sessions, or placeholder jobs presented as real state.
4. Local-first persistence: the app works from durable local state and survives restart cleanly.
5. Loud correctness: failures should surface clearly so they can be fixed.
6. Secure by default: dangerous actions stay behind explicit approval and capability boundaries.

## Architecture target

Monorepo layout:

- `apps/bridge`: local Node bridge service
- `apps/web`: Vite + React + TypeScript browser app
- `packages/protocol`: shared schemas and API contracts
- `packages/ui`: shared Chakra provider and theme helpers
- `packages/config`: shared lint configuration
- `packages/testing`: cross-package test utilities
- `docs/specs`: canonical product and architecture notes
- `scripts`: developer and security scripts

Tech stack target:

- Vite
- React
- TypeScript
- Chakra UI
- Zod
- SQLite
- Vitest
- Playwright
- Turbo + pnpm workspaces

## Runtime requirements

Frontend:

- Vite + React + TypeScript
- Chakra UI
- atomic design structure under:
  - `atoms`
  - `molecules`
  - `organisms`
  - `templates`
  - `pages`
- browser-first only
- no Electron renderer assumptions
- responsive, internal-scrollable shell
- dark mode / light mode

Bridge:

- local-only thin service
- HTTP + SSE transport
- Hermes CLI integration for:
  - profile discovery
  - session discovery
  - transcript import/export
  - cron/jobs reads
  - tools reads
  - streaming assistant updates
  - persistence access
  - reviewed tool execution
- Coding agent integration (Claude Code + Codex) — see below
- OS-agnostic local data path resolution for macOS/Linux/Windows

Coding agent integration:

- Status detection uses `claude auth status` (JSON) and `codex login status` (text) — non-interactive, no tokens spent.
- Approval-mode flag mapping:
  - `manual`: no flag (Claude) / `-a on-request -s read-only` (Codex)
  - `auto_safe`: `--permission-mode acceptEdits` (Claude) / `--full-auto` (Codex)
  - `auto_all`: `--permission-mode bypassPermissions` (Claude) / `--dangerously-bypass-approvals-and-sandbox` (Codex)
- Codex uses `spawn-per-turn` multi-turn mode (fresh `codex exec` process per turn, resumed via `codex exec resume`). Claude Code uses `stay-alive` mode (stdin kept open, `--input-format stream-json`).
- Integration `enabled` column controls soft-disable (in-app toggle, no CLI logout). Hard delete runs `claude auth logout` / `codex logout`.
- SSE replay uses `?since=<eventId>` cursor to prevent duplicate events on client reconnect.
- Stuck heuristic fires only after 5 minutes of idle with no in-flight tool calls. "No output" false-alarms are suppressed — a `job.heartbeat_warning` inline note is shown instead.

Persistence:

- SQLite is the primary local store

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jozef-barton/the-kitchen](https://github.com/jozef-barton/the-kitchen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
