---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
go build -o junto ./cmd/junto   # build the binary
go test -race ./...              # run all tests (CI requirement)
go test -race ./internal/transfer/... -run TestFallback  # run a single test
go vet ./...                     # static analysis (CI requirement)
gofmt -l .                       # check formatting; must be empty to pass CI
gofmt -w .                       # auto-format
```

Version info is injected at release time via `-X main.version=...` ldflags (see `.goreleaser.yaml`). Dev builds report `version = "dev"`.

## Architecture

junto is a leaderless watch-party coordinator. Two roles: **host** (`junto create`) and **joiner** (`junto join`). The host serves media files; joiners stream them P2P while watching in sync.

### Control plane: Nostr relays

All coordination (room hello/goodbye, playback state heartbeats, WebRTC signals) travels as NIP-44-encrypted ephemeral events over public Nostr relays (`internal/nostrx`). Each session generates a throwaway keypair; the room code encodes a 128-bit secret from which both the public room ID (relay filter tag) and the NIP-44 conversation key are deterministically derived (`internal/room`).

### Data plane: WebRTC

Media files transfer over a single persistent WebRTC data channel per file (`internal/transfer`). Signaling travels through the Nostr transport. ICE is **half-trickle** (`iceReader` in `transfer.go`): the offerer still sends a complete offer (it waits for gathering) but advertises `Signal.Trickle`; a capable answerer then sends its answer immediately and trickles its candidates as separate `"candidate"` signals, so the answer-side gather is off the connect path. Against a peer that doesn't advertise trickle (an old build), the answerer falls back to the non-trickle path (wait for gathering, one complete answer, no candidate messages) — the capability gate is required because old peers reject the unknown `"candidate"` signal kind. Offer-side trickle is a later phase (`Role` tag is already on the wire for it). The serving peer answers byte ranges on demand; the fetching peer requests ranges via JSON control frames on the data channel (schema documented on `ctrl` in `transfer.go`). NAT fallback: if STUN hole-punching fails, the downloader silently retries with `FallbackRelays` (populated at session start by `transfer.InitBuiltinRelay`).

Every byte is verified per 256 KiB chunk group against the host's announced BLAKE3/Bao root before it reaches disk (`internal/transfer/verify.go`; the host computes root + outboard tree at create, joiners fetch the tree once over the channel). Rooms whose `FileMeta` lacks the `Bao*` fields (old hosts) fall back to the legacy whole-file SHA-256 check.

**Swarm** (`internal/transfer/swarm.go`): peers advertise verified piece coverage on heartbeats (`Have`/`HaveDone` on `MsgState`); a streaming joiner runs one primary source (the host) plus up to two aux sources fetching piece-sized claims — window assist behind the primary's stride, rarest-first beyond the window. A claim table on `fileEntry` deconflicts sources; per-source preempt channels handle seeks. Any peer serves what it verifiably has (`ServeFromStore`, coverage-aware with `nak` for missing ranges); a completed streaming joiner turns host-eligible via `syncer.Deps.CanHostDynamic`. With no advertising peers the path is identical to the single-source downloader.

**Upload fairness** (`internal/transfer/fairness.go`): a serving node runs one `UploadFairness` across its concurrent viewers. Each viewer reports buffer depth (bytes ahead of its playhead) in additive `buf` data-channel frames (`Downloader.reportCushion`); a comfortably-buffered `sender` calls `fair.pace` before each chunk and briefly yields while a same-file viewer is close to stalling (capped, file-scoped, self-expiring). No-op with one viewer or when viewers don't report; degrades to the un-prioritized path. Stacks on top of the swarm (same node, same policy, more sources).

### Streaming playback path

When a joiner has no local files: a local HTTP range server (`internal/streamserver`) exposes in-progress downloads at `http://127.0.0.1:<ephemeral>/0`, `/1`, … and blocks reads on bytes not yet downloaded. mpv is pointed at these URLs. The downloader (`transfer.Downloader`) follows the playhead with an adaptive window (32–512 MB) and redirects on seek.

### Sync engine

`internal/syncer` is the session loop. It runs on both host and joiner. Design: **leaderless last-writer-wins** — every explicit user action (pause, play, seek, speed, audio track, subtitle track/delay, playlist jump) carries a version (wall-clock ms). The action with the highest version wins everywhere; ties break on largest pubkey so corrections flow one way. Between actions, 2-second heartbeats carry `PlayState` and double as drift beacons. Clock skew is estimated NTP-style on join and subtracted from every position projection. Drift > 0.5 s triggers a hard seek.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swayam-mishra/junto](https://github.com/swayam-mishra/junto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
