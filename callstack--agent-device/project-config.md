---
trigger: always_on
description: Minimal operating guide for AI coding agents in this repo.
---

# AGENTS.md

Minimal operating guide for AI coding agents in this repo.

## Agent skills

### Issue tracker

Issues and PRDs live in GitHub Issues for `callstack/agent-device`. See `docs/agents/issue-tracker.md`.

### Triage labels

Follow the issue label workflow in `docs/agents/triage-labels.md`.

### Domain docs

Single-context repo. Read `CONTEXT.md` for domain language and testing/architecture vocabulary, and `docs/adr/` for accepted architecture decisions. See `docs/agents/domain.md`.

## First 60 Seconds
- Classify task type:
  - Info-only (triage/review/questions/docs guidance): no code edits and no test runs unless explicitly requested.
  - Code change: make minimal scoped edits and run only required checks from **Testing Matrix**.
- State assumptions explicitly. If uncertain, ask.
- Read required context, not the whole repo:
  - tooling/build/linting: `package.json` and `tsconfig*.json`
  - architecture, routing, command contracts, platform boundaries, diagnostics, or review: relevant `docs/adr/`
  - durable naming/testing vocabulary: `CONTEXT.md`
- Start with at most 3 files: the owning module, one shared helper, and one downstream caller/adapter if needed. Use `rg` before opening large files.
- Define verifiable success criteria before editing.
- Decide docs/skills impact up front.

## Scope & Changes
- Keep changes scoped to one command family or module group unless the task explicitly crosses boundaries. If scope expands, stop and confirm.
- Preserve daemon session semantics and platform behavior.
- Do not inspect both iOS and Android paths unless the task is explicitly cross-platform.
- Ship the minimum code that solves the problem: no speculative features, no single-use abstractions, and no unrelated cleanup.
- Match existing style. Remove imports/variables your change made unused.
- Test through public interfaces when possible. Do not add unrelated exports just to make tests easier.
- Prefer type-level checks when TypeScript can enforce a contract or invalid shape.
- Keep modules small for agent context safety:
  - target <= 300 LOC per implementation file when practical.
  - if a file grows past 500 LOC, plan/extract focused submodules before adding new behavior.
  - if a file grows past 1,000 LOC, treat it as architecture debt unless it is generated data, a fixture snapshot, or an integration test aggregation.
  - long guidance/data tables should live behind focused modules instead of sharing a file with parser/runtime logic.
  - prefer deep modules over mechanical splits: extract when it improves locality for a concept callers already need, not just to reduce line count.
- Before finalizing a code change, do one tightening pass over touched and directly adjacent areas: drop obsolete code, redundant tests, stale helpers/fixtures, and needless duplication made unnecessary by the change.
- Prefer existing helpers. Add a helper only when it reduces real repetition or clarifies domain behavior.
- When adding new guidance, examples, schemas, or command metadata, decide whether it belongs in the command surface, CLI grammar, CLI help, MCP projection, or daemon runtime before editing.
- Prefer updating existing domain vocabulary in `CONTEXT.md` when naming a new durable module concept. Do not coin parallel names in docs, tests, and code.

## Routing
- Keep `src/daemon.ts` as a thin router.
- Keep command names centralized in `src/command-catalog.ts`; do not re-create command identity sets in handlers or request policy modules.
- Keep daemon routing and request-policy traits centralized in `src/daemon/daemon-command-registry.ts`; request modules should consume its predicates instead of recreating command string sets. See `docs/adr/0003-daemon-command-registry.md`.
- Keep command input/output contracts in the command modules:
  - command surface and shared schemas: `src/commands/command-surface.ts`, `src/commands/command-contract.ts`, `src/commands/command-input.ts`
  - typed client command execution: `src/commands/client-command-contracts.ts`
  - command families: `src/commands/interaction-command-contracts.ts`, `src/commands/batch-command.ts`, with other typed client contracts in `src/commands/client-command-contracts.ts`
  - CLI positional/flag grammar: `src/commands/cli-grammar.ts` and `src/commands/cli-grammar/*`
  - typed input to daemon request projection: `src/commands/command-projection.ts`
  - CLI/client/runtime output projection: `src/commands/cli-output.ts`, `src/commands/client-output.ts`, `src/commands/runtime-output.ts`
- Do not reintroduce CLI-shaped command adapters or schemas as a second source of truth. CLI, Node.js, and MCP should project from command contracts.
- Keep `src/daemon/request-router.ts` as request orchestration: auth, diagnostics scope, request admission, locking, handler chain, and fallback dispatch.
- New daemon handler-family commands must update `src/daemon/daemon-command-registry.ts` with the route and request-policy traits. `src/daemon/__tests__/daemon-command-registry.test.ts` guards route and policy traits; handler catalog tests keep executable handler sanity checks.
- Put request policies in focused request modules:
  - tenant/lease/selector/lock admission: `src/daemon/request-admission.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [callstack/agent-device](https://github.com/callstack/agent-device) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
