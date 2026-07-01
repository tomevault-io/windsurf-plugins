---
trigger: always_on
description: Purpose: help AI agents work on `codex-tg` without increasing complexity or weakening the operator-facing Telegram control loop.
---

# AGENTS

Purpose: help AI agents work on `codex-tg` without increasing complexity or weakening the operator-facing Telegram control loop.

Good code is code that is easy to understand, change, test, and safely extend. Good agent work is evidence-backed, small in scope, and validated through the same surfaces the operator uses.

## Repository Purpose

`codex-tg` is a Go daemon evolving into a local Codex Control Plane. Telegram is the first production adapter for observing, steering, approving, and routing local Codex App Server work.

The repo is public-facing. Keep every change safe for open-source publication: no private paths, tokens, Telegram ids, local sessions, databases, logs, screenshots with private data, or environment-specific credentials.

## Working Mode

- First understand the current design. Read nearby code, tests, ADRs, and `docs/testing/regression-map.md` when it exists before editing.
- Do not treat a vague request as enough context. Ask focused questions when requirements, constraints, or tradeoffs are unclear and cannot be discovered safely.
- Prefer small steps: inspect -> plan -> test -> implement -> refactor -> validate.
- Do not generate large rewrites, broad refactors, or speculative frameworks unless the task explicitly requires them.
- Build what is needed now. Avoid adding libraries, build tools, abstractions, or cross-platform machinery without an immediate reason.
- Match the current user's language for working plans, handoffs, and status updates when clear. Public README, wiki, demo, release notes, and GitHub-facing docs may stay English-first.

## Agent Workflow Kernel

- Do not start coding vague work. For unclear features, run alignment first: ask focused questions, lock goal, non-goals, UX, data, tests, and acceptance.
- Feature work needs a destination before implementation: a short issue, feature brief, or equivalent written target.
- Slice vertically. Prefer one small end-to-end behavior over horizontal phases such as storage first, daemon later, UI last.
- Use TDD for non-trivial behavior changes and bug fixes: failing or updated test, minimal implementation, refactor, targeted checks.
- Use fresh-context review for meaningful changes. The reviewer should inspect issue/brief, diff, tests, ADR/rules, and validation output, not the implementation chat.
- Human/live QA remains required for Telegram-facing behavior when a live contour is available.
- After compaction, context reset, or handoff, reread this file first, then pull only the docs called out by the Context Pull Map.

## Context Pull Map

- Telegram UI, routing, observer panels, lifecycle, Plan Mode, diagnostics, or rendering: read `docs/testing/regression-map.md` and the ADR named there when present.
- Public command behavior or Telegram product contract: read `docs/research/contract-matrix.md` when present.
- New feature planning, bugfix shaping, or review handoff: read `docs/process/agent-workflow.md` and the related template when present.
- Release, README, demo, or public positioning work: read README plus relevant `docs/wiki/*`, `docs/demo/*`, `CONTRIBUTING.md`, and `SECURITY.md` when present.
- If a referenced document is absent in the current branch, do not invent it or add broken links. Use this `AGENTS.md`, nearby code/tests, and existing ADRs as the source of truth.

## Core Decisions

- Current runtime state authority is Codex App Server. New control-plane work must preserve App Server authority for interactive threads, turns, approvals, live events, history, and snapshots.
- Spawned `codex app-server` over stdio remains supported, but ADR-019 allows future `unix://` and `app-server proxy` transport work.
- Durable identity is `threadId`; Telegram chat/topic is only an input and rendering surface.
- Live observer events come from the daemon session; foreign GUI/CLI activity must also be covered by polling `thread/read`.
- App Server session lifecycle transitions must be serialized and generation-aware; stale old-session close/error events must not invalidate newer sessions or create repair loops.
- Startup must remain non-blocking; never put full thread sync into synchronous startup.
- SQLite is the local source of truth for bindings, routes, callbacks, panels, observer target, delivery metadata, and daemon state.
- Do not add a second live runtime backbone for App Server state. SDK/MCP may be used only where ADR-019 allows orchestration adapters, and must not replace App Server truth for live UI/control state without a new ADR.

## Design And Code Principles

- Simple code beats clever code. Keep changes local and avoid making unrelated subsystems harder to understand.
- Prefer cohesive vertical slices and deep modules: expose small, clear interfaces that hide useful internal complexity.
- Keep modules focused. Each module should have one main reason to change.
- Make interfaces explicit with typed inputs/outputs, clear ownership, and precise error cases.
- Avoid duplication. If the same routing, rendering, lifecycle, or parsing rule appears twice, extract one source of truth.
- Inject or pass dependencies for IO, time, config, external services, and randomness. Do not hardcode them deep in business logic.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mideco-tech/codex-tg](https://github.com/mideco-tech/codex-tg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
