---
trigger: always_on
description: Sync Stripe data to PostgreSQL (and other destinations) via a message-based protocol.
---

# Sync Engine

Sync Stripe data to PostgreSQL (and other destinations) via a message-based protocol.
Sources read from APIs, destinations write to databases, and the engine wires them together
through typed async iterable streams. Connectors communicate via NDJSON when running as subprocesses.

## Quick Reference

```sh
pnpm install
pnpm build          # required before running CLI or e2e tests
pnpm test           # unit tests (no deps needed)
pnpm test:integration  # needs local Postgres
pnpm test:e2e       # needs Docker + Stripe API keys in .env
```

Before committing (CI enforces all three):

```sh
pnpm format          # prettier
pnpm lint
pnpm build
```

Minimum Node.js version: **24**. Dev with auto-rebuild: `pnpm --filter sync-engine dev`

If you add a migration, register it in `packages/state-postgres/src/migrations/index.ts`.

## Architecture at a Glance

Sources and destinations are isolated connectors that only depend on `protocol`
and approved shared utilities (`logger`, `openapi`, `util-postgres`).
The engine loads connectors (in-process or subprocess), pipes source output through
destination input, and manages state checkpoints. See [docs/architecture/packages.md](docs/architecture/packages.md)
for the full dependency graph.

## Package Map

| Package                              | Purpose                                                   | Depends on                               |
| ------------------------------------ | --------------------------------------------------------- | ---------------------------------------- |
| `packages/protocol`                  | Message types, Source/Destination interfaces, Zod schemas | `zod`, `citty`, `ix`                     |
| `packages/openapi`                   | Stripe OpenAPI spec fetching and parsing                  | `zod`                                    |
| `packages/logger`                    | Structured logging (pino) + progress UI (ink)             | `pino`, `ink`; peer: `protocol`          |
| `packages/source-stripe`             | Stripe API source connector                               | `protocol`, `openapi`, `logger`          |
| `packages/destination-postgres`      | Postgres destination connector                            | `protocol`, `util-postgres`, `logger`    |
| `packages/destination-google-sheets` | Google Sheets destination connector                       | `protocol`, `logger`                     |
| `packages/state-postgres`            | Postgres state store + migrations                         | `util-postgres`, `logger`                |
| `packages/util-postgres`             | Shared Postgres utilities (upsert, rate limiter)          | `logger`, `pg`                           |
| `packages/hono-zod-openapi`          | Hono + zod-openapi integration for spec generation        | `hono`, `zod`, `zod-openapi`             |
| `packages/test-utils`                | Shared test helpers (servers, seeds, fixtures)             | `destination-postgres`, `openapi`, `pg`  |
| `packages/ts-cli`                    | Generic TypeScript module CLI runner                      | `citty`                                  |
| `apps/engine`                        | Sync engine library + stateless CLI + HTTP API            | `protocol`, connectors, `state-postgres` |
| `apps/service`                       | Pipeline management + Temporal workflows                  | `engine`, Temporal SDK                   |
| `apps/dashboard`                     | React web UI for pipeline management                      | `openapi-fetch`, `radix-ui`              |
| `apps/visualizer`                    | Next.js data visualization tool                           | `next`, `source-stripe`, `pglite`        |
| `apps/supabase`                      | Supabase edge functions (Deno runtime)                    | `protocol`, `engine`, connectors         |
| `e2e/`                               | Cross-package conformance and layer tests                 | all packages                             |

## Key Rules

0. **This file is an index, not a rulebook** — before adding anything here, check if it belongs in [docs/architecture/principles.md](docs/architecture/principles.md), [docs/architecture/decisions.md](docs/architecture/decisions.md), or another doc first. Only add to AGENTS.md if no better home exists.
1. **Connector isolation** — sources never import destinations, both depend only on `protocol` + approved shared utilities. Enforced by `e2e/layers.test.ts`.
2. **State is a message** — connectors never access state storage directly. State in = `cursor_in`; state out = `SourceStateMessage`.
3. **Snake_case on the wire** — all Zod schemas and JSON wire format use snake_case.
4. **api_version is required** — always mandatory in Stripe source config. Never optional.
5. **Tests fail loud** — no silent skips when dependencies are unavailable.

See [docs/architecture/principles.md](docs/architecture/principles.md) for the complete list.

## Where to Find Things

- **Architecture & layers:** [docs/architecture/](docs/architecture/)
- **Design decisions:** [docs/architecture/decisions.md](docs/architecture/decisions.md)
- **Engine internals:** [docs/engine/](docs/engine/)
- **Service internals:** [docs/service/](docs/service/)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stripe/sync-engine](https://github.com/stripe/sync-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
