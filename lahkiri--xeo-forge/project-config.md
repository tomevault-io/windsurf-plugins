---
trigger: always_on
description: > Product contract: **The Control Plane for Agentic Work**. Chat is conversation; Work is governed agency.
---

# Xeo Forge V3 — Agent & Architecture Rules

> Product contract: **The Control Plane for Agentic Work**. Chat is conversation; Work is governed agency.

This file is the contract for working in this repo. Keep it short, keep it enforced.
If a change violates a rule here, the change is wrong — not the rule.

## 1. What this product is

One governed AI agent with reusable execution context:
1. **Chat surface** — conversational answers and exploration; it never creates a plan or executes write tools.
2. **Work surface** — intent-aware agent work. Normal messages stay conversational; explicit planning starts Planning, and direct execution requests pause for an auditable user choice.
3. **Planning mode** — read-only inspection, produces a structured plan for approval.
4. **Build mode** — executes an immutable approved plan or an immutable, explicitly accepted execution brief.
5. **Context layers** — Prompt Studio instructions, approved memories, Agent Profiles, and Agent Skills are compiled into the run context.

The agent:
1. receives a conversation or Work request from a user and classifies intent before selecting planning or execution,
2. loads policy, profile, skill, task context, and approved memories,
3. inspects and analyzes (planning) or executes (build),
4. returns a final result and bounded memory candidates,
5. persists full history and audit events,
6. consumes credits per run.

Plus auth, per-user credits, admin controls, ONE global model configuration,
reusable profiles and skills, context management, and an inspectable audit trail.

New capabilities must preserve the approval gate, task-scoped authorization, single source of truth, and end-to-end UI-to-persistence behavior.

## 2. Hard architecture rules (non-negotiable)

1. **Single source of truth.** One canonical schema per entity. One writer per
   resource. No dual persistence. No second copy of the same logical data that
   can drift.
2. **One delivery path for events.** Task events are persisted with a monotonic
   per-task `seq`. SSE replays from the DB only, tracks `maxSeq`, then forwards
   live events with `seq > maxSeq`. No in-memory replay buffer racing the DB.
   (This is the V1 duplication bug. Do not reintroduce it.)
3. **No silent failures.** No `catch {}` without logging. Every caught error is
   logged with context. Persistence failures must be visible, never swallowed.
4. **End-to-end or not at all.** No UI control that points at a route that does
   not exist. No route that only half-works. Build the full path:
   input → agent → tools → persistence → UI.
5. **One global model.** All users share one model config. No per-user model
   selection. Source of truth: `model_settings` row id=1, seeded from env.
   API keys are NEVER returned to any client — always masked.
6. **Credits are atomic.** Debit via conditional `UPDATE ... WHERE balance >= ?`.
   Every balance change writes a `credit_ledger` row with `balance_after`.
   No read-then-write race.
7. **Authz on every task-scoped route.** Owner-or-admin check, always.
8. **No ungoverned feature creep.** Do not add subagents, teams, connectors,
   schedules, marketplaces, plugins, analytics, or permission frameworks without
   a written V3 design, explicit authorization boundaries, and an end-to-end path.
9. **No dead code.** Don't scaffold for "future features". Delete what isn't used.
10. **Typecheck stays clean.** `tsc --noEmit` has zero errors. Never hide errors
    behind `ignoreBuildErrors`.

## 3. Tech stack

- Next.js 14 App Router, React 18, TypeScript 5 (strict).
- Tailwind CSS for styling.
- DB: SQLite (better-sqlite3) in dev, PostgreSQL (pg) in prod, via one adapter
  exposing `prepare().{get,all,run}` returning Promises.
- LLM: `openai` SDK against an OpenAI-compatible endpoint (streaming tool calls).
- Auth: cookie session (`xeo_session`), token = random 32 bytes, stored as sha256.
- Local runtime: an optional Go process broker (`native/runtime-broker`), bound to
  loopback and gated by a per-install shared secret.

## 4. Data model (the only tables)

- `users` — id, email, password_hash, display_name, is_admin, is_root_admin,
  is_suspended, created_at.
- `auth_sessions` — token_hash (pk), user_id, expires_at.
- `credits` — user_id (pk), balance, daily_grant, last_reset_at, updated_at.
- `credit_ledger` — id, user_id, delta, reason, ref_id, balance_after, created_at.
- `tasks` — id, user_id, goal, status (including awaiting_decision), mode
  (chat|planning|build), project_path, intent_kind, decision_state,
  decision_expires_at, plan (latest proposed plan), approved_plan (immutable
  snapshot frozen at approval — also holds the frozen execution brief for an
  accepted direct-execution decision), plan_version, profile_id, skill_id,
  result_summary, credits_spent, error, created_at, updated_at.
- `task_events` — id, task_id, seq, type, content (JSON), created_at.
  UNIQUE(task_id, seq).
- `messages` — id, task_id, role (user|assistant|system), content, active,
  created_at. Conversation history per task. `active` distinguishes live-context
  rows (1) from archived rows (0) that compaction has summarized away.
- `model_settings` — id (always 1), name, base_url, api_key, model_id,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lahkiri/xeo-forge](https://github.com/lahkiri/xeo-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
