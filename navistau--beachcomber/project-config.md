---
trigger: always_on
description: Project-specific instructions for contributors using Claude Code.
---

# CLAUDE.md

Project-specific instructions for contributors using Claude Code.

## Read first

Before proposing changes or reasoning about behavior, read these. They are the authoritative project state — code alone will mislead you.

- **`docs/roadmap.md`** — what's built, what's deferred, what's broken. Contains the "Known Core Issues" section; check it before claiming anything is working. If behavior seems odd, look here first.
- **`docs/canon/*`** — canonical spec major architechtural designs. Tests must match this document; code that disagrees is wrong.
- **`docs/canon/provider_source.md`** — canonical Provider/Source/Field model. The authoritative spec for how providers declare sources, invalidation strategies, lifecycle keying, cache layout, and TOML config. Read this before touching any provider, scheduler, cache, or config code.
- **`CHANGELOG.md`** — what shipped when, at what version.

@docs/canon/ are 'canonical specs' in that they are long lived and only ever edited with explicit permission. Canon
specs define test suite tdd, which defines the code, which is then communicated by the documentation/website. This is
how we resolve 'are the docs or code correct?', its a strict line of truth.

If a section of the code looks like a half-wired state machine or a config key that does nothing, check `docs/roadmap.md` → "Known Core Issues" before writing a fix. Some scaffolding is known-aspirational and has a planned rebuild.

## Basics

- **Project:** beachcomber — a daemon that caches shell environment state
- **Binary:** `comb` (not `beachcomber`)
- **Crate:** `beachcomber` (workspace root) + `beachcomber-client` (client library)
- **Rust version:** pinned in `mise.toml` — use `mise install` to get it
- **Platform:** macOS and Linux

## Branch workflow

- **`develop` is the default branch and the integration target.** All feature/fix work branches off `develop` and merges back into `develop` via PR. Routine commits land on `develop` (or short-lived branches off it). Never commit directly to `main`.
- **`main` is the release branch.** It only ever advances through a PR from `develop` → `main`. That PR is the **release gate**: branch protection on `main` requires all CI checks green before it can merge (no direct pushes, no force-pushes).
- **Releasing** = bump the version on `develop` (`cargo xtask set-version`), open a `develop` → `main` PR, wait for CI green, and merge. **Merging is the release:** `release.yml` triggers on push to `main`, reads the version from `Cargo.toml`, tags `vX.Y.Z` on the merge commit, and builds + publishes to all registries. No manual tagging. Full steps in `docs/releasing.md`.
- **CI** (`.github/workflows/ci.yml`) runs on pushes to and PRs targeting both `develop` and `main`.
- Delete branches and worktrees after merge — keep the branch list to `main` + `develop` plus whatever is actively in flight.
- `git push` still requires explicit per-instance confirmation (see the global rules); committing to `develop` does not.

## Build and Test

```sh
cargo build              # debug build
cargo nextest run        # run tests (mandatory, cargo test is designed to instantly fail)
cargo t                  # shorthand alias for `cargo nextest run`
cargo bench              # criterion benchmarks (cache, protocol, providers, socket, throughput)
cargo clippy -- -D warnings
cargo fmt -- --check
```

**Test runner:** `cargo-nextest` is the blessed runner. Run `mise install` to get it alongside Rust — it is declared in `mise.toml`. Use `cargo nextest run` or the shorter `cargo t` alias we ship. Plain `cargo test` triggers an advisory that fires immediately on binary startup (via `ctor`) before any test runs — it prints an instructive message and exits with code 2. Set `NEXTEST=1` in the environment to bypass it intentionally.

Tests that are environment-sensitive:
- `uptime_provider_executes` — needs unsandboxed environment

CI skips with: `cargo nextest run -E 'not test(uptime_provider_executes)'`.

**Watcher tests** use `FsWatcher::new_polling` (Phase 13 stabilization), so they run reliably under sandboxed hosts that can't use FSEvents / inotify. Production code still uses `FsWatcher::new` which picks the kernel-native backend.

Full test suites, formatting, builds, etc, should _always_ execute fully in under 30s. If they take longer the command
should fail and it should trigger investigation.

## Architecture

Async tokio daemon listening on a Unix socket. One task per connection.

- `src/scheduler/` — core loop: poll timers, filesystem watching (notify), demand tracking, cache lifecycle (Active / Decay1-4 / Evicted), provider execution. See `src/scheduler/lifecycle.rs` for the state machine.
- `src/server.rs` — socket accept loop, spawns connection tasks
- `src/cache.rs` — DashMap with null-byte-separated keys (`"provider\0path"`)
- `src/protocol.rs` — NDJSON wire format, `Request`/`Response` types
- `src/config.rs` — TOML config loading, XDG paths, env file parsing
- `src/provider/` — all providers implement the `Provider` trait (19 built-in)
- `src/provider/registry.rs` — registers built-in + config-defined providers
- `src/provider/library.rs` — shared library provider backend via `libloading`; loads `.so`/`.dylib` with C ABI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NavistAu/beachcomber](https://github.com/NavistAu/beachcomber) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
