---
trigger: always_on
description: <!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->
---

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Aminx Copilot Instructions

These instructions guide Copilot in understanding the codebase, adhering to best practices, and contributing effectively to the "Aminx" project.

## 1. Project Overview and Goals

This project focuses on **implementing a functional interface for ProteinMPNN**, leveraging the **JAX** ecosystem for accelerated computation.

**Primary Goals:**
* **Increased Transparency:** Provide a clear and functional interface for ProteinMPNN, enabling users to easily understand and utilize the model.
* **JAX Compatibility:** Ensure all components are JAX-compatible, allowing for efficient computation and integration with JAX's functional programming paradigm.
* **Modular Design:** Maintain a modular structure to facilitate easy updates and extensions to the model.
* **Performance Optimization:** Utilize JAX's capabilities (e.g., JIT compilation, vectorization) to optimize performance for large-scale protein design tasks.


**Key Features:**


## 2. Core Principles & Development Practices

Adherence to these principles is paramount for maintaining code quality and consistency:

### A. JAX Idioms and Functional Programming
* **Prioritize JAX-compatible code:** All new code, especially numerical operations, should be written with JAX's functional programming paradigm in mind.
* **Immutability:** Favor immutable data structures (e.g., JAX PyTrees, `equinox.Module`, `flax.linen.Module`) where applicable.
* **JIT/Vmap/Scan Compatibility:** Ensure functions are compatible with JAX's `jit`, `vmap`, and `scan` transformations for performance.
* **Static Arguments:** Utilize `static_argnums` for function arguments that are not JAX types and do not change across JAX transformations (e.g., Python built-in types, strings, tuples of static values, or dataclasses acting as auxiliary data).
* **Documentation:** Use Google-style docstrings for all functions, including type hints and examples. Ensure that JAX transformations are clearly documented, especially when using `jit`, `vmap`, or `scan`.
* **Example  Docstring:**
```python
def example_function():
    """Test the example function with a specific input.
    Args:
        None
    Returns:
        None
    Raises:
        TypeError: If the output does not match the expected value.
    Example:
        >>> example_function()
    """
    result = example_function(input_data)
    expected = expected_output
    if not isinstance(result, expected_type):
        raise TypeError(f"Expected {expected_type}, but got {type(result)}")
```

### B. Code Quality & Linting (Ruff)
* **Linter:** Use **Ruff** for linting.
* **Configuration:** Adhere to the `ruff.toml` settings.
    * `select = ["ALL"]` (all rules enabled by default)
    * `ignore = ["PD"]` (pandas-specific rules are ignored)
    * **`line-length = 100`**: This is the primary target for line length. Ensure generated code adheres to this limit.
    * `indent-width = 2`
    * `fix = true` (Ruff's autofix capabilities should be utilized).
* **Execution:** Run `ruff check src/ --fix` regularly to apply automatic fixes.
* **Fix Failure Threshold:** **If automated `ruff --fix` attempts fail more than 5 times consecutively on the same set of issues, cease further attempts and flag the code for manual review by the user.**

### C. Type Checking (Pyright)
* **Evaluator:** Use **Pyright** for static type checking.
* **Strict Mode:** All code should pass Pyright in **strict mode**. Ensure type hints are accurate and comprehensive to satisfy strict type checking.

### D. Testing
* **Availability:** All new components and features *must* be accompanied by comprehensive **unit and/or integration tests**.
* **Framework:** Use `pytest` and where relevant`chex`
* **Test Location:** Place tests in the `tests/` directory, mirroring the structure of the source code (e.g., `tests/models/`, `tests/sampling/`, etc.).
* **Execution:** Tests are located in the `tests/` directory. Run tests with `uv run pytest tests/`.
* **Python Commands:** Always use `uv run python` instead of `python` or `python3` to ensure the correct environment is used.
* **Test Philosophy:** Tests should cover typical use cases, edge cases, and ensure correctness of JAX transformations where applicable.
* **Test Coverage:** Aim for high test coverage, especially for critical components like SMC steps, resampling methods, and scoring functions.
* **Test Structure:** Organize tests by functionality (e.g., `sampling/`, `scoring/`, `utils/`) and ensure they mirror the structure of the source code for clarity.
* **Test Documentation:** Use Google-style docstrings for tests, clearly describing the purpose, inputs, and expected outputs.
* **Example Test Docstring:**
```python
def test_example_function():
    """Test the example function with a specific input.
    Args:
        None
    Returns:
        None
    Raises:
        AssertionError: If the output does not match the expected value. 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maraxen/aminx](https://github.com/maraxen/aminx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
