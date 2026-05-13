---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

NeMo-Gym is NVIDIA's library for building RL training environments for LLMs (RLVR). It uses a microservice architecture with three composable FastAPI server types that communicate over async HTTP.

## Common Commands

```bash
# Setup
uv venv && uv sync --extra dev --group docs
pre-commit install

# Run servers
ng_run "+config_paths=[resources_servers/example_single_tool_call/configs/example_single_tool_call.yaml,responses_api_models/vllm_model/configs/vllm_model.yaml]"

# Run tests for a specific server (creates .venv per server, installs deps, runs pytest)
# First run is slow. Use skip_venv_if_present config or place a .venv to skip venv creation.
ng_test +entrypoint=resources_servers/example_single_tool_call

# Run all server tests
ng_test_all

# Run core library unit tests
pytest tests/unit_tests/ -x

# Run a single test file
pytest tests/unit_tests/test_openai_utils.py -x

# Lint and format
ruff check --fix .
ruff format .

# Pre-commit (runs ruff, formatting, custom hooks)
pre-commit run --all-files

# Collect rollouts
ng_collect_rollouts +agent_name=<agent> +input_jsonl_fpath=<data.jsonl> +output_jsonl_fpath=<output.jsonl> +num_repeats=5 "+responses_create_params={max_output_tokens: 16384, temperature: 1.0}"

# Profile results (compute per-task pass rates)
ng_reward_profile +input_jsonl_fpath=<data.jsonl> +rollouts_jsonl_fpath=<rollouts.jsonl> +output_jsonl_fpath=<profiled.jsonl> +pass_threshold=1.0

# Check server health
ng_status

# Dev test (runs pytest directly in server dir, no venv isolation)
ng_dev_test +entrypoint=resources_servers/example_single_tool_call

# Dump merged config
ng_dump_config "+config_paths=[...]"

# Dataset management (HF)
ng_upload_dataset_to_hf +dataset_name=<name> +version=<ver> +input_jsonl_fpath=<path> +hf_repo_id=<repo>
ng_download_dataset_from_hf +dataset_name=<name> +version=<ver> +output_jsonl_fpath=<path> +hf_repo_id=<repo>
```

## Architecture

Three server types, all FastAPI apps communicating via aiohttp:

- **Resources Servers** (`resources_servers/`): Implement `verify()` — task verification and reward computation. Return reward 0.0 or 1.0.
- **Response API Models** (`responses_api_models/`): Implement `chat_completions()` and `responses()` — LLM inference. Four variants: openai, azure_openai, vllm, local_vllm.
- **Response API Agents** (`responses_api_agents/`): Implement `responses()` and `run()` — orchestrate model-tool call loops. `simple_agent` is the default single-turn agent; others include `proof_refinement_agent` (multi-turn correction), `verifiers_agent`, `swe_agents`, etc.

A **HeadServer** coordinates all server lifecycles, config, and Ray cluster init.

### Base Class Hierarchy

```
BaseServer (Pydantic model with config + server_client)
└── SimpleServer (FastAPI app setup, middleware stack)
    ├── SimpleResourcesServer  →  implement verify()
    ├── SimpleResponsesAPIModel  →  implement chat_completions(), responses()
    └── SimpleResponsesAPIAgent  →  implement responses(), run()
```

### Data Flow

JSONL input → agent `/run` → model `/v1/responses` → (tool calls if any) → resources server `/verify` → reward → JSONL output

### Inter-Server Communication

`ServerClient` wraps aiohttp with retry logic (3 tries, exponential backoff). Session cookies propagate through the call stack for stateful environments. The global aiohttp client is a singleton with connection pooling.

## Configuration

Hydra + OmegaConf for hierarchical YAML composition. CLI overrides use `+key=value` syntax.

Each server instance is a top-level key in YAML that maps to a server type + config:
```yaml
my_server_instance:
  resources_servers:        # server type directory
    my_server:              # server subdirectory name
      entrypoint: app.py
      domain: coding
      # ... server-specific config fields
```

Agent configs reference their resource and model servers:
```yaml
my_agent_instance:
  responses_api_agents:
    simple_agent:
      entrypoint: app.py
      resources_server:
        type: resources_servers
        name: my_server
      model_server:
        type: responses_api_models
        name: policy_model
      datasets:
      - name: my_dataset
        type: train
        jsonl_fpath: path/to/data.jsonl
```

Model endpoint config goes in `env.yaml` at project root:
```yaml
policy_base_url: http://localhost:8000/v1
policy_api_key: your-key
policy_model_name: your-model
```

## JSONL Data Schema

Each line in input JSONL:
```json
{
  "responses_create_params": {
    "input": [
      {"role": "system", "content": "..."},
      {"role": "user", "content": "..."}
    ]
  },
  "verifier_metadata": { ... }
}
```

`responses_create_params.input` follows OpenAI message format. `verifier_metadata` is passed through to the resources server's `verify()` for task-specific validation data (test cases, expected answers, etc.).

Output JSONL (from `ng_collect_rollouts`) contains the full verify response per rollout, including at minimum:
```json
{
  "reward": 1.0,
  "response": { "output_text": "..." },
  "task_index": 0
}
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA-NeMo/Gym](https://github.com/NVIDIA-NeMo/Gym) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
