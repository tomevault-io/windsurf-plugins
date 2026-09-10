---
trigger: always_on
description: Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.
---

# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:

- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:

- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

# Working in this repo

LangX v2 — a language-exchange app. Expo (iOS/Android/web) + Fastify + MongoDB,
in a pnpm workspace. Open source, BSD-3, and **the repo is public**.

Start with [`docs/architecture.md`](docs/architecture.md) for the design,
[`docs/decisions.md`](docs/decisions.md) for why things are shaped the way they
are, and [`docs/v1-reference.md`](docs/v1-reference.md) for the identifiers of
the system this replaces. Both are worth reading before changing anything structural — several
decisions look arbitrary until you know what went wrong the first time.

## Layout

| Path              | What                                                                         |
| ----------------- | ---------------------------------------------------------------------------- |
| `apps/api`        | Fastify + Better Auth + Socket.io, one process                               |
| `apps/mobile`     | Expo app; the web build comes out of the same code                           |
| `packages/shared` | zod schemas, DTO types, `PLAN_LIMITS`, `TOKEN_RULES`, languages, error codes |

## Commands

```bash
pnpm dev            # API on :4000, Expo on :8081
pnpm test           # vitest, everywhere
pnpm -r typecheck
pnpm lint           # eslint; lint:fix to autofix
pnpm format         # prettier
```

CI runs typecheck, lint, format:check and tests. All four must pass.

## Sibling repos

This repo is one of several: langx.io, token.langx.io, the GitBook docs and
v1's still-running API each have their own. [`docs/repo-map.md`](docs/repo-map.md)
says what they are, where each site is hosted, and — the load-bearing part —
which values are copied between them by hand. Three rules follow from it:

- This file is the instruction. The sibling repos are assumed to be checked out
  next to this one; paths such as `website/src/lib/data/plans.ts` in these docs
  mean that sibling.
- Every `langx/*` repo is public. Nothing that must stay private goes into any
  of them — no credentials, and no map of where they live: no local paths,
  hostnames, or file names of secret files, in `docs/` included.
- When a plan limit or a token rule changes in `packages/shared`, change the
  copies on the website and in the GitBook docs too. Nothing checks this, and a
  pricing page that drifts is a false claim, not a stale one.

## Local MongoDB must be a replica set

Better Auth wraps writes in transactions and MongoDB only offers those on a
replica set. A standalone `mongod` fails on the first sign-up with a
transaction error. See `docs/self-host.md`.

## Every worktree needs its own `pnpm install`

The install is slow and a second worktree is right there, so pointing one at
the other's `node_modules` is the obvious shortcut. It fails one of two ways
depending on which `node_modules` gets linked, and only one of them tells you.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [langx/langx](https://github.com/langx/langx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
