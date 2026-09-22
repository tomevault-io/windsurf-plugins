---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Kant is a serverless, end-to-end encrypted P2P messenger built on libp2p + libsodium. There is no central message store: peers connect directly (or via a stateless circuit-relay v2 bootstrap node when NAT/firewalls block direct dialing). The relay only ever sees encrypted noise packets and signed registry records — never plaintext.

pnpm monorepo, workspaces defined in `pnpm-workspace.yaml` (`packages/*` + `tests/lab`).

## Commands

Install (from repo root):
```bash
npx pnpm install
```

Typecheck (root `tsc --noEmit` over the whole `packages/**` tree, per `tsconfig.json` `paths` mapping `@kant/*` → `packages/*/src`):
```bash
pnpm run typecheck   # == pnpm run lint
```

Run the app in dev (Vite dev server only — relay must be started separately, see below):
```bash
pnpm run dev
# or: ./start.sh [--relay <url>] [--port <port>]
```

Build the web app:
```bash
pnpm run build
```

### Per-package builds/dev
Most packages are plain `tsc` builds; run from repo root with `pnpm --dir packages/<name> <script>` or `cd` into the package:
- `packages/core` — `pnpm run build` / `build:test` (see tests below)
- `packages/relay` — `pnpm run build`, `pnpm run dev` (runs TS directly via `ts-node/esm`), `pnpm run start` (runs compiled `dist/`)
- `packages/app` — `pnpm run dev`, `pnpm run build` (`tsc && vite build`), `pnpm run build:electron`, `pnpm run build:android[:debug]`
- `packages/cli` — `pnpm run build`, `pnpm run start`
- `packages/push-proxy` — `pnpm run build`, `pnpm run dev`, `pnpm run start`
- `packages/desktop` — Electron packaging; `pnpm run dev` runs the app + electron concurrently; `pnpm run build:*` variants bundle a `relay-runtime` deploy of `@kant/relay` plus the built app
- `packages/admin` — placeholder, not an active product surface

### Core package tests
`packages/core` has its own Node test suite (no framework — `node --test` + a libsodium loader shim). From `packages/core`:
```bash
pnpm test              # build, build:test, then run keypair/x3dh/groups/files tests
pnpm test:coverage      # same, with node --experimental-test-coverage and coverage thresholds
```
To run a single compiled test file directly (after `pnpm run build && pnpm run build:test`):
```bash
node --loader ./sodium-loader.mjs --import ./sodium-loader.mjs --test ./dist-test/groups.test.js
```
Test sources are colocated as `*.test.ts` next to the module under test (e.g. `groups.ts` / `groups.test.ts`) and compiled via `tsconfig.test.json` into `dist-test/`.

### Root-level lab / E2E tests
`tests/lab` is a disposable Docker-based lab (protocol, relay, network, browser UI, desktop smoke tests) — **not** run locally by default, designed to run on a dedicated test node:
```bash
pnpm run test:lab            # pnpm --dir tests/lab test (Playwright)
pnpm run test:lab:ui         # Playwright UI mode
pnpm run test:lab:preflight  # bash tests/lab/scripts/preflight.sh
```
See `tests/lab/README.md` for the Docker-compose based workflow (`tests/lab/scripts/run.sh lan|nat`). The lab never uses production identity/relay data and its ports must not be exposed to the Internet.

## Architecture

### Packages
| Package | Role |
| --- | --- |
| `packages/core` (`@kant/core`) | All crypto and protocol logic: identity, X3DH handshake + double-ratchet, group messaging, file transfer, onion routing, prekeys, contacts, IndexedDB-backed message/queue storage, Tor SOCKS5 transport, licensing. This is the shared library consumed by app/cli/desktop. |
| `packages/relay` (`@kant/relay`) | Stateless libp2p circuit-relay-v2 bootstrap node. Plain Node `http` server (no framework) exposing `/relay-info`, `/healthz`, `/readyz`, `/metrics` (Prometheus via `prom-client`), `/register`, `/lookup`, `/admin/*` (bearer-token gated), `/push/*`. Deterministic relay identity derived from a seed so its multiaddr is stable across restarts. |
| `packages/app` (`@kant/app`) | React 18 + Vite web client. Also the base for the Electron desktop build (`build:electron`) and Capacitor Android build (`build:android`). |
| `packages/desktop` | Electron shell wrapping `packages/app`'s build output plus a bundled `relay-runtime` (a `pnpm deploy` of `@kant/relay`), and an MCP/AI server (`ai-server.ts`, `mcp-server.ts`). |
| `packages/cli` (`@kant/cli`) | Terminal client (`blessed`-based TUI) using `@kant/core` directly, with an IndexedDB shim (`idb-shim.ts`) since there's no browser. |
| `packages/push-proxy` | Small service holding Firebase credentials; relays wake-up push pings from relay operators without giving relays access to Firebase creds directly. |
| `packages/admin` | Placeholder — part of the paid "corporate bundle" per the licensing model, not implemented in the free/community build. |

### Networking model (core)
- `createNode()` in `packages/core/src/index.ts` builds a libp2p node (WebSockets transport, noise encryption, yamux muxing, circuit-relay-v2 transport) with a deterministic Ed25519 keypair derived from the user's identity seed, so PeerID/circuit address are stable across restarts.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CecuTheMag/Kant](https://github.com/CecuTheMag/Kant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
