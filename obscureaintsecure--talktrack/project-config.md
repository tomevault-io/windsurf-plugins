---
trigger: always_on
description: Topic-specific rules are in `.claude/rules/`. `ways-of-working.md` is always loaded; the others are loaded on demand by topic.
---

# TalkTrack - CLAUDE.md

## Project Rules

Topic-specific rules are in `.claude/rules/`. `ways-of-working.md` is always loaded; the others are loaded on demand by topic.

@./.claude/rules/ways-of-working.md

- [audio-pipeline.md](.claude/rules/audio-pipeline.md) — AudioStream callback order, mute/gain scoping, MainWindow→capture access pattern.
- [per-app-audio-capture.md](.claude/rules/per-app-audio-capture.md) — Windows 11 process-loopback COM invariants (IAgileObject, device path, ctypes arg passing, generator caching gotcha).
- [ui-patterns.md](.claude/rules/ui-patterns.md) — CollapsibleSection, left-panel conventions, DAW meter fill direction, peak-sample bar semantics, Qt QSS gotchas, Catppuccin palette.

## Project Overview

TalkTrack is a Windows desktop application that records, transcribes, and diarizes audio from calls (Teams, Zoom, etc.). It is a modern clone of Evaer for Teams with AI-powered transcription and speaker identification.

## Tech Stack

- **GUI:** PyQt6
- **Audio Capture:** sounddevice + WASAPI, comtypes (Win11 per-process capture)
- **Audio Session Enumeration:** pycaw (Windows Core Audio API)
- **Transcription:** faster-whisper (local OpenAI Whisper, no internet needed)
- **Speaker Diarization:** pyannote.audio 4.0 (requires free HuggingFace token)
- **Deep Learning:** torch, torchaudio
- **Audio Processing:** scipy, pydub, soundfile, numpy
- **Process Detection:** psutil (for known audio app enumeration)
- **NLP/Embeddings:** transformers, sentence-transformers (pyannote dependencies)
- **Windows Integration:** pywin32, comtypes

## Project Structure

```
TalkTrack/
  main.py                              # Entry point, QApplication setup
  build.py                             # Build TalkTrack.exe launcher with custom icon
  start.bat                            # Launcher (auto-installs deps, uses TalkTrack.exe if present)
  start_debug.bat                      # Debug launcher with console output
  requirements.txt                     # Dependencies
  app/
    main_window.py                     # Main window + orchestration
    audio/
      __init__.py                      # Package init
      segment_player.py               # Audio clip playback for transcript segments
    recording/
      audio_capture.py                 # AudioStream, DualAudioCapture (legacy + per-app modes)
      process_audio_capture.py         # ProcessCaptureStream, ProcessAudioCapture (Win11 per-PID)
      recorder.py                      # State machine, session management
    transcription/
      transcriber.py                   # Whisper worker + dataclasses
      diarizer.py                      # Speaker diarization (pyannote)
    ai/
      __init__.py                      # Package init
      provider.py                      # AIProvider base class
      claude_provider.py               # Claude API implementation
      openai_provider.py               # OpenAI API implementation
      grok_provider.py                 # Grok (xAI) via OpenAI-compatible API
      gemini_provider.py               # Google Gemini API implementation
      mistral_provider.py              # Mistral AI API implementation
      local_provider.py                # Local model (llama-cpp-python)
      provider_factory.py              # Factory for configured provider
      summarizer.py                    # Meeting summary + action items
      search_index.py                  # Transcript search + embeddings
      chat.py                          # Chat context builder
    ui/
      source_selector.py              # Mic dropdown + per-app picker (Win11) or legacy loopback (Win10)
      recording_controls.py           # Record/Pause/Stop buttons + timer
      recording_header.py             # Recording info display with rename
      segment_widget.py               # Interactive transcript segment row
      settings_dialog.py              # Settings dialog with tabs
      speaker_name_panel.py           # Collapsible speaker name mapping panel
      status_panel.py                 # System status dialog (dependency health checks)
      transcript_viewer.py            # Display + export transcripts (with interactive segments)
      notes_panel.py                  # Call notes with timestamps
      recordings_list.py              # Past recordings browser
      level_meter.py                   # Real-time audio level meters
      waveform_display.py             # Live waveform visualization
      transcript_search_bar.py        # Find/replace for transcripts
      search_bar.py                    # Recordings search bar
      summary_panel.py                 # AI meeting summary display
      action_items_panel.py            # AI action items display
      chat_panel.py                    # Chat with transcript panel
      about_dialog.py                  # About dialog with donation link
    utils/
      audio_devices.py                # Device enumeration (sounddevice)
      audio_session_monitor.py        # Per-app audio session enumeration (pycaw)
      config.py                       # JSON config management
      dependency_checker.py           # System health checks for status panel

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ObscureAintSecure/TalkTrack](https://github.com/ObscureAintSecure/TalkTrack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
