---
trigger: always_on
description: AgentKernelArena is a controlled environment for evaluating GPU-kernel agents.
---

# AGENTS.md

## Purpose

AgentKernelArena is a controlled environment for evaluating GPU-kernel agents.
Every change must preserve reproducibility, benchmark integrity, meaningful
correctness checks, and fair baseline-versus-candidate comparison.

Read [README.md](README.md) and [CONTRIBUTING.md](CONTRIBUTING.md) before making
broad changes. Use repository-relative paths in code, configuration, docs, and
instructions; never embed paths from a local checkout.

## Repository map and sources of truth

- `main.py`: run orchestration, resume, and parallel scheduling.
- `src/`: shared prompting, workspace setup, evaluation, scoring, and reporting.
- `agents/`: isolated agent integrations and their own configuration.
- `tasks/`: self-contained kernel tasks and task-local runners.
- `example_configs/`: run-level agent, task, and GPU selections.
- `src/tools/perf/`: canonical benchmark helpers materialized into workspaces.
- `src/eval_tools/`: evaluator-side optional analysis-tool control plane.
- `tests/`: framework, integration, and regression tests.

When behavior and prose disagree, inspect the implementation. In particular:

- Agent identifiers and loading: `src/module_registration.py`
- Task definition, schema, and authoring contract: `docs/how-to/add-task.md`
- Task-specific declarations: the task's `config.yaml`
- Evaluation and scoring: `src/evaluator.py`, `src/performance.py`, `src/score.py`
- Harness protection: `src/harness_guard.py`
- Benchmark helpers: `src/tools/perf/`
- Evaluation-tool policy: `docs/how-to/use-evaluation-tools.md`

Update relevant docs in the same change when behavior or public configuration
changes. Avoid copying volatile model IDs, image tags, or complete registries
into new guidance; link to their source of truth instead.

## Architecture boundaries

Keep agents and tasks independent.

- A task defines a backend-neutral contract through its files and `config.yaml`.
  Task code must not import from `agents/` or depend on a particular agent,
  model, prompt format, provider, or authentication mechanism.
- Agent implementations must not import task modules, mutate committed task
  sources, or special-case task names and paths. Add general capability through
  task configuration and shared framework interfaces instead.
- Keep provider- and CLI-specific behavior under `agents/<agent_name>/`. Move
  code into `src/` only when it is genuinely shared by multiple integrations.
- Isolated tasks must remain self-contained after being copied into a workspace;
  they must not rely on imports from this repository's `src/` tree.

## Working practices

1. Inspect the relevant implementation, config, tests, and docs before editing.
2. Keep the change focused; preserve unrelated user changes in a dirty tree.
3. Add or update a focused regression test for behavioral changes.
4. Run the smallest relevant checks first, then broader checks when practical.
5. Report the commands run, results, hardware used, and checks not run.

GPU experiments are Docker-first. Do not reintroduce a host `python main.py`
workflow. Select a config matching the physical GPU architecture and use:

```bash
make docker-smoke
make docker-check-agents CONFIG=<run-config>
make docker-run CONFIG=<run-config>
make docker-parallel-run CONFIG=<run-config> GPU_IDS=0,1
```

Do not claim GPU validation from code inspection or CPU-only tests. If compatible
hardware, agent credentials, or external dependencies are unavailable, state
that explicitly.

## Task authoring and validation

Before adding or modifying any task, you MUST read
[Task definition, schema, and authoring](docs/how-to/add-task.md), including its
implementation-status and migration sections. This applies to task configs,
sources, references, input generators, harnesses, and benchmark scripts.
It is the canonical task guide; inspect nearby task implementations as well.
All retained tasks use schema v2 and the shared loader/evaluator/validator.
New tasks must use that contract and implement its task-owned actions; adding a
field to YAML alone does not implement the associated behavior.

- Paths in an isolated task must resolve within the task directory. Do not use
  absolute paths, undeclared downloads, or external repositories.
- Command fields in `config.yaml` are lists. Commands must exit nonzero on
  failure and honor their declared timeouts.
- Compilation checks must actually compile or syntax-check the target.
- Correctness checks must compare against a meaningful reference with sensible
  tolerances and representative cases; never accept a text-pattern check or a
  trivially passing harness.
- Performance checks must emit scoreable device timing and preserve equivalent
  work, state, and allocation boundaries for baseline and candidate.
- Use `platform_support` for real architecture constraints instead of silently
  skipping cases inside a runner.

Every new task, and every material change to a task contract or harness, must
pass `task_validator` on compatible GPU hardware **before a PR is submitted**:

```bash
make docker-run CONFIG=<validator-config>
```

Require a framework-finalized `validation_report.yaml` whose `overall_status` is
`PASS`. A `WARN` needs an explicit maintainer-approved justification and is not

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AMD-AGI/AgentKernelArena](https://github.com/AMD-AGI/AgentKernelArena) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
