---
trigger: always_on
description: Single project-context file for **every** AI agent working in this repo —
---

# AGENTS.md

Single project-context file for **every** AI agent working in this repo —
Claude Code, Codex CLI, and any other `AGENTS.md`-aware tool.

> **Edit this file, never `CLAUDE.md`.** `CLAUDE.md` is a one-line pointer
> that imports this one, so a second copy would only drift. The same goes
> for anything it links: fix the source file, not a restatement of it.

Lines beginning `@` are Claude Code file imports and are expanded into
context automatically; other agents should open those paths directly.

## What is Grimoire

Grimoire is a package manager for AI-agent config — a CLI to install,
maintain, and publish AI-agent configuration (skills, rules, prompts)
distributed through standard OCI registries. The binary is named `grim`;
the Rust crate/package is `grimoire`. Shipping: full CLI (20 subcommands),
OCI registry pipeline, catalog publishing, MCP server, TUI. One binary
crate, subsystem modules under `src/`, no stable *library* API — the
binary is the only consumer.

> **Status: stabilizing — preparing 1.0.0.** Released surfaces (CLI, JSON
> output, schemas, layouts, OCI pipeline, catalog publishing) are frozen
> contracts: **breaking changes are prohibited** — evolution is additive-only
> (Principle 9). Treat docs as contracts, flag drift when you find it.

## Project Identity

Product vision, target users, positioning, related repos, comparable tools
and research keywords → [`product-context.md`](./.claude/rules/product-context.md).
Consult when reasoning about project direction, scope trade-offs, ADR
motivation, or research framing. Canonical — keep current (update protocol
at the bottom of that file).

## Rule Catalog

Before planning, research, or an architectural decision, scan "By concern"
in the catalog below. Path-glob rules fire only when a matching file is
already open; the catalog covers everything before that.

@.claude/rules.md

## Build & Development Commands

**Task runner**: [`task`](https://taskfile.dev) (Taskfile v3) is the
primary runner. **Always check `task --list` before inventing ad-hoc
commands.** Taskfiles are tree-structured: root (`taskfile.yml`), subsystem
dirs (`test/`, `.claude/`), `taskfiles/*.taskfile.yml` for cross-cutting.

**Key workflows:**
```sh
task                           # fast check (format + clippy + cargo check)
task verify                    # full quality gate (lint, then build + tests)
task --force verify            # bypass caching — run everything
task rust:verify               # Rust-only gate
task shell:verify              # shell-only gate (shellcheck + shfmt)
task claude:tests              # AI config structural tests
task docs:check                # docs site gate — needs Node 24; task verify does not
```

**Cargo commands** (for finer control): `cargo check`, `cargo build
--release` (binary `grim`), `cargo fmt`, `cargo clippy`, `cargo test`.

**Always run `task verify` after implementation is done.** Always run
`cargo fmt` before commit. Subsystem verify tasks (`rust:verify`,
`shell:verify`, `claude:verify`) are AI dev-loop gates — run the subsystem
gate for the code changed; full `task verify` is the final gate before
commit. Conventions →
[subsystem-taskfiles.md](./.claude/rules/subsystem-taskfiles.md).

## Architecture

**Layout**: a single binary crate. All source under `src/`; binary `grim`;
crate/package `grimoire`. No workspace, no lib/CLI split. Acceptance tests
under `test/`. Rust edition 2024.

**Read the matching subsystem rule before working on code in that area** —
each carries invariants and design decisions not obvious from the code.
Claude Code auto-loads them on path match; other agents open them by hand.

| Path | Subsystem rule |
|---|---|
| `src/**` | [subsystem-file-structure.md](./.claude/rules/subsystem-file-structure.md), [subsystem-cli.md](./.claude/rules/subsystem-cli.md), [subsystem-cli-api.md](./.claude/rules/subsystem-cli-api.md), [subsystem-cli-commands.md](./.claude/rules/subsystem-cli-commands.md) |
| `test/**` (pytest, fixtures) | [subsystem-tests.md](./.claude/rules/subsystem-tests.md) |
| `.github/workflows/**` | [subsystem-ci.md](./.claude/rules/subsystem-ci.md) |
| `taskfile.yml`, `taskfiles/**` | [subsystem-taskfiles.md](./.claude/rules/subsystem-taskfiles.md) |

Beyond the path map, two rules carry weight on any change:
[`arch-principles.md`](./.claude/rules/arch-principles.md) (design
principles, boundaries, glossary) and `quality-core.md` (SOLID/DRY/YAGNI,
Block/Warn/Suggest severity tiers, refactoring discipline). Shareable,
project-independent quality guidance lives in `.claude/rules/quality-*.md`
— load the one matching the language you are editing (`quality-rust.md`,
`quality-python.md`, `quality-bash.md`), and `quality-security.md` (attack
surfaces, OWASP/STRIDE checklist) before any security review.

## Environment Variables

| Variable | Purpose | Default |
|---|---|---|
| `GRIM_HOME` | Root data directory (content store, catalog, global config, global-scope install state at `$GRIM_HOME/state/global.json`). Project-scope install state lives at `<workspace>/.grimoire/state.json`. Global-scope client output lands in vendor-native dirs — see subsystem-file-structure.md | `~/.grimoire` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grimoire-rs/grimoire](https://github.com/grimoire-rs/grimoire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
