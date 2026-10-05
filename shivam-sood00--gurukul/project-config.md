---
trigger: always_on
description: Gurukul is an Isaac Lab research workspace for robot-learning experiments, baselines, tutorials, and
---

# Gurukul Agent Guide

Gurukul is an Isaac Lab research workspace for robot-learning experiments, baselines, tutorials, and
deployment work. Optimize for correct, reproducible, understandable changes rather than turning the repository into a
generic software product.

## Default Workflow

1. Read this file and any nearer `AGENTS.md` that applies to the files being changed.
2. Inspect the relevant source, tests, task registrations, and public documentation before implementing.
3. Search for an existing implementation before adding a task, config, runner, asset, or documentation page.
4. Make the smallest coherent change that satisfies the request. Preserve public behavior unless a change is
   intentional and documented.
5. Run the smallest relevant validation first, then broader or simulator-dependent checks when justified.
6. Update the closest public docs page when user-visible behavior changes.
7. Report what was validated and what could not be run.

Preserve unrelated user changes in the worktree. Do not rewrite adjacent experimental code or perform broad cleanup
unless it is necessary for the requested change.

## Preserve Author-Written Prose

- Treat prose in `README.md` and `website/` as author-written text. Do not rewrite, rephrase, summarize, polish, or
  change its tone unless the user explicitly asks for that specific text to be edited.
- For visual, layout, navigation, or structural requests, preserve the existing wording exactly. Moving or wrapping
  text is acceptable only when its words and meaning remain unchanged.
- If wording appears inaccurate or unclear but the user did not request copy editing, point it out and propose a change
  instead of editing it directly.

## Repository Map

| Path | Purpose |
| --- | --- |
| `source/Gurukul/Gurukul/assets/` | Robot, actuator, sensor, and asset configuration |
| `source/Gurukul/Gurukul/tasks/` | Task implementations, configs, agents, and Gym registrations |
| `scripts/reinforcement_learning/` | Training, playback, distillation, and evaluation entry points |
| `scripts/tools/` | Inspection, conversion, visualization, and debugging tools |
| `tests/` | Targeted regression and contract tests |
| `unitree-sim2real/`, `engineai-sim2real/`, `deploy/` | Sim2sim, hardware, export, and deployment paths |
| `website/docs/` | Maintained public, user-facing documentation |
| `docs/` | Public-safe development notes, audits, and migration records |
| `knowledge/` | Optional gitignored local workspace for literature and implementation planning |
| `third_party/` | Vendored or adapted upstream projects |

## Reuse Before Creation

Before adding a task, config, runner, asset, or documentation page:

- Search the repository, including registrations with `rg "gym.register\(" source/Gurukul/Gurukul/tasks`.
- Check the closest page under `website/docs/tasks/` and `website/docs/reference/task-registry.md`.
- When Isaac Sim is available, use `scripts/tools/list_envs.py` as the runtime source of truth for registered tasks.
- Prefer extending, inheriting from, or composing an existing implementation over copying it.
- Do not add a new task ID merely to rename or duplicate existing behavior. Use an existing config or a documented CLI
  override when it expresses the variation clearly.
- Compatibility aliases are acceptable only when their purpose is explicit and they resolve to a maintained
  implementation.

A genuinely new task should provide distinct runnable behavior and include the relevant environment/config code, Gym
registration, agent entry point when training is supported, targeted validation, and an update to the nearest existing
public task page. Do not create a new documentation page when an existing task-family page is the clearer home.

## Public Documentation

The website is for users, not for agents or maintainers.

- Explain what the feature or task does, its prerequisites, exact commands, expected artifacts, and important
  limitations.
- Keep the public website free of prompts, agent instructions, private paths, investigation logs, and maintainer-only
  reasoning. Authoring rules belong in `AGENTS.md`, not in `website/`.
- Do not claim that a paper or method is implemented without corresponding code/configs and a runnable training,
  evaluation, or replay path.
- Describe support and validation at the exact task or runner level. Do not turn one checked path into a family-wide
  sim, sim2sim, sim2real, or hardware claim.
- Keep pages concise and scan-first: what the task is, then a video only when a real video exists, then main commands,
  then implementation details and field notes.
- Do not add placeholder sections such as "add notes here" or "video not recorded yet". Omit empty sections.
- When a task page has a Notes section, wrap it in `:::note[Field Notes]{.notebook-notes}`.
- Field Notes may include experimental observations, recommended workflows, known limitations, troubleshooting notes,
  and current interface behavior.
- Update the closest existing page rather than repeating the same instructions across multiple pages.
- If setup, training, robot assets, sim2sim, sim2real, task IDs, CLI flags, or artifact contracts change, update the
  corresponding public page in the same change.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shivam-sood00/Gurukul](https://github.com/shivam-sood00/Gurukul) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
