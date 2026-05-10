---
trigger: always_on
description: This file provides guidance to AI Code Agent when working with code in this repository.
---

# Repository Guidelines for AI

This file provides guidance to AI Code Agent when working with code in this repository.

## Project Structure & Module Organization

This repository is a Go backend plus a Vite/TanStack Router frontend workspace.

- Go entrypoints live at the repository root:
  - `main.go` is the thin CLI entrypoint
  - `frontend_dev.go` provides the development-mode frontend handler wrapper
  - `frontend_embed.go` provides the embedded production frontend handler wrapper for `-tags embed`
- Core Go packages live under `pkg/`:
  - `pkg/app/` for CLI setup and startup config
  - `pkg/server/` for server startup and HTTP route composition
  - `pkg/configservice/` for ConnectRPC handlers
  - `pkg/config/` for config loading and validation
  - `pkg/frontend/` for shared frontend handler implementations
- Protobuf and RPC sources live under:
  - `proto/` for `.proto` contracts managed by Buf
  - `pkg/rpc/` for generated Go protobuf and Connect stubs
- Frontend source lives in `frontend/`:
  - Vite entry: `frontend/src/main.tsx`
  - Router setup: `frontend/src/router.tsx`
  - Route modules: `frontend/src/routes`
  - Shared UI composites: `frontend/src/components`
  - Connect/Web generated client types: `frontend/src/rpc`
  - Shared Connect transport/helpers: `frontend/src/lib/connect`
  - Design primitives/themes: `frontend/src/ui`
  - Global styling: `frontend/src/styles.css`
  - Static assets: `frontend/public`
- Frontend production assets are emitted to `frontend/dist/`
- The production Go binary is emitted to `dist/git-plus`

Colocate new frontend feature assets with the component or route that consumes them to keep dependencies obvious.

- UI: Mantine v8. Use the context7 MCP tool with the library id `/mantine/mantine` to load docs.
- Routing: TanStack Router. Use context7 with `/websites/tanstack_router` for Router docs.
- Generated files like `routeTree.gen.ts` are auto-created by `@tanstack/router-plugin`; do not edit.
- Generated files under `pkg/rpc/` and `frontend/src/rpc/` are auto-created by Buf plugins; do not edit them by hand. Update `proto/` files and re-run code generation instead.

## Build & Development Commands

Use pnpm for workspace tasks and Go tooling for backend compilation/tests.

- `pnpm dev` starts both the frontend dev server and the Go server. The Go server proxies every non-`/api` request to the frontend dev server.
- `pnpm buf:generate` regenerates Go and TypeScript RPC code from `proto/` into `pkg/rpc/` and `frontend/src/rpc/`.
- `pnpm buf:lint` validates `.proto` files with Buf lint rules.
- `pnpm build` first builds the frontend into `frontend/dist/`, then builds `dist/git-plus` with `go build -tags embed`, embedding the frontend assets into the binary.
- `pnpm db:generate:drizzle` regenerates Drizzle SQL migrations from `db/src/schema.ts`.
- `pnpm db:generate:schema-sql` regenerates `db/schema.sql` from `db/src/schema.ts`.
- `pnpm db:generate:sqlc` regenerates Go `sqlc` query code from `db/schema.sql` and `db/queries/*.sql`.
- `pnpm db:generate` runs all database codegen steps in sequence.
- `pnpm test` runs `go test ./...` and then frontend Vitest.
- `pnpm check:types` runs `frontend` type-checking via `tsc --noEmit`.
- `pnpm lint` applies the TanStack + React ESLint rules.
- `pnpm format` runs Prettier, then ESLint autofix, then ESLint verification.

If you need frontend-only commands during local debugging, use the package-level scripts in `frontend/package.json` directly.

Database schema workflow:

- `db/src/schema.ts` is the source-of-truth for the entire SQLite schema.
- Do not handwrite or manually edit `db/migrations/*/migration.sql`.
- Do not handwrite or manually edit Drizzle snapshot files such as `db/migrations/*/snapshot.json`.
- Do not handwrite or manually edit `db/schema.sql`.
- When the database schema changes, update `db/src/schema.ts` first, then regenerate artifacts in order: `pnpm db:generate:drizzle`, `pnpm db:generate:schema-sql`, `pnpm db:generate:sqlc`.
- Always use `pnpm db:generate:schema-sql` to update `db/schema.sql`; do not edit it manually.
- Do not run `db/schema.sql` generation and `sqlc` generation in parallel. `sqlc` must read the freshly generated `db/schema.sql`.
- `db/schema.sql` is a generated downstream schema artifact for SQL tooling; do not treat it as the primary schema definition.
- Raw SQL execution is allowed only in the migration runner. All non-migration database reads and writes in application code must go through `sqlc`-generated queries defined in `db/queries/*.sql`.
- After changing database queries or any schema used by application code, regenerate database artifacts with `pnpm db:generate` unless you intentionally need only one sub-step.

Dependency upgrade policy:

- Keep `@types/node` and `eslint` at their current versions.
- Do not upgrade them unless explicitly requested.

Frontend testing uses Vitest:

- Vitest is configured via `vitest.config.ts` using `test.projects` (separate `unit` + `browser` projects).
- Root `pnpm test` runs Go tests first, then frontend Vitest.
- Frontend-only `pnpm --filter ./frontend test` runs all Vitest projects.
- Frontend-only `pnpm --filter ./frontend test:unit` runs the `unit` project.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ImSingee/git-plus](https://github.com/ImSingee/git-plus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
