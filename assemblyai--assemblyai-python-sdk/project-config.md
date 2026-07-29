---
trigger: always_on
description: Speech-to-text and audio intelligence SDK. Supports pre-recorded transcription, real-time streaming, and audio analysis features.
---

# AssemblyAI Python SDK

Speech-to-text and audio intelligence SDK. Supports pre-recorded transcription, real-time streaming, and audio analysis features.

## Quick start

```bash
pip install -U assemblyai
```

```python
import os
import assemblyai as aai

aai.settings.api_key = os.environ["ASSEMBLYAI_API_KEY"]

transcript = aai.Transcriber().transcribe(
    "https://example.com/audio.mp3",
    config=aai.TranscriptionConfig(
        speech_models=["universal-3-5-pro", "universal-2"],
        speaker_labels=True,
    ),
)

print(transcript.text)
for utterance in transcript.utterances:
    print(f"Speaker {utterance.speaker}: {utterance.text}")
```

## Auth

Set `ASSEMBLYAI_API_KEY` as an environment variable, or:

```python
aai.settings.api_key = "your-key"
```

## Key classes

- `aai.Transcriber` — Transcribe files, URLs, or streams. Methods: `transcribe()`, `transcribe_async()`, `submit()`, `list_transcripts()`
- `aai.TranscriptionConfig` — All transcription options: `speech_models`, `speaker_labels`, `sentiment_analysis`, `entity_detection`, `auto_chapters`, `content_safety`, `language_detection`, `summarization`, `word_boost`, `disfluencies`
- `aai.Transcript` — Result object with `.text`, `.status`, `.utterances`, `.words`, `.chapters`, `.entities`, `.sentiment_analysis`. Methods: `get_sentences()`, `get_paragraphs()`, `export_subtitles_srt()`, `export_subtitles_vtt()`
- `aai.SyncTranscriber` — Synchronous pre-recorded transcription: audio in, transcript out, one request (no polling). Methods: `transcribe()`, `transcribe_async()`
- `aai.SyncTranscriptionConfig` — Sync options: `model` (default `universal-3-5-pro`), `prompt`, `keyterms_prompt`, `conversation_context`, `language_codes`, `timestamps`, `sample_rate`, `channels`
- `aai.SyncTranscriptResponse` — Sync result: `.text`, `.words` (`SyncWord` with `confidence` always, `start`/`end` only when `timestamps=True`), `.confidence`, `.audio_duration_ms`, `.session_id`, `.request_time_ms`
- `assemblyai.streaming.v3.StreamingClient` — Real-time streaming with event-based API (threaded)
- `assemblyai.streaming.v3.AsyncStreamingClient` — Asyncio-native counterpart; same options/events

## Common patterns

**Transcribe a local file:**
```python
transcript = aai.Transcriber().transcribe("./recording.mp3")
```

**With multiple features:**
```python
config = aai.TranscriptionConfig(
    speech_models=["universal-3-5-pro", "universal-2"],
    speaker_labels=True,
    sentiment_analysis=True,
    entity_detection=True,
    auto_chapters=True,
    language_detection=True,
)
transcript = aai.Transcriber().transcribe(audio_url, config=config)
```

**PII redaction** (uses setter, not constructor):
```python
config = aai.TranscriptionConfig()
config.set_redact_pii(
    policies=[aai.PIIRedactionPolicy.email_address, aai.PIIRedactionPolicy.phone_number],
    substitution=aai.PIISubstitutionPolicy.hash,
)
```

**Retrieve existing transcript:**
```python
transcript = aai.Transcript.get_by_id("transcript-id")
```

## Sync transcription (pre-recorded, single request)

`SyncTranscriber` posts a whole audio file and returns the finished transcript in one
round trip — no job id, no polling, no status enum. It targets the sync API
(`sync.assemblyai.com`), distinct from `Transcriber`'s async job API. Use it for short
clips where you want the answer inline; use `Transcriber` for long-form audio, URLs, or
the rich audio-intelligence features (speaker labels, chapters, sentiment, …) the sync
API doesn't expose.

```python
import assemblyai as aai

aai.settings.api_key = os.environ["ASSEMBLYAI_API_KEY"]

result = aai.SyncTranscriber().transcribe("./call.wav")
print(result.text, result.session_id)
for w in result.words:
    print(w.text, w.confidence)  # w.start/w.end need timestamps=True (see below)
```

**Input**: a local file path, raw `bytes`, or a binary file object. **Not** a URL —
pass a path/bytes or use `Transcriber` for URL ingestion.

**Config** (all optional):
```python
config = aai.SyncTranscriptionConfig(
    prompt="Transcribe verbatim. Preserve disfluencies.",  # max 4096 chars
    keyterms_prompt=["AssemblyAI", "Lemur", "U3-Pro"],     # max 2048 chars total
    language_codes=["es"],                                 # or e.g. ["en", "es"] for multilingual; defaults to English
)
result = aai.SyncTranscriber().transcribe("./call.wav", config=config)
```

**Conversation context**: `conversation_context` carries prior turns from the same
conversation so the model keeps continuity and proper-noun spelling across a multi-turn
exchange. List them oldest-first (most recent last); a single prior turn can be a bare
string. Capped at 100 turns / 4096 chars total — over-cap context is trimmed (oldest
turns first), not rejected; oldest turns are likewise dropped first when over the
model token budget.
```python
config = aai.SyncTranscriptionConfig(
    conversation_context=[
        "I'd like to book a flight to Denver.",
        "Sure, what date were you thinking?",
    ],
)
result = aai.SyncTranscriber().transcribe("./reply.wav", config=config)
```

**Language**: `language_codes` takes a list of ISO 639-1 codes — a single-element list

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AssemblyAI/assemblyai-python-sdk](https://github.com/AssemblyAI/assemblyai-python-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
