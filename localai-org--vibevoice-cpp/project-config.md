---
trigger: always_on
description: A pragmatic guide for whoever is poking at this repo next. Concise on
---

# Maintainer's guide — vibevoice.cpp

A pragmatic guide for whoever is poking at this repo next. Concise on
purpose; the README covers what users see.

## What this project is

A C++/ggml port of Microsoft VibeVoice. One binary (`vibevoice-cli`)
does both **TTS** (text → 24 kHz WAV with voice cloning) and **ASR**
(WAV → JSON transcript). Built on stock ggml (no fork). Public C API in
[`include/vibevoice.h`](include/vibevoice.h) so other projects can
embed it via dlopen / purego / cgo.

Reference impls we trust, in order:
1. **`microsoft/VibeVoice`** (`vibevoice/modular/`) — the one that was
   actually trained. Single source of truth on weights + math.
2. **`Blaizzy/mlx-audio`** (`mlx_audio/{stt,tts}/models/vibevoice…/`) —
   closest analog to what we're writing. Useful when upstream PyTorch
   does something tricky and we want a non-PyTorch confirmation.
3. **`transformers/models/vibevoice_*/`** — a refactored re-port. Differs
   from upstream in subtle places (sampling formula, processor API).
   Cross-check before trusting.

When you suspect a numerical bug in our code, dump the matching tensor
from the chosen reference and diff. There's a template at
`/tmp/asr_ref_compare.py` (not committed; lives only on the dev box) that
shows how to load just the encoder + connector shards from the 7B ASR
checkpoint and run them standalone.

## Layout

```
include/vibevoice.h          # public C API (purego / dlopen target)
src/
  vibevoice.cpp              # public-API impl
  vibevoice_tts.{hpp,cpp}    # TTS orchestrator (M5)
  vibevoice_asr.{hpp,cpp}    # ASR orchestrator (M6)
  qwen2.{hpp,cpp}            # Qwen2 transformer block + GQA + KV cache
  acoustic_tokenizer.{hpp,cpp}# VAE encoder + decoder
  diffusion_head.{hpp,cpp}   # DiffusionHead + DPM-Solver
  conv1d.{hpp,cpp}           # SConv1d / SConvTranspose1d
  rms_norm.hpp               # ConvRMSNorm
  model_loader.{hpp,cpp}     # gguf reader (mmap → name → ggml_tensor)
  tokenizer.{hpp,cpp}        # vendored Qwen2 byte-level BPE
  audio_io.{hpp,cpp}         # dr_wav wrap + linear resampler
scripts/
  convert_tokenizer.py
  convert_vibevoice_to_gguf.py
  convert_voice_to_gguf.py
  quantize_gguf.py
tests/
  test_*.cpp                 # ~14 ctests; SKIP_RETURN_CODE=77 = skip
  fixtures/tokenizer.gguf    # tiny tokenizer fixture (committed)
docs/
  conversion.md              # tensor naming + quant notes
.github/workflows/ci.yml     # build+test (Linux+macOS) + closed-loop on dispatch
third_party/ggml             # pinned submodule
```

## Build

```bash
git clone --recursive <repo> && cd vibevoice.cpp
cmake -B build -DVIBEVOICE_BUILD_TESTS=ON -DCMAKE_BUILD_TYPE=Release
cmake --build build -j
ctest --test-dir build --output-on-failure
```

CMake options:
- `VIBEVOICE_BUILD_TESTS` — register ctests
- `VIBEVOICE_TEST_LARGE` — enable model-dependent tests (closed-loop, long-form). They still skip 77 if env vars aren't set, so this is safe to leave on.
- `VIBEVOICE_BUILD_EXAMPLES` (default ON) — `vibevoice-cli`
- `VIBEVOICE_GGML_CUDA` / `VIBEVOICE_GGML_METAL` — pass through to the ggml submodule.

## Tests at a glance

| File                            | What it does                                            | Needs models? |
| ------------------------------- | ------------------------------------------------------- | :-----------: |
| `test_smoke`                    | Lib loads, version string is non-empty                  | no            |
| `test_audio_io`                 | dr_wav round-trip                                       | no            |
| `test_tokenizer`                | Qwen2 BPE id-level parity vs HF on a fixture            | no            |
| `test_rope`                     | RoPE cos/sin tables vs PyTorch                          | no            |
| `test_qwen2_block`              | Qwen2 forward pass numerics                             | no            |
| `test_sconv1d`                  | Causal conv1d / convtranspose1d numerics                | no            |
| `test_block1d`                  | ConvNeXt Block1D forward                                | no            |
| `test_acoustic`                 | Encoder + decoder forward on tiny random weights        | no            |
| `test_diffusion_head`           | TimestepEmbedder + DiffusionHead forward                | no            |
| `test_dpm_solver`               | DPM-Solver++ multistep schedule                         | no            |
| `test_load_realtime`            | Real 0.5B gguf opens cleanly                            | yes (env)     |
| `test_e2e_tts`                  | Real TTS produces non-silent / non-clipped audio        | yes (env)     |
| `test_e2e_asr`                  | Real ASR encoder runs + tone smoke                      | yes (env)     |
| `test_closed_loop`              | TTS → ASR roundtrip; ≥80 % source-word recall           | **yes**       |
| `test_long_form_asr`            | 65 s audio (TTS×N) round-trips with multi-segment match | **yes**       |

Env vars for the model-dependent tests (set whichever ones you need):

```
VIBEVOICE_MODEL        # alias used by older tests; .gguf path
VIBEVOICE_TTS_MODEL    # closed-loop / long-form TTS path
VIBEVOICE_ASR_MODEL    # closed-loop / long-form ASR path

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [localai-org/vibevoice.cpp](https://github.com/localai-org/vibevoice.cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
