---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

kstack is a **skill pack** (not an app) distributed to Claude Code and other agent CLIs. The shipped artifacts are `SKILL.md` files plus a handful of helper shell scripts in `src/bin/`. There is no runtime service — everything is POSIX shell rendered/executed at install time or inside an agent session.

Shipped shell (`scripts/install`, everything under `src/bin/` and `src/lib/`, plus any skill `scripts/main`) must run on **bash 3.2+** — that's what macOS's `/usr/bin/bash` is, and we don't assume users have a newer bash on PATH. Concretely: no `declare -A`, no `${var,,}`/`${var^^}`, no `mapfile`/`readarray`, no `[[ =~ ]]` BASH_REMATCH patterns that rely on 4.x fixes, and no `&>` redirection. The big footgun is `set -u` + empty arrays: `"${arr[@]}"` raises *unbound variable* in 3.2 when `arr=()`. Either guard iteration with `[ ${#arr[@]} -eq 0 ] && continue`, or use `${arr[@]+"${arr[@]}"}` at the expansion site. Dev-only scripts (`scripts/*.sh`, `tests/**`) may assume a newer bash since they only run on CI/contributor machines.

## Layout

Split is: **`src/` = installer payload (what gets copied/rendered into an install root); everything else = dev infra.**

- `Makefile` — contributor dev entrypoint. Thin facade over `scripts/` (`make install`, `make test`, `make lint`, etc.).
- `src/{bin,lib,skills,schemas}/` — the payload: sources copied verbatim (`bin`, `lib`, `schemas`) or template-rendered (`skills`) into an install root.
- `scripts/install` — the installer. Reads from `src/` and writes outputs (`.kstack/`, `.<agent>/skills/`) at the repo root in dev mode.
- `scripts/bootstrap.sh` — the hosted `curl | bash` getter. Clones an upstream checkout, then execs `$UPSTREAM/scripts/install`.
- `scripts/{lint,test,test-e2e,test-evals,clean}.sh` — repo dev scripts.
- `tests/{unit,integration,e2e,evals,fixtures}/` — bats + eval harness. Dev-only; not part of the payload.
- `CLAUDE.md`, `TODO.md`, `README.md`, `LICENSE`, `assets/`, `.github/` — repo metadata.

All commands and paths below are relative to the repo root.

## Commands

- `./scripts/lint.sh` — run shellcheck across every linted shell file (severity=warning). Requires `shellcheck` (`brew install shellcheck` / `apt install shellcheck`). CI calls this script directly.
- `./scripts/test.sh` — run the fast bats tiers (`tests/unit` + `tests/integration`). Requires `bats-core` (`brew install bats-core` / `apt install bats`). Pass `--all` to also run the e2e tier.
- `./scripts/test-e2e.sh` — run the cluster-backed tier against a kind cluster named `kstack-test`. The kind lifecycle lives in `tests/e2e/lib/kind-cluster.sh` and is shared with the eval tier; the bats suite hook `tests/e2e/setup_suite.bash` is a thin wrapper around it. No prior `kind` state is required. Set `KSTACK_REUSE_CLUSTER=1` during dev loops to keep the cluster alive across runs. Requires `kind`, `kubectl`, and a running Docker daemon.
- `./scripts/test-evals.sh` — run the eval tier: plants fixtures in the kind cluster, invokes skills via `claude -p`, and scores the responses. Requires `ANTHROPIC_API_KEY`, `claude`, `jq`, and `yq` in addition to the e2e prerequisites. Exits 0 with a skip message when `ANTHROPIC_API_KEY` is unset. Env: `KSTACK_EVAL_MAX_RUNS` (override samples per scenario), `KSTACK_EVAL_BUDGET_USD` (hard cost cap). Flags: `--scenario <id>` to run one, `--include-placeholder` to run the smoke scenario.
- `bats tests/unit/<file>.bats` — run a single test file. Use `bats -f "<name pattern>" …` to run one test.
- `make install` (or `./scripts/install`) — dev mode. Renders skills into `<repo>/.<agent>/skills/<name>/…` for every agent CLI detected on `PATH`, reading sources from `src/` and writing outputs at the repo root. Slot names are bare by default; pass `--prefix=<p>` to render them as `<p><name>/`.
- `./scripts/install --local` — user-facing local install. Clones/updates `$PWD/.kstack/upstream/` at the latest release tag and renders into `$PWD/.<agent>/skills/<name>/…`. Normally driven by the hosted bootstrap (`curl … | bash -s -- --local`); the invoker's checkout is never used as the source.
- `./scripts/install --global` — clone/update `~/.config/kstack/upstream/` at the latest release tag and render into `~/.<agent>/skills/<name>/…`. Same canonical-upstream rule as `--local`.
- `make clean` (or `./scripts/clean.sh`) — remove gitignored dev-mode artifacts (`.claude/`, `.codex/`, `.kstack/`, etc.) so `make install` runs against a clean tree.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kubetail-org/kstack](https://github.com/kubetail-org/kstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
