---
trigger: always_on
description: **Server:** pixoo-mcp-server
---

# Developer Protocol

**Server:** pixoo-mcp-server
**Version:** 1.4.0
**Framework:** [@cyanheads/mcp-ts-core](https://www.npmjs.com/package/@cyanheads/mcp-ts-core) `^0.13.10`
**Engines:** Bun ≥1.4.0, Node ≥24.0.0
**MCP SDK:** `@modelcontextprotocol/server` ^2.1.0 (protocol revision 2026-07-28 alongside the 2025 era)
**Zod:** ^4.6.5

> **Read the framework docs first:** `node_modules/@cyanheads/mcp-ts-core/CLAUDE.md` contains the full API reference — builders, Context, error codes, exports, patterns. This file covers server-specific conventions only.

---

## Architecture Overview

Local-network MCP server for Divoom Pixoo LED matrix displays (Pixoo-64 primary; 16/32 supported). The server owns all composition — rendering happens in an RGBA canvas pipeline on the host via `@cyanheads/pixoo-toolkit`, and the device receives final RGB frames over its local HTTP API.

**Two distinct layers:**

- **`src/renderer/`** — Pure rendering pipeline with no device dependency. Element renderers, styled-text engine (gradient ramps, shadows, outlines, semantic alignment), theme/palette registry, icon registry, effect compiler (presets → keyframes), keyframe interpolation, preview encoding (PNG/contact-sheet/GIF). Independently unit-testable.
- **`src/services/pixoo/`** — `PixooService` wraps `@cyanheads/pixoo-toolkit`. Handles lazy init from config, command serialization with min-interval pacing, `ensureCustomChannel()`, result→error-contract mapping, and device state snapshots.

Tools call renderer + service; they don't talk to the toolkit directly.

---

## What's Next?

When the user asks what's next or needs direction, suggest options based on the current project state. Common next steps:

1. **Re-run the `setup` skill** — ensures CLAUDE.md, skills, structure, and metadata are populated and up to date with the current codebase
2. **Add tools/resources/prompts** — scaffold new definitions using the `add-tool`, `add-app-tool`, `add-resource`, `add-prompt` skills
3. **Add services** — scaffold domain service integrations using the `add-service` skill
4. **Add tests** — scaffold tests for existing definitions using the `add-test` skill
5. **Field-test definitions** — exercise tools/resources/prompts with real inputs using the `field-test` skill, get a report of issues and pain points
6. **Run `devcheck`** — lint, format, typecheck, and security audit
7. **Run the `security-pass` skill** — audit handlers for MCP-specific security gaps: output injection, scope blast radius, input sinks, tenant isolation
8. **Run the `polish-docs-meta` skill** — finalize README, CHANGELOG, metadata, and agent protocol for shipping
9. **Run the `maintenance` skill** — investigate changelogs, adopt upstream changes, and sync skills after `bun update --latest`

Tailor suggestions to what's actually missing or stale — don't recite the full list every time.

---

## Core Rules

- **Logic throws, framework catches.** Tool/resource handlers are pure — throw on failure, no `try/catch`. Plain `Error` is fine; the framework catches, classifies, and formats. Use error factories (`notFound()`, `serviceUnavailable()`, etc.) when the error code matters.
- **Use `ctx.log`** for request-scoped logging. No `console` calls.
- **Use `ctx.state`** for tenant-scoped storage. Never access persistence directly.
- **Need input the caller didn't supply?** `return ctx.requestInput(...)` and read `ctx.inputs` when the handler is re-entered. Never `await` for user input mid-handler. (`ctx.elicit` was removed in the SDK v2 migration.) The server declares `sessionMode: 'stateless'` because no handler does this today — the first one that does changes it to `{ default: 'stateful', require: 'stateful' }` in `src/index.ts`, `.env.example`, the Dockerfile, and the README.
- **Secrets in env vars only** — never hardcoded.
- **Cut noise.** Add only what earns its place: no speculative generality, no guards for states the framework already prevents (Zod-validated params, classified errors), no abstraction until a third caller proves it, no option nothing sets.
- **Every `PixooResult` checked.** No fire-and-forget device calls. `pushed: true` means `error_code: 0` from the device — and past 40 frames, that the device requested the GIF and the whole file was handed to the OS to send (the response's `finish`), since `Device/PlayTFGif` acknowledges before it downloads. That is not confirmed receipt: a GIF smaller than the socket send buffer can finish before the device reads it.
- **Adding an env var requires both files** — `server.json` (`environmentVariables[]`) and `manifest.json` (`mcp_config.env` + `user_config`). `bun run lint:packaging` verifies the names match.
- **Close the loop on issues.** When implementing work tracked by a GitHub issue, comment on the issue with what landed and close it. Do both — a comment without a close leaves stale issues open; a close without a comment leaves no record of what shipped. The comment is for future readers — state the concrete changes, not the conversation that produced them.

---

## Patterns

### Tool — display text example

```ts
import { tool, z } from '@cyanheads/mcp-ts-core';

export const pixooDisplayText = tool('pixoo_display_text', {
  title: 'pixoo_display_text',
  description:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyanheads/pixoo-mcp-server](https://github.com/cyanheads/pixoo-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
