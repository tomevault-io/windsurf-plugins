---
trigger: always_on
description: This file is read by AI coding assistants (Claude, Copilot, Cursor, etc.) so they follow the project's conventions. Humans contributing PRs should skim it once too.
---

# Contributor + AI-Assistant Guide

This file is read by AI coding assistants (Claude, Copilot, Cursor, etc.) so they follow the project's conventions. Humans contributing PRs should skim it once too.

## What this repo is

Cryptex OSS is a SvelteKit 2 + Svelte 5 + shadcn-svelte static-site app for LLM red-teaming techniques. The deliverable is a single-page browser app, served as static files. There is no backend, no database, and no auth; every piece of state lives in `localStorage`. The app exposes 25 tool routes (10 technique workbenches under `/transforms`, `/decode`, etc., plus 15 red-team labs under `/redteam/*`).

A Python CLI (`cryptex-cli`, managed with `uv`) shells out to Node to execute the canonical transformers in `src/transformers/`, so there is one source of truth for transforms: both the SvelteKit app and the CLI import the same 159 transformer files.

The v2.0 milestone hardened every tool surface with a unified `ToolShell` template, typed `CryptexError` taxonomy, Web Worker offloading for inputs at or above 50 KB, a per-tool Vault drawer (309 bundled OSS-licensed seed items + custom-add), and a persistent searchable session history via the v2 store. See the v2.0 conventions section below before adding new surfaces.

## Commands

### Primary (SvelteKit app)

```bash
cd app && npm install              # install deps
cd app && npm run dev              # Vite dev server, http://localhost:5173
cd app && npm run check            # svelte-kit sync + svelte-check (type-check)
cd app && npx vitest run           # run all unit tests
cd app && npx vitest run path/to/file.test.ts   # run a single test
cd app && npm run build            # static build → app/build/
```

### Python CLI (cryptex-cli)

```bash
uv run cryptex-cli list
uv run cryptex-cli inspect caesar --json
uv run cryptex-cli encode --transform base64 --text "Hello"
uv run cryptex-cli /base64 --decode SGVsbG8=          # slash-command form
uv run cryptex-cli auto-decode --text "SGVsbG8="
pytest                                                 # python_tests/
```

### Docker

```bash
docker compose up --build          # → http://localhost:8080
```

The image is `nginx:1.27-alpine` serving `app/build/`. Same `app/build/` as the static deploy path.

## Architecture

### Transformers (the 159 transforms — single source of truth)

- Live in `src/transformers/<category>/<name>.js` — **category = directory name** (ancient, case, cipher, encoding, fantasy, format, special, technical, unicode, visual).
- Each file does `export default new BaseTransformer({...})` from `src/transformers/BaseTransformer.js`.
- **Auto-discovered.** Adding a new file is sufficient; the registry picks it up.
- `src/transformers/loader-node.js` is the **Node-side** loader used by `scripts/cli_bridge.js` (the CLI's subprocess entry point). The web app uses Vite `import.meta.glob` instead. Same modules, two front-doors.
- `priority` (1–310) controls the universal decoder's ranking when auto-detecting format. See the priority guide at the bottom of `BaseTransformer.js`. Unique character sets (Binary, Morse, Braille) sit at 300; Unicode lookalikes default to 85; ciphers at 60; invisible-text at 1.

### Multi-provider AI gateway (BYOK)

All AI calls route through `app/src/lib/ai/gateway.ts`. It exposes:

- `chat(req) → ChatResponse`
- `streamChat(req)` — async iterator of stream chunks
- `fetchModels(signal)` — aggregates catalogs across every enabled provider
- `validateKey(providerId, candidate, opts)` — per-provider key probe
- `resolve(modelId)` — routes qualified ids (`openrouter:…`, `anthropic:…`, `openai-compat:<instance>/…`) to the right adapter; unqualified ids default to OpenRouter.

Provider config lives in `app/src/lib/ai/providers.svelte.ts` and is persisted under `cryptex.providers` in `localStorage`. Adapters in `app/src/lib/ai/adapters/` are lazy-imported.

Supported providers:
- **OpenRouter** (default, CORS-open)
- **Anthropic direct** (uses `anthropic-dangerous-direct-browser-access` header)
- **OpenAI-compatible endpoints** (Groq, Together, Fireworks, DeepInfra, Cerebras, SambaNova, custom)

Direct OpenAI / Google Gemini are **not supported** from the browser — no CORS. Users route those models through OpenRouter.

In dev mode (`npm run dev`), `app/vite.config.ts` defines a `/api/_proxy/<providerId>` proxy that forwards model-list and chat-completions requests server-side to each provider — sidestepping browser CORS on `/v1/models`. The dev-vs-prod URL resolution lives in `app/src/lib/ai/proxy-url.ts` (`effectiveBaseURL` / `effectiveDirectBaseURL`) and is consumed by the three adapters. Production static deploys do not include the proxy; direct fetches go to provider URLs and the per-preset `defaultModels` lists in `presets.ts` cover any `/models` endpoints that block CORS.

### Technique registry

`app/src/lib/techniques/registry.ts` aggregates everything available to the workbenches:

- 159 transformers (re-exported from the `src/transformers/` registry)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [m4xx101/cryptex-oss](https://github.com/m4xx101/cryptex-oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
