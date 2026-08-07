---
trigger: always_on
description: Guidance for AI agents working in this repository.
---

# AGENTS.md

Guidance for AI agents working in this repository.

## Working principles

- **No overengineering, and no MVP shortcuts.** Hold the middle path: don't build infrastructure before there's a concrete need (note the seam for later instead), and don't ship quick-and-dirty or "for now" hacks. Build each feature correctly and idiomatically — neither gold-plated nor a placeholder.
- **MVP stage — keep the architecture fluid.** The current structure is not load-bearing legacy. When a new feature doesn't fit cleanly, prefer reshaping the affected part over bolting on an awkward special case.
- **Surgical changes.** Clean up what your change orphaned; leave pre-existing dead code alone. Prefer a library's intended API over a clever shim.
- **English only.** All code, comments, identifiers, docs, and commits.
- **Announce shipped work.** When a user-facing feature or fix lands, offer a changelog entry on the `/blog` feed, then a longer post if it warrants one (`.svx` files in `web/src/posts/`; the `write-changelog` skill drafts them). Skip for internal-only changes.

## What this is

`freehire` ([freehire.me](https://freehire.me)) is an open-source IT job aggregator. Many source parsers feed a pipeline that normalizes jobs into one schema, deduplicates them, and enriches them with AI; served over an HTTP API with rich filters, consumed by a SvelteKit app under `web/`.

Stack: **Go + Fiber v2**, **PostgreSQL**, **sqlc**, **Meilisearch**, **Docker Compose**, **langchaingo**.

## Layout

`internal/<domain>/` — domain packages, the substantial ones carry their own AGENTS.md (see the table below).
`cmd/<name>/` — every binary is a **run-once-and-exit** worker except `cmd/server`. They are cron-driven, not daemons; they need `DATABASE_URL` and exit non-zero on failure.

Non-obvious:

- `migrations/` — the source for **both** sqlc codegen and Postgres initdb. Never edit an applied migration; add a new file.
- `sources/` — YAML board files, not Go. One file per ATS provider, plus `custom.yml` and `telegram.yml`.
- `design-system/` — a separate pnpm package, sibling to `web/`, linked via `link:../design-system`
  (a symlink, not a copy). **Install it before building `web/`** — pnpm does not install a
  linked package's own dependencies.
- `internal/db/` — **generated**; edit `internal/db/queries/*.sql` and run `make sqlc`.
- `services/pii-filter` — a standalone service, not a Go package.

## Commands

```bash
make up / make down / make logs       # start / stop / tail app + postgres in Docker
make run / make psql / make sqlc      # run server on host / psql into DB / regenerate internal/db
make reindex                          # rebuild the Meilisearch jobs index from Postgres
go build ./...  &&  go vet ./...
go test ./...                             # unit tests (no external deps)
go vet -tags=integration ./...            # compiles the tagged tests — run before EVERY push
go test -tags=integration ./internal/db/  # queue integration tests (needs Docker; testcontainers)
```

**`go test ./...` compiles no `//go:build integration` file, and those files are not
confined to `internal/db`** — there are 152 of them across 13 packages, and `internal/handler`
holds 65, which call unexported constructors like `newCVHandlers`. A changed signature
therefore passes every command above except the `vet` line, then fails CI, which runs
`go test -tags=integration ./...` over the whole module. The vet line is the cheap guard:
seconds, no Docker. Run the full tagged suite when you change behaviour rather than a
signature.

Worker gotchas (`go run ./cmd/<name>`, all need `DATABASE_URL`; run `ls cmd/` for the full list):

- `migrate` — run **before** deploying code that reads new schema. `-baseline` records on-disk files without executing; a pre-runner database auto-baselines on first run.
- `ingest` — takes one board file: `go run ./cmd/ingest sources/<provider>.yml` (or `SOURCES_FILE`).
- `enrich` / `tg-extract` — need `LLM_BASE_URL` / `LLM_API_KEY` / `LLM_MODEL`.
- `embed` / `search-drain` / `rollup-facets` / `reindex-companies` — need `MEILI_URL` / `MEILI_MASTER_KEY`. `search-drain` drains `search_outbox` (queued by `cmd/ingest`, atomically with each write) into the live facet index in batches — run it frequently (e.g. every 1-2 min); see [internal/searchdrain/AGENTS.md](internal/searchdrain/AGENTS.md).
- `backfill-derive` — re-derives every deterministic column (facets, `role_fingerprint`, slugs) in one keyset pass; `BACKFILL_CONCURRENCY` tunes the pool. Follow with `make reindex` — it collapses newly-clustered reposts and unions their geography.
- `capture-apply-form` — drains the apply-form capture queue: fetches each queued posting's application form from `greenhouse`/`ashby`/`workable`/`lever` and stores it in `apply_forms`. Needs nothing but `DATABASE_URL`; `APPLY_FORM_CONCURRENCY` (default 4) bounds how hard one run leans on a platform and `APPLY_FORM_MAX_PER_RUN` (default 5000) how much of the backlog it takes — the second matters because the first drain faces a ~185k backlog and an unbounded run would work for hours, which `Type=oneshot` turns into silently skipped timer firings. `recruitee` forms never reach this queue — its listing carries them, so ingest writes them directly.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [strelov1/freehire](https://github.com/strelov1/freehire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
