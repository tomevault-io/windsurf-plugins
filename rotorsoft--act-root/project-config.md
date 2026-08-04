---
trigger: always_on
description: Guidance for Claude Code working in this repository. This file is the **index**: brief project meta, plus pointers into `docs/docs/` (Docusaurus, for humans) and `.claude/skills/` (for Claude when building Act apps). When in doubt, follow the link.
---

# CLAUDE.md

Guidance for Claude Code working in this repository. This file is the **index**: brief project meta, plus pointers into `docs/docs/` (Docusaurus, for humans) and `.claude/skills/` (for Claude when building Act apps). When in doubt, follow the link.

## Before you start

This file is auto-loaded into context — that is not the same as having read it. Before drafting a slice plan or making the first edit, deliberately consult:

1. **Working on a branch** — `git branch --show-current` should not return `master` (or `main`). If it does, create a feature branch first (`act-<issue>-<slug>`); never accumulate edits or commits on master.
2. **Development Workflow** — the "Changing a port interface" rule, the "Pre-handoff workflow," the doc audit step. If a slice touches `libs/act/src/types/ports.ts`, updating `libs/act-tck/src/` is part of the same slice, not a follow-up.
3. **Rules for contributing to this repo** — durable workflow rules (100% coverage gate, naming, no manual version bumps, integration helpers in separate packages, no `--no-verify`).
4. **Safety-critical one-liners** — load-bearing per-feature gotchas. Re-skim the ones relevant to the file you're about to change.

Skipping this checklist is how duplicated work (per-adapter tests that should have lived in the TCK), master-branch edits, and unnecessary major bumps slip in. Read the rules first; they answer most "should I…?" questions before they reach the user.

## Overview

Act is an event sourcing + CQRS framework for TypeScript, built around DDD aggregates and reaction-driven workflows. The core philosophy: any system distills into **Actions → {State} ← Reactions**.

## Project Structure

pnpm monorepo with two main sections:

- **`/libs`** — core framework libraries
  - `@rotorsoft/act` — core event sourcing framework
  - `@rotorsoft/act-pg` — PostgreSQL adapter (production)
  - `@rotorsoft/act-sqlite` — SQLite/libSQL adapter (embedded/single-node)
  - `@rotorsoft/act-patch` — immutable deep-merge patch utility
  - `@rotorsoft/act-sse` — **deprecated** Server-Sent Events package; now a thin re-export shim over `@rotorsoft/act-http/sse` (the canonical home), kept only for migration and scheduled for removal
  - `@rotorsoft/act-http` — HTTP integrations (umbrella). `webhook` for reaction-driven POST delivery, `receiver` for inbound webhook ingestion, the canonical `sse` subpath for incremental state broadcast (the surface the deprecated `@rotorsoft/act-sse` re-exports), plus the auto-generated API subpaths (`trpc`, `hono`, `openapi`) that walk a built `IAct` registry and emit one route per action — guide at [docs/docs/guides/auto-generated-api.md](docs/docs/guides/auto-generated-api.md)
  - `@rotorsoft/act-pino` — pino-backed `Logger` adapter
  - `@rotorsoft/act-notify` — hybrid notify-broker decorator: `withBroker(store, broker)` delegates every durable Store method and rides an external broker (Redis implemented, Kafka scaffolded, Loopback for tests) for cross-process wakeups only — lifts the LISTEN/NOTIFY fanout ceiling without touching durability. TCK-proven over PostgresStore
  - `@rotorsoft/act-otel` — Prometheus metrics bridge: `instrument(app)` maintains the canonical metric set from the observability guide off the lifecycle events. Leaf package — core stays metrics-free by design
  - `@rotorsoft/act-crypto` — authenticated envelope encryption (AES-256-GCM + versioned wire format) for adapters that want column-level encryption with operator-controlled keys. Leaf package — adapters depend on it, core does not.
  - `@rotorsoft/act-ops` — operational primitives (idempotency, retry budgets, poison-message classification). **Zero dep on `@rotorsoft/act`** by design — so non-Act receivers (forwarded-bus consumers, Express endpoints, queue workers) can speak the same contract without pulling in the orchestrator
  - `@rotorsoft/act-tck` — Test Compatibility Kit for Store/Cache/Logger ports

- **`/packages`** — example applications
  - `calculator` — simple state machine; rebuild and close demos
  - `wolfdesk` — complex ticketing (from "Learning Domain-Driven Design")
  - `server`, `client` — tRPC + React example

## Common Commands

```bash
pnpm install          # install
pnpm build            # build all packages
pnpm test             # run all tests with coverage
pnpm typecheck        # tsc --noEmit
pnpm lint / lint:fix  # biome
pnpm clean            # remove build artifacts
pnpm scrub            # remove all node_modules + build artifacts

pnpm dev:calculator   # run examples
pnpm dev:wolfdesk
pnpm dev:http         # multi-transport demo (trpc + hono rest + openapi) — server + client concurrently

vitest                                                # watch mode
pnpm -F calculator test                               # one package
npx vitest run packages/calculator/test/invariants.spec.ts   # one file

pnpm -F wolfdesk drizzle:migrate   # wolfdesk migrations (its tests also self-migrate)

pnpm act                                  # interactive contracts explorer (current dir)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rotorsoft/act-root](https://github.com/Rotorsoft/act-root) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
