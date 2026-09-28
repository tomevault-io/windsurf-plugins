---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`jev4k` is a Kotlin DSL and client for TypeSafe's Jev "System One" model, built on the Ktor client (CIO engine) and
kotlinx.serialization. You describe a state and typed questions (Noul, Choice, Score), and it returns typed answers. The
library is `com.pambrose.jev4k` under `src/main/kotlin`. A runnable example is
`src/test/kotlin/com/pambrose/jev4k/examples/TriageExample.kt`.

## TypeSafe / Jev API docs

This project works with TypeSafe's Jev model (`POST https://api.typesafe.ai/v1/systemone`). A cached, compressed copy of
its docs lives in `jev-docs/`. Read `jev-docs/jev-api.md` before writing or changing any code that calls the API. Use
`jev-docs/sdks.md` for client behavior (retries, timeouts, errors, typing) and `jev-docs/cookbooks.md` for proven
question designs. The raw pages under `jev-docs/pages/` aren't in git, because they're TypeSafe's copyrighted docs. Run
`make api-docs` (which runs `jev-docs/refresh.sh`) to download them after cloning. The notes are committed. Don't use
the legacy preview API shape (`/preview/evaluation`, `document`, `prompts`); `jev-api.md` §9 lists every renamed field.

## Architecture

Type names follow TypeSafe's JS SDK. A question definition is a `Question`, one of `NoulQuestion`, `ChoiceQuestion` or
`ScoreQuestion`, which hold only what is asked: instructions and criteria. A `QuestionRef<A>` is the handle code holds.
It pairs an id with a `Question` (`ref.question`) and a decoder for the typed answer.

There are two DSL layers over one core model. Both produce a validated `QuestionSet`, and
`JevApi.evaluate(state, questionSet, model)` is the only call that sends questions to the network. `JevApi.models()`
(`GET v1/models`) is the other network call.

- **Inline layer** (`Builders.kt`). `jev.query(state) { noul("id", "...") ... }` uses string ids. `QueryBuilder`
  functions return `QuestionRef<A>` handles, and `include(query)` merges a `JevQuery` into the same request.
- **Typed layer** (`JevQuery.kt`). `object X : JevQuery() { val urgent by noul("...") }`. A `PropertyDelegateProvider`
  takes the id from the property name (or `id =`) and registers questions in declaration order. `JevQuery.questions` is
  built lazily, so an invalid definition fails on first use. `choice<E>()` builds options from an enum. The option key
  is the constant's name unless `JevOption.optionKey` overrides it, and `JevOption.entry` becomes the description.
- **Answers** (`JevResult.kt`, `Answers.kt`). `result[handle]` decodes through the handle's `decode` function, and also
  checks that the handle belongs to the request. By-id accessors (`noul`/`choice`/`score`/`enumChoice`) check that the
  question type matches. Raw answers are stored string-keyed. Enum choices are mapped when read, and an unknown option
  becomes `JevResponseValidationException`.
- **Wire** (`internal/Wire.kt`).
    - Requests are `@Serializable` DTOs, and a sealed `WireQuestion` writes the `"type"` discriminator.
    - `JevJson` sets `encodeDefaults = false`, so unset optional fields (Noul criteria) are omitted. Explicit JSON
      `null`s, such as an undescribed Choice option, are still sent.
    - Caller-supplied `@Serializable` state and entries are encoded with `ValueJson` (`encodeDefaults = true`), so
      default-valued fields reach the model.
- **Response mapping** (`internal/ResponseMapper.kt`).
    - Responses are parsed by hand from `JsonObject`, so every error has a field path such as `answers.<id>.noul`.
    - An answer with no `type` is read as the type of question that was asked; an unknown type becomes `UnknownAnswer`.
    - Choice probabilities are reordered to the order the options were declared. Score keys `"0".."n"` become `Int`.
    - Absent answers fail when they are read, not when the response is parsed.
- **Client** (`JevClient.kt`, `internal/HttpClientFactory.kt`, `internal/Retry.kt`).
    - `HttpRequestRetry` reproduces the official SDKs' retry rules. `RetryPolicy` sets them: 408/429/5xx, connection
      errors, timeouts, 0.5 s doubling to 5 s with 25% jitter, and `retry-after-ms`/`Retry-After` hints up to 60 s.
    - `HttpRequestRetry` must be installed **before** `HttpTimeout`, otherwise one timeout cancels every retry.
    - `expectSuccess = false`: non-2xx responses map to `JevApiException` subclasses (`apiException` in `Errors.kt`)
      after retries run out, keeping the raw body and the `x-typesafe-request-id` header.
    - `BlockingJev` (`jev.blocking`) wraps the suspend API in `runBlocking`.
- **Config** (`JevConfig.kt`). Each setting resolves as explicit value, then env var, then default; blank env values are
  ignored. The env vars are `TYPESAFE_API_KEY` (required), `TYPESAFE_BASE_URL` and `TYPESAFE_DEFAULT_MODEL`. Internal
  hooks (`env`, `retryDelay`, `random`) make tests deterministic.
- **Validation** (`Questions.kt`). Every problem is collected into one `JevValidationException` before anything is sent:
  at least one question, unique non-blank ids, non-empty instructions, 1..255 Choice options, 2..10 Score levels.

## Documentation site


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pambrose/jev4k](https://github.com/pambrose/jev4k) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
