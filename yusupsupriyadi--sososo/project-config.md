---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`sososo` is a cross-platform desktop app (Windows · macOS · Linux) for **real-time meeting/audio
transcription**. It captures system audio (WASAPI loopback on Windows; CoreAudio + BlackHole on macOS;
PulseAudio/PipeWire sink monitor on Linux) + microphone, streams both to Deepgram for live
speech-to-text, and shows live captions in a single transparent-glass window that switches to a full
transcription view (with
pause/finish) while recording, plus a library/settings/history UI when idle. User-facing UI copy is
in **English** (the app was switched from Bahasa Indonesia to English).

Stack: Tauri 2 (Rust) backend · React 19 + React Router 7 + Zustand 5 + Vite 7 + Tailwind CSS v4
(TypeScript) frontend · **Bun** package manager.

## Commands

Run from the repo root. The package manager is **Bun** — do not use npm/yarn/pnpm.

| Task                                         | Command                                        |
| -------------------------------------------- | ---------------------------------------------- |
| Run the full desktop app (dev)               | `bun run tauri dev`                            |
| Frontend only (browser, no Tauri APIs)       | `bun run dev` → http://localhost:1420          |
| Typecheck + build frontend                   | `bun run build` (runs `tsc` then `vite build`) |
| Build the installer/bundle                   | `bun run tauri build`                          |
| Rust check / lint (from `src-tauri/`)        | `cargo check` · `cargo clippy`                 |
| Audio capture smoke test (from `src-tauri/`) | `cargo run --example audio_probe -- 6`         |

- **No unit-test framework is configured.** Verification = `bun run build` (TS), `cargo check` /
  `cargo clippy` (Rust), and the `audio_probe` example (writes `audio_probe.wav` — L=mic, R=system —
  and prints per-channel RMS to confirm both channels carry audio).
- TypeScript is **strict** with `noUnusedLocals` / `noUnusedParameters`: unused vars/params fail the build.
- `cargo` is installed via rustup at `~/.cargo/bin`; `bun run tauri *` invokes it for you.

## Architecture

### One window, state-driven views

A single Vite build serves the one window (`AppRouter.tsx` via `HashRouter`, `index.html#/main` →
`windows/main/MainApp`). The **main** window is the only window (declared in `tauri.conf.json`);
there is no separate overlay window anymore.

`MainApp` is **session-state-driven**: while a session is active (`starting`/`recording`/`stopping`)
it renders `RecordingView` as a **compact, always-on-top floating widget** — a small pill with two icon
buttons (**yellow = pause/resume, red = finish**) above the live-transcript panel. `RecordingView`
shrinks the window via `enterRecordingWindow` on mount and restores it via `exitRecordingWindow` on
unmount (`lib/window.ts`; needs the `set-size`/`center`/`set-always-on-top` window capabilities and a
small `minWidth`/`minHeight` in `tauri.conf.json`). Otherwise `MainApp` renders the normal layout
(titlebar + session sidebar + library/settings/session-detail routes). When a session ends it
navigates to the session detail (where the AI summary lives) if anything was transcribed, else home.

Transparent glass = `transparent:true` + `decorations:false` window with **no** native acrylic/vibrancy;
the desktop behind shows through sharply. Styling is **Tailwind CSS v4**
(`@tailwindcss/vite`, no config file) and **utility-first**: all component styling is inline utility
classes in JSX. The single `src/styles/app.css` holds only `@import "tailwindcss"`, the `@theme` design
tokens (colors, radii, `--font-sans`, `--shadow-liquid`, `--animate-rec-pulse`), the `rec-pulse`
`@keyframes`, one custom `@utility liquid-glass`, and an `@layer base` reset (transparent bg, fonts,
scrollbar, dark `<option>`, drag-region cursor). There is no `@layer components`.

The **"liquid glass"** look = a translucent fill + a bright glassy **edge** (white border + inset top
highlight + soft inner glow); the edge is what reads as "liquid glass" and stays. Only the distracting
diagonal specular **sheen** over the panel background (former `::before`/`::after`) was removed. `@utility
liquid-glass` (panels, sidebar, titlebar, pill, recording panel) fill = `rgb(28 28 34 / var(--glass-alpha,0.58))`.
The fill **alpha** (transparency) is user-controllable in Settings → Appearance → Background transparency;
`MainApp` writes `--glass-alpha` as a `:root` var so every glass surface (shell + recording widget) reacts
live (persisted in `configStore`). There is no window blur — CSS backdrop-filter can't frost the desktop
behind a transparent window, and native acrylic was tried and removed. Buttons get the glossy edge via the
`shadow-liquid` utility. Exact non-scale values use arbitrary utilities (`text-[13px]`, `bg-[rgba(110,168,254,0.2)]`).

### Audio → STT pipeline (the core data flow)

`commands::start_session` → `session::spawn_session` → async `run_session` (Tauri/tokio runtime):

1. **Capture** (`audio/capture/`, cfg-gated per OS): mic and system-loopback each run on a dedicated

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yusupsupriyadi/sososo](https://github.com/yusupsupriyadi/sososo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-07 -->
