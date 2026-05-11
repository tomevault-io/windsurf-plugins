---
trigger: always_on
description: Your self-hosted StackChan robot assistant. A fully self-hosted voice stack for the M5Stack **StackChan** desktop robot. The default persona is "Dotty" (customizable via `make setup`). Voice I/O routes through a self-hosted xiaozhi-esp32-server; brain is ZeroClaw on whatever Linux host you've chosen for it. No cloud AI services — fully self-hosted except for the LLM call (replaceable with local Ollama).
---

# Dotty

## What This Is

Your self-hosted StackChan robot assistant. A fully self-hosted voice stack for the M5Stack **StackChan** desktop robot. The default persona is "Dotty" (customizable via `make setup`). Voice I/O routes through a self-hosted xiaozhi-esp32-server; brain is ZeroClaw on whatever Linux host you've chosen for it. No cloud AI services — fully self-hosted except for the LLM call (replaceable with local Ollama).

## Architecture

```
StackChan hardware → configured persona
  │
  │  ESP32-S3, xiaozhi firmware (built from m5stack/StackChan source)
  │  WiFi / WebSocket (Xiaozhi protocol)
  ▼
xiaozhi-esp32-server (Docker on a Linux host)
  ├─ ASR: FunASR SenseVoiceSmall (local, no cloud)
  ├─ TTS: LocalPiper (en_US-kristin-medium); EdgeTTS / StreamingEdgeTTS available as alternates
  ├─ LLM: Custom ZeroClawLLM provider (proxies to ZeroClaw host)
  └─ Emotion: Parsed from emoji in LLM response text
       │  HTTP POST /api/message
       ▼
zeroclaw-bridge (FastAPI on the ZeroClaw host)
  │  JSON-RPC 2.0 over stdio to a long-running `zeroclaw acp` child
  ▼
ZeroClaw (the brain, same host)
```

See `README.md` for the full visual architecture and message-flow diagrams.

## Network

- **Admin workstation** (this machine): Development/admin workstation. Runs Claude Code sessions.
- **Docker host**: runs xiaozhi-esp32-server. Any Linux box with Docker works. Reachable on the LAN (and optionally Tailscale).
- **ZeroClaw host**: Runs ZeroClaw + the HTTP bridge (any Linux host with a working `zeroclaw` install). Reachable on the LAN (and optionally Tailscale).
- **StackChan**: On LAN WiFi only (not on Tailnet). Needs LAN IPs for OTA and WebSocket.

SSH access is via Tailscale hostnames. Discover actual Tailscale hostnames at runtime with `tailscale status`.

This repo uses placeholders (`<XIAOZHI_HOST>`, `<ZEROCLAW_HOST>`, `<ZEROCLAW_USER>`, `<XIAOZHI_PATH>`, etc.) everywhere real values would normally appear — see the "Configuring for your environment" section of `README.md` for the full list.

## Key Paths

- **xiaozhi-server install dir** (on the Docker host): `<XIAOZHI_PATH>` (e.g. `/opt/xiaozhi-server/`)
- **Custom LLM provider** (on the Docker host): mounted into container at `/opt/xiaozhi-server/core/providers/llm/zeroclaw/`
- **ZeroClaw bridge install dir**: `<BRIDGE_PATH>` (e.g. `~/zeroclaw-bridge/`)
- **This project dir**: wherever you cloned `dotty-stackchan`

## Ports

| Service | Host | Port | Protocol |
|---------|------|------|----------|
| xiaozhi WebSocket | Docker host LAN IP | 8000 | ws:// |
| xiaozhi OTA/HTTP | Docker host LAN IP | 8003 | http:// |
| ZeroClaw bridge | ZeroClaw host LAN IP | 8080 | http:// |
| ZeroClaw gateway (ws) | ZeroClaw host localhost | 18789 | ws:// |
| ZeroClaw gateway (web UI) | ZeroClaw host localhost | 42617 | http:// |

## Config Files to Know

- `.config.yaml` (repo root; deployed to the Docker host as `data/.config.yaml`) — the xiaozhi-server override config. Never overwrite wholesale on upgrades; merge keys.
- `custom-providers/zeroclaw/zeroclaw.py` — custom LLM provider. Mounted into the container via docker-compose volume.
- `custom-providers/edge_stream/edge_stream.py` — custom streaming TTS provider. Mounted similarly.
- `custom-providers/openai_compat/openai_compat.py` — OpenAI-compatible LLM provider (alternative to ZeroClaw).
- `custom-providers/piper_local/piper_local.py` — local Piper TTS provider (offline alternative to EdgeTTS).
- `custom-providers/asr/fun_local.py` — patched FunASR provider. Adds a `language` config key (upstream hardcodes `"auto"`, which mis-detects Korean/Japanese on unclear English). Mounted as a file-level override over the upstream provider.
- `bridge.py` on the ZeroClaw host — the HTTP↔ZeroClaw translator (ACP-over-stdio client).
- `personas/default.md` — default robot persona prompt (swappable).
- `session-prompt.md` — Claude Code session prompt for infrastructure setup.

## Emotion/Expression Protocol

The LLM response MUST start with an emoji. The xiaozhi firmware parses it into a face animation:
😊=smile 😆=laugh 😢=sad 😮=surprise 🤔=thinking 😠=angry 😐=neutral 😍=love 😴=sleepy

Three layers enforce this:
1. **ZeroClaw's own agent prompt** (the configured persona) — primary source
2. **xiaozhi-server top-level `prompt:`** in `data/.config.yaml` — gets injected as system message
3. **Bridge fallback** (`_ensure_emoji_prefix` in `bridge.py`) — if the first non-whitespace char isn't a non-ASCII symbol, prepends 😐 before returning.

## Key Directories

- `custom-providers/` — all custom ASR/LLM/TTS providers (mounted into the xiaozhi container)
- `bridge/` — bridge Python dependencies (`requirements.txt`)
- `firmware/` — StackChan firmware patches, remote config, and server-side OTA assets
- `personas/` — swappable robot persona prompts
- `docs/` — deep technical reference (architecture, hardware, protocols, brain, latent capabilities)

## Make Targets


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BrettKinny/dotty-stackchan](https://github.com/BrettKinny/dotty-stackchan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
