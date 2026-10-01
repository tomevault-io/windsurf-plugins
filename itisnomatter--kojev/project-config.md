---
trigger: always_on
description: A Kotlin Multiplatform client library for **Jev**, TypeSafe AI's System One model.
---

# kojev

A Kotlin Multiplatform client library for **Jev**, TypeSafe AI's System One model.

For first-time setup and the development phases, see `docs/bootstrap.md`.
The confirmed Jev API specification lives in `docs/api-notes.md` — read it before writing any implementation.

---

## Why this library exists

kojev takes a position on three things, and every design decision is judged against them:

1. **Kotlin Multiplatform.** `jvm`, Android, and the iOS targets, so that domain types and the
   DSL can be shared with app code.
2. **Answers come back as the caller's own types - for Score as well as Choice.** A Choice is the
   caller's enum. A Score is a distribution over the caller's enum rubric
   (`mostLikely: Urgency`, `probabilities: Map<Urgency, Double>`), not a level number plus a legend.
3. **There is exactly one, typed way to read an answer.** No string-id lookup, no default
   thresholds, no helper that rounds a Score's mean into a level. This is not a feature that is
   missing; it follows from hard rules 2 and 3. The judgement calls belong to the caller, and a
   convenience that makes one on the caller's behalf will be used.

Other Kotlin clients exist and are good at what they choose to be: `pambrose/jev4k` is a JVM
library with typed choices and many conveniences (string ids, bands, level helpers);
`ufec/typesafe-sdk-kotlin` is a faithful port of the official JavaScript SDK. kojev is the
multiplatform, deliberately narrow one. **Reject any design decision that erodes one of the three
points above** - an implementation that gives them up has no reason to exist next to those libraries.

Non-goals:
- Wrapping text generation (Jev does not generate)
- Prompt templates or agent frameworks
- A 1:1 port of the official SDK (`ufec/typesafe-sdk-kotlin` already does that)

---

## Hard rules

### 1. Never guess the API

Jev was released on 15 September 2026 and **does not exist in your training data**.
Writing endpoints, field names, or response shapes from memory is forbidden.

`docs/api-notes.md` is the source of truth. If you need something that is not recorded there,
look it up starting from `https://docs.typesafe.ai/llms.txt`, **update api-notes.md first**,
then implement. Mark any implementation whose source you cannot cite with a comment.

### 2. Do not confuse the three primitives

| Type | Question | Returns |
|---|---|---|
| **Choice** | Pick one option (`instructions` + a label→description map) | the label, a probability distribution, confidence |
| **Score** | Rate against a 2–10 level rubric | the score (a `Double`: the probability-weighted mean of the level numbers), the most likely level, a distribution, confidence |
| **Noul** | "Is this statement true?" | **a probability in 0..1. There is no confidence field** |

- **Noul is not a null check.** It returns the probability that a statement holds.
- **A Noul and a two-option Choice are not interchangeable.** Never carry a threshold between them.
- **A Score's answer is not a level.** The mean usually falls between levels and can land on a
  level the model gave zero probability. The library never rounds it into a level for the caller.
- An empty Choice `criteria`, or a Score with fewer than two levels, is a request-time error.

### 3. Do not weaken type safety

- `result[key]` takes a `QuestionKey<T>` and returns the answer typed as `T`
- **Do not provide string-key lookup, not even as a convenience API** — if it exists, it will be used
- A missing, mistyped, or out-of-range value in a response raises an exception.
  It is never silently turned into a default.
- The library does not pick default thresholds. That judgement belongs to the caller.

### 4. API key assumptions

The official SDK blocks direct use from browsers, because the key would be exposed.

- Treat `baseUrl` override as a first-class feature (proxies and gateways)
- **Never hardcode a key in sample code.** Read from the environment in every example.
- The iOS and Android targets exist so that domain types and the DSL can be shared with app code,
  not so that a device can call the API directly. (BYOK apps are the exception — distinguish the two in the README.)

### 5. Tests must pass without a key

Jev is in waitlisted early access. **Do not build something that stalls without a key.**

- Verify request construction and response parsing with Ktor's `MockEngine`, with no network access
- Base mock JSON on the real schema recorded in `docs/api-notes.md`. Never invent it.
- Put live-API tests in a separate source set, run only when `TYPESAFE_API_KEY` is set
  (skipped when absent — not a failure)
- A green `jvmTest` alone is not "tested". Run every target.

---

## Stack

- Kotlin 2.0+ with Coroutines
- Ktor Client (`ktor-client-core`) + `ktor-serialization-kotlinx-json`
- kotlinx.serialization

Targets: `jvm()`, `androidTarget()`, `iosArm64()`, `iosSimulatorArm64()`, `iosX64()`.
Keep platform-specific APIs out of `commonMain` so that `js(IR)` and `wasmJs` can be added later.

**Do not pin a Ktor engine.** Take no dependency on `ktor-client-okhttp` and friends; the caller injects one.

Transport requirements:
- Retry on 408 / 429 / 5xx with exponential backoff and jitter, honouring `Retry-After`. Make it configurable.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ItisNoMatter/kojev](https://github.com/ItisNoMatter/kojev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
