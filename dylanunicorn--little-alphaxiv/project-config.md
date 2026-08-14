---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Little Alphaxiv — a self-hosted, alphaxiv-style arXiv paper-reading app. Chat with an LLM to discover papers (general chat); click a result and the PDF opens with a paper-aware assistant (paper view). Bring-your-own OpenAI-compatible API key.

User data (chat history, PDF annotations, provider config, settings) lives in a **server-side SQLite database**, scoped per-user via httpOnly session-cookie auth. The backend is no longer a stateless proxy — it owns the DB and authenticates users — but it still proxies arXiv / LLM gateways / PDFs (those send no CORS headers). The plaintext LLM API key is stored server-side **Fernet-encrypted at rest**; the browser only ever sends a `provider_id`.

Not a monorepo: `frontend/` and `backend/` are independent — run both manually.

## Autonomy

Act on your own judgment — don't checkpoint every change. If you spot a bug or have a clear idea for a feature or improvement, just fix or implement it; don't stop to ask for permission first. Pick a reasonable approach, do the work in a worktree (see Workflow below), then report what you changed and why. Only pause to ask when a decision is genuinely irreversible or outward-facing and you can't infer the right call from the code, design docs, or existing conventions.

## Workflow (always work in a fresh worktree → PR)

For every task, start a **new worktree** under `.claude/worktrees/` and do all edits, testing, and iteration there — never work directly on `main`. `main` is **protected**: it can only be updated via a passing pull request (CI: frontend `typecheck` + `vitest`, backend `pytest`). No approving reviews are required (early-stage project) and commits need not be signed — but the **CI checks must pass** before a PR can merge.

When the change is ready: push the worktree branch to `origin`, open a PR, wait for CI to go green, then merge via GitHub (merge commit is fine — `Merge worktree: ...`-style history is preserved). If another agent's PR is mid-merge, wait a bit and retry instead of racing it; if it conflicts with your branch, rebase onto the latest `main` and resolve. After the merge lands, pull `main` locally, delete the worktree (remove the `node_modules` junction first — see "Working in worktrees" below). Never `git push --force` to `main` (blocked) and never delete it (blocked).

## Commands

### Frontend (`cd frontend`)
- `npm run dev` — Vite dev server on `:5173` (proxies `/api/*` → `http://127.0.0.1:8000`)
- `npm run typecheck` — `tsc --noEmit` (the type gate; **there is no lint script** — typecheck is the gate)
- `npm run build` — `tsc --noEmit && vite build`
- `npm test` — Vitest (`vitest run`); `npm run test:watch` to watch
- One test file: `npx vitest run src/lib/dates.test.ts`
- One test by name: `npx vitest run -t "keeps input order"`

### Backend (`cd backend`)
- `./run.sh` — activates the `Agent_env` conda env if present, installs deps if missing, runs `uvicorn app.main:app --host 127.0.0.1 --port 8000 --reload`
- Windows CMD: `run.bat` — same as `./run.sh` but native to CMD. **On Windows, do NOT use `bash run.sh`**: `bash` often resolves to WSL, whose Python 3.8 can't parse the backend's `str | None` syntax (needs Python 3.10+). `run.bat` uses the Windows conda `Agent_env` (Python 3.10).
- Manual: `pip install -r requirements.txt && uvicorn app.main:app --host 127.0.0.1 --port 8000 --reload`
- **Migrations run automatically** on startup: the `lifespan` in `main.py` runs `alembic upgrade head`. There is no separate `migrate` step; `run.sh`/`run.bat` are unchanged. To create a new migration after editing `app/models.py`: `alembic revision --autogenerate -m "..."` (run from `backend/`).
- **First run** auto-creates `deploy/data/little_alphaxiv.db` and persists `LAX_SECRET_KEY` to `deploy/data/.lax_secret_key` (keep it; losing it orphans all encrypted keys + sessions). All runtime data — DB, PDF cache (`deploy/data/pdf_cache`), secret key, reset-link log — lives in one dir (`deploy/data/` for both local dev AND Docker; Docker bind-mounts it to `/app/data`). `run.sh`/`run.bat` auto-set `LAX_DATABASE_URL` + `LAX_PDF_CACHE` to `../deploy/data/` so native dev and the container share one data dir (no fork). On startup, `app/paths.py` `migrate_legacy_paths()` moves any pre-consolidation backend-root files (`little_alphaxiv.db`, `lax_reset_links.log`) into the default data dir, and `security._ensure_secret_key()` migrates the key from `backend/.env`; both are no-ops when `LAX_DATABASE_URL` is set explicitly (tests, Docker, or run-scripts). To carry over old `backend/data/` data into the new shared location, copy it manually: `cp -r backend/data/* deploy/data/`.
- **Backend tests (pytest):** `cd backend && python -m pytest` (run in `Agent_env`). Tests live in `backend/tests/`; `conftest.py` builds a per-test temp SQLite + runs the real app lifespan. The password-recovery feature is security-sensitive and is the reason backend tests exist — keep them green.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DylanUnicorn/little-alphaxiv](https://github.com/DylanUnicorn/little-alphaxiv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
