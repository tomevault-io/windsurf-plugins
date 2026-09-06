---
trigger: always_on
description: CorTeX is a distributed corpus-conversion framework for scholarly documents. We **productized** it
---

# CLAUDE.md — CorTeX conventions for agents

CorTeX is a distributed corpus-conversion framework for scholarly documents. We **productized** it
(admin-only prototype → self-installing, agent-first + human-first app) — the sprint shipped as
**v0.5.0** (PR #369, squash-merged to `main` 2026-06-18). The plan and current-state map live in
**[`docs/PRODUCTIZING_PLAN.md`](docs/PRODUCTIZING_PLAN.md)** — read it before non-trivial work.
Active branch: **`main`** (the `productize-2026` sprint branch was merged and deleted).

## What this system is (the 60-second model)

- **Postgres (Diesel 2.2) = metadata store.** It holds `corpora`, `services`, one `tasks` row per
  `(corpus, service, document-entry)`, five severity-partitioned `log_*` tables, `historical_runs`
  (per-run tallies), `historical_tasks` (per-task status snapshots), `worker_metadata`. Document
  **bytes live on a shared filesystem** (`/data/...`); `tasks.entry` is the absolute path to a
  document's source archive.
- **ZeroMQ dispatcher** (`bin/dispatcher.rs`, `src/dispatcher/`) leases TODO tasks to **workers**
  (the external `pericortex` crate), streams sources out (ventilator, port **51695**), receives
  result archives (sink, port **51696**), parses each result's `cortex.log` into a status +
  messages, and persists via the finalize thread.
- **Rocket frontend** (`bin/frontend.rs`, `src/frontend/`, Tera `templates/`) renders read-only
  reports and a few token-gated writes (rerun, save-snapshot).

## Load-bearing facts (don't get burned)

- **Task status is a signed int** (`src/helpers.rs::TaskStatus`): `TODO=0`, `NoProblem=-1`,
  `Warning=-2`, `Error=-3`, `Fatal=-4`, `Invalid=-5`, `Blocked<-5`, `Queued>0` (a positive lease
  mark). These ints are also hardcoded in `scripts/*.sh`.
- **Magic service ids:** `1=init`, `2=import`, `>2`=real services. Code relies on this.
- **DB connections are POOLED now (Arm 3 landed).** The frontend injects an **r2d2 `DbPool`** into
  handlers via Rocket `State` (~110 handlers `pool.get()`; ~5 legacy `Backend::default()` callers
  remain). The dispatcher's ventilator + finalize each hold **one long-lived** `PgConnection`, and the
  **`WorkerMetadata` writer is a single pooled background thread** (D-1 replaced the old
  thread+connection-per-ZMQ-transaction spawn). **Don't add new unpooled per-event connections.**
- **DB URL is now RUNTIME config** (Arm 1 landed): `backend::default_db_address()` reads
  `config().database.url` from figment (`src/config.rs`) — precedence: defaults → `cortex.toml` →
  `CORTEX_`-prefixed env (`CORTEX_DATABASE__URL`) → legacy `DATABASE_URL`/`.env` (loaded at runtime via
  `dotenvy`, highest precedence). **No recompile to switch databases** — e.g. point the frontend at a
  populated DB with `DATABASE_URL=… cargo run --bin frontend` (see `docs/TEST_DRIVE.md`). The old
  compile-time `dotenv!`/`DEFAULT_DB_ADDRESS` baking is gone.
- **Admin/API tokens have ONE source: the JSON token file** (`config.rs::TokenFile`), read by
  `auth_file_path()` — default `config.json` in the CWD, override `CORTEX_AUTH_FILE` (prod keeps the
  live token outside the repo at `/etc/cortex/config.json`; the repo `config.json` is the gitignored
  demo/test fixture; `config.example.json` is the tracked template). It is **not** layered with a
  `cortex.toml [auth]` section — `CortexConfig.auth` is `#[serde(skip)]`, so figment never parses one
  and there's no file-vs-file override. `cortex set-admin-token`/`revoke-token` **read/modify/write
  this same JSON file** (not `cortex.toml`); a minted token activates immediately (the frontend
  re-reads it per gated request). `cortex init` scaffolds an **empty** token file if none exists.
- **Redis has been removed** (Arm 14 #6.2). Frontend drill-down reports are served from the
  per-`(corpus, service, severity)` **`report_grain_cache`** table (`src/backend/rollup.rs`,
  `reports::task_report`) — the global `report_summary` matview was **retired** (migration
  `…retire_report_summary_matview`). A slice is (re)populated **lazily on a cold miss** and
  invalidated on the rerun / run-completion / manual-refresh paths. The heaviest slice — full-arXiv
  `info`, ~127 s (it aggregates ~255M `log_infos` rows; `EXPLAIN ANALYZE` 2026-06-28, see
  POSSIBLE_UPGRADES P-6) — is **never** computed on the request thread: `serve_report` tries a 4 s
  budget-bounded inline populate (`rollup::populate_scope_bounded`) and hands an overrun to a
  **background job** (`jobs::spawn_report_populate`), rendering a self-refreshing "report computing"
  page (`concerns::defer_cold_slice`). The old `cached/worker.rs` cache daemon, the `redis` crate,
  and the dead `CacheConfig` are gone. **The frontend boots without Redis.** (The thin uncached proxy
  formerly at `src/frontend/cached/` was renamed to `src/frontend/render.rs`.)
- **CWD-coupled:** `load_config()` reads `config.json` from the CWD (panics if missing), and
  `Rocket.toml`/`templates/`/`public/` are CWD-relative — **run binaries from the repo root.**
- **The dispatcher panics on purpose** (mutex poisoning → process abort → external restart). Don't
  "fix" those panics into silent recovery; preserve fail-fast where it's the design (see Arm 4/12).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dginev/CorTeX](https://github.com/dginev/CorTeX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
