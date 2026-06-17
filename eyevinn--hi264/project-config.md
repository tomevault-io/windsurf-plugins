---
trigger: always_on
description: - Never add "Co-Authored-By: Claude" to commit messages or pull request messages
---

# H.264/AVC Frame Decoder & Bitstream Generator in Pure Go

## Conventions

- Never add "Co-Authored-By: Claude" to commit messages or pull request messages

## Project Status

Pure Go H.264/AVC decoder for IDR and P_Skip frames with CABAC and CAVLC entropy
coding, plus a bitstream generator that produces valid H.264 test content from
grid patterns (I_16x16 DC prediction). Supports 16x16 macroblock and 8x8 block
granularity via PlaneGrid (direct Y/Cb/Cr planes, no character indirection).
Not a general-purpose encoder. Supports both CAVLC (Baseline) and CABAC (Main
profile), with P_Skip frame generation for efficient multi-frame sequences.
All processing is 8-bit 4:2:0 only (no 10-bit or 4:2:2/4:4:4 support).
Pixel-perfect match with FFmpeg across 41 golden decoder test cases and 12+
encoder verification tests.

### Implemented Features

#### Decoder
- CABAC arithmetic engine + context model initialization
- CAVLC (Exp-Golomb + VLC tables) entropy decoding
- Macroblock layer parsing (I_4x4, I_8x8, I_16x16, P_Skip)
- Inverse quantization and transform (4x4, 8x8, DC Hadamard)
- Custom scaling matrices (SPS/PPS with Table 7-2 fall-back)
- Intra prediction (all 4x4, 8x8, 16x16, and chroma modes)
- Frame reconstruction + deblocking filter
- P_Skip frame decoding (copy from reference, CAVLC and CABAC)
- Multi-frame decoding via `DecodeAllFrames` (IDR + P_Skip sequences)
- Y4M, PNG, and JPEG output support

#### Encoder (hi264gen)
- CAVLC I_16x16 IDR frame encoder with DC prediction (Baseline profile)
- CABAC I_16x16 IDR frame encoder with DC prediction (Main profile)
- CABAC arithmetic encoder engine (inverse of decoder)
- CABAC syntax element encoding (mb_type, chroma pred, qp_delta, residual)
- Per-sub-block chroma DC prediction (matching decoder's 4x4 sub-block logic)
- Forward Hadamard transform + quantization (QP 0-51)
- SPS/PPS generation (Baseline and Main profiles, configurable max_num_ref_frames)
- PlaneGrid: direct Y/Cb/Cr value planes with 16x16 or 8x8 block granularity
- PNG/JPEG image backgrounds (downsampled to block-resolution PlaneGrid)
- Grid-based pattern input with RGB or YCbCr color specification
- 8x8 block support: per-4x4-block AC residual encoding at quadrant boundaries
- Forward 4x4 integer DCT (inverse of decoder's InverseTransform4x4)
- FFmpeg-verified output across diverse test patterns (CAVLC and CABAC)
- P_Skip slice encoder (CAVLC and CABAC, all-skip MBs copying from reference)
- P-slice header writer (non-IDR, with ref list and marking syntax, CABAC alignment)
- Efficient multi-frame sequences: IDR at configurable intervals, P_Skip between
- ~76% bitstream size reduction vs all-IDR for repeated frames
- Fragmented MP4 (fMP4/CMAF) output with configurable framerate and fragment duration
- Tiling background pattern from `.gridimg` files (`-gi` with `-w`/`-h`)
- Text overlay with format patterns (`-text "%03d"`, `"%mm:%ss.%ff"`, etc.)
- Text character set: A-Z 0-9 and `! # % + - . / : = ? [ ] _ ( )` (lowercase auto-uppercased)
- Auto-scaling text (`-text-scale 0`) to fill available frame space
- Text background box (`-text-bg R,G,B`) for readability over busy patterns
- Built-in 75% SMPTE color bars pattern (`-smpte`)
- Filler NAL padding (`-bpp`) for fixed bytes-per-picture / CBR-like streams
- Stdout output (`-o -`) with explicit format flag (`-f 264`, `-f mp4`, etc.)

#### Color Space Support
- BT.601 (SD, default), BT.709 (HD), BT.2020 (UHD) color spaces
- Limited-range (Y: 16-235) and full-range (Y: 0-255) YCbCr conversion
- H.264 SPS VUI parameters signaling (colour_primaries, transfer_characteristics, matrix_coefficients, video_full_range_flag)
- Decoder extracts VUI color metadata from SPS for correct YCbCr→RGB conversion
- `.gridimg` directives: `@bt709`, `@bt2020`, `@bt601` for per-file color space; `@8x8` for 8x8 block granularity
- Y4M output with XCOLORSPACE/XCOLORRANGE tags

#### Raw Output (hi264gen)
- Raw YUV/Y4M/PNG/JPEG output from the same grid patterns (no H.264 encoding)
- Configurable color space for RGB↔YCbCr conversion (BT.601/BT.709/BT.2020)
- `.gridimg` file format with `@rgb`, `@bt709`/`@bt2020`, and `@8x8` directives

### Dependencies
- `github.com/Eyevinn/mp4ff` — SPS/PPS/SliceHeader parsing, NAL extraction, fragmented MP4 creation

### Key Reference Files
- FFmpeg: `external/ffmpeg/libavcodec/h264_cabac.c`, `h264_cavlc.c`
- Standard: `references/ISO_IEC_DIS_14496-10_Ed11.pdf`

## Build & Test

```bash
go build ./...
go test ./...
```

### CLI tools

```bash
# Decode H.264 (auto-detects .264 vs .mp4/.m4v input)
go run ./cmd/hi264dec input.264 output.yuv      # raw YUV (auto-adds _WxH_yuv420p suffix)
go run ./cmd/hi264dec input.264 output.png      # PNG (auto-detected from extension)
go run ./cmd/hi264dec input.264 output.jpg      # JPEG (-q for quality, default 85)
go run ./cmd/hi264dec input.264 output.y4m      # Y4M

# Decode IDR frames from MP4 container
go run ./cmd/hi264dec input.mp4 output.png
go run ./cmd/hi264dec -n 5 input.mp4 frames.png # extract 5 IDR frames (frames_0000.png, ...)

# Generate H.264 IDR frame from grid pattern (CAVLC, Baseline profile)
go run ./cmd/hi264gen -gi examples/sweden.gridimg -o sweden.264

# Generate H.264 IDR frame with CABAC entropy coding (Main profile)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eyevinn/hi264](https://github.com/Eyevinn/hi264) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
