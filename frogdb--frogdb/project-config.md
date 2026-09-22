---
trigger: always_on
description: FrogDB is a modern, Redis 8.x-compatible database written in Rust. It supports both standalone
---

# FrogDB

FrogDB is a modern, Redis 8.x-compatible database written in Rust. It supports both standalone
and cluster operating modes as well as replication and configurable durability/persistence.

> **Note**: `AGENTS.md` is a symlink to this file — they are the same document.

## Goals

- Correctness
  - Specified behavior proven under various failure modes
- Redis compatible (with differences documented)
  - Deviations should be improvements
- Easy to operate
  - easy introspection and observability
  - easy to adjust configuration without downtime
- Fast
  - Should be at least as fast as competing solutions
- Scalable
  - can operate as a single node with no disk, to cluster of nodes and replicas with persistence

## Development Philosophy

- FrogDB is unreleased, pre-production software. Breaking changes are acceptable — sweeping changes
  that would normally be prohibitive for production software are encouraged here when they improve
  implementation efficiency.
- Inspiration is drawn from high-quality modern database projects like CockroachDB, ScyllaDB, FoundationDB
- Upmost care should be taken to ensure the correctness of the system. Examples include:
  - Extensive regression tests derived from the official Redis test suite to ensure compatibility
  - Extensive distributed systems and concurrency testing to ensure expected behavior during various
    failure modes like network partitions, disk failures, etc.
  - Fuzz testing for security/stability
- Easy to operate in a modern cloud environment, eg:
  - Grafana/Prometheus/OpenTelemetry/dtrace for observability
  - frogctl cli tool
  - Debug web pages
  - operational debug/profiling tools
  - kubernetes operator

## Main Components

- FrogDB
  - The database binary
- frogctl
  - cli tool for managing the database (ops)
- frogdb-operator
  - a Kubernetes operator for FrogDB
- website for info/documentation/marketing
- the assets/ folder has images for branding
- .scratch/roadmap/ contains roadmap and unfinished/follow-up items

## Build System

This project uses `just` (see `Justfile`) for performing almost all tasks required in the
development lifecycle: tests (unit, concurrency, web, fuzzing, jepsen, browser, load/memtier,
regression/compatibility), linting, type checking, building (incl. cross-compilation), formatting,
benchmarking, profiling, docker, debug server, website, code generation (docs/markdown, helm,
grafana, debian), github runner, and cleanup/disk space.

Examples:

```bash
just check                              # type-check the workspace
just check frogdb-core                  # type-check a single crate
just test                               # run all tests
just test frogdb-server                 # run all tests for a specific crate
just test frogdb-server test_publish    # run tests matching a regex pattern
just lint                               # clippy on the workspace
just lint frogdb-persistence            # clippy on a specific crate
just lint-py                            # ruff check
just fmt                                # format Rust code
just fmt frogdb-core                    # format a single crate
just fmt-py                             # format Python code
```

**IMPORTANT**: Check the `Justfile` for a recipe before using custom commands like `cargo` directly.

- **BAD**: `cargo test ...`
- **GOOD**: `just test ...`

- When running a single test, target the owning crate to avoid rebuilding the entire workspace:
  `just test frogdb-server test_name`
- Never call `cargo` directly, not even for a one-off: the Justfile's `_cargo` recipe (and
  `scripts/cargo_env.py` for Python) carry the RocksDB/libclang env — without it the first build
  in a worktree compiles vendored RocksDB from source (~10 min, 1.5 GB). A worktree's missing
  `target/` is cloned from the clean-main seed before its first `just check/build/test/lint`
  (`just seed-target` does it by hand; `just seed-refresh` in the main checkout rebuilds the
  seed); `just worktree-prune` lists merged worktrees to remove. Background:
  `.scratch/build-cache/README.md`.

### Execution mode: local (default) or testbox

Builds and tests run in one of two modes. **`local` is the default.**

- **Settle the mode before the first build/test/lint/bench command of a session**, including a
  resumed one. If the prompt names a mode ("local mode", "use the testbox"), use it. Otherwise
  ask the user — never guess, and never switch mid-session on your own.
- The mode is recorded per-worktree: `just build-mode` prints it, `just build-mode testbox`
  sets it. A SessionStart hook injects the recorded value so it survives resumes; confirm it
  anyway. Record the answer so later sessions and the tb-* guard agree.
- Subagents inherit the session's mode — state it explicitly in every dispatch prompt.
- The `tb-*` recipes refuse to run in local mode (`BUILD_MODE=testbox` overrides for a one-off).

**Local mode** — everything runs on this machine, testbox untouched: full-workspace builds,
whole-suite `just test`, `just lint`, concurrency/turmoil suites, benchmarks. Heavy runs follow
the liveness rule below. Say so if a run is slow; do not reach for a testbox to fix it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [frogdb/frogdb](https://github.com/frogdb/frogdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
