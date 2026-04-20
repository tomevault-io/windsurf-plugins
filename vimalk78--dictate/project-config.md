---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is this

Dictate is a voice-to-text tool for Claude Code on Linux. It records speech using faster-whisper and outputs text via clipboard or stdout. Supports local transcription or forwarding audio to a remote GPU server over TCP. Single Python script (`dictate`), no build system.

## Development

### Running locally

```bash
bash install.sh                    # full install (system deps, venv, launcher)
bash install-service.sh            # install and start systemd services
bash update.sh                     # update after code changes (no venv rebuild)
dictate --serve &                  # start daemon (keeps Whisper model in memory)
dictate --once                     # send one request to daemon, print text
dictate                            # standalone push-to-talk mode (no daemon)

# Network transcription (remote GPU)
dictate --serve --listen 0.0.0.0:5555   # on GPU machine: headless transcription server
dictate --serve --server GPU_IP:5555    # on laptop: daemon forwarding to remote
dictate --once                          # client unchanged
```

After install, reboot or re-login once for `input` group membership (required for evdev access).

After code changes, run `bash update.sh` to deploy — it copies the script, service files, and restarts services. No venv rebuild needed.

### Testing

No test suite. Manual testing:

```bash
dictate --serve &                  # start daemon
dictate --once                     # test transcription
dictate --cpu --model small        # test CPU-only inference
dictate --list-devices             # verify audio device detection
dictate --stop                     # stop daemon

# Network transcription
dictate --serve --listen 0.0.0.0:5555   # start remote server
dictate --serve --server 127.0.0.1:5555 # start forwarding daemon
dictate --once                          # test end-to-end
```

Test local daemon, network forwarding, and push-to-talk separately — they share audio and transcription code but have different I/O paths.

## Architecture

Single Python script (`dictate`, ~720 lines) with five modes:

1. **Push-to-talk** (default) — loads model, listens for key press via evdev, records, transcribes, copies to clipboard via `wl-copy`
2. **Daemon** (`--serve`) — keeps model loaded, listens on Unix socket (`~/.local/share/dictate/dictate.sock`), maintains rolling 1-second pre-buffer, handles one request at a time
3. **Daemon forwarding** (`--serve --server H:P`) — same as daemon but skips model loading; after recording, forwards audio over TCP to a remote transcription server
4. **TCP transcription server** (`--serve --listen H:P`) — headless, no mic; receives audio over TCP, transcribes with local Whisper model, returns text
5. **Client** (`--once`) — connects to daemon socket, sends JSON request with language + hints, reads newline-delimited JSON responses, prints final text to stdout (unchanged by network mode)

### Daemon ↔ Client protocol (Unix socket)

```
Client → Daemon:  {"language": "en", "initial_prompt": "..."} + shutdown(SHUT_WR)
Daemon → Client:  {"status": "recording"}\n
                   {"status": "transcribing"}\n
                   {"text": "transcribed text here"}\n
```

### Daemon ↔ TCP server protocol (network transcription)

```
Daemon → Server:
  4 bytes:  header length (uint32 big-endian)
  N bytes:  JSON header {"language": "en", "initial_prompt": "...", "audio_length": M}
  M bytes:  raw float32 audio (16kHz mono)

Server → Daemon:
  {"text": "transcribed text here"}\n
```

### Audio pipeline

`sounddevice.InputStream` (16kHz mono float32) → RMS-based silence detection → numpy array → `faster-whisper model.transcribe()`. Silence threshold is calibrated from 0.5s ambient measurement on startup: `ambient * 1.5 + 0.01`, capped at 0.05. Calibration retries on silence (rms=0) or suspiciously high ambient (>0.03, e.g. device switching). A background mic monitor thread detects disconnects and re-calibrates on reconnect, sending desktop notifications via `notify-send`.

### Key functions

- `calibrate_mic()` — ambient RMS measurement with retry logic, sets speech threshold
- `record_until_silence()` — records until post-speech silence or timeout, respects STOP_FLAG
- `transcribe_audio()` — local transcription via faster-whisper model
- `transcribe_remote()` — forwards audio to TCP server, returns text
- `serve()` — daemon loop: socket listener + pre-buffer, transcribes locally or forwards to remote
- `serve_tcp()` — headless TCP transcription server (for GPU machine)
- `client_once()` — client: connect, send request, read JSON stream
- `push_to_talk()` — standalone: evdev key detection + record + transcribe + clipboard
- `load_hints()` — merges global (`~/.config/dictate/hints.d/`) and project (`.dictate-hints.d/`) hint files
- `find_audio_device()` — prefers pipewire ALSA device for correct Bluetooth routing
- `pick_defaults()` — CUDA auto-detection: GPU → medium/int8, CPU → small/int8
- `parse_addr()` — parses HOST:PORT strings for network modes

### Claude Code integration

- `/dictate` command (`dictate.claude-command`) — loops `dictate --once`, accumulates utterances

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vimalk78) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
