---
trigger: always_on
description: These instructions apply to the entire repository.
---

# Repository instructions for coding agents

These instructions apply to the entire repository.

## Working approach

- Inspect the current worktree and relevant code before editing. Preserve
  unrelated changes and keep each patch focused on the requested outcome.
- Carry authorized work through implementation and validation. Resolve routine
  choices from repository conventions. Ask only when a missing answer blocks
  correctness or authorization; continue independent work meanwhile.
- Apply user corrections to the ongoing task without losing completed work or
  abandoning the original objective unless requested.
- User instructions take precedence over skill guidelines. If a skill blocks
  progress, identify its exact file and instruction and explain the conflict.
- Keep updates concise and concrete: describe the outcome, validation, and
  remaining blockers in plain language.
- Review the final diff and report what changed, which checks passed, and any
  checks that could not run. Never claim unperformed validation succeeded.
- When asked to commit, stage only the intended files. A local commit request
  does not authorize pushing or publishing a release.

## Mission

Maintain a production-quality HACS custom integration that gives Home Assistant
Mistral-backed Conversation, AI Task, speech-to-text, and text-to-speech entities
through the official `mistralai` Python SDK. Preserve Home Assistant conventions
and defensive behavior; do not imply that this project is first-party, Home
Assistant-certified, or endorsed by Mistral AI.

## Read before editing

1. Read `README.md` for supported user behavior and installation requirements.
2. Read `QUALITY.md` for architectural promises and assurance boundaries.
3. Read `CONTRIBUTING.md` and the tests nearest the code being changed.
4. Check current Home Assistant and Mistral SDK APIs when behavior is uncertain.
   Do not copy examples written for older Home Assistant conversation APIs.

## Repository map

- `custom_components/mistral_conversation/` is the only runtime integration.
- `api.py` owns SDK client creation, credential validation, and model/voice
  discovery.
- `coordinator.py` owns the shared client, model metadata, availability, and
  refresh lifecycle.
- `conversation.py` is the thin Home Assistant `ConversationEntity` adapter.
- `ai_task.py`, `stt.py`, and `tts.py` are the thin Home Assistant adapters for
  structured tasks and voice pipelines.
- `entity.py` owns message conversion, streaming, capability checks,
  attachments, structured chat output, and the tool-execution loop.
- `tool_calls.py` reconstructs fragmented and parallel streamed tool calls.
- `config_flow.py`, `diagnostics.py`, and `repairs.py` implement their matching
  Home Assistant surfaces.
- `translations/en.json` and `translations/fr.json` are both maintained.
- `tests/` uses Home Assistant-native fixtures and mocked provider behavior.
- `hacs.json`, `manifest.json`, the brand assets, and
  `.github/workflows/validate.yml` form the HACS publication surface.

## Architectural invariants

### Home Assistant lifecycle

- Keep configuration UI-only: one account config entry with typed Conversation,
  AI Task, STT, and TTS config subentries.
- Store shared runtime state in typed `ConfigEntry.runtime_data`.
- Use Home Assistant's shared HTTP client when constructing the Mistral client,
  and close provider resources deterministically during unload or failed setup.
- Keep the event loop non-blocking. Put filesystem or other blocking work in
  `hass.async_add_executor_job`.
- Raise translated Home Assistant exceptions and preserve reauthentication,
  retry, repair, and entity-availability semantics.
- When persisted data changes, update config-entry migration logic and add
  migration tests. Never silently reinterpret existing user configuration.

### Mistral provider boundary

- Use the official SDK and its generated request/message types. Keep
  provider-specific objects behind the typed boundary in `api.py`, `entity.py`,
  and `tool_calls.py`.
- Preserve streamed text, reasoning, usage, finish-state, and parallel tool-call
  handling. A stream may split a tool name or JSON arguments across many chunks.
- Replay provider-native signed reasoning only in the native form expected by
  Mistral. Never invent, alter, expose, or log reasoning signatures.
- Treat discovered capability metadata as authoritative for known models, but
  continue allowing unknown custom and fine-tuned model IDs.
- Normalize SDK and transport failures through `errors.py`. Authentication,
  timeout, connection, rate-limit, and generic provider failures have different
  Home Assistant behavior and must remain distinguishable.
- Do not add a live Mistral request to normal tests or CI.
- Use `client.audio.transcriptions`, `client.audio.speech`, and
  `client.audio.voices` through their generated asynchronous SDK surfaces.
- Keep AI Task structured responses on native JSON-schema output instead of
  relying on prompt-only JSON formatting.

### Tools and attachments

- Home Assistant remains the authority for tool schemas and execution. Preserve
  the model-to-Assist round trip through `ChatLog`.
- Reject incomplete, malformed, or ambiguous streamed tool calls before
  execution.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Elijaht-dev/mistralai-conversation](https://github.com/Elijaht-dev/mistralai-conversation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
