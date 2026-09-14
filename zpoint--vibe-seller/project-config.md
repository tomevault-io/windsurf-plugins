---
trigger: always_on
description: > **For full documentation**: See [docs/dev-guide.md](docs/dev-guide.md)
---

# Vibe Seller - Claude Code Context

> **For full documentation**: See [docs/dev-guide.md](docs/dev-guide.md)
> (user-facing landing page is [README.md](README.md))

## What & Why

Team collaboration platform for e-commerce store automation. Users create browser-automation tasks organized by store, executed via browser-use CLI (Chrome/Ziniao) with real-time SSE streaming.

## Tech Stack

- **Backend**: Python 3.11+ / FastAPI / SQLAlchemy 2.0 async / SQLite (`~/.vibe-seller/data/vibe_seller.db`) / SSE
- **Frontend**: React 19 / TypeScript / Vite / Tailwind CSS 4 / react-i18next
- **Browser**: browser-use CLI with pluggable backends (Chromium engine)

## Key Conventions

- Component-based frontend: `App.tsx` (state + layout shell) delegates to `views/` (TasksView, EventsView, WorkspaceView, SettingsView), `components/` (Sidebar, modals, UI primitives), `hooks/` (useSSE), with shared `types.ts` and `api.ts`
- Optional integration bundles (e.g. Google Workspace) install per-user via Settings → Integrations; see `app/workspace/gws_integration.py`.
- Pluggable browser backends in `app/browser/` (base.py defines interface); multi-client CDP proxy in `cdp_mux_proxy.py` for concurrent per-store tasks
- AI agent abstraction in `app/ai/` (`AIAgentBackend` ABC, `ClaudeCodeBackend` impl)
- System prompts in `app/prompts/*.md` — loaded once at import via `app/prompts/__init__.py`
- Tasks auto-execute on creation in **auto mode** (default, `plan_mode=false`): `bypassPermissions`, PENDING → RUNNING → COMPLETED. Opt-in **plan mode** (`plan_mode=true`): agent plans in read-only mode, user reviews, then executes (PENDING → DESIGNING → PLANNED → RUNNING → COMPLETED). Toggle via Auto/Plan switch in task detail footer. Non-store tasks always use plan mode. Plan-mode schedules author the plan once at creation via an `is_plan_only=True` Task (user reviews, plan is frozen on `Schedule.plan`, `plan_status=ready`); each subsequent fire copies the frozen plan and skips planning. See `app/plan_states.py` + [docs/subsystems.md](docs/subsystems.md#plan-at-creation-lifecycle).
- JWT cookie auth (httpOnly, 7-day expiry)
- UUIDs for all primary keys
- API routes prefixed with `/api/`
- i18n translations in `frontend/src/i18n/locales/{en,zh}/`
- **Ad tasks declare their kind + scope before working, and are held to what they declared.** The ad skills tell the agent to call `vibe_seller_declare_ad_task` (`kind` ∈ audit/create/execute/investigate, `scope` = combos/campaigns/products) before touching a browser. That declaration — not the shape of the report — decides how much marketplace coverage the completeness gate demands and whether the user gets a review console. Declarations are append-only and a new one needs a new USER message, so an agent cannot re-declare around a gate. **Never reintroduce an "is this an audit?" heuristic over the agent's own output**: that is what let a two-campaign task be told it owed five marketplaces, which it satisfied by transcribing the previous week's report. See `app/ai/ad_declaration.py` and [docs/api.md](docs/api.md).
  - **The in-turn ratchet** (`ad_declaration.supersedes`) — a re-declaration inside the same turn may never widen reach and may never shed an obligation. Two moves qualify: narrowing at the same kind (recording campaign ids you could only learn by enumerating), and correcting `investigate` → `audit` at the same-or-narrower scope. The upgrade is legal precisely because it *adds* what the phase owes — coverage plus a console — so it is not an escape hatch; `audit` → `investigate` and every other pair stay refused. Without it, a phase that under-declared had no move but to delete the recommendations the user asked for.
  - **Scope of the guarantee** — the requirement rides on the ad skills: `declaration_gaps` lives inside `ad_completeness_review`, which those skills register via `gates:` frontmatter, so a run that never loads an ad skill can finish without declaring. That is the fail-safe direction and is intentional: no declaration → no console → nothing over-wide is offered. Do not "fix" it by inferring a declaration from the report — that is the heuristic above, wearing a new hat. Within that gate, the checks that *compare a report against an existing declaration* are not so limited, and must not be scoped to incidental formatting: the "`investigate` may not hand out decisions" check deliberately runs whether or not the report carries `## <platform> <CC>` sections, because conditioning it on headings let a bid review declared `investigate` deliver bid changes with no console and no gap raised.
- Agents write to `stores/` and `knowledge/` via MCP `vibe_seller_write_workspace_file` (not the built-in Write tool — it can't write through workspace symlinks; see [docs/workspace.md](docs/workspace.md#symlink-write-caveat))

## Commands

All Python commands must use the project venv at `.venv/`:
```bash
source .venv/bin/activate  # Activate venv first
```

```bash
./start.sh [PORT]      # Start server (default 7777)
./stop.sh [PORT]       # Stop server
./restart.sh [PORT]    # Restart on same port
```

## Code Style

Google Python style enforced via pre-commit + ruff:

- **Line length**: 80 characters
- **Quotes**: Single quotes for strings, double for docstrings/triple-quoted

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zpoint/vibe-seller](https://github.com/zpoint/vibe-seller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
