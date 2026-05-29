---
trigger: always_on
description: This file is for code-modifying agents working inside the StemStudio Engine
---

# StemStudio Engine — Agent Guide

This file is for code-modifying agents working inside the StemStudio Engine
open-source repository. It is intentionally a router and an architecture map,
not a tutorial. Treat the code as the source of truth; this guide tells you
where to look and what rules to honour.

## Core rules

1. Search before you write. Reuse existing code and patterns.
2. Ask clarifying questions when scope or product intent is ambiguous.
3. Build a short plan in `docs/planning/` before non-trivial changes, then
   implement against it.
4. Keep changes small and local. Pause before large or breaking edits.
5. Read the relevant code before editing it.
6. Prefer the code as the source of truth — when docs disagree, fix the docs.
7. Update affected docs after a change.
8. Dispose Three.js resources you create: geometries, materials, textures,
   render targets, listeners.
9. Run the right verification before declaring work done. Never use
   `eslint --fix`.

## What this build is

A browser-based 3D editor and runtime built on Three.js with a React UI, a
behavior system, an ECS-style lambda layer, Ammo.js / Rapier physics, a
Colyseus multiplayer sidecar, and a Go AI proxy that forwards calls to the
user's own provider keys (BYOK).

Projects persist locally — IndexedDB by default, or a real folder via the
File System Access API. There is no hosted backend in this build: no
authentication, no cloud project store, no telemetry. Every feature you
see in the UI runs against local state or against a service the user
explicitly configured.

## Architecture at a glance

```text
client/packages/
  editor-oss/         The editor: scene tree, viewport, behaviors,
                      lambdas, physics adapters, scheduler, rendering,
                      asset loading, serialization, the Copilot panel,
                      runtime UIKit/HUD.
  copilot/            Provider-agnostic copilot interfaces and a basic
                      chat panel for forks. Editor-oss reaches the
                      provider through ICopilotProvider — no concrete
                      provider is bundled in this build by default.
  network/            HTTP/WS adapters. The remote-go adapter in here is
                      the only thing that talks to the AI server. Other
                      paths read/write the local ProjectStore directly.
  shared/             Cross-package types, build-mode flags, queryClient,
                      Sentry, AppContainer shell.
  play/               The Player-only runtime — the entry point a built
                      game uses when it ships standalone.
  marketing/          Marketing pages used by the dashboard shell.

server/               Go HTTP/WS server. The AI subset (`cmd/ai-server`)
                      is the only binary that ships here — it proxies AI
                      provider calls using the BYOK keys.

stemstudio-multiplayer/  Colyseus server. Auto-started by `bun run dev`.
stemstudio-copilot/      Optional ACP/MCP bridge for forks that want
                         Claude Code-style tool use.

docs/                  Engine subsystem docs and planning.
scripts/               Build, export, and Playwright smokes.
```

`__BUILD_MODE__` is fixed to `oss` in this build, and the `IS_OSS` /
`IS_INTEGRATED` flags in `@web-shared/buildMode` flow from that. Code that
checks `IS_OSS` is the seam where this build deliberately diverges from
features that would require a hosted backend — auth, cloud asset storage,
telemetry, hosted multiplayer. Keep those gates intact when refactoring.

## Persistence

Project bodies (`{meta, sceneJson, ...}`) flow through the
`ProjectStore` interface in `client/packages/editor-oss/src/persistence/`:

- `IndexedDBProjectStore` — default; one row per project keyed by ID.
- `FileSystemProjectStore` — writes `<name>.<id>.stemscript.json` files
  into a folder the user picked via `showDirectoryPicker`. The picked
  handle persists in IndexedDB (`fsHandleStore.ts`) so subsequent reloads
  reattach without re-prompting on Chromium.

`projectStoreFactory.ts` is the singleton boundary. The OSS first-run
modal (`OSSBootstrapModal.tsx`) and the dashboard banners
(`OpenFolderBanner.tsx`, `ReconnectFolderBanner.tsx`) are the only
surfaces that swap the active store at runtime; everything else reads
through `getProjectStore()`.

When you add a new feature that needs to write data, route it through
`ProjectStore` — do not invent a parallel storage path.

## Behaviors

- Base type and lifecycle live in
  `client/packages/editor-oss/src/behaviors/Behavior.ts`.
- Behaviors register through `BehaviorTypeRegistry`. Saved scenes embed
  the per-instance config in `scene.userData.behaviorConfigs`. Built-in
  behaviors are referenced by id only; full configs hydrate from the
  in-process registry on load.
- Reach the engine from inside a behavior via `this.erth.*` and
  `this.gameObject`. Old `EventBus` and `this.target` style code is
  deprecated.
- Lifecycle docs: `docs/behaviors/`.

## Lambdas (ECS)

`client/packages/editor-oss/src/lambdas/` — archetype-driven systems on
top of behaviors. Use when you need batched, dependency-scheduled work.
See `docs/lambdas/` for the architecture internals.

## Scheduler, rendering, quality


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Stem-Studio/Engine](https://github.com/Stem-Studio/Engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
