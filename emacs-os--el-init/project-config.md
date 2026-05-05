---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

elinit is an Emacs Lisp package for managing background processes. It provides staged startup with dependency ordering, crash recovery, and a dashboard UI.

## Module Structure

The package is split into focused modules:

| File | Purpose |
|------|---------|
| `elinit-core.el` | Engine, parsing, scheduling, process lifecycle, state management |
| `elinit-log.el` | Log framing, encode/decode, record filtering/formatting, writer lifecycle |
| `elinit-overrides.el` | Overrides persistence, effective state getters, policy mutators |
| `elinit-libexec.el` | Libexec build targets, compiler selection, build invocation |
| `elinit-sandbox.el` | Sandbox profile argument construction and bwrap argv wrapping |
| `elinit-units.el` | Unit-file loading, validation, merge with legacy config |
| `elinit-timer.el` | Timer subsystem for scheduled and calendar-based triggers |
| `elinit-pid1.el` | PID1 integration: auto-detection, boot/shutdown policy, hook wiring |
| `elinit-dashboard.el` | UI rendering, keymaps, interactive commands |
| `elinit-cli.el` | CLI dispatcher, formatters, command handlers |
| `elinit.el` | Entry point that loads all modules and provides the `elinit` feature |

**Load order:** core (requires log, overrides, libexec, sandbox) then units, timer, pid1, dashboard, cli (loaded by entry point).

**Dependency rules:**
- `elinit-core.el` requires `elinit-log`, `elinit-overrides`, `elinit-libexec`, and `elinit-sandbox` (these are extracted subsystems, not optional)
- `elinit-log.el` uses `declare-function` for core's `elinit--log` (no hard require back to core)
- `elinit-overrides.el` uses `declare-function` for core entry accessors and helpers (no hard require back to core)
- `elinit-libexec.el` uses `declare-function` for core's `elinit--log` (no hard require back to core)
- `elinit-sandbox.el` uses `declare-function` for core entry accessors (no hard require back to core)
- `elinit-units.el` uses `declare-function` for core (no hard require)
- `elinit-pid1.el` uses `declare-function` for core (no hard require)
- `elinit-dashboard.el` requires only `elinit-core`
- `elinit-cli.el` requires only `elinit-core`
- `elinit.el` requires all modules

Cross-module calls use `declare-function` for proper byte-compilation. The extracted subsystem modules (log, overrides, libexec, sandbox) use `defvar` forward declarations for variables defined in core.

## Emacs Lisp Standards (MANDATORY)

**CRITICAL:** All code must adhere to STANDARDS in this repository. This document contains:
- GNU Coding Standards for Emacs Lisp
- Emacs Lisp coding conventions from the official manual
- Naming conventions, predicate suffixes, library loading rules
- Minor/major mode conventions
- Documentation and commenting standards

This is a **hard requirement** for all Elisp projects. Read before writing or modifying any code.

## Git Workflow (MANDATORY)

Use only basic git commands. **Never** use `git commit --amend`, `git rebase`,
`git reset`, or any history-rewriting commands. Every commit is a new commit.

```bash
git add <specific-files>
git commit -m "message"
git push
```

Rules:
- **No amending**: if a commit needs a fix, make a new commit on top.
- **No resets**: if something went wrong, fix forward with a new commit.
- **No rebasing**: keep a linear append-only history.
- **Always push** after committing to keep local and remote in sync.
- **Always `make check`** before committing.

## CI Badge Infrastructure

The README test-count badge is powered by a GitHub Gist + shields.io endpoint.

- **Gist:** `e686727a6d88c17c557003e73a9c020c` (secret gist, user `el-sloppo`)
- **Filename inside gist:** `elinit-tests.json`
- **Secret:** `GIST_TOKEN` in repo Actions secrets -- a classic GitHub token with `gist` scope
- **Updated by:** `.github/workflows/ci.yml`, `update-badge` job, on push to master

The badge aggregates all three test suites (Elisp ERT, C acutest, shell).
Each job uploads its counts as artifacts.  The `update-badge` job downloads
them, sums totals and failures, and PATCHes the Gist via the GitHub API.
The badge URL in README.org reads from that Gist via
`https://img.shields.io/endpoint?url=...`.

If the token expires, generate a new classic token with `gist` scope at
https://github.com/settings/tokens and update the `GIST_TOKEN` secret in
repo Settings -> Secrets and variables -> Actions.

## Development Commands

```bash
make check          # Run all CI checks (Elisp + C + shell)
make lint           # Run byte-compile, checkdoc, package-lint only
make test           # Run ERT tests only
make test-one TEST=elinit-test-parse-string-entry  # Run single test
make libexec-check  # Build and test C helpers (elinit-logd, elinit-runas, elinit-rlimits)
make sbin-check     # Run shellcheck + shell tests for sbin/ scripts

# Subsystem checks in isolation
make -C libexec check
make -C sbin check          # shellcheck lint + all shell tests
make -C sbin test           # shell tests only (skip shellcheck)
make -C sbin lint           # shellcheck only

# Load and test interactively
emacs -Q -l elinit.el
```

## Emacs as a Reference Tool


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emacs-os/el-init](https://github.com/emacs-os/el-init) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
