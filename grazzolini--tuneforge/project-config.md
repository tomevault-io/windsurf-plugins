---
trigger: always_on
description: Guidance for AI coding agents (Claude Code, Cursor, Copilot, Aider, Codex, etc.) working in this repository.
---

# AGENTS.md

Guidance for AI coding agents (Claude Code, Cursor, Copilot, Aider, Codex, etc.) working in this repository.

This file is the agent-facing companion to [CONTRIBUTING.md](./CONTRIBUTING.md). Humans should read CONTRIBUTING.md; agents should read both, but this file takes precedence on conventions specific to automated work.

## How to Read AGENTS Files in This Repo

- This root file applies to the whole repository unless a deeper `AGENTS.md` exists.
- Always look for a more-specific `AGENTS.md` in the directory tree you are editing.
- If rules conflict, prefer the most deeply nested file for that path.

## Project Snapshot

Tuneforge is a local-first desktop app for musicians learning songs: stem separation, chord/key/tempo detection, pitch shift, retune, export. No cloud, no account.

- **Monorepo**: pnpm workspace.
- **Backend**: `apps/backend` — FastAPI + SQLAlchemy 2 + Pydantic v2, Python 3.11, managed with `uv`. SQLite persistence, single-process job runner, audio engines (Demucs, FFmpeg, librosa-style analysis).
- **Desktop**: `apps/desktop` — Tauri 2 (Rust) shell + React/Vite/TypeScript frontend, Vitest + Testing Library.
- **Shared types**: `packages/shared-types` — TypeScript types generated from the backend OpenAPI schema. **Always regenerate after backend route/schema changes.** The JSON export is local generator output; the generated TypeScript contract is committed.

## Hard Rules

These are non-negotiable. If a task seems to require breaking one, stop and ask.

1. **Local-only stays local.** The backend binds `127.0.0.1`. Do not introduce network exposure, public binds, reverse-proxy assumptions, multi-user concepts, auth/session systems, telemetry, analytics, or external API calls (other than the Demucs model download that already exists).
2. **No cloud, no accounts.** The app must keep working with no internet after first run.
3. **Don't bundle FFmpeg.** It is a host-installed dependency by design. See [THIRD_PARTY_NOTICES.md](./THIRD_PARTY_NOTICES.md) for the licensing reason.
4. **Respect the layering.** `routes/` → `services/` → `engines/`. Routes are thin; business logic lives in services; raw audio/ML work lives in engines. Don't bypass layers.
5. **Don't commit generated files by hand.** Run the generator (see "Generated artifacts" below).
6. **Don't disable lint/type/test rules to make CI pass.** Fix the underlying issue.
7. **Don't bypass safety flags.** No `--no-verify`, no `git push --force` on shared branches, no destructive shell shortcuts.
8. **MIT-compatible deps only.** Avoid GPL/AGPL/SSPL runtime dependencies. Note any new dep's license in [THIRD_PARTY_NOTICES.md](./THIRD_PARTY_NOTICES.md).

## Repository Layout

```
apps/
  backend/                FastAPI service
    app/
      api/routes/         HTTP handlers (thin)
      services/           orchestration, persistence, caching
      engines/            pure compute: analysis, chords, stems, transform
      models.py           SQLAlchemy ORM
      schemas.py          Pydantic request/response
      errors.py           AppError + handlers
      config.py           env-driven Settings
    alembic/versions/     migrations (auto-run on startup)
    tests/                pytest suite
  desktop/
    src/                  React frontend (Vitest)
    src-tauri/            Rust shell (cargo)
packages/
  shared-types/           generated TS contracts
docs/                     product + architecture docs
scripts/                  packaging helpers
```

## Workflow Expectations

When asked to implement a change:

1. **Read before writing.** Inspect the surrounding files in the relevant layer. Match existing patterns.
2. **Plan briefly.** For multi-step work, write a short plan or todo list before editing.
3. **Edit narrowly.** Don't reformat unrelated code, don't add docstrings/comments to code you didn't touch, don't introduce new abstractions for one-time operations.
4. **Run the gates locally.** From the workspace root:
   ```sh
   pnpm lint
   pnpm typecheck
   pnpm test
   ```
   And for the backend:
   ```sh
   cd apps/backend && uv run --python 3.11 pytest
   ```
5. **Regenerate contracts if backend HTTP surface changed:**
   ```sh
   pnpm contracts:generate
   ```
   Commit `packages/shared-types/src/generated/openapi.ts` if it changes. Keep `packages/shared-types/openapi.json` ignored and local. CI fails on TypeScript contract drift.
6. **Verify Tauri compiles** if you touched anything in `apps/desktop/src-tauri/`:
   ```sh
   cd apps/desktop/src-tauri && cargo check
   ```
7. **Update docs only when behavior changes.** Don't create new markdown files to describe what you did. The PR description is the right place for that.

## Code Conventions

### Python (backend)

- Python 3.11. Use `uv run --python 3.11 ...` for everything; do not invoke a globally installed Python.
- Type hints on all new function signatures. `mypy app` must pass.
- `ruff check .` must pass. Fix lints, do not silence them.
- Pydantic v2 syntax (`model_config`, `Field(...)`, `model_validate`).
- SQLAlchemy 2 declarative + `Mapped[...]` style.
- Raise `AppError` (see `app/errors.py`) for user-facing failures, not bare `HTTPException`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grazzolini/tuneforge](https://github.com/grazzolini/tuneforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
