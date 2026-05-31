---
trigger: always_on
description: Durable reference for humans and agents maintaining parakeet.cpp.
---

# AGENTS.md

Durable reference for humans and agents maintaining parakeet.cpp.

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

parakeet.cpp is a C++17/ggml inference port of NVIDIA NeMo Parakeet ASR.
It targets CPU (GPU backends are wired but not exercised in CI) and is designed
for parity with the NeMo reference: a Python converter turns a NeMo checkpoint
into a metadata-driven GGUF, and a C++ model loader + conformer inference engine
run the same computation natively, with no Python dependency at inference time.

The public surface ships as a flat C-API (`include/parakeet_capi.h` +
`libparakeet.so`) suitable for `dlopen`/FFI/LocalAI integration.

Current status: Phase 5 complete.  Supports all offline Parakeet families -
CTC, RNNT, TDT, and hybrid TDT-CTC (0.6B/1.1B/110M, EN + multilingual v3) -
validated at WER 0 vs NeMo on every published checkpoint.  Quantization
(F16/Q8_0/K-quants) validated at WER 0.  Cache-aware streaming + EOU decoding
(`parakeet_realtime_eou_120m-v1`) is implemented: `pk::StreamingEncoder`
(per-layer conv/attention caches) + `pk::StreamingSession` (carried RNN-T
state) + `<EOU>`/`<EOB>` timed events, exposed via `parakeet_capi_stream_*` and
`parakeet-cli transcribe --stream`.  The streaming transcript matches NeMo's
cache-aware streaming byte-for-byte.

## Repository layout

```
include/             public C/C++ headers
                       parakeet.h         , C++ API
                       parakeet_capi.h    , flat C-API for FFI / dlopen
src/                 libparakeet implementation
                       model.hpp/cpp      , load-once pk::Model
                       parakeet.cpp       , thin transcribe() wrapper
                       parakeet_capi.cpp  , flat C-API implementation
                       common.hpp/cpp     , logging helpers
                       audio_io.hpp/cpp   , dr_wav load + linear resample to 16k
                       model_loader.hpp/cpp, GGUF -> ParakeetConfig + name->tensor
                       mel.cpp            , log-mel frontend
                       encoder.cpp / conformer.cpp / relpos_attention.cpp
                       ctc_decoder.cpp    , CTC head + greedy decode
                       prediction.cpp     , stacked LSTM prediction net
                       joint.cpp          , joint network
                       tdt.cpp / rnnt.cpp , TDT / RNNT greedy loops
                       streaming_encoder.hpp/cpp, cache-aware streaming FastConformer encoder
                       streaming.hpp/cpp  , pk::StreamingSession (carried RNN-T + EOU events) + run_stream_over_pcm
examples/cli/        parakeet-cli binary
                       subcommands: info, transcribe (+ --stream), quantize
scripts/             Python tooling
                       convert_parakeet_to_gguf.py, .nemo/.hf -> GGUF (--dtype f32|f16|q8_0)
                       gen_nemo_baseline.py        , NeMo intermediates -> baseline.gguf
                       gen_stream_baseline.py      , NeMo cache-aware streaming encode+decode -> stream baseline.gguf
                       validate_vs_nemo.py         , WER parity gate vs NeMo
                       publish_hf.py               , convert+quantize -> HF upload (dry-run default)
                       requirements.txt            , nemo_toolkit[asr] + gguf
tests/               ctest targets
                       test_smoke.cpp          , version string (model-independent)
                       test_audio_io.cpp       , wav load + resample (model-independent)
                       test_fft.cpp            , FFT cross-check (model-independent)
                       test_model_loader.cpp   , config + tensor map (model-dependent)
                       test_capi.cpp           , C-API load -> transcribe -> free (model-dependent)
                       test_transcribe_speech.cpp, end-to-end CTC transcript (model-dependent)
                       test_transcribe_tdt.cpp , TDT transcript on speech fixture (model-dependent)
                       test_transcribe_0_6b.cpp, regression gate for 0.6B model (model-dependent)
                       test_transcribe_ctc.cpp , standalone CTC regression (model-dependent)
                       test_transcribe_rnnt.cpp, RNNT regression (model-dependent)
                       test_transcribe_eou.cpp , offline EOU model transcript + token ids (PARAKEET_TEST_GGUF_EOU)
                       test_streaming_encoder.cpp, cache-aware streaming encoder == offline + NeMo
                       test_streaming_decode.cpp , streaming RNN-T tokens == NeMo cache-aware streaming

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mudler/parakeet.cpp](https://github.com/mudler/parakeet.cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
