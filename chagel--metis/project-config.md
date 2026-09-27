---
trigger: always_on
description: enables it on the team's github connector (`bot_enabled`,
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. `AGENTS.md` at the repo root is a symlink to this file, so pi / codex sessions on this repo see the same context (loaded once — context-file loaders dedupe by resolved path).

## Read first

Before changing code, read [`VISION.md`](VISION.md) — what Metis is, the
rules we hold to, and what we explicitly **won't build**. Most of the
guardrails (no second agent backend, no CLI-as-connector, no Rails-side
MCP runtime, no polymorphic owner, no SPA, no per-user provider keys)
are inverses of temptations already present in this codebase. Honor
them, or argue them on a PR — don't drift into them.

## Overview

Metis is a Rails 8.1 web app (Ruby 4.0.5, PostgreSQL) that puts a chat UI in
front of an agent harness. v1 ships the **pi** backend, driven via the
`pi-agent-rb` gem. Hotwire (Turbo + Stimulus, importmap, Tailwind) renders the
live streaming chat; Devise handles auth. The app is PWA-installable
(`rails/pwa` manifest + service-worker routes) and carries Hotwire Native
groundwork for the iOS app: `hotwire_native_app?` toggles a `hotwire-native`
body class, `Hotwire::PathConfigurationsController` serves the native path
config, and `Users::NativeAuthController#google` accepts the iOS app's
natively-obtained Google ID token.

## Commands

- `bin/dev` — run the app (Puma + Tailwind watch via foreman, port 3000)
- `bin/setup` — install deps, prepare the database
- `bin/rails test` — full test suite (Minitest)
- `bin/rails test test/services/agent/adapters/pi_test.rb:42` — single test by file:line
- `bin/rubocop` — lint (rubocop-rails-omakase house style)
- `bin/ci` — full CI pipeline: rubocop, bundler-audit, importmap audit, brakeman, tests, seed replant
- `bin/brakeman` / `bin/bundler-audit` — security scans
- `bin/rails metis:doctor` — configuration checklist: each subsystem (email, providers, runtime, storage, …) reported as configured / missing / defaulted, exit 1 on missing required config

Run `bin/rubocop` and the relevant tests before committing.

## Critical dependency

The [`pi-agent-rb`](https://github.com/chagel/pi-agent-rb) gem drives
`pi --mode rpc` and is the only way the app talks to the pi agent.
It comes from rubygems via the `Gemfile` — no sibling checkout needed.

Active Record encryption is used for `Message#content` and `Message#reasoning`,
so encryption keys must be present in Rails credentials for any environment
that touches that model (including tests).

## Architecture

### The Agent service layer (`app/services/agent/`)

This is the core of the app. Metis runs on a single agent harness —
pi. The Agent layer separates two concerns:

1. **`Agent::Adapters`** — *the agent*. `Adapters.for(conversation)` builds
   the `Pi` adapter, which drives pi and translates its native event
   stream. `#stream(input)` yields events. This layer decouples the chat
   UI from pi's wire protocol; it is not a multi-backend seam.
2. **`Agent::Runtime`** — *where* the agent runs. `Runtime::Local` runs pi
   as a local subprocess, `Runtime::Docker` in a container, `Runtime::E2b`
   in an isolated microVM, `Runtime::Daytona` in a Daytona elastic sandbox,
   `Runtime::Microsandbox` in a self-hosted libkrun microVM (in-process via
   the optional `microsandbox-rb` gem — no daemon).
   **`Runtime::Local` is not a security boundary** — pi has shell access.
   In production the `docker` runtime runs under **gVisor** (`runsc`, set by
   `METIS_DOCKER_RUNTIME`) as Docker-in-Docker from the containerized `job`
   worker on a single host; see `docs/coding-runtime.md`.

pi's native events are translated into **`Agent::UiEvent`**, a canonical
vocabulary (`text_delta`, `tool_call_started`, `turn_finished`, …) that
keeps the chat UI decoupled from pi's protocol. `UiEvent#native_ref`
keeps the raw payload for native view helpers.

### Request → response flow

(For the full turn-flow diagram, see `docs/architecture.md`.)

1. `ConversationTurn.start` is the single place a turn is born — it creates a
   `user` message and a `pending` `assistant` message, then enqueues `ChatJob`.
   The composer (`MessagesController` via the `Composing` concern), the
   workflow engine, and the from-chat workflow handoff (`Agent::WorkflowHandoff`,
   via the agent's `metis_start_workflow` tool) all go through it.
2. `ChatJob#perform` runs one turn: it gets the adapter, calls `#stream`, and
   for each `UiEvent` hands it to `ChatBroadcaster` while buffering text.
3. `ChatBroadcaster` maps each `UiEvent` to a Turbo Stream broadcast on the
   conversation's stream.
4. When the turn settles, `ChatJob` calls `WorkflowRun.signal_turn_finished`
   — a no-op for a normal chat, a re-enqueue of `WorkflowAdvanceJob` when a
   workflow run drives the conversation.

**Division of labor:** `ChatJob` owns *persistence* (writing the final message
content + `streaming_status`); `ChatBroadcaster` owns the *live DOM*. Keep
these separate.

### Observability

Every finished turn persists its usage onto the assistant `Message`
(`input_tokens`, `output_tokens`, `cache_read_tokens`, `cost` USD,
`model_key`) — cost and model come straight from pi's `get_session_stats`
RPC, so Metis prices nothing itself. Optionally each turn is also exported as

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chagel/metis](https://github.com/chagel/metis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
