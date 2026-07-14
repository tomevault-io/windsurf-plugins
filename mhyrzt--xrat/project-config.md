---
trigger: always_on
description: `src/` contains the Rust application code. Keep responsibilities separated:
---

# Repository Guidelines

## Project Structure & Module Organization

`src/` contains the Rust application code. Keep responsibilities separated:

- `src/cli/` defines Clap command trees, flags, and CLI parsing tests under
  `src/cli/tests/`.
- `src/app/` contains app runtime bootstrap and command handlers. Keep command
  logic under `src/app/commands/` and keep lifecycle concerns in
  `src/app/daemon/`, `src/app/runtime_service/`, and `src/xray/process_mgmt/`.
- `src/app/events.rs` records structured operational events. Event recording is
  best-effort and must not make the primary operation fail.
- `src/app/context/`, `src/app/config/`, and `src/app/paths/` contain
  application bootstrap, path resolution, and command-specific runtime settings.
- `src/config/` contains import, parsing, normalization, and protocol link
  support. Keep external input parsing separate from runtime config generation.
- `src/xray/` contains Xray parsing, generated runtime config builders, and
  process management.
- `src/singbox/` contains sing-box parsing/translation support.
- `src/prober/` contains connection test runners for TCP, ICMP, download,
  upload, and real-delay flows.
- `src/db/` contains database wiring, schema helpers, records, and repositories,
  split across `src/db/database/`, `src/db/record/`, and `src/db/repository/`.
  Keep event persistence in the existing `events` database/repository modules.
- `src/model/` contains shared domain types.
- `src/server/` contains HTTP API routes, auth, response, and server state.
- `src/tui/` contains terminal UI state, views, keymaps, and data adapters.
- `src/support/` contains small shared helpers for decode, GeoIP, cancellation,
  network, time, and URL handling.
- `migrations/sqlite/` and `migrations/postgres/` hold ordered SQL migrations.
- `docs/src/` holds user-facing documentation.
- `packaging/systemd/` holds user-service templates used by
  `xrat daemon install`; `packaging/desktop/` holds desktop entry and icon
  packaging assets.
- `install.sh` installs release archives from GitHub and runs optional first-run
  setup.
- `.github/workflows/` contains CI and release automation, including musl
  release builds, generated man pages/completions, GitHub releases, Docker image
  publishing, and crates.io publishing.
- `Dockerfile` builds the container image published to GitHub Container
  Registry.
- `testdata/` holds local fixtures such as GeoIP data.

## Build, Test, and Development Commands

- Prefer common commands from `Justfile` over manually spelling out equivalent
  `cargo`, formatting, test, Docker, docs, or packaging commands.
- `cargo build` — compile the project.
- `cargo fmt` — format Rust code.
- `cargo clippy --all-targets -- -D warnings` — run the same lint gate used by
  CI.
- `cargo test -q --locked` — run the test suite quietly with the locked
  dependency graph.
- `cargo run -- <command>` — run the CLI locally, for example:
  - `cargo run --` (defaults to the TUI)
  - `cargo run -- init`
  - `cargo run -- import <input>`
  - `cargo run -- list`
  - `cargo run -- show config <config-id>`
  - `cargo run -- show subscription <subscription-id>`
  - `cargo run -- parse <config-id>`
  - `cargo run -- test <config-id>`
  - `cargo run -- scan`
  - `cargo run -- connect <config-id>`
  - `cargo run -- disconnect`
  - `cargo run -- logs`
  - `cargo run -- serve`
  - `cargo run -- tui`
  - `cargo run -- status`
  - `cargo run -- delete config <config-id>`
  - `cargo run -- delete subscription <subscription-id>`
  - `cargo run -- purge`
  - `cargo run -- daemon install --start`
  - `cargo run -- upgrade`
  - `cargo run -- version`
  - `cargo run -- geoip status`
  - `cargo run -- manpage --output <dir>`
  - `cargo run -- completions <shell>`

Run `just fmt ci` before committing.

## Coding Style & Naming Conventions

Use standard Rust formatting via `cargo fmt` (4-space indentation, rustfmt
defaults). Prefer small modules over large files, and split by capability when
files begin mixing CLI parsing, command orchestration, and domain logic.

Naming:

- files/modules: `snake_case`
- functions: `snake_case`
- structs/enums: `PascalCase`
- CLI flags: long, explicit names such as `--database`, `--include-geoip`, and
  `--format`

Avoid one-letter variable names. Do not add code comments unless they are truly
necessary to explain non-obvious intent or constraints. A short top-of-file
comment in `mod.rs` is acceptable when it clarifies what the module is
responsible for.

## Testing Guidelines

Use Rust’s built-in test framework with `#[test]` and `#[tokio::test]`.

- Keep tests close to the code they validate.
- Prefer focused unit tests for parser/config normalization, CLI parsing, DB
  repositories, and runtime lifecycle transitions.
- Add regression tests when fixing parsing, dedup, scanner, or runtime-session
  edge cases.
- For CLI changes, add or update parser tests in `src/cli/tests/` when behavior
  can be validated without starting external services.
- For repository changes, prefer database tests that exercise both SQLite and
  Postgres paths when the existing helpers make that practical.
- Name tests descriptively, e.g. `parses_list_config_filters`.

## Commit & Pull Request Guidelines


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mhyrzt/xrat](https://github.com/mhyrzt/xrat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
