---
trigger: always_on
description: Use when starting, maintaining, flashing, running, or troubleshooting a StackChan remote-agent project, including StackChan architecture, ESP-IDF under this skill's vendor/esp-idf, vendoring m5stack/StackChan under this skill's vendor/StackChan, maintaining app_remote_agent firmware, linking it into the StackChan firmware tree, building/flashing/monitoring with idf.py, running the Bun brain server, and debugging the WebSocket protocol between StackChan and the server.
---


# StackChan Skill

Use this skill to start, operate, maintain, flash, and troubleshoot a complete StackChan remote-agent system. Treat StackChan as a thin Wi-Fi body and the local server as the agent brain.

## First Decisions

| If you need to... | Read |
|---|---|
| Understand what StackChan is and what hardware is available | `references/stackchan-overview.md` |
| Choose the repo architecture and split firmware from brain code | `references/starter-architecture.md` |
| Install ESP-IDF locally for the project | `references/esp-idf-install.md` |
| Vendor the upstream StackChan source | `references/vendor-stackchan.md` |
| Add the reusable remote-agent firmware app | `references/firmware-integration.md` |
| Build, flash, or monitor the firmware | `references/firmware-build-flash.md` |
| Implement compatible server/device messages | `references/device-protocol.md` |
| Create, run, or maintain the Bun TypeScript brain server | `references/bun-brain-starter.md` |
| Build a full voice agent server with Deepgram, TypeScript, and tools | `references/full-stack-voice-agent.md` |
| Bridge a remote brain through Tailscale without firmware changes | `references/tailscale-lan-proxy.md` |
| Add server-selected wake-word standby with microWakeWord | `references/wake-word-standby.md` |
| Add custom server-driven avatar scenes or use the local render simulator | `references/server-driven-rendering.md` |
| Diagnose build, flash, link, or runtime failures | `references/troubleshooting.md` |

## Common Workflows

| If the user asks to... | Do this |
|---|---|
| Start from a blank repo | Follow `Default Setup Path`, then verify the completion checklist. |
| Maintain existing firmware | Inspect `vendor/StackChan` status, app symlinks/files, `apps.h`, `main.cpp`, and HTTPD WebSocket support before editing. |
| Build or flash | From this skill root, source `vendor/esp-idf/export.sh`, work from `vendor/StackChan/firmware`, and use `references/firmware-build-flash.md`. |
| Run the remote agent | Open `REMOTE.AGENT` on StackChan, set the brain server `STACKY_DEVICE_WS_URL` to `ws://STACKCHAN_HOST:6001/stacky/device`, start the Bun brain server, then verify `/health`. |
| Debug a disconnected robot | Check StackChan Wi-Fi, the device-hosted WebSocket URL, brain outbound connection logs, and device telemetry. |
| Debug `Streaming mic...` after wake word or speech | Read `references/troubleshooting.md`; inspect firmware audio restart logs before changing server STT/TTS code. |
| Change protocol or commands | Update firmware, server `device/protocol`, command helpers, and docs together. |
| Add wake-word standby | Keep the server as policy owner, send `standby` with an optional `wakeWord`, and make firmware advertise `wakeWord` only after a real local detector is integrated. |
| Bridge a remote brain over Tailscale | Run `bun run proxy:stacky` on a LAN machine that can reach StackChan and is on the tailnet; point the remote brain at the proxy's Tailscale URL. |

## Default Setup Path

1. Use this skill repo's `vendor/` directory for firmware dependencies.
2. Install ESP-IDF v5.5.4 into `vendor/esp-idf`; source `vendor/esp-idf/export.sh` before using `idf.py`.
3. Clone upstream StackChan from `https://github.com/m5stack/StackChan` into `vendor/StackChan`. This upstream repo contains the official StackChan open-source resources; its ESP-IDF project lives under `vendor/StackChan/firmware/`.
4. Run `scripts/patch-stackchan.sh` to add `esp_http_server` and HTTPD WebSocket support to the vendored firmware build.
5. Copy or link `assets/app_remote_agent/` into this skill repo's `vendor/StackChan/firmware/main/apps/app_remote_agent/`.
6. Register `AppRemoteAgent` in the vendored firmware app list and `main.cpp`.
7. Run `assets/app_remote_agent/boot-into-remote-agent.sh` so StackChan opens `REMOTE.AGENT` on boot while preserving launcher/home access.
8. Build from `vendor/StackChan/firmware`, never from the repo root.
9. Flash only after choosing the correct serial port or confirming ESP-IDF auto-detect is safe.
10. Implement the brain server against the protocol in `references/device-protocol.md`.

## Run Path

1. Ensure the brain server has `STACKY_DEVICE_WS_URL` configured.
2. Start the Bun server from the target app repo, usually with `bun run dev` or `bun run start`.
3. Open `/health` and verify the server is reachable on the LAN address used by firmware.
4. Flash or run firmware; it hosts `ws://<stackchan>:6001/stacky/device`.
5. Open the `REMOTE.AGENT` app on StackChan.
6. Use the browser debug UI or `/stacky/debug` stream to verify `hello`, telemetry, commands, acks, and errors.

## Non-Negotiable Constraints

- Keep reusable firmware and firmware vendors in this skill repo, not in a downstream custom brain repo.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mountgram/stack-chan-skill](https://github.com/mountgram/stack-chan-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
