---
trigger: always_on
description: Guide for AI coding agents working in this repository.
---

# AGENTS.md

Guide for AI coding agents working in this repository.

## What this is

liseur-sync is a self-hostable reading-position sync and
reading-statistics server in Go, companion to
[Liseur](https://github.com/chmouel/liseur). Read
[docs/DESIGN.md](docs/DESIGN.md) before changing behavior; the design
doc is the authority on protocol semantics, and the implementation plan
(how each milestone maps to code) is in that document's milestones
section.

## Build, test, verify

```
go build ./...          # build everything
go vet ./...            # vet
golangci-lint run ./... # lint (or `make lint` for all linters including openapi)
go test -race ./...     # full suite, always run with -race
go tool templ generate ./internal/webui/    # regenerate after editing .templ files
```

CI (`.github/workflows/`): `test.yaml` runs vet + the full race-enabled
suite on PRs and pushes, against SQLite and a Postgres 17 service
container, checks templ output is committed-fresh, and lints
`docs/openapi.yaml` with redocly. `build.yaml` builds static
linux/amd64+arm64 binaries and a multi-arch image on push, with a
container smoke test.

The store test suite runs against SQLite by default. Set
`LISEUR_PG_TEST_DSN=postgres://...` to also exercise the PostgreSQL
backend (throwaway database).

The web reader has a browser check (`TestReaderOpensInARealBrowser`)
that drives Chromium over CDP. It skips when no Chromium is found,
including in CI. On a desktop run, set `LISEUR_CHROME=/path/to/chrome`
to opt in explicitly; CI likewise runs it only when that variable is set.
Set `LISEUR_READER_SCREENSHOT=/tmp/reader.png` to keep what it saw. Run it
after touching `static/reader*.js`, `reader.templ` or the reader's CSP:
a `srcdoc` document inherits the framing page's policy, so a page CSP
that is slightly too strict renders a blank frame with no error
anywhere.

The README's screenshots are made by `make screenshots`
(`scripts/screenshots.sh`): it fetches real books from Standard Ebooks,
seeds a throwaway server with them and photographs four pages through
the same CDP walk the UI tests use
(`internal/webui/testdata/uishots.mjs`). It needs the network, a
browser, node and jq, so it is never part of a test run. Retake them
when a page in a shot changes shape.

## Architecture in one paragraph

One binary, two subcommands (`serve`, `admin`). Storage goes through the
`store.Store` interface with SQLite (`modernc.org/sqlite`) and Postgres
(`pgx`) backends; per-user monotonic `ops.seq` comes from
`seq_counters`. The native API (`internal/api`) is the only
full-fidelity surface; legacy protocols are edge adapters
(`internal/adapter/kosync`, `internal/adapter/koplugin`) that write
native records only. The web UI (`internal/webui`) is templ + vendored
htmx, no CDN, no build pipeline beyond `go tool templ generate`.

Books come from **watched folders**
([ADR-0017](docs/adr/0017-folders-not-pipelines.md)). A folder is a row
(`id, name, root_path, kind`); `internal/content` walks it, reads
metadata and calls one store method, `ReconcileFolder`, which is the
single write path into the catalog — including for an upload. A folder
an administrator marked `accepts_uploads` can be written into by
`POST /v1/folders/{folder}/books`
([ADR-0023](docs/adr/0023-uploads-land-in-a-folder.md)), and that route
creates a file (or, in a Calibre folder, a Calibre book) and then runs
a pass. It never touches a catalog table itself, so an uploaded book
and a book somebody copied in by hand are the same kind of thing. (It
does write one non-catalog row: the uploader's `user_book_work` link, so
a book whose position was already syncing does not appear twice.)
`DELETE /v1/books/{id}` is its counterpart
([ADR-0025](docs/adr/0025-deleting-a-book.md)), bounded by the same
flag: the file goes and then the row, and only in a folder that accepts
uploads. One watcher goroutine
(`internal/content/watch.go`) triggers a pass at startup, on a
debounced fsnotify event, and on a slow safety timer. There is no ingest
job, no content-addressed store, no quota, no trash and no review queue:
an upload is a file written into a folder and nothing more, and the
only other directory the server writes to is the cover cache.

The API contract is [docs/openapi.yaml](docs/openapi.yaml) — update it
in the same commit as any route/shape change, and keep
[docs/integrating.md](docs/integrating.md) consistent with it.

The store test suite is shared across backends in
`internal/store/storetest`. PostgreSQL tests run when
`LISEUR_PG_TEST_DSN` is set (a dev database lives on civuole; DSN is in
the gitignored `.env`).

This project has **no released version**, so a page, route or shape may
be replaced outright: no deprecation shims, no compatibility redirects,
no migration UI. It does have users running published images, and their
databases are real. Issue #13 is what that distinction costs when it is
forgotten: migration 4 was correct and emptied working libraries. So a
shipped migration is never edited, an upgrade path across published
images is tested, and a migration that changes what an existing
deployment can see says so in `docs/deployment.md` before it ships.
(The compatibility that *does* matter is with other people's software:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chmouel/liseur-sync](https://github.com/chmouel/liseur-sync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
