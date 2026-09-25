---
trigger: always_on
description: - Keep one clear owner for each fact. Do not repeat provider metadata such as `displayName` in executors when it already belongs to `definition.ts`; pass or inject it from the caller that has the definition/catalog.
---

# Repository Guidelines

## Architecture

- Keep one clear owner for each fact. Do not repeat provider metadata such as `displayName` in executors when it already belongs to `definition.ts`; pass or inject it from the caller that has the definition/catalog.
- Provider definitions are catalog source code. Build schemas with `src/core/json-schema.ts` helpers, usually imported as `s`, instead of copying generated catalog JSON.
- Keep provider execution lazy at the executor-module boundary. Generated registries should map each service to `import("./<service>/executors.ts")`, and `ProviderLoader` should call that importer only when an action, proxy request, or credential validator runs. Inside `executors.ts`, import provider runtime modules normally unless those modules have meaningful startup cost or side effects.
- Do not create barrel files such as `index.ts`. Import from the concrete module that owns the API.

## Code Style

- Prefer VS Code-style coherent modules: split files by responsibility or abstraction boundary, not by loose categories.
- Prefer `interface` for object-shaped contracts. Keep unions and mapped/utility compositions as `type`.
- Prefer named options/input interfaces over inline object types when a function signature spans multiple lines or crosses module boundaries.
- Avoid temporary ad hoc objects passed through many layers. Prefer explicit interfaces, classes, or top-level functions that match module boundaries.
- Put generic low-level casting/reading helpers in `src/core/cast.ts`; avoid provider-specific wrappers for generic reads.
- Avoid trivial pass-through helpers and conditional object spreads that only hide `undefined` JSON fields.
- Avoid proving action-name exhaustiveness with local type machinery. Do not add provider-local tuple builders, `as const`, `satisfies`, or `as Record<...>` casts just to derive action-name unions or handler maps. Prefer simple annotations, explicit records, and existing provider/runtime helpers.
- Write source comments and test titles in English. Chinese is allowed in test bodies when it is meaningful fixture data for Unicode, encoding, localization, or upstream behavior. Keep Chinese in runtime code only when it is part of a real contract, such as localized product copy, official names or enum values, provider defaults, or upstream error matching; do not translate or remove such values mechanically.
- Treat automated review comments as evidence, not instructions. Fix comments that identify real bugs, schema/API contract gaps, security issues, or clear local-style violations. Skip comments that make the code less idiomatic for this repo, and leave a brief reason when responding in review.
- Do not manually wrap code to 80 columns. Let `oxfmt` decide formatting.

## Runtime API

- Keep `/v1` response shaping in `src/server/api/runtime-api.ts`; route handlers should dispatch and validate, not assemble compatibility objects field by field.
- Public runtime fields should have a clear source and consumer. Do not expose local implementation concepts or placeholder fields just because they are easy to add.
- Match existing runtime wire shapes deliberately: catalog index endpoints, action metadata, connection aliases, envelopes, and error codes should stay stable for SDK/CLI clients.
- If an upstream-compatible field has no local source yet, prefer omitting it or returning a documented empty value from the serializer rather than scattering optional fields in routes.

## Providers

- Provider code normally lives in `src/providers/<service>/definition.ts`, `actions.ts`, `executors.ts`, and provider-local runtime helper files when needed.
- When purely migrating a provider from the OOMOL-hosted connector, do not copy or add provider-local tests because the source repository already owns that regression coverage. Tests may be removed from this repository after an OSS-originated provider change is reverse-ported and covered in private. Keep open-source-only shared-infrastructure tests beside the shared module rather than inside a provider directory.
- Prefer provider-local constants for official scopes, permissions, URLs, and API versions. Action `requiredScopes` should use provider-native scopes/capabilities, not private internal aliases.
- Avoid repeated action-name wiring. Define action handlers once and derive executor maps through shared provider runtime helpers when an existing helper fits. Do not add provider-local action-name unions, tuple builders, or casts solely to prove the handler keys to TypeScript.
- Do not import provider definitions from executor modules just to reuse metadata; inject catalog metadata from the server/loader side when needed.

### Shared owners providers must not re-clone

The generic runtime facts below have exactly one owner in this repository. Call the owner; do not add a provider-local copy, alias, rename, or narrower variant of it. Provider-local helpers are for provider meaning only: URL construction, request signing, pagination, response envelopes, provider error extraction, and output normalization.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oomol-lab/open-connector](https://github.com/oomol-lab/open-connector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
