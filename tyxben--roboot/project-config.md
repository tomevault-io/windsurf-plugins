---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Roboot

A personal AI agent hub that runs on macOS. It manages multiple Claude Code sessions in iTerm2, executes shell commands, and provides a JARVIS-like voice interface -- all powered by the Arcana agent framework. Supports remote access via LAN, Telegram, and a Cloudflare Worker relay.

## Architecture

```
server.py (FastAPI)              <- Main entry point
+-- WebSocket /ws                <- Streaming chat (LLM_CHUNK events)
+-- REST /api/sessions/*         <- Direct iTerm2 session control
+-- REST /api/tts                <- Edge TTS (text -> mp3)
+-- REST /api/relay-info         <- Relay pairing URL + expiry
+-- REST /api/relay-refresh      <- Rotate relay token (POST)
+-- REST /api/relay-revoke       <- Kick all clients + rotate (POST)
+-- REST /api/relay-qr           <- QR code PNG for relay pairing URL
+-- REST /api/network-info       <- Local IP addresses
+-- REST /api/qr-code            <- QR code PNG for LAN URL
+-- REST /api/filevault-status   <- macOS FileVault probe (fdesetup)
+-- REST /api/chat-history-wipe  <- Drop all chat rows + VACUUM (POST)
+-- Static /static/console.html  <- Unified web console

relay_client.py                  <- Relay WebSocket client (connects to CF Worker)
iterm_bridge.py                  <- Persistent iTerm2 Python API connection
network_utils.py                 <- IP detection, QR code generation
soul.md                          <- Assistant's self-modifiable identity
config.yaml                      <- API keys + provider config (gitignored)

text_utils.py                    <- Shared helpers (extract_spoken_text, ...)
tts_synth.py                     <- Edge TTS helper shared by /api/tts + mobile relay
soul_review.py                   <- Review gate for soul.md overwrites (off/log/confirm)
tool_guard.py                    <- Approval gate for agent tool calls (off/log/confirm) — gates run_command via Arcana confirmation_callback
filevault_status.py              <- macOS fdesetup probe for the console warning banner

tools/                           <- Arcana tools (agent's capabilities)
+-- shell.py                     <- Terminal command execution
+-- claude_code.py               <- iTerm2 session list/read/send/create
+-- vision.py                    <- Camera capture + screenshot + face recognition
+-- face_db.py                   <- Face encoding storage and matching (.faces/)
+-- soul.py                      <- Self-modification + user memory
+-- voice_switch.py              <- Agent tool: change Telegram TTS voice

adapters/                        <- I/O adapters
+-- telegram_bot.py              <- Remote control via Telegram (voice I/O)
+-- voice.py                     <- Local mic STT + macOS TTS (CLI --voice)
+-- voice_prefs.py               <- Per-Telegram-user TTS voice store
+-- tts_streamer.py              <- Edge TTS -> parallel OGG/Opus synthesis
+-- keyboard.py                  <- Terminal text input
+-- stt/                         <- Pluggable speech-to-text backends
    +-- mlx.py                   <- mlx-whisper (Apple Silicon, default)
    +-- google.py                <- speech_recognition -> Google Web Speech
    +-- noop.py                  <- backend: none

scripts/
+-- setup.sh                     <- One-command installer (auto-uses uv)

relay/                           <- Cloudflare Worker relay server
+-- src/index.ts                 <- Worker entry point, routing, rate limiting
+-- src/relay-session.ts         <- Durable Object: daemon<->client session mgmt
+-- src/pair-page.ts             <- HTML pairing page served to mobile clients
+-- wrangler.toml                <- Cloudflare deployment config
```

## Commands

```bash
# Main web console (primary way to use)
python server.py                    # -> http://localhost:8765

# CLI modes
python run.py                       # Keyboard chat
python run.py --voice               # Voice chat

# Telegram bot (needs bot_token in config.yaml)
python -m adapters.telegram_bot

# Chainlit UI (alternative frontend)
chainlit run chainlit_app.py -w

# Install (recommended: scripts/setup.sh handles deps + ffmpeg + Whisper prewarm)
./scripts/setup.sh                                       # default: telegram extras
./scripts/setup.sh --with=core                           # minimal (no voice)
./scripts/setup.sh --with=all                            # everything

# Manual install (if you don't want setup.sh)
pip install -e .                                         # core
pip install -e '.[telegram]'                             # + Telegram voice I/O
pip install -e '.[all]'                                  # + vision + CLI voice + desktop
python -m adapters.stt prewarm                           # pre-cache Whisper model (~3 GB)

# Deploy relay (requires wrangler CLI + Cloudflare account)
cd relay && npm install && npx wrangler deploy
```

## Key Design Decisions

### Agent: Arcana 0.8.2

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tyxben/roboot](https://github.com/tyxben/roboot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
