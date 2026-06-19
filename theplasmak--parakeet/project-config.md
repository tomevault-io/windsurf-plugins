---
trigger: always_on
description: Local speech-to-text using NVIDIA Parakeet TDT (NeMo). 600M-param multilingual ASR with automatic punctuation/capitalization, word-level timestamps, and ~3380x realtime speed on GPU. Supports 25 European languages with auto-detection, long-form audio up to 3 hours, streaming output, SRT/VTT subtitles, batch processing, and URL/YouTube input.
---


# 🦜 Parakeet — NVIDIA Speech-to-Text

Local speech-to-text using NVIDIA's Parakeet TDT models via the NeMo toolkit. The default `parakeet-tdt-0.6b-v3` is a 600-million-parameter multilingual ASR model that delivers **state-of-the-art accuracy** with **automatic punctuation and capitalization**, word-level timestamps, and an insane **~3380× realtime** inference speed on GPU.

Only needs **~2GB VRAM** to run — plenty of room on an RTX 3070 (8GB).

## When to Use

Use this skill when you need:

- **Best-accuracy transcription** — Parakeet v3 achieves 6.34% average WER on the Open ASR Leaderboard
- **Automatic punctuation & capitalization** — no post-processing needed, output is clean and readable
- **European language transcription** — 25 languages with automatic detection (no prompting required)
- **Word/segment/char timestamps** — precise timing at every granularity level
- **Long audio files** — up to 24 min with full attention, up to 3 hours with `--long-form` local attention
- **Streaming output** — see segments as they're transcribed with `--streaming`
- **Subtitle generation** — SRT and VTT formats with word-level line splitting
- **Batch processing** — glob patterns, directories, skip-existing support with ETA
- **URL/YouTube transcription** — auto-downloads via yt-dlp
- **Blazing speed** — ~3380× realtime means a 10-minute file transcribes in under 0.2 seconds on GPU

**Trigger phrases:**
"transcribe with parakeet", "nemo transcribe", "nvidia speech to text", "parakeet transcribe",
"transcribe this audio", "convert speech to text", "what did they say", "make a transcript",
"audio to text", "subtitle this video", "transcribe in [language]", "European language transcription",
"best accuracy transcription", "high accuracy speech to text", "transcribe long audio",
"transcribe lecture", "transcribe meeting", "word timestamps"

### 🦜 Parakeet vs 🗣️ faster-whisper — When to Use Which

| Feature | Parakeet | faster-whisper |
|---|---|---|
| **Accuracy** | ✅ Best (6.34% avg WER) | Good (distil: 7.08% WER) |
| **Speed** | ✅ ~3380× realtime | ~20× realtime |
| **Auto punctuation** | ✅ Built-in | ❌ Requires post-processing |
| **Languages** | 25 European | ✅ 99+ worldwide |
| **Diarization** | ❌ Not supported | ✅ pyannote speaker ID |
| **Chapters/search** | ❌ Not supported | ✅ Chapter detection, search |
| **Output formats** | text, JSON, SRT, VTT | ✅ text, JSON, SRT, VTT, ASS, LRC, TTML, CSV, TSV, HTML |
| **Translation** | ❌ Not supported | ✅ Any language → English |
| **VRAM usage** | ~2GB | ~1.5GB (distil) |
| **Long audio** | ✅ Up to 3 hours | Limited by VRAM |
| **Streaming** | ✅ Chunked inference | ✅ Segment streaming |
| **Noise handling** | Good (built-in robustness) | ✅ --denoise, --normalize |
| **Filler removal** | ❌ | ✅ --clean-filler |

**Rule of thumb:**
- **Parakeet** when you want: best accuracy, fastest speed, auto-punctuation, European languages
- **faster-whisper** when you need: diarization, 99+ languages, translation, chapters/search, more output formats, audio preprocessing

**⚠️ Agent guidance — keep invocations minimal:**

_CORE RULE: default command (`./scripts/transcribe audio.wav`) is the fastest path — add flags only when the user explicitly asks for that capability._

**Transcription:**

- Only add `--timestamps` if the user asks for word-level or segment-level timestamps (auto-enabled for srt, vtt, json formats)
- Only add `--format srt/vtt` if the user asks for subtitles/captions in that format
- Only add `--format json` if the user wants structured/programmatic output
- Only add `--long-form` if the audio is longer than 24 minutes
- Only add `--streaming` if the user wants live/progressive output for long files
- Only add `-l/--language CODE` if the user specifies a language (auto-detection is usually fine)
- Only add `--model` if the user wants a specific model variant (v2, 1.1b)
- Only add `--device cpu` if GPU is not available or user requests CPU
- Only add `--batch-size N` if the user reports OOM errors
- Only add `--max-words-per-line` or `--max-chars-per-line` for subtitle readability on long segments

**Batch processing:**

- Only add `--skip-existing` when resuming interrupted batch jobs
- For batch output, always use `-o <dir>` (directory, not file)
- ETA is shown automatically for batch jobs (no flag needed)

**Output format for agent relay:**

- **Text** (default) → safe to show directly to user for short files; summarise for long ones
- **JSON** → useful for programmatic post-processing; not ideal to paste in full to user
- **SRT/VTT** → always write to `-o` file; tell the user the output path, never paste raw subtitle content

**When NOT to use:**

- Need speaker diarization → use faster-whisper with `--diarize`
- Non-European languages (Asian, African, etc.) → use faster-whisper
- Need translation to English → use faster-whisper with `--translate`
- Cloud-only environments without local compute

## Quick Reference

| Task | Command | Notes |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ThePlasmak/parakeet](https://github.com/ThePlasmak/parakeet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
