---
trigger: always_on
description: > **Canonical file.** `CLAUDE.md` is a symlink to this file for Claude Code compatibility.
---

# AGENTS.md

> **Canonical file.** `CLAUDE.md` is a symlink to this file for Claude Code compatibility.

This file provides guidance to AI coding agents when working with code in this repository.

## Project Purpose

This is **Traust**, an agent harness for automated vulnerability discovery, validation, and remediation across a software portfolio — source repositories, container images, RPM packages, Kubernetes operators, infrastructure-as-code and the services built from them — at the scale of hundreds or thousands of repositories. Which repositories that is comes from the deployment's inventory (`locations.inputs`), never from this tree; the executive-summary dashboard under the configured `progress-tracker` root reports current coverage.

The harness contains 51 active skills, slash commands, JSON schemas, report tooling, and the prompt engineering that drives autonomous multi-framework security assessments. It is designed to be agent-agnostic, with current implementations for Claude Code and Crush.

## Repository Architecture

### Skills

Every skill lives as a self-contained directory with a `SKILL.md` prompt and optional implementation scripts. This is the single source of truth. The tree has two levels:

- **Workflow skills** sit under their pipeline stage — `harnessing/<N>-<stage>/<name>/`, e.g. `harnessing/4-triage/triage/`. The nine stage directories are ①–⑨ and carry their order in the name.
- **Everything else** (gates, dashboards, graphs, corpus-QA, quarantined skills) stays at `harnessing/<name>/`, because it is not a stage and filing it under one would make the tree lie.

**Never enumerate skills by globbing `harnessing/*/`.** Go through `skill_dirs()` / `skill_dir(name)` in `src/traust/paths.py`, which spans both levels; a one-level glob silently finds only the root-level skills. Likewise, look a skill up by name rather than building `harnessing/<name>` from parts.

Agent discovery layers are symlinks, and the name an agent sees stays flat regardless of stage:
- `.claude/skills/<name>` → `../../harnessing/[<N>-<stage>/]<name>` (Claude Code)
- `.crush/skills/<name>` → `../../harnessing/[<N>-<stage>/]<name>` (Crush)
- `.claude/commands/<name>.md` — slash command wrappers (thin files that invoke the skill; they reference `.claude/skills/<name>/SKILL.md`, so they never name a stage)
- `.crush/commands/<name>.md` → `../../.claude/commands/<name>.md`

Regenerate the links with `bin/link_skills.sh`, which walks both levels.

When editing a skill, always edit the file under `harnessing/`. Never create a separate copy in `.claude/skills/` or `.crush/skills/`.

### Scripts

Multi-skill CLIs live in `src/traust/cli/` and are invoked as python3 -m traust.cli.<name>. Ops and one-shot migrations live under `src/traust/{ops,migrations}/`. Single-skill CLIs are co-located under `harnessing/<skill>/scripts/`.

## Script placement rule

- Single-skill CLI: the skill's own `scripts/` (co-located with SKILL.md, so under the stage directory for a workflow skill)
- Multi-skill CLI: `src/traust/cli/` (installed package)
- Shared library (>=3 importers): `src/traust/lib/`
- Ops / one-shot: `src/traust/{ops,migrations}/`

`validate_report`, `render_report`, `checkpoint`, `countersign`, `emit_triage_ledger_events`, `emit_validation_ledger_events`, and related tooling are package modules (many in sibling `traust-engine`). Skills invoke them via python3 -m … from the harness venv. The citation gate and symbol index are triage accelerators — they route, gate, tag, or index, and never author a verdict (see `docs/deterministic-inferential-mix.md`).

Legacy placement (pre-C8):
- Used by exactly one skill: place under `harnessing/<skill>/`
- Invoked by 2+ skills as a CLI: place under `src/traust/cli/`
- Imported as a module by 3+ callers: shared library under `src/traust/lib/` or `traust-engine`
- Ops/cron only (no skill references): `src/traust/ops/`
- One-shot migrations/backfills: `src/traust/migrations/` with a dated header

### Metrics consistency layer

`$TRAUST_CONFIG_HOME/corpus-config.yaml` (ownership tags per output tree; write only
via `/corpus-intake`) + `traust_engine.corpus.resolver` (shared discovery/identity/
dedup resolver) + `/census` (denominator authority: population,
duplication vectors, distinct-vulnerabilities headline, repo liveness)
keep every dashboard's numbers reconcilable. Dashboards embed a standard
population block and label metrics by the three-lens taxonomy (work
performed / distinct exposure / systemic patterns — see PROCESS.md).
When adding a dashboard or changing what one counts, build on corpus.py
and state the population block — never hand-roll a walker.

### Sibling repositories

The agent runs from a parent workspace with three sibling trees (the inputs inventory, `analysis-results/`, `progress-tracker/` — all resolved through `locations.yaml`). Skills reference siblings by relative path. See [docs/setup.md](docs/setup.md) for the full workspace layout.

### Deployment configuration vs shipped configuration

`config/` in this repo ships **only** estate-neutral files (`external-tools.yaml`,
`feeds.yaml`, `model-registry.yaml`) plus a `*.example.*` template for every
deployment-specific file. The real corpus registry, product map, budget policy,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openshift/traust](https://github.com/openshift/traust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
