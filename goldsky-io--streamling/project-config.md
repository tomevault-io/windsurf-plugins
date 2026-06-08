---
trigger: always_on
description: transforms: {}
---

# Streamling Development Guide

Streamling is a Rust-based data processing engine built on Arrow and DataFusion. It reads from sources (Kafka), applies transforms (SQL, HTTP, WASM), and writes to sinks (Postgres, ClickHouse, Kafka, SQS, webhook, etc.).

## Project Layout

```
crates/
├── streamling/              # Main binary
├── streamling-core/         # Core topology, operators, pipeline engine
├── streamling-config/       # Pipeline YAML parsing and configuration
├── streamling-connectors/   # Table providers (sources/sinks)
├── streamling-plugin/       # FFI-based plugin system
├── streamling-state/        # State backend (checkpointing)
├── streamling-flink-compat/ # Flink state compatibility
└── streamling-e2e/          # End-to-end test framework (see crates/streamling-e2e/AGENT.md)

plugin_examples/             # Example plugins (basic, low_level)
scripts/                     # k3s setup/teardown scripts
```

## Command Runner (just)

All common tasks use `just`. Run `just --list` to see everything.

| Command               | Purpose                                            |
| --------------------- |----------------------------------------------------|
| `just build`          | Build the workspace                                |
| `just fix`            | Auto-fix cargo issues                              |
| `just lint`           | Format + clippy for the main workspace             |
| `just test`           | Run unit tests                                     |
| `just env-setup`      | Spin up k3s with Postgres, Kafka, ClickHouse, etc. |
| `just env-status`     | Check k3s cluster health                           |
| `just env-teardown`   | Tear down k3s cluster                              |
| `just e2e-test`       | Build + run all e2e tests                          |
| `just e2e-test-debug` | E2e tests with full output                         |
| `just e2e-list`       | List available e2e tests                           |

## Development Workflow

### After every task

1. Run `just fix` to auto-fix warnings and issues.
2. Run `just lint` to verify formatting and clippy pass.

### Fixing a bug

1. Write a **unit test** that reproduces the issue first.
2. Watch it fail.
3. Implement the fix — keep the change as small and easy to understand as possible.
4. Verify the test passes.
5. Run `just fix && just lint`.

### Adding a feature

1. Prefer unit tests for quick iteration during development.
2. If the feature involves pipeline-level behavior (source -> transform -> sink), add an **e2e test** once the implementation is complete.
3. Run `just fix && just lint`.

### Before pushing

Check that CI/CD has passed for the PR:

```bash
gh pr checks 494   # use the relevant PR number
```

## Testing Strategy

### Unit tests (preferred for iteration)

```bash
just test                       # Run all unit tests
cargo test -p streamling-core   # Run tests for a specific crate
```

Unit tests are fast and should be the first choice for verifying logic changes.

### E2E tests (always run locally)

E2E tests exercise the full `streamling` binary as a black box against real infrastructure (Kafka, Postgres, ClickHouse, SQS via ElasticMQ) running in a local k3s cluster.

**E2E tests must always be created and run locally before pushing.** They are the final verification that a pipeline-level change works end-to-end.

```bash
# First time: set up the local k3s environment
just env-setup

# Run all e2e tests
just e2e-test

# Run a specific test
just e2e-test test_basic_postgres_sink

# Run with full debug output
just e2e-test-debug

# Use debug build (faster compile, slower runtime)
PROFILE=debug just e2e-test
```

See `crates/streamling-e2e/AGENT.md` for detailed guidance on writing e2e tests (test patterns, resource isolation, available helpers).

## Pipeline YAML

Pipelines are defined in YAML with three sections: `sources`, `transforms`, and `sinks`. The `transforms` key is **always required**, even if empty (`transforms: {}`).

```yaml
sources:
  my_source:
    type: kafka
    topic: my_topic
    starting_offsets: earliest
    primary_key: id
    telemetry:
      event_time:
        column: block_timestamp
        unit: seconds
      labels:
        tier: critical
        dataset: v2.evm.blocks

transforms: {}

sinks:
  my_sink:
    type: postgres
    from: my_source
    table: output_table
    schema: public
    primary_key: id
    on_conflict: update
```

### `telemetry` block (optional)

Every source, transform, and sink accepts an optional `telemetry` block:

- **`event_time`** — column that carries the record's event time; drives `streamling_event_time_watermark_milliseconds` and `streamling_event_time_lag_milliseconds` series. `unit` required for integer columns (`seconds`, `milliseconds`, `microseconds`).
- **`labels`** — map of `key: value` identity labels attached to every metric this node emits. Use for dataset identity, tier/team tagging, destination tagging.

Label constraints (enforced at config load):

| Rule | Reason |
|------|--------|
| Max 20 labels per node | Each label is a Prometheus dimension; unbounded maps blow up storage. |
| Keys match `^[a-zA-Z_][a-zA-Z0-9_]*$` | Prometheus label-name grammar. |
| Keys cannot start with `__` | Prometheus reserves that prefix for internal use. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [goldsky-io/streamling](https://github.com/goldsky-io/streamling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-08 -->
