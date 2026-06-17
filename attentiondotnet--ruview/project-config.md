---
trigger: always_on
description: WiFi-based human pose estimation using Channel State Information (CSI).
---

# Claude Code Configuration — WiFi-DensePose + Claude Flow V3

## Project: wifi-densepose

WiFi-based human pose estimation using Channel State Information (CSI).
Dual codebase: Python v1 (`v1/`) and Rust port (`rust-port/wifi-densepose-rs/`).
### Key Rust Crates
| Crate | Description |
|-------|-------------|
| `wifi-densepose-core` | Core types, traits, error types, CSI frame primitives |
| `wifi-densepose-signal` | SOTA signal processing + RuvSense multistatic sensing (14 modules) |
| `wifi-densepose-nn` | Neural network inference (ONNX, PyTorch, Candle backends) |
| `wifi-densepose-train` | Training pipeline with ruvector integration + ruview_metrics |
| `wifi-densepose-mat` | Mass Casualty Assessment Tool — disaster survivor detection |
| `wifi-densepose-hardware` | ESP32 aggregator, TDM protocol, channel hopping firmware |
| `wifi-densepose-ruvector` | RuVector v2.0.4 integration + cross-viewpoint fusion (5 modules) |
| `wifi-densepose-api` | REST API (Axum) |
| `wifi-densepose-db` | Database layer (Postgres, SQLite, Redis) |
| `wifi-densepose-config` | Configuration management |
| `wifi-densepose-wasm` | WebAssembly bindings for browser deployment |
| `wifi-densepose-cli` | CLI tool (`wifi-densepose` binary) |
| `wifi-densepose-sensing-server` | Lightweight Axum server for WiFi sensing UI |
| `wifi-densepose-wifiscan` | Multi-BSSID WiFi scanning (ADR-022) |
| `wifi-densepose-vitals` | ESP32 CSI-grade vital sign extraction (ADR-021) |

### RuvSense Modules (`signal/src/ruvsense/`)
| Module | Purpose |
|--------|---------|
| `multiband.rs` | Multi-band CSI frame fusion, cross-channel coherence |
| `phase_align.rs` | Iterative LO phase offset estimation, circular mean |
| `multistatic.rs` | Attention-weighted fusion, geometric diversity |
| `coherence.rs` | Z-score coherence scoring, DriftProfile |
| `coherence_gate.rs` | Accept/PredictOnly/Reject/Recalibrate gate decisions |
| `pose_tracker.rs` | 17-keypoint Kalman tracker with AETHER re-ID embeddings |
| `field_model.rs` | SVD room eigenstructure, perturbation extraction |
| `tomography.rs` | RF tomography, ISTA L1 solver, voxel grid |
| `longitudinal.rs` | Welford stats, biomechanics drift detection |
| `intention.rs` | Pre-movement lead signals (200-500ms) |
| `cross_room.rs` | Environment fingerprinting, transition graph |
| `gesture.rs` | DTW template matching gesture classifier |
| `adversarial.rs` | Physically impossible signal detection, multi-link consistency |

### Cross-Viewpoint Fusion (`ruvector/src/viewpoint/`)
| Module | Purpose |
|--------|---------|
| `attention.rs` | CrossViewpointAttention, GeometricBias, softmax with G_bias |
| `geometry.rs` | GeometricDiversityIndex, Cramer-Rao bounds, Fisher Information |
| `coherence.rs` | Phase phasor coherence, hysteresis gate |
| `fusion.rs` | MultistaticArray aggregate root, domain events |

### RuVector v2.0.4 Integration (ADR-016 complete, ADR-017 proposed)
All 5 ruvector crates integrated in workspace:
- `ruvector-mincut` → `metrics.rs` (DynamicPersonMatcher) + `subcarrier_selection.rs`
- `ruvector-attn-mincut` → `model.rs` (apply_antenna_attention) + `spectrogram.rs`
- `ruvector-temporal-tensor` → `dataset.rs` (CompressedCsiBuffer) + `breathing.rs`
- `ruvector-solver` → `subcarrier.rs` (sparse interpolation 114→56) + `triangulation.rs`
- `ruvector-attention` → `model.rs` (apply_spatial_attention) + `bvp.rs`

### Architecture Decisions
43 ADRs in `docs/adr/` (ADR-001 through ADR-043). Key ones:
- ADR-014: SOTA signal processing (Accepted)
- ADR-015: MM-Fi + Wi-Pose training datasets (Accepted)
- ADR-016: RuVector training pipeline integration (Accepted — complete)
- ADR-017: RuVector signal + MAT integration (Proposed — next target)
- ADR-024: Contrastive CSI embedding / AETHER (Accepted)
- ADR-027: Cross-environment domain generalization / MERIDIAN (Accepted)
- ADR-028: ESP32 capability audit + witness verification (Accepted)
- ADR-029: RuvSense multistatic sensing mode (Proposed)
- ADR-030: RuvSense persistent field model (Proposed)
- ADR-031: RuView sensing-first RF mode (Proposed)
- ADR-032: Multistatic mesh security hardening (Proposed)

### Build & Test Commands (this repo)
```bash
# Rust — full workspace tests (1,031+ tests, ~2 min)
cd rust-port/wifi-densepose-rs
cargo test --workspace --no-default-features

# Rust — single crate check (no GPU needed)
cargo check -p wifi-densepose-train --no-default-features

# Rust — publish crates (dependency order)
cargo publish -p wifi-densepose-core --no-default-features
cargo publish -p wifi-densepose-signal --no-default-features
# ... see crate publishing order below

# Python — deterministic proof verification (SHA-256)
python v1/data/proof/verify.py

# Python — test suite
cd v1 && python -m pytest tests/ -x -q
```

### Crate Publishing Order
Crates must be published in dependency order:
1. `wifi-densepose-core` (no internal deps)
2. `wifi-densepose-vitals` (no internal deps)
3. `wifi-densepose-wifiscan` (no internal deps)
4. `wifi-densepose-hardware` (no internal deps)
5. `wifi-densepose-config` (no internal deps)
6. `wifi-densepose-db` (no internal deps)
7. `wifi-densepose-signal` (depends on core)
8. `wifi-densepose-nn` (no internal deps, workspace only)
9. `wifi-densepose-ruvector` (no internal deps, workspace only)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [attentiondotnet/Ruview](https://github.com/attentiondotnet/Ruview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
