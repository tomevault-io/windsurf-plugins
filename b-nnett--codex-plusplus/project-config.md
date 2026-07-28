---
trigger: always_on
description: This file is read by AI coding agents (and humans) authoring tweaks for
---

# AGENTS.md — Codex++ tweak authoring guide

This file is read by AI coding agents (and humans) authoring tweaks for
Codex++. **Follow it.**

## Prime directive

> **Match Codex's existing UI patterns unless the user specifically requests
> otherwise.** Don't invent new visual idioms. Don't hard-code colors, sizes,
> or fonts. Use Codex's Tailwind tokens (`text-token-*`, `bg-token-*`,
> `border-token-border`, `px-row-x`, `py-row-y`, `p-panel`, `h-toolbar`,
> etc.). When in doubt, mirror what the surrounding Codex screen does.

If the user explicitly says "I want a custom look here" — only then deviate.

## Tweak shape

A tweak is a folder containing:

```
my-tweak/
  manifest.json   ← required, schema below
  index.js        ← entry, exports { start, stop }
  icon.png        ← optional, referenced by manifest.iconUrl
```

`index.js` is loaded as a CommonJS-shaped module. Bundle ESM or TypeScript to
runtime-loadable JavaScript before installing the tweak:

```js
module.exports = {
  async start(api) { /* … */ },
  async stop()    { /* … */ },
};
```

Or, with the SDK helper:

```js
const { defineTweak } = require("@codex-plusplus/sdk");
module.exports = defineTweak({
  start(api) { api.log.info("hello"); },
});
```

## Manifest schema (`manifest.json`)

| Field         | Type                            | Required | Notes |
|---------------|---------------------------------|----------|-------|
| `id`          | `string` (reverse-DNS)          | yes      | `"com.you.my-tweak"` |
| `name`        | `string`                        | yes      | Shown in the Tweaks list. |
| `version`     | `string` (semver)               | yes      | `"1.0.0"` |
| `githubRepo`  | `string` (`owner/repo`)         | yes      | GitHub repository slug for the tweak, e.g. `"you/my-tweak"`. |
| `description` | `string`                        | no       | Renders below the name. |
| `author`      | `string \| { name, url?, email? }` | no    | If a string, treated as display name. If structured with `url`, name becomes a link. |
| `homepage`    | `string` (URL)                  | no       | Linked next to the author. |
| `iconUrl`     | `string`                        | no       | `https://…`, `data:…`, or `./relative.png`. If absent, an initial avatar is rendered. |
| `tags`        | `string[]`                      | no       | e.g. `["ui", "shortcut"]`. |
| `scope`       | `"renderer" \| "main" \| "both"` | no      | Current runtime behavior is effectively `"both"` when omitted. Set it explicitly. |
| `main`        | `string`                        | no       | Custom entry path. Defaults to `index.js`/`index.mjs`/`index.cjs`. |
| `minRuntime`  | `string` (semver range)         | no       | Codex++ runtime range required. |

Full manifest example:

```json
{
  "id": "com.example.my-tweak",
  "name": "Hello World",
  "version": "0.1.0",
  "githubRepo": "example/my-tweak",
  "description": "Minimal example tweak. Adds a section to the Tweaks tab.",
  "author": {
    "name": "codex-plusplus",
    "url": "https://github.com/anomalyco/codex-plusplus"
  },
  "homepage": "https://github.com/example/my-tweak",
  "tags": ["example", "demo"],
  "scope": "renderer"
}
```

## The API (`api`)

See `@codex-plusplus/sdk` for full types. The most-used pieces:

- `api.log.{debug,info,warn,error}(…)` — goes to `preload.log` and DevTools.
- `api.storage.{get,set,delete,all}` — per-tweak persistent KV.
- `api.settings.register({ id, title, description, render })` — register a
  section that appears under your tweak's row in the Tweaks page.
- `api.settings.registerPage({ id, title, description?, iconSvg?, render })` — register a dedicated settings page in the sidebar.
- `api.react.waitForElement(selector, timeoutMs?)` — async DOM-ready wait.
- `api.react.findOwnerByName(node, "Component")` — fiber walk.
- `api.ipc.{on,send,invoke}` — channels are auto-prefixed with `codexpp:<id>:`.
- `api.fs.{read,write,exists}` — sandboxed to your tweak's data dir.
- `api.codex.runtime.{getInfo,getCapabilities}` — Owl/Electron runtime probes.
- `api.codex.windows.{create,getPrimary,focus,show}` — stable native Codex window hooks.
- `api.codex.views.create(...)` — Owl `WebContentsView`/`BrowserView` overlays inside Codex windows.
- `api.codex.cdp.{getStatus,listTargets}` — CDP status/target discovery when remote debug is enabled.
- `api.codex.native.{loadModule,createPanel,attachView,launchHelper}` — native module, AppKit/Metal view, and helper bridge.
- `api.codex.{createWindow,createBrowserView}` — backwards-compatible window/view hooks.

## UI components — copy these, don't invent new ones

All snippets below render correctly inside a `SettingsSection.render(root)`
or any DOM you mount into Codex.

### 1. Section title row

Use this above each card-grouped form section.

```js
const titleRow = document.createElement("div");
titleRow.className = "flex h-toolbar items-center justify-between gap-2 px-0 py-0";
const inner = document.createElement("div");
inner.className = "flex min-w-0 flex-1 flex-col gap-1";
const t = document.createElement("div");
t.className = "text-base font-medium text-token-text-primary";
t.textContent = "General";
inner.appendChild(t);
titleRow.appendChild(inner);
root.appendChild(titleRow);
```

Optional subtitle below:

```js

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [b-nnett/codex-plusplus](https://github.com/b-nnett/codex-plusplus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
