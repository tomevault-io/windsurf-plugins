---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Railway deployment wrapper for **Openclaw** (an AI coding assistant platform). It provides:

- A web-based setup wizard at `/setup` (protected by `SETUP_PASSWORD` when set)
- Automatic reverse proxy from public URL → internal Openclaw gateway (requires `SETUP_PASSWORD` when set)
- Persistent state via Railway Volume at `/data`
- One-click backup export of configuration and workspace

The wrapper manages the Openclaw lifecycle: onboarding → gateway startup → traffic proxying.

## Development Commands

```bash
# Local development (requires Openclaw in /openclaw or OPENCLAW_ENTRY set)
npm run dev

# Production start
npm start

# Syntax check
npm run lint

# Local smoke test (requires Docker)
npm run smoke
```

## Docker Build & Local Testing

```bash
# Build the container (builds Openclaw from source)
docker build -t openclaw-railway-template .

# Run locally with volume
docker run --rm -p 8080:8080 \
  -e PORT=8080 \
  -e SETUP_PASSWORD=test \
  -e OPENCLAW_STATE_DIR=/data/.openclaw \
  -e OPENCLAW_WORKSPACE_DIR=/data/workspace \
  -v $(pwd)/.tmpdata:/data \
  openclaw-railway-template

# Access setup wizard
open http://localhost:8080/setup  # password: test
```

## Architecture

### Request Flow

1. **User → Railway → Wrapper (Express on PORT)** → routes to:
   - `/setup/*` → setup wizard (auth: Basic with `SETUP_PASSWORD` when set; returns 500 if unset)
   - All other routes → proxied to internal gateway (auth: Basic with `SETUP_PASSWORD` when set; returns 500 if unset)

2. **Wrapper → Gateway** (localhost:18789 by default)
   - HTTP/WebSocket reverse proxy via `http-proxy`
   - Automatically injects `Authorization: Bearer <token>` header

### Security architecture (wrapper auth, then token injection)

The intended design is **wrapper-level auth first**, then token injection:

```
Internet → Railway → Wrapper (auth: Basic SETUP_PASSWORD) → [if auth OK] inject gateway token & proxy → Gateway → Agent
```

- **Wrapper** enforces authentication (Basic auth with `SETUP_PASSWORD`) on all proxy and Control UI routes before any request reaches the gateway. No unauthenticated traffic is proxied.
- **Gateway token** is injected by the wrapper only after the incoming request has been authenticated. Clients never need to know the gateway token; the wrapper acts as a trusted intermediary.
- So the gateway’s token check is not “bypassed”—it still validates the injected token. The wrapper ensures only authenticated users can trigger those requests.

### Lifecycle States

1. **Unconfigured**: No `openclaw.json` exists
   - All non-`/setup` routes redirect to `/setup`
   - User completes setup wizard → runs `openclaw onboard --non-interactive`

2. **Configured**: `openclaw.json` exists
   - Wrapper spawns `openclaw gateway run` as child process
   - Waits for gateway to respond on multiple health endpoints
   - Proxies all traffic with injected bearer token

### Key Files

- **src/server.js** (main entry): Express wrapper, proxy setup, gateway lifecycle management, configuration persistence (server logic only - no inline HTML/CSS)
- **src/public/** (static assets for setup wizard):
  - **setup.html**: Setup wizard HTML structure
  - **styles.css**: Setup wizard styling (extracted from inline styles)
  - **setup-app.js**: Client-side JS for `/setup` wizard (vanilla JS, no build step)
- **Dockerfile**: Multi-stage build (builds Openclaw from source, installs wrapper deps)

### Environment Variables

**Required (for zero-touch):** AI_PROVIDER, AI_API_KEY, TELEGRAM_*, etc. (see README)

**Recommended:**

- `SETUP_PASSWORD` — when set, protects `/setup` and gateway/Control UI (/, /openclaw) with Basic auth. When **not** set, those routes are disabled (return 500) and a prominent startup warning is logged; the deployment is not publicly accessible for setup or Control UI.

**Recommended (Railway template defaults):**

- `OPENCLAW_STATE_DIR=/data/.openclaw` — config + credentials
- `OPENCLAW_WORKSPACE_DIR=/data/workspace` — agent workspace

**Optional:**

- `OPENCLAW_GATEWAY_TOKEN` — auth token for gateway (auto-generated if unset)
- `PORT` — wrapper HTTP port (default 8080)
- `INTERNAL_GATEWAY_PORT` — gateway internal port (default 18789)
- `OPENCLAW_ENTRY` — path to `entry.js` (default `/openclaw/dist/entry.js`)

### Authentication Flow

The wrapper manages a **two-layer auth scheme**:

1. **Setup wizard auth**: Basic auth with `SETUP_PASSWORD` (src/server.js:190)
2. **Gateway auth**: Bearer token with multi-source resolution and automatic sync
   - **Token resolution order** (src/server.js:25-55):
     1. `OPENCLAW_GATEWAY_TOKEN` env variable (highest priority) ✅
     2. Persisted file at `${STATE_DIR}/gateway.token`
     3. Generate new random token and persist
   - **Token synchronization**:
     - During onboarding: Synced to `openclaw.json` with verification (src/server.js:478-511)
     - Every gateway start: Synced to `openclaw.json` with verification (src/server.js:120-143)
     - Reason: Openclaw gateway reads token from config file, not from `--token` flag
   - **Token injection**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Senpi-ai/senpi-hyperclaw-railway-template](https://github.com/Senpi-ai/senpi-hyperclaw-railway-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
