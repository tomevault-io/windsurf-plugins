---
trigger: always_on
description: This file is context for future AI agents (and humans). Read it before changing
---

# AGENTS.md — briefing for AI sessions working on tapecap

This file is context for future AI agents (and humans). Read it before changing
anything. It captures the *why* and the non-obvious gotchas that aren't visible
from the code alone.

## What this project is

`tapecap` is a macOS command-line tool that captures the **raw** bitstream a
DV or HDV camcorder/deck sends over FireWire (IEEE 1394) and writes it to disk
untouched:

- **DV** (IEC 61883-2) → raw DIF stream (`.dv`)
- **HDV** (IEC 61883-4) → raw MPEG-2 Transport Stream (`.m2t`), including the
  MPEG-1 Layer II audio and PSI/metadata.

### The core insight (why the project exists)

On macOS, **AVFoundation deliberately demuxes HDV** and only exposes the MPEG-2
*video* elementary stream, dropping the **audio** and the transport-stream
metadata. (DV is fine through AVFoundation because DV audio rides inside the DIF
frames.) The data is all on the wire, though — Premiere captures full HDV.

The fix, and what this tool does: **bypass AVFoundation and read the raw
isochronous IEC 61883 stream straight from IOKit's FireWire stack**, using
Apple's **AVCVideoServices** framework (`MPEG2Receiver` for HDV, `DVReceiver`
for DV, `AVCDevice` for connection management + AV/C control). This is the same
layer Premiere uses. See `docs/BACKGROUND.md` for the full write-up.

## Repository layout

| Path | What it is | Edit it? |
|---|---|---|
| `third_party/AVCVideoServices/` | **Vendored Apple sample code** — the isochronous capture engine (~17.5k LOC). | Avoid. It's proven; treat as a black box. |
| `src/tapecap.cpp` | The CLI: `list` / `info` / `capture`, orchestration, AV/C commands, live status, auto-naming (~900 LOC). | Yes — this is "our" code. |
| `src/dvmeta.{h,cpp}` | DV/HDV recording-date + tape-timecode parser (~400 LOC). | Yes. |
| `Makefile` | Universal (arm64+x86_64) build. | Yes. |
| `.github/workflows/build.yml` | CI build + smoke test; release job. | Yes. |
| `docs/BACKGROUND.md` | Deep technical explanation. | Keep current. |
| `third_party/.../NOTICE.md` | Provenance + license of the vendored code. | Keep accurate. |

### About the vendored AVCVideoServices

- Source: Apple **FireWire SDK 26**, `Examples/AVCVideoServices-42`, obtained
  from the Internet Archive (item `13572357-firewiresdk-26`); Apple no longer
  ships it. SHA + details in `third_party/AVCVideoServices/NOTICE.md`.
- License: Apple sample-code license — **redistribution allowed with the notice
  retained**. Every file keeps its original header.
- Only the **capture path** was vendored (22 `.cpp` + their headers). The
  non-vendored upstream modules (PanelSubunitController, MusicSubunitController,
  MPEGTrickModes, the `Virtual*` device-emulation classes, FWA_IORemapper, the
  flat `FWAVC` C API, and all `*Test`/sample `main()`s) are unreferenced by the
  capture path.
- Vendored-source edits are intentionally small:
  - `AVCVideoServices.h`: `#include`s for non-vendored modules were removed, and
    one constant (`kMaxAutoPSIDetectProgramIndex`, normally in MPEGTrickModes.h)
    was hoisted in so the compiled-but-unused `MPEG2Transmitter.cpp` still
    builds. The transmitter compiles as dead code (the managed `AVCDevice` API
    references it) but tapecap never transmits.
  - `DVReceiver.cpp`: DV frame buffers are sized for the largest known DV frame,
    and the receiver follows the actual CIP `dvMode` when a deck reports one DV
    mode via AV/C but sends another on the bus (seen on Sony HDV-VCR with PAL DV
    footage misreported as NTSC DV).

## Hard platform constraints

- **macOS 11–15 only.** macOS 15 (Sequoia) is the **last** release that ships
  the FireWire driver (`IOFireWireFamily`) and the `IOKit/firewire` + `IOKit/avc`
  SDK headers. macOS 26 (Tahoe) removed FireWire entirely. There is no path
  forward; this is a frozen, end-of-life platform.
- **CI must pin `runs-on: macos-15`.** Do NOT use `macos-latest` — it now points
  at macOS 26, whose SDK lacks the FireWire headers, so the build fails there.
- **Universal binary** (arm64 + x86_64). Apple Silicon has no FireWire port —
  users connect via an Apple Thunderbolt-to-FireWire adapter. (The 16 KB page
  size on Apple Silicon was a worried-about risk that never materialized;
  capture works on M1 Max + macOS 15.)
- Relies on **deprecated-but-present** IOKit FireWire isochronous APIs (DCL
  programs). They work through macOS 15.

## Behaviour & design decisions you must not regress

These were learned from real hardware (a **Sony HDR-HC9**); don't "simplify"
them away:

1. **Format auto-detect uses the output-plug (oPCR) signal format, not the tape
   signal-mode.** Some HDV camcorders (the HC9) report **DV** via the tape
   subunit's OUTPUT SIGNAL MODE status *even while streaming HDV*. So
   `DetectFormat()` queries the oPCR (AV/C OUTPUT PLUG SIGNAL FORMAT, opcode
   `0x19`): top byte `0xA0` = HDV/MPEG-TS, `0x80` = DV. Falls back to the
   capability flags, then to a play-then-recheck. `--format dv|hdv` overrides.
2. **Device auto-selection is lenient.** It returns the first enumerated
   tape/DV/HDV device (else the first device), *without* gating on `isAttached`
   or the async capability flags — otherwise `capture` fails to find a device

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xingrz/tapecap](https://github.com/xingrz/tapecap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
