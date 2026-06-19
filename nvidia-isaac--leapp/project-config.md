---
trigger: always_on
description: SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
---

<!--
SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
SPDX-License-Identifier: Apache-2.0
-->

# LEAPP Agent Playbook

This file teaches coding agents how to apply LEAPP quickly in user projects.

## What LEAPP is for

LEAPP traces PyTorch computations into a graph of named nodes, then exports:

- per-node models (`.pt` or `.onnx`)
- a pipeline spec (`<graph_name>.yaml`)
- optional graph visualization (`<graph_name>.png`)

Primary goal: export complex pipelines with small annotation inserts and no functional code rewrites (unless absolutely needed for tracing/export edge cases).

## Core workflow (always in this order)

```python
import leapp
from leapp import annotate

leapp.start(name="my_graph", save_path=".")
# ... trace nodes ...
leapp.stop()
leapp.compile_graph(visualize=True, validate=True)
```

Do not call `compile_graph()` before `stop()`.
Use `leapp.start()`, `leapp.stop()`, and `leapp.compile_graph()` for graph lifecycle control.
Use `annotate` only for annotation APIs such as `method()`, `input_tensors()`, and `output_tensors()`.

## Optional runtime/export settings (important knobs)

Use these to control tracing cost, validation coverage, and output artifacts.

- `leapp.start(..., max_cached_io=N)`:
  - Controls how many re-entry I/O examples LEAPP caches per node for multi-example validation.
  - Higher values improve confidence for looped/stateful pipelines, but increase memory/time.
  - Practical default: keep `N` small (`3-5`) unless user explicitly wants stronger replay coverage.
- `leapp.compile_graph(validate=True, rtol=..., atol=..., strict=True)`:
  - `validate=True` compares exported model outputs against traced outputs.
  - Tune `rtol`/`atol` if expected numeric drift exists (especially ONNX/cross-device).
  - Use `validate=False` only for rapid iteration or when user asks for speed over checks.
- `leapp.start(..., dry_run=True)`:
  - Skips real model compilation/export, but still traces graph structure.
  - Useful for debugging node boundaries, names, and pipeline wiring before expensive export.
- `leapp.start(..., non_traced=["node_a", "node_b"])`:
  - Selectively disables tracing/export for the listed nodes while still registering them in the pipeline.
  - Those nodes still capture inputs/outputs, contribute to graph connectivity, and appear in YAML.
- `leapp.compile_graph(visualize=True)`:
  - `True` emits `<graph_name>.png` graph visualization.
  - `False` is faster for CI/headless runs when the image is not needed.
- `leapp.compile_graph(dry_run=True)`:
  - Declares dry-run at compile time after normal tracing has already happened.
  - Keeps traced FX graphs and YAML generation, but skips compile/save/validate of exported artifacts.
- Also useful:
  - `leapp.start(..., verbose=True)` for detailed trace logs, including FX graph dumps for traced nodes.
  - `leapp.start(..., global_patching=False)` if numpy-related patching causes environment issues.


## Critical node declaration rule

For `TracedTensorNode` workflows (`input_tensors` / `output_tensors`), agents must follow this exactly:

- `annotate.input_tensors("node_name", ...)` can be called multiple times for the same node.
  - This is valid across helper functions, class methods, and even different files, as long as it is the same active trace and same node name.
  - Use this to accumulate/declare node inputs wherever they naturally appear in the code.
  - For raw tensors, always pass a top-level dict of named tensors. Bare tensors are not supported; use `TensorSemantics(...)` if you want a single named semantic input without a dict.
- `annotate.output_tensors("node_name", ...)` is the node finalization declaration and should be done once for the initial trace of that node.
  - After this, the node is compiled/finalized.
  - Any later calls in re-entry loops are validation/tag-update behavior, not a second independent output declaration.


Example:

```python
leapp.start(
    name="my_graph",
    max_cached_io=5,
    dry_run=False,
    verbose=True,
)
# ... trace ...
leapp.stop()
leapp.compile_graph(
    visualize=True,
    validate=True,
    rtol=1e-3,
    atol=1e-5,
    strict=True,
)
```

## Semantic info injection (for downstream frameworks)

Use semantic metadata when deployers need tensors to carry meaning (not just shape/dtype).
LEAPP supports semantic injection via `TensorSemantics` wrappers passed to `input_tensors()` / `output_tensors()`.

- Current supported semantic fields:
  - `kind`: high-level semantic role string/enum for a tensor.
  - `element_names`: per-element labels (for vector/joint/channel interpretability).
- For `kind`, LEAPP provides two semantic enum families:
  - `InputKindEnum` for input/state/command-like inputs.
  - `OutputKindEnum` for output/target/control-like outputs.
- Output location:
  - semantic fields are serialized into the generated YAML tensor entries.
- Input format rules:
  - pass a single `TensorSemantics` or a list of `TensorSemantics`.
  - do not wrap `TensorSemantics` in dicts, and do not mix raw tensors with `TensorSemantics` in the same list.

Example:

```python
import torch
import leapp
from leapp import annotate, TensorSemantics, InputKindEnum, OutputKindEnum


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nvidia-isaac/leapp](https://github.com/nvidia-isaac/leapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
