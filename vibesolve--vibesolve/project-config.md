---
trigger: always_on
description: Guide for AI coding agents working in this repository.
---

# AGENTS.md

Guide for AI coding agents working in this repository.

## What this project is

`vibesolve` is a multi-agent pipeline that converts a free-text optimization
problem description (e.g. "schedule deliveries", "assign nurses to shifts") into a complete, runnable [Timefold Solver](https://timefold.ai/) Quarkus project, validated by Docker before it lands on disk.

A user writes a sentence; a sequence of LLM agents (parser → model builder →
constraint builder → IO → integrator, plus optional reviewer/fixer/user
validator) emits Java domain classes, constraint streams, REST endpoints,
`pom.xml`, tests, and `solverConfig.xml`. The output is a Maven project that
compiles and runs in a containerized JDK 17.

This is a **Python tool that generates Java projects** — do not confuse the
Python source under `src/vibesolve/` with the generated artifacts under
`results/`.

## Setup (one-time)

```bash
conda activate vibesolve            # ALWAYS activate first — required for every command
pip install -e ".[dev]"             # installs the package, CLI, and test deps (pytest)
cp .env.example .env.local          # then fill in OPENAI_API_KEY
docker build -t timefold-validator docker/   # pre-bakes Maven deps into the validator image
```

Python 3.11+ is required (modern type annotations). If `conda activate vibesolve` fails, create the env first: `conda create -n vibesolve python=3.11 -y`.

For the Anthropic/Claude provider, add `ANTHROPIC_API_KEY=...` to `.env.local` and pass `--provider claude` (or set it in `config.yaml`).

Make sure the Docker daemon is running before the `docker build` (Linux:
`sudo systemctl start docker`; macOS/Windows: launch Docker Desktop).

## CLI

There is a single `vibesolve` command (defined in `pyproject.toml [project.scripts]` as `vibesolve.cli.main:app`) with two subcommands. Run them from the repo root.

| Command | Purpose | Source |
|---|---|---|
| `vibesolve run [input.txt]` | Run the pipeline on one problem | `src/vibesolve/cli/run_single.py` |
| `vibesolve batch [files...]` | Parallel batch over `user_input/*.txt` | `src/vibesolve/cli/run_batch.py` |

`cli/main.py` is the entry point; it registers the two functions as subcommands.
Run `vibesolve --help` (or `vibesolve run --help` / `vibesolve batch --help`) for the full option list.

Flags shared by both subcommands:

- `--config path/to.yaml` — use a different config file (the root `config.yaml` auto-loads otherwise)
- `--provider openai|claude` — pick the LLM provider
- `--no-validation-loop` — skip the Docker validation/fixer loop entirely (prompt-debugging only)
- `--max-iterations N` — cap fixer retries
- `--serve` — on success, emit a portable `Dockerfile` + `docker-run.sh` into the generated project

`run` only:

- `--reasoning-effort low|medium|high` — overrides every agent's effort at once (per-agent defaults live in the `efforts:` config block; see below)
- `--user-validate` — pause after parsing to let the user review/correct the `ProblemSpec` interactively before code generation

`batch` only:

- `--workers N` — size of the parallel container pool
- `--input-dir DIR` — directory to scan for `*.txt` (default `user_input/`)
- Every batch ends with a benchmark table (Compiles · Solver runs · Quarkus runs · Endpoints work · Docker works · Cost · Tokens). Compiles/Solver/Cost/Tokens are free from pipeline results; Quarkus/Endpoints/Docker are measured by a serial post-batch Docker pass (`src/vibesolve/benchmarking/`). The Docker column needs `--serve` (else 0); `--no-validation-loop` skips the table entirely.

## Pipeline architecture

```
user_input/*.txt
   │
   ▼   Parser (gpt-5-mini)                            → ProblemSpec
   │
   ▼   [User Validator — Explain / Update]            ← --user-validate (optional, interactive)
   │
   ▼   Model Builder      → Delta → ProjectManifest   (domain classes + skeleton pom.xml)
   ▼   Constraint Builder → Delta → ProjectManifest   (ConstraintProvider)
   ▼   IO Agent           → Delta → ProjectManifest   (JsonIO)
   ▼   Integrator         → Delta → ProjectManifest   (Main, REST, solverConfig, tests, full pom.xml)
   │
   ▼   Reviewer            → Delta (pre-flight static fixes; on by default)
   ▼   Docker validate    (mvn clean compile  →  mvn exec:java [timeout 30s]  →  mvn test)
   │
   ├─ PASS  → write ProblemSpec.json, ProjectManifest.json, project dir + .zip
   └─ FAIL  → Fixer (gpt-5-mini, high effort) → re-validate, up to N iterations
```

Each agent except Parser/UserValidator returns a `Delta` (`changed_files`, `deleted_files`, optional `projectName`/`basePackage`, optional `explanation`), which is merged into the accumulated `ProjectManifest` by
`utils.patch_utils.apply_delta`. Agents emit only the files they changed — never the whole project — which keeps later-stage output small.

## Where things live

```
src/vibesolve/
├── agents/
│   ├── client.py          BaseAgentCaller + OpenAIAgentCaller + AnthropicAgentCaller + make_caller_factory
│   └── prompts.py         load_prompt() + _PROMPT_FILES (agent → filename map)
├── cli/                   main.py (entry point) + run_single.py (`run`) + run_batch.py (`batch`)
├── config/settings.py     AppSettings (pydantic-settings) + load_settings(yaml)
├── models/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vibesolve/vibesolve](https://github.com/vibesolve/vibesolve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
