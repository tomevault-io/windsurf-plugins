---
trigger: always_on
description: This file is for **coding agents** (Hermes, Cursor, Claude Code, etc.) working in this repository. Follow it for install, benchmark runs, and safe edits.
---

# HermesBench — AGENTS.md

This file is for **coding agents** (Hermes, Cursor, Claude Code, etc.) working in this repository. Follow it for install, benchmark runs, and safe edits.

## What this repo is

**HermesBench v0.3.0** (canonical repo: **https://github.com/am423/hermes-bench-tool-call**) benchmarks **models inside real [Hermes Agent](https://github.com/NousResearch/hermes-agent)** — tool use (terminal, patch, search, execute_code, web, memory, …), not chat-only QA.

- **51 tasks** under `tasks/**/task.yaml` (48 core categories + 3 `t12_real_world`, difficulties 1–3)
- Each task: isolated git worktree, prompt, verifier under `tasks/.../verifier.py`
- **Official benchmark command:** `hermesbench run` → spawns `run_agent.py` from the Hermes Agent checkout

There is **no** fake-agent benchmark path in the CLI anymore. `tests/support/fake_hermes.py` exists only for optional pipeline tests.

## Repository layout (agent-relevant)

```
hermesbench/
  cli.py              # CLI entry (run, validate, doctor, setup, report, …)
  run_real.py         # Benchmark engine (run_agent.py per task)
  preflight.py        # doctor checks + pip --install
  hermes_invocation.py # find_hermes_agent(), hermes_python()
  runner.py           # Legacy tmux/fake pipeline — not used by `hermesbench run`
tasks/                # Task definitions + verifiers
results/<run_id>/     # summary.json + per-task JSON
traces/<run_id>/      # logs, trace.jsonl, worktrees
scripts/bootstrap.sh  # Clone-and-run venv setup
docs/GETTING_STARTED.md
docs/PROVIDERS.md
docs/RUN_LAYOUT.md
presets/               # Rerun presets (see presets/README.md)
```

## Prerequisites (check before benchmarking)

| Requirement | Why |
|-------------|-----|
| Python **3.11+** | Package + tests |
| **Hermes Agent** checkout | `run_agent.py`, toolsets, providers |
| Hermes **`.venv`** with `pip install -e .` | `fire`, `python-dotenv`, agent deps |
| **`~/.hermes/config.yaml`** (or `OPENAI_*`) | Model provider / API |
| **tmux**, **bash** | Hermes terminal backend in benchmarks |
| **git** | Worktrees per task |

Optional: **ffmpeg**, **agg** (casts), **Node 18+** (HyperFrames video via `hermesbench report --render-video`).

## Install (clone → runnable)

Run from repo root:

```bash
./scripts/bootstrap.sh
source .venv/bin/activate
hermesbench doctor --install
hermesbench setup --hermes --check-only
hermesbench validate
```

If Hermes Agent is missing:

```bash
git clone https://github.com/NousResearch/hermes-agent ~/.hermes/hermes-agent
cd ~/.hermes/hermes-agent && python3 -m venv .venv && .venv/bin/pip install -e .
```

Set provider in `~/.hermes/config.yaml`. For xAI OAuth / config-driven models, always use **`--use-hermes-config`** on runs (see below).

Override checkout path: `export HERMES_AGENT_PATH=/path/to/hermes-agent`

## Benchmark commands (only entrance: `hermesbench run`)

### Smoke (one task)

```bash
hermesbench run --use-hermes-config \
  --model YOUR_MODEL_ID \
  --task t01_terminal_smoke/t01_echo \
  --toolsets all
```

### Full suite (51 tasks)

```bash
hermesbench run --use-hermes-config \
  --model YOUR_MODEL_ID \
  --all \
  --toolsets all \
  --run-id my_run_$(date +%Y%m%d)
```

### Resume a partial real run

Skips tasks already **PASS** or **FAIL** in `results/<run_id>/summary.json`; merges new results into the same summary.

```bash
hermesbench run --resume my_run_20260618 --all --use-hermes-config --model YOUR_MODEL_ID

hermesbench run --run-id my_run_20260618 --resume-skipped --category t03_patch_edit \
  --use-hermes-config --model YOUR_MODEL_ID
```

See `docs/RUN_LAYOUT.md` for layout and legacy vs real resume behavior.

### Legacy engine

```bash
hermesbench run --engine legacy --task t01_terminal_smoke/t01_echo \
  --model local-model --base-url http://127.0.0.1:8080/v1
```

### Category

```bash
hermesbench run --use-hermes-config --model YOUR_MODEL_ID \
  --category t03_patch_edit --toolsets all
```

### OpenAI-compatible env (no Hermes config file)

Omit `--use-hermes-config` and set:

```bash
export OPENAI_API_KEY=...
export OPENAI_BASE_URL=http://127.0.0.1:8080/v1   # optional override
hermesbench run --model local-model --all --base-url "$OPENAI_BASE_URL"
```

For local no-auth servers, `OPENAI_API_KEY` may be unset; HermesBench injects a `dummy` key into `run_agent.py` so the run stays on the supplied `--base-url`. It also sets `TERMINAL_CWD` to each isolated task worktree so relative file/tool operations do not escape into the source checkout. If logs show a cloud/OAuth endpoint instead of the local URL, mark that run invalid and rerun with the current endpoint-routing fix.

### Dry-run (task selection only)

```bash
hermesbench run --dry-run --all --model x
```

Prints selected tasks; does not call the API or Hermes.

### After a run

```bash
hermesbench report --run-id <run_id>
# optional video:
hermesbench report --run-id <run_id> --render-video
```

Artifacts:

- `results/<run_id>/summary.json` — pass/fail, pass_rate, per-task reasons
- `results/<run_id>/REPORT.md` — human summary (via `report`)
- `traces/<run_id>/<task_id>/` — `run_agent.log`, `trace.jsonl`, `verifier_result.json`

## CLI reference (agents)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [am423/hermes-bench-tool-call](https://github.com/am423/hermes-bench-tool-call) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
