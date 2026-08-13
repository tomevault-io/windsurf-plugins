---
trigger: always_on
description: vibecheck is a mobile PWA bridge for Mistral Vibe. It hooks into Vibe's Textual/Python event system (AgentLoop callbacks) and exposes them over WebSocket to a Svelte 5 mobile client. See README.md for the full product brief, docs/PLAN.md for the architecture, and docs/IMPLEMENTATION.md for the work unit punchlist.
---

# vibecheck — Project Instructions

## What This Is

vibecheck is a mobile PWA bridge for Mistral Vibe. It hooks into Vibe's Textual/Python event system (AgentLoop callbacks) and exposes them over WebSocket to a Svelte 5 mobile client. See README.md for the full product brief, docs/PLAN.md for the architecture, and docs/IMPLEMENTATION.md for the work unit punchlist.

## Architecture

```
Phone (PWA) → HTTPS/WSS → EC2 (Caddy → FastAPI bridge → Vibe AgentLoop)
```

- **Backend:** FastAPI + Starlette, in-process with AgentLoop
- **Frontend:** Svelte 5 + Vite PWA
- **Key hooks:** `set_approval_callback()`, `set_user_input_callback()`, `message_observer`
- **Events:** `BaseEvent` hierarchy (AssistantEvent, ToolCallEvent, ToolResultEvent, etc.)

## Tech Stack

- Python 3.12+ via uv (never bare `python` or `pip`)
- FastAPI + Starlette (async, WebSocket)
- Svelte 5 + Vite (frontend)
- Mistral SDK (`mistralai`) for Voxtral, Ministral, Small APIs
- pywebpush for VAPID push notifications

## Environment Variables

These are set globally on the development machine. Do not hardcode, auto-generate, or prompt for them.

| Variable | Purpose |
|----------|---------|
| `VIBECHECK_PSK` | Pre-shared key for API auth. Sent as `X-PSK` header. Already set in the environment. |
| `MISTRAL_API_KEY` | Mistral SDK key. Only needed for live voice/translation (L3+). Mock in tests. |

## Key Files

| File | Purpose |
|------|---------|
| `docs/IMPLEMENTATION.md` | **Work unit punchlist — start here for tasks** |
| `docs/PLAN.md` | Hackathon execution plan (layers L0-L9) |
| `README.md` | Full product brief |
| `docs/DEMO.md` | Presentation script and demo setup |
| `docs/ANALYSIS-vibe.md` | Vibe v2.2.1 internals (AgentLoop, events, callbacks) |
| `docs/REMOTING-UI.md` | Mobile-to-TUI bridge architecture research |
| `WORKLOG.md` | **Running log of all major actions** — installations, config changes, decisions, not just WU completions |
| `scripts/` | Smoke tests, event replay |
| `vibecheck/` | Python backend package |
| `vibecheck/frontend/` | Svelte 5 + Vite PWA |
| `vibecheck/tests/` | Backend pytest suite |
| `prototypes/` | Standalone browser-API test pages |
| `frontend-prototype/` | **Standalone STT+TTS voice loop** (Voxtral + ElevenLabs) — reuse for WU-17/18/29/30 |
| `tests/fixtures/` | Sample event sequences for replay/testing |
| `reference/mistral-vibe/` | **Vibe source checkout** — read this for AgentLoop, event types, callback signatures |

---

## Test-Driven Development

**Every change must be verifiable. Write tests first, implement second, verify third.**

### The Loop

1. **Write the test** — define expected behavior before implementation
2. **Run the test** — confirm it fails (proves test is meaningful)
3. **Implement** — minimal code to pass the test
4. **Run the test** — confirm it passes
5. **Run all related tests** — confirm nothing broke

### Backend Tests (Python)

```bash
# Run all backend tests
uv run pytest vibecheck/tests/ -v

# Run a specific test file
uv run pytest vibecheck/tests/test_api.py -v

# Run a single test
uv run pytest vibecheck/tests/test_api.py::test_health_endpoint -v

# Run with coverage
uv run pytest vibecheck/tests/ --cov=vibecheck --cov-report=term-missing
```

**Test requirements:**
- Every REST endpoint: happy path, auth failure (no PSK, bad PSK), error cases
- Every Pydantic model: serialization round-trip, validation errors
- WebSocket: connect/disconnect, auth, broadcast, backlog delivery
- Bridge: state transitions, approval/input Future resolution, event broadcasting
- API integrations (Voxtral, Ministral, translation): mock the Mistral SDK, test our proxy logic

**Mock the external world:**
- Use `MockBridge` (no real Vibe instance needed for API tests)
- Use `unittest.mock.patch` for Mistral SDK calls
- Use httpx `AsyncClient` for FastAPI test client
- Never require network access or running services in unit tests

### Frontend Tests

```bash
# Build must succeed (minimum bar)
cd vibecheck/frontend && npm run build

# Dev server must start
npm run dev  # verify port responds

# Component tests (if vitest configured)
npm test
```

### Frontend Deploy

Vite builds into `vibecheck/static/` which FastAPI serves directly. **After any frontend change, you must rebuild and restart the server** for the change to be live:

```bash
cd vibecheck/frontend && npm run build   # outputs to ../static/
# Then restart the server (kill and re-run):
uv run vibecheck-vibe                    # recommended: Vibe TUI + bridge
# or: uv run python -m vibecheck        # server-only bridge
```

Without the rebuild, the server continues serving stale cached bundles. The build produces hashed filenames (e.g. `index-BHv5LYMU.css`) so the old files won't match.

### Prototype Tests

Each prototype has a `test.sh`:
```bash
cd prototypes/websocket-reconnect && ./test.sh
```

### Smoke Tests (Integration)

```bash
# Against local or remote server
scripts/smoke_test.sh http://localhost:7870
scripts/smoke_test.sh https://vibecheck.shisa.ai
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lhl/vibecheck](https://github.com/lhl/vibecheck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
