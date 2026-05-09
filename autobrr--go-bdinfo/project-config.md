---
trigger: always_on
description: This project is a Go implementation of BDInfo, a tool for analyzing Blu-ray disc structures. The original is written in C# and we're creating a pure Go version without CGO dependencies.
---

# go-bdinfo Project Documentation

## Overview
This project is a Go implementation of BDInfo, a tool for analyzing Blu-ray disc structures. The original is written in C# and we're creating a pure Go version without CGO dependencies.

**IMPORTANT**: Always refer to the original C# source code for implementation guidance. Workspace convention:
- C# reference source: `~/github/oss/BDInfo-src/BDInfo.Core/BDCommon/rom/` (clone UniqProject/BDInfo if missing)

## Parity Loop (Official BDInfo)
Loop-to-done: generate official report + our report for the same disc path (folder or ISO), `diff -u --text`, fix mismatches, add regression tests.

Commands:
```bash
off=~/github/oss/bdinfo-official/bdinfo_linux_v2.0.5_extracted/BDInfo
disc="/mnt/storage/torrents/<disc-folder-or-iso>"
out=/tmp/bdinfo-parity
mkdir -p "$out"

"$off" -p "$disc" -o "$out/official.txt"
go build -o /tmp/bdinfo ./cmd/bdinfo
/tmp/bdinfo -p "$disc" -o "$out/ours.txt"

diff -u --text "$out/official.txt" "$out/ours.txt"
```

Notes:
- IO: don’t sweep all of `/mnt/storage/torrents*`; sample a few discs per type.
- ISO/UDF: BD-ROM ISOs commonly use a metadata partition map and multi-extent files; UDF reads must be concurrency-safe (use `ReadAt`-based access, no shared `Seek`).
- Speed loop: always measure official vs ours on the same sample path and compare wall time with exact command logs.
- Current perf policy: stream scans default to 1 worker (override with `BDINFO_WORKERS`) to avoid seek thrash on this storage profile.
- Harness: `scripts/speed_parity_loop.sh --disc "<disc-or-iso>" --reps 3` (matched toggles, per-rep parity check, median ratio).
- Diagnostics parity loop: derive stream diagnostics order from PMT stream order probe (`detectPMTStreamOrder`) with scan/CLPI fallback; verify on both anchors:
  - Network UHD (`00007/00009` hidden DV ordering)
  - Excalibur UHD (`00004` DV + audio/PGS ordering)
- Playlist same-language ordering parity: for English audio/graphics/text streams of same type, keep PID ascending (regression anchor: `The.Man.Who.Wasnt.There...`, 39.435 kbps subtitle before 68.796 kbps).
- Perf hotspot loop: if Network-like discs regress, check `internal/bdrom/streamfile.go` clip-target matching path first (active target cursor), then re-run harness.
- Sample cadence: smoke with `--reps 1` on ISO + Static + Network, then `--reps 3` on the regressing sample.
- Debug helper: `go run ./cmd/debugudf -iso "<path>.iso"` (lists key dirs/files, sanity-checks headers/sizes).

The C# code serves as the authoritative reference for:
- Binary format specifications
- Parsing algorithms
- Codec analysis logic
- Output format compatibility
- Edge case handling

## Original BDInfo Features
- Scans Blu-ray discs (Full HD, Ultra HD, 3D) from folders or ISO files
- Analyzes video, audio, and subtitle streams
- Detects codecs, bitrates, and technical specifications
- Generates detailed reports in various formats
- Supports BD+, BD-Java, and other disc features
- Includes BDExtractor for ISO extraction

## Architecture Mapping (C# to Go)

### Core Components

#### 1. BDROM (BDCommon/rom/BDROM.cs → pkg/bdrom/)
- **Purpose**: Main class representing a Blu-ray disc
- **Key Methods**:
  - `Scan()`: Discovers and analyzes disc structure
  - `ScanStreamFiles()`: Analyzes M2TS files
  - `ScanPlaylistFiles()`: Processes MPLS files
- **Go Implementation**: 
  - Use interfaces for file system abstraction
  - Implement parallel scanning with goroutines
  - Replace events with channels for progress reporting

#### 2. Stream Parsing (TSStreamFile.cs → pkg/stream/)
- **Transport Stream Parser**: 
  - Reads 192-byte packets (188 + 4 timestamp)
  - Extracts PAT/PMT tables
  - Identifies elementary streams
- **Key Algorithms**:
  - Sync byte detection (0x47)
  - PID filtering
  - PES packet assembly
  - PCR timestamp extraction

#### 3. Codec Analyzers (TSCodec*.cs → pkg/codec/)
Each codec has specific parsing logic:
- **H.264/AVC**: NAL unit parsing, SPS/PPS extraction
- **H.265/HEVC**: VPS/SPS/PPS, HDR metadata
- **DTS/DTS-HD**: Sync word detection (0x7FFE8001), channel config
- **AC-3**: Sync word (0x0B77), bitrate tables
- **TrueHD**: Major sync detection, Atmos flags

#### 4. Playlist Files (TSPlaylistFile.cs → pkg/bdrom/playlist.go)
- **MPLS Format**: Binary format with header "MPLS0100/0200/0300"
- **Key Structures**:
  - PlayList section: clip references
  - PlayItem section: stream selections
  - STN_table: stream attributes

#### 5. Clip Info Files (TSStreamClipFile.cs → pkg/bdrom/clipinfo.go)
- **CLPI Format**: Contains pre-analyzed stream info
- **Sections**:
  - ClipInfo: basic metadata
  - Sequence Info: stream properties
  - Program Info: stream PIDs and types

### Binary Parsing Utilities

#### TSStreamBuffer (→ pkg/buffer/bitreader.go)
Critical for codec parsing:
```go
type BitReader struct {
    data []byte
    pos  int
    bits int
}

// Key methods needed:
// - ReadBits(n int) uint64
// - ReadByte() byte
// - ReadUInt16() uint16
// - ReadUInt32() uint32
// - ReadExpGolomb() int (for H.264/H.265)
// - Skip(n int)
// - BytesLeft() int
```

### File System Abstraction

#### Interfaces (IO/*.cs → pkg/fs/)
```go

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [autobrr/go-bdinfo](https://github.com/autobrr/go-bdinfo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
