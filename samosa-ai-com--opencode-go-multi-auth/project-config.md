---
trigger: always_on
description: > Repo-specific notes for OpenCode sessions working in `opencode-go-multi-auth`.
---

# AGENTS.md

> Repo-specific notes for OpenCode sessions working in `opencode-go-multi-auth`.
> Keep this file short. Add only what an agent would otherwise get wrong.

## What this repo is

A TypeScript proxy plugin that pools multiple OpenCode Go + Zen API subscriptions
into a single endpoint, with a local web UI dashboard. It runs as an
**OpenCode plugin** (auto-starts a shared detached daemon) or as a
**standalone CLI**. See `README.md` for end-user docs; this file is for
agents editing the code.

## Build, typecheck, run

```bash
npm install
npm run typecheck   # tsc --noEmit
npm run build       # tsc AND cp -r src/dashboard/public/. dist/dashboard/public/
npm run dev         # tsx watch src/bin.ts (standalone mode, no plugin)
npm run start       # node dist/bin.js (standalone mode)
npm run clean       # rm -rf dist
```

**Critical:** `src/dashboard/public/` is a static asset directory that is
**not compiled by `tsc`**. `npm run build` is a two-step pipeline: it
runs `tsc` first, then `cp -r` the dashboard public/ into `dist/`. If you
edit a `.html`, `.css`, or `.js` file under `src/dashboard/public/`, the
copy step is what makes the change visible. Running `tsc` alone, or
running `tsx` directly, will not pick up dashboard UI changes. The `.ts`
source under `src/dashboard/server.ts` is compiled normally.

There is **no test suite, no linter, and no formatter configured** in
`package.json`. Verification = `npm run typecheck && npm run build`.
Add new tooling only after confirming with the user.

## Runtime topology — do not break this

- The **OpenCode plugin entry** is `src/opencode-plugin.ts` (compiled to
  `dist/opencode-plugin.js`). The plugin's only job is to spawn (or
  reuse) a single detached daemon.
- The **daemon entry** is `src/bin.ts` (compiled to `dist/bin.js`). The
  daemon listens on:
  - `18905` — the proxy / API endpoint (this is what opencode points at
    via `provider.opencode-go.options.baseURL` and any custom
    `provider.<name>.options.baseURL` ending in `/zen` for the Zen
    upstream; see the Dual upstream section below)
  - `18904` — the dashboard web UI
- The daemon is **detached** and **shared across opencode sessions**.
  Closing the opencode UI does **not** stop the daemon. Killing the
  opencode session does **not** stop the daemon. The plugin's
  `dispose()` is intentionally a no-op for this reason.
- The active daemon writes its PID to `~/.opencode/router.pid` and a
  bootstrap lock to `~/.opencode/router-bootstrap.lock`. Both paths are
  hardcoded in `src/runtime/daemon.ts`.

### Hot-swap pattern (when an agent needs to restart the daemon)

**The agent must NOT kill or restart the daemon itself.** The agent's
own shell session is itself routed through the proxy on port 18905.
Restarting the daemon tears down the proxy the agent is using, which
can corrupt the in-flight `opencode` turn, the WebSocket log stream,
and the live edits the user is making. **Always ask the user to run
the restart command themselves** and wait for them to confirm before
proceeding.

When backend code changes (`src/`, `npm run build`) require a restart,
or when something is stuck, the agent should:

1. Build: `npm run build` (the agent is allowed to do this — it
   writes to `dist/` only and does not touch running processes).
2. Tell the user to run `./restart-router.sh` from the repo root
   (or the equivalent one-liner below) and wait.
3. Verify with `curl -sf http://127.0.0.1:18904/healthz` after the
   user reports back.
4. Never `kill` the daemon, never `kill -9` it, never run
   `node dist/bin.js` in the foreground. Restarting the router also
   does **not** affect the OpenCode session (`opencode serve` /
   `opencode -s ...`) — those are independent processes.

The repo ships `restart-router.sh` for this purpose. It is the
canonical recipe:

```bash
PID=$(cat ~/.opencode/router.pid 2>/dev/null | jq -r .pid 2>/dev/null)
[ -n "$PID" ] && kill "$PID" 2>/dev/null
sleep 1
cd "$(dirname "$(readlink -f "$0")")"
nohup env OPENCODE_ROUTER_PLUGIN_MODE=1 node dist/bin.js > /dev/null 2>&1 &
disown
sleep 2
curl -sf http://127.0.0.1:18904/healthz && echo " — daemon is healthy"
```

`OPENCODE_ROUTER_PLUGIN_MODE=1` suppresses the "run setup wizard"
prompt at boot. If a stale WebSocket log-stream connection from the
OpenCode dashboard keeps the previous daemon's `server.close()` from
resolving, the old process stays in `ps` holding no LISTEN sockets;
`kill -KILL <old-pid>` is safe in that state and is the user's call,
not the agent's.

## Routing / request flow

```
opencode CLI
  → provider.opencode-go.options.baseURL  (default http://localhost:18905)
  → provider.opencode-zen.options.baseURL (default http://localhost:18905/zen)
  → src/proxy/server.ts handleRequest
     → keyManager.selectKey (routing strategy + session stickiness)
     → circuitBreaker.isAvailable
     → buildUpstreamHeaders + cache-header passthrough
     → fetch(upstreamUrl, signal)
  → https://opencode.ai/zen/go/v1/{messages|chat/completions}  (Go, no /zen prefix)
  → https://opencode.ai/zen/v1/{messages|chat/completions}     (Zen, /zen prefix stripped)
  → response streamed byte-for-byte back to opencode
```

Key invariants to preserve:

- **The proxy is a byte-for-byte pass-through on the request and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [samosa-ai-com/opencode-go-multi-auth](https://github.com/samosa-ai-com/opencode-go-multi-auth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
