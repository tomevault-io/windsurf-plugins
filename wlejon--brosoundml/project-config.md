---
trigger: always_on
description: Audio-ML model inference library. brosoundml is the **expression layer** for
---

# CLAUDE.md — brosoundml

Audio-ML model inference library. brosoundml is the **expression layer** for
neural audio models — it composes `brotensor`'s audio op family into runnable
TTS / STT / neural-codec / wake-word models, the same way `brodiffusion`
composes the diffusion ops and `brolm` composes the text-model ops. One flat
namespace, `brosoundml::`.

**Status: operational.** Four model families are complete (CPU FP32; CUDA where
noted) and back the `bro.tts` / `bro.stt` / `bro.wake` JS bindings in bro:

- **Kokoro-82M** — text-to-speech (StyleTTS 2 derivative, 24 kHz).
- **Qwen3-TTS** — text-to-speech (12 Hz multi-codebook, end-to-end discrete
  token, 24 kHz). Device-neutral CPU + CUDA; CustomVoice presets, VoiceDesign
  instruct prompts, Base-variant zero-shot voice cloning.
- **Whisper** — speech-to-text (HF checkpoints, tiny → large-v3).
- **Wake-word** — BC-ResNet streaming keyword spotter + its training toolchain.

Plus an in-tree English **G2P** (`brosoundml::g2p::`) so Kokoro phonemizes with
no misaki/Python dependency. Full per-model detail is in `README.md`.

## Layout

```
include/brosoundml/
  version.h          library version constants + version_string()
  audio.h            AudioBuffer (mono FP32 PCM) + WAV read/write; CancelCheck
  modules.h          inference-only nn-modules over brotensor ops:
                     Linear, LayerNorm, Conv1d, LSTM/BiLSTM, ada_in_1d, MHA
  kokoro.h           Kokoro-82M: KokoroConfig, Voice, the Kokoro pipeline
  kokoro_modules.h   Kokoro-specific module graph (plBERT, iSTFTNet, AdaIN)
  qwen_tts.h         Qwen3-TTS: QwenTts pipeline, Talker/CodePredictor/codec
                     configs, synthesize / synthesize_clone / encode/decode
  whisper.h          Whisper: WhisperConfig + the encoder/decoder pipeline
  whisper_modules.h  Whisper-specific modules (conv stems, cross-attn decoder)
  wake.h             WakeWord streaming detector (front-end + model + policy)
  bc_resnet.h        BC-ResNet wake model: forward, streaming, train_step
  bc_resnet2d.h      2D BC-ResNet variant (freq×time) + training surface
  mel.h              shared log-mel front-end helpers
  wake_data.h        wake training-dataset binary format
  g2p/               in-tree English G2P: pos_tagger, lexicon, morphology,
                     special_cases, normalizer, phoneme_adapter, phonemizer
  detail/json.h      vendored JSON parser (kept in sync with brolm's)
src/                 one .cpp per header above; qwen_tts split across
                     qwen_tts{,_talker,_code_predictor,_generate,_codec,
                     _codec_encoder,_speaker_encoder}.cpp
tools/               CLI drivers — see "Tools" below
tests/               one test_*.cpp per model + module layer (ctest)
```

## Build

```sh
# CPU-only
cmake -S . -B build && cmake --build build --config Release
ctest --test-dir build -C Release

# CPU + CUDA — forwarded to brotensor's CUDA backend (brosoundml ships no kernels)
cmake -S . -B build -DBROTENSOR_WITH_CUDA=ON && cmake --build build --config Release
```

On Windows use the Visual Studio multi-config generator (`--config` picks the
config); on Linux/macOS use a separate build dir per config. brosoundml builds
no GPU language of its own — `BROTENSOR_WITH_CUDA` / `_WITH_METAL` only forward
the backend choice so a standalone GPU build resolves brotensor's backend.

## Dependencies

Code-side siblings, resolved by the standard multi-repo pattern (standalone
repo at `../<name>`, else a `third_party/` submodule fallback — see
`bro/docs/multi-repo-workflow.md`):

- **bromath** — header-only math.
- **brotensor** — the unified `Tensor` + the device-neutral op surface. All of
  brosoundml's compute goes through `<brotensor/ops.h>`; brosoundml writes no
  kernels. The audio op family it leans on (FFT/STFT, conv1d, vocoder/codec
  activations, codec quantization, resampling, `sample_logits`) is FP32 on all
  three backends — CPU, CUDA, Metal.
- **brolm** — tokenizers for the speech models: `brolm::whisper::Tokenizer` for
  Whisper, the Qwen BPE tokenizer for Qwen3-TTS.

Data sibling, separate from the code dependency chain — loaders take paths,
the application resolves them:

- **brosoundml-data** (`../brosoundml-data`) — trained weights and packed data
  artifacts (POS tagger `model.bin`, the packed English lexicon, Kokoro voice
  packs not shipped with upstream checkpoints, wake-word checkpoints). Local-
  only for now; eventual home is a Hugging Face dataset repo. Path-resolution
  convention used by callers/tools: caller-supplied path > `BROSOUNDML_DATA_DIR`
  env var > `../brosoundml-data`. The library itself never touches the
  filesystem beyond the paths handed to it.

## Conventions

- **One flat namespace, `brosoundml::`.** No sub-namespaces (except the in-tree
  `brosoundml::g2p::` for the English phonemizer).
- **Compute is brotensor; brosoundml is composition.** A new model is a graph
  of `brotensor` op calls plus weight loading and pre/post-processing — never a
  new kernel. If an op is genuinely missing, add it to `brotensor` (and mirror
  it across CPU/CUDA/Metal there), not here.
- **`AudioBuffer` is the waveform currency** — mono FP32 PCM nominally in
  [-1, 1], carrying its `sample_rate`. Synthesis returns one; file I/O consumes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wlejon/brosoundml](https://github.com/wlejon/brosoundml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
