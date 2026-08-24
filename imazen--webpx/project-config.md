---
trigger: always_on
description: WebP encoding/decoding crate using FFI bindings to libwebp via `libwebp-sys`.
---

# webpx Project Instructions

## Overview

WebP encoding/decoding crate using FFI bindings to libwebp via `libwebp-sys`.

**Origin / role:** webpx was LLM-created as a **parity oracle** for developing
[`zenwebp`](https://github.com/imazen/zenwebp) — a faithful wrapper over the
reference libwebp C library, used to verify that zenwebp (the pure-Rust
`#![forbid(unsafe_code)]` port) matches libwebp 100% across the encode/decode
surface. That is why webpx's `zencodec` trait surface mirrors zenwebp's, and why
the README, SECURITY.md, and the security advisory steer new projects to
zenwebp. webpx remains a published, maintained crate (0.4.0) for callers who
specifically need the libwebp-backed path — it is the reference, not the
recommended default.

## Project Status

Initial implementation complete. All core features working:
- Static encode/decode (RGB, RGBA, YUV)
- Streaming encode/decode
- Animation encode/decode
- ICC/EXIF/XMP metadata
- Content presets

## Key Files

- `src/lib.rs` - Main entry, re-exports
- `src/encode.rs` - Static encoding, Encoder builder
- `src/decode.rs` - Static decoding, Decoder builder
- `src/streaming.rs` - StreamingDecoder, StreamingEncoder
- `src/animation.rs` - AnimationEncoder, AnimationDecoder
- `src/mux.rs` - ICC/EXIF/XMP metadata operations
- `src/config.rs` - Preset enum, EncoderConfig, DecoderConfig
- `src/types.rs` - ImageInfo, ColorMode, YuvPlanes
- `src/error.rs` - Error types

## Build Commands

Use justfile commands for common tasks:

```bash
just test       # Run all tests
just clippy     # Run clippy
just fmt        # Format code
just semver     # Check semver compatibility
just ci         # Full CI check (fmt, clippy, test, semver)
just prepublish # Pre-publish check (ci + doc)
just bench      # Run benchmarks
just coverage   # Generate coverage report
just doc        # Build and open docs
```

Or directly:

```bash
cargo test --all-features
cargo clippy --all-targets --all-features -- -D warnings
cargo fmt
cargo semver-checks check-release --all-features
```

## Publishing Checklist

Before publishing a new version:

1. Run `just prepublish` to verify all checks pass
2. Update version in `Cargo.toml`
3. Commit version bump
4. Tag with `git tag vX.Y.Z`
5. Push: `git push && git push origin vX.Y.Z`
6. Verify CI passes on GitHub
7. Publish: `cargo publish`

**IMPORTANT:** `cargo-semver-checks` runs in CI and must pass before release. It detects breaking API changes that require a major version bump.

## Test Fixtures

Test WebP files are in `tests/fixtures/`:
- `lossless_2color.webp` - 314 bytes, lossless
- `lossy_rgb.webp` - 2.2K, lossy RGB
- `lossy_alpha.webp` - 1.3K, lossy with alpha
- `animated.webp` - 11K, animated

Larger test files available at `~/work/codec-corpus/image-rs/test-images/webp/`.

## libwebp API Notes

- Use `WebPDemuxInternal` with `WEBP_DEMUX_ABI_VERSION` (not `WebPDemux`)
- Use `WebPMuxCreateInternal` with `WEBP_MUX_ABI_VERSION` (not `WebPMuxCreate`)
- Use `WebPAnimEncoderOptionsInitInternal` and `WebPAnimEncoderNewInternal`
- `WebPConfigInitInternal` takes `WEBP_ENCODER_ABI_VERSION` (NOT decoder)
- Animation timestamps are END times, not START times

## FFI soundness invariants (audit checklist)

These are the patterns that have bitten webpx; treat any new FFI call site as suspect until verified.

1. **Never use `libwebp_sys::Webp*::new()` bindgen helpers.** They're generated as `MaybeUninit::uninit() + WebP*Init + assume_init`, which is technically UB because the bindgen-generated structs expose libwebp's reserved `pad` arrays as ordinary Rust fields and libwebp's `*Init*` functions don't touch them. Use explicit `MaybeUninit::<libwebp_sys::T>::zeroed()` + the matching `*Init*` call, or webpx's `crate::ffi::picture::Picture` / `crate::ffi::mem_writer::MemWriter` RAII wrappers. Audited in 0.2.3, 0.3.3, PR #8 — every `WebPDecoderConfig::new`, `WebPConfig::new`, `WebPConfig::new_with_preset`, `WebPPicture::new` call site has been replaced.

2. **`WebPIUpdate` does NOT copy its input buffer.** It stashes the raw pointer and re-reads on subsequent calls. Never expose it through a Rust API that takes `&[u8]` without a lifetime tying the buffer to the decoder. webpx routes `update` through `append` (which uses `WebPIAppend`, copying); see 0.3.4 for the UAF fix.

3. **Any libwebp callback invoked from C frames** (progress hook, writer hook) must wrap the user-supplied Rust closure in `catch_unwind`. Letting a panic unwind through libwebp's C stack is UB. Capture the panic payload, signal abort, and re-raise from a Rust frame after the libwebp call returns. See `encode.rs::progress_hook` and `streaming.rs::write_callback` for the pattern.

4. **Strides cast to `i32` must go through `crate::ffi::validate::stride_fits_i32`.** A stride `>= 2^31` wraps negative; libwebp's row-pointer arithmetic walks backwards through process memory. Centralized in 0.2.1; never hand-roll this check.

5. **`width × bytes_per_pixel`, `stride × rows`, `slice.len() × bpp`** must use `saturating_mul`. On 32-bit `usize` (i686 in CI matrix), unchecked multiplications wrap and bypass downstream length guards.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imazen/webpx](https://github.com/imazen/webpx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
