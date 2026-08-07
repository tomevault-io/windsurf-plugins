---
trigger: always_on
description: This file provides guidance to Codex and other coding agents when working in
---

# AGENTS.md

This file provides guidance to Codex and other coding agents when working in
this repository. Treat `CLAUDE.md` as the upstream source of truth; this file is
the Codex-compatible mirror of those project rules.

## Multi-IDE Rules

This repo is developed across multiple AI-assisted environments. The following
rules apply to all code changes:

1. **Update docs on code change** - When changing behavior, architecture, or
   APIs, update both the English and Vietnamese docs. Keep numbers, flows,
   endpoints, and states accurate with the code. Platform docs are in `docs/`;
   lamp-specific docs are in `devices/lamp/docs/`.

   **Platform docs** (`docs/` + `docs/vi/`):

   | Code area | English doc | Vietnamese doc |
   |-----------|-------------|----------------|
   | os-server, API, startup | `docs/os-server.md` | `docs/vi/os-server_vi.md` |
   | Setup flow, provisioning | `docs/setup-flow.md` | `docs/vi/setup-flow_vi.md` |
   | Web UI, configuration pages | `docs/web-ui.md` | `docs/vi/web-ui_vi.md` |
   | Flow Monitor (turn pipeline, JSONL, SSE) | `docs/flow-monitor.md` | `docs/vi/flow-monitor_vi.md` |
   | Overall structure | `docs/overview.md` | `docs/vi/overview_vi.md` |
   | MQTT, dispatch, publish | `docs/mqtt.md` | `docs/vi/mqtt_vi.md` |
   | OTA, bootstrap | `docs/bootstrap-ota.md` | `docs/vi/bootstrap-ota.md` |
   | Speech emotion recognition (SER) | `docs/speech-emotion.md` | `docs/vi/speech-emotion_vi.md` |
   | Realtime voice agent (HAL `realtime`, Gemini Live / OpenAI Realtime, delegate) | `docs/realtime-voice.md` | `docs/vi/realtime-voice_vi.md` |
   | Perception service (cloud DL inference), load balancer, encryption, models | `docs/perception-service.md` | `docs/vi/perception-service_vi.md` |
   | Hermes agent backend (`agent_runtime`, runtimes/hermes) | `docs/agentic/hermes.md` | `docs/vi/agentic/hermes_vi.md` |
   | PicoClaw agent backend (`agent_runtime`, runtimes/picoclaw, WebSocket) | `docs/agentic/picoclaw.md` | `docs/vi/agentic/picoclaw_vi.md` |
   | Adding/changing an agentic backend (AgentGateway contract, switch, install/presync, migration, skills, hooks, reset) | `docs/agentic/adding-agent-runtime.md` | `docs/vi/agentic/adding-agent-runtime_vi.md` |
   | Safety engine (SAFETY.md bounds, deterministic enforcement gate) | `docs/safety.md` | `docs/vi/safety_vi.md` |

   **Lamp-specific docs** (`devices/lamp/docs/` + `devices/lamp/docs/vi/`):

   | Code area | English doc | Vietnamese doc |
   |-----------|-------------|----------------|
   | LED, effects, states, animations | `devices/lamp/docs/led-control.md` | `devices/lamp/docs/vi/led-control_vi.md` |
   | Sensing behavior, sound escalation, reactions | `devices/lamp/docs/sensing-behavior.md` | `devices/lamp/docs/vi/sensing-behavior_vi.md` |
   | Sensing threshold tuning | `devices/lamp/docs/sensing-tuning.md` | `devices/lamp/docs/vi/sensing-tuning_vi.md` |
   | Habit tracking, pattern building, habit-aware nudge phrasing | `devices/lamp/docs/habit-tracking.md` | `devices/lamp/docs/vi/habit-tracking_vi.md` |
   | Vision tracking, object follow, servo track | `devices/lamp/docs/vision-tracking.md` | `devices/lamp/docs/vi/vision-tracking_vi.md` |
   | Physical controls (GPIO button, TTP223 touchpad, gestures, pet response) | `devices/lamp/docs/physical-controls.md` | `devices/lamp/docs/vi/physical-controls_vi.md` |
   | Autonomous Buddy (Mac companion app) | `integrations/companions/autonomous-buddy/docs/autonomous-buddy.md`, `integrations/companions/autonomous-buddy/docs/autonomous-buddy-mvp.md`, `integrations/companions/autonomous-buddy/docs/release-signing.md` | `integrations/companions/autonomous-buddy/docs/vi/autonomous-buddy_vi.md`, `integrations/companions/autonomous-buddy/docs/vi/autonomous-buddy-mvp_vi.md`, `integrations/companions/autonomous-buddy/docs/vi/release-signing_vi.md` |
   | Security test checklist | `devices/lamp/docs/security-test.md` | _(no vi version)_ |

2. **Comments in English** - Project standard.
3. **Code is the single source of truth** - Docs reflect code, not the other
   way around.
4. **Do not commit binary artifacts** - Version is injected via ldflags at
   build time.

See `docs/DEV-MULTI-IDE.md` for full conventions.

## Working Style

- The user reviews and commits by hand. Do not create commits unless explicitly
  asked.
- Work in small, reviewable chunks. When a task spans multiple concerns, split
  it by concern and verify each batch before moving to the next.
- Stay in scope. Flag unrelated issues instead of fixing them opportunistically.
- Verify with concrete evidence such as focused tests, builds, greps, `bash -n`,
  or compile checks. Report what was and was not verified.
- Do not "clean up" inherited drift such as unrelated gofmt churn, duplicate
  dependency metadata, or upstream-preserved style unless it is required for the
  task.
- Respond to the user in Vietnamese unless they request otherwise.
- Do not auto-deploy to devices. Default to repo changes plus local verification;
  any on-device SSH/SCP/restart step is opt-in and must be confirmed first.

## Parallel Work / Subagents

When work can be split across independent, file-scoped tasks, use available
parallelism instead of doing everything sequentially. In Codex, prefer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [autonomous-ai/autonomous-os](https://github.com/autonomous-ai/autonomous-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
