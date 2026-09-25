---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository status: early skill library

`techne` (τέχνη — Greek for *craft · skill · art*) is a fresh start from the
old `atools-js` utility library, but it is no longer docs-only. Current `main`
contains:

- `skills/` as the source of truth for tool-neutral skill bodies.
- `.claude-plugin/` as the Claude Code plugin skin and development marketplace.
- `.cursor-plugin/plugin.json` and `gemini-extension.json` as thin host skins.
- `INSTALL.md` as the install matrix for Claude, Codex, Cursor, Gemini, Kimi,
  and the universal Agent Skills fallback.
- `skills/anchor-viz/`, the first real skill: a typed diagram router for codebase
  architecture, request interactions, data models, state models, and type
  structures, with a mechanical provenance gate.
- `skills/anchor-repro/`, the second real skill: a repro-first bugfix gate that forces
  fail → fix → same-probe verification through a run ledger.
- `skills/anchor-vet/`, the third real skill: an evidence-gated diff review helper
  that computes scope, blast radius, claims, findings, and verdict
  admissibility.
- `skills/anchor-intake/`, the fourth real skill: a written engineering brief
  interrogation gate that accounts a fixed rubric before work starts.

There is still **no root app, no root package manager, no CI, and no repo-wide
test runner**. `skills/anchor-viz/scripts/package.json` only pins helper dependencies
for that skill's Mermaid validator; `skills/anchor-repro/scripts/repro_ledger.py`,
`skills/anchor-vet/scripts/vet_gate.py`, and `skills/anchor-intake/scripts/intake_gate.py`
are dependency-free Python 3 stdlib (POSIX-only). Do not infer commands,
dependencies, or architecture from the legacy library; that direction was
abandoned.

## Project intent

Per the README, techne is a home for **"Skills and agents for the AI era."**
Absent other direction from the user, treat new work as improving shared
skill/agent bodies and their thin distribution skins, not as rebuilding a
published utility package.

The current product rule is **one body, multiple skins**: keep the real skill
content in `skills/`; host-specific files should be manifests, marketplaces, or
install instructions that point back to that shared body.

## Current skill surface

Four skills are seeded: `skills/anchor-viz`, `skills/anchor-repro`, `skills/anchor-vet`,
and `skills/anchor-intake`.

`skills/anchor-viz` (coding/investigate):

- `SKILL.md` forces the cognitive procedure: route the diagram kind, read real
  evidence, draw only evidenced relationships, enforce complexity gates, mark
  provenance, then validate/store/build the viewer.
- Supported `diagramKind` values are `architecture`, `interaction`,
  `data-model`, `state-model`, and `type-structure`.
- Supported Mermaid types are `flowchart` / `graph`, `sequenceDiagram`,
  `erDiagram`, `stateDiagram-v2` / `stateDiagram`, and `classDiagram`.
- `scripts/validate-mermaid.mjs` parses and counts, and — with
  `--project <root>` — mechanically verifies `%% techne:source` /
  `%% techne:inferred` provenance annotations against the target project
  (paths, symbols, citation strength) and computes coverage; without
  `--project` it syntax-checks annotations only. It intentionally does **not**
  call `mermaid.render()` under Node/jsdom; browser rendering is checked
  through the self-contained file viewer.
- `scripts/store_viz.py` runs the validator itself (provenance enforced, no
  bypass flags) and derives `diagramKind`, `type`, `sourceFiles`, `coverage`,
  and `nodeCount` from validator output — there are no self-reported metadata
  flags. It writes diagrams to a target project's `.techne/viz/*.md` and
  `.index.json`, and idempotently adds `.techne/` to that target project's
  `.gitignore`.
- `scripts/build_viewer.py` builds a self-contained `.techne/viz/index.html`.
  It does not start a server.

`skills/anchor-repro` (coding/debug):

- `SKILL.md` forces the cognitive procedure: trigger-check the task as a
  behavioral bug, capture a stable `--expect` anchor, demonstrate the failure
  through the ledger before editing, fix, verify with the byte-identical probe
  identity, then report citing the `close` JSON and strength rung.
- `scripts/repro_ledger.py` (Python 3 stdlib, POSIX/macOS/Linux only) executes
  and records probes in a target project's `.techne/repro/<bug>.jsonl`.
  Classification is computed, never declared: probe identity is
  `{mode, argv | shellCommand, cwd, timeoutSec}`, and `close` exits 0 only on a
  fail → later same-identity pass, or on a loud `mark-unreproduced`
  speculative path.

`skills/anchor-vet` (coding/review-diff):

- `SKILL.md` forces review through a git-anchored scope, full hunk read,
  blast-radius walk, claim cross-examination, severity-honest findings, and
  verdict closure through `vet_gate.py`.
- `scripts/vet_gate.py` (Python 3 stdlib, POSIX/macOS/Linux only) writes
  `.techne/review/<slug>/scope.json`, consumes reviewer-authored
  `review.json`, computes `report.json`, and closes `verdict.json`.
  Classification and admissibility are computed, never self-reported.

`skills/anchor-intake` (general/interrogate):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lynxlangya/techne](https://github.com/lynxlangya/techne) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
