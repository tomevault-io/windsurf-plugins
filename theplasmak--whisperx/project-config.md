---
trigger: always_on
description: Speech-to-text with word-level timestamps, speaker diarization, and forced alignment using WhisperX. Built on faster-whisper with batched inference for 70x realtime speed.
---


# WhisperX

Speech-to-text with **word-level timestamps**, **speaker diarization**, and **forced alignment** — built on faster-whisper with batched inference for up to **70x realtime** transcription speed.

WhisperX extends Whisper with three key capabilities that faster-whisper alone doesn't provide:
1. **Forced alignment** — precise word-level timestamps via phoneme ASR models (wav2vec2)
2. **Speaker diarization** — label who said what (via pyannote.audio)
3. **Batched inference** — process audio in parallel chunks for massive speedup

## When to Use

Use this skill when you need to:
- **Transcribe with word-level timing** — subtitles, captions, karaoke-style highlighting
- **Identify speakers** — meetings, interviews, podcasts, multi-speaker recordings
- **Generate subtitle files** — SRT, VTT with accurate word timestamps
- **Translate speech to English** — from any supported language
- **Batch transcribe** — efficient processing of multiple files
- **Transcribe a section** — extract just part of a long recording

**Trigger phrases:** "transcribe with speakers", "who said what", "diarize", "make subtitles", "word timestamps", "speaker identification", "meeting transcript", "karaoke subtitles"

**When NOT to use:**
- Simple transcription without speaker/timing needs → use **faster-whisper** (lighter, faster)
- Real-time/streaming transcription
- Files <10 seconds where setup overhead isn't worth it

**WhisperX vs faster-whisper:**
| Feature | faster-whisper | WhisperX |
|---------|---------------|----------|
| Basic transcription | ✅ | ✅ |
| Word timestamps | ✅ (approximate) | ✅ (precise, aligned) |
| Speaker diarization | ❌ | ✅ |
| Forced alignment | ❌ | ✅ |
| Batched inference | ❌ | ✅ |
| Word-level subtitles | ❌ | ✅ (karaoke-style) |
| Subtitle generation | Manual | Built-in (SRT/VTT/TSV) |
| Time range trimming | ❌ | ✅ (--start/--end) |
| Hotwords | ❌ | ✅ (boost specific terms) |
| Initial prompt | ❌ | ✅ (domain terms) |
| Speaker renaming | ❌ | ✅ (--speaker-names) |
| Stdin pipe | ❌ | ✅ (read from `-`) |
| Setup complexity | Simple | Requires HF token for diarization |

## Quick Reference

All commands use `./scripts/transcribe` (the skill wrapper), **not** the `whisperx` CLI directly. The wrapper applies a required PyTorch compatibility patch — see "PyTorch 2.6+ Compatibility" below.

| Task | Command | Notes |
|------|---------|-------|
| **Basic transcription** | `./scripts/transcribe audio.mp3` | Word-aligned by default |
| **With speakers** | `./scripts/transcribe audio.mp3 --diarize` | Auto-reads `~/.cache/huggingface/token` |
| **Clean speaker output** | `./scripts/transcribe audio.mp3 --diarize --merge-speakers` | Merges consecutive same-speaker segments |
| **Named speakers** | `./scripts/transcribe audio.mp3 --diarize --speaker-names "Alice,Bob"` | Replaces SPEAKER_00, SPEAKER_01 |
| **SRT subtitles** | `./scripts/transcribe audio.mp3 --srt -o subs.srt` | Ready for video players |
| **Word-level SRT** | `./scripts/transcribe audio.mp3 --srt --word-level` | Karaoke-style, one word per cue |
| **Wrapped subtitles** | `./scripts/transcribe audio.mp3 --srt --max-line-width 42` | Standard TV subtitle width |
| **VTT subtitles** | `./scripts/transcribe audio.mp3 --vtt -o subs.vtt` | Web-compatible with `<v>` speaker tags |
| **JSON output** | `./scripts/transcribe audio.mp3 --json` | Full data with word timestamps |
| **TSV output** | `./scripts/transcribe audio.mp3 --tsv` | Spreadsheet-friendly |
| **Translate to English** | `./scripts/transcribe audio.mp3 --translate` | Any language → English |
| **Fast, no alignment** | `./scripts/transcribe audio.mp3 --no-align` | Skip forced alignment |
| **Specific language** | `./scripts/transcribe audio.mp3 -l en` | Faster than auto-detect |
| **Partial transcription** | `./scripts/transcribe audio.mp3 --start 1:30 --end 5:00` | Only a section |
| **Boost terms** | `./scripts/transcribe audio.mp3 --hotwords "Kubernetes gRPC"` | Improve rare term recognition |
| **Domain accuracy** | `./scripts/transcribe audio.mp3 --initial-prompt "OpenAI, GPT-4"` | Condition the model |
| **From stdin** | `cat audio.mp3 \| ./scripts/transcribe -` | Pipe from other tools |
| **Auto-detect format** | `./scripts/transcribe audio.mp3 -o out.srt` | Format from extension |

**⚠️ Do NOT run `whisperx` CLI directly** — it will crash on PyTorch 2.6+ with pyannote models. Always use this skill's `./scripts/transcribe` wrapper.

## Model Selection

| Model | Size | Speed | Accuracy | Use Case |
|-------|------|-------|----------|----------|
| `tiny` | 39M | Fastest | Basic | Quick drafts, testing |
| `base` | 74M | Very fast | Good | General use |
| `small` | 244M | Fast | Better | Default for whisperx CLI |
| `medium` | 769M | Moderate | High | Quality transcription |
| `large-v2` | 1.5GB | Slower | Excellent | Best diarization compat |
| `large-v3` | 1.5GB | Slower | Best | Maximum accuracy |
| **`large-v3-turbo`** | 809M | Fast | Excellent | **Recommended (default)** |

**Note:** WhisperX defaults to `small` but this skill defaults to `large-v3-turbo` for the best speed/accuracy balance with GPU.

## Setup

### First-Time Setup


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ThePlasmak/whisperx](https://github.com/ThePlasmak/whisperx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
