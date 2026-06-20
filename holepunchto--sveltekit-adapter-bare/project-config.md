---
trigger: always_on
description: This skill should be used when building, modifying, or debugging a SvelteKit application running on the Bare runtime with the Holepunch / pear stack (Hypercore, Hyperswarm, Hyperbee, HyperDB, Corestore, DistributedDrive, Localdrive, Hyperdrive). Triggers on requests mentioning "svelte bare app", "SvelteKit on bare", "P2P svelte", "sveltekit-adapter-bare", "SSE in SvelteKit", "hyperswarm + svelte", "live stats stream", "ghost drive", or any task that involves wiring SvelteKit server endpoints to 
---


# SvelteKit + Bare app

A SvelteKit application whose server side runs inside the Bare runtime and owns a long-lived P2P stack (Corestore + Hyperswarm + HyperDB / DistributedDrive). The server is a singleton; the browser only ever talks to SvelteKit endpoints.

If you only remember five things:

1. **Never block rendering.** Every load function must return immediately — put async work in a `Promise` value so SvelteKit can stream. Use `{#await}` in templates, never `.then()` chains.
2. **The server stack is a long-lived singleton, not request-scoped.** Boot it in `hooks.server.ts`; park it on `event.locals`.
3. **All load logic lives in `$lib/server/loaders.ts`.** Route server files are thin coordinators that import from loaders and return streamed promises.
4. **TypeScript for the untyped holepunch world.** Use `src/lib/server/ambient.d.ts` for bare-\* and holepunch packages that ship no types. Use `import type` for circular deps between server modules.
5. **`sveltekit:close` is your cleanup hook.** Register teardown in `process.on('sveltekit:close', ...)` inside `hooks.server.ts`.

## Architecture at a glance

```
┌─────────────────────────────────────────────────────────────────┐
│  Bare process                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  SvelteKit (sveltekit-adapter-bare)                        │ │
│  │                                                            │ │
│  │  hooks.server.ts ──► GhostDriveApp ──► event.locals.app   │ │
│  │                                                            │ │
│  │  $lib/server/loaders.ts  ◄── routes/**/+page.server.ts    │ │
│  │  $lib/server/app.ts                                        │ │
│  │  $lib/server/session.ts                                    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  Long-lived stack                                          │ │
│  │  Corestore ── Hyperswarm ── HyperDB ── DistributedDrive   │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Project scaffolding

### `package.json`

Every Bare SvelteKit app needs these exact groups. Missing any of the bare-\* runtime deps causes a silent build failure or a crash at startup.

**Runtime dependencies** (bundled into the app):

```json
"dependencies": {
  "bare-crypto":  "^1.13.6",
  "bare-fetch":   "^3.0.1",
  "bare-http1":   "^4.5.6",
  "bare-module":  "^6.2.0",
  "bare-native":  "^0.1.2",
  "bare-process": "^4.4.1"
}
```

**Dev dependencies**:

```json
"devDependencies": {
  "bare-build":            "^0.5.5",
  "sveltekit-adapter-bare": "...",
  "@sveltejs/kit":         "...",
  "svelte":                "...",
  "vite":                  "..."
}
```

**Build scripts** (substitute `<app>`, `<AppName>`, `com.<org>.<app>`):

```json
"scripts": {
  "dev":          "vite dev",
  "build":        "vite build",
  "make:darwin":  "bare-build --out out/<app>-darwin-arm64 --host darwin-arm64 --icon <app>.png --name <AppName> --runtime bare-native/runtime build/index.js",
  "make:android": "bare-build --out out/android-arm64 --resources resources/android --host android-arm64 --manifest manifest.xml --identifier com.<org>.<app> --name <AppName> --runtime bare-native/runtime build/index.js"
}
```

> **Icon caveat**: `--icon <app>.png` requires a real PNG in the project root. This file cannot be auto-generated — ask the user to provide it before wiring up `make:darwin`. Do not synthesise a placeholder and proceed silently.

### `svelte.config.js`

Two settings beyond the adapter are required and non-obvious:

1. **`runes` scoped** — enforce runes mode only outside `node_modules`; some Holepunch deps don't use runes and will break with `runes: true` globally.
2. **`csrf: { checkOrigin: false }`** — form actions are broken without this inside Bare (the request origin never matches the server origin).

```js
import adapter from 'sveltekit-adapter-bare'

const config = {
  compilerOptions: {
    runes: ({ filename }) => (filename.split(/[/\\]/).includes('node_modules') ? undefined : true)
  },
  kit: {
    adapter: adapter({ window: { width: 1200, height: 800 } }),
    csrf: { checkOrigin: false }
  }
}

export default config
```

### `manifest.xml` (Android only)

Required by `make:android`. Place it in the project root. Minimum viable manifest — substitute `package`, `android:label`, and version fields:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<manifest
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:versionCode="1"
    android:versionName="1.0"
    package="com.yourorg.yourapp"
>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [holepunchto/sveltekit-adapter-bare](https://github.com/holepunchto/sveltekit-adapter-bare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
