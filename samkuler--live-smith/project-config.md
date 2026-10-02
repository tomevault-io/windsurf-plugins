---
trigger: always_on
description: This file defines durable collaboration rules. It is not a product manual,
---

# Live Smith Contributor Guide

This file defines durable collaboration rules. It is not a product manual,
implementation plan, or record of individual changes.

## Documentation responsibilities

- `README.md` introduces the product, user workflow, limits, and privacy.
- `docs/DEVELOPMENT.md` owns setup, build, verification, packaging, and local
  development data instructions.
- `docs/ARCHITECTURE.md` owns module responsibilities, lifecycle, and safety
  invariants; `docs/MODEL_PROVIDERS.md` owns model and connection contracts.
- Keep documentation about the implemented product and lasting constraints.
  Do not add task narratives, review outcomes, commit references, temporary
  checklists, or conversation context. Keep migration compatibility details
  where they describe supported data formats.
- Never commit superpowers workflow documents or temporary execution plans.
  Keep execution notes outside repository documentation.

## Project map

- `src/app/` owns application orchestration. Its `chat/`, `audio/`, `plugins/`,
  `model/`, `context/`, and `session/` directories own the corresponding
  workflows; request and dialog coordinators remain at the application root.
  Provider-specific application workflows stay in their provider directory,
  such as `src/app/audio/suno/`; shared audio workflows consume Plugin methods
  and adapter capabilities. Bind private credentials and host facilities in
  `src/app/plugins/built-in-plugin-runtime.ts`.
- `src/audio-services/` owns shared audio contracts and provider directories
  for protocol implementations. Suno website, Suno Platform, and SunoAPI remain
  separate provider boundaries.
- `src/agent/` contains the provider-neutral bounded tool loop and strict Live
  action schemas.
- `src/live/` observes Ableton state, resolves targets, and executes validated
  actions.
- `src/model/` contains Profiles, capability resolution, transport selection,
  and provider protocol adapters.
- `src/runtime/` owns explicit extension-host capability boundaries for Fetch
  and cancellation APIs.
- `src/attachments/` validates and extracts supported attachment formats;
  `src/skills/` owns Skill format rules and bundled definitions.
- `src/storage/` persists Profiles, sessions, session events, and raw model
  discovery metadata, and owns private attachment and User Skill storage.
- `src/ui/` contains state serialization and dialogs for the chat interface.
- Typed browser validators live in `src/ui/client/wire-contracts/`. Connection
  state and editing have dedicated browser modules; keep their state ownership
  out of the transport bridge and derive form views from the canonical snapshot.
- `tests/` mirrors source module ownership and contains behavior tests,
  module-local `support/` helpers, and attachment fixtures. Real chat DOM
  behavior tests live under interaction domains in `tests/ui/`; coordinator
  and bridge suites use subject directories under `tests/app/`.
  Provider protocol tests live under `tests/audio-services/`. Plugin examples
  and compatibility packages remain under `test-fixtures/plugins/`.

See `docs/ARCHITECTURE.md` and `docs/MODEL_PROVIDERS.md` before changing a
cross-module contract.

## Provider contracts

There are two API families and three supported modes:

- OpenAI Responses
- OpenAI Chat Completions
- Anthropic Messages

A named Profile owns one complete connection plus one or more model
configurations. Each model configuration owns its generation parameters,
capability overrides, hosted-tool policy, and Extra Body. A `direct-api`
connection owns family, mode, base URL, and API key. An `oauth-subscription`
connection owns only an OpenAI, Anthropic, or Google provider identity; its
credential, backend, auth generation, and signed-in catalog are isolated to the
exact Profile ID and provider. Credentials remain in private OAuth storage.
Google subscription OAuth uses Antigravity browser PKCE with the registered
hosted callback; the user pastes its one-time authorization code into the exact
pending Profile. Its catalog and generation requests use the Antigravity
product protocol without bundling or requiring the Antigravity runtime.
Do not add endpoint or vendor presets. OpenAI-compatible services use an
ordinary Direct API OpenAI Profile with the protocol they implement.

Keep Direct API provider-specific request mapping, streaming, tool-call replay,
and opaque response state inside `src/model/transports/`; keep OAuth login,
refresh, product-protocol, and lifecycle mapping inside `src/model/oauth/`. The
agent loop and Live executor must remain provider-neutral. Resolve feature
decisions from capabilities, not from model-name checks inside either boundary.

Preserve supported, unsupported, and unverified capability evidence. Direct API
discovery metadata belongs to its exact Profile connection; OAuth catalogs stay
modal-only and scoped to the selected Profile/provider connection and its
current auth generation. Configuration, discovery evidence, and Session model
selection have separate owners.

## Safety invariants

- Observe the relevant Live state before mutating it.
- Execute only actions accepted by the descriptors in
  `src/agent/action-schema.ts` and the plan validator in `src/agent/actions.ts`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SamKuler/live-smith](https://github.com/SamKuler/live-smith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
