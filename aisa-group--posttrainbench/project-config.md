---
trigger: always_on
description: Guidelines for AI agents working on the PostTrainBench codebase.
---

# AGENTS.md

Guidelines for AI agents working on the PostTrainBench codebase.

## Project Overview

PostTrainBench is a benchmark framework that measures AI agents' ability to perform **LLM post-training** - improving pre-trained small language models on specific benchmarks through automated research. Agents are typically given 10 hours on an H100 GPU to improve base models, with performance measured by benchmark scores.

## Directory Structure

```
PostTrainBench/
├── agents/              # Agent implementations (claude, codex, gemini, opencode, qwen3max, glm5, ...)
├── cluster/             # Cluster-specific helper artifacts
├── containers/          # Apptainer/Singularity container definitions and helpers
├── dev_utils/           # Development/debugging utilities (run failure triage, trace extraction, ...)
├── scripts/             # Result aggregation and analysis
├── src/
│   ├── baselines/             # Baseline score computation
│   ├── commit_utils/          # HTCondor job submission utilities (incl. set_env_vars.sh)
│   ├── judges/                # Reward-hacking judges (one folder per judge; see Safety)
│   ├── eval/
│   │   ├── general/           # Prompt generation (get_prompt.py, prompt.txt)
│   │   ├── tasks/             # Evaluation benchmarks (aime2025, aime2026, gsm8k, ...)
│   │   └── templates/         # Chat templates (Jinja2)
│   ├── trace_parsing/         # Per-agent trace parsers (claude/codex/gemini/opencode)
│   ├── utils/                 # Utility scripts (check_cuda, system_monitor, timestamp_lines, ...)
│   └── run_task.sh            # Main task execution orchestrator
└── results/             # Evaluation results storage (path controlled by POST_TRAIN_BENCH_RESULTS_DIR)
```

## Key Files

| File | Purpose |
|------|---------|
| `src/run_task.sh` | Main task execution orchestrator (runs agent, then 4 judges, then evaluation) |
| `src/commit_utils/commit.sh` | Batch job submission across agents × benchmarks × models |
| `src/commit_utils/set_env_vars.sh` | Sources `.env` and exports `POST_TRAIN_BENCH_*` env vars |
| `src/commit_utils/single_task.sub` | HTCondor submission template |
| `src/commit_utils/single_task_gemini.sub` | Gemini-specific HTCondor submission template |
| `src/eval/general/get_prompt.py` | Generates agent prompts |
| `src/eval/general/prompt.txt` | Agent prompt template |
| `src/trace_parsing/parse_trace.py` | Dispatches to per-agent parser to produce human-readable trace |
| `src/utils/update_agent_cli.sh` | Auto-updates an agent's CLI harness to latest and records its version |
| `src/judges/run_judges.sh` | Runs judges on an existing result dir (`--judges` to select a subset); each writes its own per-judge JSON |
| `src/judges/get_judge_prompt.py` | Generates judge prompts (`--judge <judge_name>`) |
| `containers/standard.def` | Main container definition (other `.def` files exist per-agent) |
| `scripts/constants.py` | Agent/benchmark mappings |
| `example.env` | Template for the `.env` file (API keys + `POST_TRAIN_BENCH_*` paths) |

## Adding a New Agent

1. Create directory: `agents/<agent_name>/`
2. Create entry point: `agents/<agent_name>/solve.sh`
3. The `solve.sh` script receives the system prompt via the `$PROMPT` environment variable
   (also some agents take it as a CLI arg). It runs inside the apptainer sandbox with `/home/ben/task`
   as the working directory and should produce `final_model/` there when finished.
4. Add an agent-specific submission template if needed: `src/commit_utils/single_task_<agent>.sub`
5. Per-agent parsing: add a parser in `src/trace_parsing/<agent>_parser.py` and register it in
   `parse_trace.py` so `solve_parsed.txt` is human-readable.
6. If the agent needs persistent OAuth state (e.g. `codex_non_api`), drop an `auth.json` or
   `oauth_token` next to `solve.sh`; `run_task.sh` bind-mounts them into the sandbox.
7. Auto-update the CLI harness: call `bash /home/ben/update_agent_cli.sh <cli-binary>` in
   `solve.sh` just before launching the CLI (e.g. `... update_agent_cli.sh claude`). This upgrades
   the harness to the latest npm release and writes its version to `cli_version.txt` (surfaced in
   the result dir). The helper (`src/utils/update_agent_cli.sh`, copied into the sandbox by
   `run_task.sh`) holds the binary→npm-package mapping; add a `case` entry there if the agent uses
   a CLI not already covered (`claude`, `codex`, `gemini`, `opencode`). The update is best-effort —
   a failure falls back to the container's pinned version and still records what actually ran.
   Set `POST_TRAIN_BENCH_SKIP_CLI_UPDATE=1` in `.env` to disable the update globally and pin CLI
   versions to whatever the container ships; `cli_version.txt` still records what ran
   (`update: skipped`).

Example structure:
```bash
#!/bin/bash
# agents/myagent/solve.sh
myagent-cli --model "$AGENT_CONFIG" "$PROMPT"
```

**Required: `agents/<agent_name>/api_keys.json`** declares the third-party API keys the agent is
allowed to receive, e.g. `{"allowed_api_keys": ["CODEX_API_KEY"]}` (use `[]` for subscription-auth
agents like `codex_non_api`). `run_task.sh` passes ONLY these keys — plus any the benchmark
requires — into the `-c --cleanenv` sandbox; every other provider key is never injected (so it is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aisa-group/PostTrainBench](https://github.com/aisa-group/PostTrainBench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
