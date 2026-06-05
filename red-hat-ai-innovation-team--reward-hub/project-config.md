---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RewardHub is a Python library for reward model annotation and evaluation, supporting both Process Reward Models (PRMs) and Outcome Reward Models (ORMs). The library implements a unified interface across multiple serving backends (HuggingFace, VLLM, OpenAI) and includes DrSow (Density Ratio of Strong-over-weak) functionality for preference annotation.

## Development Commands

### Installation
```bash
uv pip install -e .              # Basic installation
uv pip install -e .[dev]         # Development installation (includes pytest, ruff, pre-commit)
```

### Testing
RewardHub has a dual-mode test structure:

```bash
pytest                        # Run unit tests only (default, uses mocks, fast)
pytest -m unit                # Explicitly run unit tests
pytest -m e2e                 # Run end-to-end tests (requires GPU and real models)
pytest -m ""                  # Run all tests
```

Run specific test files:
```bash
pytest tests/unit/test_autorm.py              # Unit tests for AutoRM factory
pytest tests/e2e/test_hf_orm.py               # E2E HuggingFace ORM tests
pytest tests/e2e/test_vllm_prm.py             # E2E VLLM PRM tests
pytest tests/e2e/test_openai_drsow.py         # E2E DrSow tests
```

### Code Quality
```bash
ruff check .                  # Lint code
ruff format .                 # Format code
```

### Launching Models
Launch single reward model:
```bash
bash scripts/launch_reward.sh [model_path]
```

Launch DrSow (strong/weak model pair):
```bash
bash scripts/launch_vllm_drsow.sh [strong_model] [weak_model]
```

## Architecture

### Core Components

1. **AutoRM Factory** (`reward_hub/__init__.py`): Main entry point that validates model/backend compatibility and instantiates the appropriate reward model class
2. **Abstract Base Classes** (`reward_hub/base.py`):
   - `AbstractOutcomeRewardModel` / `AbstractProcessRewardModel`: Defines reward model interfaces
   - `PRMResult`: Encapsulates step-by-step scores with configurable aggregation (product, min, last, model)
3. **Backend Implementations**:
   - `reward_hub/hf/` - Direct local model loading with HuggingFace transformers
   - `reward_hub/vllm/` - Optimized local serving via VLLM
   - `reward_hub/openai/` - Remote API access for OpenAI-compatible endpoints
4. **DrSow Module** (`reward_hub/drsow.py`): Parallel multiprocessing for computing density ratios between strong/weak models
5. **LLM Judge Module** (`reward_hub/llm_judge/`): LiteLLM-based judges for conversation evaluation and ranking

### Model Support Matrix

Models and their supported backends are defined in `reward_hub/utils.py:SUPPORTED_BACKENDS`. The AutoRM factory validates this mapping at load time and raises an error if the model/backend combination is unsupported. Each model maps to specific reward model classes (e.g., `VllmProcessRewardModel`, `HuggingFaceOutcomeRewardModel`).

### Key Design Patterns

- **Factory Pattern**: `AutoRM.load()` uses model name and load method to select the correct class from `load_method_to_class` and `SUPPORTED_BACKENDS` mappings
- **Backend Abstraction**: All backends implement the same abstract interfaces, enabling seamless switching between HuggingFace, VLLM, and OpenAI
- **Unified Input Format**: All models accept OpenAI chat completion format (`List[dict]` with role/content keys)
- **PRM Aggregation**: Process reward models return `PRMResult` objects with configurable aggregation via `AggregationMethod` enum (product, min, last, model)
- **Parallel Processing**: DrSow uses Python multiprocessing to fetch logprobs from strong/weak models concurrently, storing results in a shared `Manager().dict()`

### Adding New Models

To add support for a new reward model:

1. **Update Model Registry** (`reward_hub/utils.py:SUPPORTED_BACKENDS`):
   ```python
   SUPPORTED_BACKENDS = {
       "your-org/your-model": [HuggingFaceOutcomeRewardModel],  # List supported classes
   }
   ```

2. **Implement Backend-Specific Logic** (if needed):
   - For HuggingFace: Extend classes in `reward_hub/hf/reward.py`
   - For VLLM: Extend classes in `reward_hub/vllm/reward.py`
   - For OpenAI: Extend classes in `reward_hub/openai/reward.py`

3. **Add Tests**:
   - Unit tests: `tests/unit/` (mocked, fast)
   - E2E tests: `tests/e2e/` (requires GPU, mark with `@pytest.mark.e2e`)

### Test Infrastructure

The test suite uses a dual-mode architecture controlled by `tests/conftest.py`:

- **Unit Mode** (default): `pytest_configure` hook mocks `transformers` and `vllm` imports before test collection, preventing actual model downloads
- **E2E Mode** (`pytest -m e2e`): Mocking is skipped, allowing real model loading for integration testing
- **Mock Behavior**: Configured in `conftest.py` to return realistic mock objects:
  - Models have `.eval()` and `.get_score()` methods
  - Tokenizers have `.apply_chat_template()`, `.encode()`, `.batch_decode()` with tensor-like return values
  - Mock templates include `.to()` method for device placement

When writing tests:
- Use fixtures from `conftest.py` for standard test data (e.g., `sample_single_turn_messages`, `sample_batch_messages`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Red-Hat-AI-Innovation-Team/reward_hub](https://github.com/Red-Hat-AI-Innovation-Team/reward_hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
