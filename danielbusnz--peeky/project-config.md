---
trigger: always_on
description: Orientation and rules for humans and agents working in this repo. The first half maps the codebase (what it is, how a voice turn flows, where things live). The second half is the working rules. If you use an agent, run it from the repo root so it picks this file up automatically.
---

# Peeky Agent Guide

Orientation and rules for humans and agents working in this repo. The first half maps the codebase (what it is, how a voice turn flows, where things live). The second half is the working rules. If you use an agent, run it from the repo root so it picks this file up automatically.

## Overview

Peeky is a voice-controlled AI cursor for Linux, written in Rust. Hold a push-to-talk hotkey, say something, release. The transcript is classified into one of five intents, the matching path runs, and the reply streams back as speech (and, when relevant, the cursor moves or a real click/type fires).

A single voice turn flows like this:

1. **Hotkey** (`peeky/src/hotkey/`) flips a global `RECORDING` atomic. On Hyprland, `bind`/`bindr` send `SIGUSR1` (press) / `SIGUSR2` (release) to the process; a signal-hook listener thread flips the flag. Other platforms use the `global-hotkey` crate polled from the winit loop.
2. **Audio capture** (`peeky/src/audio/input.rs`) holds a persistent `cpal` mic stream and forwards PCM to the STT channel while the key is held, with a pre-roll ring and a post-release grace window.
3. **STT** (`peeky/src/providers/stt_deepgram.rs`) streams PCM over a Deepgram websocket and returns the final transcript on release, after a short quiescence wait for multi-segment utterances.
4. **Classify** (hybrid cascade). An exact-match keyword allowlist (`peeky/src/intent.rs`) catches bare transport commands ("play", "skip") sub-millisecond. Otherwise the on-device routelet classifier (`peeky/src/routelet/`) runs (~5-30ms) and returns an intent with a confidence; a confident, non-`none` result is used directly. Only a low-confidence or `none` (reject) result falls through to the LLM classifier (`peeky/src/providers/claude/classifier.rs`), a forced-tool Claude call spawned in parallel with the screenshot capture so its latency is mostly hidden.
5. **Dispatch** (`peeky/src/orchestrator.rs`) routes to one of five paths, all under `peeky/src/providers/claude/`: `find_action`, `integration`, `chat`, `memory`, `agent`.
6. **TTS** (`peeky/src/providers/tts_cartesia.rs`). Claude deltas are split into sentences and streamed to Cartesia, which synthesizes PCM into the `rodio` sink. The first flush is permissive (fast first audio); later flushes are strict (natural prosody).
7. **Barge-in** (`peeky/src/barge_in.rs`). A watchdog polls the hotkey; a re-press mid-turn cancels the in-flight Claude and Cartesia streams so the next turn starts clean.

`peeky/src/orchestrator.rs` is the core loop: one voice turn per iteration. Start there.

## Architecture

**Workspace** (`Cargo.toml`). Three Rust members: `peeky` (the agent binary plus its library), `demos` (hand-run dev tools and benchmarks), and `console/src-tauri` (the Tauri desktop GUI: onboarding, sign-in, settings). The `proxy/` directory is **not** a workspace member: it is a TypeScript Cloudflare Worker.

**The `peeky` crate** splits into `lib.rs` (every subsystem exposed as a public module) and a thin `main.rs`, so the out-of-tree `demos` crate builds against the same modules. Default feature is `hyprland`; the winit/X11 path builds with `--no-default-features`.

**Platform-backend pattern.** Several subsystems (`hotkey/`, `input/`, `desktop/`, `screenshot/`, `mouse_position/`, `ai_cursor/`) share one shape: a `mod.rs` facade, a `backend.rs` trait, and per-OS implementations (`hyprland.rs`, `macos.rs`, `windows.rs`, `crossplatform.rs`/`winit.rs`) selected by feature flags. When adding a platform capability, add it to the trait and implement it in every backend.

**Sibling crates and services:**

- **console** (`console/src-tauri/`): Tauri 2 desktop GUI. Frontend pages under `console/ui/` (`onboarding/` one-time setup, `settings/` the signed-in surface, `shared/`, `icons/`). First-run onboarding collects an invite code or the user's own API keys (stored in the OS keychain), requests macOS TCC permissions, then spawns the `peeky` binary as a child with the right env. If `~/.config/peeky/onboarded` exists, it spawns silently and exits. Also hosts GitHub sign-in and (soon) the settings/integrations surface.
- **proxy** (`proxy/src/index.ts`): the Cloudflare Worker that holds the real API keys and enforces per-tier usage caps. Deployed with Wrangler. See routes below.

## API Proxy

The app ships without API keys. By default every provider call routes through the Worker, which holds the secrets and meters usage against a per-UTC-day budget. The tier is resolved per request: an invite code (`x-peeky-invite-code`) is the demo tier, else a valid session token (`Authorization: Bearer`) is the account tier for signed-in users, else the anonymous trial tier. When the trial budget is spent the agent speaks an upgrade prompt and opens GitHub sign-in in the browser (`peeky/src/upgrade.rs`); a successful login writes the session token the agent reads on its next turn, no restart. Each provider can be pointed straight at its upstream with an `PEEKY_*_DIRECT=1` env var plus the matching key (see "Use your own API keys" in the README).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danielbusnz/Peeky](https://github.com/danielbusnz/Peeky) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
