---
trigger: always_on
description: This file is read by AI tools that support the agents.md vendor-neutral
---

# AGENTS.md

<!--
  This file is read by AI tools that support the agents.md vendor-neutral
  standard. The content below is duplicated in CLAUDE.md (read by Claude
  Code) so that contributors using either receive the same instructions
  without needing to cross-reference. Keep CLAUDE.md and AGENTS.md
  byte-identical below the H1 title (the sync-policy comment above is the
  one place they intentionally differ) — if you edit one, edit the other.
-->

## Required reading

These documents are the source of truth. If anything in this file conflicts with
them, they win. Read them before opening a non-trivial issue or PR.

- [`docs/reference/ipc-contract.md`](docs/reference/ipc-contract.md) — the IPC
  contract the clean-room helper implements: command surface, wire framing,
  message shapes, keycodes (with companion `keycodes.json` / `commands.json`).
  The technical source of truth for helper behaviour.
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — what we accept, what goes upstream to
  Wispr, the bash + Rust style requirements, the AI-attribution policy.
- [`docs/index.md`](docs/index.md) — entry point for the operator/contributor
  docs (building, configuration, troubleshooting, decisions, learnings).
- [`docs/styleguides/bash_styleguide.md`](docs/styleguides/bash_styleguide.md) —
  shell conventions (forked from YSAP). Tabs, 80 cols, `[[ ]]`, no `set -e`.
- [`SECURITY.md`](SECURITY.md) — vulnerability reporting; what's in scope vs.
  what belongs to Wispr.

This file is a fast reference for the highest-leverage rules and the project's
accumulated conventions. New policy goes in the style guides or CONTRIBUTING.md.

## Project overview

This is an **unofficial** Linux port of the proprietary **Wispr Flow**
voice-dictation app (an Electron 42 / electron-forge app shipped as a Squirrel
Windows installer). It is two things:

1. A **repackaging pipeline** (`scripts/`, `build.sh`) that extracts the app,
   patches its bundle for Linux, rebuilds native modules, stages a Linux
   Electron, and produces `.deb` / `.rpm` / AppImage packages.
2. A **clean-room Rust helper** that reimplements the one native capability Wispr
   Flow ships only for macOS (Swift) and Windows (C#): injecting transcribed text
   into the focused application. It is built from the documented IPC contract
   (`docs/reference/`) and contains **no Wispr Flow code**. It now lives
   in its own repo (`wispr-flow-linux/helper`); see the Repo layout below.

## Repo layout

The project spans **two repositories** under the `wispr-flow-linux` org:

- **`wispr-flow-linux/wispr-flow-linux` (this repo)** — the public-domain build
  scripts and the local packaging makers.
- **`wispr-flow-linux/helper`** — the clean-room Rust helper. It was extracted
  from this repo and no longer lives here as a local source tree. The helper is
  consumed as a **prebuilt binary** pinned in `helper-version.txt` and staged
  via the `HELPER_BIN` env var (build-linux.sh resolves it).

> The hosted distribution layer — the `gh-pages` APT/DNF tree, the `v*` tag
> Releases, the gated publish/heartbeat workflows, and a `wispr-flow-linux/worker`
> Cloudflare Worker fronting `pkg.wispr-flow-linux.dev`

This repo's tree:

- `build.sh` — top-level orchestrator: dispatches the staging pipeline and the
  per-format packaging makers (`--build deb|rpm|appimage`, `--clean`,
  `--doctor`).
- `scripts/`
  - `setup/` — host detection, dependency install, Wispr Flow / Electron
    download helpers.
  - `patches/` — the app patches: `helper-resolver.sh` (adds the `'linux'`
    helper-path branch), `mac-gates.sh` (gates the macOS Applications-folder
    guard to darwin), and the V8 14.8 `better-sqlite3-multiple-ciphers` compat
    patch. `verify-patches.sh` static-greps the repacked bundle for the markers.
  - `packaging/` — `deb.sh`, `rpm.sh`, `appimage.sh` makers; shared signature
    `<maker>.sh <dist_dir> <version> <arch>`.
  - `launcher-common.sh` — the runtime `/usr/bin/wispr-flow` launcher library.
  - `doctor.sh` — the `wispr-flow --doctor` diagnostic surface.
  - `build-linux.sh` — the Phase-0 staging pipeline (see safety rules below).
- `helper-version.txt` — the pinned helper release tag fetched from the helper
  repo and staged via `HELPER_BIN`.
- `docs/reference/` — the documented stdin/fd-3 IPC protocol (`ipc-contract.md`
  + `keycodes.json` / `commands.json`).
- `tests/` — bats unit tests, per-format artifact tests, and the manual
  VM-matrix validators.
- `docs/` — building / configuration / troubleshooting / decisions / learnings /
  style guides.
- `nix/`, `flake.nix` — Nix packaging.
- `.github/workflows/` — CI gates (`shellcheck`, `codespell`, `test-flags`,
  `tests`) that run on every push/PR, plus the **tag-driven release/publish
  pipeline** (`ci.yml` build→test→release→APT→DNF→AUR, reusable
  `build-amd64`/`build-arm64`/`test-artifacts`, `check-wispr-version`,
  `apt-repo-heartbeat`, `cleanup-runs`, `update-flake-lock`). The publish chain
  runs on a `v*` tag push; see [`RELEASING.md`](RELEASING.md). The worker lives
  in its own repo (`wispr-flow-linux/worker`).

## Code style

### Bash

All shell scripts follow the
[Bash Style Guide](docs/styleguides/bash_styleguide.md):

- Tabs for indentation, lines under 80 chars (exception: URLs and regex).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wispr-flow-linux/wispr-flow-linux](https://github.com/wispr-flow-linux/wispr-flow-linux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-08 -->
