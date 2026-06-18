---
trigger: always_on
description: Guidance for coding agents (and humans) working in the VoicERA monorepo. The goal is that
---

# AGENTS.md

Guidance for coding agents (and humans) working in the VoicERA monorepo. The goal is that
you can navigate the codebase, make changes safely, and run/verify them without needing a
human to explain the architecture.

> **Scope note:** This file describes what is **actually implemented** today. VoicERA does
> **not** currently have a "skill" plugin framework, a tool/function-calling abstraction, or
> cross-call memory. Agents are **configuration-driven** (a JSON document + system prompt).
> The real extension points are documented under [Extending VoicERA](#extending-voicera).

---

## Repository layout

This is a polyglot monorepo of independently deployable services.

| Path | Service | Stack | Port |
|------|---------|-------|------|
| `voicera_frontend/` | Web dashboard | Next.js / TypeScript | 3000 |
| `voicera_backend/` | REST API, auth, integrations, RAG ingest | FastAPI / Python | 8000 |
| `voice_2_voice_server/` | Real-time voice pipeline | FastAPI + Pipecat / Python | 7860 |
| `ai4bharat_stt_server/` | Local Indic STT (optional, GPU) | Python | 8001 |
| `ai4bharat_tts_server/` | Local Indic TTS (optional, GPU) | Python | 8002 |
| `llm_server/` | Optional self-hosted vLLM (Qwen) | Python | — |
| `docs/` | MkDocs documentation site | Markdown | — |
| `docker-compose.yml`, `nginx.conf`, `Makefile` | Orchestration | — | — |

Supporting infrastructure: **MongoDB** (27017) for application data, **MinIO** (9000/9001)
for recordings, transcripts, and uploaded documents.

---

## Service boundaries & communication

```
Browser/Phone ──► Frontend (3000) ──HTTP(JWT)──► Backend (8000) ──► MongoDB / MinIO
                      │                              ▲
                      └──WebSocket(audio)──► Voice Server (7860) ──HTTP(X-API-Key)──┘
                                                  │
                                                  ├──► STT / TTS / LLM provider APIs
                                                  └──► AI4Bharat / Bhashini (optional, local/cloud)
```

- **Frontend → Backend:** REST with a JWT `Authorization: Bearer <token>` (user-scoped).
- **Voice Server → Backend:** internal REST with `X-API-Key: <INTERNAL_API_KEY>` header.
  The same secret must be set in both `voicera_backend/.env` and
  `voice_2_voice_server/.env`.
- **Backend → MongoDB:** pymongo. **Backend/Voice Server → MinIO:** S3 API.
- **Multi-tenancy:** almost everything is scoped by `org_id` (agents, integrations, KB docs).
  Provider API keys are usually stored per-organization in the `Integrations` collection and
  fetched at call time (see `fetch_integration_key` in
  `voice_2_voice_server/api/services.py`), with environment variables as fallback.

---

## The agent model (config-driven)

There is **no agent class hierarchy or skill registry**. An "agent" is a document in the
MongoDB `AgentConfig` collection. Its shape is defined by the Pydantic schemas in
[voicera_backend/app/models/schemas.py](voicera_backend/app/models/schemas.py)
(`AgentConfigCreate` / `AgentConfigResponse` / `AgentConfigUpdate`).

Top-level fields include `agent_type`, `agent_id`, `org_id`, `telephony_provider`
(`Vobiz` / `Plivo`), and `vobiz_*` / `plivo_*` linkage fields. The behavior lives in a free-form
`agent_config: Dict[str, Any]` blob, typically:

```jsonc
{
  "system_prompt": "You are a helpful voice assistant. Keep replies short.",
  "greeting_message": "Hi! How can I help you today?",
  "language": "English",
  "llm_model": { "name": "OpenAI", "args": { "model": "gpt-4o", "temperature": 0.7 },
                 "knowledge_base_enabled": true,
                 "knowledge_document_ids": ["<doc_id>"],
                 "knowledge_top_k": 3 },
  "stt_model": { "name": "Deepgram", "language": "English", "args": {} },
  "tts_model": { "name": "ElevenLabs", "language": "English",
                 "args": { "voice_id": "<voice_id>" } }
}
```

### Where prompts live and how they're used
- Authored/edited in the dashboard; persisted in `AgentConfig.agent_config.system_prompt`.
- Loaded at call time in
  [voice_2_voice_server/api/bot.py](voice_2_voice_server/api/bot.py): the config is fetched
  from the backend, then the system prompt seeds an `OpenAILLMContext`
  (`OpenAILLMContext([{"role": "system", "content": system_prompt}])`).
- Conversation context is **per-call only** — it is not persisted across calls.

---

## Voice pipeline (Pipecat)

The real-time loop is a Pipecat pipeline assembled per call:

```
audio in → STT → VAD / interruption → transcript → LLM (+ optional RAG) → TTS → audio out
```

Key files in `voice_2_voice_server/`:
- `api/server.py` — FastAPI app and WebSocket entrypoints:
  - `WS /agent/{agent_id}` — generic/Vobiz audio
  - `WS /plivo/agent/{agent_id}` — Plivo audio
  - `WS /browser/agent/{agent_id}` — **browser testing** (no telephony)
  - `POST /outbound/call/` — initiate an outbound call
- `api/bot.py` — builds the pipeline, loads config, wires context.
- `api/services.py` — **provider factory**: instantiates the chosen STT/TTS/LLM service.
- `config/llm_mappings.py`, `config/stt_mappings.py` — default models & language code maps.
- `services/` — custom integrations: `ai4bharat/`, `bhashini/`, `kenpath_llm/`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [COSS-India/voicera_mono_repository](https://github.com/COSS-India/voicera_mono_repository) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
