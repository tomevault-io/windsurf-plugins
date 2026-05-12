---
trigger: always_on
description: **Humla** is a personal macOS meeting-notes app inspired by Granola. You take freeform notes during a meeting; in parallel, the app records mic + system audio, transcribes the call, and produces a structured AI summary that fuses your notes with the transcript. Built for personal/small-team use, not SaaS — your data, your API keys, local SQLite, no backend.
---

# Humla — project notes

## What this app is

**Humla** is a personal macOS meeting-notes app inspired by Granola. You take freeform notes during a meeting; in parallel, the app records mic + system audio, transcribes the call, and produces a structured AI summary that fuses your notes with the transcript. Built for personal/small-team use, not SaaS — your data, your API keys, local SQLite, no backend.

The name is Norwegian for "bumblebee" (think: small, hum, personal).

## Core capabilities

- **Hybrid capture (parallel streams)** — mic input + macOS system audio recorded simultaneously via a Swift sidecar, kept as **two separate streams end-to-end** (no mixdown). Each gets its own VAD-bounded chunk WAVs, its own full.wav, its own Whisper invocations with its own `initial_prompt` trail context, and its own diarization treatment. In-person meetings produce only mic chunks (system stays silent → no chunks emitted) and the diarizer runs on the mic stream so multiple humans in the same room get distinct labels. Remote calls produce both streams: mic chunks get tagged "You" by channel attribution (no diarize needed — every mic chunk is the same person) and system chunks get diarized for remote-side speakers.
- **Two transcription providers** — pick per-note between OpenAI (Whisper / gpt-4o-transcribe / gpt-4o-mini-transcribe / gpt-4o-transcribe-diarize) or **on-device Whisper** via Metal.
- **Whisper quality preset** — `Fast` (greedy, snappy) / `Balanced` (beam=3) / `Quality` (beam=5, low no_speech threshold) for the local provider; bundles sampling strategy + confidence thresholds together so the user picks one knob.
- **Per-note transcription language** — global Settings → Language is the default for new notes; each note has its own language chip that overrides for that note.
- **Offline speaker diarization on stop** — a second Swift sidecar (`speaker-diarize`, FluidAudio CoreML) runs *after* `recording_stop`. Uses FluidAudio's `OfflineDiarizerManager` (community-1 segmentation + VBx clustering with PLDA) — the upgrade from the 3.1-based streaming `DiarizerManager` we used initially. Branches on which streams produced content: in-person mode (mic-only) diarizes `mic_full.wav` and emits `Speaker 1:` / `Speaker 2:` for the room's voices; remote/hybrid mode (both streams have content) labels every mic chunk `You:` and runs the diarizer only on `sys_full.wav` to separate remote-side speakers. Picked over streaming online ID because the streaming path drifts on long recordings (the failure mode that drove the switch was a 13-min 2-speaker call producing 9 speakers) and because community-1 counts/assigns speakers more accurately on dense single-mic captures (e.g. in-person meetings where everyone shares the same acoustic context).
- **Speaker rename + colour-coded pills** — each unique speaker gets one of four semantic colours from the design tokens (interactive blue, success green, warning gold, accent red, cycling for 5+). A chip strip above the transcript lets the user click any speaker to rename inline; rename is a regex line-anchored rewrite of the transcript text — no separate metadata table.
- **Two-source summaries** — the model gets `[Notater]` (your typed notes) and `[Transkripsjon]` (the meeting transcript) as separate inputs, with a system prompt that tells it to favour your notes for intent and the transcript for facts.
- **Per-note presets** — Meeting / 1:1 / Lecture / Interview / Brainstorm / Voice memo, each with its own summary prompt. Custom prompts also supported.
- **Custom vocabulary** — a per-user list of names, tech terms, and phrases sent as part of Whisper's `initial_prompt` to bias decoding toward those tokens.
- **Trailing transcript context** — every chunk's transcription receives the last ~150 committed words alongside the custom vocabulary as Whisper's `initial_prompt`, so decoding stays anchored to the conversation rather than treating each chunk as a cold start. Single biggest mitigation against silence-driven hallucinations and proper-noun drift across the meeting.
- **VAD-bounded chunks** — the audio-capture sidecar rotates each chunk at natural speech pauses (min 1.0 s / max 15 s / 500 ms silence trigger) instead of a fixed timer, so chunk boundaries land mid-pause rather than mid-word.
- **Reasoning-model temperature handling** — gpt-5.x and o-series models reject custom temperature; `openai::summarize` detects them via `is_reasoning_model()` and omits the parameter, while keeping `temperature=0.2` for traditional chat models.
- **Folders** — flat folder list, per-note assignment, search across titles/bodies/transcripts/folder names with auto-expand on hits.
- **Click-to-edit transcript** — styled view by default with coloured pills + plain text; clicking enters a textarea for edits. Locked while a recording is in flight to avoid clobber.

## Architecture overview

```
┌─────────────────────────────────────────────────────────────┐

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [michaelwilhelmsen/humla](https://github.com/michaelwilhelmsen/humla) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
