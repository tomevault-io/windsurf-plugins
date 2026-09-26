---
trigger: always_on
description: This document is intended for AI coding agents and developers working on or using the `onnx-gomlx` codebase.
---

# ONNX-GoMLX Agent Guide

This document is intended for AI coding agents and developers working on or using the `onnx-gomlx` codebase.
It provides instructions on using the library, understanding its internal architecture, implementing new operations, materializing constant subexpressions, handling fused operations, and debugging.

---

## 1. Using ONNX-GoMLX

`onnx-gomlx` translates ONNX computational graphs (`.onnx` protobuf models) into [GoMLX](https://github.com/gomlx/gomlx) execution graphs, enabling high-performance ML inference, fine-tuning, and graph composition in Go.

### 1.1 Standard Inference Workflow

```go
package main

import (
	"fmt"
	"github.com/gomlx/gomlx/backends"
	_ "github.com/gomlx/gomlx/backends/default" // Registers default backend (e.g. XLA, Go, or ONNX)
	"github.com/gomlx/gomlx/core/graph"
	"github.com/gomlx/gomlx/core/tensors"
	"github.com/gomlx/gomlx/ml/model"
	"github.com/gomlx/onnx-gomlx/onnx"
)

func main() {
	// 1. Load and parse the ONNX model file.
	onnxModel, err := onnx.ReadFile("path/to/model.onnx")
	if err != nil {
		panic(err)
	}

	// 2. Extract weights/initializers into a GoMLX model.Store.
	store := model.NewStore()
	if err := onnxModel.VariablesToScope(store.RootScope()); err != nil {
		panic(err)
	}

	// 3. Create an execution function.
	backend := backends.New()
	exec := model.MustNewExec(backend, store, func(scope *model.Scope, inputIDs, attentionMask *graph.Node) *graph.Node {
		g := inputIDs.Graph()
		outputs := onnxModel.CallGraph(scope, g, map[string]*graph.Node{
			"input_ids":      inputIDs,
			"attention_mask": attentionMask,
		})
		return outputs[0]
	})
	defer exec.Finalize()

	// 4. Run execution with tensors.
	// ... construct inputTensors ...
	// out := exec.MustCall1(tIDs, tMask)
}
```

### 1.2 Dynamic Shapes Support

Some backends (such as the Go portable backend `GOMLX_BACKEND=go` and the ONNX Runtime backend `GOMLX_BACKEND=onnx:cpu`) natively support dynamic dimensions (unknown batch size, variable sequence lengths) without requiring graph recompilation for every shape variation.

To enable dynamic shapes, configure `exec.WithDynamicAxes`:

```go
exec := model.MustNewExec(backend, store, func(scope *model.Scope, inputIDs, attentionMask, tokenTypeIDs *graph.Node) *graph.Node {
    g := inputIDs.Graph()
    outputs := onnxModel.CallGraph(scope, g, map[string]*graph.Node{
        "input_ids":      inputIDs,
        "attention_mask": attentionMask,
        "token_type_ids": tokenTypeIDs,
    })
    return outputs[0]
})

// Specify dynamic axis names for each parameter:
exec.WithDynamicAxes(
    []string{"batch", "seq"}, // input_ids: shape [?, ?]
    []string{"batch", "seq"}, // attention_mask: shape [?, ?]
    []string{"batch", "seq"}, // token_type_ids: shape [?, ?]
)
defer exec.Finalize()

// Subsequent calls with different batch sizes or sequence lengths will run
// on the same compiled graph without triggering recompilation:
outBatch1 := exec.MustCall1(tIDsBatch1, tMaskBatch1, tTypesBatch1)
outBatch2 := exec.MustCall1(tIDsBatch2, tMaskBatch2, tTypesBatch2)
```

Backends that do not support dynamic shapes (like static XLA PJRT) will pad inputs to fixed static dimensions.

---

## 2. Architecture & Codebase Layout

- **`onnx/`**: Public API package. Provides `ReadFile`, `Parse`, and the user-facing `Model` type wrapper.
- **`internal/onnxgomlx/`**: Core conversion engine.
  - **`model.go`**: Manages model state, graph translation lifecycle, and input/output node mappings.
  - **`ops.go`**: Operator converter dispatch table (`m.converters[opType]`) and individual operator converters (`convertMatMul`, `convertConv`, `convertWhere`, `convertExpand`, etc.).
  - **`materialize.go`**: Constant subexpression evaluation engine. Evaluates ONNX nodes that compute tensor parameters (shapes, slices, broadcasting dimensions) at graph building time.
  - **`fusion/`**: Fused operation matchers (`sdpa.go`, `qkvdense.go`, `dense.go`). Detects multi-op subgraphs and emits GoMLX fused operations.
- **`internal/onnxgraph/`**: Topological ordering, dependency tracking, and graph representation of ONNX protobuf nodes.
- **`internal/benchmarks/`**: Parity tests, benchmarks vs. ONNX Runtime, and integration tests across backends.

---

## 3. Developing and Extending ONNX-GoMLX

### 3.1 Implementing New Operator Converters

When adding a new ONNX operator:
1. Define the converter method in `internal/onnxgomlx/ops.go`:
   ```go
   func (m *Model) convertMyOp(node *protos.NodeProto, inputs []*graph.Node) *graph.Node
   ```
2. Register it in `init()` or `registerDefaultConverters()`:
   ```go
   m.converters["MyOp"] = (*Model).convertMyOp
   ```
3. **Pay close attention to broadcasting rules**:
   - ONNX follows NumPy-style right-aligned multidirectional broadcasting.
   - GoMLX nodes expect explicit shapes or use `graph.BroadcastInDim` / `graph.BroadcastToDims` for broadcast adjustments.
4. **Extracting node attributes**:
   Use helper functions in `internal/onnxgomlx/` to read attributes from `node.Attribute` (e.g. `getAttrInt`, `getAttrFloat`, `getAttrStrings`, `getAttrTensor`).

---

### 3.2 Materializing Constant Subexpressions

#### Why Materialization is Required

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gomlx/onnx-gomlx](https://github.com/gomlx/onnx-gomlx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
