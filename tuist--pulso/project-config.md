---
trigger: always_on
description: A headless, open-source observability backend: unified logs/metrics/traces access, built-in alerting, and a native Model Context Protocol (MCP) interface, built on Elixir/OTP with a Rust hot path for columnar data.
---

# Pulso

A headless, open-source observability backend: unified logs/metrics/traces access, built-in alerting, and a native Model Context Protocol (MCP) interface, built on Elixir/OTP with a Rust hot path for columnar data.

Pulso is agent-native from day one. It does not ship a UI. Consumers are humans through their own dashboards and, first-class, AI agents through MCP.

## Read this before any non-trivial design work

**[`docs/architecture.md`](./docs/architecture.md)** — source of truth for Pulso's architecture. Load it before you reason about ingest, storage, coordination, alerting, the Elixir/Rust split, or the MCP boundary. If you're about to add a database, a WAL, a leader-election protocol, or any cluster-visible mutable state, read the "Core bets" and "What Pulso deliberately does not have" sections first.

## Where the project is right now

Early scaffolding. In place:

- Phoenix 1.8 headless app (no HTML, no assets, no Ecto)
- `Pulso.Loki` — read-only Loki HTTP client wrapping `query_range`
- `Pulso.MCP` — JSON-RPC 2.0 dispatcher (`initialize`, `tools/list`, `tools/call`, `ping`)
- `Pulso.MCP.Tools` — tool registry, currently one read-only tool (`query_logs`)
- `PulsoWeb.MCPController` at `POST /mcp` (handles single and batched JSON-RPC)
- `PulsoWeb.OTLPController` at `POST /v1/logs` — OTLP/HTTP JSON logs ingest
- `PulsoWeb.LokiController` at `POST /loki/api/v1/push` — Loki push ingest, JSON and Snappy-compressed protobuf (decoded in Rust by `Pulso.Codec.NIF`)
- `PulsoWeb.CompressedBodyReader` — gzip-aware Plug.Parsers body reader, so JSON receivers accept compressed bodies

Not yet built: alerting, ingestion, Mimir/Tempo clients, storage engine, remediation surface, HITL wiring, distribution (Horde/libcluster/ra).

## Design bet

Grafana's Loki/Mimir/Tempo (and the VictoriaMetrics stack) are already headless, API-only storage — but they're three systems coordinated externally, and existing "AI-ready" MCP servers on top are thin read-only wrappers bolted on after the fact.

Pulso's bet: build the ingest, storage-facade, alerting, and MCP-interface layer as one coherent system, on a runtime (BEAM/OTP) whose concurrency and supervision model fits this problem shape unusually well.

**v1 shape**: MCP-native gateway + shared alerting over proven headless stores (Mimir/Loki/Tempo or VictoriaMetrics underneath). A unified storage engine is a longer-term option once the interface layer proves itself.

## Why Elixir/OTP

- **Per-process resource isolation** — each stream/tenant/query can be its own process with an independent heap and `max_heap_size` cap. Kill a runaway unit of work without taking down the node.
- **Scheduler fairness** — preemptive, reduction-based scheduling; one expensive query cannot easily starve the rest.
- **Backpressure-aware ingestion** — GenStage/Broadway for demand-driven pipelines. Process mailboxes are unbounded by default, so this must be designed for deliberately.
- **Self-healing alerting** — each alert rule as a supervised process; "let it crash and restart" maps directly onto rule-evaluation correctness.
- **Clustering without external coordinators** — distributed Erlang + Horde (CRDT distributed supervisor/registry) for cluster-wide singleton ownership. `libcluster` for discovery.
- **Phoenix PubSub** — distributed pub/sub for cross-signal correlation and alert fan-out.

## Known constraints to design around

- Default distributed Erlang is a full mesh, doesn't scale cleanly past ~100–200 nodes without partitioned topologies.
- Horde's CRDT sync is *eventually consistent* — fine for shard ownership, not strong enough alone for "exactly-once alert firing." Plan to use `ra` (RabbitMQ's Raft, in Erlang) for that specific guarantee.
- Binary sub-references can pin large buffers in memory — `:binary.copy/1` discipline is needed when parsing large payloads and keeping small slices.
- Distributed Erlang's cookie auth is weak by default — cluster must stay inside a VPC, or use TLS distribution.

## MCP read/write boundary (load-bearing rule)

Pulso deliberately splits its MCP surface by risk class:

| Tier | Where it lives | Examples |
|---|---|---|
| Read-only diagnosis | Pulso's MCP server (this repo) | query logs/metrics/traces, correlate signals, fetch alert context |
| Write within Pulso | Pulso's MCP server (this repo) | silence/ack alert, attach annotation |
| Infrastructure remediation | **Separate MCP server, not this repo** | restart service, roll back deploy, scale |

**Do not** add infrastructure remediation tools to `Pulso.MCP.Tools`. Their blast radius must be contained by construction, not by hoping the agent behaves. Anything destructive should be gated through the human-in-the-loop pause that the agent runtime (e.g. Google AX) provides natively.

## Repo layout

```
lib/
  pulso/
    application.ex        # OTP supervision root
    loki.ex               # Read-only Loki HTTP client
    mcp.ex                # JSON-RPC dispatcher (public MCP entry point)
    mcp/
      tools.ex            # Tool registry — read-only tools only
  pulso_web/
    controllers/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tuist/pulso](https://github.com/tuist/pulso) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
