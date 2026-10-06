---
trigger: always_on
description: Turns records from a Databricks Lakeflow streaming pipeline into durable Dapr Workflow
---

# AGENTS.md — dapr.ext.databricks

Turns records from a Databricks Lakeflow streaming pipeline into durable Dapr Workflow
executions. Databricks keeps doing what it's good at (ingestion, transformation, streaming
checkpoints, Lakeflow execution); Dapr Workflow takes over the business process a record
triggers (durable execution, retries, timers, human interaction, external systems).

## Source layout

```
dapr/ext/databricks/
├── __init__.py            # Public API exports
├── sink.py                 # register_workflow_sink() + lazy pyspark.pipelines import
├── batch_handler.py         # DaprWorkflowBatchHandler — the reusable low-level API
├── scheduling.py             # ensure_workflow_scheduled() — the dedup/retry-safety core
├── identity.py                # Deterministic instance-ID derivation + sanitization
├── mapping.py                  # default_row_mapper() — Row -> JSON-safe dict
├── config.py                    # WorkflowSinkConfig dataclass + validation
├── exceptions.py                 # DaprDatabricksError hierarchy
├── _typing.py                     # RowLike / BatchDataFrameLike Protocols (no pyspark import)
└── py.typed

tests/ext/databricks/
├── _fakes.py                # Shared test doubles (FakeRow, FakeDataFrame, FakeWorkflowClient, ...)
├── test_identity.py          # Deterministic IDs, sanitization, Unicode, hashing
├── test_mapping.py            # Default + custom row mappers
├── test_scheduling.py          # ensure_workflow_scheduled: new/existing/race/lost-response
├── test_batch_handler.py        # End-to-end batch processing, retry, concurrency, metadata
├── test_config.py                 # WorkflowSinkConfig validation
└── test_sink.py                    # register_workflow_sink + pyspark.pipelines wiring

tests/integration/test_databricks_sink.py   # Runs against a real Dapr sidecar (no Databricks needed)
examples/databricks/                          # Fraud-remediation example (see its own README)
```

Installed via the `databricks` extra: `pip install "dapr[databricks]"`. Like `workflow`, this
extra has **no third-party runtime dependencies** — see "Dependency design" below.

## Architecture

```
┌───────────────────────────────────────────────────────────┐
│ Databricks Lakeflow pipeline                               │
│   @dp.append_flow(target=<sink>) -> readStream              │
└──────────────────────────┬──────────────────────────────────┘
                            │ micro-batch (DataFrame, batch_id)
┌──────────────────────────▼──────────────────────────────────┐
│ register_workflow_sink()                                     │
│  - lazily imports pyspark.pipelines                           │
│  - registers @dp.foreach_batch_sink(name=...) wrapping         │
│    DaprWorkflowBatchHandler.process                             │
└──────────────────────────┬────────────────────────────────────┘
                            │
┌──────────────────────────▼────────────────────────────────────┐
│ DaprWorkflowBatchHandler.process(df, batch_id)                  │
│  - df.toLocalIterator(prefetchPartitions=True) — never .collect() │
│  - bounded concurrency via ThreadPoolExecutor(max_in_flight)        │
│  - per row: identity.py -> scheduling.py -> optional metadata wrap    │
│  - any unresolved row -> raises, failing the micro-batch               │
└──────────────────────────┬────────────────────────────────────────────┘
                            │
┌──────────────────────────▼────────────────────────────────────────────┐
│ scheduling.ensure_workflow_scheduled(client, workflow, instance_id, ...) │
│  1. get_workflow_state(instance_id) -> exists? already accepted, done      │
│  2. else schedule_new_workflow(...); ALREADY_EXISTS response -> also done    │
│  3. any other error propagates (auth, unavailable, timeout, throttling, ...)  │
└──────────────────────────┬──────────────────────────────────────────────────┘
                            │
                 dapr.ext.workflow.DaprWorkflowClient (existing, unmodified)
                            │
                      Dapr sidecar / endpoint (gRPC)
```

No new workflow engine, no new transport: this extension is entirely a scheduling and
identity layer on top of the existing `dapr.ext.workflow.DaprWorkflowClient`.

## Public API

```python
from dapr.ext.databricks import (
    register_workflow_sink,     # high-level: registers a foreach_batch_sink
    DaprWorkflowBatchHandler,   # low-level: reusable batch-processing handler
    WorkflowSinkConfig,        # typed config, if constructing a handler directly
    default_row_mapper,       # the default Row -> dict mapper
    DaprDatabricksError,     # base exception
    SinkConfigurationError, # bad register_workflow_sink() config
    MissingBusinessKeyError, # configured id_field/id_fields absent or null on a row
    DaprDatabricksSinkError, # a micro-batch could not be durably handed off
)
```

`register_workflow_sink(name, workflow, *, id_field=None, id_fields=None, namespace='default',
generation='v1', input_mapper=None, instance_id_factory=None, metadata=True, max_in_flight=8,
max_records_per_batch=None, host=None, port=None, allow_batch_position_identity=False)` is the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dapr/python-sdk](https://github.com/dapr/python-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
