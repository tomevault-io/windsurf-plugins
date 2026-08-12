---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

zccache is a local-first compiler cache (21 crates) for C/C++/Rust/Emscripten, inspired by sccache but optimized for warm-hit latency. Architecture: a persistent `zccache-daemon` holds an in-memory metadata cache and a filesystem watcher; the `zccache` CLI (binary in `zccache-cli`) shells out per compile but talks to the daemon over a single length-prefixed bincode IPC roundtrip (Unix sockets / Windows named pipes via `zccache-ipc`). The daemon is lazily started by the CLI when not running. See @docs/CLAUDE.md for which architecture doc to read based on what you're working on, and where to document new features.

> [!IMPORTANT]
> ## Performance work → read [PERF.md](PERF.md) FIRST
>
> **The local Linux Docker matrix in `ci/perf_local.py --matrix` is the sanctioned path for zccache perf work.**
> If you are testing, measuring, optimizing, or regressing zccache's performance —
> read **[PERF.md](PERF.md)** before doing anything else.
>
> GitHub Actions does not run the large wall-clock fixture matrix. Use narrow
> local cells while iterating, then run all eight cells with `--matrix` before merge.
>
> Do not invent ad-hoc benchmarks (`criterion`, `divan`, `hyperfine` in a one-off
> script). The local Docker matrix is the regression-blocking measurement; everything else
> is diagnostic.
>
> **When iterating on a perf problem: reproduce in local Docker first.** Retain
> `.perf-local/results/<fixture>/<scenario>/` as evidence and never use Actions
> as the timing feedback loop.
>
> **Every perf fix lands with a perf unit test.** Without a test, the bug comes
> back. Either extend `crates/zccache-daemon/tests/perf_bench_test.rs` + add a
> threshold row in `ci/perf_guard.py`, or add a `#[test]` `Duration` budget
> assertion in the crate where the regression lived. See PERF.md →
> "Preventing regressions — add a perf unit test."

## Essential Rules

- **Always use `soldr <tool>` directly** to execute Rust commands. Bare cargo/rustc, legacy root trampolines, and `uv run cargo` are blocked by hook. soldr resolves repo-local `.cargo` / `.rustup` homes and the rustup-managed toolchain pinned by `rust-toolchain.toml`.
- **Always use `uv` for Python.** Bare `python`/`pip` are blocked by hook. Use `uv run ...` or `uv pip ...`.
- MSRV: 1.94.1 | Edition: 2021 | Toolchain: 1.94.1 (clippy + rustfmt)
- CI: Linux, macOS, Windows. All warnings denied (`RUSTFLAGS="-D warnings"`)
- Every directory with files must have a README.md (enforced by hook)

## Commands

```bash
./test                      # unit tests only (fast, no compiler needed)
./test --integration        # integration tests only (need clang on PATH)
./test --full               # unit + integration + stress + perf tests
./test -p <crate> -- <test_name>
soldr cargo check --workspace --all-targets
soldr cargo clippy --workspace --all-targets -- -D warnings
soldr cargo fmt --all
RUSTDOCFLAGS="-D warnings" soldr cargo doc --workspace --no-deps
soldr cargo bench -p zccache-hash
./perf.sh                   # performance benchmark (zccache vs sccache vs bare clang)
```

See [PERF.md](PERF.md) for the scenario-driven local Docker gate (cold-tar-untar-warm and friends).

## Distribution

Native binaries are built via GitHub Actions and downloaded locally for packaging. PyPI is the distribution channel - no Python in the runtime hot path.

```bash
# Build all platforms (triggers GH Actions, waits, downloads to dist/)
uv run python ci/build_dist.py --ref main

# Download from a specific run
uv run python ci/build_dist.py --run-id <run_id>

# Re-download latest successful build (no new build)
uv run python ci/build_dist.py --skip-build
```

- **Workflow**: `.github/workflows/build.yml` (workflow_dispatch, 8 targets)
- **Script**: `ci/build_dist.py` - orchestrates `gh` CLI to trigger, wait, download, organize
- **Output**: `dist/` with per-platform subdirs + `manifest.json` (gitignored)
- **Targets**: linux-x86_64, linux-aarch64, macos-x86_64, macos-aarch64, windows-x86_64, windows-arm64

### Publishing

- **Automation**: `.github/workflows/release-auto.yml` is the only supported release entrypoint. It validates release metadata, fails fast when the current version is already fully published on PyPI/crates.io, builds wheel/release artifacts, publishes PyPI wheels, publishes Rust crates, and creates the GitHub release.
- **Helper module**: `ci/release_workflow.py` contains workflow-only Python helpers for preflight checks, wheel assembly, and crates.io publish order. It does not dispatch other GitHub workflows.
- **Three trigger paths**, all converge on the same publish pipeline:
  1. **Auto on push-to-main** (the everyday path): the `detect-bump` job reads `[workspace.package].version` from `Cargo.toml`, compares it to the prior commit's version, and proceeds iff the version was bumped. Merge a `chore(release): bump … -> X.Y.Z` PR and the release ships on its own.
  2. **Tag push**: push `1.3.0` or `v1.3.0`; the workflow normalizes the tag and requires it to match `[workspace.package].version` in `Cargo.toml`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zackees/zccache](https://github.com/zackees/zccache) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
