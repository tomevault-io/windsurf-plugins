---
trigger: always_on
description: This repository publishes a public-safe Codex preset system: sanitized agent and
---

# Repository agent instructions

## Purpose

This repository publishes a public-safe Codex preset system: sanitized agent and
skill catalog examples, an installable Codex Next plugin, architecture and
migration documentation, a read-only dashboard generator, and legacy/local-dev
scripts for managing repo-local Codex runtime entrypoints.

The V2 production storyline is "source catalog -> plugin package ->
marketplace install". Local suites and repo-local `.codex` entrypoint symlinks
are V1 legacy or local-development compatibility paths. Keep changes aligned
with that plugin-first model and with the architecture-first SDLC catalog shape.

## Codex startup behavior

- Codex is normally started from the repository root, so this file is the
  startup router for repo-local instructions.
- Subdirectory `AGENTS.md` files are on-demand navigation cards. They are not
  guaranteed to be in the startup context when Codex starts from the root.
- Before editing under any directory whose row below says `Yes`, read that
  directory's `AGENTS.md` with `cat <path>/AGENTS.md`.
- If multiple nested `AGENTS.md` files apply, read them from shallow to deep
  before making changes.
- If started from a subdirectory, Codex may automatically load the nearest
  path-chain `AGENTS.md`; still use this root file as the directory router.
- `AGENTS_CN.md` files are human-facing translation/reference files. Codex does
  not treat them as project instruction files unless the user explicitly asks
  to read or update them.

## Directory map

| Path | Responsibility | Local AGENTS.md | Read when |
|---|---|---:|---|
| `README.md`, `README_CN.md` | Top-level English and Chinese project introductions | No | Keep them aligned with public catalog structure when editing user-facing docs |
| `AGENTS.md` | Root startup router for Codex agents | This file | Any repository task from the root |
| `AGENTS_CN.md` | Chinese reference for repository guidelines, not a Codex-loaded instruction file | No | Only when the user asks to sync Chinese guidance |
| `docs/` | Architecture, usage, discovery, migration, public/private, and model catalog documentation | No | Read the specific doc being changed and adjacent docs that reference the same public model |
| `dashboard/` | Stdlib Python read-only dashboard generator, HTML template, and example config | Yes | Before editing `build_dashboard.py`, dashboard templates, config examples, or dashboard docs |
| `scripts/` | Legacy/local-dev filesystem automation for repo-local `.codex` entrypoint symlinks | Yes | Before editing symlink management behavior, CLI flags, cleanup logic, or related tests |
| `tests/` | Unit tests for repository scripts | No | Follow `scripts/AGENTS.md` when changing tests for `scripts/` behavior |
| `examples/catalog/` | Sanitized public agent and skill source catalog | Yes | Before changing agent TOML, skill folders, catalog group docs, or publication boundaries |
| `examples/runtime/` | Public-safe example runtime `AGENTS.md` instructions | Yes | Before changing runtime instruction examples |
| `examples/suites/` | V1 legacy/local-dev suite symlink pattern notes and examples | No | Read `examples/suites/README.md` before changing suite documentation |
| `plugins/` | Installable Codex plugin package parent directory | No | Read the specific plugin card when a plugin package has one |
| `plugins/codex-next/` | Canonical packaged Codex Next plugin built from public-safe skills | Yes | Before changing the plugin manifest, README, bundled skills, package layout, or validation guidance |
| `.agents/plugins/marketplace.json` | Repo marketplace that exposes checked-in plugins | No | Before changing plugin availability or marketplace metadata |
| `.codex/` | Machine-local runtime entrypoint symlinks or project-owned local state | No | Do not edit unless the task explicitly targets runtime entrypoints and the user accepts local filesystem changes |

## On-demand cat protocol

Before editing files under a directory with a local `AGENTS.md`:

1. Run `cat <path>/AGENTS.md` for the nearest listed directory.
2. Follow any stricter local rules in that file.
3. If the target spans multiple listed directories, read each relevant card
   before editing.
4. If a target directory ever contains `AGENTS.override.md`, stop and ask the
   user how to handle the override; do not add or update an ignored plain
   `AGENTS.md` in the same directory.

## Commands

This repository has no committed package manager config, Makefile, or CI
workflow. Confirmed commands come from the checked-in Python scripts, tests, and
repository docs.

| Command | Purpose | Scope | Sandbox notes |
|---|---|---|---|
| `python3 -m unittest discover -s tests -v` | Run committed unit tests for `scripts/sync_codex_entrypoints.py` | repo | OK; uses temporary directories |
| `python3 dashboard/build_dashboard.py --help` | Validate dashboard CLI loads | `dashboard/` | OK |
| `python3 dashboard/build_dashboard.py --config ~/.codex/dashboard/config.toml --json-only` | Generate dashboard state without rendering HTML | `dashboard/` plus configured local paths | Requires local config outside repo; may read local Codex roots and write configured output, normally outside repo |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BlueSkyXN/Codex-is-all-you-need](https://github.com/BlueSkyXN/Codex-is-all-you-need) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
