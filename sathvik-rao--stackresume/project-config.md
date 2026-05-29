---
trigger: always_on
description: handles fenced blocks, trailing commas, and preamble text robustly.
---

# AGENTS.md

Guidance for AI coding assistants (Claude Code, Cursor, Codex, Aider, Copilot,
Gemini, Windsurf, …) working on **StackResume**.

This file follows the open [agents.md](https://agents.md) spec — most modern
coding agents read it automatically. For agents that prefer their own filename,
create symlinks instead of duplicating content (see the bottom of this file).

- **Humans** → read [`CONTRIBUTING.md`](CONTRIBUTING.md) first.
- **AI agents** → read `CONTRIBUTING.md` *and* the rules below.

---

## Project shape

| Layer | Where | What |
|---|---|---|
| Backend | `backend/app/` | FastAPI + LangGraph + async SQLAlchemy/SQLite |
| Frontend | `frontend/` | Vanilla HTML/CSS/JS — single `index.html`, no build step, no npm |
| Tests | `backend/tests/` | pytest, mirrors `app/` 1:1; runs fully offline |
| Pipeline | `backend/app/agents/graph/` | Multi-agent LangGraph; one file per node under `nodes/` |
| Docs | `README.md`, `CONTRIBUTING.md`, `backend/tests/README.md` | Authoritative |

Read `CONTRIBUTING.md` for the test ↔ code mapping and branch/PR conventions.

---

## Hard rules

1. **No new runtime dependencies** in `backend/requirements.txt` without a
   one-line justification in the PR. The prod image ships this file — keep it lean.
2. **Tests are mandatory.** Every change to `backend/app/**.py` needs a
   matching test under `backend/tests/**.py`. The full suite must pass:
   `cd backend && pytest` (~30 s, offline).
3. **LLMs in tests = the `fake_llm` fixture.** Never call a real provider.
   See `backend/tests/fixtures/llm_fakes.py`. Drive responses with
   `fake_llm.set("agent_key", payload)`; force errors with `fake_llm.fail_with(...)`.
4. **Don't break the offline guarantee.** `backend/tests/conftest.py` scrubs
   `OPENAI_API_KEY` / `ANTHROPIC_API_KEY` / `GOOGLE_API_KEY` / `LANGSMITH_API_KEY`
   at import time. Don't read live env vars in tests.
5. **No new `*.md` docs** unless explicitly requested. Don't auto-generate
   READMEs, CHANGELOGs, design docs, or planning notes.
6. **No emojis in code or comments** unless the user explicitly asks. UI text
   (sidebar labels, toasts, agent step descriptions) is the only exception.
7. **No `--no-verify`, `--force-push`, or destructive `git reset --hard`**
   without explicit user approval — even if the user pre-approved a similar
   action earlier in the session.

---

## Code style — match what's already here

- **Comments explain *why*, not *what*.** The identifiers already say what.
  Write a comment only when the *why* is non-obvious: a hidden constraint, a
  subtle invariant, a workaround for a specific bug, behavior that would
  surprise a future reader.
- **Python:** explicit `if`/`elif`, no clever metaprogramming. Type hints on
  signatures where they help readers. Keep pure functions pure; isolate I/O.
- **One concern per file.** When a module passes ~250 lines, split it the way
  this codebase already does (see "Splitting patterns" below).
- **Validate at boundaries only** (HTTP request bodies, file uploads, JSON
  parsed back from the LLM). Internal callers don't need defensive checks
  for shapes the type system already guarantees.
- **Names match the existing vocabulary.** `_extract_json`, `_strip_dashes`,
  `_call_llm_timed`, `_safe_score`, "Intent Guard", "Resume Generator",
  "JD Tailoring", `sessionProcessing`, `inFlightSends`. Don't rename
  established concepts.
- **Finish what you start.** No TODO comments for the next session, no
  half-wired feature flags, no dead branches.
- **Three similar lines is better than a premature abstraction.** Wait for a
  real third use case before extracting a helper.

---

## Splitting / decoupling patterns the codebase already uses

When a file you're editing has grown too big, the accepted moves are:

- **Per-feature router files** — `app/api/sessions_routes.py`,
  `messages_routes.py`, `document_routes.py`, … (not one giant `routes.py`).
  Shared internals live in `app/api/_pipeline.py`. The underscore prefix
  signals *package-internal*.
- **Per-node agent files** — `app/agents/graph/nodes/parse.py`,
  `generate.py`, `review.py`, … Re-exported from `app/agents/graph/__init__.py`
  so importers keep using `from app.agents.graph import generate_resume_node`.
- **Per-prompt files** — `app/agents/prompts/<role>.py`, re-exported from
  the prompts `__init__.py`.
- **Shared graph helpers under `_*.py`** — `_helpers.py`, `_intent.py`,
  `_routing.py`, `_builder.py`. Underscore = "not part of the public API surface."

Follow the same shape when you create new modules.

---

## Architectural patterns to follow

- **Two-tier settings.** `.env` baseline → `app_settings` DB overlay
  (`app/runtime_settings.py`). To add a new tunable:
  1. Add the field to `Settings` in `app/config.py` (with a default).
  2. Add the column to `AppSettings` in `app/models.py`.
  3. Append an `ALTER TABLE` entry to `_MIGRATIONS` in `app/database.py`.
  4. Add the field to `AppSettingsUpsert` in `app/api/settings_routes.py`.
  5. Surface it in the Settings UI (`frontend/js/section-prefs.js` /
     `settings.js` / `keys.js`).
- **LangGraph nodes** have the signature `def fn(state: AgentState) -> AgentState`.
  Append a `_trace_event(...)` for every meaningful step. Wrap LLM calls

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sathvik-Rao/StackResume](https://github.com/Sathvik-Rao/StackResume) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
