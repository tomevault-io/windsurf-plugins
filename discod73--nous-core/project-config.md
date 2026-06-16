---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

NOUS is a Danish-language, locally-hosted personal AI assistant. It runs across two machines on a private LAN with the inference host air-gapped from the internet.

| Device | IP (example) | Role |
|---|---|---|
| Pi 5 16GB (primary host) | `$NOUS_PI5_IP` | Wake-word detection, VAD, TTS, ingest, proxy, RAG |
| Jetson Orin Nano 8GB (inference host) | `$NOUS_JETSON_IP` | LLM inference (port 11434), STT (Whisper, port 8080) |

The inference host has no default route. All internet access goes through `nous-proxy` on the primary host (port 8090). Set actual IPs via environment variables , see `.env.example`.

## Python environment

The main venv used by all scripts and the systemd service is `/srv/nous/pipeline/.venv`. This must be first in PATH when running under systemd (critical , without it, `piper` and `sox` are not found).

```bash
source /srv/nous/pipeline/.venv/bin/activate
```

`/srv/nous/app/.venv` is a separate venv used only by Kuzu-related code.

## Key commands

```bash
# System health check
bash /srv/nous/scripts/smoke-test.sh

# Live service logs
journalctl -u nous-voice-assistant.service -f

# Restart voice assistant service
sudo systemctl restart nous-voice-assistant.service

# Manual one-shot voice session (bypasses wake-word)
source /srv/nous/pipeline/.venv/bin/activate
python3 /srv/nous/scripts/voice_chat.py

# Text chat (no voice)
python3 /srv/nous/scripts/nous_chat.py 'dit spørgsmål'

# Start the internet proxy manually (normally a service)
cd /srv/nous/proxy && uvicorn nous_proxy:app --host 0.0.0.0 --port 8090

# Backup persistent data
bash /srv/nous/scripts/nous-backup.sh

# Set up Qdrant collections (run after Qdrant restart or new wing install)
python3 /srv/nous/scripts/nous-setup-collections.py

# Ingest a document interactively
python3 /srv/nous/scripts/nous-ingest-manual.py /sti/til/fil.pdf

# Promote document chunks between wings with PII review
python3 /srv/nous/scripts/promote.py <source_wing> <target_scope> [doc_id]

# PII detection/anonymization CLI
python3 /srv/nous/pipeline/privacy_guard.py detect "tekst"
python3 /srv/nous/pipeline/privacy_guard.py mask "tekst" PUBLIC
python3 /srv/nous/pipeline/privacy_guard.py audit "tekst" SWARM

# Repair a corrupted Qdrant collection
python3 /srv/nous/pipeline/fix_wing.py <wing_name>

# Test audio pipeline (TTS + playback) without full voice session
bash /srv/nous/scripts/voice_test.sh

# Docker infrastructure (Qdrant + SearXNG)
cd /srv/nous && docker compose up -d
cd /srv/nous && docker compose restart qdrant
cd /srv/nous && docker compose logs -f qdrant

# Check Ollama/Whisper on inference host (replace with $NOUS_JETSON_IP)
curl http://$NOUS_JETSON_IP:8080/health   # Whisper STT
curl http://$NOUS_JETSON_IP:11434/health  # Ollama LLM
```

## Architecture

### Voice pipeline (end-to-end)

```
ReSpeaker XVF3800 (hw:2,0 on Pi 5)
  → voice_assistant.py: openWakeWord ("hey_jarvis", threshold 0.5)
  → stream.close() + 0.8s ALSA release + Piper "Ja?" ACK
  → voice_chat.py subprocess:
      Silero VAD (threshold 0.4, 16-frame pre-roll)
      → Whisper STT (Jetson:8080, language=da, temperature=0.0)
      → Qwen3-4B (Jetson:8081, /v1/chat/completions, tools)
      → nous-proxy tool execution (Pi:8090)
      → clean_for_tts() strips markdown/emoji/URLs
      → Piper TTS streaming (da.onnx) via asyncio pipeline
      → sox + ALSA 48kHz stereo → speaker
  → 2.0s cooldown → reopen stream → listen again
```

The subprocess model (`voice_assistant.py` spawning `voice_chat.py`) ensures full ALSA device release between sessions. Consolidation into one process is a known P0 item.

### Services on Pi 5

| Service | File | Port |
|---|---|---|
| `nous-voice-assistant.service` | `/etc/systemd/system/nous-voice-assistant.service` | , |
| nous-proxy | `/srv/nous/proxy/nous_proxy.py` | 8090 |
| Qdrant | Docker (`docker-compose.yml`) | 6333, 6334 |
| SearXNG | Docker (`docker-compose.yml`) | 8080 (localhost) |

Proxy endpoints: `/health`, `/time`, `/weather?location=`, `/search?q=`, `/fetch?url=`

### LLM tool-calling

`nous_chat.py` and `voice_chat.py` both implement the same three tools against nous-proxy:
- `get_time` , current Danish datetime
- `get_weather(location)` , via Open-Meteo (no API key)
- `search_web(query)` , via SearXNG

The LLM uses the OpenAI-compatible `/v1/chat/completions` endpoint on the Jetson with `model: "qwen3"`.

### Task Router (circuit breaker)

`pipeline/task_router.py` exports a global `router` instance. It tracks Jetson availability with a circuit breaker: 3 failures → OPEN (60s timeout) → HALF_OPEN → CLOSED after 2 successes. Import it as `from task_router import router`.

### Data storage and privacy model

All persistent data lives under `/mnt/nous-data/`. Qdrant vectors are at `/mnt/nous-data/qdrant/`.

Documents are stored in **wings** (topic-based Qdrant collections) with a **scope** (access level):

| Scope | Anonymization |
|---|---|
| SECRET | None , only on Pi |
| PRIVATE | None , only on owner devices |
| SWARM | All PII (CPR, phone, address, email, names) |
| PUBLIC | CPR, phone, address, email , names preserved |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Discod73/nous-core](https://github.com/Discod73/nous-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
