---
trigger: always_on
description: Guidance for coding agents working in this repository. `CLAUDE.md` is a symlink to this file.
---

# AGENTS.md

Guidance for coding agents working in this repository. `CLAUDE.md` is a symlink to this file.

## Project Overview

Strands-SGLang is an SGLang model provider for the Strands Agents SDK with Token-In/Token-Out (TITO) support for on-policy agentic reinforcement learning training. It captures exact token IDs and logprobs during generation without retokenization drift, which is critical for accurate gradient computation in RL training.

## Commands

### Setup
```bash
uv sync              # installs the dev group by default
pre-commit install   # hook types come from default_install_hook_types
```

### Linting
```bash
pre-commit run --all-files   # what CI's lint job runs; the tools below are a subset
ruff check src/ tests/ examples/
ruff format --check src/ tests/ examples/
mypy src/strands_sglang
```

### Testing
```bash
# Unit tests (no server needed)
pytest tests/unit/ -v

# Single test file
pytest tests/unit/test_sglang.py -v

# Single test
pytest tests/unit/test_tool_parser.py::TestHermesToolParser::test_parse_single_tool_call -v

# Unit tests with coverage
pytest tests/unit/ -v --cov=src/strands_sglang --cov-report=html

# Integration tests (requires running SGLang server)
pytest tests/integration/ -v --sglang-base-url=http://localhost:30000
# Or via env var: SGLANG_BASE_URL=http://localhost:30000 pytest tests/integration/
```

## Architecture

The package lives in `src/strands_sglang/` with 7 core modules:

**SGLangModel** (`sglang.py`) - Main entry point implementing the Strands `Model` interface. Requires `client` and `tokenizer` (keyword-only). Formats messages using HuggingFace chat templates (`apply_chat_template()`), calls SGLang's `/generate` endpoint (non-streaming by design for RL throughput), tracks TITO trajectory, and parses tool calls. VLM support is auto-detected server-side via `SGLangClient.is_multimodal()` (queries `/model_info` for `has_image_understanding`, cached after the first call). When multimodal, `collect_image_data()` derives `image_data` (base64 data URLs) from the messages on every call and forwards them to SGLang — the server handles image token expansion. Configuration via `SGLangConfig` TypedDict (sampling_params, return_logprob, return_routed_experts, enable_thinking), which inherits `context_window_limit` from Strands' `BaseModelConfig` — set it, or conversation managers and `estimate_utilization()` fall back to a hardcoded 200k. `reset()` starts a new trajectory, banking the finished one in `rollout_history`.

**SGLangClient** (`client.py`) - Async HTTP client using aiohttp with connection pooling and aggressive retry (60 attempts by default, aligned with slime RL framework). All error classification is centralized in `_classify_http_error()`, which maps HTTP responses to custom exceptions (`SGLangContextLengthError`, `SGLangThrottledError`, etc.). Non-retryable errors: 401, 403, 404, context-length 400. Uses lazy session creation to avoid aiohttp's event-loop warnings.

**Utilities** (`utils.py`) - `lru_cache`-backed factories for shared client and tokenizer instances: `get_client()`, `get_client_from_slime_args()`, `get_tokenizer()`. Ensures connection pooling and tokenizer reuse across RL workers without explicit lifecycle management.

**Exceptions** (`exceptions.py`) - Custom exception hierarchy rooted at `SGLangClientError`. HTTP errors are classified into `SGLangHTTPError` (base), `SGLangContextLengthError` (400 + length patterns), and `SGLangThrottledError` (429/503). Connection failures become `SGLangConnectionError`, non-JSON responses become `SGLangDecodingError`. These exceptions form the contract between `client.py` and `sglang.py` — the model layer never inspects raw HTTP status codes.

**Rollout** (`rollout.py`) - Pydantic model for segment-based token accumulation for TITO. Tokens are appended via `add_prompt()` (loss_mask=0: system, user, tool results) and `add_response()` (loss_mask=1: model output), matching multi-turn conversation structure. `_add_segment()` is the single place that maintains the cross-field length invariant (`token_ids`/`loss_mask`/`logprobs` stay equal length). Exposes flat `token_ids`, `loss_mask`, `logprobs`, plus `routed_experts` and `image_data` lists and `segment_info` (`(is_output, length)` per segment). Also tracks routed experts (`add_routed_experts()`, `decode_routed_experts()`) and the `initial_prompt_length` property. The `SGLangModel` exposes the in-progress one as `model.rollout`; earlier ones (closed by `reset()`) live in `model.rollout_history`, so read `[*model.rollout_history, model.rollout]` for a whole episode. `image_data` is assigned from `collect_image_data()` on each successful call, not accumulated — it is derived from the messages so that re-running a turn cannot duplicate an image.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [strands-rl/strands-sglang](https://github.com/strands-rl/strands-sglang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
