---
trigger: always_on
description: Guidelines for agents (and humans) working on this plugin.
---

# AGENTS.md — Tlon Plugin for OpenClaw

Guidelines for agents (and humans) working on this plugin.

---

## Commands

```bash
# Development
pnpm dev                    # Run dev environment (uses Docker)

# Testing
pnpm test                   # Run unit tests
pnpm test:watch             # Watch mode
pnpm test:security          # Security tests only
pnpm test:integration       # Integration tests (ephemeral fakezods)
pnpm test:integration:dev   # Integration tests against running dev
pnpm test:integration:watch # Watch mode for dev environment

# Linting & Formatting (monorepo root ESLint/Prettier configs)
pnpm lint                   # ESLint
pnpm lint:format            # Format with prettier

# Type checking
pnpm tsc --noEmit           # Full type check
```

---

## Goals / Purpose

-   Bridge OpenClaw ↔ Tlon/Urbit messaging
-   Support DMs and group channels with proper access control
-   Rich content: images, markdown → Tlon story format
-   Runtime-configurable authorization (settings store)

---

## Architecture

-   **SSE for inbound** — `sse-client.ts` handles real-time events from the ship
-   **HTTP pokes for outbound** — avoids SSE conflicts, simpler error handling
-   **Settings store** — runtime config that persists (channel rules, blocked ships, etc.)
-   **Approval flow** — DM requests from unknown ships queue for admin approval

### File Organization

```
src/
  channel.ts        # ChannelPlugin implementation (outbound, setup, status)
  monitor/          # Inbound message handling (SSE event loop)
    index.ts        # Main monitor loop
    approval.ts     # DM/channel approval workflow
    discovery.ts    # Auto-discover channels in joined groups
    history.ts      # Message history and caching
    utils.ts        # Bot mention detection, allowlist checks
  urbit/            # Low-level Urbit communication
    sse-client.ts   # SSE subscription client
    auth.ts         # +code authentication
    send.ts         # Send DMs and group messages
    story.ts        # Build Urbit story format
  settings.ts       # Runtime settings store (hot-reload via settings-store)
  types.ts          # Shared types
  config-schema.ts  # Zod schema for config validation
  targets.ts        # Target parsing (DM ship or channel nest)
```

### Key Patterns

-   **Plugin SDK**: Implements `ChannelPlugin` interface from `openclaw/plugin-sdk`
-   **Dual message paths**: Monitor uses SSE for inbound; outbound uses HTTP-only pokes
-   **Settings hot-reload**: Config can be updated via Urbit's settings-store without restart
-   **Authorization cascade**: Settings store overrides file config; default to "restricted" mode

### Dependencies

-   `@tloncorp/api` — Tlon API library (use this first!)
-   `openclaw/plugin-sdk` — Plugin interfaces and utilities
-   `@urbit/http-api` / `@urbit/aura` — Urbit primitives

### Dev Environment Setup

This package lives in the tlon-apps monorepo at `packages/openclaw`.

1. From `packages/openclaw`, run `./dev/setup.sh` (clones tlonbot next to the monorepo)
2. Configure `.env` with ship credentials
3. Run `pnpm dev` (uses Docker)

---

## Security

See [SECURITY.md](SECURITY.md) for the full security model (authorization, credentials, invariants).

Quick reminders:

-   ❌ `Math.random()` → ✅ `crypto.randomUUID()`
-   ❌ Raw `fetch()` with user URLs → ✅ `urbitFetch` with SSRF policy
-   ❌ Unsanitized input to `spawn()` → ✅ Validate/allowlist first
-   ❌ Forgetting `release()` → ✅ Always cleanup in `finally` blocks

---

## Coding Practices

### Use `@tloncorp/api` First

If the API package supports it, use it. Don't write raw HTTP calls for things the SDK handles:

-   Channel operations (subscribe, poke)
-   Scries for data fetching
-   Types for Tlon data structures (Post, Writ, etc.)

Only drop to raw `urbitFetch` when the SDK doesn't cover the use case.

### Imports

-   Node built-ins: always use `node:` prefix (`import crypto from "node:crypto"`)
-   Prefer named imports from `openclaw/plugin-sdk`
-   Keep third-party, SDK, and local imports grouped

### Types

-   Use types from `@tloncorp/api` for Tlon structures
-   Use types from `openclaw/plugin-sdk` for plugin interfaces
-   Avoid `any` — if you need an escape hatch, use `unknown` and narrow

### Error Handling

-   Let errors bubble up with context (don't swallow silently)
-   Use `runtime.error?.()` for logging errors in monitor
-   Wrap external calls (urbitFetch, API) in try/catch with meaningful messages

### Async Patterns

-   Always handle abort signals when passed (`opts.abortSignal`)
-   Clean up resources in `finally` blocks (release urbitFetch, close connections)
-   Use retry logic with exponential backoff for network calls

### What NOT to Do

-   ❌ `Math.random()` for IDs — use `crypto.randomUUID()`
-   ❌ Raw `fetch()` for user-provided URLs — use `urbitFetch`
-   ❌ Hardcoded ship names or URLs in logic
-   ❌ Skipping SSRF policy when making requests

---

## Testing

### Unit Tests (Local)

Test pure logic in isolation:

```bash
pnpm test              # run all tests
pnpm test:watch        # watch mode during development
```

What to unit test:

-   Ship normalization (`~ship` handling)
-   Channel nest parsing
-   Story/markdown conversion
-   Message deduplication logic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tloncorp/tlon-apps](https://github.com/tloncorp/tlon-apps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
