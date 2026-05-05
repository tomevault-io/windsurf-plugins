---
trigger: always_on
description: WiFi-based human pose estimation using Channel State Information (CSI).
---

# Claude Code Configuration — WiFi-DensePose + Claude Flow V3

## Project: wifi-densepose

WiFi-based human pose estimation using Channel State Information (CSI).
Dual codebase: Python v1 (`v1/`) and Rust port (`rust-port/wifi-densepose-rs/`).

### Key Rust Crates
- `wifi-densepose-signal` — SOTA signal processing (conjugate mult, Hampel, Fresnel, BVP, spectrogram)
- `wifi-densepose-train` — Training pipeline with ruvector integration (ADR-016)
- `wifi-densepose-mat` — Disaster detection module (MAT, multi-AP, triage)
- `wifi-densepose-nn` — Neural network inference (DensePose head, RCNN)
- `wifi-densepose-hardware` — ESP32 aggregator, hardware interfaces

### RuVector v2.0.4 Integration (ADR-016 complete, ADR-017 proposed)
All 5 ruvector crates integrated in workspace:
- `ruvector-mincut` → `metrics.rs` (DynamicPersonMatcher) + `subcarrier_selection.rs`
- `ruvector-attn-mincut` → `model.rs` (apply_antenna_attention) + `spectrogram.rs`
- `ruvector-temporal-tensor` → `dataset.rs` (CompressedCsiBuffer) + `breathing.rs`
- `ruvector-solver` → `subcarrier.rs` (sparse interpolation 114→56) + `triangulation.rs`
- `ruvector-attention` → `model.rs` (apply_spatial_attention) + `bvp.rs`

### Architecture Decisions
28 ADRs in `docs/adr/` (ADR-001 through ADR-028). Key ones:
- ADR-014: SOTA signal processing (Accepted)
- ADR-015: MM-Fi + Wi-Pose training datasets (Accepted)
- ADR-016: RuVector training pipeline integration (Accepted — complete)
- ADR-017: RuVector signal + MAT integration (Proposed — next target)
- ADR-024: Contrastive CSI embedding / AETHER (Accepted)
- ADR-027: Cross-environment domain generalization / MERIDIAN (Accepted)
- ADR-028: ESP32 capability audit + witness verification (Accepted)

### Build & Test Commands (this repo)
```bash
# Rust — full workspace tests (1,031 tests, ~2 min)
cd rust-port/wifi-densepose-rs
cargo test --workspace --no-default-features

# Rust — single crate check (no GPU needed)
cargo check -p wifi-densepose-train --no-default-features

# Python — deterministic proof verification (SHA-256)
python v1/data/proof/verify.py

# Python — test suite
cd v1 && python -m pytest tests/ -x -q
```

### Validation & Witness Verification (ADR-028)

**After any significant code change, run the full validation:**

```bash
# 1. Rust tests — must be 1,031+ passed, 0 failed
cd rust-port/wifi-densepose-rs
cargo test --workspace --no-default-features

# 2. Python proof — must print VERDICT: PASS
cd ../..
python v1/data/proof/verify.py

# 3. Generate witness bundle (includes both above + firmware hashes)
bash scripts/generate-witness-bundle.sh

# 4. Self-verify the bundle — must be 7/7 PASS
cd dist/witness-bundle-ADR028-*/
bash VERIFY.sh
```

**If the Python proof hash changes** (e.g., numpy/scipy version update):
```bash
# Regenerate the expected hash, then verify it passes
python v1/data/proof/verify.py --generate-hash
python v1/data/proof/verify.py
```

**Witness bundle contents** (`dist/witness-bundle-ADR028-<sha>.tar.gz`):
- `WITNESS-LOG-028.md` — 33-row attestation matrix with evidence per capability
- `ADR-028-esp32-capability-audit.md` — Full audit findings
- `proof/verify.py` + `expected_features.sha256` — Deterministic pipeline proof
- `test-results/rust-workspace-tests.log` — Full cargo test output
- `firmware-manifest/source-hashes.txt` — SHA-256 of all 7 ESP32 firmware files
- `crate-manifest/versions.txt` — All 15 crates with versions
- `VERIFY.sh` — One-command self-verification for recipients

**Key proof artifacts:**
- `v1/data/proof/verify.py` — Trust Kill Switch: feeds reference signal through production pipeline, hashes output
- `v1/data/proof/expected_features.sha256` — Published expected hash
- `v1/data/proof/sample_csi_data.json` — 1,000 synthetic CSI frames (seed=42)
- `docs/WITNESS-LOG-028.md` — 11-step reproducible verification procedure
- `docs/adr/ADR-028-esp32-capability-audit.md` — Complete audit record

### Branch
Default branch: `main`

---

## Behavioral Rules (Always Enforced)

- Do what has been asked; nothing more, nothing less
- NEVER create files unless they're absolutely necessary for achieving your goal
- ALWAYS prefer editing an existing file to creating a new one
- NEVER proactively create documentation files (*.md) or README files unless explicitly requested
- NEVER save working files, text/mds, or tests to the root folder
- Never continuously check status after spawning a swarm — wait for results
- ALWAYS read a file before editing it
- NEVER commit secrets, credentials, or .env files

## File Organization

- NEVER save to root folder — use the directories below
- `docs/adr/` — Architecture Decision Records
- `rust-port/wifi-densepose-rs/crates/` — Rust workspace crates (signal, train, mat, nn, hardware)
- `v1/src/` — Python source (core, hardware, services, api)
- `v1/data/proof/` — Deterministic CSI proof bundles
- `.claude-flow/` — Claude Flow coordination state (committed for team sharing)
- `.claude/` — Claude Code settings, agents, memory (committed for team sharing)

## Project Architecture

- Follow Domain-Driven Design with bounded contexts
- Keep files under 500 lines
- Use typed interfaces for all public APIs
- Prefer TDD London School (mock-first) for new code
- Use event sourcing for state changes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [euaziel/WiFi-CSI-Human-Pose-Detection](https://github.com/euaziel/WiFi-CSI-Human-Pose-Detection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
