---
trigger: always_on
description: > Local-first pipeline that turns long-form podcast videos into short
---

# PodClipper — Project Context for Claude

> Local-first pipeline that turns long-form podcast videos into short
> vertical (9:16) reels for TikTok / Reels / Shorts. Whisper transcribes,
> Claude picks reel-worthy moments, YOLO + MediaPipe lock on the active
> speaker, OpenCV/FFmpeg renders the crop, and PIL burns karaoke captions.

Repo: `github.com/LoukikNaik/PodClipper` · Live demo: `podclipper.loukik.dev`

## Read this before naming or renaming anything

`docs/ubiquitous-language.md` is the single source of truth for every named
thing in this repo (entity, state, worker, action, concept). The contract:

1. **Look it up before naming.** About to call something "the scheduled
   item card"? Check the table first — there may already be a term for it.
2. **Add before use.** Introducing a new concept? Write the row in the
   glossary first, then use the name in code.
3. **Update on rename.** Rename in code → rename in the table in the
   *same commit*.

If a name isn't in the table and you're tempted to use it, that's a signal
to add it (or pick an existing term). The glossary is alphabetized, flat,
one row per name, no sections. Keep it that way.

## Pipeline (high level)

```
Source video
  ↓  ingest             ffprobe metadata
  ↓  audio              ffmpeg → 16 kHz mono WAV
  ↓  transcribe (1st)   faster-whisper base, parallel chunks
  ↓  analyze            Claude picks reel-worthy clips (JSON list)
  │
  ↓  per clip:
  │     extract         ffmpeg cut with ±2 s pad
  │     detect          YOLOv8 person bboxes + MediaPipe face attribution
  │     transcribe (2)  faster-whisper large-v3 (cached to words.json)
  │     shot-classify   per-frame single vs wide (≥2 people)
  │     crop            shot-aware:
  │                       single  → 9:16 follow-the-speaker
  │                       stacked → two 9:8 panels, one person each
  │                       (pose-anchored, body-IoU lock, lerp transitions)
  │     subtitles       karaoke (classic) or 1-2-word pop overlay
  │                     cfg.subtitles.style picks the renderer
  │     evaluate        LLM-as-judge scorecard + publish/review/skip verdict
  │
  ↓  outputs/<timestamp>/reel_NN_<slug>.mp4
```

## Module map

| File | Role |
|---|---|
| `src/podclipper/main.py` | CLI entry point (`podclipper = "podclipper.main:main"` in pyproject.toml). Loads `.env`, parses flags, calls `run_pipeline`. |
| `src/podclipper/pipeline.py` | Orchestrator. Holds the per-clip loop and the `crop.mode` branch. |
| `src/podclipper/ingest.py` | ffprobe wrapper, returns `VideoMeta`. |
| `src/podclipper/audio.py` | Whole-video audio extraction. |
| `src/podclipper/transcribe.py` | Whisper 1st/2nd-pass + `transcribe_second_pass_cached` (JSON cache). |
| `src/podclipper/transcribe_cleanup.py` | LLM post-pass to fix Whisper mis-spellings / transliterate non-Latin. |
| `src/podclipper/analyze.py` | LLM clip selection — reads transcript, returns `Clip[]`. |
| `src/podclipper/llm/` | Provider abstraction. `claude_cli.py` (default, configurable timeout), `litellm_provider.py` (unified gateway: Anthropic, OpenAI, Gemini, Groq, Ollama, ...). |
| `src/podclipper/detect.py` | YOLOv8 + MediaPipe BlazeFace. **Two entry points:** `detect_humans_per_frame` (single primary, legacy) and `detect_humans_all_per_frame` (all persons + face flags, used by shot-aware path). |
| `src/podclipper/timeline.py` | Two responsibilities: (1) legacy `build_speaker_timeline` for the single-crop path, (2) `classify_wide_shot_frames` for the new shot-aware path. |
| `src/podclipper/crop.py` | Two renderers: legacy `smart_crop_916` (single-panel timeline-driven) and new `smart_crop_916_stacked` (shot-aware, single ↔ stacked dual-panel). |
| `src/podclipper/subtitles.py` | Two renderers: `_burn_classic` (karaoke word highlight + fading title + audio fade-out) and `_burn_pop` (1–2 huge sheared words, alternating highlight color). `burn_subtitles` dispatches by `cfg.subtitles.style` (`classic` \| `pop`). |
| `src/podclipper/evaluate.py` | LLM-as-judge scorer; writes `verdict` + numeric scores into reel sidecar. |
| `src/podclipper/diarize.py` | **No longer in the hot path.** pyannote.audio + mouth-motion linking; kept for reference / single-locked-camera future use. |
| `src/podclipper/types.py` | Shared dataclasses: `BBox`, `Word`, `Clip`, `Timeline`, etc. |
| `src/podclipper/config.py` | YAML loader → `SimpleNamespace` tree. |
| `src/podclipper/config/default.yaml` | All knobs (bundled inside the wheel via `importlib.resources`). Key: `crop.mode` = `auto` (shot-aware) or `single` (legacy). |
| `src/podclipper/config/__init__.py` | `load_config(path)` for explicit `-c` flag; `load_default_config()` for the packaged file (used when no `-c` given). |
| `src/podclipper/prompts/` | 5 system prompts (`reel_detector`, `reel_refiner`, `trailer_picks/refiner/evaluator`) loaded via `load_prompt(name)` — also bundled in the wheel. |

## The shot-aware crop path (current default)

This is the load-bearing recent work. It replaces the old timeline +
diarization-driven crop that had jitter and mis-framing in wide shots.

**Per frame:**

1. YOLO detects all persons in the frame.
2. **Shot classifier** (`classify_wide_shot_frames`) — frame is `wide` iff

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LoukikNaik/PodClipper](https://github.com/LoukikNaik/PodClipper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
