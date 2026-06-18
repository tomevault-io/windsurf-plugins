---
trigger: always_on
description: Agents working in this repository must treat [`PROJECT_BASIS.md`](PROJECT_BASIS.md) as a required long-lived constraints document.
---

# Repository Guidelines

## Instruction Priority

Agents working in this repository must treat [`PROJECT_BASIS.md`](PROJECT_BASIS.md) as a required long-lived constraints document.

- Follow `PROJECT_BASIS.md` for project goals, directory boundaries, documentation placement, coding standards, command entrypoints, and maintenance rules.
- If this file and `PROJECT_BASIS.md` overlap, apply the stricter rule.
- If a task-specific user instruction conflicts with `PROJECT_BASIS.md`, follow the user instruction for that task and keep other `PROJECT_BASIS.md` rules intact.

- Review [`dev-bitter-lesson.md`](dev-bitter-lesson.md) before debugging frontend deployment, browser/devtools tooling, multi-tenant permissions, or session-scoped config issues. It captures recurring high-cost mistakes and the corresponding fixed workflow.

## Project Structure & Module Organization

This repository's active product surface is the AINRF runtime plus WebUI, while the docs tree remains the long-lived product/reference knowledge base:

- `frontend/`: React + Vite WebUI for AINRF.
- `src/ainrf/`: Python package, CLI, backend API, and runtime code.
- `docs/`: Obsidian-style research notes and design docs. Key areas are `docs/framework/`, `docs/projects/`, and `docs/summary/`.
- `docs-site/`: Astro + Starlight product documentation site (deployed to GitHub Pages).
- `tests/`: CLI smoke tests for the Python package.
- `scripts/`: local build helpers.

Reference repositories live under `ref-repos/` and are treated as read-only research inputs.

## Project Overview

`scholar-agent` currently centers on the AINRF frontend/backend product surface. `src/ainrf/` and `frontend/` contain the active CLI, backend API, WebUI, and runtime capabilities, while `docs/`, `ref-repos/`, and the historical research notes remain long-lived knowledge and reference assets that support product design, implementation choices, and traceability. Notes continue to use Chinese content with English file slugs. Product documentation is built with Astro + Starlight in `docs-site/` and deployed to GitHub Pages.

## LLM Working Log

- `docs/LLM-Working/` is versioned working memory for plans, checklists, smoke notes, and agent-side implementation records.
- Daily work logs must live under `docs/LLM-Working/worklog/` using one file per day named `YYYY-MM-DD.md`.
- Before or during a work session, if today's file does not exist yet, create it first and keep appending to that same file for the rest of the day.
- The default unit is one changelog entry per completed modification plan or work slice, not one line per atomic edit/validation/commit action.
- Each changelog entry must record at least the time, the completed slice or plan label, the substantive change summary, and the validation outcome. If that slice produced commits, append the commit hash and subject in the same entry.
- Do not use the worklog as a transcript of commit subjects or atomic slice labels; summarize what the completed batch actually changed and verified.
- Treat the worklog as append-only session history. Do not silently rewrite earlier entries unless you are correcting an objective factual mistake.

## Build, Test, and Development Commands

- `cd docs-site && npm run dev`: start the docs site dev server with hot reload.
- `cd docs-site && npm run build`: build the static docs site for production.
- `cd docs-site && npm run preview`: preview the production build locally.
- `UV_CACHE_DIR=/tmp/uv-cache uv run ainrf --help`: inspect the CLI scaffold.
- `UV_CACHE_DIR=/tmp/uv-cache uv run pytest tests/ -n auto`: run the Python test suite in parallel across CPU cores via pytest-xdist (the `addopts` default is serial, so pass `-n auto` explicitly). Use `-n 0` or drop `-n` for serial execution when debugging an ordered failure.
- `UV_CACHE_DIR=/tmp/uv-cache uv run ruff check src tests`: run lint checks.
- `UV_CACHE_DIR=/tmp/uv-cache uv run ruff format --check src tests`: verify formatting.

### Build & Serve Shortcuts

- `cd docs-site && npm run build`: build the static docs site.
- `cd docs-site && npm run dev`: run the local docs dev server with hot reload.

Dependencies are managed by `uv`. Prefer `uv run ...` over manual venv activation so execution stays aligned with the lockfile.

### Frontend Command Constraints

- All frontend tooling lives under `frontend/`. Commands must either `cd frontend && ...` or use `npm --prefix frontend ...`.
- **Prefer `--prefix`**: `npm --prefix frontend run <script>` works regardless of current `pwd` and avoids the most common worktree mistake (running `npm` from the repo root).
- Frontend type-check: `npm --prefix frontend run build` wraps `tsc -b`. Do **not** invoke `tsc -b` directly from outside `frontend/`.
- Frontend tests: `npm --prefix frontend run test:run` (vitest runs test files in parallel by default; pass `--no-file-parallelism` for serial execution).
- Frontend lint: `npm --prefix frontend run lint`.
- Do **not** use `npx tsc --noEmit`, `npx tsc -p tsconfig.app.json`, or run plain `tsc` / `npx tsc -b` from the repo root — the latter may install an unrelated `tsc` npm package.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kozmosa/scholar-agent](https://github.com/Kozmosa/scholar-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
