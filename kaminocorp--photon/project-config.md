---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

```bash
cargo build                         # Dev build
cargo build --release               # Release build
cargo check                         # Type-check without building
cargo test                          # Run all tests (~138 tests across workspace)
cargo test -p photon-core           # Test core library only
cargo test -p photon                # Test CLI only
cargo test <test_name>              # Run a single test by name
cargo test -- --nocapture           # Show println/tracing output
cargo bench -p photon-core          # Run criterion benchmarks
cargo run -- process image.jpg      # Run CLI
cargo run -- models download        # Download SigLIP model from HuggingFace
```

No custom lint or format configuration — standard `cargo fmt` and `cargo clippy`. CI enforces:

```bash
cargo fmt --all -- --check
cargo clippy --workspace -- -D warnings
```

## Architecture

**Rust workspace** with two crates:
- `crates/photon-core` — Embeddable library (~7K lines). All processing logic lives here.
- `crates/photon` — CLI binary (~1.4K lines) using `clap`. Thin wrapper that calls into photon-core.

### Public API Surface (lib.rs re-exports)

```rust
pub use config::Config;
pub use embedding::EmbeddingEngine;
pub use error::{ConfigError, PhotonError, PipelineError, PipelineResult, Result};
pub use output::{OutputFormat, OutputWriter};
pub use pipeline::{ImageProcessor, ProcessOptions};
pub use types::{EnrichmentPatch, ExifData, OutputRecord, ProcessedImage, ProcessingStats, Tag};
```

### Processing Pipeline

`ImageProcessor` (`pipeline/processor.rs`) orchestrates a sequential pipeline:

```
Validate → Decode → EXIF → Hash (BLAKE3) → Perceptual Hash → Thumbnail (WebP) → Embed (SigLIP) → Tag (SigLIP) → ProcessedImage
                                                                                                                       ↓
                                                                                                              Enricher (LLM) → EnrichmentPatch
```

Each stage is a separate module under `photon-core/src/pipeline/`. The processor produces a `ProcessedImage` struct (defined in `types.rs`) serialized as JSON/JSONL via `OutputWriter`.

When `--llm` is active, the CLI uses a **dual-stream output** model: core pipeline results (`OutputRecord::Core`) emit immediately at full speed, then LLM descriptions follow as `OutputRecord::Enrichment` patches. Without `--llm`, output is identical to pre-LLM behavior.

The CLI's `process.rs` is decomposed into a `ProcessContext` struct and helper functions: `setup_processor()`, `process_single()`, `process_batch()`, `run_enrichment_collect()`, `run_enrichment_stdout()`. The actual `execute()` function is ~16 lines of orchestration.

### Embedding System

- `EmbeddingEngine` wraps a `SigLipSession` which holds `Mutex<ort::Session>` (ONNX Runtime)
- Loaded optionally via `ImageProcessor::load_embedding()` — processor works without it
- Inference runs in `tokio::task::spawn_blocking` with configurable timeout
- Model: `Xenova/siglip-base-patch16-224`, stored at `~/.photon/models/siglip-base-patch16/visual.onnx`
- Input preprocessing: resize to 224x224 or 384x384 (Lanczos3), normalize to [-1, 1]
- Output: 768-dim L2-normalized `Vec<f32>` from `pooler_output`
- Two model variants: 224 (fast, default) and 384 (higher detail, `--quality high`)

### Tagging System

The tagging subsystem is the most complex part of photon-core (~2.8K lines across 10 files in `tagging/`):

- **`TagScorer`** (`scorer.rs`) — holds `Vocabulary` + `LabelBank` + `TaggingConfig`. Brute-force scoring: dot product of image embedding × vocabulary matrix → SigLIP sigmoid → confidence. SigLIP scaling: `logit = 117.33 * cosine + (-12.93)`, then `sigmoid(logit)` (learned constants).
- **`Vocabulary`** (`vocabulary.rs`) — ~68K WordNet nouns + ~260 supplemental terms. Source files in `data/vocabulary/`, installed to `~/.photon/vocabulary/`.
- **`LabelBank`** (`label_bank.rs`) — pre-computed N×768 text embedding matrix cached at `~/.photon/taxonomy/label_bank.bin`. Cache invalidated via vocabulary hash in `label_bank.meta` sidecar.
- **`SigLipTextEncoder`** (`text_encoder.rs`) — encodes vocabulary terms through SigLIP text model (`text_model.onnx`). Must use `pooler_output` (2nd output), not `last_hidden_state`.
- **`RelevanceTracker`** (`relevance.rs`, 670 lines) — self-organizing three-pool system (Active/Warm/Cold) that promotes frequently-hit terms and demotes idle ones, reducing scoring cost over time.
- **`HierarchyDedup`** (`hierarchy.rs`) — suppresses ancestor tags when more specific descendants are present (e.g., removes "animal" when "dog" scores higher), with WordNet path annotations.
- **`ProgressiveEncoder`** (`progressive.rs`) — reduces cold-start from ~90min to ~30s by encoding a seed vocabulary first, then background-encoding remaining terms in chunks.
- **`NeighborExpander`** (`neighbors.rs`) — expands vocabulary graph with WordNet neighbors of high-scoring terms.
- **`SeedSelector`** (`seed.rs`) — picks high-value seed terms for progressive encoding.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kaminocorp/photon](https://github.com/kaminocorp/photon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
