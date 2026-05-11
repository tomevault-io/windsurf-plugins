---
trigger: always_on
description: Python CLI/GUI for audio recording + transcription via APIs (Mistral Voxtral). MVP: manual stop, API-based, zero-footprint defaults (temp files, no persistent dirs unless overridden), results in `transcripts/` when persisted.
---

# SuperVoxtral — Agent Guide

## Project overview
Python CLI/GUI for audio recording + transcription via APIs (Mistral Voxtral). MVP: manual stop, API-based, zero-footprint defaults (temp files, no persistent dirs unless overridden), results in `transcripts/` when persisted.

### Core Design Principles

1. **Centralized Pipeline**: All recording/transcription flows through `RecordingPipeline` (svx/core/pipeline.py) for consistency between CLI and GUI
2. **Config-driven**: Structured `Config` dataclass (svx/core/config.py) loaded from user's config.toml; CLI args override specific values
3. **Zero-footprint defaults**: Temp files auto-deleted unless `keep_*` flags or `--save-all` enabled; no project directories created by default. Long recordings (> chunk_duration) auto-activate save_all for data protection.
4. **Provider abstraction**: `Provider` protocol (svx/providers/base.py) for pluggable transcription services
5. **User-standard paths**: Data files (recordings, transcripts, logs) stored in platform-standard data directories (`USER_DATA_DIR`), not cwd. Config in `USER_CONFIG_DIR`.

### Module Structure

- **svx/cli.py**: Typer CLI entrypoint; orchestration only, delegates to Config and Pipeline. Two main commands: `record` (mic recording) and `process` (existing audio/video file). During recording, runs the pipeline in a background thread while the main thread drives a Rich `Live` animated panel (level meters + elapsed time + config info). The root logger's StreamHandler is temporarily replaced with a `RichHandler` tied to the same `Console` instance to prevent cursor-tracking desync.
- **svx/core/**:
  - `config.py`: Config dataclasses, TOML loading, prompt resolution (supports multiple prompts via [prompt.key] sections), logging setup. `get_user_data_dir()` / `get_user_config_dir()` for platform-standard paths. `keep_raw_audio` / `keep_compressed_audio` control WAV and compressed file retention independently.
  - `pipeline.py`: RecordingPipeline class - records (single or dual device), auto-chunks long recordings, transcribes with diarization, saves conditionally, copies to clipboard. Accepts an optional `level_monitor` (AudioLevelMonitor) and calls `push_mic`/`push_loop` from its recording callbacks.
  - `audio.py`: WAV recording (sounddevice), ffmpeg detection/conversion to MP3/Opus, audio duration extraction (`get_audio_duration`)
  - `chunking.py`: Split long WAV files into overlapping chunks (`split_wav`), merge transcription segments (`merge_segments`) with crossfade deduplication, merge texts (`merge_texts`). Chunk transcription runs in parallel via `ThreadPoolExecutor`.
  - `meeting_audio.py`: Dual-device recording (`record_dual_wav`) — mic + loopback mixed to mono with configurable per-source gain. `find_loopback_device()` for device discovery.
  - `formatting.py`: Format diarized transcription segments with speaker labels and timestamps (`format_diarized_transcript`)
  - `level_monitor.py`: `AudioLevelMonitor` — framework-agnostic, push-based peak accumulator (no sounddevice streams). Pipeline feeds RMS values via `push_mic()`/`push_loop()` from its recording callbacks; consumers call `get_and_reset_peaks()` at their own cadence. Shared between CLI and GUI.
  - `prompt.py`: Multi-prompt resolution from config dict (key-based: "default", "test", etc.)
  - `storage.py`: Save transcripts/JSON conditionally based on keep_transcript_files
  - `clipboard.py`: Cross-platform clipboard copy
- **svx/providers/**:
  - `base.py`: Provider protocol, TranscriptionResult/TranscriptionSegment TypedDicts, ProviderError
  - `mistral.py`: Mistral Voxtral implementation (dedicated transcription endpoint + text-based LLM chat, diarization support)
  - `openai.py`: OpenAI Whisper implementation
  - `__init__.py`: Provider registry (get_provider)
- **svx/ui/**:
  - `tk_app.py`: Pure-stdlib tkinter GUI (RecorderWindow/RecorderWorker) using Pipeline; dynamic buttons per prompt key; persistent checkboxes for `keep_raw_audio` / `keep_compressed_audio` via JSON settings file (override TOML without editing it). `AudioLevelMonitor` adapter polls `_CoreMonitor` (push mode) via `root.after()` at 20 Hz. Single-row segmented LED-style level meters (MIC always, LOOP when loopback configured).

### Execution Flow

1. **Entry**: CLI parses args. Two commands: `record` (--prompt, --save-all, --gui, --transcribe) and `process <file>` (same options minus --gui)
2. **Config Load**: Config.load() reads config.toml (supports [prompt.default], [prompt.other], etc.); `chat_model` for text LLM; API keys in [providers.mistral] or [providers.openai]
3. **Context Bias**: Optional `context_bias` list in `[defaults]` (up to 100 items) — passed to Mistral's transcription endpoint to improve recognition of specific vocabulary (proper nouns, technical terms). Stored in `DefaultsConfig`, read by `MistralProvider.__init__`.
4. **Prompt Resolution**:
   - CLI: Uses "default" prompt key unless --prompt/--prompt-file overrides
   - GUI: Dynamic buttons for each [prompt.key]; "Transcribe" button bypasses prompt

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vlebert/supervoxtral](https://github.com/vlebert/supervoxtral) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
