---
trigger: always_on
description: - Always use `bun` as the package manager (not npm/yarn/pnpm)
---

# CommunityFix

## Development

- Always use `bun` as the package manager (not npm/yarn/pnpm)
- Secrets are managed via **Doppler** — run `doppler setup --project communityfix --config dev` once, then all scripts automatically inject env vars via `doppler run --`
- Run dev server: `bun run dev`
- Type check: `bunx vue-tsc --noEmit`
- To run an ad-hoc command with the Doppler env: `doppler run -- <command>`
- To target a different config: `doppler run -c stg -- <command>` (or `prd`)

## Database

- PostgreSQL via Drizzle ORM (`drizzle-orm/postgres-js`) with PostGIS, pgvector, and tsvector full-text search
- **Local (dev + tests)**: Docker Postgres via `docker-compose.yml` — image built from `docker/Dockerfile.postgres` (postgis + pgvector). Start with `bun run docker:up`
- **Production**: Neon Postgres, reached from Cloudflare Workers through a Hyperdrive binding (`HYPERDRIVE` in `wrangler.jsonc`). `server/plugins/hyperdrive.ts` hydrates `NUXT_DATABASE_URL` from the binding at request/scheduled time so `useDB()` works unchanged
- Schema: `server/database/schema.ts`
- Custom migrations (extensions, generated columns, GIN/HNSW/GIST indexes): `server/database/migrations/custom/`
- Seed files: `server/database/seed/` (numbered SQL files, applied in order)
- Generate migrations: `bun run db:generate`
- Run migrations (drizzle + custom): `bun run db:migrate` — reads `NUXT_DATABASE_URL`. For Neon prod use the **direct** (non-pooled) connection string for DDL; runtime traffic goes through Hyperdrive on the pooled URL
- Seed: `bun run db:seed` (runs `psql $NUXT_DATABASE_URL` against each file in `server/database/seed/*.sql`)
- Drizzle Studio: `bun run db:studio`

## Deployment

- Builds and deploys run from GitHub Actions (`.github/workflows/ci.yml`), not Cloudflare Workers Builds. Pushing to `master` runs migrations against Neon production then `wrangler deploy`s the `communityfix` Worker; pushing to `staging` does the same against Neon staging and deploys to `communityfix-staging` via `wrangler deploy --name communityfix-staging`.
- The branch-aware Hyperdrive / KV IDs in `nuxt.config.ts` are picked by the `WORKERS_CI_BRANCH` env var, which the workflow sets explicitly (`master` or `staging`) before each build.
- Required GitHub repo secrets: `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID`, `DOPPLER_TOKEN`. Runtime Worker secrets (`NUXT_DATABASE_URL`, `NUXT_SESSION_PASSWORD`, `NUXT_OPENAI_API_KEY`, `NUXT_ANTHROPIC_API_KEY`, etc.) are managed in Doppler (`prd` / `stg` configs) and synced to the Workers on every deploy via `wrangler secret bulk` — Doppler is the source of truth, so add new secrets there, not via the Cloudflare dashboard.
- PRs run `typecheck`, `test`, and `build` (Worker bundle compiles cleanly). All three are required checks on `master` and `staging` per `.github/rulesets/`.

## Nitro Tasks

- Docs: https://nitro.build/guide/tasks
- Tasks live in `server/tasks/` (nested dirs become colon-separated names, e.g. `review/issue.ts` → `review:issue`)
- Enabled via `nitro.experimental.tasks` in `nuxt.config.ts`
- Scheduled tasks (cron) configured in `nitro.scheduledTasks`
- Run programmatically with `runTask('task:name', { payload: { ... } })`
- Dev endpoints: `/_nitro/tasks` (list) and `/_nitro/tasks/:name` (run)

## AI moderation

- AI moderation (OpenAI embeddings + Claude calls + DB writes, ~10–40s) runs as a **Cloudflare Workflow** for durability — Cloudflare cancels `waitUntil()` work 30s after the response, which used to leave issues/case-studies stuck `pending`.
- **All moderation logic lives in a standalone Worker** (`workers/moderation/`), which hosts the `ModerationWorkflow` and talks to Neon directly via its own `HYPERDRIVE` binding (reusing `server/database/schema.ts`). The Nuxt app does **not** contain the review logic — it only triggers reviews. The main Worker binds to the workflow cross-script via `nitro.cloudflare.wrangler.workflows` (binding `MODERATION_WORKFLOW`, branch-aware name `moderation` / `moderation-staging`).
- Flow: trigger sites call `triggerModeration(kind, id)` (`server/utils/moderation-trigger.ts`) → `env.MODERATION_WORKFLOW.create({ params: { kind, id } })`. The worker runs the pipeline as durable, independently-retried steps. Issue review = `prepare` → (`moderate` ∥ `classify-tags` ∥ `map-sdgs`) → `finalize` → on approval an **enrich** pass (`curate` ∥ `resolve-location`) → structural pass. `kind` is `'issue' | 'case-study' | 'structure'`.
- **Prompts are data, not code.** Each AI step's `system`/`user` templates, output JSON Schema, `model`, `maxTokens`, and `version` live in a YAML file under `workers/moderation/src/steps/<kind>/*.yaml`, loaded + Zod-validated once at init by `src/steps.ts` (bundled as text via a Wrangler `Text` rule). Single-shot steps run via `runStep(...)`; **agentic** steps declare `tools: [...]` and run via `runAgent(...)` (model calls tools across turns, then `submit_result`). The imperative `prepare*`/`finalize*`/`apply*` steps in `src/pipelines.ts` build the `vars` and own all DB writes. To tune a prompt, edit the YAML + bump `version`, then `bunx vitest run tests/unit` (snapshots in `tests/unit/moderation-steps.test.ts` guard against silent prompt drift).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mathix420/communityfix](https://github.com/mathix420/communityfix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
