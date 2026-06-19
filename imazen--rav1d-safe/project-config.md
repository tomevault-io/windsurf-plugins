---
trigger: always_on
description: Safe SIMD fork of rav1d — 160k lines of hand-written assembly replaced by safe Rust intrinsics.
---

# rav1d-safe

Safe SIMD fork of rav1d — 160k lines of hand-written assembly replaced by safe Rust intrinsics.

## Porting Status

**All major DSP modules ported.** 59k lines of safe Rust SIMD in `src/safe_simd/`.

Completed modules (AVX2 + NEON, 8bpc + 16bpc):
- mc (motion compensation) — including warp_affine
- itx (inverse transforms) — 160 transforms each bpc
- ipred (intra prediction) — all 14 modes
- cdef (directional enhancement)
- loopfilter
- looprestoration (Wiener + SGR)
- filmgrain
- pal (palette)
- refmvs (reference MVs)
- msac (SSE2 adapt4/adapt8/hi_tok when unchecked+x86_64, branchless scalar otherwise, serial loop adapt16)

**Remaining (not ported, scalar fallback):**
- Scaled MC (put_8tap_scaled, prep_8tap_scaled, bilin_scaled) — complex per-pixel filter selection, ~2% of profile
- AVX-512 paths (~26k lines) — falls back to AVX2
- SSE-only paths (~52k lines) — falls back to scalar
- ARM SVE2, dotprod, i8mm — falls back to NEON

## Conformance

784/803 dav1d test vectors pass at all bit depths and all CPU levels (scalar, SSE4, AVX2).
19 failures are infrastructure (1 sframe, 6 SVC operating points, 12 vq_suite decode modes).

## Benchmarks (2026-05-24 after SIMD row 1D transforms for dct8/16/32 + adst16)

**4K photo AVIF (3840x2561) — interleaved A/B (30 iters/build, 4 runs):**

| Build | ms/iter | vs ASM |
|-------|---------|--------|
| ASM | 119.0 | 1.0x |
| Safe Checked | 198.3 | **1.66x** |
| Safe Unchecked | 187.9 | **1.57x** |

Progress vs 2026-02-13 baseline (Checked):
- 4K AVIF: 1.98x → **1.66x** (~16% absolute gap closed; ~33% of remaining gap)

Optimizations in this session (2026-05-24, all in safe checked, `#![forbid(unsafe_code)]`):
- Dispatch refactor: loopfilter outer dispatch wrapped in #[arcane] V2, inner
  loop_filter_4_8bpc switched to #[rite] so per-edge SIMD helpers inline
  (no wall-clock change; cleaner symbol table)
- SIMD row DCT-32 8bpc for 32x32, 32x16, 32x8, 32x64 (was scalar dct32_1d
  per row; now dct32_1d_cols8 + 32x8→8x32 transpose, 8 rows in parallel
  via SIMD lanes)
- SIMD row DCT-16 8bpc for 16x16, 16x8, 16x32, 16x64 dct_dct + 3 dct-row
  mixed variants (dct_adst, dct_flipadst, dct_identity at 16x16)
- SIMD row DCT-8 8bpc for 8x8, 8x16, 8x32
- SIMD row ADST-16 8bpc for 8 adst-row 16x16 variants (adst_dct, adst_adst,
  adst_flipadst, adst_identity, flipadst_dct, flipadst_flipadst,
  flipadst_adst, flipadst_identity)
- Helpers: `simd_row_dct{8,16,32}_8bpc_8rows`, `simd_row_adst{8,16}_8bpc_8rows`
  in safe_simd/itx.rs

Pattern: load 8 i16 per column × N columns (contiguous in column-major
coeff), cvtepi16_epi32, optional rect2_scale, {dct,adst}_1d_cols8, round+
clip, 8x8 i32 transpose chunk(s), store row-major. Each helper is #[rite]
target_feature so it inlines into the #[arcane] outer transform.

Wall-clock impact (4K AVIF, checked, vs prior session baseline 1.78x):
- After dispatch refactor: 1.78x (no change)
- After SIMD row dct32: 1.71x (4.5% gap closure)
- After SIMD row dct16: 1.69x (1% more)
- After SIMD row dct8: 1.66x (2% more)
- After SIMD row mixed dct/adst 16x16: 1.66x (small but consistent)

Optimizations landed (all in safe checked, `#![forbid(unsafe_code)]`):
- `cfl_pred` SIMD (8bpc+16bpc)
- DCT column-pass SIMD wired into 8x8 (both bpcs), 16x16 (both bpcs), 32x32 dct_dct (square)
- DCT column-pass SIMD wired into 8x16, 16x8, 16x32, 32x16, 8x32, 32x8, 16x4, 8x4, 64x16, 64x32 dct_dct + 16bpc equivalents (rectangular)
- ADST / Identity / FlipADST column SIMD wired into all 14 non-trivial 16x16 transform combinations (both bpcs)
- ADST / FlipADST / Identity column SIMD wired into all 14 non-trivial 8x8 transform combinations (8bpc)
- Identity column SIMD inlined into identity_identity rectangular 8x32/32x8, 16x32/32x16 (both bpcs) plus 16bpc identity rect variants
- **AVX-512 (Server64/X64V4Token) column transforms `dct16_cols_avx512` + `dct32_cols_avx512`**, wired into 16x16, 32x32, 16x32, 32x16, 64x16, 64x32 (both bpcs) — 16 cols/chunk vs 8 with AVX2
- **Per-row DisjointMut guard batching** in inv_txfm_add scalar fallback + ipred scalar paths (splat_dc, cfl_pred, paeth, smooth_v/h, smooth, z1, z2): one wider `strided_slice_mut` covering the whole block instead of N guards per row
- **SIMD v-direction loopfilter (8bpc)** for all widths (wd=4/6/8/16). Four #[arcane] functions with X64V2Token; contiguous 4-byte column loads + cvtepu8_epi32. Three-way branchless mask-select between 14-tap/8-tap/4-tap based on fm + flat8in + flat8out
- **SIMD h-direction loopfilter (8bpc) for narrow + wd=8** via 4×4 i32 transpose loads: load each row's contiguous N-byte chunk, transpose to pixel-position vectors, compute, transpose back. wd=6 and wd=16 h-filter remain scalar

Remaining biggest gaps (without unsafe):
1. **Loopfilter** — `loop_filter_4_8bpc` is fully scalar at 10% of profile vs <1% ASM (biggest single remaining win, ~500-1000 lines of SIMD work)
2. **msac** entropy decoding inside `decode_coefs` (37% of profile, mostly scalar)
3. **DisjointMut BorrowTracker** overhead (~9% checked-only; eliminated only by the `unchecked` feature which uses unsafe)
4. 32x32 ADST/identity column SIMD (32x32 is mostly dct-only in practice)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imazen/rav1d-safe](https://github.com/imazen/rav1d-safe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
