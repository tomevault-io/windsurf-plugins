---
trigger: always_on
description: Celeste: unified multi-modal AI SDK for Python (3.12+, uv). One client API across 20+ providers — text, images, audio, videos, embeddings.
---

# CLAUDE.md

Celeste: unified multi-modal AI SDK for Python (3.12+, uv). One client API across 20+ providers — text, images, audio, videos, embeddings.

## Core concepts

- **Modality**: the one modality, in addition to text, essential to the model's purpose — the modality the model cannot function without.
- **Domain**: the resource you work with.
- Never describe Modality as an output type or Domain as an input type.
- Key enums, all in `src/celeste/core.py`:
  - `Modality` (text, embeddings, images, videos, audio)
  - `Domain` (text, images, audio, videos, documents)
  - `Operation` — what you do (generate, edit, analyze, speak, transcribe, embed, upscale)
  - `Provider` — which backend serves the request
  - `Protocol` (chatcompletions, openresponses)
- `(Domain, Operation) → Modality` inference: `DOMAIN_OPERATION_TO_MODALITY` in `core.py`.

## Architecture: three layers

1. `src/celeste/providers/<vendor>/<api>/` — wire/HTTP mixins per vendor API.
   - Dirs are named after the API: `anthropic/messages`, `cohere/chat`, `elevenlabs/text_to_speech`.
   - Parameter mapper classes here carry no `.name`.
2. `src/celeste/modalities/<modality>/providers/<vendor>/` — composition layer.
   - `client.py` composes wire mixin + modality client; `parameters.py` binds unified `.name`s; `models.py` is the model catalog.
3. `src/celeste/protocols/` + `src/celeste/modalities/text/protocols/` — shared protocol base clients: a protocol is a wire format served by many vendors (currently `chatcompletions`, `openresponses`).
   - RULE: editing a protocol changes every inheriting vendor (DeepSeek, Moonshot, Groq, Mistral, HuggingFace, ...).
   - Vendor-specific behavior goes in that vendor's override (`client.py` / `parameters.py`), never in the shared base.

Multi-backend providers (one provider, several wire APIs — e.g. Google):

- The `PROVIDERS` registry always maps a `Provider` to one `{Provider}{Modality}Client` class. It never branches and never holds functions.
- `client.py` is then a dispatcher: `{Provider}{Modality}Client({Modality}Client)` selects the backend once in `model_post_init` (by auth and/or model id) via `object.__setattr__(self, "_strategy", ...)`, copies the endpoint ClassVars it routes on, concatenates all backend mapper lists in `parameter_mappers()`, and forwards every hook a backend customizes to `self._strategy` — a missing delegate silently runs the base implementation. Enforced by `tests/unit_tests/test_dispatcher_delegation.py`, which auto-discovers dispatchers from the registry and fails CI naming the hook, dispatcher, and backend; `ModalityClient`'s docstring (`src/celeste/client.py`) describes the hook surface.
- Each backend is the old single-backend client body in an api-named file (`interactions.py`, `vertex.py`), classes renamed `{Provider}{Api}{Modality}Client`; every mapper set in `parameters.py` takes the backend token too (`VertexTemperatureMapper`, `GOOGLE_VERTEX_PARAMETER_MAPPERS`) — a bare set never coexists with a tokened one.
- Where variation lives, in escalation order: endpoint ClassVar (`_generate_endpoint`, `_edit_endpoint`) for same-wire operations → `_build_url` auth branch for the same wire format on another host (Vertex) → `_strategy` dispatcher only for genuinely different wire formats.

Naming: wire mixin `{Provider}{Api}Client` (`GoogleInteractionsClient`); registered composite `{Provider}{Modality}Client` (`GoogleTextClient`); per-backend composite `{Provider}{Api}{Modality}Client` (`GoogleInteractionsTextClient`). The prefix is always the `Provider` enum name, never a brand or model family (`Google`, not `Gemini`). Composition files always alias the wire imports — `{Provider}{Api}Client as {Provider}{Api}Mixin`, `{Provider}{Api}Stream as _{Provider}{Api}Stream` — and never alias `config`.

Template-first: new files start from `templates/` — `cp` the template, then edit. Never write provider/modality files from scratch. Routing map: `templates/README.md`. Registration checklist: CONTRIBUTING.md "Adding a provider" / "Adding a second API to an existing provider".

## Model catalog rules (`models.py` entries)

- `Model.streaming` means celeste's adapter transport streams this model — not the vendor's advertised capability. Flip it only after a live call through celeste's own streaming path.
- A `MAX_TOKENS` Range comes only from a provider-documented max-completion/output figure. A context window is never a completion cap; if no completion max is documented, omit the constraint.
- Choice/enum literals verbatim from the provider's API request-parameter reference, not marketing/model pages (those use display tokens). If two official pages conflict, keep the current value and record the conflict.
- When the API reference and live serving disagree, live serving wins for constraints and tests; record the conflict in the PR.
- One `parameter_constraints` dict serves every call mode (unary AND streaming). Constrain to the union; let the provider reject mode-specific invalids.
- Remove a model id only after a live authenticated call hard-rejects it. Lifecycle/deprecation tables are announcements; providers keep retired aliases serving or silently redirect them.

## Validation philosophy


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [withceleste/celeste-python](https://github.com/withceleste/celeste-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
