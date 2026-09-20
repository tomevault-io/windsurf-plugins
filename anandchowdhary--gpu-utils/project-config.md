---
trigger: always_on
description: Tiny task-specific models, trained from scratch, running on WebGPU in the browser.
---

# gpu-utils: agent guide

Tiny task-specific models, trained from scratch, running on WebGPU in the browser.
Each package is one task, one model, zero runtime dependencies, under a strict Brotli
size budget. This file is the contract every contributor and every coding agent follows.
Read it fully before touching a package.

## The recipe (do not deviate without a written reason in the package README)

1. **CPU pre-pass.** `tokenize()` from `@gpu-utils/runtime` splits text into runs by
   character class. Each token gets a handful of sparse hashed feature ids (word hash,
   consonant-skeleton hash, shape, first/last char, length bucket, task-specific flags).
   No learned vocabulary. The featurizer in `src/features.ts` and
   `training/<snake>/features.py` must be byte-for-byte equivalent. Give every feature
   family its own block of ids in one embedding table (see the template) so ids never
   collide; `slots` is the fixed number of ids per token.
2. **Model: use the shared families.** `gpu_utils_training.models` provides two reference
   families with one interface, `forward(rows, mask) -> {"tags": [B,T,L], "pooled": [B,P] | None}`,
   whose CPU forward (`packages/runtime/src/models.ts`) and WGSL kernels
   (`packages/runtime/src/wgsl/{scan,conv}_tagger.wgsl`) already exist and are parity-tested
   on every CI run:
   - *Scan family* `ScanTagger(feature_rows, hidden, tags, pooled_out=0, scan_layers=1)`:
     summed sparse embeddings → bidirectional gated affine scans
     `h = a*h_prev + (1-a)*tanh(u)`, `a = sigmoid(...)` → residual 5-tap depthwise conv →
     mean-pooled context → two-layer head (+ optional pooled head). Best for short
     natural-language inputs (queries, schedules, commands, references). ~30–60K params.
   - *Conv family* `ConvTagger(feature_rows, embed, hidden, blocks, dilations, tags, pooled_out=0)`:
     embeddings → projection → residual blocks `x + relu(conv3_dilated(x)) @ W2` →
     per-token head (+ optional pooled head). Best for long documents (logs, prose, email).
     100K–1M params.
   **Only write a custom `nn.Module` (with matching cpu.ts and shader.wgsl) if the package
   README explains why neither family fits.** Extra heads (a boundary logit, a kind
   classifier) fit the families: use `tags` for per-token outputs and `pooled_out` for
   per-sequence outputs, then split the logits in the decoder.
   The model only tags tokens or emits a small set of roles. It never generates free text.
3. **Decoder.** Viterbi over the tags on the CPU (`viterbi` + `bioTransitions` +
   `bioToSpans` from the runtime), then a deterministic TypeScript compiler turns tags into
   the typed output (filter AST, cron string, spans, RRULE). All semantics, validation, and
   error messages live in the compiler, not the model.
4. **Training.** PyTorch CPU, `uv`-managed, through `gpu_utils_training.loop.train`
   (AdamW, warm-up + cosine, int6 QAT from epoch 1, evaluation of the quantized model every
   epoch, best checkpoint, `history.json`, wall-clock budget). Data is synthetic-first,
   generated from a grammar with a teacher where one exists; real data is held out for
   evaluation. Training must run on CPU in under 30 minutes for the default config.
5. **Export.** `gpu_utils_training.export.export_package` writes `model/manifest.json`,
   `model/weights.txt` (int6 text encoding, see `quant.py`) and `model/fixtures.json` in
   the canonical format `{"cases": [{"input", "rows", "logits", "pooled"}]}`, computed from
   the decoded int6 weights so parity is exact.
6. **Runtime.** `src/cpu.ts` calls the family forward from the runtime (the reference).
   `src/gpu.ts` calls `runScanTagger` / `runConvTagger` on the canonical kernels and must
   match the CPU path to 1e-4 (`training/tests/test_wgsl.py` checks this on Mesa lavapipe
   in CI). "auto" backend uses CPU for small inputs (GPU readback dominates below ~256
   tokens). Never silently return empty results when WebGPU is missing: fall back to CPU.

## Package layout

```
packages/<name>/
  package.json          name, description, gpuUtils.sizeBudget (bytes, Brotli)
  README.md             install, one usage example, how it works, size table, limitations
  MODEL_CARD.md         architecture, params, data, eval tables, latency, checkpoint id
  src/index.ts          public API only: parse(text, options)
  src/features.ts       featurizer (parity with Python)
  src/cpu.ts            reference forward pass (family forward from the runtime)
  src/gpu.ts            WebGPU forward pass (runScanTagger / runConvTagger)
  src/decode.ts         Viterbi + tag→typed output compiler
  src/model.ts          loads model/manifest.json + weights.txt
  src/shader.wgsl       ONLY for a justified custom kernel (see step 2)
  model/                promoted checkpoint artefacts (committed)
  test/                 vitest: features, decode, parity against model/fixtures.json
  training/             uv project: data.py, model.py, train.py, evaluate.py, export.py, tests/
video/scenes/<name>.md  storyboard
video/<snake>_pipeline.py  Manim explainer
```

Scaffold with `pnpm new <name> "<description>"`. The scaffold ships a random-weight
scan model and a placeholder task so every check passes before you train.

## Shared modules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AnandChowdhary/gpu-utils](https://github.com/AnandChowdhary/gpu-utils) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
