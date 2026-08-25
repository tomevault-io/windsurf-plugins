---
trigger: always_on
description: <!-- BEGIN:nextjs-agent-rules -->
---

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

Next.js is pinned to `16.2.10`; APIs and file conventions may differ from training data. Before changing Next-specific behavior, read the relevant guide under `node_modules/next/dist/docs/` and heed deprecations.
<!-- END:nextjs-agent-rules -->

# Stackray repo guide

## Runtime and commands

- Use Node `24.x` and `pnpm@10.26.1`; CI enables pnpm with Corepack and installs with `pnpm install --frozen-lockfile`.
- CI quality order is `pnpm lint` -> `pnpm typecheck` -> `pnpm test` -> `pnpm test:performance-contracts` -> `pnpm test:railway-template` -> `pnpm build`.
- Focused unit tests: `pnpm vitest run path/to/file.test.ts` (`vitest.config.ts` includes `**/*.test.{ts,tsx}` and uses `jsdom`).
- E2E: `pnpm test:e2e` starts `pnpm db:migrate:startup && pnpm dev --hostname 127.0.0.1 --port ${PLAYWRIGHT_PORT:-3100}` and needs Postgres; set `STACKRAY_E2E_USE_SYSTEM_CHROME=true` to use system Chrome like CI.
- DB-backed smoke tests need Postgres and startup migrations first: `pnpm test:scan-pipeline-smoke` exercises fake scanners plus `http`/`intel`/`browser` workers; `pnpm test:railway-template` validates the Railway service template.
- `pnpm build` runs `next build` and then `scripts/copy-next-standalone-assets.ts`; `pnpm start` always runs `pnpm db:migrate:startup` before `.next/standalone/server.js`.
- `pnpm dev:local` is the default local stack: it creates `.env.local` if missing, chooses per-worktree ports, starts Postgres/MinIO, runs migrations, seeds `admin@stackray.local` / `StackrayDev123!`, then runs host Next.js plus Docker workers.
- `pnpm dev:local:down` stops this worktree's Docker services without deleting volumes; `pnpm dev:local:wipe` deletes local Postgres/MinIO volumes.

## Local environment and services

- `.env.local` takes precedence over `.env`, and setup scripts never overwrite an existing `.env.local`; check it is not pointing at Railway before worker or migration testing.
- Before launching `pnpm dev:local`, `pnpm dev`, or local hands-on QA servers, check tmux for an existing Stackray dev session. In the main `~/projects/stackray` checkout, reuse the existing session if one is running; in a separate worktree, start that worktree's own tmux session with `pnpm dev:local` so it gets isolated ports, Compose project, and volumes.
- Local app runs on the host; Postgres and MinIO run in Docker; worker containers provide `httpx`, `nuclei`, `subfinder`, nuclei templates, and browser/screenshot Linux dependencies.
- First local stack defaults: app `http://localhost:3000`, Postgres `127.0.0.1:5432`, MinIO API `127.0.0.1:9000`, MinIO console `127.0.0.1:9001` with `minioadmin` / `minioadmin`.
- Worker roles are split by `STACKRAY_WORKER_ROLE`: `http` handles `http_probe/run_scan`, `intel` handles subfinder/nuclei/ip/finalize/schedules, `browser` handles headless/browser fallback, and `all` handles every task.
- Worker production images are intentionally slim and built by `worker/Dockerfile`; when changing worker startup commands, runtime imports, scripts, or assets, verify the Dockerfile copies every required file into `/app`.

## Architecture boundaries

- Stackray intentionally uses HTTP/JSON plus SSE, not `tRPC`, `oRPC`, or GraphQL. The web UI, agent API, and workers share persisted records as the source of truth.
- API/UI entrypoints live under `app/`; product route handlers call services in `lib/server/**`; shared payload schemas live in `lib/contracts/**`; worker orchestration lives in `worker/**`.
- `drizzle/schema.ts` is the database source of truth; `lib/db/schema.ts` is only the app-facing re-export.
- Product-resource API routes should accept either Better Auth session cookies or bearer API keys via `requireSessionOrBearerActor`; account/admin/API-key/user-product-state routes should stay session-only via `requireAppSession`. `lib/session/route-auth-boundaries.test.ts` enforces this split.
- Queue Graphile work through `enqueueGraphileJob`; scan creation queues `http_probe`, and worker task selection is role-based rather than one monolithic worker path.
- Scan progress is SSE from persisted `scan_events`; results must be written before being streamed.

## Drizzle, Graphile, and Railway migrations

- `scripts/startup-migrate.ts` is the canonical runtime migration path. Keep it: it loads `.env.local`/`.env`, takes a Postgres advisory lock, applies checked-in Drizzle migrations, then runs Graphile Worker migrations.
- Normal schema changes: edit `drizzle/schema.ts`, run `pnpm db:generate`, run `pnpm db:migrate:startup` against local Postgres, test with `pnpm dev:local`, and commit the generated SQL plus `drizzle/migrations/meta/*`.
- Do not rewrite the checked-in `0000_*` baseline for normal schema evolution. Current migration history already includes incremental `0001+` files; append the next migration.
- Avoid manually editing generated Drizzle artifacts (`drizzle/migrations/*.sql`, `drizzle/migrations/meta/*`) during normal work. If generated SQL looks wrong, fix the schema/source and regenerate.
- A full migration-history reset is exceptional and intended only for fresh/reset databases and template cutovers. If any database has applied an older migration lineage, do not reset checked-in history without an explicit reconciliation plan.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CarlosCommits/stackray](https://github.com/CarlosCommits/stackray) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
