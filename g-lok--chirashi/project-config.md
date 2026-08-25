---
trigger: always_on
description: Pure Go CLI for converting sliced instrument formats. Parses/encodes REX/RX2/RCY in-process (no SDK, no CGo, no Zig).
---

# AGENTS.md — chirashi agent instructions

## Project overview

Pure Go CLI for converting sliced instrument formats. Parses/encodes REX/RX2/RCY in-process (no SDK, no CGo, no Zig).

- **Go**: cmd/, internal/engine/, internal/engine/rex2/, tests/
- **No Zig, no C, no CGo, no REX SDK**

REX2 pure Go implementation: `internal/engine/rex2/` — IFF parser, DWOP decoder/encoder, slice model, legacy PTI/OT fallback, REX1 detection.

## History / Why Pure Go

### REX SDK → Pure Go migration (v0.4.0)

**Why we abandoned the REX SDK:**
- **macOS Intel + Zig dynlb issue**: Zig 0.16.0's dynamic library binding (`dynlb`) crashed on macOS Intel when loading the REX framework via CGo. The crash happened deep in Zig's linker during module initialization — not fixable without upstream Zig changes.
- **v26 breaking change**: Zig 0.16.x broke the CGo calling convention in a way that affected `cgo(callback)` handling. The project was stuck on Zig 0.15.x which had different APIs.
- **macOS framework complexity**: Embedding the REX Shared Library.framework required rpath patching, framework bundling, and different handling per architecture (Apple Silicon vs Intel).
- **Windows DLL loader issues**: The REX.c dynamic loader had path resolution issues on Windows.
- **CI complexity**: Required GPG-encrypted tarballs of the SDK, GPG keys in secrets, and decryption steps in GitHub Actions.

**Solution**: Implemented REX2 format decoding/encoding entirely in Go:
- `rex2/reader.go`: IFF chunk parser, handles CAT REX2, HEAD, CREI, TRSH, SINF, GLOB, SLCE, SDAT
- `rex2/dwop.go`: DPCM decoder with predictor state per channel, variable-length bit stuffing
- `rex2/encoder.go`: Produces valid CAT REX2 files with DWOP compression
- `rex2/legacy.go`: PTI and OT legacy format readers (same SliceInfo output)

**REX2 encoder status (v1.3.0):**
- Bit-perfect DWOP encoding achieved via predictor symmetry (inversion of applyPredictor logic).
- Stereo coupling (L + Delta) bitstream alignment corrected.
- Word alignment (4-byte padding) enforced for SDAT chunks.
- SLCE chunks sorted by sample start position.
- **REX2 OUTPUT ENABLED** and verified via ReCycle bitstream parity.

**v1.4.0: Metadata & Categorization Update**
- **Categorization**: Added `--category` flag to set hardware folder tags (OP-1 `original_folder`).
- **Clean Metadata**: Removed all legacy "REXConverter" and "SOLE DISPLAY" artifacts. Encoders now use sanitized source filenames for internal instrument names.

**v1.3.0: Robust Batch Release**
- **Fault Tolerance**: Runner logs per-file errors to Stderr and continues batch jobs.
- **Header Guard**: Explicit `<!DOCTYPE html>` detection to catch failed web downloads.

**v1.1.0: The Open Ecosystem Update**
- SFZ Export: Plain WAV + .sfz sidecar mapping.
- Decent Sampler Export: Plain WAV + .dspreset sidecar mapping.
- Akai MPC XPM Export: Modern XML-based drum programs for MPC Live/One/X.
- Sample Rate Auto-Detection: Automatically uses source sample rate if not specified via -s.
- WAV/AIFF/CAF Input: Full support for reading sliced audio formats as batch input.

### REX2 encoder fix details (v0.5.1)
- **Predictor residual inversion**: Corrected case 2-4 logic to sequentially subtract accumulated deltas, matching the decoder's symmetric addition.
- **Stereo DPCM**: Aligned L+Delta channel encoding to process channels sequentially while maintaining bit-alignment.
- **Word alignment**: Added 4-byte zero-padding to SDAT chunk payloads to match word-aligned DWOP streams.
- **Slice sorting**: Enforced mandatory sorting of SLCE chunks by sample start position per REX2 specification.

## Key constraints

### REX2 Encoder issues (known)
- DWOP compression produces different byte-for-byte output than original SDK
- SDAT size differs from original by ~600 bytes for typical files
- ReCycle validation fails even though IFF structure is correct
- Internal roundtrip (encode→decode) passes PCM validation
- Root cause likely in DWOP predictor state or bit-stuffing implementation

### REX2 decoder details
- `rex2/types.go`: FileInfo, SliceInfo, CreatorInfo, REX2File structs. PTR stubs (format uses 4-byte PTR/length, not 8-byte offset for SDAT chunks, which differ from many docs).
- `rex2/reader.go`: Decode reads CAT REX2 → iterates chunks. Handles HEAD (glob+slice metadata), CREI (creator name/copyright/url/email/free text), TRSH (transient sensitivity/decay/freeze), SINF (sample info), GLOB (BPM, grids), SLCE (slice boundaries+ppq), SDAT (DWOP compressed audio). Raw PCM validated via `MTHD` frame count vs actual decoded frames.
- `rex2/dwop.go`: DWOP decoder — DPCM with predictor state per channel (L + delta for stereo). Supports 8/16/24/32-bit. Bit stuffing: variable-length code per sample, states 0-4 determine encoding length.
- `rex2/encoder.go` (`rex2.Encode`): Produces CAT REX2 with HEAD, CREI, SINF, GLOB, SLCE, SDAT (DWOP). Stereo encodes as L + delta channel. Bit depth from FileInfo.BitDepth. Predictor state per channel.
- `rex2/legacy.go`: PTI (`TI\x01`/`PTI\x00`) and OT (`FORM...DPS1`/`OT\x00\x00`) readers for legacy ReCycle formats. Returns same SliceInfo slice.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [g-lok/chirashi](https://github.com/g-lok/chirashi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
