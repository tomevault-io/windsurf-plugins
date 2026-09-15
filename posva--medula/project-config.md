---
trigger: always_on
description: Headless devtools built on [devframe](https://devfra.me). A coding agent reads and changes the
---

# medula

Headless devtools built on [devframe](https://devfra.me). A coding agent reads and changes the
state of an open web page through MCP. No panel UI: only a plain HTML instructions page. medula
always runs as a **dock of a hub** (Vite DevTools, Nuxt DevTools 4, or its own hub in Next): the
hub owns the connection, the auth gate and the MCP route. It never runs a devframe of its own next
to another one (two devframes on one page fight over the shared `__DEVFRAME_CONNECTION__`).

## Commands

```bash
pnpm build                                   # tsdown (lib) + vite (config page, connect.js)
pnpm build:lib                               # lib only
pnpm test                                    # build + coverage + typecheck
pnpm exec vitest run src/client/path.spec.ts # one test file
pnpm lint / pnpm lint:fix                    # oxlint
pnpm test:types                              # tsc
pnpm play:vue | play:react | play:svelte | play:next | play:nuxt   # playgrounds (run pnpm build first)
pnpm e2e:agent                               # Claude Code changes the fixture state over MCP
pnpm e2e:agent:codex                         # same with Codex
pnpm e2e:agent:vue | e2e:agent:svelte        # zero-config playground scenarios
```

Playground ports: vue-vite 5173, react-vite 5174, svelte-vite 5175, e2e fixture 5199, nextjs 3000,
nuxt 3001.

## Important

Keep this file up to date when commands, structure or tooling change.

## Architecture

Zero app code: a host adapter inlines the hook bootstrap into the app page in dev and registers
medula as a hub dock; the hub client runtime imports the page script (the dock `clientScript`,
`eager: true`) into the page, and that script discovers frameworks like the official devtools do.

| Piece                    | Runs in | Purpose                                                                                                                                                           |
| ------------------------ | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `src/page/bootstrap.ts`  | browser | `BOOTSTRAP_SCRIPT`: inline `<head>` script = Vue hook shim (`src/page/vue-hook.ts`) + React hook shim (`src/react/hook.ts`). Must run before the frameworks load. |
| `src/panel/connect.ts`   | browser | The page script (`dist-client/connect.js`, dock `clientScript`): state channel + Vue/Pinia/Router + React + Svelte discovery. Never connects on its own.          |
| `src/panel/main.ts`      | browser | The dock page: static instructions; reads the hub connection of the parent window to show the MCP URL (`resolveMcpUrl` in `src/shared.ts`)                        |
| `src/vue/internal.ts`    | browser | Component tree walker + StateEditor-like setter (mirrors Vue DevTools), Pinia/Router tools                                                                        |
| `src/react/internals.ts` | browser | Fiber walker + `overrideHookState`/`overrideProps` through the hook shim (mirrors React DevTools)                                                                 |
| `medula`                 | node    | `createMedula()` devframe definition (+ `help` tool and resource), `medulaDockClientScript()`                                                                     |
| `medula/vite`            | node    | `medula()` Vite plugins: `createPluginFromDevframe` (Vite DevTools dock at `/__medula/`) + bootstrap injection + Svelte instrumentation. Needs Vite DevTools.     |
| `medula/next`            | node    | `<Medula />` head component (hook shims only). The hub is the app's: `nextDevframeHub({ devframes: [medulaHubEntry()] })` as in devframe's `hub-next` example     |
| `medula/nuxt`            | node    | Nuxt module: adds the Vite plugins (Nuxt DevTools 4 hosts Vite DevTools docks), inlines the bootstrap through `app.head`                                          |
| `medula/client`          | browser | Manual escape hatch: `exposeState(name, { get, set })` for state no devtools can reach                                                                            |
| `medula/vue              | react   | svelte`                                                                                                                                                           | browser | Manual helpers over `exposeState`; not needed for Vue/React apps |

`src/panel/` also holds the dock page (plain HTML + CSS); `panel.vite.config.ts` builds it and
`connect.js` (keeps its default export: the hub runtime calls it) into `dist-client/`, which is
the definition's `clientAssets`. The hub serves them at the dock base.

### How a tool call reaches the page

1. App code calls `exposeState()` (`src/client/registry.ts`). The registry lives on
   `globalThis[Symbol.for('medula:registry')]` so several bundles share it.
2. The first call creates the in-page channel (`src/client/channel.ts`,
   `createPageScriptChannel`). Its functions carry `agent` metadata, so devframe registers them in
   its global browser-agent registry.
3. The hub client runtime (Vite DevTools / Nuxt DevTools `embedded.js`, or `@devframes/hub-ui` in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [posva/medula](https://github.com/posva/medula) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
