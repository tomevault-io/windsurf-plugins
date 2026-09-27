---
trigger: always_on
description: When a user asks to set up, configure, add, remove, update, or troubleshoot
---

# Agent-managed onboarding

When a user asks to set up, configure, add, remove, update, or troubleshoot
Bifrost Model Router, own the requirements gathering and implementation. Do not
require the user to understand Bifrost JSON, Docker networking, adapters, model
catalogs, or Codex provider configuration.

## Conversation contract

- Treat vague requests such as “set it up” as the start of requirements
  gathering, not as missing information or a reason to stop.
- Ask focused questions until the provider and plan scope is clear. Prefer one
  consolidated question at a time over a long questionnaire.
- First establish which provider plans or API accounts the user wants available
  in Codex. Support any Bifrost-native or OpenAI-compatible provider without
  advertising, recommending, or implying affiliation with specific downstream
  companies.
- After authenticated discovery, ask whether each provider should expose all
  available models or a model/family allowlist. Do not make users enumerate
  models before showing them what their account can access.
- Always retain OpenAI through the user's Codex login. This is an invariant,
  not a requirements question.
- Inspect `~/.config/bifrost-model-router`, the managed Codex block, and the
  named Docker containers to detect first-time versus additive setup. Preserve
  existing providers, credentials, and model entries by default. Ask only if
  the user explicitly requests removal or discovered state is contradictory.
- Treat the virtual key currently stored in the active Codex provider block as
  part of the existing state. A newly declared bootstrap key does not update an
  older persisted virtual key automatically.
- Default new Codex threads to `gpt-5.6-sol` with `medium` reasoning. Do not ask
  the user to choose a default. Change it only when the user explicitly asks or
  authenticated discovery proves it unavailable; in that case select the
  closest available OpenAI coding model and explain the fallback.
- Resolve informal names and likely typos through research, then confirm the
  interpretation instead of rejecting the request.
- Explain material constraints and tradeoffs in plain language. Ask for user
  input when a plan is ambiguous, API access is uncertain, provider
  documentation conflicts with account discovery, or a change would remove
  existing access.
- Never claim every provider is automatically compatible. Bifrost-native and
  OpenAI-compatible Chat Completions providers should normally work. A provider
  with another protocol or unusual authentication may require an adapter; if
  so, explain the gap and offer to implement or configure it.

## Requirements to discover

For each requested provider, determine:

1. Provider name and the exact subscription, coding plan, or API product.
2. Whether that plan includes API access and which API endpoint applies.
3. Credential type and a clear environment variable name.
4. Models actually available to that account and plan—not the provider's full
   marketing catalog.
5. Wire protocol: Bifrost native, OpenAI-compatible Chat Completions, or native
   Responses.
6. Verified model capabilities, context windows, modalities, reasoning levels,
   and tool support.
7. Existing local router state that must be merged and preserved.
8. The model permissions of the exact virtual key installed in Codex.

After authenticated, account-aware discovery, summarize the available models
and ask whether to expose all of them or only a subset. If the user wants a
subset, ask which exact models or model families should be visible. Do not ask
them to curate IDs before discovery, and do not silently expose everything
without confirming that choice.

Use current official provider documentation as the primary source. Use an
authenticated model-list endpoint when it is account-aware. If discovery is
global rather than plan-aware, intersect it with plan documentation. When
availability remains uncertain, perform a minimal model request after warning
that it may consume a small amount of plan quota.

Resolve context windows and other capability metadata independently from model
availability. Fields returned by the configured provider always take
precedence. When its model endpoint omits a context window, let the router use
the context length from an exact or unambiguous OpenRouter model-ID match.
Retain the conservative provider default only when neither dynamic source has
a trustworthy value. Do not hardcode per-model context windows in application
configuration merely to compensate for an incomplete model endpoint, and
never raise a provider-wide default merely because most models are larger.

Never invent model IDs, context windows, plan entitlements, or capabilities.
For an OpenAI-compatible provider with unverified richer capabilities, use the
conservative `chat_polyfill` and text-only catalog contract.

## Credential handoff

Never ask the user to paste a provider secret into chat, commit it, place it in
the Bifrost JSON, or place it in `~/.codex/config.toml`.

Create these local files outside the repository:

```text
~/.config/bifrost-model-router/config.json
~/.config/bifrost-model-router/providers.env
```

Create `providers.env` with mode `0600` and one empty assignment for each

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [applyinnovations/bifrost-model-router](https://github.com/applyinnovations/bifrost-model-router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
