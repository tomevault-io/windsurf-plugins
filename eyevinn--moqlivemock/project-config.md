---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

## Project Overview

moqlivemock is a Go-based MoQ (Media over QUIC) live streaming mock implementation. It provides:

- **mlmpub**: A publisher that serves live media content over MoQ Transport
- **mlmsub**: A subscriber that receives and processes media streams

## Build Commands

```bash
go build ./...              # Build all packages
go test ./...               # Run all tests
go mod tidy                 # Update dependencies
go mod vendor               # Vendor dependencies
```

## Architecture

### Key Components

- `cmd/mlmpub/` - Publisher application serving media tracks (video, audio, subtitles)
- `cmd/mlmsub/` - Subscriber application receiving media
- `cmd/mlmtest/` - Interop test client for [moq-interop-runner](https://github.com/englishm/moq-interop-runner)
- `cmd/locmaf/` - LOCMAF reference-asset generator and `roundtrip` validator
- `internal/` - Shared internal packages:
  - `asset.go` - Asset loading and track management
  - `catalog.go` - MSF/CMSF catalog generation (CMAF, LOC, LOCMAF packaging)
  - `subtitle.go` - Dynamic subtitle generation (WVTT/STPP)
  - `moqgroup.go` - MoQ group/object handling
  - `locmaf.go` - LOCMAF full-moof encode/decode and the `moov`→`initData` codec
  - `locmaf_delta.go` - `MoofDeltaCompressor` / `MoofDeltaDecompressor` keeping
    the per-track previous-moof state used for delta moofs
- `docs/LOCMAF.md` - LOCMAF packaging design, wire format, and version policy

### MoQ Transport Dependency

Uses `github.com/Eyevinn/moqtransport` (forked from mengelbart/moqtransport) with draft-14 and draft-16 support.
Version negotiation uses ALPN (`moqt-16` for draft-16, `moq-00` for draft-14) and `WT-Available-Protocols` for WebTransport.
- Session creation: `&moqtransport.Session{Handler: ..., SubscribeHandler: ...}`
- Call `session.Run(conn)` to start
- Subscriptions use separate `SubscribeHandler` interface

### Multi-Namespace Architecture

mlmpub announces one or more namespaces. CMSF and LOCMAF namespaces each have
their own catalog filtered by protection type; LOC uses an MSF catalog; moq-mi
is catalogless.

CMSF (CMAF chunks):
- `cmsf/clear` — always present, clear (unencrypted) tracks
- `cmsf/drm-{scheme}` — when `-drmpath` is set, commercial DRM tracks (`_drm` suffix)
- `cmsf/eccp-{scheme}` — when `-kid`/`-iv` are set, ClearKey/ECCP tracks (`_eccp` suffix)

LOCMAF (Low Overhead CMAF — LOC-style key-value encoding of `moof`/`moov`):
- `locmaf/clear` — always present, clear tracks
- `locmaf/drm-{scheme}` — when `-drmpath` is set
- `locmaf/eccp-{scheme}` — when `-kid`/`-iv` are set
- Tracks carry `packaging: "locmaf"` and a `locmafVersion` string
  (currently `"0.1"`, tracked by the `internal.LocmafVersion` constant)

### LOCMAF specification versioning

The LOCMAF wire format and behaviour are versioned independently of
moqlivemock releases. When the version changes (`internal.LocmafVersion`,
the `Revision history` table in `docs/LOCMAF.md`, and the document version
banner near the top of `docs/LOCMAF.md`):

1. Bump `internal.LocmafVersion` and the docs in the same commit.
2. After that commit lands on `main`, tag it with an annotated tag
   `locmaf-v<MAJOR>.<MINOR>` so the spec is permanently citable as
   `docs/LOCMAF.md@locmaf-v<MAJOR>.<MINOR>`:

   ```
   git tag -a locmaf-v0.2 <sha> -m "LOCMAF specification v0.2"
   git push origin locmaf-v0.2
   ```

   `locmaf-v0.1` was tagged on commit `d5c5e04`.

Tags use the `locmaf-v` prefix to keep them separate from moqlivemock
release tags (`vX.Y.Z`).

LOC (raw codec frames, one per object) and moq-mi (catalogless):
- `msf/clear` — LOC packaging (AVC + AAC/Opus, HEVC for `hev1.*`)
- `moq-mi/clear` — moq-mi packaging with fixed track names `video0` / `audio0`

Key types in `internal/pub/pub.go`:
- `NamespaceEntry` — pairs a namespace tuple with its catalog
- `Handler.Namespaces []NamespaceEntry` — all announced namespaces

Key types in `internal/asset.go`:
- `ProtectionType` — enum: `ProtectionNone`, `ProtectionDRM`, `ProtectionECCP`
- `ContentTrack.Protection` — identifies how a track is encrypted
- `Asset.Drm` / `Asset.Eccp` — independent DRM configs

`GenCMAFCatalogEntry(namespace, protectionType, timestamp)` generates a CMSF or
LOCMAF catalog filtered to the specified protection type.

### Handler Pattern

Publishers implement:
- `Handler` - for ANNOUNCE messages
- `SubscribeHandler` - for SUBSCRIBE messages (returns `*SubscribeResponseWriter`)
- `FetchHandler` - for FETCH messages (returns `*FetchResponseWriter`)

Subscribers implement:
- `Handler` - for ANNOUNCE messages (accept/reject)
- `SubscribeHandler` - for SUBSCRIBE messages (typically reject as they don't publish)

### Subtitle Tracks

Subtitles are dynamically generated (not loaded from files) showing UTC time and group number:
- **WVTT** (WebVTT in CMAF) - codec: `wvtt`, uses `mp4.VttcBox`/`mp4.PaylBox`
- **STPP** (TTML in CMAF) - codec: `stpp.ttml.im1t`, uses Go templates (`stpptime.xml`)

Key types in `internal/subtitle.go`:
- `SubtitleTrack` - track configuration (format, language, timing)
- `SubtitleData` - implements `CodecSpecificData` interface for init segments

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eyevinn/moqlivemock](https://github.com/Eyevinn/moqlivemock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
