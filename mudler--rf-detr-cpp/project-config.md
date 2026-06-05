---
trigger: always_on
description: Durable reference for humans and agents maintaining rf-detr.cpp.
---

# AGENTS.md

Durable reference for humans and agents maintaining rf-detr.cpp.

## What this project is

rf-detr.cpp is a C++/ggml inference engine for Roboflow RF-DETR. It runs
detection and segmentation natively on CPU with parity to the PyTorch
reference, and is published on HuggingFace as a set of 32 quantized GGUF
models (5 detection variants x 4 quants + 3 segmentation variants x 4 quants
plus a few extras).

The repo also exposes a flat C-API (`include/rfdetr_capi.h`) intended for
dlopen-based language bindings, and is integrated into LocalAI as a native
backend.

## Repository layout

```
src/                 C++ implementation
                     dinov2, projector, two_stage, decoder, heads,
                     segmentation, transformer_ops, postprocess,
                     model_loader, image_io, backend, trace,
                     rfdetr, rfdetr_model, rfdetr_capi
include/             public headers
                     rfdetr.h        (native C++/C API)
                     rfdetr_capi.h   (flat C-API for FFI / dlopen)
examples/cli/        rfdetr-cli with subcommands:
                     detect, bench, info, quantize
scripts/             converter, batch scripts, benchmark, plot, HF publish
tests/               ctest targets (parity, capi, CI smoke fixtures)
tests/ci/            compare_detections.py used by GitHub Actions smoke job
tests/fixtures/      baseline_torch*.gguf + small CI image and expected JSONs
benchmarks/          results JSON + matplotlib plots
third_party/         vendored ggml submodule, stb_image, patches
third_party/ggml-patches/  local ggml patches applied at configure time
models/              output dir for converted GGUFs (gitignored;
                     MANIFEST.md tracks the expected set)
docs/                conversion, finetuning, parity, variants references
.github/workflows/   ci.yml (build + smoke-test jobs)
```

## Build

```
cmake -B build -DRFDETR_BUILD_TESTS=ON -DRFDETR_BUILD_CLI=ON -DGGML_NATIVE=ON
cmake --build build -j
ctest --test-dir build --output-on-failure
```

Expected: 24/24 tests pass on a typical Linux dev box. Parity tests require
`tests/fixtures/baseline_torch*.gguf` to be present (committed to the repo).

### CMake options

| Option                | Default | Purpose                                       |
| --------------------- | ------- | --------------------------------------------- |
| `RFDETR_BUILD_TESTS`  | OFF     | Compile and register ctest targets            |
| `RFDETR_BUILD_CLI`    | ON      | Build the `rfdetr-cli` example binary         |
| `RFDETR_SHARED`       | OFF     | Build `librfdetr` as a shared library (dlopen)|
| `RFDETR_GGML_CUDA`    | OFF     | Forwarded to ggml (`GGML_CUDA`)               |
| `RFDETR_GGML_METAL`   | OFF     | Forwarded to ggml (`GGML_METAL`)              |
| `RFDETR_GGML_VULKAN`  | OFF     | Forwarded to ggml (`GGML_VULKAN`)             |
| `RFDETR_GGML_HIPBLAS` | OFF     | Forwarded to ggml (`GGML_HIPBLAS`)            |

Notes:
- GPU backends are wired through but not exercised in CI. CPU is the
  supported path today.
- For LocalAI integration build with `-DRFDETR_SHARED=ON` to get
  `librfdetr.so`.

## Converting a model

Set up a Python venv with the upstream rfdetr package first.

```
python3 -m venv .venv
.venv/bin/pip install rfdetr

.venv/bin/python scripts/convert_rfdetr_to_gguf.py \
    --variant base --dtype f16 \
    --output models/rfdetr-base-f16.gguf
```

Supported `--variant`:
- Detection: `nano`, `small`, `base`, `medium`, `large`
- Segmentation: `seg-nano`, `seg-small`, `seg-medium`, `seg-large`,
  `seg-xlarge`, `seg-2xlarge`

Supported `--dtype` (handled by the Python writer):
`f32`, `f16`, `q4_0`, `q4_1`, `q5_0`, `q5_1`, `q8_0`

For K-quants (`Q4_K`, `Q5_K`, `Q6_K`) the Python `gguf` writer doesn't have
support, so quantize an F32 GGUF with the CLI:

```
build/bin/rfdetr-cli quantize \
    models/rfdetr-base-f32.gguf \
    models/rfdetr-base-q4_K.gguf \
    q4_K
```

Custom fine-tuned checkpoints: pass `--checkpoint path/to/local.pth` to
override the pretrained download.

## Running inference

```
build/bin/rfdetr-cli detect \
    --model models/rfdetr-base-f16.gguf \
    --input image.jpg \
    --threshold 0.5 \
    --output dets.json
```

For segmentation models, also pass `--masks /path/to/mask_output_dir/` to
write one PNG per detection.

Other CLI subcommands: `bench`, `info`, `quantize`.

## GGUF schema

See `docs/conversion.md` for the full schema. Quick summary:

- Format version metadata key: `"2"`.
- Tensor naming convention mirrors the PyTorch state dict, with `.` swapped
  for `.` and a small set of fixups (backbone, projector, decoder, heads,
  segmentation prototype mask head).
- Only 2D weight tensors with both dims `>= 32` and divisible by the block
  size of the target quant get quantized. Embeddings, biases, norms and
  small projection matrices stay F32.

## Test fixtures

CI smoke uses small committed fixtures under `tests/fixtures/ci/`:
- `test_image.jpg`: the test input
- `expected_<variant>-<dtype>.json`: expected detections at T=0.55

To regenerate after a deliberate behavior change:

```
for v in nano-f32 nano-f16 nano-q8_0 nano-q4_K base-f16; do
    ./build/bin/rfdetr-cli detect \
        --model models/rfdetr-${v}.gguf \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mudler/rf-detr.cpp](https://github.com/mudler/rf-detr.cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
