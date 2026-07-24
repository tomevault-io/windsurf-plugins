---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository. It captures invariants and "where to look" pointers —
for specifics (file contents, constants, config-key catalog, error categories)
read the code, which is authoritative.

## Project

Go client library for QuestDB ingestion. Three transports:

- **HTTP / HTTPS** and **TCP / TCPS** — the legacy InfluxDB Line Protocol (ILP).
- **WS / WSS (QWP)** — QuestDB's binary columnar wire protocol over WebSocket.
  The only transport exposing the full type system (int8/16/32, float32, char,
  date, timestamp-nanos, uuid, varchar, geohash, int64 arrays). **QWP is not a
  version of ILP** — distinct framing, codecs, and server handshake.

Module path: `github.com/questdb/go-questdb-client/v4` — the `/v4` segment is
load-bearing when importing within this repo. Minimum Go: 1.23 (go.mod pins
`go 1.23` with a `1.24.4` toolchain).

## Commands

```bash
# Required for interop_test.go.
git submodule update --init --recursive

# Static analysis (run by CI).
go vet ./...
go run honnef.co/go/tools/cmd/staticcheck@v0.7.0 ./...

# Tests. Integration suites spin up QuestDB containers via
# testcontainers-go, so Docker must be running for those.
go test -v ./...

# Single suite — testify suites dispatch via the top-level
# Test*Suite entry point plus the method name.
go test -v -run TestIntegrationSuite/TestE2EValidWrites .
go test -v -run TestQwpIntegrationSuite .

# Allocation-tracked benchmark on the QWP hot path.
go test -v -bench BenchmarkQwpSenderSteadyState -benchmem -run ^$ .
```

`examples/` ships compilable `main.go` files referenced by
`examples.manifest.yaml`, which questdb.io uses to render docs — keep paths and
filenames stable.

## Architecture

Public surface: `LineSender` interface in `sender.go`. Every transport satisfies
it. `QwpSender` (in `qwp_sender.go`) is a superset for QWP-only column types —
callers wanting them must type-assert.

Two entry points: `LineSenderFromConf(ctx, "schema::addr=...;key=value")`
(parser in `conf_parse.go`; schemas: `http`, `https`, `tcp`, `tcps`, `ws`,
`wss`) and `NewLineSender(ctx, opts...)` (requires one of `WithHttp`, `WithTcp`,
`WithQwp`). Both funnel through `lineSenderConfig` and `newLineSender` in
`sender.go`. **`conf_parse.go` is the single source of truth for supported
config keys.**

### ILP (HTTP / TCP)

Three protocol versions: V1 text-only, V2 adds binary `float64` and
n-dimensional float arrays, V3 adds decimals. Each transport has three concrete
structs — `httpLineSender{,V2,V3}` in `http_sender.go`, `tcpLineSender{,V2,V3}`
in `tcp_sender.go`.

HTTP auto-negotiates the version; TCP requires `WithProtocolVersion(...)` or
`protocol_version=2|3`. **Adding a new ILP column type or feature touches all
six structs**, the `LineSender` interface, `buffer.go` (raw ILP encoding), and
the `Messages` / `MsgCount` / `BufLen` / `ProtocolVersion` switch helpers in
`export_test.go`. Keep those switches exhaustive.

### QWP (WebSocket columnar protocol)

Everything QWP lives in `qwp_*.go`. The buffer (`qwp_buffer.go`), encoder
(`qwp_encoder.go`), wire primitives (`qwp_wire.go`), and transport
(`qwp_transport.go`) form the columnar codec stack. The sender (`qwp_sender.go`
+ `qwp_sender_cursor.go`) implements `LineSender` and `QwpSender` on top of it.

**All wire I/O — memory-backed *and* disk-backed — goes through the cursor
engine + send loop** in `qwp_sf_*.go`. `sf_dir` empty selects memory-backed
segments; set selects disk-backed under `<sf_dir>/<sender_id>/*.sfa` (that
per-sender directory is itself the slot — there is no extra slot level),
on-disk-compatible with the Java client's `MmapSegment.java`. The producer
encodes a batch into `qwpSfCursorEngine` via `engineAppendBlocking`; the
`qwpSfSendLoop` goroutine drains it to the WebSocket, parses ACKs, advances
`engineAckedFsn`, and owns reconnect + replay from `engineAckedFsn() + 1`.

**Cursor frames are self-sufficient** — full schema definitions plus the full
symbol dictionary from id 0, every flush. This is what makes
reconnect/replay/orphan-adoption safe across a fresh server connection.

**The wire carries no schema id and no schema mode byte.** A table block is
`table_name, row_count, col_count, inline columns, column data`; the inline
column definitions are the authoritative schema, repeated on every frame. There
is no `nextSchemaId` accumulator on the sender, no per-table `schemaId` field on
the table buffer, no schema-change detection, and no reference mode. (QWP once
carried a mode byte + schema id plus a schema-reference optimisation; it was
removed across the server and all clients.) On egress, the decoder parses the
schema from the first `RESULT_BATCH` of a query (`batch_seq == 0`) into
`qwpQueryDecoder.querySchema` and reuses it for that query's continuation
batches; `qwpEgressIO.dispatcherRun` calls `resetQuerySchema` at the start of
every query so a schema never leaks across query boundaries.

Symbol-dict tracking (`maxSentSymbolId`, `batchMaxSymbolId`) is still in place,
and both fields are load-bearing. The encoder always passes `-1` as the
`maxSentId` arg of `encodeMultiTableWithDeltaDict` to force "full dict from id

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [questdb/go-questdb-client](https://github.com/questdb/go-questdb-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
