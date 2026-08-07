---
trigger: always_on
description: PoseCap (clean rewrite of Corridor Digital's "Human Input Device" proof of concept): a Blender plugin that drives SMPL-X body models from live webcam pose estimation (PEAR engine), pelvis-locked — world position is a deferred software problem, and the POC's Arduino rig is dropped from scope. The POC at `C:\Dev\CorridorRig-Original` is read-only reference; this repo replaces it with a tested, layered implementation (addon, engine bridge, installers). Hard constraint: SMPL-X model assets carry the
---

# AGENTS.md

## Project Overview

PoseCap (clean rewrite of Corridor Digital's "Human Input Device" proof of concept): a Blender plugin that drives SMPL-X body models from live webcam pose estimation (PEAR engine), pelvis-locked — world position is a deferred software problem, and the POC's Arduino rig is dropped from scope. The POC at `C:\Dev\CorridorRig-Original` is read-only reference; this repo replaces it with a tested, layered implementation (addon, engine bridge, installers). Hard constraint: SMPL-X model assets carry the MPI research (non-commercial) license — never commit or redistribute them; the repo is private now but goes public later, so git history must stay license-clean from the first commit (no licensed binary ever committed, even briefly). Commercial production use of the models requires a Meshcapade license, independent of the plugin's own license.

**Stack:** Python >=3.11 (addon runs in Blender's bundled interpreter; engine bridge in a uv-managed venv), Blender >= 4.2 LTS and 5.x (bpy, extension platform), and isolated Pose Backends per ADR-0010/ADR-0011: PEAR (PyTorch, CUDA required, optional installer module) and MediaPipe Lite (CPU, account-free). torch is not a workspace dependency; it ships only inside the PEAR backend payload.
**Entry points:** uv workspace packages `contracts/`, `core/`, `engine/` (src layout, `posecap_*` import names). Engine CLIs: `posecap-engine` (PEAR) and `posecap-mediapipe` (`engine/pyproject.toml [project.scripts]`). The Blender extension lives in `addon/` (`blender_manifest.toml`).

## Setup, Build, Test

```bash
# Install (engine bridge + dev tooling)
uv sync

# Test (single file preferred over full suite)
uv run pytest tests/<file>.py
uv run pytest

# Run before any commit
uv run ruff check .
uv run ruff format --check .
uv run pyright --pythonplatform Windows
uv run pyright --pythonplatform Linux
uv run lint-imports
uv run pytest
```

Quality gates run as: `uv run ruff check .`, `uv run ruff format --check .`, `uv run pyright --pythonplatform Windows`, `uv run pyright --pythonplatform Linux`, `uv run lint-imports`, `uv run pytest`.

Addon code executes inside Blender's bundled Python: stdlib + `bpy`/`mathutils`/`numpy` only; third-party deps must be vendored in the extension wheel, never uv-installed.

## Quality Gates

See [`GUIDELINES.md`](GUIDELINES.md) §8 for the full reference. Non-negotiable subset:

* Hooks wired via pre-commit; new clones run `uv run pre-commit install` once.
* Pre-commit runs ruff, format check, private-key detection, large-file cap, and licensed-binary blocking.
* Pre-push runs DCO, documentation-link, workflow-security, pyright against explicit Windows and Linux platform stubs, pytest default tags, and import-linter checks.
* CI exposes one stable required check (`CI required`) after DCO, title, Linux/Windows quality, dependency, workflow-security, licensed-binary, and package smoke checks pass.
* Never bypass: no `--no-verify`, no skipped hooks, no deleted failing tests.

## Code Style

See [`GUIDELINES.md`](GUIDELINES.md) §2–§4 for the full reference. Non-negotiable subset:

* ruff is the single formatter and linter; pyright strict on `contracts/`/`core/`; `# type: ignore` only at the `bpy` boundary with reason inline.
* `bpy`, `torch`, `serial`, sockets, filesystem never imported in `contracts/` or `core/` (GUIDELINES §1 dependency rule).
* Wire formats defined once in `contracts/` — never duplicated per consumer.

## Architectural Principles

Binding decisions live in [`doc/adr/`](doc/adr/). Do not reinvent, and do not rely on any digest of that directory: read the ADRs relevant to the layer you are touching (each file carries its own Status).

## Repository Layout

* `contracts/` — wire formats, backend manifests, model-asset checks (stdlib only)
* `core/` — pose math, retarget domain, ports (stdlib + numpy + contracts)
* `engine/` — backend adapters (PEAR, MediaPipe), TCP stream server, CLIs
* `addon/` — Blender extension (bpy boundary; engine launched via subprocess)
* `tests/` — mirrors the source tree per layer; `tools/` — gate and build scripts
* `packaging/` — Windows suite installer and backend payload builds (ADR-0011)
* `assets/` — local test media; licensed model assets are never committed
* `doc/product/`, `doc/specs/`, `doc/tasks/`, `doc/adr/` — product scope, feature specs, task files, decision records
* `doc/guides/`, `doc/reference/`, `doc/workflows.md` — user guides, external reference notes, product flow diagrams; agent workflow rules live in `AGENTS.md` and `GUIDELINES.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CorridorTech/PoseCap](https://github.com/CorridorTech/PoseCap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
