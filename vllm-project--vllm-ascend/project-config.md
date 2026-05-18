---
trigger: always_on
description: This document provides instructions for contributors to the vLLM Ascend project. Please read and follow these guidelines to ensure code quality, maintainability, and consistency.
---

# vLLM Ascend Development Guidelines

This document provides instructions for contributors to the vLLM Ascend project. Please read and follow these guidelines to ensure code quality, maintainability, and consistency.

---

## Table of Contents

- [Setup and Environment](#setup-and-environment)
    - [Environment Variables](#environment-variables)
    - [Environment Variable Review Requirement](#environment-variable-review-requirement)
- [Testing](#testing)
    - [Unit and System Tests](#unit-and-system-tests)
    - [Running Tests](#running-tests)
- [Code Style](#code-style)
    - [Python Conventions](#python-conventions)
    - [Naming Conventions](#naming-conventions)
- [NPU-Specific Considerations](#npu-specific-considerations)
    - [Tensor item() Operations](#tensor-item-operations)
    - [Memory and Performance](#memory-and-performance)
- [Model and Plugin Architecture](#model-and-plugin-architecture)
    - [vLLM Ascend Plugin Architecture](#vllm-ascend-plugin-architecture)
    - [Patching Requirement](#patching-requirement)
    - [Model Runner Changes](#model-runner-changes)
- [Commit Messages and Pull Requests](#commit-messages-and-pull-requests)
    - [Commit Message Format](#commit-message-format)
- [Review Checklist](#review-checklist)
    - [Code Quality](#code-quality)
    - [Testing](#testing-1)
    - [Documentation](#documentation)
    - [NPU Considerations](#npu-considerations)
    - [Commit and PR](#commit-and-pr)
- [Quick Start for Contributors](#quick-start-for-contributors)
- [References](#references)

---

## Setup and Environment

### Environment Variables

All environment variables must be defined in `vllm_ascend/envs.py` using the centralized `env_variables` dictionary.

**Requirements:**

- Add documentation for each environment variable in the `env_variables` dict comment
- Specify default values and valid ranges
- Indicate whether the variable is sensitive (credentials, keys)

**Example:**

```python
import os

env_variables = {
    "VLLM_ASCEND_ENABLE_NZ": lambda: int(os.getenv("VLLM_ASCEND_ENABLE_NZ", 1)),
    # ...
}
```

**Never**: Hardcode environment variable names throughout the codebase. Reference them from the central module using `from vllm_ascend import envs`.

### Environment Variable Review Requirement

**Strict Review Required**: All new environment variables must undergo code review.

Reviewers must verify:

- The variable name follows the `VLLM_ASCEND_*` naming convention
- Default value is appropriate for all supported hardware
- Documentation is added to the `env_variables` dict
- The variable is used in a performance-critical path

---

## Testing

### Unit and System Tests

**Requirement**: All new functionality requires corresponding tests.

- **Unit Tests (UT)**: Located in `tests/ut/`, cover core logic, edge cases, and error conditions
- **System Tests (ST)**: Located in `tests/e2e/`, verify end-to-end behavior and integration points
- **Nightly Tests**: Include benchmarks for NPU-specific code paths in `tests/e2e/nightly/`

**Test Coverage Guidelines:**

- New features: Tests must cover happy path and failure modes
- Bug fixes: Tests must include a regression test for the bug
- Performance-critical code: Include benchmarks and performance regression tests

### Running Tests

```bash


# Run specific unit test file
pytest -sv tests/ut/ops/test_prepare_finalize.py

# Run specific unit test
pytest -sv tests/ut/ops/test_prepare_finalize.py::test_prepare_inputs

# Run NPU-specific tests (requires NPU hardware)
pytest -sv tests/e2e/singlecard/test_piecewise_res_consistency.py
```

**Requirement**: Run all tests locally before requesting review. Verify tests pass on NPU hardware for NPU-specific changes.

---

## Code Style

### Python Conventions

- **Imports**: All imports at the top of the file. Valid exceptions:
    - Circular imports (use inline imports)
    - Lazy loading for worker/isolation processes
    - Type-checking imports wrapped in `if TYPE_CHECKING:`

- **Global Variables**: Avoid new global variables. Pass dependencies explicitly through function parameters.

    **Allowed:**
    - Constants named `ALL_UPPER_CASE` (e.g., `MAX_BATCH_SIZE` in `envs.py`)
    - Immutable configuration objects

    **Requires Approval:**
    - Any new mutable global state

- **No Magic Numbers**: Use named constants with descriptive names:

    ```python
    # Bad
    if seq_len > 2048: ...

    # Good
    MAX_CONTEXT_LENGTH = 2048
    if seq_len > MAX_CONTEXT_LENGTH: ...
    ```

- **Descriptive Naming**: Use names that describe functionality, not implementation details.

    ```python
    # Bad
    is_deepseek_v3_r1
    flag1
    tmp_var

    # Good
    supports_dynamic_temperature
    uses_speculative_decoding
    ```

### Naming Conventions

- **Classes**: `PascalCase` (e.g., `NPUModelRunner`, `AscendSampler`, `ACLGraphManager`)
- **Functions/Methods**: `snake_case` (e.g., `forward_pass`, `compute_attention`)
- **Constants**: `ALL_UPPER_CASE` (e.g., `MAX_BATCH_SIZE`, `VLLM_ASCEND_ENABLE_NZ`)
- **Variables**: `snake_case` (e.g., `token_ids`, `sequence_lengths`)

---

## NPU-Specific Considerations

### Tensor item() Operations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vllm-project/vllm-ascend](https://github.com/vllm-project/vllm-ascend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
