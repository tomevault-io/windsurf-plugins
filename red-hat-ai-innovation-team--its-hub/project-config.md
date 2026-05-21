---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Installation and Setup
```bash
# Development installation with uv (recommended)
uv sync --extra dev

# Alternative: pip installation
pip install -e ".[dev]"

# Production installation
pip install its_hub
```

### Contribution
When commit or raising PR, never mention it is by ClaudeCode.
never say 🤖 Generated with [Claude Code](https://claude.ai/code)" in the commit statment, don't mention claude!

### Testing
```bash
# Run all tests
uv run pytest tests/

# Run specific test file
uv run pytest tests/test_algorithms.py

# Run tests with coverage
uv run pytest tests/ --cov=its_hub

# Run tests with verbose output
uv run pytest tests/ -v
```

### Code Quality
```bash
# Run linter checks
uv run ruff check its_hub/

# Fix auto-fixable linting issues
uv run ruff check its_hub/ --fix

# Format code with ruff
uv run ruff format its_hub/
```

### Git Workflow
```bash
# Create commits with sign-off
git commit -s -m "commit message"

# For any git commits, always use the sign-off flag (-s)
```

### Running Examples
```bash
# Test basic functionality
python scripts/test_math_example.py

# Benchmark algorithms (see script help for full options)
python scripts/benchmark.py --help
```

### IaaS Service (Inference-as-a-Service)
```bash
# Start IaaS service
uv run its-iaas --host 0.0.0.0 --port 8108

# Or using justfile (if available)
just iaas-start

# Check service health
curl -s http://localhost:8108/v1/models | jq .

# Configure the service (example: self-consistency algorithm)
curl -X POST http://localhost:8108/configure \
  -H "Content-Type: application/json" \
  -d '{"endpoint": "http://localhost:8100/v1", "api_key": "NO_API_KEY", "model": "your-model-name", "alg": "self-consistency"}'

# For comprehensive IaaS setup (multi-GPU, reward models, etc.), see docs/iaas-service.md
```

## Additional Tips
- Use `rg` in favor of `grep` whenever it's available
- Use `uv` for Python environment management: always start with `uv sync --extra dev` to init the env and run stuff with `uv run`
- In case of dependency issues during testing, try commenting out `reward_hub` and `vllm` temporarily in @pyproject.toml and retry.

## Architecture Overview

**its_hub** is a library for inference-time scaling of LLMs, focusing on mathematical reasoning tasks. The architecture separates public interfaces (`its_hub/api/`) from implementations (`its_hub/core/`).

### Directory Structure

```
its_hub/
├── __init__.py                 # Top-level exports (import from here)
├── api/                        # Public interfaces (stable API)
│   ├── lm.py                  # AbstractLanguageModel
│   ├── algorithm.py           # AbstractScalingAlgorithm, AbstractScalingResult
│   ├── orchestrator.py        # AbstractOrchestrator
│   ├── types.py               # ChatMessage, ChatMessages
│   ├── errors.py              # APIError, RateLimitError, etc.
│   └── reward_models/
│       ├── orm.py             # AbstractOutcomeRewardModel
│       └── prm.py             # AbstractProcessRewardModel
├── core/                       # Implementations (internal)
│   ├── algorithms/
│   │   ├── self_consistency.py
│   │   ├── bon.py
│   │   ├── beam_search.py     # Experimental
│   │   ├── particle_gibbs.py  # Experimental
│   │   └── planning_wrapper.py
│   ├── lms/
│   │   ├── openai_lm.py      # OpenAICompatibleLanguageModel
│   │   └── step_generation.py # StepGeneration
│   ├── reward_models/
│   │   ├── llm_judge.py       # LLMJudge
│   │   └── local_vllm_prm.py # LocalVllmProcessRewardModel
│   ├── orchestrator.py        # LMOrchestrator
│   └── utils.py               # System prompts, helpers
```

### Key Base Classes (`its_hub/api/`)
- `AbstractLanguageModel`: Interface for async LM generation (`agenerate()`, `agenerate_single()`)
- `AbstractScalingAlgorithm`: Base for all scaling algorithms with `ainfer()` (async) and `infer()` (sync wrapper)
- `AbstractScalingResult`: Base for algorithm results with `the_one` property returning a `dict`
- `AbstractOrchestrator`: Interface for managing parallel LM calls
- `AbstractOutcomeRewardModel`: Interface for outcome-based reward models
- `AbstractProcessRewardModel`: Interface for process-based reward models (step-by-step scoring)

### Main Components

#### Language Models (`its_hub/core/lms/`)
- `OpenAICompatibleLanguageModel`: Primary LM implementation supporting vLLM and OpenAI APIs. Supports async context manager (`async with`) and requires `close()` for cleanup.
- `StepGeneration`: Handles incremental generation with configurable step tokens and stop conditions
- Async-first design with concurrency limits and backoff strategies

#### Algorithms (`its_hub/core/algorithms/`)
All algorithms follow the same interface: `ainfer(lm, prompt_or_messages, budget, return_response_only=True, tools=None, tool_choice=None)` (async primary) or `infer(...)` (sync wrapper)

- **Self-Consistency**: Generate multiple responses, select most common answer (supports tool voting)
- **Best-of-N**: Generate N responses, select highest scoring via outcome reward model

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Red-Hat-AI-Innovation-Team/its_hub](https://github.com/Red-Hat-AI-Innovation-Team/its_hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
