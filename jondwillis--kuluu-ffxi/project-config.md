---
trigger: always_on
description: Guidance for AI coding agents working in this repository. `CLAUDE.md` is a symlink to this file, so Claude Code and any AGENTS.md-aware tool read the same instructions.
---

# AGENTS.md

Guidance for AI coding agents working in this repository. `CLAUDE.md` is a symlink to this file, so Claude Code and any AGENTS.md-aware tool read the same instructions.

## What this is

Kuluu is a faithful, open-source FINAL FANTASY XI **client** rebuilt in Rust + Bevy. It speaks the FFXI wire protocol to community-run private servers (LandSandBoat / Phoenix), **not** retail. It is **not a server**, and it **ships no game assets** — geometry/textures/audio/animation come from a user-provided retail install read at runtime from `FFXI_DAT_PATH` (default `vendor/game-files/SquareEnix/FINAL FANTASY XI`). Tables derived from LSB/POLUtils are baked in as compile-time constants, never as game content. The single source of truth for all durable work — including the grounded parity backlog — is beads (`bd ready`, label `roadmap`; see the Issue tracking section).

## Build, test, lint

`scripts/checks.sh` is the **single source of truth** for check commands — both the `pre-push` hook and CI call it, so they can't drift. Prefer it over spelling out cargo flags:

```bash
scripts/checks.sh fmt clippy            # what the pre-push hook runs
scripts/checks.sh fmt clippy test build # the full CI gate
cargo fmt --all                         # autofix formatting
```

`checks.sh` owns the full workspace invocations; don't restate them here. Everything compiles under one feature set, `--features native-window` (the default for `ffxi-client`/viewer) — match it for ad-hoc cargo runs so artifacts are reused across stages, e.g. a single test:

```bash
cargo test -p ffxi-proto framing::tests::roundtrip --features native-window
```

- **Nightly is required.** `rust-toolchain.toml` pins a dated nightly; the dev profile uses the Cranelift codegen backend (gated by `[unstable] codegen-backend` in `.cargo/config.toml`), which makes a *stable* cargo error out. Cranelift is dev-only — `--release` and the Steam Deck cross-build use LLVM.
- **Integration tests that need a live LSB server self-skip** when it's unreachable, so the test stage is safe on a network-isolated machine. Fixtures using `mysql_async` stamp out isolated accounts against a real MariaDB and only run when one is reachable.
- **Enable the hooks once per clone:** `cargo xtask install-hooks` (sets `core.hooksPath=.githooks`). Bypass a push with `git push --no-verify`; `PREPUSH_FAST=1 git push` runs fmt only.
- `xtask` is excluded from `default-members`, so plain `cargo build`/`test` skip it; run it via the `cargo xtask` alias.

## Running

Credentials and the DAT path come from env vars (never committed/logged). The launcher prompts for any unset credential and lists characters by name.

```bash
export FFXI_USER=... FFXI_PASS=... FFXI_CHAR="Exact Name" FFXI_SERVER=127.0.0.1
export FFXI_DAT_PATH="/path/to/SquareEnix/FINAL FANTASY XI"   # or: cargo xtask game

cargo run -p ffxi-client -- play                          # native window (default)
cargo run -p ffxi-client --no-default-features -- play --headless  # JSON event-stream agent session, no Bevy
```

`cargo xtask game [path|--copy|--download]` detects/validates/symlinks a retail install into `vendor/game-files/`. Client subcommands: `play`, `model-viewer`, `provision`, `create-char`.

## Issue tracking (beads)

Durable work items live in [beads](https://github.com/steveyegge/beads) (`bd`), a git-backed tracker checked into `.beads/`. Install with `brew install beads`. The dependency graph is a local Dolt db (gitignored); `.beads/issues.jsonl` is the diffable, PR-reviewable export — that's the file that crosses into git, so review it like code.

- `bd ready` — actionable work (open, unblocked). `bd list --status=in_progress` — active work.
- `bd create --title=… --description=… --type=feature --priority=2` — new issue. `bd update <id> --claim`, `bd close <id>`, `bd show <id>`.
- Priorities are `0`–`4` (0 = critical), **not** high/medium/low. Don't run `bd edit` (opens `$EDITOR`, blocks agents).

Beads is the **single source of truth for all durable work** (there is no `docs/ROADMAP.md` — that scoreboard was removed as a redundant hand-kept projection of beads). The grounded parity backlog is the `roadmap`-labelled beads, each citing `file:line` evidence and carrying `vanilla`/`enhanced` plus an area label (`hud`, `combat-action`, …). Pick work with `bd ready`. MEMORY.md auto-memory sits alongside beads and is **not** replaced by it — do **not** migrate it into `bd remember`.

**Commit authority (repository-profile grant).** The beads `bd prime` session protocol defaults to *conservative* — no commits without granted authority. This repository **grants standing authority to commit liberally**: group finished, uncontroversial work into clear, coherent commits as you go, without stopping to ask. This is the sanctioned override of the conservative default. Still **confirm before `git push`** (outward-facing) and before `bd dolt push` / remote sync, and never force-push or rewrite shared history. In a tree that mixes another session's edits, stage only your own hunks (`git add -p`), never `-A`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jondwillis/kuluu-ffxi](https://github.com/jondwillis/kuluu-ffxi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
