---
trigger: always_on
description: End-to-end video localization. Transcribe spoken audio in any language Whisper supports (Spanish, English, Portuguese, French, Italian, Japanese, Korean, etc.), translate into a chosen target language (Simplified Chinese and English are first-class; other targets work via the same pipeline if a TTS voice is available), generate punctuation-bounded SRT subtitles, optionally burn them into the video, and optionally produce a time-aligned voice dub. Defaults to single-speaker — uses one voice for t
---


# translate-video

## Purpose

End-to-end video localization pipeline. Given a video with spoken
audio in **any language Whisper recognizes**, this skill produces:

1. A timestamped transcript SRT in the source language.
2. A translated SRT in the user's chosen target language, segmented
   at punctuation boundaries (no mid-sentence breaks).
3. Optional: hard-burned subtitles in the chosen target language.
4. Optional: a time-aligned TTS voice dub in the target language,
   with the original audio optionally preserved as a low-volume bed.

Outputs are SRT and MP4 — usable directly in Final Cut Pro,
Premiere Pro, CapCut, DaVinci Resolve, or `ffmpeg`.

### Source language

Pass `--language es` (or `en`, `pt`, `fr`, `it`, `ja`, `ko`, etc.)
to whisper to lock detection. Auto-detect can mis-route on short or
heavily accented clips, so always pin the source explicitly when
known.

### Target language

This skill is fully validated for two targets:

- **Simplified Chinese (zh-CN)** — Volcano (豆包) TTS for dub,
  Chinese-specific subtitle line conventions.
- **English (en)** — edge-tts multilingual neural voices for dub,
  English subtitle line conventions.

Other targets (Japanese, Korean, French, etc.) work mechanically via
the same pipeline; the bottleneck is finding a good TTS voice — the
edge-tts catalog covers most major languages, but cap-test before
promising.

Picking from user phrasing:

- "翻成中文 / 中文字幕 / 中文配音" → `zh-CN`.
- "translate to English / English subs / English dub" → `en`.
- "bilingual" → produce both `.zh-CN.srt` and `.en.srt`; for dubs
  ask which one to render (or render both).
- Ambiguous → default to whichever the user has historically chosen
  in the project; otherwise ask once.

The canonical worked example throughout this doc is **Spanish → Chinese**
because that was the original validation scenario, but every step
applies to other source/target pairs unchanged.

### Number of speakers — default to one

**Default: assume one speaker.** Use a single voice for the entire
dub. This is the right answer for monologues, vlogs, recorded talks,
narrator-only clips, and the overwhelming majority of videos people
ask about. Don't run diarization, don't tag the SRT with `[A]`/`[B]`,
don't bring up multi-speaker complexity.

**Switch to multi-speaker only when the user explicitly says so** —
phrasings like "two people", "interview", "dialogue", "conversation
between", "separate the speakers", "different voice for each", or a
direct request to do diarization. When triggered, follow the
"Multi-speaker dubbing (advanced, opt-in)" section near the end of
this doc.

If you're unsure whether a video is one speaker or many, ship the
single-voice version first. Adding speaker separation later is
cheap (just regenerate the dub); shipping confused multi-speaker
output by default wastes the user's time.

---

## When to Use

Use this skill when the user asks to:

- Transcribe spoken audio from a video in any source language
- Translate video speech into Chinese or English (or other languages
  with available TTS)
- Add subtitles to a video (soft-muxed or hardcoded/burned-in)
- Generate `.srt` from audio or video
- Translate an existing subtitle file into another language
- Create bilingual or trilingual subtitles
- Fix, polish, or re-time translated subtitles
- Produce a voice dub of a foreign-language video, optionally with
  the original audio kept as a low-volume bed underneath

---

## Input Types

The user may provide:

- A video file: `.mp4`, `.mov`, `.mkv`, `.avi`
- An audio file: `.mp3`, `.wav`, `.m4a`, `.aac`
- A subtitle file: `.srt`, `.vtt`, `.ass`
- A Spanish transcript pasted into the chat
- A rough subtitle draft that needs translation or repair

---

## Default Workflow

### Step 1: Inspect the Source

Determine what the user provided:

- If the user provides a video or audio file, transcribe the Spanish speech.
- If the user provides an existing Spanish subtitle file, preserve its timestamps and translate line by line.
- If the user provides only transcript text, translate it and create subtitles only if timing information is available.
- If no timing information exists, do not invent exact timestamps unless the user explicitly asks for approximate timing.

---

### Step 2: Transcribe the Source Audio

When transcribing (any source language):

- Preserve the original meaning.
- Keep sentence boundaries clear.
- Do not hallucinate unclear words.
- Mark unclear audio as `[inaudible]` only when necessary.
- Preserve names, places, brands, dates, and numbers.
- If multiple speakers are obvious, label them only when useful.
- Keep the transcript aligned with timestamps where possible.

Preferred working format:

```text
[00:00:01.200 --> 00:00:04.800] Spanish transcript here.
[00:00:04.800 --> 00:00:08.500] Spanish transcript here.
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jianshuo/translate-video](https://github.com/jianshuo/translate-video) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
