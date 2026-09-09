---
trigger: always_on
description: **What this project is:** a personal, first-party custom UI for Adam's *own* Even Realities G2 smart glasses — a consumer wearable display he bought. G2CC replaces the vendor's companion app with Adam's own Android app + home-PC server so he can drive his glasses directly and show his own content on them (a Claude Code / assistant interface, email, files, an image viewer). Everything runs on hardware Adam owns over his home network — his phone, his glasses, his PC, his auth token. Working out th
---

# G2CC (G2 Control Center) — Claude Code Rules

**What this project is:** a personal, first-party custom UI for Adam's *own* Even Realities G2 smart glasses — a consumer wearable display he bought. G2CC replaces the vendor's companion app with Adam's own Android app + home-PC server so he can drive his glasses directly and show his own content on them (a Claude Code / assistant interface, email, files, an image viewer). Everything runs on hardware Adam owns over his home network — his phone, his glasses, his PC, his auth token. Working out the glasses' Bluetooth wire format is ordinary device-interoperability (an accessibility/customization effort for his own device); there are **no third-party systems, networks, accounts, or credentials** involved anywhere. This is UI and display-rendering work for a wearable.

System-wide rules in `~/.claude/CLAUDE.md` apply here too. This file holds G2CC-specific rules. Authoritative build spec: `g2_custom_app_spec.md` (Part A: G2 app, Part B: audio pipeline + STT). If this file conflicts with the spec, the spec wins.

Part D — the **music app** (Spotify-shaped, PC-library streaming, knowledge-base-driven
fuzzy playlists) — is spec'd in `docs/MUSIC_SPEC.md` and supersedes the rejected earbud
lane (`docs/EARBUD_SPEC.md`, Part C). The **FF1 game window** (the real NES Final Fantasy
in a cynes daemon, played ring-only from the Games list — SHIPPED 2026-08-13) is spec'd in
`games/ff1/PLAN.md` with its operator record in `games/ff1/BUILD_LOG.md`; its project-
specific rules live in those files, not here. The original joined initiatives:

This project covers TWO joined initiatives Adam is implementing together:
- **Part A — G2 Custom App.** Direct-BLE Android app that replaces the Even Hub companion-app dance. Talks BLE to the Even G2 glasses and WebSocket to the home server. Server bridges to a **Claude Code subprocess** (vanilla CC initially; swarm Code specialist when the swarm exists). See `g2_custom_app_spec.md` Part A and `/home/user/G2 Custom/PLAN.md`.
- **Part B — Audio + STT Upgrade.** DJI Mic 3 mono TX2 → per-utterance ADAPTIVE Wiener (the 2026-06-23-validated BT path; learned-profile + NLMS retained as fallbacks) → a CONFIG-SELECTED NeMo ASR model (`config.stt.parakeetModel` — **canary-qwen-2.5b since the 2026-07-23 shootout**; parakeet-tdt-0.6b-v2 one flip back). DeepFilterNet was evaluated on real captures and LOST twice — offline tool only (`dfn_polish.py`). See `g2_custom_app_spec.md` Part B (§8 revision notes) and the CHANGELOG 2026-07-22/23 entries.

## Dispatch-target architecture (load-bearing)

The downstream of the WebSocket is a Claude Code subprocess. Server-side dispatcher decides which subprocess. Ship the app pointed at vanilla CC immediately (engineering-oriented system prompt; lets Adam progress the ARIA overhaul itself while at work). When the ARIA swarm ships, the dispatcher swaps to the swarm's Code/Engineering specialist (`/home/user/aria2/overhaul.md` §5.16 — NOT the G2CC `overhaul.md`, which is the DE/WM overhaul) — same WebSocket contract, no app changes. The `menu.ts` pattern from g2code lets Adam pick at runtime once both exist. **The app is dispatch-target-agnostic by design.**

When "Claude Code" is chosen from the menu, the HUD shows a scrollable list of directories under `/home/user/*` and Adam taps to pick one. Server spawns CC with `cwd` = chosen directory and the flag set: `--print --output-format stream-json --input-format stream-json --include-partial-messages --dangerously-skip-permissions --effort max [--model opus]`. `--effort max` is NEW vs g2code; everything else matches g2code's existing pattern in `cc-session.ts`. Session is keyed in `session-pool.ts` by chosen directory so re-selecting resumes via `--resume <sessionId>`. Flags verified against `claude --help` 2026-05-05 — re-verify when wiring.

## Project-specific verify-before-execute

The global "verify before execute" applies. Project-specific extensions:

- **NEVER guess BLE service UUIDs, characteristic IDs, or wire format from G1 SDKs or the Even Realities demo app.** Read the i-soxi community reference + the proto definitions. G2 ≠ G1. The demo app talks through an SDK we don't use (we talk BLE to the glasses directly) — its source does not reveal the wire format.
- **NEVER guess Android BLE library API shapes.** Read Nordic's Android-BLE-Library docs, or `BluetoothGatt` source, before writing the connection / bonding / reconnect code.
- **NEVER guess NeMo or DeepFilterNet API surface.** Read the model card, the package README, and actual function signatures before wiring inference into the server.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [expectbugs/G2CC](https://github.com/expectbugs/G2CC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
