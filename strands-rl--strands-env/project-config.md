---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Strands-env is an RL environment abstraction for Strands agents — step, observe, reward. It provides a base `Environment` class that wraps a Strands `Agent` with token-level observation tracking (TITO), reward computation, and termination handling. Supports SGLang, Bedrock, OpenAI, and Kimi (Moonshot AI via LiteLLM) model backends.

## Commands

### Setup
```bash
pip install -e ".[dev]"
```

### Linting
```bash
ruff check src/ tests/ examples/
ruff format --check src/ tests/ examples/
mypy src/strands_env
```

### Testing
```bash
# Unit tests (no server needed)
pytest tests/unit/ -v

# Single test
pytest tests/unit/core/test_environment.py::TestStep::test_successful_step -v

# Unit tests with coverage
pytest tests/unit/ -v --cov=src/strands_env --cov-report=html

# Integration tests (requires running SGLang server; model ID auto-detected via /get_model_info)
# Tests skip automatically if server is unreachable (/health check)
pytest tests/integration/ -v --sglang-base-url=http://localhost:30000
# Or via env var: SGLANG_BASE_URL=http://localhost:30000 pytest tests/integration/
```

### Integration Tests with Remote GPU Server

```bash
# 1. Launch SGLang on the remote server in docker
ssh <remote-host> "sudo docker run -d --gpus all --name sglang-test -p 30000:30000 --ipc=host lmsysorg/sglang:<tag> python3 -m sglang.launch_server --model-path <model-id> --host 0.0.0.0 --port 30000 --tp <num_gpus> --mem-fraction-static 0.7"
# 2. Tunnel the port locally
ssh -L 30000:localhost:30000 -N -f <remote-host>
# 3. Run tests locally
pytest tests/integration/ -v
```

## Architecture

The package lives in `src/strands_env/` with these modules:

### `core/`

**types.py** — All data types. `Action` carries a user message + `TaskContext` (ground truth, conversation history, arbitrary metadata via `extra="allow"`). `Observation` holds messages, metrics, and optional `TokenObservation` for TITO training. `TerminationReason` maps agent exceptions to enum values via `from_error()` which walks exception cause chains. `StepResult` bundles observation + reward + termination reason.

**models.py** — `ModelFactory = Callable[[], Model]` type and four factory functions (`sglang_model_factory`, `bedrock_model_factory`, `openai_model_factory`, `kimi_model_factory`). Each returns a zero-arg lambda that creates a fresh Model instance per `step()` call for concurrent isolation. Bedrock, OpenAI, and Kimi remap `max_new_tokens` → `max_tokens` with a shallow dict copy to avoid mutating defaults. The Kimi factory targets Moonshot AI via LiteLLM (requires `MOONSHOT_API_KEY`) and uses a custom subclass that preserves `reasoning_content` for multi-turn conversations.

**environment.py** — Base `Environment` class. `EnvironmentConfig` TypedDict defines the serializable config shape (`system_prompt`, `max_tool_iters`, `max_tool_calls`, `max_parallel_tool_calls`, `verbose`). `__init__` takes `model_factory`, `reward_fn`, and `**config: Unpack[EnvironmentConfig]`. Subclass configs inherit from `EnvironmentConfig` to add env-specific keys. `step(action)` creates a fresh model via factory, attaches a `TokenManager`, builds an `Agent` with tools/hooks (always includes `ToolLimiter`), runs `invoke_async`, then collects metrics and optional reward. Subclasses override `get_tools()` and `get_hooks()` to customize. Messages are sliced so only new messages from the current step appear in the observation.

### `cli/`

**__init__.py** — CLI entry point with `strands-env` command group. Registers subcommand groups.

**eval.py** — Evaluation CLI commands: `strands-env eval list` shows registered/unavailable benchmarks, `strands-env eval run` executes benchmark evaluation. Env hooks are specified as dotted paths (`--env examples.eval.simple_math.calculator_env`). Environment config is passed as `--env-config` (inline JSON or path to JSON file) via custom `JsonType` click parameter. Supports distributed evaluation via `--n-actors-per-node` which creates an `EnvironmentActorPool` backed by Ray. `create_distributed_env_factory()` bridges CLI's `ModelConfig` to the eval hook contract for use inside Ray actors.

**models.py** — `SamplingParams` and `ModelConfig` dataclasses. `ModelConfig` includes `max_connections` for SGLang client pooling. `build_model_factory(config)` creates SGLang, Bedrock, or Kimi model factories with `match/case` dispatch.

### `eval/`

**evaluator.py** — `EvalSample` bundles step result with an `aborted` flag for checkpoint resume. `Evaluator` class orchestrates concurrent rollouts with JSONL checkpointing and pass@k metrics. Takes an `env_factory` (`AsyncEnvFactory`) for local evaluation or an `env_actor_pool` (`EnvironmentActorPool`) for distributed evaluation via Ray. Uses tqdm with `logging_redirect_tqdm` for clean progress output. Subclasses implement `load_dataset()` for different benchmarks and optionally override `validate_sample()` to mark failed samples as aborted (excluded from metrics, retried on resume).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [strands-rl/strands-env](https://github.com/strands-rl/strands-env) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
