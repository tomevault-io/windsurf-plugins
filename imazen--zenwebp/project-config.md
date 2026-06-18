---
trigger: always_on
description: Enables AVX2 autovectorization of prediction functions. Eliminates
---

# zenwebp CLAUDE.md

See global ~/.claude/CLAUDE.md for general instructions.
Historical investigation notes and resolved bugs are in [LOG.md](LOG.md).

## zenwebp-recompress (nested workspace, added 2026-05-28)

`zenwebp/zenwebp-recompress/` is a **self-contained nested Cargo workspace**
(crate `zenwebp-recompress` + `zwr-calibrate` binary). It recompresses
already-encoded WebPs to a target zensim Profile A score, picking the
optimal strategy (LosslessRemux / Reencode / LosslessReencode; CoeffEdit +
DeblockReencode are de-selected — see below). Public API: `recompress()` +
`plan()`, with `Budget::{OneShot, MaxIterations, MaxTime}`.

**zenwebp is deliberately NOT a Cargo workspace.** The recompress crate
path-depends on `../zensim`, `../zenpixels`, `../zenanalyze`, which are not
checked out on zenwebp's core CI runners; making it a workspace member would
break zenwebp's `cargo build`/`test` (Cargo resolves the whole member graph
upfront). Build it from its own directory: `cd zenwebp-recompress && cargo
test`. Its own CI is `.github/workflows/recompress.yml` (isolated, path-
filtered, checks out the three siblings).

**DeblockReencode is FALSIFIED** (measured net-negative; VP8 already
deblocks in-loop). The router never selects it; the artifact-aware filter
survives as `expert::deblock_rgba`. Don't re-add it to the router without a
source config where it measurably wins. See
`zenwebp-recompress/benchmarks/deblock_experiment_2026-05-28.md`.

**CoeffEdit is BUILT then RD-FALSIFIED for WebP** (2026-05-28). `src/vp8x/`
is a complete, self-contained VP8 keyframe coefficient transcoder (boolean
coder + parse + emit + edits), and its verbatim/no-op path is **pixel-exact**
with libwebp (MAD 0). But every size-reducing coefficient edit (AC-drop and
level requantization) is RD-dominated by `Reencode` at matched output size —
because VP8 predicts each block from neighbours' reconstructed pixels, so
editing coefficients drifts the whole frame, and there's no RD re-optimisation.
Coefficient transcoding is the right tool for prediction-free codecs (baseline
JPEG), not VP8. **Drift compensation was also tried and falsified**
(`vp8x::compensate`, closed-loop DC trim): Jacobi-unstable on the prediction
chain, recovers only ~10% of the drift, never closes the gap to Reencode —
complete stable compensation just *is* re-encoding. The transcoder + edits +
compensator stay reachable via `expert::run_coeff_edit{,_keep,_requant}` /
`vp8x::compensate`; the router never selects CoeffEdit (its only loss-free
point is verbatim = `LosslessRemux`). Don't re-attempt coefficient-domain size
reduction for WebP — the prediction chain defeats it. See
`zenwebp-recompress/benchmarks/coeff_edit_experiment_2026-05-28.md`.

Calibration is **per content class** (photo/screen/line-art/mixed) in
`src/calibration/calib_tables.rs` (AUTO-GENERATED — regenerate via
`zwr-calibrate/fit_calibration.py`, never hand-edit). Fit from a disciplined
50-ref/class, multi-size (Mitchell, downscale-only), q20–100-step-2 sweep
(248,501 cells; held-out MAE screen 3.56 / mixed 3.98 / photo 7.89 / line-art
8.49). The `data.rs` functions take `ContentClass`; the router threads
`analysis.content_class`. Provenance + re-fit recipe:
`benchmarks/calibration_2026-05-28.md`; raw CSVs + corpus on `/mnt/v`. Mixed is
thin (~10 source refs) — weakest table; the size guard keeps it correct.

## Canonical training data + indexes (added 2026-05-20)

**The canonical index for all ML data lives at `~/work/zen/DATA_PROVENANCE.md`.**

Quick paths:
- Trainer input: `/mnt/v/zen/zensim-training/canonical-2026-05-21/`
- Master inventory: `~/work/zen/_ml-inventory-2026-05-20/00-MASTER-SYNTHESIS.md`
- Per-codec picker audit: `~/work/zen/_ml-inventory-2026-05-20/05-per-codec-pickers.md`

## ML/picker status (2026-05-20)

zenwebp's 2026-04-29 to 2026-05-03 picker spike at `src/encoder/picker/` is currently in tree but **never wired into the public API** per the 2026-05-20 per-codec audit. Considered dead code; removal pending separate cleanup.

For working picker reference: `~/work/zen/zenavif/src/auto_tune.rs` + `EncoderConfig::auto_tune()` is the only production-shipped zen-codec picker today.

## Performance & Testing

See `docs/PERFORMANCE.md` for benchmarks, `docs/CALL-TREE.md` for SIMD tiers, `docs/ARCHITECTURE-CLEANUP.md` for code organization.

**Decoder v2** (`src/decoder/vp8v2/`): bit-exact with libwebp, 1.09-1.15x speed, streaming cache→RGB (~100KB peak). Use for all new decode work.

**Pixel-exact gate**: `tests/v2_pixel_perfect.rs` (tolerance 0 vs libwebp). Lossless: `examples/lossless_rt_check.rs` (24/24 exact).

**Benchmarks**: `benches/decode_compare.rs` (14 images), `benches/decode_lossless_compare.rs`. zenbench, no `-C target-cpu=native`.

## Key Files

**Encoder (lossy):**
- `src/encoder/vp8/mod.rs` - Main encoder orchestration, single-pass token loop
- `src/encoder/vp8/residuals.rs` - TokenBuffer, coefficient token recording/emission
- `src/encoder/vp8/header.rs` - Bitstream header encoding
- `src/encoder/vp8/mode_selection.rs` - I16/I4/UV mode selection
- `src/encoder/vp8/prediction.rs` - Block prediction + transform
- `src/encoder/api.rs` - Public API, EncoderConfig, EncoderParams, Preset enum

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imazen/zenwebp](https://github.com/imazen/zenwebp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
