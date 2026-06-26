---
trigger: always_on
description: Myco captures project memory in a local vault and serves it back through context injection, MCP tools, and skills. This file is intentionally small: keep durable rules here, and let Myco carry dynamic project intelligence.
---

# Myco — Collective Agent Intelligence

Myco captures project memory in a local vault and serves it back through context injection, MCP tools, and skills. This file is intentionally small: keep durable rules here, and let Myco carry dynamic project intelligence.

## Use Myco First

- `AGENTS.md` is for stable project rules, not changing project history.
- Use Myco context, spores, sessions, and plans for recent work, prior decisions, and dynamic guidance.
- When a rule depends on current initiative state or recent architecture change, prefer Myco over adding more static prose here.

## Dogfooding

- We develop Myco using Myco. The project-local vault lives at `.myco/`.
- Session data from development sessions is real vault data. Avoid destructive vault operations unless you mean it.
- After changing hook or daemon code, run `make build` and then `myco-dev restart`. Hooks pick up new code on the next invocation; the daemon does not.
- In git worktrees, prefer not to restart the daemon. Shared vault capture continuity is more valuable than forcing daemon restarts during isolated testing.
- If a worktree must restart for debugging, run the local CLI entry (`node packages/myco/dist/src/cli.js restart`) from that worktree; avoid global `myco-dev restart` from worktrees.
- Use `make dev-link` only from the main checkout; it rewrites shared `~/.local/bin/myco-*` symlinks.
- In git worktrees, use `make dev-link-worktree`; it builds the worktree binary and writes a worktree-local `.myco/runtime.command` pointing at it, without changing shared symlinks. Hooks, MCP, and CLI then route to the worktree build; capture still attaches to the main project vault via `git-common-dir`. See the `dogfood-worktree` skill for caveats (shared-vault schema hazard, vendor-asset build gotcha).
- `make dev-unlink` removes shared dev symlinks and `.myco/runtime.command`; `make dev-unlink-worktree` removes only the worktree runtime pin.

## Core Invariants

- `AGENTS.md` is the canonical rules file. Agent-specific instruction files should stay thin and point back here.
- Hooks in `src/hooks/` must stay thin and delegate to the daemon. Do not put business logic or long-running processing in hook entry points.
- The daemon is the authority for event processing, session recording, spores, and digest work.
- Recurring daemon work must go through the PowerManager. Do not add ad hoc polling timers.
- Session ID is the durable key. Do not tie persistent state to hook lifecycle events.
- Write paths must be additive and idempotent. Do not overwrite or delete accumulated vault history casually.
- Maintain one canonical source of truth per concern. Derived files, stubs, and mirrors should stay thin and point back to it.
- License is **Apache 2.0** (relicensed from MIT on 2026-04-29). New files must carry the Apache header; do not introduce GPL- or AGPL-licensed dependencies.

## Non-Negotiable Rules

- Think before coding. Surface assumptions and ambiguities instead of guessing.
- Verify before asserting state. Before claiming which project/grove/daemon/database something belongs to — or that two of them share state — run the command that proves it and report only what the output shows. A URL slug, an injected memory, or a helper run in isolation is a hypothesis, not proof; label it as such until checked.
- Never build a claim on an unverified claim. If a step was a guess, verify it before it becomes the premise for the next conclusion.
- Prefer extending existing patterns over one-off patches.
- Plans are guideposts, not scripts: reconcile a plan's code against the current code before writing, and when a step would duplicate an existing surface or fight an existing pattern, follow the plan's intent and the existing pattern — surface the deviation, do not transcribe literally into a violation. Reviewers MUST check pattern-fit and duplication, not just fidelity to the plan.
- Prefer established architectural patterns like Vertical Slice Architecture, CLEAN architecture, CQRS, Dependency Injection, etc. when they are appropriate.
- Keep code DRY. Extract helpers or shared patterns when they remove real duplication.
- Write idiomatic TypeScript. Reach for the language's own constructs — higher-order functions, generics, discriminated unions, type guards, `as const` — instead of hand-rolled imperative equivalents. A cross-cutting concern that recurs across call sites (a guard, gate, claim/ownership check, audit, retry) MUST be expressed ONCE as a reusable abstraction applied at the definition — a higher-order wrapper or a single chokepoint every caller funnels through — never as a check copied into each caller. A check duplicated across entry points is a bypass waiting to be added: the next entry point that forgets it silently defeats the rule.
- One implementation per operation across surfaces. UI, MCP/symbiont tools, and CLI MUST share the same underlying code path for the same operation; a user and a symbiont doing the same thing MUST get the same result. Divergent implementations for one operation are a bug.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [goondocks-co/myco](https://github.com/goondocks-co/myco) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
