---
trigger: always_on
description: Public operating guide for AI agents and automated coding assistants working in
---

# AGENTS.md

Public operating guide for AI agents and automated coding assistants working in
`advanced-gguf-quantizer`.

This repository is a llama.cpp-derived GGUF quantization toolkit focused on
NVFP4, MXFP6_E2M3, mixed NVFP4/MXFP6 models, calibration, evaluation,
repair/edit workflows, and release-quality artifacts.

## Read First

- Start with [README.md](README.md) for the human workflow.
- Use [docs/README.md](docs/README.md) as the detailed documentation index.
- Use [SKILLS.md](SKILLS.md) for agent-driven quantization runs.
- Use [docs/advanced-gguf-quantizer-flags.md](docs/advanced-gguf-quantizer-flags.md)
  for recipe and CLI details.
- Use [docs/advanced-gguf-quantizer-layer-policy.md](docs/advanced-gguf-quantizer-layer-policy.md)
  for tensor grouping and output/head/MTP policy.
- Use [docs/advanced-gguf-quantizer-cuda-acceleration.md](docs/advanced-gguf-quantizer-cuda-acceleration.md)
  before changing CUDA or runtime patch evaluation behavior.
- Use [docs/advanced-gguf-quantizer-imatrix-kld.md](docs/advanced-gguf-quantizer-imatrix-kld.md)
  before changing imatrix, saved-logit KLD, or evidence reuse behavior.
- Use [docs/advanced-gguf-quantizer-nvfp4-contract.md](docs/advanced-gguf-quantizer-nvfp4-contract.md)
  before changing NVFP4 GGUF writer, loader, or scale tensor behavior.
- Preserve [NOTICE.md](NOTICE.md), including Michael Wand attribution and MIT
  HAN Lab Four Over Six attribution.

## Project Boundaries

This project quantizes local GGUF files. It does not download models, import
Hugging Face checkpoints, or convert source checkpoints. Users should create
source GGUFs with a separate llama.cpp checkout and use this repository for
advanced quantization and evidence.

Keep the retained public tool surface focused on:

- `llama-quantize`
- `advanced-gguf-quantizer`
- `llama-imatrix`
- `llama-perplexity`
- `llama-completion`
- optional `llama-bench`
- optional `llama-fit-params`

Do not add server, web UI, model download, checkpoint import, or unrelated app
features unless the maintainer explicitly asks for them.

## Development Principles

- Preserve llama.cpp style and upstream-compatible behavior where possible.
- Keep changes scoped and reviewable.
- Prefer recipe/project fields over environment-variable product controls.
- Do not add compatibility aliases for old private commands or local research
  paths.
- Do not remove advanced quantization features to make code smaller unless the
  feature is genuinely dead or duplicated.
- Preserve MTP/NextN metadata and tensors by default.
- Keep output/head policy explicit and conservative.
- Treat MXFP6_E2M3 as experimental and unsupported by NVIDIA and llama.cpp; do
  not remove the compatibility warning from docs or the TUI.
- Treat NVFP4 as the speed-first format. Quality, correctness, and
  reproducibility still require full evidence; speed claims require separate,
  quiet-GPU benchmarks rather than assumptions.
- Make frequent commits for meaningful checkpoints when operating as an agent.

## Build

Use an existing `build` directory when present:

```bash
cd build
ninja -j 20
```

For a fresh local build:

```bash
cmake -S . -B build -DGGML_CUDA=ON -DCMAKE_BUILD_TYPE=Release
cmake --build build -j 20
```

Optional retained helper tools:

```bash
cmake -S . -B build \
  -DGGML_CUDA=ON \
  -DADVANCED_GGUF_QUANTIZER_BUILD_BENCH=ON \
  -DADVANCED_GGUF_QUANTIZER_BUILD_FIT_PARAMS=ON \
  -DCMAKE_BUILD_TYPE=Release
cmake --build build -j 20
```

## Verification

After C++ or CMake edits, run:

```bash
cd build
ninja -j 20
```

For CLI surface checks:

```bash
./build/bin/advanced-gguf-quantizer --help
./build/bin/llama-quantize --help
./build/bin/llama-imatrix --help
./build/bin/llama-perplexity --help
./build/bin/llama-completion --help
```

For text UI changes, run the PTY capture smoke:

```bash
python3 tools/quantize/advanced/capture_advanced_gguf_quantizer_tui.py \
  --out /tmp/agq-tui-capture \
  --rows 24 \
  --cols 100 \
  ./build/bin/advanced-gguf-quantizer
```

When tests are configured in a build tree, run the relevant `ctest` subset or
the full suite if the change touches shared runtime behavior.

## Quantization Workflow Expectations

Use recipes and projects for serious work:

```bash
./build/bin/advanced-gguf-quantizer recipe init --profile nvfp4_mxfp6 --output recipe.toml
./build/bin/advanced-gguf-quantizer recipe validate recipe.toml
./build/bin/advanced-gguf-quantizer plan recipe.toml
./build/bin/advanced-gguf-quantizer run recipe.toml --project runs/model --yes
```

Use `plan` and `recipe validate` for inspection. Use `run` when the user wants a
real model artifact. Do not use fake quantization passes as a substitute for a
real artifact run.

For saved-logit KLD base generation, use
`advanced-gguf-quantizer kld-command` and run the printed command as-is. It
should only name the reference GGUF, evaluation corpus, and output KLD base; do
not add runtime-shape or scheduling overrides.

Every production run should preserve:

- source GGUF path and hash when available;
- locked recipe;
- calibration corpus and imatrix;
- evaluation corpus and KLD base when used;
- final GGUF;
- tensor assignment log;
- run manifest;
- `checkpoint-key.json`;
- quantization report;
- validation smoke script.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [michaelw9999/advanced-gguf-quantizer](https://github.com/michaelw9999/advanced-gguf-quantizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
