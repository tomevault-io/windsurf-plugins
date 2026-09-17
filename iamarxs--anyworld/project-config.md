---
trigger: always_on
description: This section describes current implementation; differences from the original requirements are not
---

# Anyworld (ArtificialDungeon): current maintenance guide

This section describes current implementation; differences from the original requirements are not
automatically approved product changes. Proposed fixes belong in TASKS.md.

## Working conventions

- Preserve user edits and ignore `venv/` and `.venv/` in reviews/searches.
- Read-only reviews may run offline tests when temporary files are acceptable. Fixtures remove
  test artifacts on teardown; in-process ASGI tests are allowed, but do not launch a live server
  or run live inference/benchmarks. Edit documentation only when explicitly requested.
- Keep party chat outside LLM context. Private DM guidance and hidden dice stay out of public
  events; server-side HTML transcripts include them, and private-check rolls appear in server logs.
  Treat these files as private archives, not player-safe exports.
- Preserve join-order input collection and simultaneous, causally coherent round resolution.
- Keep network/file I/O outside state-mutation locks; use logic/models.py protocols rather than
  importing concrete transport into game logic. An ended game must not be revived by stale inference.

## Current architecture and contracts

- Python 3.11+, FastAPI, Pydantic, vanilla JS/CSS. Package and CLI name: `anyworld`.
- `app.py` validates passwords and starts Uvicorn with HTTPS. `api/tls_bootstrap.py` handles
  IP discovery and self-signed certificates under `certs/`.
- `core/config.py` exports lowercase singleton `settings`. The llm schema additionally contains
  provider (`compatible`/`openai`), tokenizer_encoding and system_prompt. Compatible `/props`
  discovery overrides context_window_size when successful. Server passwords must be distinct.
- `api/server.py` owns GET /, /static, /ws/{client_id}, ConnectionManager and an engine/resolver
  per ASGI lifespan. Lifespan validates passwords and closes sockets, tasks and clients.
  Client IDs must be canonical UUIDs. No multi-session or multi-worker coordination exists.
- `logic/models.py` contains GameState (including ENDED), Player and dependency protocols.
  `logic/lobby.py` owns authentication, scenario setup, start/end, chat and reconnect snapshots.
  `logic/engine.py` owns the lock, turn deque, action buffer and round orchestration.
- `logic/validation.py` validates text; `logic/presentation.py` normalizes player outcome names.
- Host authentication precedes scenario setup. `generate_scenario_title()` returns only a title
  through ScenarioTitle; it does not remember narrative. Joining players see the host-typed prompt.
  Start Game calls `generate_start_state()` with joined names and broadcasts the generated opening.
  Missing player names are rejected; role, goal, and prose coherence remain prompt instructions.
  No generation occurs just because a player joins.
- Client envelope: event_type plus object data. Events: auth, chat, action, scenario_init,
  start_game, end_game, retry_round. Auth sends name and SHA-256 password_digest of password +
  client ID. Reauthentication additionally requires the private reconnect_token from auth_ok,
  retained in browser sessionStorage. The ID/token pair is also saved per name in localStorage
  for recovery after re-entering name/password at the same origin; passwords/digests are not stored
  there. Pending sockets cannot subscribe, replace a player or act.
- Server envelope: type plus object payload. Types: state_update, chat_echo, turn_directive,
  error, system_msg, auth_ok, scenario_ready, round_start, action_echo, player_roster,
  dm_thinking, game_ended, token_usage. Turn directives use active_player_id.
- `core/schemas.py`: RoundResolution player_resolutions keys must be exact player names; runtime
  validation rejects other keys. ScenarioTitle contains only title. DicePlan has rolls and hidden_rolls.
  ContextSummary has world_state, player_states, important_npcs and unresolved_threads. Models forbid extra fields and coercion.
- `logic/dice.py` generates integers 0..100 inclusive. Do not silently change the probability
  distribution.
- Disconnected players get idle actions when progression is possible; departure/return annotations
  inform the LLM, and persistently absent players are omitted from later outcomes.
- `logic/transcript.py` writes escaped HTML under .logged_games/YYYY-MM-DD-title[-suffix].html,
  not TXT. Appends are thread-offloaded; directory creation at construction is synchronous.
  Original scenario prompt and Opening scenario are separate sections; private guidance and hidden
  checks are included. Token usage is broadcast to all after rounds.
  Writes/finalization are serialized; cancellation waits for outstanding file writes.
- Inference runs as an owned task outside socket receive loops. Generation IDs prevent stale
  commits. Failed rounds pause with actions/dice intact; the host can retry or end. Reconnection
  resumes empty active turns; versioned presence transitions survive older round completions.

## Context and cache guidance

Requests contain system prompt, fixed scenario/private guidance, separate durable memory, recent
history and current input. Dice planning receives the same authoritative context plus the public

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iamarxs/AnyWorld](https://github.com/iamarxs/AnyWorld) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
