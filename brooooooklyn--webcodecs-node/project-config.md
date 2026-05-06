---
trigger: always_on
description: WebCodecs API implementation for Node.js using FFmpeg, built with napi-rs (Rust → Node.js native addon). Provides W3C WebCodecs spec-compliant video/audio encoding/decoding with FFmpeg as the backend.
---

# @napi-rs/webcodecs - Project Context

## Overview

WebCodecs API implementation for Node.js using FFmpeg, built with napi-rs (Rust → Node.js native addon). Provides W3C WebCodecs spec-compliant video/audio encoding/decoding with FFmpeg as the backend.

**Package:** `@napi-rs/webcodecs` | **Version:** `0.0.0` | **License:** MIT

## Project Status

**Status:** Feature-complete, production-ready

| Component           | Status      | Notes                                                               |
| ------------------- | ----------- | ------------------------------------------------------------------- |
| VideoEncoder        | ✅ Complete | H.264, H.265 (with Alpha), VP8, VP9 (with Alpha), AV1 + EventTarget |
| VideoDecoder        | ✅ Complete | All codecs + AV1 drain + EventTarget                                |
| AudioEncoder        | ✅ Complete | AAC, Opus, MP3, FLAC + EventTarget                                  |
| AudioDecoder        | ✅ Complete | All codecs with resampling + EventTarget                            |
| VideoFrame          | ✅ Complete | All pixel formats, async copyTo, format conversion, Canvas support  |
| AudioData           | ✅ Complete | All sample formats                                                  |
| ImageDecoder        | ✅ Complete | JPEG, PNG, WebP, GIF, BMP, AVIF, JPEG XL                            |
| Mp4Demuxer          | ✅ Complete | H.264, H.265, AV1, AAC, Opus                                        |
| WebMDemuxer         | ✅ Complete | VP8, VP9, AV1, Opus, Vorbis                                         |
| MkvDemuxer          | ✅ Complete | All codecs                                                          |
| Mp4Muxer            | ✅ Complete | H.264, H.265, AV1 + AAC, Opus, MP3, FLAC                            |
| WebMMuxer           | ✅ Complete | VP8, VP9, AV1 + Opus, Vorbis                                        |
| MkvMuxer            | ✅ Complete | All codecs                                                          |
| Threading           | ✅ Complete | Non-blocking Drop, proper lifecycle                                 |
| W3C Spec Compliance | ✅ Complete | All APIs aligned                                                    |
| Type Definitions    | ✅ Complete | ~1,100 lines in index.d.ts                                          |
| Test Coverage       | ✅ Complete | 917 tests (34 files), all passing                                   |
| Hardware Encoding   | ✅ Complete | Zero-copy GPU path, auto-tuning                                     |

**Remaining Work:** None - All core features complete.

## Architecture

Three-layer design with clean separation of concerns:

```
src/
├── ffi/           # Low-level FFmpeg FFI bindings (hand-written, no bindgen)
│   ├── types.rs       # AVCodecID, AVPixelFormat, AVRational, etc.
│   ├── accessors.rs/c # Thin C library for FFmpeg struct field access
│   ├── avcodec.rs     # Video codec functions
│   ├── avutil.rs      # Utility functions (logging, options)
│   ├── swscale.rs     # Video scaling/format conversion
│   ├── swresample.rs  # Audio resampling
│   └── hwaccel.rs     # Hardware acceleration
├── codec/         # Mid-level RAII wrappers around FFmpeg
│   ├── context.rs     # CodecContext (encoder/decoder)
│   ├── frame.rs       # Frame handling
│   ├── packet.rs      # Packet handling
│   ├── audio_buffer.rs# Audio sample buffers
│   ├── scaler.rs      # Video scaling
│   ├── resampler.rs   # Audio resampling
│   ├── hwdevice.rs    # Hardware device management
│   └── hwframes.rs    # Hardware frame context (GPU frame pools)
├── webcodecs/     # High-level WebCodecs API classes (NAPI)
│   ├── video_encoder.rs    # VideoEncoder class
│   ├── video_decoder.rs    # VideoDecoder class
│   ├── audio_encoder.rs    # AudioEncoder class
│   ├── audio_decoder.rs    # AudioDecoder class
│   ├── video_frame.rs      # VideoFrame class
│   ├── audio_data.rs       # AudioData class
│   ├── encoded_video_chunk.rs
│   ├── encoded_audio_chunk.rs
│   ├── image_decoder.rs    # ImageDecoder class
│   ├── demuxer_base.rs     # Shared demuxer implementation
│   ├── muxer_base.rs       # Shared muxer implementation
│   ├── mp4_demuxer.rs      # Mp4Demuxer class
│   ├── mp4_muxer.rs        # Mp4Muxer class
│   ├── webm_demuxer.rs     # WebMDemuxer class
│   ├── webm_muxer.rs       # WebMMuxer class
│   ├── mkv_demuxer.rs      # MkvDemuxer class
│   ├── mkv_muxer.rs        # MkvMuxer class
│   ├── codec_string.rs     # Codec string parsing
│   └── error.rs            # Native DOMException helpers
└── lib.rs         # Crate root, module init, re-exports

__test__/          # Test suite (ava)
├── helpers/       # Test utilities (frame/audio generators, codec matrix)
└── integration/   # Integration tests (roundtrip, lifecycle, performance)
```

## Build Commands

```bash
pnpm build         # Release build
pnpm build:debug   # Debug build
pnpm test          # Run tests (ava, 2 min timeout)
pnpm bench         # Run benchmarks
pnpm typecheck     # TypeScript type checking
pnpm lint          # Run oxlint
pnpm format        # Format code (prettier, rustfmt, taplo)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Brooooooklyn/webcodecs-node](https://github.com/Brooooooklyn/webcodecs-node) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
