---
trigger: always_on
description: Guidance for coding agents working in this repository. `CLAUDE.md` is a symlink to this file.
---

# AGENTS.md

Guidance for coding agents working in this repository. `CLAUDE.md` is a symlink to this file.

## Project Overview

Strands-env is a framework for building **agent environments** with Strands Agents. An *agent environment* turns a Strands `Agent` into an RL environment whose unit of interaction is a full agent loop (prompt → tool calls → response), not a single model call — with token-level observation tracking (TITO), reward computation, and termination handling. Supports SGLang, Bedrock, Bedrock Mantle (GPT via the OpenAI Responses API), and OpenAI model backends.

## Commands

### Setup
```bash
uv sync                    # installs the dev group by default
uv sync --extra harbor     # plus an optional extra
```

### Linting
```bash
pre-commit run --all-files   # what CI's lint job runs; the individual tools below are a subset
ruff check src/ tests/ examples/
ruff format --check src/ tests/ examples/
mypy src/strands_env
```

### Testing
```bash
# Unit tests (no server needed)
pytest tests/unit/ -v

# Single test
pytest tests/unit/core/test_environment.py::TestRollout::test_successful_rollout -v

# Unit tests with coverage
pytest tests/unit/ -v --cov=src/strands_env --cov-report=html

# Integration tests (requires running SGLang server; model ID auto-detected via /model_info)
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

Holds the environment base, data types, model factories, and shared reward/tool primitives.

**types.py** — All data types. A `Task` is the flat, per-sample input to a rollout: `id`, `message`, `ground_truth`, `conversation_history`, `trace_attributes` (per-sample OTel span attributes, forwarded to the Agent; `extra="allow"` for ad-hoc transit); each typed environment ships a `Task` subclass with declared fields (`HarborTask`, `Tau2BenchTask`, ...). `TaskT` (`TypeVar` bound to `Task`, default `Task`) parameterizes `Environment`/`RewardFunction`/`Evaluator`. `RolloutResult` is the flattened output of one `rollout()`: `messages`, an optional `rollout` (token-level trajectory) for TITO training, `metrics`, an optional `reward_result` (`RewardResult`), `termination_reason`, and a `final_response` property. `RewardFunction` is the abstract base (async `compute(task, result) -> RewardResult`); `RewardResult` carries `reward` + `info`. `TerminationReason` maps agent exceptions to enum values via `from_error()` which walks exception cause chains: typed exceptions match first, then `from_keywords()` matches provider-specific failures (`TIMEOUT`, `CONNECTION_ERROR`, `AUTH_ERROR`, `THROTTLED`) by substring against the exception class name plus the AWS error code — boto surfaces most Bedrock failures as one uninformative `ClientError`, so the code carries the identity. Each reason's substrings live on its `keywords` property, and declaration order is match priority.

**models.py** — `ModelFactory = Callable[[], Model]` type and four factory functions (`sglang_model_factory`, `bedrock_model_factory`, `bedrock_mantle_model_factory`, `openai_model_factory`). Each returns a zero-arg lambda that creates a fresh Model instance per `rollout()` call for concurrent isolation. Bedrock and OpenAI remap `max_new_tokens` → `max_tokens` with a shallow dict copy to avoid mutating defaults; Bedrock Mantle remaps it to `max_output_tokens`. `bedrock_mantle_model_factory` builds an `OpenAIResponsesModel` for GPT models served via the Bedrock Mantle OpenAI Responses API: it passes `bedrock_mantle_config={"region": ...}` so the SDK derives the regional base URL (`openai.gpt-5.*` → `/openai/v1`, else `/v1`) and mints a fresh SigV4 token per request via `aws_bedrock_token_generator`, forwards a `reasoning` config, and leaves server-side conversation state at the SDK default (`stateful=False`, like the other stateless backends) so the full transcript is sent each turn and `agent.messages` stays intact for `Environment` observation capture (with `stateful=True` the SDK clears `agent.messages` for server-managed conversations and discards the trajectory). Requires `strands-agents[openai]>=1.46.0` (a main dependency, so no extra to install). Also home to the `ModelConfig` dataclass and `build_model_factory(config)` (a `match/case` dispatch over `sglang | bedrock | bedrock-mantle`), used by the eval CLI. (`openai_model_factory` exists but is not wired into `build_model_factory`.)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [strands-rl/strands-env](https://github.com/strands-rl/strands-env) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
