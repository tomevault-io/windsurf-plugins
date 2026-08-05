---
trigger: always_on
description: Canonical guidance for AI coding agents (Claude Code, Cursor, Copilot, etc.)
---

# AGENTS.md

Canonical guidance for AI coding agents (Claude Code, Cursor, Copilot, etc.)
working in this repository. `CLAUDE.md` is a symlink to this file.

For the do/don't list of public **API call patterns**, see `usage-rules.md` — read
it before writing or changing public API call sites. Dependency usage rules are
inlined under the managed marker block at the end of this file (refresh with
`mix usage_rules.sync`).

## Project

KafkaEx is an Elixir client for Apache Kafka. v1.0+ is built on [Kayrock](https://github.com/dantswain/kayrock) for protocol (de)serialization and performs **automatic API version negotiation** — there is no `kafka_version` config. Requires Elixir 1.14+, OTP 24+, Kafka 0.11.0+.

`KafkaEx.API` is the primary, modern interface; the older top-level `KafkaEx.*` worker API is retained only for backward compatibility. See `usage-rules.md` for the canonical do/don't list of API call patterns — read it before writing or changing public API call sites.

## Commands

```bash
# Unit tests — no Kafka cluster needed (excludes auth/consume/consumer_group/chaos/lifecycle/produce tags)
mix test.unit

# Integration tests — requires the Docker cluster (includes those tags)
./scripts/docker_up.sh          # start 3-broker test cluster first
mix test.integration

# Chaos / network-resilience tests (Docker + Testcontainers)
ENABLE_TESTCONTAINERS=true mix test.chaos

# Everything
mix test

# A single test file / line / tag
mix test test/kafka_ex/some_test.exs
mix test test/kafka_ex/some_test.exs:42
mix test --only consumer_group        # also: produce, consume, auth, lifecycle
mix test --include sasl               # SASL tests are excluded by default

# Static checks (all required for PRs)
mix format --check-formatted
mix credo --strict
mix dialyzer
mix compile --warnings-as-errors
```

Integration test broker ports: 9092–9094 no-auth (SSL), 9192–9194 SASL/PLAIN, 9292–9294 SASL/SCRAM, 9392–9394 SASL/OAUTHBEARER.

## Architecture

Request flow, top to bottom:

1. **`KafkaEx.API`** (`lib/kafka_ex/api.ex`) — public surface. Every function takes a `client` as the first arg. `use KafkaEx.API, client: ...` generates the same functions bound to a configured client. Returns `{:ok, result}` / `{:error, reason}`; results are native `KafkaEx.Messages.*` structs.
2. **`KafkaEx.Client`** (`lib/kafka_ex/client/client.ex`) — the `GenServer` that owns a connection to a cluster. Holds `Client.State` + `Cluster.ClusterMetadata`, selects the target broker via `Client.NodeSelector`, builds requests (`RequestBuilder`), parses responses (`ResponseParser`), and drives retries (`Support.Retry`). Network I/O goes through `KafkaEx.Network.*` (sockets, SSL).
3. **`KafkaEx.Protocol.KayrockProtocol`** (`lib/kafka_ex/protocol/kayrock_protocol.ex`) — the dispatch hub. `build_request(operation, api_version, opts)` and the parse side route by `{operation, version}` to the per-operation modules below. This module is the *only* place the rest of the client talks to the protocol layer; it's slated to become a separate package post-1.0.
4. **Per-operation protocol modules** (`lib/kafka_ex/protocol/kayrock/<operation>/`) — each operation (produce, fetch, metadata, offset_commit, join_group, …) defines two Elixir **protocols**, `Request` and `Response`, both with `@fallback_to_any true`, plus one `vN_request_impl.ex` / `vN_response_impl.ex` `defimpl` per supported Kafka API version, an `any_*_impl.ex` forward-compat fallback, and shared `request_helpers.ex` / `response_helpers.ex`. The `defimpl` is keyed on the concrete Kayrock request/response struct for that version.

**Adding support for a new API version of an operation** means adding `vN_request_impl.ex` + `vN_response_impl.ex` under that operation's directory (mirroring the existing vN files) and updating the operation's `@moduledoc` version table — not editing the dispatch hub.

### Other key areas

- **`KafkaEx.Cluster.*`** — broker/topic/partition metadata model (`ClusterMetadata`, `Broker`, `Topic`, `TopicPartition`, `PartitionInfo`).
- **`KafkaEx.Messages.*`** — the native structs returned to callers (`Fetch`, `Fetch.Record`, `RecordMetadata`, `Offset`, consumer-group descriptions, etc.). Protocol response impls produce these.
- **`KafkaEx.Consumer.*`** — `GenConsumer` behaviour (`handle_message_set/2` → `{:async_commit, state}` or `{:sync_commit, state}` — no `:no_commit`), `ConsumerGroup` coordinator with `Manager`, `Heartbeat`, and `PartitionAssignment`, plus `Stream`.
- **`KafkaEx.Auth.*`** — SASL: `plain`, `scram` (`scram_flow`), `oauthbearer`, `msk_iam`. PLAIN is enforced to require SSL (`:plain_requires_tls`).
- **`KafkaEx.Producer.*`** — partitioner and produce path; `Legacy` is the backward-compat producer.
- **`KafkaEx.Telemetry`** — 27+ events under the `[:kafka_ex, ...]` prefix.
- **`KafkaEx.Support.OptionalDeps`** — compression/auth backends (`snappyer`, `ezstd`, `lz4b`, `aws_signature`, …) are optional deps; `Client.init` validates them at boot so misconfiguration crashes loudly instead of at first use.

### API version resolution


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kafkaex/kafka_ex](https://github.com/kafkaex/kafka_ex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
