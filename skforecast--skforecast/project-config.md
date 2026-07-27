---
trigger: always_on
description: Use when writing, updating, or reviewing docstrings in skforecast source code. Covers NumPy-style format, section order, parameter/return formatting, type annotations, deprecation notices, version tags, and cross-reference conventions.
---

# Skforecast Docstring Guidelines

## Format

NumPy-style docstrings. Every public class and public method/function must have a docstring.

## Before Modifying a Docstring

Before writing or modifying a docstring, **read the existing docstrings** in the same file and the neighboring parameters to match the established style exactly. Do not reformat existing content that you are not changing.

## Common Mistakes — Do NOT

These are the most frequent errors. Violating any of these rules is always wrong in skforecast.

| Wrong | Correct | Why |
|-------|---------|-----|
| ` ``True`` ` (double backticks) | `` `True` `` (single backticks) | Skforecast never uses rST double-backtick literals |
| `pd.Series` | `pandas Series` | Docstring types use readable names, not aliases |
| `pd.DataFrame` | `pandas DataFrame` | Same as above |
| `np.ndarray` | `numpy ndarray` | Same as above |
| `:class:\`OrdinalEncoder\`` | `OrdinalEncoder` | No rST cross-reference directives |
| Adding `Raises` section | *(omit it)* | Skforecast docstrings never include Raises |
| Adding `Warnings` section | *(omit it)* | Skforecast docstrings never include Warnings |
| Adding `See Also` section | *(omit it)* | Skforecast docstrings never include See Also |
| Adding `Yields` section | *(omit it)* | Skforecast docstrings never include Yields |

**Additional rules:**
- NEVER use double backticks (`` `` ``) for any inline code or value. Always single backticks.
- NEVER add docstrings, comments, or type annotations to code you did not change.
- NEVER invent parameters or attributes that do not exist in the actual code.
- NEVER use en dashes (–) or em dashes (—) when generating code comments, docstrings, and documentation. Use commas, colons, semicolons, or parentheses for punctuation instead.

## Section Order

Follow this exact section order (omit sections that don't apply):

1. **Summary** — one-line or short paragraph
2. **Parameters** — constructor or function arguments
3. **Attributes** — class-level only (after Parameters in classes)
4. **Returns** — what the method/function returns
5. **Notes** — implementation details, caveats, behavioral notes
6. **References** — numbered references using `.. [1]` syntax

## Summary

- First line: concise description of the class/method purpose.
- Separated from sections by a blank line.
- For classes: describe what the class does, not how to use it.
- For methods: describe what the method does, starting with a verb (e.g., "Training Forecaster.", "Predict n steps ahead.").

## Parameters Section

```python
Parameters
----------
y : pandas Series
    Training time series.
exog : pandas Series, pandas DataFrame, default None
    Exogenous variable/s included as predictor/s. Must have the same
    number of observations as `y` and their indexes must be aligned.
steps : int, str, pandas Timestamp
    Number of steps to predict. 

    - If steps is int, number of steps to predict. 
    - If str or pandas Datetime, the prediction will be up to that date.
```

### Rules

- **Type line format**: `name : type[, type[, ...]][, default value]`
- **Default values**: written as `default None`, `default True`, `default 123`, `default 'auto'` — always on the type line, not in the description.
- **Description indentation**: 4 spaces from the left margin (one level deeper than the parameter name).
- **Sub-items** (enumerated options): insert a blank line between the description and the first bullet. Bullets use the same indentation as the description text. Continuation lines for a bullet align with the dash (`-`), **not** indented further to align with the text after the dash. No blank lines between consecutive bullets.

  Correct example (inside a class docstring, 4-space base indent from `"""`):
  ```
      encoding : str, None, default 'ordinal'
          Encoding used to identify the different series.
  ​
          - If `'ordinal'`, a single column is created with integer values from 0
          to n_series - 1.
          - If `'onehot'`, a binary column is created for each series.
          - If None, no column is created to identify the series. Internally, the
          series are identified as an integer from 0 to n_series - 1, but no column
          is created in the training matrices.
  ```
  Notice: continuation line `to n_series - 1.` starts at the same column as the `-` dash, not at the column of the text after `- `.

- **Backticks**: always single backticks — never double. Use for parameter names, values, and attribute references (`y`, `None`, `self.last_window_`, `True`, `False`).
- **Multi-line descriptions**: continuation lines align with the first line of the description (same indent level as description start).
- **Type naming conventions** (critical — these are the most common source of errors):
  - `pandas Series`, `pandas DataFrame` — NEVER `pd.Series` or `pd.DataFrame`
  - `numpy ndarray` — NEVER `np.ndarray`
  - `str`, `int`, `float`, `bool`, `dict`, `list`, `tuple`, `Callable`, `object`
  - Union types separated by commas: `int, list, numpy ndarray, range`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skforecast/skforecast](https://github.com/skforecast/skforecast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
