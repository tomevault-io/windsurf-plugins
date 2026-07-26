---
trigger: always_on
description: This file is the quick orientation map for future coding agents.
---

# AGENT Guide: StreamFlow Architecture

This file is the quick orientation map for future coding agents.
Use it to decide where changes belong before editing code.

## 1) First Principles

- Keep HTTP routes thin and move behavior into domain handlers/services.
- Prefer extending extracted modules over adding logic back into large legacy files.
- For write endpoints, validate request payloads with schema helpers in `backend/apps/api/schemas.py`.
- Return consistent API envelopes via shared response helpers (`success_response`, `error_response`).
- Preserve SQL-backed behavior (avoid reintroducing JSON-file state for setup/automation flows).

## 2) Architecture Map

### Backend (`backend/apps`)

- `api/`: HTTP entry points and request/response boundary logic.
  - `web_api.py`: route registration and app wiring; avoid new heavy inline logic.
  - `*_handlers.py`: domain-specific endpoint handlers (automation, scheduling, channels, regex, sessions, telemetry, etc.).
  - `schemas.py`: payload validation/dataclass parsing for API writes.
  - `middleware.py`: cross-cutting API middleware (rate limiting, etc.).
- `automation/`: automation configuration and matching orchestration.
  - `automation_config_manager.py`: profile/period/assignment persistence and behavior toggles.
  - `automated_stream_manager.py`: update/matching pipeline coordination.
- `stream/`: stream checking, session tracking, runtime transport helpers.
  - `stream_checker_service.py`: large orchestrator for stream checks.
  - `stream_checker_components.py`: extracted stream-checking support components.
  - `acestream_session_service.py`: AceStream session lifecycle/scoring logic.
  - `udp_proxy.py`: runtime UDP proxy behavior.
- `database/`: persistence layer.
  - `models.py`: SQLAlchemy models.
  - `manager.py`: broad DB manager (legacy-heavy, still in use).
  - `repositories/`: newer repository abstractions (prefer extending here for new focused queries).
- `udi/`: Dispatcharr-backed cached data access layer.

### Frontend (`frontend/src`)

- `pages/`: route-level orchestration pages.
  - `pages/ChannelConfiguration.jsx`: still large; keep extracting render-heavy and dialog-heavy sections into components.
- `components/channel-configuration/`: extracted Channel Configuration units.
  - `ChannelCard.jsx`, `RegexTableRow.jsx`, `PeriodDialogs.jsx`, `patternUtils.js`.
- `services/`: API clients and cache/session helpers.
  - `api.js`, `streamSessions.js`, `aceStreamMonitoring.js`, `channelCache.js`.
- `components/ui/`: shared UI primitives.

## 3) Where To Change Code (Task Routing)

### A) Add or change an API endpoint

1. Add/update handler logic in the relevant file under `backend/apps/api/*_handlers.py`.
2. Add request schema parsing/validation in `backend/apps/api/schemas.py` for write payloads.
3. Wire route in `backend/apps/api/web_api.py` (delegate to handler, keep route body minimal).
4. Use standardized success/error envelopes and explicit HTTP status codes.
5. Add or update backend tests under `backend/tests/`.

### B) Change automation profiles/periods/assignments

1. Start in `backend/apps/api/automation_handlers.py` and `backend/apps/api/schemas.py`.
2. If persistence rules change, update `backend/apps/automation/automation_config_manager.py`.
3. Validate frontend flows in `frontend/src/pages/ChannelConfiguration.jsx` and related extracted components.
4. Ensure batch/single assignment variants stay behaviorally aligned.

### C) Change stream matching behavior (regex/TVG)

1. Backend matching/orchestration: `backend/apps/automation/automated_stream_manager.py`.
2. Regex API behavior: `backend/apps/api/regex_handlers.py`.
3. Frontend pattern UI: `frontend/src/components/channel-configuration/RegexTableRow.jsx` and page orchestration.
4. Keep safety validation in place for risky regex patterns.

### D) Change stream checking or ranking behavior

1. Core behavior: `backend/apps/stream/stream_checker_service.py`.
2. Prefer adding reusable helpers in `backend/apps/stream/stream_checker_components.py`.
3. Update related APIs in `backend/apps/api/stream_checker_handlers.py`.
4. Validate with targeted tests under `backend/tests/test_stream_*` and relevant automation integration tests.

### E) Change setup wizard or initialization state

1. Use SQL-backed settings and channel regex tables; do not reintroduce JSON-file wizard state.
2. Primary API surface: `backend/apps/api/setup_wizard_handlers.py`.
3. Keep readiness checks aligned with Dispatcharr configuration state.

### F) UI-only changes in Channel Configuration

1. Keep page-level orchestration in `frontend/src/pages/ChannelConfiguration.jsx`.
2. Place reusable UI blocks in `frontend/src/components/channel-configuration/`.
3. Keep API calls in service modules when practical; avoid duplicating request logic across components.

## 4) Current Refactor Guardrails

- Avoid adding new complex business logic to `backend/apps/api/web_api.py`.
- Avoid expanding `frontend/src/pages/ChannelConfiguration.jsx` with new giant inline subcomponents.
- Prefer extending extracted component/service modules instead of adding more in-file monolith logic.
- For new write APIs, require schema-based parsing in `schemas.py` before handler execution.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [krinkuto11/streamflow](https://github.com/krinkuto11/streamflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
