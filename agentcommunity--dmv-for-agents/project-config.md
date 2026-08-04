---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Project

```bash
uv run python -m http.server 8080
# open http://localhost:8080
```

No build system for the SPA. Native ES modules via browser importmap. Serve from project root.

## Deployment

Cloudflare Workers Static Assets + a Cloudflare Container (Node 20 + `@napi-rs/canvas` Skia) for card rendering. Worker `dmv-agentcommunity` on Taqanu account, container instance type `lite`. The Cloudflare Git integration's automatic build from merged `main` is the single authoritative production deploy path. Observe that build; use `pnpm cf:deploy` only after confirming that an automatic build is neither active nor already started, and never run both paths concurrently. Local dev: `pnpm cf:dev`. Architecture, cache hierarchy, and drift invariants: [CLOUDFLARE.md](CLOUDFLARE.md). Broader migration history + container rename gotcha: `docs/admin/CLOUDFLARE-MIGRATION.md` in the `agentcommunity_page` repo.

## Architecture

Pre-registration system for `.agent` domain identities. Two interfaces, one backend:

1. **Web CRT terminal** — 3D retro TV with interactive form (for humans & organizations)
2. **CLI CRT terminal** — ASCII art terminal in the shell (for AI agents, operator required)

All flows are **pre-registration** (not registration). Pre-registration records interest in a `.agent` domain. It does not guarantee assignment.

**Web data flow:** Scroll (or tap-the-monitor on landing — 3s GSAP `scrollTop` tween) drives camera zoom → CRT boots at 60% progress → type selector (org/individual) → conditional form fields with validation → review/submit (TnC + Charter links, submit button) → invisible Turnstile widget executes → POST to same-origin `/api/register` worker → worker verifies Turnstile (hostname + `dmv_register` action) → worker checks shared CF rate limits → worker forwards to Supabase → review state holds until response, then processing bar plays → `CRTTerminal.onComplete(formData)` fires with the authoritative cert ID → `HoloCard.show(formData)` draws holographic card with rarity-based shader effects → card bobs + tilts toward mouse/gyro → card is clickable to zoom. **Full navigation/exit contract: [NAVIGATION.md](NAVIGATION.md).**

**CLI data flow:** Boot screen (about/terms/charter menu) → step-by-step form (agent name → operator [required] → email → description) → confirmation summary → Y/n gate → POST to `/api/register` worker (with `signup_source: 'cli'` and `machine_fingerprint`) → worker checks shared CF rate limits → worker checks DMV-local KV fingerprint cooldown → worker forwards to Supabase → success screen: view card link (permalink to holographic card), share nudge (invite command + card URL), save card (direct PNG download URL), badge markdown snippet, email verification note.

**Module graph:**
```
app.js ─┬─► TV.js ──► CRTTerminal.js   (TV owns CRT, uses its canvas as Three.js texture)
        ├─► Intro.js ──► volumetric-pass.js   (cinematic click-to-enter intro: god-ray lamp + music sync)
        ├─► HoloCard.js ──► card-draw.js ──► qr-encode.js
        ├─► WallSign.js                (wall sign above TV, fluorescent flicker animation)
        └─► AboutPoster.js             (about panel)
```

- `app.js` — Entry point. Wires TV + HoloCard + AboutPoster, events (scroll, click, keyboard, resize, gyro), sound toggle, clock, permalink routing. Top-level await, no exports.
- `TV.js` — Three.js scene: GLTF model loading (Draco), camera, renderer, night mode toggle, raycaster, card/about zoom/unzoom, `onRender(cb)` callbacks, render loop with delta time.
- `CRTTerminal.js` — Pure Canvas2D, no Three.js dependency. 8-phase boot state machine (off, flicker, boot text, type selector, form, review/submit, processing, done), conditional form fields with validation, color scheme swapping, CRT visual effects.
- `HoloCard.js` — Self-contained holographic card module. Custom ShaderMaterial (GLSL) with rainbow iridescence, foil lines, glare, fresnel, sparkle. Front + back faces with Canvas2D content. Rarity system, identicon, scannable QR code (real encoder). Bob + tilt animation. See [CARD.md](CARD.md).
- `WallSign.js` — Wall sign above the TV. PlaneGeometry + CanvasTexture with "DEPT. OF MACHINE VERIFICATION" title and "SELF-SERVE KIOSK" subtitle. Fluorescent tube flicker-on animation (GSAP timeline) fires ~1.2s after page load (not scroll-linked). Ambient flicker loop (random subtle opacity dips every 4-7s) runs after startup. Theme-aware via CSS custom properties. Self-contained with `dispose()` cleanup.
- `AboutPoster.js` — PlaneGeometry + CanvasTexture. UI-style about text, toggle show/hide.
- `Intro.js` / `volumetric-pass.js` — Cinematic "video mode" intro: a **click-to-enter** gate (one tap starts intro + music together, so audio is in sync in every browser incl. Safari), a real shadow-aware **volumetric god-ray "lamp"** that arcs behind→over→front (silhouette → beams → front-lit reveal), the fade-to-black handoff to the normal scene, and **music synced so the track's drop lands on the reveal**. Tunable via `?tune` (`js/intro-control-panel.js`, DEV-TUNE-gated). See [INTRO.md](INTRO.md).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentcommunity/DMV_for_agents](https://github.com/agentcommunity/DMV_for_agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
