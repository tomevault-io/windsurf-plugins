---
trigger: always_on
description: Guidance for Claude Code working in this repository. This file is comprehensive
---

# CLAUDE.md

Guidance for Claude Code working in this repository. This file is comprehensive
and self-contained; `README.md` is the longer narrative reference (it opens with
the design history). When the two overlap, the code is authoritative — verify
before relying on either.

---

## What this project is

**HA-MCP-Gateway** is a Cloudflare Worker + Durable Object that bridges a Home
Assistant smart home to LLMs. It:

- holds a persistent WebSocket to Home Assistant and mirrors live entity state
  in memory,
- streams every meaningful HA event into a queryable D1 **forensic log**,
- exposes the whole surface as an **MCP server** (`POST /mcp`) for clients like
  Claude Desktop and Claude Code,
- runs a built-in **chat agent** — "Ranger," GLM 5.2 Fast on Fireworks
  (runtime-configurable; see LLM configuration) with reasoning enabled and a
  native tool-calling loop — reachable at a web
  `/chat` UI and via the `ai_chat` MCP tool,
- short-circuits deterministic cover (garage/bay door) commands via a sub-500ms
  fast path that skips the LLM entirely.

It is a single-household production deployment — the author's home. There is no
staging environment, and a push to `main` deploys to the live house (see
Build, test, deploy below). Treat changes with appropriate care.

The autonomous "heartbeat" agent that once ran every 60 seconds has been
removed — see the Story section in `README.md`. There is exactly one execution
path now: user-driven chat.

---

## Build, test, deploy

Working directory: `C:\Users\obert\ha-mcp-gateway`. Shell is **PowerShell on
Windows**.

```powershell
npm install          # dependencies (only devDependency is vitest)
npm test             # vitest run — 6 suites (forensic filter, light sanitizer,
                     #   scheduler, fast path, coalesce, llm-config)
```

- **Deploy is git-driven.** A push to `main` triggers a **Cloudflare Workers
  Builds** pipeline (set up in the Cloudflare dashboard, so it leaves no
  artifact in the repo) that runs the build and deploys. A merged commit goes
  live — treat `git push` to `main` as a production deploy.
- `wrangler deploy` from the repo root is the manual / local alternative; it
  runs the same build.
- **Build** (either path) runs the `wrangler.toml` `[build]` directive —
  `npx esbuild src/worker.js --bundle --outdir=dist --format=esm --outbase=src`
  — producing `dist/worker.js`.
- **`dist/worker.js` is a build artifact. Never edit it by hand** — it is
  overwritten on every build. All source lives in `src/`.
- A deploy reconciles bindings, cron triggers, and Durable Object migrations
  with Cloudflare.
- Tests run with `vitest` — 6 suites under `test/` (98 cases): the forensic
  noise filter (`should-log-state-change`), the light-service sanitizer, the
  scheduler, the cover fast path, the flat-arg coalescer, and the runtime
  LLM-config helpers (`llm-config`). Add tests alongside these when changing
  pure, testable logic; the pure helpers keep a synced stub copy in their test.

---

## Critical gotchas — read before changing code

1. **Durable Object isolate staleness.** The persistent HA WebSocket keeps the
   DO's V8 isolate resident across deploys — Cloudflare never hibernates it, so
   it never reloads new code on an ordinary deploy. **Worker-side changes
   (`src/worker.js`) take effect immediately. DO-side changes
   (`src/ha-websocket.js`) do not.** To force a fresh DO isolate you must rename
   the DO class via a `renamed_classes` migration in `wrangler.toml` and update
   `class_name` in the `durable_objects.bindings`. The class is currently
   `HAWebSocketV29` — it has been renamed 28 times for exactly this reason.
   Procedure for any DO-side change you need live:
   - bump the class name (`HAWebSocketV29` → `HAWebSocketV30`) in the `export
     class` line in `src/ha-websocket.js`, every `HAWebSocketV29.` static
     reference, the `export {}` at the file end, and the `worker.js` import;
   - add a `[[migrations]]` block with `tag = "v30"` and
     `renamed_classes = [{ from = "HAWebSocketV29", to = "HAWebSocketV30" }]`;
   - update `class_name` in `[[durable_objects.bindings]]`.
   **Exception — the LLM config is now runtime-configurable** (V29): the chat/
   agent model, endpoint, and reasoning mode can be swapped live via the
   `/admin/llm-config` route (DO storage override) with no rename or redeploy.
   See the LLM configuration section. A model swap no longer needs this dance.
   DO storage (snapshot, chat history, memory, cursors) is preserved across
   renames. Diagnose staleness with `wrangler tail --format json` (compare
   per-event `scriptVersion.id` to the latest deploy) or by comparing
   `/admin/version` (Worker) against the DO `/version` route.

2. **Never edit `dist/`.** It is generated.

3. **Pooling discipline.** Every Workers AI embedding call must use
   `pooling: "cls"`. The `ha-knowledge` Vectorize index was built with cls
   pooling; mismatched pooling between backfill and query produces near-random
   rankings.

4. **Cloudflare Access fronts the Worker.** A direct `curl` /
   `Invoke-RestMethod` hits the Access login page, not the app. Use
   `cloudflared access curl` or a service token for any HTTP probe.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [praxeo/ha-mcp-gateway](https://github.com/praxeo/ha-mcp-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
