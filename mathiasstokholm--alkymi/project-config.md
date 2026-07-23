---
trigger: always_on
description: Alkymi is a pure Python (3.8+) library for describing and executing data processing pipelines with built-in caching
---

# alkymi – Repository Guide for AI Agents

## Overview

Alkymi is a pure Python (3.8+) library for describing and executing data processing pipelines with built-in caching
and conditional evaluation based on checksums. It allows users to write pipelines as decorated Python functions that
are automatically cached to disk. On subsequent runs, only the parts of the pipeline whose inputs have changed are
re-executed.

Key properties of the project:
- **No DSL** – pipelines are plain Python functions, so they work with linters, type checkers, and IDEs.
- **Automatic caching** – return values are serialized to disk regardless of type (supports numpy, pandas, arbitrary
  pickled objects, and plain JSON-serializable types).
- **Dirty-checking** – each recipe tracks checksums of its inputs and the hash of its own function body. A recipe is
  re-run only when something it depends on has changed.
- **CLI generation** – the `Lab` type turns a set of recipes into a Makefile-like command-line tool.
- **Cross-platform** – tested on Linux, Windows, and macOS.

---

## Repository Layout

```
alkymi/                     # Main library source package
  __init__.py               # Public API (re-exports recipe, foreach, Lab, recipes, utils, config)
  core.py                   # Graph construction and status computation engine
  recipe.py                 # Recipe[R] – the core building block
  foreach_recipe.py         # ForeachRecipe[R] – maps a function over a list/dict
  decorators.py             # @alk.recipe() and @alk.foreach() decorators
  lab.py                    # Lab – generates a CLI from a set of recipes
  recipes.py                # Built-in recipes: glob_files, file, arg, zip_results
  checksums.py              # Checksum computation (xxhash or MD5 fallback)
  serialization.py          # Disk serialization and deserialization of cached outputs
  config.py                 # AlkymiConfig singleton (global settings)
  types.py                  # Enums: Status, ProgressType, EvaluateProgress, CacheType, …
  utils.py                  # Helper: call() for subprocesses, run_on_thread()
  logging.py                # Logging helpers
  progress.py               # Rich-based progress visualization
  version.py                # __version__ string

tests/                      # pytest unit tests (one file per module)
  test_core.py
  test_caching.py
  test_serialization.py
  test_graph.py
  test_hashing.py
  test_foreach.py
  test_lab.py
  test_builtin_recipes.py
  test_utils.py

docs/                       # Sphinx documentation source
  source/
    getting_started/        # Installation, quickstart
    examples/               # MNIST, CLI, notebook
    advanced/               # Caching, checksums, sequences, configuration, execution
    api/                    # Auto-generated API reference

examples/                   # Standalone runnable examples
  mnist/                    # End-to-end MNIST dataset pipeline
  cli/                      # Command-line interface example
  notebook/                 # Jupyter notebook example

labfile.py                  # Alkymi's own build/test/lint recipes (self-hosting)
pyproject.toml              # Package metadata, dependencies, and dev dependency groups
requirements.txt            # Runtime dependencies: networkx, rich, markdown-it-py
mypy.ini                    # Mypy configuration
```

---

## Core Concepts

### Recipe

A `Recipe[R]` wraps a Python function and represents one step in a pipeline.

```python
import alkymi as alk

@alk.recipe()
def compute() -> int:
    return 42

result = compute.brew()   # executes, caches, returns 42
result = compute.brew()   # returns 42 from cache (no recomputation)
```

Important attributes:
- `ingredients` – upstream `Recipe` objects whose outputs are passed as arguments.
- `transient` – if `True`, the recipe is never cached and always re-run.
- `cache` – `CacheType.Cache`, `CacheType.NoCache`, or `CacheType.Auto`.

### ForeachRecipe

`ForeachRecipe[R]` applies a function to each element of a list or dict, caching results per element so that only
changed items are recomputed.

```python
@alk.foreach(list_recipe)
def process(item: str) -> str:
    return item.upper()
```

### Lab

`Lab` registers recipes and exposes them as a CLI tool, similar to a Makefile.

```python
lab = alk.Lab("my_project")
lab.add_recipes(recipe_a, recipe_b)
lab.open()   # parses sys.argv, e.g. `python labfile.py brew recipe_a`
```

### Status / Dirty-checking

Before executing, alkymi builds a DAG of all recipes and computes the `Status` of each node:

| Status | Meaning |
|--------|---------|
| `Ok` | Cached result is up-to-date |
| `NotEvaluatedYet` | No cached result exists |
| `IngredientDirty` | An upstream recipe needs recomputation |
| `InputsChanged` | Input checksums differ from the cached run |
| `BoundFunctionChanged` | The function's source code has changed |
| `OutputsInvalid` | The cached output is missing or corrupted |
| `CustomDirty` | A user-supplied cleanliness function returned `False` |

### Caching and Serialization

Outputs are stored under `.alkymi_cache/{module_name}/{recipe_name}/cache.json`.

Serialization tokens embedded in the JSON indicate how a value was stored:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MathiasStokholm/alkymi](https://github.com/MathiasStokholm/alkymi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
