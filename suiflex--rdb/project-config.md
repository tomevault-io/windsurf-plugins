---
trigger: always_on
description: Native cross-platform database manager (PostgreSQL, MySQL, Redis, MongoDB, SQLite, Cassandra) built with Rust + Slint UI. Monorepo workspace.
---

# RDB — Agent Instructions

## Project

Native cross-platform database manager (PostgreSQL, MySQL, Redis, MongoDB, SQLite, Cassandra) built with Rust + Slint UI. Monorepo workspace.

Alongside the Rust workspace (`app/`, `crates/*`) the repo also holds: `website/`
(Astro marketing site, deployed by Cloudflare Pages' Git integration on every
push to `develop` — not part of the Rust build), `scripts/` (`install.sh` /
`install.ps1`, the curl/iwr entry points for a direct install), and
`npm/`/`packaging/` (the npm postinstall wrapper and package-manager
distribution files for Homebrew/Scoop).

## Build, Lint, Test

A root `Makefile` wraps the common cargo invocations and splits FE (the
`rdb` UI binary) from BE (the `crates/*` libraries) so each side builds and
tests independently. Run `make help` for the full target list.

```bash
make fe-build     # build the rdb UI (FE)
make fe-run       # run the UI
make be-build     # build backend crates only (no FE)
make be-test      # test backend crates only
make fmt-check    # format check   (make fmt to apply)
make lint         # clippy, warnings as errors
make test         # test the whole workspace
make all          # fmt-check + lint + test + build (CI gate)
cargo build --release -p rdb   # release binary
```

## CI

One GitHub Actions workflow per component in `.github/workflows/` — `rdb-app`
plus one per crate: `rdb-core`, `rdb-connstore`, `rdb-driver-postgres`,
`rdb-driver-mysql`, `rdb-driver-redis`, `rdb-driver-mongo`,
`rdb-driver-sqlite`, `rdb-driver-cassandra`, `rdb-driver-mssql`,
`rdb-driver-clickhouse`.
Each has a `paths:` filter, so editing one component only runs that
component's CI (lean).

- All ten backend crates are in the root `Makefile`'s `BE_PKGS` (used by
  `be-build`/`be-test`/`be-check`), so the make targets and CI agree on scope.
  Adding a crate means adding it to both.
- Dependents also watch `crates/core/**`, so a `core` change fans out to retest
  core + all dependents (connstore, drivers, app). Other crates stay independent.
- Backend jobs run `cargo {fmt,clippy} -p <pkg>` and `cargo test -p <pkg> --lib`
  (scoped with `-p`, not the workspace-wide `make` targets). `--lib` runs unit
  tests only; the `tests/integration.rs` targets need Docker, so they stay out
  of CI and run locally via `make test-it`.
- The app job installs Slint system libs and runs `cargo build -p rdb`.
- `audit.yml` runs `cargo audit` on every `Cargo.toml`/`Cargo.lock` change plus
  a weekly sweep (new advisories land with no code change).
- `website.yml` is a CI check only for `website/**` — actual deploy is
  Cloudflare Pages' own Git integration on push to `develop`.

Releases are handled separately by `release-please.yml` (single workspace
release on `develop`): conventional commits drive an auto-maintained release
PR that bumps the version and `app/CHANGELOG.md`; merging it tags `vX.Y.Z`
and cuts a GitHub Release. The `app` package (`rdb`) is the tracked version.

`release-build.yml` does the actual packaging once that tag lands: builds
per-target native binaries (macOS `.dmg` with an ad-hoc-codesigned `.app`,
Windows bare `.exe`, Linux `.tar.gz`), attaches them to the GitHub Release,
and publishes to the `suiflex/homebrew-tap` formula/cask, the
`suiflex/scoop-bucket`, and npm (`@suiflex/rdb`, postinstall downloads the
matching asset). `scripts/install.sh` / `scripts/install.ps1` are the direct
(non-package-manager) install path and hit the same GitHub Releases API.

Release note sections are configured in `release-please-config.json` and are
triggered by conventional commit **type**:

- `feat(app): ...` -> **App Features**
- `feature(driver-<engine>): ...` or `feature(driver): ...` ->
  **Driver Features** (`release-please` treats `feature` like `feat` for minor
  version bumps)
- `fix(<scope>): ...` -> **Bug Fixes**
- `perf(<scope>): ...` -> **Performance Improvements**

Keep the scope specific (`app`, `driver-postgres`, `driver-mysql`, `core`,
`connstore`) so generated changelog lines stay readable.

## Architecture

- `app/` — Slint UI binary (main entry point)
  - `app/src/main.rs` — builds the state, builds the shared closures, calls the
    wiring modules, runs the event loop. It used to hold all of it; keep new
    callbacks out of it.
  - `app/src/wire/*.rs` — one module per area (connect, picker, query, runner,
    browse, tabs, edit, grid, split_pane, editor, find, schema, settings,
    conn_form, update). Each exposes `wire(&MainWindow, &AppState, …)` and
    installs that area's callbacks. `runner` and `editor` are the exceptions:
    they *build* closures (`run_sql`/`run_stream`, `sync_editor`/
    `load_editor_text`) that `main` hands on to the others.
  - `AppState` / `AppFns` (`main.rs`) — the shared `Rc`/`Arc` state, and the
    long-lived closures built from it. A wiring module destructures what it
    needs from these instead of `main` cloning handles per callback.
    `AppState` is deliberately `!Send`: anything crossing onto a tokio task
    clones the specific `Arc` it needs.
  - `app/src/pane.rs` — the `set_p_*`/`get_p_*` accessors. The window exposes a
    separate property per result pane (`cells` / `p1_cells`), so these wrap the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [suiflex/rdb](https://github.com/suiflex/rdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
