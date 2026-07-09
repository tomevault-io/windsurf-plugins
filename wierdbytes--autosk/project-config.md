---
trigger: always_on
description: This file provides guidance when working with code in this repository.
---


# autosk - task manager and workflow manager for coding agents

This file provides guidance when working with code in this repository.

## Architecture in one paragraph

autosk is split into a **Go front end** (the `autosk` CLI + the `autosk lazy`
TUI) and a **Bun/TypeScript daemon** (`autoskd`). The Go binary is a pure
JSON-RPC client — it owns no storage and links no database; it auto-spawns and
talks to `autoskd` over a Unix socket (proto-v2). `autoskd` (under `daemon/`)
is the sole owner of a project's `.autosk/` directory: tasks, comments, and
session transcripts live as files (there is **no database**), and workflows +
agents are **code** registered by extensions. The Tauri desktop GUI (`gui/`)
is a third front end — also a pure JSON-RPC client of `autoskd`.

There is no Rust workspace and no embedded database engine anywhere in this repo
any more (the only `cargo` consumer is the standalone Tauri backend,
`gui/src-tauri`).

## Build, test, lint

The Go binary builds CGO-free with plain `go build` / `go test` — no build tags,
no native libraries.

- `make build` — compiles `bin/autosk` (entrypoint: `cmd/autosk`), CGO-free
- `make build-autoskd` — compiles the Bun daemon into `bin/autoskd` with
  `bun build --compile`; the compiled binary embeds the Bun runtime, so **no
  global `bun` is needed at runtime**. There are no bundled extensions: the
  reference `@autosk/feature-dev` workflow is an npm package the daemon installs
  on first run (`ensureGlobalBootstrap`)
- `make test` — builds `autoskd` first, then runs `go test ./...` with
  `AUTOSKD_BIN` pointed at the compiled daemon (the verb-test harness seeds its
  own temp `~/.autosk/extensions/` fixture + an empty `settings.json`, so no
  real `npm install` runs)
- `make test-short` — same with `-short`
- `make vet` — `go vet ./...`
- `make fmt` — `gofmt`
- `make lint` — `golangci-lint run ./...` (requires `golangci-lint` installed)

The `cmd/autosk` verb tests are RPC clients: they need a live `autoskd` on disk,
which they **auto-spawn** after locating it via `$AUTOSKD_BIN`. A direct
`go test ./...` works without any tag, but the verb tests `t.Skip` unless
`autoskd` has been built (`make build-autoskd`) and `$AUTOSKD_BIN` is set —
which is why `make test` builds it first.

### Bun daemon workspace (`daemon/`)

`autoskd` and its public SDK + shipped extensions live in a Bun workspace under
`daemon/`. Checks run from `daemon/`:

- `bun install` — install + symlink the workspace packages (`bun install
  --frozen-lockfile` in CI)
- `bun run typecheck` — `tsc --noEmit` across every workspace package
- `bun test` — every package's `*.test.ts` (pure unit tests; no spawned daemon)
- `bun build --compile core/src/index.ts --outfile <path>` — produce the
  standalone daemon binary (`make build-autoskd` / `scripts/package-autoskd.sh`
  wrap this)

### Tauri GUI (`gui/`)

The desktop app is a React/Vite front end over a thin Tauri (Rust) backend that
is a **pure JSON-RPC client of `autoskd`** — `gui/src-tauri` is a standalone
cargo crate (there is no root cargo workspace) and links no storage engine.
Front-end checks run from `gui/`:

- `npm ci` — install (uses `gui/package-lock.json`)
- `npm run typecheck` — `tsc --noEmit` **plus** the
  `scripts/check-ipc-discipline.mjs` guard (one `invoke` site in
  `src/services/ipc.ts`, one `listen` site in `src/services/events.ts`)
- `npm run build` — `tsc && vite build` (production bundle)
- `npm run test` — `vitest` (pure reducer/selector/ipc logic; no browser or daemon)

Backend check runs from `gui/src-tauri/`:

- `cargo check` — typecheck the Tauri backend (a self-contained crate)

### Marketing site (`website/`)

The landing page is a static **Astro + Tailwind v4** site (dark-first, reuses the
GUI brand tokens) with **no backend** — `npm run build` emits a plain `dist/`
deployable to any static host. Checks run from `website/`:

- `npm install` — install (uses `website/package-lock.json`)
- `npm run dev` — `astro dev` at http://localhost:4321
- `npm run build` — `astro build` → `website/dist/`
- `npm run check` — `astro check` (type/diagnostics)
- `npm run preview` — serve the built `dist/` locally

App icons: sources live in `gui/src-tauri/icons/src/` (`autosk.icon` +
`autosk.png`) and all binary icon artifacts under `icons/` are stored in Git LFS
(run `git lfs install` after cloning). Regenerate the Liquid Glass `Assets.car` +
flat `.icns`/`.ico`/PNG fallback with `scripts/update-gui-icons.sh` (no args =
repo sources; pass `<App.icon> <icon-1024.png>` to adopt a new icon); full usage
is in the script header.

## Repo layout

- `cmd/autosk/` — Cobra CLI entrypoint (proto-v2 RPC client)
- `internal/lazy/` — lazygit-style TUI (`autosk lazy`), built on `jesseduffield/gocui`
- `internal/daemon/` — JSON-RPC client of `autoskd` (`rpcclient`, auto-spawn)
  plus the shared wire/view types (`api`, `runstore`) mirrored from
  `daemon/sdk/src/proto.ts`
- `internal/projectdb/`, `internal/store/` — `.autosk/` directory resolution
  (walk-up; no DB file) + the task/workflow/agent/session view types the front
  ends render over RPC
- `daemon/` — the Bun/TypeScript daemon workspace:
  - `daemon/sdk/` — `@autosk/sdk`: the public, extension-facing types

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wierdbytes/autosk](https://github.com/wierdbytes/autosk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
