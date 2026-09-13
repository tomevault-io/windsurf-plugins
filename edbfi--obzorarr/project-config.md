---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this
repository.

Obzorarr is a "Wrapped for Plex" app: SvelteKit 2 / Svelte 5 on Bun, SQLite via Drizzle,
shipped as a single Bun server (`svelte-adapter-bun`).

## Commands

Bun is the only package manager. `bun install` also runs `svelte-kit sync` (`prepare`) and
`prek install` (`postinstall`) — the latter installs the git hooks, including
`no-commit-to-branch --branch main`, so after installing you must work on a branch.

| Command | Purpose |
| --- | --- |
| `bun run dev` | Dev server. **Does not load `.env`** — configure Plex via onboarding/admin UI |
| `bun run dev:env` | Dev server with `--env-file=.env`; only needed to exercise env-locked settings UI |
| `bun run check` | `svelte-kit sync && svelte-check` — the type gate |
| `bun run test` | Full suite (`bun test --env-file=.env.test`) |
| `bun run check:biome` | Lint + format check; `bun run lint:fix` / `bun run format` to autofix |
| `bun run build` | Production build into `build/` |
| `bun run start` | Production server from `build/index.js` with `NODE_ENV=production` |
| `bun run db:generate` | Generate a migration in `drizzle/` from `schema.ts` |
| `bun run db:migrate` | Apply migrations standalone (the app also migrates on boot) |

Single file / single case:

```bash
bun test --env-file=.env.test tests/unit/sharing/service.test.ts
bun test --env-file=.env.test -t "rejects stale settingsVersion"
```

`bunfig.toml` forces coverage on with `line = 0.8, function = 0.8`, so a partial run reports a
failing coverage table even when its tests pass — judge partial runs by the pass/fail counts,
not the exit code, and gate on a full `bun run test`.

`$lib/*` and `$app/*` resolve through `.svelte-kit/tsconfig.json` for both `svelte-check` and
`bun test`. If imports suddenly fail to resolve, run `bunx svelte-kit sync`.

Local order matches CI (`.github/workflows/ci.yml`): `check:biome` → `check` → `test`,
plus `build` and `smoke:production`. Commits are Conventional Commits (enforced at `commit-msg`).

## Layout

- `src/lib/server/**` — server-only domain modules. SvelteKit keeps them out of the client bundle.
- `src/lib/**` (non-`server`) — shared zod schemas, client helpers, components. The server
  re-exports shared schemas (e.g. `src/lib/server/slides/types.ts` does `export * from
  '$lib/slides/types'`).
- `src/routes/**` — thin loads and form actions delegating to `$lib/server/*`. No business logic.
- `tests/` — `unit/` mirrors the domain folders, plus `property/` (fast-check), `integration/`,
  `helpers/`. Bun's runner only: `tests/unit/test-architecture.test.ts` fails the build if a test
  imports Vitest/Jest APIs or nests under `tests/unit/server/`.

Generated, do not hand-edit: `drizzle/meta/**` (drizzle-kit snapshots), `.svelte-kit/`, `build/`.
A `drizzle/NNNN_*.sql` body may be hand-extended only for data salvage — `0010_cynical_screwball.sql`
is the worked example. `docs/` is gitignored scratch space; never add tracked docs there.

## Gotchas

- **The test DB schema is hand-mirrored.** `tests/setup.ts` builds the in-memory schema with raw
  `CREATE TABLE` statements; migrations never run against `:memory:`. Any change to
  `src/lib/server/db/schema.ts` must be mirrored there, and new tables added to
  `sharedTestDbTables` / `resetSharedTestDb()` in `tests/helpers/db.ts`.
- **Never wrap migration SQL in `BEGIN`/`COMMIT`** — drizzle's bun-sqlite migrator already runs
  each file in one transaction.
- **One DB handle.** Import `db` / `sqlite` from `src/lib/server/db/client.ts`; do not construct
  another `Database` in app code (pure/property tests may, and must say so).
- **Env outranks DB for settings.** `src/lib/server/admin/settings.service.ts` owns `app_settings`,
  and `clearConflictingDbSettings()` drops DB rows an env var now controls at startup. Read
  settings through the service, not `process.env`, inside routes.
- **Admin settings writes go through OCC**, not `setAppSetting`: expose
  `settingsVersionISO(await getAppSettingsUpdatedAt(KEYS))` in `load`, call `inlineOccCheck` or
  `externalOccCheck` in the action, wrap the action map in `requireAdminActions({...})`, and pass
  `surfaceOccConflict` to superForm's `onUpdate` — without the last step a discarded 409 write
  still renders a success toast.
- **`OCC_CONFLICT_CODE` / `OCC_CONFLICT_MESSAGE` are duplicated across the server/client boundary**
  (`server/admin/occ-helpers.ts` and `utils/occ-form.ts`) and asserted verbatim by many tests.
  Change all three in one commit.
- **Guard form actions explicitly.** Actions do not run the route's redirecting `load`, so an
  unauthenticated POST lands in the handler with `locals.user` undefined. Use
  `requireAdminActions` / `requireUserActions` from `$lib/server/auth/guards` rather than the
  route load or ad-hoc `if (!locals.user?.isAdmin)`.
- **Hook order in `src/hooks.server.ts` is load-bearing.** `proxyHandle` is the only place
  `X-Forwarded-*` may rewrite the URL (gated on `TRUST_PROXY`), so everything downstream — the HSTS
  decision included — reads `event.url`, never the raw headers.
- **CSRF is app-owned.** `svelte.config.js` sets `csrf.trustedOrigins: ['*']` on purpose because

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [edbfi/obzorarr](https://github.com/edbfi/obzorarr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
