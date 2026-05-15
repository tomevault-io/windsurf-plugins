---
trigger: always_on
description: Pure Rust inference engine. Apache-2.0. github.com/ohdearquant/lattice
---

# Lattice Development Guidelines

Pure Rust inference engine. Apache-2.0. github.com/ohdearquant/lattice

## AI-Assisted Contribution Policy

- Do not open PRs with unverified AI-generated code.
- Every claim in a PR description must match the actual diff.
- SIMD code requires manual verification — AI models cannot reason about intrinsic correctness.
- Include `cargo test` and `cargo bench` output for performance-sensitive changes.

## Common Rules

- Do not add CUDA support. Lattice targets CPU (AVX2/NEON) + Metal (Apple Silicon) + WGPU fallback.
- Do not add external ML runtime dependencies (ONNX, PyTorch, TensorFlow).
- Do not add `unsafe` blocks outside of SIMD intrinsics and raw pointer arithmetic in hot paths.
- Do not use `unwrap()` or `expect()` in library code. Tests and examples only.
- Do not add upward dependencies (embed cannot depend on tune, inference cannot depend on embed).
- Do not push directly to `main`. All changes go through feature branch → PR → review → merge.
- Internal path dependencies must include a version for crates.io: `lattice-foo = { version = "0.1.0", path = "../foo" }`.

```rust
// === BAD — unwrap in library code ===
pub fn load_model(path: &str) -> Model {
    let data = std::fs::read(path).unwrap();
    Model::from_bytes(&data).unwrap()
}

// === GOOD — propagate errors ===
pub fn load_model(path: &str) -> Result<Model, InferenceError> {
    let data = std::fs::read(path)?;
    Model::from_bytes(&data)
}
```

```rust
// === BAD — allocating in hot path ===
pub fn forward(&self, input: &[f32]) -> Vec<f32> {
    let mut buffer = vec![0.0f32; self.hidden_size]; // allocation every call
    self.compute(input, &mut buffer);
    buffer
}

// === GOOD — pre-allocated buffer ===
pub fn forward(&self, input: &[f32], buffer: &mut [f32]) {
    self.compute(input, buffer);
}
```

```rust
// === BAD — unsafe without safety comment ===
unsafe fn dot_neon(a: *const f32, b: *const f32, n: usize) -> f32 {
    let v = vld1q_f32(a);
    // ...
}

// === GOOD — documented safety invariant ===
/// # Safety
/// `a` and `b` must point to at least `n` valid f32 values.
/// CPU must support NEON (guaranteed on aarch64).
#[target_feature(enable = "neon")]
unsafe fn dot_neon(a: *const f32, b: *const f32, n: usize) -> f32 {
    // SAFETY: caller guarantees a and b have n elements; NEON checked by target_feature
    let v = vld1q_f32(a);
    // ...
}
```

```rust
// === BAD — clone in hot path ===
fn similarity(query: &[f32], docs: &[Vec<f32>]) -> Vec<f32> {
    docs.iter().map(|d| cosine(query, &d.clone())).collect()
}

// === GOOD — borrow ===
fn similarity(query: &[f32], docs: &[Vec<f32>]) -> Vec<f32> {
    docs.iter().map(|d| cosine(query, d)).collect()
}
```

## Crate Structure

```
inference (57K)  fann (7.5K)  transport (5.3K)   ← leaf, zero internal deps
    |               |
  embed (14K)    tune (13K)                       ← depend on leaves only
```

### lattice-inference — Transformer kernel

| Module           | Purpose                      | Key Exports                                                                       |
| ---------------- | ---------------------------- | --------------------------------------------------------------------------------- |
| `model/`         | Model configs and loaders    | `BertConfig`, `BertModel`, `QwenConfig`, `QwenModel`, `CrossEncoderModel`         |
| `tokenizer/`     | Pure Rust tokenizers         | `Tokenizer` trait, `WordPieceTokenizer`, `SentencePieceTokenizer`, `BpeTokenizer` |
| `weights/`       | Weight storage formats       | `F32Weights`, `F16Weights`, `Q8Weights`, `Q4Weights`                              |
| `attention/`     | Attention mechanisms         | `flash_attention`, `gqa_attention`, `GatedDeltaNetState`                          |
| `forward/`       | Compute backends             | `cpu/`, `metal_qwen35.rs` (Metal MSL), NEON/AVX2 kernels                          |
| `kv_cache/`      | KV cache for generation      | `FlatKVCache`, `PagedKVCache`                                                     |
| `generate.rs`    | Autoregressive generation    | `GenerateConfig`                                                                  |
| `speculative.rs` | Speculative decoding         | `NgramSpeculator`, `MtpVerifier`                                                  |
| `lora_hook.rs`   | LoRA adapter injection trait | `LoraHook`, `NoopLoraHook`                                                        |
| `rope.rs`        | Rotary positional encoding   | `RopeTable`                                                                       |
| `sampling.rs`    | Token sampling               | `SamplingConfig`, `sample_token`                                                  |
| `download.rs`    | HuggingFace model download   | `ensure_model_files`                                                              |

### lattice-embed — Embedding service

| Module      | Purpose                        | Key Exports                                                        |
| ----------- | ------------------------------ | ------------------------------------------------------------------ |
| `model/`    | Model variant registry         | `EmbeddingModel` (9 variants), `ModelConfig`, `ModelProvenance`    |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ohdearquant/lattice](https://github.com/ohdearquant/lattice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
