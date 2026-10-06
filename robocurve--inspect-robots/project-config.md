---
trigger: always_on
description: This is the single instruction file for every coding agent in this repo. Codex
---

# Inspect Robots — agent guide

This is the single instruction file for every coding agent in this repo. Codex
and most agents read `AGENTS.md` directly; Claude Code loads it through the
one-line `CLAUDE.md` (`@AGENTS.md`). Edit this file, never `CLAUDE.md`.

Inspect Robots is the **"Inspect AI for robotics"**: an open-source evaluation
framework for physical AI / VLA (vision-language-action) models. This repo is the
*framework*; concrete benchmarks and backend adapters live elsewhere (see below).

## The one big idea

LLM evals have one swappable input (the model). Robotics evals have **two**:

- **`Policy`** (the VLA "brain") — observation → `ActionChunk` (open-loop horizon).
- **`Embodiment`** (the robot/sim "body + world") — executes actions, owns spaces.

A **`Task`** (a dataset of `Scene`s + scorers) is defined independently of both.
`eval()` checks a `(policy, embodiment)` pair is compatible, runs the closed-loop
rollout, scores it, and writes an immutable `EvalLog`. Mirrors Inspect AI's
`Task = dataset + solver + scorer`, `eval()`, `EvalLog`, registry/decorator model.

## Layout

- `src/inspect_robots/` — the package (see `src/inspect_robots/CLAUDE.md` for the module map).
- `tests/` — pytest; the `CubePick` mock world exercises the whole stack with no
  hardware or sim.
- `plans/` — design docs. `plans/0001-foundation-design.md` is the authoritative
  spec (read its §9–§11 "binding resolutions" before changing core interfaces).
- `examples/` — runnable demos (`quickstart.py`).
- `plugins/*` — first-party plugin packages (concrete sims/VLAs that are out of
  scope for the numpy-only core), each its own package with its own pyproject,
  entry point, tests, and coverage scope. A uv workspace (`[tool.uv.workspace]`)
  ties them in: `uv sync --all-packages --extra dev` installs core + all plugins
  editable. They never count toward the core 100% gate (coverage is scoped to
  `inspect_robots`). E.g. `plugins/inspect-robots-isaacsim/` (Isaac Lab
  embodiment), `plugins/inspect-robots-ros/` (ROS 1/ROS 2 embodiment speaking
  rosbridge with no ROS dependency), `plugins/inspect-robots-xpolicylab/`
  (policy adapter speaking the XPolicyLab websocket protocol — 40+ served VLAs,
  no xpolicylab dep), and
  `plugins/inspect-robots-agent/` (LLMs as policies via the OpenAI-compatible
  wire formats or Anthropic's native Messages API — httpx only, no provider
  SDKs; registered as `agent`), and
  `plugins/inspect-robots-capx/` (CaP-X code-as-policy over perception and IK
  HTTP servers; registered as `capx`), and
  `plugins/inspect-robots-voice/` (local microphone transcription with Parakeet and
  local Kokoro policy narration; registered as the `voice` operator input and
  `speaker` sink).

## Working here

- Dev loop: `uv pip install -e ".[dev]"`, `uv run pre-commit install`, then
  `uv run pytest --cov`.
- Gates that must pass: `ruff check .`, `ruff format --check .`, `mypy` (strict,
  covers `src` **and** `tests`), and `pytest --cov` at **100% coverage**
  (`--cov-fail-under=100`). Pre-commit runs ruff+mypy on commit and the coverage
  gate on push (`.pre-commit-config.yaml`). CI runs all gates on Linux+macOS /
  py3.11-3.12 as **blocking, required PR checks** (coverage below 100% fails the
  build), plus a **non-blocking** `test-extra` tier for py3.10/py3.13, Windows,
  and the NumPy floor — together covering the py3.10–3.13 range pyproject claims.
- Ruff's D1 rules require a docstring on every public module, class, and function.
  State the contract or invariant the caller needs; do not restate the symbol name.
- **Core stays NumPy-only.** New deps are optional extras, lazily imported; the
  `core-only-import` CI job enforces this.
- Test-driven; commit/push in small focused steps.
- **Changelog entries are fragments**: add `changelog.d/<issue>.<type>.md` (or
  `+<slug>.<type>.md` with no issue); never edit `CHANGELOG.md` in a feature PR.
  See `changelog.d/README.md`. Before merging an older PR that still edits
  `CHANGELOG.md`, move its entry into a fragment.
- Public API is fenced by `inspect_robots.__all__` and guarded by
  `tests/test_api_snapshot.py` — update both together.
- Documentation source lives in `docs/`, and the Docusaurus site lives in
  `website/`. Generate the ignored API page first with `uv run python
  scripts/gen_api_docs.py`, then run `npm ci` and `npm run build` from
  `website/` (`npm run start` previews it). A fresh checkout has no
  `docs/api/index.md`; the npm pre-scripts fail with the generation command
  instead of a sidebar error.

## Out of scope (separate repos / plugins)

Specific benchmarks ("Inspect Evals for robotics"), specific simulators
(ManiSkill/MuJoCo/Isaac), and specific VLA weights (OpenVLA/π0/Octo) ship as
separate plugin packages registered via entry points (`inspect_robots.embodiments`,
`inspect_robots.policies`, …) — either in their own repos or as in-repo `plugins/*`
workspace members (see Layout). Either way they stay out of the numpy-only core
and its 100% coverage gate; `plugins/inspect-robots-isaacsim/` is the reference example.

## CI, merging, and releases

- **main is PR-only** — a branch ruleset (admins included) blocks direct pushes,
  force pushes, and deletion. Merging requires the `ci-ok` check green and the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robocurve/inspect-robots](https://github.com/robocurve/inspect-robots) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
