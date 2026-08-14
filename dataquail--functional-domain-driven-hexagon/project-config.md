---
trigger: always_on
description: Effect v4 monorepo, hexagonal architecture, DDD. Full rationale lives in `docs/adr/`; the working-memory digests live in `.claude/rules/`.
---

# Project conventions

Effect v4 monorepo, hexagonal architecture, DDD. Full rationale lives in `docs/adr/`; the working-memory digests live in `.claude/rules/`.

**Before working in an area, read its rule file** — `.claude/rules/` is not auto-loaded, so pull in the relevant one:

| Working on…                                           | Read                                             | Backing ADRs                            |
| ----------------------------------------------------- | ------------------------------------------------ | --------------------------------------- |
| Adding/moving files in a server feature module        | `.claude/rules/server-module-layout.md`          | 0002, 0003, 0013, 0022–0024             |
| New file kind, test, fake, stereotype (parity/layout) | `.claude/rules/server-file-taxonomy.md`          | 0008                                    |
| Writing or running server/jobs tests                  | `.claude/rules/server-testing.md`                | 0009                                    |
| Handlers, layers, event buses, SQL, auth (server)     | `.claude/rules/server-effect-and-persistence.md` | 0004, 0006, 0007, 0012, 0016–0017, 0020 |
| Frontend (`packages/web`, `packages/components`)      | `.claude/rules/frontend.md`                      | 0014, 0015, 0018                        |
| Any Effect v4 API you are not certain of              | `.claude/rules/effect-v4-source.md`              | —                                       |
| Writing comments (any package)                        | `.claude/rules/comments.md`                      | —                                       |
| Lint config, custom rules, rule probes                | `.oxlintrc.json`, `scripts/lint-rules/`          | 0008, 0025                              |

## Monorepo map

| Package             | What it is                                                                                                                                                                                                  |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `@org/server`       | The Effect BFF backend (`src/modules/`, `src/platform/`, HTTP). Bulk of the rules.                                                                                                                          |
| `@org/web`          | Next.js App Router renderer + `/api/*` proxy; the server stays the BFF (ADR-0018).                                                                                                                          |
| `@org/components`   | Bespoke component library (primitives + patterns) + Storybook (ADR-0015).                                                                                                                                   |
| `@org/contracts`    | Shared HTTP API contracts, schemas, errors — consumed by server and clients.                                                                                                                                |
| `@org/cqrs`         | Standalone typed CQRS library: `Command`/`Query`/`Event`, per-module dispatchers, one event bus whose subscriptions pick their consistency model, the `UnitOfWork` port, sagas, middleware — ADR-0006/0007. |
| `@org/database`     | DB access kernel (slonik, `RowSchemas`, `db.makeQuery`) + migrations.                                                                                                                                       |
| `@org/jobs`         | Background/cron jobs.                                                                                                                                                                                       |
| `@org/cli`          | Command-line client (device-flow auth, organizations, todos).                                                                                                                                               |
| `@org/mcp`          | MCP (stdio) server exposing the CLI surface as tools.                                                                                                                                                       |
| `@org/api-client`   | Shared typed client + credential store for the CLI and MCP.                                                                                                                                                 |
| `@org/acceptance`   | Playwright acceptance tests (specs / drivers / pages / infrastructure).                                                                                                                                     |
| `@org/test-drivers` | Tier-agnostic page-driver contracts + per-tier adapters (Playwright / RTL).                                                                                                                                 |

## Commands

| Command                                                | What it runs                                                                                                 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dataquail/functional-domain-driven-hexagon](https://github.com/dataquail/functional-domain-driven-hexagon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
