---
trigger: always_on
description: Self-hosted multi-model AI chat platform powered by the official `@github/copilot-sdk`. A modern alternative to ChatGPT, Claude, and Gemini — with access to all Copilot models (GPT-4.1, o-series, Claude, Gemini) through a single interface. Users authenticate with GitHub Device Flow, pick a model, and chat over WebSocket with real-time token streaming.
---

# Copilot Unleashed — Project Instructions

## Project Overview

Self-hosted multi-model AI chat platform powered by the official `@github/copilot-sdk`. A modern alternative to ChatGPT, Claude, and Gemini — with access to all Copilot models (GPT-4.1, o-series, Claude, Gemini) through a single interface. Users authenticate with GitHub Device Flow, pick a model, and chat over WebSocket with real-time token streaming.

> See [docs/ARCHITECTURE.md](../docs/ARCHITECTURE.md) for full architecture details, data flow diagrams, and component inventory.

## Architecture

- **Full-stack**: SvelteKit 5 with `adapter-node` (replaces Express + vanilla JS)
- **Frontend**: 20 Svelte 5 components with 4 rune-based stores — dark theme, mobile-first
- **Real-time**: WebSocket via custom `server.js` entry, per-user `CopilotClient` lifecycle
- **Auth**: GitHub Device Flow only (no client secret, no redirect URI)
- **Session**: Express sessions bridged to SvelteKit via `x-session-id` header in `hooks.server.ts`
- **Skills**: `.github/skills/` directory with `SKILL.md` definitions, discovered server-side and passed to the SDK via `skillDirectories`
- **Settings**: Persisted server-side via `/api/settings` and mirrored in `localStorage`
- **Deployment**: Docker container → Azure Container Apps via `azd up`

### Deployment
- **Azure**: Key Vault (RBAC-only) for secrets, Basic ACR, EmptyDir volume; `COPILOT_CONFIG_DIR=/data/copilot-home`
- **Local Docker**: Bind mount `~/.copilot` preserved for CLI sync; `COPILOT_CONFIG_DIR` stays at `/home/node/.copilot`

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Runtime | Node.js 24 (node:24-slim in Docker) |
| Language | TypeScript 5.7 (strict mode, ES2022) |
| Framework | SvelteKit 5 with `adapter-node` |
| Reactivity | Svelte 5 runes ($state, $derived, $effect, $props) |
| AI Engine | `@github/copilot-sdk` ^0.1.32 |
| WebSocket | `ws` ^8.18 via custom server.js |
| Markdown | `marked` + `dompurify` + `highlight.js` (npm, bundled by Vite) |
| Security | Custom CSP/HSTS in hooks.server.ts, rate limiting, DOMPurify |
| Sessions | express-session bridged to SvelteKit locals |
| Build | `npm run build` (`tsc -p tsconfig.node.json` + `vite build`) → `build/` via adapter-node |
| Testing | Vitest (colocated `*.test.ts`) + Playwright (Desktop Chrome, Pixel 7, iPhone 14) |
| Container | Multi-stage Dockerfile (builder + runtime) |
| IaC | Bicep (Azure Container Apps) |

## Project Structure

```
server.js                       # Custom entry: HTTP + express-session + WebSocket + SvelteKit handler
.github/skills/                 # SDK skill definitions (currently 1 skill)
└── git-commit/SKILL.md         # Built-in git commit workflow skill
tests/                          # Playwright E2E suites and helpers
svelte.config.js                # SvelteKit config (adapter-node)
vite.config.ts                  # Vite config
vitest.config.ts                # Vitest config for colocated unit tests

scripts/
└── generate-vapid-keys.mjs     # VAPID key generation utility for push notifications
static/
├── manifest.json               # PWA manifest (name, icons, start_url, display: standalone)
└── sw.js                       # Service worker: precaching, push handler, notification click
infra/
├── modules/
│   ├── container-apps.bicep    # Container Apps Environment + App (Consumption, scale-to-zero)
│   ├── container-registry.bicep # Azure Container Registry (Basic SKU)
│   ├── key-vault.bicep         # Azure Key Vault (RBAC-only, secrets management)
│   ├── managed-identity.bicep  # User-assigned MI (AcrPull + Key Vault Secrets User)
│   └── monitoring.bicep        # Log Analytics workspace

src/
├── app.html                    # SvelteKit shell (viewport, theme-color, PWA meta)
├── app.css                     # Global reset, design tokens, highlight.js theme
├── hooks.server.ts             # Session bridge, CSP headers, rate limiting, CSRF, token revalidation
├── hooks.server.test.ts        # Example colocated unit test pattern
│
├── lib/
│   ├── components/             # 20 Svelte 5 components (see ARCHITECTURE.md)
│   ├── stores/                 # 4 rune stores: auth, chat, settings, ws
│   ├── server/                 # Server-only code
│   │   ├── auth/
│   │   │   ├── github.ts              # Device Flow OAuth (fetch-based)
│   │   │   └── guard.ts               # Auth middleware + 30-min token revalidation
│   │   ├── copilot/
│   │   │   ├── client.ts              # CopilotClient factory
│   │   │   └── session.ts             # Session config (model, reasoning, tools, hooks)
│   │   ├── ws/
│   │   │   ├── handler.ts             # WebSocket handler: 22+ message types, SDK events
│   │   │   └── session-pool.ts        # Per-user session pool with TTL + buffer
│   │   ├── push/
│   │   │   ├── subscription-store.ts  # Push subscription CRUD per user (10 sub cap)
│   │   │   └── sender.ts             # web-push wrapper with 60-min TTL, auto-cleanup

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devartifex/copilot-unleashed](https://github.com/devartifex/copilot-unleashed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
