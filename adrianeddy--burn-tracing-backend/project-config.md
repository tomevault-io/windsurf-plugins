---
trigger: always_on
description: This document describes the data format of `trace_data.js` files produced by the [burn-tracing-backend](https://github.com/AdrianEddy/burn-tracing-backend) profiler for the [Burn](https://burn.dev) deep learning framework. Use this to write targeted Python scripts that parse trace data for ML model optimization, profiling, and performance analysis.
---

# Burn Tracing Backend — Trace Data Format Reference

This document describes the data format of `trace_data.js` files produced by the [burn-tracing-backend](https://github.com/AdrianEddy/burn-tracing-backend) profiler for the [Burn](https://burn.dev) deep learning framework. Use this to write targeted Python scripts that parse trace data for ML model optimization, profiling, and performance analysis.

## File Format

The trace file is a JavaScript file defining a single constant:

```js
const TRACE_DATA = [ ...array of event objects... ];
```

### How to Load in Python

```python
import json

def load_trace(path: str) -> list[dict]:
    """Load trace_data.js into a list of event dicts."""
    text = open(path).read()
    # File format is: const TRACE_DATA = [...];\n
    return json.loads(text[len("const TRACE_DATA = "):].strip().rstrip(";"))

events = load_trace("trace_data.js")
```

## Event Object Schema

Every event in the array is a JSON object. Fields marked **(optional)** are omitted when not applicable (use `.get()` in Python).

### Core Fields (always present)

| Field | Type | Description |
|---|---|---|
| `name` | `string` | Operation name. Examples: `"float_random"`, `"float_matmul"`, `"float_add"`, `"sync"`, `"fusion::elementwise"`, `"fusion::matmul"`, `"my_marker"` |
| `category` | `string` | One of: `"float"`, `"int"`, `"bool"`, `"module"`, `"activation"`, `"quantized"`, `"transaction"`, `"sync"`, `"fusion"`, `"marker"` |
| `start_us` | `float` | Microseconds since trace start (wall-clock offset from first event) |
| `duration_us` | `float` | Duration in microseconds (CPU-side timing) |
| `op_index` | `int` | Sequential index — events are ordered by execution sequence |

### Shape & Memory Fields (optional)

| Field | Type | Description |
|---|---|---|
| `input_shapes` | `list[list[int]]` | Shapes of input tensors, e.g. `[[4, 16], [16, 32]]` for matmul |
| `output_shape` | `list[int]` | Shape of the output tensor, e.g. `[4, 32]` |
| `memory_bytes` | `int` | Estimated memory allocation in bytes for this operation's output |

### Tensor Data Preview Fields (optional, requires `trace-data` feature)

| Field | Type | Description |
|---|---|---|
| `data_preview` | `list[float]` | First N values of the output tensor (default N=64). Used for non-fusion ops |
| `data_shape` | `list[int]` | Shape of the tensor previewed in `data_preview` |

### Caller Location (optional, requires `trace-caller` feature)

| Field | Type | Description |
|---|---|---|
| `caller` | `string` | Source location like `"./examples/sample_inference.rs:15"`. Not available for fused ops |

### GPU Sync Fields (optional)

| Field | Type | Description |
|---|---|---|
| `is_sync` | `bool` | `true` if this op forces GPU synchronization. Appears on explicit `sync()` calls and implicit syncs like `into_data()` / `to_data()`. Sync events are performance-critical — they block until GPU work completes |

### Fusion Fields (optional, only on `category == "fusion"` events)

Fusion events represent multiple operations that the GPU compiler fused into a single kernel launch.

| Field | Type | Description |
|---|---|---|
| `fusion_kind` | `string` | One of: `"elementwise"`, `"matmul"`, `"reduce"`, `"reducebroadcasted"` |
| `num_fused_ops` | `int` | Number of operations fused into this kernel |
| `fused_ops` | `list[FusedOpInfo]` | Details of each individual operation in the fusion block (see below) |
| `fusion_inputs` | `list[FusionTensorData]` | Input tensor data for the fusion block (with `trace-data` feature) |
| `fusion_outputs` | `list[FusionTensorData]` | Output tensor data for the fusion block (with `trace-data` feature) |
| `kernel_sources` | `list[string]` | Compiled GPU kernel source code(s) (with `trace-data` feature). Typically one per fusion, sometimes two for matmul |

#### `FusedOpInfo` Object

Each element of `fused_ops` describes one operation within a fusion block:

| Field | Type | Description |
|---|---|---|
| `name` | `string` | Operation name with category prefix, e.g. `"float::Add"`, `"float::Exp"`, `"module::Conv2d"` |
| `input_shapes` | `list[list[int]]` | Input tensor shapes for this sub-operation |
| `output_shapes` | `list[list[int]]` | Output tensor shapes for this sub-operation |
| `input_ids` | `list[string]` | Tensor IDs of inputs (may be empty). Links tensors across operations within a fusion |
| `output_ids` | `list[string]` | Tensor IDs of outputs (may be empty) |
| `is_fallback` | `bool` or absent | `true` if this op was NOT fused but ran as a separate kernel (fallback). Absent or `null` means it was successfully fused |

#### `FusionTensorData` Object

Each element of `fusion_inputs` / `fusion_outputs`:

| Field | Type | Description |
|---|---|---|
| `shape` | `list[int]` | Tensor shape |
| `dtype` | `string` | Element type, e.g. `"F32"`, `"Bool(U32)"` |
| `data` | `list[float]` | First N values as float64 |
| `tensor_id` | `string` or absent | Links to a specific fused op's input/output ID |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AdrianEddy/burn-tracing-backend](https://github.com/AdrianEddy/burn-tracing-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
