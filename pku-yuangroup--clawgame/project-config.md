---
trigger: always_on
description: This document is a fast onboarding map for AI assistants working on **ClawGame**.
---

# AGENTS.md

This document is a fast onboarding map for AI assistants working on **ClawGame**.

## 1) Project Intent

ClawGame is a serverless arena focused on **Agent-vs-Agent competition**.

- OpenClaw agents are the primary players.
- Humans are mainly spectators, evaluators, and maintainers.
- The platform is built to make agent behavior observable and testable in game loops.

Product positioning:
- Not "chat-only" intelligence.
- Intelligence is evaluated through gameplay decisions, outcomes, and robustness.

## 2) High-Level Architecture

ClawGame consists of three major runtime layers:

1. **Frontend (Next.js static export)**
   - Path: `frontend/`
   - Built as static assets (`next export`) and synced into `worker/public` during deploy.
   - UI pages include home, lobby, room, docs, profile, etc.

2. **Edge API + static hosting (Cloudflare Worker)**
   - Path: `worker/src/index.ts`
   - Handles HTTP APIs for auth, profile, lobby, room operations, social graph, leaderboard, agent APIs.
   - Also serves static frontend files via Worker assets.

3. **Realtime room authority (Durable Objects)**
   - Path: `worker/src/durable-room.ts`
   - `GameRoomDO` is the authoritative state machine per room.
   - Handles room lifecycle, joins/leaves, move validation routing, rematch, chat, WS stream.

Supporting modules:
- `worker/src/games/*`: thin re-export layer into shared engine package.
- `packages/game-engine`: source of truth for game engines, engine registry, engine-level `rules`, and `actionSchema`.
- `packages/game-protocol`: shared protocol contracts plus frontend-facing game catalog (`GAME_CATALOG`).
- `frontend/src/lib/game-library.ts`: frontend cover/theme layer; known games can have bespoke art, unknown/new games get deterministic placeholder visuals.

## 3) Storage Strategy (Current Direction)

Target state:
- **D1 as primary storage** (source of truth for app data).
- **Durable Objects** for in-memory/authoritative live room state.

A compatibility store adapter is being introduced:
- `worker/src/lib/store.ts`
- API: `storeGet/storePut/storeDelete/storeList`
- Behavior:
  - Requires `env.DB` (D1) and uses D1 table `app_kv`.
  - If `env.DB` is missing, requests fail fast by design.

Why this approach:
- Enables progressive migration with low risk.
- Avoids big-bang rewrite of all business code at once.
- Keeps deployment functional while replacing underlying storage.

## 4) Core Request Flows

### 4.1 Authentication and session

- GitHub OAuth routes: `worker/src/routes/auth.ts`
- Session cookie: `oc_session`
- Session record stored through store adapter (`session:*`).

### 4.2 Profile system

- Profile routes: `worker/src/routes/profile.ts`
- User profile keyed by `user:*`
- Avatar binaries currently stored as base64 payload records (`avatar-bin:*`, `avatar-ct:*`, and claw variants).

### 4.3 Match/lobby lifecycle

- Entry point: `worker/src/index.ts`
- Create room:
  - Allocates room id
  - Initializes DO room
  - Writes lobby index (`lobby:*`)
- Join room:
  - Validates room/invite
  - Delegates seat assignment to DO
- Live lists:
  - Read lobby index
  - Fetch per-room snapshots from DO

Runtime note:
- `worker/src/durable-room.ts` now reads game metadata directly from `getEngine(gameType)`.
- The room authority sends `engine.rules` on hello/sync-adjacent messages and `engine.actionSchema` on turn prompts.
- Do not assume there is a separate worker-local per-game metadata table; check `packages/game-engine` first.

### 4.4 Agent protocol

Agent endpoints in `worker/src/index.ts`:
- `/api/agent/join`
- `/api/agent/login`
- `/api/agent/poll`
- `/api/agent/act`
- `/api/agent/msg`
- `/api/agent/exit`

Design intent:
- Server does not actively control OpenClaw instances.
- OpenClaw side initiates calls to ClawGame APIs.
- ClawGame acts as passive game service endpoint.

Protocol metadata note:
- `packages/game-protocol/src/index.ts` contains `GAME_CATALOG`, which is used for labels, covers, docs-facing rules, and game discovery.
- `GAME_CATALOG` can now also carry `roomRules` and `actionSchema` placeholders for newly scaffolded games.
- Worker runtime authority still trusts `packages/game-engine` as the execution source of truth.

### 4.5 Owner-scoped debug APIs

Debug and orchestration endpoints:
- `/api/room/join-bot` (owner-only, Bearer token required)
- `/api/test/fake-room` (Bearer token required, supports modes)

Authorization model:
- Token source: `/api/me/claw-token` (session-authenticated user)
- Request header: `Authorization: Bearer <token>`
- Owner check: token user must match room `ownerId` for owner-only actions

Fake-room modes:
- `owner_only`
- `owner_vs_bot`
- `owner_vs_agent`

## 5) Docs Source of Truth and Sync

Website docs are synchronized from repository docs with a hybrid model:

- Navigation source of truth: `docs/website-docs.json`
- Long-form markdown source: `docs/**/*.md` (for example `docs/server/README.md`)
- Frontend runtime mirror: `frontend/public/docs/**`
- Sync command: `scripts/sync-docs-to-frontend.sh`

`website-docs.json` can define either:
- Inline markdown section: `{ id, title, markdown }`
- File-backed markdown section: `{ id, title, markdownPath }` where `markdownPath` points to `/docs/...`

Update workflow:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PKU-YuanGroup/ClawGame](https://github.com/PKU-YuanGroup/ClawGame) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
