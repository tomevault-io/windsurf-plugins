---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# KROMA

Self-hosted, direct-play media stack: a Rust server (axum + SQLite + out-of-process
module sidecars) and a Bun workspace monorepo of web, TV, mobile and desktop clients
sharing one universal component library.

See [`README.md`](README.md) for the product overview, [`ARCHITECTURE.md`](ARCHITECTURE.md)
for the structural north-star, and [`CONTRIBUTING.md`](CONTRIBUTING.md) for setup.

## Read before writing code

- [`CODE_STYLE.md`](CODE_STYLE.md) for how code is written here. **The default is no
  comment.** Document exported API only; never private functions or fields. The one
  exception is a kit component's props, which are its public API.
  Never narrate your work in a source file.
- [`CONVENTIONS.md`](CONVENTIONS.md) for cross-file house rules (zod at trust
  boundaries, where secrets live, the values that are never written raw).
- [`packages/ui/src/components/README.md`](packages/ui/src/components/README.md) for
  the component hierarchy: the six levels, what earns a place at each, and the
  three doors out of the kit.
- [`packages/ui/src/components/DESIGN.md`](packages/ui/src/components/DESIGN.md) for
  **how a component's API is shaped**: the part vocabulary, when a `data` prop
  beats children, the controlled/uncontrolled signature, prop naming, and why
  this kit has no `asChild`. Read it before adding a component or changing
  one's props.
- [`modules/README.md`](modules/README.md) for authoring a module.
- [`docs/tv-pairing.md`](docs/tv-pairing.md) for the three roads a television
  takes to an account, what "the same network" means, and which shells can hear
  a television on the link rather than being told about it.

## Checks

These are the CI hard gates (`.github/workflows/ci.yml`, see
[`docs/ci.md`](docs/ci.md)):

```bash
bun run typecheck        # every TS workspace (= `bun run ci typecheck`, a bounded pool)
bun run test             # vitest (two projects: web + native)
bun run check            # biome format + lint  (check:fix to write)
bun run ci rust clippy   # server workspace + every module workspace
bun run ci rust test     # the same, under coverage (cargo llvm-cov)
```

`bun run ci <command>` is `packages/ci-tools`, the CLI every workflow step
runs; `bun run ci lanes --json` tells you which jobs a branch will trigger.
The Rust commands walk the module workspaces too: modules are separate cargo
workspaces, so `--workspace` from `server/` does not reach them (see below).
`cargo clippy --workspace --all-targets` and `cargo test --workspace` in
`server/` are still the quick local loop for the server alone.

`bun run modules:check` (manifests valid + generated output in sync) and
`bun run deadcode` (knip) are **not** wired into any workflow today. Run them by
hand after touching a module or a generator.

Rust is pinned by `rust-toolchain.toml` (1.96.1, with clippy + rustfmt); the
workspace `rust-version` floor is 1.88. `cargo fmt --check` is non-blocking: the
codebase uses a custom import grouping, so do not reformat files wholesale.

## Running things

```bash
bun install
bun run dev              # server:watch (:4040) + web (:3000) + tizen shell (:5174)
bun run dev:webonly      # server + web only
bun run dev:web          # web alone (Vite proxies /api -> :4040)
bun run dev:tizen        # :5174   Samsung   (arrow keys + Enter act as the remote)
bun run dev:webos        # :5175   LG
bun run dev:kit          # design-system workbench
bun run server           # cargo run  (no feature flags: the ML backends are the
                         # whisper and vector sidecars' own build choice)
```

With no media configured the server seeds demo titles. Point it at real files with
`KROMA_MEDIA_DIRS=/path/to/media`. Full env-var table in
[`server/README.md`](server/README.md); `RUST_LOG=debug` for logs.

Every root script is `<verb>:<target>` (`dev:` / `build:` / `deploy:` / `kit:`);
`bun run` with no argument lists them all. Anything targeting one workspace is
`bun run --filter '@kroma/<name>' <script>`.

### Running a single test

`bun run test` is `vitest run` over the whole repo. To narrow:

```bash
bun run test packages/core/src/hevc.test.ts        # one file
bun run test --project web -t 'rejects a range'    # one test name, one project
cd server && cargo test -p kroma-scene             # one Rust crate
cd server && cargo test --workspace parse_episode  # one Rust test filter
```

There are **two vitest projects** because the repo has two module-resolution
universes: `web` (`.web.*` files win, mirrors the shells' Vite config) and `native`
(Metro precedence, plain file wins). A test that must run under Metro resolution is
named `*.native.test.ts`; the include globs are derived from the web list so the two
cannot drift. Default environment is `node`; a test needing a DOM opts in with
`// @vitest-environment jsdom`.

## Architecture

### Server (Rust): layered, compiler-enforced

`server/` is a cargo workspace. The layers are crates, so the inward-only dependency
rule is enforced by the compiler, not by a CI grep.

```
server/
  src/            kroma-server BINARY: main.rs + api/ (router + handlers only)
  crates/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maxscharwath/kroma](https://github.com/maxscharwath/kroma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
