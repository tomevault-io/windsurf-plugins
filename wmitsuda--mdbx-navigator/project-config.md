---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md

Guidance for AI coding agents working in this repository.

## What this project is

**MDBX Navigator** is a read-only web UI for browsing the contents of a
[libmdbx](https://github.com/erthink/libmdbx) key-value database, in the spirit
of the table browsers that exist for SQL databases. It is split into two parts
that ship as a **single binary**:

- A **Go backend** that opens an `mdbx.dat` file from the local filesystem
  (read-only, exclusive mode) and exposes the raw KV data over a small REST API.
- A **React Router 7 / React frontend** (SPA, framework mode) that consumes
  those APIs and renders a read-only browser. In production the built frontend
  is embedded into the Go binary via `go:embed` and served by the same process;
  during development it can be run separately against the backend.

Everything the UI shows is raw bytes rendered as `0x…` hex strings — there is no
schema interpretation of keys/values.

## Repository layout

```
main.go                 CLI entrypoint (urfave/cli v3): flags, opens the env, starts the HTTP server
mdbxnav/                Core mdbx reading logic, decoupled from HTTP
  types.go              Table / KVResult / ValueResult structs (JSON-tagged)
  tables.go             ReadTables: enumerates DBIs + per-table stats at startup
routes/                 chi HTTP handlers (Backend struct + REST endpoints)
  router.go             Route table, CORS, and the embedded-FS static file server
  alltables.go          GET /api/tables
  table_forward.go      GET /api/table/{table}/forward  (+ readForward cursor helper)
  table_backward.go     GET /api/table/{table}/backward (+ readBackward cursor helper)
  table_search.go       GET /api/table/{table}/search   (prefix / range search)
  getvalue.go           GET /api/table/{table}/value     (single full value)
  util.go               Shared query-param parsing (readParams / readKey)
web/                    Frontend (React Router 7 SPA + Tailwind v4)
  static.go             package web — go:embed all:build/client → web.FS
  react-router.config.ts  React Router framework config (ssr: false → SPA mode)
  vite.config.ts        Vite 8 + @react-router/dev + @tailwindcss/vite plugins
  eslint.config.mjs     Flat ESLint config (ESLint 9)
  app/routes.ts         Route manifest (flatRoutes() — keeps the app/routes/ file convention)
  app/routes/           File-based routes (clientLoader-only, ssr disabled)
  app/components/       UI components (Results, NavBar, hex rendering, etc.)
  app/tailwind.css      Tailwind v4 entry (@import "tailwindcss" + @theme overrides)
  app/types.ts          Shared TS types + constants (incl. BACKEND_URL)
.goreleaser.yaml        Release/build config; embeds the frontend into the binary
doc/                    Screenshot used by README
```

## Build, run, and dev workflows

### Toolchain
- **Go** 1.26.x with **CGO enabled** — `mdbx-go` is a cgo binding, so a working
  C toolchain is required and `CGO_ENABLED=1` must be set. Pure-Go cross
  compilation will not work. (The `go` directive in `go.mod` is `1.26.4`.)
- **Node** 24.16.0 (pinned in `web/.nvmrc`; run `nvm install` inside `web/`).
- **pnpm** is the package manager (not npm). Install it standalone or via
  `corepack enable`. The frontend has a committed `web/pnpm-lock.yaml`.
- **goreleaser** for producing the embedded single-binary build.

### Full binary (frontend embedded)
```shell
goreleaser build --snapshot --clean --single-target
```
goreleaser's `before` hooks run `go mod tidy`, `pnpm --dir web install
--frozen-lockfile`, and `pnpm --dir web run build` first, then builds the Go
binary. The output lands in `./dist/mdbx-navigator_<arch>/`.

> Important: the Go build embeds `web/build/client` via `//go:embed all:build/client`.
> A plain `go build ./...` **fails** unless that directory exists. Build the
> frontend first (`pnpm --dir web run build`) or use goreleaser, which does it
> for you.

### Run
```shell
./mdbx-navigator --data <path-to-your-mdbx.dat>
```
CLI flags (see `main.go`):
- `--data` (required) — path to the `mdbx.dat` file.
- `--host` (default `127.0.0.1`) — bind address.
- `--port` (default `56516`) — bind port.
- `--lengthcap` (default `32`) — max value length (bytes) returned in list
  results before truncation; full values come from the `/value` endpoint.

The DB is opened **read-only in exclusive mode** with `OptMaxDB = 1000`. The app
is browse-only by design — do not add write paths. The UI is served at
`http://127.0.0.1:56516/`.

> Exclusive mode means the open **fails if another process already holds the
> file** (e.g. a running Erigon node). Stop the writer first, or point `--data`
> at a copy/snapshot. Real Erigon chaindata has ~100+ tables, many with
> `entries=0` — that is normal, and `forward`/`backward`/`search` on an empty
> table just return `[]`.

### Frontend-only dev loop (inside `web/`)
```shell
pnpm install       # restore deps from pnpm-lock.yaml
pnpm run dev       # React Router dev server (Vite)
pnpm run build     # production build → build/client (what gets embedded)
pnpm run lint      # eslint (flat config)
pnpm run typecheck # react-router typegen && tsc
```
Run a backend separately (`go run . --data <file>` after building the frontend,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wmitsuda/mdbx-navigator](https://github.com/wmitsuda/mdbx-navigator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
