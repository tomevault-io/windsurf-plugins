---
trigger: always_on
description: Drive Chrome via the DevTools Protocol from JavaScript. Run JS snippets through the `browser-harness-js` CLI — it auto-spawns a long-lived bun HTTP server holding a fully-typed CDP `Session`, and every call (`browser-harness-js 'await session.Page.navigate(...)'`) executes against the same persistent connection. Session, active target, and globals survive across calls. Use when the user wants to automate, script, or inspect a Chrome browser via CDP — single tab or multi-tab, attach to existing C
---


# CDP — `browser-harness-js` skill

Custom codegen'd CDP SDK (every method from browser_protocol.json + js_protocol.json gets a typed wrapper) plus a tiny HTTP server that holds one persistent CDP `Session`. The `browser-harness-js` CLI auto-starts the server on first use and forwards JS snippets to it.

The SDK lives in the skill's `sdk/` directory. In the rest of this doc, `<skill-dir>` refers to wherever `npx skills add` installed the skill (Claude Code: `~/.claude/skills/cdp`; Cursor: `~/.cursor/skills/cdp`; other agents vary). The CLI should be on PATH as `browser-harness-js`.

## Setup (once, first use)

`npx skills add` drops the skill into your agent's skills directory but does NOT put the CLI on PATH. Before the first call, verify it's reachable and symlink it into any directory on your PATH if not:

```bash
# macOS (Apple Silicon + Homebrew)
command -v browser-harness-js >/dev/null || ln -sf <skill-dir>/sdk/browser-harness-js /opt/homebrew/bin/browser-harness-js

# macOS (Intel) / most Linux — may need sudo
command -v browser-harness-js >/dev/null || ln -sf <skill-dir>/sdk/browser-harness-js /usr/local/bin/browser-harness-js

# Linux without sudo (ensure ~/.local/bin is on PATH)
command -v browser-harness-js >/dev/null || { mkdir -p ~/.local/bin && ln -sf <skill-dir>/sdk/browser-harness-js ~/.local/bin/browser-harness-js; }
```

The CLI auto-installs `bun` on first run if it's missing (the server is Bun-native). Set `BROWSER_HARNESS_SKIP_BUN_INSTALL=1` to opt out.

## How to use

Just run `browser-harness-js '<JS>'`. The first call spawns the server in the background; subsequent calls hit the same process and so reuse the same `session`, the same WebSocket to Chrome, and any globals you set.

```bash
browser-harness-js 'await session.connect()'
browser-harness-js 'await session.Page.navigate({url:"https://example.com"})'
browser-harness-js '(await session.Runtime.evaluate({expression:"document.title",returnByValue:true})).result.value'
```

Output is the **raw result content** — no `{ok,result}` envelope.

| Result type | stdout |
|---|---|
| string                       | bare text, no JSON quotes (e.g. `Example Domain`) |
| number / boolean             | `42`, `true` |
| object / array (non-empty)   | compact JSON (e.g. `{"frameId":"..."}`, `[1,2,3]`) |
| `undefined` / `null` / `""` / `{}` / `[]` | empty (no output) |

**Errors** go to **stderr**, exit code `1`. The CDP error message and JS stack are printed verbatim, e.g.:
```
Error: CDP -32602: invalid params
    at _call (.../session.ts:117:33)
    ...
```
Detect failure with `if browser-harness-js '...'; then ...; else handle_error; fi` or by checking `$?`.

**Multi-line snippets via stdin (heredoc).** Important: a multi-statement snippet does NOT auto-return the last expression — write `return X` explicitly. Single-expression snippets passed as the first argument DO auto-return.

```bash
browser-harness-js <<'EOF'
const tabs = await listPageTargets();
globalThis.tid = tabs[0].targetId;
await session.use(globalThis.tid);
return globalThis.tid;
EOF
```

## CLI commands

| Command | Behavior |
|---|---|
| `browser-harness-js '<js>'`     | Auto-start server if needed, eval the JS, print result. |
| `browser-harness-js <<EOF…EOF`  | Same, code from stdin. |
| `browser-harness-js --status`   | Print health JSON (uptime, connected, sessionId) or exit 1 if down. |
| `browser-harness-js --start`    | Explicit start (no-op if already running). |
| `browser-harness-js --stop`     | Graceful shutdown. Drops session state. |
| `browser-harness-js --restart`  | Stop + start fresh. |
| `browser-harness-js --logs`     | `tail -f` the server log (`/tmp/browser-harness-js.log`). |

Env vars: `CDP_REPL_PORT` (default `9876`), `CDP_REPL_LOG` (default `/tmp/browser-harness-js.log`).

## API surface inside snippets

These globals are pre-loaded — no imports needed:

- `session` — the persistent `Session`. Has every CDP domain mounted: `session.Page`, `session.DOM`, `session.Runtime`, `session.Network`, … 56 domains, 652 methods total.
- `listPageTargets()` — list real page targets via CDP's `Target.getTargets` (works on Chrome 144+ too), with `chrome://` and `devtools://` URLs filtered out. No args — uses the connected session.
- `detectBrowsers()` — scan OS-specific profile dirs for running Chromium-based browsers with remote debugging on. Returns `[{name, profileDir, port, wsPath, wsUrl, mtimeMs}]`, sorted by most recently launched.
- `resolveWsUrl(opts)` — resolve a WS URL from `{wsUrl}` | `{port, host?}` | `{profileDir}`. For the no-args auto-detect flow, call `session.connect()` directly instead.
- `CDP` — the generated namespaces (`CDP.Page`, `CDP.Runtime`, …) for type-name reference.

### Calling a CDP method


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [browser-use/browser-harness-js](https://github.com/browser-use/browser-harness-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
