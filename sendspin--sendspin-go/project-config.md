---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
This guidance is aimed at Claude Code but may also be suitable for other AI tooling, such as GitHub Copilot and OpenAI Codex.

## Project Overview

`sendspin-go` is the Go implementation of the [Sendspin Protocol](https://github.com/Sendspin/website/blob/main/src/spec.md) for synchronized multi-room audio streaming. It ships as a library (`pkg/`) plus two CLI binaries: `sendspin-player` (root `main.go`) and `sendspin-server` (`./cmd/sendspin-server`). The Python sibling is [`aiosendspin`](https://github.com/Sendspin/aiosendspin); the two implementations are wire-compatible and share the role-family vocabulary (`player`, `controller`, `metadata`, `artwork`, `visualizer`).

**Note**: If uncertain about how something in Sendspin is supposed to work, fetch and refer to the [protocol specification](https://github.com/Sendspin/website/blob/main/src/spec.md) for authoritative implementation details.

## Commands

Native deps: `libopus` only. FLAC is pure-Go (`mewkiz/flac`); the Makefile, CI, and release pipelines all build with `GOFLAGS=-tags=nolibopusfile` so `gopkg.in/hraban/opus.v2`'s `opus.Stream` parts (the only consumer of `libopusfile`) are skipped and the binary doesn't link `libopusfile` at runtime. Use `./install-deps.sh` (handles brew/apt/dnf/pacman) or the per-OS commands in README.md. `ffmpeg` is only required for HLS/m3u8 server input. If you ever need to build with the opus.Stream API, override the tag: `make BUILDTAGS= test`.

```bash
make                            # Build sendspin-player + sendspin-server
make player                     # Build sendspin-player only
make server                     # Build sendspin-server only
make test                       # go test ./...
make test-coverage              # -race + HTML coverage report
make lint                       # golangci-lint --timeout=5m
make conformance                # Run protocol conformance suite (clones ../conformance on first run; needs uv)
go test ./pkg/sendspin -run TestPlayer_Connect -race -v   # Single test
```

Pre-commit (`.pre-commit-config.yaml`) runs gofmt, goimports, go-mod-tidy, golangci-lint, and `go test -race -v ./...` on every commit. Run pre-commit before pushing any commit to ensure it is valid.

## Architecture

Library-first. The public API in `pkg/` is what consumers import; the CLI binaries are thin wrappers. `internal/` is private to the module by Go's rules — only `pkg/sendspin` legitimately reaches into it.

### Public layers (`pkg/`)

- **`pkg/sendspin`** (`receiver.go`, `player.go`, `server.go`): High-level API. The library splits the player into a `Receiver` (connect, handshake, clock sync, decode, schedule — emits `<-chan audio.Buffer`) and a `Player` (a thin wrapper composing `Receiver` + `pkg/audio/output.Output`). Use `Receiver` directly for visualizers, DSP, or custom output backends — no audio device required. Each `Receiver`/`Player` owns its own `*sync.ClockSync`; the package-level `SetGlobalClockSync` / `ServerMicrosNow` shims are deprecated.

- **`pkg/audio`** (`types.go`, `resample.go`, plus `decode/`, `encode/`, `output/` subpackages): Core `Format` and `Buffer` types, sample conversion, linear resampler, codec encoders/decoders (PCM, Opus, FLAC). `output` ships a single `malgo` backend; the `oto` backend was removed in v1.2.

- **`pkg/protocol`** (`messages.go`, `client.go`, `server_conn.go`): Wire messages plus `Client` (player side) and `ServerConn` (CGO-free helpers for serving binary frames).

- **`pkg/sync`** (`clock.go`, `timefilter.go`): `ClockSync` plus `TimeFilter`, the 2D Kalman filter tracking offset and drift per the Sendspin time-filter spec. This is what makes hi-res multi-room sync actually work; do not regress it without re-running `make conformance`.

- **`pkg/discovery`** (`mdns.go`): mDNS browse/advertise.

### Server-side group / role model (`pkg/sendspin`)

Two-level architecture, mirroring `aiosendspin` so the wire behavior matches across implementations.

- **`Group`** (`group.go`): Owns the typed event bus for one playback group. Publishes `ClientJoinedEvent`, `ClientLeftEvent`, `ClientStateChangedEvent`, `GroupStateChangedEvent`, etc. `ClientJoinedEvent` carries a live `*ServerClient` (the connection is fully alive at publish); `ClientLeftEvent` intentionally drops the pointer because the client is mid-teardown.

- **`GroupRole`** (`group_role.go`): One implementation per role family, coordinating across all member roles in the group. Built-in implementations: `ControllerGroupRole` (`role_controller.go`), `MetadataGroupRole` (`role_metadata.go`), `PlayerGroupRole` (`role_player.go`). Add new server-side behavior by writing a new `GroupRole` and registering it via `activateRoles`, **not** by editing `server_dispatch.go` directly.

- **`ServerClient`** (`server_client.go`): Per-connection state with typed accessors (`State()`, `Volume()`, `Muted()`, `Codec()`). Message dispatch lives in `server_dispatch.go`, audio streaming in `server_stream.go`, and per-client send-ahead pacing in `buffer_tracker.go`.

### Audio Pipeline

```
Server: AudioSource → codec negotiation per client (Opus/FLAC/PCM)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sendspin/sendspin-go](https://github.com/Sendspin/sendspin-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
