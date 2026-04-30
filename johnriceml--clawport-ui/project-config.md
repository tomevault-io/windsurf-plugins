---
trigger: always_on
description: npm run setup        # Auto-detect OpenClaw config, write .env.local
---

# ClawPort -- Developer Guide

## Quick Reference

```bash
npm run setup        # Auto-detect OpenClaw config, write .env.local
npm run dev          # Start dev server (Turbopack, port 3000)
npm test             # Run all 781 tests via Vitest (32 suites)
npx tsc --noEmit     # Type-check (expect 0 errors)
npx next build       # Production build
```

### CLI (global install)

> The npm package is `clawport-ui`. The CLI command is `clawport`. The separate `clawport` npm package is unrelated.

```bash
npm install -g clawport-ui
clawport setup       # Auto-detect config, write .env.local into package dir
clawport dev         # Start dev server
clawport start       # Build + start production server
clawport status      # Check gateway reachability + env config
clawport help        # Show usage
```

The CLI resolves its own package root via `import.meta.url`, so all commands work regardless of the user's current working directory. Entry point: `bin/clawport.mjs`.

## Project Overview

ClawPort is a Next.js 16 dashboard for managing OpenClaw AI agents. It provides an org chart (Org Map), direct agent chat with multimodal support, cron monitoring, a cost dashboard, an activity console with live log streaming, and memory browsing. All AI calls route through the OpenClaw gateway -- no separate API keys needed.

## Tech Stack

- Next.js 16.1.6 (App Router, Turbopack)
- React 19.2.3, TypeScript 5
- Tailwind CSS 4 with CSS custom properties for theming
- Vitest 4 with jsdom environment
- OpenAI SDK (routed to Claude via OpenClaw gateway at localhost:18789)
- React Flow (@xyflow/react) for org chart

## Environment Variables

```env
WORKSPACE_PATH       # Required -- path to .openclaw workspace (auto-detected)
OPENCLAW_BIN         # Required -- path to openclaw binary
OPENCLAW_GATEWAY_TOKEN  # Required -- gateway auth token
ELEVENLABS_API_KEY   # Optional -- voice indicators
```

Run `npm run setup` to auto-detect all required values from your local OpenClaw installation.

**Workspace detection order:** `~/.openclaw/agents/main/workspace` (current layout) → `~/.openclaw/workspace` (legacy). Falls back to manual prompt if neither exists.

**Global install:** When installed via `npm install -g clawport-ui`, `.env.local` may not be writable in the package directory. Setup falls back to `~/.config/clawport-ui/.env.local` (XDG-compliant). The CLI (`bin/clawport.mjs`) checks both locations when loading env vars.

### Dev Server

`next.config.mjs` sets `allowedDevOrigins: ["*"]` so the dev server works over Tailscale, LAN, or any non-localhost origin without cross-origin errors.

## Architecture

### Agent Registry Resolution

```
loadRegistry() checks:
  1. $WORKSPACE_PATH/clawport/agents.json  (user override)
  2. Auto-discovered from $WORKSPACE_PATH   (agents/ directory scan)
  3. Bundled lib/agents.json               (default example)
```

`lib/agents-registry.ts` exports `loadRegistry()`. `lib/agents.ts` calls it to build the full agent list (merging in SOUL.md content from the workspace).

**Auto-discovery** scans `$WORKSPACE_PATH/agents/` for subdirectories containing a `SOUL.md` file. Each becomes an agent entry with sensible defaults (color from rotating palette, name from SOUL.md heading or directory slug). If `$WORKSPACE_PATH/SOUL.md` exists, it becomes the root orchestrator. This means any OpenClaw workspace works out of the box -- no `agents.json` needed.

Users can still drop a `clawport/agents.json` into their workspace for full control over names, colors, hierarchy, and tools.

### operatorName Flow

```
OnboardingWizard / Settings page
  -> ClawPortSettings.operatorName (localStorage)
  -> settings-provider.tsx (React context)
  -> NavLinks.tsx (dynamic initials + display name)
  -> ConversationView.tsx (sends operatorName in POST body)
  -> /api/chat/[id] route (injects into system prompt: "You are speaking with {operatorName}")
```

No hardcoded operator names anywhere. Falls back to "Operator" / "??" when unset.

### Chat Pipeline (Text)

```
Client -> POST /api/chat/[id] -> OpenAI SDK -> localhost:18789/v1/chat/completions -> Claude
                                             (streaming SSE response)
```

### Chat Pipeline (Images/Vision)

The gateway's HTTP endpoint strips image_url content. Vision uses the CLI agent pipeline:

```
Client resizes image to 1200px max (Canvas API)
  -> base64 data URL in message
  -> POST /api/chat/[id]
  -> Detects image in LATEST user message only (not history)
  -> execFile: openclaw gateway call chat.send --params <json> --token <token>
  -> Polls: openclaw gateway call chat.history every 2s
  -> Matches response by timestamp >= sendTs
  -> Returns assistant text via SSE
```

Key files: `lib/anthropic.ts` (send + poll logic), `app/api/chat/[id]/route.ts` (routing)

**Why send-then-poll?** `chat.send` is async -- it returns `{runId, status: "started"}` immediately. The `--expect-final` flag doesn't block for this method. We poll `chat.history` until the assistant's response appears.

**Why CLI and not WebSocket?** The gateway WebSocket requires device keypair signing for `operator.write` scope (needed by `chat.send`). The CLI has the device keys; custom clients don't.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JohnRiceML/clawport-ui](https://github.com/JohnRiceML/clawport-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
