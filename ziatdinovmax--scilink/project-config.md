---
trigger: always_on
description: Forward-looking design decisions and conventions. Intended for AI assistants
---

# SciLink — Architecture Notes

Forward-looking design decisions and conventions. Intended for AI assistants
and contributors working on the orchestrator stack. Codebase tour and
per-module docs are elsewhere; this file is about *direction*.

## The mode universe is fixed at three

Every chat-driven orchestrator in SciLink falls into one of three modes —
this is a settled architectural commitment, not a refactoring waypoint:

| Mode | Class | Domain |
|---|---|---|
| `analyze` | `AnalysisOrchestratorAgent` | Experimental data analysis (microscopy, spectroscopy, …) |
| `plan` | `PlanningOrchestratorAgent` | Experimental campaign design |
| `simulate` | `SimulationOrchestratorAgent` | Computational simulations (DFT today, LAMMPS later) |

Anything in scientific workflow falls under one of these three. There will
**not** be a fourth mode. Future capability growth happens *inside* one of
the three, or as a meta-agent on top (see below).

## Capability expansion through skills, not new agents

Going forward, SciLink intends to extend its agentic capabilities primarily through skill
bundles rather than by adding more specialized subagents. New domains,
techniques, or methods are integrated as skill bundles (knowledge +
tools, co-located) under an existing subagent whose shape already fits;
a new subagent class is justified only when its execution structure
itself cannot be expressed within an existing agent. This applies
across all three modes. For example, adding an XRD or Raman skill for 
existing CurveFittingAgent is strongly preferred over creating two new agents
for Ramand and XRD.

## Plan-mode capability boundaries

Two settled conventions on where capability lives in plan mode:

**Plan-mode skills are knowledge-only.** Skill bundles under
`scilink/skills/planning/<name>/` are markdown — no per-skill
`.py` / `TOOL_SPEC` tools. Plan mode reasons and synthesizes; it does
not execute domain numerics. `PlanningAgent` produces plan text, heavy
compute is `BOAgent`'s, and executable artifacts flow through
`generate_implementation_code` — codegen *guided by* the skill's
`implementation` section, so the skill shapes the code rather than
shipping it. Planning subagents deliberately do not consume the
`_shared/_registry` tool inventory. A planning skill that seems to need
a vetted tool is mis-scoped.

**The scalarizer is the lightweight analysis tier.** `ScalarizerAgent`
does simple LLM-generated extraction (pandas / numpy / scipy) over
tabular or otherwise simple data, reduced to scalars plus the BO
input/target schema. It gets no vetted `.py` tools — needing one is the
tripwire that the task is not lightweight and belongs in analyze mode.
Heavy "data → number" extraction is reused, not rebuilt: `run_analysis`
does the hard work with its skill tools, then the scalarizer reduces the
result (`run_analysis → scalarize`). That cross-mode chain is gated on
the future `run_task` contract; until it exists, run the analysis
standalone and feed the resulting scalar in as a data file.

## Why no `BaseChatOrchestrator` refactor

The three orchestrators share a near-identical chat-loop / message-history /
MCP / autonomy / checkpoint shape (~600 lines each). Reflexively extracting
a base class is tempting and **not what we want at this stage**. The rule
of three says abstract on the third copy when the duplication actually
hurts; bug-fix propagation across three files is acceptable cost.

The trigger to do the refactor is "fixes are diverging across copies" or
"a fourth case appears" — neither holds. The fourth case won't appear
(the universe is fixed at three), so the only legitimate trigger is
maintenance pain. We have not hit it.

When building `SimulationOrchestratorAgent`, copy the structure of
`AnalysisOrchestratorAgent`. Don't refactor the other two.

## What the simulate orchestrator looks like

Structure-centric, iterative, two-surface. **Different from analyze mode**
in three ways: no data file required to start, structure-centric
(not analysis-driven), and includes a post-run feedback loop.

### Tool surface

```
Structure phase
  generate_structure(description)             # one cycle, no validator loop
  validate_structure(path)                    # standalone, post-edit re-run
  refine_structure(path, feedback)            # one refinement cycle
  view_structure(path)                        # 3-axis renders

Inputs phase
  generate_vasp_inputs(poscar, request, method='llm'|'atomate2')
  validate_incar(incar, request)              # literature validation
  apply_incar_improvements(...)

Post-run (currently orphaned in the codebase)
  analyze_vasp_output(output_dir)             # OUTCAR / vasprun.xml summary
  suggest_incar_fixes(log_path)               # wraps existing VaspUpdater

Pipeline shortcut
  run_complete_dft_workflow(description)      # what analyze mode exposes today

Session
  list_generated_structures()
  compare_structures(path_a, path_b)
  set_default_calc_params(...)
```

### Session layout

Structure-centric, not analysis-centric:

```
simulate_session_YYYYMMDD_HHMMSS/
├── structures/
│   └── <structure_slug>/
│       ├── POSCAR / INCAR / KPOINTS
│       ├── script_*.py
│       ├── POSCAR_view_{x,y,z}.png
│       └── outputs/        # user drops VASP run results here

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ziatdinovmax/SciLink](https://github.com/ziatdinovmax/SciLink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
