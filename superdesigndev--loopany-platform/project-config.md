---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**Loopany** - multi-user scheduled **agent loops**. The server (TanStack Start)
schedules/stores/authenticates/notifies; execution is **BYOA** - claude-code runs on
each user's own machine via the `@crewlet/loopany` daemon. **Zero-exec invariant:
the server runs no LLM and executes no user code** - it only stores/reads bytes and
computes pure functions. Run instructions: `README.md`.

## Layout (pnpm monorepo)

- `packages/server` (`@loopany/server`) - TanStack Start UI + server fns +
  in-process Scheduler (croner) + machine routes + Better Auth + push notifications.
  Drizzle over Postgres (tiered driver: embedded pglite when `DATABASE_URL` is
  unset, postgres-js on Supabase when set).
  - `src/scheduler/` - cron engine (tick -> pending run -> Dispatcher).
  - `src/gateway/` - machine gateway (`index.ts`: `MachineGateway`, the
    poll/report run-lifecycle core + owner verbs + retention/GC; `cli.ts`:
    `CliGateway`, the credential-keyed CLI dispatch for /api/machine/cli +
    /agent-api/loop; `validate.ts`: the ONE ui/workflow/schema validator module
    both write surfaces import; `sync.ts`: `ArtifactSync`, the sync/blob byte
    ingress - boot shares ONE blob store between the classes), run tokens,
    delivery, prompt, notify, blobstore (R2/in-memory), artifacts.
  - `src/db/` - Drizzle schema
    (machines/loops/runs/blobs/artifact_files/run_snapshots/run_leases/connect_keys)
    + store + auth-schema.
  - `src/server/` - boot (`ensureServer`), adapters (Loop/Run -> JobSummary/JobDetail),
    loopApi server fns.
  - `src/skill/` - ALL prompt/skill prose (see "The skill" below).
  - `src/routes/` - pages + server-only route files.
- `packages/daemon` (`@crewlet/loopany`) - one binary, two roles: poll-loop daemon
  and the in-run `loopany` callback; spawns claude.

## Commands

- `pnpm dev` - server on :3000 (UI + scheduler + machine routes).
- `pnpm -r typecheck` - both packages (server typecheck runs `tsr generate` first,
  so a fresh checkout typechecks with no prior build).
- `pnpm --filter @loopany/server test` / `pnpm --filter @crewlet/loopany test` -
  vitest; single file: append the path; single test: `vitest run -t "<name>"`.
- `pnpm --filter @loopany/server db:generate` / `db:migrate` - Drizzle migrations.
- `bash scripts/demo-cookie-unified.sh` - e2e demo loop through the unified server.
- Prod: nitro build, then `pnpm start` = `scripts/prestart.mjs` +
  `node .output/server/index.mjs`. prestart applies pending migrations via the
  postgres-js migrator over `DIRECT_DATABASE_URL` for the hosted Supabase tier
  (when `DATABASE_URL` is set; fails loud if that would route DDL over the :6543
  pooler); the embedded pglite tier migrates in-process at boot - prestart just
  gates it (no `DATABASE_URL` requires the explicit `LOOPANY_DB=pglite` opt-in,
  exit 1 otherwise, so a lost DB secret can't silently boot an empty pglite).

## Core model

- Scheduler tick creates a pending run; the bound machine's **HTTP poll** claims
  it (stateless, not WS: an IDLE daemon opts into a server-held long-poll -
  `wait:true`, ~20s hold, the Dispatcher wakes it on a new pending run for
  near-zero dispatch latency; with a run in flight it stays the classic ~3s
  short poll so the progress heartbeat flows; old daemons/servers degrade to
  plain short-poll on both sides); the daemon spawns claude; the agent talks back via
  run-token verbs (`loopany report/show/set-*/reschedule/finish`, `/agent-api/loop`);
  the final `report()` persists transcript/metrics/artifacts and retires the run lease.
- Run roles: `exec` (scheduled run), `evolve` (self-improvement pass), `edit`
  (owner-requested change). Only exec runs produce user-facing notifications,
  success or failure.
- **Open vs closed loops**: closed-ness derives from `loops.goal != null` (no kind
  column). A closed loop's exec run gets `canFinish` and may call `loopany finish`
  when the goal is met, stamping `completedAt`/`completionReason` + `enabled=false`.
  The invariant `completedAt != null implies goal != null` is enforced at the single
  write chokepoint `store.updateLoop`, which also runs lifecycle side effects for
  every caller: `goal:null` clears completion stamps; `enabled:true` on a completed
  loop is a reopen; `enabled:false` is a plain pause.
- A loop's standing brief lives ONLY in its task file's `## Spec` (there is no
  `task` column). The exec run's instructions live ENTIRELY in the first user turn
  (`buildExecTask` ← `skill/run/exec-core.md`): the self-sufficient CORE (identity +
  untrusted-data guard + the non-negotiable fallback core - read task file first, do
  the work / surface only what changed, end with exactly ONE `loopany report`/`finish`,
  `{{stateLine}}` report grammar, one pass then stop + per-run trigger + a pointer to
  the installable loopany skill for the deep protocol). `buildLoopSystemPrompt` returns
  `""`; on an OLD daemon `--append-system-prompt-file` then points at an empty file (a
  harmless no-op, so batches 1-2 shipped server-first with no daemon change), and the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [superdesigndev/loopany-platform](https://github.com/superdesigndev/loopany-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
