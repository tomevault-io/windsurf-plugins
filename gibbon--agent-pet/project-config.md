---
trigger: always_on
description: Guide for humans and AI coding agents working in this repo. If you're an LLM, read this top-to-bottom before touching code — it will save you (and the reviewer) round-trips.
---

# AGENTS.md

Guide for humans and AI coding agents working in this repo. If you're an LLM, read this top-to-bottom before touching code — it will save you (and the reviewer) round-trips.

For end-user / integrator docs, read the [README](./README.md). For release/PR mechanics, read [CONTRIBUTING.md](./CONTRIBUTING.md). This file covers the parts that aren't obvious from those.

---

## What this is

`agent-pet` is a tiny animated companion-pet widget. It ships in three forms from one source tree:

| Bundle | Entry | Where it's used |
|---|---|---|
| IIFE (`dist/agent-pet-widget.iife.js`) | `src/widget/index.ts` | CDN `<script>` tag — sets `window.AgentPet` |
| Widget ESM (`dist/agent-pet-widget.es.js`) | `src/widget/widget-es.ts` | `import { createAgentPetAPI } from 'agent-pet/widget'` for non-React frameworks |
| React ESM (`dist/agent-pet.js`) | `src/index.ts` | `import { PetProvider, PetOverlay } from 'agent-pet'` |

Hard constraints — keep these in mind on every change:

- **IIFE bundle ≤ 15 KB gzip.** CI fails above that (`.github/workflows/ci.yml`). Vanilla DOM only — no React, no Preact, no framework runtime in the widget code path.
- **No baked spritesheets.** Spritesheets live on a CDN or in the host site's `public/`. The bundle ships zero pet image data.
- **No backend.** The widget makes no network calls beyond the spritesheet `<img src>` you point it at.
- **`/v0.X/` paths on the CDN are immutable.** Once shipped they don't change. New versions get a new bucket. See `CONTRIBUTING.md` for release flow.

## Repo layout

```
src/
  core/                  framework-agnostic logic
    atlas.ts             Codex 8×9 atlas spec + custom-atlas plumbing
    pets.ts, types.ts    public types (PetState, PetMessages, PetIcons, ...)
    image.ts             sprite loading + sizing
    providers/           pet-source providers (codex, hatchery, registry, types)
    adapters/default.ts  state → atlas-row mapping
  widget/                vanilla-DOM widget — the IIFE/ESM target
    api.ts               createAgentPetAPI() — public surface
    mount.ts overlay.ts sprite.ts queue.ts observer.ts
    registry.ts          multi-pet registry (window.AgentPet.create(id, ...))
    index.ts             IIFE entry — boots from <script> data-* attrs
    widget-es.ts         ESM entry — exports factories, no auto-boot
  react/                 React components (PetOverlay, PetSettings, PetRail, ...)
  shared/global.ts       window.AgentPet typing
scripts/
  vendor-pet.mjs         download a codex-pets.net spritesheet to public/sprites/
  build-pages.mjs        compose Cloudflare Pages output
  sri.mjs                emit dist/SRI.json with sha384 hashes
  screenshot.mjs         hero screenshot for README
public/                  Cloudflare Pages site (demo + versioned bundles)
examples/                hand-authored HTML demos — also referenced from the README
docs/                    README assets (hero.png etc) — not user-facing docs
```

If you're adding a new file, ask yourself which bucket it belongs in *before* creating it — `core` for logic that should be reusable from React or vanilla, `widget` for things that touch DOM directly, `react` for JSX. Don't import React from anywhere under `core/` or `widget/`.

## Dev loop

```bash
pnpm install
pnpm typecheck       # tsc --noEmit
pnpm test            # vitest run
pnpm build           # all three bundles + dist/SRI.json
```

Try the examples locally:

```bash
npx serve . -p 5174
# http://localhost:5174/examples/auto-mount.html
# http://localhost:5174/examples/multi-pet.html
# http://localhost:5174/examples/observe.html
# http://localhost:5174/examples/self-hosted/index.html
```

For UI work, **actually load an example in a browser and exercise the change** before claiming the task is done. Type-checks and unit tests verify code correctness, not feature correctness — if you can't verify a UI change visually, say so explicitly in the PR.

Node ≥ 20, pnpm ≥ 9. macOS / Linux / WSL2.

## Sprites: getting them, generating them, shipping them

There are four legitimate ways to get a sprite onto a page — pick whichever matches the task.

### 1. By codex-pets.net id (zero setup)

```html
<script src="https://agent-pet.pages.dev/v0.8/agent-pet-widget.iife.js"
        data-codex-pet="homelander"></script>
```

The widget resolves the URL through `src/core/providers/codex.ts` and applies the standard 8×9 Codex atlas layout. Slugs come from the URL path on [codex-pets.net](https://codex-pets.net/) (e.g. `/pets/homelander`).

### 2. Vendored locally (zero external requests at runtime)

```bash
pnpm vendor-pet homelander guga totoro
# → public/sprites/<id>.webp
```

Then point the widget at the local path:

```html
<script src="/agent-pet-widget.iife.js"
        data-image-url="/sprites/homelander.webp"
        data-use-codex-atlas></script>
```

`scripts/vendor-pet.mjs` is intentionally dumb: a `fetch` of `https://codex-pets.net/assets/pets/<id>/spritesheet.webp` written to `public/sprites/`. Don't commit downloaded sprites to this repo — they belong to their creators.

### 3. From the `codex-pets` CLI (locally installed pets)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gibbon/agent-pet](https://github.com/gibbon/agent-pet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
