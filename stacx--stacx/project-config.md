---
trigger: always_on
description: handles, then return immediately. The driver stream goes to a log that
---

# STACX Agent Runbook

This is the first file a coding agent should read in this repo. It is a router
and a set of verified happy paths. Prefer these commands over scattered ad hoc docs.

Rules for agents:

- Work from the repo root unless a command says otherwise.
- Read only the section needed for the task, plus its referenced detail doc.
- Before launching any shell script, read its header comments. The headers are
  part of the run documentation and list current knobs/defaults.
- Do not start a full multi-hour run until the matching smoke or data/setup
  verification passes.
- Large models, task trees, image builds, logs, results, and checkpoints live
  outside git under `external/`, `/scratch/$USER/...`, or a configured profile
  scratch directory.

## 0. Big Picture

STACX trains and evaluates coding agents in real Docker sandboxes. Agents run
in one of **two lanes**, and every task in this runbook is one cell of the
lane × train/eval matrix:

- **In-house agent lane** — the agent loop is implemented in this repo
  (`rl_engine/examples/{swe_agent,retool,mle_dojo}`), so the trainer and
  evaluator see every token natively. The model is always served in-process
  inside the run's own container.
- **Installed-agent lane** — an external scaffold (OpenHands / Terminus-2) is
  installed black-box inside the task container. Any benchmark in Harbor task
  format runs through the shared infra in `rl_engine/examples/shared/`.

```text
installed lane: source benchmark -> Harbor-format task dirs
  -> tasks.jsonl + tests/ bundle -> rollout scheduler
  -> installed scaffold inside Rock sandbox
  -> tests/test.sh verifier reward -> eval JSONL or trainer update

in-house lane:  dataset JSONL -> rollout scheduler -> in-house agent loop
  (tool calls executed in Rock sandboxes) -> verifier reward
  -> eval JSONL or trainer update
```

Routing matrix — entry point and model serving per cell:

| Lane × mode | Entry point | Model serving | Section |
| --- | --- | --- | --- |
| Installed-agent eval | `shared/docker_run_eval.sh` | **external SGLang server** (`serve_sglang.sh`, `.venv-sglang`) | 4 |
| Installed-agent train | `shared/docker_run_train.sh` | in-process, inside the slime-rl image | 5 |
| In-house train | `scripts/train/swe/` recipes; ReTool `scripts/rl.sh` | in-process | 6.1, 6.2 |
| In-house eval | per-example `scripts/eval.sh` (SWE agent via `docker_launch.sh`) | in-process | 6.3 |

Important directories:

| Path | Purpose |
| --- | --- |
| `rl_engine/trainer/` | Training engine and algorithm recipes. |
| `rl_engine/rollout/` | Scheduler, evaluator routing, installed-agent runtime. |
| `rl_engine/examples/shared/` | Shared launchers: data prep, image build, eval, train, SGLang serve. |
| `rl_engine/examples/<bench>/` | Installed-lane benchmark integrations: config, evaluator profile, README. |
| `rl_engine/examples/{swe_agent,retool,mle_dojo}/` | In-house agent loops: agents, tools, sandbox clients, standalone eval scripts. |
| `rl_engine/examples/adapters/` | Local adapters from raw benchmark data to Harbor-format task dirs. |
| `env_engine/` | Rock sandbox service and CLI. |
| `scripts/` | SWE-agent end-to-end recipe scripts. |

Environment standard: two small pinned host uv envs (Rock, SGLang serving);
train and eval drivers run inside the slime-rl docker image. No conda envs,
no host RL venv.

- Rock env: `env_engine/.venv`, created with `uv sync --all-extras`, used by
  `rock admin start`. Host-only by necessity (spawns sibling containers).
- SGLang serving env: `.venv-sglang` at the repo root, pinned
  `sglang[all]==0.5.10rc0`, used by `serve_sglang.sh` for installed-agent
  eval only (Section 2.5).
- Train/eval env: the `lichangh20/slime-rl:stable` image, entered via
  `shared/docker_run_train.sh` / `shared/docker_run_eval.sh`.
- Repo env (`.venv` from `scripts/setup_env.sh`): optional, for host-side
  development and CPU tests only — not needed to train or eval.

Serving asymmetry (structural, not an accident): exactly one cell of the
matrix uses the external SGLang server — **installed-agent eval**. There
only token generation leaves the image, so any model a new-enough external
SGLang can serve (e.g. Qwen3.5) is evaluable long before it is trainable.
Every other cell — installed-agent train, in-house train, in-house eval —
is **self-contained**: it serves SGLang in-process on its own GPUs inside
the slime-rl image (training also syncs weights in-process), so the model
must be supported by the image's matched slime/SGLang/Megatron stack
(Qwen3 family; the image predates Qwen3.5, so Qwen3.5 training is
unsupported). The external server and `.venv-sglang` belong to
installed-agent eval alone; nothing else touches them.

## 1. Task Router

| If asked to... | Read/run |
| --- | --- |
| Set up a new machine | Section 2, then Section 3 for the needed data/model/task assets. |
| Evaluate a benchmark with an installed scaffold (OpenHands / Terminus-2) | Sections 2, 3.1, 3.2, and 4. Then the benchmark README under `rl_engine/examples/<bench>/README.md`. |
| Train with the shared installed-agent scaffold (SFT / GRPO) | Sections 2, 3.2, 3.3, and 5. Detail: headers of `rl_engine/examples/shared/train.sh` and `docker_run_train.sh`. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [STACX/stacx](https://github.com/STACX/stacx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
