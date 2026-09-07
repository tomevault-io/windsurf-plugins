---
trigger: always_on
description: NebulaMat is a local-first, reproducible AI research workbench. The project
---

# NebulaMat

NebulaMat is a local-first, reproducible AI research workbench. The project
uses DeepSeek Harness as its agent runtime; scientific features remain modular
application and MCP modules above that runtime boundary.

## Workspace and startup

- For NebulaMat source maintenance, work from the registered repository root,
  normally `C:\Users\泰\Desktop\Open Science - 2\open-science-source`.
  For a scientific conversation, the DSH-provided current directory is the
  authoritative session workspace (normally `sessions/<date-time>`). Keep all
  generated structures, figures, scripts, reports, notebooks, and run state
  there; never redirect conversation output to the repository root merely
  because this ancestor `AGENTS.md` lives there.
- Read this file before acting. Read `runtime/harness/README.md` and the
  harness manifest only when the task changes runtime composition, tool policy,
  or workspace seeding. Do not load every role or workflow for a routine task.
- Keep all project files and code in English; communicate with the user in
  Simplified Chinese unless requested otherwise.
- Stay inside the active workspace. Preserve unrelated user changes and do
  not reset or publish the repository.

## Lightweight agent routing

`planner` is the default coordinator, but it is not a mandatory extra model
turn. It chooses the smallest route and may execute a fast-path task itself.

| Route | Use for | Required modules |
| --- | --- | --- |
| `fast` | reading, explanation, local checks, small config/code edits | planner/primary only; no subagent or reviewer |
| `standard` | literature synthesis, candidate generation, MatterSim/UMA screening, structure analysis | planner plus only the needed `literature` or `materials` module; one compact synthesis |
| `high-risk` | VASP/DFT, remote compute, experiments, external claims, runtime/model policy changes | planner, `compute`/`experiment`, explicit scope-cost audit, human approval, and targeted review |

Do not create a role or review stage merely because a role exists. Parallelize
only independent work with a clear benefit. A task may be promoted to a higher
route when new risk appears; it should never be forced through the full route
in advance.

Stable roles:

- `planner`: classify, schedule, bound cost, and summarize the run.
- `materials`: MatterGen constraints, structure validation, MatterSim and UMA
  screening, and provenance.
- `literature`: retrieve and extract evidence with source links and uncertainty.
- `compute`: prepare and validate VASP/DFT inputs and remote execution plans.
- `experiment`: translate candidates into synthesis and electrochemical tests.
- `reviewer`: independent, targeted review only for high-risk work or when the
  user asks for review.

## Materials discovery contract

The following electrocatalysis sequence is a reference recipe, not a mandatory
DAG:

`MatterGen` -> controlled bulk/surface standardization -> surface and adsorbate
construction -> fairchem `UMA` (`uma-s-1p2p1`, task `oc25`) adsorption screen ->
ASE + UMA surface molecular dynamics on a `2 x 2 x 1` in-plane expansion ->
VASP validation -> experiment. MatterSim remains an optional bulk relaxation
proxy; it is not a mandatory thermal-stability stage before surface dynamics.

For a real task, the planner should first call `get_material_capabilities()`
and submit a `capability_plan` to `create_materials_workflow`. The plan names
only the required modules, explicit dependencies, parameters, and acceptance
tests. It may skip stages, branch MatterSim and UMA after a shared structure,
or add VASP/experiment only when justified. Unknown capabilities, duplicate
task IDs, and cycles are rejected; the runtime never appends hidden reviewer,
completion, or model stages.

MatterGen generates candidates; it does not prove stability or synthesizability.
MatterSim is a first-stage energy/force/stress and relaxation proxy trained
primarily for bulk materials. Slab and surface results are qualitative until
calibrated against VASP. UMA adsorption energies are same-model ML descriptors,
not complete activity, selectivity, solvent, potential, pH, or free-energy
predictions. Never mix MatterSim and UMA energies in one adsorption expression.

For MatterGen output, "standardization" has one precise meaning: expand the
primitive cell to a controlled bulk atom-count window (40 atoms by default),
and, when requested, build a Miller-index slab with an exact real atomic-plane
count along `cross(a,b)`, a total vacuum gap, and a comparable in-plane atom
window. The default surface target is about 96 atoms; require both in-plane
vectors to be at least 12 A, `c/min(a,b) <= 4`, and slab thickness no greater
than 20 A. Use
`standardize_mattergen_structure` or the MatterGen runner's manifest-backed
outputs. Sorting atoms, reducing a formula, changing symmetry notation, or
rewriting an unchanged CIF/POSCAR is not standardization. Keep every generated
file inside the active session workspace; never write results to a repository
root or sibling session.

The governed tools are `run_mattersim_stability_screen`,
`run_uma_adsorption_energy_screen`, and `run_uma_surface_md`. Each result must
retain model/checkpoint identity, checkpoint SHA-256, software version, input

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tai609/NebulaMat](https://github.com/Tai609/NebulaMat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
