---
trigger: always_on
description: **v1.8.17 — Subscription conversion funnel polish.** Phase 1 (platform), Phase 1+ (resources hub), Phase 2 (TTS/STT), Phase 3 (voice conversation), Phase 4 (multi-language support), Phase 5 (Stripe subscriptions), Phase 6 (Listening exercises), Phase 7 (Reading exercises), Phase 8 (Feedback board), Phase 9 (LLM Memory), Phase 10 (Multi-Language), and Phase 11 (User Reviews) are complete. Japanese (`ja-JP`), Korean (`ko-KR`), and Mainland Chinese (`zh-CN`) now have backend curriculum, grammar, vo
---

# AGENTS.md — FreeLingo

## Project state

**v1.8.17 — Subscription conversion funnel polish.** Phase 1 (platform), Phase 1+ (resources hub), Phase 2 (TTS/STT), Phase 3 (voice conversation), Phase 4 (multi-language support), Phase 5 (Stripe subscriptions), Phase 6 (Listening exercises), Phase 7 (Reading exercises), Phase 8 (Feedback board), Phase 9 (LLM Memory), Phase 10 (Multi-Language), and Phase 11 (User Reviews) are complete. Japanese (`ja-JP`), Korean (`ko-KR`), and Mainland Chinese (`zh-CN`) now have backend curriculum, grammar, vocabulary, phrasebook, and assessment data. Static grammar, phrasebook, vocabulary resources, lessons, newly generated lesson exercises, exercise hints, and newly generated lesson vocabulary include native-language learning support. Email verification and password reset are also included. Voice conversations are persisted as text transcripts alongside chat conversations. The AI tutor persona is named Lingu. The repo contains `backend/`, `frontend/`, `docker-compose.yml`, `.env.example`, and CI/CD via GitHub Actions. See [CHANGELOG.md](CHANGELOG.md) for the full version history.

## Architecture at a glance

Monorepo: `backend/` (Python 3.14 FastAPI) + `frontend/` (Next.js 16 App Router) deployed via Docker Compose with PostgreSQL 16 and Redis 7. The backend proxies all external services (Ollama, Kokoro, Whisper) — the frontend never calls them directly.

## Key constraints

- **Users can learn multiple languages simultaneously** — each language gets an isolated study plan, progress, flashcards, conversations, and competencies. Supported target languages: `en-US`, `en-GB`, `es-ES`, `it-IT`, `pt-PT`, `de-DE`, `fr-FR`, `ja-JP`, `ko-KR`, `zh-CN`. User's native language (asked at registration) is used for flashcard translations, tutor feedback, lesson and exercise `native_explanation` content, and cached native-language help in static grammar, phrasebook, and vocabulary resources.
- **User settings and memories are global (per user), not per language.** Profile (avatar, bio, display name, email, password, native language, UI locale), conversation limits (max duration, inactivity timeout, daily/weekly minutes, weekly sessions), token quota, subscription, and LLM memories are stored on the `users` table or keyed by `user_id` only — they do not change when switching the active study language. The `study_plan_id` column on `memories` tags each memory with the plan it was created in, but the settings page (`GET /api/memories`) lists all memories regardless of language. In chat/conversation context, memories are optionally filtered by `study_plan_id` to inject language-relevant context into the LLM prompt.
- **First registered user becomes admin automatically** when `FIRST_USER_IS_ADMIN=true` (default).
- **Registration gating**: `ALLOW_REGISTRATION=false` blocks public signups; admin creates users or generates single-use invite links (48h expiry in Redis).
- **Ollama should run on the host for GPU access**, accessed via `host.docker.internal:11434`. On Linux, the backend service needs `extra_hosts: ["host.docker.internal:host-gateway"]`.
- **Default target language is `en-GB`** — all fallback defaults across backend (service params, Query params, model column defaults, chat context, onboarding form) and frontend (`DEFAULT_TARGET_LANGUAGE` in `target-languages.ts`) use `en-GB`. `en-US` remains a supported language but is never used as a fallback default.

## Documentation maintenance (MANDATORY)

**Any code change that affects behaviour, models, endpoints, configuration, or dependencies MUST be followed by an update to all affected spec and MD files.**

Rules that apply without exception:

1. **Proactively identify affected docs.** After every implementation change, review which of the files below are impacted and list them explicitly before closing the task.
2. **Always inform and ask for confirmation.** Before updating any spec or MD file, state exactly what will change and wait for explicit user approval. Never silently update documentation.
3. **No task is complete without docs in sync.** A feature or fix is considered unfinished if the relevant spec files, `README.md`, `AGENTS.md`, or `CHANGELOG.md` have not been updated (or the user has explicitly opted out).
4. **Version and changelog.** Any user-visible change must be reflected in `CHANGELOG.md` and `specs/version.md` (with a version bump if warranted).

Files most commonly affected by code changes:

- **New/modified endpoint** — `specs/api-endpoints.instructions.md`, `specs/rate-limiting.instructions.md`
- **New/modified model or migration** — `specs/database-models.instructions.md`, `specs/architecture-backend.instructions.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArtCC/freelingo](https://github.com/ArtCC/freelingo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
