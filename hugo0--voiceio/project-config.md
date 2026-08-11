---
trigger: always_on
description: git clone https://github.com/Hugo0/voiceio.git
---

# Contributing to voiceio

## Quick start

```bash
git clone https://github.com/Hugo0/voiceio.git
cd voiceio
uv pip install -e ".[linux,dev]"
uv run voiceio setup   # bootstraps CLI symlinks into ~/.local/bin/
uv run pytest tests/ -x -q
```

## Architecture

```
voiceio/
├── app.py           # State machine (IDLE/RECORDING/FINALIZING/ERROR), health watchdog
├── cli.py           # CLI: setup, doctor, test, toggle, correct, history, service, logs
├── config.py        # Config schema + TOML loading
├── platform.py      # OS/DE detection, distro-aware pkg_install(), open_in_terminal()
├── recorder.py      # Audio capture with 1s pre-buffer ring
├── transcriber.py   # Whisper subprocess with crash recovery
├── streaming.py     # Real-time transcription with word-level corrections
├── postprocess.py   # Rule-based cleanup + shared post-processing pipeline
├── numbers.py       # Spoken number → digit conversion ("twenty five" → "25")
├── commands.py      # Voice commands: "new line", "scratch that", "correct that"
├── corrections.py   # Corrections dictionary (auto-replace misheard words)
├── autocorrect.py   # Frequency analysis + Levenshtein clustering + LLM review
├── autocorrect_state.py # Persistent mining state: scan cursor + deferred/dismissed terms
├── audit.py         # Teacher-model audit: retires bad rules, ages vocab, drift rollback
├── snapshots.py     # Corrections/vocabulary snapshots for audit rollback
├── postcorrect.py   # Constrained final-pass LLM rewrite of misheard words (cloud)
├── retention.py     # Local per-utterance audio + context storage for mining/audit
├── wordfreq.py      # Word frequency lookup via wordfreq package
├── llm.py           # Optional LLM post-processing via Ollama
├── llm_api.py       # OpenAI-compatible chat completions client (OpenRouter/OpenAI/etc.)
├── hints.py         # Contextual CLI hints (silenceable, frequency-limited)
├── vad.py           # Voice Activity Detection (Silero neural net / RMS fallback)
├── vocabulary.py    # Custom vocabulary for Whisper conditioning
├── history.py       # Transcription history (JSONL log)
├── clipboard_read.py # Read/write text from/to system clipboard / primary selection
├── demo.py          # Interactive guided tour (voiceio demo)
├── health.py        # Diagnostic probes for all backends + features
├── feedback.py      # Sound playback (persistent sounddevice stream) + notifications
├── service.py       # Systemd service + CLI symlink management
├── wizard.py        # Interactive setup wizard with Ollama integration
├── worker.py        # Whisper worker subprocess
├── hotkeys/         # evdev, pynput, Unix socket backends + chain resolution
├── typers/          # ibus, ydotool, wtype, xdotool, clipboard, pynput + chain
├── ibus/            # IBus engine process (GLib main loop + socket listener)
├── tray/            # Animated tray icon (AppIndicator3 / pystray fallback)
├── tts/             # Text-to-speech engines (piper, espeak, edge-tts) + chain
└── sounds/          # WAV audio cues
```

## Key patterns

**State machine** — `app.py` uses `_State` enum protected by `_hotkey_lock`. Generation counter prevents stale finalizers from stomping newer recordings.

**Chain & probe** — Every backend implements `probe() → ProbeResult`. `chain.select()` picks the first working one. Runtime failures trigger automatic re-probe and fallback.

**Adding a backend** — Create `voiceio/typers/my_backend.py`, implement `TyperBackend` (or `StreamingTyper` for preedit), register in `__init__.py`, add to `chain.py`, add probe test.

**Hotkey deduplication** — evdev and socket both fire for the same keypress. `on_hotkey()` uses lock + 0.3s debounce.

**Streaming** — IBus path uses preedit (underlined preview) + commit. Fallback path uses word-level append with char-level diff on final.

**Incremental finalization** — during recording, once the un-frozen audio tail exceeds `output.streaming_freeze_secs` (default 15s), it is cut at the quietest interior speech pause (searched relative to the tail's own RMS — mic-gain independent), beam-decoded and frozen; interim and stop-time passes only decode the remaining tail. Long dictations finalize in O(tail) instead of O(recording) (~10x faster stop for multi-minute notes). Frozen text conditions the tail decode via per-call `context` prompt. Freeze passes never touch the display; decode failures never advance the boundary.

**Post-processing pipeline** — `postprocess.apply_pipeline()` is the single shared pipeline: cleanup → numbers → commands → corrections → LLM (final only). Used by both streaming and batch modes.

**LLM integration** — Optional Ollama-based grammar/spelling cleanup. Runs on final pass only (never during streaming). `llm.py` has shared helpers (install, start, pull, diagnose) used by wizard, doctor, and app.

**Tray icon** — Pre-rendered PNG frames in freedesktop icon theme. AppIndicator3 subprocess under system Python (avoids GTK/venv conflicts). Phase-matched transitions between states. App works fine without it.

## Code style

- `ruff check voiceio/` (runs in CI)
- Python 3.11+, type hints on public APIs
- [Conventional Commits](https://www.conventionalcommits.org/) (feat/fix/refactor/docs/test/ci/chore)
- DRY: reuse existing utilities and patterns before writing new code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hugo0/voiceio](https://github.com/Hugo0/voiceio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
