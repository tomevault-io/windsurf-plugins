---
trigger: always_on
description: You are a coding agent setting up **Collider-Bench** for a researcher. The
---

# CLAUDE.md

You are a coding agent setting up **Collider-Bench** for a researcher. The
human will not run any of the commands below — you will. Read this top-to-
bottom, execute, and only ask the human when you hit a decision the file
doesn't cover.

## What you're setting up

A benchmark that scores LLM-agent recasts of CMS LHC analyses. Each task
gives an agent a published paper + a null-filled HEPData-style YAML; the
agent has to build a simulation + selection pipeline and fill predicted
bin yields. The harness scores the filled YAML against a hidden reference
and writes per-run artifacts under `runs/`.

**The HEP stack (MadGraph5, Pythia8, Delphes, Prospino) lives only inside
the container image** at `ghcr.io/dfaroughy/lhc-bench:latest`. The host
needs none of it — just Python + a container runtime + one agent CLI.

## Setup sequence (execute in order)

```bash
# 1. Venv + harness — no conda required
python3 -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"

# 2. Container engine — pick one that's available on this host
#    Mac:   brew install podman && podman machine init && podman machine start
#    Linux: docker / podman / apptainer are typically pre-installed
docker pull ghcr.io/dfaroughy/lhc-bench:latest
# (or: podman pull / apptainer pull docker://...)

# 3. One vendor agent CLI on the host (the sandbox bind-mounts it in)
npm i -g @anthropic-ai/claude-code   # claude → ~/.local/bin/claude
# alternatives: @openai/codex, @google/gemini-cli, aider, forge-code

# 4. Pick an auth mode (see "Auth modes" below) and configure it
#    — for `auth: api` (configs/claude.yaml default):
export ANTHROPIC_API_KEY=...      # claude
# OPENAI_API_KEY=, GEMINI_API_KEY=, DEEPSEEK_API_KEY= for other runners
#    — for `auth: oauth` (cheaper / subscription-funded):
# claude /login        # (or: codex /login, gemini auth login)
# Then change configs/claude.yaml's `auth: api` → `auth: oauth`.

# 5. Smoke test — should print run paths and exit 0 within ~20 min
scripts/run-agent --config configs/claude.yaml --task sus-16-046_sim-T5Wg
```

The smoke test runs the smallest task (`sus-16-046_sim-T5Wg`, ~4 bins) end
to end and writes a scored run under `runs/claude_opus-4-7/...`. If it
fails, see the pitfall table below before asking the human.

## Auth modes — `oauth` vs `api`

Each major vendor (Anthropic / OpenAI / Google) supports two paths. **Pick
the cheaper one (`oauth`) unless the user explicitly says otherwise.**

| Mode | What it uses | When to pick it | Caveats |
|---|---|---|---|
| `oauth` | The vendor CLI's own login session (`claude /login`, `codex /login`, `gemini auth login`) — usage gets billed to that account's subscription / free quota. | **Default for individual users.** A Claude / ChatGPT / Gemini subscription is ~$20–200/month flat and covers thousands of runs. Most researchers already have one. | Needs an interactive login on the host first. Refresh tokens rotate — the harness syncs them back to the host on exit ([sandbox.py:_sync_credentials_back_to_host](agent_runtime/sandbox.py)). Batch / headless / SLURM execution is fragile. |
| `api` | A direct API key (`ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GEMINI_API_KEY`, `DEEPSEEK_API_KEY`). | Production batch eval, SLURM jobs, CI, parallel sweeps, anywhere you need headless reliability.| No interactive login, just an exported env var. The validator ([config.py:validate_api_auth_env](agent_runtime/config.py)) fails fast if the key is missing, so misconfigured runs don't spend tokens. |

In configs:

```yaml
auth: oauth     # uses the existing CLI login on the host
# OR
auth: api       # requires the matching *_API_KEY env var
```

Both shipped configs (`configs/claude.yaml`, `configs/claude_slurm.yaml`)
default to `auth: api` for batch / SLURM convenience. **If you're setting
up for a single human researcher, switch the field to `oauth` and have them
run `claude /login` once.** That alone can reduce their per-run cost from
$10 to a few cents (subscription-amortized).

## Sanity checks (before the smoke test)

```bash
# Harness imports cleanly
python -c "import agent_runtime.config, agent_runtime.sandbox; print('OK')"

# Container engine is reachable
docker info >/dev/null && echo "docker OK"

# Image is local
docker images | grep lhc-bench

# Agent CLI is on PATH
command -v claude && claude --version

# Test suite (no SLURM, no LLM, ~100s)
python -m pytest -q
```

All five should pass before invoking `scripts/run-agent`.

## Repository layout

Three top-level layers — read code only when something below isn't enough.

| Layer | Path | What's in it |
|---|---|---|
| Benchmark | [`ColliderBench/`](ColliderBench/) | Tasks ([`tasks/`](ColliderBench/tasks/)), scorer ([`Evals/`](ColliderBench/Evals/)), agent-facing CLIs ([`tools/CLI/`](ColliderBench/tools/CLI/)), CLI shims ([`bin/`](ColliderBench/bin/)) |
| Harness | [`agent_runtime/`](agent_runtime/) | Entrypoint ([`launch.py`](agent_runtime/launch.py)), YAML loader+validator ([`config.py`](agent_runtime/config.py)), sandbox abstraction ([`sandbox.py`](agent_runtime/sandbox.py)), runner registry ([`vendors.py`](agent_runtime/vendors.py)), shell glue ([`shell/agent_env.sh`](agent_runtime/shell/agent_env.sh)) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dfaroughy/Collider-Bench](https://github.com/dfaroughy/Collider-Bench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
