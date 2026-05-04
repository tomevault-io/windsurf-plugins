---
trigger: always_on
description: This document defines the implementation plan for an open-source Electron desktop client that provides a Discord-like user experience while connecting each joined "server" to its own independent backend service.
---

# OpenChat Client Planning Document

## 1) Purpose
This document defines the implementation plan for an open-source Electron desktop client that provides a Discord-like user experience while connecting each joined "server" to its own independent backend service.

This repository is **client-only**.

## Agent Execution Rule
- Do not run any `git` commands in this repository.
- Do not run any commands unless they are explicitly approved by the user or explicitly allowed in this `AGENTS.md`.
- Allowed without additional approval: `yarn build`, `yarn typecheck`, `yarn run build`, `yarn run typecheck`, `corepack yarn build`, `corepack yarn typecheck`.
- Never reintroduce code/content that the user previously removed.
- If an edit conflicts or patching fails, prefer minimal targeted edits; do not replace entire files unless the user explicitly asks for a full rewrite.

## 2) Scope and Boundaries

### In scope
- Electron desktop app (macOS, Windows, Linux).
- Discord-like UI/UX patterns (layout, navigation model, interaction flows).
- Multi-server client that can connect to different backend hosts per server.
- Frontend state management, transport abstractions, and rendering pipeline.
- Client-side security controls and trust UI.
- Contributor workflows, build/release pipeline, and documentation standards.

### Out of scope
- Backend server implementation.
- Database schema design for backend services.
- Backend deployment infrastructure.
- Auth service implementation details beyond client integration behavior.

## 3) Product Direction

### Core idea
- One app, many communities.
- Every joined server has its own backend endpoint and independent configuration.
- Users interact through a consistent UI regardless of backend differences.

### UX target
- Familiar Discord-like information architecture:
  - Server rail
  - Channel list
  - Main chat pane
  - Member/activity side pane
  - Settings modals and overlays
- Fast keyboard-first navigation.
- Smooth real-time updates with clear connection state handling.

### Product principles
- Consistency: unified client behavior across heterogeneous server backends.
- Safety: explicit trust signals for server endpoints and permissions.
- Transparency: visible server identity, capabilities, and connection state.
- Extensibility: plugin-like feature toggles driven by server capabilities.
- User sovereignty: user profile and identity metadata stay local to the client; servers receive only protocol-required unique user identifiers and proofs.

## 4) Architecture Plan (Client-Only)

### High-level modules
- `electron-main`: window lifecycle, protocol handling, secure process boundaries.
- `renderer-app`: UI shell, routing, feature surfaces.
- `shared-sdk`: typed contracts for transport, events, and domain models.
- `server-connection-layer`: endpoint configuration, session management, reconnect logic.
- `identity-layer`: local identity generation/import, key management, server UID projection.
- `state-layer`: global/app state, server-scoped caches, optimistic updates.
- `design-system`: reusable components and interaction primitives.
- `docs`: architecture records, feature specs, contribution docs.

### Frontend stack decision
- Renderer framework: `Vue 3`.
- UI component layer: `PrimeVue` in **unstyled mode**.
- Styling system: custom design tokens + utility/component CSS to match Discord-like UX.
- Component usage rule: PrimeVue components must be wrapped by local design-system components before broad app usage.
- Goal: use PrimeVue for accessibility and behavior primitives while owning all visual presentation.

### State management decision
- Primary client state store: `Pinia`.
- Data-fetching and request lifecycle logic: service layer first, with optional `@tanstack/vue-query` adoption for cache/sync patterns where it reduces complexity.
- Store design rule: server-scoped domain data must be keyed by `server_id` and never mixed across server contexts.

### Planned Pinia store boundaries
- `useAppUiStore`: global shell state (layout panes, modals, navigation context).
- `useIdentityStore`: local user identity profile, key references, and privacy controls.
- `useSessionStore`: current auth/session metadata and active server context.
- `useServerRegistryStore`: joined servers and server configuration metadata.
- `useChannelStore`: channel trees and per-server channel selection state.
- `useMessageStore`: message timelines, optimistic sends, pagination windows.
- `usePresenceStore`: typing, presence, and ephemeral real-time indicators.
- `useSettingsStore`: user preferences (theme, keybinds, notification settings).

### State persistence and security rules
- Persist only non-sensitive UX settings by default.
- Store credentials/tokens using secure Electron + OS keychain mechanisms, not plain local storage.
- Store local identity/profile data in encrypted local storage controlled by the user.
- Apply TTL and bounded cache limits for message/presence state.
- Clear server-scoped volatile state on sign-out and on server removal.
- Never transmit user profile attributes to servers unless explicitly approved by a future ADR.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [porthorian/openchat-client](https://github.com/porthorian/openchat-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
