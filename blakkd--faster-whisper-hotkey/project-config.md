---
trigger: always_on
description: **faster-whisper-hotkey** is a minimalist push-to-talk transcription tool for Linux that leverages cutting-edge ASR models. Hold a hotkey, speak, release → text appears instantly in your focused field.
---

## 📋 Project Overview

**faster-whisper-hotkey** is a minimalist push-to-talk transcription tool for Linux that leverages cutting-edge ASR models. Hold a hotkey, speak, release → text appears instantly in your focused field.

### Key Value Proposition
- **Speed**: Near-instant transcription even on CPU with smaller models
- **Flexibility**: Multiple model backends (faster-whisper, parakeet, canary, voxtral, cohere)
- **Integration**: Works in terminals, editors, chat apps anywhere you can type

---

## 🏗 Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     User Interaction Layer                  │
│  (hotkey → audio capture → transcription → paste)           │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                      Core Application                       │
│  transcribe.py ──→ settings.py ──→ ui.py (curses menu)      │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    Transcription Pipeline                   │
│  transcriber.py ──→ models.py ──→ model backends            │
│  (recording)     (model wrapper) (whisper/parakeet/etc)     │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                     Output Delivery                         │
│  clipboard.py ──→ paste.py ──→ terminal.py                  │
│  (backup/restore) (X11/Wayland) (window detection)          │
└─────────────────────────────────────────────────────────────┘
```

---

## 📁 Key Files & Responsibilities

| File | Purpose | Owner to Contact |
|------|---------|------------------|
| `__main__.py` | CLI entry point, debug flag setup | Anyone |
| `transcribe.py` | Main app logic, model config flow | Core team |
| `models.py` | Model abstraction layer, ASR backends | Core team |
| `transcriber.py` | Recording, hotkey handling, text delivery | Core team |
| `settings.py` | Settings persistence (`~/.config/faster_whisper_hotkey/`) | Anyone |
| `ui.py` | Curses-based menus (model selection, config) | UI specialists |
| `terminal.py` | X11/Wayland window detection for paste targeting | Platform experts |
| `paste.py` | Clipboard & keyboard paste logic | Platform experts |
| `clipboard.py` | Clipboard backup/restore utilities | Anyone |
| `llm_corrector.py` | Optional LLM-based text correction | Core team |
| `config.py` | Loads model/language config from JSON | Anyone |

---

## 🧠 Model Backends (CRITICAL KNOWLEDGE)

### Supported Models & Their Constraints

| Model | Type | Device | Key Notes |
|-------|------|--------|-----------|
| **cohere-transcribe-03-2026** | cohere | CPU/GPU | 15 languages, NO auto-detection, smart about hesitations |
| **parakeet-tdt-0.6b-v3** | parakeet | CPU/GPU | 25 langs, auto-detection, multilingual recording supported |
| **canary-1b-v2** | canary | CPU (F16) | 25 langs, transcription + translation possible |
| **Voxtral-Mini-3B-2507** | voxtral | GPU only | English + 7 langs, smart formatting, max 30s chunks |
| **faster-whisper** | whisper | CPU/GPU | Many langs, translation when source ≠ target |

### Important: Model Loading in `models.py`

Each model type has specific initialization logic. When adding a new backend:

1. Add to `_load_model()` method with proper device handling
2. Implement `transcribe()` with chunking if needed (see Voxtral's 30s limit)
3. Handle temporary file creation for models requiring it (canary, voxtral)
4. Add to config in `available_languages.json`

**⚠️ Critical**: The model wrapper uses `suppress_output()` context manager to hide OneLogger/NeMo initialization spam at import time. Keep this!

---

## 🔧 Configuration Flow

### Settings File Location
```
~/.config/faster_whisper_hotkey/transcriber_settings.json
```

### Setting Fields (Settings dataclass)
- `device_name`: PulseAudio input device name
- `model_type`: whisper/parakeet/canary/voxtral/cohere
- `model_name`: Hugging Face model identifier
- `compute_type`: int8/float16/int4 (model-dependent)
- `device`: cpu/cuda
- `language`: Language code or "auto"
- `hotkey`: pause/f4/f8/insert
- `llm_correction_enabled`: Boolean for LLM cleanup
- `llm_endpoint`: OpenAI-compatible endpoint URL
- `llm_model_name`: Model to use for correction

### Debug Logging
Set environment variable to enable detailed logging:
```bash
export FASTER_WHISPER_HOTKEY_DEBUG=1
# or run with --debug flag
faster-whisper-hotkey --debug
```

---

## 🚨 Known Limitations & Workarounds

| Issue | Status | Workaround |
|-------|--------|------------|
| Voxtral 30s audio limit | **Hardcoded** | Chunked processing in `_transcribe_single_chunk_voxtral()` |
| VSCode/VSCodium terminal paste | **Unsupported** | No workaround currently |
| Windows support | **Separate branch** | Use eutychius's feature/supportWindows branch |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blakkd/faster-whisper-hotkey](https://github.com/blakkd/faster-whisper-hotkey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
