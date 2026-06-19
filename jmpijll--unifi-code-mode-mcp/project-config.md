---
trigger: always_on
description: >-
---


# UniFi Code-Mode MCP — operating manual

The server exposes only **two tools** and a sandboxed JavaScript runtime.
You drive the entire UniFi API surface by *writing JavaScript that the
sandbox executes*, not by calling per-endpoint tools. This guide tells you
how to do that effectively.

## 1. The two tools

| Tool | Purpose | Sandbox kind |
|---|---|---|
| `search` | Search the OpenAPI catalogue for operationIds, paths, summaries, parameters. | Sync. Returns a JSON list of operations. |
| `execute` | Run JavaScript that calls the UniFi APIs through the sandbox. Returns whatever your script's last expression evaluates to. | QuickJS (synchronous; host calls block from JS's perspective). |

You always start with `search` to find the operationIds you need, then
`execute` to call them. **Never invent operationIds** — always confirm with
`search` first; the spec changes per controller version.

### Sandbox JavaScript dialect — quick rules

- **Top-level `return` is not allowed.** The script body is evaluated as a
  module body, not a function body. Write the result as the last expression
  statement: `unifi.local.callOperation('getSiteOverviewPage').data.length;`
- **Top-level `await` is not allowed.** Host calls (`callOperation`,
  `request`, tag-grouped accessors) block the QuickJS VM until they
  resolve. Don't write `await unifi.local.callOperation(...)` — write
  `unifi.local.callOperation(...)` and the value is returned to you
  synchronously. If you genuinely need `async`/`await` (e.g. to use
  `Promise.all` over fan-out), wrap the whole script in an async IIFE:
  `(async () => { … return result; })()` — the IIFE's promise is what
  the executor awaits and unwraps.
- **The last expression's value is what `execute` returns to the caller.**
  Object/array literals, ternary expressions, function calls — anything
  that evaluates is fine. `console.log` does not affect the return value
  but its output is captured into the warnings list.

## 2. Five sandbox surfaces

Inside `execute`, the global `unifi` namespace exposes up to five surfaces.
Any surface may be missing if its credentials or spec are not configured —
check `unifi.<surface>.spec` for `{ title, version, sourceUrl }` before
relying on it.

| Surface | Reaches | Credentials |
|---|---|---|
| `unifi.local.*` | A local UniFi controller's Network Integration API (`https://<controller>/proxy/network/integration/v1/...`). | Local API key. |
| `unifi.cloud.*` | UniFi Site Manager (`https://api.ui.com/v1/...`). | Cloud API key. |
| `unifi.cloud.network(consoleId).*` | The Network Integration API of a remote console, **proxied through Site Manager** (`/v1/connector/consoles/{id}/proxy/network/integration`). | **Cloud** API key only. |
| `unifi.local.protect.*` | A local controller's UniFi Protect Integration API (`https://<controller>/proxy/protect/integration/v1/...`) — cameras + PTZ, NVRs, sensors, lights, chimes, viewers, live-views, plus the full official surface when the loader can fetch `apidoc-cdn.ui.com/protect/v<version>/integration.json`. | Local API key (Protect must be installed on the controller). |
| `unifi.cloud.protect(consoleId).*` | Protect Integration API tunneled through the Site Manager connector at `/v1/connector/consoles/{id}/proxy/protect/integration`. URL pattern is officially documented by Ubiquiti (`developer.ui.com/protect/v7.0.107/...`, "Remote" base-URL selector). | Cloud API key only. |

Pick the surface based on what the user has:

- **Local controller, on the same LAN as the MCP host, Network only.** → `unifi.local`.
- **Local controller, with Protect installed.** → `unifi.local` for Network, `unifi.local.protect` for cameras/sensors/lights/etc.
- **Cloud-managed console, you only have a cloud API key.** →
  `unifi.cloud.network(consoleId)` for Network ops; `unifi.cloud` for
  Site-Manager-only ops (multi-console listing, ISP metrics, SD-WAN);
  `unifi.cloud.protect(consoleId)` for Protect (live-verified against a
  real UDM-Pro running Protect 7.0.107 — see §10).
- **Multiple consoles under one Site Manager account.** → discover them
  with `unifi.cloud.callOperation('listHosts')` (or
  `request('GET', '/v1/hosts')`) and then build per-console proxies.

## 3. Each surface offers three call shapes

```js
// 1. Typed-ish operationId call (preferred — readable, future-proof)
const sites = unifi.local.callOperation('getSiteOverviewPage', {
  pageSize: 100,
});

// 2. Tag-grouped accessor (Proxy sugar for the same operation)
//    e.g.  unifi.local.sites.getSiteOverviewPage(...)
const sites2 = unifi.local.sites.getSiteOverviewPage({ pageSize: 100 });

// 3. Raw escape hatch (use when the spec doesn't cover what you need)
const raw = unifi.local.request({
  method: 'GET',
  path: '/v1/sites',
  query: { pageSize: 100 },
});
```

The first form is the default. Reach for `request()` only when the spec
genuinely doesn't cover the call you need — that's a signal to also
`search` for a real operation.

## 4. The `search → execute` loop

Standard recipe — follow it every time:

```text
1. Call search("<keywords>") with the user's question keywords.
2. Read the operationIds, methods and paths in the result.
3. Construct ONE execute() script that:
     a. fetches what you need with callOperation,
     b. shapes the result (pick fields, aggregate counts, build a table),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jmpijll/unifi-code-mode-mcp](https://github.com/jmpijll/unifi-code-mode-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
