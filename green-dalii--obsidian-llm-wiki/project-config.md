---
trigger: always_on
description: **Last Updated:** 2026-06-29
---

# LLM Wiki Plugin Project Development Standards

**Last Updated:** 2026-06-29

---

## Current Phase: v1.22.5 (released) → v1.23.0 (Graph Engine direction)

### Completed (v1.22.5) — Hotfix: Responses API for #207 follow-up (2026-06-29)

Closed the second half of #207 — reasoning model family (gpt-5.1+ / gpt-5.5 / o1-o4) now uses OpenAI's Responses API, and the Test Connection Notice surfaces the provider's full error body (e.g. "insufficient_quota") instead of bare "status 429". #207 stays open for real-world user testing before final close.

- ✅ **#207 follow-up — Reasoning model family uses OpenAI Responses API.** v1.22.4's `max_tokens` ↔ `max_completion_tokens` probe was necessary but not sufficient — `gpt-5.1-chat-latest` / `gpt-5.5` / `o1` / `o3` / `o4-mini` still failed Test Connection with 400 because Chat Completions has compatibility issues for the reasoning family. Per OpenAI's official GPT-5.5 migration guide ("GPT-5.5 works best in the Responses API"), v1.22.5 routes the reasoning family to `/v1/responses` with `reasoning: { effort: 'low' }`. `gpt-5-chat-latest`, `gpt-4.1`, `gpt-3.5-turbo`, and all non-OpenAI baseUrls (Ollama, LM Studio, DeepSeek, etc.) continue on `/v1/chat/completions` unchanged. Detection is a pure-function `isResponsesApiModel(model, baseUrl)` export, gated to `https://api.openai.com/v1` only.
- ✅ **Test Connection Notice now surfaces the provider's full error body.** Obsidian's `requestUrl` throws on 4xx WITHOUT populating the Error with the provider body, so v1.22.4's `extractProviderErrorMessage()` couldn't see the actual diagnostic. v1.22.5 wraps the failing request in a `window.fetch` re-fetch (5s timeout) and merges the provider body into `Error.message` — users now see e.g. `"status 429: You exceeded your current quota, please check your plan and billing details"`. Raw body also logged at `console.warn` for DevTools investigation. Non-OpenAI baseUrls get the same enrichment via the existing Chat Completions path.
- ✅ **429/5xx now retry with exponential backoff on the Responses API path.** v1.22.4's `withRetry` (3 attempts, 1s/2s/4s + jitter) covered only the Chat Completions path. v1.22.5 wraps the new Responses API path in the same `withRetry` so transient 429 quota bumps no longer immediately fail Test Connection.
- ✅ **Tests: 1104 passing.** +28 since v1.22.4 (new `llm-client-responses-api.test.ts` with 28 tests covering endpoint routing, body shape, error enrichment, withRetry integration, custom baseUrl compatibility, and reasoning-family model coverage). Existing dot-naming gpt-5.x regression test (v1.22.4) and `thinking.type='disabled'` Chat Completions tests refactored to use `gpt-5-mini`/`gpt-5-nano`/`gpt-4.1` (the Chat Completions path models).

### Completed (v1.22.4) — Hotfix: GPT-5.x probe + provider error UX (2026-06-27)

Closed two user-reported issues in v1.22.3 user testing — both PATCH scope (backward-compatible bug fixes):

- ✅ **#207 — GPT-5.x models no longer fail Test Connection with 400.** v1.20.0's `params.model.startsWith('gpt-5-')` prefix-matching heuristic only matched the dash-suffixed OpenAI gpt-5 family (`gpt-5-mini`, `gpt-5-nano`, etc.) and silently broke for every new gpt-5.x release (`gpt-5.1`, `gpt-5.4-mini`, `gpt-5.5`). This was a regression of the same root-cause class as #143 in v1.20.0. Replaced with a runtime probe-then-cache mechanism: first request uses `max_tokens`, if the backend rejects with 400 we cache the alternate key (`max_completion_tokens` or vice versa) and retry. New `MaxTokenKey` type and `detectRejectedMaxTokenKey()` exported pure function. Stream path mirrors the same pattern in `createMessageStream`. Per-client isolation ensures baseUrl changes start a fresh cache.
- ✅ **Test Connection UI now surfaces the provider's actual error message.** Previously, `requestUrl` errors were re-wrapped as `status 400: ${data.error.message}` (or just "status 400" when the response body was lost to requestUrl's 4xx-throw-without-body behavior), and the provider's actual diagnostic was never visible. New `extractProviderErrorMessage()` enriches the thrown error in both `createMessage` and `createMessageStream` so Test Connection Notice text reads `status 400: <provider message>` instead of a generic HTTP wrapper. Test Connection is now self-diagnostic without needing the console.
- ✅ **Lint performance knobs centralised in `src/constants.ts`.** Yield cadences, candidate batch sizing, prep batch read, and source-analyzer batch sizing now live in one place. Previously these values were duplicated across `controller.ts`, `duplicate-detection.ts`, `preparation.ts`, and `batch-limits.ts` — including a literal `MAX_TOKENS=16000` copy of `MAX_TOKENS_BATCH`. Tuning lint performance is now a single-file change.
- ✅ **Tests: 1076 passing.** +12 since v1.22.3.

### Completed (v1.22.3) — Hotfix hardening (2026-06-26)
- ✅ **log header detection hardened to language-agnostic structural marker.** Switched from text-based detection (`view operation history` / `操作历史`) to embedded `<!-- llm-wiki-log-header-start -->` HTML-comment marker. v1.22.2 log files auto-upgrade on next startup.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [green-dalii/obsidian-llm-wiki](https://github.com/green-dalii/obsidian-llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
