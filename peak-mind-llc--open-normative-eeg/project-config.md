---
trigger: always_on
description: - `open_normative/` — Core library (parameters, pipeline, preprocessing, spectral, connectivity, channels, normative, compare, source, datasets/)
---

# Open Normative EEG — Claude Code Guide

## Architecture

- `open_normative/` — Core library (parameters, pipeline, preprocessing, spectral, connectivity, channels, normative, compare, source, datasets/)
- `open_normative/data/` — Pre-computed source localization assets (TMs, forward models, ROI labels, BA labels, DK labels, Brodmann table)
- `scripts/` — CLI tools (build_norms, distribute, downloaders, QC scripts, validation, visualization, cloud_recompute)
- `scripts/normative/` — Dataset-specific QC scripts (dortmund_qc, srm_qc, trt_qc, depress_qc)
- `tests/` — pytest tests
- `Dockerfile` + `requirements-pinned.txt` + `scripts/batch_entrypoint.sh` — container image for AWS Batch (published to `ghcr.io/peak-mind-llc/open-normative-eeg` on main pushes)
- `infra/aws/` — Terraform module: S3 bucket, Batch compute env, queue, IAM, CloudWatch, Budgets
- `.github/workflows/` — `publish-image.yml` (GHCR build on main), `tests.yml` (pytest on PRs)

## Testing

```bash
# Run passing tests (skip asrpy-dependent ones that fail due to upstream numpy 2.x bug)
python -m pytest tests/ --ignore=tests/test_pipeline.py --ignore=tests/test_preprocessing.py
```

## Key Conventions

- Supports both 19-channel 10-20 and 37-channel 10-10 montages (`--channels 19|37`)
- 37ch set: 19 standard + AF3 AF4 FC3 FC1 FC2 FC4 FT7 FT8 CP3 CP1 CP2 CP4 TP7 TP8 PO3 PO4 P1 P2 (matched to pre-computed forward model)
- Dual z-scores: uncorrected (raw band power) + corrected (periodic-only via specparam)
- Distribution honesty (Wood et al. 2024): norm cells carry raw `skewness`/`kurtosis`, a scoring-space `normality_p`, and a `transform_normalized` flag; `compare.py` reports a percentile-derived `robust_z` alongside the parametric z and flags non-normal cells (`parametric_z_unreliable`) and divergence (`z_discrepancy_flag`). See `scripts/distribution_report.py` (disclosure report) and `scripts/compute_trt_reliability.py` (ICC/MDC/heteroscedasticity from the TRT test-retest sessions).
- Log-space subtraction for specparam: `periodic_log10 = log10(full_psd) - log10(aperiodic)`
- `PIPELINE_PARAMS` in parameters.py is the single canonical config dict
- Scripts use argparse, pathlib, checkpoint/resume pattern, logging to stderr
- Workers call `np.seterr(all="warn")` to prevent FloatingPointError from crashing the pool
- ASR includes NaN/Inf detection — reverts to pre-ASR data if ASR corrupts the signal
- ICA catches FloatingPointError/LinAlgError and skips gracefully

## Datasets

| Dataset | Loader | Downloader | QC | Subjects | Ages | Channels |
|---------|--------|------------|-----|----------|------|----------|
| LEMON | `lemon.py` | `lemon_download.py` | `lemon_qc.py` | ~205 | 20-77 | 62ch BrainVision |
| Dortmund | `dortmund.py` | `dortmund_download.py` | `dortmund_qc.py` | ~608 | 20-70 | 64ch BrainProducts |
| SRM | `srm.py` | `srm_download.py` | `srm_qc.py` | 111 | 17-71 | 64ch BioSemi EDF |
| TRT | `trt.py` | `trt_download.py` | `trt_qc.py` | 60 | 18-28 | 64ch BrainVision |
| Depress | `depress.py` | `depress_download.py` | `depress_qc.py` | ~55-70 healthy | 18-24 | 64ch Neuroscan .set |
| HBN | `hbn.py` | `hbn_download.py` | — | ~2800 | 5-21 | 128ch EGI |
| MIPDB | `mipdb.py` | — | — | — | — | — |

All loaders registered in `open_normative/datasets/__init__.py` DATASETS dict.

## Source Localization

### sLORETA Source Power
- Pre-computed transformation matrices map channel power → 2394 MNI voxels → Brodmann areas
- `compute_sloreta_source_power()` accepts `power_key` param: "absolute" or "corrected_absolute"
- Produces both `source_power` and `corrected_source_power` (specparam periodic-only at scalp level, then projected)
- Output channels: `_src_ba_{BA_label}` (e.g., `_src_ba_Brodmann area 17`)

### DICS Beamformer Connectivity (Unified DK-as-Canonical)
- Pre-computed forward model → CSD → DICS beamformer → ROI/DK time courses → spectral connectivity
- **2 direct extractions, 3 atlases:**
  - **18 merged functional ROIs** — direct extraction (`_src_conn_{ROI_A}_{ROI_B}`, 153 pairs). Used because the merged functional ROIs (DLPFC, mPFC, etc.) span multiple DK parcels and are conceptually different from anatomical parcels.
  - **68 individual DK parcels** — direct extraction (`_src_dk_{parcel-lh}_{parcel-rh}`, 2,278 pairs). Canonical anatomical atlas.
  - **~44 Brodmann areas** — DERIVED from DK by aggregation (`_src_ba_conn_BA{A}-{hemi}_BA{B}-{hemi}`). For each BA, average corrected power and connectivity from its mapped DK parcels using the `_DK_TO_BA` table. Single source of truth.
- Connectivity methods: dwPLI, coherence, imaginary coherence
- Network-level aggregation: within/between for 7 networks (from ROI labels)
- Adaptive regularization: tries reg=0.05 → 0.1 → 0.2 → 0.5 on ill-conditioned CSD matrices
- Volume conduction detection: flags high-coherence + low-dwPLI pairs

### DICS Source-Level Specparam (corrected_dics_power)
- Broadband DICS (1-50 Hz) → source time courses → Welch PSD per label → specparam → periodic-only band power
- Theoretically superior to scalp-level correction because aperiodic subtraction happens at the source
- `_broadband_dics()` computes stcs once, shared across ROI and DK label sets

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [peak-mind-llc/open-normative-eeg](https://github.com/peak-mind-llc/open-normative-eeg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
