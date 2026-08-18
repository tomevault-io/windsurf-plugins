---
trigger: always_on
description: Durable reference for humans and agents maintaining voice-detect.cpp.
---

# AGENTS.md

Durable reference for humans and agents maintaining voice-detect.cpp.

## AI-assisted contributions

This project follows the Linux kernel project's
[guidelines for AI coding assistants](https://docs.kernel.org/process/coding-assistants.html)
(the same policy LocalAI uses). Key rules for commits:

- **No `Signed-off-by` from AI.** Only a human submitter may sign off on the
  Developer Certificate of Origin.
- **No `Co-Authored-By: <AI>` trailers.** The human contributor owns the change.
- **Use an `Assisted-by:` trailer** to attribute AI involvement. Format:
  `Assisted-by: AGENT_NAME:MODEL_VERSION [TOOL1] [TOOL2]`
  (e.g. `Assisted-by: Claude:claude-opus-4-8 [Claude Code]`).
- The human submitter is responsible for reviewing, testing, and understanding
  every line of generated code.

## What this project is

voice-detect.cpp is a C++17/ggml inference engine for speaker recognition (voice
biometrics). A Python converter turns a speaker-embedding checkpoint (SpeechBrain
ECAPA-TDNN, or an ONNX WeSpeaker / 3D-Speaker encoder) into a metadata-driven
GGUF; the C++ model loader + encoder run the same computation natively with no
Python dependency at inference time.

Pipeline: decode/resample audio -> 16 kHz mono -> 80-dim Kaldi-compatible FBank
features -> speaker encoder -> L2-normalized embedding. Plus `verify` (cosine
distance vs threshold) and `analyze` (age/gender/emotion, phased last).

The public surface ships as a flat C-API (`include/voicedetect_capi.h` +
`libvoicedetect.so`) suitable for `dlopen`/FFI, and is the native replacement for
LocalAI's Python `speaker-recognition` backend.

## Performance invariants (do not regress)

These mirror parakeet.cpp's measured wins; keep them when implementing graphs:

- **Keep the persistent `ggml_gallocr`** in `src/backend.cpp`. Reusing one
  allocator across the many small per-utterance graphs (no per-call alloc/free)
  is the core throughput lever. The scheduler is used ONLY as a per-graph
  fallback when the active GPU backend lacks a kernel for some op.
- **Zero-copy weights.** `clone_weight` returns loader tensors directly so the
  same device buffer is reused every call; do not copy weights per call.

## Repository layout

```
include/             public C/C++ headers
                       voicedetect.h        , version + tiny C++ namespace vd layer
                       voicedetect_capi.h   , flat C-API for FFI / dlopen
src/                 libvoicedetect implementation
                       voicedetect.cpp      , version + thin vd::embed/verify wrappers
                       voicedetect_capi.cpp , flat C-API implementation
                       model.hpp/cpp        , load-once vd::Model orchestration (stub)
                       model_loader.hpp/cpp , GGUF -> VoiceDetectConfig + name->tensor
                       backend.hpp/cpp      , persistent ggml_backend_t + gallocr
                       fbank.hpp/cpp        , Kaldi-compatible FBank front end (stub)
                       audio_io.hpp/cpp     , dr_wav load + linear resample to 16k
                       common.hpp/cpp       , logging helpers
examples/cli/        voicedetect-cli binary (info, embed, verify, analyze, bench)
scripts/             Python tooling (reference-side only)
                       convert_voicedetect_to_gguf.py , checkpoint -> GGUF (--dtype f32|f16|q8_0)
                       gen_baseline.py                , reference intermediates -> baseline.gguf
                       apply_ggml_patches.sh          , optional in-tree ggml patches
                       requirements.txt               , speechbrain/torch/torchaudio/onnxruntime/gguf
tests/               ctest targets
                       parity.hpp           , header-only golden compare + cosine
                       test_smoke.cpp       , version + ABI (model-independent)
                       test_fbank.cpp       , FBank golden parity (RC-77 skip)
                       fixtures/            , small clips (not committed; see README)
                       python/check_convert.py , converter round-trip
docs/
  conversion.md     , GGUF schema reference
  parity.md         , coverage matrix + parity gates (cosine>=0.9999, max|d|<=1e-3, identical verdict)
  quantization.md   , quantization allowlist + policy
third_party/         vendored deps: ggml/ (submodule), dr_wav.h (single header)
models/              output dir for converted GGUFs (gitignored; MANIFEST.md tracked)
.github/workflows/   ci.yml, docker.yml, release.yml
```

## Build

```
cmake -B build -DVOICEDETECT_BUILD_TESTS=ON -DGGML_NATIVE=ON && cmake --build build -j
```

Use `-DGGML_NATIVE=OFF` when building for CI or portable binaries.

## Running tests

```
# Model-independent (run anywhere)
ctest --test-dir build --output-on-failure -LE model    # test_smoke

# Model/baseline-dependent (need venv + reference baseline; SKIP/77 without)
export VOICEDETECT_TEST_GGUF=/tmp/model.gguf
export VOICEDETECT_TEST_BASELINE=/tmp/baseline.gguf
export VOICEDETECT_TEST_AUDIO=tests/fixtures/clip.wav
ctest --test-dir build --output-on-failure
```

## C-API and LocalAI integration

`include/voicedetect_capi.h` defines the flat C-API. Build `libvoicedetect.so`
with `-DVOICEDETECT_SHARED=ON`. Verify exports with

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [localai-org/voice-detect.cpp](https://github.com/localai-org/voice-detect.cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
