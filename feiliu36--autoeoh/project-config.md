---
trigger: always_on
description: AutoEoH is a natural-language orchestrated workflow for automated heuristic
---

# AutoEoH Agent Instructions

AutoEoH is a natural-language orchestrated workflow for automated heuristic
algorithm design. Use these instructions in any agent environment unless a
tool-specific adapter overrides only the entry wording.

## Objective

Given either:

- a text problem description,
- an existing solver/codebase,
- a saved task package, or
- an existing EoH run directory,

execute the appropriate AutoEoH workflow and leave the repository in a state
where outputs are written to disk and easy to inspect.

## Source of Truth

The execution contract is defined by:

- `agents/task_creator/agent.md`
- `agents/codebase_analyzer/agent.md`
- `agents/task_formulator/agent.md`
- `agents/task_checker/agent.md`
- `agents/evaluation_checker/agent.md`
- `agents/eoh_runner/agent.md`
- `agents/results_checker/agent.md`
- `shared/config_loader.py`
- `shared/task_package.py`

If those files disagree, prefer the implemented Python modules over prose.

## Pipeline Modes

Choose one entry mode based on what you have available:

| Mode | When to use |
|---|---|
| `from_description` | Start from a text problem description |
| `from_code` | Start from existing algorithm code |
| `from_task` | TaskPackage already exists on disk |
| `check_only` | Re-check an existing run directory |
| `eval_check_only` | Diagnose null-score issues without a full run |

### Data Flow

```
config.yaml
    -> build_llm()
    -> TaskCreatorAgent  (from_description)
       CodebaseAnalyzeAgent -> TaskFormulatorAgent  (from_code)
       TaskPackage.load()  (from_task / check_only / eval_check_only)
    -> TaskCheckerAgent
    -> EvaluationCheckerAgent
    -> EoHRunnerAgent
    -> ResultsCheckerAgent
    -> Run summary
```

## Required Workflow

### Mode 1 — `from_description`

> Start from a `description.md` or user-provided text.

- [ ] Step 1 — Load `config.yaml` and build LLM backend
- [ ] Step 2 — Run `TaskCreator` → writes task package to disk
- [ ] Step 3 — Run `TaskChecker` → validates task package
- [ ] Step 4 — Run `EvaluationChecker` → validates evaluation function
- [ ] Step 5 — Run `EoHRunner` → executes evolution
- [ ] Step 6 — Run `ResultsChecker` → summarises best result

### Mode 2 — `from_code`

> Read the target codebase and optional dataset path.

- [ ] Step 1 — Load `config.yaml` and build LLM backend
- [ ] Step 2 — Run `CodebaseAnalyzer` → identifies the key heuristic component
  - `TaskFormulatorAgent` auto-loads `description.md` from the example folder
    (`code_dir` or its parent) and passes `Problem Statement` + `Heuristic Goal`
    as `problem_description` to both `CodebaseAnalyzer` and `TaskFormulator`.
- [ ] Step 3 — Run `TaskFormulator` → writes task package to disk using that analysis
- [ ] Step 4 — Run `TaskChecker` → validates task package
- [ ] Step 5 — Run `EvaluationChecker` → validates evaluation function
- [ ] Step 6 — Run `EoHRunner` → executes evolution
- [ ] Step 7 — Run `ResultsChecker` → summarises best result

### Mode 3 — `from_task`

> Load an existing `TaskPackage` from disk.

- [ ] Step 1 — Load `config.yaml` and build LLM backend
- [ ] Step 2 — Load `TaskPackage` from disk
- [ ] Step 3 — Run `TaskChecker` if package was newly generated or edited
- [ ] Step 4 — Run `EvaluationChecker` → validates evaluation function
- [ ] Step 5 — Run `EoHRunner` → executes evolution
- [ ] Step 6 — Run `ResultsChecker` → summarises best result

### Mode 4 — `check_only`

> Re-check results of a completed run without re-running EoH.

- [ ] Step 1 — Load `config.yaml` and build LLM backend
- [ ] Step 2 — Run `ResultsChecker` on the existing run directory

### Mode 5 — `eval_check_only`

> Diagnose null-score issues without starting a full EoH run.

- [ ] Step 1 — Load `config.yaml` and build LLM backend
- [ ] Step 2 — Load `TaskPackage` from disk
- [ ] Step 3 — Run `EvaluationChecker` to diagnose the evaluation function

Do not skip `TaskChecker` for newly created tasks.

## Configuration

Load configuration from `config.yaml` using `shared.config_loader.load_config()`.
Build the LLM backend using `shared.config_loader.build_llm()`.

Provider-neutral environment variable precedence:

1. `AUTOEOH_LLM_API_KEY`
2. Provider fallback such as `OPENAI_API_KEY` or `ANTHROPIC_API_KEY`

## Agent Expectations

- Prefer deterministic file outputs over conversational summaries.
- Save generated tasks via `TaskPackage.save(directory, config=config)`.
  Always pass the current `config` dict so that `runEoH.py` and
  `run_pipeline.py` are generated with all LLM credentials and EoH parameters
  embedded.
- `TaskPackage.save()` writes two user-facing scripts:
  - `<task_dir>/runEoH.py` — self-contained EoH-only re-run entry point.
  - `<task_dir>/../run_pipeline.py` — full pipeline entry point (TaskChecker
    → EvaluationChecker → EoHRunner → ResultsChecker) written one level above
    the task directory, alongside the `output/` folder.  Users run
    `python run_pipeline.py` from the example directory to execute the entire
    pipeline in a single process without any additional authorisation steps.
- Keep task metadata populated with provenance where available.
- When a documented path in prose does not exist, inspect the actual repository
  and use the real path instead of assuming the docs are current.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FeiLiu36/AutoEoH](https://github.com/FeiLiu36/AutoEoH) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
