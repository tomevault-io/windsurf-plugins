---
trigger: always_on
description: - This repo is currently **Lynk**: an Android bubble/chat/voice endpoint that can route work to host-side agents or an on-device local model. Do not describe the product as OpenAgent or OpenClaw-only even though many classes and docs still carry older migration names.
---

# AGENTS.md

## Product Shape

- This repo is currently **Lynk**: an Android bubble/chat/voice endpoint that can route work to host-side agents or an on-device local model. Do not describe the product as OpenAgent or OpenClaw-only even though many classes and docs still carry older migration names.
- The supported backends are **OpenClaw**, **Hermes**, **Codex**, and **Local LiteRT-LM**. Android phone control is an optional tool target across these paths, not the default purpose of every request.
- The app has two user modes:
  - **Host bridge**: Android connects to the PC bridge over `/phone`; the bridge exposes a harness router for OpenClaw, Hermes, and Codex chat sessions.
  - **Local phone**: Android runs an imported `.litertlm` model on-device, emits the same `chat.*` timeline events locally, and can call Android/local app-private tools.
- OpenClaw is currently the default host harness and has the most Gateway-specific code. Hermes and Codex are real supported harnesses, not merely documentation footnotes. `PHONE_AGENT_USE_FALLBACK=1` is a deliberate bridge fallback path for testing.
- Local phone mode uses `local-litertlm`, supports Android phone tools and app-private workspace tools, and gates write/Termux developer tools behind settings. It is not yet a full desktop shell/git/build environment.

## Repo Map

- `pc/`: Node 24+, ESM, strict TypeScript bridge. Uses `zod`, `ws`, `tsx`, and the MCP SDK.
- `pc/src/bridge/`: WebSocket registration, HTTP APIs, host chat bridge, harness routing, audit/status, realtime session setup, task queueing, web search, pet catalog. Some files are still named `OpenClaw*` because OpenClaw was the first host path.
- `pc/src/bridge/harness/` and `pc/src/bridge/AgentHarness.ts`: host harness router for OpenClaw, Hermes, and Codex. This is the key source for model/session namespacing.
- `pc/src/dispatcher/`: adapter boundary for legacy `user_request` and realtime delegated tasks. `OpenClawSessionClient`, `HermesSessionClient`, and `CodexAppServerClient` all exist behind this boundary.
- `pc/src/mcp/`: `android-phone` MCP server and phone tool schemas. Keep these aligned with Android command execution.
- `pc/src/protocol/messages.ts`: canonical TypeScript source for WebSocket message validation, phone commands, MCP tool-name mapping, realtime tool names, model IDs, and reasoning options.
- `android/`: Kotlin Android app. Package/application id is `app.lynk`; source namespace is `dev.androidagent`.
- `android/app/src/main/java/dev/androidagent/net/`: bridge WebSocket client and inbound JSON parsing.
- `android/app/src/main/java/dev/androidagent/accessibility/`: Android command executor and screen observation.
- `android/app/src/main/java/dev/androidagent/overlay/`, `chat/`, `agentchat/`, `ui/`: bubble, panel, timeline, model/session controls, markdown/status rendering.
- `android/app/src/main/java/dev/androidagent/voice/`: OpenAI Realtime WebRTC state, tool-call accumulation, transcript normalization, tool-result events, transcription helpers.
- `android/app/src/main/java/dev/androidagent/localmodel/`: LiteRT-LM local mode, local sessions, local tool specs, app-private workspace tooling, Termux placeholder policy.
- `docs/`: setup, pairing, protocol, safety, OpenClaw migration notes, Codex docs, demo notes, and limitations. Some docs are still OpenClaw-skewed; verify source before copying that framing into new agent guidance.

## Commands

- PC install: `cd pc && npm install`
- Global bridge install after npm publish: `npm install -g lynk-bridge`
- Global bridge command: `lynk-bridge`
- Global host CLI: `lynk-bridge-host pairing --qr`, `lynk-bridge-host refresh`, `lynk-bridge-host mcp`, `lynk-bridge-host install-service`, `lynk-bridge-host uninstall-service`, `lynk-bridge-host service-status`, `lynk-bridge-host diagnostics`
- Global MCP server command: `lynk-bridge-mcp`
- PC host integration refresh: `cd pc && npm run host:refresh`
- PC host pairing payload: `cd pc && npm run host:pairing`
- PC host pairing QR: `cd pc && npm run host:pairing:qr`
- PC host service plan: `cd pc && npm run host:service-plan`
- PC host install/startup service: `cd pc && npm run host:install-service`
- PC host uninstall startup service: `cd pc && npm run host:uninstall-service`
- PC host service status: `cd pc && npm run host:service-status`
- PC host diagnostics: `cd pc && npm run host:diagnostics`
- PC type check: `cd pc && npm run check`
- PC build: `cd pc && npm run build`
- PC tests: `cd pc && npm test`
- PC bridge: `cd pc && npm run bridge` loads `pc/.env.local` via `tsx --env-file-if-exists`; shell env vars override it.
- PC MCP server: `cd pc && npm run mcp`
- Register available phone MCP integrations: `cd pc && npm run host:mcp`
- Register phone MCP with OpenClaw: `cd pc && npm run openclaw:mcp`
- Register phone MCP with Hermes: `cd pc && npm run hermes:mcp`
- Register phone MCP with Codex: `cd pc && npm run codex:mcp`
- Bridge health: `cd pc && npm run phone:health`
- USB test setup: `cd pc && npm run phone:usb`
- Tailscale pairing URL: `cd pc && npm run phone:tailscale`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [am-will/lynk](https://github.com/am-will/lynk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-30 -->
