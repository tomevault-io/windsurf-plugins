---
trigger: always_on
description: GoMLX is deliberately "non-eager": one builds the computation graph, and then executes it on a backend.
---

# GoMLX is machine learning framework (like PyTorch, JAX, TensorFlow, etc.) for Go.

GoMLX is deliberately "non-eager": one builds the computation graph, and then executes it on a backend.

It supports multiple backends (`github.com/gomlx/compute.Backend` interface) to compile and execute the computation
graphs built from the user's GoMLX code. Currently implemented:
- `go`: a pure Go backend, implemented in package `github.com/gomlx/compute/gobackend`.
- `xla`: uses XLA for acceleration (it JIT-compiles and executes using XLA's PJRT). It is implemented in 
  package `github.com/gomlx/go-xla/compute/xla`.

Both these engines are imported by importing the meta-package `import _ "github.com/gomlx/gomlx/backends/default`.

The XLA backend only support static shapes (known in "graph building time"). Currently it uses "fixed shapes", like XLA. So a different shape requires a dynamic JIT-recompilation. There is work in progress for the Go backend to add support for dynamic shapes.

## File Structure

- `core`: core API definitions, to allow building of graphs and their executions, with no reference to trainable 
  variables and such.
  - `core/graph`: defines a `Graph` (a computation grpah building built) and `Node` (represents a value and the output
    of an operation in the `Graph`). It also defines `Exec` that takes a graph building function from the user code as
    a paramter and executes it -- building and JIT-compiling the graph to each new input shape.
  - `core/tensors`: define a `Tensor`, concrete multi-dimensional array (or scalar) with values used as input
    and outputs for graph execution.
- `ml`: ML support libraries, including training, variables handling (`ml/model`), optimizers, layers, datasets, 
  checkpointing, etc.
  - `ml/model`: defines `Variable` (trainable variables/weights) and a `Store` hierarchical container of variables.
    It also defines `Scope`, a pointer to a `Store` setting a "current directory" (scope) for new variables.
    Finally, it defines an `Exec` object that calls a user graph functions and automatically handle passing 
    used variables as "side inputs" and changed variaables as "side outputs" of the execution.
- `support`: support libraries, including xsync, xerrors, etc.
- `internal`: internal libraries and generators.
  - `internal/cmd`: command line tools: mostly generators used by `go generate ...`. They output `gen_*.go` files.
- `examples`: examples of usage of the library. Some well maintained, others may need to be outdated.
- `ui`: UI tools, for intance to display a training progress bar, or if using GoNB (a Jupyter kernel for Go), to
  dynamically plot a graph with the training progress.
  It is separated in its own top-level directory, to prevend UI packages dependencies to pure GoMLX.
- `.github/workflows`: continuous integration, one file per platform. 

## Coding Style In GoMLX

### Auto-generated code

Files that start with `gen_` are auto-generated and don't include a copyright line
directly -- the copyright line is in their generators.
Many are created with generators included under `internal/cmd/...`, and the generated file 
includes a comment stating which tool was used to generate them.

### Graph Building Functions

GoMLX works by building computation graphs and then JIT-compiling and executing them in a backend.

The graph building functions take `*graph.Node`, `*graph.Graph` and/or `*model.Scope` (for variables) as arguments, 
and return updated `*graph.Node` of the graph.

Graph building functions are assumed to be executed sequentially (no concurrency between graph building functions), 
so no need for mutexes, etc. Once compiled they can be executed in parallel.

Also, different than standard Go, graph building functions return errors by throwing (panicking) instead of returning 
an error, to simplify the code. But everywhere else, use standard Go error handling (by returning an error).

The compiled and execution of the graphs later is parallelized and can be executed in concurrently as one wishes.

Files that mostly define graph building functions, by convention, should dot-import the 
`github.com/gomlx/gomlx/core/graph` package: having `graph.` repeated everywhere makes the math harder to read.
This is commonly the case for libraries under `ml/layers`.

### Testing

Never attempt to test everything (`go test ./...`)! Many of the tests are very long, and parallel testing 
doesn't work (because the XLA backend for CUDA allocates all the memory on startup, so there can't be more
than one instance at a time).

If you are testing more than one package at a time (using ellipsis "..."), remember to use the flag `-p 1` (e.g., `go test -p 1 ./ml/model/...`), since creating more than one CUDA backend simultaneously fails.

Instead test only one package at a time, or even, preferably, only the tests related to the code you are 
currently modifying (`go test <./.../package> -run <test_pattern>`).

### Executing models and graphs:

If executing a model with variables, those are stored in `ml/model.Store` objects (they can also be referenced by
`model.Scope` objects, pointers into a `Store` with a scope), and require `model.Exec` object to execute. 


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gomlx/gomlx](https://github.com/gomlx/gomlx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
