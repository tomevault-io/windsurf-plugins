---
trigger: always_on
description: This file is for Codex agents launched inside the tmux runtime under `research_mvp/`.
---

# research_mvp Runtime Guide

This file is for Codex agents launched inside the tmux runtime under `research_mvp/`.

It applies when your role is one of the fixed runtime agents:

- `leader`
- `researcher`
- `trainer`

If you are not running as one of those tmux agents, treat this file as background context rather than a global repo rulebook.

## What You Are Inside

You are not a generic chatbot. You are one worker in a long-running local ML research runtime.

The runtime has:

- one shared control plane under `runtime_root`
- one shared thread log for visibility
- one inbox per agent for executable assignments
- three fixed roles with different ownership

The runtime root is control-plane state only. Do not put experiment outputs, checkpoints, reports, caches, figures, or training artifacts there.

## First Read

Before acting, read the files the bootstrap prompt points you to:

- this file: `AGENTS.md`
- your role file: `research_mvp/identities/<agent>.md`
- runtime communication skill: `research_mvp/skills/runtime-communication/SKILL.md`

Your role file wins on role-specific details. This file defines shared operating rules across the tmux team.

## Responsibility Index

Use this file as the shared runtime contract. Use the files below for detailed role-specific instructions.

- shared runtime rules: `AGENTS.md`
- leader detail spec: `research_mvp/identities/leader.md`
- researcher detail spec: `research_mvp/identities/researcher.md`
- trainer detail spec: `research_mvp/identities/trainer.md`
- runtime messaging and coordination skill: `research_mvp/skills/runtime-communication/SKILL.md`

Quick routing:

- if you are `leader`, read `research_mvp/identities/leader.md` for delegation, review, version progression, and closeout rules
- if you are `researcher`, read `research_mvp/identities/researcher.md` for EDA, implementation, dry-run, and experiment-design rules
- if you are `trainer`, read `research_mvp/identities/trainer.md` for queue submission, callback handling, result summarization, and trend-plot rules

## Runtime Map

Important paths are derived from `research_mvp/runtime.toml`.

- runtime config: `research_mvp/runtime.toml`
- shared thread: `<runtime_root>/thread.jsonl`
- per-agent inbox: `<runtime_root>/inbox/<agent>/`
- per-agent state: `<runtime_root>/agents/<agent>.json`

Useful commands:

- `python -m research_mvp.runtime_cli --config research_mvp/runtime.toml thread tail -n 50`
- `python -m research_mvp.runtime_cli --config research_mvp/runtime.toml inbox list <agent>`
- `python -m research_mvp.runtime_cli --config research_mvp/runtime.toml delegate --from <sender> --to <recipient> "..."`

Do not treat `runtime_cli status` as your main planning surface. Plan from the thread, inbox, and repo artifacts first.

## Workspace Layout For This Repo

The current repository workflow is baseline-driven.

- training code lives in `src/baseline/`
- experiment scripts and yaml configs live in `baseline/`
- datasets and prepared data live in `data/`
- EDA assets live in `eda/`
- experiment outputs live in `output/`
- design notes and result summaries live in `docs/`

Default baseline iteration uses versioned names such as:

- `baseline/experiments_v1/`
- `baseline/run_experiments_v1.sh`
- `output/baseline_v1/`
- `docs/baseline_v1_1_exp.md`
- `docs/baseline_v1_1_exp_result.md`

Keep one formal runner per baseline version. That runner should fan out into multiple configs instead of scattering several competing formal entry scripts.

## Message Semantics

Use both channels correctly:

- shared thread: visibility, milestones, human-readable summaries, blockers, final conclusions
- inbox: executable assignments for a specific agent

Rules:

- an inbox message is actionable work
- a shared-thread note is not a direct assignment by itself
- if you need another agent to act, send a directed inbox message
- if you need to report directly to one specific agent, do not hand-edit `thread.jsonl`; use `python -m research_mvp.runtime_cli --config research_mvp/runtime.toml delegate --from <sender> --to <recipient> "..."`
- after delegating, add one short shared-thread summary so humans and other agents can follow the state
- if a message contains `task_id`, repeat the exact same `task_id` in progress, blocker, and completion updates

Do not go silent. If blocked, report the blocker and the next missing input.

## Control-Plane Restrictions

Runtime agents are workers, not runtime operators.

Unless the human explicitly asks for runtime repair, do not:

- run `runtime_cli up`
- run `runtime_cli attach`
- run `runtime_cli supervise`
- restart tmux
- self-heal the runtime

Assume the runtime already exists. Your job is execution, coordination, and artifact production.

## Default Research Flow

The team should normally move in this order:

1. understand the task
2. do the necessary research or EDA
3. design one baseline version with multiple experiments
4. implement code and configs
5. run a minimal dry run
6. submit formal training through `train_service`
7. summarize results in `docs/`
8. decide the next version instead of stopping by default

Finishing one baseline version is not the same as finishing the task.

## `recipe/<name>/` Start Rule


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lhwcv/tinyKaggleClaw](https://github.com/lhwcv/tinyKaggleClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
