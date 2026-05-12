---
trigger: always_on
description: This guide helps AI agents (Claude, Copilot, etc.) work effectively on the KickJS codebase.
---

# AGENTS.md — AI Agent Guide for KickJS

This guide helps AI agents (Claude, Copilot, etc.) work effectively on the KickJS codebase.

## Before You Start

1. Read `CLAUDE.md` for project conventions and commands
2. Run `pnpm build` to verify the project compiles
3. Run `pnpm test` to verify tests pass

## Where to Find Things

### Source Code

| What                 | Where                                                             |
| -------------------- | ----------------------------------------------------------------- |
| DI container         | `packages/core/src/container.ts`                                  |
| All decorators       | `packages/core/src/decorators.ts`                                 |
| Module system        | `packages/core/src/app-module.ts`                                 |
| Adapter interface    | `packages/core/src/adapter.ts`                                    |
| Error classes        | `packages/core/src/errors.ts`                                     |
| Logger               | `packages/core/src/logger.ts`                                     |
| Express app wrapper  | `packages/http/src/application.ts`                                |
| Bootstrap function   | `packages/http/src/bootstrap.ts`                                  |
| RequestContext       | `packages/http/src/context.ts`                                    |
| Router builder       | `packages/http/src/router-builder.ts`                             |
| Middleware           | `packages/http/src/middleware/*.ts`                               |
| Query parsing        | `packages/http/src/query/`                                        |
| Config/env           | `packages/config/src/`                                            |
| CLI commands         | `packages/cli/src/commands/`                                      |
| Code generators      | `packages/cli/src/generators/`                                    |
| Generator patterns   | `packages/cli/src/generators/patterns/{rest,ddd,cqrs,minimal}.ts` |
| Template functions   | `packages/cli/src/generators/templates/`                          |
| Drizzle templates    | `packages/cli/src/generators/templates/drizzle/`                  |
| Prisma templates     | `packages/cli/src/generators/templates/prisma/`                   |
| TemplateContext type | `packages/cli/src/generators/templates/types.ts`                  |
| ModuleConfig type    | `packages/cli/src/config.ts`                                      |
| PrismaModelDelegate  | `packages/prisma/src/types.ts`                                    |
| Swagger decorators   | `packages/swagger/src/decorators.ts`                              |
| OpenAPI builder      | `packages/swagger/src/openapi-builder.ts`                         |
| Prisma adapter       | `packages/prisma/src/prisma.adapter.ts`                           |
| Prisma query adapter | `packages/prisma/src/query-adapter.ts`                            |
| WebSocket adapter    | `packages/ws/src/ws-adapter.ts`                                   |
| WebSocket decorators | `packages/ws/src/decorators.ts`                                   |
| WebSocket context    | `packages/ws/src/ws-context.ts`                                   |
| Room manager         | `packages/ws/src/room-manager.ts`                                 |

### Configuration

| What                       | Where                                         |
| -------------------------- | --------------------------------------------- |
| TypeScript base config     | `tsconfig.base.json`                          |
| Wireit build orchestration | Per-package `wireit` config in `package.json` |
| Prettier config            | `.prettierrc`                                 |
| Vitest config              | `vitest.config.ts`                            |
| Pre-commit hook            | `.husky/pre-commit`                           |
| VitePress config           | `docs/.vitepress/config.mts`                  |
| CI pipeline                | `.github/workflows/ci.yml`                    |
| Release pipeline           | `.github/workflows/release.yml`               |
| Docs deploy                | `.github/workflows/deploy-docs.yml`           |

### Reference Implementations

When adding new features, use these as templates:

| Task              | Reference File                                                         |
| ----------------- | ---------------------------------------------------------------------- |
| New middleware    | `packages/http/src/middleware/csrf.ts`                                 |
| New adapter       | `packages/swagger/src/swagger.adapter.ts`                              |
| New package       | `packages/prisma/` (full package structure)                            |
| New example app   | `examples/minimal-api/` (simple) or `examples/task-prisma-api/` (full) |
| New test file     | `tests/container.test.ts`                                              |
| Package exports   | `packages/http/package.json` (exports map)                             |
| Vite build config | `packages/http/vite.config.ts` (multi-entry)                           |

## Checklist: Adding a Feature

### New Middleware

- [ ] Create `packages/http/src/middleware/<name>.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [forinda/kick-js](https://github.com/forinda/kick-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
