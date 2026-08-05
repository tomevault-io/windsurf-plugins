---
trigger: always_on
description: > If you are an agent (or human) about to do any work in this repo: **read this file fully, then read the required-reading files listed below before you make any changes.** Skipping this step is the most common way agents produce work that doesn't fit the project.
---

# Lich v1 — Agent Context

> If you are an agent (or human) about to do any work in this repo: **read this file fully, then read the required-reading files listed below before you make any changes.** Skipping this step is the most common way agents produce work that doesn't fit the project.

## What this project is

**Lich** is a worktree-scoped dev stack orchestrator. One YAML file describes your stack (containers, host processes, env, lifecycle, profiles, custom commands); the lich CLI runs it with per-worktree isolation so multiple stacks can coexist on one machine without colliding. Primary use case: parallel agent-driven dev workflows.

It's a single binary that wraps `docker compose` + host process supervision + an HTTP dashboard. It is NOT a framework, NOT a runtime, NOT a plugin ecosystem.

## Current state (v1 shipped 2026-05-25)

- **v0 (`levelzero`)** was a multi-package plugin-based implementation. It's **deleted** from `packages/` (post-LEV-445/446 cleanup). Only `docs/superpowers/specs/archive-v0/` and `docs/superpowers/plans/archive-v0/` remain as historical record. Do not follow their guidance.
- **v1 (`lich`)** is the single live codebase at `packages/lich/`. Plans 0-6 shipped. Post-v1 follow-ups (e.g. the `lich:instrument` agent skill, the dogfood-stack feature expansion) are tracked as separate Linear projects; the core orchestrator is complete and dogfooded.

## REQUIRED READING — read these files in order before starting any task

1. **`docs/superpowers/specs/2026-05-23-lich-v1-testing-standards.md`** — how we test lich v1. Defines the two-tier requirement (unit + e2e), what every command's e2e tests must verify, anti-patterns to avoid. Non-negotiable; read this first because it shapes every implementation step.
2. **`docs/superpowers/specs/2026-05-23-lich-v1-design.md`** — the product spec. Source of truth for what lich does. Read the sections relevant to your task; skim the rest.
3. **The plan that owns your task.** Find it yourself — do not assume. Process:
   - Look at `ls docs/superpowers/plans/*.md` to see every active plan (ignore `archive-v0/`).
   - Each plan filename is `YYYY-MM-DD-<topic>.md`. Match your task to one of the plans by scope.
   - When uncertain (multiple plans seem to overlap, or your task isn't clearly named anywhere), STOP and ask. Do not guess; doing the wrong plan's work is worse than waiting for clarification.
   - Once identified, read that plan **fully** — not just the task you've been given. Plans contain shared context (architecture, file structure, conventions) that earlier sections establish for later tasks. Tasks read out of context produce out-of-context code.
4. **`packages/e2e/fixtures/dogfood-stack/lich.yaml`** — the canonical example config. Postgres compose service + api/web/tunnel_demo owned services + profile coverage (dev:fast is the default; dev opt-in for DB; dev:env-override for env precedence demos).

If you find yourself wanting to read anything under any `archive-v0/` directory, stop. Those describe a different system. They will mislead you.

## Project layout (where things live)

```
packages/lich/                # the v1 codebase (single TS package, compiled to single binary)
  src/                        # engine + CLI source
  src/daemon/dashboard/ui/    # the dashboard React SPA (separate vite build)
  tests/unit/                 # fast unit tests
  dist/lich                   # compiled binary (after `bun run build`)
  dist/lich-daemon            # daemon companion binary

packages/e2e/fixtures/dogfood-stack/       # the canonical example — Next + Express + Postgres
  apps/web/                   # Next.js frontend
  apps/api/                   # Express API (Bun.sql against postgres)
  db/                         # migrations + seed
  compose.yaml                # postgres compose passthrough (image/healthcheck/tmpfs)
  lich.yaml                   # the stack config

packages/e2e/                 # end-to-end tests; spawn real binary, run against dogfood-stack
  tests/                      # per-feature .test.ts files
  helpers/                    # shared helpers (tmpdir, lich spawn, wait, dbmode, urls)
  vitest.workspace.ts         # dual-pool config (fast = dev:fast, heavy = dev + sandbox/Tart)
  _pool-manifest.ts           # which tests need the heavy pool (long timeouts, singleFork)
  AUDIT.md                    # per-test pool assignment + hardening notes

docs/superpowers/
  specs/                      # v1 design + testing standards (v0 in archive-v0/)
  plans/                      # implementation plans (v0 in archive-v0/)
```

## Rules for v1 work

1. **Both tiers, every feature.** Unit tests AND e2e tests. The testing standards doc explains why this is non-negotiable and what each tier must cover.
2. **The real binary in e2e tests.** No mocking the CLI. Spawn `packages/lich/dist/lich` (built first) and assert observable behavior.
3. **Bite-sized commits.** Each task in the plan is a coherent unit of work that gets its own commit. Don't accumulate work across tasks; commit at the end of each one.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RPate97/lich](https://github.com/RPate97/lich) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-04 -->
