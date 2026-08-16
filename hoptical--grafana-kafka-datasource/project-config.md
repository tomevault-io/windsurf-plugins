---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Grafana data source plugin that streams Kafka topics into Grafana dashboards in real time. It has two halves that must be kept in sync:

- **Backend** (Go, `pkg/`): a Grafana backend plugin (`grafana-plugin-sdk-go`) that connects to Kafka via `segmentio/kafka-go`, decodes messages, and streams data frames to the frontend over Grafana Live.
- **Frontend** (TypeScript/React, `src/`): the Config Editor and Query Editor UI, plus a thin `DataSource` class that mostly delegates to the backend (`DataSourceWithBackend`).

## Common Commands

### Frontend

- `pnpm install` — install deps (pnpm is pinned via `packageManager`; use `corepack enable && corepack install`)
- `pnpm run dev` — webpack watch build
- `pnpm run build` — production build
- `pnpm run typecheck` — `tsc --noEmit`
- `pnpm run lint` / `pnpm run lint:fix` — ESLint (+ Prettier on fix)
- `pnpm run lint:md` — markdownlint for `**/*.md`
- `pnpm run test` — Jest watch mode, only changed files
- `pnpm run test:ci` — Jest single run (used in CI)
- `pnpm exec jest src/QueryEditor.test.tsx` — run a single frontend test file
- `pnpm run e2e` — Playwright E2E tests (needs Grafana/Kafka running, see below)

### Backend (Go)

- `mage buildAll` — cross-compile the backend binary for all platforms (also `mage build:backend` for current platform, wired to `pnpm run build:backend`)
- `mage test` — run Go unit tests
- `mage testRace` — run Go tests with `-race` (required in CI)
- `mage coverage` — run tests with coverage (used in CI)
- `golangci-lint run` — backend lint (CI uses `golangci-lint-action`)
- `go test ./pkg/plugin/... -run TestName -v` — run a single Go test
- `go test ./pkg/plugin/... -bench BenchmarkName -benchmem` — run a specific benchmark (many exist, e.g. `frame_builder_bench_test.go`, `json_utils_bench_test.go`, `stream_manager_bench_test.go`)

### Local dev environment

- `pnpm run server` — `docker compose up --build`: starts Grafana + Kafka (KRaft mode, SASL/SSL listeners) + Schema Registry + Redpanda Console, with this plugin mounted. Must be run **on the host**, not inside the Dev Container (no Docker daemon access there).
- Inside the Dev Container: run `pnpm`/`go`/`mage` commands; if you need Grafana/Kafka, point at the host with `GRAFANA_URL=http://host.docker.internal:3000` (macOS/Windows Docker Desktop ignores `--network=host`).
- Sample data producers live in `example/go` and `example/python` (see `example/README.md`) for generating JSON/Avro/Protobuf/plaintext test messages.

### Pre-commit

`pre-commit install` wires up ESLint+Prettier, `golangci-lint run --fix`, and `markdownlint-cli2 --fix` on commit.

## Architecture

### Backend request flow (`pkg/plugin/plugin.go`)

`KafkaDatasource` implements the SDK's `QueryDataHandler`, `CheckHealthHandler`, `StreamHandler`, and `CallResourceHandler`. Key entry points:

- `QueryData` / `query` — non-streaming query path (largely a no-op placeholder; real data arrives via streaming).
- `CallResource` — routes `/partitions`, `/topics`, `/schema-registry/validate`, `/avro/validate`, `/protobuf/validate` resource calls (topic/partition discovery and schema validation used by the Query Editor UI).
- `CheckHealth` — datasource connectivity check (used by the Config Editor "Save & Test").
- `SubscribeStream` / `RunStream` — the real work. `RunStream` spawns one goroutine per partition (`StreamManager.readFromPartition`) that pulls messages via `KafkaClientAPI.ConsumerPull`, converts them to Grafana `data.Frame`s, and fans them in over a buffered channel (`streamMessageBuffer = 100`) to a single sender loop that pushes frames to the `backend.StreamSender`.

`KafkaClientAPI` is the interface `kafka_client.KafkaClient` implements; it's abstracted so `pkg/plugin` tests can inject fakes without a real Kafka broker.

### Message processing pipeline (`pkg/plugin/stream_manager.go`)

`StreamManager.ProcessMessageToFrame` is the core per-message pipeline: decode (JSON/Avro/Protobuf/Plaintext) → flatten → build a `data.Frame` field-by-field via `FieldBuilder` (`frame_builder.go`). Line Protocol messages take a separate path (`ProcessMessageFrames` in `lineprotocol_frame.go`) because one Kafka message can expand into multiple LP lines → multiple frame rows with a fixed long-format schema (`Time | _measurement | _field | value | value_str | <tag columns> | offset`).

Field ordering matters for Grafana Live's schema-per-channel model: `time`, `partition` (if "all"), `offset`, key fields (alphabetical), then value fields (alphabetical) — see `docs/MESSAGE_KEYS.md`. `sortedFlatKeys` caches the sorted key order across messages when the field set is unchanged (see Performance section).

Avro/Protobuf schema resolution supports both inline schemas and Schema Registry lookups (Confluent wire format: magic byte + 4-byte schema ID, plus a protobuf message-index array). Registry schemas are cached by ID/subject (`getSchemaWithCache` family) — see `docs/PERFORMANCE_OPTIMIZATIONS.md`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hoptical/grafana-kafka-datasource](https://github.com/hoptical/grafana-kafka-datasource) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
