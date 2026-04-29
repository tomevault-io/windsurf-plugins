---
trigger: always_on
description: This file is for coding agents working in this repository.
---

# AGENTS.md

This file is for coding agents working in this repository.
It summarizes the commands, workflows, and code conventions that are visible in the current codebase.

This repository uses the Shared Context Engineering (SCE) approach for AI-assisted software delivery with explicit, versioned context: `https://sce.crocoder.dev/`

## Repository shape

- Root repo contains three main working areas:
- `cli/` - Rust CLI (`sce`)
- `config/` - generated agent config, skills, and Pkl sources
- `context/` - shared context docs, plans, decisions, and handovers

## Rule files checked

- No root `AGENTS.md` existed before this file.
- No `.cursor/rules/` directory was found.
- No `.cursorrules` file was found.
- No `.github/copilot-instructions.md` file was found.
- If any of those files are added later, update this document to fold their instructions in.

## Tooling and environment

- Nix is the primary reproducible entrypoint at repo root.
- Root `flake.nix` provides Bun, TypeScript, Pkl, jq, and the Rust toolchain.
- Root `flake.nix` defines Crane-based Rust packaging and check derivations for the CLI.
- Run Cargo via Nix, not directly from the host shell. Prefer `nix develop -c sh -c 'cd cli && <cargo command>'`.
- For validation, prefer `nix flake check` and avoid running `cargo test` directly unless a user explicitly requests it.
- Optional local Nix tuning can live in user-level `~/.config/nix/nix.conf`; recommended values are `max-jobs = auto` and `cores = 0`.
- `auto-optimise-store = true` is intentionally treated as a system-level `/etc/nix/nix.conf` setting, not a repo-managed user setting.
- Bun is used for repo-owned config/plugin workflows; prefer Bun rather than npm or pnpm scripts when working in those areas.
- Rust edition is `2021`.
- TypeScript is still used in repo-owned config/plugin sources and should remain strict-mode friendly.

## High-value commands

### Root-level setup

- Enter dev shell: `nix develop`
- Run all flake checks visible at root: `nix flake check`
- Run generated-output parity check: `nix run .#pkl-check-generated`
- Regenerate and sync `.opencode` config: `nix run .#sync-opencode-config`

### Rust CLI commands

Run these through Nix from repo root unless noted otherwise.

- Build CLI: `nix develop -c sh -c 'cd cli && cargo build'`
- Run CLI: `nix develop -c sh -c 'cd cli && cargo run -- --help'`
- Build packaged CLI output: `nix build .#default`
- Run packaged CLI app: `nix run .#sce -- --help`
- Preferred repo-level verification: `nix flake check`
- Run a single Rust test by exact name when explicitly needed: `nix develop -c sh -c 'cd cli && cargo test parser_routes_mcp -- --exact'`
- Run Rust tests in one module/file pattern when explicitly needed: `nix develop -c sh -c 'cd cli && cargo test setup'`
- Run ignored? none were found; do not assume ignored-test flows exist.
- Rust format verification is covered by `nix flake check`
- Auto-format: `nix develop -c sh -c 'cd cli && cargo fmt'`
- Rust lint verification is covered by `nix flake check`

### Bun config/plugin commands

Run these from `config/lib/bash-policy-plugin/` when working on the Bun-owned bash-policy runtime.

- Run plugin/runtime test suite: `bun test`
- Run a single Bun test by name: `bun test -t "<test name>"`

### Useful combined validation flows

- Preferred repo validation from repo root: `nix flake check`
- Config/plugin validation from repo root: `nix develop -c sh -c 'cd config/lib/bash-policy-plugin && bun test'`
- Generated-config validation from repo root: `nix run .#pkl-check-generated`

## Testing notes

- Rust tests live inline in source files and in module test files such as `cli/src/services/setup/tests.rs`.
- Rust/Cargo commands should be executed through `nix develop`, even for one-off builds, tests, fmt, and clippy runs.
- Prefer `nix flake check` for routine verification and avoid `cargo test` unless the user explicitly asks for it.
- Rust single-test selection uses standard Cargo substring matching; add `-- --exact` for deterministic one-test runs.
- Bun tests use `bun:test` and support `-t` name filtering.
- Bun/plugin tests under `config/lib/bash-policy-plugin/` are lighter-weight repo validation and remain part of the flake check surface.

## CI and release hints

- GitHub Actions publish Tessl tiles from `config/.opencode/skills/**` and `config/.claude/skills/**`.
- Release workflow packages agent files from `config/.opencode/agent/Shared Context.md` and `config/.claude/agents/shared-context.md`.
- Root `flake.nix` packages `sce` through Crane's `buildDepsOnly` + `buildPackage` pipeline and runs `cli-tests`, `cli-clippy`, and `cli-fmt` through Crane-backed checks.
- Changes under generated config trees may need a Pkl regeneration or parity check.

## Code style: general

- Follow existing local patterns before introducing new abstractions.
- Keep changes scoped and incremental.
- Prefer deterministic behavior and stable output text; this matters in CLI tests.
- Use explicit constants for repeated strings, timeouts, intervals, exit codes, and numeric formatting.
- Prefer small helper functions when they improve readability of branching or setup code.
- Avoid introducing framework-heavy patterns; this repo is mostly plain Rust, Bun, shell, and config assets.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crocoder-dev/shared-context-engineering](https://github.com/crocoder-dev/shared-context-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
