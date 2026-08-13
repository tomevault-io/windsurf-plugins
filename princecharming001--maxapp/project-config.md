---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repo shape

Monorepo for **Max**, a looksmaxxing coaching app.

- `backend/` — FastAPI API (Python **3.12**), the main server. Postgres via Supabase. (The root `README.md` is stale: it says "FastAPI + MongoDB / Python 3.8" — neither is true.)
- `mobile/` — Expo / React Native app (Expo SDK 54, RN 0.81), iOS-first, ships to TestFlight via EAS.
- `cannon_facial_analysis/` — a **separate** FastAPI + MediaPipe service computing face-scan metrics. The backend calls it over HTTP (`settings.facial_analysis_api_url`); it is not imported.
- `web/` — a few static/Stripe-embedded pages. `docs/`, `legal/`, `data/`, `rds_templates/` — assets/docs.

## Commands

### Backend
```bash
cd backend
# CRITICAL: Python 3.14 breaks langchain/pydantic at import. Use 3.12.
# A prebuilt venv exists at backend/.venv312 (create with: python3.12 -m venv .venv312 && .venv312/bin/pip install -r requirements.txt)
.venv312/bin/python -m uvicorn main:app --host 0.0.0.0 --port 8000   # add --reload for dev
.venv312/bin/pytest                                                  # all tests
.venv312/bin/pytest tests/test_chat_routing.py::test_name -q         # one test
```
- The local backend connects to the **production Supabase DB** — local test users/data land in prod.
- For reliable local chat, pin the LLM: prefix uvicorn with `LLM_PROVIDER=openai` (the default `gemini` has no local key and the failover hits a Claude timeout, returning the "trouble reaching my brain" fallback).
- Prod start (Render, see `backend/Dockerfile`): `uvicorn main:app --host 0.0.0.0 --port ${PORT}`.

### Mobile
```bash
cd mobile
npx expo start --lan          # or: npm run start:clear
# compile-check one file without a full bundle:
node -e "require('@babel/core').transformFileSync('screens/x.tsx', {presets:['babel-preset-expo']})"
```
- No JS unit-test runner. UI tests are **Maestro** flows in `mobile/maestro/*.yaml`.
- iOS builds: `eas build --platform ios --profile production --auto-submit`. `buildNumber` lives in `app.json`. The production API URL for **builds** comes from `eas.json` (builds ignore the `.env*` files).
- **Local dev API override lives in `mobile/.env.development.local`** (gitignored; loaded ONLY when `NODE_ENV=development`, i.e. `expo start`) — point it at the Mac's **LAN IP** for a real device, or `localhost` for the simulator. **NEVER use `mobile/.env.local`** for this: Expo loads that in *every* environment including production exports, so it leaks into OTA bundles and points every phone at localhost (see the OTA notes under Deploy & ops).

## Backend architecture

- **Entry** `backend/main.py` registers ~23 routers under `/api` (auth, users, scans, payments, courses, chat, schedules, maxes, marketplace, personalization, achievements, …). A global exception handler redacts internals in production and maps DB-connectivity errors to a clean 503.
- **DB** `backend/db/sqlalchemy.py` — SQLAlchemy async + asyncpg against Supabase. Production **must** use the transaction pooler: host `aws-…pooler.supabase.com`, `SUPABASE_DB_PORT=6543`, user `postgres.<project-ref>`. **Gotcha:** never send `search_path` as an asyncpg startup parameter through the pooler — Supavisor closes the connection on the first query ("connection was closed in the middle of operation"). The DB role already defaults `search_path` to `public, extensions`. `/health` returns `{build, db}`; boot logs print `[DB] mode=transaction|session|direct`.
- **Config** `backend/config.py` — pydantic settings, all from env. `is_production` hard-gates dev-only endpoints (faux-signup, dev Google, test-activate) so they can't mint paid accounts in prod.
- **LLM** `backend/services/lc_providers.py` — multi-provider with failover, selected by `LLM_PROVIDER` (`huggingface` fine-tuned default, `gemini`, `openai`, `mistral`). `backend/services/claude_service.py` is Anthropic single-shot (used for task guides). No provider SDKs are imported outside these two files.
- **Chat agent** `backend/services/lc_agent.py` — a LangChain tool-calling `AgentExecutor` (~22 tools: schedule CRUD, `recommend_product`, `search_knowledge`, `web_search`, `remember_about_user`). `build_agent_system_prompt` assembles the prompt: persona (`services/prompt_constants.MAX_CHAT_SYSTEM_PROMPT`) + appended voice/MCQ/product-link rules + injected `KNOWN PROFILE` (user facts). `backend/api/chat.py` routes a turn through a **fast RAG path** (`answer_from_rag`, early-returns) OR the **full agent** (`run_chat_agent`) — they are mutually exclusive. MCQ markers `[CHOICES]a|b|c[/CHOICES]` / `[CHOICES_MULTI]` are emitted by the model and parsed out into a `choices` array in `api/chat.py`.
- **RAG / maxxes** the five programs are `skinmax`, `hairmax`, `fitmax`, `heightmax`, `bonemax`; their coaching docs live in `backend/rag_content/<maxx>/`.
- **Personalization** `services/personalization.py` (unified brief) + `services/user_facts_service.py` (the persisted `KNOWN PROFILE` blob) — injected into the chat prompt; this is the "never re-ask known facts" mechanism.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [princecharming001/maxapp](https://github.com/princecharming001/maxapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
