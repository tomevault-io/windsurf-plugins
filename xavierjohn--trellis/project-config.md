---
trigger: always_on
description: ﻿# GitHub Copilot Instructions for Trellis
---

﻿# GitHub Copilot Instructions for Trellis

## Project overview

Trellis is an AI-native framework that helps create consistent, reliable enterprise software with railway-oriented programming, DDD primitives, ASP.NET integration, EF Core integration, and value objects.

These instructions are for repository workflow and contribution conventions only. They are not the source of truth for how to use Trellis APIs.

## API usage source of truth

Before writing or changing code that uses Trellis APIs, read the relevant files in `docs/docfx_project/api_reference/`.

Start with `docs/docfx_project/api_reference/trellis-api-cookbook.md`. Use its task lookup table to find the right recipe, then read the package reference files for exact signatures, overloads, namespaces, and examples. Do not infer Trellis API behavior from these Copilot instructions.

### Recommended context size

The full set of API references is ~1,040 KB across 22 reference files (~266K tokens); the cookbook alone is ~228 KB (~58K tokens). These figures grow as the docs do — treat them as approximate; the current reference set is the `trellis-api-*.md` files under `docs/docfx_project/api_reference/`. Together with framework source needed for cross-checking, project source under edit, and accumulated tool output across a typical 30–50 turn session, the working set is **1.5–2.5 MB**.

| Tier | Context | When this is enough |
|---|---|---|
| **Minimum** | 200K | Narrow, single-file tasks. Forces a strict "load only the area-specific reference per task" discipline; cross-cutting work is error-prone at this tier. |
| **Recommended** | 400–500K | Most consumer projects. Lets the cookbook + 5–6 area-specific references stay resident through a PR-sized session. |
| **Comfortable** | 1M | Framework-internal work and greenfield projects with multiple integration points. Lets all 22 references stay resident from turn 1 without eviction. |

### Mandatory loads at session start

For any non-trivial Trellis work, load these **before** writing the first line of code or running the first sub-agent:

1. `trellis-api-cookbook.md` — always. Its task lookup table is the entry point.
2. `trellis-api-servicedefaults.md` — always. **Most** pipeline-module `services.AddXxx()` extensions have a matching `TrellisServiceBuilder.UseXxx()` slot — but leaf/store registrations are deliberate exceptions with **no** slot (e.g. `AddInMemoryIdempotencyStore`, `AddTrellisRouteConstraint`/`AddTrellisRouteConstraints`). Designing or modifying a registration helper without reading this file either silently misses a builder slot that should exist, or wrongly adds one where none belongs.
3. The area-specific reference for the package being modified (from the table below).
4. The reference for **every package whose pipeline this work composes with**. Specifically: anything touching the Mediator pipeline must also load `trellis-api-efcore.md` (transactional behavior) and `trellis-api-authorization.md` (resource-authorization behavior); anything touching ASP must also load `trellis-api-mediator.md`.

| When touching... | Read first |
|---|---|
| Result, Maybe, Error, ROP operations, aggregates, entities, specifications | `docs/docfx_project/api_reference/trellis-api-core.md` |
| Ready-to-use value objects and primitive attributes | `docs/docfx_project/api_reference/trellis-api-primitives.md` |
| Choosing a value-object category (scalar / symbolic / structured / optional) | `docs/docfx_project/api_reference/trellis-value-object-taxonomy.md` |
| ASP.NET Core response mapping, validation, ETags, Prefer handling | `docs/docfx_project/api_reference/trellis-api-asp.md` |
| ASP.NET Core API versioning (versioned `Location`/route + pagination URLs) | `docs/docfx_project/api_reference/trellis-api-asp-apiversioning.md` |
| EF Core integration | `docs/docfx_project/api_reference/trellis-api-efcore.md` |
| Transactional outbox and domain/integration event publishing | `docs/docfx_project/api_reference/trellis-api-efcore-outbox.md` |
| Authorization | `docs/docfx_project/api_reference/trellis-api-authorization.md` |
| FluentValidation integration | `docs/docfx_project/api_reference/trellis-api-fluentvalidation.md` |
| HttpClient extensions | `docs/docfx_project/api_reference/trellis-api-http.md` |
| HTTP transport abstractions (`WriteOutcome`, `HttpError`, ETag/precondition value types) | `docs/docfx_project/api_reference/trellis-api-http-abstractions.md` |
| Mediator pipeline behaviors | `docs/docfx_project/api_reference/trellis-api-mediator.md` |
| FluentValidation in the Mediator pipeline | `docs/docfx_project/api_reference/trellis-api-mediator-fluentvalidation.md` |
| State machine integration | `docs/docfx_project/api_reference/trellis-api-statemachine.md` |
| Service defaults and composition root setup | `docs/docfx_project/api_reference/trellis-api-servicedefaults.md` |
| Testing helpers | `docs/docfx_project/api_reference/trellis-api-testing-reference.md` |
| ASP.NET Core integration-test helpers | `docs/docfx_project/api_reference/trellis-api-testing-aspnetcore.md` |
| Worker / `BackgroundService` test harness | `docs/docfx_project/api_reference/trellis-api-testing-worker.md` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xavierjohn/Trellis](https://github.com/xavierjohn/Trellis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
