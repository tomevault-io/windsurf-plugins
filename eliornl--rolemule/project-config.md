---
trigger: always_on
description: **Product name:** RoleMule (repo folder: `rolemule`).
---

# RoleMule — Cursor Rules Index

**Product name:** RoleMule (repo folder: `rolemule`).

All detailed coding rules live in `.cursor/rules/`. This file is the index.

---

## Rule Files

| File | Covers |
|------|--------|
| `rolemule-core.mdc` | App name, error system (`APIError`/`ErrorCode`, `CFG_6xxx` no-API-key, **`RES_3002` duplicate workflow**, post-start duplicate / `uq_user_job_company`, content fingerprint NFKC + `utils/application_dedupe.py`), **`POST /workflow/start` job file uploads** (`.pdf`/`.txt`/`.docx`, 5 MB, `_JOB_FILE_MAGIC`), background task DB, **workflow failure (no partial outputs, applications list join)** |
| `python-conventions.mdc` | Python style, file sections, FastAPI endpoint template |
| `database-patterns.mdc` | SQLAlchemy JSONB, `flag_modified()`, soft delete, **partial indexes**, race conditions, migrations, `UserWorkflowPreferences` upsert |
| `auth-patterns.mdc` | JWT structure, `_make_jwt`, all auth endpoints, token revocation, lockout, timing attack fix |
| `security-python.mdc` | XSS, file upload MIME, `secrets` vs `random`, no-JWT-in-URL rule, user enumeration prevention |
| `codeql-security-scanning.mdc` | CodeQL CI, `sanitize_log_value()` / `mask_email()` logging, query-filters, secret scanning test keys, Pydantic `field_validator`, extension entity decode |
| `security-middleware.mdc` | Middleware stack, CORS explicit headers, no wildcard origins, maintenance mode, **CSP nonce-based style-src** (no `'unsafe-inline'`), `.is-hidden` / CSS utility class reference |
| `settings-and-env.mdc` | All env vars, `get_settings()`, `ENCRYPTION_KEY` requirement, `.env` commit rules |
| `llm-integration.mdc` | Multi-provider LLM (`get_llm_client`), per-user BYOK (Gemini/OpenAI/Anthropic/Ollama), model allowlists in `utils/llm/models.py`, grounding, `CFG_6001`, `asyncio.wait_for()`, **`generate_stream` / `SpeakFieldStreamer`**, JSON parsing |
| `agent-patterns.mdc` | 5 workflow agents + standalone on-demand (interview prep, CV optimizer, Practice Interview, Hiring Outreach) + 6 career tools, **any workflow agent failure fails the run**, workflow_preferences injection, BYOK model override, **Company Research — `_has_usable_company_name` / unnamed-posting / disambiguation / staffing / `research_quality`** |
| `interview-prep-feature.mdc` | Standalone agent, background task, Redis atomic lock, rate limit (static Interview tab — not Practice Interview) |
| `cv-optimizer-feature.mdc` | CV Optimizer loop — hiring manager + reviser agents, `cv_optimization` JSONB, ownership-before-cache, WebSocket iteration events, Optimize CV tab |
| `mock-interview-feature.mdc` | Practice Interview — HR/Pro/Manager, duration minutes, browser STT/TTS, BYOK, speak streaming / `speak_delta`, `ws-guard`, `mock_interview` JSONB |
| `hiring-outreach-feature.mdc` | Hiring Outreach — public web contact finder + copy-only draft messages, `hiring_outreach` JSONB, Redis lock, WS `hiring_outreach_*`, 10th Outreach tab |
| `career-tools.mdc` | 6 tools, endpoints, rate limits, agent implementation pattern, output JSON schemas, copy button pattern, no-placeholder rules, UI CSS classes |
| `caching-redis.mdc` | Cache TTLs, helpers, rate limiting, auth-specific Redis keys |
| `websocket-patterns.mdc` | Endpoints, connection limits, broadcast helpers, WS-only JWT query-param exception, mock interview `speak_delta` + client session filter |
| `logging-patterns.mdc` | StructuredLogger, **`sanitize_log_value()` / `mask_email()`**, request context, sensitive data redaction, `exc_info=True` requirement, bulk-script safety |
| `google-oauth.mdc` | 3-step OAuth flow, CSRF state in Redis, exchange-code pattern, open-redirect prevention |
| `email-and-misc-utils.mdc` | Gmail SMTP, resume parser, BYOK encryption |
| `frontend-js-strict.mdc` | TypeScript strict mode, null safety, anti-patterns table, event bus, **no `style=` HTML attributes** |
| `landing-page.mdc` | Landing page sections, **screenshot showcase** (`.ss-tabs`, `.ss-panel`, `ssActivateTab`), screenshot file locations, update workflow |
| `dashboard-home.mdc` | Dashboard application list — **list/stats join `workflow_sessions` (hide workflow-failed)**, **funnel stats (Applied card + response rate formula)**, search/filter/sort, toasts (`notifyReady` **`c:`/`f:`** dedupe keys, `formatWorkflowFailureDetail`, duplicate headline), **`isPlaceholderCompanyName` / `displayCompanyNameOrUnknown` (Unknown employer label)**, card CSS, auth guard, **EXISTS pagination / Load More dedupe** |
| `ui-application-detail.mdc` | **10-tab** layout (Optimize CV + Practice Interview + Outreach), sub-tab patterns (Resume + Interview), cover letter card, **`ui/src/pages/cv-optimizer.ts`**, **`ui/src/pages/mock-interview.ts`**, **`ui/src/pages/hiring-outreach.ts`**, **`View posting` / `additional_locations`**, render functions, **`isPlaceholderCompanyName` / Unknown employer / About this opportunity**, company research notices (`research_quality` / staffing), all CSS classes |
| `accessibility.mdc` | WCAG 2.1 AA, heading hierarchy, landmarks, aria-label rules |
| `analytics-consent-onboarding.mdc` | PostHog, cookie consent API, onboarding tour |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eliornl/rolemule](https://github.com/eliornl/rolemule) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
