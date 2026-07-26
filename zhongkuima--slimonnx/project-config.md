---
trigger: always_on
description: > **Law**. Conventions/rules for slimonnx. Code obeys this. Change via deliberate revision.
---


> **Law**. Conventions/rules for slimonnx. Code obeys this. Change via deliberate revision.

# Slimonnx Conventions

This file defines style and documentation conventions for the slimonnx package.
Use it as a **checklist** — when writing or reviewing code, check each item below
one by one.

---

## 1. Module Docstrings

Every `.py` file begins with a module docstring.

### Rules

| # | Rule | Pass/Fail |
|---|------|-----------|
| 1.1 | **First line**: short summary of the module's purpose (one sentence) | ☐ |
| 1.2 | **Extended description** (optional): 1-2 paragraphs after a blank line, covering the module's role in the optimization pipeline | ☐ |
| 1.3 | **Format**: ReST plain text, no `:param:` or `:return:` tags at module level | ☐ |
| 1.4 | Always followed by `__docformat__ = "restructuredtext"` | ☐ |
| 1.5 | **No author, date, or version lines** — git history is authoritative | ☐ |
| 1.6 | **No non-ASCII characters** in docstrings — use ASCII equivalents | ☐ |

### Patterns

| File type | Style | Example |
|-----------|-------|---------|
| Operation module (`_bn_conv.py`, `_gemm.py`) | One line | `"""Fuse BatchNormalization with Conv and ConvTranspose operators."""` |
| Pipeline module (`_optimize.py`) | One line | `"""Main optimization orchestration for ONNX models."""` |
| Registry module (`registry.py`) | One line | `"""Pattern detection registry."""` |
| Utility module (`utils.py`) | One line | `"""Utility functions for ONNX model manipulation and analysis."""` |
| Constants module (`constants.py`) | Summary + paragraph | `"""Package-level constants... Holds ONNX-spec defaults..."""` |
| Entry point (`slimonnx.py`) | One line | `"""SlimONNX: ONNX model optimization and analysis toolkit."""` |
| `__init__.py` | One line describing the subpackage | `"""Pattern detection for ONNX model optimization."""` |

---

## 2. Function/Class Docstrings

### 2.1 Structure

```python
def func_name(param1: type, param2: type) -> return_type:
    """
    Short imperative description of what the function computes.

    Extended description (optional) — the algorithm or pipeline step.

    :param param1: Description of param1 (capitalized, ends with period).
    :param param2: Description of param2.

    :return: Description of return value(s) (capitalized, ends with period).
    :raises ValueError: When and why this exception is raised.
    """
```

### 2.2 Rules

| # | Rule | Pass/Fail |
|---|------|-----------|
| 2.1 | **First line**: imperative mood, describes what the function computes, ends with period | ☐ |
| 2.2 | Use `:param name:`, `:return:`, and `:raises ExceptionType:` tags — no `:type:` tags | ☐ |
| 2.3 | `:param` descriptions: **capitalized, end with period**, describe semantics not types | ☐ |
| 2.4 | `:return` description: **capitalized, end with period** | ☐ |
| 2.5 | Private helpers (`_apply_*`, `_check_*`) may use a single-line docstring without `:param:` tags | ☐ |
| 2.6 | Public API functions (`optimize_onnx`, `slim`, `detect_all_patterns`) require full `:param:` documentation for every parameter | ☐ |
| 2.7 | No docstring on `__init__` of a dataclass (the class docstring covers it) | ☐ |
| 2.8 | **No non-ASCII characters** in docstrings | ☐ |
| 2.9 | **No bold-header sections** in function docstrings — no `**Example**:`, `**Note**:`; body contains only description prose, `:param:`, `:return:`, `:raises:` | ☐ |

### 2.3 Good examples

```python
def slim(
    self,
    onnx_path: str,
    target_path: str | None = None,
    config: OptimizationConfig | None = None,
    validation: bool = False,
) -> dict | None:
    """
    Optimize ONNX model.

    The optimization pipeline:
    1. Load model
    2. Convert to Opset 21 for compatibility with shapeonnx
    3. Apply optimizations based on config
    4. Validate output if requested
    5. Save optimized model

    :param onnx_path: Path to input ONNX model.
    :param target_path: Path to save optimized model (default: {input}_simplified.onnx).
    :param config: Optimization configuration (default: OptimizationConfig()).
    :param validation: Whether to run validation comparing outputs.
    :return: Optimization report if validation enabled, else None.
    """
```

```python
def clear_onnx_docstring(model: ModelProto) -> ModelProto:
    """Remove all doc_string entries from nodes in the ONNX model.

    :param model: ONNX model to clean.
    :return: Model with all doc_strings cleared.
    """
```

---

## 3. Inline Comments

| # | Rule | Pass/Fail |
|---|------|-----------|
| 3.1 | Comment **why**, not what — the code already says what | ☐ |
| 3.2 | Only add comments when the reasoning is non-obvious (algorithm rationale, ordering constraints) | ☐ |
| 3.3 | **No inline shape comments** on function signatures — shapes belong in `:param:`/`:return:` docstrings | ☐ |
| 3.4 | No commented-out code — delete it | ☐ |
| 3.5 | `# TODO:` comments require an associated issue reference (enforced by ruff TD001) | ☐ |
| 3.6 | Section comments in large functions: `# Step 1: ...`, `# Step 2: ...` for numbered pipeline stages | ☐ |

---

## 4. Naming Conventions

| # | Rule | Pass/Fail |
|---|------|-----------|
| 4.1 | **Classes**: PascalCase — `SlimONNX`, `OptimizationConfig`, `ValidationConfig` | ☐ |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZhongkuiMa/slimonnx](https://github.com/ZhongkuiMa/slimonnx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
