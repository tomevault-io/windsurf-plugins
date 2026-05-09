---
trigger: always_on
description: This repository contains a Python CLI and a browser-based recovery kit. This file defines stable
---

# AGENTS.md

## Purpose

This repository contains a Python CLI and a browser-based recovery kit. This file defines stable
conventions and a glob-based, working-tree inventory contract for contributors and coding agents.

## Scope and Baseline

- Baseline: the current working tree (including uncommitted and untracked files).
- Inventory scope: `src/`, `kit/`, `tests/`, `scripts/`, `.github/`, and `docs/`.
- Inventory excludes transient/generated directories unless called out explicitly:
  `src/ethernity/__pycache__/`, `.venv/`, `.mypy_cache/`, `.pytest_cache/`, `.ruff_cache/`,
  `build/`, `dist/`, `kit/node_modules/`, `kit/dist/`, `src/ethernity.egg-info/`,
  `scripts/__pycache__/`, `tests/**/__pycache__/`, `src/.claude/`.

## Stable Rules (Do / Don't)

- Imports: no nested (runtime) imports; keep imports at top-level.
- Exports: prefer explicit public exports; do not re-export underscore helpers.
- Python readability with Ruff:
  - Keep `ruff format` as the only Python formatter; do not add Black for this repo.
  - Keep Python line length at 100 unless a deliberate repo-wide style change is approved.
  - Use blank lines only for real phase boundaries such as normalize, validate, execute, and report.
  - If a function needs many manual spacing groups to read clearly, split it into small named helpers.
  - Prefer module imports over long symbol lists when importing many names from one module.
  - Avoid broad barrel exports unless the re-exported surface is intentionally public.
  - In tests, replace large `mock.patch(...)` stacks with fixtures, helper context managers, or test support helpers when the same patch bundle repeats.
  - Use `# fmt: off` / `# fmt: on` only for rare, justified hotspots where structure cannot be improved.
- Text: keep ASCII-only edits unless a file already uses Unicode.
- Artifacts: do not edit generated artifacts directly; edit sources and rebuild.
- Kit bundle: do not edit `kit/dist/` or `src/ethernity/resources/kit/recovery_kit.bundle.html` directly;
  rebuild via `kit/build_kit.mjs`.
- Rendering contract: `RenderInputs` requires explicit `doc_type`; do not set
  `context["doc_type"]` and do not infer from template filename.
- Recovery rendering contract: recovery documents must provide structured
  `RenderInputs.recovery_meta`; do not infer recovery semantics by parsing `key_lines`.
- Recovery metadata source: build recovery metadata from runtime values via
  `src/ethernity/render/recovery_meta.py` (for example `build_recovery_meta(...)`) in
  backup/recovery flows, then pass it through `RenderService.recovery_inputs(...)`.
- Key line contract: treat `key_lines` as display/fallback text only; do not attach behavior to
  specific line strings.
- Template capabilities: behavior toggles belong in design style.json files (for example
  `src/ethernity/resources/templates/forge/style.json`) under the `capabilities` object, not ad-hoc
  template-name checks.
- Layout diagnostics: use `RenderInputs.layout_debug_json_path` when layout diagnostics are needed;
  this emits a JSON sidecar and should not alter render behavior.
- Template designs: discovery/prompt surfaces must expose only supported design names:
  `archive`, `forge`, `ledger`, `maritime`, `sentinel`.
  Legacy aliases or stale copied names must not be surfaced. Enforcement point:
  `src/ethernity/config/installer.py`.
- Forge icons: Forge templates must use local material symbols assets via
  `src/ethernity/resources/templates/_shared/partials/material_symbols_local.j2`; do not depend on remote
  icon CDNs.
- Recovery kit index: backup flow may emit a separate `recovery_kit_index.pdf` when the active
  template design provides a compatible `kit_index_document.html.j2`.
- CLI prompts: Questionary is the only prompt library for CLI UI.
- Fallback parser contract: for fallback section filtering, non-empty normalized lines that contain
  characters outside the z-base-32 alphabet must be treated as invalid input (reject), not silently
  discarded.
- Shard version typing: shard payload `version` validation must require a strict integer value
  (`int == 1`); bool and non-integer numeric values are out of profile and must be rejected.
- Format spec source of truth: `docs/format.md` is the only normative format specification.
- Format rationale and operations: keep non-normative guidance in `docs/format_notes.md`.
- Format change ledger: append each format-related delta to `docs/format_changes.md` with
  compatibility and version/profile bump rationale.
- Format PR discipline: when normative format behavior changes, update spec + implementation + tests
  in the same change.

## Architecture Notes

### CLI

- Entry point and command wiring: `src/ethernity/cli/bootstrap/app.py`,
  `src/ethernity/cli/bootstrap/registry.py`.
- Core command surfaces: `backup`, `recover`, `kit`, `config`, `render`.
- Command definitions live in `src/ethernity/cli/features/*/command.py`.
- Shared CLI infrastructure lives in `src/ethernity/cli/shared/`.
- Feature orchestration lives in `src/ethernity/cli/features/`; keep planning and execution
  separated where possible.

### Rendering

- Primary pipeline: spec + model + template + geometry/layout + page assembly + PDF render.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MinorGlitch/ethernity](https://github.com/MinorGlitch/ethernity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
