---
trigger: always_on
description: Index and search your entire digital life. Fully local, fully private.
---

# Omnesis

Index and search your entire digital life. Fully local, fully private.

CLAUDE.md is the **agent contract** for working in this repo. It is kept intentionally short. The public docs (under `website/docs/`) are deliberately concise and user-facing; for architectural or behavioral detail, **read the code** — it is the only complete reference.

## Where to read what

All paths below are local files. Use `Read` (or `Grep` to find the right page), not URLs.

| Topic                                                                                                                                                  | Where                               |
| ------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------- |
| Engineering conventions (façade splits, defineSource, zod-at-boundary, …) — read first when establishing or imitating a pattern                        | `docs/conventions.md`               |
| iOS-specific agent rules (XcodeGen, device deploy, SwiftUI preview + snapshot self-critique loop, TestFlight) — read before editing anything in `ios/` | `ios/AGENTS.md` (= `ios/CLAUDE.md`) |
| Public docs — what users see; update alongside behavior changes (see § Documentation maintenance)                                                      | `website/docs/`                     |
| Architecture, per-source detail, HTTP API, config knobs                                                                                                | the code (repo map below)           |

The repo at a glance:

```
packages/core/         shared types, branded IDs, define-source helpers, people-utils, attachments, the shared doctor health check (`@omnesis/core/doctor`)
packages/config/       config schema (omnesisConfigSchema, zod validators, defaults)
packages/gateway-client/ shared HTTP + WS GatewayClient implementation (HttpGatewayClient, GatewayWsClient) — extracted from collector
packages/gateway/      HTTP server, SQLite store, indexer, scheduler, search, analytics-db
packages/collector/    sync engine, source manager, auth subprocess
packages/cli/          unified `omnesis` CLI — read commands hit /search, /documents, /analytics, /status, /whoami; admin commands hit /admin/*; also hosts the daemons (`gateway serve` / `collector run`) and the `service`/`update` lifecycle commands
packages/providers/*   one package per provider (google, apple, notion, whatsapp, …)
packages/watch/     the Watch V2 DSL, validator, runtime and compiler — standalone by contract, imports nothing from the gateway; the gateway hosts it through `packages/gateway/src/watch/`
packages/agent-integration/ shared off-host integration runtime for OpenClaw and Hermes — transcript ingestion, subscription delivery, and scoped management tools
extension/             Chrome MV3 browser-capture extension — pairs as a `browser` device with a `write:web` token and pushes visited pages to the gateway-hosted `web` source; store listing and privacy declarations under `extension/store/`, packaging in `extension/scripts/` (see docs/releasing.md)
scripts/release/       publish pipeline — stage packages, transform src-pointing manifests to dist at publish time (see docs/releasing.md)
ios/                   native iPhone app — pairs with the gateway, hosts Apple Health
website/               static site Cloudflare publishes to omnesis.dev — landing page, public docs (website/docs/), privacy policy, installer mirror
docs/                  internal design docs, process docs (e.g. issue-labels.md)
```

## Documentation maintenance — your obligation

The public docs are fourteen hand-written static HTML pages under `website/docs/`, published with the landing site at https://omnesis.dev/docs (the `site` workflow deploys `website/` to Cloudflare on every push to `main` that touches it — the site workflow verifies the generated blog, with no generated documentation reference). `website/docs/docs.css` + `docs.js` carry the shared chrome (design tokens extracted from `website/index.html`); every page embeds the same nav / sidebar / footer, so structural changes must be applied to all pages.

When you change user-visible behavior — defaults, CLI commands, source semantics, setup flows, new or removed features — check the affected page and update it **in the same commit**:

| Page                                 | Covers                                                                                                                                                   |
| ------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `website/docs/index.html`            | what Omnesis is; core concepts (gateway, collector, devices & tokens and scopes, sources, index, people & links, analytics); data layout; help & support |
| `website/docs/install.html`          | installer, Docker, from-source, first run, uninstall                                                                                                     |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omnesis-dev/Omnesis](https://github.com/omnesis-dev/Omnesis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
