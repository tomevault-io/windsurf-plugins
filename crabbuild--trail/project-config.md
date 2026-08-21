---
trigger: always_on
description: This file is the repository-level operating guide for AI coding agents working on Trail. It applies to the entire tree unless a more specific `AGENTS.md` exists below the directory being changed.
---

# AGENTS.md

This file is the repository-level operating guide for AI coding agents working on Trail. It applies to the entire tree unless a more specific `AGENTS.md` exists below the directory being changed.

## Start here

Before editing:

1. Run `git status --short` and preserve all user changes. Keep unrelated edits and generated files out of the patch.
2. Read the affected crate's `Cargo.toml`, its `src/lib.rs` or binary entry point, and the nearest unit and integration tests.
3. Find the closest existing implementation and keep the change at that ownership boundary.
4. Read [README.md](README.md) for the product model and [docs/README.md](docs/README.md) for documentation routing.
5. Read the relevant design and reference documents listed below before changing a public contract, storage behavior, lane lifecycle, or integration.
6. Check the matching GitHub Actions workflow before choosing final verification. Continuous integration (CI) is the authority for platform-specific gates.

The root Cargo workspace contains `trail` and `trail-environment-adapter-sdk`. The Agent Client Protocol (ACP) reference peer under `tools/acp-v1-reference-peer` is an independent workspace used by an interoperability script. Supporting directories such as `docs/`, `scripts/`, and `tools/` remain in scope for matching work. Other embedded product source trees are not Trail dependencies and are out of scope unless the task names them.

## Disk and external-checkout policy

The main disk is limited to 100 GB. Rust build artifacts and repositories used for real-repository Trail qualification belong on the mounted workspace volume, not in this checkout or on the main disk.

- Before any Cargo command that can compile (`build`, `check`, `test`, `clippy`, `bench`, `doc`, `install`, `package`, or a Make target that invokes one), set `CARGO_TARGET_DIR` beneath `/Volumes/Workspace/crabbuild-target`.
- Give every repository checkout and worktree its own target directory. Never share one Cargo target directory between repositories or concurrent worktrees because feature sets, build scripts, and locks can collide.
- Use a stable, descriptive directory such as `/Volumes/Workspace/crabbuild-target/trail-main` for the primary checkout and `/Volumes/Workspace/crabbuild-target/trail-<worktree_name>` for another Trail worktree. Include both the repository and checkout or worktree name for another project.
- Set the variable on every shell or tool invocation. Do not assume an export from an earlier command persists. For example:

  ```bash
  CARGO_TARGET_DIR=/Volumes/Workspace/crabbuild-target/trail-main \
    cargo test -p trail --locked
  ```

- Verify that `/Volumes/Workspace` is mounted and the selected directory is writable before a long build. Create only the specific per-checkout directory needed. If the volume is unavailable, stop and report it instead of falling back to a local `target/` directory.
- Do not delete, clean, or reuse another repository's target directory. Run `cargo clean` only with the intended `CARGO_TARGET_DIR` set explicitly and only when the task requires invalidating or reclaiming those artifacts.
- Look for repositories used by real-repository lane, changed-path ledger, and scale qualification under `/Volumes/Workspace/Github` first. When a task requires a missing public repository, clone it under `/Volumes/Workspace/Github/<owner>/<repository>`. Do not clone qualification repositories into the Trail tree, `/tmp`, or the main-disk GitHub directory.
- Treat external qualification repositories as read-only inputs. Do not modify, update, reset, or clean an existing checkout unless the task explicitly requires it. Keep generated `.trail/` state and qualification artifacts outside tracked source, or remove only artifacts created by the current task.

Some Makefile targets consume binaries through a literal local `target/` path after Cargo finishes. Prefer direct Cargo commands with `CARGO_TARGET_DIR` for normal verification. Before using packaging, installation, or release targets, inspect the target and ensure its artifact lookup also points at the selected external directory. Do not allow it to trigger a second local build silently.

## Product model

Trail is a native, local-first operation database for code and text worktrees. Git remains the shared source-control and publication layer. Trail records the high-frequency local work between Git commits: operations, roots, line identity, branches, lanes, sessions, turns, traces, approvals, readiness gates, handoffs, and merges.

The released package and binary are both named `trail`. The binary entry point is `trail/src/main.rs`, and reusable behavior is exposed through `trail/src/lib.rs` and the `Trail` type.

Preserve these product boundaries:

- Trail must remain usable through the command-line interface (CLI) without a daemon. Hypertext Transfer Protocol (HTTP) and Model Context Protocol (MCP) servers are opt-in integration surfaces.
- `.trail/` is private workspace state. `.trailignore` lives at the workspace root. Never record, materialize over, export, or expose internal state accidentally.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crabbuild/trail](https://github.com/crabbuild/trail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
