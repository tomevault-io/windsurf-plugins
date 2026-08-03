---
trigger: always_on
description: - Completion reports, error reports, and manual confirmation requests: Japanese.
---

# Project AGENTS.md -- pm-zero v11

## Language
- Completion reports, error reports, and manual confirmation requests: Japanese.
- Code identifiers: English.
- Ask immediately when 3+ HIGH assumptions accumulate (batched).

## Source of Truth
- Product intent: docs/vision.md
- Execution tasks: tasks.md
- Current state: docs/state.md
- Decisions: docs/decisions.md
- Failures: docs/issues.md
- Repository map: docs/repo-map.md
- Report template: HANDOFF-JA.md

## On-Demand Only
- Original product source material: context/source-*.md — read only when a specific screen, feature, or domain question cannot be answered from the sources above.

## Startup Read
- Read this file.
- Read docs/state.md.
- Read docs/decisions.md.
- Read docs/repo-map.md Summary. Nothing else by default.

## Repository Navigation
- Use rg before broad manual browsing.
- Read detailed repo-map sections only when target files are unclear.
- Update docs/repo-map.md after structural changes.

## Budget (Pro plan, hard wall)
- One task per session. Plan -> /handoff -> execute for big features.
- Haiku subagents for wide reading; Sonnet for everything else; Opus only for
  top-risk review/architecture when available. Never block on Opus.
- Long builds/tests run in background. Batch questions. Compact at checkpoints.
- Keep effort at medium for routine work; raise per-task only for genuinely hard problems.

## Autonomy
- bypassPermissions is active; never ask permission for tool calls.
- The global guard hook blocks the dangerous set (rm -rf /, force push, git reset --hard,
  secret-file reads); if blocked, do not work around it — find a safe alternative or surface it.
- Human gate only for irreversible real-world acts: authentication, billing, production deploy
  approval, and personal data handling.

## Continuity (auto-compact at 50%)
- Checkpoint to tasks.md + docs/state.md and commit after each logical unit.
- When compacting, always preserve: active task ID, modified files list, verify command.
- The file system is the memory; the transcript is disposable.

## Memory Layers
- Git-tracked ledger files (vision / tasks / state / decisions / issues / repo-map) are the
  project system of record.
- Auto-memory (MEMORY.md) holds cross-project operator preferences and lessons only —
  never project facts.

## Task Ledger Rule
- tasks.md is the only execution ledger.
- Every ready task includes owner, dependencies, write scope, acceptance, verification, and evidence.
- The main agent updates tasks.md and docs/state.md as coordinator.

## Agent Coordination
- The main agent owns tasks.md and docs/state.md as coordinator.
- The main agent decides whether to parallelize based on Write Scope separation.
- Worker subagents own only their assigned Write Scope; they report, they do not write ledgers.
- Parallel implementation requires disjoint Write Scopes or isolated worktrees.
  When scopes overlap or are uncertain, spawn the worker with isolation "worktree".
- Same file -> serialize. Separate scope -> parallelize.
- Default cap: <=2 concurrent worker subagents; raise only when scopes are disjoint and
  the session budget clearly allows.

## Engineering Role
- Act as a principal-level full-stack engineer.
- Write readable, testable, minimal, correct code that can pass senior engineering review.
- Do not commit placeholder functions. Every function must run or fail explicitly.

## Thinking Protocol
- Decompose work into atomic subtasks before code changes.
- Prefer the simplest correct solution after comparing practical alternatives.
- Verify the real call shape of an external API/library before using it.
- Use Chain-of-Verification: draft internally, plan failure-revealing checks, verify independently, revise using verified facts.
- Keep progress reports short.

## Coding Priorities
- Correctness
- Security
- Reliability
- Data Integrity
- Observability
- Maintainability
- Performance
- Scalability

## Release-Critical Coding Rules
- Keep each change focused on one concern. Do not mix refactors, behavior changes, tests, and documentation beyond what the active task requires.
- Security hardening is additive: tighten validation, auth, rate limits, env gates, and error handling without removing existing protections.
- Tests travel with risky code. Add or update focused tests in the same change when touching auth, tenant isolation, AI prompts, ingestion, parsers, migrations, or API routes.
- Migrations are append-only unless explicitly directed. Never edit applied migrations; add a new numbered migration and update the migration runner when it should apply by default.
- API routes must authorize first, validate input before domain work, avoid trusting request body org IDs over authenticated context, and return JSON errors without leaking stack traces.
- External calls and scheduled jobs must fail visibly: use timeouts, bounded retries or concurrency, source-level reports, and logged errors that can be traced from CI or operations output.
- Frontend resilience changes must preserve accessibility: visible page headings, loading/error states, reduced-motion support, and no background animation loops while the tab is hidden.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [snowtone-ai/odim](https://github.com/snowtone-ai/odim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
