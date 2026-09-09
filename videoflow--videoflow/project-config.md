---
trigger: always_on
description: Videoflow is a Python framework for building video/stream processing applications as a
---

# CLAUDE.md — videoflow core

Videoflow is a Python framework for building video/stream processing applications as a
**directed graph of nodes**, plus a runtime that executes that graph either as local
subprocesses or as one Kubernetes workload per node, and a `videoflow` CLI that deploys it.

The single idea that explains most of the design: **the graph is built on one machine and
executed on many.** Each node is serialized to `(class path, params)` and reconstructed inside
its own worker process, which has no access to the object that built the graph. Nearly every
convention below follows from that.

## Repo map

| Path | What lives there |
|---|---|
| `videoflow/core/` | The abstractions: `node.py` (Node hierarchy), `flow.py` (Flow), `graph.py` (validation + `Diagnostic`s), `task.py` (per-node run loop, `invoke_node`), `engine.py` (Messenger/ExecutionEngine interfaces), `errors.py` (the error taxonomy + dispositions), `policies.py` (JoinPolicy, DeliveryPolicy), `supervision.py` (breaker, progress deadline, restart policy), `compiler.py` (Flow → `NodeSpec`s), `remote.py`, `constants.py` |
| `videoflow/runtime/` | Runs **inside a worker container**: `worker.py` (the one-node entrypoint), `provision.py`, `health.py`, `idempotency.py`, `logging_config.py` |
| `videoflow/deploy/` | Runs on the **operator's machine**: `cli.py`, `compile.py`, `manifests.py`, `images.py`, `build.py`, `cluster.py`, `gpu.py`, `solution.py`, `infra.py`, `localinfra.py` |
| `videoflow/wire/` | `serialization.py` — the transport-independent envelope format (msgpack v2/v3, protobuf v4) |
| `videoflow/components/` | `descriptor.py` (component.yaml loading/validation), `oci.py` (descriptors as OCI artifacts) |
| `videoflow/engines/` | `local.py` (subprocess per node), `kubernetes.py` (pod per node) |
| `videoflow/messaging/` | NATS JetStream transport. `topology.py` is the single source of truth for subject/stream/durable naming |
| `videoflow/producers/`, `processors/`, `consumers/` | Built-in nodes |
| `videoflow/v1/` | **Generated** protobuf modules — never hand-edit (see hard rules) |
| `videoflow/utils/` | Graph algorithms, model downloader, parsers, transforms |
| `videoflow/*.py` (root) | Only `version.py` and five frozen compatibility shims — see below |
| `spec/` | Language-agnostic protocol contract: `PROTOCOL.md`, `proto/`, golden `vectors/`, `rfcs/` |
| `docker/`, `k8s/` | Base images (CPU + CUDA); dev broker manifests plus the kind cluster the k8s tests run on |
| `docs/` | Sphinx site (`docs/source/`) |
| `solutions/` | The four dependency-free `toy_*` solutions — deployable apps that double as the end-to-end test fixtures (see below) |
| `tests/`, `examples/` | Unit tests (no broker, no cluster) and runnable examples |
| `tests/integration/` | Three buckets: `broker/` (JetStream contract), `local/` (worker subprocesses), `k8s/` (a kind cluster) — see its `README.md` |

Sibling repo: `../videoflow-contrib` — community components and the ML solutions. It has its
own `CLAUDE.md`.

## The toy solutions are the end-to-end tests

`solutions/{toy_calculator,toy_fusion,toy_router,toy_recovery}` are complete, deployable solutions
built from core nodes only — no models, no footage, no `videoflow_contrib` packages — and they run
on every CI build **twice**: `tests/integration/local/test_toy_solutions.py` drives them with
`videoflow run-local`, and `tests/integration/k8s/test_k8s_solutions.py` deploys them to a kind
cluster. Between them they cover the framework paths no in-process test reaches:

| Solution | Flow type | What it gates | Success artifact |
|---|---|---|---|
| `toy_calculator` | BATCH | fan-out, trace join re-aligning branches, competing replicas, stateful aggregation, two-parent consumer, `metadata=True` consumer, prep hook | `report.json` → `matches_expected: true` |
| `toy_router` | BATCH | `partition_by` routing, `ctx.set_partition_key`, `async def process`, replica identity, idempotent sink | `counts.json` → `matches_expected` **and** `sticky` |
| `toy_fusion` | REALTIME | independent producers fused by event time, `tolerance_ms`/`quorum`/`collect`, unbounded sources, `ctx.input_info` | `fusion_summary.json` → complete moments |
| `toy_recovery` | BATCH | the error taxonomy end to end: a poison message dead-lettered on its first failure, a worker-fatal error handed back and the worker restarted, DLQ inspection | `recovery_report.json` → `matches_expected: true` |

Three things to know before touching them:

- **They are tests *and* published examples.** A change to a graph, a node or a config key means
  updating the solution's `README.md`, its `config.example.yaml`/`config.template.yaml`, and the
  shared config dict in `tests/integration/support_solutions.py` together. Keep every stream short
  enough that a full local run stays ~10s.
- **The local and cluster runs share their configs and assertions**, both imported from
  `tests/integration/support_solutions.py`. That is the point: a graph built on one machine is
  supposed to run unchanged on many, so anything asserted in only one of them is a claim nobody is
  checking. The only legitimate divergence is `K8S_FUSION_CONFIG` — `toy_fusion` needs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [videoflow/videoflow](https://github.com/videoflow/videoflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
