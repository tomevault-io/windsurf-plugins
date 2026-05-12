---
trigger: always_on
description: Culpa is a deterministic replay and counterfactual debugging engine for AI agents in production. It captures every LLM call, tool invocation, file change, and terminal command with full fidelity — then lets developers replay failures deterministically and fork at any decision point to test "what if?" scenarios.
---

# CLAUDE.md — Culpa Project Context

## What Is Culpa

Culpa is a deterministic replay and counterfactual debugging engine for AI agents in production. It captures every LLM call, tool invocation, file change, and terminal command with full fidelity — then lets developers replay failures deterministically and fork at any decision point to test "what if?" scenarios.

**One-liner:** A flight recorder for AI agents. Observability tells you what happened. Culpa tells you why.

**Domain:** culpa.dev
**App URL:** app.culpa.dev
**Docs URL:** docs.culpa.dev
**License:** MIT

## Architecture Overview

Culpa has three layers:

### 1. SDK (`sdk/culpa/`)
Python package that developers install via `pip install culpa`. Contains:

- **`recorder.py`** — Core recording engine. Thread-safe. Creates sessions, records events (LLM calls, tool calls, file changes, terminal commands). Each event gets a ULID, microsecond timestamp, parent event ID, and sequence number.
- **`replay.py`** — Deterministic replay engine. Intercepts LLM calls and returns pre-recorded stub responses instead of calling the real API. Zero API cost. Includes divergence detection.
- **`fork.py`** — Counterfactual fork engine. Replays up to a fork point using recorded data, injects an alternative LLM response, then continues live execution in a sandboxed temp directory.
- **`models.py`** — Pydantic v2 models for all event types: `LLMCallEvent`, `ToolCallEvent`, `FileChangeEvent`, `TerminalCommandEvent`, `Session`, `ForkResult`.
- **`interceptors/`** — Monkey-patches for LLM SDKs:
  - `anthropic.py` — Patches `anthropic.Anthropic.messages.create`
  - `openai.py` — Patches `openai.OpenAI.chat.completions.create`
  - `litellm.py` — Patches `litellm.completion`
- **`watchers/filesystem.py`** — Uses `watchdog` to monitor working directory for file changes. Captures full before/after content and diffs. Links changes to triggering LLM calls via timing.
- **`proxy.py`** — Async HTTP proxy server (aiohttp) that sits between coding agents (Claude Code, Cursor) and LLM APIs. Transparently forwards requests while recording. Handles SSE streaming — forwards chunks immediately while accumulating for recording.
- **`proxy_parser.py`** — SSE chunk parsers for Anthropic and OpenAI streaming formats.
- **`cli.py`** — CLI commands: `culpa record`, `culpa replay`, `culpa sessions`, `culpa upload`, `culpa login`, `culpa serve`, `culpa proxy start/stop/status/env`.
- **`__init__.py`** — Exports `culpa.init()` for zero-config recording. Handles auto-upload via `CULPA_RECORD_OUTPUT` env var for subprocess handoff.

### 2. Server (`server/`)
FastAPI backend with SQLite storage.

- **`main.py`** — App entry point. CORS, static files, startup cleanup.
- **`api/auth.py`** — Register, login, logout, /me, API key CRUD. JWT (httpOnly cookies) + API key auth (Bearer culpa_xxx header).
- **`api/sessions.py`** — Session CRUD, scoped by user_id. Supports `?scope=mine|team|all`.
- **`api/events.py`** — Event queries, filterable by type and time range.
- **`api/forks.py`** — Fork execution endpoint.
- **`api/billing.py`** — Stripe Checkout, webhooks (checkout.completed, subscription.deleted, payment.failed), Customer Portal.
- **`api/teams.py`** — Team CRUD, invites, member management, session visibility.
- **`storage/database.py`** — SQLite init, migrations. Tables: users, api_keys, sessions, events, forks, file_snapshots, teams, team_members, team_invites.
- **`storage/repositories.py`** — Data access for sessions, events, forks.
- **`storage/user_repository.py`** — User and API key data access.
- **`storage/team_repository.py`** — Team, member, invite data access.
- **`services/auth.py`** — bcrypt hashing, JWT encode/decode, API key generation (culpa_ prefix, SHA-256 stored).
- **`services/plans.py`** — Plan limits (free: 3 sessions/7-day retention/5 forks; pro: unlimited/90-day/unlimited). Enforcement checks.
- **`services/email.py`** — Resend integration. Templates: welcome, first session, session expiring, limit reached.
- **`dependencies.py`** — `require_user` FastAPI dependency. Accepts JWT or API key.

### 3. Dashboard (`dashboard/`)
React 18 + TypeScript + Vite + Tailwind CSS.

- **Pages:** `SessionsList`, `SessionDetail`, `SessionCompare`, `Landing`, `Login`, `Register`, `ApiKeys`, `Billing`, `BillingSuccess`, `Team`
- **Components:** `Timeline`, `EventDetail`, `LLMCallDetail`, `FileChangeDetail`, `TerminalDetail`, `SessionOverview`, `ReplayControls`, `ForkModal`, `ForkComparison`, `CommandPalette`, `KeyboardShortcutsHelp`, `Skeleton`, `Logo`
- **Hooks:** `useSession`, `useReplay`, `useFork`
- **Auth:** `AuthContext` provider, JWT in httpOnly cookies, protected routes

## Database Schema

```sql
users (id, email, password_hash, name, plan, stripe_customer_id, stripe_subscription_id, plan_expires_at, email_notifications, created_at, updated_at)
api_keys (id, user_id, key_hash, key_prefix, name, created_at, last_used_at, revoked_at)
sessions (id, user_id, name, metadata_json, started_at, ended_at, status, expires_at, visibility)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AnshKanyadi/culpa](https://github.com/AnshKanyadi/culpa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
