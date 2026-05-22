---
trigger: always_on
description: Always follow the scientific workflow in `skills/exploration-confirmation/SKILL.md`
---

# IBL AI Agent Runtime Instructions

## Scientific Workflow

Always follow the scientific workflow in `skills/exploration-confirmation/SKILL.md`

- Exploratory analysis to refine or change the original question in view of the data, define precise hypotheses, and provide preliminary evidence.
- Confirmatory analysis to statistically confirm hypotheses
- Report writing

## Be flexible

The user's original question might not be answerable from the data, or might not be as interesting as something else that comes up in exploration. Feel free to suggest refined or alternative questions to the user.

Scientific metrics have tunable parameters. Pick them carefully from exploratory analysis before before locking for confirmatory tests. Consider grid search or other systematic methods of choice, and explain to the user why you picked these parameters.


## Start small

Don't run big analyses, without first testing on a small scale: test code on individual cells, experiments, etc before running on all data. Perform diagnostics on metrics on a small scale within the exploration set.

Before performing a large run, estimate how long it will take based on previous small scale tests. Consult with the user before starting, presenting acceleration options such as vectorization or parallelization if available.

## Save intermediate results

Many analyses require intermediate computational steps such as ACGs or PSTHs, which are subsequently used in ways not yet finalized. To save time, consider saving these as checkpoint files in `projects/<project_slug>/artifacts` in a way that allows them to be created and resused as needed. Make sure the artifacts can be added to, so existing computations do not need to be rerun.

Do this whenever you think the results of a computation will possibly be used again. If you find yourself running a computation twice, that is a sign you should have saved an artifact.

## Keep a running plan

Before starting work, make a list of steps to perform in `projects/<project_slug>/TODO.md`. Give each step a check box `[ ]`. When a step is complete make it `[X]`, also listing any code and output files generated.

The TODO should also list points when to consult the user for feedback.  Before starting execution, show this plan to the user and ask for feedback, including on how often they want to be consulted. By default, consult the user often during exploratory analysis, and certainly before proceding to confirmatory analysis.  When consulting the user provide plentiful explanatory plots.

Keep a running summary of the explicated question and term definitions in `projects/<project_slug>/question.md`

Both `TODO.md` and `question.md` can be dynamic: as exploration proceeds, the question and TODO items not yet performed can change.  But don't change items already completed. When making changes to either file, save in `projects/<project_slug>/change-log.md`


## Context-window diagnostics

Always tell the user when you read a skill or reference file into context, and why you did it. If you read only part of a file, say which part and why.

## Required Load Packets

Plain IBL scientific question:
- `skills/exploration-confirmation/SKILL.md`
- `skills/ibl-analyze/SKILL.md`
- `skills/ibl-report/SKILL.md` when reporting results

Ambiguous scientific metric:
- `skills/ibl-analyze/references/scientific_context_and_metric_semantics.md`
- applicable caveat cards under `skills/ibl-analyze/references/scientific_caveats/`

IBL data loading:
- `skills/ibl-access/SKILL.md` when endpoint, auth, or query mode matters
- `skills/ibl-load/SKILL.md`
- `skills/ibl-load/references/data_loading.md`
- `docs/data_locations.md` when local data paths are needed

Brain Wide Map question:
- `skills/ibl-load/references/bwm_runtime_policy.md`
- `skills/ibl-load/references/bwm_ephys_spike_example.md` for local spike-shard code
- `skills/ibl-analyze/references/bwm_analysis_patterns.md`

Anatomical brain atlas navigation or brain region-based visualization:
- `skills/ibl-anatomy/SKILL.md`
- `skills/ibl-anatomy/references/atlas_navigation.md`

Raw Neuropixels or SpikeGLX preprocessing:
- `skills/ibl-neuropixel/SKILL.md`
- `skills/ibl-neuropixel/references/neuropixel_routing.md`

Skill maintenance:
- `skills/skill-maintenance/SKILL.md`

## Project directory

Before starting a scientific analysis, check for an optional repo-root
`ibl-agent.local.yaml`. If present and it defines `project_root`, use that
directory as the project root. Resolve a relative `project_root` against the
repository root.

If no local config is present, use the repository-local `projects/` directory.
All outputs for a session belong under `<project_root>/<project_slug>/`, and
nowhere else.

- `<project_root>/<project_slug>/question.md` a dynamic document containing the original question, current refined explication, definitions of terms, and definition of exploration and confirmation sets;
- `<project_root>/<project_slug>/TODO.md` for a sequential list of steps performed and planned. Change [ ] to [X] on completion and list output files generated. You can change future plans in the list but do not change descriptions of steps already performed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [int-brain-lab/ibl-ai-agent](https://github.com/int-brain-lab/ibl-ai-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
