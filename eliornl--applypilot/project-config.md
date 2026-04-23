---
trigger: always_on
description: **Product name:** ApplyPilot (repo folder: `applypilot`).
---

# ApplyPilot — Cursor Rules Index

**Product name:** ApplyPilot (repo folder: `applypilot`).

All detailed coding rules live in `.cursor/rules/`. This file is the index.

---

## Rule Files

| File | Covers |
|------|--------|
| `applypilot-core.mdc` | App name, error system (`APIError`/`ErrorCode`, `CFG_6xxx` no-API-key, **`RES_3002` duplicate workflow**, post-start duplicate / `uq_user_job_company`, content fingerprint NFKC + `utils/application_dedupe.py`), **`POST /workflow/start` job file uploads** (`.pdf`/`.txt`/`.docx`, 5 MB, `_JOB_FILE_MAGIC`), background task DB, **workflow failure (no partial outputs, applications list join)** |
| `python-conventions.mdc` | Python style, file sections, FastAPI endpoint template |
| `database-patterns.mdc` | SQLAlchemy JSONB, `flag_modified()`, soft delete, **partial indexes**, race conditions, migrations, `UserWorkflowPreferences` upsert |
| `auth-patterns.mdc` | JWT structure, `_make_jwt`, all auth endpoints, token revocation, lockout, timing attack fix |
| `security-python.mdc` | XSS, file upload MIME, `secrets` vs `random`, no-JWT-in-URL rule, user enumeration prevention |
| `security-middleware.mdc` | Middleware stack, CORS explicit headers, no wildcard origins, maintenance mode, **CSP nonce-based style-src** (no `'unsafe-inline'`), `.is-hidden` / CSS utility class reference |
| `settings-and-env.mdc` | All env vars, `get_settings()`, `ENCRYPTION_KEY` requirement, `.env` commit rules |
| `llm-integration.mdc` | Gemini client, BYOK, `user_facing_message_from_llm_exception()`, `asyncio.wait_for()` timeout, JSON parsing |
| `agent-patterns.mdc` | 5 workflow agents + 1 standalone interview prep + 6 career tool agents, **any workflow agent failure fails the run**, workflow_preferences injection, BYOK model override, **Company Research — `_has_usable_company_name` / unnamed-posting (dash-only placeholders)** |
| `interview-prep-feature.mdc` | Standalone agent, background task, Redis atomic lock, rate limit |
| `career-tools.mdc` | 6 tools, endpoints, rate limits, agent implementation pattern, output JSON schemas, copy button pattern, no-placeholder rules, UI CSS classes |
| `caching-redis.mdc` | Cache TTLs, helpers, rate limiting, auth-specific Redis keys |
| `websocket-patterns.mdc` | Endpoints, connection limits, broadcast helpers, WS-only JWT query-param exception |
| `logging-patterns.mdc` | StructuredLogger, request context, sensitive data redaction, `exc_info=True` requirement, bulk-script safety |
| `google-oauth.mdc` | 3-step OAuth flow, CSRF state in Redis, exchange-code pattern, open-redirect prevention |
| `email-and-misc-utils.mdc` | Gmail SMTP, resume parser, BYOK encryption |
| `frontend-js-strict.mdc` | JSDoc strict mode, null safety, anti-patterns table, event bus, **no `style=` HTML attributes** |
| `landing-page.mdc` | Landing page sections, **screenshot showcase** (`.ss-tabs`, `.ss-panel`, `ssActivateTab`), screenshot file locations, update workflow |
| `dashboard-home.mdc` | Dashboard application list — **list/stats join `workflow_sessions` (hide workflow-failed)**, search/filter/sort, toasts (`notifyReady` **`c:`/`f:`** dedupe keys, `formatWorkflowFailureDetail`, duplicate headline), **`isPlaceholderCompanyName` / `displayCompanyNameOrUnknown` (Unknown employer label)**, card CSS, auth guard |
| `ui-application-detail.mdc` | 8-tab layout, sub-tab patterns (Resume + Interview), cover letter card, render functions, **`isPlaceholderCompanyName` / Unknown employer / About this opportunity**, all CSS classes |
| `accessibility.mdc` | WCAG 2.1 AA, heading hierarchy, landmarks, aria-label rules |
| `analytics-consent-onboarding.mdc` | PostHog, cookie consent API, onboarding tour |
| `chrome-extension.mdc` | Extension structure, **dev/prod `IS_DEV` URL toggle**, popup design system, icon generation, landing-page mockup sync, **`extractPageContent` split-view / detail-pane root** (`popup.js` + `service-worker.js`), `InputMethod.EXTENSION` |
| `adding-new-features.mdc` | Checklists: new endpoint, new agent, new tool, new migration, new static asset, **new preference** |
| `settings-page.mdc` | Settings tabs, Preferences tab, AI Setup tab, auto-save pattern, custom slider/toggle/dropdown, **account-icon color variants**, self-hosted Font Awesome |
| `frontend-build-pipeline.mdc` | esbuild pipeline, `asset_url()`, adding new JS/CSS files, manifest, CI/Docker build, `make build-frontend` (macOS quarantine fix) |
| `mobile-responsive.mdc` | Breakpoints, navbar collapse (`navbar-expand-xl` landing / `navbar-expand-lg` dashboard), scrollable tab bars, `app.css` global mobile utilities, per-page patterns |
| `unit-testing.mdc` | 195 agent unit tests + 96 API integration tests (`tests/test_api/`); NullPool engine; `authed_client` vs `authed_client_with_user`; LLM mock pattern; rate-limit 429 override; status code quirks (422 vs 400); `asyncio_default_fixture_loop_scope=session` |
| `e2e-testing.mdc` | 24 Playwright spec files (~1,450 tests); tier system (mocked CI / live-server / static); `setupAuth` + `page.route()` patterns; JWT/cookie-consent setup; SPA state waits; data shape pitfalls; redirect chain assertions |

---

## Non-Negotiables (always true, no exceptions)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eliornl/applypilot](https://github.com/eliornl/applypilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
