---
trigger: always_on
description: This file is the single source of truth for any AI agent (or human) modifying this repo. Read it top-to-bottom before touching code. If something you learn here contradicts what you see in the code, the **code wins** — update this file in the same commit.
---

# AGENTS.md — working notes for coding agents (and humans)

This file is the single source of truth for any AI agent (or human) modifying this repo. Read it top-to-bottom before touching code. If something you learn here contradicts what you see in the code, the **code wins** — update this file in the same commit.

User-facing install / usage documentation lives in [`README.md`](./README.md). Do **not** duplicate it here.

---

### Purpose

One plugin, one job: make `opencode` talk to Kimi's `kimi-for-coding` endpoint **exactly the way the official `kimi-cli` does**. Everything in this repo exists to minimize drift from upstream kimi-cli.

### The one rule that matters

> Moonshot's backend picks the model (K2.5 vs K2.6) from the **auth token type**, not the model-name string.

- Static `sk-kimi-...` API key → K2.5.
- OAuth JWT with `scope: kimi-code` → K2.6.

Every design decision here follows from that: we do device-flow OAuth, we do not accept API keys, we do not let the upstream SDK attach its own Authorization header.

### Non-goals

- No support for K2.5 or any non-`kimi-for-coding` model. opencode already handles those via Moonshot / Baseten / Alibaba-CN / etc.
- No support for static API keys. Users who want that can use a different opencode provider entry.
- No custom SSE parser, tool-call normalizer, or message rewriter. `@ai-sdk/openai-compatible` already does SSE/`reasoning_content` correctly.

---

### Architecture

Three files, 1 job each. Do not add a fourth unless the existing three genuinely can't hold a new concern.

| File               | Responsibility                                                                 |
|--------------------|--------------------------------------------------------------------------------|
| `src/constants.ts` | Pinned strings that must mirror upstream kimi-cli (version, endpoints, client id, scope). |
| `src/headers.ts`   | The seven `X-Msh-*` / UA headers + the persistent `~/.kimi/device_id` file.    |
| `src/oauth.ts`     | Device-code start, device-code poll, refresh-token exchange, and `GET /coding/v1/models` discovery. |
| `src/index.ts`     | Plugin entry. Wires `auth` (login + loader) plus the Kimi chat hooks/body rewrite. |

Data flow on a chat request:

1. opencode asks the `@ai-sdk/openai-compatible` provider for a language model.
2. Before instantiating it, opencode calls our `auth.loader`. We return `{ apiKey, fetch }`.
3. The SDK uses our `fetch` for every HTTP call (models, chat, whatever).
4. Our `fetch` calls `ensureFresh()` → prefers the live opencode auth-store entry over stale `OPENCODE_AUTH_CONTENT` snapshots → maybe refreshes (sharing one in-flight promise in-process and a lock across plugin instances so they don't race the same refresh token) → lazily discovers `/coding/v1/models` when needed → sets Authorization + the seven `X-Msh-*` headers → on 401 refreshes once and retries.
5. Separately, opencode runs `chat.headers` and `chat.params`. `chat.headers` computes `thinking`, `reasoning_effort`, and `prompt_cache_key` from `input.model.options` plus the selected `input.message.model.variant`, then passes them to `loader.fetch` via private `x-opencode-kimi-*` headers. `loader.fetch` strips those headers and injects the wire fields into the JSON body. `chat.params` mirrors the same keys into `output.options` only as a forward-compat fallback if opencode later fixes its openai-compatible providerOptions namespace mismatch.

### Contracts to keep intact

These are the invariants that, if broken, silently degrade K2.6 → K2.5 or produce fingerprint-based throttling. Do not "clean them up" without reading the linked upstream.

1. **`X-Msh-Version` and `User-Agent` must track `kimi-cli`.** Bumping involves exactly one line in `src/constants.ts`. See upstream `research/kimi-cli/src/kimi_cli/constant.py`. The UA prefix is `KimiCLI/` (not `KimiCodeCLI/`) — Moonshot's `kimi-for-coding` backend 403s with `access_terminated_error: only available for Coding Agents such as Kimi CLI, Claude Code, Roo Code…` on any other prefix. Likewise, `X-Msh-Device-Model` must mirror kimi-cli's `_device_model()` shape, including the Darwin/Windows special cases (`macOS <version> <arch>`, `Windows 10/11 <arch>`, Linux `"{system} {release} {machine}"`) — NOT just `{arch}` — and `X-Msh-Os-Version` is the kernel build string from `os.version()`, NOT `"{type} {release}"`. Tested live against `api.kimi.com/coding/v1` on 2026-04-17 — any of those three fields off-spec → 403.
2. **`X-Msh-Device-Id` must be stable across runs.** Never regenerate a fresh UUID at import time. `getDeviceId()` reads/writes `~/.kimi/device_id`; that path is shared with `kimi-cli` on purpose.
3. **`Authorization` header is owned by `loader.fetch`.** Anything else (opencode core, the SDK, future hooks) must be overridden. Our `loader` deletes both `authorization` and `Authorization` before setting its own. The private `x-opencode-kimi-*` transport headers are also consumed and stripped there; they must never leak upstream.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lemon07r/opencode-kimi-full](https://github.com/lemon07r/opencode-kimi-full) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
