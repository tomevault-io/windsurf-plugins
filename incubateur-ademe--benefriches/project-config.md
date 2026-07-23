---
trigger: always_on
description: > **Purpose**: Context for AI assistants and developers. For app-specific patterns, see individual CLAUDE.md files.
---

# Benefriches - Monorepo Guide

> **Purpose**: Context for AI assistants and developers. For app-specific patterns, see individual CLAUDE.md files.

## Quick Lookup

| I need to...                 | Go to...                                               |
| ---------------------------- | ------------------------------------------------------ |
| Add API feature              | [apps/api/CLAUDE.md](apps/api/CLAUDE.md)               |
| Add Web feature              | [apps/web/CLAUDE.md](apps/web/CLAUDE.md)               |
| Add E2E test                 | [apps/e2e-tests/CLAUDE.md](apps/e2e-tests/CLAUDE.md)   |
| Add shared type/DTO          | [packages/shared/CLAUDE.md](packages/shared/CLAUDE.md) |
| See complete feature example | [docs/feature-example.md](docs/feature-example.md)     |
| Run quality checks           | [Standard Commands](#standard-commands)                |

## Critical DON'Ts

1. **Don't use `npm`** - Always use `pnpm`
2. **Don't skip reinstalling `shared`** - Always reinstall in dependent apps after changes (see [Shared Package Workflow](#shared-package-workflow))
3. **Don't add framework deps to `shared`** - Keep it pure TypeScript only
4. **Don't skip cross-app testing** - Changes to `shared` can break both `api` and `web`
5. **Don't commit without quality checks** - Husky pre-commit hooks enforce lint + format (tests must be run manually)
6. **Don't use relative imports across apps** - Use workspace protocol: `import from "shared"`
7. **Don't modify database without migration** - Create a Knex migration for all schema changes
8. **Don't add an env var without updating env files** - Add to the relevant app's `.env.example` (empty/off), `.env.e2e` at the repo root (with the value needed for e2e tests), AND `docker-compose.e2e.yml` under the relevant service's `environment:` block
9. **Don't duplicate Zod schemas** - Before writing a new Zod schema, check `packages/shared` for reusable ones (e.g., `surfaceAreaSchema`, `soilsDistributionSchema`)
10. **Don't bypass the Talisman pre-commit hook** — Talisman scans commits for secrets (API keys, tokens, private keys). Never use `--no-verify` to skip it.

## Monorepo Structure

```
benefriches/
├── apps/api/         # NestJS REST API (PostgreSQL + Clean Architecture)
├── apps/web/         # React SPA (Vite + Redux + Clean Architecture)
├── apps/e2e-tests/   # Playwright end-to-end tests
├── packages/shared/  # Shared TypeScript types and utilities (framework-agnostic)
└── README.md         # Full setup instructions (French)
```

**Getting Started**: See [README.md](README.md) for installation and setup.

**Required**: Node 24 (`"engines": { "node": "24" }` in package.json).

## Essential pnpm Commands

```bash
# Run in ALL workspaces
pnpm -r build && pnpm -r typecheck && pnpm -r test

# Run in specific app
pnpm --filter api <command>
pnpm --filter web <command>
pnpm --filter shared <command>

# Common commands
pnpm --filter api dev     # Start API dev server
pnpm --filter web dev           # Start web dev server
pnpm --filter shared build      # Build shared package (required after changes!)
```

## Shared Package Workflow

**CRITICAL**: Changes to `shared` types can break both `api` and `web`. Always reinstall and verify both apps after modifying shared.

```bash
# After modifying shared:
pnpm --filter shared build                              # Rebuild shared package
pnpm --filter api install && pnpm --filter web install  # Reinstall in dependent apps
pnpm -r typecheck && pnpm -r test

# When actively developing shared (rebuilds on every change):
pnpm --filter shared dev
```

Note: We don't use monorepo dependency solutions (nx, turborepo). You must manually run `pnpm --filter shared build` after modifying the shared package, or use `pnpm --filter shared dev` for watch mode.

## Database Migrations

- Always use the `/create-database-migration` skill when creating database migrations. Never create migration files manually.
- Use the project's `pnpm` commands for migration generation, not raw SQL files.

## Testing Strategy

> **How we design tests** (what to test, structure, where it lives): [.claude/rules/testing.md](.claude/rules/testing.md) — AAA planning, one behavior per test, self-contained setup, testing the real path, colocation, e2e scope. Read it when **writing or brainstorming** tests, not only when editing `*.spec.ts` files.

### Test Types

- **Unit tests** (`.spec.ts`): No real I/O. Lives next to the code under test, in `core/` or `adapters/`. HTTP adapter tests with mocked transport (e.g., `mock.fn()` from `node:test` on `HttpService`) belong here — they verify request/response mapping without booting testcontainers.
- **Integration tests** (`.integration-spec.ts` in `adapters/`): Real database or real network calls. Boots testcontainers.
- **E2E tests** (`.spec.ts` in `e2e-tests/tests/`): Full user flows with Playwright against running stack

### Required Tests by Change Type

| Change                        | Required Tests                                                                    |
| ----------------------------- | --------------------------------------------------------------------------------- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [incubateur-ademe/benefriches](https://github.com/incubateur-ademe/benefriches) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
