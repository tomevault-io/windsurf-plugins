---
trigger: always_on
description: `llm_router` is a Codex-facing Responses compatibility runtime, not a generic
---

# Repository Guidelines

## Project Role

`llm_router` is a Codex-facing Responses compatibility runtime, not a generic
HTTP proxy. It accepts Codex/OpenAI-style `/v1/responses` traffic, maintains
local response state for router-owned routes, adapts requests to
provider-specific Chat APIs, and reconstructs Responses-shaped output for
Codex. The explicit `responses_passthrough` route type is the exception:
there, a compatible upstream owns native Responses state.

## Hard Gate Before Editing

Do not modify any file in this repository unless both evidence requirements are
satisfied.

1. Codex source must be available locally and current. The maintainer must
   provide a local Codex source directory, then the developer must verify it is
   up to date with `git pull`; if no local copy exists, clone Codex locally
   before making changes. Router behavior must be checked against that Codex
   source and the relevant guides/reference files in `docs/`.
2. Provider API documentation must be available for the provider being changed,
   and real behavior must be verified through logs, live requests, or focused
   regression tests. For provider work, do not rely on OpenAI compatibility by
   assumption; use that provider's own API docs and observed behavior.

If either requirement is missing, stop. Ask for the missing Codex source path,
clone permission, provider API documentation, or a real reproduction log. Until
those are available, the model must not edit any file.

Behavioral changes must be grounded in three sources together:

- local Codex source, usually provided by the maintainer
- official provider API documentation supplied or identified for the task
- real router logs or focused regression/live tests

When these disagree, do not guess. Reproduce the actual request shape first,
then adapt the provider boundary.

## Code Map

- `llm_router/server.py`: HTTP entrypoint and request orchestration. Keep this
  thin; do not hide provider quirks here when an adapter can own them.
- `llm_router/config.py`: model routing, route type, upstream model rewrite.
- `llm_router/codex_recovery.py`: narrow Codex compatibility recovery, such as
  Plan-mode retry steering and diagnostics helpers.
- `llm_router/provider_errors.py`: client-visible provider error mapping.
- `llm_router/responses_state/`: response IDs, input normalization, pending
  tool validation, SSE event construction, Responses tool normalization, and
  usage normalization.
- `llm_router/session_store.py`: persisted Responses session state.
- `llm_router/deepseek/`: DeepSeek official Chat API adapter.
- `llm_router/mirothinker/`: MiroThinker MCP-first adapter.
- `llm_router/openai_chat.py`: generic OpenAI-compatible Chat adapter.
- `tests/`: unit and integration regression tests.
- `tests/live/`: opt-in live Codex/router/provider tests.
- `docs/`: current architecture, Codex runtime rules, provider adapter guide,
  testing guide, and future work.

## Adding A Provider

Add provider support by creating or extending a provider adapter. The adapter
owns provider-specific request filtering, tool schema conversion, model
parameter translation, response parsing, and provider-private replay state.

Typical flow:

1. Add a route in `router.toml` with `pattern`, `type`, `upstream`, and optional
   `upstream_model`.
2. Add an adapter module if behavior differs from existing adapters.
3. Wire adapter selection in the smallest possible place, normally near
   `_chat_adapter_for`.
4. Preserve the normalized Responses state-machine contract for router-owned
   routes, or use an explicit `responses_passthrough` route when the upstream
   owns a compatible native Responses API.
5. Add tests that prove the provider-specific behavior and the Codex-facing
   response shape.

Do not add broad provider hacks to `llm_client.py`. That file should remain
transport-level. Do not globally inject provider parameters such as
`repetition_penalty`, `top_k`, `service_tier`, or `client_metadata`.

## Adapter Responsibilities

Provider adapters decide what the upstream receives. For router-owned routes,
the Responses state machine decides what Codex receives.

Adapters should:

- strip unsupported request fields by allowlist
- map documented provider parameters explicitly
- normalize unsupported roles such as `developer` only when required
- convert Codex tools without dropping them silently
- preserve provider-required replay data such as DeepSeek `reasoning_content`
- return normalized output items for the state machine to commit on
  router-owned routes

Adapters should not:

- own global conversation state
- mutate durable sessions directly
- decide Codex collaboration policy
- expose provider-native streaming chunks directly to Codex
- silently discard multimodal or hosted-tool payloads

## State Machine Boundary

`/v1/responses` must behave statefully for all router-owned route types. The
route type normally selects provider conversion behavior; it must not bypass
state management implicitly.

`responses_passthrough` is the explicit exception. Use it only for a provider
that exposes a compatible native `/v1/responses` API and owns response IDs,
`previous_response_id`, pending tool state, and continuation semantics. Official

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ansatzX/llm_router](https://github.com/ansatzX/llm_router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
