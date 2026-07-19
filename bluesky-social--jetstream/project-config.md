---
trigger: always_on
description: Jetstream is a full-network archive and live-streaming service for atproto. Backfill is served as HTTPS segment-file downloads; live tail is the same JSON websocket protocol as Jetstream v1.
---

# AGENTS.md

## Orientation

Jetstream is a full-network archive and live-streaming service for atproto. Backfill is served as HTTPS segment-file downloads; live tail is the same JSON websocket protocol as Jetstream v1.

- `README.md` covers running the app, tests, and the simulator.
- `docs/*` is for documentation that is intended to be read by humans and agents alike
- `docs/README.md` is the source of truth for the system. Read it before any non-trivial change, especially anything touching the on-disk segment format.
- This file is the team's coding conventions. It overrides anything inferred from existing code.
- `specs/*` is documentation intended to be read only by agents
- `specs/notes/*` are specs and plans documentation that catalogs our train of thought while working on tasks as we go

Agent-facing living docs, in a good reading order for getting oriented:

- `specs/architecture.md` — the high-level map: the big subsystems, how they fit, and a "where to look" table routing each topic to its authoritative source. Start here.
- `specs/invariants.md` — the short list of rules that must never break. Read before changing anything on the ingest, storage, or serve paths.
- `specs/glossary.md` — one-line definitions of the terms that show up everywhere.
- `specs/gotchas.md` — accepted limitations and hard-won lessons: things that look like bugs but are deliberate, and mistakes not worth making twice.
- `specs/client.md` — the client protocol end to end: archive negotiation, download/decode, cutover, live tail, wire compression, and the failure modes at each seam. Read before changing `internal/client`, the module-root API, or anything on the /subscribe-v2 wire contract.
- `specs/oracle.md` — the source of truth for the oracle/simulator testing rig.
- `specs/mutation.md` — how the mutation campaign measures the oracle's bug-detection power.

These summarize and route; `docs/README.md` and each package's `doc.go` remain authoritative. When a living doc disagrees with them, fix the living doc.

## Repo layout

```
cmd/
  jetstream/      main binary: serve, inspect-segment, timestamp import, version
  simulator/      local PLC + PDS + Relay on :7777
segment/          on-disk segment file format (header, blocks, footer, reader, writer, sealer); public API
internal/
  ingest/         the segment Writer (append/flush/seal, seq, readable log)
    backfill/     initial full-network backfill (listRepos + getRepo)
    live/         live firehose consumer (subscribeRepos)
    orchestrator/ ingestion lifecycle state machine + merge/cutover
    syncstate/    sync 1.1 resync bookkeeping
  subscribe/      websocket /subscribe endpoint (v1 protocol parity) + cold reader
  xrpcapi/        archive download over HTTP/XRPC (planBackfill, getSegment, getBlock)
  client/         thick Go client: archive negotiation, fold, cutover to live
  server/         HTTP listeners (public :8080, opt-in debug :6060) and middleware
  store/          pebble-backed cursor + metadata store
  manifest/       segment manifest (directory scan + self-describing headers)
  tombstone/      delete/update/account tombstone set for compaction
  timestamp/      operator timestamp-import pipeline
  importer/       import job manager
  repoexport/     reconstruct a repo CAR/MST from archived events
  identity/       DID resolution
  status/         /status endpoint collector
  diskspace/      data-dir free-space accounting
  crashpoint/     deterministic crash-injection seams (test-gated)
  simulator/      fake atproto network: world (traffic), http (PLC/PDS/relay), fanout
  oracle/         end-to-end correctness harness (see specs/oracle.md)
  corpus/         real-data corpus tests (independent of the lifecycle oracle)
  format/         shared wire/format helpers
  obs/            metrics, tracing, slog setup
  lifecycle/      graceful start/stop helpers
  jetstreamd/     process runtime wiring (options, startup, shutdown)
  version/        build version stamp
  web/            static debug UI assets
```

Atproto lexicon JSON (authoritative for XRPC and record schemas) lives at `~/go/src/github.com/bluesky-social/atproto/lexicons` on dev machines.

## Working in the codebase

The justfile is the single source of truth for build/test/lint. Prefer `just` recipes over invoking `go test` / `golangci-lint` directly so behaviour matches CI.

Frequently useful beyond what's in the README:

```sh
just test ./segment -run TestX  # one test (gotestsum forwards args after `--`)
just bench ./segment            # benchmarks
just fuzz 30s ./segment         # fuzz every Fuzz* target for 30s each
just modernize                  # apply gopls modernize rewrites
```

Oracle tests live in `internal/oracle` and compare Jetstream's durable output against a simulator model. Run them after changes to ingest, segment persistence, lifecycle/orchestrator phases, cursor handling, or restart recovery:

```sh
just test ./internal/oracle                                      # fast short-mode oracle checks
just test-long ./internal/oracle -run TestOracle_Restart -v      # non-short restart/recovery oracle

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bluesky-social/jetstream](https://github.com/bluesky-social/jetstream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
