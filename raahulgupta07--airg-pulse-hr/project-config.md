---
trigger: always_on
description: Other agents (Claude, Cursor, Copilot, devs) read this file first. Skim takes 30s. Useful work in 5min.
---

# AGENTS.md — Read this before touching code

Other agents (Claude, Cursor, Copilot, devs) read this file first. Skim takes 30s. Useful work in 5min.

---

## What this app is (one line)

**PULSE — multi-tenant HR/talent/comms platform.** FastAPI + SvelteKit 5 + PG18+pgvector. Docker Compose local, ECS Fargate prod-ready.

**Acronym:** People · Updates · Lifecycle · Sourcing · Engagement.

**Brand:** brutalist/industrial. Dark `#383832` ink, yellow `#feffd6` surface, green `#00fc40` CTA, zero border-radius, hard stamp shadows.

---

## Read these in order (priority)

1. `AGENTS.md` (this file) — orientation
2. `ARCHITECTURE.md` — one-page diagram + data flows
3. `CLAUDE.md` — full feature map, env vars, security, debug recipes
4. `compose.yaml` — runtime topology
5. `backend/main.py` — entry, middleware, health
6. `backend/core/` — DB, auth, OCR, LLM, settings, migrations
7. `backend/routes/` — 22 route modules (one per feature)
8. `frontend/src/routes/` — page-per-folder
9. `db/migrations/` — additive-only SQL (sha256-tracked)
10. `tests/` — pytest

If you read 1–4 and the relevant route file, you can ship a feature.

---

## Hard rules (do not break)

- **Migrations are ADDITIVE ONLY.** Guard in `backend/core/migrations.py` blocks DROP COLUMN, DROP TABLE, RENAME COLUMN, ALTER COLUMN TYPE, TRUNCATE. Override flag exists (`ALLOW_DESTRUCTIVE_MIGRATIONS=1`) — never use without explicit user approval.
- **Secrets NEVER in code.** `.env` only. Bcrypt hash on disk, not plaintext.
- **All `{@html}` sinks must escape user input first** (`mdLite`, `renderNoteWithMentions` already do).
- **LLM calls go through `llm_call()`** in `backend/core/config.py` (timeout + cost cap + LLM_GATE semaphore). Never call OpenAI client directly.
- **File access via signed URLs** (`/candidates/{id}/file/sign`) — not bare `?token=`.
- **Auth:** JWT-first via `validate_token`, legacy hex fallback for old tokens. Both live side-by-side.
- **Frontend Svelte 5 Runes only** (`$state`, `$effect`, `$derived`). Never legacy stores or `export let`.
- **Brutalist design only.** Zero border-radius (forced via `* { border-radius: 0 !important }`). Ink borders 2px, stamp shadows `4px 4px 0`, no blur.

---

## How to run locally

```bash
cp .env.example .env   # already populated for dev
docker compose up -d --build
# wait ~6s
open http://localhost:8090/login
# login: pulse_admin / PulseAdmin#2026!
```

Health: `curl http://localhost:8090/api/health`

---

## How to add a feature without breaking data

1. New SQL migration in `db/migrations/NNN_<name>.sql` — **additive only** (CREATE TABLE / ADD COLUMN / CREATE INDEX). Auto-runs on next boot.
2. Route handler in `backend/routes/<area>.py` — register router in `backend/main.py`.
3. Frontend page in `frontend/src/routes/<area>/+page.svelte`.
4. Test in `tests/test_<area>.py`.
5. `docker compose up -d --build api`.

---

## How to NOT break data on upgrade

```bash
./scripts/pre_upgrade.sh                    # snapshot DB + /data/cvs
git pull
docker compose up -d --build
# verify
curl -s http://localhost:8090/api/health | head -c 200
# rollback if broken:
./scripts/restore.sh <dump-file> --confirm
```

Backup sidecar already runs daily (`pg_dump`, 14d retention, `data/backups/`). Symlink `latest.dump.gz` always points to newest.

---

## Concurrency tuning (current: 10 users)

All in `.env`:

| Var | Value | Why |
|-----|-------|-----|
| `WORKERS` | 4 | uvicorn worker processes |
| `DB_POOL_MIN` | 5 | per-worker min |
| `DB_POOL_MAX` | 20 | per-worker max (4×20=80 < 100 PG cap) |
| `OCR_CONCURRENCY` | 2 | PaddleOCR parallel jobs (RAM cap) |
| `LLM_MAX_CONCURRENT` | 8 | OpenRouter calls per worker |
| `LLM_DAILY_CAP_USD` | 200 | per-tenant daily $ cap |
| `RATE_LIMIT` | 600/minute | global default; per-route stricter |
| `OCR_UPSCALE_MIN_PX` | 2000 | low-res image upscale target before OCR |
| `OCR_UPSCALE_MAX_PX` | 4000 | upscale ceiling |
| `REDIS_URL` | `redis://redis:6379/0` | embed (24h) + tool-result (60s) cache; in-mem fallback if down |
| `AGENT_V2` | `false` | master flag for Chat v2 (Agno agent). False = legacy keyword chat |
| `AGNO_MAX_STEPS` | 8 | max tool-loop iterations per agent turn |
| `AGNO_TOOL_TIMEOUT_S` | 15 | per-tool timeout |
| `AGNO_MEMORY_TOPK` | 5 | pgvector recall hits injected per turn |
| `AGENT_SESSION_CAP_USD` | 1 | per-session $ ceiling (read from `agent_runs.cost_usd`) |

To scale to 50 users: add pgbouncer + Redis cache + ECS multi-task. Terraform modules already exist in `deploy/aws/terraform/`.

---

## LLM models (via OpenRouter)

| Slot | Model | Use |
|------|-------|-----|
| `CHAT_MODEL` | `google/gemini-3-flash-preview` | chat, vision, structure step |
| `DEEP_MODEL` | `openai/gpt-5.4-mini` | complex reasoning |
| `LITE_MODEL` | `google/gemini-3.1-flash-lite-preview` | classify, enrich, quality, tag |
| `VERIFIER_MODEL` | `anthropic/claude-opus-4.7` | digit-precision verifier (Step 2.5) |
| Embeddings | `gemini-embedding-2-preview` | 1536-dim |

All overridable via env. Set in `backend/core/config.py`.

---

## Where features live (cheat sheet)

| Feature | File |
|---------|------|
| Login UI | `frontend/src/routes/login/+page.svelte` |
| Auth backend | `backend/routes/auth.py` + `backend/core/jwt_auth.py` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [raahulgupta07/airg-pulse-hr](https://github.com/raahulgupta07/airg-pulse-hr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
