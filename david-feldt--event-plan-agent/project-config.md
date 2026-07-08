---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**Woodsy** — AI agent that coordinates social plans with friends over voice and WhatsApp/SMS. Calls users weekly, learns availability, proposes plans, sends invites. React Native app shows plans and pending invites.

## Commands

```bash
# Development
./dev.py server          # Start FastAPI backend (port 8000, auto-reload)
./dev.py tunnel          # Start ngrok tunnel for Twilio webhooks
./dev.py dev             # Server + ngrok together (typical dev setup)
./dev.py mobile          # Start Expo app (from mobile/)
./dev.py seed            # Seed database via migrations/seed.py

# Simulator (test without external services)
./dev.py simulate                              # Engine/scorer offline only
./dev.py simulate --llm                        # + Gemini conversation
./dev.py simulate --text --test               # In-memory text chat (no Supabase)
./dev.py simulate --text --test --phone +16471234567  # With area code inference
./dev.py simulate --text                       # Interactive WhatsApp-like chat (needs Supabase)
./dev.py simulate --demo                       # Simple 2-user demo (Alice ↔ Bob) — auto-plays a message grid showing engine matching; persists to .woodsy_demo.db (real agent + queue, needs GOOGLE_API_KEY; runs with WOODSY_DEMO=1 to skip memory-extraction + web-search calls)
./dev.py simulate --scenarios                  # Same scripted scenarios, silent + engine timeline (persists to .woodsy_demo.db, needs GOOGLE_API_KEY)
./dev.py simulate --multi                      # Interactive multi-user chat: type "1 <msg>" to speak as user 1

# Trigger outbound call manually
curl -X POST "http://localhost:8000/api/voice/calls/outbound?user_id=<uuid>"
```

There is no traditional test suite — `tests/` is empty. Use simulator modes for isolated testing.

## Architecture

### Channels

Two independent ingress paths share the same agent and tools:

| Channel | Entry Point | Transport |
|---------|-------------|-----------|
| Voice | `GET /api/voice/twiml` → `WS /api/voice/stream` | Twilio Media Streams + Deepgram STT/TTS |
| WhatsApp/SMS | `POST /api/messaging/inbound` | Twilio webhook → TwiML reply |

An **admin dashboard** at `GET /admin` (`app/api/admin.py`, Jinja2 templates in `templates/`) renders a read-only view of the engine queue, users + intent/availability, active suggestions, and plans. All queries are wrapped in `_safe_query` so missing tables degrade gracefully.

### Voice Pipeline (end-to-end)

Scheduler or manual trigger → Twilio calls user → Twilio fetches TwiML → Twilio opens WebSocket to `/api/voice/stream` → audio frames streamed to Deepgram STT → transcripts fed to `VoicePipeline` (Gemini-2.5-Flash + tool loop) → response text → Deepgram TTS → mulaw audio chunks → back to Twilio → phone. On `stop` event: full transcript saved, memories extracted async.

### Text Pipeline (end-to-end)

Twilio webhook POST → `route_message()` in `app/agent/router.py` checks user state:
1. Unknown phone → start onboarding (`app/agent/onboarding.py`, multi-step Gemini conversation)
2. Onboarding in progress → continue onboarding
3. Pending RSVP → detect intent (`app/agent/rsvp.py`) → update `plan_members.invite_status`
4. Default → Woodsy agent (`app/agent/woodsy.py`, Gemini + tool loop)

Response wrapped in TwiML and returned synchronously to Twilio.

### Agent Tools (`app/agent/tools.py`)

7 tools available in both voice and text agents:

| Tool | Purpose |
|------|---------|
| `get_user_context` | User profile, memories, recent plans |
| `record_availability` | Save availability windows (writes to `users.availability_windows`, sets `plan_intent`, pushes an `availability_updated` engine job) |
| `search_events` | Match events to interests via pgvector |
| `search_web` | Brave Search for local events/activity ideas (needs `BRAVE_API_KEY`) |
| `propose_plans` | Get new-plan clusters and join-existing suggestions |
| `send_invite` | Invite friends (prioritized: friends first) |
| `update_memory` | Store facts (preference/constraint/feedback/relationship_note) |

Tool calling loop: max 5 iterations, results fed back to Gemini as context.

### Planning Engine (`app/engine/`)

- `scorer.py` — Compatibility scoring: interest similarity (pgvector cosine), availability overlap, location, friendship strength
- `planner.py` — Identifies clusters of 3+ compatible users for new plans; finds open plans to join
- `availability.py` — Fuzzy time parsing ("Saturday afternoon" → ISO windows) + overlap detection
- `lifecycle.py` — Plan state machine: `FORMING → PROPOSED → CONFIRMED → LIVE → PAST/CANCELLED`
- `invites.py` — Prioritized invite sequencing
- `queue.py` — Supabase-backed job queue (`engine_jobs` table) that decouples engine runs from the request/scheduler.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [David-Feldt/event-plan-agent](https://github.com/David-Feldt/event-plan-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
