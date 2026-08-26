---
trigger: always_on
description: LibreFolio is a **self-hosted, open-source financial portfolio tracker** — alternative to Ghostfolio. It supports traditional assets (ETFs, stocks, bonds, crypto), multi-provider FX rates, technical analysis (EMA, MACD, RSI, Bollinger), and import from 11+ brokers.
---

# LibreFolio — Copilot Instructions

## What is LibreFolio

LibreFolio is a **self-hosted, open-source financial portfolio tracker** — alternative to Ghostfolio. It supports traditional assets (ETFs, stocks, bonds, crypto), multi-provider FX rates, technical analysis (EMA, MACD, RSI, Bollinger), and import from 11+ brokers.

## Tech Stack

| Layer | Technology | Details |
|-------|-----------|---------|
| **Backend** | Python 3.13 + FastAPI | SQLModel/SQLite, Alembic, Pipenv |
| **Frontend** | SvelteKit 2 + Svelte 5 | Tailwind CSS 4, ECharts 6, Zodios, Playwright |
| **Docs** | MkDocs Material | i18n (suffix strategy), Mermaid, LaTeX (KaTeX) |
| **Deploy** | Docker single-image | Backend serves frontend as static files |
| **CLI** | `dev.py` | Single entry point for everything |

## Project Structure

```text
LibreFolio/
├── backend/           # Python FastAPI (API, DB, services, providers)
├── frontend/          # SvelteKit SPA (routes, components, stores, i18n)
├── mkdocs_src/        # Documentation (MkDocs Material, i18n)
├── scripts/           # CLI tools (imported by dev.py)
├── dev.py             # Main CLI — ALWAYS use this, never manual commands
└── .github/           # Copilot instructions, skills
```

## Key Architectural Decisions

1. **All calculations in Backend** — the frontend is pure presentation
2. **FIFO at Runtime** — cost matching computed on-demand, not persisted
3. **Provider Registry Pattern** — auto-discovery for FX, Asset and BRIM providers with `params_schema` for dynamic forms
4. **Multi-Provider with Fallback** — FX rates from ECB→FED→BOE→SNB + MANUAL sentinel
5. **Svelte 5 Runes** — `$state`, `$derived`, `$effect` in new components
6. **Zodios API Client** — types from OpenAPI, Zod runtime validation
7. **Data Separation prod/test** — completely isolated folders
8. **Dual View** — card grid + DataTable for Assets and FX, toggle persisted in localStorage

## Fundamental Rules

- **Caveman mode ultra** — always active. Use `caveman` skill at `ultra` intensity. Max compression, abbrevs, arrows for causality (X → Y). Off only if user says "stop caveman" or "normal mode".
- **ALWAYS use `./dev.py`** for complex operations — never manual commands
- **Legacy support case-by-case** — tend to migrate to the correct design; keep an old path only with a concrete reason (e.g. not breaking released installs/APIs)
- **Code in English** — comments, docstrings, variables, README
- **Multilingual UI** — only the graphical interface in EN/IT/FR/ES
- **Edit > Rewrite** — prefer targeted edits to avoid regressions
- **Incremental Alembic migrations (released)** — ship schema changes as new migrations to protect existing installs; edit `001_initial.py` only for brand-new never-shipped tables; `db create-clean` only for fresh/test DBs
- **After modifying API** — run `./dev.py api sync` to regenerate TypeScript client
- **After modifying DB models** — add an incremental Alembic migration (`./dev.py db migrate "…"`); use `db create-clean` only for fresh/test DBs
- **NEVER run `git commit`** — the agent must only *propose* commit messages
  (e.g. write them to `/tmp/libreFolio_commit_*.txt` or print them inline).
  The user performs the actual commit manually. Staging (`git add`) and read-only
  git commands (`git status`, `git diff`, `git log --no-pager`) are allowed.
  Same rule for `git push`, `git reset --hard`, `git rebase`, `git checkout -- …`
  and any other history-mutating operation: **never** execute, only suggest.

## Terminal Command Rules

- **Long commands (>10 lines)** → do NOT paste them directly into the shell. Instead, write the command/script to a temporary file under `/tmp/` (e.g. `/tmp/libreFolio_<descr>.sh` or `.py`) and execute that file. This avoids quoting/escape issues and keeps the terminal log readable.
- **Truncated output (`tail`, `head`, `grep -m`, `| head -n`, etc.)** → always `tee` the full output to a file in `/tmp/` *before* truncating, so the complete log can be re-inspected without re-running the command. Pattern:
    ```bash
    <command> 2>&1 | tee /tmp/libreFolio_<descr>.log | tail -n 100
    ```
    Then, if more context is needed, read `/tmp/libreFolio_<descr>.log` instead of re-executing the command.
- **Rationale**: avoid re-running expensive commands (tests, builds, db operations) just to see output that was truncated earlier.

## Async I/O Rule (Event Loop Safety)

In `async def` handlers, **every sync library doing I/O** MUST be wrapped in `await asyncio.to_thread(...)`. Never call `requests.get()`, `yf.Ticker().info`, etc. directly — they block the entire event loop. If an endpoint only does light sync I/O (e.g. `Path.exists()`), define it as `def` (not `async def`).

## Frontend Conventions

- **Svelte 5 Runes**: use `$state()`, `$derived()`, `$effect()` — never old `$:` reactive
- **Tailwind CSS 4**: config via `@theme {}` in `app.css` — no `tailwind.config.ts`
- **Dark mode**: `html.dark` with Tailwind `dark:*` classes
- **Icons**: lucide-svelte
- **Selectors**: always use `data-testid` — never CSS classes or text (fragile with i18n)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Librefolio/LibreFolio](https://github.com/Librefolio/LibreFolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
