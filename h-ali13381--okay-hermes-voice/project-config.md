---
trigger: always_on
description: Guidance for AI agents working in this repository.
---

# AGENTS.md

Guidance for AI agents working in this repository.

## Project summary

`okay-hermes-voice` is a Linux voice add-on for Hermes Agent. It provides a local "Okay Hermes" wake phrase, captures the user's spoken request, transcribes it, routes the request through Hermes Agent, and plays the answer out loud.

Core goals:

- Keep always-on idle listening local and lightweight.
- Preserve normal Hermes Agent behavior: same model config, memory, tools, skills, and personality.
- Prefer native/event-driven Linux integration over polling subprocesses.
- Keep runtime behavior practical and measurable: verify CPU, memory, service status, and tests after daemon/tray changes.

Primary runtime path:

1. Native wake listener detects the wake phrase.
2. Python activation handler or warm activation server receives the activation payload.
3. Activation flow records/transcribes speech, launches optional visualization, routes the request, calls Hermes, and plays TTS.
4. Conversation mode remains active until a close phrase is heard.

## Repository map

- `src/okay_hermes_voice/wakeword_daemon.py`
  - Main Python CLI entrypoint: `okay-hermes-voice`.
  - Owns daemon orchestration only: config, warmup, wake loop, activation handoff, cooldown.

- `src/okay_hermes_voice/daemon_config.py`
  - Config defaults, YAML loading, logging, signal handling.
  - User config lives at `~/.hermes/wakeword/config.yaml`.

- `src/okay_hermes_voice/audio/`
  - Wakeword capture/inference, recording, WAV helpers, transcription provider wrappers, audio device helpers.
  - `audio/wake/` contains wake capture/session/inference mechanics.
  - `audio/nemotron/` and `audio/parakeet/` contain NVIDIA streaming ASR integrations.

- `src/okay_hermes_voice/activation_flow.py`
  - Public facade for activation handling.
  - Implementation lives under `src/okay_hermes_voice/activation/flow/`.

- `src/okay_hermes_voice/activation/flow/`
  - Voice-session orchestration: setup, routing, turn input, timing, visualization, cancellation, outcomes.

- `src/okay_hermes_voice/activation_archive/` and `src/okay_hermes_voice/activation/archive/summary/`
  - Activation audio/metadata persistence and latency summary reporting.

- `src/okay_hermes_voice/hermes_runtime.py`
  - Warm/in-process Hermes runtime integration.

- `src/okay_hermes_voice/hermes_subprocess/`
  - Subprocess-based Hermes execution, output parsing, cancellation, termination.

- `src/okay_hermes_voice/interaction_router.py`, `interaction_clients/`, `interaction_routes/`, `voice_routing/`
  - Fast interaction routing, close phrase handling, acknowledgement generation/playback, optional small-model path.
  - Keep router responsibilities split: classification in `interaction_clients/router_classification.py`, provider-client caching in `interaction_clients/router_client_cache.py`, startup prewarm in `interaction_clients/router_prewarm.py`, and daemon orchestration in `native_activation_server.py`.
  - Router prewarm resolves/caches the provider client only; do not add dummy classification calls at startup unless explicitly measuring token/API tradeoffs.

- `src/okay_hermes_voice/playback/response/`
  - Beeps, TTS playback, sink selection, process waiting/termination.

- `src/okay_hermes_voice/visualization/`
  - Optional terminal popup state, launcher, rendering pipeline, input handling.
  - `voice_activation_popup.py` is the installed CLI entrypoint facade.

- `src/okay_hermes_voice/native_activation_handler.py`
  - Short-lived Python handler launched by the native wake listener after an activation.
  - Can forward to the warm activation server when enabled.

- `src/okay_hermes_voice/native_activation_server.py`
  - Warm Unix-socket activation server that keeps STT/Hermes initialized for lower latency.
  - Writes the `.ready` marker watched by the native tray.

- `native/okay-hermes-wake-listener.c`
  - Native always-on wake listener. This is the low-idle-residency path used by the user systemd service.

- `native/build_wake_listener.sh`
  - Builds the native C wake listener into `~/.hermes/wakeword/bin/okay-hermes-wake-listener` during service install.

- `native/wakeword-tray/`
  - Native Qt/KDE system tray control for the wakeword services.
  - Uses Qt Widgets, QtDBus, QFileSystemWatcher, and PulseAudioQt.
  - Shows red/off, yellow/starting, green/on, and gray/no-microphone states.
  - Right-click menu is intentionally minimal: `Turn ON`, `Turn OFF`, `Exit`.

- `systemd/hermes-wakeword.service`
  - User service for the native wake listener.

- `scripts/install_user_service.sh`
  - Installs the Python package into the Hermes venv, builds the native listener, installs/enables the user service.

- `scripts/install_wakeword_tray.sh`
  - Builds and installs the native tray binary to `~/.local/bin/okay-hermes-wakeword-tray` and adds a desktop autostart entry.

- `tests/`
  - Main test suite.
  - `tests/voice_conversation/` covers daemon/config/audio/activation/native listener/tray behavior.
  - `tests/interaction_router/` covers routing decisions and client behavior.

## Architecture conventions

This repo intentionally follows a root-to-leaf structure:

- Root modules tell the application story or preserve public facades.
- Implementation mechanics live in semantic subpackages.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [H-Ali13381/okay-hermes-voice](https://github.com/H-Ali13381/okay-hermes-voice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
