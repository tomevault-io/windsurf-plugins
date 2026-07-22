---
trigger: always_on
description: Instructions for AI coding agents working on servo-fetch. This file complements, not replaces, [`CONTRIBUTING.md`](CONTRIBUTING.md) (human-oriented) and [`skills/servo-fetch/SKILL.md`](skills/servo-fetch/SKILL.md) (consumer-oriented). Treat this file as living documentation; update it when conventions change.
---

# AGENTS.md

Instructions for AI coding agents working on servo-fetch. This file complements, not replaces, [`CONTRIBUTING.md`](CONTRIBUTING.md) (human-oriented) and [`skills/servo-fetch/SKILL.md`](skills/servo-fetch/SKILL.md) (consumer-oriented). Treat this file as living documentation; update it when conventions change.

## Project overview

servo-fetch is a Rust workspace with two crates:

- **`servo-fetch`** (library): wraps the [Servo](https://servo.org) browser engine to fetch, render, and extract web content. Main entry points: `fetch()`, `crawl()` / `crawl_each()` (streaming), `map()` (sitemap discovery).
- **`servo-fetch-cli`** (binary): CLI + HTTP API + MCP server. Depends on `servo-fetch`.

Language bindings live outside the cargo workspace in `bindings/python/` (PyO3/maturin) and `bindings/node/` (TypeScript wrapper over the CLI; `bindings/node` is excluded from the workspace).

Toolchain pinned in `rust-toolchain.toml`; MSRV declared as `rust-version` in `Cargo.toml` — do not use features stabilized after the MSRV. Workspace-wide version bumping via `[workspace.package].version`.

The Servo dependency dominates build time (several minutes on first build, ~30-40 minutes for release). Plan long-running commands accordingly.

## Setup and commands

Run these from the workspace root unless noted. Copy-paste ready:

```sh
cargo build                                             # Debug build
cargo build --release                                   # Release build (slow)
cargo +nightly fmt --all                                # Format
cargo +nightly fmt --all -- --check                     # CI fmt check
cargo clippy --workspace --all-targets -- -D warnings   # Lint (pedantic, warnings are errors)
cargo test --workspace                                  # Unit + integration tests (fast)
cargo test -- --ignored                                 # Include Servo-backed e2e tests (slow)
cargo deny check                                        # License + advisory audit
taplo check                                             # TOML lint (install: cargo install taplo-cli --locked)
taplo fmt                                               # TOML format
typos                                                   # Spell check
```

Nextest profiles (preferred over `cargo test` in CI):

```sh
cargo nextest run --workspace --profile ci                                   # Unit + integration, CI output
cargo nextest run -p servo-fetch-cli --run-ignored only --profile ci-e2e     # E2E, retries on flake
```

Cargo aliases for exact CI parity (defined in `.cargo/config.toml`):

```sh
cargo lint        # cargo-clippy job
cargo test-ci     # cargo-test job (unit/integration via nextest)
cargo test-doc    # cargo-test job (doc tests)
cargo test-e2e    # e2e job (Servo-backed, retries on flake)
```

Ignored tests require a working Servo environment. Servo uses `SoftwareRenderingContext` internally, so no X server is needed in CI (the apt install list in `release.yml` includes X11 libs for linking, not for runtime). If an `ignored` test fails locally, first retry; if it still fails, check whether the test hits the network or a site that requires `--settle`.

## Code conventions

### Rust style

- **`cargo clippy` is a gate.** Use `-D warnings` in CI; keep it passing locally before pushing.
- **Inline `format!` args** — `format!("x={x}")`, not `format!("x={}", x)`. See the clippy rule `uninlined_format_args`.
- **`#[non_exhaustive]` on public structs/enums** that we expect to grow (e.g., `Page`, `CrawlResult`, `ConsoleMessage`). Do not add new public enum variants or struct fields without it unless the type is intentionally sealed.
- **Fields `pub(crate)` + builder methods** for option structs like `CrawlOptions`, `FetchOptions`, `MapOptions`. Callers construct via `Options::new(url)` and chain `.method(value)`. Adding a new `pub(crate)` field + builder method is always non-breaking.
- **`Result<T, Error>` end-to-end** — crate-local `error::Result`. No `unwrap()` in non-test code except where a panic is documented as the only sane behaviour (e.g., poisoned mutex).
- **Single-concern modules over strict line counts.** Large files are acceptable when they remain one semantic concern (e.g., `crawl.rs` owns all crawl logic). Split when a second concern emerges, not when LoC crosses a threshold.
- **No `#[async_trait]`** — prefer native RPITIT (`fn foo(&self, ...) -> impl Future<Output = T> + Send`) when the return type is nameable.

### Commits

Conventional Commits, **subject-only** by default. Use the body only when critical context cannot fit in the subject. Scopes used in this repo:

- `chore(deps)` — external dependencies the **product ships** (cargo crates, docker base image).
- `chore(ci)` — CI/workflow configuration, github-actions entries, Dependabot config itself.
- `fix(<area>)` — bug fixes; `<area>` is typically a module (`crawl`, `map`, `build`, `tests`) or `ci`.
- `feat(<area>)` — user-facing additions.
- `refactor(<area>)`, `test(<area>)`, `docs`, `perf`, `build`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [konippi/servo-fetch](https://github.com/konippi/servo-fetch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
