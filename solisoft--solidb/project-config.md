---
trigger: always_on
description: handles are now kept and joined from `StorageEngine`'s `Drop`, by whichever
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SoliDB is a lightweight, high-performance multi-document database written in Rust. It features a custom query language (SDBQL), multi-node replication, sharding, ACID transactions, Lua scripting, and WebSocket-based real-time subscriptions.

## Build & Development Commands

```bash
# Build
cargo build                    # Debug build
cargo build --release          # Release build

# Run server
./target/release/solidb --port 6745 --data-dir ./data

# Testing — use --profile ci, not --release (see below)
cargo test --profile ci --test <name>         # Specific test file (e.g., cargo test --profile ci --test http_api_test)
cargo test --profile ci <pattern>             # Tests matching pattern (e.g., cargo test --profile ci sdbql)
cargo test --profile ci -- --nocapture        # Show test output
cargo test --profile ci                       # FULL suite — CI's job, not yours. See below.

# The five benchmarks that live in tests/ are behind a feature and #[ignore]d
cargo test --profile ci --features bench-tests -- --ignored

# Code quality (required before commits)
cargo fmt -- --check           # Check formatting
cargo clippy -- -D warnings    # Lint checks
```

### Test with `--profile ci`, not `--release`

`[profile.ci]` (in `Cargo.toml`) is what CI runs and what you should run. It
inherits `release`, so dependencies — RocksDB above all — stay at `opt-level 3`
and the suite still executes in seconds. What it drops is thin-LTO and
release-level codegen for the workspace crates, which is what the ~96 separate
test binaries were each paying for.

That mattered more than it sounds: CI measured **95m 07s of compilation against
18.4s of test execution**. Do not "fix" this by relaxing `[profile.release]` —
the `build-binaries` and `docker` jobs ship what it produces.

Consequence to know about: `--profile ci` writes to `target/ci/`, *alongside*
`target/release/`. Building both doubles that part of your disk.

### Do not run the full suite locally

`cargo test --profile ci` with no filter builds and runs ~96 test binaries.
Compiling them takes tens of minutes even on a warm cache, and each integration
test opens its own RocksDB instance, so a dev box that is also running the
server and a fleet of app dev servers goes to swap. Runs get killed part-way and
tell you nothing.

Run the targeted forms above — the specific `--test <name>`, or a pattern — and
let the `test` job in CI run the whole thing. The other six CI jobs (`fmt`,
`clippy`, `docs-sync`, `audit`, `msrv`, `windows-check`) are all cheap enough to
reproduce locally and are the ones worth checking before a push.

### Known local-only failures

There are none right now.

- **`queue::jobs::tests::validate_target_accepts_webhook_only`** used to fail on
  any machine whose resolver wildcards `.test` to `127.0.0.1` (a common local
  dev setup): the SSRF guard resolved `example.test`, saw loopback, and
  rejected it. **Fixed** — the fixture is a literal public IP, so the test does
  no DNS at all. This became load-bearing when the guard stopped treating an
  unresolvable host as allowed: `example.test` would now fail in CI too, where
  the name does not resolve.

The entry that used to live here — **`rbac_admin_endpoints_tests`
aborting at process *exit*** with `SIGSEGV` or `std::bad_alloc`/`SIGABRT` after
all its tests reported ok — is **fixed**. `PendingCfDrops::spawn_dropper`
(`src/storage/pending_drops.rs`) detached its thread and nothing joined it, so a
column-family drop could still be inside RocksDB's `PersistRocksDBOptions` while
the main thread's static destructors freed the global option-type registry. The
handles are now kept and joined from `StorageEngine`'s `Drop`, by whichever
clone is the last one alive. Measured before and after on the same binaries,
60 runs each: `db_authorization_tests` 5/60 → 0/60, `rbac_admin_endpoints_tests`
→ 0/60. The server's shutdown path went through the same `Drop`, so it is fixed
there too.

## Releasing

When bumping the version, update all three of these in the same commit:

1. `version` in `Cargo.toml` (and `Cargo.lock` — `cargo update -p solidb --offline`).
2. The version pill in `doc/app/views/home/index.html.slv`
   (`<span class="ver-pill">vX.Y.Z</span>`).
3. **A section for the new version in `doc/app/views/docs/changelog.html.slv`**,
   and move anything under *Unreleased* into it.

Steps 2 and 3 are enforced by `scripts/check_docs_sync.sh`, which fails when
the docs site does not describe the version in `Cargo.toml`. It runs as the
`docs-sync` CI job (which gates `release`) and at the top of
`scripts/release.sh`, so a stale docs site stops the release before a tag or a
crates.io publish exists.

Step 3 is the one that used to get forgotten: the docs changelog is
hand-written and duplicates `CHANGELOG.md` (which release-please generates from
conventional commits). Between v0.31.0 and v0.32.2 the page was never touched,
so the docs site advertised a release three versions behind the shipped binary.
Copy the `CHANGELOG.md` entry across, or write the section directly if the
release was tagged by hand.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [solisoft/solidb](https://github.com/solisoft/solidb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
