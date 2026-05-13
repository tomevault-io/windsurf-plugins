---
trigger: always_on
description: This file is for AI agents (and humans) writing new operations or working with the library programmatically. It covers the patterns that aren't obvious from reading the type signatures.
---

# Agent guide: extending krnel-graph

This file is for AI agents (and humans) writing new operations or working with the library programmatically. It covers the patterns that aren't obvious from reading the type signatures.

## The type hierarchy

Every OpSpec has a return type baked into its class hierarchy. An op that produces a column of vectors inherits from `VectorColumnType`, not just `OpSpec`. This is how the fluent API works: the methods on `VectorColumnType` are available on anything that produces vectors.

```python
from krnel.graph import OpSpec, VectorColumnType, ExcludeFromUUID
from typing import Annotated

# The op inherits from its *output* type, not from OpSpec directly.
class MyEmbeddingOp(VectorColumnType):
    source: VectorColumnType          # OpSpec-typed field → DAG edge (dependency)
    scale_factor: float = 1.0         # scalar field → parameter (not a DAG edge)
    description: Annotated[str, ExcludeFromUUID()] = ""  # excluded from UUID
```

The full type hierarchy lives in `src/krnel/graph/types.py`. The concrete operations live in `*_ops.py` files. When adding a new op, inherit from the appropriate `*Type` class.

## Fields: dependencies vs. parameters

The rule is simple: **if a field is typed as an `OpSpec` subclass, it is a DAG edge**. Everything else is a parameter.

```python
class MyOp(ScoreColumnType):
    input: VectorColumnType      # dependency — creates a DAG edge to whatever produced the vectors
    threshold: float = 0.5       # parameter — stored in UUID computation, not a graph node
```

DAG edges are traversed by runners and by `subs()`. Parameters are not.

## ExcludeFromUUID

Fields annotated with `ExcludeFromUUID()` are serialized for provenance but excluded from UUID computation. Use this for fields that don't affect results: local file paths, human-readable labels, debug flags.

```python
from typing import Annotated
from krnel.graph import ExcludeFromUUID

class MyOp(DatasetType):
    data: DatasetType
    label: Annotated[str, ExcludeFromUUID()] = ""  # won't change the UUID
```

Two `MyOp` instances with the same `data` but different `label` values will have the same UUID and share cached results.

## EphemeralOpMixin

Inherit from `EphemeralOpMixin` when an op is cheap enough to always re-run rather than cache. The runner skips writing results to storage for ephemeral ops.

```python
from krnel.graph.op_spec import EphemeralOpMixin

class SliceRowsOp(DatasetType, EphemeralOpMixin):
    dataset: DatasetType
    start: int = 0
    end: int | None = None
```

Examples: `TakeRowsOp`, `SelectColumnOp`, `BooleanLogicOp`. Use it for any operation that is a pure in-memory transformation with negligible cost.

## Adding a runner implementation

After defining the OpSpec, register an implementation on the runner. The decorator infers the op type from the second parameter's annotation.

```python
import pyarrow as pa
from krnel.graph.runners.local_runner import LocalArrowRunner
from my_module import MyEmbeddingOp

@LocalArrowRunner.implementation
def run_my_embedding_op(runner: LocalArrowRunner, op: MyEmbeddingOp) -> None:
    source_table = runner.to_arrow(op.source)        # materialize dependency
    # ... compute result ...
    result = pa.array([...])
    runner.write_arrow(op, result)                   # write result to storage
```

The implementation must call one of `runner.write_arrow()`, `runner.write_json()`, `runner.write_numpy()`, or `runner.write_sklearn_estimator()` to persist the result. The return value is ignored.

Implementations are registered globally when the module is imported, so make sure the module containing `@LocalArrowRunner.implementation` is imported before the runner is used.

## subs(): swapping graph nodes

`subs()` reconstructs the graph with targeted changes. The most common usage is swapping a dataset to re-run a subgraph on new data:

```python
# Build the graph once
ds_train = runner.from_parquet("train.parquet")
X = ds_train.col_text("text").llm_layer_activations(model="hf:gpt2", layer=-1)

# Reuse the graph with a different dataset — no re-specification needed
ds_test = runner.from_parquet("test.parquet")
X_test = X.subs(ds_train, file_path="test.parquet")  # update a field on ds_train
# or equivalently:
X_test = X.subs((ds_train, ds_test))                  # swap the node wholesale
```

`subs()` does not mutate anything. It always returns a new OpSpec.

## Working with Runner

`Runner` is a factory function, not a class. Calling `Runner()` returns a `LocalArrowRunner` by default (or whatever is configured). Subsequent calls with the same arguments return the cached instance.

```python
from krnel.graph import Runner
runner = Runner()                  # returns LocalArrowRunner by default
result = runner.to_json(some_op)   # dict
table  = runner.to_arrow(some_op)  # pa.Table
array  = runner.to_numpy(some_op)  # np.ndarray
df     = runner.to_pandas(some_op) # pd.DataFrame
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [krnel-ai/krnel-graph](https://github.com/krnel-ai/krnel-graph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
