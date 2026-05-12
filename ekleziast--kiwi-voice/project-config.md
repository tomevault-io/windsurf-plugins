---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Kiwi Voice is a multi-language real-time voice assistant integrated with the OpenClaw AI backend. It captures microphone audio, transcribes speech via configurable STT engines (Faster Whisper, ElevenLabs cloud, or MLX Whisper), detects the wake word "kiwi", identifies speakers via pyannote embeddings, communicates with OpenClaw over WebSocket, and speaks responses through configurable TTS providers.

All code comments and docstrings are in English. User-facing strings are externalized into YAML locale files (`kiwi/locales/*.yaml`) supporting 15 languages.

## Running the Service

```bash
# Activate the virtual environment
source venv/Scripts/activate   # Windows/MSYS2
source venv/bin/activate       # Linux

# Install dependencies
pip install -r requirements.txt

# Run the main service
python -m kiwi
```

Smoke tests: `pytest tests/test_smoke.py`

## Configuration

**Precedence:** `config.yaml` → environment variables (`.env`) → hardcoded defaults

- `config.yaml` — primary config (language, WebSocket, STT, TTS, wake word, VAD, speaker priority, security)
- `.env` — secrets and provider overrides (see `.env.example` for available vars)
- Key env vars: `KIWI_LANGUAGE` (ru | en | es | de | fr | ...), `KIWI_TTS_PROVIDER` (qwen3 | piper | elevenlabs), `KIWI_QWEN_BACKEND` (runpod | local), `KIWI_STT_ENGINE` (faster-whisper | mlx-whisper | elevenlabs)

## Architecture

### Audio Pipeline

```
Microphone (24kHz) → Audio Callback (energy + Silero VAD) → Audio Queue
  → KiwiListener._record_loop() → STT (Faster Whisper | ElevenLabs | MLX Whisper) → Wake Word Detection ("kiwi")
  → Speaker ID (pyannote embedding) → Priority Check (OWNER > FRIEND > GUEST > BLOCKED)
  → Voice Security (Telegram approval for dangerous commands from non-OWNER)
  → OpenClaw WebSocket (ws://127.0.0.1:18789, Protocol v3: chat.send → delta/final events)
  → TTS Provider → Speaker Output (with barge-in detection)
  → Loop back to listening
```

### Speaker Priority System

```python
class VoicePriority(IntEnum):
    SELF = -1      # TTS echo filtering
    OWNER = 0      # Full access, cannot be blocked
    FRIEND = 1     # Dangerous commands need Telegram approval
    GUEST = 2      # All potentially dangerous commands need approval
    BLOCKED = 99   # Blacklisted
```

## Internationalization (i18n)

All user-facing strings are externalized into YAML locale files in `kiwi/locales/`. The system is managed by `kiwi/i18n.py`.

### How it works

```python
from kiwi.i18n import setup, t

setup("en", fallback="ru")       # Initialize with locale
t("responses.greeting")           # → "Hello! I'm Kiwi..."
t("responses.heard", command=cmd) # → "Heard: {command}" with placeholder
t("hallucinations.phrases")       # → returns a list
t("wake_word.typos")              # → returns a dict
```

- `setup(locale, fallback)` is called in `service.py` at startup
- `t(key)` resolves dot-notation keys, falls back to fallback locale, then returns the raw key
- Supports `{placeholder}` formatting via kwargs for string values
- Returns lists/dicts as-is for non-string values

### Language switching

```yaml
# config.yaml
language: "en"    # or: ru, es, pt, fr, it, de, tr, pl, zh, ja, ko, hi, ar, id
```

Or via env var: `KIWI_LANGUAGE=en`

### Locale file structure

Each locale YAML has 19 sections:
- `system` — voice system prompt for the LLM
- `responses` — spoken user-facing strings (~37 keys)
- `status` — long-task status announcements
- `security` — warning messages, Telegram buttons
- `speakers` — speaker identification responses
- `speaker_access` — access control messages
- `wake_word` — keyword, typos, fuzzy blacklist (per-language)
- `hallucinations` — Whisper hallucination phrases/patterns (per-language)
- `text_processing` — abbreviations, incomplete/complete patterns, emotion keywords (per-language)
- `security_patterns` — dangerous command regexes (per-language + universal)
- `dangerous_commands` — example dangerous commands
- `owner_commands` — voice control command phrases
- `owner_control_patterns` — owner command regexes
- `name_patterns` — name extraction regexes and filter words
- `commands` — command keywords (stop, calibrate, approval, etc.)
- `cli_errors`, `ws_errors` — error messages
- `tool_activity`, `tool_errors` — tool status descriptions

### Adding a new language

1. Copy `kiwi/locales/en.yaml` → `kiwi/locales/{code}.yaml`
2. Translate all values (preserve keys and `{placeholders}`)
3. Adapt language-specific sections: `wake_word.typos`, `hallucinations`, `text_processing`, `security_patterns`, `commands`
4. Set `language: "{code}"` in `config.yaml`

### Code pattern for i18n strings

```python
from kiwi.i18n import t

# User-facing strings — always use t()
self._speak(t("responses.greeting"))
self._speak(t("responses.heard", command=cmd))

# Developer-facing log messages — do NOT use t()
kiwi_log("TAG", "Internal log message", level="INFO")
```

Module-level constants (e.g. `WAKE_WORD`, `HALLUCINATION_PHRASES`) are kept as fallback defaults. Instance attributes override them from i18n at init time:
```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ekleziast/kiwi-voice](https://github.com/ekleziast/kiwi-voice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
