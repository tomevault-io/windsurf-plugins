---
trigger: always_on
description: subrouter is like OpenRouter, but for **personal AI subscriptions**. Most users pay for several subscriptions (Claude Pro/Max, ChatGPT Plus/Pro, SuperGrok / Grok Build, Cursor, opencode Go). When one runs out of credits, the user has to stop working, switch models in every harness, and re-login constantly. subrouter fixes both problems:
---

# subrouter agent instructions

subrouter is like OpenRouter, but for **personal AI subscriptions**. Most users pay for several subscriptions (Claude Pro/Max, ChatGPT Plus/Pro, SuperGrok / Grok Build, Cursor, opencode Go). When one runs out of credits, the user has to stop working, switch models in every harness, and re-login constantly. subrouter fixes both problems:

- it cycles through multiple subscriptions when one runs out of credits
- it cycles between subscriptions of **different providers**, following the user's preferred ranking, and only errors when every subscription is exhausted

**Personal use only.** Using subrouter to serve tenant/third-party traffic is against the terms of use of most (if not all) subscription providers. Never add features that encourage multi-tenant serving.

## Core design principle: transparent API passthrough

subrouter must **never translate between AI wire formats**. This is the main robustness property of the project; protect it in every change.

- The router (`RouterModel`) delegates `doGenerate`/`doStream` directly to official `@ai-sdk/*` provider packages (`@ai-sdk/anthropic`, `@ai-sdk/openai`, `@ai-sdk/xai`, `@ai-sdk/openai-compatible`). The result streams/objects are returned to the harness **untouched**.
- Format understanding is the harness's job, wired through `@subrouter/opencode` and `@subrouter/pi`. OpenCode uses the AI SDK. Pi uses its native provider streams. Subrouter only picks *which* subscription serves the request.
- Adapters only touch requests where the subscription gateway **requires** it, and any rewrite must be reversed on the way out:
  - anthropic: OAuth traffic must look like Claude Code CLI (identity system block, tool-name renames, beta headers). The response stream maps tool names **back** to the originals, so the harness never sees the spoofing.
  - openai (Codex backend): `store: false` forced, `max_output_tokens` stripped, URL rewritten to `chatgpt.com/backend-api/codex/responses`. These are hard endpoint requirements (verified against the real API), not conveniences. Codex is stream-only.
  - xai / opencode zen / poe / minimax / kimi / zai / alibaba: bearer injection only.
  - github-copilot: bearer injection, API-family routing, and removal of the unsupported Anthropic tool-streaming field.
- The Pi plugin does not use these AI SDK request adapters. It supplies the selected account token to Pi's matching native provider, which owns the required request shape and returns native Pi events.
- Never add "smart" body transformations, prompt mutation, output post-processing, or cross-format proxying (no anthropic→openai translation like generic LLM proxies do). If a provider needs a new quirk, implement the **minimal** request patch in that provider's adapter fetch, document why, and keep everything else byte-transparent.

### Why protocol ownership stays in the harness

Subrouter is a **transparent subscription router**, not a general-purpose compatibility API. It chooses an account and model, then delegates the complete request and stream lifecycle to the protocol implementation that the harness already uses:

- OpenCode delegates through the matching official AI SDK provider. Subrouter returns the provider's `LanguageModelV3` result without converting it to another provider format.
- Pi delegates through `@earendil-works/pi-ai`'s matching native provider. Pi builds the provider request, parses SSE or WebSocket responses, and emits native Pi events. Subrouter forwards those events unchanged.

Do not add a unified message, tool, reasoning, usage, or streaming protocol inside Subrouter. General API gateways such as OpenRouter and CLIProxyAPI need bidirectional translators because they let one client protocol call many different provider protocols. That broader goal requires request converters, response converters, and stateful stream parsers for every supported format pair.

Subrouter deliberately avoids that translation matrix. Reusing the harness's provider implementations keeps the package small and removes whole classes of bugs involving tool calls, reasoning blocks, media, usage accounting, event ordering, and new provider fields. A provider-specific gateway requirement is still allowed, but patch only that requirement and never turn the adapter into a general format converter.

## Repo layout

pnpm workspace, flat `./*` packages. **One root README only, no per-package READMEs.**

- `cli/` — npm package `@subrouter/cli`. Everything lives here:
  - `src/store.ts` — accounts, presets, cooldown state under `~/.subrouter` (override: `SUBROUTER_HOME`). JSON files, 0600, lock-dir locking.
  - `src/adapters/` — one adapter per provider (login flow, token refresh, fetch wrapper, `createModel`). Shared failure classification in `adapters/index.ts`.
  - `src/router.ts` — `RouterModel` (AI SDK `LanguageModelV3`) + `createSubrouter` provider factory. Resolves a preset to ranked candidates, skips cooldowns, fails over on rotate-worthy errors.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [remorses/subrouter](https://github.com/remorses/subrouter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
