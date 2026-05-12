---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**LiveClawBench** is a benchmark suite for evaluating LLM agents on complex, real-world assistant
tasks. The core scientific question: how does agent capability degrade when tasks stack multiple
complexity factors along three axes (Environment Complexity, Cognitive Demand, Runtime Adaptability)?

### Three-Repo Architecture

| Repository | Role | URL |
|---|---|---|
| **LiveClawBench** (this repo) | Task corpus — 30 harbor-format benchmark tasks | — |
| **claw-harbor** | Evaluation framework (fork of harbor with OpenClaw support) | https://github.com/Mosi-AI/claw-harbor |
| **OpenClaw** | Agent platform running inside task containers | https://github.com/openclaw/openclaw |

The agent under evaluation is **OpenClaw**. Harbor orchestrates Docker containers, launches the
OpenClaw agent inside each container, and verifies the result. LiveClawBench provides the tasks.

## Local Worktrees

When creating additional Git worktrees for agent-driven branch work, place them under
`./.worktrees/<branch-name>` at the repository root.

- Do not create worktrees under `./.claude/`
- Keep `./.claude/` for Claude-specific local settings/session data only
- Treat `./.worktrees/` as disposable local workspace state and keep it untracked

## Setup

### Quick Setup (recommended)

```bash
# From the LiveClawBench/ directory:
./setup.sh
```

`setup.sh` will:
1. Check prerequisites (git, uv, Docker, Python ≥ 3.12, Bun)
2. Create a local `.venv` and install `harbor` from the claw-harbor GitHub URL
3. Build the shared `liveclawbench-base:latest` Docker image
4. Build Bun mock binaries and per-task Docker images
5. Copy `.env.example` → `.env` for API key configuration

Then activate the venv before running any `harbor` commands:

```bash
source .venv/bin/activate
```

### Manual Setup

```bash
uv venv .venv
source .venv/bin/activate
uv pip install "harbor @ git+https://github.com/Mosi-AI/claw-harbor.git@v0.1.0"
```

### API Key Configuration

Edit `.env` and uncomment the block for your provider. Agent credentials are injected at runtime via `--ae`:

| Provider | Model format | Key env var |
|---|---|---|
| VolcEngine | `volcengine/` or `volcengine-plan/<model-id>` | `VOLCANO_ENGINE_API_KEY` |
| Anthropic | `anthropic/<model-id>` | `ANTHROPIC_API_KEY` |
| OpenAI | `openai/<model-id>` | `OPENAI_API_KEY` |
| Gemini | `gemini/<model-id>` | `GEMINI_API_KEY` |
| Any OpenAI-compatible | `custom/<model-id>` | `CUSTOM_API_KEY` + `CUSTOM_BASE_URL` (+ optional `CUSTOM_CONTEXT_WINDOW` / `CUSTOM_MAX_TOKENS` / `CUSTOM_REASONING` / `CUSTOM_API`) |

## Running Tasks

### Single Task

```bash
# Activate venv first (or prefix with .venv/bin/harbor)
source .venv/bin/activate

# Generic form — works with any OpenAI-compatible endpoint
harbor run -p tasks/<task-name> -a openclaw \
  -m custom/<YOUR_MODEL_ID> \
  -n 1 -o jobs \
  --ae CUSTOM_BASE_URL="<YOUR_BASE_URL>" \
  --ae CUSTOM_API_KEY="<YOUR_API_KEY>" \
  --timeout-multiplier 2.0 --debug

# Example: VolcEngine (explicitly registered in the openclaw adapter)
harbor run -p tasks/watch-shop -a openclaw \
  -m volcengine-plan/kimi-k2.5 \
  -n 1 -o jobs \
  --ae VOLCANO_ENGINE_API_KEY="$VOLCANO_ENGINE_API_KEY" \
  --debug
```

### Full Dataset

```bash
# Generic form (includes LLM judge credentials for the 5 judge tasks)
harbor run --dataset liveclawbench@0.1.0 -a openclaw \
  -m custom/<YOUR_MODEL_ID> \
  --n-concurrent 4 \
  -o jobs \
  --ae CUSTOM_BASE_URL="<YOUR_BASE_URL>" \
  --ae CUSTOM_API_KEY="<YOUR_API_KEY>" \
  --ee JUDGE_BASE_URL="<JUDGE_BASE_URL>" \
  --ee JUDGE_API_KEY="<JUDGE_API_KEY>"

# Example: Anthropic
harbor run --dataset liveclawbench@0.1.0 -a openclaw \
  -m anthropic/claude-opus-4-1 \
  --n-concurrent 4 \
  --ae ANTHROPIC_API_KEY="$ANTHROPIC_API_KEY" \
  --ee JUDGE_BASE_URL="<JUDGE_BASE_URL>" \
  --ee JUDGE_API_KEY="<JUDGE_API_KEY>"
```

### Check Results

```bash
cat jobs/*/*/verifier/reward.txt   # 1.0 = solved, 0.5 = partial credit
```

### Common Flags

| Flag | Purpose |
|---|---|
| `-p tasks/<name>` | Run a specific task directory |
| `-a openclaw` | Use OpenClaw as the agent |
| `-m <provider>/<model>` | Model to evaluate |
| `-n <int>` | Number of attempts per task |
| `-o jobs` | Output directory for job results |
| `--ae KEY=VALUE` | Pass environment variable into the **agent process** only (via `openclaw.json`); repeatable |
| `--ee KEY=VALUE` | Pass environment variable into the **container environment** (agent + verifier both see it); repeatable |
| `--timeout-multiplier 2.0` | Scale all `task.toml` timeouts (useful for hard tasks) |
| `--debug` | Verbose logging |
| `--n-concurrent <int>` | Parallel task execution |

> **LLM-judge tasks** (5 tasks: `conflict-repair-acb`, `incremental-update-ctp`,
> `live-web-research-sqlite-fts5`, `mixed-tool-memory`, `noise-filtering`) use `--ee` (not `--ae`)
> for judge credentials because `llm_judge.py` runs in the verifier phase, outside the OpenClaw
> agent process. **Missing `--ee` will cause the verifier to fail with
> `RuntimeError: JUDGE_BASE_URL is not set`.**
>
> Required vars: `JUDGE_BASE_URL`, `JUDGE_API_KEY`;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mosi-AI/LiveClawBench](https://github.com/Mosi-AI/LiveClawBench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
