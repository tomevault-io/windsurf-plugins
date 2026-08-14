---
trigger: always_on
description: Guidance for working in the **Prism** repository.
---

# CLAUDE.md

Guidance for working in the **Prism** repository.

## What Prism is

Prism is an open-source, cross-platform **virtual audio middleware** that sits
between a physical microphone and any application. It captures mic input,
processes it in real time (AI noise removal, voice isolation), and routes the
result to a **virtual audio cable** so apps like Discord, Zoom, OBS, and
browsers can use the processed stream as their microphone.

**Positioning:** the only open-source, Windows-first tool combining noise
removal + voice isolation in one pipeline, aimed at non-technical users
(gamers, party chat, remote workers, streamers).

**Core principles that should guide design decisions:**
- Zero config / non-technical friendly — works out of the box, no audio
  knowledge required, complexity hidden behind one on/off toggle.
- Hardware agnostic — **CPU only, no GPU required**.
- Low latency — target **< 20ms end-to-end**, **< 5% CPU at idle**.
- Windows first; Linux and macOS later.

## Current state

Early/MVP. **Phases 1–2 done; Phase 3 (voice isolation) in progress** — the
Silero VAD speech gate shipped, target-speaker extraction is next. Chain today:
mic capture →
high-pass → AI denoiser → noise gate → virtual cable routing. The denoiser is
**swappable** (`config.DENOISER`): RNNoise (light, the Windows default), GTCRN
(ultra-light neural, tiny model; the default off-Windows), or DeepFilterNet3
(stronger, heavier). The UI adds a live
**strength** slider (dry/wet) and a **noise meter** (room-noise floor + how much
is being stripped).

```
physical mic → [high-pass → AI denoiser → noise gate] → CABLE Input (VB-Audio)
```

Note the gate runs **after** RNNoise (not the PRD's gate-first order): on the
cleaned signal its threshold can sit low (-45 dBFS) and gate true silence
without clipping soft speech onsets. A gate on the raw mic had to sit above
the ~-35 dBFS noise floor (-25) and chopped quiet consonants. The gate also
has a hold time so word ends/brief gaps aren't cut.

The end-of-chain gate is swappable via `config.GATE_MODE`: **"rms"** (the
default `NoiseGate`, opens on loudness) or **"vad"** (`SileroVAD`, opens on
detected speech). The VAD gate keeps quiet speech the RMS gate would clip and
drops loud non-speech (keyboard, fan) it would pass. Missing onnxruntime/model
→ `build_gate()` falls back to the RMS gate.

Silero VAD notes ([prism/dsp/silero_vad.py](prism/dsp/silero_vad.py)): a tiny
~2.3 MB MIT ONNX detector (Phase 3's speech-detection piece). It runs at 16 kHz
on fixed 512-sample windows, so the stage **observes a downsampled copy** to
update a running speech probability while the audio passes through untouched —
the model's ~32 ms window adds **no latency to the signal** (the detection lag
is absorbed by the gate's hold). It reuses GTCRN's `_Decimator` for 48→16 kHz
and the noise-gate attack/release/hold envelope, gated on `speech_prob` instead
of RMS. Input names differ by model version (v4: input/sr/h/c; v5:
input/state/sr), so the stage **introspects the graph at load** rather than
hardcoding. Model isn't committed; fetch with `scripts/fetch_silero_vad.py`.

RNNoise notes ([prism/dsp/rnnoise_denoise.py](prism/dsp/rnnoise_denoise.py)):
binds via ctypes to the shared library bundled in the `pyrnnoise` wheel; the
package's own Python wrapper is **never imported** (broken `audiolab`/`av`
dependency chain, and unneeded for streaming). The stream runs at 48 kHz with
480-sample blocks so one block == one RNNoise frame (no rechunk latency). If
pyrnnoise is missing the pipeline degrades gracefully to Phase 1. Measured on
this machine: ~1.3 ms per 10 ms block during speech, ~0.55 ms on silence —
the gate sitting before RNNoise keeps idle CPU low.

DeepFilterNet notes ([prism/dsp/deepfilternet.py](prism/dsp/deepfilternet.py)):
the torch-free streaming DFN3 export runs via `onnxruntime` (CPU, ~13 MB wheel,
no torch). The whole DSP chain (STFT, ERB/spec features, GRU net, deep
filtering, ISTFT) lives **inside the ONNX graph** — the stage just feeds one
512-sample frame and carries the model's 12 recurrent state tensors across
calls. A FIFO rechunks the 480-sample blocks to 512 and back (one frame of
buffering); the model adds ~32 ms of its own latency (measured impulse delay =
3 frames), so a partial `mix` delays the dry path by 3 frames to stay phase-
aligned. Measured ~5.7 ms per 10.7 ms frame: real-time but ~5x RNNoise's CPU —
hence the "stronger but heavier" positioning. The 13 MB model isn't committed;
fetch it with `scripts/fetch_deepfilternet.py`. If onnxruntime or the model is
missing, `build_denoiser()` prints why and falls back to RNNoise.

GTCRN notes ([prism/dsp/gtcrn.py](prism/dsp/gtcrn.py)): a tiny ~48 K-param,
~0.5 MB ONNX model — the lightest-CPU neural option. Unlike the other two it
runs at **16 kHz with the STFT outside the model**, so the stage owns the bits
they don't: stateful 48↔16 kHz resampling (clean 3:1, anti-alias FIRs) and a
streaming STFT/overlap-add (512 fft, 256 hop, sqrt-Hann; COLA so plain OLA
reconstructs) wrapped around the model's three per-frame cache tensors. The
model is frame-synchronous (no latency of its own); the FIR group delay (~3 ms)
and the STFT fill (~30 ms) are measured once at import to size the wet FIFO

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Olaiwonismail/prism](https://github.com/Olaiwonismail/prism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
