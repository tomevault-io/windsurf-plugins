---
trigger: always_on
description: This file provides guidance when working with code in this repository.
---

# Agents.md

This file provides guidance when working with code in this repository.

## Read the Patterns Docs First (non-negotiable)

`docs/patterns/` is the normative spec for this codebase, not background reading. Code that contradicts a pattern doc is wrong even if it compiles, passes tests, and looks like the file next to it — neighboring code may predate the doc.

**Before writing a line of code, open the docs for the layers you are about to touch.** Do not infer conventions by copying a nearby file; a plausible-looking imitation of the wrong pattern is the single most common failure mode in this repo, and it gets caught in review rather than by the compiler. If a doc and existing code disagree, the doc wins — follow the doc and flag the drift.

| If you are... | Read first (in `docs/patterns/`) |
|---|---|
| Changing **any** existing public request/response shape or behavior | `api-versioning-patterns.md` — and see the breaking-change rule below |
| Adding or changing an endpoint's request/response shape | `api-resource-conventions.md`, `nullable-field-patterns.md` |
| Writing a service, mediator, repository, or transaction | `architecture-patterns.md` |
| Touching identity, permissions, or actor checks | `authentication-patterns.md`, `authorization-check-patterns.md` |
| Adding or changing a list endpoint, filter, or sort | `performant-list-endpoint-patterns.md` |
| Adding a domain model, mock, or entry point | `domain-layer-patterns.md` |
| Emitting or changing audit events | `audit-event-patterns.md` |
| Adding a new entity or ID prefix | `entity-id-patterns.md` |
| Adding or changing an enum | `constants-enum-patterns.md` |
| Writing or fixing e2e tests | `e2e-test-patterns.md` |
| Writing or editing any comment | `comment-conventions.md` |
| Changing config structs | `config-patterns.md` |
| Touching logging or tracing | `canonical-log-patterns.md` |
| Touching production step graphs | `production-step-graph-patterns.md` |

Read the doc **before** you write, not after review. Reading the section you need is enough; you do not have to read a doc end to end.

**Self-check before reporting work as done:** name the pattern docs that govern the files you changed, re-read the relevant sections, and confirm each rule actually holds in your diff. "I didn't know that doc existed" is not an acceptable outcome — the table above is the index, and `important-patterns.md` is the short list of rules that apply everywhere.

## Breaking Changes Require a New API Version (non-negotiable)

**Any breaking change to the public API requires a new API version plus a version transformer that preserves the old shape.** A client pinned to a supported `OpenMRP-Version` must keep receiving byte-compatible request and response shapes for as long as that version is supported. Shipping a shape change without a transformer silently breaks every pinned consumer — that is never an acceptable trade for a smaller diff, and it is not something to "follow up on later."

Breaking — needs a new version **and** a transformer:

- Removing or renaming a request or response field.
- Changing a field's type, format, nullability, or enum value set.
- Moving previously-unconditional data behind an `?include=` key.
- Changing a default, a status code, an error `type`/`param`, or pagination semantics.
- Tightening validation so a previously-accepted body is now rejected.
- Changing the meaning of an existing field or parameter, even at an identical shape.

Not breaking — ships straight on `Latest`: new endpoints, new **response** fields, new **optional** request fields or query parameters, new include keys, new values on a field documented as an open enum, and bug fixes where the documented contract was always the fixed behavior.

**When in doubt, treat it as breaking.** An unnecessary transformer costs little; a silently broken consumer costs a lot.

The backend only ever speaks `Latest` — no version conditionals anywhere below the gateway edge. Older versions exist purely as edge transformations in `services/api-gateway/internal/versiontransforms/`. A shape-changing change is not complete until all of these ship with it:

1. The new version declared in `shared/version/version.go` (constant added, prepended to `Supported`, `Latest` repointed) with `version_test.go` updated.
2. The transformer(s), `FromVersion` = new and `ToVersion` = previous, with unit tests covering single, list, nested, and data-missing payloads.
3. `ObjectType` declared on **every** endpoint that returns or accepts the changed resource — an endpoint without it is invisible to the transformer chain and leaks the new shape.
4. `ForcedIncludes` wherever the old shape needs data the new shape gates behind an include.
5. Version-compat e2e tests (`tests/e2e/api/version_compat_<resource>_test.go`) pinned to the previous version.
6. `make openapi` re-run and the regenerated spec committed.

Transformers reshape real data and **never** fabricate values, and a transformer is immutable once its `ToVersion` has shipped. `docs/patterns/api-versioning-patterns.md` is the full checklist — follow it, don't approximate it.

## Build & Development Commands

```bash
make dev                      # Start local dev environment (Tilt + K8s)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-mrp/api](https://github.com/open-mrp/api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
