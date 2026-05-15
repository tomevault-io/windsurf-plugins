---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

0xClaw is a general-purpose **autonomous hackathon agent** platform. Given a hackathon URL,
it autonomously runs a 7-phase pipeline on behalf of a human participant:

**Research → Ideation → Selection → Planning → Implementation → Testing → Documentation**

A single user can run 0xClaw against multiple hackathons in sequence. Each run produces
an independent set of gitignored artefacts under `workspace/hackathon/`.

The generated project (Layer 2) is created fresh per hackathon and lives at
`workspace/hackathon/project/` (gitignored at runtime).

---

## Environment — always do this first

```bash
conda activate 0xclaw          # Python 3.11, all deps installed
cp .env.example .env           # first time only; fill in real API keys
pip install -e .               # runtime deps only
pip install -e .[dev]          # includes ruff linter
./scripts/verify_setup.sh      # confirms runtime import + workspace + API keys
0xclaw                         # canonical runtime entrypoint
0xclaw --logs                  # launch with loguru output visible
0xclaw gateway                 # start Telegram/WhatsApp channel listeners
0xclaw whatsapp login          # one-time WhatsApp bridge auth (writes ~/.0xclaw/whatsapp-auth/)
```

`./scripts/start.sh` is an optional wrapper that activates conda and loads `.env` before running `0xclaw`.
`./scripts/verify_setup.sh` is a preflight checker, not the runtime entrypoint.

Gateway port and heartbeat are configured under `gateway.port` / `gateway.heartbeat` in `0xclaw/config/config.json`; `--port` overrides the configured port for one run.

### Test and lint

```bash
# Run all tests
python -m unittest discover -s tests -p "test_*.py" -v

# Run a single test file
python -m unittest tests.test_router -v

# Run a single test method
python -m unittest tests.test_router.KeywordMatchTests.test_exact_english_keyword_per_phase -v

# Lint
ruff check tests
```

Tests use Python `unittest` (not pytest). Each test file adds `0xclaw/` to `sys.path` and imports from `orchestration.*` directly.

CI runs on push/PR to master: `ruff check tests` + `python -m unittest discover` on Python 3.11 and 3.12.

### Required `.env` keys

| Key | Purpose | Notes |
|-----|---------|-------|
| `ZAI_API_KEY` | Primary LLM via Z.ai (international) | `zhipu` provider in config; model `glm-4.5` |
| `FLOCK_API_KEY` | Secondary LLM via FLock.io | HTTP 400 = budget exhausted |
| `BRAVE_API_KEY` | Web search | Optional |

---

## Two-layer architecture

```
Layer 1 — 0xClaw (the agent we maintain)
  0xclaw/main.py            CLI entry point, interactive REPL, slash commands
  0xclaw/orchestration/     Phase routing, state machine, write guards, model profiles
  0xclaw/config/            config.json (providers), model_profiles.json (per-phase settings)
  0xclaw/runtime/           Integrated agent runtime engine (modify carefully; prefer leaving core engine untouched unless a runtime bug fix is required)
  launcher/                 CLI entry point wrapper (resolves 0x hex-literal import issue)
  workspace/                Agent identity, skills, pipeline state

Layer 2 — Generated project (per-hackathon, gitignored)
  workspace/hackathon/project/    The built project (gitignored)
  workspace/hackathon/submission/ README, pitch, and submission docs (gitignored)
```

---

## Key files

| File | Purpose |
|------|---------|
| `0xclaw/main.py` | Entry point: CLI loop, AgentLoop wiring, slash commands, reset/resume/stop |
| `0xclaw/config/config.json` | Provider config (edit to set your LLM provider + API key). Env vars substituted at load time |
| `0xclaw/config/model_profiles.json` | Per-phase model + timeout overrides |
| `0xclaw/orchestration/state.py` | `PipelineStateStore`, `OrchestratorStateMachine`, `PHASE_ALLOWED_WRITE_DIRS` — phase deps, artifact requirements, and per-phase write allowlist |
| `0xclaw/orchestration/router.py` | `SkillRouter` — keyword + LLM fallback routing. Supports English and Chinese triggers |
| `0xclaw/orchestration/contracts.py` | `Envelope`, `ArtifactMeta` dataclasses for CLI → AgentLoop messages |
| `0xclaw/orchestration/model_profiles.py` | `ModelProfileResolver`, `MetricsLogger` |
| `0xclaw/orchestration/session_control.py` | `SessionControl` — `/resume` logic, `PHASE_TO_COMMAND` map |
| `0xclaw/orchestration/write_guard.py` | `install_phase_write_guards()` — phase-scoped filesystem write protection |
| `0xclaw/runtime/providers/registry.py` | Provider spec registry (safe to modify — add new providers here) |
| `0xclaw/runtime/config/schema.py` | `ProvidersConfig` Pydantic schema (safe to modify — add provider fields here) |
| `workspace/SOUL.md` | Agent identity and mission (loaded every turn) |
| `workspace/AGENTS.md` | 7-phase pipeline protocol (loaded every turn) |
| `workspace/skills/*/SKILL.md` | Spawn task templates — one per pipeline phase |

---

## Orchestration layer

The `0xclaw/orchestration/` package sits between `main.py` and the agent runtime.

**`SkillRouter`** maps free-form user input to a pipeline phase via keyword rules, with an

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xclaw-ai/0xClaw](https://github.com/0xclaw-ai/0xClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
