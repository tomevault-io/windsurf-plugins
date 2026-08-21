---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Global Rules

- Always search for best practices from the latest online research. Don't invent or assume, and don't be a pleaser. Be honest and factual.
- Look at the whole plan from top to bottom. Leave no stone unturned.
- Ask clarifying questions if you aren't 100% sure how to do something. Do not make assumptions.

## Project Overview

RigControl Web (`v1.0.0`) is a full-stack web + Electron desktop application for controlling amateur radio equipment via Hamlib's `rigctld`. It provides a real-time dashboard with frequency/mode/meter display, bidirectional Opus audio, browser-native H.264 video streaming, POTA/SOTA/WWFF spot integration, a CW iambic keyer, a GGMorse-based CW decoder, Hamlib UDP and FT-710 (FT4222 USB-SPI) spectrum scopes, solar/propagation data, a user-configurable panel grid layout, JWT-based user authentication with an admin panel, radio power on/off control (`set_powerstat`) with auto-reconnect, and a WSJTX rig-control/audio bridge for remote digital-mode operation. All transport runs over HTTPS (self-signed EC P-256 certificate, auto-generated at startup).

## Commands

```bash
# Development
npm run dev              # Start Express + Socket.io backend (tsx server.ts)
npm start                # Start the backend standalone (node server.ts) — no Electron; this is the headless entry point, see the wiki's Headless Deployment page
npm run build            # Build Vite frontend to dist/
npm run lint             # TypeScript type-check (tsc --noEmit)
npm run test             # Run Vitest unit tests (jsdom, hardware-independent)
npm run test:watch       # Vitest in watch mode
npm run test:ui          # Vitest interactive UI
npm run test:e2e         # Run Playwright e2e tests against a Dummy rigctld + synthetic UDP (hardware-independent)
npm run test:e2e:ui      # Playwright interactive UI, for debugging e2e tests
npm run test:hardware    # Run Playwright e2e tests requiring a real, USB-connected FT-710 (manual/local only, never CI)
npm run clean            # Remove dist/, dist-electron/, build/
npm run build:cw-helper      # Compile cw-key-helper.c for the current platform (scripts/build-cw-helper.mjs)
npm run build:ft4222-reader  # Compile ft4222-scope-reader.c for the current platform (scripts/build-ft4222-reader.mjs)
npm run build:rigctld-win-cross  # Cross-compile bin/windows/rigctld.exe from Linux via mingw64 (scripts/build-rigctld-win-cross.mjs) — for iterating on Windows-side rigctld fixes without a full CI run; see script header for one-time Fedora setup

# Electron
npm run electron:dev     # Run as Electron desktop app in dev mode
npm run build:electron   # Bundle electron/main.ts and electron/preload.ts via esbuild
npm run electron:build   # Full Electron production build (frontend + electron + package)
```

There is no hot-reload for `server.ts` or any module under `server/` — restart manually after backend changes.

## Testing

Three layers, split by what they need and whether real hardware is required:

- **Vitest unit tests** (`npm test`, jsdom environment) — pure logic only: hook reducers against a stub Socket.io client (`src/hooks/useSpectrum.test.ts`), server-side parsers (`server/rigComm.test.ts`). No real DOM canvas rendering, no real sockets — see `vitest.config.ts`. Server-only test files that don't need jsdom add a `// @vitest-environment node` docblock at the top.
- **Playwright e2e, hardware-independent** (`npm run test:e2e`) — drives the real app in a real browser against a real, isolated server instance, using three techniques to avoid needing physical hardware:
  - **Rig-status panels**: a real `rigctld` bound to Hamlib's built-in **Dummy** rig backend (model 1), not a hand-rolled fake — see `tests/fixtures/rigctld-dummy.ts`. This exercises the actual Hamlib protocol/parsing path end-to-end. Pilot: `tests/e2e/vfo-panel.spec.ts`.
  - **Hamlib UDP spectrum**: `server/spectrum.ts`'s multicast listener trusts any correctly-shaped sender, so `tests/fixtures/synthetic-udp.ts` sends synthetic Hamlib-5.x-shaped JSON packets directly — zero production code changes needed. Pilot: `tests/e2e/spectrum-hamlib-panel.spec.ts`.
  - **Backend audio** (`AudioFeedPanel`/`CwDecodePanel`/`SpectrumAudioPanel`): a real PipeWire `pw-loopback` (`tests/fixtures/audio-loopback.ts`) gives `server/audio.ts`'s `naudiodon` a real device to open, paired with Chromium `--use-fake-device-for-media-stream`/`--use-file-for-fake-audio-capture` flags (`tests/fixtures/fake-audio-wav.ts`) feeding a real WAV signal as the browser mic. See the Backlog note below for the non-obvious constraints this involved. Pilot: `tests/e2e/audio-panels.spec.ts`.
- **Playwright e2e, hardware-dependent** (`npm run test:hardware`, separate `playwright.hardware.config.ts` + `tests/e2e-hardware/`) — requires a real, USB-connected FT-710 with `libft4222` installed. The FT4222 USB-SPI chain can't be meaningfully faked, so this drives the real hardware. Fails fast with a clear message (not a hang) if the reader can't open the device. **Deliberately excluded from CI** — no GitHub-hosted (or typical self-hosted) runner has a physical FT-710 attached.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jbdubbs/Rig-Control-Web](https://github.com/jbdubbs/Rig-Control-Web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
