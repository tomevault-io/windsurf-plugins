---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this
repository. It documents the **current** state of the pipeline and its main results. The
detailed, dated history of what was tried, rejected, or superseded to get here lives in
`docs/experiments.md` (the experiment register) and `docs/open-questions.md` (open items) --
consult those before re-deriving something that may already have a documented answer.

## What this is

`earthpv` detects individual large rooftop solar PV arrays (target > 400 m², the practical
floor for per-pixel supervision at Sentinel-2's 10 m GSD) from Sentinel-2 L2A imagery by
fine-tuning the open-source **TerraMind** geospatial foundation model (IBM/ESA, via
**TerraTorch**). Labels come from OpenStreetMap solar mapping (through Overture Maps);
building footprints classify detections as rooftop/ground. It is **recall-first**: candidates
are meant to be human-validated against high-res imagery in OSM workflows, so false positives
are tolerated. Installations below the 400 m² floor are not targeted by segmentation at all --
that gap is closed by a separate per-building classifier, `roofclf` (see "Main workflow"
below). Trained on Germany, inferred on Pakistan (primary AOI) and Gujarat, India. Read
`README.md` for the narrative and current headline numbers.

**Why two detectors, not one**: Germany's legally-complete PV register (MaStR) shows **65.5%
of rooftop capacity sits in installations below the 400 m² floor** (97.2% of installations by
count) -- see "MaStR validation" below. A segmentation model trained only above that floor is
structurally blind to roughly two-thirds of the capacity a "rooftop solar" headline implies,
which is why `roofclf` exists and why it is not optional infrastructure.

## Main workflow (default pipeline, primary output)

This is the project's default, documented workflow, and the **evidence atlas** is its primary
output. Two detectors, split by placement and by calibration coverage rather than cleanly by
size, combined into one product:

- **Segmentation** (`infer` → `postprocess` → `density`) -- the TerraMind fine-tune,
  outlining panels directly. Produces every mapping lead regardless of size, and is the only
  instrument for ground-mount at any size (`roofclf` has no footprint to classify there). It
  remains the authoritative rooftop instrument for individual arrays **≥ 400 m²** everywhere
  `roofclf` has not been calibrated to replace it. Production checkpoint: `v3_combined_india`
  (`terramind-pv-epoch=22-step=9062.ckpt`), confirmed by the owner 2026-08-07, no retrain
  planned. (Two earlier checkpoints, `v2_combined` and an undocumented `pk16085` variant, were
  deleted from disk at some point and can no longer be independently re-verified; a Gujarat
  atlas built before this was noticed is flagged in `docs/results/gujarat.md`.)
- **`roofclf`** (`roof-classifier` → `roofclf-score-national` → `sub400-capacity` →
  `ge400-roof-capacity`) -- a per-building "does this roof carry PV?" classifier, cross-checked
  with the zero-training **SPPI** spectral index (roofclf AND SPPI agreeing) as an internal
  floor under the atlas's headline figure. Covers every building **< 400 m²** and, since
  2026-08-07, also **replaces** segmentation's own rooftop estimate for buildings **≥ 400 m²**
  inside a density-calibrated domain of cells, where it measures better (AUC ~0.76-0.78 vs
  segmentation's ~0.50-0.78, strongly conditional on quadrat). Both capacity functions are
  domain-restricted and refuse to rescale to a national total on their own; an AND-gate variant
  can additionally cover cells *outside* the calibrated domain as an explicitly-flagged
  extrapolation, but that component is **no longer published** (dropped from the atlas
  2026-08-15; see "Density stage" below).
- **`atlas.build_evidence_atlas`** combines both into **Best estimate**, this project's own
  highest defensible figure (hand-mapped OSM, plus segmentation's ground-mount detections,
  roofclf's rooftop replacement in-domain plus segmentation's own recall-corrected rooftop
  out-of-domain, and roofclf-alone density below 400 m²) -- de-duplicated against each other
  and against OSM, and
  floored per cell at hand-mapped OSM plus the stricter roofclf+SPPI agreement population
  (internally still called "Verified" in the code, but no longer surfaced anywhere in the
  published atlas, docs, or README -- 2026-08-12 decision). Reports a 90% credible interval on
  the headline figure (see "Density stage").

The end-to-end command sequence is in `docs/reproduce.md`'s "The full pipeline"; the short
version:

```bash
earthpv labels --aoi <aoi>       && earthpv chips --aoi <aoi>
earthpv train  --config configs/terramind_pv.yaml
earthpv infer  --aoi <aoi> --checkpoint <ckpt>
earthpv postprocess --aoi <aoi> --threshold 0.3
earthpv export --aoi <aoi>
earthpv density --aoi <aoi> --districts && earthpv check-density --aoi <aoi>

earthpv roof-classifier --aoi <aoi>                 # needs mapped calibration quadrats
earthpv roofclf-score-national --aoi <aoi>          # long: hours at country scale

# ESSENTIAL, not optional -- run after every national scoring pass. See
# docs/methods/roofclf-national-validation.md.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-energy-transition/earthpv](https://github.com/open-energy-transition/earthpv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
