---
trigger: always_on
description: <sub>Last reviewed: 2026-05-29.</sub>
---

# AGENTS.md

<sub>Last reviewed: 2026-05-29.</sub>

This file is the single source of truth for AI coding agents and human contributors working on the `@bitrix24/b24jssdk` repository. The four detailed guides under `.github/contributing/` are referenced from the relevant sections below — load them only when they apply to your task.

## Project Overview

`@bitrix24/b24jssdk` is a JS/TS SDK for the Bitrix24 REST API. It is a pnpm 11 monorepo that ships:

- a framework-agnostic core SDK (`packages/jssdk`, published as `@bitrix24/b24jssdk`, ESM + UMD only since v0.4.0),
- a thin Nuxt module wrapper (`packages/jssdk-nuxt`),
- a public docs site (`docs/`, deployed to GitHub Pages),
- manual smoke playgrounds (`playgrounds/cli`, `playgrounds/nuxt`),
- workspace-level Vitest projects under `test/` that hit a real Bitrix24 portal.

The core SDK exposes three concrete entry points over a shared abstract base — `B24Frame` (iframe apps), `B24Hook` (server-side webhooks), `B24OAuth` (local OAuth apps) — plus cross-cutting modules for HTTP, limiting, helpers, push (Pull), logging, and types.

## Project Structure

```
packages/
├── jssdk/                          # core SDK (published)
│   ├── src/
│   │   ├── core/                   # AbstractB24, Result, SdkError
│   │   │   ├── actions/            # b24.actions.vX.<call|batch|batchByChunk|callList|fetchList>.make()
│   │   │   ├── http/               # transports + limiters
│   │   │   │   ├── limiters/       # rate / operating / adaptive delay
│   │   │   │   ├── ajax-result.ts
│   │   │   │   ├── ajax-error.ts
│   │   │   │   ├── v2.ts
│   │   │   │   └── v3.ts
│   │   │   └── tools/              # internal helpers
│   │   ├── frame/                  # B24Frame + iframe managers (auth, slider, dialog, …)
│   │   ├── hook/                   # B24Hook (server-side webhook auth)
│   │   ├── oauth/                  # B24OAuth (local OAuth apps)
│   │   ├── helper/                 # B24HelperManager, useB24Helper composable
│   │   ├── pullClient/             # WebSocket / long-poll Pull client
│   │   ├── tools/                  # public tools (Text, Type, Browser, formatters)
│   │   ├── types/                  # public types and enums
│   │   ├── logger/                 # LoggerBrowser / LoggerFactory
│   │   ├── loader-b24frame.ts      # initializeB24Frame()
│   │   └── index.ts                # public surface — treat as a contract
│   ├── README-AI.md                # caller-facing API guide (being absorbed into this guide)
│   └── build.config.ts             # unbuild config (replaces __SDK_VERSION__ etc.)
├── jssdk-nuxt/                     # Nuxt module — only registers runtime/plugin
playgrounds/
├── cli/                            # Node smoke
└── nuxt/                           # Nuxt smoke (live SDK)
docs/
└── content/docs/                   # documentation site (English only)
test/
├── 0_setup/                        # integration + under-load setup, env loading
├── integration/                    # *.spec.ts — 30s timeouts, real portal
├── under-load/                     # sequential, 40-min timeouts
├── some-code-from-docs/            # manually mirrored snippets from docs/ (not auto-run)
└── umd/                            # browser smoke (browser.html)
scripts/
└── b24-self-task/                  # Python automation that drives Claude Code
```

## Architecture

The core SDK is organised around one abstract base + three concrete entry points:

- [packages/jssdk/src/core/abstract-b24.ts](packages/jssdk/src/core/abstract-b24.ts) — `AbstractB24` exposes the action surface (`b24.actions.vX.<name>.make()`) and owns the v2 + v3 HTTP clients, the restriction/rate-limiter stack, the logger, and helper sub-managers. The legacy shortcuts (`callMethod`, `callBatch`, `callListMethod`, `fetchListMethod`, `callBatchByChunk`) live on this class too but are `@deprecated` — see the `@removed` tag on each method for the target removal version. Do not call them from new code.
- [packages/jssdk/src/frame](packages/jssdk/src/frame) — `B24Frame`, used inside the Bitrix24 iframe. Talks to the parent window via `postMessage`, auto-refreshes auth on 401, and exposes UI managers (`auth`, `parent`, `slider`, `dialog`, `placement`, `options`). **Bootstrap only via `initializeB24Frame()`** in [packages/jssdk/src/loader-b24frame.ts](packages/jssdk/src/loader-b24frame.ts) — it deduplicates concurrent inits and parses `window.name` for the portal handshake. Do not expose an alternative constructor path.
- [packages/jssdk/src/hook](packages/jssdk/src/hook) — `B24Hook` for server-side webhook auth (`B24Hook.fromWebhookUrl(url)`). **Must only be used in server-side (Node.js / edge runtime) code.** A webhook URL contains a secret access key; shipping it in a browser bundle exposes the key to every visitor. The class emits a runtime warning when it detects a browser context — do not suppress it with `offClientSideWarning()`.
- [packages/jssdk/src/oauth](packages/jssdk/src/oauth) — `B24OAuth` for OAuth-based local apps; manages refresh-token errors.

Cross-cutting modules:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bitrix24/b24jssdk](https://github.com/bitrix24/b24jssdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
