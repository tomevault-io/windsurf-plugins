---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Tile38 client that speaks RESP over `net.Conn` directly. It has **no runtime
dependencies** — that is the point of the repository, not an accident. Do not
add one without being asked; testcontainers is a test-only dependency and stays
behind the `integration` build tag.

Speaking the protocol directly is what makes live geofences possible: a Redis
client cannot hold a connection open while the server streams events into it.

## Commands

```bash
make help                                       # list targets
make lint                                       # golangci-lint, both tag sets
make test                                       # unit tests, no Docker needed
make test-integration                           # against tile38/tile38:edge via testcontainers
make test-all

go test -run TestFenceStream ./...              # a single unit test
go test -tags=integration -run TestLiveFence ./...
```

The supported server is pinned in `.version`, which `TestMain` reads and
`TILE38_IMAGE` overrides. It is an `edge` digest rather than a release tag
because A5 is merged into upstream master but ships in no tag yet — `1.38.0` has
no `internal/server/a5.go`. Pinning the digest keeps a push to upstream master
from silently changing what CI tests against; the `Tile38 Upstream Check`
workflow opens an issue when a release lands or the digest moves, so the pin
gets bumped deliberately rather than drifting.

Integration tests are behind the `integration` build tag, so a plain
`go vet ./...` or `golangci-lint run ./...` skips that file entirely — always
run both tag sets, which is what `make lint` and `make vet` do.

The linter version is pinned in `.golangci-version`, which the CI workflow reads
and `make lint` checks the local binary against before running. It was `latest`
in CI once, so a new golangci-lint release silently changed what CI enforced and
a clean local run stopped predicting it — which is how a `goconst` finding got
past review. Bump the file to adopt a new version deliberately, the same way
`.version` handles the server.

`.golangci.yml` follows the sibling `common` repo. Its exclusions are
deliberate and explained inline: complexity metrics are waived for `resp.go`
(flat protocol switches) and `dupl` for the parallel search builders. Prefer
fixing a finding over widening those.

## Layout

A Go package is a directory, so the whole public API is `package tile38` in the
repo root — it cannot be split across folders without changing the import path.
The root therefore holds **only public API**, and the wire plumbing lives under
`internal/`:

```
tile38.go        Client, New + Option funcs, Close/Ping/Do — transport façade
commands.go      command entry points (c.Set, c.Nearby, …)
types.go         result types
builders.go      write, read, search, hook builders + protocol token types
intersects.go    INTERSECTS builder
geometry.go      Area values + the TEST builder
server.go        STATS, CONFIG, GC, HEALTHZ, READONLY, FOLLOW, TIMEOUT
channel.go       SETCHAN / channel builders
json.go          JSET / JGET / JDEL builders
management.go    DROP, RENAME, TTL, KEYS, BOUNDS, … builders
stream.go        Stream (live fences and subscriptions)
fence_event.go   FenceEvent
parse.go         reply → result-type decoding
internal/resp/   RESP codec: AppendCommand, ReadReply, Error
internal/conn/   Conn and Pool — dialling, pooling, deadlines, pipelining
```

`internal/resp` depends on nothing; `internal/conn` depends on `resp`; the root
depends on both. Keep it that way — `parse.go` stays in the root precisely
because it decodes into public result types, and moving it down would create an
import cycle.

`ServerError` is a type alias for `resp.Error` so the internal codec can return
it while callers still `errors.As` against the public name.

## Architecture

**Builders** (`builders.go`, `intersects.go`, `channel.go`, `json.go`,
`management.go`). Every command is a struct holding `args []any` plus a
`*Client`. Chained methods append tokens; a terminal method (`Do`, `Count`,
`Fence`) sends them. Adding a command means: entry point in `commands.go`,
builder type next to its siblings, parse helper in `parse.go` if the reply
shape is new.

**Search builders are generic over their element type.** `NearbyCmd[E]`,
`WithinCmd[E]`, `IntersectsCmd[E]`, `ScanCmd[E]` and `SearchCmd[E]` all hold a
shared `*searchState` plus a `format[E]`, and one `Do(ctx) ([]E, error)` serves
every output format. An output-format method — `Points`, `Objects`, `Rects`,
`Hashes`, `A5Cells`, `PointsWithDistance`, `Strings`, `IDs` — returns the *same
builder at a different `E`*; a fresh command is `[string]`, which emits `IDS`.

The parameter is the **element**, not the page, and that is load-bearing: it is
what lets `Iter` be a method. A method cannot declare type parameters of its own
(`method must have no type parameters`; golang/go#49085), so the only names a
method signature can use are the receiver's. With the page as the parameter, an
element-at-a-time `iter.Seq2[E, error]` is unspellable — `undefined: E` — and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GO-VIRTUAL-bv/tile38.go](https://github.com/GO-VIRTUAL-bv/tile38.go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
