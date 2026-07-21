---
trigger: always_on
description: > **Primary context file for coding agents.** Read this entire file before making changes.
---

# VoxBento — Agent Guide

> **Primary context file for coding agents.** Read this entire file before making changes.
> All implementation details are derived from the live codebase. Constraints below are non-negotiable.

---

## Quick-Start Context

Read these files in order before any task:

1. **This file** — guardrails, invariants, ownership
2. [`.agents/context/REPOSITORY_CONTEXT.md`](.agents/context/REPOSITORY_CONTEXT.md) — stack, auth, booth identity, ports, role hierarchy
3. [`.agents/context/CHANGE_IMPACT_MAP.md`](.agents/context/CHANGE_IMPACT_MAP.md) — which files to touch for your specific task
4. [`.agents/context/ROUTE_MAP.md`](.agents/context/ROUTE_MAP.md) — all HTTP + WS routes
5. [`.agents/context/DATABASE_MAP.md`](.agents/context/DATABASE_MAP.md) — table schemas, migrations, CRUD helpers
6. [`.agents/context/TRANSCRIPTION_MAP.md`](.agents/context/TRANSCRIPTION_MAP.md) — provider architecture, audio pipeline
7. [`.agents/context/AI_WORKFLOWS.md`](.agents/context/AI_WORKFLOWS.md) — step-by-step task playbooks
8. [`.agents/context/TECHNICAL_DEBT_REPORT.md`](.agents/context/TECHNICAL_DEBT_REPORT.md) — known issues and gaps

### File-Specific Instructions (apply when editing)

| File pattern | Instruction file |
|---|---|
| `**/*.py` | [`.github/instructions/python.instructions.md`](.github/instructions/python.instructions.md) |
| `**/*.{js,ts,vue}` | [`.github/instructions/js.instructions.md`](.github/instructions/js.instructions.md) |
| `**/jinja-templates/**/*.jinja` | [`.github/instructions/jinja.instructions.md`](.github/instructions/jinja.instructions.md) |

### Skills (use for specialised tasks)

| Task | Skill |
|---|---|
| Navigate codebase | [`.agents/skills/repo-navigation/SKILL.md`](.agents/skills/repo-navigation/SKILL.md) |
| Architecture review | [`.agents/skills/architecture-review/SKILL.md`](.agents/skills/architecture-review/SKILL.md) |
| Route analysis | [`.agents/skills/route-analysis/SKILL.md`](.agents/skills/route-analysis/SKILL.md) |
| Database analysis | [`.agents/skills/database-analysis/SKILL.md`](.agents/skills/database-analysis/SKILL.md) |
| Transcription changes | [`.agents/skills/transcription-analysis/SKILL.md`](.agents/skills/transcription-analysis/SKILL.md) |
| Provider integration | [`.agents/skills/provider-analysis/SKILL.md`](.agents/skills/provider-analysis/SKILL.md) |
| PR review | [`.agents/skills/pr-review/SKILL.md`](.agents/skills/pr-review/SKILL.md) |
| Security audit | [`.agents/skills/security-audit/SKILL.md`](.agents/skills/security-audit/SKILL.md) |
| Docker / infra | [`.agents/skills/docker-review/SKILL.md`](.agents/skills/docker-review/SKILL.md) |
| Deployment | [`.agents/skills/deployment-review/SKILL.md`](.agents/skills/deployment-review/SKILL.md) |
| Incident response | [`.agents/skills/incident-investigation/SKILL.md`](.agents/skills/incident-investigation/SKILL.md) |
| Writing tests | [`.agents/skills/test-generation/SKILL.md`](.agents/skills/test-generation/SKILL.md) |
| Production readiness | [`.agents/skills/production-readiness-review/SKILL.md`](.agents/skills/production-readiness-review/SKILL.md) |

---

## Product Intent

VoxBento is a production-grade **browser-first interpretation booth console** for Eventyay live events.

- Interpreters monitor the floor session via a self-hosted Jitsi iframe (receive-only).
- Interpreters publish audio via browser WebRTC → WHIP → MediaMTX.
- Attendees receive sub-second audio via WHEP from MediaMTX.
- All coordination (booth state, roles, chat, handoff) flows through FastAPI WebSockets.

**Stack:** FastAPI (ASGI/uvicorn) + MediaMTX (WHIP/WHEP/RTSP) + self-hosted Jitsi Meet (stable-9823).
**No** Flask, Socket.IO, aiortc.

---

## Module Ownership

| File / Directory | Owns |
|---|---|
| `fastapi_app.py` | Application lifespan, router aggregation, global exception handlers |
| `portal/routers/` | All HTTP routes (pages, admin, REST API), Jinja2 template rendering |
| `portal/websockets/` | WebSocket connection manager, message handlers (`_handle_join`, etc.) |
| `portal/booth_state.py` | In-memory `BoothRegistry`, `Booth`, `Participant`, handoff policy, chat history |
| `portal/auth.py` | JWT create/decode, bcrypt password, `require_admin`, `require_user`, `resolve_booth_role`, `can_perform_role` |
| `portal/config.py` | `Settings` — pydantic-settings; all env vars / `.env` |
| `portal/models.py` | SQLAlchemy declarative models: `Event`, `Room`, `DBBooth`, `InviteToken`, `User`, `EventMembership`, `BoothMembership` |
| `portal/database.py` | Async engine, session factory (`get_session`), all CRUD helpers |
| `portal/booth_identity.py` | `make_booth_id`, `make_mediamtx_path`, `parse_booth_id`, `validate_event_slug`, `validate_language_code` |
| `portal/roles.py` | `Permission` enum, `ROLE_PERMISSIONS` dict, `ALL_ROLES` set, `_ROLE_RANK` |
| `portal/crypto.py` | `encrypt_val` / `decrypt_val` (Fernet, SHA-256 key derivation) |
| `portal/transcription/` | Transcription subsystem — see `TRANSCRIPTION_MAP.md` |
| `templates/` | Jinja2 HTML — `base.html`, `interpreter_booth.html`, `listener-event.html`, `admin/` |
| `static/js/interpreter-booth.js` | Booth UI — WebRTC/WHIP, WebSocket, Jitsi iframe, mic controls, level meter |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fossasia/eventyay-interpretation-portal](https://github.com/fossasia/eventyay-interpretation-portal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
