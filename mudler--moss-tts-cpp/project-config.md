---
trigger: always_on
description: Durable reference for humans and agents maintaining this repo. Concise on
---

# Maintainer's guide — moss-tts.cpp

Durable reference for humans and agents maintaining this repo. Concise on
*why*; the README covers what users see.

## AI-assisted contributions

This project follows the same policy as the sibling repos (parakeet.cpp,
vibevoice.cpp) — the Linux kernel project's
[guidelines for AI coding assistants](https://docs.kernel.org/process/coding-assistants.html).
Commit rules:

- **No `Signed-off-by` from AI.** Only a human submitter signs off the DCO.
- **No `Co-Authored-By: <AI>` trailers.** The human contributor owns the change.
- **Use an `Assisted-by:` trailer** to attribute AI involvement. Format:
  `Assisted-by: AGENT_NAME:MODEL_VERSION [TOOL]`
  (e.g. `Assisted-by: Claude:claude-opus-4-8 [Claude Code]`).
- The human submitter reviews, tests, and understands every generated line.

## What this project is

moss-tts.cpp is a C++17/ggml inference port of the OpenMOSS
**MOSS-Audio-Tokenizer** (the "Cat" — Causal Audio Tokenizer with Transformer),
a CNN-free, pure-Transformer RVQ neural codec: 24 kHz waveform ↔ 32 RVQ code
streams at 12.5 Hz. It runs entirely on stock ggml (no fork), with no
Python/ONNX/torch at inference time, and ships a flat C-API
(`include/moss_tts_capi.h`) for `dlopen`/FFI/LocalAI integration.

This is the **Foundation** (F1 repo scaffold + F2 audio tokenizer) of a larger
program: porting the whole MOSS-TTS family to ggml. The decomposition, ordered
dependency-first, is `F1 → F2 → V1 → V2 → V3 → V4`:

```
FOUNDATION (this repo)  shared by every variant
  F1  repo scaffold (CMake + ggml + dr_wav + loader + audio_io + C-API/CLI)
  F2  MOSS-Audio-Tokenizer: encoder (wav→codes) + decoder (codes→wav)
        every variant decodes its audio through F2
V1  MossTTSDelay (8B)      Qwen3 backbone + 33 emb/heads + delay SM + sampling
V2  MossTTSLocal (1.7B)    depth-transformer backbone, time-sync RVQ blocks
V3  MossTTSRealtime (1.7B) hierarchical text-audio inputs + streaming
V4  MOSS-TTS-Nano (~100M)  separate upstream repo, 48 kHz
```

F2 comes first because every variant's audio is produced by this codec, it is
the riskiest novel ggml work (ONNX-only upstream), and it can be validated fully
standalone (reconstruction round-trip vs the ONNX tokenizer — no backbone). The
full design + rationale is in
[`docs/superpowers/specs/2026-06-03-foundation-audio-tokenizer-design.md`](docs/superpowers/specs/2026-06-03-foundation-audio-tokenizer-design.md).

### Reference sources (single sources of truth, in order of trust)

1. **`OpenMOSS-Team/MOSS-Audio-Tokenizer`** (HF) — `config.json` (dims) +
   `modeling_moss_audio_tokenizer.py` (module logic). Authoritative.
2. **`Blaizzy/mlx-audio`** `mlx_audio/codec/models/moss_audio_tokenizer/` —
   clean non-PyTorch reference for the algorithms.
3. **`OpenMOSS-Team/MOSS-Audio-Tokenizer-ONNX`** — the encoder/decoder ONNX we
   benchmark + parity-test against (the upstream torch-free pipeline).
4. The model's `model.safetensors.index.json` — authoritative tensor names.

When a numeric bug is suspected, dump the matching tensor from (1) and diff.
Magnitude alone is not a reliable signal (the vibevoice lesson).

## Layout

```
include/
  moss_tts.h            C++ API (moss::Codec: load/sample_rate/num_quantizers/encode/decode/reconstruct)
  moss_tts_capi.h       flat C-API for FFI / dlopen
src/
  moss_tts.cpp          C++ API impl (version + thin Codec pimpl over AudioTokenizer)
  moss_tts_capi.cpp     flat C-API shim
  audio_tokenizer.{hpp,cpp}  orchestrator: wires encoder towers + quantizer + decoder towers
  model_loader.{hpp,cpp}     GGUF -> config + name->ggml_tensor map; builds towers from metadata
  backend.{hpp,cpp}     ggml backend init + persistent ggml_gallocr; compute_graph_with_inputs()
  patchify.{hpp,cpp}    CNN-free down/up reshape-permute subgraph (PatchedPretransform)
  transformer.{hpp,cpp} ProjectedTransformer: RoPE MHA + LayerScale + erf-GELU FFN + sliding-window mask
  quantizer.{hpp,cpp}   ResidualLFQ: encode (argmin loop) + decode (gather+sum)
  rope.hpp              RoPE cos/sin (interleaved-pair convention)
  audio_io.{hpp,cpp}    dr_wav load/save + linear resample to 24 kHz
  common.{hpp,cpp}      logging
  ggml_extend.hpp       small ggml helpers
examples/cli/main.cpp   moss-tts-cli: info | encode | decode | reconstruct
scripts/
  convert_audio_tokenizer_to_gguf.py  safetensors -> GGUF (WNConv fusion, identity-proj drop, stage metadata)
  gen_test_fixtures.py                numpy/scipy reference dumps for the model-independent parity tests
  gen_onnx_reference.py               upstream ONNX encoder+decoder -> ref_recon.wav + ref_codes.npy (parity ref)
  requirements.txt                    converter deps + reference-only (onnxruntime, soundfile)
tests/
  test_*.cpp            ctest targets; SKIP_RETURN_CODE=77 means "skipped"
  fixtures/             committed tiny GGUF + numpy reference dumps
bench.sh                reconstruction RTF: moss-tts-cli vs ONNX on samples/*.wav
docs/
  conversion.md         authoritative GGUF tensor-naming + converter reference
  superpowers/specs/    the F1+F2 design spec
third_party/ggml        pinned submodule (no fork)
models/                 output dir for converted GGUFs (gitignored)
```

### Memory model (carried from parakeet.cpp — do not regress)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mudler/moss-tts.cpp](https://github.com/mudler/moss-tts.cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
