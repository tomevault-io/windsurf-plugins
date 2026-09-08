---
trigger: always_on
description: This file provides guidance for AI coding agents working in this repository.
---

# AGENTS.md

This file provides guidance for AI coding agents working in this repository.

## Overview

WebChat is a decentralized, serverless browser extension that enables anonymous P2P chat on any website using WebRTC. Built with WXT framework for cross-browser support (Chrome, Firefox, Edge).

## Key Technologies

- **WXT**: Browser extension framework (config: `wxt.config.ts`)
- **Remesh**: DDD framework for domain logic with true UI/logic separation (RxJS-based reactive state management)
- **Artico**: Default WebRTC P2P room transport, using WebChat's owned signaling endpoint
- **Trystero**: Supported alternative WebRTC P2P room transport using its default public Nostr strategy
- **React 19** with TypeScript
- **Tailwind CSS v4** with shadcn/ui components
- **Valibot**: Runtime schema validation

## Development Commands

```bash
# Development
npm run dev                  # Chrome dev mode with hot reload
npm run dev:firefox          # Firefox dev mode

# Type checking
npm run check                # Run TypeScript compiler without emitting files

# Linting
npm run format               # oxfmt write mode (formats source in place)
npm run format:check         # read-only format check
npm run lint                 # oxlint safe fixes
npm run lint:check           # read-only lint check

# Building
npm run build                # Production build for all browsers
npm run build:chrome         # Chrome production build only
npm run build:firefox        # Firefox production build only

# Packaging
npm run pack                 # Create zip files for all browsers
npm run pack:chrome          # Create Chrome zip only
npm run pack:firefox         # Create Firefox zip only

# Maintenance
npm run clear                # Remove .output directory
npm run prepare              # Setup husky git hooks
npm run postinstall          # WXT preparation (auto-runs after install)
```

## Architecture

### Extension Structure

WebChat uses WXT's app-based structure (not entrypoints):

- **src/app/content/** - Content-script UI; each page connects to the shared Runtime through comctx
- **src/app/background/** - Extension actions, notifications, Runtime coordination, and the Firefox MV2 Runtime host
- **src/app/offscreen/** - Chrome MV3 host document for the shared headless Runtime
- **src/app/options/** - Options page UI for user profile settings
- Entry files: `index.ts` or `index.tsx` in each app directory

### Domain-Driven Design (Remesh)

Remesh is used across two ownership layers:

**Application/page Domains** (`src/domain/`):

- `ChatRoom.ts` and `WorldRoom.ts` - UI-facing room state and Runtime event projection
- `Message.ts`, `MessageList.ts`, and `MessageProjection.ts` - Local record model, persistence workflow, ordering, and reaction LWW projection
- `UserInfo.ts`, `AppStatus.ts`, `Danmaku.ts`, `Notification.ts`, and `Toast.ts` - Page and extension behavior
- `domain/externs/` - Application dependency contracts
- `domain/impls/` - Page-side adapters, including the origin-owned message store and Runtime client
- `domain/modules/` - Reusable domain sub-modules

**Headless Runtime Domains** (`src/domain/runtime/`):

- `Network.ts` - Trusted peer transport, World/Chat room orchestration, identity binding, and bounded history synchronization
- `Lifecycle.ts` - Per-origin page leases and the shared host lifecycle
- `Delivery.ts` - Volatile delivery and durable-settlement acknowledgement
- `runtime/Server.ts` creates the headless Remesh store and injects clock, identity, wire, and page-port externs through adapters.

### P2P Communication Architecture

The content pages are UI/comctx clients. They do not own peer rooms or duplicate durable history. A single shared headless Runtime lives in a Chrome MV3 offscreen document or the persistent Firefox MV2 background page. The Runtime owns one selected WebRTC room transport, trusted `sourcePeerId` context, World/Chat sessions, decode and delivery queues, and history scheduling, supply, cancellation, and admission. Artico is the shipped build-time default; Trystero remains a build-time alternative using its default Nostr strategy. `RoomTransportProvider.ts` instantiates exactly one provider for each Runtime host, with no runtime switch, automatic fallback, or simultaneous provider connection. After the last page for an origin detaches, that origin's domain state enters a five-second grace period; this does not release the shared Runtime or any other domain.

`src/protocol/` is the third-party-facing peer boundary:

- Chat v2 is the closed union `session | text | reaction | history-request | history-response`.
- World v2 has no message `type`; trusted room context selects its strict `{sessionId,user,sites}` shape.
- Peer frames use the fixed `base64(deflate(UTF8(JSON)))` codec and strict schemas. Payload identity never replaces the transport-provided source identity.

**Connection Flow**:

1. A content page registers its `{ domain, pageId }` lease and attaches to the shared Runtime through comctx.
2. The Runtime joins the v2 World room and the origin-derived v2 Chat room, then projects trusted snapshots and events to attached pages.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [molvqingtai/WebChat](https://github.com/molvqingtai/WebChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
