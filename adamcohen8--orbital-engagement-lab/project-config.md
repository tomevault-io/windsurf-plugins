---
trigger: always_on
description: Orbital Engagement Lab agents should orchestrate documented workflows. They
---

# OEL Agent Instructions

Orbital Engagement Lab agents should orchestrate documented workflows. They
should not replace, approximate, or silently bypass the deterministic physics
engine.

Agents are general helpers for OEL, not runners for a fixed example catalog.
Use examples and task cards as onboarding rails and regression fixtures, then
generalize from the documented interfaces to the user's actual question.

This file is intentionally public-safe. It helps AI coding agents such as
Codex, Cursor, Claude Code, Gemini CLI, and Grok Build work with the
open-source OEL core.
For the fuller public agent playbook, read `agents/public/AGENTS.md` and
`docs/oel-agents.md`. For routing broad user intents to documented workflows,
read `docs/agent-capability-routing.md`. For repeatable recipe execution,
completed-run inspection, comparison packets, and standard review plots, read
`docs/agent-task-runner.md`.

For code navigation after the God-file decomposition, use the stable façade
first, then follow the public implementation maps in
`docs/config-api-architecture.md`, `docs/runtime-architecture.md`, and
`docs/plotting.md` to the focused owner.

## Instruction Scope And Product Boundary

- Follow the closest applicable `AGENTS.md`. This root file is the public-safe
  baseline, and `agents/public/AGENTS.md` expands public workflows. A private
  workspace may provide closer instructions for its authorized surfaces.
- Keep public and non-public evidence separate. Do not copy private source, configs,
  customer inputs, provider responses, validation references, or generated
  report packets into public files or public answers.
- Public-core single-scenario execution uses deterministic serial object
  stepping. Automatic within-scenario object workers, hierarchical
  campaign/object planning, Monte Carlo, sensitivity, config queues, and
  controller benchmarks are not public-core capabilities. When those are unavailable,
  use the documented deterministic public fallback rather than recreating
  private orchestration.

## Default Agent Posture

- Treat scenario YAML, CLI commands, Python APIs, tests, docs, and generated
  artifacts as the supported interface.
- Treat **OGP** as the product name for the **OEL General Propagator**:
  OEL's catalog-style general-perturbations family for TLE/mean-element
  products. **OGP-SGP4** is the supported near-Earth SGP4 path, and
  **OGP-SDP4** is the supported deep-space/resonance SDP4 path.
- Treat **ONP** as the product name for the **OEL Numerical Propagator**:
  OEL's configurable numerical propagation path for two-body and
  special-perturbation force-model studies. Use **OGP** for explicit passive
  catalog-style general-perturbations propagation or SGP4/SDP4-family
  mean-element products. Reserve **HPOP** for external reference/validation
  workflows or legacy command names, not as the name of OEL's native
  propagator.
- Interpret the user's intent first. Choose a nearby example only when it
  genuinely fits; otherwise create the minimum viable validated scenario that
  answers the request.
- Prefer small, inspectable changes that match existing OEL patterns.
- Start with the simplest deterministic scenario that can answer the user's
  question. Do not add unrequested physics, sensors, estimators, controllers,
  plots, animations, or campaign machinery.
- Generate scenario YAML from natural language only when the resulting config
  can be validated before execution. For user-provided or otherwise unfamiliar
  YAML, run `--safe-validate` before ordinary importing validation.
- Treat `--safe-validate` as an inspection boundary, not permission to execute
  an untrusted config. Trust referenced plugins, modules, and external paths
  before ordinary validation or execution.
- Run `.venv/bin/python run_simulation.py --config <path> --validate-only` before running
  a new or edited scenario.
- Treat unknown-field errors as intent failures; do not remove or rename fields
  until the normalized contract is understood. Non-empty Cartesian initial
  states require both position and velocity, and scenarios must select exactly
  one orbital-state form.
- Use the checked-in physics models, controllers, mission logic, and output
  writers. Do not invent shortcut physics in agent scripts or reports.
- Prefer the review store query API over ad hoc parsing of large run logs when
  `review/run.sqlite` is available.
- Use `.venv/bin/python -m sim.agent_task` when a bundled recipe, comparison,
  standard plot, or portable `agent_evidence_packet.json` would make the
  workflow more reproducible.
- Explain orbital mechanics, equations, controllers, and outputs from public
  source and public docs only.
- Call out uncertainty, missing validation evidence, and model limits plainly.
- Treat output folders as derived evidence. Confirm that artifacts belong to
  the current config/run before citing them; rerun when provenance or freshness
  is uncertain.
- For performance changes, compare the same inputs before and after, require
  deterministic output parity at the appropriate tolerance, and run the
  applicable external validation workflow. Report timing separately from
  physics correctness and identify the measured hardware/configuration.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adamcohen8/orbital-engagement-lab](https://github.com/adamcohen8/orbital-engagement-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
