---
trigger: always_on
description: This repository uses the DOX AGENTS.md model, adapted from
---

# Repository Guidelines and DOX Rail

## DOX Framework

This repository uses the DOX AGENTS.md model, adapted from
`agent0ai/dox`. AGENTS.md files are binding work contracts for their subtrees.
The root AGENTS.md is the project-wide rail for durable workflow rules,
preferences, constraints, and the Child DOX Index.

### Core Contract

- Work products, source materials, instructions, records, assets, and durable
  docs must stay understandable from the nearest applicable AGENTS.md plus each
  parent AGENTS.md above it.
- The closer an AGENTS.md file is to the work, the more specific and practical
  it should be. A child AGENTS.md may specialize local rules, but it must not
  weaken this root contract.
- Do not rely on memory for repository rules. Re-read the applicable DOX chain
  in the current session before editing.

### Read Before Editing

Before editing files:

1. Read the root AGENTS.md.
2. Identify every file or folder expected to change.
3. Walk from the repository root to each target path.
4. Read every AGENTS.md found along each route.
5. If a parent AGENTS.md lists a child AGENTS.md whose scope contains the path,
   read that child and continue from there.
6. Use the nearest AGENTS.md as the local contract, plus parent docs for
   repo-wide rules.

### Update After Editing

Every meaningful change requires a DOX pass before the task is done. Update the
closest owning AGENTS.md when a change affects:

- purpose, scope, ownership, or responsibilities;
- durable structure, contracts, workflows, or operating rules;
- required inputs, outputs, permissions, constraints, side effects, or artifacts;
- durable user preferences about behavior, communication, process,
  organization, or quality;
- AGENTS.md creation, deletion, move, rename, or index contents.

Update parent docs when parent-level structure, ownership, workflow, or child
index entries change. Update child docs when parent changes alter local rules.
Small edits that do not change behavior or contracts may leave docs unchanged,
but the DOX pass still must happen.

### Child Doc Shape

Create a child AGENTS.md when a folder becomes a durable boundary with its own
purpose, rules, responsibilities, workflow, materials, or quality standards.
Default section order:

- Purpose
- Ownership
- Local Contracts
- Work Guidance
- Verification
- Child DOX Index

Keep docs concise, current, and operational. Document stable contracts, not
diary entries. Put broad rules in parent docs and concrete details in child
docs. Delete stale or contradictory text instead of explaining history.

### Closeout

Before finishing a task:

1. Re-check changed paths against the applicable DOX chain.
2. Update nearest owning docs and any affected parents or children.
3. Refresh affected Child DOX Index entries.
4. Remove stale or contradictory text.
5. Run existing verification when relevant.
6. Report any docs intentionally left unchanged and why.

### Child DOX Index

There are currently no child AGENTS.md files. This root file owns the top-level
contracts for these durable areas:

- `.github/workflows/`: CI, version bumping, release, PyPI, Homebrew, and AUR
  automation. Keep workflow changes aligned with release checks and actionlint.
  Homebrew bottle publishing should stay pinned to macOS 15 unless package
  coverage is deliberately moved to a different supported bottle target.
- `src/plextui/`: Python/Textual app source, Plex API mapping, artwork,
  config/auth, mpv playback, and smoke entry points.
- `tests/`: pytest coverage for app helpers/navigation, service mapping,
  config/auth/player/artwork behavior, and release workflow checks.
- `scripts/`: release/package maintenance scripts used by checks, deterministic
  release staging, and post-release automation.
- `packaging/`: Homebrew and AUR notes plus source AUR metadata.
- `docs/`: research notes and README visual assets.
- Root docs and config examples: README, DESIGN, PACKAGING, RELEASE, ROADMAP,
  CHANGELOG, SECURITY, Makefile, pyproject, and `config.example.toml`.

Add child AGENTS.md files only when a subtree needs local rules that would make
this root rail too broad.

## Project Structure & Module Organization

This is a Python/Textual terminal app for browsing Plex and launching playback
through `mpv`.

- `src/plextui/`: application source.
  - `app.py`: Textual UI, navigation, settings, and high-level actions.
  - `plex_service.py`: Plex API mapping and media detail extraction.
  - `player.py`: `mpv` launch, stream selection, and playback diagnostics.
  - `config.py`, `auth.py`, `artwork.py`, `models.py`: supporting modules.
- `tests/`: pytest suite, split by app helpers/navigation and service modules.
- `.github/workflows/`: CI plus PyPI/TestPyPI/AUR validation workflows.
- `packaging/`: Homebrew and AUR maintenance notes; `packaging/aur/` contains
  the source copy of `PKGBUILD` and `.SRCINFO`.
- `README.md`, `CONTRIBUTING.md`, `PACKAGING.md`, `RELEASE.md`, `ROADMAP.md`:
  user, contributor, and release docs.
- `config.example.toml`: example user configuration.

## Build, Test, and Development Commands

Use the project Makefile from the repository root:

```bash
make install-dev   # install editable package with dev dependencies
make run           # start the TUI locally

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [so1omon563/plex-tui](https://github.com/so1omon563/plex-tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
